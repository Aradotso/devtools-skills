---
name: sesion-mental-health-platform
description: Intelligent practice management SaaS for psychology clinics with AI scheduling, WhatsApp automation, AFIP billing, and video consultations
triggers:
  - how do I set up Sesión for a psychology clinic
  - integrate WhatsApp automation with Sesión appointments
  - configure AFIP electronic invoicing in Sesión
  - implement Claude AI note summarization with Sesión
  - set up video consultations using Sesión platform
  - build custom integrations with Sesión REST API
  - deploy Sesión microservices architecture
  - configure Argentine compliance in Sesión
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive mental health practice orchestration platform built for Argentine psychology clinics. It unifies appointment scheduling, WhatsApp automation (Baileys), AFIP-compliant invoicing, secure video consultations (LiveKit), and AI-powered clinical assistance (Claude Opus 4.6 + Sonnet 4.6) into a single SaaS platform built with SvelteKit 5, NestJS, Prisma, and TypeScript.

## Architecture Overview

Sesión uses a microservices architecture with:
- **Frontend**: SvelteKit 5 + Svelte 5 + TailwindCSS
- **Backend**: NestJS with Prisma ORM
- **Database**: PostgreSQL for relational data, Redis for caching
- **AI**: Anthropic Claude Opus 4.6 (deep analysis) and Sonnet 4.6 (real-time chat)
- **Video**: LiveKit WebRTC infrastructure
- **WhatsApp**: Baileys library for WhatsApp Web automation
- **Payments**: Stripe (international) + Mercado Pago (Argentina)

## Installation & Setup

### Prerequisites

```bash
# Required tools
node >= 20.x
pnpm >= 9.x
docker >= 24.x
docker-compose >= 2.x
postgresql >= 15.x
redis >= 7.x
```

### Initial Setup

```bash
# Clone repository
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# .env
DATABASE_URL="postgresql://user:pass@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="your_anthropic_api_key"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./sessions/whatsapp"

# LiveKit Video
LIVEKIT_API_KEY="your_livekit_key"
LIVEKIT_API_SECRET="your_livekit_secret"
LIVEKIT_URL="wss://your-livekit-server.com"

# Payment Providers
STRIPE_SECRET_KEY="sk_live_..."
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."

# AFIP (Argentine Tax)
AFIP_CUIT="20123456789"
AFIP_CERT_PATH="./certs/afip.pem"
AFIP_KEY_PATH="./certs/afip.key"

# Application
JWT_SECRET="your_jwt_secret"
APP_URL="https://sesion.clinic"
```

### Database Setup

```bash
# Run migrations
pnpm prisma migrate dev

# Seed initial data
pnpm prisma db seed

# Generate Prisma client
pnpm prisma generate
```

### Start Development Environment

```bash
# Start all services with Docker Compose
docker-compose up -d

# Start backend (NestJS)
cd apps/backend
pnpm dev

# Start frontend (SvelteKit)
cd apps/frontend
pnpm dev
```

## Core Modules

### 1. Appointment Scheduling

#### Create Appointment (Backend)

```typescript
// apps/backend/src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateAppointmentDto } from './dto/create-appointment.dto';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async create(dto: CreateAppointmentDto) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: dto.practitionerId,
        startTime: { lt: dto.endTime },
        endTime: { gt: dto.startTime },
        status: { notIn: ['CANCELLED', 'COMPLETED'] }
      }
    });

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: dto.patientId,
        practitionerId: dto.practitionerId,
        startTime: dto.startTime,
        endTime: dto.endTime,
        sessionType: dto.sessionType, // PRESENCIAL, VIRTUAL, EVALUACION
        status: 'SCHEDULED'
      },
      include: {
        patient: true,
        practitioner: true
      }
    });

    // Trigger WhatsApp confirmation
    await this.sendWhatsAppConfirmation(appointment);

    return appointment;
  }

  async findAvailableSlots(practitionerId: string, date: Date) {
    const dayStart = new Date(date.setHours(0, 0, 0, 0));
    const dayEnd = new Date(date.setHours(23, 59, 59, 999));

    const booked = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: dayStart, lte: dayEnd },
        status: { notIn: ['CANCELLED'] }
      },
      select: { startTime: true, endTime: true }
    });

    // Generate available 45-minute slots
    const workHours = { start: 9, end: 20 };
    const slots = [];
    
    for (let hour = workHours.start; hour < workHours.end; hour++) {
      const slotStart = new Date(date);
      slotStart.setHours(hour, 0, 0, 0);
      const slotEnd = new Date(slotStart);
      slotEnd.setMinutes(45);

      const isBooked = booked.some(apt => 
        slotStart < apt.endTime && slotEnd > apt.startTime
      );

      if (!isBooked) {
        slots.push({ start: slotStart, end: slotEnd });
      }
    }

    return slots;
  }
}
```

