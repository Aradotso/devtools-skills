---
name: sesion-mental-health-platform
description: AI-powered mental health practice management platform for psychologists with scheduling, WhatsApp automation, AFIP billing, and video consultations
triggers:
  - "set up Sesión mental health platform"
  - "integrate WhatsApp automation for patient scheduling"
  - "configure AFIP electronic invoicing for psychology clinic"
  - "implement Claude AI for clinical note summarization"
  - "deploy video consultation system with LiveKit"
  - "configure Mercado Pago payment integration for therapy sessions"
  - "build psychology practice management dashboard"
  - "integrate Baileys WhatsApp business API"
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It orchestrates appointment scheduling, automated WhatsApp messaging via Baileys, AFIP-compliant electronic invoicing (facturación electrónica), secure video consultations with LiveKit, and AI-powered clinical workflows using Claude Opus 4.6 and Sonnet 4.6.

**Tech Stack:**
- **Backend:** NestJS (TypeScript) with microservices architecture
- **Frontend:** SvelteKit 5 + Tailwind CSS
- **Database:** Prisma ORM with PostgreSQL
- **WhatsApp:** Baileys library for multi-device API
- **Video:** LiveKit WebRTC infrastructure
- **AI:** Anthropic Claude Opus/Sonnet via REST API
- **Payments:** Stripe (international) + Mercado Pago (Argentina)
- **Real-time:** Redis for caching and pub/sub

## Installation

### Prerequisites

```bash
node >= 18.0.0
pnpm >= 8.0.0
postgresql >= 14
redis >= 6.2
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
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="sk-ant-..."
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your-webhook-secret"

# LiveKit Video
LIVEKIT_API_KEY="APIkey..."
LIVEKIT_API_SECRET="secret..."
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."
STRIPE_SECRET_KEY="sk_test_..."

# AFIP (Argentina Tax Authority)
AFIP_CUIT="20-12345678-9"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
AFIP_ENVIRONMENT="testing" # or "production"

# App Config
FRONTEND_URL="http://localhost:5173"
BACKEND_URL="http://localhost:3000"
JWT_SECRET="your-jwt-secret"
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

### Start Development Servers

```bash
# Start backend (NestJS)
cd apps/backend
pnpm dev

# Start frontend (SvelteKit) - separate terminal
cd apps/frontend
pnpm dev
```

## Core Architecture

### Prisma Schema Structure

```prisma
// schema.prisma (simplified)
model Practitioner {
  id            String        @id @default(cuid())
  email         String        @unique
  name          String
  fiscalCategory String       // Monotributo, Responsable Inscripto
  cuitCuil      String        @unique
  appointments  Appointment[]
  invoices      Invoice[]
  aiUsage       AIUsage[]
  createdAt     DateTime      @default(now())
}

model Patient {
  id            String        @id @default(cuid())
  name          String
  phone         String        @unique // WhatsApp number
  email         String?
  journeyStage  String        @default("primer_contacto")
  appointments  Appointment[]
  consents      Consent[]
  createdAt     DateTime      @default(now())
}

model Appointment {
  id              String      @id @default(cuid())
  practitionerId  String
  patientId       String
  startTime       DateTime
  endTime         DateTime
  type            String      // presencial, virtual, evaluacion
  status          String      // scheduled, confirmed, completed, cancelled
  roomId          String?
  videoRoomToken  String?
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  patient         Patient      @relation(fields: [patientId], references: [id])
  clinicalNotes   ClinicalNote[]
  invoice         Invoice?
}

