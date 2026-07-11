---
name: sesion-mental-health-platform
description: Expertise in developing and deploying the Sesión mental health practice management platform for Argentine psychologists with AI orchestration, WhatsApp automation, AFIP billing, and video consultations.
triggers:
  - "help me integrate WhatsApp automation with Sesión"
  - "how do I configure AFIP electronic invoicing in Sesión"
  - "set up Claude AI orchestration for clinical notes"
  - "implement video consultation features with Sesión"
  - "configure the Sesión appointment scheduling engine"
  - "integrate Mercado Pago payments in Sesión"
  - "deploy Sesión with NestJS and SvelteKit"
  - "troubleshoot Sesión WhatsApp Baileys integration"
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What is Sesión?

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina, providing intelligent appointment scheduling, automated WhatsApp patient communication, AFIP-compliant electronic invoicing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Tech Stack

- **Backend**: NestJS (TypeScript) with microservices architecture
- **Frontend**: SvelteKit 5 + Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **Caching**: Redis
- **WhatsApp**: Baileys library for WhatsApp Web API
- **Video**: LiveKit WebRTC
- **AI**: Anthropic Claude Opus 4.6 and Sonnet 4.6
- **Payments**: Stripe + Mercado Pago
- **Message Queue**: Apache Kafka (for event-driven workflows)

## Installation & Setup

### Prerequisites

```bash
# Required tools
node >= 18.0.0
pnpm >= 8.0.0
docker >= 24.0.0
docker-compose >= 2.0.0
postgresql >= 15.0
redis >= 7.0
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic
pnpm install
```

### Environment Configuration

Create `.env` files in backend and frontend directories:

**Backend `.env`:**

```env
# Database
DATABASE_URL="postgresql://postgres:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4-6"
CLAUDE_SONNET_MODEL="claude-sonnet-4-6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# AFIP (Argentina Tax Authority)
AFIP_CERT_PATH="./certs/afip.crt"
AFIP_KEY_PATH="./certs/afip.key"
AFIP_CUIT="${AFIP_CUIT}"
AFIP_ENVIRONMENT="production" # or "testing"

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="${LIVEKIT_WS_URL}"

# Payments
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"

# JWT
JWT_SECRET="${JWT_SECRET}"
JWT_EXPIRATION="15m"
REFRESH_TOKEN_EXPIRATION="7d"

# Kafka
KAFKA_BROKERS="localhost:9092"
KAFKA_CLIENT_ID="sesion-backend"
```

**Frontend `.env`:**

```env
PUBLIC_API_URL="http://localhost:3000"
PUBLIC_LIVEKIT_URL="${PUBLIC_LIVEKIT_URL}"
VITE_SENTRY_DSN="${VITE_SENTRY_DSN}" # optional
```

### Database Setup

```bash
cd backend
npx prisma generate
npx prisma migrate dev --name init
npx prisma db seed
```

### Start Development Environment

```bash
# Start infrastructure (PostgreSQL, Redis, Kafka)
docker-compose up -d

# Start backend (NestJS)
cd backend
pnpm run start:dev

# Start frontend (SvelteKit) in new terminal
cd frontend
pnpm run dev
```

## Core Architecture Patterns

### 1. Microservices Structure (NestJS Backend)

```typescript
// backend/src/modules/appointments/appointments.module.ts
import { Module } from '@nestjs/common';
import { AppointmentsService } from './appointments.service';
import { AppointmentsController } from './appointments.controller';
import { PrismaModule } from '../prisma/prisma.module';
import { WhatsappModule } from '../whatsapp/whatsapp.module';
import { EventsModule } from '../events/events.module';

@Module({
  imports: [PrismaModule, WhatsappModule, EventsModule],
  controllers: [AppointmentsController],
  providers: [AppointmentsService],
  exports: [AppointmentsService],
})
export class AppointmentsModule {}
```

### 2. Appointment Scheduling Service

