---
name: sesion-mental-health-platform
description: AI-powered mental health practice management platform for psychologists in Argentina with intelligent scheduling, WhatsApp automation, AFIP billing, and video consultations
triggers:
  - how do I set up the Sesión clinic management platform
  - configure WhatsApp automation for patient appointments
  - integrate Claude AI models with Sesión
  - implement AFIP compliant invoicing in Sesión
  - set up video consultation infrastructure
  - create appointment scheduling workflows
  - configure Mercado Pago payments in Sesión
  - deploy Sesión microservices architecture
---

# Sesión Mental Health Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina that orchestrates appointment scheduling, WhatsApp automation (via Baileys), AFIP-compliant electronic invoicing, video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit 5 frontend, Prisma ORM, and deployed as microservices.

The platform handles the complete patient journey from first contact through treatment completion, with specialized modules for:
- Intelligent agenda management with conflict detection
- State-machine driven WhatsApp messaging
- Argentine tax compliance (Facturas A/B/C)
- WebRTC video sessions with bandwidth adaptation
- AI-powered clinical note summarization and patient insights

## Architecture Components

### Backend Services (NestJS)

```typescript
// Main orchestration service structure
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { PrismaModule } from './prisma/prisma.module';
import { AgendaModule } from './agenda/agenda.module';
import { WhatsAppModule } from './whatsapp/whatsapp.module';
import { BillingModule } from './billing/billing.module';
import { VideoModule } from './video/video.module';
import { AIModule } from './ai/ai.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),
    PrismaModule,
    AgendaModule,
    WhatsAppModule,
    BillingModule,
    VideoModule,
    AIModule,
  ],
})
export class AppModule {}
```

### Database Schema (Prisma)

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Patient {
  id            String        @id @default(cuid())
  firstName     String
  lastName      String
  email         String?       @unique
  phone         String        @unique
  whatsappOptIn Boolean       @default(true)
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt
  appointments  Appointment[]
  invoices      Invoice[]
  clinicalNotes ClinicalNote[]
}

model Appointment {
  id              String           @id @default(cuid())
  patientId       String
  practitionerId  String
  sessionType     SessionType
  startTime       DateTime
  endTime         DateTime
  status          AppointmentStatus @default(SCHEDULED)
  reminderSent24h Boolean          @default(false)
  reminderSent2h  Boolean          @default(false)
  videoRoomId     String?
  patient         Patient          @relation(fields: [patientId], references: [id])
  practitioner    Practitioner     @relation(fields: [practitionerId], references: [id])
  invoice         Invoice?
  
  @@index([practitionerId, startTime])
  @@index([patientId])
}

enum SessionType {
  PRESENCIAL
  VIRTUAL
  EVALUACION
}

enum AppointmentStatus {
  SCHEDULED
  CONFIRMED
  COMPLETED
  CANCELLED
  NO_SHOW
}

model Invoice {
  id              String      @id @default(cuid())
  appointmentId   String      @unique
  patientId       String
  amount          Decimal     @db.Decimal(10, 2)
  afipType        AFIPType
  afipCAE         String?
  afipCAEExpiry   DateTime?
  paymentMethod   PaymentMethod
  paymentStatus   PaymentStatus @default(PENDING)
  issuedAt        DateTime    @default(now())
  appointment     Appointment @relation(fields: [appointmentId], references: [id])
  patient         Patient     @relation(fields: [patientId], references: [id])
}

enum AFIPType {
  A
  B
  C
  M
}

enum PaymentMethod {
  EFECTIVO
  TRANSFERENCIA
  MERCADO_PAGO
  TARJETA
}

enum PaymentStatus {
  PENDING
  PAID
  OVERDUE
  CANCELLED
}

model Practitioner {
  id           String        @id @default(cuid())
  firstName    String
  lastName     String
  email        String        @unique
  licenseNumber String       @unique
  afipCUIT     String
  appointments Appointment[]
}

model ClinicalNote {
  id              String   @id @default(cuid())
  patientId       String
  practitionerId  String
  content         String   @db.Text
  aiSummary       String?  @db.Text
  createdAt       DateTime @default(now())
  patient         Patient  @relation(fields: [patientId], references: [id])
  
  @@index([patientId, createdAt])
}
```

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 18.0.0
npm >= 9.0.0
docker >= 24.0.0
docker-compose >= 2.0.0
```