#### Appointment Calendar (Frontend)

```svelte
<!-- apps/frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import type { Appointment } from '$lib/types';

  let appointments = $state<Appointment[]>([]);
  let selectedDate = $state(new Date());
  let availableSlots = $state<Date[]>([]);

  async function fetchAppointments() {
    const response = await fetch('/api/appointments', {
      headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
    });
    appointments = await response.json();
  }

  async function fetchAvailableSlots() {
    const practitionerId = localStorage.getItem('practitionerId');
    const response = await fetch(
      `/api/appointments/available?practitionerId=${practitionerId}&date=${selectedDate.toISOString()}`
    );
    availableSlots = await response.json();
  }

  function formatTime(date: Date) {
    return new Intl.DateTimeFormat('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false
    }).format(new Date(date));
  }

  onMount(() => {
    fetchAppointments();
    fetchAvailableSlots();
  });
</script>

<div class="p-6 max-w-7xl mx-auto">
  <h1 class="text-3xl font-bold mb-6">Agenda Inteligente</h1>

  <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
    <!-- Calendar View -->
    <div class="col-span-2 bg-white rounded-lg shadow p-4">
      {#each appointments as apt}
        <div class="border-l-4 border-blue-500 pl-4 py-3 mb-3 hover:bg-gray-50">
          <div class="flex justify-between items-start">
            <div>
              <h3 class="font-semibold">{apt.patient.name}</h3>
              <p class="text-sm text-gray-600">
                {formatTime(apt.startTime)} - {formatTime(apt.endTime)}
              </p>
              <span class="text-xs px-2 py-1 rounded bg-blue-100 text-blue-800">
                {apt.sessionType}
              </span>
            </div>
            <button class="text-gray-400 hover:text-gray-600">
              <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                <path d="M10 6a2 2 0 110-4 2 2 0 010 4zM10 12a2 2 0 110-4 2 2 0 010 4zM10 18a2 2 0 110-4 2 2 0 010 4z"/>
              </svg>
            </button>
          </div>
        </div>
      {/each}
    </div>

    <!-- Available Slots -->
    <div class="bg-white rounded-lg shadow p-4">
      <h2 class="text-lg font-semibold mb-4">Horarios Disponibles</h2>
      {#each availableSlots as slot}
        <button class="w-full text-left px-3 py-2 mb-2 border rounded hover:bg-blue-50">
          {formatTime(slot.start)}
        </button>
      {/each}
    </div>
  </div>
</div>
```

### 2. WhatsApp Automation

#### WhatsApp Service Setup

