---
name: psique-workflow-clinic
description: Sesión - Mental health practice management platform with AI orchestration, WhatsApp automation, AFIP billing, and video consultations for Argentine psychology clinics
triggers:
  - how do I set up Sesión for a psychology clinic
  - configure WhatsApp automation for patient reminders
  - integrate AFIP electronic invoicing in Sesión
  - set up Claude AI for clinical note summarization
  - implement video consultation workflow
  - configure patient journey pipeline stages
  - automate appointment scheduling with conflict detection
  - integrate Mercado Pago payment processing
---

# Sesión Workflow Clinic

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that unifies appointment scheduling, WhatsApp automation, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Architecture Overview

Sesión uses a microservices architecture with:

- **Frontend**: SvelteKit 5 with TypeScript and TailwindCSS
- **Backend**: NestJS microservices
- **Database**: PostgreSQL with Prisma ORM
- **Cache**: Redis for sessions and real-time presence
- **Search**: Elasticsearch for patient records
- **Storage**: MinIO for encrypted documents
- **Messaging**: Apache Kafka for event-driven workflows
- **WhatsApp**: Baileys library for WhatsApp Business integration
- **Video**: LiveKit for WebRTC video consultations
- **AI**: Anthropic Claude Opus 4.6 and Sonnet 4.6
- **Payments**: Stripe and Mercado Pago integration

## Installation

### Prerequisites

```bash
# Required versions
node --version  # v20+
npm --version   # v10+
docker --version # v24+
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic
npm install
```

### Environment Configuration

Create `.env` file in project root:

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="sk-ant-..."
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your-webhook-secret"

# LiveKit Video
LIVEKIT_API_KEY="your-livekit-api-key"
LIVEKIT_API_SECRET="your-livekit-api-secret"
LIVEKIT_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="20123456789"
AFIP_CERTIFICATE_PATH="./certificates/afip-cert.pem"
AFIP_PRIVATE_KEY_PATH="./certificates/afip-key.pem"
AFIP_ENVIRONMENT="production" # or "homologacion" for testing

# Payment Gateways
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."
STRIPE_SECRET_KEY="sk_live_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# Application
JWT_SECRET="your-jwt-secret"
FRONTEND_URL="https://app.sesion.com.ar"
BACKEND_URL="https://api.sesion.com.ar"
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

### Start Services

```bash
# Development mode with hot reload
npm run dev

# Production build
npm run build
npm run start

# Docker Compose (all services)
docker-compose up -d
```

## Core Module Usage

### 1. Appointment Scheduling Engine

**Schema Definition** (Prisma):

```prisma
model Appointment {
  id            String   @id @default(cuid())
  patientId     String
  practitionerId String
  sessionType   SessionType
  startTime     DateTime
  endTime       DateTime
  status        AppointmentStatus
  roomId        String?
  isVirtual     Boolean  @default(false)
  notes         String?
  
  patient       Patient      @relation(fields: [patientId], references: [id])
  practitioner  Practitioner @relation(fields: [practitionerId], references: [id])
  room          Room?        @relation(fields: [roomId], references: [id])
  
  @@index([practitionerId, startTime])
  @@index([patientId])
}

enum SessionType {
  INDIVIDUAL
  COUPLES
  FAMILY
  EVALUATION
  FOLLOWUP
}

enum AppointmentStatus {
  SCHEDULED
  CONFIRMED
  CANCELLED
  COMPLETED
  NO_SHOW
}
```

**Creating Appointments with Conflict Detection**:

```typescript
// services/appointment.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from './prisma.service';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class AppointmentService {
  constructor(
    private prisma: PrismaService,
    private eventEmitter: EventEmitter2
  ) {}

  async createAppointment(data: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflicts = await this.findConflicts(
      data.practitionerId,
      data.startTime,
      data.endTime,
      data.roomId
    );

    if (conflicts.length > 0) {
      throw new ConflictException({
        message: 'Scheduling conflict detected',
        conflicts: conflicts,
        suggestions: await this.suggestAlternatives(data)
      });
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: data.patientId,
        practitionerId: data.practitionerId,
        sessionType: data.sessionType,
        startTime: data.startTime,
        endTime: data.endTime,
        isVirtual: data.isVirtual,
        roomId: data.roomId,
        status: 'SCHEDULED'
      },
      include: {
        patient: true,
        practitioner: true
      }
    });

    // Emit event for WhatsApp automation
    this.eventEmitter.emit('appointment.created', appointment);

    return appointment;
  }

  private async findConflicts(
    practitionerId: string,
    startTime: Date,
    endTime: Date,
    roomId?: string
  ) {
    return this.prisma.appointment.findMany({
      where: {
        AND: [
          {
            OR: [
              { practitionerId },
              { roomId: roomId || undefined }
            ]
          },
          {
            status: { in: ['SCHEDULED', 'CONFIRMED'] }
          },
          {
            OR: [
              {
                AND: [
                  { startTime: { lte: startTime } },
                  { endTime: { gt: startTime } }
                ]
              },
              {
                AND: [
                  { startTime: { lt: endTime } },
                  { endTime: { gte: endTime } }
                ]
              },
              {
                AND: [
                  { startTime: { gte: startTime } },
                  { endTime: { lte: endTime } }
                ]
              }
            ]
          }
        ]
      }
    });
  }

  private async suggestAlternatives(data: CreateAppointmentDto) {
    const duration = data.endTime.getTime() - data.startTime.getTime();
    const searchStart = new Date(data.startTime);
    searchStart.setDate(searchStart.getDate() - 1);
    const searchEnd = new Date(data.startTime);
    searchEnd.setDate(searchEnd.getDate() + 7);

    // Find free slots in the next week
    const allSlots = await this.generateTimeSlots(
      data.practitionerId,
      searchStart,
      searchEnd,
      duration
    );

    return allSlots.slice(0, 5);
  }
}
```

### 2. WhatsApp Automation with Baileys

**WhatsApp Service Implementation**:

```typescript
// services/whatsapp.service.ts
import makeWASocket, { 
  DisconnectReason, 
  useMultiFileAuthState,
  WAMessage 
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class WhatsAppService {
  private sock: any;
  private qrCode: string;

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true
    });

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect, qr } = update;
      
      if (qr) {
        this.qrCode = qr;
      }

      if (connection === 'close') {
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output
          ?.statusCode !== DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.initialize();
        }
      }
    });

    this.sock.ev.on('creds.update', saveCreds);
    this.sock.ev.on('messages.upsert', this.handleIncomingMessage.bind(this));
  }

  @OnEvent('appointment.created')
  async sendAppointmentConfirmation(appointment: any) {
    const patient = appointment.patient;
    const practitioner = appointment.practitioner;
    
    const message = this.formatConfirmationMessage(appointment);
    
    await this.sendMessage(patient.phoneNumber, message, {
      buttons: [
        { buttonId: `confirm:${appointment.id}`, buttonText: { displayText: 'Confirmar ✅' } },
        { buttonId: `cancel:${appointment.id}`, buttonText: { displayText: 'Cancelar ❌' } }
      ]
    });
  }

  @OnEvent('appointment.reminder.24h')
  async send24HourReminder(appointment: any) {
    const message = `
Hola ${appointment.patient.firstName}! 👋

Recordatorio: tenés turno mañana a las ${this.formatTime(appointment.startTime)} con ${appointment.practitioner.fullName}.

${appointment.isVirtual ? '📹 Sesión virtual - El link se enviará 15 minutos antes.' : '📍 Dirección: ' + appointment.practitioner.address}

¿Confirmás tu asistencia?
    `.trim();

    await this.sendMessage(appointment.patient.phoneNumber, message);
  }

  private async sendMessage(
    phoneNumber: string, 
    text: string, 
    options?: any
  ) {
    const jid = `${phoneNumber}@s.whatsapp.net`;
    
    if (options?.buttons) {
      await this.sock.sendMessage(jid, {
        text,
        footer: 'Sesión - Tu agenda inteligente',
        buttons: options.buttons,
        headerType: 1
      });
    } else {
      await this.sock.sendMessage(jid, { text });
    }
  }

  private async handleIncomingMessage(m: { messages: WAMessage[] }) {
    const message = m.messages[0];
    
    if (!message.message) return;
    
    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text || '';
    
    // Crisis detection keywords
    const crisisKeywords = [
      'suicidio', 'quitarme la vida', 'no puedo más',
      'voy a hacerme daño', 'crisis', 'emergencia'
    ];
    
    const isCrisis = crisisKeywords.some(keyword => 
      text.toLowerCase().includes(keyword)
    );
    
    if (isCrisis) {
      await this.escalateCrisis(message);
    }
    
    // Button response handling
    if (message.message.buttonsResponseMessage) {
      const buttonId = message.message.buttonsResponseMessage.selectedButtonId;
      await this.handleButtonResponse(buttonId, message);
    }
  }

  private formatTime(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false
    }).format(date);
  }
}
```

