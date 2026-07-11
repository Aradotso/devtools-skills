---
name: sesion-mental-health-platform
description: Build and integrate with Sesión, an AI-powered mental health practice management platform for Argentine psychologists with WhatsApp automation, AFIP billing, and video consultations.
triggers:
  - "help me build a psychology clinic management system"
  - "integrate WhatsApp appointment reminders for mental health practice"
  - "create AFIP-compliant invoicing for Argentine clinic"
  - "set up video consultation system with Claude AI"
  - "build patient journey pipeline for therapists"
  - "implement appointment scheduling with WhatsApp automation"
  - "add AI clinical note summarization with Claude"
  - "create mental health SaaS platform"
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS orchestration platform for psychology clinics in Argentina, built with NestJS backend, SvelteKit frontend, and integrating Claude AI (Opus 4.6 + Sonnet 4.6) for clinical intelligence. The platform unifies:

- **Intelligent scheduling** with conflict detection and resource allocation
- **WhatsApp automation** via Baileys for appointment reminders and patient communication
- **AFIP-compliant electronic invoicing** (Facturación Electrónica)
- **Secure video consultations** using LiveKit WebRTC infrastructure
- **AI-powered clinical workflows** with Claude for note summarization and patient insights
- **Payment processing** via Mercado Pago (Argentina) and Stripe (international)

The tech stack: **TypeScript, NestJS, SvelteKit 5, Svelte 5, Prisma ORM, PostgreSQL, Redis, TailwindCSS**.

## Installation & Setup

### Prerequisites

```bash
# Required tools
node >= 18.x
pnpm >= 8.x
postgresql >= 14.x
redis >= 7.x
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install
```

### Environment Configuration

Create `.env` in project root:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic Claude
ANTHROPIC_API_KEY="your-api-key-here"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your-webhook-secret"

# LiveKit Video
LIVEKIT_API_KEY="your-livekit-key"
LIVEKIT_API_SECRET="your-livekit-secret"
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# AFIP (Argentine Tax Authority)
AFIP_CUIT="20-12345678-9"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
AFIP_ENVIRONMENT="testing" # or "production"

# Payment Processors
MERCADOPAGO_ACCESS_TOKEN="your-mp-token"
STRIPE_SECRET_KEY="your-stripe-key"

# Application
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
JWT_SECRET="your-jwt-secret"
```

### Database Setup

```bash
# Run Prisma migrations
pnpm prisma migrate dev

# Seed initial data (optional)
pnpm prisma db seed
```

### Run Development Servers

```bash
# Start backend (NestJS)
cd backend
pnpm run start:dev

