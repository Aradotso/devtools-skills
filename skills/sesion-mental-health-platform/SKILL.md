---
name: sesion-mental-health-platform
description: Full-stack SaaS platform for psychology clinics in Argentina with AI-powered scheduling, WhatsApp automation, AFIP billing, and video consultations
triggers:
  - "set up Sesión mental health platform"
  - "integrate WhatsApp automation for clinic appointments"
  - "configure AFIP electronic invoicing for Argentina"
  - "implement Claude AI for clinical notes"
  - "add video consultation to psychology practice"
  - "create patient journey pipeline with Sesión"
  - "configure Mercado Pago billing integration"
  - "set up LiveKit for therapy video calls"
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS orchestration platform for psychology clinics in Argentina, providing intelligent appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance (Claude Opus 4.6 and Sonnet 4.6). Built with SvelteKit 5, NestJS, Prisma, and TypeScript.

## Architecture Overview

The platform uses a microservices architecture with:
- **Frontend**: SvelteKit 5 + Tailwind CSS
- **Backend**: NestJS with Prisma ORM
- **Database**: PostgreSQL (relational data), Redis (caching), Elasticsearch (search)
- **Messaging**: Baileys (WhatsApp automation)
- **Video**: LiveKit WebRTC
- **AI**: Anthropic Claude Opus 4.6 and Sonnet 4.6
- **Payments**: Stripe (international), Mercado Pago (Argentina)
- **Storage**: MinIO (encrypted documents)

## Installation

### Prerequisites

```bash
# Required versions
node >= 18.0.0
pnpm >= 8.0.0
docker >= 24.0.0
postgresql >= 15.0
redis >= 7.0
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

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=sk-ant-xxxxx
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET=your-webhook-secret

# LiveKit Video
LIVEKIT_API_KEY=your-api-key
LIVEKIT_API_SECRET=your-api-secret
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# Argentina AFIP
AFIP_CUIT=your-cuit-number
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
AFIP_PRODUCTION=false

# Payment Gateways
MERCADOPAGO_ACCESS_TOKEN=your-mercadopago-token
STRIPE_SECRET_KEY=sk_test_xxxxx

# App
JWT_SECRET=your-jwt-secret
APP_URL="http://localhost:5173"
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
# Start all services with Docker Compose
docker-compose up -d

# Start backend (NestJS)
cd backend
pnpm dev

# Start frontend (SvelteKit)
cd frontend
pnpm dev
```

## Core Module Usage

### 1. Patient Management (Prisma)

```typescript
// backend/src/patients/patients.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class PatientsService {
  constructor(private prisma: PrismaService) {}

  async createPatient(data: {
    firstName: string;
    lastName: string;
    email: string;
    phone: string;
    dni: string;
  }) {
    return this.prisma.patient.create({
      data: {
        firstName: data.firstName,
        lastName: data.lastName,
        email: data.email,
        phone: data.phone,
        dni: data.dni,
        status: 'ACTIVE',
        journeyStage: 'PRIMER_CONTACTO',
      },
    });
  }

  async getPatientWithHistory(patientId: string) {
    return this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        appointments: {
          orderBy: { scheduledAt: 'desc' },
          take: 10,
        },
        invoices: {
          orderBy: { issuedAt: 'desc' },
        },
        clinicalNotes: {
          orderBy: { createdAt: 'desc' },
        },
      },
    });
  }
}
```

### 2. Appointment Scheduling

```typescript
// backend/src/appointments/appointments.service.ts
import { Injectable, BadRequestException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async scheduleAppointment(data: {
    patientId: string;
    practitionerId: string;
    scheduledAt: Date;
    duration: number;
    type: 'PRESENCIAL' | 'VIRTUAL' | 'EVALUACION';
  }) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        scheduledAt: {
          gte: data.scheduledAt,
          lt: new Date(data.scheduledAt.getTime() + data.duration * 60000),
        },
        status: { notIn: ['CANCELLED'] },
      },
    });

    if (conflicts.length > 0) {
      throw new BadRequestException('Time slot already booked');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: data.patientId,
        practitionerId: data.practitionerId,
        scheduledAt: data.scheduledAt,
        duration: data.duration,
        type: data.type,
        status: 'SCHEDULED',
      },
      include: { patient: true, practitioner: true },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(appointment);

    return appointment;
  }

  async sendReminders() {
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        scheduledAt: {
          gte: new Date(tomorrow.setHours(0, 0, 0, 0)),
          lt: new Date(tomorrow.setHours(23, 59, 59, 999)),
        },
        status: 'SCHEDULED',
      },
      include: { patient: true, practitioner: true },
    });

    for (const appointment of appointments) {
      await this.whatsapp.sendReminder(appointment);
    }
  }
}
```