### Local Development Setup

```bash
# Clone repository
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install

# Setup environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# backend/.env
DATABASE_URL="postgresql://postgres:password@localhost:5432/sesion?schema=public"
REDIS_URL="redis://localhost:6379"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"

# AFIP Integration
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certificates/afip.crt"
AFIP_KEY_PATH="./certificates/afip.key"
AFIP_PRODUCTION="false"

# Mercado Pago
MERCADO_PAGO_ACCESS_TOKEN="${MERCADO_PAGO_ACCESS_TOKEN}"
MERCADO_PAGO_PUBLIC_KEY="${MERCADO_PAGO_PUBLIC_KEY}"

# LiveKit (Video)
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_WS_URL="wss://sesion.livekit.cloud"

# Anthropic AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# JWT
JWT_SECRET="${JWT_SECRET}"
JWT_EXPIRATION="24h"

# Email (optional)
SMTP_HOST="smtp.gmail.com"
SMTP_PORT="587"
SMTP_USER="${SMTP_USER}"
SMTP_PASSWORD="${SMTP_PASSWORD}"
```

### Database Initialization

```bash
# Generate Prisma client
cd backend
npx prisma generate

# Run migrations
npx prisma migrate dev --name init

# Seed initial data (optional)
npx prisma db seed
```

### Running Services

```bash
# Start infrastructure with Docker Compose
docker-compose up -d postgres redis elasticsearch minio kafka

# Start backend (NestJS)
cd backend
npm run start:dev

# Start frontend (SvelteKit)
cd frontend
npm run dev
```

## Core Module Implementation

### Agenda Management Service

```typescript
// backend/src/agenda/agenda.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { Appointment, SessionType } from '@prisma/client';
import { addMinutes, isWithinInterval } from 'date-fns';

interface CreateAppointmentDto {
  patientId: string;
  practitionerId: string;
  sessionType: SessionType;
  startTime: Date;
}

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(dto: CreateAppointmentDto): Promise<Appointment> {
    // Calculate end time based on session type
    const duration = this.getSessionDuration(dto.sessionType);
    const endTime = addMinutes(dto.startTime, duration);

    // Check for conflicts
    await this.checkConflicts(dto.practitionerId, dto.startTime, endTime);

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: dto.patientId,
        practitionerId: dto.practitionerId,
        sessionType: dto.sessionType,
        startTime: dto.startTime,
        endTime: endTime,
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    return appointment;
  }

  private getSessionDuration(sessionType: SessionType): number {
    const durations = {
      PRESENCIAL: 45,
      VIRTUAL: 45,
      EVALUACION: 90,
    };
    return durations[sessionType] || 45;
  }

  private async checkConflicts(
    practitionerId: string,
    startTime: Date,
    endTime: Date,
  ): Promise<void> {
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: { notIn: ['CANCELLED', 'COMPLETED'] },
        OR: [
          {
            startTime: { lte: startTime },
            endTime: { gt: startTime },
          },
          {
            startTime: { lt: endTime },
            endTime: { gte: endTime },
          },
          {
            startTime: { gte: startTime },
            endTime: { lte: endTime },
          },
        ],
      },
    });

    if (conflicts.length > 0) {
      throw new ConflictException('Practitioner has conflicting appointments');
    }
  }

  async findAvailableSlots(
    practitionerId: string,
    date: Date,
    sessionType: SessionType,
  ): Promise<Date[]> {
    const duration = this.getSessionDuration(sessionType);
    const workingHours = { start: 9, end: 19 }; // 9 AM to 7 PM
    const slots: Date[] = [];

    // Get existing appointments for the day
    const existingAppointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: {
          gte: new Date(date.setHours(0, 0, 0, 0)),
          lt: new Date(date.setHours(23, 59, 59, 999)),
        },
        status: { notIn: ['CANCELLED'] },
      },
    });

    // Generate potential slots
    for (let hour = workingHours.start; hour < workingHours.end; hour++) {
      for (let minute = 0; minute < 60; minute += 15) {
        const slotStart = new Date(date);
        slotStart.setHours(hour, minute, 0, 0);
        const slotEnd = addMinutes(slotStart, duration);

        // Check if slot conflicts with existing appointments
        const hasConflict = existingAppointments.some((apt) =>
          isWithinInterval(slotStart, {
            start: apt.startTime,
            end: apt.endTime,
          }) ||
          isWithinInterval(slotEnd, {
            start: apt.startTime,
            end: apt.endTime,
          }),
        );

        if (!hasConflict && slotEnd.getHours() <= workingHours.end) {
          slots.push(slotStart);
        }
      }
    }

    return slots;
  }
}
```