### 3. AFIP Electronic Invoicing

**Invoice Generation Service**:

```typescript
// services/afip-billing.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import { Afip } from '@afipsdk/afip.js';

@Injectable()
export class AfipBillingService {
  private afip: Afip;

  constructor() {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: readFileSync(process.env.AFIP_CERTIFICATE_PATH, 'utf8'),
      key: readFileSync(process.env.AFIP_PRIVATE_KEY_PATH, 'utf8'),
      production: process.env.AFIP_ENVIRONMENT === 'production'
    });
  }

  async generateInvoice(sessionData: GenerateInvoiceDto) {
    // Determine invoice type based on practitioner's fiscal category
    const invoiceType = this.determineInvoiceType(sessionData.practitionerFiscalCategory);
    
    const lastVoucher = await this.afip.ElectronicBilling.getLastVoucher(
      sessionData.salePoint,
      invoiceType
    );

    const invoiceNumber = lastVoucher + 1;

    const invoiceData = {
      'CantReg': 1,
      'PtoVta': sessionData.salePoint,
      'CbteTipo': invoiceType,
      'Concepto': 2, // Services
      'DocTipo': 96, // DNI
      'DocNro': sessionData.patientDni,
      'CbteDesde': invoiceNumber,
      'CbteHasta': invoiceNumber,
      'CbteFch': this.formatAfipDate(new Date()),
      'FchServDesde': this.formatAfipDate(sessionData.sessionDate),
      'FchServHasta': this.formatAfipDate(sessionData.sessionDate),
      'FchVtoPago': this.formatAfipDate(sessionData.dueDate),
      'ImpTotal': sessionData.amount,
      'ImpTotConc': 0,
      'ImpNeto': sessionData.amount,
      'ImpOpEx': 0,
      'ImpIVA': 0,
      'ImpTrib': 0,
      'MonId': 'PES',
      'MonCotiz': 1
    };

    const result = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    // Store invoice in database
    const invoice = await this.prisma.invoice.create({
      data: {
        cae: result.CAE,
        caeExpirationDate: result.CAEFchVto,
        invoiceNumber: invoiceNumber,
        invoiceType: invoiceType,
        salePoint: sessionData.salePoint,
        amount: sessionData.amount,
        patientId: sessionData.patientId,
        sessionId: sessionData.sessionId,
        afipResponse: result
      }
    });

    // Emit event to send invoice via WhatsApp
    this.eventEmitter.emit('invoice.generated', invoice);

    return invoice;
  }

  private determineInvoiceType(fiscalCategory: string): number {
    // 1: Factura A, 6: Factura B, 11: Factura C, 51: Factura M
    const mapping = {
      'RESPONSABLE_INSCRIPTO': 1,
      'MONOTRIBUTO': 11,
      'EXENTO': 6
    };
    return mapping[fiscalCategory] || 11;
  }

  private formatAfipDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  async getInvoicePDF(invoiceId: string): Promise<Buffer> {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id: invoiceId },
      include: { patient: true, session: true }
    });

    // Generate PDF using template
    const pdf = await this.generatePDFFromTemplate(invoice);
    return pdf;
  }
}
```

### 4. Claude AI Integration

**AI Orchestration Service**:

```typescript
// services/claude-orchestration.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeOrchestrationService {
  private anthropic: Anthropic;
  private opusModel = process.env.CLAUDE_OPUS_MODEL;
  private sonnetModel = process.env.CLAUDE_SONNET_MODEL;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }

  async summarizeClinicalNotes(sessionNotes: string, patientHistory?: string) {
    // Use Opus for deep cognitive workload
    const prompt = `
Sos un psicólogo clínico argentino especializado en resumir notas de sesión.

${patientHistory ? `Contexto del paciente:\n${patientHistory}\n\n` : ''}

Notas de la sesión actual:
${sessionNotes}

Generá un resumen estructurado siguiendo este formato:
1. Presentación del paciente
2. Temas principales abordados
3. Intervenciones realizadas
4. Evolución observada
5. Plan terapéutico para próxima sesión

