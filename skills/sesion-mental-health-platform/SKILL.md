---
name: sesion-mental-health-platform
description: Orchestration platform for psychology clinics with intelligent scheduling, WhatsApp automation, AFIP billing, video consultations, and Claude AI integration for Argentine practitioners
triggers:
  - "help me set up Sesión for a psychology clinic"
  - "how do I integrate WhatsApp automation with Sesión"
  - "configure AFIP electronic invoicing in Sesión"
  - "set up Claude AI for clinical notes in Sesión"
  - "implement video consultation workflows"
  - "create appointment scheduling with Sesión"
  - "integrate Mercado Pago billing"
  - "configure patient journey pipelines"
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS orchestration platform for psychology clinics in Argentina, providing intelligent appointment scheduling, WhatsApp automation via Baileys, AFIP-compliant electronic invoicing, secure video consultations with LiveKit, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit 5 frontend, Prisma ORM, and designed for Argentine healthcare compliance.

## Installation & Setup

### Prerequisites

```bash
node >= 18.0.0
pnpm >= 8.0.0
postgresql >= 14
redis >= 6.2
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic
pnpm install
```

### Environment Configuration

Create `.env` file in project root:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Authentication
JWT_SECRET="your-jwt-secret-here"
SESSION_SECRET="your-session-secret"

# Anthropic AI
ANTHROPIC_API_KEY="sk-ant-api03-..."
ANTHROPIC_OPUS_MODEL="claude-opus-4.6"
ANTHROPIC_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_QR_TIMEOUT=60000

# LiveKit Video
LIVEKIT_API_KEY="APIxxxxxx"
LIVEKIT_API_SECRET="secretxxxxx"
LIVEKIT_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax)
AFIP_CUIT="your-cuit-number"
AFIP_CERTIFICATE_PATH="./certs/afip-cert.crt"
AFIP_PRIVATE_KEY_PATH="./certs/afip-key.key"
AFIP_ENVIRONMENT="production" # or "staging"

# Mercado Pago
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."
MERCADOPAGO_PUBLIC_KEY="APP_USR-..."

# Stripe (optional)
STRIPE_SECRET_KEY="sk_live_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# Frontend
PUBLIC_API_URL="http://localhost:3000"
PUBLIC_WS_URL="ws://localhost:3000"
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
# Backend (NestJS)
cd apps/backend
pnpm dev

# Frontend (SvelteKit)
cd apps/frontend
pnpm dev
```

## Core Architecture

### Project Structure

```
psique-workflow-clinic/
├── apps/
│   ├── backend/          # NestJS microservices
│   │   ├── src/
│   │   │   ├── agenda/   # Scheduling module
│   │   │   ├── whatsapp/ # WhatsApp automation
│   │   │   ├── billing/  # AFIP invoicing
│   │   │   ├── video/    # LiveKit integration
│   │   │   ├── ai/       # Claude orchestration
│   │   │   └── patients/ # Patient management
│   └── frontend/         # SvelteKit 5 UI
│       ├── src/
│       │   ├── routes/   # SvelteKit pages
│       │   ├── lib/      # Shared components
│       │   └── stores/   # Svelte stores
├── prisma/
│   └── schema.prisma     # Database schema
└── packages/
    └── shared/           # Shared types/utils
```

## Key Modules & Usage

### 1. Appointment Scheduling (Agenda)

#### Creating Appointments

```typescript
// apps/backend/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number; // minutes
    sessionType: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        startTime: {
          gte: data.startTime,
          lt: new Date(data.startTime.getTime() + data.duration * 60000),
        },
        status: { not: 'cancelled' },
      },
    });

    if (conflicts.length > 0) {
      throw new Error('Time slot conflict detected');
    }

    return this.prisma.appointment.create({
      data: {
        ...data,
        endTime: new Date(data.startTime.getTime() + data.duration * 60000),
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });
  }

  async findAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number = 45,
  ) {
    const workingHours = await this.prisma.practitionerSchedule.findFirst({
      where: {
        practitionerId,
        dayOfWeek: date.getDay(),
      },
    });

    if (!workingHours) return [];

    const appointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: {
          gte: new Date(date.setHours(0, 0, 0, 0)),
          lt: new Date(date.setHours(23, 59, 59, 999)),
        },
        status: { not: 'cancelled' },
      },
      orderBy: { startTime: 'asc' },
    });

    // Calculate available slots
    const slots = [];
    let currentTime = new Date(
      date.setHours(workingHours.startHour, workingHours.startMinute),
    );
    const endTime = new Date(
      date.setHours(workingHours.endHour, workingHours.endMinute),
    );

    while (currentTime < endTime) {
      const slotEnd = new Date(currentTime.getTime() + duration * 60000);
      const isAvailable = !appointments.some(
        (apt) => currentTime >= apt.startTime && currentTime < apt.endTime,
      );

      if (isAvailable) {
        slots.push({
          startTime: new Date(currentTime),
          endTime: new Date(slotEnd),
        });
      }

      currentTime = slotEnd;
    }

    return slots;
  }
}
```

#### Frontend Svelte Component

```svelte
<!-- apps/frontend/src/lib/components/AgendaCalendar.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  let selectedDate = $state(new Date());
  let availableSlots = $state([]);
  let loading = $state(false);

  async function loadSlots() {
    loading = true;
    const response = await fetch(
      `/api/agenda/available-slots?practitionerId=${practitionerId}&date=${selectedDate.toISOString()}&duration=45`,
    );
    availableSlots = await response.json();
    loading = false;
  }

  async function bookAppointment(slot: any) {
    const response = await fetch('/api/agenda/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        patientId: $currentPatient.id,
        practitionerId,
        startTime: slot.startTime,
        duration: 45,
        sessionType: 'virtual',
      }),
    });

    if (response.ok) {
      alert('Cita agendada exitosamente');
      await loadSlots();
    }
  }

  $effect(() => {
    loadSlots();
  });