```typescript
// backend/src/modules/appointments/appointments.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsappService } from '../whatsapp/whatsapp.service';
import { EventsService } from '../events/events.service';
import { CreateAppointmentDto } from './dto/create-appointment.dto';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsappService,
    private events: EventsService,
  ) {}

  async create(createDto: CreateAppointmentDto) {
    // Check for conflicts
    const conflict = await this.checkConflicts(
      createDto.practitionerId,
      createDto.startTime,
      createDto.endTime,
    );

    if (conflict) {
      throw new ConflictException('Time slot already booked');
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        practitionerId: createDto.practitionerId,
        patientId: createDto.patientId,
        startTime: new Date(createDto.startTime),
        endTime: new Date(createDto.endTime),
        type: createDto.type, // 'presencial' | 'virtual' | 'evaluacion'
        status: 'scheduled',
        roomId: createDto.roomId,
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

    // Emit event for Kafka
    await this.events.emitAppointmentCreated(appointment);

    return appointment;
  }

  private async checkConflicts(
    practitionerId: string,
    startTime: string,
    endTime: string,
  ) {
    return this.prisma.appointment.findFirst({
      where: {
        practitionerId,
        status: { in: ['scheduled', 'in_progress'] },
        OR: [
          {
            startTime: { lte: new Date(startTime) },
            endTime: { gt: new Date(startTime) },
          },
          {
            startTime: { lt: new Date(endTime) },
            endTime: { gte: new Date(endTime) },
          },
        ],
      },
    });
  }

  async scheduleReminders() {
    // Find appointments in next 24 hours
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);

    const upcomingAppointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: new Date(),
          lte: tomorrow,
        },
        status: 'scheduled',
        reminderSent: false,
      },
      include: { patient: true, practitioner: true },
    });

    for (const appointment of upcomingAppointments) {
      await this.whatsapp.sendAppointmentReminder(
        appointment.patient.phone,
        appointment,
      );

      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent: true },
      });
    }
  }
}
```

### 3. WhatsApp Automation with Baileys

```typescript
// backend/src/modules/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import * as QRCode from 'qrcode';

@Injectable()
export class WhatsappService implements OnModuleInit {
  private sock: ReturnType<typeof makeWASocket>;
  private qrCodeData: string;

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  private async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './sessions',
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('connection.update', async (update) => {
      const { connection, lastDisconnect, qr } = update;

      if (qr) {
        this.qrCodeData = await QRCode.toDataURL(qr);
      }

      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;

        if (shouldReconnect) {
          await this.connectToWhatsApp();
        }
      }

      if (connection === 'open') {
        console.log('WhatsApp connected successfully');
      }
    });

    this.sock.ev.on('creds.update', saveCreds);

    // Handle incoming messages
    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentConfirmation(phone: string, appointment: any) {
    const message = `¡Hola! Tu sesión con ${appointment.practitioner.name} ha sido confirmada.\n\n📅 Fecha: ${this.formatDate(appointment.startTime)}\n⏰ Hora: ${this.formatTime(appointment.startTime)}\n📍 Modalidad: ${appointment.type === 'virtual' ? 'Videollamada' : 'Presencial'}\n\n¿Confirmas tu asistencia? Responde SÍ o NO.`;

    await this.sendMessage(phone, message);
  }

  async sendAppointmentReminder(phone: string, appointment: any) {
    const message = `Recordatorio: Tu sesión con ${appointment.practitioner.name} es mañana a las ${this.formatTime(appointment.startTime)}.\n\n${appointment.type === 'virtual' ? '🔗 El link de videollamada llegará 15 minutos antes.' : '📍 Dirección: ' + appointment.practitioner.address}`;

    await this.sendMessage(phone, message);
  }

  async sendInvoice(phone: string, invoiceUrl: string, invoiceData: any) {
    const message = `Tu factura ${invoiceData.number} está lista.\n\n💰 Total: $${invoiceData.amount}\n📄 Descarga: ${invoiceUrl}\n\nGracias por confiar en nosotros.`;

    await this.sendMessage(phone, message);
  }

  private async sendMessage(phone: string, text: string) {
    // Format phone number (Argentina: +54 9 11 XXXX-XXXX)
    const formattedPhone = phone.replace(/\D/g, '') + '@s.whatsapp.net';

    try {
      await this.sock.sendMessage(formattedPhone, { text });
    } catch (error) {
      console.error('WhatsApp send error:', error);
      throw error;
    }
  }

  private async handleIncomingMessage(message: WAMessage) {
    if (!message.message || message.key.fromMe) return;

    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text;
    
    const phone = message.key.remoteJid.replace('@s.whatsapp.net', '');

    // Check for crisis keywords
    const crisisKeywords = ['suicidio', 'no puedo más', 'quiero morir', 'crisis'];
    const isCrisis = crisisKeywords.some(keyword => 
      text?.toLowerCase().includes(keyword)
    );

    if (isCrisis) {
      await this.escalateCrisis(phone, text);
      return;
    }

    // Handle appointment confirmations
    if (text?.toLowerCase().match(/^(sí|si|yes|confirmo)$/)) {
      await this.confirmAppointment(phone);
    } else if (text?.toLowerCase().match(/^(no|cancel|cancelar)$/)) {
      await this.cancelAppointment(phone);
    }
  }

  private formatDate(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      day: '2-digit',
      month: 'long',
      year: 'numeric',
    }).format(date);
  }

  private formatTime(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false,
    }).format(date);
  }

  getQRCode(): string {
    return this.qrCodeData;
  }
}
```

