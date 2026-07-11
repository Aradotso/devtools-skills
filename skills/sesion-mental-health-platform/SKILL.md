---
name: sesion-mental-health-platform
description: Build and integrate with Sesión, an AI-powered mental health practice management SaaS for Argentine psychologists with scheduling, WhatsApp automation, AFIP billing, and video consultations.
triggers:
  - how do I integrate Sesión scheduling into my app
  - set up WhatsApp automation for patient reminders
  - implement AFIP compliant invoicing with Sesión
  - configure Claude AI orchestration for clinical notes
  - build a patient journey pipeline in Sesión
  - integrate Sesión video consultations
  - query Sesión API for appointment data
  - troubleshoot Sesión webhook events
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, WhatsApp patient communications, AFIP-compliant electronic invoicing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit frontend, Prisma ORM, and integrations with Baileys (WhatsApp), LiveKit (video), MercadoPago/Stripe (payments), and Anthropic Claude models.

## Architecture Overview

Sesión is a microservices architecture with:

- **Backend**: NestJS (TypeScript) with modular domain services
- **Frontend**: SvelteKit 5 with TailwindCSS
- **Database**: PostgreSQL with Prisma ORM
- **Cache/Queue**: Redis for sessions and job queues
- **WhatsApp**: Baileys library for automation
- **Video**: LiveKit WebRTC engine
- **AI**: Anthropic Claude API (Opus 4.6 + Sonnet 4.6)
- **Payments**: MercadoPago (Argentina) and Stripe
- **Search**: Elasticsearch for patient records

## Installation & Setup

### Prerequisites

```bash
node >= 18.0.0
postgresql >= 14
redis >= 6.2
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
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db?schema=public"

# Redis
REDIS_URL="redis://localhost:6379"

# JWT Authentication
JWT_SECRET="your-jwt-secret-here"
JWT_EXPIRES_IN="7d"

# Anthropic Claude
ANTHROPIC_API_KEY="sk-ant-..."
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_AUTO_RECONNECT="true"

# LiveKit Video
LIVEKIT_API_KEY="your-livekit-api-key"
LIVEKIT_API_SECRET="your-livekit-secret"
LIVEKIT_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="your-clinic-cuit"
AFIP_CERT_PATH="./afip-cert.pem"
AFIP_KEY_PATH="./afip-key.key"
AFIP_PRODUCTION="false"

# MercadoPago
MERCADOPAGO_ACCESS_TOKEN="APP_USR-..."
MERCADOPAGO_PUBLIC_KEY="APP_USR-..."

# Stripe (optional)
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# Application
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
PORT="3000"
```

### Database Setup

```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate dev

# Seed initial data (optional)
npx prisma db seed
```

### Run Development Servers

```bash
# Backend (NestJS)
npm run start:dev

# Frontend (SvelteKit)
cd frontend
npm run dev
```

## Core Module Structure

### 1. Appointment Scheduling Module

**Create an Appointment**

```typescript
// src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateAppointmentDto } from './dto/create-appointment.dto';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async create(dto: CreateAppointmentDto, practitionerId: string) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { lte: dto.endTime },
        endTime: { gte: dto.startTime },
        status: { not: 'CANCELLED' }
      }
    });

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: dto.patientId,
        practitionerId,
        startTime: dto.startTime,
        endTime: dto.endTime,
        type: dto.type, // 'PRESENCIAL' | 'VIRTUAL' | 'EVALUACION'
        status: 'SCHEDULED',
        notes: dto.notes
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

  private async sendWhatsAppConfirmation(appointment: any) {
    // Implementation in WhatsApp module
  }
}
```

**DTO Definition**