### 3. WhatsApp Automation (Baileys)

```typescript
// backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  useMultiFileAuthState,
  DisconnectReason,
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
  }

  async sendAppointmentConfirmation(appointment: any) {
    const message = `Hola ${appointment.patient.firstName}! 👋

Tu sesión con ${appointment.practitioner.firstName} ${appointment.practitioner.lastName} está confirmada:

📅 Fecha: ${appointment.scheduledAt.toLocaleDateString('es-AR')}
⏰ Hora: ${appointment.scheduledAt.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}
${appointment.type === 'VIRTUAL' ? '💻 Modalidad: Virtual' : '🏥 Modalidad: Presencial'}

Responde SÍ para confirmar o NO para reagendar.`;

    await this.sock.sendMessage(
      `${appointment.patient.phone}@s.whatsapp.net`,
      { text: message },
    );
  }

  async sendReminder(appointment: any) {
    const message = `Recordatorio: Tu sesión es mañana a las ${appointment.scheduledAt.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}. ⏰`;

    await this.sock.sendMessage(
      `${appointment.patient.phone}@s.whatsapp.net`,
      { text: message },
    );
  }

  async handleIncomingMessage(from: string, text: string) {
    // Crisis detection keywords
    const crisisKeywords = ['suicidio', 'quiero morir', 'no puedo más', 'ayuda urgente'];
    if (crisisKeywords.some((kw) => text.toLowerCase().includes(kw))) {
      // Alert practitioner immediately
      await this.alertPractitionerCrisis(from, text);
    }
  }
}
```

### 4. AFIP Electronic Invoicing

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as Afip from '@afipsdk/afip.js';

@Injectable()
export class AfipService {
  private afip: any;