### WhatsApp Automation (Baileys)

```typescript
// backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '../prisma/prisma.service';
import { subHours, isBefore } from 'date-fns';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  private async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions',
    );

    this.sock = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.sock.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      if (connection === 'close') {
        const shouldReconnect =
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.connectToWhatsApp();
        }
      }
    });

    this.sock.ev.on('creds.update', saveCreds);
    this.sock.ev.on('messages.upsert', this.handleIncomingMessage.bind(this));
  }

  async sendAppointmentReminder(appointmentId: string): Promise<void> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment || !appointment.patient.whatsappOptIn) {
      return;
    }

    const message = `Hola ${appointment.patient.firstName}! 👋\n\nTe recordamos tu sesión con ${appointment.practitioner.firstName} ${appointment.practitioner.lastName}.\n\n📅 Fecha: ${appointment.startTime.toLocaleDateString('es-AR')}\n🕐 Hora: ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n\n¿Podés confirmar tu asistencia? Responde SI para confirmar.`;

    await this.sendMessage(appointment.patient.phone, message);
  }

  async sendInvoice(invoiceId: string): Promise<void> {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id: invoiceId },
      include: { patient: true, appointment: true },
    });

    if (!invoice || !invoice.patient.whatsappOptIn) {
      return;
    }

    const message = `Hola ${invoice.patient.firstName}! 📄\n\nTu factura de la sesión del ${invoice.appointment.startTime.toLocaleDateString('es-AR')} está lista.\n\n💰 Monto: $${invoice.amount.toString()}\n📋 CAE: ${invoice.afipCAE}\n\nPodés abonar por:\n- Transferencia\n- Mercado Pago\n- Efectivo en próxima sesión`;

    await this.sendMessage(invoice.patient.phone, message);
  }

  private async sendMessage(phone: string, text: string): Promise<void> {
    const jid = `${phone.replace(/\D/g, '')}@s.whatsapp.net`;
    await this.sock.sendMessage(jid, { text });
  }

  private async handleIncomingMessage(m: any) {
    const message = m.messages[0];
    if (!message.message) return;

    const from = message.key.remoteJid;
    const text = message.message.conversation || '';
    const phone = from.split('@')[0];

    // Check for confirmation responses
    if (text.toLowerCase().includes('si') || text.toLowerCase().includes('confirmo')) {
      await this.handleConfirmation(phone);
    }

    // Crisis detection keywords
    const crisisKeywords = ['crisis', 'emergencia', 'suicidio', 'ayuda urgente'];
    if (crisisKeywords.some((keyword) => text.toLowerCase().includes(keyword))) {
      await this.handleCrisisEscalation(phone, text);
    }
  }

  private async handleConfirmation(phone: string): Promise<void> {
    const appointment = await this.prisma.appointment.findFirst({
      where: {
        patient: { phone },
        status: 'SCHEDULED',
        startTime: { gte: new Date() },
      },
      orderBy: { startTime: 'asc' },
    });

    if (appointment) {
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { status: 'CONFIRMED' },
      });

      await this.sendMessage(
        phone,
        '✅ Perfecto! Tu sesión está confirmada. Nos vemos pronto!',
      );
    }
  }

  private async handleCrisisEscalation(phone: string, message: string): Promise<void> {
    const patient = await this.prisma.patient.findUnique({
      where: { phone },
    });

    if (patient) {
      // Send immediate response to patient
      await this.sendMessage(
        phone,
        '⚠️ Recibimos tu mensaje. Un profesional se contactará contigo lo antes posible.\n\nSi necesitás asistencia inmediata, comunicate al:\n🆘 135 (Centro de Asistencia al Suicida)\n🆘 (011) 5275-1135',
      );

      // Alert practitioner (implementation depends on notification system)
      // Could send email, SMS, or push notification
      console.log(`CRISIS ALERT: Patient ${patient.id} - ${message}`);
    }
  }
}
```

