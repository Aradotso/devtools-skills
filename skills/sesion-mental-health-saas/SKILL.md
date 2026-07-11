---
name: sesion-mental-health-saas
description: Build and customize the Sesión mental health practice management platform with appointment scheduling, WhatsApp automation, AFIP billing, and Claude AI integration
triggers:
  - how do I set up the Sesión mental health platform
  - integrate WhatsApp automation with Sesión
  - configure AFIP electronic billing in Sesión
  - set up Claude AI models in Sesión
  - build video consultation features in Sesión
  - customize appointment scheduling in Sesión
  - deploy Sesión for Argentine psychology clinics
  - integrate MercadoPago payments in Sesión
---

# Sesión Mental Health Practice Management Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It orchestrates appointment scheduling, automated WhatsApp messaging, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6 models.

## Architecture Overview

Sesión uses a microservices architecture built with:
- **Backend**: NestJS (TypeScript) for API orchestration
- **Frontend**: SvelteKit 5 with Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **Messaging**: Baileys for WhatsApp automation
- **Video**: LiveKit for WebRTC consultations
- **AI**: Anthropic Claude (Opus 4.6 + Sonnet 4.6)
- **Payments**: Stripe + MercadoPago
- **Cache**: Redis for session management

## Installation

### Prerequisites

```bash
# Required versions
node >= 18.0.0
postgresql >= 14.0
redis >= 6.0
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="your_anthropic_key"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your_webhook_secret"

# LiveKit Video
LIVEKIT_API_KEY="your_livekit_key"
LIVEKIT_API_SECRET="your_livekit_secret"
LIVEKIT_URL="wss://your-livekit-server.com"

# Payment Processors
STRIPE_SECRET_KEY="sk_test_..."
MERCADOPAGO_ACCESS_TOKEN="your_mp_token"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="your_clinic_cuit"
AFIP_CERTIFICATE_PATH="./afip-cert.pem"
AFIP_PRIVATE_KEY_PATH="./afip-key.pem"

# Application
JWT_SECRET="your_jwt_secret"
APP_URL="http://localhost:3000"
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

### Start Development Server

```bash
# Start backend
npm run dev:backend

# Start frontend (separate terminal)
npm run dev:frontend

# Start both with concurrently
npm run dev
```

## Prisma Schema Key Models

```prisma
// prisma/schema.prisma

model Practitioner {
  id            String        @id @default(uuid())
  email         String        @unique
  name          String
  fiscalCategory String       // Monotributo, Responsable Inscripto
  appointments  Appointment[]
  patients      Patient[]
  invoices      Invoice[]
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt
}

model Patient {
  id              String        @id @default(uuid())
  name            String
  email           String?
  phone           String        @unique
  whatsappOptIn   Boolean       @default(false)
  practitionerId  String
  practitioner    Practitioner  @relation(fields: [practitionerId], references: [id])
  appointments    Appointment[]
  journeyStage    String        @default("primer_contacto")
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
}

model Appointment {
  id              String       @id @default(uuid())
  practitionerId  String
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  patientId       String
  patient         Patient      @relation(fields: [patientId], references: [id])
  startTime       DateTime
  endTime         DateTime
  sessionType     String       // presencial, virtual, evaluacion
  status          String       @default("scheduled") // scheduled, confirmed, cancelled, completed
  roomUrl         String?      // LiveKit room URL
  invoiceId       String?      @unique
  invoice         Invoice?     @relation(fields: [invoiceId], references: [id])
  createdAt       DateTime     @default(now())
  updatedAt       DateTime     @updatedAt
}

