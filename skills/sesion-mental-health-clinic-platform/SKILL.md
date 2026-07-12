---
name: sesion-mental-health-clinic-platform
description: AI-powered practice management platform for psychologists with scheduling, WhatsApp automation, AFIP billing, video calls, and Claude AI integration
triggers:
  - "set up sesion mental health platform"
  - "configure sesion for psychology clinic"
  - "integrate claude ai with sesion"
  - "configure whatsapp automation for clinic"
  - "set up afip electronic invoicing"
  - "configure livekit video consultations"
  - "deploy sesion workflow orchestration"
  - "configure mercadopago billing integration"
---

# Sesión Mental Health Clinic Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Architecture Overview

Sesión uses a microservices architecture with:
- **Backend**: NestJS (TypeScript) with Prisma ORM
- **Frontend**: SvelteKit 5 with Tailwind CSS
- **Database**: PostgreSQL with Prisma
- **AI**: Anthropic Claude (Opus 4.6 for deep analysis, Sonnet 4.6 for real-time)
- **Video**: LiveKit WebRTC infrastructure
- **Messaging**: Baileys (WhatsApp Web API)
- **Payments**: Stripe (international), MercadoPago (Argentina)
- **Caching**: Redis for sessions and real-time data

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 20.x
postgresql >= 15.x
redis >= 7.x
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies for backend
cd backend
npm install

# Install dependencies for frontend
cd ../frontend
npm install
```

### Environment Configuration

Create `.env` files in both backend and frontend directories:

**Backend `.env`:**

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic Claude AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_DIR="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
AFIP_ENVIRONMENT="testing" # or "production"

# Payment Providers
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"
STRIPE_WEBHOOK_SECRET="${STRIPE_WEBHOOK_SECRET}"
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
MERCADOPAGO_PUBLIC_KEY="${MERCADOPAGO_PUBLIC_KEY}"

# Application
JWT_SECRET="${JWT_SECRET}"
APP_URL="http://localhost:3000"
API_URL="http://localhost:4000"
```

**Frontend `.env`:**

```env
PUBLIC_API_URL="http://localhost:4000"
PUBLIC_LIVEKIT_WS_URL="wss://your-livekit-server.com"
```

### Database Setup

```bash
cd backend

# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate deploy

# Seed initial data (optional)
npx prisma db seed
```

### Start Development Servers

```bash
# Terminal 1 - Backend (NestJS)
cd backend
npm run start:dev

# Terminal 2 - Frontend (SvelteKit)
cd frontend
npm run dev

# Terminal 3 - Redis (if not running as service)
redis-server
```

## Core Module Usage

### 1. Appointment Scheduling Module

**Creating an appointment with conflict detection:**

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from './prisma.service';
import { AppointmentConflictError } from './errors';

