---
name: sesion-mental-health-platform
description: SaaS platform for psychology clinics with AI-powered scheduling, WhatsApp automation, AFIP billing, and video consultations for Argentine practitioners
triggers:
  - how do I set up Sesión for a psychology clinic
  - integrate WhatsApp automation with Sesión
  - configure AFIP electronic invoicing in Sesión
  - implement Claude AI clinical notes in Sesión
  - set up video consultations with LiveKit in Sesión
  - deploy Sesión mental health platform
  - configure Sesión appointment scheduling
  - integrate MercadoPago billing in Sesión
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina, combining intelligent appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical workflows using Claude Opus 4.6 and Sonnet 4.6. Built with SvelteKit 5 frontend, NestJS backend, Prisma ORM, TypeScript, and TailwindCSS.

## Architecture Stack

- **Frontend**: SvelteKit 5, Svelte 5, TailwindCSS
- **Backend**: NestJS (TypeScript)
- **Database**: PostgreSQL with Prisma ORM
- **AI**: Anthropic Claude (Opus 4.6, Sonnet 4.6)
- **WhatsApp**: Baileys library (WhatsApp Web multi-device)
- **Video**: LiveKit WebRTC
- **Payments**: MercadoPago (Argentina), Stripe (international)
- **Caching**: Redis
- **Search**: Elasticsearch (optional)

## Installation & Setup

### Prerequisites

```bash
# Node.js 18+ required
node --version  # v18.0.0 or higher

# PostgreSQL 14+
psql --version

# Redis 6+
redis-cli --version
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
npm install

# Or if using pnpm
pnpm install
```

### Environment Configuration

Create `.env` file in project root:

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=sk-ant-your-key-here

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"

# LiveKit Video
LIVEKIT_API_KEY=your-livekit-api-key
LIVEKIT_API_SECRET=your-livekit-api-secret
LIVEKIT_URL=wss://your-livekit-server.com

# MercadoPago (Argentina)
MERCADOPAGO_ACCESS_TOKEN=your-mercadopago-token
MERCADOPAGO_PUBLIC_KEY=your-mercadopago-public-key

# AFIP (Argentina Tax Authority)
AFIP_CUIT=your-clinic-cuit
AFIP_CERTIFICATE_PATH="./afip-cert.crt"
AFIP_PRIVATE_KEY_PATH="./afip-key.key"

# Stripe (International)
STRIPE_SECRET_KEY=sk_test_your-stripe-key
STRIPE_WEBHOOK_SECRET=whsec_your-webhook-secret

# App Config
APP_URL=http://localhost:5173
JWT_SECRET=your-jwt-secret-key
```

### Database Setup

```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate deploy

# Seed initial data (optional)
npx prisma db seed
```

## Core Module Usage

### 1. Appointment Scheduling

**Prisma Schema (appointments)**:

```prisma
model Appointment {
  id            String   @id @default(cuid())
  patientId     String
  practitionerId String
  startTime     DateTime
  endTime       DateTime
  type          AppointmentType // PRESENCIAL, VIRTUAL, EVALUACION
  status        AppointmentStatus // SCHEDULED, CONFIRMED, CANCELLED, COMPLETED
  roomId        String?
  
  patient       Patient @relation(fields: [patientId], references: [id])
  practitioner  Practitioner @relation(fields: [practitionerId], references: [id])
  
  @@index([practitionerId, startTime])
  @@index([patientId])
}
```

**NestJS Service (appointments.service.ts)**:

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { Appointment, AppointmentType } from '@prisma/client';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    type: AppointmentType;
  }): Promise<Appointment> {
    // Calculate end time (default 45 minutes)
    const endTime = new Date(data.startTime.getTime() + 45 * 60000);

    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        OR: [
          {
            AND: [
              { startTime: { lte: data.startTime } },
              { endTime: { gt: data.startTime } }
            ]
          },
          {
            AND: [
              { startTime: { lt: endTime } },
              { endTime: { gte: endTime } }
            ]
          }
        ]
      }
    });

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    return this.prisma.appointment.create({
      data: {
        ...data,
        endTime,
        status: 'SCHEDULED'
      },
      include: {
        patient: true,
        practitioner: true
      }
    });
  }

  async findUpcoming(practitionerId: string): Promise<Appointment[]> {
    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: new Date() },
        status: { notIn: ['CANCELLED'] }
      },
      orderBy: { startTime: 'asc' },
      include: {
        patient: true
      }
    });
  }
}
```