model Invoice {
  id              String       @id @default(uuid())
  practitionerId  String
  practitioner    Practitioner @relation(fields: [practitionerId], references: [id])
  appointment     Appointment?
  invoiceNumber   String       @unique
  afipCae         String?      // AFIP authorization code
  amount          Decimal
  fiscalType      String       // A, B, C, M
  paymentMethod   String
  paymentStatus   String       @default("pending")
  issuedAt        DateTime     @default(now())
  createdAt       DateTime     @default(now())
}
```

## Appointment Scheduling API

### Create Appointment

```typescript
// backend/src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: {
    practitionerId: string;
    patientId: string;
    startTime: Date;
    sessionType: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Validate time slot availability
    const conflict = await this.checkConflicts(
      data.practitionerId,
      data.startTime,
    );

    if (conflict) {
      throw new Error('Time slot already occupied');
    }

    // Calculate end time (45 min default)
    const endTime = new Date(data.startTime.getTime() + 45 * 60000);

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime,
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation if patient opted in
    if (appointment.patient.whatsappOptIn) {
      await this.whatsapp.sendAppointmentConfirmation(appointment);
    }

    return appointment;
  }

  private async checkConflicts(practitionerId: string, startTime: Date) {
    const endTime = new Date(startTime.getTime() + 45 * 60000);

    return this.prisma.appointment.findFirst({
      where: {
        practitionerId,
        status: { not: 'cancelled' },
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
}
```

### Appointment Controller

```typescript
// backend/src/appointments/appointments.controller.ts
import { Controller, Post, Body, UseGuards } from '@nestjs/common';
import { AppointmentsService } from './appointments.service';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';

@Controller('appointments')
@UseGuards(JwtAuthGuard)
export class AppointmentsController {
  constructor(private appointmentsService: AppointmentsService) {}

  @Post()
  async create(@Body() createDto: CreateAppointmentDto) {
    return this.appointmentsService.createAppointment(createDto);
  }
}
```

## WhatsApp Automation with Baileys

### WhatsApp Service Setup

```typescript
// backend/src/whatsapp/whatsapp.service.ts
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
    await this.connectToWhatsApp();
  }

  private async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH,
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;

      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;

        if (shouldReconnect) {
          this.connectToWhatsApp();
        }
      }
    });

    this.sock.ev.on('creds.update', saveCreds);
    this.sock.ev.on('messages.upsert', this.handleIncomingMessage.bind(this));
  }

  async sendAppointmentConfirmation(appointment: any) {
    const message = this.buildConfirmationMessage(appointment);
    const phoneNumber = this.formatPhoneNumber(appointment.patient.phone);

    await this.sock.sendMessage(phoneNumber, { text: message });
  }

  async sendReminder(appointment: any, hoursBeforeMin: number) {
    const message = `🔔 Recordatorio: Tenés sesión ${
      hoursBeforeMin === 2 ? 'en 2 horas' : 'mañana'
    }\n\n` +
      `📅 ${this.formatDateTime(appointment.startTime)}\n` +
      `👨‍⚕️ ${appointment.practitioner.name}\n` +
      `📍 ${appointment.sessionType === 'virtual' ? 'Videollamada' : 'Presencial'}\n\n` +
      `Respondé SI para confirmar o NO para cancelar.`;

    const phoneNumber = this.formatPhoneNumber(appointment.patient.phone);
    await this.sock.sendMessage(phoneNumber, { text: message });
  }

  private async handleIncomingMessage({ messages }: { messages: WAMessage[] }) {
    for (const msg of messages) {
      if (!msg.message || msg.key.fromMe) continue;

      const text = msg.message.conversation?.toLowerCase();
      const phone = msg.key.remoteJid;

      // Handle confirmation responses
      if (text === 'si' || text === 'sí') {
        await this.confirmAppointment(phone);
      } else if (text === 'no') {
        await this.handleCancellationRequest(phone);
      }
    }
  }

  private formatPhoneNumber(phone: string): string {
    // Convert to WhatsApp format (Argentina: 549 + area + number)
    return `549${phone.replace(/[^0-9]/g, '')}@s.whatsapp.net`;
  }

  private buildConfirmationMessage(appointment: any): string {
    return `✅ Sesión agendada\n\n` +
      `📅 ${this.formatDateTime(appointment.startTime)}\n` +
      `👨‍⚕️ ${appointment.practitioner.name}\n` +
      `📍 ${appointment.sessionType === 'virtual' ? 'Videollamada' : 'Presencial'}\n\n` +
      `Te enviaremos recordatorios 24hs y 2hs antes de la sesión.`;
  }

  private formatDateTime(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
    }).format(date);
  }
}
```

### Scheduled Reminder Jobs

```typescript
// backend/src/appointments/reminder.scheduler.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class ReminderScheduler {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: tomorrow,
          lt: new Date(tomorrow.getTime() + 60 * 60000),
        },
        status: 'scheduled',
        patient: { whatsappOptIn: true },
      },
      include: { patient: true, practitioner: true },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendReminder(apt, 24);
    }
  }

  @Cron(CronExpression.EVERY_30_MINUTES)
  async send2HourReminders() {
    const twoHoursFromNow = new Date();
    twoHoursFromNow.setHours(twoHoursFromNow.getHours() + 2);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: twoHoursFromNow,
          lt: new Date(twoHoursFromNow.getTime() + 30 * 60000),
        },
        status: 'scheduled',
        patient: { whatsappOptIn: true },
      },
      include: { patient: true, practitioner: true },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendReminder(apt, 2);
    }
  }
}
```

## AFIP Electronic Billing Integration

### AFIP Service

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import Afip from '@afipsdk/afip.js';

@Injectable()
export class AfipService {
  private afip: any;

  constructor() {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: readFileSync(process.env.AFIP_CERTIFICATE_PATH),
      key: readFileSync(process.env.AFIP_PRIVATE_KEY_PATH),
      production: process.env.NODE_ENV === 'production',
    });
  }

  async generateInvoice(data: {
    fiscalType: 'A' | 'B' | 'C' | 'M';
    amount: number;
    patientCuit?: string;
    concept: number; // 1=Productos, 2=Servicios, 3=Productos y Servicios
  }) {
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      1, // Punto de venta
      data.fiscalType === 'A' ? 1 : data.fiscalType === 'B' ? 6 : 11,
    );

    const invoiceNumber = lastInvoice + 1;

    const invoiceData = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: data.fiscalType === 'A' ? 1 : data.fiscalType === 'B' ? 6 : 11,
      Concepto: data.concept,
      DocTipo: data.patientCuit ? 80 : 99, // 80=CUIT, 99=Sin identificar
      DocNro: data.patientCuit || 0,
      CbteDesde: invoiceNumber,
      CbteHasta: invoiceNumber,
      CbteFch: this.formatAfipDate(new Date()),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.fiscalType === 'A' ? data.amount / 1.21 : data.amount,
      ImpOpEx: 0,
      ImpIVA: data.fiscalType === 'A' ? data.amount - data.amount / 1.21 : 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    };

    if (data.fiscalType === 'A') {
      invoiceData['Iva'] = [
        {
          Id: 5, // 21%
          BaseImp: data.amount / 1.21,
          Importe: data.amount - data.amount / 1.21,
        },
      ];
    }

    const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    return {
      invoiceNumber: `${String(1).padStart(4, '0')}-${String(invoiceNumber).padStart(8, '0')}`,
      cae: result.CAE,
      caeExpiration: result.CAEFchVto,
      afipResult: result,
    };
  }

  private formatAfipDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }
}
```