```typescript
// src/appointments/dto/create-appointment.dto.ts
import { IsDateString, IsEnum, IsString, IsUUID, IsOptional } from 'class-validator';

export enum AppointmentType {
  PRESENCIAL = 'PRESENCIAL',
  VIRTUAL = 'VIRTUAL',
  EVALUACION = 'EVALUACION'
}

export class CreateAppointmentDto {
  @IsUUID()
  patientId: string;

  @IsDateString()
  startTime: string;

  @IsDateString()
  endTime: string;

  @IsEnum(AppointmentType)
  type: AppointmentType;

  @IsOptional()
  @IsString()
  notes?: string;
}
```

### 2. WhatsApp Automation Module

**WhatsApp Service with Baileys**

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
    await this.connectWhatsApp();
  }

  private async connectWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions'
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true
    });

    this.sock.ev.on('creds.update', saveCreds);
    
    this.sock.ev.on('connection.update', async (update: any) => {
      const { connection, lastDisconnect } = update;
      
      if (connection === 'close') {
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
        
        if (shouldReconnect && process.env.WHATSAPP_AUTO_RECONNECT === 'true') {
          await this.connectWhatsApp();
        }
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }: any) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentReminder(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true }
    });

    if (!appointment) return;

    const phoneNumber = this.formatPhoneNumber(appointment.patient.phone);
    const message = this.generateReminderMessage(appointment);

    await this.sock.sendMessage(phoneNumber, { text: message });

    await this.prisma.whatsAppMessage.create({
      data: {
        patientId: appointment.patientId,
        appointmentId: appointment.id,
        phoneNumber,
        message,
        direction: 'OUTBOUND',
        status: 'SENT'
      }
    });
  }

  private formatPhoneNumber(phone: string): string {
    // Argentina format: +54 9 11 XXXX-XXXX -> 5491111111111@s.whatsapp.net
    const cleaned = phone.replace(/\D/g, '');
    return `${cleaned}@s.whatsapp.net`;
  }

  private generateReminderMessage(appointment: any): string {
    const dateStr = new Date(appointment.startTime).toLocaleString('es-AR', {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    });

    return `Hola ${appointment.patient.firstName},\n\n` +
           `Te recordamos tu sesión con ${appointment.practitioner.firstName} ${appointment.practitioner.lastName}\n` +
           `📅 ${dateStr}\n` +
           `📍 ${appointment.type === 'VIRTUAL' ? 'Videollamada' : 'Presencial'}\n\n` +
           `Responde SI para confirmar o REPROGRAMAR si necesitas cambiar el horario.`;
  }

  private async handleIncomingMessage(message: any) {
    if (!message.key.fromMe && message.message) {
      const phoneNumber = message.key.remoteJid;
      const text = message.message.conversation || message.message.extendedTextMessage?.text || '';

      // Crisis detection keywords
      const crisisKeywords = ['suicidio', 'hacerme daño', 'no puedo más', 'crisis'];
      const isCrisis = crisisKeywords.some(keyword => text.toLowerCase().includes(keyword));

      if (isCrisis) {
        await this.escalateToPractitioner(phoneNumber, text);
      }

      // Store message
      await this.prisma.whatsAppMessage.create({
        data: {
          phoneNumber,
          message: text,
          direction: 'INBOUND',
          status: 'RECEIVED',
          isCrisisAlert: isCrisis
        }
      });
    }
  }

  private async escalateToPractitioner(phoneNumber: string, message: string) {
    // Find patient by phone and notify their practitioner
    const patient = await this.prisma.patient.findFirst({
      where: { phone: { contains: phoneNumber.replace('@s.whatsapp.net', '') } },
      include: { practitioner: true }
    });

    if (patient?.practitioner) {
      // Send email/push notification to practitioner
      // Implementation depends on notification service
    }
  }
}
```

**Schedule Automated Reminders**

```typescript
// src/appointments/appointments.scheduler.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsScheduler {
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

    for (const appointment of appointments) {
      await this.whatsapp.sendAppointmentReminder(appointment.id);
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent24h: true }
      });
    }
  }
}
```

### 3. AFIP Electronic Invoicing

**AFIP Billing Service**

```typescript
// src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { AfipServices } from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(private prisma: PrismaService) {
    this.afip = new AfipServices({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.AFIP_PRODUCTION === 'true'
    });
  }

  async createInvoice(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
        invoice: true
      }
    });

    if (!appointment || appointment.invoice) {
      throw new Error('Appointment not found or already invoiced');
    }

    // Determine invoice type based on patient fiscal category
    const voucherType = this.determineVoucherType(appointment.patient);
    
    const invoiceData = {
      CantReg: 1,
      PtoVta: 1, // Point of sale
      CbteTipo: voucherType, // 1=A, 6=B, 11=C
      Concepto: 2, // 1=Products, 2=Services, 3=Both
      DocTipo: 96, // 96=DNI, 80=CUIT
      DocNro: appointment.patient.documentNumber,
      CbteDesde: 1,
      CbteHasta: 1,
      CbteFch: this.formatDate(new Date()),
      ImpTotal: appointment.sessionPrice,
      ImpTotConc: 0,
      ImpNeto: appointment.sessionPrice,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      MonId: 'PES', // Argentine Pesos
      MonCotiz: 1
    };

    const response = await this.afip.ElectronicBilling.createVoucher(invoiceData);

    // Store invoice
    const invoice = await this.prisma.invoice.create({
      data: {
        appointmentId: appointment.id,
        patientId: appointment.patientId,
        practitionerId: appointment.practitionerId,
        voucherType,
        voucherNumber: response.CAE,
        amount: appointment.sessionPrice,
        currency: 'ARS',
        status: 'ISSUED',
        afipData: response,
        issuedAt: new Date()
      }
    });

    return invoice;
  }

  private determineVoucherType(patient: any): number {
    // A: Registered taxpayer (Responsable Inscripto)
    // B: Final consumer (Consumidor Final)
    // C: Exempt (Exento)
    switch (patient.fiscalCategory) {
      case 'RESPONSABLE_INSCRIPTO':
        return 1;
      case 'CONSUMIDOR_FINAL':
        return 6;
      case 'EXENTO':
        return 11;
      default:
        return 6;
    }
  }

  private formatDate(date: Date): string {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}${month}${day}`;
  }
}
```