model Invoice {
  id              String      @id @default(cuid())
  appointmentId   String      @unique
  practitionerId  String
  amount          Decimal
  afipReceiptType String      // A, B, C, M
  afipCAE         String?     // AFIP authorization code
  caeExpiration   DateTime?
  status          String      // draft, issued, paid, cancelled
  pdfUrl          String?
  sentViaWhatsApp Boolean     @default(false)
  createdAt       DateTime    @default(now())
}
```

## NestJS Backend Patterns

### Appointment Scheduling Module

```typescript
// apps/backend/src/appointments/appointments.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async scheduleAppointment(dto: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflict = await this.prisma.appointment.findFirst({
      where: {
        practitionerId: dto.practitionerId,
        OR: [
          {
            startTime: { lte: dto.startTime },
            endTime: { gt: dto.startTime },
          },
          {
            startTime: { lt: dto.endTime },
            endTime: { gte: dto.endTime },
          },
        ],
      },
    });

    if (conflict) {
      throw new ConflictException('Horario no disponible');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        ...dto,
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(appointment);

    // Schedule reminders
    await this.scheduleReminders(appointment.id);

    return appointment;
  }

  private async scheduleReminders(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true },
    });

    // 24-hour reminder
    const reminderTime24h = new Date(appointment.startTime);
    reminderTime24h.setHours(reminderTime24h.getHours() - 24);

    // 2-hour reminder
    const reminderTime2h = new Date(appointment.startTime);
    reminderTime2h.setHours(reminderTime2h.getHours() - 2);

    // Queue jobs (using Bull or similar)
    await this.queueWhatsAppReminder(appointmentId, reminderTime24h);
    await this.queueWhatsAppReminder(appointmentId, reminderTime2h);
  }
}
```

### WhatsApp Integration with Baileys

```typescript
// apps/backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  async onModuleInit() {
    await this.connectWhatsApp();
  }

  private async connectWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH,
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
          this.connectWhatsApp();
        }
      }
    });

    // Handle incoming messages
    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    const message = `Hola ${appointment.patient.name}! 👋\n\nTu sesión con ${appointment.practitioner.name} está confirmada:\n\n📅 Fecha: ${this.formatDate(appointment.startTime)}\n⏰ Hora: ${this.formatTime(appointment.startTime)}\n📍 Modalidad: ${appointment.type}\n\nResponde "CONFIRMAR" para confirmar o "CANCELAR" si necesitas modificarla.`;

    await this.sock.sendMessage(appointment.patient.phone + '@s.whatsapp.net', {
      text: message,
    });
  }

  async sendInvoiceViaWhatsApp(invoice: any, patient: any) {
    const pdfBuffer = await this.generateInvoicePDF(invoice);

    await this.sock.sendMessage(patient.phone + '@s.whatsapp.net', {
      document: pdfBuffer,
      fileName: `Factura_${invoice.id}.pdf`,
      mimetype: 'application/pdf',
      caption: `Factura de tu sesión. CAE: ${invoice.afipCAE}`,
    });
  }

  private async handleIncomingMessage(message: WAMessage) {
    if (!message.message) return;

    const text = message.message.conversation?.toLowerCase() || '';
    const phone = message.key.remoteJid?.replace('@s.whatsapp.net', '');

    // Crisis keyword detection
    const crisisKeywords = ['suicidio', 'matarme', 'no quiero vivir', 'crisis'];
    if (crisisKeywords.some((kw) => text.includes(kw))) {
      await this.triggerCrisisAlert(phone, text);
    }

    // Appointment confirmation
    if (text.includes('confirmar')) {
      await this.confirmAppointmentByPhone(phone);
    }

    // Cancellation request
    if (text.includes('cancelar')) {
      await this.initiateCancellationFlow(phone);
    }
  }

  private formatDate(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      weekday: 'long',
      day: 'numeric',
      month: 'long',
    }).format(date);
  }

  private formatTime(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false,
    }).format(date);
  }
}
```

### AFIP Electronic Invoicing

```typescript
// apps/backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as soap from 'soap';

@Injectable()
export class AfipService {
  private wsdlUrl =
    process.env.AFIP_ENVIRONMENT === 'production'
      ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx?WSDL'
      : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx?WSDL';

