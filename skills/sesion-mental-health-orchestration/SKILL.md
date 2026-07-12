---
name: sesion-mental-health-orchestration
description: SaaS platform for psychology clinics in Argentina with AI-powered scheduling, WhatsApp automation, AFIP billing, and video consultations
triggers:
  - how do I set up Sesión for a psychology clinic
  - integrate WhatsApp automation with Sesión
  - configure AFIP electronic invoicing in Sesión
  - implement Claude AI for clinical notes in Sesión
  - set up video consultations with Sesión
  - create appointment scheduling with Sesión workflow
  - configure Sesión for Argentine mental health practice
  - integrate Mercado Pago payments in Sesión
---

# Sesión Mental Health Orchestration Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It orchestrates appointment scheduling, automated WhatsApp messaging (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

**Tech Stack:**
- **Backend:** NestJS + TypeScript
- **Frontend:** SvelteKit 5 + Tailwind CSS
- **Database:** Prisma ORM (PostgreSQL)
- **AI:** Anthropic Claude (Opus & Sonnet)
- **WhatsApp:** Baileys library
- **Video:** LiveKit
- **Payments:** Stripe + Mercado Pago

## Installation & Setup

### Prerequisites

```bash
node >= 18.0.0
postgresql >= 14
redis >= 6
```

### Project Setup

```bash
# Clone repository
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=your_anthropic_api_key

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"

# LiveKit Video
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret
LIVEKIT_URL=wss://your-livekit-server.com

# AFIP Argentina
AFIP_CUIT=your_clinic_cuit
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.key"

# Payments
STRIPE_SECRET_KEY=your_stripe_secret_key
MERCADOPAGO_ACCESS_TOKEN=your_mercadopago_token

# App
JWT_SECRET=your_jwt_secret
APP_URL=http://localhost:5173
API_URL=http://localhost:3000
```

### Database Migration

```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate dev

# Seed initial data
npx prisma db seed
```

### Start Development Servers

```bash
# Start backend (NestJS)
npm run start:dev

# Start frontend (SvelteKit) in separate terminal
cd frontend
npm run dev
```

## Core Architecture

### Prisma Schema Structure

```prisma
// prisma/schema.prisma
model Patient {
  id            String   @id @default(cuid())
  name          String
  email         String   @unique
  phone         String
  whatsappOptIn Boolean  @default(true)
  createdAt     DateTime @default(now())
  
  appointments  Appointment[]
  invoices      Invoice[]
  clinicalNotes ClinicalNote[]
}

model Appointment {
  id          String   @id @default(cuid())
  patientId   String
  practitionerId String
  startTime   DateTime
  endTime     DateTime
  type        AppointmentType // PRESENCIAL, VIRTUAL, EVALUACION
  status      AppointmentStatus // SCHEDULED, CONFIRMED, CANCELLED, COMPLETED
  
  patient     Patient  @relation(fields: [patientId], references: [id])
  practitioner Practitioner @relation(fields: [practitionerId], references: [id])
  videoSession VideoSession?
}

model Invoice {
  id              String   @id @default(cuid())
  patientId       String
  appointmentId   String
  afipReceiptType String   // A, B, C, M
  amount          Decimal
  status          InvoiceStatus
  afipCAE         String?  // AFIP authorization code
  afipCAEExpiry   DateTime?
  
  patient         Patient  @relation(fields: [patientId], references: [id])
}
```

## NestJS Backend Services

### Appointment Scheduling Service

