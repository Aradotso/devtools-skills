---
name: sesion-clinic-workflow-platform
description: Mental health practice management platform with intelligent scheduling, WhatsApp automation, AFIP billing, video consultations, and Claude AI integration for Argentine psychologists
triggers:
  - set up sesion clinic workflow platform
  - configure sesion appointment scheduling and whatsapp automation
  - integrate claude ai with sesion mental health platform
  - implement afip compliant invoicing with sesion
  - configure livekit video consultations in sesion
  - work with sesion psychology practice management
  - set up mercadopago billing in sesion
  - build sesion whatsapp patient communication workflows
---

# Sesión Clinic Workflow Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It orchestrates appointment scheduling, automated WhatsApp patient communication, AFIP-compliant electronic invoicing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6 models.

## Architecture Overview

Sesión is built as a microservices ecosystem with:

- **Frontend**: SvelteKit 5 with TypeScript and TailwindCSS
- **Backend**: NestJS microservices
- **Database**: PostgreSQL with Prisma ORM
- **Cache/Sessions**: Redis
- **Search**: Elasticsearch
- **Storage**: MinIO (S3-compatible)
- **Messaging**: Apache Kafka for event-driven workflows
- **Video**: LiveKit WebRTC infrastructure
- **WhatsApp**: Baileys library for WhatsApp Web API
- **AI**: Anthropic Claude (Opus 4.6 for deep analysis, Sonnet 4.6 for real-time assistance)
- **Payments**: Stripe (international), Mercado Pago (Argentina)

## Installation and Setup

### Prerequisites

```bash
# Required tools
node >= 20.x
pnpm >= 8.x
docker >= 24.x
docker-compose >= 2.x
postgresql >= 15.x
redis >= 7.x
```

### Clone and Install Dependencies

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"
ELASTICSEARCH_URL="http://localhost:9200"

# MinIO Storage
MINIO_ENDPOINT="localhost"
MINIO_PORT=9000
MINIO_ACCESS_KEY="${MINIO_ACCESS_KEY}"
MINIO_SECRET_KEY="${MINIO_SECRET_KEY}"
MINIO_BUCKET="sesion-documents"

# Authentication
JWT_SECRET="${JWT_SECRET}"
JWT_EXPIRES_IN="15m"
REFRESH_TOKEN_SECRET="${REFRESH_TOKEN_SECRET}"
REFRESH_TOKEN_EXPIRES_IN="7d"

# Anthropic Claude AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"
CLAUDE_MAX_TOKENS=4096

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="wss://your-livekit-instance.com"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# Payment Gateways
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"
STRIPE_WEBHOOK_SECRET="${STRIPE_WEBHOOK_SECRET}"
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
MERCADOPAGO_PUBLIC_KEY="${MERCADOPAGO_PUBLIC_KEY}"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip.crt"
AFIP_KEY_PATH="./certs/afip.key"
AFIP_PRODUCTION=false

# Kafka
KAFKA_BROKERS="localhost:9092"
KAFKA_CLIENT_ID="sesion-clinic"
KAFKA_CONSUMER_GROUP="sesion-workers"

# Application
NODE_ENV="development"
PORT=3000
API_PORT=4000
FRONTEND_URL="http://localhost:3000"
BACKEND_URL="http://localhost:4000"
```

### Database Setup

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate deploy

# Seed initial data (optional)
pnpm prisma db seed
```

### Start Services

```bash
# Start infrastructure (Docker Compose)
docker-compose up -d postgres redis elasticsearch kafka minio

# Start backend services
cd backend
pnpm run start:dev

# Start frontend (separate terminal)
cd frontend
pnpm run dev
```

## Key Components and Usage

### 1. Agenda Management (Appointment Scheduling)

#### Prisma Schema for Appointments

