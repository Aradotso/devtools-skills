---
name: sesion-clinic-workflow
description: Mental health practice management platform with intelligent scheduling, WhatsApp automation, AFIP billing, video consultations, and Claude AI orchestration for Argentine psychologists
triggers:
  - "set up Sesión clinic workflow platform"
  - "integrate WhatsApp automation for patient appointments"
  - "configure AFIP electronic invoicing for psychology practice"
  - "implement Claude AI for clinical note summarization"
  - "build video consultation with LiveKit integration"
  - "create automated appointment reminders via WhatsApp"
  - "configure MercadoPago payment processing for sessions"
  - "set up multi-practitioner scheduling system"
---

# Sesión Clinic Workflow Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, automated WhatsApp messaging via Baileys, AFIP-compliant electronic invoicing, secure LiveKit video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit 5 frontend, Prisma ORM, and designed for Argentine healthcare compliance.

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 20.x
pnpm >= 8.x
postgres >= 15.x
redis >= 7.x
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies for monorepo
pnpm install

# Set up environment variables
cp .env.example .env
```

### Core Environment Configuration

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"

# Redis Cache
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="your_anthropic_api_key"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your_webhook_secret"

# AFIP (Argentine Tax Authority)
AFIP_CUIT="your_clinic_cuit"
AFIP_CERTIFICATE_PATH="./certs/afip-cert.pem"
AFIP_PRIVATE_KEY_PATH="./certs/afip-key.pem"
AFIP_PRODUCTION_MODE="false"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="your_mp_access_token"
STRIPE_SECRET_KEY="your_stripe_secret_key"

# LiveKit Video
LIVEKIT_API_KEY="your_livekit_api_key"
LIVEKIT_API_SECRET="your_livekit_api_secret"
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# App Configuration
JWT_SECRET="your_jwt_secret"
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
```

### Database Migration

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate deploy

# Seed initial data
pnpm prisma db seed
```

### Start Development Servers

```bash
# Start backend (NestJS)
cd apps/backend
pnpm dev

# Start frontend (SvelteKit) - in separate terminal
cd apps/frontend
pnpm dev
```

## Project Structure

```
psique-workflow-clinic/
├── apps/
│   ├── backend/          # NestJS API server
│   │   ├── src/
│   │   │   ├── agenda/        # Scheduling module
│   │   │   ├── whatsapp/      # Baileys integration
│   │   │   ├── billing/       # AFIP invoicing
│   │   │   ├── video/         # LiveKit module
│   │   │   ├── ai/            # Claude orchestration
│   │   │   └── patients/      # Patient management
│   │   └── prisma/
│   │       └── schema.prisma
│   └── frontend/         # SvelteKit UI
│       ├── src/
│       │   ├── routes/        # File-based routing
│       │   ├── lib/           # Shared components
│       │   └── stores/        # Svelte stores
└── packages/
    ├── shared/           # Shared types & utils
    └── config/           # Shared config
