---
name: sesion-mental-health-platform
description: AI-powered mental health practice management platform for psychologists with scheduling, WhatsApp automation, AFIP billing, and secure video consultations
triggers:
  - how do I set up Sesión for a psychology clinic
  - integrate WhatsApp automation for patient reminders
  - configure AFIP compliant invoicing in Sesión
  - implement Claude AI for clinical note summarization
  - set up secure video consultations with Sesión
  - configure appointment scheduling and agenda management
  - use Sesión API for patient management
  - deploy Sesión mental health platform
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**Sesión** is a comprehensive SaaS platform for psychology clinics and independent practitioners, primarily serving the Argentine market. It unifies appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, LiveKit-powered video consultations, and Claude AI (Opus 4.6 + Sonnet 4.6) for clinical workflows.

### Core Stack
- **Backend**: NestJS (TypeScript) microservices
- **Frontend**: SvelteKit 5 + Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **AI**: Anthropic Claude (Opus 4.6 for deep analysis, Sonnet 4.6 for real-time)
- **Video**: LiveKit WebRTC
- **Messaging**: Baileys (WhatsApp automation)
- **Payments**: Stripe + Mercado Pago
- **Billing**: AFIP electronic invoicing integration

## Installation & Setup

### Prerequisites

```bash
# Required tools
node >= 20.x
pnpm >= 8.x
docker >= 24.x
postgresql >= 15.x
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

# AI Models
ANTHROPIC_API_KEY="sk-ant-..."
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./sessions/whatsapp"
WHATSAPP_WEBHOOK_URL="https://your-domain.com/webhooks/whatsapp"

# LiveKit Video
LIVEKIT_API_KEY="APIxxxxx"
LIVEKIT_API_SECRET="xxxxx"
LIVEKIT_WS_URL="wss://your-livekit.com"

# Payment Providers
STRIPE_SECRET_KEY="sk_live_..."
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."

# AFIP (Argentina Tax Authority)
AFIP_CUIT="20123456789"
AFIP_CERTIFICATE_PATH="./certs/afip-cert.pem"
AFIP_PRIVATE_KEY_PATH="./certs/afip-key.key"
AFIP_ENVIRONMENT="production" # or "sandbox"

# Application
JWT_SECRET="your-jwt-secret"
APP_URL="https://sesion.app"
SESSION_TIMEOUT_MINUTES="15"
```

### Database Setup

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate deploy

# Seed database with initial data
pnpm prisma db seed
```

### Run Development Environment

```bash
# Start backend services
pnpm run dev:backend

# Start frontend (separate terminal)
pnpm run dev:frontend