  async generateElectronicInvoice(invoiceData: {
    practitionerId: string;
    amount: number;
    patientCuit?: string;
    receiptType: 'A' | 'B' | 'C';
  }) {
    const practitioner = await this.getPractitioner(invoiceData.practitionerId);

    // Get AFIP authorization token
    const authToken = await this.getAfipAuthToken();

    const client = await soap.createClientAsync(this.wsdlUrl);

    const invoiceRequest = {
      Auth: {
        Token: authToken.token,
        Sign: authToken.sign,
        Cuit: process.env.AFIP_CUIT,
      },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: 1,
          CbteTipo: this.getReceiptTypeCode(invoiceData.receiptType),
        },
        FeDetReq: {
          FECAEDetRequest: {
            Concepto: 2, // Services
            DocTipo: 99, // General consumer
            DocNro: 0,
            CbteDesde: await this.getNextInvoiceNumber(),
            CbteHasta: await this.getNextInvoiceNumber(),
            CbteFch: this.formatAfipDate(new Date()),
            ImpTotal: invoiceData.amount,
            ImpTotConc: 0,
            ImpNeto: invoiceData.amount,
            ImpOpEx: 0,
            ImpIVA: 0,
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
          },
        },
      },
    };

    const response = await client.FECAESolicitarAsync(invoiceRequest);
    const result = response[0].FECAESolicitarResult;

    if (result.Errors) {
      throw new Error(`AFIP Error: ${result.Errors.Err[0].Msg}`);
    }

    return {
      cae: result.FeDetResp.FECAEDetResponse[0].CAE,
      caeExpiration: result.FeDetResp.FECAEDetResponse[0].CAEFchVto,
      receiptNumber: result.FeDetResp.FECAEDetResponse[0].CbteDesde,
    };
  }

  private async getAfipAuthToken() {
    // Implementation of AFIP WSAA authentication
    // Requires certificate signing and ticket request
    const cert = fs.readFileSync(process.env.AFIP_CERT_PATH);
    const key = fs.readFileSync(process.env.AFIP_KEY_PATH);

    // Simplified - actual implementation requires XML signing
    return {
      token: 'TOKEN_FROM_AFIP_WSAA',
      sign: 'SIGNATURE_FROM_AFIP_WSAA',
    };
  }

  private getReceiptTypeCode(type: 'A' | 'B' | 'C'): number {
    const codes = { A: 1, B: 6, C: 11 };
    return codes[type];
  }

  private formatAfipDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }
}
```

### Claude AI Integration

```typescript
// apps/backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(
    sessionHistory: string[],
    useOpus = false,
  ): Promise<string> {
    const model = useOpus
      ? process.env.CLAUDE_OPUS_MODEL
      : process.env.CLAUDE_SONNET_MODEL;

    const prompt = `Eres un asistente clínico especializado en psicología argentina. Resume las siguientes notas de sesiones manteniendo confidencialidad y enfoque terapéutico.

Sesiones previas:
${sessionHistory.join('\n\n---\n\n')}

Genera un resumen profesional que incluya:
1. Motivo de consulta principal
2. Evolución observada
3. Técnicas terapéuticas aplicadas
4. Objetivos terapéuticos actuales
5. Recomendaciones para próximas sesiones`;

    const message = await this.anthropic.messages.create({
      model,
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    const summary = message.content[0].text;

    // Log usage for cost tracking
    await this.logAIUsage({
      model,
      inputTokens: message.usage.input_tokens,
      outputTokens: message.usage.output_tokens,
      cost: this.calculateCost(message.usage, model),
    });

    return summary;
  }

  async generateInsuranceReport(
    patientName: string,
    sessions: any[],
  ): Promise<string> {
    const prompt = `Genera un informe para obra social siguiendo normativas argentinas.

Paciente: ${patientName}
Cantidad de sesiones: ${sessions.length}
Período: ${sessions[0].date} a ${sessions[sessions.length - 1].date}

El informe debe incluir:
- Diagnóstico presuntivo (CIE-10)
- Objetivos terapéuticos
- Técnicas aplicadas
- Evolución clínica
- Pronóstico
- Recomendación de continuidad

Usa lenguaje técnico-profesional apropiado para auditoría de obras sociales.`;

    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL, // Use Opus for complex reports
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });

    return message.content[0].text;
  }

  private calculateCost(
    usage: { input_tokens: number; output_tokens: number },
    model: string,
  ): number {
    // Pricing as of 2026 (example)
    const prices = {
      'claude-opus-4.6': { input: 0.000015, output: 0.000075 },
      'claude-sonnet-4.6': { input: 0.000003, output: 0.000015 },
    };

    const rate = prices[model] || prices['claude-sonnet-4.6'];
    return usage.input_tokens * rate.input + usage.output_tokens * rate.output;
  }
}
```

### LiveKit Video Service

```typescript
// apps/backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  async createVideoRoom(appointmentId: string): Promise<string> {
    const roomName = `session-${appointmentId}`;
    
    // LiveKit automatically creates rooms on first participant join
    return roomName;
  }

  async generateParticipantToken(
    roomName: string,
    participantName: string,
    isPractitioner: boolean,
  ): Promise<string> {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: participantName,
      },
    );

    at.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: isPractitioner, // Only practitioners can share screen
    });

    return at.toJwt();
  }

  async endVideoSession(appointmentId: string) {
    const roomName = `session-${appointmentId}`;
    
    // Signal all participants to disconnect
    // LiveKit rooms auto-clean after all participants leave
    
    return { roomName, ended: true };
  }
}
```

## SvelteKit Frontend Patterns

### Appointment Scheduling Component

```svelte
<!-- apps/frontend/src/routes/appointments/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { format } from 'date-fns';
  import { es } from 'date-fns/locale';

  let appointments = $state([]);
  let selectedDate = $state(new Date());
  let availableSlots = $state([]);

  async function loadAppointments() {
    const res = await fetch('/api/appointments');
    appointments = await res.json();
  }

  async function checkAvailability(date: Date) {
    const res = await fetch(`/api/appointments/available?date=${date.toISOString()}`);
    availableSlots = await res.json();
  }

  async function scheduleAppointment(slot: any) {
    const res = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        patientId: selectedPatient,
        startTime: slot.start,
        endTime: slot.end,
        type: selectedType,
      }),
    });

    if (res.ok) {
      await loadAppointments();
      alert('Turno agendado. Se enviará confirmación por WhatsApp.');
    }
  }

  onMount(() => {
    loadAppointments();
    checkAvailability(selectedDate);
  });

  $effect(() => {
    checkAvailability(selectedDate);
  });