```typescript
// src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';
import { AppointmentType, AppointmentStatus } from '@prisma/client';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    type: AppointmentType;
  }) {
    // Check for conflicts
    const conflicts = await this.checkConflicts(
      data.practitionerId,
      data.startTime,
    );
    
    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime: new Date(data.startTime.getTime() + 45 * 60000), // 45 min
        status: AppointmentStatus.SCHEDULED,
      },
      include: {
        patient: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(appointment);

    return appointment;
  }

  async checkConflicts(practitionerId: string, startTime: Date) {
    const endTime = new Date(startTime.getTime() + 45 * 60000);
    
    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: {
          in: [AppointmentStatus.SCHEDULED, AppointmentStatus.CONFIRMED],
        },
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

  async scheduleReminders(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true },
    });

    // Schedule 24h reminder
    const reminder24h = new Date(appointment.startTime.getTime() - 24 * 60 * 60000);
    // Schedule 2h reminder
    const reminder2h = new Date(appointment.startTime.getTime() - 2 * 60 * 60000);

    // Use job queue for scheduling (e.g., Bull)
    await this.scheduleWhatsAppReminder(appointment, reminder24h);
    await this.scheduleWhatsAppReminder(appointment, reminder2h);
  }
}
```

### WhatsApp Automation with Baileys

```typescript
// src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, { DisconnectReason, useMultiFileAuthState } from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

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
          (lastDisconnect.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.connectToWhatsApp();
        }
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    if (!appointment.patient.whatsappOptIn) return;

    const message = `Hola ${appointment.patient.name},

Tu sesión ha sido agendada para:
📅 ${this.formatDate(appointment.startTime)}
⏰ ${this.formatTime(appointment.startTime)}
${appointment.type === 'VIRTUAL' ? '💻 Sesión virtual' : '🏥 Sesión presencial'}

Por favor confirmá tu asistencia respondiendo con SÍ.`;

    await this.sendMessage(appointment.patient.phone, message);
  }

  async sendMessage(phoneNumber: string, text: string) {
    const formattedPhone = `${phoneNumber}@s.whatsapp.net`;
    await this.sock.sendMessage(formattedPhone, { text });
  }

  async handleIncomingMessage(message: any) {
    const text = message.message?.conversation?.toLowerCase();
    const phone = message.key.remoteJid.replace('@s.whatsapp.net', '');

    // Crisis detection
    const crisisKeywords = ['crisis', 'suicidio', 'emergencia', 'ayuda urgente'];
    if (crisisKeywords.some(keyword => text?.includes(keyword))) {
      await this.escalateCrisis(phone, text);
      return;
    }

    // Appointment confirmation
    if (text === 'sí' || text === 'si') {
      await this.confirmAppointment(phone);
    }
  }

  async escalateCrisis(phone: string, message: string) {
    // Find patient
    const patient = await this.prisma.patient.findFirst({
      where: { phone },
      include: { appointments: { include: { practitioner: true } } },
    });

    // Alert practitioner immediately
    // Implementation depends on notification system
  }

  private formatDate(date: Date): string {
    return date.toLocaleDateString('es-AR', {
      weekday: 'long',
      day: 'numeric',
      month: 'long',
    });
  }

  private formatTime(date: Date): string {
    return date.toLocaleTimeString('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
    });
  }
}
```

### AFIP Electronic Invoicing