  constructor() {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.AFIP_PRODUCTION === 'true',
    });
  }

  async createInvoice(data: {
    patientName: string;
    patientDni: string;
    amount: number;
    sessionDate: Date;
    concept: string;
  }) {
    const invoiceData = {
      CantReg: 1,
      PtoVta: 1, // Punto de venta
      CbteTipo: 6, // Factura B
      Concepto: 2, // Servicios
      DocTipo: 96, // DNI
      DocNro: data.patientDni,
      CbteDesde: 1,
      CbteHasta: 1,
      CbteFch: parseInt(data.sessionDate.toISOString().slice(0, 10).replace(/-/g, '')),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    };

    const response = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    return {
      cae: response.CAE,
      caeExpiration: response.CAEFchVto,
      invoiceNumber: response.CbteDesde,
      afipResponse: response,
    };
  }

  async getLastInvoiceNumber() {
    return this.afip.ElectronicBilling.getLastVoucher(1, 6); // Punto vta 1, Tipo 6 (Factura B)
  }
}
```

### 5. AI Clinical Notes (Claude)

```typescript
// backend/src/ai/claude.service.ts
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

  async summarizeClinicalNote(rawNotes: string, patientHistory: string) {
    const response = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 2048,
      system: `Eres un asistente especializado en psicología clínica argentina. 
Tu tarea es resumir notas de sesión de forma profesional, confidencial y estructurada.
Nunca incluyas información identificable del paciente en el resumen.`,
      messages: [
        {
          role: 'user',
          content: `Historial del paciente (últimas 5 sesiones):
${patientHistory}

Notas de la sesión actual:
${rawNotes}

Por favor, genera un resumen estructurado con:
1. Presentación del paciente
2. Temas principales tratados
3. Intervenciones realizadas
4. Evolución observada
5. Plan terapéutico siguiente`,
        },
      ],
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  }

  async generateSessionReport(appointmentId: string) {
    // Use Sonnet for faster interactive responses
    const response = await this.anthropic.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Genera un informe de sesión para obra social/prepaga basado en el appointment ID: ${appointmentId}`,
        },
      ],
    });

    return response.content[0].type === 'text'
      ? response.content[0].text
      : '';
  }

  async detectRiskFactors(patientMessages: string[]) {
    const response = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 512,
      system: `Analiza mensajes de pacientes para detectar factores de riesgo psicológico.
Categorías: suicidio, autolesión, violencia, abuso de sustancias, crisis aguda.`,
      messages: [
        {
          role: 'user',
          content: `Mensajes del paciente: ${patientMessages.join('\n\n')}`,
        },
      ],
    });

    return response.content[0].type === 'text'
      ? response.content[0].text
      : '';
  }
}
```

### 6. Video Consultations (LiveKit)

```typescript
// backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  async createRoomToken(roomName: string, participantName: string) {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: participantName,
        name: participantName,
      },
    );

    at.addGrant({ 
      roomJoin: true, 
      room: roomName,
      canPublish: true,
      canSubscribe: true,
    });

    return at.toJwt();
  }

  async createTherapySession(appointmentId: string, practitionerId: string, patientId: string) {
    const roomName = `session-${appointmentId}`;
    
    const practitionerToken = await this.createRoomToken(
      roomName,
      `practitioner-${practitionerId}`,
    );
    
    const patientToken = await this.createRoomToken(
      roomName,
      `patient-${patientId}`,
    );

    return {
      roomName,
      practitionerToken,
      patientToken,
      wsUrl: process.env.LIVEKIT_WS_URL,
    };
  }
}
```

## Frontend Components (SvelteKit 5)

### Appointment Calendar Component

```svelte
<!-- frontend/src/lib/components/AppointmentCalendar.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  
  let appointments = writable<any[]>([]);
  let selectedDate = writable(new Date());

  async function loadAppointments() {
    const response = await fetch('/api/appointments', {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
    });
    const data = await response.json();
    appointments.set(data);
  }

  async function createAppointment(patientId: string, scheduledAt: Date) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({
        patientId,
        scheduledAt,
        duration: 45,
        type: 'PRESENCIAL'
      })
    });
    
    if (response.ok) {
      await loadAppointments();
    }
  }

  onMount(() => {
    loadAppointments();
  });
</script>

<div class="calendar-container">
  <div class="calendar-header">
    <h2>Agenda del Día</h2>
    <button on:click={() => loadAppointments()}>Actualizar</button>
  </div>
  
  <div class="appointments-list">
    {#each $appointments as appointment}
      <div class="appointment-card">
        <div class="time">{new Date(appointment.scheduledAt).toLocaleTimeString('es-AR')}</div>
        <div class="patient">{appointment.patient.firstName} {appointment.patient.lastName}</div>
        <div class="type">{appointment.type}</div>
        {#if appointment.type === 'VIRTUAL'}
          <button on:click={() => window.location.href = `/video/${appointment.id}`}>
            Iniciar Videollamada
          </button>
        {/if}
      </div>
    {/each}
  </div>
</div>

<style>
  .calendar-container {
    padding: 2rem;
    background: white;
    border-radius: 8px;
  }
  
  .appointment-card {
    display: flex;
    gap: 1rem;
    padding: 1rem;
    border-left: 4px solid #3b82f6;
    margin-bottom: 0.5rem;
    background: #f9fafb;
  }
</style>
```

### Video Room Component

```svelte
<!-- frontend/src/routes/video/[appointmentId]/+page.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';
  import { page } from '$app/stores';

  let videoContainer: HTMLDivElement;
  let room: Room;
  let audioEnabled = $state(true);
  let videoEnabled = $state(true);

  async function joinRoom() {
    const response = await fetch(`/api/video/${$page.params.appointmentId}/token`, {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
    });
    
    const { wsUrl, token } = await response.json();
    
    room = new Room();
    
    room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
      if (track.kind === 'video') {
        const element = track.attach();
        videoContainer.appendChild(element);
      }
    });

    await room.connect(wsUrl, token);
    
    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
  }

  function toggleAudio() {
    audioEnabled = !audioEnabled;
    room.localParticipant.setMicrophoneEnabled(audioEnabled);
  }

  function toggleVideo() {
    videoEnabled = !videoEnabled;
    room.localParticipant.setCameraEnabled(videoEnabled);
  }

  onMount(() => {
    joinRoom();
  });

  onDestroy(() => {
    room?.disconnect();
  });
</script>

<div class="video-room">
  <div bind:this={videoContainer} class="video-container"></div>
  
  <div class="controls">
    <button on:click={toggleAudio}>
      {audioEnabled ? 'Silenciar' : 'Activar Audio'}
    </button>
    <button on:click={toggleVideo}>
      {videoEnabled ? 'Desactivar Video' : 'Activar Video'}
    </button>
    <button on:click={() => room?.disconnect()}>Terminar Sesión</button>
  </div>
</div>

<style>
  .video-room {
    height: 100vh;
    background: #1f2937;
  }
  
  .video-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
    gap: 1rem;
    padding: 1rem;
  }
  
  .controls {
    position: fixed;
    bottom: 2rem;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 1rem;
  }
</style>
```

## Common Patterns

### Patient Journey Tracking

```typescript
// Update patient journey stage automatically
async function updateJourneyStage(patientId: string) {
  const patient = await prisma.patient.findUnique({
    where: { id: patientId },
    include: {
      appointments: {
        where: { status: 'COMPLETED' },
        orderBy: { scheduledAt: 'desc' },
      },
    },
  });

  let newStage = patient.journeyStage;

  if (patient.appointments.length === 0) {
    newStage = 'PRIMER_CONTACTO';
  } else if (patient.appointments.length >= 1 && patient.appointments.length < 4) {
    newStage = 'ADMISION';
  } else if (patient.appointments.length >= 4) {
    newStage = 'SESION_ACTIVA';
  }

  const lastAppointment = patient.appointments[0];
  const daysSinceLastSession = lastAppointment
    ? Math.floor((Date.now() - lastAppointment.scheduledAt.getTime()) / (1000 * 60 * 60 * 24))
    : 999;

  if (daysSinceLastSession > 60) {
    newStage = 'SEGUIMIENTO';
  }

  if (newStage !== patient.journeyStage) {
    await prisma.patient.update({
      where: { id: patientId },
      data: { journeyStage: newStage },
    });
  }
}
```

### Multi-Model AI Routing

```typescript
// Route complex queries to Opus, simple ones to Sonnet
async function routeAIQuery(query: string, complexity: 'simple' | 'complex') {
  const model = complexity === 'complex'
    ? process.env.CLAUDE_OPUS_MODEL
    : process.env.CLAUDE_SONNET_MODEL;

  const response = await anthropic.messages.create({
    model,
    max_tokens: complexity === 'complex' ? 4096 : 1024,
    messages: [{ role: 'user', content: query }],
  });

  return response.content[0].type === 'text' ? response.content[0].text : '';
}
```

## CLI Commands

```bash
# Database management
pnpm prisma studio                 # Open Prisma Studio GUI
pnpm prisma migrate dev            # Create and apply migration
pnpm prisma db push                # Push schema without migration

# Development
pnpm dev                           # Start dev servers
pnpm build                         # Build production bundles
pnpm start                         # Start production server

# Testing
pnpm test                          # Run unit tests
pnpm test:e2e                      # Run E2E tests
pnpm test:cov                      # Generate coverage report

# WhatsApp
pnpm whatsapp:connect              # Generate QR code for WhatsApp
pnpm whatsapp:logout               # Disconnect WhatsApp session

# AFIP
pnpm afip:test-connection          # Test AFIP credentials
pnpm afip:generate-cert            # Generate AFIP certificate
```

## Troubleshooting

### WhatsApp Connection Issues

```typescript
// Clear session and reconnect
import { rm } from 'fs/promises';

async function resetWhatsAppSession() {
  await rm(process.env.WHATSAPP_SESSION_PATH, { recursive: true, force: true });
  // Restart the WhatsApp service to generate new QR code
}
```

### AFIP Certificate Errors

```bash
# Verify certificate format
openssl x509 -in ./certs/afip-cert.pem -text -noout

# Check certificate expiration
openssl x509 -in ./certs/afip-cert.pem -enddate -noout

# Regenerate certificate from AFIP admin portal if expired
```

### LiveKit Video Quality Issues

```typescript
// Adjust video quality dynamically based on bandwidth
const videoOptions = {
  resolution: {
    width: 1280,
    height: 720,
    frameRate: 30,
  },
  // Reduce for low bandwidth
  adaptiveStream: true,
  dynacast: true,
};

await room.localParticipant.setCameraEnabled(true, videoOptions);
```

### Claude API Rate Limits

```typescript
// Implement exponential backoff
async function callClaudeWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: process.env.CLAUDE_SONNET_MODEL,
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }],
      });
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
      } else {
        throw error;
      }
    }
  }
}
```

### Database Migration Conflicts

```bash
# Reset database (DANGER: deletes all data)
pnpm prisma migrate reset

# Create migration from schema changes
pnpm prisma migrate dev --name add-new-field

# Resolve migration conflicts manually
pnpm prisma migrate resolve --applied "migration_name"
```

### Mercado Pago Webhook Verification

```typescript
import crypto from 'crypto';

function verifyMercadoPagoWebhook(payload: string, signature: string) {
  const secret = process.env.MERCADOPAGO_WEBHOOK_SECRET;
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(payload);
  const digest = hmac.digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(digest)
  );
}
```

## Production Deployment

```bash
# Build optimized bundles
pnpm build

# Run database migrations
pnpm prisma migrate deploy

# Start with PM2
pm2 start ecosystem.config.js

# Monitor logs
pm2 logs sesion-backend
pm2 logs sesion-frontend
```

### Docker Deployment

```bash
# Build images
docker-compose -f docker-compose.prod.yml build

# Start services
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose logs -f backend
```