# Start all services with Docker Compose
docker-compose up -d
```

## Project Structure

```
psique-workflow-clinic/
├── apps/
│   ├── backend/           # NestJS microservices
│   │   ├── agenda/        # Scheduling module
│   │   ├── whatsapp/      # WhatsApp automation
│   │   ├── billing/       # AFIP invoicing
│   │   ├── video/         # LiveKit integration
│   │   └── ai/            # Claude AI orchestration
│   └── frontend/          # SvelteKit application
├── packages/
│   ├── prisma/            # Shared database schema
│   ├── types/             # TypeScript definitions
│   └── utils/             # Shared utilities
└── docker-compose.yml
```

## Key Features & Implementation

### 1. Appointment Scheduling (Agenda)

#### Create Appointment

```typescript
// apps/backend/agenda/src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@prisma/client';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number; // minutes
    type: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        startTime: {
          lte: new Date(data.startTime.getTime() + data.duration * 60000),
        },
        endTime: {
          gte: data.startTime,
        },
        status: { not: 'cancelled' },
      },
    });

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    const endTime = new Date(data.startTime.getTime() + data.duration * 60000);

    return this.prisma.appointment.create({
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
  }

  async findAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number = 45
  ) {
    const startOfDay = new Date(date.setHours(0, 0, 0, 0));
    const endOfDay = new Date(date.setHours(23, 59, 59, 999));

    const existingAppointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: startOfDay, lte: endOfDay },
        status: { not: 'cancelled' },
      },
      orderBy: { startTime: 'asc' },
    });

    // Logic to calculate free slots based on working hours
    const workingHours = { start: 9, end: 18 }; // 9 AM to 6 PM
    const slots = [];
    
    let currentTime = new Date(startOfDay.setHours(workingHours.start));
    const endTime = new Date(startOfDay.setHours(workingHours.end));

    while (currentTime < endTime) {
      const slotEnd = new Date(currentTime.getTime() + duration * 60000);
      const hasConflict = existingAppointments.some(apt => 
        (currentTime >= apt.startTime && currentTime < apt.endTime) ||
        (slotEnd > apt.startTime && slotEnd <= apt.endTime)
      );

      if (!hasConflict) {
        slots.push({ start: new Date(currentTime), end: slotEnd });
      }

      currentTime = new Date(currentTime.getTime() + 15 * 60000); // 15-min intervals
    }

    return slots;
  }
}
```

#### SvelteKit Frontend Component

```svelte
<!-- apps/frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  
  let appointments = $state([]);
  let selectedDate = $state(new Date());
  let availableSlots = $state([]);

  async function loadAppointments() {
    const res = await fetch(`/api/appointments?date=${selectedDate.toISOString()}`);
    appointments = await res.json();
  }

  async function loadAvailableSlots(practitionerId: string) {
    const res = await fetch(
      `/api/appointments/available?practitionerId=${practitionerId}&date=${selectedDate.toISOString()}`
    );
    availableSlots = await res.json();
  }

  onMount(() => {
    loadAppointments();
  });
</script>

<div class="agenda-container">
  <h1 class="text-2xl font-bold mb-4">Agenda Inteligente</h1>
  
  <div class="calendar-view">
    {#each appointments as appointment}
      <div class="appointment-card p-4 border rounded-lg mb-2">
        <div class="flex justify-between">
          <span class="font-semibold">{appointment.patient.name}</span>
          <span class="text-gray-600">
            {new Date(appointment.startTime).toLocaleTimeString('es-AR', { 
              hour: '2-digit', 
              minute: '2-digit' 
            })}
          </span>
        </div>
        <span class="text-sm text-gray-500">{appointment.type}</span>
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
</style>
```

### 2. WhatsApp Automation (Baileys)

#### WhatsApp Service

```typescript
// apps/backend/whatsapp/src/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '@prisma/client';

@Injectable()
export class WhatsappService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.initializeWhatsApp();
  }

  async initializeWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('creds.update', saveCreds);

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      
      if (connection === 'close') {
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode 
          !== DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.initializeWhatsApp();
        }
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async handleIncomingMessage(message: WAMessage) {
    if (!message.key.fromMe && message.message) {
      const phoneNumber = message.key.remoteJid?.split('@')[0];
      const messageText = message.message.conversation || 
                         message.message.extendedTextMessage?.text || '';

      // Check for crisis keywords
      const crisisKeywords = ['suicidio', 'autolesión', 'urgencia', 'crisis'];
      const isCrisis = crisisKeywords.some(kw => 
        messageText.toLowerCase().includes(kw)
      );

      if (isCrisis) {
        await this.handleCrisisMessage(phoneNumber, messageText);
      }

      // Store message in database
      await this.prisma.whatsappMessage.create({
        data: {
          phoneNumber,
          message: messageText,
          direction: 'incoming',
          isCrisis,
        },
      });
    }
  }

  async sendAppointmentReminder(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment) return;

    const message = `Hola ${appointment.patient.name}! 👋

Te recordamos tu sesión con ${appointment.practitioner.name}:

📅 ${new Date(appointment.startTime).toLocaleDateString('es-AR')}
🕐 ${new Date(appointment.startTime).toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}
📍 ${appointment.type === 'virtual' ? 'Videollamada' : 'Presencial'}

Por favor confirma tu asistencia respondiendo SÍ o NO.`;

    await this.sock.sendMessage(
      `${appointment.patient.phoneNumber}@s.whatsapp.net`,
      { text: message }
    );

    await this.prisma.whatsappMessage.create({
      data: {
        phoneNumber: appointment.patient.phoneNumber,
        message,
        direction: 'outgoing',
        appointmentId,
      },
    });
  }

  async handleCrisisMessage(phoneNumber: string, message: string) {
    // Alert all practitioners immediately
    const practitioners = await this.prisma.practitioner.findMany({
      where: { notifications: { crisisAlerts: true } },
    });

    for (const practitioner of practitioners) {
      await this.sock.sendMessage(
        `${practitioner.phoneNumber}@s.whatsapp.net`,
        { 
          text: `🚨 ALERTA DE CRISIS\n\nPaciente: ${phoneNumber}\nMensaje: ${message}` 
        }
      );
    }
  }
}
```

#### Schedule Automated Reminders

```typescript
// apps/backend/whatsapp/src/whatsapp-scheduler.service.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '@prisma/client';
import { WhatsappService } from './whatsapp.service';

