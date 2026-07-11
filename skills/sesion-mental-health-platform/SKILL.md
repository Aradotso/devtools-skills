---
name: sesion-mental-health-platform
description: AI-powered mental health practice management platform for psychologists with scheduling, WhatsApp automation, AFIP billing, video calls, and Claude AI integration
triggers:
  - how do I set up Sesión for a psychology clinic
  - configure WhatsApp automation for patient appointments
  - integrate Claude AI for clinical notes in Sesión
  - set up AFIP compliant invoicing in Sesión
  - implement video consultation workflows
  - manage patient journeys in Sesión platform
  - configure multi-practitioner scheduling
  - automate appointment reminders via WhatsApp
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit 5 frontend, Prisma ORM, and TypeScript throughout.

## Project Structure

```
├── apps/
│   ├── api/          # NestJS backend microservices
│   ├── web/          # SvelteKit 5 frontend
│   └── workers/      # Background job processors
├── packages/
│   ├── shared/       # Shared TypeScript types
│   └── prisma/       # Database schema and migrations
└── docker-compose.yml
```

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 20.0.0
pnpm >= 8.0.0
docker >= 24.0.0
postgresql >= 15.0
redis >= 7.0
```

### Initial Setup

```bash
# Clone and install dependencies
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic
pnpm install

# Set up environment variables
cp .env.example .env

# Required environment variables
cat > .env << EOF
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# Claude AI
ANTHROPIC_API_KEY=sk-ant-xxxxx
CLAUDE_OPUS_MODEL=claude-opus-4.6
CLAUDE_SONNET_MODEL=claude-sonnet-4.6

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH=./whatsapp-sessions
WHATSAPP_WEBHOOK_SECRET=your-webhook-secret

# Video (LiveKit)
LIVEKIT_API_URL=wss://your-instance.livekit.cloud
LIVEKIT_API_KEY=your-livekit-key
LIVEKIT_API_SECRET=your-livekit-secret

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN=your-mp-token
STRIPE_SECRET_KEY=sk_test_xxxxx

# AFIP (Argentina Tax Authority)
AFIP_CUIT=your-clinic-cuit
AFIP_CERT_PATH=./certs/afip.crt
AFIP_KEY_PATH=./certs/afip.key
EOF

# Initialize database
pnpm prisma:migrate:dev
pnpm prisma:generate

# Start development servers
pnpm dev
```

### Docker Deployment

```bash
# Production build
docker-compose up -d

# View logs
docker-compose logs -f api web