Usá lenguaje técnico apropiado pero claro.
    `.trim();

    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 2048,
      temperature: 0.3,
      system: 'Sos un asistente especializado en psicología clínica en Argentina.',
      messages: [
        { role: 'user', content: prompt }
      ]
    });

    return message.content[0].text;
  }

  async interactiveAssistant(query: string, context: any) {
    // Use Sonnet for real-time interaction
    const systemPrompt = `
Sos un asistente inteligente para psicólogos argentinos usando la plataforma Sesión.
Ayudás con:
- Búsqueda rápida de información de pacientes
- Generación de plantillas de notas clínicas
- Recordatorios de próximos turnos
- Sugerencias de intervenciones basadas en el historial

Respondé de manera concisa y práctica.
    `.trim();

    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      temperature: 0.7,
      system: systemPrompt,
      messages: [
        { role: 'user', content: this.formatContextualQuery(query, context) }
      ]
    });

    // Log usage for cost attribution
    await this.logAIUsage({
      model: this.sonnetModel,
      practitionerId: context.practitionerId,
      inputTokens: message.usage.input_tokens,
      outputTokens: message.usage.output_tokens,
      cost: this.calculateCost(message.usage)
    });

    return message.content[0].text;
  }

  async predictTherapeuticOutcome(patientId: string) {
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        sessions: {
          orderBy: { date: 'asc' },
          include: { notes: true }
        },
        assessments: true
      }
    });

    const sessionHistory = patient.sessions
      .map(s => `Sesión ${s.sessionNumber}: ${s.notes?.summary}`)
      .join('\n\n');

    const prompt = `
Analizá el siguiente historial de tratamiento de un paciente y proporcioná:
1. Indicadores de progreso terapéutico
2. Factores de riesgo de deserción
3. Recomendaciones para fortalecer la alianza terapéutica

Historial:
${sessionHistory}

Evaluaciones psicológicas:
${JSON.stringify(patient.assessments, null, 2)}
    `.trim();

    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 3072,
      temperature: 0.2,
      messages: [
        { role: 'user', content: prompt }
      ]
    });

    return {
      analysis: message.content[0].text,
      confidence: this.extractConfidenceScore(message.content[0].text)
    };
  }

  private formatContextualQuery(query: string, context: any): string {
    return `
Contexto:
- Psicólogo: ${context.practitionerName}
- Pacientes activos: ${context.activePatients}
- Próximo turno: ${context.nextAppointment}

Consulta: ${query}
    `.trim();
  }

  private calculateCost(usage: any): number {
    // Approximate costs per 1M tokens (update with actual pricing)
    const inputCostPerMillion = 3.0;
    const outputCostPerMillion = 15.0;
    
    return (
      (usage.input_tokens / 1_000_000) * inputCostPerMillion +
      (usage.output_tokens / 1_000_000) * outputCostPerMillion
    );
  }

  private extractConfidenceScore(text: string): number {
    // Simple confidence extraction - enhance with actual logic
    return 0.85;
  }
}
```

### 5. LiveKit Video Consultation

**Video Service Implementation**:

```typescript
// services/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  private apiKey = process.env.LIVEKIT_API_KEY;
  private apiSecret = process.env.LIVEKIT_API_SECRET;
  private livekitUrl = process.env.LIVEKIT_URL;

  async createVideoSession(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true }
    });

    if (!appointment.isVirtual) {
      throw new Error('Appointment is not configured for virtual session');
    }

    const roomName = `session-${appointmentId}`;

    // Create waiting room
    const waitingRoom = await this.prisma.waitingRoom.create({
      data: {
        appointmentId,
        roomName,
        status: 'WAITING',
        createdAt: new Date()
      }
    });

    return {
      roomName,
      waitingRoomId: waitingRoom.id,
      practitionerToken: this.generateToken(
        roomName,
        appointment.practitioner.id,
        appointment.practitioner.fullName,
        { canPublish: true, canSubscribe: true, canRecord: true }
      ),
      patientToken: this.generateToken(
        roomName,
        appointment.patient.id,
        appointment.patient.fullName,
        { canPublish: true, canSubscribe: true, canRecord: false }
      ),
      livekitUrl: this.livekitUrl
    };
  }

  private generateToken(
    roomName: string,
    identity: string,
    name: string,
    permissions: any
  ): string {
    const token = new AccessToken(this.apiKey, this.apiSecret, {
      identity,
      name
    });

    token.addGrant({
      room: roomName,
      roomJoin: true,
      canPublish: permissions.canPublish,
      canSubscribe: permissions.canSubscribe,
      canPublishData: true,
      canUpdateOwnMetadata: true
    });

    return token.toJwt();
  }

  async admitPatientFromWaitingRoom(waitingRoomId: string) {
    const waitingRoom = await this.prisma.waitingRoom.update({
      where: { id: waitingRoomId },
      data: { status: 'ADMITTED', admittedAt: new Date() }
    });

    // Notify patient via WebSocket
    this.eventEmitter.emit('waitingRoom.admitted', waitingRoom);

    return waitingRoom;
  }

  async startRecording(appointmentId: string, consent: boolean) {
    if (!consent) {
      throw new Error('Recording requires explicit patient consent');
    }

    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId }
    });

    // Start recording via LiveKit API
    // Implementation depends on LiveKit Egress setup

    await this.prisma.sessionRecording.create({
      data: {
        appointmentId,
        consentGranted: true,
        consentGrantedAt: new Date(),
        status: 'RECORDING'
      }
    });
  }
}
```