### Invoice Service

```typescript
// backend/src/billing/invoice.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { AfipService } from './afip.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class InvoiceService {
  constructor(
    private prisma: PrismaService,
    private afip: AfipService,
    private whatsapp: WhatsAppService,
  ) {}

  async createInvoiceForAppointment(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { practitioner: true, patient: true },
    });

    if (!appointment) throw new Error('Appointment not found');

    // Determine session amount (configurable per practitioner)
    const amount = 15000; // Example amount in ARS

    // Generate AFIP invoice
    const afipResult = await this.afip.generateInvoice({
      fiscalType: appointment.practitioner.fiscalCategory === 'Responsable Inscripto' ? 'A' : 'B',
      amount,
      concept: 2, // Servicios
    });

    // Save invoice
    const invoice = await this.prisma.invoice.create({
      data: {
        practitionerId: appointment.practitionerId,
        invoiceNumber: afipResult.invoiceNumber,
        afipCae: afipResult.cae,
        amount,
        fiscalType: appointment.practitioner.fiscalCategory === 'Responsable Inscripto' ? 'A' : 'B',
        paymentMethod: 'pending',
        paymentStatus: 'pending',
      },
    });

    // Link to appointment
    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: { invoiceId: invoice.id },
    });

    // Send invoice via WhatsApp
    if (appointment.patient.whatsappOptIn) {
      await this.sendInvoiceViaWhatsApp(invoice, appointment);
    }

    return invoice;
  }

  private async sendInvoiceViaWhatsApp(invoice: any, appointment: any) {
    const message = `🧾 Factura Electrónica\n\n` +
      `Nº ${invoice.invoiceNumber}\n` +
      `CAE: ${invoice.afipCae}\n` +
      `Monto: $${invoice.amount.toFixed(2)}\n\n` +
      `Podés abonar por:\n` +
      `💳 Mercado Pago\n` +
      `💵 Transferencia\n` +
      `💰 Efectivo en próxima sesión`;

    await this.whatsapp.sock.sendMessage(
      this.whatsapp.formatPhoneNumber(appointment.patient.phone),
      { text: message },
    );
  }
}
```

## Claude AI Integration