### 4. Claude AI Orchestration

```typescript
// backend/src/modules/ai/ai.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AIService {
  private anthropic: Anthropic;
  private opusModel = process.env.CLAUDE_OPUS_MODEL;
  private sonnetModel = process.env.CLAUDE_SONNET_MODEL;

  constructor(private prisma: PrismaService) {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(sessionId: string): Promise<string> {
    // Fetch session notes
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        patient: {
          include: {
            sessions: {
              orderBy: { date: 'desc' },
              take: 5,
            },
          },
        },
      },
    });

    if (!session.notes) {
      throw new Error('No clinical notes found');
    }

    // Use Opus for deep analysis
    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 2000,
      system: `Eres un asistente especializado en psicología clínica en Argentina. Tu tarea es resumir notas de sesión de manera clara, confidencial y profesional, siguiendo las normativas del Colegio de Psicólogos de Argentina.`,
      messages: [
        {
          role: 'user',
          content: `Resume las siguientes notas de sesión, identificando:\n1. Motivo de consulta principal\n2. Técnicas terapéuticas aplicadas\n3. Objetivos terapéuticos abordados\n4. Evolución del paciente\n5. Plan para próxima sesión\n\nNotas:\n${session.notes}\n\nHistorial reciente del paciente (últimas 5 sesiones):\n${this.formatSessionHistory(session.patient.sessions)}`,
        },
      ],
    });

    const summary = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    // Store summary
    await this.prisma.session.update({
      where: { id: sessionId },
      data: { aiSummary: summary },
    });

    // Log AI usage for billing
    await this.prisma.aiUsage.create({
      data: {
        sessionId,
        model: this.opusModel,
        inputTokens: message.usage.input_tokens,
        outputTokens: message.usage.output_tokens,
        cost: this.calculateCost(message.usage),
      },
    });

    return summary;
  }

  async chatAssistant(query: string, context: any): Promise<string> {
    // Use Sonnet for real-time interactions
    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 1000,
      system: `Eres un asistente de Sesión, una plataforma para psicólogos en Argentina. Ayudas a los profesionales a gestionar su práctica, responder preguntas sobre pacientes, y realizar tareas administrativas. Siempre responde en español argentino.`,
      messages: [
        {
          role: 'user',
          content: `Contexto: ${JSON.stringify(context)}\n\nPregunta del psicólogo: ${query}`,
        },
      ],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async predictOutcome(patientId: string): Promise<any> {
    // Fetch comprehensive patient history
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        sessions: {
          orderBy: { date: 'asc' },
        },
        assessments: true,
      },
    });

    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 3000,
      system: `Eres un modelo especializado en análisis predictivo de tratamientos psicológicos. Analiza el historial del paciente y predice posibles resultados del tratamiento basándote en patrones identificados. Responde en formato JSON con campos: riesgo_abandono, progreso_esperado, recomendaciones.`,
      messages: [
        {
          role: 'user',
          content: `Analiza el siguiente historial de paciente:\n\nDatos demográficos: ${JSON.stringify(patient)}\n\nSesiones: ${this.formatSessionHistory(patient.sessions)}\n\nEvaluaciones: ${JSON.stringify(patient.assessments)}`,
        },
      ],
    });

    const analysis = message.content[0].type === 'text'
      ? JSON.parse(message.content[0].text)
      : null;

    return analysis;
  }

  private formatSessionHistory(sessions: any[]): string {
    return sessions
      .map(
        (s, i) =>
          `Sesión ${i + 1} (${s.date.toISOString().split('T')[0]}):\n${s.notes || 'Sin notas'}`,
      )
      .join('\n\n');
  }

  private calculateCost(usage: any): number {
    // Opus 4.6 pricing (example rates)
    const inputCostPerMToken = 15; // $15 per million tokens
    const outputCostPerMToken = 75; // $75 per million tokens

    const inputCost = (usage.input_tokens / 1_000_000) * inputCostPerMToken;
    const outputCost = (usage.output_tokens / 1_000_000) * outputCostPerMToken;

    return inputCost + outputCost;
  }
}
```