# Run migrations
docker-compose exec api pnpm prisma:migrate:deploy
```

## Core Modules

### 1. Appointment Scheduling

```typescript
// apps/api/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/prisma/prisma.service';
import { AppointmentType, RecurrencePattern } from '@prisma/client';

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    practitionerId: string;
    patientId: string;
    startTime: Date;
    type: AppointmentType;
    recurrence?: RecurrencePattern;
  }) {
    // Check for conflicts
    const conflicts = await this.findConflicts(
      data.practitionerId,
      data.startTime,
      data.type
    );

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        practitioner: { connect: { id: data.practitionerId } },
        patient: { connect: { id: data.patientId } },
        startTime: data.startTime,
        endTime: this.calculateEndTime(data.startTime, data.type),
        type: data.type,
        status: 'SCHEDULED',
        recurrencePattern: data.recurrence,
      },
      include: {
        practitioner: true,
        patient: true,
      },
    });

    // Trigger WhatsApp confirmation
    await this.whatsappService.sendConfirmation(appointment);

    return appointment;
  }

  private calculateEndTime(start: Date, type: AppointmentType): Date {
    const duration = {
      INDIVIDUAL: 45,
      COUPLES: 90,
      FAMILY: 90,
      ASSESSMENT: 60,
    }[type];

    return new Date(start.getTime() + duration * 60000);
  }

  async findAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number
  ) {
    const dayStart = new Date(date.setHours(0, 0, 0, 0));
    const dayEnd = new Date(date.setHours(23, 59, 59, 999));

    const booked = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: dayStart, lte: dayEnd },
        status: { not: 'CANCELLED' },
      },
      select: { startTime: true, endTime: true },
    });

    // Generate available slots (8 AM - 8 PM)
    const slots = [];
    for (let hour = 8; hour < 20; hour++) {
      for (let minute = 0; minute < 60; minute += 15) {
        const slotStart = new Date(date);
        slotStart.setHours(hour, minute, 0, 0);
        const slotEnd = new Date(slotStart.getTime() + duration * 60000);

        const isAvailable = !booked.some(
          (b) => slotStart < b.endTime && slotEnd > b.startTime
        );

        if (isAvailable) {
          slots.push({ start: slotStart, end: slotEnd });
        }
      }
    }

    return slots;
  }
}
```

### 2. WhatsApp Automation (Baileys)

```typescript
// apps/api/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '@/prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private socket: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  private async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    this.socket = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.socket.ev.on('creds.update', saveCreds);

    this.socket.ev.on('connection.update', async (update: any) => {
      const { connection, lastDisconnect } = update;

      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;

        if (shouldReconnect) {
          await this.connectToWhatsApp();
        }
      }
    });

    this.socket.ev.on('messages.upsert', async (m: any) => {
      await this.handleIncomingMessage(m);
    });
  }

  async sendAppointmentReminder(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    const phone = appointment.patient.phone.replace(/\D/g, '') + '@s.whatsapp.net';
    const message = `Hola ${appointment.patient.firstName}! 👋\n\nTe recordamos tu sesión con ${appointment.practitioner.fullName}:\n\n📅 ${appointment.startTime.toLocaleDateString('es-AR')}\n🕐 ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n\n¿Confirmás tu asistencia? Respondé:\n✅ SI para confirmar\n❌ NO para cancelar`;

    await this.socket.sendMessage(phone, { text: message });

    await this.prisma.whatsAppMessage.create({
      data: {
        patientId: appointment.patientId,
        appointmentId: appointment.id,
        direction: 'OUTBOUND',
        content: message,
        status: 'SENT',
      },
    });
  }

  private async handleIncomingMessage(messageUpdate: any) {
    const message = messageUpdate.messages[0];
    if (!message.message || message.key.fromMe) return;

    const phone = message.key.remoteJid.replace('@s.whatsapp.net', '');
    const text = message.message.conversation || '';

    // Check for crisis keywords
    const crisisKeywords = [
      'suicidio',
      'matarme',
      'no puedo más',
      'emergencia',
    ];
    if (crisisKeywords.some((kw) => text.toLowerCase().includes(kw))) {
      await this.handleCrisisEscalation(phone, text);
      return;
    }

    // Parse confirmation responses
    if (/^(si|sí|confirmo|ok)$/i.test(text.trim())) {
      await this.confirmAppointment(phone);
    } else if (/^(no|cancelar|cancelo)$/i.test(text.trim())) {
      await this.cancelAppointment(phone);
    }
  }

  private async handleCrisisEscalation(phone: string, message: string) {
    const patient = await this.prisma.patient.findFirst({
      where: { phone: { contains: phone } },
      include: { practitioner: true },
    });

    if (patient) {
      // Alert practitioner immediately
      await this.socket.sendMessage(
        patient.practitioner.phone + '@s.whatsapp.net',
        {
          text: `🚨 ALERTA DE CRISIS\n\nPaciente: ${patient.fullName}\nMensaje: "${message}"\n\nSe requiere contacto inmediato.`,
        }
      );
    }
  }
}
```

### 3. AFIP Electronic Invoicing

```typescript
// apps/api/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/prisma/prisma.service';
import AfipSDK from 'afip-sdk';
import * as fs from 'fs';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new AfipSDK({
      cuit: parseInt(process.env.AFIP_CUIT),
      cert: fs.readFileSync(process.env.AFIP_CERT_PATH, 'utf8'),
      key: fs.readFileSync(process.env.AFIP_KEY_PATH, 'utf8'),
      production: process.env.NODE_ENV === 'production',
    });
  }

  async generateInvoice(data: {
    appointmentId: string;
    amount: number;
    paymentMethod: string;
  }) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: data.appointmentId },
      include: { patient: true, practitioner: true },
    });

    // Determine invoice type based on patient's fiscal condition
    const invoiceType = this.determineInvoiceType(
      appointment.patient.fiscalCondition
    );

    const invoice = await this.afip.ElectronicBilling.createVoucher({
      CantReg: 1,
      PtoVta: appointment.practitioner.puntoVenta,
      CbteTipo: invoiceType,
      Concepto: 1, // Servicios
      DocTipo: 80, // CUIT
      DocNro: appointment.patient.cuit || appointment.patient.dni,
      CbteFch: new Date().toISOString().split('T')[0].replace(/-/g, ''),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES',
      MonCotiz: 1,
    });

    // Save to database
    const dbInvoice = await this.prisma.invoice.create({
      data: {
        appointmentId: data.appointmentId,
        patientId: appointment.patientId,
        practitionerId: appointment.practitionerId,
        afipCae: invoice.CAE,
        afipCaeExpiration: new Date(invoice.CAEFchVto),
        invoiceNumber: invoice.CbteDesde,
        invoiceType,
        amount: data.amount,
        paymentMethod: data.paymentMethod,
        status: 'ISSUED',
      },
    });

    // Send invoice via WhatsApp
    await this.sendInvoiceToPatient(dbInvoice.id);

    return dbInvoice;
  }

  private determineInvoiceType(fiscalCondition: string): number {
    // 1 = Factura A, 6 = Factura B, 11 = Factura C
    const typeMap: Record<string, number> = {
      RESPONSABLE_INSCRIPTO: 1,
      MONOTRIBUTO: 6,
      CONSUMIDOR_FINAL: 11,
    };
    return typeMap[fiscalCondition] || 11;
  }

  async getMonthlyReport(practitionerId: string, year: number, month: number) {
    const invoices = await this.prisma.invoice.findMany({
      where: {
        practitionerId,
        createdAt: {
          gte: new Date(year, month - 1, 1),
          lt: new Date(year, month, 1),
        },
      },
    });

    const total = invoices.reduce((sum, inv) => sum + inv.amount, 0);
    const byType = invoices.reduce((acc, inv) => {
      acc[inv.invoiceType] = (acc[inv.invoiceType] || 0) + inv.amount;
      return acc;
    }, {} as Record<number, number>);

    return {
      totalRevenue: total,
      invoiceCount: invoices.length,
      byType,
      iibbEstimate: total * 0.03, // Approx. IIBB rate
    };
  }
}
```

### 4. Claude AI Integration

```typescript
// apps/api/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '@/prisma/prisma.service';

