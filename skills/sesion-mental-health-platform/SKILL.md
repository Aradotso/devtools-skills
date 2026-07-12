---
name: sesion-mental-health-platform
description: AI-powered mental health practice management platform for psychologists with scheduling, WhatsApp automation, AFIP billing, video consultations, and Claude AI integration
triggers:
  - how do I set up the Sesión mental health platform
  - configure WhatsApp automation for patient appointments
  - integrate AFIP electronic invoicing in Sesión
  - set up video consultations with LiveKit
  - use Claude AI for clinical notes in Sesión
  - configure MercadoPago payment integration
  - implement patient journey workflows
  - troubleshoot Sesión WhatsApp connection
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It orchestrates appointment scheduling, automated WhatsApp messaging (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with SvelteKit 5, NestJS, Prisma, and TypeScript.

## Installation

### Prerequisites

```bash
# Required versions
node >= 18.0.0
pnpm >= 8.0.0
postgresql >= 14.0
redis >= 7.0
```

### Clone and Setup

```bash
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
ANTHROPIC_API_KEY="sk-ant-..."
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_DIR="./sessions"
WHATSAPP_QR_TIMEOUT=60000

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="wss://your-livekit.cloud.livekit.cloud"

# Payment Gateways
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"

# AFIP Integration
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip.crt"
AFIP_KEY_PATH="./certs/afip.key"
AFIP_PRODUCTION=false

# Application
JWT_SECRET="${JWT_SECRET}"
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
```

### Database Setup

```bash
# Run migrations
pnpm prisma migrate dev

# Seed initial data
pnpm prisma db seed
```

### Start Development Servers

```bash
# Frontend (SvelteKit)
pnpm dev

# Backend (NestJS)
pnpm api:dev

# Full stack
pnpm dev:full
```

## Architecture Overview

```
sesion/
├── apps/
│   ├── web/          # SvelteKit 5 frontend
│   ├── api/          # NestJS backend
│   └── worker/       # Background job processor
├── packages/
│   ├── db/           # Prisma schema & migrations
│   ├── ai/           # Claude AI orchestration
│   ├── whatsapp/     # Baileys integration
│   ├── video/        # LiveKit wrapper
│   ├── billing/      # AFIP invoicing
│   └── shared/       # Common types & utils
└── prisma/
    └── schema.prisma
```

## Core Modules

### 1. Appointment Scheduling

#### Create Appointment (NestJS Controller)

```typescript
// apps/api/src/appointments/appointments.controller.ts
import { Controller, Post, Body, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { AppointmentsService } from './appointments.service';
import { CreateAppointmentDto } from './dto/create-appointment.dto';

@Controller('appointments')
@UseGuards(JwtAuthGuard)
export class AppointmentsController {
  constructor(private readonly appointmentsService: AppointmentsService) {}

  @Post()
  async create(@Body() dto: CreateAppointmentDto) {
    return this.appointmentsService.create(dto);
  }
}
```

#### Appointment Service with Conflict Detection

```typescript
// apps/api/src/appointments/appointments.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from '@sesion/db';
import { addMinutes, isWithinInterval } from 'date-fns';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async create(dto: CreateAppointmentDto) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: dto.practitionerId,
        status: { not: 'CANCELLED' },
        scheduledAt: {
          gte: dto.scheduledAt,
          lt: addMinutes(dto.scheduledAt, dto.durationMinutes || 45)
        }
      }
    });

    if (conflicts.length > 0) {
      throw new ConflictException('Time slot already booked');
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        ...dto,
        status: 'SCHEDULED'
      },
      include: {
        patient: true,
        practitioner: true
      }
    });

    // Trigger WhatsApp confirmation
    await this.whatsappService.sendConfirmation(appointment);

    return appointment;
  }

  async findUpcoming(practitionerId: string) {
    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        scheduledAt: { gte: new Date() },
        status: { in: ['SCHEDULED', 'CONFIRMED'] }
      },
      include: { patient: true },
      orderBy: { scheduledAt: 'asc' }
    });
  }
}
```

#### Frontend: Appointment Calendar (Svelte 5)

```svelte
<!-- apps/web/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  
  let appointments = $state([]);
  let selectedDate = $state(new Date());
  let loading = $state(false);

  async function loadAppointments() {
    loading = true;
    const res = await fetch('/api/appointments/upcoming', {
      headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
    });
    appointments = await res.json();
    loading = false;
  }

  async function createAppointment(event: CustomEvent) {
    const res = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify(event.detail)
    });
    
    if (res.ok) {
      await loadAppointments();
    }
  }

  onMount(loadAppointments);
</script>

<div class="agenda-container">
  <header>
    <h1>Agenda</h1>
    <button onclick={() => goto('/appointments/new')}>
      Nueva Cita
    </button>
  </header>

  {#if loading}
    <div class="skeleton"></div>
  {:else}
    <div class="appointments-grid">
      {#each appointments as apt}
        <div class="appointment-card" class:urgent={apt.isUrgent}>
          <time>{new Date(apt.scheduledAt).toLocaleString('es-AR')}</time>
          <h3>{apt.patient.name}</h3>
          <span class="type">{apt.type}</span>
          <span class="status">{apt.status}</span>
        </div>
      {/each}
    </div>
  {/if}
</div>
```

### 2. WhatsApp Automation (Baileys)

#### WhatsApp Connection Service

```typescript
// packages/whatsapp/src/whatsapp.service.ts
import makeWASocket, { 
  DisconnectReason, 
  useMultiFileAuthState,
  makeInMemoryStore 
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class WhatsAppService {
  private sock: any;
  private readonly logger = new Logger(WhatsAppService.name);

  async connect() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_DIR || './sessions'
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true
    });

    this.sock.ev.on('creds.update', saveCreds);

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect, qr } = update;
      
      if (qr) {
        this.logger.log('QR Code generated, scan with WhatsApp');
      }

      if (connection === 'close') {
        const shouldReconnect = 
          (lastDisconnect?.error as Boom)?.output?.statusCode !== 
          DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.logger.log('Reconnecting...');
          this.connect();
        }
      } else if (connection === 'open') {
        this.logger.log('WhatsApp connected successfully');
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendReminder(appointment: Appointment) {
    const patientNumber = appointment.patient.phone.replace(/\D/g, '') + '@s.whatsapp.net';
    
    const message = `Hola ${appointment.patient.name}! 👋\n\n` +
      `Te recordamos tu sesión programada:\n` +
      `📅 ${format(appointment.scheduledAt, "dd/MM/yyyy 'a las' HH:mm", { locale: es })}\n` +
      `👨‍⚕️ Con: ${appointment.practitioner.name}\n\n` +
      `Por favor confirma tu asistencia respondiendo SI o NO.`;

    await this.sock.sendMessage(patientNumber, { text: message });
    
    await this.prisma.whatsappMessage.create({
      data: {
        appointmentId: appointment.id,
        to: patientNumber,
        content: message,
        type: 'REMINDER',
        status: 'SENT'
      }
    });
  }

  async handleIncomingMessage(message: any) {
    if (!message.message?.conversation) return;

    const text = message.message.conversation.toLowerCase();
    const from = message.key.remoteJid;

    // Check for confirmation replies
    if (['si', 'sí', 'confirmo'].includes(text)) {
      await this.confirmAppointment(from);
    } else if (['no', 'cancelar'].includes(text)) {
      await this.requestCancellation(from);
    }
    
    // Crisis keyword detection
    if (this.detectCrisisKeywords(text)) {
      await this.alertPractitioner(from, text);
    }
  }

  private detectCrisisKeywords(text: string): boolean {
    const crisisKeywords = [
      'suicidio', 'matarme', 'terminar con todo', 
      'no puedo más', 'crisis', 'emergencia'
    ];
    return crisisKeywords.some(kw => text.includes(kw));
  }
}
```

#### Automated Reminder Scheduler

```typescript
// apps/worker/src/jobs/reminder.job.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '@sesion/db';
import { WhatsAppService } from '@sesion/whatsapp';
import { subHours } from 'date-fns';

@Injectable()
export class ReminderJob {
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
        scheduledAt: {
          gte: tomorrow,
          lt: new Date(tomorrow.getTime() + 60 * 60 * 1000) // +1 hour window
        },
        status: 'SCHEDULED',
        reminderSent24h: false
      },
      include: { patient: true, practitioner: true }
    });

    for (const apt of appointments) {
      await this.whatsapp.sendReminder(apt);
      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent24h: true }
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

#### Invoice Generation Service

```typescript
// packages/billing/src/afip.service.ts
import { Injectable } from '@nestjs/common';
import Afip from '@afipsdk/afip.js';
import { PrismaService } from '@sesion/db';

@Injectable()
export class AfipService {
  private afip: Afip;

  constructor(private prisma: PrismaService) {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.AFIP_PRODUCTION === 'true'
    });
  }

  async generateInvoice(sessionId: string) {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: { patient: true, practitioner: true }
    });

    // Get last invoice number
    const lastVoucher = await this.afip.ElectronicBilling.getLastVoucher(
      session.practitioner.puntoVenta,
      6 // Tipo factura B
    );

    const invoiceData = {
      'CantReg': 1,
      'PtoVta': session.practitioner.puntoVenta,
      'CbteTipo': 6, // Factura B
      'Concepto': 2, // Servicios
      'DocTipo': 96, // DNI
      'DocNro': session.patient.dni,
      'CbteDesde': lastVoucher + 1,
      'CbteHasta': lastVoucher + 1,
      'CbteFch': Math.floor(Date.now() / 1000),
      'ImpTotal': session.amount,
      'ImpTotConc': 0,
      'ImpNeto': session.amount,
      'ImpOpEx': 0,
      'ImpIVA': 0,
      'ImpTrib': 0,
      'MonId': 'PES',
      'MonCotiz': 1
    };

    const response = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    // Save invoice record
    const invoice = await this.prisma.invoice.create({
      data: {
        sessionId: session.id,
        patientId: session.patientId,
        practitionerId: session.practitionerId,
        afipCae: response.CAE,
        afipCaeExpiration: new Date(response.CAEFchVto),
        invoiceNumber: `${session.practitioner.puntoVenta}-${lastVoucher + 1}`,
        amount: session.amount,
        type: 'FACTURA_B',
        status: 'ISSUED'
      }
    });

    return invoice;
  }

  async getPdfUrl(invoiceId: string): Promise<string> {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id: invoiceId }
    });

    // Generate PDF using AFIP data
    const pdfBuffer = await this.generatePdfBuffer(invoice);
    
    // Upload to MinIO or cloud storage
    const url = await this.uploadPdf(pdfBuffer, invoiceId);
    
    return url;
  }
}
```

### 4. Video Consultations (LiveKit)

#### Video Room Service

```typescript
// packages/video/src/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken, RoomServiceClient } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  private roomService: RoomServiceClient;

  constructor() {
    this.roomService = new RoomServiceClient(
      process.env.LIVEKIT_WS_URL!,
      process.env.LIVEKIT_API_KEY!,
      process.env.LIVEKIT_API_SECRET!
    );
  }

  async createRoom(appointmentId: string) {
    const roomName = `session-${appointmentId}`;
    
    await this.roomService.createRoom({
      name: roomName,
      emptyTimeout: 60 * 10, // 10 minutes
      maxParticipants: 2
    });

    return roomName;
  }

  async generateToken(
    roomName: string, 
    participantName: string, 
    role: 'practitioner' | 'patient'
  ): Promise<string> {
    const token = new AccessToken(
      process.env.LIVEKIT_API_KEY!,
      process.env.LIVEKIT_API_SECRET!,
      {
        identity: participantName,
        name: participantName
      }
    );

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: role === 'practitioner'
    });

    return await token.toJwt();
  }

  async endSession(roomName: string) {
    await this.roomService.deleteRoom(roomName);
  }
}
```

#### Video Room Component (Svelte)

```svelte
<!-- apps/web/src/lib/components/VideoRoom.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, VideoPresets } from 'livekit-client';

  let { appointmentId } = $props();
  let room = $state<Room | null>(null);
  let videoElement = $state<HTMLVideoElement>();
  let audioElement = $state<HTMLAudioElement>();
  let isConnected = $state(false);

  async function connect() {
    const res = await fetch(`/api/video/token/${appointmentId}`);
    const { token, roomName } = await res.json();

    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: VideoPresets.h720.resolution
      }
    });

    room.on('trackSubscribed', (track, publication, participant) => {
      if (track.kind === 'video' && videoElement) {
        track.attach(videoElement);
      } else if (track.kind === 'audio' && audioElement) {
        track.attach(audioElement);
      }
    });

    await room.connect(process.env.PUBLIC_LIVEKIT_WS_URL!, token);
    await room.localParticipant.enableCameraAndMicrophone();
    isConnected = true;
  }

  async function disconnect() {
    await room?.disconnect();
    isConnected = false;
  }

  onMount(connect);
  onDestroy(disconnect);