### 5. AFIP Electronic Invoicing

```typescript
// backend/src/modules/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as AfipJS from '@afipsdk/afip.js';
import * as fs from 'fs';

@Injectable()
export class AfipService {
  private afip: any;

  constructor() {
    this.afip = new AfipJS({
      CUIT: process.env.AFIP_CUIT,
      cert: fs.readFileSync(process.env.AFIP_CERT_PATH),
      key: fs.readFileSync(process.env.AFIP_KEY_PATH),
      production: process.env.AFIP_ENVIRONMENT === 'production',
    });
  }

  async generateInvoice(invoiceData: {
    patientName: string;
    patientCuit: string;
    amount: number;
    sessionDate: Date;
    conceptType: 'sesion' | 'evaluacion' | 'informe';
  }) {
    const puntoVenta = 1; // Sales point number
    const tipoComprobante = this.getInvoiceType(invoiceData.patientCuit);

    // Get last invoice number
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      puntoVenta,
      tipoComprobante,
    );

    const invoiceNumber = lastInvoice + 1;

    // Create invoice
    const data = {
      CantReg: 1,
      PtoVta: puntoVenta,
      CbteTipo: tipoComprobante,
      Concepto: 2, // Servicios
      DocTipo: 80, // CUIT
      DocNro: invoiceData.patientCuit.replace(/\D/g, ''),
      CbteDesde: invoiceNumber,
      CbteHasta: invoiceNumber,
      CbteFch: this.formatDate(new Date()),
      ImpTotal: invoiceData.amount,
      ImpTotConc: 0,
      ImpNeto: invoiceData.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
      FchServDesde: this.formatDate(invoiceData.sessionDate),
      FchServHasta: this.formatDate(invoiceData.sessionDate),
      FchVtoPago: this.formatDate(new Date()),
    };

    const response = await this.afip.ElectronicBilling.createVoucher(data);

    return {
      cae: response.CAE,
      caeExpiration: response.CAEFchVto,
      invoiceNumber,
      invoiceType: tipoComprobante,
      pdfUrl: await this.generatePDF(data, response),
    };
  }

  private getInvoiceType(cuit: string): number {
    // 1: Factura A (Responsable Inscripto)
    // 6: Factura B (Consumidor Final, Monotributista)
    // 11: Factura C (IVA Exento)
    
    // This is simplified - in production, check patient tax status
    return cuit ? 6 : 6; // Default to Factura B
  }

  private formatDate(date: Date): string {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}${month}${day}`;
  }

  private async generatePDF(data: any, response: any): Promise<string> {
    // Generate PDF using AfipJS built-in PDF generator
    const pdf = await this.afip.ElectronicBilling.createPDF({
      ...data,
      CAE: response.CAE,
      CAEFchVto: response.CAEFchVto,
    });

    // Save to storage and return URL
    const fileName = `invoice_${data.CbteDesde}_${Date.now()}.pdf`;
    const filePath = `./storage/invoices/${fileName}`;
    
    fs.writeFileSync(filePath, pdf);

    return `/invoices/${fileName}`;
  }

  async getMonthlyReport(year: number, month: number) {
    const invoices = await this.afip.ElectronicBilling.getVouchersByMonth(
      year,
      month,
    );

    const total = invoices.reduce((sum, inv) => sum + inv.ImpTotal, 0);

    return {
      totalInvoiced: total,
      invoiceCount: invoices.length,
      invoices,
    };
  }
}
```

### 6. LiveKit Video Integration

```typescript
// backend/src/modules/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  private apiKey = process.env.LIVEKIT_API_KEY;
  private apiSecret = process.env.LIVEKIT_API_SECRET;

  async createRoomToken(
    roomName: string,
    participantName: string,
    isPractitioner: boolean,
  ): Promise<string> {
    const at = new AccessToken(this.apiKey, this.apiSecret, {
      identity: participantName,
      ttl: 3600, // 1 hour
    });

    at.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: isPractitioner, // Only practitioner can send chat
      hidden: false,
    });

    return at.toJwt();
  }

  async generateSessionRoom(appointmentId: string) {
    const roomName = `session_${appointmentId}`;
    
    return {
      roomName,
      wsUrl: process.env.LIVEKIT_WS_URL,
    };
  }

  async recordSession(roomName: string): Promise<void> {
    // Implement LiveKit recording start logic
    // Requires LiveKit Egress service
  }
}
```

### 7. SvelteKit Frontend Component (Appointment Calendar)

```svelte
<!-- frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Calendar } from '$lib/components';
  import type { Appointment } from '$lib/types';

  let appointments: Appointment[] = $state([]);
  let selectedDate: Date = $state(new Date());
  let loading: boolean = $state(false);

  async function loadAppointments(date: Date) {
    loading = true;
    try {
      const response = await fetch(
        `/api/appointments?date=${date.toISOString()}`,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        }
      );
      appointments = await response.json();
    } catch (error) {
      console.error('Failed to load appointments:', error);
    } finally {
      loading = false;
    }
  }

  async function createAppointment(data: any) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
      body: JSON.stringify(data),
    });

    if (response.ok) {
      await loadAppointments(selectedDate);
    }
  }

  onMount(() => {
    loadAppointments(selectedDate);
  });