### AI Orchestration Service

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

  async summarizeClinicalNotes(notes: string[], patientContext: string) {
    // Use Opus for deep analysis
    const response = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 2048,
      temperature: 0.3,
      system: `Sos un asistente especializado en psicología clínica en Argentina. 
Tu tarea es resumir notas de sesiones terapéuticas de manera clara y estructurada,
respetando la confidencialidad y terminología profesional.`,
      messages: [
        {
          role: 'user',
          content: `Contexto del paciente: ${patientContext}\n\n` +
            `Notas de sesiones:\n${notes.join('\n\n---\n\n')}\n\n` +
            `Generá un resumen clínico estructurado que incluya:\n` +
            `1. Motivo de consulta\n` +
            `2. Evolución del tratamiento\n` +
            `3. Técnicas aplicadas\n` +
            `4. Observaciones relevantes\n` +
            `5. Plan terapéutico sugerido`,
        },
      ],
    });

    return response.content[0].type === 'text' ? response.content[0].text : '';
  }

  async generateQuickResponse(query: string, context: string) {
    // Use Sonnet for fast responses
    const response = await this.anthropic.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL,
      max_tokens: 512,
      temperature: 0.5,
      system: `Sos un asistente clínico que ayuda a psicólogos durante sus sesiones.
Respondé de manera concisa y profesional en español argentino.`,
      messages: [
        {
          role: 'user',
          content: `Contexto: ${context}\n\nConsulta: ${query}`,
        },
      ],
    });

    return response.content[0].type === 'text' ? response.content[0].text : '';
  }

  async analyzePatientRisk(patientHistory: any) {
    const response = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 1024,
      temperature: 0.2,
      system: `Sos un sistema de detección de riesgo clínico. Analizá el historial del paciente
e identificá indicadores de abandono de tratamiento o crisis. Respondé en formato JSON.`,
      messages: [
        {
          role: 'user',
          content: JSON.stringify(patientHistory),
        },
      ],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '{}';
    return JSON.parse(text);
  }
}
```

### AI Controller with Model Routing

```typescript
// backend/src/ai/ai.controller.ts
import { Controller, Post, Body, UseGuards } from '@nestjs/common';
import { ClaudeService } from './claude.service';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';

@Controller('ai')
@UseGuards(JwtAuthGuard)
export class AIController {
  constructor(private claude: ClaudeService) {}

  @Post('summarize-notes')
  async summarizeNotes(@Body() dto: { notes: string[]; patientId: string }) {
    // Fetch patient context
    const patient = await this.getPatientContext(dto.patientId);
    
    return this.claude.summarizeClinicalNotes(dto.notes, patient);
  }

  @Post('quick-assist')
  async quickAssist(@Body() dto: { query: string; context: string }) {
    return this.claude.generateQuickResponse(dto.query, dto.context);
  }

  @Post('risk-assessment')
  async assessRisk(@Body() dto: { patientId: string }) {
    const history = await this.getPatientHistory(dto.patientId);
    return this.claude.analyzePatientRisk(history);
  }
}
```

## LiveKit Video Consultation

### Video Service

```typescript
// backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  async createRoomToken(
    roomName: string,
    participantName: string,
    isPractitioner: boolean,
  ) {
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
      canPublishData: isPractitioner, // Only practitioner can send chat
    });

    return at.toJwt();
  }

  async generateSessionUrl(appointmentId: string, isPractitioner: boolean) {
    const roomName = `session-${appointmentId}`;
    const participantName = isPractitioner ? 'practitioner' : 'patient';

    const token = await this.createRoomToken(
      roomName,
      participantName,
      isPractitioner,
    );

    return {
      url: `${process.env.APP_URL}/video/${roomName}`,
      token,
      serverUrl: process.env.LIVEKIT_URL,
    };
  }
}
```

### Video Component (SvelteKit)

```svelte
<!-- frontend/src/routes/video/[roomName]/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';
  
  export let data; // { token, serverUrl }
  
  let videoContainer: HTMLDivElement;
  let room: Room;
  
  onMount(async () => {
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
    });
    
    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.Disconnected, handleDisconnect);
    
    await room.connect(data.serverUrl, data.token);
    
    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
  });
  
  function handleTrackSubscribed(track, publication, participant) {
    if (track.kind === 'video' || track.kind === 'audio') {
      const element = track.attach();
      videoContainer.appendChild(element);
    }
  }
  
  function handleDisconnect() {
    console.log('Disconnected from room');
  }
  
  async function toggleMute() {
    await room.localParticipant.setMicrophoneEnabled(
      !room.localParticipant.isMicrophoneEnabled
    );
  }
  
  async function toggleVideo() {
    await room.localParticipant.setCameraEnabled(
      !room.localParticipant.isCameraEnabled
    );
  }
</script>

<div class="video-consultation">
  <div bind:this={videoContainer} class="video-grid"></div>
  
  <div class="controls">
    <button on:click={toggleMute} class="btn-control">
      {room?.localParticipant.isMicrophoneEnabled ? '🎤' : '🔇'}
    </button>
    <button on:click={toggleVideo} class="btn-control">
      {room?.localParticipant.isCameraEnabled ? '📹' : '🚫'}
    </button>
  </div>
</div>

<style>
  .video-consultation {
    height: 100vh;
    display: flex;
    flex-direction: column;
  }
  
  .video-grid {
    flex: 1;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1rem;
    padding: 1rem;
  }
  
  .controls {
    display: flex;
    justify-content: center;
    gap: 1rem;
    padding: 1rem;
    background: #1f2937;
  }
  
  .btn-control {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    font-size: 24px;
    border: none;
    background: #374151;
    cursor: pointer;
  }
</style>
```

## Payment Integration

### MercadoPago