# Start frontend (SvelteKit) - in new terminal
cd frontend
pnpm run dev
```

## Core Architecture

### Backend Structure (NestJS)

```
backend/
├── src/
│   ├── agenda/           # Appointment scheduling module
│   ├── whatsapp/         # WhatsApp automation via Baileys
│   ├── billing/          # AFIP invoicing + payment processing
│   ├── video/            # LiveKit video consultation
│   ├── ai/               # Claude AI orchestration
│   ├── patients/         # Patient management
│   ├── practitioners/    # Practitioner profiles
│   ├── auth/             # Authentication & authorization
│   └── common/           # Shared utilities
├── prisma/
│   └── schema.prisma     # Database schema
└── package.json
```

### Frontend Structure (SvelteKit)

```
frontend/
├── src/
│   ├── routes/           # SvelteKit file-based routing
│   ├── lib/
│   │   ├── components/   # Svelte 5 components
│   │   ├── stores/       # State management
│   │   └── api/          # API client functions
│   └── app.html
└── package.json
```

## Key Modules & APIs

### 1. Agenda Management (Appointment Scheduling)

#### Backend Service (NestJS)

```typescript
// backend/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    practitionerId: string;
    patientId: string;
    startTime: Date;
    duration: number; // minutes
    type: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Check for conflicts
    const conflicts = await this.findConflicts(
      data.practitionerId,
      data.startTime,
      data.duration
    );

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        practitionerId: data.practitionerId,
        patientId: data.patientId,
        startTime: data.startTime,
        endTime: new Date(data.startTime.getTime() + data.duration * 60000),
        type: data.type,
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Trigger WhatsApp confirmation
    await this.triggerWhatsAppConfirmation(appointment);

    return appointment;
  }

  async findConflicts(practitionerId: string, startTime: Date, duration: number) {
    const endTime = new Date(startTime.getTime() + duration * 60000);

    return this.prisma.appointment.findMany({
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

  async getAvailableSlots(
    practitionerId: string,
    date: Date,
    sessionDuration: number = 45
  ) {
    const workdayStart = new Date(date);
    workdayStart.setHours(9, 0, 0, 0);
    const workdayEnd = new Date(date);
    workdayEnd.setHours(20, 0, 0, 0);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: workdayStart, lt: workdayEnd },
        status: { not: 'cancelled' },
      },
      orderBy: { startTime: 'asc' },
    });

    const slots = [];
    let currentTime = workdayStart;

    for (const apt of appointments) {
      while (currentTime < apt.startTime) {
        const slotEnd = new Date(currentTime.getTime() + sessionDuration * 60000);
        if (slotEnd <= apt.startTime) {
          slots.push({ start: new Date(currentTime), end: slotEnd });
          currentTime = slotEnd;
        } else {
          break;
        }
      }
      currentTime = apt.endTime;
    }

    while (currentTime < workdayEnd) {
      const slotEnd = new Date(currentTime.getTime() + sessionDuration * 60000);
      if (slotEnd <= workdayEnd) {
        slots.push({ start: new Date(currentTime), end: slotEnd });
      }
      currentTime = slotEnd;
    }

    return slots;
  }
}
```

#### Frontend Component (Svelte 5)

```svelte
<!-- frontend/src/lib/components/AppointmentScheduler.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { api } from '$lib/api/client';
  
  let {
    practitionerId = $bindable(),
    patientId = $bindable(),
  } = $props();

  let selectedDate = $state(new Date());
  let availableSlots = $state([]);
  let selectedSlot = $state(null);

  async function loadAvailableSlots() {
    const response = await api.get(`/agenda/available-slots`, {
      params: {
        practitionerId,
        date: selectedDate.toISOString(),
        duration: 45,
      },
    });
    availableSlots = response.data;
  }

  async function bookAppointment() {
    if (!selectedSlot) return;

    await api.post('/agenda/appointments', {
      practitionerId,
      patientId,
      startTime: selectedSlot.start,
      duration: 45,
      type: 'virtual',
    });

    alert('Cita agendada con éxito');
  }

  $effect(() => {
    if (practitionerId) {
      loadAvailableSlots();
    }
  });
</script>

<div class="scheduler-container">
  <h2>Agendar Sesión</h2>
  
  <input
    type="date"
    bind:value={selectedDate}
    onchange={() => loadAvailableSlots()}
  />

  <div class="slots-grid">
    {#each availableSlots as slot}
      <button
        class:selected={selectedSlot === slot}
        onclick={() => selectedSlot = slot}
      >
        {new Date(slot.start).toLocaleTimeString('es-AR', {
          hour: '2-digit',
          minute: '2-digit'
        })}
      </button>
    {/each}
  </div>

  <button onclick={bookAppointment} disabled={!selectedSlot}>
    Confirmar Cita
  </button>
</div>

<style>
  .slots-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 0.5rem;
  }
  
  button.selected {
    @apply bg-blue-600 text-white;
  }