```

## Core Module Usage

### 1. Intelligent Appointment Scheduling

#### Create Appointment (Backend - NestJS)

```typescript
// apps/backend/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsappService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AgendaService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsappService
  ) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number; // minutes
    type: 'PRESENCIAL' | 'VIRTUAL' | 'EVALUACION';
  }) {
    // Check for scheduling conflicts
    const conflict = await this.prisma.appointment.findFirst({
      where: {
        practitionerId: data.practitionerId,
        status: { not: 'CANCELLED' },
        OR: [
          {
            startTime: {
              lte: data.startTime,
            },
            endTime: {
              gte: data.startTime,
            },
          },
        ],
      },
    });

    if (conflict) {
      throw new Error('Scheduling conflict detected');
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime: new Date(data.startTime.getTime() + data.duration * 60000),
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(appointment);

    return appointment;
  }

  async getAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number = 45
  ) {
    const dayStart = new Date(date.setHours(0, 0, 0, 0));
    const dayEnd = new Date(date.setHours(23, 59, 59, 999));

    const existingAppointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: dayStart, lte: dayEnd },
        status: { not: 'CANCELLED' },
      },
      orderBy: { startTime: 'asc' },
    });

    const workingHours = await this.prisma.practitionerSchedule.findFirst({
      where: {
        practitionerId,
        dayOfWeek: date.getDay(),
      },
    });

    if (!workingHours) return [];

    // Generate available slots
    const slots = [];
    let currentTime = new Date(
      date.setHours(
        workingHours.startHour,
        workingHours.startMinute,
        0,
        0
      )
    );
    const endTime = new Date(
      date.setHours(workingHours.endHour, workingHours.endMinute, 0, 0)
    );

    while (currentTime < endTime) {
      const slotEnd = new Date(currentTime.getTime() + duration * 60000);
      
      const hasConflict = existingAppointments.some(
        (apt) =>
          currentTime < new Date(apt.endTime) &&
          slotEnd > new Date(apt.startTime)
      );

      if (!hasConflict) {
        slots.push({
          startTime: new Date(currentTime),
          endTime: slotEnd,
        });
      }

      currentTime = slotEnd;
    }

    return slots;
  }
}
```

#### Frontend Scheduling Component (Svelte 5)

```svelte
<!-- apps/frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import type { Appointment } from '$lib/types';

  let appointments = $state<Appointment[]>([]);
  let selectedDate = $state(new Date());
  let loading = $state(false);

  async function loadAppointments() {
    loading = true;
    const response = await fetch(`/api/appointments?date=${selectedDate.toISOString()}`);
    appointments = await response.json();
    loading = false;
  }

  async function createAppointment(data: {
    patientId: string;
    startTime: Date;
    duration: number;
  }) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    if (response.ok) {
      await loadAppointments();
    }
  }

  onMount(() => {
    loadAppointments();
  });
</script>

