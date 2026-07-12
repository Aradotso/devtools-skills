---
name: sesion-workflow-clinic
description: Mental health practice management platform with AI-powered scheduling, WhatsApp automation, AFIP billing, and video consultations for Argentine clinics
triggers:
  - how do I integrate Sesión into my psychology clinic
  - set up WhatsApp automation for patient appointments
  - configure AFIP electronic invoicing in Sesión
  - implement Claude AI for clinical note summarization
  - create appointment scheduling with Sesión
  - build video consultation feature with Sesión
  - configure Mercado Pago payment integration
  - set up patient journey pipeline in Sesión
---

# Sesión Workflow Clinic

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive mental health practice orchestration platform designed for psychology clinics in Argentina. It unifies appointment scheduling, automated WhatsApp messaging, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Tech Stack

- **Frontend**: SvelteKit 5 + Svelte 5, TailwindCSS
- **Backend**: NestJS, Prisma ORM
- **Database**: PostgreSQL (relational), Redis (caching), Elasticsearch (search)
- **AI**: Anthropic Claude Opus 4.6 & Sonnet 4.6
- **Video**: LiveKit (WebRTC)
- **Messaging**: Baileys (WhatsApp automation)
- **Payments**: Stripe, Mercado Pago
- **Language**: TypeScript

## Installation

### Prerequisites

```bash
# Required dependencies
node >= 18.0.0
postgresql >= 14
redis >= 6.2
docker >= 20.10 (optional but recommended)
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
npm install

# Setup environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# AI Configuration
ANTHROPIC_API_KEY=your_anthropic_key_here
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp Automation
WHATSAPP_SESSION_PATH="./whatsapp-session"
WHATSAPP_WEBHOOK_SECRET=your_webhook_secret

# Video Consultations
LIVEKIT_API_KEY=your_livekit_key
LIVEKIT_API_SECRET=your_livekit_secret
LIVEKIT_WS_URL="wss://your-livekit-instance.com"

# Payment Processing
STRIPE_SECRET_KEY=your_stripe_key
MERCADOPAGO_ACCESS_TOKEN=your_mercadopago_token

# AFIP Integration (Argentina Tax Authority)
AFIP_CUIT=your_cuit_number
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
```

### Database Setup

```bash
# Run Prisma migrations
npx prisma migrate dev

# Generate Prisma client
npx prisma generate

# Seed initial data
npm run db:seed
```

### Start Development Server

```bash
# Start backend (NestJS)
npm run dev:api

# Start frontend (SvelteKit)
npm run dev:web

# Start all services with Docker Compose
docker-compose up -d
```

## Core Modules

### 1. Appointment Scheduling

**Prisma Schema (appointments)**:

```typescript
// prisma/schema.prisma
model Appointment {
  id            String   @id @default(cuid())
  patientId     String
  practitionerId String
  startTime     DateTime
  endTime       DateTime
  sessionType   SessionType
  status        AppointmentStatus @default(SCHEDULED)
  roomId        String?
  notes         String?
  
  patient       Patient       @relation(fields: [patientId], references: [id])
  practitioner  Practitioner  @relation(fields: [practitionerId], references: [id])
  room          Room?         @relation(fields: [roomId], references: [id])
  
  @@index([patientId, startTime])
  @@index([practitionerId, startTime])
}

enum SessionType {
  PRESENCIAL
  VIRTUAL
  EVALUACION
  PAREJA
  FAMILIAR
}

enum AppointmentStatus {
  SCHEDULED
  CONFIRMED
  IN_PROGRESS
  COMPLETED
  CANCELLED
  NO_SHOW
}
```

**Creating Appointments (NestJS Service)**:

```typescript
// src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        status: { not: 'CANCELLED' },
        OR: [
          {
            AND: [
              { startTime: { lte: data.startTime } },
              { endTime: { gt: data.startTime } },
            ],
          },
          {
            AND: [
              { startTime: { lt: data.endTime } },
              { endTime: { gte: data.endTime } },
            ],
          },
        ],
      },
    });

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
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
    duration: number = 45,
  ) {
    const startOfDay = new Date(date);
    startOfDay.setHours(0, 0, 0, 0);
    const endOfDay = new Date(date);
    endOfDay.setHours(23, 59, 59, 999);

    const existingAppointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: startOfDay, lte: endOfDay },
        status: { not: 'CANCELLED' },
      },
      orderBy: { startTime: 'asc' },
    });

    // Algorithm to find available slots
    const workingHours = { start: 9, end: 20 }; // 9 AM to 8 PM
    const slots: Date[] = [];
    
    let currentTime = new Date(date);
    currentTime.setHours(workingHours.start, 0, 0, 0);
    
    const endTime = new Date(date);
    endTime.setHours(workingHours.end, 0, 0, 0);

    while (currentTime < endTime) {
      const slotEnd = new Date(currentTime.getTime() + duration * 60000);
      
      const hasConflict = existingAppointments.some(apt => {
        return currentTime < apt.endTime && slotEnd > apt.startTime;
      });

      if (!hasConflict) {
        slots.push(new Date(currentTime));
      }

      currentTime = new Date(currentTime.getTime() + 15 * 60000); // 15-min increments
    }

    return slots;
  }
}
```

**SvelteKit Appointment Scheduler**:

```svelte
<!-- src/routes/appointments/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import type { Appointment } from '$lib/types';
  
  let selectedDate = $state(new Date());
  let selectedPractitioner = $state('');
  let availableSlots = $state<Date[]>([]);
  let loading = $state(false);

  async function loadAvailableSlots() {
    loading = true;
    try {
      const response = await fetch('/api/appointments/available-slots', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          practitionerId: selectedPractitioner,
          date: selectedDate.toISOString(),
          duration: 45,
        }),
      });
      availableSlots = await response.json();
    } finally {
      loading = false;
    }
  }

  async function bookAppointment(slot: Date) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        practitionerId: selectedPractitioner,
        patientId: $page.data.currentPatient.id,
        startTime: slot,
        endTime: new Date(slot.getTime() + 45 * 60000),
        sessionType: 'VIRTUAL',
      }),
    });

    if (response.ok) {
      alert('Cita agendada correctamente');
      loadAvailableSlots();
    }
  }

  $effect(() => {
    if (selectedPractitioner && selectedDate) {
      loadAvailableSlots();
    }
  });
</script>

<div class="appointment-scheduler">
  <h2 class="text-2xl font-bold mb-4">Agendar Sesión</h2>
  
  <input
    type="date"
    bind:value={selectedDate}
    class="border rounded px-4 py-2 mb-4"
  />

  {#if loading}
    <p class="text-gray-500">Cargando horarios disponibles...</p>
  {:else if availableSlots.length === 0}
    <p class="text-gray-500">No hay horarios disponibles para esta fecha</p>
  {:else}
    <div class="grid grid-cols-4 gap-2">
      {#each availableSlots as slot}
        <button
          onclick={() => bookAppointment(slot)}
          class="border border-blue-500 text-blue-500 hover:bg-blue-500 hover:text-white rounded px-4 py-2 transition"
        >
          {slot.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}
        </button>
      {/each}
    </div>
  {/if}
</div>
```

### 2. WhatsApp Automation

**WhatsApp Service with Baileys**:

```typescript
// src/whatsapp/whatsapp.service.ts
import { Injectable } from '@nestjs/common';
import makeWASocket, {
  useMultiFileAuthState,
  DisconnectReason,
  MessageType,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService {
  private socket: any;
  private sessionPath = process.env.WHATSAPP_SESSION_PATH || './wa-session';

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(this.sessionPath);
    
    this.socket = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.socket.ev.on('creds.update', saveCreds);
    
    this.socket.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      
      if (connection === 'close') {
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.initialize(); // Reconnect
        }
      }
    });

    this.socket.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessages(messages);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    const patient = appointment.patient;
    const message = `Hola ${patient.firstName}! 👋\n\n` +
      `Tu sesión con ${appointment.practitioner.name} ha sido confirmada:\n\n` +
      `📅 Fecha: ${appointment.startTime.toLocaleDateString('es-AR')}\n` +
      `🕐 Hora: ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n` +
      `💻 Modalidad: ${appointment.sessionType}\n\n` +
      `Por favor responde *SI* para confirmar tu asistencia.`;

    await this.socket.sendMessage(
      `${patient.phoneNumber}@s.whatsapp.net`,
      { text: message },
    );
  }

  async sendReminder24Hours(appointment: any) {
    const patient = appointment.patient;
    const message = `Recordatorio: Tu sesión es mañana a las ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })} 📆`;

    await this.socket.sendMessage(
      `${patient.phoneNumber}@s.whatsapp.net`,
      { text: message },
    );
  }

  private async handleIncomingMessages(messages: any[]) {
    for (const msg of messages) {
      if (msg.key.fromMe) continue;

      const text = msg.message?.conversation || msg.message?.extendedTextMessage?.text;
      const from = msg.key.remoteJid;

      // Crisis keywords detection
      const crisisKeywords = ['suicidio', 'morir', 'acabar con todo', 'no puedo más'];
      if (crisisKeywords.some(keyword => text?.toLowerCase().includes(keyword))) {
        await this.escalateCrisis(from, text);
      }

      // Confirmation parsing
      if (text?.toLowerCase() === 'si' || text?.toLowerCase() === 'sí') {
        await this.handleConfirmation(from);
      }
    }
  }

  private async escalateCrisis(phoneNumber: string, message: string) {
    // Notify practitioner immediately
    // Log to crisis management system
    console.error(`CRISIS ALERT from ${phoneNumber}: ${message}`);
    // Trigger emergency notification workflow
  }

  private async handleConfirmation(phoneNumber: string) {
    // Update appointment status to CONFIRMED
    const phone = phoneNumber.replace('@s.whatsapp.net', '');
    await this.prisma.appointment.updateMany({
      where: {
        patient: { phoneNumber: phone },
        status: 'SCHEDULED',
        startTime: { gte: new Date() },
      },
      data: { status: 'CONFIRMED' },
    });
  }
}
```

**Automated Reminder Cron Job**:

```typescript
// src/appointments/appointments.cron.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsCronService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    tomorrow.setHours(0, 0, 0, 0);
    
    const dayAfterTomorrow = new Date(tomorrow);
    dayAfterTomorrow.setDate(dayAfterTomorrow.getDate() + 1);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: { gte: tomorrow, lt: dayAfterTomorrow },
        status: { in: ['SCHEDULED', 'CONFIRMED'] },
        reminderSent24h: false,
      },
      include: { patient: true, practitioner: true },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendReminder24Hours(apt);
      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent24h: true },
      });
    }
  }

  @Cron('0 */2 * * *') // Every 2 hours
  async send2HourReminders() {
    const now = new Date();
    const twoHoursLater = new Date(now.getTime() + 2 * 60 * 60 * 1000);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: { gte: now, lte: twoHoursLater },
        status: { in: ['SCHEDULED', 'CONFIRMED'] },
        reminderSent2h: false,
      },
      include: { patient: true },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendMessage(
        apt.patient.phoneNumber,
        `Tu sesión comienza en 2 horas. Prepara tu espacio y conexión 🧘‍♀️`,
      );
      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent2h: true },
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

**AFIP Integration Service**:

```typescript
// src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import Afip from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.NODE_ENV === 'production',
    });
  }

  async generateInvoice(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment) throw new Error('Appointment not found');

    const lastInvoice = await this.afip.ElectronicBilling.getLastVoucher(
      1, // Punto de venta
      11, // Tipo factura C
    );

    const invoiceData = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: 11, // Factura C
      Concepto: 2, // Servicios
      DocTipo: 96, // DNI
      DocNro: appointment.patient.dni || 0,
      CbteDesde: lastInvoice + 1,
      CbteHasta: lastInvoice + 1,
      CbteFch: this.formatDate(new Date()),
      ImpTotal: appointment.amount,
      ImpTotConc: 0,
      ImpNeto: appointment.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    };

    const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    // Store invoice in database
    const invoice = await this.prisma.invoice.create({
      data: {
        appointmentId: appointment.id,
        patientId: appointment.patientId,
        practitionerId: appointment.practitionerId,
        invoiceNumber: `${invoiceData.PtoVta}-${invoiceData.CbteHasta}`,
        cae: result.CAE,
        caeExpiration: new Date(result.CAEFchVto),
        amount: appointment.amount,
        invoiceType: 'C',
        status: 'ISSUED',
      },
    });

    return invoice;
  }

  private formatDate(date: Date): number {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return parseInt(`${year}${month}${day}`);
  }

  async getMonthlyReport(practitionerId: string, year: number, month: number) {
    const startDate = new Date(year, month - 1, 1);
    const endDate = new Date(year, month, 0, 23, 59, 59);

    const invoices = await this.prisma.invoice.findMany({
      where: {
        practitionerId,
        createdAt: { gte: startDate, lte: endDate },
      },
      include: { appointment: true },
    });

    const totalIncome = invoices.reduce((sum, inv) => sum + inv.amount, 0);
    const sessionCount = invoices.length;

    return {
      totalIncome,
      sessionCount,
      invoices,
      taxSummary: this.calculateTaxes(totalIncome, practitionerId),
    };
  }

  private calculateTaxes(income: number, practitionerId: string) {
    // Simplified tax calculation for Argentine practitioners
    // Real implementation would fetch fiscal category from practitioner profile
    return {
      monotributo: income * 0.05, // Example 5%
      iibb: income * 0.03, // Ingresos Brutos 3%
      ganancias: income * 0.15, // Ganancias 15%
    };
  }
}
```

### 4. AI Clinical Assistant (Claude Integration)

**AI Orchestration Service**:

```typescript
// src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private opusClient: Anthropic;
  private sonnetClient: Anthropic;

  constructor() {
    const apiKey = process.env.ANTHROPIC_API_KEY;
    this.opusClient = new Anthropic({ apiKey });
    this.sonnetClient = new Anthropic({ apiKey });
  }

  async summarizeClinicalNotes(sessionNotes: string, patientHistory?: string) {
    const prompt = `Eres un asistente clínico para psicólogos en Argentina. 
    
Resume las siguientes notas de sesión de manera profesional y estructurada:

${sessionNotes}

${patientHistory ? `Historial previo del paciente:\n${patientHistory}` : ''}

Proporciona un resumen que incluya:
1. Motivo de consulta principal
2. Observaciones clínicas relevantes
3. Intervenciones realizadas
4. Plan terapéutico sugerido
5. Próximos pasos

Usa terminología profesional argentina y mantén la confidencialidad.`;

    const message = await this.opusClient.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 2048,
      messages: [{ role: 'user', content: prompt }],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async generateSessionReport(appointmentId: string) {
    // Fetch appointment with patient history
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: {
          include: {
            appointments: {
              where: { status: 'COMPLETED' },
              orderBy: { startTime: 'desc' },
              take: 10,
              include: { clinicalNotes: true },
            },
          },
        },
      },
    });

    const historyContext = appointment.patient.appointments
      .map(apt => apt.clinicalNotes?.content)
      .join('\n\n---\n\n');

    return this.summarizeClinicalNotes(
      appointment.notes || '',
      historyContext,
    );
  }

  async interactiveAssistant(query: string, context?: any) {
    // Fast responses with Sonnet for real-time assistance
    const message = await this.sonnetClient.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4',
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Eres un asistente en tiempo real para psicólogos. Responde de manera concisa:\n\n${query}`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async predictOutcome(patientId: string) {
    // Use Opus for complex predictive analysis
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        appointments: {
          where: { status: 'COMPLETED' },
          include: { clinicalNotes: true },
        },
      },
    });

    const clinicalHistory = patient.appointments
      .map(apt => ({
        date: apt.startTime,
        notes: apt.clinicalNotes?.content,
        outcome: apt.clinicalNotes?.outcomeRating,
      }))
      .filter(apt => apt.notes);

    const prompt = `Analiza el siguiente historial clínico y proporciona un análisis predictivo del progreso terapéutico:

${JSON.stringify(clinicalHistory, null, 2)}

Considera:
1. Adherencia al tratamiento
2. Evolución de síntomas
3. Factores de riesgo de abandono
4. Áreas de mejora
5. Pronóstico terapéutico

Proporciona respuesta en formato JSON estructurado.`;

    const message = await this.opusClient.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 3072,
      messages: [{ role: 'user', content: prompt }],
    });

    const responseText = message.content[0].type === 'text' ? message.content[0].text : '{}';
    return JSON.parse(responseText);
  }
}
```

**SvelteKit AI Assistant Interface**:

```svelte
<!-- src/routes/ai-assistant/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  
  let query = $state('');
  let conversation = $state<Array<{ role: string; content: string }>>([]);
  let loading = $state(false);

  async function askAssistant() {
    if (!query.trim()) return;
    
    conversation.push({ role: 'user', content: query });
    loading = true;
    
    try {
      const response = await fetch('/api/ai/assistant', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ query }),
      });
      
      const data = await response.json();
      conversation.push({ role: 'assistant', content: data.response });
      query = '';
    } finally {
      loading = false;
    }
  }
</script>

<div class="ai-assistant max-w-4xl mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">Asistente Clínico IA</h2>
  
  <div class="conversation-history bg-gray-50 rounded-lg p-4 mb-4 h-96 overflow-y-auto">
    {#each conversation as msg}
      <div class="message mb-4 {msg.role === 'user' ? 'text-right' : 'text-left'}">
        <div class="inline-block max-w-xs {msg.role === 'user' ? 'bg-blue-500 text-white' : 'bg-gray-200'} rounded-lg px-4 py-2">
          {msg.content}
        </div>
      </div>
    {/each}
    
    {#if loading}
      <div class="text-gray-500 italic">Pensando...</div>
    {/if}
  </div>
  
  <form onsubmit|preventDefault={askAssistant} class="flex gap-2">
    <input
      bind:value={query}
      placeholder="Pregunta algo al asistente..."
      class="flex-1 border rounded-lg px-4 py-2"
      disabled={loading}
    />
    <button
      type="submit"
      disabled={loading || !query.trim()}
      class="bg-blue-500 text-white px-6 py-2 rounded-lg disabled:opacity-50"
    >
      Enviar
    </button>
  </form>
</div>
```

### 5. Video Consultations (LiveKit)

**LiveKit Room Service**:

```typescript
// src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  private apiKey = process.env.LIVEKIT_API_KEY;
  private apiSecret = process.env.LIVEKIT_API_SECRET;
  private wsUrl = process.env.LIVEKIT_WS_URL;

  async createRoomToken(roomName: string, participantName: string, metadata?: any) {
    const token = new AccessToken(this.apiKey, this.apiSecret, {
      identity: participantName,
      metadata: JSON.stringify(metadata),
    });

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
    });

    return {
      token: token.toJwt(),
      wsUrl: this.wsUrl,
    };
  }

  async createConsultationRoom(appointmentId: string) {
    const appointment = await this.pris