@Injectable()
export class AppointmentService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number; // minutes
    type: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    const endTime = new Date(data.startTime.getTime() + data.duration * 60000);

    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        OR: [
          {
            startTime: { lte: data.startTime },
            endTime: { gt: data.startTime },
          },
          {
            startTime: { lt: endTime },
            endTime: { gte: endTime },
          },
        ],
        status: { not: 'cancelled' },
      },
    });

    if (conflicts.length > 0) {
      throw new AppointmentConflictError(
        'Practitioner has conflicting appointments'
      );
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime,
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Trigger WhatsApp confirmation
    await this.whatsappService.sendAppointmentConfirmation(appointment);

    return appointment;
  }

  async suggestAlternativeSlots(
    practitionerId: string,
    preferredDate: Date,
    duration: number
  ) {
    const startOfDay = new Date(preferredDate);
    startOfDay.setHours(0, 0, 0, 0);
    const endOfDay = new Date(preferredDate);
    endOfDay.setHours(23, 59, 59, 999);

    const bookedSlots = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: startOfDay, lte: endOfDay },
        status: { not: 'cancelled' },
      },
      select: { startTime: true, endTime: true },
      orderBy: { startTime: 'asc' },
    });

    // Get practitioner availability rules
    const availability = await this.prisma.practitionerAvailability.findMany({
      where: { practitionerId, dayOfWeek: preferredDate.getDay() },
    });

    const suggestions: Date[] = [];
    for (const slot of availability) {
      const slotStart = new Date(preferredDate);
      slotStart.setHours(slot.startHour, slot.startMinute, 0, 0);
      const slotEnd = new Date(preferredDate);
      slotEnd.setHours(slot.endHour, slot.endMinute, 0, 0);

      let currentTime = slotStart;
      while (currentTime.getTime() + duration * 60000 <= slotEnd.getTime()) {
        const isFree = !bookedSlots.some(
          (booked) =>
            currentTime >= booked.startTime && currentTime < booked.endTime
        );
        if (isFree) {
          suggestions.push(new Date(currentTime));
        }
        currentTime = new Date(currentTime.getTime() + 15 * 60000); // 15-min increments
      }
    }

    return suggestions.slice(0, 5); // Return top 5 suggestions
  }
}
```

### 2. WhatsApp Automation (Baileys Integration)

**Setting up WhatsApp session and sending automated reminders:**

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import * as cron from 'node-cron';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private socket: ReturnType<typeof makeWASocket>;

  async onModuleInit() {
    await this.initializeWhatsApp();
    this.scheduleReminders();
  }

  private async initializeWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_DIR || './whatsapp-sessions'
    );

    this.socket = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.socket.ev.on('creds.update', saveCreds);

    this.socket.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.initializeWhatsApp();
        }
      }
    });

    // Handle incoming messages
    this.socket.ev.on('messages.upsert', async (m) => {
      await this.handleIncomingMessage(m.messages[0]);
    });
  }

  async sendAppointmentConfirmation(appointment: {
    id: string;
    patient: { name: string; phone: string };
    startTime: Date;
    type: string;
  }) {
    const message = `Hola ${appointment.patient.name},\n\n` +
      `Tu sesión ${appointment.type} está confirmada para:\n` +
      `📅 ${appointment.startTime.toLocaleDateString('es-AR')}\n` +
      `🕒 ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n\n` +
      `Por favor responde *CONFIRMAR* para confirmar tu asistencia.\n\n` +
      `Sesión - Tu asistente de salud mental`;

    await this.socket.sendMessage(
      this.formatPhoneNumber(appointment.patient.phone),
      { text: message }
    );
  }

  private scheduleReminders() {
    // Run every hour
    cron.schedule('0 * * * *', async () => {
      const now = new Date();
      const in24Hours = new Date(now.getTime() + 24 * 60 * 60 * 1000);
      const in2Hours = new Date(now.getTime() + 2 * 60 * 60 * 1000);

      // 24-hour reminders
      const upcoming24h = await this.prisma.appointment.findMany({
        where: {
          startTime: {
            gte: new Date(in24Hours.getTime() - 30 * 60 * 1000), // 30-min window
            lte: new Date(in24Hours.getTime() + 30 * 60 * 1000),
          },
          status: 'scheduled',
          reminderSent24h: false,
        },
        include: { patient: true },
      });

      for (const apt of upcoming24h) {
        await this.sendReminder(apt, '24 horas');
        await this.prisma.appointment.update({
          where: { id: apt.id },
          data: { reminderSent24h: true },
        });
      }

      // 2-hour reminders
      const upcoming2h = await this.prisma.appointment.findMany({
        where: {
          startTime: {
            gte: new Date(in2Hours.getTime() - 15 * 60 * 1000),
            lte: new Date(in2Hours.getTime() + 15 * 60 * 1000),
          },
          status: 'scheduled',
          reminderSent2h: false,
        },
        include: { patient: true },
      });

      for (const apt of upcoming2h) {
        await this.sendReminder(apt, '2 horas');
        await this.prisma.appointment.update({
          where: { id: apt.id },
          data: { reminderSent2h: true },
        });
      }
    });
  }

  private async sendReminder(appointment: any, timeframe: string) {
    const message = `🔔 Recordatorio: Tu sesión es en ${timeframe}\n\n` +
      `📅 ${appointment.startTime.toLocaleDateString('es-AR')}\n` +
      `🕒 ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n\n` +
      `Nos vemos pronto!`;

    await this.socket.sendMessage(
      this.formatPhoneNumber(appointment.patient.phone),
      { text: message }
    );
  }

  private async handleIncomingMessage(message: WAMessage) {
    if (!message.message || message.key.fromMe) return;

    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text || '';
    const from = message.key.remoteJid;

    // Crisis detection keywords
    const crisisKeywords = ['suicidio', 'morir', 'acabar', 'terminar todo', 'crisis'];
    if (crisisKeywords.some(keyword => text.toLowerCase().includes(keyword))) {
      await this.handleCrisisEscalation(from, text);
      return;
    }

    // Appointment confirmation parsing
    if (text.toLowerCase().includes('confirmar')) {
      await this.handleAppointmentConfirmation(from);
    }
  }

  private async handleCrisisEscalation(phone: string, message: string) {
    // Alert practitioner immediately
    const patient = await this.prisma.patient.findUnique({
      where: { phone: this.normalizePhoneNumber(phone) },
      include: { practitioner: true },
    });

    if (patient?.practitioner) {
      // Send urgent notification to practitioner
      await this.socket.sendMessage(
        this.formatPhoneNumber(patient.practitioner.phone),
        {
          text: `🚨 ALERTA DE CRISIS 🚨\n\nPaciente: ${patient.name}\nMensaje: "${message}"\n\nContactar inmediatamente.`,
        }
      );
    }

    // Send supportive auto-response
    await this.socket.sendMessage(phone, {
      text: 'Tu mensaje es importante. Tu psicólogo será notificado inmediatamente. ' +
            'Si necesitas ayuda urgente, llama al 135 (Argentina - Línea de prevención del suicidio).',
    });
  }

  private formatPhoneNumber(phone: string): string {
    // Convert to WhatsApp JID format
    return `${phone.replace(/\D/g, '')}@s.whatsapp.net`;
  }

  private normalizePhoneNumber(jid: string): string {
    return jid.split('@')[0];
  }
}
```