</script>

<div class="container mx-auto p-6">
  <h1 class="text-3xl font-bold mb-6">Agenda</h1>

  <Calendar
    bind:selectedDate
    {appointments}
    {loading}
    on:dateChange={(e) => loadAppointments(e.detail)}
    on:createAppointment={(e) => createAppointment(e.detail)}
  />

  <div class="mt-8">
    <h2 class="text-xl font-semibold mb-4">
      Sesiones del {selectedDate.toLocaleDateString('es-AR')}
    </h2>

    {#if loading}
      <p>Cargando...</p>
    {:else if appointments.length === 0}
      <p class="text-gray-500">No hay sesiones programadas para este día.</p>
    {:else}
      <div class="space-y-4">
        {#each appointments as appointment}
          <div class="border rounded-lg p-4 hover:shadow-md transition">
            <div class="flex justify-between items-start">
              <div>
                <h3 class="font-medium">{appointment.patient.name}</h3>
                <p class="text-sm text-gray-600">
                  {new Date(appointment.startTime).toLocaleTimeString('es-AR', {
                    hour: '2-digit',
                    minute: '2-digit',
                  })} - 
                  {new Date(appointment.endTime).toLocaleTimeString('es-AR', {
                    hour: '2-digit',
                    minute: '2-digit',
                  })}
                </p>
                <span class="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded mt-2 inline-block">
                  {appointment.type}
                </span>
              </div>
              
              <div class="flex gap-2">
                {#if appointment.type === 'virtual'}
                  <a
                    href="/video/{appointment.id}"
                    class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700"
                  >
                    Iniciar Videollamada
                  </a>
                {/if}
                <button
                  class="px-4 py-2 border rounded hover:bg-gray-50"
                  on:click={() => viewDetails(appointment.id)}
                >
                  Ver Detalles
                </button>
              </div>
            </div>
          </div>
        {/each}
      </div>
    {/if}
  </div>
</div>
```

### 8. Prisma Schema Example

```prisma
// backend/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