```typescript
// apps/backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, { DisconnectReason, useMultiFileAuthState } from '@whiskeysockets/baileys';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.initializeWhatsApp();
  }

  async initializeWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './sessions/whatsapp'
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true
    });

    this.sock.ev.on('creds.update', saveCreds);
    
    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentReminder(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true }
    });

    if (!appointment || !appointment.patient.phone) return;

    const reminderText = `🗓️ *Recordatorio de Sesión*\n\n` +
      `Hola ${appointment.patient.name},\n\n` +
      `Te recordamos tu sesión con ${appointment.practitioner.name}:\n` +
      `📅 ${new Intl.DateTimeFormat('es-AR', { dateStyle: 'full', timeStyle: 'short' }).format(appointment.startTime)}\n\n` +
      `Tipo: ${appointment.sessionType}\n\n` +
      `Por favor confirma tu asistencia respondiendo *SI* o cancela con *NO*.`;

    const jid = `${appointment.patient.phone}@s.whatsapp.net`;
    
    await this.sock.sendMessage(jid, { text: reminderText });

    await this.prisma.whatsAppMessage.create({
      data: {
        appointmentId,
        phone: appointment.patient.phone,
        message: reminderText,
        direction: 'OUTBOUND',
        status: 'SENT'
      }
    });
  }

  async handleIncomingMessage(message: any) {
    if (!message.message) return;

    const phone = message.key.remoteJid.replace('@s.whatsapp.net', '');
    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text || '';

    // Crisis keyword detection
    const crisisKeywords = ['suicidio', 'morir', 'acabar', 'terminar todo', 'no puedo más'];
    const isCrisis = crisisKeywords.some(kw => text.toLowerCase().includes(kw));

    if (isCrisis) {
      await this.escalateCrisis(phone, text);
      return;
    }

    // Parse confirmation responses
    const confirmWords = ['si', 'sí', 'confirmo', 'confirmar'];
    const cancelWords = ['no', 'cancelar', 'suspender'];

    if (confirmWords.some(w => text.toLowerCase().includes(w))) {
      await this.confirmAppointment(phone);
    } else if (cancelWords.some(w => text.toLowerCase().includes(w))) {
      await this.cancelAppointment(phone);
    }

    // Log message
    await this.prisma.whatsAppMessage.create({
      data: {
        phone,
        message: text,
        direction: 'INBOUND',
        status: 'RECEIVED'
      }
    });
  }

  async escalateCrisis(phone: string, message: string) {
    const patient = await this.prisma.patient.findFirst({
      where: { phone },
      include: { practitioner: true }
    });

    if (!patient) return;

    // Send urgent notification to practitioner
    const alertJid = `${patient.practitioner.phone}@s.whatsapp.net`;
    await this.sock.sendMessage(alertJid, {
      text: `🚨 *ALERTA DE CRISIS*\n\n` +
        `Paciente: ${patient.name}\n` +
        `Mensaje: ${message}\n\n` +
        `Por favor contactar de inmediato.`
    });

    // Log crisis event
    await this.prisma.crisisAlert.create({
      data: {
        patientId: patient.id,
        message,
        status: 'ESCALATED'
      }
    });
  }

  async confirmAppointment(phone: string) {
    const appointment = await this.prisma.appointment.findFirst({
      where: {
        patient: { phone },
        status: 'SCHEDULED',
        startTime: { gte: new Date() }
      },
      orderBy: { startTime: 'asc' }
    });

    if (appointment) {
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { status: 'CONFIRMED' }
      });

      const jid = `${phone}@s.whatsapp.net`;
      await this.sock.sendMessage(jid, {
        text: '✅ Tu sesión ha sido confirmada. ¡Te esperamos!'
      });
    }
  }
}
```

#### Schedule Reminders (Cron Job)

```typescript
// apps/backend/src/whatsapp/whatsapp-scheduler.service.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from './whatsapp.service';

@Injectable()
export class WhatsAppSchedulerService {
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
      }
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.id);
      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent24h: true }
      });
    }
  }

  @Cron(CronExpression.EVERY_30_MINUTES)
  async send2HourReminders() {
    const twoHoursAhead = new Date();
    twoHoursAhead.setHours(twoHoursAhead.getHours() + 2);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: new Date(),
          lte: twoHoursAhead
        },
        status: { in: ['SCHEDULED', 'CONFIRMED'] },
        reminderSent2h: false
      }
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.id);
      await this.prisma.appointment.update({
        where: { id: apt.id },
        data: { reminderSent2h: true }
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

#### AFIP Billing Service

```typescript
// apps/backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import * as soap from 'soap';
import * as fs from 'fs';

interface AFIPInvoiceData {
  invoiceType: 'A' | 'B' | 'C' | 'M';
  salePoint: number;
  invoiceNumber: number;
  invoiceDate: string;
  totalAmount: number;
  taxedAmount: number;
  exemptAmount: number;
  vatAmount: number;
  customerCuit: string;
  customerName: string;
}

@Injectable()
export class AFIPService {
  private wsaa: any;
  private wsfe: any;

  constructor(private prisma: PrismaService) {}

