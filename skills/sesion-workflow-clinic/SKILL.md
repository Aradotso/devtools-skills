---
name: sesion-workflow-clinic
description: SaaS platform for psychology clinics with intelligent scheduling, WhatsApp automation, AFIP billing, videollamadas, and Claude AI integration
triggers:
  - set up sesion clinic management platform
  - integrate whatsapp automation for patient appointments
  - implement afip compliant invoicing for argentina
  - configure claude ai for clinical notes
  - build psychology practice management system
  - create appointment scheduling with video consultations
  - add mercadopago payment processing
  - implement livekit video sessions
---

# Sesión Workflow Clinic Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, WhatsApp patient communication, AFIP-compliant electronic invoicing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Architecture Overview

The platform is built as a microservices architecture with:
- **Backend**: NestJS (TypeScript) for RESTful API and orchestration
- **Frontend**: SvelteKit 5 with Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **WhatsApp**: Baileys for automation
- **Video**: LiveKit for WebRTC consultations
- **AI**: Anthropic Claude (Opus & Sonnet models)
- **Payments**: Stripe (international) + Mercado Pago (Argentina)
- **Caching**: Redis for sessions and real-time data

## Installation

### Prerequisites

```bash
# Required versions
node >= 18.0.0
pnpm >= 8.0.0
postgresql >= 14.0
redis >= 7.0
```

### Clone and Setup

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=your_key_here
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET=your_webhook_secret

# LiveKit Video
LIVEKIT_API_KEY=your_livekit_key
LIVEKIT_API_SECRET=your_livekit_secret
LIVEKIT_WS_URL="wss://your-livekit-instance.com"

# AFIP (Argentina Tax Authority)
AFIP_CUIT=your_cuit_number
AFIP_CERT_PATH="./afip-cert.pem"
AFIP_PRIVATE_KEY_PATH="./afip-key.key"
AFIP_PRODUCTION=false

# Mercado Pago
MERCADOPAGO_ACCESS_TOKEN=your_mp_token
MERCADOPAGO_PUBLIC_KEY=your_mp_public_key

# Stripe
STRIPE_SECRET_KEY=your_stripe_secret
STRIPE_WEBHOOK_SECRET=your_stripe_webhook_secret

# App
JWT_SECRET=your_jwt_secret
FRONTEND_URL="http://localhost:5173"
BACKEND_URL="http://localhost:3000"
```

### Database Setup

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate dev

# Seed initial data (optional)
pnpm prisma db seed
```

### Run Development Servers

```bash
# Backend (NestJS)
cd backend
pnpm dev

# Frontend (SvelteKit)
cd frontend
pnpm dev
```

## Core Module Integration

### 1. Patient Management Module

**Prisma Schema (prisma/schema.prisma):**

```prisma
model Patient {
  id            String   @id @default(uuid())
  firstName     String
  lastName      String
  email         String?  @unique
  phone         String   @unique
  whatsappId    String?  @unique
  dateOfBirth   DateTime?
  status        PatientStatus @default(ACTIVE)
  
  appointments  Appointment[]
  clinicalNotes ClinicalNote[]
  invoices      Invoice[]
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  @@index([phone])
  @@index([whatsappId])
}

enum PatientStatus {
  PRIMER_CONTACTO
  ADMISION
  ACTIVO
  FINALIZADO
  SEGUIMIENTO
}
```

**NestJS Service (backend/src/patients/patients.service.ts):**

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { Patient, PatientStatus } from '@prisma/client';

@Injectable()
export class PatientsService {
  constructor(private prisma: PrismaService) {}

  async createPatient(data: {
    firstName: string;
    lastName: string;
    phone: string;
    email?: string;
  }): Promise<Patient> {
    return this.prisma.patient.create({
      data: {
        ...data,
        status: PatientStatus.PRIMER_CONTACTO,
      },
    });
  }

  async updatePatientStatus(
    patientId: string,
    status: PatientStatus,
  ): Promise<Patient> {
    return this.prisma.patient.update({
      where: { id: patientId },
      data: { status },
    });
  }