<div class="agenda-container">
  <h1>Agenda Inteligente</h1>
  
  <input 
    type="date" 
    bind:value={selectedDate}
    onchange={loadAppointments}
  />

  {#if loading}
    <div class="spinner">Cargando...</div>
  {:else}
    <div class="appointments-list">
      {#each appointments as appointment}
        <div class="appointment-card">
          <div class="time">
            {new Date(appointment.startTime).toLocaleTimeString('es-AR', { 
              hour: '2-digit', 
              minute: '2-digit' 
            })}
          </div>
          <div class="patient-name">{appointment.patient.name}</div>
          <div class="type-badge" class:virtual={appointment.type === 'VIRTUAL'}>
            {appointment.type}
          </div>
        </div>
      {/each}
    </div>
  {/if}
</div>
```

### 2. WhatsApp Automation with Baileys

```typescript
// apps/backend/src/whatsapp/whatsapp.service.ts
import { Injectable, Logger } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsappService {
  private readonly logger = new Logger(WhatsappService.name);
  private sock: any;
  private connected = false;

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions'
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
          this.initialize();
        }
      } else if (connection === 'open') {
        this.logger.log('WhatsApp connection established');
        this.connected = true;
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    if (!this.connected) {
      this.logger.warn('WhatsApp not connected, skipping message');
      return;
    }

    const message = `
🗓️ *Confirmación de Turno*

Hola ${appointment.patient.name},

Tu sesión está confirmada para:
📅 ${new Date(appointment.startTime).toLocaleDateString('es-AR')}
🕐 ${new Date(appointment.startTime).toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}
👤 Profesional: ${appointment.practitioner.name}
📍 ${appointment.type === 'VIRTUAL' ? 'Videollamada (recibirás el link 10 min antes)' : appointment.practitioner.address}

Respondé *SI* para confirmar o *NO* para cancelar.
`;

    const phoneNumber = appointment.patient.phone.replace(/\D/g, '');
    const jid = `${phoneNumber}@s.whatsapp.net`;

    await this.sock.sendMessage(jid, { text: message });
  }

  async sendAppointmentReminder(appointment: any, hoursAhead: number) {
    const message = `
⏰ *Recordatorio de Turno*

Tu sesión es en ${hoursAhead} horas:
🕐 ${new Date(appointment.startTime).toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}

${appointment.type === 'VIRTUAL' ? '🔗 Link de videollamada: ' + appointment.videoRoomUrl : '📍 ' + appointment.practitioner.address}

¡Te esperamos!
`;

    const phoneNumber = appointment.patient.phone.replace(/\D/g, '');
    const jid = `${phoneNumber}@s.whatsapp.net`;

    await this.sock.sendMessage(jid, { text: message });
  }

  private async handleIncomingMessage(message: WAMessage) {
    if (!message.message) return;

    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text;
    const from = message.key.remoteJid;

    this.logger.log(`Received from ${from}: ${text}`);

    // Handle confirmation responses
    if (text?.toUpperCase() === 'SI' || text?.toUpperCase() === 'SÍ') {
      // Mark appointment as confirmed
      await this.sock.sendMessage(from, { 
        text: '✅ Turno confirmado. ¡Gracias!' 
      });
    } else if (text?.toUpperCase() === 'NO') {
      await this.sock.sendMessage(from, { 
        text: '❌ Entendido. ¿Querés reagendar? Contactate con la recepción.' 
      });
    }

    // Crisis detection keywords
    const crisisKeywords = ['crisis', 'urgencia', 'emergencia', 'ayuda'];
    if (crisisKeywords.some(keyword => text?.toLowerCase().includes(keyword))) {
      // Alert practitioner immediately
      await this.notifyPractitionerCrisis(from, text);
    }
  }

  private async notifyPractitionerCrisis(patientJid: string, message: string) {
    // Implementation for alerting practitioner
    this.logger.warn(`CRISIS ALERT from ${patientJid}: ${message}`);
  }
}
```

### 3. AFIP Electronic Invoicing

```typescript
// apps/backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import Afip from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERTIFICATE_PATH,
      key: process.env.AFIP_PRIVATE_KEY_PATH,
      production: process.env.AFIP_PRODUCTION_MODE === 'true',
    });
  }

  async generateInvoice(data: {
    appointmentId: string;
    patientCUIT: string;
    amount: number;
    invoiceType: 'A' | 'B' | 'C';
  }) {
    // Get last invoice number
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      1, // Punto de venta
      this.getInvoiceTypeCode(data.invoiceType)
    );

    const invoiceNumber = lastInvoice + 1;

    // Create invoice data
    const invoiceData = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
      Concepto: 3, // Servicios
      DocTipo: 80, // CUIT
      DocNro: data.patientCUIT.replace(/\D/g, ''),
      CbteDesde: invoiceNumber,
      CbteHasta: invoiceNumber,
      CbteFch: this.formatDate(new Date()),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    };

    // Request CAE (Código de Autorización Electrónica)
    const response = await this.afip.ElectronicBilling.createVoucher(
      invoiceData
    );

    if (response.CAE) {
      // Store invoice in database
      const invoice = await this.prisma.invoice.create({
        data: {
          appointmentId: data.appointmentId,
          invoiceNumber: `${String(1).padStart(4, '0')}-${String(invoiceNumber).padStart(8, '0')}`,
          invoiceType: data.invoiceType,
          cae: response.CAE,
          caeExpiration: this.parseDate(response.CAEFchVto),
          amount: data.amount,
          status: 'AUTHORIZED',
        },
      });

      return invoice;
    } else {
      throw new Error(`AFIP Error: ${response.Observaciones}`);
    }
  }

  private getInvoiceTypeCode(type: 'A' | 'B' | 'C'): number {
    const types = { A: 1, B: 6, C: 11 };
    return types[type];
  }

  private formatDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  private parseDate(dateStr: string): Date {
    const year = dateStr.substring(0, 4);
    const month = dateStr.substring(4, 6);
    const day = dateStr.substring(6, 8);
    return new Date(`${year}-${month}-${day}`);
  }

  async generateMonthlyReport(practitionerId: string, month: number, year: number) {
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
      include: {
        appointment: {
          include: {
            patient: true,
          },
        },
      },
    });

    const totalAmount = invoices.reduce((sum, inv) => sum + inv.amount, 0);

    return {
      month,
      year,
      totalInvoices: invoices.length,
      totalAmount,
      invoices,
    };
  }
}
```

### 4. Claude AI Orchestration

```typescript
// apps/backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class ClaudeService {
  private client: Anthropic;

  constructor(private prisma: PrismaService) {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNote(sessionId: string): Promise<string> {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (!session.clinicalNotes) {
      throw new Error('No clinical notes available');
    }

    const prompt = `
Eres un asistente especializado en psicología clínica en Argentina.

Resume las siguientes notas de sesión en un formato estructurado que incluya:
- Motivo de consulta
- Observaciones principales
- Intervenciones realizadas
- Objetivos terapéuticos
- Próximos pasos

Notas de sesión:
${session.clinicalNotes}

Mantené un lenguaje profesional pero accesible, adecuado para el sistema de salud argentino.
`;

    const message = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4.6',
      max_tokens: 2048,
      messages: [{ role: 'user', content: prompt }],
    });

    const summary = message.content[0].text;

    // Store AI-generated summary
    await this.prisma.session.update({
      where: { id: sessionId },
      data: { aiSummary: summary },
    });

    return summary;
  }

  async assistClinicalNote(partialNotes: string, patientHistory: string): Promise<string> {
    const prompt = `
Como asistente de psicología clínica, ayudá a completar estas notas de sesión.

Historia del paciente:
${patientHistory}

Notas parciales de la sesión actual:
${partialNotes}

Sugerí:
1. Áreas que podrían necesitar mayor documentación
2. Posibles patrones observables basados en la historia
3. Intervenciones terapéuticas apropiadas según el contexto argentino

Recordá que estas son solo sugerencias para el profesional.
`;

    const message = await this.client.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4.6',
      max_tokens: 1024,
      messages: [{ role: 'user', content: prompt }],
    });

    return message.content[0].text;
  }

  async predictTherapeuticOutcome(patientId: string): Promise<{
    prediction: string;
    confidence: number;
    factors: string[];
  }> {
    const sessions = await this.prisma.session.findMany({
      where: { patientId },
      orderBy: { sessionDate: 'desc' },
      take: 20,
      include: {
        patient: true,
      },
    });

    const sessionSummaries = sessions
      .map(
        (s) =>
          `Fecha: ${s.sessionDate.toISOString()}\nNotas: ${s.clinicalNotes}`
      )
      .join('\n\n');

    const prompt = `
Basándote en el historial de sesiones, analizá el progreso terapéutico y predecí la probabilidad de alcanzar los objetivos terapéuticos.

Historial de sesiones:
${sessionSummaries}

Proporcioná:
1. Una predicción de resultado (FAVORABLE, NEUTRAL, REQUIERE_AJUSTE)
2. Nivel de confianza (0-100)
3. Factores clave que influyen en el pronóstico
4. Recomendaciones para optimizar el tratamiento

Formato de respuesta en JSON.
`;

    const message = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4.6',
      max_tokens: 2048,
      messages: [{ role: 'user', content: prompt }],
    });

    const responseText = message.content[0].text;
    const jsonMatch = responseText.match(/\{[\s\S]*\}/);
    
    if (jsonMatch) {
      return JSON.parse(jsonMatch[0]);
    }

    throw new Error('Failed to parse AI response');
  }
}
```

### 5. LiveKit Video Consultations

```typescript
// apps/backend/src/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class VideoService {
  constructor(private prisma: PrismaService) {}

  async createVideoRoom(appointmentId: string): Promise<{
    roomName: string;
    practitionerToken: string;
    patientToken: string;
  }> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (!appointment) {
      throw new Error('Appointment not found');
    }

    const roomName = `session-${appointmentId}`;

    // Create tokens for practitioner and patient
    const practitionerToken = this.generateToken(
      roomName,
      `practitioner-${appointment.practitionerId}`,
      {
        canPublish: true,
        canPublishData: true,
        canSubscribe: true,
        roomRecord: true, // Can record session
      }
    );

    const patientToken = this.generateToken(
      roomName,
      `patient-${appointment.patientId}`,
      {
        canPublish: true,
        canPublishData: true,
        canSubscribe: true,
        roomRecord: false,
      }
    );

    // Update appointment with video room info
    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: {
        videoRoomName: roomName,
        videoRoomUrl: `${process.env.APP_URL}/video/${roomName}`,
      },
    });

    return {
      roomName,
      practitionerToken,
      patientToken,
    };
  }

  private generateToken(
    roomName: string,
    identity: string,
    permissions: {
      canPublish: boolean;
      canPublishData: boolean;
      canSubscribe: boolean;
      roomRecord: boolean;
    }
  ): string {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity,
      }
    );

    at.addGrant({
      roomJoin: true,
      room: roomName,
      ...permissions,
    });

    return at.toJwt();
  }
}
```

#### Video Room Component (Svelte 5)

```svelte
<!-- apps/frontend/src/routes/video/[roomName]/+page.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { page } from '$app/stores';
  import {
    Room,
    RoomEvent,
    Track,
    type RemoteParticipant,
    type RemoteTrack,
  } from 'livekit-client';

  const roomName = $page.params.roomName;
  let videoContainer: HTMLDivElement;
  let room: Room;
  let localVideoTrack = $state<HTMLVideoElement | null>(null);
  let remoteVideoTrack = $state<HTMLVideoElement | null>(null);
  let connected = $state(false);
  let audioEnabled = $state(true);
  let videoEnabled = $state(true);

  async function connectToRoom(token: string) {
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: {
          width: 1280,
          height: 720,
          frameRate: 24,
        },
      },
    });

    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.TrackUnsubscribed, handleTrackUnsubscribed);
    room.on(RoomEvent.Disconnected, handleDisconnect);

    await room.connect(process.env.PUBLIC_LIVEKIT_WS_URL, token);

    connected = true;

    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
  }

  function handleTrackSubscribed(
    track: RemoteTrack,
    publication: any,
    participant: RemoteParticipant
  ) {
    if (track.kind === Track.Kind.Video) {
      const element = track.attach();
      element.style.width = '100%';
      element.style.height = '100%';
      element.style.objectFit = 'cover';
      remoteVideoTrack = element;
    }
  }

  function handleTrackUnsubscribed(track: RemoteTrack) {
    track.detach();
  }

  function handleDisconnect() {
    connected = false;
  }

  async function toggleAudio() {
    audioEnabled = !audioEnabled;
    await room.localParticipant.setMicrophoneEnabled(audioEnabled);
  }

  async function toggleVideo() {
    videoEnabled = !videoEnabled;
    await room.localParticipant.setCameraEnabled(videoEnabled);
  }

  async function endCall() {
    await room.disconnect();
    window.close();
  }

  onMount(async () => {
    // Fetch token from API
    const response = await fetch(`/api/video/token/${roomName}`);
    const { token } = await response.json();
    await connectToRoom(token);
  });

  onDestroy(() => {
    if (room) {
      room.disconnect();
    }
  });
</script>

<div class="video-room">
  <div class="video-container" bind:this={videoContainer}>
    {#if remoteVideoTrack}
      {@html remoteVideoTrack.outerHTML}
    {:else}
      <div class="waiting-message">Esperando al otro participante...</div>
    {/if}
  </div>

  <div class="local-video
