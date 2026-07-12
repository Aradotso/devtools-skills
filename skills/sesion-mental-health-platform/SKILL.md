---
name: sesion-mental-health-platform
description: SaaS platform for Argentine psychology clinics with intelligent scheduling, WhatsApp automation, AFIP billing, video calls, and Claude AI integration
triggers:
  - how do I set up Sesión for my psychology clinic
  - integrate WhatsApp automation with Sesión
  - configure AFIP electronic invoicing in Sesión
  - set up video consultations with Sesión
  - use Claude AI models in Sesión workflow
  - implement patient appointment scheduling in Sesión
  - configure MercadoPago payment integration
  - deploy Sesión platform on my infrastructure
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**Sesión** is a comprehensive SaaS orchestration platform designed specifically for Argentine psychology clinics and independent practitioners. It unifies:

- **Intelligent agenda management** (appointment scheduling with conflict detection)
- **WhatsApp automation** via Baileys (appointment reminders, confirmations, surveys)
- **AFIP-compliant electronic invoicing** (facturas tipo A, B, C, M)
- **Secure video consultations** via LiveKit (WebRTC with E2E encryption)
- **AI-powered workflows** using Claude Opus 4.6 and Sonnet 4.6 (clinical note summarization, patient journey analysis)

The platform is built with **NestJS** (backend microservices), **SvelteKit 5** (frontend), **Prisma** (ORM), **PostgreSQL** (data), and **TypeScript** throughout.

## Technology Stack

- **Backend**: NestJS microservices architecture
- **Frontend**: SvelteKit 5 with Svelte 5 Runes API
- **Database**: PostgreSQL with Prisma ORM
- **WhatsApp**: Baileys library for WhatsApp Web automation
- **Video**: LiveKit for WebRTC video consultations
- **AI**: Anthropic Claude (Opus 4.6 for deep analysis, Sonnet 4.6 for real-time)
- **Payments**: Stripe (international) + MercadoPago (Argentina)
- **Styling**: TailwindCSS

## Project Structure

```
psique-workflow-clinic/
├── apps/
│   ├── api/                 # NestJS backend services
│   │   ├── src/
│   │   │   ├── agenda/      # Appointment scheduling module
│   │   │   ├── whatsapp/    # WhatsApp automation module
│   │   │   ├── billing/     # AFIP invoicing module
│   │   │   ├── video/       # LiveKit video module
│   │   │   ├── ai/          # Claude AI orchestration
│   │   │   └── patients/    # Patient management module
│   ├── web/                 # SvelteKit frontend
│   │   ├── src/
│   │   │   ├── routes/      # SvelteKit file-based routing
│   │   │   ├── lib/         # Shared components and utilities
│   │   │   └── stores/      # Svelte stores for state management
├── packages/
│   ├── database/            # Prisma schema and migrations
│   └── shared/              # Shared TypeScript types
└── prisma/
    └── schema.prisma        # Database schema definition
```

## Installation & Setup

### Prerequisites

```bash
# Required tools
node >= 18.x
pnpm >= 8.x
postgresql >= 14.x
redis >= 7.x
```

### Initial Setup

```bash
# Clone repository
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create `.env` in the project root:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion?schema=public"

# Redis (for caching and sessions)
REDIS_URL="redis://localhost:6379"

# Anthropic Claude API
ANTHROPIC_API_KEY="your_anthropic_key_here"
ANTHROPIC_OPUS_MODEL="claude-opus-4.6"
ANTHROPIC_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your_webhook_secret"

# LiveKit Video
LIVEKIT_API_KEY="your_livekit_api_key"
LIVEKIT_API_SECRET="your_livekit_api_secret"
LIVEKIT_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="your_clinic_cuit"
AFIP_CERTIFICATE_PATH="./certs/afip.crt"
AFIP_PRIVATE_KEY_PATH="./certs/afip.key"
AFIP_PRODUCTION="false"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="your_mp_token"
STRIPE_SECRET_KEY="your_stripe_key"

# Application
JWT_SECRET="your_jwt_secret"
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
```