**SvelteKit Component (AppointmentCalendar.svelte)**:

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  
  let appointments = $state<Appointment[]>([]);
  let loading = $state(true);

  async function fetchAppointments() {
    const response = await fetch('/api/appointments/upcoming');
    appointments = await response.json();
    loading = false;
  }

  async function createAppointment(data: AppointmentData) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (response.ok) {
      await fetchAppointments();
    }
  }

  onMount(fetchAppointments);
</script>

<div class="calendar-container">
  {#if loading}
    <div class="loading">Cargando agenda...</div>
  {:else}
    <div class="appointments-grid">
      {#each appointments as appointment}
        <div class="appointment-card">
          <h3>{appointment.patient.name}</h3>
          <p>{new Date(appointment.startTime).toLocaleString('es-AR')}</p>
          <span class="badge badge-{appointment.type.toLowerCase()}">
            {appointment.type}
          </span>
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  .calendar-container {
    @apply p-4 bg-white rounded-lg shadow;
  }
  
  .appointments-grid {
    @apply grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4;
  }
  
  .appointment-card {
    @apply p-4 border rounded hover:shadow-md transition;
  }
</style>
```

### 2. WhatsApp Automation with Baileys

**WhatsApp Service (whatsapp.service.ts)**:

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, { DisconnectReason, useMultiFileAuthState } from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions'
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true
    });

    this.sock.ev.on('creds.update', saveCreds);

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      
      if (connection === 'close') {
        const shouldReconnect = 
          (lastDisconnect?.error as Boom)?.output?.statusCode !== 
          DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.connectToWhatsApp();
        }
      } else if (connection === 'open') {
        console.log('WhatsApp connected successfully');
      }
    });
  }

  async sendAppointmentReminder(
    phoneNumber: string,
    appointmentData: {
      patientName: string;
      dateTime: Date;
      practitionerName: string;
    }
  ) {
    const formattedPhone = phoneNumber.replace(/\D/g, '') + '@s.whatsapp.net';
    
    const message = `Hola ${appointmentData.patientName}! 👋

Este es un recordatorio de tu sesión:
📅 Fecha: ${appointmentData.dateTime.toLocaleDateString('es-AR')}
🕐 Hora: ${appointmentData.dateTime.toLocaleTimeString('es-AR', { 
  hour: '2-digit', 
  minute: '2-digit' 
})}
👨‍⚕️ Profesional: ${appointmentData.practitionerName}

¿Podés confirmar tu asistencia? Respondé SÍ para confirmar.`;

    await this.sock.sendMessage(formattedPhone, { text: message });
  }

  async sendInvoice(phoneNumber: string, invoicePdfUrl: string) {
    const formattedPhone = phoneNumber.replace(/\D/g, '') + '@s.whatsapp.net';
    
    await this.sock.sendMessage(formattedPhone, {
      document: { url: invoicePdfUrl },
      mimetype: 'application/pdf',
      fileName: 'factura.pdf',
      caption: '📄 Tu comprobante de pago está listo. Gracias por confiar en nosotros.'
    });
  }
}
```

**Scheduler for Automated Reminders (appointments.scheduler.ts)**:

```typescript
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentScheduler {
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

    for (const appointment of appointments) {
      await this.whatsapp.sendAppointmentReminder(
        appointment.patient.phone,
        {
          patientName: appointment.patient.name,
          dateTime: appointment.startTime,
          practitionerName: appointment.practitioner.name
        }
      );

      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent24h: true }
      });
    }
  }
}
```

### 3. Claude AI Integration

**AI Service (claude.service.ts)**:

```typescript
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }

  async summarizeClinicalNotes(
    sessionNotes: string[],
    patientContext?: string
  ): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: 'claude-opus-4-20250514',
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `Sos un asistente clínico especializado en psicología.
          
${patientContext ? `Contexto del paciente: ${patientContext}` : ''}

Resume las siguientes notas clínicas de manera estructurada, identificando:
1. Temas principales abordados
2. Evolución del paciente
3. Intervenciones terapéuticas aplicadas
4. Puntos de seguimiento

Notas de sesiones:
${sessionNotes.map((note, i) => `Sesión ${i + 1}: ${note}`).join('\n\n')}

Formato la respuesta en español argentino, siguiendo las mejores prácticas clínicas.`
        }
      ]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateSessionPlan(
    patientHistory: string,
    therapeuticGoals: string[]
  ): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: 'claude-sonnet-4-20250514', // Faster for interactive use
      max_tokens: 1500,
      messages: [
        {
          role: 'user',
          content: `Como psicólogo clínico, ayudame a estructurar la próxima sesión.

Historia del paciente:
${patientHistory}

Objetivos terapéuticos:
${therapeuticGoals.map((goal, i) => `${i + 1}. ${goal}`).join('\n')}

Sugerí:
- Temas para abordar en la próxima sesión
- Técnicas terapéuticas recomendadas
- Preguntas de apertura
- Recursos para asignar como tarea

Formato español argentino, profesional pero accesible.`
        }
      ]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async assistClinicalDecision(
    clinicalQuestion: string,
    patientData: any
  ): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: 'claude-opus-4-20250514',
      max_tokens: 2048,
      system: `Sos un consultor clínico especializado en salud mental. Proporcionás análisis basados en evidencia, siempre recordando que la decisión final es del profesional tratante. Cumplís con las normas éticas del Colegio de Psicólogos de Argentina.`,
      messages: [
        {
          role: 'user',
          content: `Pregunta clínica: ${clinicalQuestion}

Datos del paciente (anonimizados):
${JSON.stringify(patientData, null, 2)}

Proporcioná un análisis profesional considerando:
1. Marco teórico aplicable
2. Evidencia científica disponible
3. Consideraciones éticas
4. Recomendaciones prácticas

Respondé en español argentino.`
        }
      ]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
}
```

**SvelteKit AI Assistant Component (AIAssistant.svelte)**:

```svelte
<script lang="ts">
  let query = $state('');
  let response = $state('');
  let loading = $state(false);

  async function askClaude() {
    loading = true;
    try {
      const res = await fetch('/api/ai/assist', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ 
          question: query,
          context: 'clinical_decision'
        })
      });
      
      const data = await res.json();
      response = data.response;
    } finally {
      loading = false;
    }
  }
</script>

<div class="ai-assistant">
  <textarea
    bind:value={query}
    placeholder="Hacé una consulta clínica..."
    class="w-full p-4 border rounded-lg"
    rows="4"
  ></textarea>
  
  <button 
    onclick={askClaude}
    disabled={loading || !query.trim()}
    class="btn btn-primary mt-2"
  >
    {loading ? 'Consultando...' : 'Consultar a Claude'}
  </button>

  {#if response}
    <div class="response-box mt-4 p-4 bg-blue-50 rounded-lg">
      <h3 class="font-semibold mb-2">Respuesta:</h3>
      <div class="prose">{@html response}</div>
    </div>
  {/if}
</div>
```

### 4. AFIP Electronic Invoicing

**AFIP Service (afip.service.ts)**:

```typescript
import { Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as https from 'https';

interface InvoiceData {
  amount: number;
  patientName: string;
  patientDNI: string;
  serviceDescription: string;
  sessionDate: Date;
}

@Injectable()
export class AFIPService {
  private cuit: string;
  private certificate: string;
  private privateKey: string;

  constructor() {
    this.cuit = process.env.AFIP_CUIT!;
    this.certificate = fs.readFileSync(
      process.env.AFIP_CERTIFICATE_PATH!,
      'utf8'
    );
    this.privateKey = fs.readFileSync(
      process.env.AFIP_PRIVATE_KEY_PATH!,
      'utf8'
    );
  }

  async generateFactura(
    invoiceData: InvoiceData,
    type: 'A' | 'B' | 'C' = 'B'
  ): Promise<{ cae: string; pdfUrl: string }> {
    // Get authorization token
    const token = await this.getAuthToken();

    // Request CAE (Código de Autorización Electrónico)
    const caeResponse = await this.requestCAE({
      ...invoiceData,
      type,
      token
    });

    // Generate PDF
    const pdfUrl = await this.generateInvoicePDF({
      ...invoiceData,
      cae: caeResponse.cae,
      type
    });

    return {
      cae: caeResponse.cae,
      pdfUrl
    };
  }

  private async getAuthToken(): Promise<string> {
    // AFIP WSAA authentication flow
    // Implementation requires X.509 certificate authentication
    // Simplified for demonstration
    return 'afip-token-placeholder';
  }

  private async requestCAE(data: any): Promise<{ cae: string }> {
    // Call AFIP Web Service for electronic invoice authorization
    // Actual implementation uses SOAP/XML
    return { cae: '1234567890' };
  }

  private async generateInvoicePDF(data: any): Promise<string> {
    // Generate PDF with QR code, CAE, and invoice details
    // Return URL to stored PDF
    return '/invoices/factura-example.pdf';
  }
}
```

### 5. LiveKit Video Consultations

**Video Service (video.service.ts)**:

```typescript
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  private apiKey: string;
  private apiSecret: string;
  private serverUrl: string;

  constructor() {
    this.apiKey = process.env.LIVEKIT_API_KEY!;
    this.apiSecret = process.env.LIVEKIT_API_SECRET!;
    this.serverUrl = process.env.LIVEKIT_URL!;
  }

  async createSessionToken(
    appointmentId: string,
    participantName: string,
    isPractitioner: boolean
  ): Promise<string> {
    const token = new AccessToken(this.apiKey, this.apiSecret, {
      identity: participantName,
      ttl: 3600 // 1 hour
    });

    token.addGrant({
      roomJoin: true,
      room: `session-${appointmentId}`,
      canPublish: true,
      canSubscribe: true,
      canPublishData: isPractitioner, // Only practitioner can share screen/files
      roomRecord: isPractitioner // Only practitioner can record
    });

    return token.toJwt();
  }

  getConnectionDetails(appointmentId: string) {
    return {
      serverUrl: this.serverUrl,
      roomName: `session-${appointmentId}`
    };
  }
}
```

**SvelteKit Video Component (VideoSession.svelte)**:

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';

  export let appointmentId: string;
  export let participantName: string;
  export let isPractitioner: boolean = false;

  let videoContainer: HTMLDivElement;
  let room: Room;
  let connected = $state(false);

  async function joinSession() {
    const response = await fetch('/api/video/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ appointmentId, participantName, isPractitioner })
    });

    const { token, serverUrl, roomName } = await response.json();

    room = new Room();

    room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
      const element = track.attach();
      videoContainer.appendChild(element);
    });

    await room.connect(serverUrl, token);
    connected = true;

    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
  }

  async function leaveSession() {
    await room?.disconnect();
    connected = false;
  }

  onMount(() => {
    return () => {
      leaveSession();
    };
  });