### AFIP Billing Integration

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { AFIPType, Invoice } from '@prisma/client';
import * as fs from 'fs';
import * as soap from 'soap';

interface AFIPResponse {
  CAE: string;
  CAEFchVto: string;
}

@Injectable()
export class AFIPService {
  private wsdlUrl = process.env.AFIP_PRODUCTION === 'true'
    ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx?WSDL'
    : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx?WSDL';

  constructor(private prisma: PrismaService) {}

  async generateInvoice(
    appointmentId: string,
    amount: number,
    afipType: AFIPType,
  ): Promise<Invoice> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    if (!appointment) {
      throw new Error('Appointment not found');
    }

    // Get authentication token
    const token = await this.getAuthToken();

    // Request CAE from AFIP
    const afipResponse = await this.requestCAE({
      token,
      cuit: appointment.practitioner.afipCUIT,
      invoiceType: this.mapAFIPType(afipType),
      amount,
    });

    // Create invoice record
    const invoice = await this.prisma.invoice.create({
      data: {
        appointmentId,
        patientId: appointment.patientId,
        amount,
        afipType,
        afipCAE: afipResponse.CAE,
        afipCAEExpiry: new Date(afipResponse.CAEFchVto),
        paymentMethod: 'TRANSFERENCIA',
        paymentStatus: 'PENDING',
      },
    });