### 4. Claude AI Orchestration

**AI Service with Model Router**

```typescript
// src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;
  private opusModel: string;
  private sonnetModel: string;

  constructor(private prisma: PrismaService) {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.opusModel = process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4.6';
    this.sonnetModel = process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4.6';
  }

  async summarizeClinicalNotes(sessionId: string): Promise<string> {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: { 
        patient: true,
        clinicalNotes: true
      }
    });

    if (!session) throw new Error('Session not found');

    const prompt = `Eres un asistente especializado en psicología clínica en Argentina. 
Resume las siguientes notas de sesión de forma clara, estructurada y profesional.

Paciente: ${session.patient.firstName} (ID anonimizado)
Fecha de sesión: ${session.date}
Duración: ${session.duration} minutos

Notas originales:
${session.clinicalNotes.map(note => note.content).join('\n\n')}

Genera un resumen que incluya:
1. Temas principales tratados
2. Observaciones clínicas relevantes
3. Progreso terapéutico
4. Próximos pasos sugeridos

Formato: Lenguaje profesional, argentino, conciso.`;

    const message = await this.anthropic.messages.create({
      model: this.opusModel, // Use Opus for deep analysis
      max_tokens: 2000,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    const summary = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    // Log AI usage
    await this.prisma.aiUsageLog.create({
      data: {
        model: this.opusModel,
        sessionId,
        promptTokens: message.usage.input_tokens,
        completionTokens: message.usage.output_tokens,
        task: 'CLINICAL_NOTE_SUMMARY'
      }
    });

    return summary;
  }

  async generateQuickResponse(query: string, context: any): Promise<string> {
    const prompt = `Eres un asistente de consulta rápida para psicólogos en Argentina.