@Injectable()
export class ClaudeService {
  private client: Anthropic;

  constructor(private prisma: PrismaService) {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        clinicalNotes: true,
        patient: {
          include: {
            appointments: {
              where: { status: 'COMPLETED' },
              include: { clinicalNotes: true },
              orderBy: { startTime: 'desc' },
              take: 10,
            },
          },
        },
      },
    });

    const context = this.buildPatientContext(appointment);

    const message = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `Sos un psicólogo clínico especializado en resumir historias clínicas. Analizá el siguiente contexto del paciente y generá un resumen conciso de los últimos encuentros, destacando patrones, avances terapéuticos y áreas que requieren atención.

Contexto del paciente:
${context}

Generá un resumen estructurado en formato markdown con las siguientes secciones:
- Progreso terapéutico
- Temas recurrentes
- Intervenciones efectivas
- Recomendaciones para próximas sesiones`,
        },
      ],
    });

    const summary = message.content[0].text;

    await this.prisma.aiAnalysis.create({
      data: {
        appointmentId,
        modelUsed: process.env.CLAUDE_OPUS_MODEL,
        analysisType: 'CLINICAL_SUMMARY',
        input: context,
        output: summary,
        tokensUsed: message.usage.input_tokens + message.usage.output_tokens,
      },
    });

    return summary;
  }

  async generateSessionNotes(sessionData: {
    appointmentId: string;
    observations: string;
    interventions: string;
  }) {
    const response = await this.client.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Generá una nota clínica profesional basada en:

Observaciones: ${sessionData.observations}
Intervenciones: ${sessionData.interventions}

Formato: Notas SOAP (Subjetivo, Objetivo, Análisis, Plan)`,
        },
      ],
    });

    return response.content[0].text;
  }

  private buildPatientContext(appointment: any): string {
    const history = appointment.patient.appointments
      .map((appt: any) => {
        const notes = appt.clinicalNotes
          .map((n: any) => n.content)
          .join('\n');
        return `Sesión ${appt.startTime.toLocaleDateString('es-AR')}:\n${notes}`;
      })
      .join('\n\n---\n\n');

    return `Paciente: ${appointment.patient.firstName} (ID: ${appointment.patient.id})
Edad: ${appointment.patient.age}
Diagnóstico principal: ${appointment.patient.primaryDiagnosis || 'No especificado'}

Historial de sesiones:
${history}`;
  }

  async detectCrisisRisk(messageContent: string): Promise<{
    riskLevel: 'LOW' | 'MEDIUM' | 'HIGH';
    reasoning: string;
  }> {
    const response = await this.client.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL,
      max_tokens: 512,
      messages: [
        {
          role: 'user',
          content: `Evaluá el nivel de riesgo de crisis en este mensaje de un paciente. Respondé ÚNICAMENTE en formato JSON.

Mensaje: "${messageContent}"

Formato de respuesta:
{
  "riskLevel": "LOW" | "MEDIUM" | "HIGH",
  "reasoning": "explicación breve"
}`,
        },
      ],
    });

    return JSON.parse(response.content[0].text);
  }
}
```

### 5. Video Consultation (LiveKit)

```typescript
// apps/api/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken, RoomServiceClient } from 'livekit-server-sdk';
import { PrismaService } from '@/prisma/prisma.service';

@Injectable()
export class LiveKitService {
  private roomService: RoomServiceClient;

  constructor(private prisma: PrismaService) {
    this.roomService = new RoomServiceClient(
      process.env.LIVEKIT_API_URL,
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET
    );
  }

  async createConsultationRoom(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    const roomName = `session-${appointmentId}`;

    // Create room with recording enabled
    await this.roomService.createRoom({
      name: roomName,
      emptyTimeout: 300, // 5 minutes
      maxParticipants: 2,
    });

    // Generate tokens
    const practitionerToken = this.generateToken(
      roomName,
      appointment.practitioner.id,
      {
        canPublish: true,
        canSubscribe: true,
        canPublishData: true,
        canUpdateOwnMetadata: true,
        recorder: true, // Can start/stop recording
      }
    );

    const patientToken = this.generateToken(
      roomName,
      appointment.patient.id,
      {
        canPublish: true,
        canSubscribe: true,
        canPublishData: true,
      }
    );

    await this.prisma.videoSession.create({
      data: {
        appointmentId,
        roomName,
        status: 'WAITING',
      },
    });

    return {
      roomName,
      practitionerToken,
      patientToken,
      url: `${process.env.LIVEKIT_API_URL}?token=`,
    };
  }

  private generateToken(
    roomName: string,
    identity: string,
    grants: any
  ): string {
    const token = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity,
      }
    );

    token.addGrant({ room: roomName, ...grants });

    return token.toJwt();
  }

  async startRecording(appointmentId: string) {
    const session = await this.prisma.videoSession.findUnique({
      where: { appointmentId },
    });

    // Verify double consent
    const consent = await this.prisma.recordingConsent.findFirst({
      where: {
        appointmentId,
        patientConsent: true,
        practitionerConsent: true,
      },
    });

    if (!consent) {
      throw new Error('Recording consent not obtained from both parties');
    }

    await this.roomService.updateRoomMetadata(session.roomName, {
      recording: 'true',
    });

    await this.prisma.videoSession.update({
      where: { appointmentId },
      data: { isRecording: true, recordingStartedAt: new Date() },
    });
  }
}
```

### 6. Frontend Components (SvelteKit 5)

```svelte
<!-- apps/web/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Calendar } from '$lib/components/Calendar';
  import { AppointmentModal } from '$lib/components/AppointmentModal';
  import type { Appointment } from '$lib/types';

  let appointments = $state<Appointment[]>([]);
  let selectedDate = $state(new Date());
  let showModal = $state(false);
  let selectedSlot = $state<{ start: Date; end: Date } | null>(null);

  async function loadAppointments(date: Date) {
    const response = await fetch(`/api/appointments?date=${date.toISOString()}`);
    appointments = await response.json();
  }

  async function createAppointment(data: {
    patientId: string;
    type: string;
    recurrence?: string;
  }) {
    await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        ...data,
        startTime: selectedSlot?.start.toISOString(),
      }),
    });

    showModal = false;
    await loadAppointments(selectedDate);
  }

  onMount(() => {
    loadAppointments(selectedDate);
  });
</script>

<div class="agenda-container">
  <header>
    <h1>Agenda Inteligente</h1>
    <button onclick={() => (showModal = true)}>Nueva Sesión</button>
  </header>

  <Calendar
    {appointments}
    {selectedDate}
    onDateChange={(date) => {
      selectedDate = date;
      loadAppointments(date);
    }}
    onSlotClick={(slot) => {
      selectedSlot = slot;
      showModal = true;
    }}
  />

  {#if showModal}
    <AppointmentModal
      slot={selectedSlot}
      onConfirm={createAppointment}
      onCancel={() => (showModal = false)}
    />
  {/if}
</div>

<style>
  .agenda-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
  }

  button {
    background: var(--primary-color);
    color: white;
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 8px;
    cursor: pointer;
  }
</style>
```

## Database Schema (Prisma)

```prisma
// packages/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Practitioner {
  id           String        @id @default(cuid())
  email        String        @unique
  fullName     String
  phone        String
  cuit         String        @unique
  puntoVenta   Int
  fiscalType   FiscalType
  appointments Appointment[]
  patients     Patient[]
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
}

model Patient {
  id                String              @id @default(cuid())
  firstName         String
  lastName          String
  email             String?
  phone             String              @unique
  dni               String?
  cuit              String?
  age               Int?
  fiscalCondition   String              @default("CONSUMIDOR_FINAL")
  primaryDiagnosis  String?
  practitionerId    String
  practitioner      Practitioner        @relation(fields: [practitionerId], references: [id])
  appointments      Appointment[]
  whatsappMessages  WhatsAppMessage[]
  journeyStage      JourneyStage        @default(FIRST_CONTACT)
  createdAt         DateTime            @default(now())
  updatedAt         DateTime            @updatedAt
}

model Appointment {
  id                String            @id @default(cuid())
  practitionerId    String
  practitioner      Practitioner      @relation(fields: [practitionerId], references: [id])
  patientId         String
  patient           Patient           @relation(fields: [patientId], references: [id])
  startTime         DateTime
  endTime           DateTime
  type              AppointmentType
  status            AppointmentStatus @default(SCHEDULED)
  recurrencePattern RecurrencePattern?
  clinicalNotes     ClinicalNote[]
  invoice           Invoice?
  videoSession      VideoSession?
  aiAnalysis        AIAnalysis[]
  createdAt         DateTime          @default(now())
  updatedAt         DateTime          @updatedAt

  @@index([practitionerId, startTime])
}

model ClinicalNote {
  id            String      @id @default(cuid())
  appointmentId String
  appointment   Appointment @relation(fields: [appointmentId], references: [id])
  content       String      @db.Text
  isConfidential Boolean   @default(true)
  createdAt     DateTime    @default(now())
}

model Invoice {
  id                 String      @id @default(cuid())
  appointmentId      String      @unique
  appointment        Appointment @relation(fields: [appointmentId], references: [id])
  patientId          String
  practitionerId     String
  afipCae            String
  afipCaeExpiration  DateTime
  invoiceNumber      Int
  invoiceType        Int
  amount             Float
  paymentMethod      String
  status             InvoiceStatus @default(ISSUED)
  pdfUrl             String?
  createdAt          DateTime    @default(now())
}

model WhatsAppMessage {
  id            String             @id @default(cuid())
  patientId     String
  patient       Patient            @relation(fields: [patientId], references: [id])
  appointmentId String?
  direction     MessageDirection
  content       String             @db.Text
  status        MessageStatus
  createdAt     DateTime           @default(now())
}

model VideoSession {
  id                  String      @id @default(cuid())
  appointmentId       String      @unique
  appointment         Appointment @relation(fields: [appointmentId], references: [id])
  roomName            String
  status              VideoStatus @default(WAITING)
  isRecording         Boolean     @default(false)
  recordingStartedAt  DateTime?
  recordingUrl        String?
  startedAt           DateTime?
  endedAt             DateTime?
  createdAt           DateTime    @default(now())
}

model AIAnalysis {
  id            String       @id @default(cuid())
  appointmentId String
  appointment   Appointment  @relation(fields: [appointmentId], references: [id])
  modelUsed     String
  analysisType  AnalysisType
