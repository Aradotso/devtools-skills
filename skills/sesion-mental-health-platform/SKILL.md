---
name: sesion-mental-health-platform
description: Orchestrate psychology clinic workflows with AI-powered scheduling, WhatsApp automation, AFIP billing, and video consultations for Argentine practitioners
triggers:
  - "set up Sesión mental health platform"
  - "configure WhatsApp automation for clinic appointments"
  - "implement AFIP compliant invoicing for psychology practice"
  - "integrate Claude AI for clinical note summarization"
  - "build video consultation workflow with Sesión"
  - "manage patient journey and scheduling automation"
  - "deploy Argentine psychology clinic SaaS"
  - "configure Baileys WhatsApp integration"
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina, providing intelligent appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with SvelteKit 5, NestJS microservices, Prisma ORM, and TypeScript.

## Installation

### Prerequisites

- Node.js 20+
- PostgreSQL 15+
- Redis 7+
- PNPM 8+

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

Create `.env` file with required credentials:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# LiveKit (Video)
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="wss://sesion.livekit.cloud"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip.crt"
AFIP_KEY_PATH="./certs/afip.key"

# Application
JWT_SECRET="${JWT_SECRET}"
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
```

### Database Migration

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate deploy

# Seed initial data (optional)
pnpm prisma db seed
```

### Start Development Servers

```bash
# Start all services concurrently
pnpm dev

# Or start individually:
pnpm dev:frontend  # SvelteKit app (port 5173)
pnpm dev:backend   # NestJS API (port 3000)
pnpm dev:whatsapp  # WhatsApp service (port 3001)
pnpm dev:video     # Video service (port 3002)
```

## Core Architecture

### Microservices Structure

```
/apps
  /frontend      # SvelteKit 5 + Tailwind CSS
  /api           # NestJS REST API
  /whatsapp      # Baileys WhatsApp automation
  /video         # LiveKit video orchestration
  /billing       # AFIP invoicing engine
/packages
  /prisma        # Shared Prisma schema
  /types         # TypeScript type definitions
  /ai            # Claude AI orchestration
```

## Key Components

### 1. Patient Management

#### Create Patient (API)

```typescript
// apps/api/src/patients/patients.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { PrismaService } from '@prisma/client';

@Controller('patients')
export class PatientsController {
  constructor(private prisma: PrismaService) {}

  @Post()
  async createPatient(@Body() data: CreatePatientDto) {
    return this.prisma.patient.create({
      data: {
        firstName: data.firstName,
        lastName: data.lastName,
        email: data.email,
        phone: data.phone,
        whatsappOptIn: data.whatsappOptIn,
        dateOfBirth: new Date(data.dateOfBirth),
        address: {
          create: {
            street: data.address.street,
            city: data.address.city,
            province: data.address.province,
            postalCode: data.address.postalCode
          }
        }
      }
    });
  }
}
```

#### Patient Component (Frontend)

```svelte
<!-- apps/frontend/src/routes/patients/[id]/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import type { Patient } from '@sesion/types';
  
  export let data;
  
  let patient: Patient = data.patient;
  let appointments = data.appointments;
  
  async function scheduleAppointment() {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        patientId: patient.id,
        practitionerId: data.practitioner.id,
        startTime: selectedSlot.start,
        duration: 45, // minutes
        sessionType: 'INDIVIDUAL'
      })
    });
    
    if (response.ok) {
      appointments = [...appointments, await response.json()];
    }
  }
</script>

<div class="patient-profile">
  <h1>{patient.firstName} {patient.lastName}</h1>
  
  <section class="appointments">
    <h2>Próximas Sesiones</h2>
    {#each appointments as appt}
      <div class="appointment-card">
        <time>{new Date(appt.startTime).toLocaleString('es-AR')}</time>
        <span>{appt.sessionType}</span>
      </div>
    {/each}
  </section>
  
  <button on:click={scheduleAppointment}>Agendar Nueva Sesión</button>
</div>
```

### 2. WhatsApp Automation (Baileys)