@Injectable()
export class WhatsappSchedulerService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsappService
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async sendAppointmentReminders() {
    const now = new Date();
    const twentyFourHoursLater = new Date(now.getTime() + 24 * 60 * 60 * 1000);

    // Find appointments in next 24 hours
    const upcomingAppointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: now,
          lte: twentyFourHoursLater,
        },
        status: 'scheduled',
        reminderSent: false,
      },
    });

    for (const appointment of upcomingAppointments) {
      await this.whatsapp.sendAppointmentReminder(appointment.id);
      
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent: true },
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

#### AFIP Service

```typescript
// apps/backend/billing/src/afip/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as AfipServices from '@afipsdk/afip.js';
import * as fs from 'fs';

@Injectable()
export class AfipService {
  private afip: any;

  constructor() {
    this.afip = new AfipServices({
      CUIT: process.env.AFIP_CUIT,
      cert: fs.readFileSync(process.env.AFIP_CERTIFICATE_PATH),
      key: fs.readFileSync(process.env.AFIP_PRIVATE_KEY_PATH),
      production: process.env.AFIP_ENVIRONMENT === 'production',
    });
  }

  async generateInvoice(data: {
    patientCuit: string;
    patientName: string;
    sessionDate: Date;
    amount: number;
    invoiceType: 'A' | 'B' | 'C' | 'M';
  }) {
    const invoiceData = {
      CantReg: 1, // Cantidad de facturas
      PtoVta: 1, // Punto de venta
      CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
      Concepto: 2, // Servicios
      DocTipo: 80, // CUIT
      DocNro: data.patientCuit.replace(/-/g, ''),
      CbteDesde: await this.getNextInvoiceNumber(),
      CbteHasta: await this.getNextInvoiceNumber(),
      CbteFch: this.formatDate(new Date()),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      FchServDesde: this.formatDate(data.sessionDate),
      FchServHasta: this.formatDate(data.sessionDate),
      FchVtoPago: this.formatDate(new Date()),
      MonId: 'PES', // Pesos
      MonCotiz: 1,
    };

    try {
      const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);
      
      return {
        cae: result.CAE, // Código de Autorización Electrónico
        caeExpiration: result.CAEFchVto,
        invoiceNumber: result.CbteDesde,
        qrCode: this.generateQRCode(result),
      };
    } catch (error) {
      throw new Error(`AFIP invoice generation failed: ${error.message}`);
    }
  }

  private getInvoiceTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type] || 6;
  }

  private async getNextInvoiceNumber(): Promise<number> {
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(1, 6);
    return lastInvoice + 1;
  }

  private formatDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  private generateQRCode(afipResponse: any): string {
    // Generate AFIP-compliant QR code data
    const qrData = {
      ver: 1,
      fecha: afipResponse.CbteFch,
      cuit: process.env.AFIP_CUIT,
      ptoVta: afipResponse.PtoVta,
      tipoCmp: afipResponse.CbteTipo,
      nroCmp: afipResponse.CbteDesde,
      importe: afipResponse.ImpTotal,
      moneda: 'PES',
      ctz: 1,
      tipoDocRec: afipResponse.DocTipo,
      nroDocRec: afipResponse.DocNro,
      tipoCodAut: 'E',
      codAut: afipResponse.CAE,
    };
    
    return `https://www.afip.gob.ar/fe/qr/?p=${Buffer.from(JSON.stringify(qrData)).toString('base64')}`;
  }
}
```

#### Billing Controller

```typescript
// apps/backend/billing/src/billing.controller.ts
import { Controller, Post, Body, Get, Param } from '@nestjs/common';
import { AfipService } from './afip/afip.service';
import { PrismaService } from '@prisma/client';