### Database Setup

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate dev

# Seed database (optional)
pnpm prisma db seed
```

### Start Development Servers

```bash
# Start all services (API + Web)
pnpm dev

# Or start individually
pnpm --filter api dev      # Backend on http://localhost:3000
pnpm --filter web dev      # Frontend on http://localhost:5173
```

## Core Module Usage

### 1. Agenda Module (Appointment Scheduling)

**NestJS Service Example**:

```typescript
// apps/api/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { Appointment, Prisma } from '@prisma/client';

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number; // minutes
    type: 'presencial' | 'virtual' | 'evaluacion';
  }): Promise<Appointment> {
    // Check for conflicts
    const conflicts = await this.findConflicts(
      data.practitionerId,
      data.startTime,
      data.duration
    );

    if (conflicts.length > 0) {
      throw new Error('Time slot conflicts with existing appointment');
    }

    return this.prisma.appointment.create({
      data: {
        patientId: data.patientId,
        practitionerId: data.practitionerId,
        startTime: data.startTime,
        endTime: new Date(data.startTime.getTime() + data.duration * 60000),
        type: data.type,
        status: 'scheduled',
      },
    });
  }

  async findConflicts(
    practitionerId: string,
    startTime: Date,
    duration: number
  ): Promise<Appointment[]> {
    const endTime = new Date(startTime.getTime() + duration * 60000);

    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: { not: 'cancelled' },
        OR: [
          {
            AND: [
              { startTime: { lte: startTime } },
              { endTime: { gt: startTime } },
            ],
          },
          {
            AND: [
              { startTime: { lt: endTime } },
              { endTime: { gte: endTime } },
            ],
          },
        ],
      },
    });
  }

  async getAvailableSlots(
    practitionerId: string,
    date: Date,
    sessionDuration: number = 45
  ): Promise<Date[]> {
    const dayStart = new Date(date);
    dayStart.setHours(8, 0, 0, 0);
    const dayEnd = new Date(date);
    dayEnd.setHours(20, 0, 0, 0);

    const bookedSlots = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: dayStart, lt: dayEnd },
        status: { not: 'cancelled' },
      },
      orderBy: { startTime: 'asc' },
    });

    const availableSlots: Date[] = [];
    let currentTime = new Date(dayStart);

    while (currentTime < dayEnd) {
      const slotEnd = new Date(currentTime.getTime() + sessionDuration * 60000);
      const hasConflict = bookedSlots.some(
        (apt) => apt.startTime < slotEnd && apt.endTime > currentTime
      );

      if (!hasConflict) {
        availableSlots.push(new Date(currentTime));
      }

      currentTime = new Date(currentTime.getTime() + 15 * 60000); // 15min increments
    }

    return availableSlots;
  }
}
```

**SvelteKit Frontend Component**:

```svelte
<!-- apps/web/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  
  let appointments = writable([]);
  let selectedDate = $state(new Date());
  let availableSlots = $state([]);

  async function fetchAppointments(date: Date) {
    const response = await fetch(
      `/api/appointments?date=${date.toISOString()}`
    );
    const data = await response.json();
    appointments.set(data);
  }

  async function fetchAvailableSlots(practitionerId: string, date: Date) {
    const response = await fetch(
      `/api/agenda/available-slots?practitionerId=${practitionerId}&date=${date.toISOString()}`
    );
    availableSlots = await response.json();
  }

  async function bookAppointment(slotTime: Date) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        patientId: 'patient-123',
        practitionerId: 'practitioner-456',
        startTime: slotTime,
        duration: 45,
        type: 'presencial'
      })
    });

    if (response.ok) {
      await fetchAppointments(selectedDate);
    }
  }

  onMount(() => {
    fetchAppointments(selectedDate);
  });
</script>