</script>

<div class="video-session">
  {#if !connected}
    <button onclick={joinSession} class="btn btn-primary">
      Ingresar a la sesión
    </button>
  {:else}
    <div bind:this={videoContainer} class="video-grid"></div>
    <button onclick={leaveSession} class="btn btn-danger">
      Finalizar sesión
    </button>
  {/if}
</div>

<style>
  .video-grid {
    @apply grid grid-cols-1 md:grid-cols-2 gap-4 p-4;
  }
  
  .video-grid :global(video) {
    @apply w-full rounded-lg shadow-lg;
  }
</style>
```

## Configuration Patterns

### Multi-Tenant Setup

```typescript
// prisma/schema.prisma
model Clinic {
  id            String   @id @default(cuid())
  name          String
  slug          String   @unique
  cuit          String   // Argentine tax ID
  
  practitioners Practitioner[]
  patients      Patient[]
  settings      ClinicSettings?
}

model ClinicSettings {
  id                String  @id @default(cuid())
  clinicId          String  @unique
  
  sessionDuration   Int     @default(45) // minutes
  timezone          String  @default("America/Argentina/Buenos_Aires")
  reminderHours     Int[]   @default([24, 2]) // Hours before appointment
  
  afipEnabled       Boolean @default(true)
  whatsappEnabled   Boolean @default(true)
  
  clinic            Clinic  @relation(fields: [clinicId], references: [id])
}
```

### Payment Gateway Configuration

```typescript
// payments/mercadopago.service.ts
import { Injectable } from '@nestjs/common';
import mercadopago from 'mercadopago';

@Injectable()
export class MercadoPagoService {
  constructor() {
    mercadopago.configure({
      access_token: process.env.MERCADOPAGO_ACCESS_TOKEN!
    });
  }

  async createPaymentLink(amount: number, description: string) {
    const preference = await mercadopago.preferences.create({
      items: [
        {
          title: description,
          unit_price: amount,
          quantity: 1,
          currency_id: 'ARS' // Argentine Peso
        }
      ],
      back_urls: {
        success: `${process.env.APP_URL}/payments/success`,
        failure: `${process.env.APP_URL}/payments/failure`,
        pending: `${process.env.APP_URL}/payments/pending`
      },
      auto_return: 'approved'
    });

    return preference.body.init_point;
  }
}
```

## Common Workflows

### Complete Patient Journey

```typescript
// workflows/patient-journey.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';
import { AppointmentsService } from '../appointments/appointments.service';

@Injectable()
export class PatientJourneyService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
    private appointments: AppointmentsService
  ) {}

  async onboardNewPatient(data: {
    name: string;
    phone: string;
    email: string;
    clinicId: string;
  }) {
    // 1. Create patient record
    const patient = await this.prisma.patient.create({
      data: {
        ...data,
        stage: 'PRIMER_CONTACTO'
      }
    });

    // 2. Send welcome WhatsApp
    await this.whatsapp.sendMessage(patient.phone, 
      `Hola ${patient.name}! Bienvenid@ a nuestra clínica. ` +
      `Estamos listos para ayudarte. ¿Te gustaría agendar tu primera consulta?`
    );

    // 3. Update stage
    await this.prisma.patient.update({
      where: { id: patient.id },
      data: { stage: 'ADMISION' }
    });

    return patient;
  }

  async scheduleFirstAppointment(
    patientId: string,
    practitionerId: string,
    startTime: Date
  ) {
    const appointment = await this.appointments.createAppointment({
      patientId,
      practitionerId,
      startTime,
      type: 'EVALUACION'
    });

    await this.prisma.patient.update({
      where: { id: patientId },
      data: { stage: 'SESION_ACTIVA' }
    });

    return appointment;
  }
}
```

## Troubleshooting

### WhatsApp Connection Issues

```typescript
// Check authentication state
const sessionPath = process.env.WHATSAPP_SESSION_PATH;
const authFiles = fs.readdirSync(sessionPath);

