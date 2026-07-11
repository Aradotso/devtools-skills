---
name: sesion-mental-health-clinic-platform
description: Orchestrate psychology clinic operations with AI-powered scheduling, WhatsApp automation, AFIP billing, and secure video consultations for Argentine mental health practitioners
triggers:
  - how do I set up Sesión for a psychology clinic
  - integrate WhatsApp automation with Sesión
  - configure AFIP electronic invoicing in Sesión
  - implement Claude AI for clinical notes in Sesión
  - set up video consultations with Sesión
  - automate appointment reminders with Sesión
  - deploy Sesión mental health platform
  - configure Sesión for Argentine tax compliance
---

# Sesión Mental Health Clinic Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for Argentine psychology clinics that unifies appointment scheduling, automated WhatsApp messaging, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Architecture Overview

Sesión is built as a microservices ecosystem with:

- **Frontend**: SvelteKit 5 with TailwindCSS
- **Backend**: NestJS microservices
- **Database**: PostgreSQL with Prisma ORM
- **Cache/Session**: Redis
- **Search**: Elasticsearch
- **Storage**: MinIO (S3-compatible)
- **Message Queue**: Apache Kafka
- **WhatsApp**: Baileys library
- **Video**: LiveKit
- **AI**: Anthropic Claude Opus 4.6 & Sonnet 4.6
- **Payments**: MercadoPago (Argentina), Stripe (international)

## Installation

### Prerequisites

```bash
# Required tools
node >= 20.0.0
docker >= 24.0.0
docker-compose >= 2.20.0
pnpm >= 8.0.0
```

### Clone and Setup

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env

# Start infrastructure services
docker-compose up -d postgres redis elasticsearch minio kafka

# Run database migrations
pnpm prisma:migrate

# Start development servers
pnpm dev
```

## Environment Configuration

Create a `.env` file with the following structure:

```env
# Database
DATABASE_URL="postgresql://sesion:password@localhost:5432/sesion_db"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=sk-ant-your-key-here
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET=your-webhook-secret

# LiveKit Video
LIVEKIT_API_KEY=your-livekit-key
LIVEKIT_API_SECRET=your-livekit-secret
LIVEKIT_URL=wss://your-livekit-instance.com

# AFIP (Argentina Tax Authority)
AFIP_CUIT=your-clinic-cuit
AFIP_CERTIFICATE_PATH="./afip-cert.pem"
AFIP_PRIVATE_KEY_PATH="./afip-key.pem"
AFIP_ENVIRONMENT="testing" # or "production"

# MercadoPago
MERCADOPAGO_ACCESS_TOKEN=your-mp-token
MERCADOPAGO_PUBLIC_KEY=your-mp-public-key

# Stripe (optional, for international)
STRIPE_SECRET_KEY=sk_test_your-stripe-key
STRIPE_WEBHOOK_SECRET=whsec_your-webhook-secret