<div class="agenda-container">
  <h1>Agenda del día</h1>
  
  <div class="calendar">
    <input 
      type="date" 
      bind:value={selectedDate}
      onchange={() => fetchAppointments(selectedDate)}
    />
  </div>

  <div class="appointments-list">
    {#each $appointments as appointment}
      <div class="appointment-card">
        <span class="time">{appointment.startTime.toLocaleTimeString('es-AR')}</span>
        <span class="patient">{appointment.patient.name}</span>
        <span class="type badge-{appointment.type}">{appointment.type}</span>
      </div>
    {/each}
  </div>

  <div class="available-slots">
    <h2>Horarios disponibles</h2>
    {#each availableSlots as slot}
      <button onclick={() => bookAppointment(slot)}>
        {slot.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}
      </button>
    {/each}
  </div>
</div>
```

### 2. WhatsApp Automation Module

**Baileys Integration Service**:

```typescript
// apps/api/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.initializeWhatsApp();
  }

  async initializeWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('creds.update', saveCreds);

    this.sock.ev.on('connection.update', (update) => {
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

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentReminder(
    phoneNumber: string,
    appointmentData: {
      patientName: string;
      practitionerName: string;
      date: Date;
      type: string;
    }
  ) {
    const formattedPhone = `549${phoneNumber}@s.whatsapp.net`;
    const message = `Hola ${appointmentData.patientName}! 👋

Recordatorio de sesión con ${appointmentData.practitionerName}:
📅 ${appointmentData.date.toLocaleDateString('es-AR', {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric',
    })}
⏰ ${appointmentData.date.toLocaleTimeString('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
    })}
📍 ${appointmentData.type === 'virtual' ? 'Videollamada' : 'Presencial'}

Responde:
1️⃣ CONFIRMO para confirmar
2️⃣ CANCELAR para cancelar

Gracias! 💙`;

    await this.sock.sendMessage(formattedPhone, { text: message });

    // Log message in database
    await this.prisma.whatsAppMessage.create({
      data: {
        phoneNumber,
        message,
        type: 'appointment_reminder',
        status: 'sent',
      },
    });
  }

  async handleIncomingMessage(message: WAMessage) {
    if (!message.message?.conversation) return;

    const phoneNumber = message.key.remoteJid?.replace('@s.whatsapp.net', '');
    const text = message.message.conversation.toLowerCase().trim();

    // Crisis keyword detection
    const crisisKeywords = [
      'suicidio',
      'quiero morir',
      'no puedo más',
      'ayuda urgente',
    ];
    if (crisisKeywords.some((keyword) => text.includes(keyword))) {
      await this.escalateCrisis(phoneNumber, text);
      return;
    }

    // Appointment confirmation
    if (text === 'confirmo' || text === '1') {
      await this.confirmAppointment(phoneNumber);
    } else if (text === 'cancelar' || text === '2') {
      await this.cancelAppointment(phoneNumber);
    }
  }

  async escalateCrisis(phoneNumber: string, message: string) {
    // Alert practitioner immediately
    const patient = await this.prisma.patient.findUnique({
      where: { phoneNumber },
      include: { practitioner: true },
    });

    if (patient?.practitioner.emergencyPhone) {
      await this.sock.sendMessage(
        `549${patient.practitioner.emergencyPhone}@s.whatsapp.net`,
        {
          text: `🚨 ALERTA DE CRISIS 🚨\n\nPaciente: ${patient.name}\nMensaje: ${message}\n\nContactar urgentemente.`,
        }
      );
    }
  }

  async confirmAppointment(phoneNumber: string) {
    const appointment = await this.prisma.appointment.findFirst({
      where: {
        patient: { phoneNumber },
        status: 'scheduled',
        startTime: { gte: new Date() },
      },
      orderBy: { startTime: 'asc' },
    });

    if (appointment) {
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { status: 'confirmed' },
      });

      await this.sock.sendMessage(`549${phoneNumber}@s.whatsapp.net`, {
        text: '✅ Sesión confirmada. Te esperamos! 💙',
      });
    }
  }

  async cancelAppointment(phoneNumber: string) {
    // Implementation similar to confirmAppointment
  }
}
```

**Schedule Automated Reminders**:

```typescript
// apps/api/src/agenda/agenda-reminders.service.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AgendaRemindersService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: new Date(tomorrow.getTime() - 30 * 60000), // 24h ± 30min window
          lte: new Date(tomorrow.getTime() + 30 * 60000),
        },
        status: 'scheduled',
        reminderSent24h: false,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.patient.phoneNumber, {
        patientName: apt.patient.name,
        practitionerName: apt.practitioner.name,
        date: apt.startTime,
        type: apt.type,
      });

      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent24h: true },
      });
    }
  }

  @Cron(CronExpression.EVERY_10_MINUTES)
  async send2HourReminders() {
    // Similar implementation for 2-hour reminders
  }
}
```

### 3. AFIP Electronic Invoicing

**AFIP Service**:

```typescript
// apps/api/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as Afip from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      res_folder: process.env.AFIP_CERTIFICATE_PATH,
      ta_folder: './storage/afip-ta',
      production: process.env.AFIP_PRODUCTION === 'true',
    });
  }

  async generateInvoice(data: {
    patientId: string;
    appointmentId: string;
    amount: number;
    invoiceType: 'A' | 'B' | 'C' | 'M';
  }) {
    const patient = await this.prisma.patient.findUnique({
      where: { id: data.patientId },
    });

    const invoiceData = {
      CantReg: 1, // Cantidad de registros
      PtoVta: 1, // Punto de venta
      CbteTipo: this.getInvoiceTypeCode(data.invoiceType), // Tipo de comprobante
      Concepto: 2, // Servicios
      DocTipo: patient.documentType === 'DNI' ? 96 : 80, // 96=DNI, 80=CUIT
      DocNro: parseInt(patient.documentNumber),
      CbteDesde: await this.getNextInvoiceNumber(data.invoiceType),
      CbteHasta: await this.getNextInvoiceNumber(data.invoiceType),
      CbteFch: this.formatAfipDate(new Date()),
      ImpTotal: data.amount,
      ImpTotConc: 0, // Importe neto no gravado
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES', // Pesos argentinos
      MonCotiz: 1,
    };

    const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    // Store invoice in database
    const invoice = await this.prisma.invoice.create({
      data: {
        patientId: data.patientId,
        appointmentId: data.appointmentId,
        amount: data.amount,
        type: data.invoiceType,
        afipCae: result.CAE,
        afipCaeExpiration: new Date(result.CAEFchVto),
        afipVoucherNumber: result.CbteDesde,
        status: 'issued',
      },
    });

    return invoice;
  }

  private getInvoiceTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type];
  }

  private async getNextInvoiceNumber(type: string): Promise<number> {
    const lastInvoice = await this.prisma.invoice.findFirst({
      where: { type },
      orderBy: { afipVoucherNumber: 'desc' },
    });
    return (lastInvoice?.afipVoucherNumber || 0) + 1;
  }

  private formatAfipDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }
}
```

**Invoice Generation Endpoint**:

```typescript
// apps/api/src/billing/billing.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { AfipService } from './afip.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Controller('billing')
export class BillingController {
  constructor(
    private afipService: AfipService,
    private whatsapp: WhatsAppService
  ) {}

