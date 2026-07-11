---
name: sesion-mental-health-platform
description: SaaS platform for psychology clinics with intelligent scheduling, WhatsApp automation, AFIP billing, video consultations, and Claude AI integration
triggers:
  - "set up Sesión mental health platform"
  - "integrate WhatsApp automation for clinic appointments"
  - "configure AFIP electronic invoicing for psychology practice"
  - "implement video consultation with LiveKit"
  - "use Claude AI for clinical note summarization"
  - "create appointment scheduling system with Prisma"
  - "build psychology clinic management dashboard"
  - "configure Mercado Pago payment integration"
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, automated WhatsApp messaging, AFIP-compliant billing, secure video consultations, and AI-powered clinical workflows using Claude Opus 4.6 and Sonnet 4.6.

## Architecture Overview

The platform is built as a microservices ecosystem with:
- **Backend**: NestJS with TypeScript
- **Frontend**: SvelteKit 5 with Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **WhatsApp**: Baileys library for automation
- **Video**: LiveKit for WebRTC consultations
- **AI**: Anthropic Claude models (Opus & Sonnet)
- **Payments**: Stripe + Mercado Pago
- **Caching**: Redis for session management

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 18.x
pnpm >= 8.x
postgresql >= 14.x
redis >= 7.x
```

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Setup environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="your-key-here"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your-webhook-secret"

# LiveKit Video
LIVEKIT_API_KEY="your-livekit-key"
LIVEKIT_API_SECRET="your-livekit-secret"
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# Payments
STRIPE_SECRET_KEY="sk_test_..."
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."

# AFIP (Argentina Tax Authority)
AFIP_CUIT="your-tax-id"
AFIP_CERT_PATH="./certs/afip.crt"
AFIP_KEY_PATH="./certs/afip.key"
AFIP_ENVIRONMENT="testing" # or "production"
```

### Database Setup

```bash
# Run Prisma migrations
pnpm prisma migrate dev

# Seed initial data
pnpm prisma db seed

# Generate Prisma client
pnpm prisma generate
```

## Core Modules

### 1. Appointment Scheduling

**Prisma Schema** (`prisma/schema.prisma`):

```prisma
model Appointment {
  id            String   @id @default(cuid())
  patientId     String
  practitionerId String
  startTime     DateTime
  endTime       DateTime
  type          SessionType
  status        AppointmentStatus @default(SCHEDULED)
  roomId        String?
  notes         String?
  
  patient       Patient @relation(fields: [patientId], references: [id])
  practitioner  Practitioner @relation(fields: [practitionerId], references: [id])
  room          Room? @relation(fields: [roomId], references: [id])
  invoice       Invoice?
  
  @@index([practitionerId, startTime])
  @@index([patientId, startTime])
}

enum SessionType {
  PRESENCIAL
  VIRTUAL
  EVALUACION
  PAREJA
  FAMILIAR
}

enum AppointmentStatus {
  SCHEDULED
  CONFIRMED
  IN_PROGRESS
  COMPLETED
  CANCELLED
  NO_SHOW
}
```