  async getPatientsByStatus(status: PatientStatus): Promise<Patient[]> {
    return this.prisma.patient.findMany({
      where: { status },
      include: {
        appointments: {
          orderBy: { scheduledAt: 'desc' },
          take: 5,
        },
      },
    });
  }
}
```

### 2. WhatsApp Automation with Baileys

**WhatsApp Service (backend/src/whatsapp/whatsapp.service.ts):**

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, { 
  DisconnectReason, 
  useMultiFileAuthState,
  WAMessage 
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  private async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions'
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      if (connection === 'close') {
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.connectToWhatsApp();
        }
      }
    });

    this.sock.ev.on('creds.update', saveCreds);
    this.sock.ev.on('messages.upsert', this.handleIncomingMessage.bind(this));
  }

  private async handleIncomingMessage({ messages }: { messages: WAMessage[] }) {
    const msg = messages[0];
    if (!msg.message || msg.key.fromMe) return;

    const text = msg.message.conversation || msg.message.extendedTextMessage?.text;
    const from = msg.key.remoteJid;

    // Crisis detection keywords (Spanish)
    const crisisKeywords = ['suicidio', 'matarme', 'no puedo más', 'quiero morir'];
    if (crisisKeywords.some(keyword => text?.toLowerCase().includes(keyword))) {
      await this.escalateCrisis(from, text);
    }

    // Parse appointment confirmations
    if (text?.toLowerCase().includes('confirmo') || text?.toLowerCase().includes('sí')) {
      await this.confirmAppointment(from);
    }
  }

  async sendAppointmentReminder(
    phoneNumber: string,
    appointmentDetails: {
      date: Date;
      practitionerName: string;
      type: string;
    }
  ): Promise<void> {
    const formattedNumber = phoneNumber.includes('@s.whatsapp.net') 
      ? phoneNumber 
      : `${phoneNumber}@s.whatsapp.net`;

    const message = `Hola! Recordatorio de tu sesión ${appointmentDetails.type} con ${appointmentDetails.practitionerName} el ${appointmentDetails.date.toLocaleDateString('es-AR')} a las ${appointmentDetails.date.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}.\n\nResponde "Confirmo" para confirmar tu asistencia.`;

    await this.sock.sendMessage(formattedNumber, { text: message });
  }

  async sendInvoice(phoneNumber: string, invoiceUrl: string): Promise<void> {
    const formattedNumber = `${phoneNumber}@s.whatsapp.net`;
    await this.sock.sendMessage(formattedNumber, {
      text: `Tu factura electrónica está lista. Podés descargarla desde: ${invoiceUrl}`,
    });
  }

  private async escalateCrisis(from: string, message: string) {
    // Alert practitioner immediately
    console.error(`CRISIS ALERT from ${from}: ${message}`);
    // Implementation: send push notification to practitioner
  }

  private async confirmAppointment(from: string) {
    // Implementation: update appointment status
  }
}
```

### 3. AFIP Electronic Invoicing

**AFIP Service (backend/src/billing/afip.service.ts):**