```typescript
// src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { AfipWebService } from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';
import * as fs from 'fs';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new AfipWebService({
      CUIT: process.env.AFIP_CUIT,
      cert: fs.readFileSync(process.env.AFIP_CERT_PATH),
      key: fs.readFileSync(process.env.AFIP_KEY_PATH),
      production: process.env.NODE_ENV === 'production',
    });
  }

  async generateInvoice(data: {
    patientId: string;
    appointmentId: string;
    amount: number;
    receiptType: 'A' | 'B' | 'C' | 'M';
  }) {
    const patient = await this.prisma.patient.findUnique({
      where: { id: data.patientId },
    });

    // Get last invoice number
    const lastInvoice = await this.prisma.invoice.findFirst({
      where: { afipReceiptType: data.receiptType },
      orderBy: { createdAt: 'desc' },
    });

    const invoiceNumber = lastInvoice ? lastInvoice.id + 1 : 1;

    // Request CAE from AFIP
    const caeResponse = await this.afip.ElectronicBilling.createVoucher({
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: this.getReceiptTypeCode(data.receiptType),
      Concepto: 2, // Services
      DocTipo: 80, // CUIT
      DocNro: patient.taxId || 0,
      CbteDesde: invoiceNumber,
      CbteHasta: invoiceNumber,
      CbteFch: this.formatAfipDate(new Date()),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    });

    // Save invoice with CAE
    const invoice = await this.prisma.invoice.create({
      data: {
        patientId: data.patientId,
        appointmentId: data.appointmentId,
        afipReceiptType: data.receiptType,
        amount: data.amount,
        status: 'APPROVED',
        afipCAE: caeResponse.CAE,
        afipCAEExpiry: this.parseAfipDate(caeResponse.CAEFchVto),
      },
    });

    return invoice;
  }

  private getReceiptTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type];
  }

  private formatAfipDate(date: Date): string {
    return date.toISOString().slice(0, 10).replace(/-/g, '');
  }

  private parseAfipDate(dateString: string): Date {
    return new Date(
      `${dateString.slice(0, 4)}-${dateString.slice(4, 6)}-${dateString.slice(6, 8)}`,
    );
  }
}
```

### Claude AI Integration

```typescript
// src/ai/claude.service.ts
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

  async generateClinicalSummary(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: {
          include: {
            clinicalNotes: {
              orderBy: { createdAt: 'desc' },
              take: 10,
            },
          },
        },
      },
    });

    const context = appointment.patient.clinicalNotes
      .map((note) => `[${note.createdAt.toISOString()}]\n${note.content}`)
      .join('\n\n---\n\n');

    const message = await this.anthropic.messages.create({
      model: 'claude-opus-4-20250514',
      max_tokens: 2000,
      messages: [
        {
          role: 'user',
          content: `Eres un asistente clínico especializado en psicología. Resume las últimas 10 sesiones del paciente ${appointment.patient.name}, destacando patrones, progreso terapéutico y áreas de atención.

Contexto de sesiones previas:
${context}

Genera un resumen clínico estructurado en español argentino, respetando la confidencialidad y ética profesional.`,
        },
      ],
    });

    return message.content[0].text;
  }

  async assistNoteGeneration(rawNotes: string, sessionType: string) {
    const message = await this.anthropic.messages.create({
      model: 'claude-sonnet-4-20250514', // Faster for real-time
      max_tokens: 1500,
      messages: [
        {
          role: 'user',
          content: `Eres un asistente para psicólogos en Argentina. El profesional dictó las siguientes notas durante una sesión de ${sessionType}:

"${rawNotes}"

Estructura estas notas en formato profesional con las siguientes secciones:
- Motivo de consulta
- Observaciones clínicas
- Intervenciones realizadas
- Plan terapéutico
- Próximos pasos

Mantén el tono profesional y utiliza terminología clínica apropiada.`,
        },
      ],
    });

    return message.content[0].text;
  }

  async detectRiskFactors(patientHistory: string) {
    const message = await this.anthropic.messages.create({
      model: 'claude-opus-4-20250514',
      max_tokens: 1000,
      system: `Eres un sistema de análisis clínico que identifica factores de riesgo en pacientes psicológicos. Evalúa el historial y reporta SOLO si detectas indicadores de:
- Riesgo suicida
- Autolesión
- Riesgo a terceros
- Deterioro funcional severo

Responde en JSON: {"risk_detected": boolean, "factors": [], "urgency": "low|medium|high"}`,
      messages: [
        {
          role: 'user',
          content: patientHistory,
        },
      ],
    });

    return JSON.parse(message.content[0].text);
  }
}
```

### LiveKit Video Service