# Application
JWT_SECRET=your-jwt-secret
FRONTEND_URL=http://localhost:5173
BACKEND_URL=http://localhost:3000
```

## Key Components

### 1. Appointment Scheduling

#### Create Appointment (NestJS Service)

```typescript
// apps/backend/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AgendaService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    endTime: Date;
    sessionType: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Check for conflicts
    const conflict = await this.prisma.appointment.findFirst({
      where: {
        practitionerId: data.practitionerId,
        OR: [
          {
            startTime: { lte: data.startTime },
            endTime: { gt: data.startTime },
          },
          {
            startTime: { lt: data.endTime },
            endTime: { gte: data.endTime },
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
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Schedule WhatsApp reminders
    await this.scheduleReminders(appointment);

    return appointment;
  }

  private async scheduleReminders(appointment: any) {
    const reminderTimes = [
      { hours: 24, message: '24 horas antes de tu sesión' },
      { hours: 2, message: '2 horas antes de tu sesión' },
    ];

    for (const reminder of reminderTimes) {
      const sendAt = new Date(
        appointment.startTime.getTime() - reminder.hours * 60 * 60 * 1000
      );

      await this.whatsapp.scheduleMessage({
        to: appointment.patient.phone,
        message: `Hola ${appointment.patient.firstName}, te recordamos tu sesión con ${appointment.practitioner.name} el ${appointment.startTime.toLocaleDateString('es-AR')} a las ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}. Responde SÍ para confirmar.`,
        scheduledFor: sendAt,
        appointmentId: appointment.id,
      });
    }
  }
}
```

#### SvelteKit Appointment Component

```svelte
<!-- apps/frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import type { Appointment } from '$lib/types';
  
  let appointments: Appointment[] = $state([]);
  let selectedDate = $state(new Date());
  let loading = $state(false);

  async function loadAppointments(date: Date) {
    loading = true;
    try {
      const response = await fetch(`/api/appointments?date=${date.toISOString()}`);
      appointments = await response.json();
    } finally {
      loading = false;
    }
  }

  async function createAppointment(data: Partial<Appointment>) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    
    if (response.ok) {
      await loadAppointments(selectedDate);
    }
  }

  onMount(() => loadAppointments(selectedDate));
</script>

<div class="agenda-container">
  <header class="flex justify-between items-center mb-6">
    <h1 class="text-2xl font-bold">Agenda</h1>
    <button 
      class="btn-primary"
      onclick={() => createAppointment({ 
        startTime: new Date(),
        sessionType: 'presencial' 
      })}
    >
      Nueva Sesión
    </button>
  </header>

  <div class="calendar-grid">
    {#each appointments as appt}
      <div class="appointment-card" class:virtual={appt.sessionType === 'virtual'}>
        <div class="time">{appt.startTime.toLocaleTimeString('es-AR')}</div>
        <div class="patient">{appt.patient.fullName}</div>
        <div class="type">{appt.sessionType}</div>
        {#if appt.status === 'pending_confirmation'}
          <span class="badge badge-warning">Pendiente confirmación</span>
        {/if}
      </div>
    {/each}
  </div>
</div>

<style>
  .agenda-container {
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }

  .calendar-grid {
    display: grid;
    gap: 1rem;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  }

  .appointment-card {
    border: 1px solid #e5e7eb;
    border-radius: 0.5rem;
    padding: 1rem;
    background: white;
  }

  .appointment-card.virtual {
    border-left: 4px solid #3b82f6;
  }
</style>
```

### 2. WhatsApp Automation

#### WhatsApp Service Implementation

```typescript
// apps/backend/src/whatsapp/whatsapp.service.ts
import { Injectable } from '@nestjs/common';
import makeWASocket, { DisconnectReason, useMultiFileAuthState } from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService {
  private sock: any;
  private connectionState: 'disconnected' | 'connecting' | 'connected' = 'disconnected';

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
          (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.initialize();
        }
      } else if (connection === 'open') {
        this.connectionState = 'connected';
        console.log('WhatsApp connected successfully');
      }
    });

    // Handle incoming messages
    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      for (const msg of messages) {
        if (!msg.key.fromMe) {
          await this.handleIncomingMessage(msg);
        }
      }
    });
  }

  async sendMessage(to: string, message: string) {
    // Format phone number to WhatsApp format
    const formattedNumber = to.replace(/\D/g, '') + '@s.whatsapp.net';
    
    await this.sock.sendMessage(formattedNumber, { text: message });
  }

  async scheduleMessage(data: {
    to: string;
    message: string;
    scheduledFor: Date;
    appointmentId: string;
  }) {
    // Store in database for scheduled sending
    await this.prisma.scheduledMessage.create({
      data: {
        recipientPhone: data.to,
        message: data.message,
        scheduledFor: data.scheduledFor,
        appointmentId: data.appointmentId,
        status: 'pending',
      },
    });
  }

  private async handleIncomingMessage(msg: any) {
    const from = msg.key.remoteJid;
    const text = msg.message?.conversation || msg.message?.extendedTextMessage?.text;

    if (!text) return;

    // Crisis keyword detection
    const crisisKeywords = ['suicidio', 'matarme', 'no aguanto', 'crisis'];
    const isCrisis = crisisKeywords.some(keyword => 
      text.toLowerCase().includes(keyword)
    );

    if (isCrisis) {
      await this.escalateCrisis(from, text);
      return;
    }

    // Appointment confirmation parsing
    if (text.toLowerCase().includes('sí') || text.toLowerCase().includes('confirmo')) {
      await this.confirmAppointment(from);
    }
  }

  private async escalateCrisis(phone: string, message: string) {
    const patient = await this.prisma.patient.findUnique({
      where: { phone },
      include: { practitioner: true },
    });

    if (patient?.practitioner) {
      // Send urgent notification to practitioner
      await this.sendMessage(
        patient.practitioner.phone,
        `⚠️ ALERTA: ${patient.firstName} ha enviado un mensaje que requiere atención inmediata. Por favor contacta al paciente cuanto antes.`
      );

      // Log crisis event
      await this.prisma.crisisEvent.create({
        data: {
          patientId: patient.id,
          message,
          escalatedAt: new Date(),
        },
      });
    }
  }

  private async confirmAppointment(phone: string) {
    const pendingAppointment = await this.prisma.appointment.findFirst({
      where: {
        patient: { phone },
        status: 'pending_confirmation',
        startTime: { gte: new Date() },
      },
      orderBy: { startTime: 'asc' },
    });

    if (pendingAppointment) {
      await this.prisma.appointment.update({
        where: { id: pendingAppointment.id },
        data: { status: 'confirmed' },
      });

      await this.sendMessage(
        phone,
        '✅ Tu sesión ha sido confirmada. Te esperamos!'
      );
    }
  }
}
```

### 3. AFIP Electronic Invoicing

#### AFIP Integration Service

```typescript
// apps/backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import * as soap from 'soap';
import { SignedXml } from 'xml-crypto';