```typescript
import { Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as soap from 'soap';

interface AFIPAuthTicket {
  token: string;
  sign: string;
  expirationTime: Date;
}

@Injectable()
export class AFIPService {
  private authTicket: AFIPAuthTicket | null = null;
  private wsaaUrl = process.env.AFIP_PRODUCTION === 'true'
    ? 'https://wsaa.afip.gov.ar/ws/services/LoginCms'
    : 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms';

  async authenticate(): Promise<AFIPAuthTicket> {
    if (this.authTicket && this.authTicket.expirationTime > new Date()) {
      return this.authTicket;
    }

    const cert = fs.readFileSync(process.env.AFIP_CERT_PATH);
    const privateKey = fs.readFileSync(process.env.AFIP_PRIVATE_KEY_PATH);

    const client = await soap.createClientAsync(this.wsaaUrl);
    
    const tra = this.generateTRA();
    const cms = this.signTRA(tra, cert, privateKey);

    const response = await client.loginCmsAsync({ in0: cms });
    const credentials = this.parseCredentials(response);

    this.authTicket = credentials;
    return credentials;
  }

  async generateInvoice(data: {
    patientName: string;
    amount: number;
    sessionDate: Date;
    invoiceType: 'A' | 'B' | 'C' | 'M';
  }): Promise<{ cae: string; caeDueDate: Date; invoiceNumber: number }> {
    const auth = await this.authenticate();

    const wsfeUrl = process.env.AFIP_PRODUCTION === 'true'
      ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx'
      : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx';

    const client = await soap.createClientAsync(wsfeUrl);

    // Get last invoice number
    const lastInvoice = await client.FECompUltimoAutorizadoAsync({
      Auth: { Token: auth.token, Sign: auth.sign, Cuit: process.env.AFIP_CUIT },
      PtoVta: 1,
      CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
    });

    const nextInvoiceNumber = lastInvoice[0].FECompUltimoAutorizadoResult.CbteNro + 1;

    // Create invoice request
    const invoiceRequest = {
      Auth: { Token: auth.token, Sign: auth.sign, Cuit: process.env.AFIP_CUIT },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: 1,
          CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
        },
        FeDetReq: {
          FECAEDetRequest: [{
            Concepto: 2, // Services
            DocTipo: 99, // No document (for individuals)
            DocNro: 0,
            CbteDesde: nextInvoiceNumber,
            CbteHasta: nextInvoiceNumber,
            CbteFch: data.sessionDate.toISOString().split('T')[0].replace(/-/g, ''),
            ImpTotal: data.amount,
            ImpTotConc: 0,
            ImpNeto: data.amount,
            ImpOpEx: 0,
            ImpIVA: 0,
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
          }],
        },
      },
    };

    const result = await client.FECAESolicitarAsync(invoiceRequest);
    const invoice = result[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse[0];

    return {
      cae: invoice.CAE,
      caeDueDate: new Date(invoice.CAEFchVto),
      invoiceNumber: nextInvoiceNumber,
    };
  }

  private getInvoiceTypeCode(type: 'A' | 'B' | 'C' | 'M'): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type];
  }

  private generateTRA(): string {
    // Implementation: generate TRA (Ticket de Requerimiento de Acceso)
    return '';
  }

  private signTRA(tra: string, cert: Buffer, privateKey: Buffer): string {
    // Implementation: sign TRA with certificate
    return '';
  }

  private parseCredentials(response: any): AFIPAuthTicket {
    // Implementation: parse SOAP response
    return {
      token: '',
      sign: '',
      expirationTime: new Date(),
    };
  }
}
```

### 4. Claude AI Integration for Clinical Notes

**AI Service (backend/src/ai/claude.service.ts):**

```typescript
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private opusClient: Anthropic;
  private sonnetClient: Anthropic;

  constructor() {
    this.opusClient = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.sonnetClient = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalSession(sessionNotes: string): Promise<string> {
    // Use Opus for deep analysis
    const response = await this.opusClient.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 2000,
      messages: [
        {
          role: 'user',
          content: `Como asistente clínico de psicología en Argentina, resume la siguiente sesión en formato estructurado. Incluye: 1) Motivo de consulta, 2) Observaciones principales, 3) Intervenciones realizadas, 4) Plan para próxima sesión.\n\nNotas de sesión:\n${sessionNotes}`,
        },
      ],
    });

    return response.content[0].type === 'text' ? response.content[0].text : '';
  }

  async generateInsuranceReport(
    patientHistory: string[],
    sessionCount: number,
  ): Promise<string> {
    const combinedHistory = patientHistory.join('\n---\n');

    const response = await this.opusClient.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 3000,
      messages: [
        {
          role: 'user',
          content: `Genera un informe de evolución psicológica para obra social/prepaga basado en ${sessionCount} sesiones. Debe seguir estándares de salud mental argentinos.\n\nHistorial:\n${combinedHistory}`,
        },
      ],
    });

    return response.content[0].type === 'text' ? response.content[0].text : '';
  }

  async chatAssistant(query: string, context?: string): Promise<string> {
    // Use Sonnet for real-time assistance
    const response = await this.sonnetClient.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4',
      max_tokens: 1000,
      messages: [
        {
          role: 'user',
          content: context 
            ? `Contexto del paciente:\n${context}\n\nConsulta: ${query}`
            : query,
        },
      ],
    });

    return response.content[0].type === 'text' ? response.content[0].text : '';
  }

  async predictTherapeuticOutcome(
    patientData: {
      age: number;
      sessionCount: number;
      diagnosis: string;
      notes: string[];
    },
  ): Promise<{ riskLevel: string; recommendations: string[] }> {
    const prompt = `Analiza el siguiente caso clínico y proporciona:
1. Nivel de riesgo de abandono (bajo/medio/alto)
2. Recomendaciones terapéuticas específicas

Datos del paciente:
- Edad: ${patientData.age}
- Sesiones completadas: ${patientData.sessionCount}
- Diagnóstico: ${patientData.diagnosis}
- Notas recientes: ${patientData.notes.slice(-3).join('; ')}`;

    const response = await this.opusClient.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4',
      max_tokens: 1500,
      messages: [{ role: 'user', content: prompt }],
    });

    const analysis = response.content[0].type === 'text' ? response.content[0].text : '';
    
    // Parse structured response
    return {
      riskLevel: 'medio', // Parse from response
      recommendations: [], // Parse from response
    };
  }
}
```

### 5. LiveKit Video Consultation

**Video Service (backend/src/video/livekit.service.ts):**

```typescript
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  async createVideoRoomToken(
    roomName: string,
    participantName: string,
    isPractitioner: boolean,
  ): Promise<string> {
    const token = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: participantName,
        ttl: '2h',
      },
    );

    token.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: isPractitioner,
      recorder: isPractitioner, // Only practitioner can record
    });

    return token.toJwt();
  }

  async generateSessionRoom(appointmentId: string): Promise<{
    roomName: string;
    practitionerToken: string;
    patientToken: string;
  }> {
    const roomName = `session-${appointmentId}`;

    const practitionerToken = await this.createVideoRoomToken(
      roomName,
      'practitioner',
      true,
    );

    const patientToken = await this.createVideoRoomToken(
      roomName,
      'patient',
      false,
    );

    return { roomName, practitionerToken, patientToken };
  }
}
```

### 6. Appointment Scheduling Logic

**Appointments Service (backend/src/appointments/appointments.service.ts):**

```typescript
import { Injectable, BadRequestException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { Appointment, SessionType } from '@prisma/client';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async scheduleAppointment(data: {
    patientId: string;
    practitionerId: string;
    scheduledAt: Date;
    sessionType: SessionType;
    duration?: number;
  }): Promise<Appointment> {
    // Check for conflicts
    const conflicts = await this.checkConflicts(
      data.practitionerId,
      data.scheduledAt,
      data.duration || 45,
    );

    if (conflicts.length > 0) {
      throw new BadRequestException('Time slot already booked');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: data.patientId,
        practitionerId: data.practitionerId,
        scheduledAt: data.scheduledAt,
        sessionType: data.sessionType,
        duration: data.duration || 45,
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentReminder(
      appointment.patient.phone,
      {
        date: appointment.scheduledAt,
        practitionerName: `${appointment.practitioner.firstName} ${appointment.practitioner.lastName}`,
        type: appointment.sessionType.toLowerCase(),
      },
    );

    return appointment;
  }

  async checkConflicts(
    practitionerId: string,
    scheduledAt: Date,
    duration: number,
  ): Promise<Appointment[]> {
    const endTime = new Date(scheduledAt.getTime() + duration * 60000);

    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: { not: 'CANCELLED' },
        OR: [
          {
            scheduledAt: {
              gte: scheduledAt,
              lt: endTime,
            },
          },
          {
            AND: [
              { scheduledAt: { lte: scheduledAt } },
              {
                scheduledAt: {
                  gte: new Date(scheduledAt.getTime() - duration * 60000),
                },
              },
            ],
          },
        ],
      },
    });
  }

  async sendReminders(): Promise<void> {
    // Find appointments 24 hours from now
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        scheduledAt: {
          gte: new Date(),
          lte: tomorrow,
        },
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const apt of appointments) {
      await this.whatsapp.sendAppointmentReminder(apt.patient.phone, {
        date: apt.scheduledAt,
        practitionerName: `${apt.practitioner.firstName} ${apt.practitioner.lastName}`,
        type: apt.sessionType.toLowerCase(),
      });
    }
  }
}
```

## Frontend Integration (SvelteKit)

### Patient Dashboard Component