```typescript
// src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class LiveKitService {
  constructor(private prisma: PrismaService) {}

  async createVideoSession(appointmentId: string, userId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment) {
      throw new Error('Appointment not found');
    }

    // Create video session record
    const videoSession = await this.prisma.videoSession.create({
      data: {
        appointmentId,
        roomName: `session-${appointmentId}`,
        status: 'ACTIVE',
      },
    });

    return videoSession;
  }

  generateToken(roomName: string, participantName: string, isPractitioner: boolean) {
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
      canPublishData: isPractitioner, // Only practitioner can share screen
    });

    return at.toJwt();
  }

  async endVideoSession(sessionId: string) {
    await this.prisma.videoSession.update({
      where: { id: sessionId },
      data: {
        status: 'COMPLETED',
        endedAt: new Date(),
      },
    });
  }
}
```

## SvelteKit Frontend Components

### Appointment Scheduler Component

```svelte
<!-- frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Calendar } from '$lib/components/calendar';
  import type { Appointment } from '$lib/types';

  let appointments: Appointment[] = $state([]);
  let selectedDate: Date = $state(new Date());
  let loading = $state(false);

  onMount(async () => {
    await loadAppointments();
  });

  async function loadAppointments() {
    loading = true;
    const response = await fetch('/api/appointments', {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
      },
    });
    appointments = await response.json();
    loading = false;
  }

  async function createAppointment(data: {
    patientId: string;
    startTime: Date;
    type: string;
  }) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
      },
      body: JSON.stringify(data),
    });

    if (response.ok) {
      await loadAppointments();
    } else {
      const error = await response.json();
      alert(`Error: ${error.message}`);
    }
  }
</script>

<div class="agenda-container">
  <h1>Agenda Inteligente</h1>
  
  <Calendar
    appointments={appointments}
    selectedDate={selectedDate}
    onCreate={createAppointment}
    {loading}
  />
</div>

<style>
  .agenda-container {
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }
</style>
```

### AI Clinical Notes Assistant

```svelte
<!-- frontend/src/routes/session/[id]/notes/+page.svelte -->
<script lang="ts">
  import { page } from '$app/stores';
  import { Button } from '$lib/components/ui';

  let rawNotes = $state('');
  let structuredNotes = $state('');
  let generating = $state(false);

  async function generateStructuredNotes() {
    generating = true;
    
    const response = await fetch('/api/ai/structure-notes', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
      },
      body: JSON.stringify({
        appointmentId: $page.params.id,
        rawNotes,
        sessionType: 'individual',
      }),
    });

    const data = await response.json();
    structuredNotes = data.structuredNotes;
    generating = false;
  }

  async function saveNotes() {
    await fetch(`/api/appointments/${$page.params.id}/notes`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
      },
      body: JSON.stringify({ content: structuredNotes }),
    });
  }
</script>

<div class="notes-editor">
  <h2>Notas de Sesión</h2>
  
  <div class="editor-section">
    <label for="raw-notes">Notas dictadas:</label>
    <textarea
      id="raw-notes"
      bind:value={rawNotes}
      placeholder="Paciente reporta mejoría en ansiedad..."
      rows="8"
    />
    
    <Button onclick={generateStructuredNotes} disabled={generating}>
      {generating ? 'Generando...' : '✨ Estructurar con IA'}
    </Button>
  </div>

  {#if structuredNotes}
    <div class="editor-section">
      <label for="structured-notes">Notas estructuradas:</label>
      <textarea
        id="structured-notes"
        bind:value={structuredNotes}
        rows="12"
      />
      
      <Button onclick={saveNotes}>Guardar Notas</Button>
    </div>
  {/if}
</div>

<style>
  .notes-editor {
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem;
  }

  .editor-section {
    margin-bottom: 2rem;
  }

  textarea {
    width: 100%;
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 8px;
    font-family: inherit;
  }
</style>
```

## API Endpoints

### REST API Reference