#### Initialize WhatsApp Client

```typescript
// apps/whatsapp/src/whatsapp.service.ts
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  makeInMemoryStore
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

export class WhatsAppService {
  private sock: any;
  private store: any;
  
  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH!
    );
    
    this.store = makeInMemoryStore({});
    
    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true
    });
    
    this.sock.ev.on('creds.update', saveCreds);
    
    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      
      if (connection === 'close') {
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode 
          !== DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.initialize();
        }
      }
    });
    
    this.sock.ev.on('messages.upsert', async (m) => {
      await this.handleIncomingMessage(m);
    });
  }
  
  async sendAppointmentReminder(
    patientPhone: string,
    appointment: Appointment
  ) {
    const formattedPhone = patientPhone.replace(/\D/g, '') + '@s.whatsapp.net';
    
    const message = `🗓️ *Recordatorio de Sesión*\n\n` +
      `Hola! Te recordamos tu cita:\n` +
      `📅 ${new Date(appointment.startTime).toLocaleDateString('es-AR')}\n` +
      `🕐 ${new Date(appointment.startTime).toLocaleTimeString('es-AR', { 
        hour: '2-digit', 
        minute: '2-digit' 
      })}\n` +
      `👤 ${appointment.practitioner.name}\n\n` +
      `¿Confirmas tu asistencia? Responde:\n` +
      `1️⃣ SÍ - Confirmo\n` +
      `2️⃣ NO - Necesito reprogramar`;
    
    await this.sock.sendMessage(formattedPhone, { text: message });
  }
  
  async handleIncomingMessage(messageUpdate: any) {
    const message = messageUpdate.messages[0];
    
    if (!message.message || message.key.fromMe) return;
    
    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text;
    const from = message.key.remoteJid;
    
    // Crisis keyword detection
    const crisisKeywords = ['suicidio', 'hacerme daño', 'no puedo más'];
    if (crisisKeywords.some(kw => text.toLowerCase().includes(kw))) {
      await this.escalateCrisis(from, text);
      return;
    }
    
    // Appointment confirmation parsing
    if (text === '1' || text.toLowerCase().includes('confirmo')) {
      await this.confirmAppointment(from);
    } else if (text === '2' || text.toLowerCase().includes('reprogramar')) {
      await this.initiateRescheduling(from);
    }
  }
}
```

#### Schedule Automated Reminders

```typescript
// apps/api/src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '@prisma/client';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService
  ) {}
  
  // Run every hour
  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);
    
    const upcomingAppointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: new Date(),
          lte: tomorrow
        },
        status: 'SCHEDULED',
        reminderSent24h: false
      },
      include: {
        patient: true,
        practitioner: true
      }
    });
    
    for (const appt of upcomingAppointments) {
      if (appt.patient.whatsappOptIn) {
        await this.whatsapp.sendAppointmentReminder(
          appt.patient.phone,
          appt
        );
        
        await this.prisma.appointment.update({
          where: { id: appt.id },
          data: { reminderSent24h: true }
        });
      }
    }
  }
}
```

### 3. AFIP Electronic Invoicing

#### Generate Compliant Invoice