</script>

<div class="video-container">
  <video bind:this={videoElement} autoplay playsinline></video>
  <audio bind:this={audioElement} autoplay></audio>
  
  <div class="controls">
    <button onclick={disconnect}>Finalizar Sesión</button>
  </div>
</div>

<style>
  video {
    width: 100%;
    max-height: 80vh;
    border-radius: 8px;
  }
</style>
```

### 5. Claude AI Integration

#### Clinical Notes Assistant

```typescript
// packages/ai/src/claude.service.ts
import Anthropic from '@anthropic-ai/sdk';
import { Injectable } from '@nestjs/common';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }

  async summarizeSession(sessionNotes: string): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 2000,
      messages: [{
        role: 'user',
        content: `Como psicólogo clínico en Argentina, resume las siguientes notas de sesión en formato estructurado:

Notas originales:
${sessionNotes}

Genera un resumen con:
- Motivo de consulta
- Observaciones principales
- Intervenciones realizadas
- Plan de tratamiento
- Próximos pasos

Usa terminología clínica argentina y respeta la confidencialidad.`
      }],
      system: 'Eres un asistente especializado en psicología clínica para profesionales en Argentina.'
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async chatAssistant(query: string, context: string): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4',
      max_tokens: 1000,
      messages: [{
        role: 'user',
        content: query
      }],
      system: `Contexto del paciente: ${context}\n\nResponde consultas rápidas sobre el historial clínico.`
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async predictOutcome(patientHistory: any[]): Promise<{ risk: string, recommendations: string[] }> {
    const historyText = JSON.stringify(patientHistory, null, 2);

    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 1500,
      messages: [{
        role: 'user',
        content: `Analiza el siguiente historial de sesiones y predice factores de riesgo:

${historyText}

Genera un JSON con:
{
  "risk": "bajo|medio|alto",
  "recommendations": ["recomendación 1", "recomendación 2"]
}`
      }]
    });

    const text = message.content[0].type === 'text' ? message.content[0].text : '{}';
    return JSON.parse(text);
  }
}
```

#### AI-Assisted Note Taking (Frontend)

```svelte
<!-- apps/web/src/routes/session/[id]/notes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  
  let { data } = $props();
  let notes = $state('');
  let summary = $state('');
  let loading = $state(false);

  async function generateSummary() {
    loading = true;
    const res = await fetch(`/api/ai/summarize`, {
      method: 'POST',
      headers: { 
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({ notes })
    });
    const { summary: generated } = await res.json();
    summary = generated;
    loading = false;
  }

  async function saveNotes() {
    await fetch(`/api/sessions/${data.session.id}/notes`, {
      method: 'PUT',
      headers: { 
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({ notes, summary })
    });
  }
</script>

<div class="notes-editor">
  <textarea 
    bind:value={notes} 
    placeholder="Notas de sesión..."
    rows="15"
  ></textarea>

  <button onclick={generateSummary} disabled={loading}>
    {loading ? 'Generando...' : 'Generar Resumen con IA'}
  </button>

  {#if summary}
    <div class="summary-panel">
      <h3>Resumen Clínico</h3>
      <div>{summary}</div>
    </div>
  {/if}

  <button onclick={saveNotes}>Guardar Notas</button>
</div>
```

## Prisma Schema Highlights

```prisma
// packages/db/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Practitioner {
  id            String   @id @default(cuid())
  email         String   @unique
  name          String
  license       String   @unique
  puntoVenta    Int
  fiscalCategory String  // Monotributo, Responsable Inscripto
  cuit          String   @unique
  
  appointments  Appointment[]
  patients      Patient[]
  invoices      Invoice[]
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}

model Patient {
  id              String   @id @default(cuid())
  name            String
  dni             String   @unique
  phone           String
  email           String?
  
  practitionerId  String
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  
  appointments    Appointment[]
  sessions        Session[]
  invoices        Invoice[]
  
  consentGiven    Boolean  @default(false)
  consentDate     DateTime?
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}

model Appointment {
  id                String   @id @default(cuid())
  scheduledAt       DateTime
  durationMinutes   Int      @default(45)
  type              String   // PRESENCIAL, VIRTUAL, EVALUACION
  status            String   // SCHEDULED, CONFIRMED, CANCELLED, COMPLETED
  
  patientId         String
  patient           Patient  @relation(fields: [patientId], references: [id])
  
  practitionerId    String
  practitioner      Practitioner @relation(fields: [practitionerId], references: [id])
  
  reminderSent24h   Boolean  @default(false)
  reminderSent2h    Boolean  @default(false)
  
  session           Session?
  
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  
  @@index([practitionerId, scheduledAt])
}

model Session {
  id              String   @id @default(cuid())
  appointmentId   String   @unique
  appointment     Appointment @relation(fields: [appointmentId], references: [id])
  
  notes           String?
  aiSummary       String?
  recordingUrl    String?
  
  amount          Float
  paid            Boolean  @default(false)
  
  patientId       String
  patient         Patient  @relation(fields: [patientId], references: [id])
  
  invoice         Invoice?
  
  startedAt       DateTime?
  endedAt         DateTime?
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}

model Invoice {
  id                  String   @id @default(cuid())
  invoiceNumber       String   @unique
  type                String   // FACTURA_A, FACTURA_B, FACTURA_C
  amount              Float
  status              String   // ISSUED, PAID, CANCELLED
  
  afipCae             String
  afipCaeExpiration   DateTime
  
  sessionId           String   @unique
  session             Session  @relation(fields: [sessionId], references: [id])
  
  patientId           String
  patient             Patient  @relation(fields: [patientId], references: [id])
  
  practitionerId      String
  practitioner        Practitioner @relation(fields: [practitionerId], references: [id])
  
  pdfUrl              String?
  sentViaWhatsApp     Boolean  @default(false)
  
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
}

model WhatsAppMessage {
  id              String   @id @default(cuid())
  to              String
  content         String
  type            String   // REMINDER, CONFIRMATION, INVOICE, CRISIS_ALERT
  status          String   // SENT, DELIVERED, READ, FAILED
  
  appointmentId   String?
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([to, createdAt])
}
```

## Common Workflows

### Patient Onboarding Flow

```typescript
// apps/api/src/patients/onboarding.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@sesion/db';
import { WhatsAppService } from '@sesion/whatsapp';

@Injectable()
export class OnboardingService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService
  ) {}

  async onboardNewPatient(data: {
    name: string;
    dni: string;
    phone: string;
    practitionerId: string;
  }) {
    // 1. Create patient record
    const patient = await this.prisma.patient.create({
      data: {
        ...data,
        