```typescript
// Key endpoints

// Authentication
POST /api/auth/login
POST /api/auth/register
POST /api/auth/refresh

// Appointments
GET    /api/appointments
POST   /api/appointments
PATCH  /api/appointments/:id
DELETE /api/appointments/:id
GET    /api/appointments/:id/conflicts

// Patients
GET    /api/patients
POST   /api/patients
GET    /api/patients/:id
PATCH  /api/patients/:id
GET    /api/patients/:id/history

// WhatsApp
POST   /api/whatsapp/send
GET    /api/whatsapp/messages/:patientId
POST   /api/whatsapp/opt-out/:patientId

// Billing
POST   /api/invoices
GET    /api/invoices
GET    /api/invoices/:id/pdf
POST   /api/invoices/:id/send-whatsapp

// Video
POST   /api/video/sessions
GET    /api/video/sessions/:id/token
POST   /api/video/sessions/:id/end

// AI
POST   /api/ai/structure-notes
POST   /api/ai/generate-summary
POST   /api/ai/detect-risk
```

### API Client Usage

```typescript
// frontend/src/lib/api/client.ts
export class SesionAPIClient {
  private baseURL: string;
  private token: string | null;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.token = localStorage.getItem('token');
  }

  async request(endpoint: string, options: RequestInit = {}) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { 'Authorization': `Bearer ${this.token}` }),
        ...options.headers,
      },
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }

    return response.json();
  }

  // Appointments
  async createAppointment(data: CreateAppointmentDTO) {
    return this.request('/api/appointments', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  async getAppointments(filters?: { startDate?: Date; endDate?: Date }) {
    const params = new URLSearchParams();
    if (filters?.startDate) params.set('startDate', filters.startDate.toISOString());
    if (filters?.endDate) params.set('endDate', filters.endDate.toISOString());
    
    return this.request(`/api/appointments?${params}`);
  }

  // AI
  async structureNotes(appointmentId: string, rawNotes: string) {
    return this.request('/api/ai/structure-notes', {
      method: 'POST',
      body: JSON.stringify({ appointmentId, rawNotes }),
    });
  }

  // Video
  async createVideoSession(appointmentId: string) {
    return this.request('/api/video/sessions', {
      method: 'POST',
      body: JSON.stringify({ appointmentId }),
    });
  }

  async getVideoToken(sessionId: string, participantName: string) {
    return this.request(`/api/video/sessions/${sessionId}/token`, {
      method: 'POST',
      body: JSON.stringify({ participantName }),
    });
  }
}
```

## Common Patterns

### Multi-Tenant Architecture

```typescript
// src/common/decorators/tenant.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentTenant = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.tenant; // Set by TenantMiddleware
  },
);

// Usage in controller
@Get('appointments')
async getAppointments(@CurrentTenant() tenant: Tenant) {
  return this.appointmentsService.findAll(tenant.id);
}
```

### Event-Driven Workflows

```typescript
// src/appointments/appointment-created.listener.ts
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { AppointmentCreatedEvent } from './events/appointment-created.event';

@Injectable()
export class AppointmentCreatedListener {
  constructor(
    private whatsapp: WhatsAppService,
    private calendar: CalendarService,
  ) {}

  @OnEvent('appointment.created')
  async handleAppointmentCreated(event: AppointmentCreatedEvent) {
    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(event.appointment);
    
    // Sync to Google Calendar
    await this.calendar.syncAppointment(event.appointment);
    
    // Schedule reminders
    await this.scheduleReminders(event.appointment);
  }
}
```

### Caching Strategy

```typescript
// src/patients/patients.service.ts
import { Injectable } from '@nestjs/common';
import { Cache } from '@nestjs/cache-manager';

@Injectable()
export class PatientsService {
  constructor(
    private prisma: PrismaService,
    private cacheManager: Cache,
  ) {}

  async getPatient(id: string) {
    const cacheKey = `patient:${id}`;
    const cached = await this.cacheManager.get(cacheKey);
    
    if (cached) return cached;

    const patient = await