    return invoice;
  }

  private async getAuthToken(): Promise<string> {
    // Read AFIP certificate and private key
    const cert = fs.readFileSync(process.env.AFIP_CERT_PATH);
    const key = fs.readFileSync(process.env.AFIP_KEY_PATH);

    const client = await soap.createClientAsync(
      'https://wsaahomo.afip.gov.ar/ws/services/LoginCms?wsdl',
    );

    const ticket = await this.generateTicket();
    const response = await client.loginCmsAsync({
      in0: ticket,
    });

    return response[0].loginCmsReturn.token;
  }

  private async requestCAE(params: {
    token: string;
    cuit: string;
    invoiceType: number;
    amount: number;
  }): Promise<AFIPResponse> {
    const client = await soap.createClientAsync(this.wsdlUrl);

    const lastInvoice = await this.getLastInvoiceNumber(params.cuit);
    const invoiceNumber = lastInvoice + 1;

    const response = await client.FECAESolicitarAsync({
      Auth: { Token: params.token, Sign: '', Cuit: params.cuit },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: 1,
          CbteTipo: params.invoiceType,
        },
        FeDetReq: {
          FECAEDetRequest: {
            Concepto: 2, // Services
            DocTipo: 99, // Other
            DocNro: 0,
            CbteDesde: invoiceNumber,
            CbteHasta: invoiceNumber,
            CbteFch: new Date().toISOString().split('T')[0].replace(/-/g, ''),
            ImpTotal: params.amount,
            ImpTotConc: 0,
            ImpNeto: params.amount,
            ImpOpEx: 0,
            ImpIVA: 0,
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
          },
        },
      },
    });

    const result = response[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse[0];

    if (result.Resultado !== 'A') {
      throw new Error(`AFIP error: ${result.Observaciones?.[0]?.Msg}`);
    }

    return {
      CAE: result.CAE,
      CAEFchVto: result.CAEFchVto,
    };
  }

  private mapAFIPType(type: AFIPType): number {
    const mapping = {
      A: 1,
      B: 6,
      C: 11,
      M: 51,
    };
    return mapping[type];
  }

  private async getLastInvoiceNumber(cuit: string): Promise<number> {
    const lastInvoice = await this.prisma.invoice.findFirst({
      where: { appointment: { practitioner: { afipCUIT: cuit } } },
      orderBy: { issuedAt: 'desc' },
    });

    // Extract invoice number from CAE or return 0
    return lastInvoice ? 1000 : 0; // Simplified - actual implementation would parse from AFIP
  }

  private generateTicket(): string {
    // Generate WSAA ticket request (simplified)
    const now = new Date();
    const expiration = new Date(now.getTime() + 12 * 60 * 60 * 1000);

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
}
```

### AI Orchestration (Claude)

```typescript
// backend/src/ai/ai.service.ts
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

  async summarizeClinicalNote(noteId: string): Promise<string> {
    const note = await this.prisma.clinicalNote.findUnique({
      where: { id: noteId },
      include: { patient: true },
    });

    if (!note) {
      throw new Error('Clinical note not found');
    }

    // Use Opus for deep analysis
    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Sos un psicólogo clínico argentino especializado en resumir notas de sesión. Tu tarea es crear un resumen estructurado y conciso de la siguiente nota clínica.

Nota de sesión:
${note.content}

Genera un resumen que incluya:
1. Motivo de consulta principal
2. Observaciones clínicas relevantes
3. Intervenciones realizadas
4. Plan de acción para próximas sesiones

El resumen debe ser profesional, mantener confidencialidad, y usar terminología clínica apropiada. Responde en español argentino.`,
        },
      ],
    });

    const summary = message.content[0].type === 'text' ? message.content[0].text : '';

    // Update note with AI summary
    await this.prisma.clinicalNote.update({
      where: { id: noteId },
      data: { aiSummary: summary },
    });

    return summary;
  }

  async analyzePatientRisk(patientId: string): Promise<{
    riskLevel: 'low' | 'medium' | 'high';
    insights: string;
  }> {
    // Get patient history
    const notes = await this.prisma.clinicalNote.findMany({
      where: { patientId },
      orderBy: { createdAt: 'desc' },
      take: 20,
    });

    const appointments = await this.prisma.appointment.findMany({
      where: { patientId },
      orderBy: { startTime: 'desc' },
      take: 10,
    });

    // Calculate attendance rate
    const totalAppointments = appointments.length;
    const completedAppointments = appointments.filter(
      (a) => a.status === 'COMPLETED',
    ).length;
    const attendanceRate = completedAppointments / totalAppointments;

    const notesContext = notes
      .slice(0, 5)
      .map((n) => n.aiSummary || n.content.substring(0, 500))
      .join('\n\n---\n\n');

    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `Sos un psicólogo clínico especializado en evaluación de riesgo. Analiza el siguiente contexto del paciente y proporciona una evaluación de riesgo.

Tasa de asistencia: ${(attendanceRate * 100).toFixed(1)}%
Cantidad de sesiones: ${totalAppointments}

Últimas notas clínicas:
${notesContext}

Proporciona:
1. Nivel de riesgo (bajo/medio/alto)
2. Factores de riesgo identificados
3. Recomendaciones de seguimiento

IMPORTANTE: Esta es una herramienta de apoyo. Toda decisión clínica debe ser tomada por el profesional tratante. Responde en español argentino.`,
        },
      ],
    });

    const insights = message.content[0].type === 'text' ? message.content[0].text : '';

    // Simple heuristic for risk level
    let riskLevel: 'low' | 'medium' | 'high' = 'low';
    if (attendanceRate < 0.5 || notes.length < 3) {
      riskLevel = 'medium';
    }
    if (insights.toLowerCase().includes('alto riesgo') || attendanceRate < 0.3) {
      riskLevel = 'high';
    }

    return { riskLevel, insights };
  }

  async generateSessionReport(appointmentId: string): Promise<string> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    const note = await this.prisma.clinicalNote.findFirst({
      where: {
        patientId: appointment.patientId,
        createdAt: { gte: appointment.startTime },
      },
    });

    // Use Sonnet for faster interactive report generation
    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Genera un informe de sesión estructurado para presentar a obras sociales o prepagas.

Fecha: ${appointment.startTime.toLocaleDateString('es-AR')}
Tipo: ${appointment.sessionType}
Paciente: ${appointment.patient.firstName} ${appointment.patient.lastName}
Profesional: ${appointment.practitioner.firstName} ${appointment.practitioner.lastName}

Nota clínica:
${note?.content || 'No disponible'}

El