</script>

<div class="agenda-container">
  <h1 class="text-2xl font-bold mb-4">Agenda Inteligente</h1>

  <div class="calendar-grid">
    <input
      type="date"
      bind:value={selectedDate}
      class="border rounded p-2"
    />

    <div class="slots-container mt-4">
      {#each availableSlots as slot}
        <button
          onclick={() => scheduleAppointment(slot)}
          class="slot-button bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
        >
          {format(new Date(slot.start), 'HH:mm', { locale: es })} -
          {format(new Date(slot.end), 'HH:mm', { locale: es })}
        </button>
      {/each}
    </div>
  </div>

  <div class="appointments-list mt-6">
    <h2 class="text-xl font-semibold mb-3">Próximas Sesiones</h2>
    {#each appointments as appt}
      <div class="appointment-card border rounded p-4 mb-2">
        <div class="flex justify-between">
          <span>{appt.patient.name}</span>
          <span class="badge badge-{appt.status}">{appt.status}</span>
        </div>
        <div class="text-sm text-gray-600">
          {format(new Date(appt.startTime), "EEEE d 'de' MMMM, HH:mm", { locale: es })}
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .agenda-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  .slots-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 1rem;
  }
</style>
```

### Video Consultation Room

```svelte
<!-- apps/frontend/src/routes/video/[appointmentId]/+page.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';
  import { page } from '$app/stores';

  let videoContainer: HTMLDivElement;
  let room: Room;
  let isConnected = $state(false);
  let participants = $state([]);

  async function joinRoom() {
    const res = await fetch(`/api/video/token?appointmentId=${$page.params.appointmentId}`);
    const { token, url } = await res.json();

    room = new Room({
      adaptiveStream: true,
      dynacast: true,
    });

    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.ParticipantConnected, updateParticipants);
    room.on(RoomEvent.ParticipantDisconnected, updateParticipants);

    await room.connect(url, token);
    isConnected = true;

    // Publish local audio and video
    await room.localParticipant.enableCameraAndMicrophone();
    attachLocalVideo();
  }

  function handleTrackSubscribed(track, publication, participant) {
    if (track.kind === 'video') {
      const element = track.attach();
      element.className = 'remote-video';
      videoContainer.appendChild(element);
    }
  }

  function attachLocalVideo() {
    const localVideo = room.localParticipant.videoTracks.values().next().value;
    if (localVideo) {
      const element = localVideo.track.attach();
      element.className = 'local-video';
      videoContainer.prepend(element);
    }
  }

  function updateParticipants() {
    participants = Array.from(room.participants.values());
  }

  async function leaveRoom() {
    await room?.disconnect();
    isConnected = false;
  }

  onMount(() => {
    joinRoom();
  });

  onDestroy(() => {
    leaveRoom();
  });