  async authenticate(): Promise<string> {
    // Generate WSAA authentication ticket
    const tra = this.generateTRA();
    const cms = this.signTRA(tra);

    const client = await soap.createClientAsync(
      'https://wsaa.afip.gov.ar/ws/services/LoginCms?wsdl'
    );

    const result = await client.loginCmsAsync({
      in0: cms
    });

    const token = this.parseToken(result);
    return token;
  }

  private generateTRA(): string {
    const now = new Date();
    const expiration = new Date(now.getTime() + 12 * 60 * 60 * 1000); // 12 hours

    return `<?xml version="1.0" encoding="UTF-8"?>
<loginTicketRequest version="1.0">
  <header>
    <uniqueId>${Date.now()}</uniqueId>
    <generationTime>${now.toISOString()}</generationTime>
    <expirationTime>${expiration.toISOString()}</expirationTime>
  </header>
  <service>wsfe</service>
</loginTicketRequest>`;
  }

  private signTRA(tra: string): string {
    const crypto = require('crypto');
    const privateKey = fs.readFileSync(process.env.AFIP_KEY_PATH!);
    const certificate = fs.readFileSync(process.env.AFIP_CERT_PATH!);

    const sign = crypto.createSign('SHA256');
    sign.update(tra);
    const signature = sign.sign(privateKey, 'base64');

    return `${certificate}\n${signature}`;
  }