if (authFiles.length === 0) {
  console.log('No auth session found. Scan QR code to connect.');
}

// Force re-authentication
async function resetWhatsAppSession() {
  const sessionPath = process.env.WHATSAPP_SESSION_PATH;
  fs.rmSync(sessionPath, { recursive: true, force: true });
  fs.mkdirSync(sessionPath, { recursive: true });
  // Restart service to re-scan QR
}
```

### AFIP Certificate Errors

```bash
# Verify certificate validity
openssl x509 -in afip-cert.crt -text -noout

# Check private key matches certificate
openssl rsa -in afip-key.key -check

# Test AFIP connection
curl -X POST https://wsaahomo.afip.gov.ar/ws/services/LoginCms \
  --cert afip-cert.crt \
  --key afip-key.key
```

### Database Migration Issues

```bash
# Reset database (development only)
npx prisma migrate reset

# Create migration without applying
npx prisma migrate dev --create-only

# View migration status
npx prisma migrate status

# Apply pending migrations
npx prisma migrate deploy
```

### LiveKit Connection Debugging

```typescript
// Enable LiveKit debug logs
import { setLogLevel, LogLevel } from 'livekit-client';

setLogLevel(LogLevel.debug);

// Test token generation
const testToken = await videoService.createSessionToken(
  'test-session',
  'Test User',
  true
);

console.log('Token:', testToken);
console.log('Server URL:', process.env.LIVEKIT_URL);
```

## Performance Optimization

### Redis Caching Strategy

```typescript
import { Injectable } from '@nestjs/common';
import { Redis } from 'ioredis';

@Injectable()
export class CacheService {
  private redis: Redis;

  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
  }

  async cacheAppointments(practitionerId: string,