  @Post('generate-invoice')
  async generateInvoice(
    @Body()
    data: {
      patientId: string;
      appointmentId: string;
      amount: number;
      invoiceType: 'A' | 'B' | 'C' | 'M';
    }
  ) {
    // Generate AFIP invoice
    const invoice = await this.afipService.generateInvoice(data);

    // Send invoice via WhatsApp
    const patient = await this.prisma.patient.findUnique({
      where: { id: data.patientId },
    });

    const invoiceUrl = `${process.env.APP_URL}/invoices/${invoice.id}/pdf`;

    await this.whatsapp.sendMessage(patient.phoneNumber, {
      text: `Factura generada 💳\n\nMonto: $${data.amount}\nCAE: ${invoice.afipCae}\n\nDescargar: ${invoiceUrl}`,
    });

    return invoice;
  }
}
```

### 4. LiveKit Video Consultations

**Video Service**:

```typescript
// apps/api/src/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  private apiKey = process.env.LIVEKIT_API_KEY;
  private apiSecret = process.env.LIVEKIT_API_SECRET;
  private livekitUrl = process.env.LIVEKIT_URL;

  async createVideoRoom(appointmentId: string): Promise<string> {
    return `session-${appointmentId}`;
  }

  async generateAccessToken(
    roomName: string,
    participantName: string,
    participantId: string
  ): Promise<string> {
    const token = new AccessToken(this.apiKey, this.apiSecret, {
      identity: participantId,
      name: participantName,
    });

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
    });

    return token.toJwt();
  }

  getLivekitUrl(): string {
    return this.livekitUrl;
  }
}
```

**SvelteKit Video Component**:

```svelte
<!-- apps/web/src/routes/session/[appointmentId]/video/+page.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent, Track } from 'livekit-client';
  import { page } from '$app/stores';

  let room: Room;
  let localVideoElement: HTMLVideoElement;
  let remoteVideoElement: HTMLVideoElement;
  let isAudioMuted = $state(false);
  let isVideoMuted = $state(false);

  onMount(async () => {
    const appointmentId = $page.params.appointmentId;
    
    // Fetch LiveKit credentials
    const response = await fetch(`/api/video/token/${appointmentId}`);
    const { token, url } = await response.json();

    // Initialize room
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
    });

    // Handle track subscriptions
    room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
      if (track.kind === Track.Kind.Video) {
        track.attach(remoteVideoElement);
      }
    });

    // Connect to room
    await room.connect(url, token);

    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
    const videoTrack = room.localParticipant.videoTrackPublications.values().next().value?.track;
    if (videoTrack) {
      videoTrack.attach(localVideoElement);
    }
  });

  async function toggleAudio() {
    if (room) {
      await room.localParticipant.setMicrophoneEnabled(!isAudioMuted);
      isAudioMuted = !isAudioMuted;
    }
  }

  async function toggleVideo() {
    if (room) {
      await room.localParticipant.setCameraEnabled(!isVideoMuted);
      isVideoMuted = !isVideoMuted;
    }
  }

  async function endCall() {
    if (room) {
      await room.disconnect();
      window.location.href = '/dashboard';
    }
  }

  onDestroy(() => {
    if (room) {
      room.disconnect();
    }
  });