  async generateInvoice(sessionId: string): Promise<AFIPInvoiceData> {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        patient: true,
        practitioner: { include: { fiscalConfig: true } }
      }
    });

    if (!session) throw new Error('Session not found');

    const token = await this.authenticate();
    const fiscalConfig = session.practitioner.fiscalConfig;

    // Determine invoice type based on practitioner and patient fiscal status
    const invoiceType = this.determineInvoiceType(
      fiscalConfig.fiscalCategory,
      session.patient.fiscalStatus
    );

    // Get next invoice number
    const lastInvoice = await this.prisma.invoice.findFirst({
      where: {
        practitionerId: session.practitionerId,
        invoiceType,
        salePoint: fiscalConfig.salePoint
      },
      orderBy: { invoiceNumber: 'desc' }
    });

    const invoiceNumber = (lastInvoice?.invoiceNumber || 0) + 1;

    const client = await soap.createClientAsync(
      'https://servicios1.afip.gov.ar/wsfev1/service.asmx?WSDL'
    );

    const invoiceData = {
      Auth: { Token: token, Sign: token, Cuit: process.env.AFIP_CUIT },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: fiscalConfig.salePoint,
          CbteTipo: this.getInvoiceTypeCode(invoiceType)
        },
        FeDetReq: {
          FECAEDetRequest: {
            Concepto: 2, // Servicios
            DocTipo: 80, // CUIT
            DocNro: session.patient.cuit || 0,
            CbteDesde: invoiceNumber,
            CbteHasta: invoiceNumber,
            CbteFch: new Date().toISOString().split('T')[0].replace(/-/g, ''),
            ImpTotal: session.amount,
            ImpTotConc: 0,
            ImpNeto: session.amount / 1.21,
            ImpOpEx: 0,
            ImpIVA: session.amount - (session.amount / 1.21),
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
            Iva: {
              AlicIva: {
                Id: 5, // 21%
                BaseImp: session.amount / 1.21,
                Importe: session.amount - (session.amount / 1.21)
              }
            }
          }
        }
      }
    };

    const response = await client.FECAESolicitarAsync(invoiceData);
    const cae = response[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse[0].CAE;
    const caeExpiration = response[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse[0].CAEFchVto;

    // Store invoice in database
    const invoice = await this.prisma.invoice.create({
      data: {
        sessionId,
        practitionerId: session.practitionerId,
        patientId: session.patientId,
        invoiceType,
        salePoint: fiscalConfig.salePoint,
        invoiceNumber,
        cae,
        caeExpiration: new Date(caeExpiration),
        totalAmount: session.amount,
        taxedAmount: session.amount / 1.21,
        vatAmount: session.amount - (session.amount / 1.21),
        status: 'APPROVED'
      }
    });

    return {
      invoiceType,
      salePoint: fiscalConfig.salePoint,
      invoiceNumber,
      invoiceDate: new Date().toISOString(),
      totalAmount: session.amount,
      taxedAmount: session.amount / 1.21,
      exemptAmount: 0,
      vatAmount: session.amount - (session.amount / 1.21),
      customerCuit: session.patient.cuit || '',
      customerName: session.patient.name
    };
  }

  private determineInvoiceType(
    practitionerCategory: string,
    patientStatus: string
  ): 'A' | 'B' | 'C' | 'M' {
    // Responsable Inscripto to Responsable Inscripto = A
    // Responsable Inscripto to Consumidor Final = B
    // Monotributo = C
    // Monotributo to Responsable Inscripto = M

    if (practitionerCategory === 'RESPONSABLE_INSCRIPTO') {
      return patientStatus === 'RESPONSABLE_INSCRIPTO' ? 'A' : 'B';
    }
    return patientStatus === 'RESPONSABLE_INSCRIPTO' ? 'M' : 'C';
  }

  private getInvoiceTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type as keyof typeof codes];
  }
}
```

### 4. Claude AI Integration

#### AI Orchestration Service

```typescript
// apps/backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class ClaudeService {
  private client: Anthropic;

  constructor(private prisma: PrismaService) {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }

  async summarizeSession(sessionId: string): Promise<string> {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        patient: true,
        clinicalNotes: true
      }
    });

    if (!session || !session.clinicalNotes) {
      throw new Error('Session or notes not found');
    }

    // Use Opus for deep cognitive analysis
    const response = await this.client.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4.6',
      max_tokens: 2000,
      temperature: 0.3,
      system: `Eres un asistente especializado en psicología clínica en Argentina. 
Tu tarea es resumir notas de sesión de manera profesional, confidencial y estructurada.
Identifica: objetivos terapéuticos, técnicas utilizadas, progreso del paciente, y tareas para próxima sesión.`,
      messages: [
        {
          role: 'user',
          content: `Resumí las siguientes notas de sesión:\n\n${session.clinicalNotes.content}`
        }
      ]
    });

    const summary = response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';

    // Store AI-generated summary
    await this.prisma.sessionSummary.create({
      data: {
        sessionId,
        content: summary,
        model: 'claude-opus-4.6',
        tokensUsed: response.usage.input_tokens + response.usage.output_tokens
      }
    });

    return summary;
  }

  async chatAssistant(
    practitionerId: string,
    query: string,
    context?: { patientId?: string; sessionId?: string }
  ): Promise<string> {
    let systemContext = `Eres un asistente inteligente para psicólogos argentinos. 
Ayudás con búsqueda de información de pacientes, redacción de notas, y consultas sobre prácticas clínicas.`;

    if (context?.patientId) {
      const patient = await this.prisma.patient.findUnique({
        where: { id: context.patientId },
        include: {
          sessions: { take: 5, orderBy: { date: 'desc' } },
          diagnoses: true
        }
      });

      if (patient) {
        systemContext += `\n\nContexto del paciente:
Nombre: ${patient.name}
Edad: ${patient.age}
Diagnósticos: ${patient.diagnoses.map(d => d.code).join(', ')}
Últimas sesiones: ${patient.sessions.length}`;
      }
    }

    // Use Sonnet for real-time interactive responses
    const response = await this.client.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4.6',
      max_tokens: 1000,
      temperature: 0.7,
      system: systemContext,
      messages: [
        { role: 'user', content: query }
      ]
    });

    // Log usage for billing
    await this.prisma.aiUsageLog.create({
      data: {
        practitionerId,
        model: 'claude-sonnet-4.6',
        tokensUsed: response.usage.input_tokens + response.usage.output_tokens,
        operation: 'chat',
        context: context ? JSON.stringify(context) : null
      }
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  }

  async predictOutcome(patientId: string): Promise<{
    riskScore: number;
    factors: string[];
    recommendations: string[];
  }> {
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        sessions: { orderBy: { date: 'desc' } },
        diagnoses: true,
        assessments: true
      }
    });

    if (!patient) throw new Error('Patient not found');

    const response = await this.client.messages.create({
      model: process.env.CLAUDE_
