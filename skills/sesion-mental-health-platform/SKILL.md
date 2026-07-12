---
name: sesion-mental-health-platform
description: Intelligent practice management platform for psychology clinics with AI-powered scheduling, WhatsApp automation, AFIP billing, and secure video consultations
triggers:
  - "set up Sesión platform for psychology practice"
  - "configure WhatsApp automation for appointment reminders"
  - "integrate AFIP electronic invoicing in Sesión"
  - "implement Claude AI for clinical notes"
  - "create video consultation sessions with LiveKit"
  - "build appointment scheduling with Sesión API"
  - "deploy Sesión mental health platform"
  - "configure Mercado Pago payments for therapy sessions"
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina, combining intelligent appointment scheduling, automated WhatsApp patient communication, AFIP-compliant electronic invoicing, secure video consultations via LiveKit, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit frontend, Prisma ORM, and TypeScript.

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 18.0.0
postgresql >= 14
redis >= 6.2
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### Environment Configuration

**Backend (.env)**

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Authentication
JWT_SECRET="${JWT_SECRET}"
JWT_EXPIRES_IN="7d"

# Anthropic AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_MODEL_OPUS="claude-opus-4-20250514"
CLAUDE_MODEL_SONNET="claude-sonnet-4-20250514"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
MERCADOPAGO_PUBLIC_KEY="${MERCADOPAGO_PUBLIC_KEY}"
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"
STRIPE_WEBHOOK_SECRET="${STRIPE_WEBHOOK_SECRET}"

# AFIP Integration
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="wss://your-livekit-instance.com"

# Application
PORT=3000
FRONTEND_URL="http://localhost:5173"
```

**Frontend (.env)**

```bash
PUBLIC_API_URL="http://localhost:3000"
PUBLIC_LIVEKIT_URL="wss://your-livekit-instance.com"
```

### Database Setup

```bash
cd backend

# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate deploy

# Seed initial data (optional)
npx prisma db seed
```

## Running the Platform

### Development Mode

```bash
# Backend (NestJS)
cd backend
npm run start:dev

# Frontend (SvelteKit)
cd frontend
npm run dev
```

### Production Build

```bash
# Backend
cd backend
npm run build
npm run start:prod

# Frontend
cd frontend
npm run build
npm run preview
```

## Core Architecture

### Backend Structure (NestJS)

```
backend/
├── src/
│   ├── modules/
│   │   ├── agenda/          # Appointment scheduling
│   │   ├── patients/        # Patient management
│   │   ├── whatsapp/        # WhatsApp automation
│   │   ├── billing/         # AFIP invoicing
│   │   ├── video/           # LiveKit integration
│   │   ├── ai/              # Claude orchestration
│   │   └── auth/            # Authentication
│   ├── common/
│   │   ├── guards/
│   │   ├── decorators/
│   │   └── interceptors/
│   └── prisma/
├── prisma/
│   └── schema.prisma
└── package.json
```

## Appointment Scheduling

### Creating Appointments

```typescript
// backend/src/modules/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsappService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AgendaService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsappService,
  ) {}

  async createAppointment(dto: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflicts = await this.checkConflicts(
      dto.practitionerId,
      dto.startTime,
      dto.duration,
    );

    if (conflicts.length > 0) {
      throw new ConflictException('Time slot already occupied');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        practitionerId: dto.practitionerId,
        patientId: dto.patientId,
        startTime: dto.startTime,
        duration: dto.duration,
        type: dto.type, // 'PRESENCIAL' | 'VIRTUAL' | 'EVALUACION'
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(
      appointment.patient.phone,
      appointment,
    );

    return appointment;
  }

  private async checkConflicts(
    practitionerId: string,
    startTime: Date,
    duration: number,
  ) {
    const endTime = new Date(startTime.getTime() + duration * 60000);

    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: { not: 'CANCELLED' },
        OR: [
          {
            startTime: { gte: startTime, lt: endTime },
          },
          {
            AND: [
              { startTime: { lte: startTime } },
              {
                endTime: { gt: startTime },
              },
            ],
          },
        ],
      },
    });
  }
}
```

### Frontend Appointment Calendar

```svelte
<!-- frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Calendar } from '$lib/components/calendar';
  import { agendaStore } from '$lib/stores/agenda';
  
  let appointments = $state([]);
  let selectedDate = $state(new Date());

  onMount(async () => {
    await loadAppointments();
  });

  async function loadAppointments() {
    const response = await fetch('/api/appointments', {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
    });
    appointments = await response.json();
  }

  async function createAppointment(event: CustomEvent) {
    const { patientId, startTime, duration, type } = event.detail;
    
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({
        patientId,
        startTime,
        duration,
        type
      })
    });

    if (response.ok) {
      await loadAppointments();
    }
  }