@Controller('billing')
export class BillingController {
  constructor(
    private afip: AfipService,
    private prisma: PrismaService
  ) {}

  @Post('invoice/generate')
  async generateInvoice(@Body() data: {
    appointmentId: string;
    invoiceType: 'A' | 'B' | 'C' | 'M';
  }) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: data.appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment) {
      throw new Error('Appointment not found');
    }

    const afipInvoice = await this.afip.generateInvoice({
      patientCuit: appointment.patient.cuit,
      patientName: appointment.patient.name,
      sessionDate: appointment.startTime,
      amount: appointment.practitioner.sessionPrice,
      invoiceType: data.invoiceType,
    });

    const invoice = await this.prisma.invoice.create({
      data: {
        appointmentId: data.appointmentId,
        cae: afipInvoice.cae,
        caeExpiration: new Date(afipInvoice.caeExpiration),
        invoiceNumber: afipInvoice.invoiceNumber,
        amount: appointment.practitioner.sessionPrice,
        type: data.invoiceType,
        qrCode: afipInvoice.qrCode,
      },
    });

    return invoice;
  }

  @Get('invoice/:id/pdf')
  async downloadInvoicePDF(@Param('id') id: string) {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id },
      include: { appointment: { include: { patient: true, practitioner: true } } },
    });

    // Generate PDF (use library like pdfkit or puppeteer)
    // Return PDF buffer
    return invoice;
  }
}
```

### 4. Claude AI Integration

#### AI Orchestration Service

```typescript
// apps/backend/ai/src/claude/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private opus: Anthropic;
  private sonnet: Anthropic;

  constructor() {
    this.opus = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    this.sonnet = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(sessionNotes: string[]): Promise<string> {
    // Use Opus for deep analysis
    const message = await this.opus.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 2048,
      temperature: 0.3,
      system: `Eres un asistente especializado en psicología clínica en Argentina. 
Tu tarea es resumir notas de sesiones terapéuticas manteniendo confidencialidad 
y destacando patrones relevantes según marcos teóricos argentinos.`,
      messages: [{
        role: 'user',
        content: `Resume las siguientes notas de sesiones:\n\n${sessionNotes.join('\n\n---\n\n')}`
      }]
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async generateClinicalReport(
    patientId: string,
    sessionHistory: any[]
  ): Promise<string> {
    const sessionSummaries = sessionHistory.map(s => 
      `Sesión ${s.sessionNumber} (${new Date(s.date).toLocaleDateString('es-AR')}): ${s.notes}`
    ).join('\n\n');

    const message = await this.opus.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 4096,
      temperature: 0.2,
      system: `Eres un psicólogo clínico experto que genera informes profesionales 
siguiendo estándares éticos del Colegio de Psicólogos de Argentina.`,
      messages: [{
        role: 'user',
        content: `Genera un informe clínico profesional basado en el siguiente historial:\n\n${sessionSummaries}`
      }]
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async chatAssistant(query: string, context?: string): Promise<string> {
    // Use Sonnet for real-time responses
    const message = await this.sonnet.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL,
      max_tokens: 1024,
      temperature: 0.5,
      system: `Eres un asistente de consulta rápida para psicólogos. 
Responde de forma concisa y práctica en español argentino.`,
      messages: [{
        role: 'user',
        content: context 
          ? `Contexto: ${context}\n\nPregunta: ${query}`
          : query
      }]
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async predictOutcome(patientData: {
    age: number;
    diagnosis: string;
    sessionsCompleted: number;
    adherence: number;
  }): Promise<{ risk: 'low' | 'medium' | 'high'; recommendation: string }> {
    const message = await this.opus.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 512,
      temperature: 0.1,
      system: `Eres un sistema de predicción de adherencia terapéutica. 
Analiza datos y devuelve JSON con formato: {"risk": "low|medium|high", "recommendation": "texto"}`,
      messages: [{
        role: 'user',
        content: `Analiza el riesgo de abandono: ${JSON.stringify(patientData)}`
      }]
    });

    const response = message.content[0].type === 'text' ? message.content[0].text : '{}';
    return JSON.parse(response);
  }
}
```

#### AI Controller

```typescript
// apps/backend/ai/src/ai.controller.ts
import { Controller, Post, Body, Get, Param } from '@nestjs/common';
import { ClaudeService } from './claude/claude.service';
import { PrismaService } from '@prisma/client';

@Controller('ai')
export class AiController {
  constructor(
    private claude: ClaudeService,
    private prisma: PrismaService
  ) {}

  @Post('summarize')
  async summarizeSession(@Body() data: { sessionIds: string[] }) {
    const sessions = await this.prisma.session.findMany({
      where: { id: { in: data.sessionIds } },
    });

    const notes = sessions.map(s => s.clinicalNotes);
    const summary = await this.claude.summarizeClinicalNotes(notes);

    return { summary };
  }

  @Get('report/:patientId')
  async generateReport(@Param('patientId') patientId: string) {
    const sessions = await this.prisma.session.findMany({
      where: { patientId },
      orderBy: { date: 'asc' },
    });

    const report = await this.claude.generateClinicalReport(patientId, sessions);
    
    return { report };
  }

  @Post('chat')
  async chatQuery(@Body() data: { query: string; context?: string }) {
    const response = await this.claude.chatAssistant(data.query, data.context);
    return { response };
  }
}
```

### 5. LiveKit Video Consultations

#### Video Service

```typescript
// apps/backend/video/src/livekit/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken, RoomServiceClient } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  private roomService: RoomServiceClient;

  constructor() {
    this.roomService = new RoomServiceClient(
      process.env.LIVEKIT_WS_URL,
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET
    );
  }

  async createRoom(appointmentId: string): Promise<string> {
    const room = await this.roomService.createRoom({
      name: `session-${appointmentId}`,
      emptyTimeout: 60 * 10, // 10 minutes
      maxParticipants: 2,
    });

    return room.name;
  }

  async generateToken(
    roomName: string,
    participantName: string,
    isPractitioner: boolean
  ): Promise<string> {
    const token = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: participantName,
        ttl: 3600, // 1 hour
      }
    );

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: isPractitioner, // Only practitioner can share screen
    });

    return token.toJwt();
  }

  async endSession(roomName: string): Promise<void> {
    await this.roomService.deleteRoom(roomName);
  }

  async listParticipants(roomName: string) {
    return await this.roomService.listParticipants(roomName);
  }
}
```

#### Video Controller

```typescript
// apps/backend/video/src/video.controller.ts
import { Controller, Post, Body, Delete, Param } from '@nestjs/common';
import { LiveKitService } from './livekit/livekit.service';
import { PrismaService } from '@prisma/client';

@Controller('video')
export class VideoController {
  constructor(
    private livekit: LiveKitService,
    private prisma: PrismaService
  ) {}

  @Post('session/start')
  async startVideoSession(@Body() data: { appointmentId: string }) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: data.appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (appointment.type !== 'virtual') {