**Appointment Service** (`src/appointments/appointments.service.ts`):

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { Appointment, SessionType } from '@prisma/client';
import { DateTime } from 'luxon';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    type: SessionType;
  }): Promise<Appointment> {
    // Calculate end time based on session type
    const duration = this.getSessionDuration(data.type);
    const endTime = DateTime.fromJSDate(data.startTime)
      .plus({ minutes: duration })
      .toJSDate();

    // Check for conflicts
    const conflicts = await this.findConflicts(
      data.practitionerId,
      data.startTime,
      endTime
    );

    if (conflicts.length > 0) {
      throw new Error('Schedule conflict detected');
    }

    return this.prisma.appointment.create({
      data: {
        ...data,
        endTime,
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });
  }

  private getSessionDuration(type: SessionType): number {
    const durations = {
      PRESENCIAL: 45,
      VIRTUAL: 45,
      EVALUACION: 60,
      PAREJA: 90,
      FAMILIAR: 90,
    };
    return durations[type];
  }

  async findConflicts(
    practitionerId: string,
    startTime: Date,
    endTime: Date
  ): Promise<Appointment[]> {
    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: { notIn: ['CANCELLED', 'NO_SHOW'] },
        OR: [
          {
            startTime: { lte: startTime },
            endTime: { gt: startTime },
          },
          {
            startTime: { lt: endTime },
            endTime: { gte: endTime },
          },
        ],
      },
    });
  }

  async suggestAlternativeSlots(
    practitionerId: string,
    preferredDate: Date,
    type: SessionType
  ) {
    const duration = this.getSessionDuration(type);
    const availability = await this.prisma.practitionerAvailability.findMany({
      where: { practitionerId },
    });

    // Logic to suggest 3-5 alternative time slots
    const suggestions = [];
    const dayOfWeek = DateTime.fromJSDate(preferredDate).weekday;
    
    for (const slot of availability) {
      if (slot.dayOfWeek === dayOfWeek) {
        const potentialStart = DateTime.fromJSDate(preferredDate)
          .set({ hour: slot.startHour, minute: slot.startMinute });
        const potentialEnd = potentialStart.plus({ minutes: duration });

        const conflicts = await this.findConflicts(
          practitionerId,
          potentialStart.toJSDate(),
          potentialEnd.toJSDate()
        );

        if (conflicts.length === 0) {
          suggestions.push({
            startTime: potentialStart.toJSDate(),
            endTime: potentialEnd.toJSDate(),
          });
        }
      }
    }

    return suggestions.slice(0, 5);
  }
}
```

### 2. WhatsApp Automation

**WhatsApp Service** (`src/whatsapp/whatsapp.service.ts`):

```typescript
import { Injectable } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService {
  private sock: any;

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('creds.update', saveCreds);
    this.sock.ev.on('connection.update', this.handleConnectionUpdate);
    this.sock.ev.on('messages.upsert', this.handleIncomingMessage);
  }

  private handleConnectionUpdate = async (update: any) => {
    const { connection, lastDisconnect } = update;
    if (connection === 'close') {
      const shouldReconnect =
        (lastDisconnect?.error as Boom)?.output?.statusCode !==
        DisconnectReason.loggedOut;
      if (shouldReconnect) {
        await this.initialize();
      }
    }
  };

  private handleIncomingMessage = async ({ messages }: { messages: WAMessage[] }) => {
    const msg = messages[0];
    if (!msg.message) return;

    const text = msg.message.conversation || 
                 msg.message.extendedTextMessage?.text || '';
    
    // Crisis detection
    const crisisKeywords = ['suicidio', 'morir', 'crisis', 'emergencia'];
    if (crisisKeywords.some(keyword => text.toLowerCase().includes(keyword))) {
      await this.escalateCrisis(msg.key.remoteJid!, text);
    }

    // Parse confirmation responses
    if (text.toLowerCase() === 'confirmar' || text.toLowerCase() === 'si') {
      await this.processConfirmation(msg.key.remoteJid!);
    }
  };

  async sendAppointmentReminder(
    phoneNumber: string,
    appointmentData: {
      patientName: string;
      practitionerName: string;
      dateTime: Date;
      type: string;
    }
  ) {
    const formattedDate = DateTime.fromJSDate(appointmentData.dateTime)
      .setLocale('es-AR')
      .toFormat("dd/MM/yyyy 'a las' HH:mm");

    const message = `Hola ${appointmentData.patientName},

Te recordamos tu sesión de ${appointmentData.type.toLowerCase()} con ${appointmentData.practitionerName} 
programada para el ${formattedDate}.

Respondé *CONFIRMAR* para confirmar tu asistencia o *CANCELAR* si necesitás reprogramar.

Equipo Sesión`;

    await this.sock.sendMessage(phoneNumber, { text: message });
  }

  async sendInvoice(phoneNumber: string, invoiceUrl: string, amount: number) {
    const message = `Tu factura electrónica está disponible.

Monto: $${amount.toLocaleString('es-AR')}
Link: ${invoiceUrl}

Podés pagar por transferencia, Mercado Pago o efectivo en tu próxima sesión.`;

    await this.sock.sendMessage(phoneNumber, { text: message });
  }

  private async escalateCrisis(phoneNumber: string, message: string) {
    // Alert practitioner immediately
    await this.prisma.alert.create({
      data: {
        type: 'CRISIS',
        severity: 'HIGH',
        source: phoneNumber,
        content: message,
        timestamp: new Date(),
      },
    });

    // Send automated crisis support message
    const crisisResponse = `Detectamos que podés estar pasando por un momento difícil. 
Tu bienestar es importante.

Líneas de ayuda inmediata (24hs):
🆘 Centro de Asistencia al Suicida: 135
📞 Línea de salud mental: 0800-999-0091

Tu terapeuta será notificado de forma urgente.`;

    await this.sock.sendMessage(phoneNumber, { text: crisisResponse });
  }

  private async processConfirmation(phoneNumber: string) {
    // Update appointment status
    const patient = await this.prisma.patient.findUnique({
      where: { phone: phoneNumber },
    });

    if (patient) {
      await this.prisma.appointment.updateMany({
        where: {
          patientId: patient.id,
          status: 'SCHEDULED',
          startTime: { gte: new Date() },
        },
        data: { status: 'CONFIRMED' },
      });
    }
  }
}
```

**Automated Reminder Job** (`src/jobs/reminder.job.ts`):

```typescript
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { WhatsAppService } from '../whatsapp/whatsapp.service';
import { PrismaService } from '../prisma/prisma.service';
import { DateTime } from 'luxon';