</script>

<div class="video-consultation">
  <div class="video-grid" bind:this={videoContainer}>
    <!-- Video elements attached dynamically -->
  </div>

  <div class="controls">
    <button onclick={() => room?.localParticipant.setMicrophoneEnabled(!room.localParticipant.isMicrophoneEnabled)}>
      {room?.localParticipant.isMicrophoneEnabled ? '🎤' : '🔇'}
    </button>
    
    <button onclick={() => room?.localParticipant.setCameraEnabled(!room.localParticipant.isCameraEnabled)}>
      {room?.localParticipant.isCameraEnabled ? '📹' : '📷'}
    </button>

    <button onclick={leaveRoom} class="btn-danger">
      Finalizar Sesión
    </button>
  </div>

  <div class="session-info">
    <p>Participantes: {participants.length + 1}</p>
    <p>Duración: <span id="timer">00:00</span></p>
  </div>
</div>

<style>
  .video-consultation {
    height: 100vh;
    display: flex;
    flex-direction: column;
    background: #1a1a1a;
  }

  .video-grid {
    flex: 1;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1rem;
    padding: 1rem;
  }

  :global(.remote-video), :global(.local-video) {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 8px;
  }

  .controls {
    padding: 1rem;
    display: flex;
    justify-content: center;
    gap: 1rem;
    background: #2a2a2a;
  }

  .controls button {
    padding: 1rem 2rem;
    border-radius: 50%;
    font-size: 1.5rem;
    background: #4a4a4a;
    border: none;
    cursor: pointer;
  }

  .btn-danger {
    background: #dc2626 !important;
    color: white;
  }
</style>
```

## API Endpoints Reference

### Appointments

```typescript
GET    /api/appointments              // List all appointments
POST   /api/appointments              // Create appointment
GET    /api/appointments/:id          // Get appointment details
PATCH  /api/appointments/:id          // Update appointment
DELETE /api/appointments/:id          // Cancel appointment
GET    /api/appointments/available    // Check available slots
```

### WhatsApp

```typescript
POST   /api/whatsapp/send             // Send manual message
GET    /api/whatsapp/messages/:phone  // Get message history
POST   /api/whatsapp/qr               // Generate QR for pairing
```

### Billing

```typescript
GET    /api/invoices                  // List invoices
POST   /api/invoices/generate         // Generate AFIP invoice
GET    /api/invoices/:id/pdf          // Download PDF
POST   /api/invoices/:id/send         // Send via WhatsApp
GET    /api/invoices/summary          // Monthly tax summary
```

### AI

```typescript
POST   /api/ai/summarize              // Summarize clinical notes
POST   /api/ai/report                 // Generate insurance report
GET    /api/ai/usage                  // AI usage statistics
```

### Video

```typescript
POST   /api/video/room                // Create video room
GET    /api/video/token               // Get participant token
DELETE /api/video/room/:id            // End session
```

## Common Workflows

### Complete Patient Journey

```typescript
// 1. Patient inquiry via WhatsApp
// System automatically creates patient record and sends greeting

// 2. Schedule first appointment
const appointment = await fetch