### 3. AFIP Electronic Invoicing

**Generating AFIP-compliant invoices:**

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from './prisma.service';
import * as AfipSDK from '@afipsdk/afip.js';
import * as fs from 'fs';

@Injectable()
export class BillingService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new AfipSDK({
      CUIT: process.env.AFIP_CUIT,
      cert: fs.readFileSync(process.env.AFIP_CERT_PATH),
      key: fs.readFileSync(process.env.AFIP_KEY_PATH),
      production: process.env.AFIP_ENVIRONMENT === 'production',
    });
  }

  async generateInvoiceForSession(sessionId: string) {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        patient: true,
        practitioner: {
          include: { fiscalConfiguration: true },
        },
      },
    });

    if (!session) throw new Error('Session not found');

    const practitionerFiscal = session.practitioner.fiscalConfiguration;
    
    // Determine invoice type based on patient's fiscal category
    const invoiceType = this.determineInvoiceType(
      practitionerFiscal.category,
      session.patient.fiscalCategory
    );

    const invoiceData = {
      CantReg: 1,
      PtoVta: practitionerFiscal.puntoVenta,
      CbteTipo: invoiceType, // 1: A, 6: B, 11: C
      Concepto: 2, // Servicios
      DocTipo: 80, // CUIT (or 96 for DNI)
      DocNro: session.patient.cuit || session.patient.dni,
      CbteDesde: await this.getNextInvoiceNumber(practitionerFiscal.id, invoiceType),
      CbteHasta: await this.getNextInvoiceNumber(practitionerFiscal.id, invoiceType),
      CbteFch: this.formatDateAFIP(new Date()),
      ImpTotal: session.price,
      ImpTotConc: 0,
      ImpNeto: invoiceType === 1 ? session.price / 1.21 : session.price, // Neto for type A
      ImpOpEx: 0,
      ImpIVA: invoiceType === 1 ? session.price - (session.price / 1.21) : 0,
      ImpTrib: 0,
      MonId: 'PES', // Pesos
      MonCotiz: 1,
      Iva: invoiceType === 1 ? [
        {
          Id: 5, // 21%
          BaseImp: session.price / 1.21,
          Importe: session.price - (session.price / 1.21),
        }
      ] : null,
    };

    // Request CAE (Código de Autorización Electrónico) from AFIP
    const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    if (result.CAE) {
      const invoice = await this.prisma.invoice.create({
        data: {
          sessionId,
          patientId: session.patientId,
          practitionerId: session.practitionerId,
          type: invoiceType,
          number: invoiceData.CbteDesde,
          amount: session.price,
          cae: result.CAE,
          caeExpiration: this.parseAFIPDate(result.CAEFchVto),
          afipResponse: JSON.stringify(result),
        },
      });

      // Send invoice via WhatsApp
      await this.whatsappService.sendInvoice(invoice);

      return invoice;
    } else {
      throw new Error(`AFIP error: ${result.Errors}`);
    }
  }

  private determineInvoiceType(
    practitionerCategory: string,
    patientCategory?: string
  ): number {
    // Simplified logic - real implementation would be more complex
    if (practitionerCategory === 'RESPONSABLE_INSCRIPTO') {
      if (patientCategory === 'RESPONSABLE_INSCRIPTO') return 1; // A
      if (patientCategory === 'MONOTRIBUTISTA') return 6; // B
      return 6; // B (for consumers)
    }
    if (practitionerCategory === 'MONOTRIBUTISTA') return 11; // C
    return 11; // C (default)
  }

  private async getNextInvoiceNumber(
    fiscalConfigId: string,
    invoiceType: number
  ): Promise<number> {
    const lastInvoice = await this.prisma.invoice.findFirst({
      where: { practitioner: { fiscalConfigurationId: fiscalConfigId }, type: invoiceType },
      orderBy: { number: 'desc' },
    });
    return (lastInvoice?.number || 0) + 1;
  }

  private formatDateAFIP(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  private parseAFIPDate(dateStr: string): Date {
    const year = parseInt(dateStr.substring(0, 4));
    const month = parseInt(dateStr.substring(4, 6)) - 1;
    const day = parseInt(dateStr.substring(6, 8));
    return new Date(year, month, day);
  }

  async generateMonthlyReport(practitionerId: string, year: number, month: number) {
    const startDate = new Date(year, month - 1, 1);
    const endDate = new Date(year, month, 0);

    const invoices = await this.prisma.invoice.findMany({
      where: {
        practitionerId,
        createdAt: { gte: startDate, lte: endDate },
      },
      include: { patient: true },
    });

    const summary = {
      totalInvoiced: invoices.reduce((sum, inv) => sum + inv.amount, 0),
      invoicesByType: {
        A: invoices.filter((inv) => inv.type === 1).length,
        B: invoices.filter((inv) => inv.type === 6).length,
        C: invoices.filter((inv) => inv.type === 11).length,
      },
      ivaCollected: invoices
        .filter((inv) => inv.type === 1)
        .reduce((sum, inv) => sum + (inv.amount - inv.amount / 1.21), 0),
    };

    return summary;
  }
}
```

### 4. LiveKit Video Consultations

**Creating and managing video sessions:**

```typescript
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from './prisma.service';