```typescript
// apps/billing/src/afip/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as AfipSDK from '@afipsdk/afip.js';

@Injectable()
export class AfipService {
  private afip: any;
  
  constructor() {
    this.afip = new AfipSDK({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.NODE_ENV === 'production'
    });
  }
  
  async generateInvoice(session: Session, patient: Patient) {
    const puntoVenta = 1; // Sales point number
    const concepto = 2; // Services (1=Products, 2=Services, 3=Both)
    
    // Determine invoice type based on patient fiscal category
    const invoiceType = patient.fiscalCategory === 'RESPONSABLE_INSCRIPTO' 
      ? 1  // Factura A
      : 6; // Factura B
    
    const invoiceData = {
      CantReg: 1,
      PtoVta: puntoVenta,
      CbteTipo: invoiceType,
      Concepto: concepto,
      DocTipo: 80, // CUIT (80) or DNI (96)
      DocNro: patient.taxId || patient.dni,
      CbteDesde: await this.getNextInvoiceNumber(puntoVenta, invoiceType),
      CbteHasta: await this.getNextInvoiceNumber(puntoVenta, invoiceType),
      CbteFch: new Date().toISOString().split('T')[0].replace(/-/g, ''),
      ImpTotal: session.amount,
      ImpTotConc: 0,
      ImpNeto: session.amount / 1.21, // Net without VAT
      ImpOpEx: 0,
      ImpIVA: session.amount - (session.amount / 1.21), // VAT 21%
      ImpTrib: 0,
      MonId: 'PES', // Argentine Peso
      MonCotiz: 1,
      Iva: [
        {
          Id: 5, // 21% VAT
          BaseImp: session.amount / 1.21,
          Importe: session.amount - (session.amount / 1.21)
        }
      ]
    };
    
    const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);
    
    return {
      cae: result.CAE,
      caeExpiration: result.CAEFchVto,
      invoiceNumber: `${puntoVenta.toString().padStart(4, '0')}-${result.CbteDesde.toString().padStart(8, '0')}`,
      invoiceType,
      amount: session.amount
    };
  }
  
  private async getNextInvoiceNumber(
    puntoVenta: number,
    invoiceType: number
  ): Promise<number> {
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      puntoVenta,
      invoiceType
    );
    
    return lastInvoice + 1;
  }
}
```

#### Invoice Storage

```typescript
// apps/billing/src/invoices/invoices.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@prisma/client';
import { AfipService } from '../afip/afip.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class InvoicesService {
  constructor(
    private prisma: PrismaService,
    private afip: AfipService,
    private whatsapp: WhatsAppService
  ) {}
  
  async createInvoiceForSession(sessionId: string) {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: { patient: true, practitioner: true }
    });
    
    // Generate AFIP-compliant invoice
    const afipInvoice = await this.afip.generateInvoice(
      session,
      session.patient
    );
    
    // Store in database
    const invoice = await this.prisma.invoice.create({
      data: {
        sessionId: session.id,
        patientId: session.patient.id,
        practitionerId: session.practitioner.id,
        invoiceNumber: afipInvoice.invoiceNumber,
        invoiceType: afipInvoice.invoiceType,
        cae: afipInvoice.cae,
        caeExpiration: new Date(afipInvoice.caeExpiration),
        amount: afipInvoice.amount,
        currency: 'ARS',
        status: 'ISSUED'
      }
    });
    
    // Send invoice via WhatsApp
    if (session.patient.whatsappOptIn) {
      await this.whatsapp.sendInvoice(
        session.patient.phone,
        invoice
      );
    }
    
    return invoice;
  }
}
```

### 4. Video Consultations (LiveKit)

#### Create Video Room

```typescript
// apps/video/src/livekit/livekit.service.ts
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
  
  async createConsultationRoom(appointment: Appointment) {
    const roomName = `session-${appointment.id}`;
    
    // Create room with custom settings
    await this.roomService.createRoom({
      name: roomName,
      emptyTimeout: 60 * 10, // 10 minutes
      maxParticipants: 2, // Practitioner + Patient
      metadata: JSON.stringify({
        appointmentId: appointment.id,
        practitionerId: appointment.practitionerId,
        patientId: appointment.patientId
      })
    });
    
    return roomName;
  }
  
  generateToken(
    roomName: string,
    participantName: string,
    participantId: string,
    canRecord: boolean = false
  ): string {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY!,
      process.env.LIVEKIT_API_SECRET!,
      {
        identity: participantId,
        name: participantName
      }
    );
    
    at.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
      canUpdateOwnMetadata: true,
      ...(canRecord && { roomRecord: true })
    });
    
    return at.toJwt();
  }
}
```

#### Video Room Component (Frontend)