Contexto del paciente:
${JSON.stringify(context, null, 2)}

Pregunta del profesional:
${query}

Responde de forma concisa y práctica en menos de 100 palabras.`;

    const message = await this.anthropic.messages.create({
      model: this.sonnetModel, // Use Sonnet for fast responses
      max_tokens: 500,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async predictOutcome(patientId: string): Promise<any> {
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        appointments: {
          orderBy: { startTime: 'desc' },
          take: 20
        },
        sessions: {
          include: { clinicalNotes: true },
          orderBy: { date: 'desc' },
          take: 10
        }
      }
    });

    if (!patient) throw new Error('Patient not found');

    const attendanceRate = this.calculateAttendanceRate(patient.appointments);
    const sessionSummaries = patient.sessions
      .map(s => s.clinicalNotes.map(n => n.content).join(' '))
      .join('\n');

    const prompt = `Analiza el siguiente historial clínico anonimizado y predice:
1. Probabilidad de abandono del tratamiento (0-100%)
2. Progreso terapéutico estimado
3. Factores de riesgo identificados

Tasa de asistencia: ${(attendanceRate * 100).toFixed(1)}%
Total de sesiones: ${patient.sessions.length}

Resumen de sesiones recientes:
${sessionSummaries.slice(0, 3000)} 

Responde en formato JSON estructurado.`;

    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 1500,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    const responseText = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '{}';

    return JSON.parse(responseText);
  }

  private calculateAttendanceRate(appointments: any[]): number {
    if (appointments.length === 0) return 0;
    const attended = appointments.filter(a => a.status === 'COMPLETED').length;
    return attended / appointments.length;
  }
}
```

### 5. LiveKit Video Integration

**Video Service**

```typescript
// src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class LiveKitService {
  private apiKey: string;
  private apiSecret: string;
  private livekitUrl: string;

  constructor(private prisma: PrismaService) {
    this.apiKey = process.env.LIVEKIT_API_KEY;
    this.apiSecret = process.env.LIVEKIT_API_SECRET;
    this.livekitUrl = process.env.LIVEKIT_URL;
  }

  async createVideoSession(appointmentId: string, userId: string, role: 'practitioner' | 'patient'): Promise<any> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true }
    });

    if (!appointment) throw new Error('Appointment not found');
    if (appointment.type !== 'VIRTUAL') {
      throw new Error('Appointment is not virtual');
    }

    const roomName = `session-${appointmentId}`;
    const participantName = role === 'practitioner' 
      ? `${appointment.practitioner.firstName} ${appointment.practitioner.lastName}`
      : `${appointment.patient.firstName} ${appointment.patient.lastName}`;

    const token = new AccessToken(this.apiKey, this.apiSecret, {
      identity: userId,
      name: participantName
    });

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: role === 'practitioner' // Only practitioner can share screen
    });

    const jwt = token.toJwt();

    // Create or update video session record
    await this.prisma.videoSession.upsert({
      where: { appointmentId },
      create: {
        appointmentId,
        roomName,
        status: 'ACTIVE',
        startedAt: new Date()
      },
      update: {
        status: 'ACTIVE',
        startedAt: new Date()
      }
    });

    return {
      token: jwt,
      url: this.livekitUrl,
      roomName
    };
  }

  async endVideoSession(appointmentId: string) {
    await this.prisma.videoSession.update({
      where: { appointmentId },
      data: {
        status: 'ENDED',
        endedAt: new Date()
      }
    });

    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: { status: 'COMPLETED' }
    });
  }
}
```

### 6. API Integration Patterns

**REST API Controller Example**