@Injectable()
export class VideoService {
  constructor(private prisma: PrismaService) {}

  async createVideoSession(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment) throw new Error('Appointment not found');
    if (appointment.type !== 'virtual') {
      throw new Error('Appointment is not virtual');
    }

    const roomName = `session-${appointmentId}`;

    const videoSession = await this.prisma.videoSession.create({
      data: {
        appointmentId,
        roomName,
        status: 'waiting',
      },
    });

    return {
      sessionId: videoSession.id,
      roomName,
      patientToken: this.generateToken(roomName, appointment.patient.id, 'patient'),
      practitionerToken: this.generateToken(
        roomName,
        appointment.practitioner.id,
        'practitioner'
      ),
    };
  }

  private generateToken(roomName: string, userId: string, role: string): string {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: userId,
        metadata: JSON.stringify({ role }),
      }
    );

    at.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
      // Practitioners can record, patients cannot
      canUpdateOwnMetadata: role === 'practitioner',
    });

    return at.toJwt();
  }

  async startRecording(sessionId: string, practitionerId: string) {
    const session = await this.prisma.videoSession.findUnique({
      where: { id: sessionId },
      include: { appointment: true },
    });

    if (session.appointment.practitionerId !== practitionerId) {
      throw new Error('Unauthorized');
    }

    // Verify double consent
    const consent = await this.prisma.recordingConsent.findFirst({
      where: {
        videoSessionId: sessionId,
        patientConsent: true,
        practitionerConsent: true,
      },
    });

    if (!consent) {
      throw new Error('Recording consent not obtained from both parties');
    }

    await this.prisma.videoSession.update({
      where: { id: sessionId },
      data: { recordingStarted: new Date() },
    });

    // Actual LiveKit recording start would happen via webhook
    return { recording: true };
  }

  async endSession(sessionId: string) {
    const session = await this.prisma.videoSession.update({
      where: { id: sessionId },
      data: {
        status: 'ended',
        endedAt: new Date(),
      },
    });

    return session;
  }
}
```

**Frontend Svelte component for video session:**

```svelte
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent, Track } from 'livekit-client';
  
  export let token: string;
  export let wsUrl: string;
  export let role: 'patient' | 'practitioner';

  let room: Room;
  let videoElement: HTMLVideoElement;
  let remoteVideoElement: HTMLVideoElement;
  let isConnected = false;
  let isMuted = false;
  let isVideoOff = false;

  onMount(async () => {
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: { width: 1280, height: 720 },
      },
    });

    room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
      if (track.kind === Track.Kind.Video) {
        track.attach(remoteVideoElement);
      }
    });

    room.on(RoomEvent.Disconnected, () => {
      isConnected = false;
    });

    await room.connect(wsUrl, token);
    
    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
    const videoTrack = room.localParticipant.videoTrackPublications.values().next().value;
    if (videoTrack) {
      videoTrack.track?.attach(videoElement);
    }

    isConnected = true;
  });

  onDestroy(() => {
    room?.disconnect();
  });

  async function toggleMute() {
    await room.localParticipant.setMicrophoneEnabled(isMuted);
    isMuted = !isMuted;
  }

  async function toggleVideo() {
    await room.localParticipant.setCameraEnabled(isVideoOff);
    isVideoOff = !isVideoOff;
  }

  async function endCall() {
    await fetch(`/api/video/end-session`, { method: 'POST' });
    room.disconnect();
  }