</script>

<div class="calendar-container">
  <h2>Agenda Disponible</h2>
  <input type="date" bind:value={selectedDate} onchange={loadSlots} />

  {#if loading}
    <p>Cargando horarios...</p>
  {:else if availableSlots.length === 0}
    <p>No hay horarios disponibles para esta fecha.</p>
  {:else}
    <div class="slots-grid">
      {#each availableSlots as slot}
        <button
          class="slot-button"
          onclick={() => bookAppointment(slot)}
        >
          {new Date(slot.startTime).toLocaleTimeString('es-AR', {
            hour: '2-digit',
            minute: '2-digit'
          })}
        </button>
      {/each}
    </div>
  {/if}
</div>

<style>
  .slots-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 1rem;
  }
  .slot-button {
    padding: 1rem;
    background: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
</style>
```

### 2. WhatsApp Automation (Baileys)

#### WhatsApp Service Implementation

```typescript
// apps/backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  async onModuleInit() {
    await this.connectWhatsApp();
  }

  async connectWhatsApp() {
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
    this.sock.ev.on('messages.upsert', async (m) => {
      const message = m.messages[0];
      if (message.key.fromMe) return;

      await this.handleIncomingMessage(message);
    });
  }

  async sendAppointmentReminder(
    phoneNumber: string,
    appointment: {
      patientName: string;
      practitionerName: string;
      startTime: Date;
      sessionType: string;
    },
  ) {
    const formattedNumber = phoneNumber.replace(/\D/g, '') + '@s.whatsapp.net';
    const message = `Hola ${appointment.patientName}! 👋

Recordatorio de tu sesión ${appointment.sessionType}:
📅 Fecha: ${appointment.startTime.toLocaleDateString('es-AR')}
🕐 Hora: ${appointment.startTime.toLocaleTimeString('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
    })}
👨‍⚕️ Profesional: ${appointment.practitionerName}

¿Confirmas tu asistencia? Responde:
1️⃣ para confirmar
2️⃣ para cancelar`;

    await this.sock.sendMessage(formattedNumber, { text: message });
  }

  async sendInvoice(phoneNumber: string, invoiceUrl: string, amount: number) {
    const formattedNumber = phoneNumber.replace(/\D/g, '') + '@s.whatsapp.net';
    const message = `📄 Nueva factura disponible

Monto: $${amount.toLocaleString('es-AR')}
Descarga tu factura aquí: ${invoiceUrl}

Gracias por confiar en nosotros! 💚`;

    await this.sock.sendMessage(formattedNumber, { text: message });
  }

  private async handleIncomingMessage(message: any) {
    const text = message.message?.conversation?.toLowerCase() || '';
    const from = message.key.remoteJid;

    // Crisis keyword detection
    const crisisKeywords = [
      'suicidio',
      'matarme',
      'no puedo más',
      'crisis',
      'urgencia',
    ];
    if (crisisKeywords.some((kw) => text.includes(kw))) {
      await this.sock.sendMessage(from, {
        text: '🚨 Detectamos que podrías necesitar ayuda urgente. Tu psicólogo será notificado inmediatamente.\n\nSi estás en crisis, llama al 135 (Línea de Emergencias de Salud Mental).',
      });

      // Notify practitioner (implement notification service)
      // await this.notificationService.alertPractitioner(...)
    }

    // Appointment confirmation handling
    if (text === '1' || text.includes('confirm')) {
      // Update appointment status
      await this.sock.sendMessage(from, {
        text: '✅ Sesión confirmada! Te esperamos.',
      });
    }
  }
}
```

#### Schedule Automatic Reminders

```typescript
// apps/backend/src/whatsapp/whatsapp-scheduler.service.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from './whatsapp.service';