</style>
```

### 2. WhatsApp Automation (Baileys)

```typescript
// backend/src/whatsapp/whatsapp.service.ts
import { Injectable } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService {
  private sock: any;

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('creds.update', saveCreds);

    this.sock.ev.on('connection.update', (update: any) => {
      const { connection, lastDisconnect } = update;
      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.initialize();
        }
      }
    });

    this.sock.ev.on('messages.upsert', async (m: any) => {
      await this.handleIncomingMessage(m);
    });
  }

  async sendAppointmentReminder(appointment: any) {
    const patientPhone = this.formatPhone(appointment.patient.phone);
    const message = this.generateReminderMessage(appointment);

    await this.sock.sendMessage(patientPhone, { text: message });
  }

  async sendInvoice(invoice: any, patientPhone: string) {
    const phone = this.formatPhone(patientPhone);
    const pdfBuffer = await this.generateInvoicePDF(invoice);

    await this.sock.sendMessage(phone, {
      document: pdfBuffer,
      mimetype: 'application/pdf',
      fileName: `Factura_${invoice.number}.pdf`,
      caption: `Factura Nº ${invoice.number} - ${invoice.amount} ARS`,
    });
  }

  private generateReminderMessage(appointment: any): string {
    const time = new Date(appointment.startTime).toLocaleString('es-AR', {
      weekday: 'long',
      day: 'numeric',
      month: 'long',
      hour: '2-digit',
      minute: '2-digit',
    });

    return `Hola ${appointment.patient.firstName},\n\n` +
      `Te recordamos tu sesión con ${appointment.practitioner.name} ` +
      `el ${time}.\n\n` +
      `Tipo: ${appointment.type === 'virtual' ? 'Videollamada' : 'Presencial'}\n\n` +
      `Si necesitas reprogramar, responde a este mensaje.`;
  }

  private formatPhone(phone: string): string {
    // Convert to WhatsApp format: 549XXXXXXXXXX@s.whatsapp.net
    const cleaned = phone.replace(/\D/g, '');
    return `549${cleaned}@s.whatsapp.net`;
  }

  async handleIncomingMessage(messageUpdate: any) {
    const message = messageUpdate.messages[0];
    if (!message.message || message.key.fromMe) return;

    const text = message.message.conversation?.toLowerCase() || '';
    const senderPhone = message.key.remoteJid;

    // Crisis keyword detection
    const crisisKeywords = ['crisis', 'emergencia', 'suicidio', 'ayuda urgente'];
    if (crisisKeywords.some(keyword => text.includes(keyword))) {
      await this.escalateCrisis(senderPhone, text);
    }
  }

  private async escalateCrisis(phone: string, message: string) {
    // Alert practitioner immediately
    console.log(`CRISIS ALERT from ${phone}: ${message}`);
    // Send notification to practitioner's emergency contact
  }
}
```

### 3. AFIP Electronic Invoicing

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { Afip } from '@afipsdk/afip.js';
import * as fs from 'fs';

@Injectable()
export class AfipService {
  private afip: any;

  constructor() {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: fs.readFileSync(process.env.AFIP_CERT_PATH),
      key: fs.readFileSync(process.env.AFIP_KEY_PATH),
      production: process.env.AFIP_ENVIRONMENT === 'production',
    });
  }

  async generateInvoice(data: {
    patientId: string;
    amount: number;
    sessionDate: Date;
    concept: string;
  }) {
    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(1, 11); // Tipo 11 = Factura C
    const nextNumber = lastInvoice + 1;

    const invoice = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: 11, // Factura C
      Concepto: 2, // Servicios
      DocTipo: 99, // Consumidor Final
      DocNro: 0,
      CbteDesde: nextNumber,
      CbteHasta: nextNumber,
      CbteFch: this.formatDate(data.sessionDate),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    };

    const result = await this.afip.ElectronicBilling.createVoucher(invoice);

    return {
      number: nextNumber,
      cae: result.CAE,
      caeDueDate: result.CAEFchVto,
      amount: data.amount,
      date: data.sessionDate,
      concept: data.concept,
    };
  }

  private formatDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  async getMonthlyResumen(month: number, year: number) {
    // Generate monthly tax summary for AFIP filing
    const startDate = new Date(year, month - 1, 1);
    const endDate = new Date(year, month, 0);

    // Query all invoices for the period
    const invoices = await this.prisma.invoice.findMany({
      where: {
        createdAt: { gte: startDate, lte: endDate },
      },
    });

    return {
      totalInvoiced: invoices.reduce((sum, inv) => sum + inv.amount, 0),
      invoiceCount: invoices.length,
      iibbWithholding: this.calculateIIBB(invoices),
    };
  }

  private calculateIIBB(invoices: any[]): number {
    // IIBB (Ingresos Brutos) calculation - province-specific
    const total = invoices.reduce((sum, inv) => sum + inv.amount, 0);
    const rate = 0.035; // Example rate for Buenos Aires
    return total * rate;
  }
}
```