```typescript
// prisma/schema.prisma
model Appointment {
  id                String   @id @default(cuid())
  patientId         String
  practitionerId    String
  sessionType       SessionType
  startTime         DateTime
  endTime           DateTime
  status            AppointmentStatus @default(SCHEDULED)
  roomId            String?
  notes             String?
  reminderSent24h   Boolean  @default(false)
  reminderSent2h    Boolean  @default(false)
  confirmationToken String?  @unique
  
  patient           Patient       @relation(fields: [patientId], references: [id])
  practitioner      Practitioner  @relation(fields: [practitionerId], references: [id])
  room              Room?         @relation(fields: [roomId], references: [id])
  invoice           Invoice?
  videoSession      VideoSession?
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([practitionerId, startTime])
  @@index([patientId, startTime])
}

enum SessionType {
  PRESENCIAL
  VIRTUAL
  EVALUACION
  COUPLES
  FAMILY
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

#### Appointment Service (NestJS)

```typescript
// backend/src/appointments/appointments.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { EventEmitter2 } from '@nestjs/event-emitter';
import { Appointment, SessionType } from '@prisma/client';
import { addMinutes, isBefore, isAfter } from 'date-fns';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private eventEmitter: EventEmitter2,
  ) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    sessionType: SessionType;
    startTime: Date;
    roomId?: string;
  }): Promise<Appointment> {
    // Determine session duration based on type
    const duration = this.getSessionDuration(data.sessionType);
    const endTime = addMinutes(data.startTime, duration);

    // Check for conflicts
    await this.checkAvailability(
      data.practitionerId,
      data.startTime,
      endTime,
      data.roomId,
    );

    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime,
        confirmationToken: this.generateConfirmationToken(),
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Emit event for downstream processing (WhatsApp, calendar sync)
    this.eventEmitter.emit('appointment.created', appointment);

    return appointment;
  }

  private getSessionDuration(sessionType: SessionType): number {
    const durations = {
      PRESENCIAL: 45,
      VIRTUAL: 45,
      EVALUACION: 60,
      COUPLES: 90,
      FAMILY: 90,
    };
    return durations[sessionType] || 45;
  }

  private async checkAvailability(
    practitionerId: string,
    startTime: Date,
    endTime: Date,
    roomId?: string,
  ): Promise<void> {
    // Check practitioner availability
    const practitionerConflicts = await this.prisma.appointment.findMany({
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

    if (practitionerConflicts.length > 0) {
      throw new ConflictException(
        'Practitioner has a conflicting appointment',
      );
    }

    // Check room availability if specified
    if (roomId) {
      const roomConflicts = await this.prisma.appointment.findMany({
        where: {
          roomId,
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

      if (roomConflicts.length > 0) {
        throw new ConflictException('Room is not available at this time');
      }
    }
  }

  private generateConfirmationToken(): string {
    return Math.random().toString(36).substring(2, 15);
  }

  async suggestAlternativeSlots(
    practitionerId: string,
    preferredDate: Date,
    sessionType: SessionType,
  ): Promise<Date[]> {
    const duration = this.getSessionDuration(sessionType);
    const suggestions: Date[] = [];

    // Get practitioner's working hours
    const workingHours = await this.prisma.practitionerSchedule.findMany({
      where: { practitionerId },
    });

    // Find available slots within 7 days
    // Implementation details for slot suggestion algorithm
    return suggestions;
  }
}
```

### 2. WhatsApp Automation with Baileys

#### WhatsApp Service

```typescript
// backend/src/whatsapp/whatsapp.service.ts
import { Injectable, Logger } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { OnEvent } from '@nestjs/event-emitter';
import { Appointment } from '@prisma/client';
import { format, subHours } from 'date-fns';
import { es } from 'date-fns/locale';

@Injectable()
export class WhatsAppService {
  private sock: any;
  private readonly logger = new Logger(WhatsAppService.name);

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH,
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('creds.update', saveCreds);

    this.sock.ev.on('connection.update', (update: any) => {
      const { connection, lastDisconnect } = update;
      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.initialize();
        }
      } else if (connection === 'open') {
        this.logger.log('WhatsApp connection opened successfully');
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }: any) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  private async handleIncomingMessage(message: WAMessage) {
    const messageText = message.message?.conversation || '';
    const from = message.key.remoteJid;

    // Crisis keyword detection
    const crisisKeywords = [
      'suicidio',
      'hacerme daño',
      'no aguanto más',
      'crisis',
      'emergencia',
    ];
    const isCrisis = crisisKeywords.some((keyword) =>
      messageText.toLowerCase().includes(keyword),
    );

    if (isCrisis) {
      // Alert practitioner immediately
      this.logger.warn(`Crisis message detected from ${from}`);
      // Trigger emergency protocol
    }

    // Parse confirmation responses
    if (messageText.toLowerCase().includes('confirmo')) {
      await this.handleAppointmentConfirmation(from, message);
    }
  }

  @OnEvent('appointment.created')
  async handleAppointmentCreated(appointment: Appointment & any) {
    // Schedule reminder 24 hours before
    const reminderTime24h = subHours(appointment.startTime, 24);
    // Use job scheduler (e.g., Bull) to send reminder at reminderTime24h
  }

  async sendAppointmentReminder(appointment: Appointment & any) {
    const formattedDate = format(appointment.startTime, "EEEE d 'de' MMMM", {
      locale: es,
    });
    const formattedTime = format(appointment.startTime, 'HH:mm');

    const message = `Hola ${appointment.patient.firstName},

Te recordamos tu sesión ${appointment.sessionType.toLowerCase()} con ${appointment.practitioner.firstName} ${appointment.practitioner.lastName}.

📅 Fecha: ${formattedDate}
🕐 Hora: ${formattedTime}
${appointment.sessionType === 'VIRTUAL' ? '💻 Tipo: Videollamada' : '🏥 Tipo: Presencial'}

Por favor, confirmá tu asistencia respondiendo "CONFIRMO" a este mensaje.

Si necesitás reprogramar, comunicate con nosotros.

Equipo Sesión`;

    await this.sendMessage(appointment.patient.phone, message);
  }

  async sendInvoice(invoiceData: any) {
    const message = `Hola ${invoiceData.patient.firstName},

Adjunto encontrarás tu comprobante de sesión:

📄 Factura: ${invoiceData.invoiceNumber}
💰 Monto: $${invoiceData.amount}
📅 Fecha: ${format(invoiceData.date, 'dd/MM/yyyy')}

Gracias por confiar en nosotros.

Equipo Sesión`;

    await this.sendMessage(invoiceData.patient.phone, message);

    // Send PDF invoice
    if (invoiceData.pdfBuffer) {
      await this.sock.sendMessage(invoiceData.patient.phone, {
        document: invoiceData.pdfBuffer,
        mimetype: 'application/pdf',
        fileName: `factura-${invoiceData.invoiceNumber}.pdf`,
      });
    }
  }

  private async sendMessage(to: string, text: string) {
    try {
      await this.sock.sendMessage(to, { text });
      this.logger.log(`Message sent to ${to}`);
    } catch (error) {
      this.logger.error(`Failed to send message to ${to}:`, error);
    }
  }

  private async handleAppointmentConfirmation(from: string, message: WAMessage) {
    // Extract appointment from context and mark as confirmed
    // Implementation details
  }
}
```

### 3. AFIP Electronic Invoicing

#### AFIP Service

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { Afip } from '@afipsdk/afip.js';
import * as fs from 'fs';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: fs.readFileSync(process.env.AFIP_CERT_PATH),
      key: fs.readFileSync(process.env.AFIP_KEY_PATH),
      production: process.env.AFIP_PRODUCTION === 'true',
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

    // Determine invoice type based on practitioner fiscal category
    const invoiceType = this.determineInvoiceType(
      appointment.practitioner.fiscalCategory,
    );

    // Get next invoice number
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      1, // Sale point
      invoiceType,
    );

    const invoiceNumber = lastInvoice + 1;

    // Calculate amounts
    const amount = appointment.practitioner.sessionRate;
    const taxableAmount = amount / 1.21; // IVA 21%
    const vatAmount = amount - taxableAmount;

    const data = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: invoiceType,
      Concepto: 3, // Services
      DocTipo: 96, // DNI
      DocNro: appointment.patient.documentNumber,
      CbteDesde: invoiceNumber,
      CbteHasta: invoiceNumber,
      CbteFch: this.formatDate(new Date()),
      ImpTotal: amount,
      ImpTotConc: 0,
      ImpNeto: taxableAmount,
      ImpOpEx: 0,
      ImpIVA: vatAmount,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
      Iva: [
        {
          Id: 5, // IVA 21%
          BaseImp: taxableAmount,
          Importe: vatAmount,
        },
      ],
    };

    // Generate invoice via AFIP
    const result = await this.afip.ElectronicBilling.createVoucher(data);

    // Save invoice to database
    const invoice = await this.prisma.invoice.create({
      data: {
        appointmentId,
        invoiceNumber: `${result.CbteTipo}-${result.PtoVta}-${invoiceNumber}`,
        invoiceType,
        cae: result.CAE,
        caeExpirationDate: this.parseAfipDate(result.CAEFchVto),
        amount,
        taxableAmount,
        vatAmount,
        status: 'ISSUED',
      },
    });

    return invoice;
  }

  private determineInvoiceType(fiscalCategory: string): number {
    // Tipo A: 1, Tipo B: 6, Tipo C: 11, Tipo M: 51
    const types = {
      RESPONSABLE_INSCRIPTO: 1,
      MONOTRIBUTO: 6,
      CONSUMIDOR_FINAL: 11,
    };
    return types[fiscalCategory] || 6;
  }

  private formatDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  private parseAfipDate(afipDate: string): Date {
    return new Date(
      `${afipDate.substring(0, 4)}-${afipDate.substring(4, 6)}-${afipDate.substring(6, 8)}`,
    );
  }

  async getMonthlyResumen(practitionerId: string, year: number, month: number) {
    const invoices = await this.prisma.invoice.findMany({
      where: {
        appointment: {
          practitionerId,
        },
        createdAt: {
          gte: new Date(year, month - 1, 1),
          lt: new Date(year, month, 1),
        },
      },
    });

    const totalAmount = invoices.reduce((sum, inv) => sum + inv.amount, 0);
    const totalVat = invoices.reduce((sum, inv) => sum + inv.vatAmount, 0);

    return {
      period: `${year}-${String(month).padStart(2, '0')}`,
      totalInvoices: invoices.length,
      totalAmount,
      totalVat,
      taxableAmount: totalAmount - totalVat,
      invoices,
    };
  }
}
```

### 4. LiveKit Video Consultations

#### Video Session Service

```typescript
// backend/src/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class VideoService {
  constructor(private prisma: PrismaService) {}

  async createVideoSession(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    const roomName = `session-${appointmentId}`;

    // Create room metadata
    const videoSession = await this.prisma.videoSession.create({
      data: {
        appointmentId,
        roomName,
        status: 'WAITING',
      },
    });

    return videoSession;
  }

  async generateToken(
    appointmentId: string,
    userId: string,
    userType: 'patient' | 'practitioner',
  ): Promise<string> {
    const videoSession = await this.prisma.videoSession.findUnique({
      where: { appointmentId },
    });

    if (!videoSession) {
      throw new Error('Video session not found');
    }

    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: userId,
        ttl: '2h',
      },
    );

    at.addGrant({
      roomJoin: true,
      room: videoSession.roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
      // Practitioners have additional controls
      ...(userType === 'practitioner' && {
        roomAdmin: true,
        roomRecord: true,
      }),
    });

    return at.toJwt();
  }

  async admitPatient(appointmentId: string) {
    return this.prisma.videoSession.update({
      where: { appointmentId },
      data: { status: 'ACTIVE' },
    });
  }

  async endSession(appointmentId: string) {
    const videoSession = await this.prisma.videoSession.update({
      where: { appointmentId },
      data: { status: 'ENDED', endedAt: new Date() },
    });

    // Update appointment status
    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: { status: 'COMPLETED' },
    });

    return videoSession;
  }
}
```

#### Frontend Video Component (Svelte)

```svelte
<!-- frontend/src/lib/components/VideoConsultation.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent, Track } from 'livekit-client';
  
  export let token: string;
  export let serverUrl: string;
  
  let videoElement: HTMLVideoElement;
  let remoteVideoElement: HTMLVideoElement;
  let room: Room;
  let isConnected = false;
  let isMuted = false;
  let isVideoOff = false;
  
  onMount(async () => {
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: { width: 1280, height: 720, frameRate: 24 }
      }
    });
    
    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.TrackUnsubscribed, handleTrackUnsubscribed);
    room.on(RoomEvent.Disconnected, handleDisconnect);
    
    try {
      await room.connect(serverUrl, token);
      isConnected = true;
      
      // Publish local tracks
      await room.localParticipant.enableCameraAndMicrophone();
      
      // Attach local video
      const videoTrack = room.localParticipant.getTrackPublication(Track.Source.Camera);
      if (videoTrack?.track) {
        videoTrack.track.attach(videoElement);
      }
    } catch (error) {
      console.error('Failed to connect to room:', error);
    }
  });
  
  function handleTrackSubscribed(track: any, publication: any, participant: any) {
    if (track.kind === Track.Kind.Video) {
      track.attach(remoteVideoElement);
    }
  }
  
  function handleTrackUnsubscribed(track: any) {
    track.detach();
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
    // Notify backend
    await fetch(`/api/video/end-session`, { method: 'POST' });
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
      {#if !isConnected}
        <div class="waiting-room">
          <p>Esperando al terapeuta...</p>
        </div>
      {/if}
    </div>
    <div class="local-video">
      <video bind:this={videoElement} autoplay playsinline muted />
    </div>
  </div>
  
  <div class="controls">
    <button on:click={toggleMute} class:active={!isMuted}>
      {isMuted ? '🔇' : '🎤'}
    </button>
    <button on:click={toggleVideo} class:active={!isVideoOff}>
      {isVideoOff ? '📷' : '📹'}
    </button>
    <button on:click={endCall} class="end-call">
      Finalizar
    </button>
  </div>
</div>

<style>
  .video-consultation {
    position: relative;
    width: 100%;
    height: 100vh;
    background: #1a1a1a;
  }
  
  .video-grid {
    width: 100%;
    height: calc(100% - 80px);
    position: relative;
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
    bottom: 20px;
    right: 20px;
    width: 240px;
    height: 180px;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  }
  
  .local-video video {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .waiting-room {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(0,0,0,0.8);
    color: white;
  }
  
  .controls {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 80px;
    display: flex;
    gap: 16px;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.6);
  }
  
  .controls button {
    width: 56px;
    height: 56px;
    border-radius: 50%;
    border: none;
    font-size: 24px;
    cursor: pointer;
    background: #333;
    color: white;
    transition: all 0.2s;
  }
  
  .controls button:hover {
    background: #444;
  }
  
  .controls button.active {
    background: #4CAF50;
  }
  
  .controls button.end-call {
    background: #f44336;
    width: auto;
    padding: 0 24px;
    border-radius: 28px;
  }
</style>
```

### 5. Claude AI Integration

#### AI Orchestration Service

```typescript
// backend/src/ai/claude.service.ts
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
    appointmentIds: string[],
  ): Promise<string> {
    // Fetch all appointments with