@Injectable()
export class WhatsAppSchedulerService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    const tomorrowEnd = new Date(tomorrow);
    tomorrowEnd.setHours(23, 59, 59);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: { gte: tomorrow, lte: tomorrowEnd },
        status: 'scheduled',
        reminderSent24h: false,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.patient.phone, {
        patientName: apt.patient.firstName,
        practitionerName: apt.practitioner.name,
        startTime: apt.startTime,
        sessionType: apt.sessionType,
      });

      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent24h: true },
      });
    }
  }

  @Cron(CronExpression.EVERY_10_MINUTES)
  async send2HourReminders() {
    const twoHoursFromNow = new Date();
    twoHoursFromNow.setHours(twoHoursFromNow.getHours() + 2);
    const rangeEnd = new Date(twoHoursFromNow);
    rangeEnd.setMinutes(rangeEnd.getMinutes() + 10);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: { gte: twoHoursFromNow, lte: rangeEnd },
        status: 'scheduled',
        reminderSent2h: false,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.patient.phone, {
        patientName: apt.patient.firstName,
        practitionerName: apt.practitioner.name,
        startTime: apt.startTime,
        sessionType: apt.sessionType,
      });

      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent2h: true },
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

#### AFIP Service Implementation

```typescript
// apps/backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as AfipSDK from '@afipsdk/afip.js';
import * as fs from 'fs';

@Injectable()
export class AfipService {
  private afip: any;

  constructor() {
    this.afip = new AfipSDK({
      CUIT: parseInt(process.env.AFIP_CUIT),
      production: process.env.AFIP_ENVIRONMENT === 'production',
      cert: fs.readFileSync(process.env.AFIP_CERTIFICATE_PATH, 'utf8'),
      key: fs.readFileSync(process.env.AFIP_PRIVATE_KEY_PATH, 'utf8'),
    });
  }

  async generateInvoice(data: {
    patientCuit: string;
    amount: number;
    sessionDate: Date;
    concept: string;
    invoiceType: 'A' | 'B' | 'C';
  }) {
    const invoiceTypeCode = { A: 1, B: 6, C: 11 }[data.invoiceType];

    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      1, // Punto de venta
      invoiceTypeCode,
    );

    const invoiceNumber = lastInvoice + 1;

    const invoice = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: invoiceTypeCode,
      Concepto: 2, // Servicios
      DocTipo: 80, // CUIT
      DocNro: parseInt(data.patientCuit.replace(/\D/g, '')),
      CbteDesde: invoiceNumber,
      CbteHasta: invoiceNumber,
      CbteFch: this.formatDate(data.sessionDate),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      FchServDesde: this.formatDate(data.sessionDate),
      FchServHasta: this.formatDate(data.sessionDate),
      FchVtoPago: this.formatDate(data.sessionDate),
      MonId: 'PES',
      MonCotiz: 1,
    };

    const result = await this.afip.ElectronicBilling.createVoucher(invoice);

    return {
      invoiceNumber: `${String(1).padStart(5, '0')}-${String(invoiceNumber).padStart(8, '0')}`,
      cae: result.CAE,
      caeExpiration: result.CAEFchVto,
      afipResult: result,
    };
  }

  private formatDate(date: Date): string {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}${month}${day}`;
  }

  async getInvoicePDF(cae: string, invoiceNumber: string): Promise<Buffer> {
    // Generate PDF with invoice data and AFIP QR code
    // Implementation depends on your PDF library (e.g., PDFKit)
    return Buffer.from('');
  }
}
```

#### Billing Controller

```typescript
// apps/backend/src/billing/billing.controller.ts
import { Controller, Post, Body, Get, Param } from '@nestjs/common';
import { AfipService } from './afip.service';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Controller('billing')
export class BillingController {
  constructor(
    private afip: AfipService,
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  @Post('generate-invoice')
  async generateInvoice(
    @Body()
    data: {
      appointmentId: string;
      invoiceType: 'A' | 'B' | 'C';
    },
  ) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: data.appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (!appointment) {
      throw new Error('Appointment not found');
    }

    // Generate AFIP invoice
    const invoice = await this.afip.generateInvoice({
      patientCuit: appointment.patient.cuit,
      amount: appointment.practitioner.sessionPrice,
      sessionDate: appointment.startTime,
      concept: `Consulta psicológica - ${appointment.sessionType}`,
      invoiceType: data.invoiceType,
    });

    // Save to database
    const savedInvoice = await this.prisma.invoice.create({
      data: {
        appointmentId: appointment.id,
        patientId: appointment.patientId,
        practitionerId: appointment.practitionerId,
        invoiceNumber: invoice.invoiceNumber,
        amount: appointment.practitioner.sessionPrice,
        cae: invoice.cae,
        caeExpiration: new Date(invoice.caeExpiration),
        invoiceType: data.invoiceType,
        status: 'issued',
      },
    });

    // Send via WhatsApp
    const pdfUrl = `${process.env.PUBLIC_API_URL}/billing/invoice/${savedInvoice.id}/pdf`;
    await this.whatsapp.sendInvoice(
      appointment.patient.phone,
      pdfUrl,
      savedInvoice.amount,
    );

    return savedInvoice;
  }