</script>

<div class="agenda-container">
  <Calendar 
    {appointments} 
    {selectedDate}
    on:create={createAppointment}
  />
</div>
```

## WhatsApp Automation

### Setting Up WhatsApp Session

```typescript
// backend/src/modules/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  useMultiFileAuthState,
  DisconnectReason,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsappService implements OnModuleInit {
  private sock: any;

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  async connectToWhatsApp() {
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
          this.connectToWhatsApp();
        }
      }
    });

    this.sock.ev.on('messages.upsert', async (m) => {
      await this.handleIncomingMessage(m);
    });
  }

  async sendAppointmentConfirmation(phone: string, appointment: any) {
    const message = `Hola ${appointment.patient.firstName}! 👋

Tu sesión ha sido confirmada:
📅 ${this.formatDate(appointment.startTime)}
⏰ ${this.formatTime(appointment.startTime)}
👨‍⚕️ Con ${appointment.practitioner.fullName}
📍 ${appointment.type === 'VIRTUAL' ? 'Videollamada' : 'Presencial'}

Responde SI para confirmar o NO para cancelar.`;

    await this.sock.sendMessage(`${phone}@s.whatsapp.net`, {
      text: message,
    });

    // Store message for tracking
    await this.prisma.whatsappMessage.create({
      data: {
        phone,
        message,
        type: 'APPOINTMENT_CONFIRMATION',
        appointmentId: appointment.id,
        status: 'SENT',
      },
    });
  }

  async sendReminder24Hours(appointment: any) {
    const message = `Recordatorio: Mañana tenés sesión a las ${this.formatTime(appointment.startTime)} con ${appointment.practitioner.fullName}. 

¿Necesitás reprogramar? Respondé REPROGRAMAR.`;

    await this.sock.sendMessage(
      `${appointment.patient.phone}@s.whatsapp.net`,
      { text: message },
    );
  }

  private async handleIncomingMessage(m: any) {
    const message = m.messages[0];
    if (!message.message) return;

    const text = message.message.conversation?.toLowerCase() || '';
    const phone = message.key.remoteJid.replace('@s.whatsapp.net', '');

    // Crisis detection
    if (this.detectCrisis(text)) {
      await this.escalateToPractitioner(phone, text);
      return;
    }

    // Appointment confirmation
    if (text === 'si' || text === 'sí') {
      await this.confirmLatestAppointment(phone);
    } else if (text === 'no') {
      await this.cancelLatestAppointment(phone);
    }
  }

  private detectCrisis(text: string): boolean {
    const crisisKeywords = [
      'suicidio',
      'matarme',
      'no puedo más',
      'crisis',
      'emergencia',
    ];
    return crisisKeywords.some((keyword) => text.includes(keyword));
  }

  private formatDate(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric',
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

### Automated Reminder Cron Job

```typescript
// backend/src/modules/whatsapp/whatsapp.scheduler.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { WhatsappService } from './whatsapp.service';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsappScheduler {
  constructor(
    private whatsapp: WhatsappService,
    private prisma: PrismaService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    tomorrow.setHours(0, 0, 0, 0);

    const dayAfter = new Date(tomorrow);
    dayAfter.setDate(dayAfter.getDate() + 1);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: tomorrow,
          lt: dayAfter,
        },
        status: 'SCHEDULED',
        reminderSent24h: false,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const appointment of appointments) {
      await this.whatsapp.sendReminder24Hours(appointment);

      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent24h: true },
      });
    }
  }

  @Cron('*/15 * * * *') // Every 15 minutes
  async send2HourReminders() {
    const now = new Date();
    const twoHoursLater = new Date(now.getTime() + 2 * 60 * 60 * 1000);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: now,
          lte: twoHoursLater,
        },
        status: 'SCHEDULED',
        reminderSent2h: false,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const appointment of appointments) {
      await this.whatsapp.sendReminder2Hours(appointment);

      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent2h: true },
      });
    }
  }
}
```

## AFIP Electronic Invoicing

### Generating Compliant Invoices

```typescript
// backend/src/modules/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { AfipClient } from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private afipClient: AfipClient;

  constructor(private prisma: PrismaService) {
    this.afipClient = new AfipClient({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.NODE_ENV === 'production',
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

    if (!appointment) {
      throw new NotFoundException('Appointment not found');
    }

    // Determine invoice type based on practitioner fiscal category
    const invoiceType = this.getInvoiceType(
      appointment.practitioner.fiscalCategory,
    );

    const invoiceData = {
      CantReg: 1, // Number of invoices in batch
      PtoVta: appointment.practitioner.afipPointOfSale || 1,
      CbteTipo: invoiceType, // 1=A, 6=B, 11=C
      Concepto: 2, // 1=Products, 2=Services, 3=Both
      DocTipo: 96, // 96=DNI, 80=CUIT
      DocNro: appointment.patient.dni,
      CbteDesde: 1, // Invoice number (auto-increment)
      CbteHasta: 1,
      CbteFch: this.formatDateAfip(new Date()),
      ImpTotal: appointment.amount,
      ImpTotConc: 0, // Non-taxable amount
      ImpNeto: appointment.amount,
      ImpOpEx: 0, // Exempt amount
      ImpIVA: 0, // IVA amount (psychology services are IVA exempt)
      ImpTrib: 0, // Other taxes
      MonId: 'PES', // Currency
      MonCotiz: 1, // Exchange rate
    };

    // Request CAE (Electronic Authorization Code) from AFIP
    const response = await this.afipClient.ElectronicBilling.createVoucher(
      invoiceData,
    );

    if (response.CAE) {
      // Save invoice to database
      const invoice = await this.prisma.invoice.create({
        data: {
          appointmentId: appointment.id,
          practitionerId: appointment.practitionerId,
          patientId: appointment.patientId,
          invoiceNumber: response.CbteDesde,
          invoiceType,
          amount: appointment.amount,
          cae: response.CAE,
          caeExpiration: this.parseAfipDate(response.CAEFchVto),
          afipResponse: response,
          status: 'ISSUED',
        },
      });

      return invoice;
    } else {
      throw new Error(`AFIP error: ${response.Errors}`);
    }
  }

  private getInvoiceType(fiscalCategory: string): number {
    const types = {
      RESPONSABLE_INSCRIPTO: 1, // Factura A
      MONOTRIBUTO: 6, // Factura B
      EXENTO: 11, // Factura C
    };
    return types[fiscalCategory] || 6;
  }

  private formatDateAfip(date: Date): string {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}${month}${day}`;
  }

  private parseAfipDate(dateStr: string): Date {
    const year = parseInt(dateStr.substring(0, 4));
    const month = parseInt(dateStr.substring(4, 6)) - 1;
    const day = parseInt(dateStr.substring(6, 8));
    return new Date(year, month, day);
  }

  async sendInvoiceViaWhatsApp(invoiceId: string) {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id: invoiceId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    const pdfBuffer = await this.generateInvoicePDF(invoice);

    // Send via WhatsApp
    await this.whatsapp.sendInvoice(
      invoice.patient.phone,
      pdfBuffer,
      invoice.invoiceNumber,
    );
  }
}
```

## Claude AI Integration

### Clinical Note Summarization with Opus

```typescript
// backend/src/modules/ai/claude.service.ts
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
    patientId: string,
    sessionIds: string[],
  ): Promise<string> {
    // Fetch session notes
    const sessions = await this.prisma.session.findMany({
      where: {
        id: { in: sessionIds },
        patientId,
      },
      orderBy: { date: 'asc' },
      include: {
        notes: true,
      },
    });

    const notesContext = sessions
      .map(
        (session) =>
          `Sesión ${session.date.toISOString()}: ${session.notes.content}`,
      )
      .join('\n\n');

    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_MODEL_OPUS,
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `Sos un asistente clínico especializado en salud mental en Argentina. 

A continuación te comparto las notas de múltiples sesiones de un paciente. Tu tarea es generar un resumen clínico que identifique:

1. Temas recurrentes y patrones de comportamiento
2. Evolución del paciente a lo largo de las sesiones
3. Objetivos terapéuticos alcanzados
4. Áreas que requieren atención continua
5. Recomendaciones para el plan de tratamiento

NOTAS DE SESIONES:
${notesContext}

Por favor, genera un resumen profesional en español argentino, manteniendo la confidencialidad y el lenguaje clínico apropiado.`,
        },
      ],
    });

    const summary = message.content[0].text;

    // Save AI-generated summary
    await this.prisma.clinicalSummary.create({
      data: {
        patientId,
        sessionIds,
        content: summary,
        model: process.env.CLAUDE_MODEL_OPUS,
        generatedAt: new Date(),
      },
    });

    return summary;
  }

  async generateSessionReport(sessionId: string): Promise<string> {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        patient: true,
        practitioner: true,
        notes: true,
      },
    });

    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_MODEL_OPUS,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Generá un informe de sesión profesional basado en estas notas clínicas:

Paciente: ${session.patient.firstName} ${session.patient.lastName}
Fecha: ${session.date}
Duración: ${session.duration} minutos

Notas del terapeuta:
${session.notes.content}

El informe debe incluir:
- Motivo de consulta
- Observaciones clínicas
- Intervenciones realizadas
- Plan de acción
- Próximos pasos

Formato: profesional, conciso, apto para historial clínico.`,
        },
      ],
    });

    return message.content[0].text;
  }
}
```

### Real-Time Assistant with Sonnet

```typescript
// backend/src/modules/ai/assistant.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class AssistantService {
  private anthropic: Anthropic;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async queryPatientInfo(query: string, context: any): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_MODEL_SONNET,
      max_tokens: 512,
      messages: [
        {
          role: 'user',
          content: `Sos un asistente clínico en tiempo real. Respondé brevemente a la siguiente consulta usando el contexto del paciente provisto:

CONSULTA: ${query}

CONTEXTO DEL PACIENTE:
${JSON.stringify(context, null, 2)}

Respondé de forma concisa y directa, en español argentino.`,
        },
      ],
    });

    return message.content[0].text;
  }

  async suggestDiagnosis(symptoms: string[], history: string): Promise<any> {
    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_MODEL_SONNET,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Basándote en los siguientes síntomas y el historial del paciente, sugerí posibles diagnósticos DSM-5 para considerar (esto es solo asistencia clínica, el diagnóstico final lo hace el profesional):

SÍNTOMAS:
${symptoms.join('\n- ')}

HISTORIAL:
${history}

Formato de respuesta: JSON con array de posibles diagnósticos, cada uno con código DSM-5, nombre, nivel de confianza (bajo/medio/alto), y justificación breve.`,
        },
      ],
    });

    return JSON.parse(message.content[0].text);
  }
}
```

## Video Consultations with LiveKit

### Creating Video Sessions

```typescript
// backend/src/modules/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class VideoService {
  constructor(private prisma: PrismaService) {}

  async createVideoSession(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (appointment.type !== 'VIRTUAL') {
      throw new BadRequestException('Appointment is not virtual');
    }

    // Create room name
    const roomName = `session-${appointmentId}`;

    // Generate practitioner token
    const practitionerToken = this.generateToken(
      roomName,
      appointment.practitioner.id,
      appointment.practitioner.fullName,
      { canPublish: true, canSubscribe: true, canRecord: true },
    );

    // Generate patient token
    const patientToken = this.generateToken(
      roomName,
      appointment.patient.id,
      appointment.patient.fullName,
      { canPublish: true, canSubscribe: true, canRecord: false },
    );

    // Save session info
    const videoSession = await this.prisma.videoSession.create({
      data: {
        appointmentId,
        roomName,
        status: 'WAITING',
        startedAt: null,
      },
    });

    return {
      roomName,
      practitionerToken,
      patientToken,
      wsUrl: process.env.LIVEKIT_WS_URL,
      sessionId: videoSession.id,
    };
  }

  private generateToken(
    roomName: string,
    userId: string,
    userName: string,
    permissions: any,
  ): string {
    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: userId,
        name: userName,
      },
    );

    at.addGrant({
      room: roomName,
      roomJoin: true,
      canPublish: permissions.canPublish,
      canSubscribe: permissions.canSubscribe,
      canPublishData: true,
    });

    return at.toJwt();
  }

  async startRecording(sessionId: string) {
    // Implement LiveKit recording start
    // This requires LiveKit Egress service configured
    await this.prisma.videoSession.update({
      where: { id: sessionId },
      data: { recording: true },
    });
  }

  async endSession(sessionId: string) {
    const session = await this.prisma.videoSession.update({
      where: { id: sessionId },
      data: {
        status: 'COMPLETED',
        endedAt: new Date(),
      },
    });

    return session;
  }
}
```

### Frontend Video Component

```svelte
<!-- frontend/src/lib/components/VideoRoom.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import {
    Room,
    RoomEvent,
    Track,
    RemoteParticipant,
  } from 'livekit-client';

  export let wsUrl: string;
  export let token: string;
  export let roomName: string;

  let room: Room;
  let localVideo: HTMLVideoElement;
  let remoteVideo: HTMLVideoElement;
  let isMuted = $state(false);
  let isCameraOff = $state(false);

  onMount(async () => {
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
    });

    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.TrackUnsubscribed, handleTrackUnsubscribed);

    await room.connect(wsUrl, token);

    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();

    const localTrack = room.localParticipant.videoTracks.values().next().value;
    if (localTrack && localVideo) {
      localTrack.track.attach(localVideo);
    }
  });

  function handleTrackSubscribed(
    track: RemoteTrackPublication,
    publication: RemoteTrackPublication,
    participant: RemoteParticipant
  ) {
    if (track.kind === Track.Kind.Video && remoteVideo) {
      track.attach