</script>

<div class="video-consultation">
  <div class="video-grid">
    <div class="remote-video">
      <video bind:this={remoteVideoElement} autoplay playsinline />
    </div>
    <div class="local-video">
      <video bind:this={localVideoElement} autoplay playsinline muted />
    </div>
  </div>

  <div class="controls">
    <button class="control-btn" onclick={toggleAudio}>
      {isAudioMuted ? '🔇' : '🎤'}
    </button>
    <button class="control-btn" onclick={toggleVideo}>
      {isVideoMuted ? '📹' : '📷'}
    </button>
    <button class="control-btn end-call" onclick={endCall}>
      📞 Finalizar
    </button>
  </div>
</div>

<style>
  .video-grid {
    display: grid;
    grid-template-columns: 1fr;
    height: calc(100vh - 100px);
  }
  .remote-video {
    position: relative;
  }
  .local-video {
    position: absolute;
    bottom: 20px;
    right: 20px;
    width: 200px;
    height: 150px;
  }
  video {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 8px;
  }
  .controls {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 12px;
  }
  .control-btn {
    padding: 12px 24px;
    border-radius: 50px;
    border: none;
    background: #333;
    color: white;
    cursor: pointer;
  }
  .end-call {
    background: #e74c3c;
  }
</style>
```

### 5. Claude AI Integration

**AI Orchestration Service**:

```typescript
// apps/api/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;

  constructor(private prisma: PrismaService) {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(
    patientId: string,
    sessionIds: string[]
  ): Promise<string> {
    const sessions = await