</script>

<div class="video-container">
  <div class="remote-video">
    <video bind:this={remoteVideoElement} autoplay playsinline />
  </div>
  
  <div class="local-video">
    <video bind:this={videoElement} autoplay playsinline muted />
  </div>

  <div class="controls">
    <button on:click={toggleMute} class:active={!isMuted}>
      {isMuted ? '🔇' : '🎤'}
    </button>
    <button on:click={toggleVideo} class:active={!isVideoOff}>
      {isVideoOff ? '📹' : '📷'}
    </button>
    <button on:click={endCall} class="end-call">Finalizar</button>
  </div>

  {#if !isConnected}
    <div class="loading">Conectando...</div>
  {/if}
</div>

<style>
  .video-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: #000;
  }

  .remote-video {
    width: 100%;
    height: 100%;
  }

  .remote-video video {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .local-video {
    position: absolute;
    bottom: 80px;
    right: 20px;
    width: 200px;
    height: 150px;
    border-radius: 8px;
    overflow: hidden;
    border: 2px solid #fff;
  }

  .local-video video {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .controls {
    position: absolute;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 12px;
  }

  .controls button {
    padding: 12px 24px;
    border-radius: 24px;
    border: none;
    background: rgba(255, 255, 255, 0.2);
    color: white;
    cursor: pointer;
    transition: background 0.2s;
  }

  .controls button:hover {
    background: rgba(255, 255, 255, 0.3);
  }

  .controls button.active {
    background: #4CAF50;
  }

  .controls button.end-call {
    background: #f44336;
  }

  .loading {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: white;
    font-size: 24px;
  }
</style>
```

### 5. Claude AI Integration

**Clinical note summarization with Claude Opus:**

```typescript
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from './prisma.service';

@Injectable()
export class AIService {
  private anthropic: Anthropic;

  constructor(private prisma: PrismaService)