@Injectable()
export class ReminderJob {
  constructor(
    private whatsapp: WhatsAppService,
    private prisma: PrismaService
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = DateTime.now().plus({ days: 1 });
    const tomorrowStart = tomorrow.startOf('day').toJSDate();
    const tomorrowEnd = tomorrow.endOf('day').toJSDate();

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: { gte: tomorrowStart, lte: tomorrowEnd },
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.patient.phone, {
        patientName: apt.patient.firstName,
        practitionerName: apt.practitioner.fullName,
        dateTime: apt.startTime,
        type: apt.type,
      });
    }
  }

  @Cron('*/30 * * * *') // Every 30 minutes
  async send2HourReminders() {
    const twoHoursFromNow = DateTime.now().plus({ hours: 2 });
    const startWindow = twoHoursFromNow.minus({ minutes: 30 }).toJSDate();
    const endWindow = twoHoursFromNow.toJSDate();

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: { gte: startWindow, lte: endWindow },
        status: { in: ['SCHEDULED', 'CONFIRMED'] },
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.patient.phone, {
        patientName: apt.patient.firstName,
        practitionerName: apt.practitioner.fullName,
        dateTime: apt.startTime,
        type: apt.type,
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

**Invoice Service** (`src/billing/invoice.service.ts`):

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import * as AfipSDK from '@afipsdk/afip.js';

@Injectable()
export class InvoiceService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new AfipSDK({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.AFIP_ENVIRONMENT === 'production',
    });
  }

  async generateInvoice(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (!appointment) throw new Error('Appointment not found');

    // Determine invoice type based on patient's fiscal condition
    const invoiceType = this.determineInvoiceType(
      appointment.patient.fiscalCondition
    );

    // Calculate amount
    const sessionRates = await this.prisma.sessionRate.findFirst({
      where: {
        practitionerId: appointment.practitionerId,
        sessionType: appointment.type,
      },
    });

    const baseAmount = sessionRates?.amount || 0;
    const taxAmount = this.calculateTax(baseAmount, invoiceType);

    // Generate AFIP invoice
    const afipInvoice = await this.afip.ElectronicBilling.createVoucher({
      CantReg: 1,
      PtoVta: appointment.practitioner.salesPoint,
      CbteTipo: invoiceType,
      Concepto: 2, // Services
      DocTipo: 80, // CUIT
      DocNro: appointment.patient.cuit || 0,
      CbteDesde: 1,
      CbteHasta: 1,
      CbteFch: DateTime.now().toFormat('yyyyMMdd'),
      ImpTotal: baseAmount + taxAmount,
      ImpTotConc: 0,
      ImpNeto: baseAmount,
      ImpOpEx: 0,
      ImpIVA: taxAmount,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
      Tributos: null,
      Iva: [
        {
          Id: 5, // 21%
          BaseImp: baseAmount,
          Importe: taxAmount,
        },
      ],
    });

    // Store invoice in database
    const invoice = await this.prisma.invoice.create({
      data: {
        appointmentId,
        invoiceNumber: afipInvoice.CAE,
        invoiceType,
        baseAmount,
        taxAmount,
        totalAmount: baseAmount + taxAmount,
        cae: afipInvoice.CAE,
        caeExpiration: DateTime.fromFormat(
          afipInvoice.CAEFchVto,
          'yyyyMMdd'
        ).toJSDate(),
        afipResponse: afipInvoice,
      },
    });

    return invoice;
  }

  private determineInvoiceType(fiscalCondition: string): number {
    const types = {
      RESPONSABLE_INSCRIPTO: 1, // Factura A
      MONOTRIBUTO: 6, // Factura B
      CONSUMIDOR_FINAL: 6, // Factura B
      EXENTO: 11, // Factura C
    };
    return types[fiscalCondition] || 6;
  }

  private calculateTax(baseAmount: number, invoiceType: number): number {
    // Factura A and B include 21% IVA
    if (invoiceType === 1 || invoiceType === 6) {
      return baseAmount * 0.21;
    }
    return 0;
  }

  async generateMonthlyReport(practitionerId: string, month: number, year: number) {
    const startDate = DateTime.fromObject({ year, month, day: 1 }).toJSDate();
    const endDate = DateTime.fromObject({ year, month, day: 1 })
      .plus({ months: 1 })
      .toJSDate();

    const invoices = await this.prisma.invoice.findMany({
      where: {
        appointment: { practitionerId },
        createdAt: { gte: startDate, lt: endDate },
      },
      include: {
        appointment: {
          include: { patient: true },
        },
      },
    });

    const summary = {
      totalInvoices: invoices.length,
      totalRevenue: invoices.reduce((sum, inv) => sum + inv.totalAmount, 0),
      totalTax: invoices.reduce((sum, inv) => sum + inv.taxAmount, 0),
      byType: this.groupByInvoiceType(invoices),
    };

    return summary;
  }

  private groupByInvoiceType(invoices: any[]) {
    return invoices.reduce((acc, inv) => {
      const type = inv.invoiceType;
      if (!acc[type]) {
        acc[type] = { count: 0, total: 0 };
      }
      acc[type].count++;
      acc[type].total += inv.totalAmount;
      return acc;
    }, {});
  }
}
```

### 4. Video Consultations with LiveKit

**Video Service** (`src/video/video.service.ts`):

```typescript
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class VideoService {
  constructor(private prisma: PrismaService) {}

  async createVideoSession(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (!appointment) throw new Error('Appointment not found');
    if (appointment.type !== 'VIRTUAL') {
      throw new Error('Appointment is not virtual');
    }

    const roomName = `session-${appointmentId}`;

    // Create session record
    const session = await this.prisma.videoSession.create({
      data: {
        appointmentId,
        roomName,
        status: 'WAITING',
      },
    });

    return session;
  }

  generateToken(
    roomName: string,
    identity: string,
    metadata: { role: 'practitioner' | 'patient'; name: string }
  ): string {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity,
        metadata: JSON.stringify(metadata),
      }
    );

    at.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
    });

    return at.toJwt();
  }

  async getPractitionerToken(appointmentId: string, practitionerId: string) {
    const session = await this.prisma.videoSession.findFirst({
      where: { appointmentId },
      include: {
        appointment: { include: { practitioner: true } },
      },
    });

    if (!session) throw new Error('Video session not found');
    if (session.appointment.practitionerId !== practitionerId) {
      throw new Error('Unauthorized');
    }

    return this.generateToken(
      session.roomName,
      `practitioner-${practitionerId}`,
      {
        role: 'practitioner',
        name: session.appointment.practitioner.fullName,
      }
    );
  }

  async getPatientToken(appointmentId: string, patientId: string) {
    const session = await this.prisma.videoSession.findFirst({
      where: { appointmentId },
      include: {
        appointment: { include: { patient: true } },
      },
    });

    if (!session) throw new Error('Video session not found');
    if (session.appointment.patientId !== patientId) {
      throw new Error('Unauthorized');
    }

    // Check if session is in waiting room
    if (session.status === 'WAITING') {
      throw new Error('Practitioner has not started the session yet');
    }

    return this.generateToken(
      session.roomName,
      `patient-${patientId}`,
      {
        role: 'patient',
        name: session.appointment.patient.firstName,
      }
    );
  }

  async admitPatient(appointmentId: string, practitionerId: string) {
    const session = await this.prisma.videoSession.findFirst({
      where: { appointmentId },
      include: { appointment: true },
    });

    if (!session) throw new Error('Video session not found');
    if (session.appointment.practitionerId !== practitionerId) {
      throw new Error('Unauthorized');
    }

    return this.prisma.videoSession.update({
      where: { id: session.id },
      data: { status: 'ACTIVE' },
    });
  }

  async endSession(appointmentId: string) {
    const session = await this.prisma.videoSession.findFirst({
      where: { appointmentId },
    });

    if (!session) throw new Error('Video session not found');

    await this.prisma.videoSession.update({
      where: { id: session.id },
      data: {
        status: 'ENDED',
        endedAt: new Date(),
      },
    });

    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: { status: 'COMPLETED' },
    });
  }
}
```

**SvelteKit Video Component** (`src/lib/components/VideoConsultation.svelte`):

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { Room, RoomEvent, Track } from 'livekit-client';
  
  export let token: string;
  export let serverUrl: string;
  export let role: 'practitioner' | 'patient';
  
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
        resolution: {
          width: 1280,
          height: 720,
          frameRate: 30,
        },
      },
    });
    
    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.Disconnected, handleDisconnect);
    
    await room.connect(serverUrl, token);
    
    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
    isConnected = true;
    
    // Attach local video
    const videoTrack = room.localParticipant.videoTrackPublications.values().next().value?.track;
    if (videoTrack && videoElement) {
      videoTrack.attach(videoElement);
    }
    
    return () => {
      room.disconnect();
    };
  });
  
  function handleTrackSubscribed(track: any, publication: any, participant: any) {
    if (track.kind === Track.Kind.Video && remoteVideoElement) {
      track.attach(remoteVideoElement);
    }
  }
  
  function handleDisconnect() {
    isConnected = false;
  }
  
  async function toggleMute() {
    if (room.localParticipant) {
      isMuted = !isMuted;
      await room.localParticipant.setMicrophoneEnabled(!isMuted);
    }
  }
  
  async function toggleVideo() {
    if (room.localParticipant) {
      isVideoOff = !isVideoOff;
      await room.localParticipant.setCameraEnabled(!isVideoOff);
    }
  }
  
  async function endCall() {
    await room.disconnect();
    // Navigate back or show end screen
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
    <button on:click={toggleMute} class:active={isMuted}>
      {isMuted ? '🔇' : '🎤'}
    </button>
    <button on:click={toggleVideo} class:active={isVideoOff}>
      {isVideoOff ? '📹' : '📷'}
    </button>
    <button on:click={endCall} class="end-call">
      📞 Finalizar
    </button>
  </div>
  
  {#if !isConnected}
    <div class="connecting">Conectando...</div>
  {/if}
</div>

<style>
  .video-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: #000;
  }
  
  .remote-video video {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .local-video {
    position: absolute;
    top: 1rem;
    right: 1rem;
    width: 200px;
    height: 150px;
    border-radius: 0.5rem;
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
    bottom: 2rem;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 1rem;
  }
  
  .controls button {
    padding: 1rem 2rem;
    border-radius: 50%;
    border: none;
    background: rgba(255, 255, 255, 0.2);
    color: white;
    cursor: pointer;
    transition: all 0.2s;
  }
  
  .controls button:hover {
    background: rgba(255, 255, 255, 0.3);
  }
  
  .controls button.active {
    background: rgba(239, 68, 68, 0.8);
  }
  
  .end-call {
    background: rgba(239, 68, 68, 0.8) !important;
  }
</style>
```

### 5. Claude AI Integration

**AI Service** (`src/ai/ai.service.ts`):

```typescript
import { Injectable } from '@nestjs/common';
import Anthrop