**frontend/src/routes/patients/[id]/+page.svelte:**

```svelte
<script lang="ts">
  import { page } from '$app/stores';
  import { onMount } from 'svelte';
  
  let patient: any = null;
  let appointments: any[] = [];
  let aiSummary: string = '';
  let loading = true;

  onMount(async () => {
    const patientId = $page.params.id;
    
    // Fetch patient data
    const res = await fetch(`/api/patients/${patientId}`);
    patient = await res.json();
    
    // Fetch appointments
    const aptsRes = await fetch(`/api/appointments?patientId=${patientId}`);
    appointments = await aptsRes.json();
    
    // Generate AI summary
    const summaryRes = await fetch(`/api/ai/patient-summary/${patientId}`);
    const data = await summaryRes.json();
    aiSummary = data.summary;
    
    loading = false;
  });

  async function scheduleAppointment() {
    // Implementation
  }

  async function sendWhatsApp() {
    await fetch('/api/whatsapp/send', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        phoneNumber: patient.phone,
        message: 'Hola! ¿Cómo estás?'
      })
    });
  }
</script>

<div class="container mx-auto p-6">
  {#if loading}
    <p>Cargando...</p>
  {:else if patient}
    <div class="bg-white rounded-lg shadow-md p-6">
      <h1 class="text-2xl font-bold mb-4">
        {patient.firstName} {patient.lastName}
      </h1>
      
      <div class="grid grid-cols-2 gap-4 mb-6">
        <div>
          <p class="text-sm text-gray-600">Estado</p>
          <span class="inline-block px-3 py-1 rounded-full bg-green-100 text-green-800">
            {patient.status}
          </span>
        </div>
        <div>
          <p class="text-sm text-gray-600">Teléfono</p>
          <p>{patient.phone}</p>
        </div>
      </div>

      <div class="mb-6">
        <h2 class="text-lg font-semibold mb-2">Resumen Clínico (IA)</h2>
        <div class="bg-blue-50 p-4 rounded">
          <p class="text-sm whitespace-pre-wrap">{aiSummary}</p>
        </div>
      </div>

      <div class="mb-6">
        <h2 class="text-lg font-semibold mb-2">Próximas Sesiones</h2>
        <div class="space-y-2">
          {#each appointments as apt}
            <div class="border-l-4 border-blue-500 pl-4 py-2">
              <p class="font-medium">
                {new Date(apt.scheduledAt).toLocaleDateString('es-AR')}
                {new Date(apt.scheduledAt).toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}
              </p>
              <p class="text-sm text-gray-600">{apt.sessionType}</p>
            </div>
          {/each}
        </div>
      </div>

      <div class="flex gap-4">
        <button 
          on:click={scheduleAppointment}
          class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        >
          Agendar Sesión
        </button>
        <button 
          on:click={sendWhatsApp}
          class="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700"
        >
          Enviar WhatsApp
        </button>
      </div>
    </div>
  {/if}
</div>
```

## Common Patterns

### Cron Jobs for Reminders

**backend/src/schedulers/reminders.scheduler.ts:**

```typescript
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { AppointmentsService } from '../appointments/appointments.service';

@Injectable()
export class RemindersScheduler {
  constructor(private appointments: AppointmentsService) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    await this.appointments.sendReminders();
  }

  @Cron(CronExpression.EVERY_30_MINUTES)
  async send2HourReminders() {
    // Implementation for 2-hour reminders
  }
}
```

### Multi-tenant Data Isolation

```typescript
// middleware/tenant.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class TenantMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const clinicId = req.headers['x-clinic-id'];
    req['clinicId'] = clinicId;
    next();
  }
}
```

## Troubleshooting

### WhatsApp Connection Issues

```bash
# Clear session and re-authenticate
rm -rf ./whatsapp-sessions/*
pnpm dev

# QR code will appear in terminal for scanning
```

### AFIP Certificate Errors

```bash
# Verify certificate validity
openssl x509 -in afip-cert.pem -text -noout

# Check key-cert match
openssl x509 -noout -modulus -in afip-cert.pem | openssl md5
openssl rsa -noout -modulus -in afip-key.key | openssl md5
```

### Database Migration Conflicts

```bash
# Reset database (development only