## Frontend Implementation (SvelteKit 5)

**Appointment Calendar Component**:

```svelte
<!-- routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Calendar } from '$lib/components/calendar';
  import { appointmentStore } from '$lib/stores/appointments';
  
  let appointments = $state([]);
  let selectedDate = $state(new Date());
  let showConflicts = $state(false);

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

  async function createAppointment(data: any) {
    try {
      const response = await fetch('/api/appointments', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: JSON.stringify(data)
      });

      if (!response.ok) {
        const error = await response.json();
        if (error.conflicts) {
          showConflicts = true;
          // Show conflict resolution UI
          return;
        }
        throw new Error(error.message);
      }

      await loadAppointments();
    } catch (err) {
      console.error('Error creating appointment:', err);
    }
  }
</script>

<div class="agenda-container">
  <header class="flex justify-between items-center mb-6">
    <h1 class="text-3xl font-bold text-gray-900">Agenda</h1>
    <button
      onclick={() => createAppointment({ /* ... */ })}
      class="btn-primary"
    >
      Nuevo Turno
    </button>
  </header>

  <Calendar
    {appointments}
    {selectedDate}
    onDateChange={(date) => selectedDate = date}
    onAppointmentClick={(apt) => /* handle click */}
  />

  {#if showConflicts}
    <ConflictResolutionModal
      onClose={() => showConflicts = false}
    />
  {/if}
</div>
```

**AI Assistant Chat Interface**:

```svelte
<!-- lib/components/AIAssistant.svelte -->
<script lang="ts">
  let messages = $state([]);
  let input = $state('');
  let isLoading = $state(false);

  async function sendMessage() {
    if (!input.trim()) return;

    const userMessage = { role: 'user', content: input };
    messages = [...messages, userMessage];
    input = '';
    isLoading = true;

    try {
      const response = await fetch('/api/ai/assistant', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: JSON.stringify({
          query: userMessage.content,
          context: {
            practitionerId: user.id,
            activePatients: patientCount,
            nextAppointment: nextAppointment
          }
        })
      });

      const data = await response.json();
      messages = [...messages, { role: 'assistant', content: data.response }];
    } catch (err) {
      console.error('AI Assistant error:', err);
    } finally {
      isLoading = false;
    }
  }
</script>

<div class="ai-assistant-panel">
  <div class="messages-container">
    {#each messages as message}
      <div class="message message-{message.role}">
        {message.content}
      </div>
    {/each}
    
    {#if isLoading}
      <div class="message message-assistant loading">
        <span class="animate-pulse">Pensando...</span>
      </div>
    {/if}
  </div>

  <form onsubmit={sendMessage} class="input-form">
    <input
      type="text"
      bind:value={input}
      placeholder="Preguntame algo sobre tus pacientes..."
      class="input-field"
    />
    <button type="submit" class="btn-send">Enviar</button>
  </form>
</div>
```

## Configuration

### Prisma Schema Essentials

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Practitioner {
  id              String   @id @default(cuid())
  email           String   @unique
  fullName        String
  cuit            String   @unique
  fiscalCategory  String
  specialization  String[]
  licenseNumber   String
  appointments    Appointment[]
  patients        Patient[]
  createdAt       DateTime @default(now())
}

model Patient {
  id              String   @id @default(cuid())
  