```svelte
<!-- apps/frontend/src/routes/sessions/[id]/video/+page.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';
  
  export let data;
  
  let videoElement: HTMLVideoElement;
  let room: Room;
  let isConnected = false;
  let remoteParticipant: any;
  
  onMount(async () => {
    // Get access token from API
    const response = await fetch(`/api/video/token?sessionId=${data.session.id}`);
    const { token, wsUrl } = await response.json();
    
    // Initialize LiveKit room
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: { width: 1280, height: 720 }
      }
    });
    
    // Handle remote participant
    room.on(RoomEvent.ParticipantConnected, (participant) => {
      remoteParticipant = participant;
      participant.on('trackSubscribed', (track) => {
        if (track.kind === 'video' && videoElement) {
          track.attach(videoElement);
        }
      });
    });
    
    room.on(RoomEvent.Connected, () => {
      isConnected = true;
    });
    
    // Connect to room
    await room.connect(wsUrl, token);
    
    // Enable local camera and microphone
    await room.localParticipant.enableCameraAndMicrophone();
  });
  
  onDestroy(() => {
    room?.disconnect();
  });
  
  async function toggleScreenShare() {
    await room.localParticipant.setScreenShareEnabled(
      !room.localParticipant.isScreenShareEnabled
    );
  }
</script>

<div class="video-consultation">
  {#if isConnected}
    <div class="video-grid">
      <video bind:this={videoElement} autoplay playsinline />
      
      <div class="controls">
        <button on:click={() => room.localParticipant.setMicrophoneEnabled(false)}>
          Silenciar
        </button>
        <button on:click={toggleScreenShare}>
          Compartir Pantalla
        </button>
        <button on:click={() => room.disconnect()} class="end-call">
          Finalizar Sesión
        </button>
      </div>
    </div>
  {:else}
    <p>Conectando a la videollamada...</p>
  {/if}
</div>
```

### 5. AI-Powered Clinical Notes (Claude)

#### Note Summarization Service

```typescript
// packages/ai/src/claude.service.ts
import Anthropic from '@anthropic-ai/sdk';
import { Injectable } from '@nestjs/common';

@Injectable()
export class ClaudeService {
  private client: Anthropic;
  
  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }
  
  async summarizeSession(
    sessionNotes: string,
    patientHistory: string[]
  ): Promise<string> {
    const prompt = `Eres un asistente especializado en psicología clínica en Argentina. 
Tu tarea es resumir las notas de sesión siguiendo el formato SOAP (Subjetivo, Objetivo, 
Análisis, Plan) respetando la confidencialidad y el contexto cultural argentino.

HISTORIAL DEL PACIENTE (últimas 3 sesiones):
${patientHistory.join('\n\n')}

NOTAS DE LA SESIÓN ACTUAL:
${sessionNotes}

Genera un resumen clínico estructurado que incluya:
1. Presentación del paciente (estado anímico, verbalizaciones clave)
2. Observaciones objetivas (lenguaje corporal, tono emocional)
3. Análisis clínico (hipótesis diagnósticas, progreso terapéutico)
4. Plan de acción (intervenciones futuras, tareas para el paciente)`;

    const message = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL!,
      max_tokens: 2048,
      temperature: 0.3,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  async generateInsightsSonnet(patientId: string): Promise<string[]> {
    const sessions = await this.prisma.session.findMany({
      where: { patientId },
      orderBy: { startTime: 'desc' },
      take: 10,
      select: { clinicalNotes: true, emotionalState: true }
    });
    
    const prompt = `Analiza las últimas 10 sesiones y genera insights clínicos:

${sessions.map((s, i) => `Sesión ${i + 1}:\nEstado: ${s.emotionalState}\nNotas: ${s.clinicalNotes}`).join('\n\n')}

Identifica:
- Patrones emocionales recurrentes
- Evolución sintomática
- Factores desencadenantes
- Áreas de progreso terapéutico`;

    const message = await this.client.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL!,
      max_tokens: 1024,
      temperature: 0.5,
      messages: [{ role: 'user', content: prompt }]
    });
    
    const text = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
    
    return text.split('\n').filter(line => line.trim().startsWith('-'));
  }
}
```