```typescript
// src/appointments/appointments.controller.ts
import { Controller, Get, Post, Body, Param, UseGuards, Req } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { AppointmentsService } from './appointments.service';
import { CreateAppointmentDto } from './dto/create-appointment.dto';

@Controller('api/appointments')
@UseGuards(JwtAuthGuard)
export class AppointmentsController {
  constructor(private appointmentsService: AppointmentsService) {}

  @Post()
  async create(@Body() dto: CreateAppointmentDto, @Req() req: any) {
    return this.appointmentsService.create(dto, req.user.id);
  }

  @Get()
  async findAll(@Req() req: any) {
    return this.appointmentsService.findAllByPractitioner(req.user.id);
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.appointmentsService.findOne(id);
  }

  @Post(':id/complete')
  async complete(@Param('id') id: string) {
    return this.appointmentsService.complete(id);
  }

  @Post(':id/cancel')
  async cancel(@Param('id') id: string, @Body() reason: string) {
    return this.appointmentsService.cancel(id, reason);
  }
}
```

**Webhook Handler for External Integrations**

```typescript
// src/webhooks/webhooks.controller.ts
import { Controller, Post, Body, Headers, RawBodyRequest, Req } from '@nestjs/common';
import { MercadoPagoService } from '../payments/mercadopago.service';
import { StripeService } from '../payments/stripe.service';

@Controller('webhooks')
export class WebhooksController {
  constructor(
    private mercadopago: MercadoPagoService,
    private stripe: StripeService
  ) {}

  @Post('mercadopago')
  async handleMercadoPago(@Body() body: any, @Headers('x-signature') signature: string) {
    // Verify webhook signature
    const isValid = await this.mercadopago.verifyWebhook(body, signature);
    if (!isValid) {
      throw new Error('Invalid webhook signature');
    }

    if (body.type === 'payment') {
      await this.mercadopago.processPayment(body.data.id);
    }

    return { received: true };
  }

  @Post('stripe')
  async handleStripe(@Req() req: RawBodyRequest<Request>, @Headers('stripe-signature') signature: string) {
    const event = await this.stripe.constructEvent(req.rawBody, signature);
    
    switch (event.type) {
      case 'payment_intent.succeeded':
        await this.stripe.handlePaymentSuccess(event.data.object);
        break;
      case 'payment_intent.payment_failed':
        await this.stripe.handlePaymentFailure(event.data.object);
        break;
    }

    return { received: true };
  }
}
```

## Frontend Integration (SvelteKit)

**Appointment Booking Component**

```svelte
<!-- frontend/src/routes/appointments/new/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  
  let patients: any[] = [];
  let selectedPatient = '';
  let appointmentType = 'PRESENCIAL';
  let startTime = '';
  let duration = 45;
  let notes = '';
  let loading = false;

  onMount(async () => {
    const res = await fetch('/api/patients');
    patients = await res.json();
  });

  async function handleSubmit() {
    loading = true;
    
    const startDate = new Date(startTime);
    const endDate = new Date(startDate.getTime() + duration * 60000);

    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        patientId: selectedPatient,
        startTime: startDate.toISOString(),
        endTime: endDate.toISOString(),
        type: appointmentType,
        notes
      })
    });

    if (response.ok) {
      goto('/appointments');
    } else {
      const error = await response.json();
      alert(error.message);
    }
    
    loading = false;
  }
</script>

<div class="max-w-2xl mx-auto p-6">
  <h1 class="text-3xl font-bold mb-6">Nueva Sesión</h1>
  
  <form on:submit|preventDefault={handleSubmit} class="space-y-4">
    <div>
      <label class="block text-sm font-medium mb-1">Paciente</label>
      <select bind:value={selectedPatient} required class="w-full p-2 border rounded">
        <option value="">Seleccionar paciente</option>
        {#each patients as patient}
          <option value={patient.id}>{patient.firstName} {patient.lastName}</option>
        {/each}
      </select>
    </div>

    <div>
      <label class="block text-sm font-medium mb-1">Tipo de sesión</label>
      <select bind:value={appointmentType} class="w-full p-2 border rounded">
        <option value="PRESENCIAL">Presencial</option>
        <option value="VIRTUAL">Videollamada</option>
        <option value="EVALUACION">Evaluación