  @Get('invoice/:id/pdf')
  async getInvoicePDF(@Param('id') id: string) {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id },
      include: {
        patient: true,
        practitioner: true,
        appointment: true,
      },
    });

    const pdf = await this.afip.getInvoicePDF(invoice.cae, invoice.invoiceNumber);
    return pdf;
  }
}
```

### 4. Video Consultations (LiveKit)

#### LiveKit Service

```typescript
// apps/backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  async createRoomToken(
    roomName: string,
    participantIdentity: string,
    participantName: string,
  ): Promise<string> {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: participantIdentity,
        name: participantName,
      },
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

  async createVideoSession(appointmentId: string) {
    const roomName = `session-${appointmentId}`;
    return { roomName, livekitUrl: process.env.LIVEKIT_URL };
  }
}
```

#### Svelte Video Component

```svelte
<!-- apps/frontend/src/lib/components/VideoConsultation.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Room, RoomEvent, Track } from 'livekit-client';

  export let appointmentId: string;
  export let participantName: string;

  let room: Room;
  let localVideoElement: HTMLVideoElement;
  let remoteVideoElement: HTMLVideoElement;
  let connected = $state(false);
  let error = $state('');

  async function joinRoom() {
    try {
      const response = await fetch('/api/video/join', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ appointmentId, participantName }),
      });

      const { token, roomName, livekitUrl } = await response.json();

      room = new Room({
        adaptiveStream: true,
        dynacast: true,
      });

      room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
        if (track.kind === Track.Kind.Video) {
          track.attach(remoteVideoElement);
        }
      });

      room.on(RoomEvent.LocalTrackPublished, (publication) => {
        if (publication.track?.kind === Track.Kind.Video) {
          publication.track.attach(localVideoElement);
        }
      });

      await room.connect(livekitUrl, token);
      connected = true;

      // Publish local tracks
      await room.localParticipant.enableCameraAndMicrophone();
    } catch (e) {
      error = `Error al conectar: ${e.message}`;
      console.error(e);
    }
  }

  async function leaveRoom() {
    await room?.disconnect();
    connected = false;
  }

  onMount(() => {
    return () => {
      leaveRoom();
    };
  });
</script>

<div class="video-consultation">
  {#if !connected}
    <button onclick={joinRoom}>Iniciar Videollamada</button>
    {#if error}
      <p class="error">{error}</p>
    {/if}
  {:else}
    <div class="video-grid">
      <div class="remote-video">
        <video bind:this={remoteVideoElement} autoplay playsinline />
        <span class="label">Paciente</span>
      </div>
      <div class="local-video">
        <video bind:this={localVideoElement} autoplay playsinline muted />
        <span class="label">Tú</span>
      </div>
    </div>
    <button onclick={leaveRoom} class="disconnect">Finalizar</button>
  {/if}
</div>

<style>
  .video-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  .remote-video, .local-video {
    position: relative;
  }
  video {
    width: 100%;
    height: auto;
    border-radius: 8px;
  }
  .local-video {
    position: absolute;
    bottom: 1rem;
    right: 1rem;
    width: 200px;
  }
</style>
```

### 5. Claude AI Integration

#### AI Service with Model Orchestration

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
    sessionNotes: string,
    patientHistory: string[],
  ): Promise<string> {
    // Use Opus for deep analysis
    const message = await this.anthropic.messages.create({
      model: process.env.ANTHROPIC_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: `Como psicólogo clínico experto, analiza las siguientes notas de sesión y el historial del paciente. Genera un resumen profesional destacando:
1. Temas principales abordados
2. Progreso terapéutico observable
3. Áreas de preocupación
4. Recomendaciones para próximas sesiones

Historial previo:
${patientHistory.join('\n\n')}

Notas de sesión actual:
${sessionNotes}

Resumen clínico:`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async generateSessionReport(
    appointmentId: string,
    sessionNotes: string,
  ): Promise<string> {
    // Use Sonnet for quick report generation
    const message = await this.anthropic.messages.create({
      model: process.env.ANTHROPIC_SONNET_MODEL || 'claude-sonnet-4',
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `Genera un informe de sesión profesional basado en las siguientes notas clí