#### AI Assistant Chat Interface

```svelte
<!-- apps/frontend/src/lib/components/AIAssistant.svelte -->
<script lang="ts">
  import { writable } from 'svelte/store';
  
  let messages = writable<Array<{role: string, content: string}>>([]);
  let input = '';
  let isLoading = false;
  
  async function sendMessage() {
    if (!input.trim()) return;
    
    const userMessage = { role: 'user', content: input };
    messages.update(m => [...m, userMessage]);
    
    isLoading = true;
    input = '';
    
    const response = await fetch('/api/ai/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        messages: [...$messages, userMessage],
        model: 'sonnet' // Fast responses for chat
      })
    });
    
    const { reply } = await response.json();
    messages.update(m => [...m, { role: 'assistant', content: reply }]);
    
    isLoading = false;
  }
</script>

<div class="ai-assistant">
  <div class="messages">
    {#each $messages as msg}
      <div class="message {msg.role}">
        {msg.content}
      </div>
    {/each}
    
    {#if isLoading}
      <div class="message assistant loading">
        Claude está pensando...
      </div>
    {/if}
  </div>
  
  <form on:submit|preventDefault={sendMessage}>
    <input
      bind:value={input}
      placeholder="Pregunta sobre un paciente, resume una sesión..."
      disabled={isLoading}
    />
    <button type="submit" disabled={isLoading}>Enviar</button>
  </form>
</div>
```

## Database Schema (Prisma)

### Key Models

```prisma
// packages/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Patient {
  id            String   @id @default(cuid())
  firstName     String
  lastName      String
  email         String?  @unique
  phone         String   @unique
  whatsappOptIn Boolean  @default(false)
  dateOfBirth   DateTime
  dni           String?  @unique
  taxId         String?  // CUIT/CUIL
  fiscalCategory String? // RESPONSABLE_INSCRIPTO, MONOTRIBUTO, etc.
  
  address       Address?
  appointments  Appointment[]
  sessions      Session[]
  invoices      Invoice[]
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}

model Appointment {
  id              String   @id @default(cuid())
  patientId       String
  practitionerId  String
  startTime       DateTime
  duration        Int      // minutes
  sessionType     SessionType
  status          AppointmentStatus @default(SCHEDULED)
  
  reminderSent24h Boolean  @default(false)
  reminderSent2h  Boolean  @default(false)
  confirmationReceived Boolean @default(false)
  
  patient         Patient  @relation(fields: [patientId], references: [id])
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  session         Session?
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([patientId])
  @@index([practitionerId])
  @@index([startTime])
}

model Session {
  id              String   @id @default(cuid())
  appointmentId   String   @unique
  patientId       String
  practitionerId  String
  
  clinicalNotes   String?  @db.Text
  emotionalState  String?
  aiSummary       String?  @db.Text
  
  amount          Decimal  @db.Decimal(10, 2)
  paymentMethod   PaymentMethod?
  paymentStatus   PaymentStatus @default(PENDING)
  
  videoRoomName   String?
  recordingUrl    String?
  
  appointment     Appointment @relation(fields: [appointmentId], references: [id])
  patient         Patient  @relation(fields: [patientId], references: [id])
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  invoice         Invoice?
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}

model Invoice {
  id              String   @id @default(cuid())
  sessionId       String   @unique
  patientId       String
  practitionerId  String
  
  invoiceNumber   String   @unique
  invoiceType     Int      // AFIP invoice type (1=A, 6=B, etc.)
  cae             String   // CAE from AFIP
  caeExpiration   DateTime
  
  amount          Decimal  @db.Decimal(10, 2)
  currency        String   @default("ARS")
  status          InvoiceStatus @default(ISSUED)
  
  pdfUrl          String?
  sentViaWhatsApp Boolean  @default(false)
  
  session         Session  @relation(fields: [sessionId], references: [id])
  patient         Patient  @relation(fields: [patientId], references: [id])
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}

enum SessionType {
  INDIVIDUAL
  COUPLE
  FAMILY
  EVALUATION