### 4. AI Orchestration with Claude

```typescript
// backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(
    notes: string[],
    patientContext: any
  ): Promise<string> {
    // Use Opus for deep analysis
    const message = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `Eres un asistente especializado en psicología clínica argentina. 
          
Paciente: ${patientContext.firstName} ${patientContext.lastName}
Diagnóstico: ${patientContext.diagnosis || 'En evaluación'}
Sesiones previas: ${notes.length}

Notas de sesiones:
${notes.join('\n\n---\n\n')}

Por favor, genera un resumen clínico conciso que incluya:
1. Evolución del paciente
2. Temas recurrentes
3. Intervenciones efectivas
4. Recomendaciones para próximas sesiones

Formato: profesional, respetando confidencialidad y ética.`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async generateSessionReport(
    sessionData: any,
    quickMode: boolean = false
  ): Promise<string> {
    // Use Sonnet for real-time report generation
    const model = quickMode
      ? process.env.CLAUDE_SONNET_MODEL
      : process.env.CLAUDE_OPUS_MODEL;

    const message = await this.client.messages.create({
      model,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Genera un informe de sesión profesional basado en:

Fecha: ${new Date(sessionData.date).toLocaleDateString('es-AR')}
Duración: ${sessionData.duration} minutos
Observaciones del terapeuta: ${sessionData.notes}

El informe debe estar en formato estándar para obra social/prepaga argentina.`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async analyzePatientRisk(patientHistory: any): Promise<{
    riskLevel: 'low' | 'medium' | 'high';
    indicators: string[];
    recommendations: string[];
  }> {
    const message = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Analiza el historial del paciente y evalúa indicadores de riesgo:

${JSON.stringify(patientHistory, null, 2)}

Responde SOLO con JSON válido:
{
  "riskLevel": "low|medium|high",
  "indicators": ["lista de indicadores detectados"],
  "recommendations": ["recomendaciones clínicas"]
}`,
        },
      ],
    });

    const text = message.content[0].type === 'text' ? message.content[0].text : '{}';
    return JSON.parse(text);
  }
}
```

### 5. Video Consultation (LiveKit)

```typescript
// backend/src/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  async generateRoomToken(
    roomName: string,
    participantName: string,
    metadata: any
  ): Promise<string> {
    const token = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: participantName,
        metadata: JSON.stringify(metadata),
      }
    );

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
    });

    return token.toJwt();
  }

  async startSession(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    const roomName = `session-${appointmentId}`;

    const practitionerToken = await this.generateRoomToken(
      roomName,
      `practitioner-${appointment.practitioner.id}`,
      { role: 'practitioner' }
    );

    const patientToken = await this.generateRoomToken(
      roomName,
      `patient-${appointment.patient.id}`,
      { role: 'patient' }
    );

    return {
      roomName,
      wsUrl: process.env.LIVEKIT_WS_URL,
      practitionerToken,
      patientToken,
    };
  }
}
```

## Database Schema (Prisma)

```prisma
// backend/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Practitioner {
  id            String        @id @default(cuid())
  email         String        @unique
  name          String
  specialization String
  cuit          String        @unique
  afipCategory  String        // monotributo, responsable_inscripto
  appointments  Appointment[]
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt
}

model Patient {
  id           String        @id @default(cuid())
  firstName    String
  lastName     String
  phone        String
  email        String?
  diagnosis    String?
  appointments Appointment[]
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
}

model Appointment {
  id             String       @id @default(cuid())
  practitionerId String
  practitioner   Practitioner @relation(fields: [practitionerId], references: [id])
  patientId      String
  patient        Patient      @relation(fields: [patientId], references: [id])
  startTime      DateTime
  endTime        DateTime
  type           String       // presencial, virtual, evaluacion
  status         String       // scheduled, confirmed, completed, cancelled
  notes          String?
  videoRoomId    String?
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt

  @@index([practitionerId, startTime])
}

model Invoice {
  id            String   @id @default(cuid())
  number        Int
  cae           String
  caeDueDate    String
  amount        Float
  concept       String
  patientId     String
  appointmentId String?
  status        String   // pending, paid, cancelled
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  @@index([patientId])
}
```

## Common Patterns

### Patient Journey Pipeline

```typescript
// backend/src/patients/pipeline.service.ts
export class PatientPipelineService {
  async moveToStage(patientId: string, stage: string) {
    const patient = await this.prisma.patient.update({
      where: { id: patientId },
      data: { pipelineStage: stage },
    });

    // Trigger automation based on stage
    switch (stage) {
      case 'admision':
        await this.sendWelcomeMessage(patient);
        break;
      case 'sesion_activa':
        await this.scheduleFollowUp(patient);
        break;
      case 'finalizacion':
        await this.sendSatisfactionSurvey(patient);
        break;
    }

    return patient;
  }

  async detectAtRisk() {
    const twoWeeksAgo = new Date();
    twoWeeksAgo.setDate(twoWeeksAgo.getDate() - 14);

    const atRisk = await this.prisma.patient.findMany({
      where: {
        pipelineStage: 'sesion_activa',
        appointments: {
          none: {
            startTime: { gte: twoWeeksAgo },
          },
        },
      },
    });

    // Alert practitioners
    for (const patient of atRisk) {
      await this.notifyPractitioner(patient);
    }

    return atRisk;
  }
}
```

### Automated Reminder Scheduling

```typescript
// backend/src/whatsapp/reminder.scheduler.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
export class ReminderScheduler {
  constructor(
    private whatsapp: WhatsAppService,
    private prisma: PrismaService
  ) {}

  @Cron('0 */30 * * * *') // Every 30 minutes
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: tomorrow,
          lt: new Date(tomorrow.getTime() + 30 * 60000), // 30-min window
        },
        status: 'scheduled',
        reminder24hSent: false,
      },
      include: { patient: true, practitioner: true },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt);
      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminder24hSent: true },
      });
    }
  }
}
```

## Configuration

### Prisma Configuration

```typescript
// backend/src/prisma/prisma.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }

  async enableShutdownHooks(app: any) {
    this.$on('beforeExit', async () => {
      await app.close();
    });
  }
}
```

### API Client (Frontend)

```typescript
// frontend/src/lib/api/client.ts
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000',
  headers: {
    'Content-Type': 'application/json',
  },
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('auth_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('auth_token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

## Troubleshooting

### WhatsApp Connection Issues

```bash
# Clear session and re-authenticate
rm -rf ./whatsapp-sessions/*
# Restart backend - QR code will appear in terminal
pnpm run start:dev
```

### AFIP Certificate Errors

```bash
# Verify certificate permissions
chmod 600 ./certs/afip-cert.pem
chmod 600 ./certs/afip-key.pem

# Test AFIP connection
curl -X POST http://localhost:3000/billing/test-afip-connection
```

### Database Migration Failures

```bash
#