@Injectable()
export class AfipService {
  private wsaaUrl: string;
  private wsfevUrl: string;
  private certificate: Buffer;
  private privateKey: Buffer;

  constructor() {
    this.wsaaUrl = process.env.AFIP_ENVIRONMENT === 'production'
      ? 'https://wsaa.afip.gov.ar/ws/services/LoginCms'
      : 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms';

    this.wsfevUrl = process.env.AFIP_ENVIRONMENT === 'production'
      ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx'
      : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx';

    this.certificate = readFileSync(process.env.AFIP_CERTIFICATE_PATH);
    this.privateKey = readFileSync(process.env.AFIP_PRIVATE_KEY_PATH);
  }

  async authenticate() {
    const tra = this.createTRA();
    const cms = this.signTRA(tra);

    const client = await soap.createClientAsync(this.wsaaUrl);
    const result = await client.loginCmsAsync({ in0: cms });

    return {
      token: result[0].return.token,
      sign: result[0].return.sign,
      expirationTime: result[0].return.expirationTime,
    };
  }

  async generateInvoice(data: {
    patientName: string;
    patientCuit: string;
    amount: number;
    sessionDate: Date;
    invoiceType: 'A' | 'B' | 'C';
  }) {
    const auth = await this.authenticate();

    const client = await soap.createClientAsync(this.wsfevUrl);

    // Get next invoice number
    const lastInvoice = await client.FECompUltimoAutorizadoAsync({
      Auth: { Token: auth.token, Sign: auth.sign, Cuit: process.env.AFIP_CUIT },
      PtoVta: 1,
      CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
    });

    const nextNumber = lastInvoice[0].FECompUltimoAutorizadoResult.CbteNro + 1;

    // Create invoice request
    const invoiceRequest = {
      Auth: { Token: auth.token, Sign: auth.sign, Cuit: process.env.AFIP_CUIT },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: 1,
          CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
        },
        FeDetReq: {
          FECAEDetRequest: {
            Concepto: 2, // Servicios
            DocTipo: 80, // CUIT
            DocNro: data.patientCuit,
            CbteDesde: nextNumber,
            CbteHasta: nextNumber,
            CbteFch: data.sessionDate.toISOString().split('T')[0].replace(/-/g, ''),
            ImpTotal: data.amount,
            ImpTotConc: 0,
            ImpNeto: data.amount,
            ImpOpEx: 0,
            ImpIVA: 0,
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
          },
        },
      },
    };

    const result = await client.FECAESolicitarAsync(invoiceRequest);

    if (result[0].FECAESolicitarResult.FeCabResp.Resultado === 'A') {
      const cae = result[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse.CAE;
      const caeExpiration = result[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse.CAEFchVto;

      return {
        invoiceNumber: nextNumber,
        cae,
        caeExpiration,
        success: true,
      };
    } else {
      throw new Error('AFIP invoice generation failed');
    }
  }

  private getInvoiceTypeCode(type: 'A' | 'B' | 'C'): number {
    return { A: 1, B: 6, C: 11 }[type];
  }

  private createTRA(): string {
    const now = new Date();
    const expiration = new Date(now.getTime() + 12 * 60 * 60 * 1000);

    return `<?xml version="1.0" encoding="UTF-8"?>
      <loginTicketRequest version="1.0">
        <header>
          <uniqueId>${now.getTime()}</uniqueId>
          <generationTime>${now.toISOString()}</generationTime>
          <expirationTime>${expiration.toISOString()}</expirationTime>
        </header>
        <service>wsfe</service>
      </loginTicketRequest>`;
  }

  private signTRA(tra: string): string {
    const sig = new SignedXml();
    sig.addReference("//*[local-name(.)='loginTicketRequest']");
    sig.signingKey = this.privateKey;
    sig.computeSignature(tra);
    return sig.getSignedXml();
  }
}
```

### 4. AI-Powered Clinical Notes

#### Claude Integration for Note Generation

```typescript
// apps/backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private client: Anthropic;
  private opusModel: string;
  private sonnetModel: string;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.opusModel = process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4.6';
    this.sonnetModel = process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4.6';
  }

  async summarizeSession(sessionNotes: string, patientHistory: string[]) {
    const message = await this.client.messages.create({
      model: this.opusModel,
      max_tokens: 4096,
      temperature: 0.3,
      system: `Eres un asistente clínico especializado en psicología. Tu tarea es generar resúmenes concisos y profesionales de sesiones terapéuticas, respetando la terminología clínica argentina y las normativas éticas del Colegio de Psicólogos.`,
      messages: [
        {
          role: 'user',
          content: `Genera un resumen estructurado de esta sesión terapéutica:

NOTAS DE LA SESIÓN:
${sessionNotes}

HISTORIAL RELEVANTE DEL PACIENTE:
${patientHistory.join('\n\n')}

Incluye:
1. Motivo de consulta / Tema principal
2. Observaciones clínicas relevantes
3. Intervenciones realizadas
4. Plan de acción para próxima sesión
5. Alertas o seguimientos necesarios`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async generateProgressReport(patientId: string, sessionsCount: number) {
    const sessions = await this.prisma.session.findMany({
      where: { patientId },
      orderBy: { date: 'desc' },
      take: sessionsCount,
      select: { summary: true, date: true, observations: true },
    });

    const message = await this.client.messages.create({
      model: this.opusModel,
      max_tokens: 8192,
      temperature: 0.2,
      system: `Eres un psicólogo clínico argentino generando un informe de evolución terapéutica. Usa terminología profesional adecuada para compartir con otros profesionales de la salud o instituciones.`,
      messages: [
        {
          role: 'user',
          content: `Genera un informe de evolución terapéutica basado en las siguientes ${sessionsCount} sesiones:

${sessions.map((s, i) => `
SESIÓN ${i + 1} (${s.date.toLocaleDateString('es-AR')}):
${s.summary}
Observaciones: ${s.observations || 'N/A'}
`).join('\n\n')}

El informe debe incluir:
- Resumen del proceso terapéutico
- Evolución observada
- Objetivos alcanzados
- Recomendaciones futuras`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async chatAssistant(query: string, context: Record<string, any>) {
    // Use Sonnet for fast interactive queries
    const message = await this.client.messages.create({
      model: this.sonnetModel,
      max_tokens: 2048,
      temperature: 0.7,
      system: `Eres un asistente clínico rápido que ayuda a psicólogos a encontrar información sobre pacientes y recordar detalles de sesiones previas. Responde de forma concisa y útil.`,
      messages: [
        {
          role: 'user',
          content: `Contexto del sistema: ${JSON.stringify(context, null, 2)}

Pregunta del profesional: ${query}`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async detectRiskFactors(sessionTranscript: string) {
    const message = await this.client.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      temperature: 0.1,
      system: `Analiza transcripciones de sesiones y detecta factores de riesgo como ideación suicida, autolesión, violencia doméstica o crisis aguda. Responde en formato JSON estructurado.`,
      messages: [
        {
          role: 'user',
          content: `Analiza esta transcripción e identifica factores de riesgo:

${sessionTranscript}

Devuelve JSON con: { "riskLevel": "none" | "low" | "medium" | "high" | "critical", "factors": [], "recommendations": [] }`,
        },
      ],
    });

    const response = message.content[0].type === 'text' ? message.content[0].text : '{}';
    return JSON.parse(response);
  }
}
```

### 5. Video Consultation Setup

#### LiveKit Integration

```typescript
// apps/backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  private apiKey: string;
  private apiSecret: string;
  private url: string;

  constructor() {
    this.apiKey = process.env.LIVEKIT_API_KEY;
    this.apiSecret = process.env.LIVEKIT_API_SECRET;
    this.url = process.env.LIVEKIT_URL;
  }

  async createSessionToken(data: {
    roomName: string;
    participantName: string;
    participantRole: 'practitioner' | 'patient';
  }) {
    const at = new AccessToken(this.apiKey, this.apiSecret, {
      identity: data.participantName,
      ttl: '2h',
    });

    at.addGrant({
      roomJoin: true,
      room: data.roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: data.participantRole === 'practitioner',
      canUpdateOwnMetadata: true,
    });

    return {
      token: at.toJwt(),
      url: this.url,
    };
  }

  async createConsultationRoom(appointmentId: string) {
    const token = await this.createSessionToken({
      roomName: `session-${appointmentId}`,
      participantName: 'practitioner',
      participantRole: 'practitioner',
    });

    return {
      roomName: `session-${appointmentId}`,
      practitionerToken: token,
      joinUrl: `${process.env.FRONTEND_URL}/video/${appointmentId}`,
    };
  }
}
```

#### SvelteKit Video Component

```svelte
<!-- apps/frontend/src/routes/video/[appointmentId]/+page.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';
  import { page } from '$app/stores';

  let videoContainer: HTMLDivElement;
  let room: Room | null = null;
  let connected = $state(false);
  let participants = $state([]);

  async function connectToRoom() {
    const response = await fetch(`/api/video/token/${$page.params.appointmentId}`);
    const { token, url } = await response.json();

    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: { width: 1280, height: 720 },
      },
    });

    room.on(RoomEvent.ParticipantConnected, (participant) => {
      console.log('Participant connected:', participant.identity);
      participants = [...participants, participant];
    });

    room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
      if (track.kind === 'video' || track.kind === 'audio') {
        const element = track.attach();
        videoContainer.appendChild(element);
      }
    });

    await room.connect(url, token);
    connected = true;

    // Publish camera and microphone
    await room.localParticipant.enableCameraAndMicrophone();
  }

  async function toggleMute() {
    if (room) {
      await room.localParticipant.setMicrophoneEnabled(
        !room.localParticipant.isMicrophoneEnabled
      );
    }
  }

  async function toggleVideo() {
    if (room) {
      await room.localParticipant.setCameraEnabled(
        !room.localParticipant.isCameraEnabled
      );
    }
  }

  onMount(() => {
    connectToRoom();
  });

  onDestroy(() => {
    if (room) {
      room.disconnect();
    }
  });
</script>

<div class="video-consultation">
  <div class="video-grid" bind:this={videoContainer}>
    <!-- Video tracks will be attached here -->
  </div>

  <div class="controls">
    <button onclick={toggleMute} class="btn-control">
      {#if room?.localParticipant.isMicrophoneEnabled}
        🎤 Silenciar
      {:else}
        🔇 Activar Mic
      {/if}
    </button>

    <button onclick={toggleVideo} class="btn-control">
      {#if room?.localParticipant.isCameraEnabled}
        📹 Ocultar Cámara
      {:else}
        📹 Mostrar Cámara
      {/if}
    </button>

    <button 
      onclick={() => room?.disconnect()}
      class="btn-control btn-danger"
    >
      🔚 Finalizar Consulta
    </button>
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
    grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
