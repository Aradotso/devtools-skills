---
name: sesion-mental-health-clinic-platform
description: Orchestrate psychology clinic operations with intelligent scheduling, WhatsApp automation, AFIP-compliant billing, secure video consultations, and Claude AI integration for Argentine mental health practices.
triggers:
  - set up sesion clinic management platform
  - integrate whatsapp automation for patient appointments
  - configure afip electronic invoicing for psychology practice
  - implement claude ai clinical note summarization
  - build patient scheduling with sesion workflow
  - add video consultation system for mental health
  - create intelligent agenda for psychology clinic
  - automate appointment reminders via whatsapp
---

# Sesion Mental Health Clinic Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for Argentine psychology clinics that unifies appointment scheduling, WhatsApp automation, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance. Built with NestJS backend, SvelteKit frontend, Prisma ORM, and Anthropic Claude models (Opus 4.6 + Sonnet 4.6).

## Architecture Overview

The platform uses a microservices architecture with:
- **Backend**: NestJS with TypeScript
- **Frontend**: SvelteKit 5 + TailwindCSS
- **Database**: PostgreSQL via Prisma
- **Cache/Queue**: Redis
- **WhatsApp**: Baileys library for automation
- **Video**: LiveKit for WebRTC consultations
- **AI**: Anthropic Claude (Opus 4.6 for deep analysis, Sonnet 4.6 for real-time)
- **Payments**: MercadoPago (AR) + Stripe (international)

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 20.x
pnpm >= 8.x
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
# .env file structure
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="sk-ant-xxx"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./sessions/whatsapp"
WHATSAPP_WEBHOOK_SECRET="your-webhook-secret"

# LiveKit Video
LIVEKIT_API_KEY="APIxxx"
LIVEKIT_API_SECRET="secretxxx"
LIVEKIT_WS_URL="wss://your-instance.livekit.cloud"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="20-12345678-9"
AFIP_CERT_PATH="./certs/afip-cert.crt"
AFIP_KEY_PATH="./certs/afip-key.key"
AFIP_ENVIRONMENT="homologacion" # or "produccion"

# Payment Gateways
MERCADOPAGO_ACCESS_TOKEN="APP_USR-xxx"
STRIPE_SECRET_KEY="sk_test_xxx"

# Application
JWT_SECRET="your-jwt-secret"
APP_URL="http://localhost:3000"
```

### Database Setup

```bash
# Run Prisma migrations
pnpm prisma migrate dev

# Generate Prisma client
pnpm prisma generate

# Seed initial data
pnpm prisma db seed
```

### Run Development Servers

```bash
# Backend (NestJS)
pnpm run dev:backend

# Frontend (SvelteKit)
pnpm run dev:frontend

# Both concurrently
pnpm run dev
```

## Core Domain Models (Prisma Schema)

### Patient Entity

```prisma
model Patient {
  id            String   @id @default(cuid())
  firstName     String
  lastName      String
  email         String?  @unique
  phone         String   @unique
  whatsappOptIn Boolean  @default(true)
  
  dateOfBirth   DateTime?
  document      String?  @unique // DNI in Argentina
  healthInsurance String?
  
  status        PatientStatus @default(ACTIVE)
  stage         PatientStage  @default(PRIMER_CONTACTO)
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  appointments  Appointment[]
  invoices      Invoice[]
  clinicalNotes ClinicalNote[]
  
  @@index([phone])
  @@index([status, stage])
}

enum PatientStatus {
  ACTIVE
  INACTIVE
  ARCHIVED
}

enum PatientStage {
  PRIMER_CONTACTO
  ADMISION
  SESION_ACTIVA
  FINALIZACION
  SEGUIMIENTO
}
```

### Appointment Entity

```prisma
model Appointment {
  id            String   @id @default(cuid())
  
  patientId     String
  patient       Patient  @relation(fields: [patientId], references: [id])
  
  practitionerId String
  practitioner  Practitioner @relation(fields: [practitionerId], references: [id])
  
  startTime     DateTime
  endTime       DateTime
  duration      Int      @default(45) // minutes
  
  type          AppointmentType
  modality      SessionModality
  
  status        AppointmentStatus @default(SCHEDULED)
  
  roomId        String?
  videoRoomId   String?  // LiveKit room ID
  
  reminderSent24h Boolean @default(false)
  reminderSent2h  Boolean @default(false)
  
  notes         String?
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  invoice       Invoice?
  clinicalNote  ClinicalNote?
  
  @@index([patientId, startTime])
  @@index([practitionerId, startTime])
  @@index([status, startTime])
}

enum AppointmentType {
  INDIVIDUAL
  COUPLE
  FAMILY
  EVALUATION
  FOLLOW_UP
}

enum SessionModality {
  PRESENCIAL
  VIRTUAL
  HYBRID
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

### Invoice Entity (AFIP Compliant)

```prisma
model Invoice {
  id            String   @id @default(cuid())
  
  invoiceNumber String   @unique
  afipCAE       String?  // AFIP Electronic Authorization Code
  afipCAEExpiry DateTime?
  
  patientId     String
  patient       Patient  @relation(fields: [patientId], references: [id])
  
  appointmentId String?  @unique
  appointment   Appointment? @relation(fields: [appointmentId], references: [id])
  
  type          InvoiceType
  fiscalCategory FiscalCategory
  
  subtotal      Decimal  @db.Decimal(10, 2)
  tax           Decimal  @db.Decimal(10, 2) @default(0)
  total         Decimal  @db.Decimal(10, 2)
  
  currency      String   @default("ARS")
  
  status        InvoiceStatus @default(PENDING)
  
  paymentMethod PaymentMethod?
  paidAt        DateTime?
  
  sentViaWhatsApp Boolean @default(false)
  whatsappSentAt  DateTime?
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  @@index([patientId, status])
  @@index([createdAt])
}

enum InvoiceType {
  A  // Responsable Inscripto
  B  // Consumidor Final
  C  // Exempt
  M  // Monotributo
}

enum FiscalCategory {
  RESPONSABLE_INSCRIPTO
  MONOTRIBUTO
  EXENTO
}

enum InvoiceStatus {
  PENDING
  SENT
  PAID
  OVERDUE
  CANCELLED
}

enum PaymentMethod {
  EFECTIVO
  TRANSFERENCIA
  MERCADOPAGO
  TARJETA
  STRIPE
}
```

## NestJS Backend Services

### Appointment Scheduling Service

```typescript
// apps/backend/src/appointments/appointments.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateAppointmentDto } from './dto/create-appointment.dto';
import { AppointmentStatus } from '@prisma/client';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async create(dto: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflict = await this.prisma.appointment.findFirst({
      where: {
        practitionerId: dto.practitionerId,
        status: { not: AppointmentStatus.CANCELLED },
        OR: [
          {
            AND: [
              { startTime: { lte: dto.startTime } },
              { endTime: { gt: dto.startTime } },
            ],
          },
          {
            AND: [
              { startTime: { lt: dto.endTime } },
              { endTime: { gte: dto.endTime } },
            ],
          },
        ],
      },
    });

    if (conflict) {
      throw new ConflictException(
        'Practitioner has conflicting appointment at this time'
      );
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: dto.patientId,
        practitionerId: dto.practitionerId,
        startTime: dto.startTime,
        endTime: dto.endTime,
        duration: dto.duration || 45,
        type: dto.type,
        modality: dto.modality,
        status: AppointmentStatus.SCHEDULED,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    return appointment;
  }

  async findAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number = 45
  ) {
    const startOfDay = new Date(date);
    startOfDay.setHours(0, 0, 0, 0);
    
    const endOfDay = new Date(date);
    endOfDay.setHours(23, 59, 59, 999);

    // Get practitioner's schedule for the day
    const bookedAppointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: startOfDay, lte: endOfDay },
        status: { not: AppointmentStatus.CANCELLED },
      },
      orderBy: { startTime: 'asc' },
    });

    // Generate available slots (example: 9 AM to 7 PM)
    const workStart = new Date(date);
    workStart.setHours(9, 0, 0, 0);
    
    const workEnd = new Date(date);
    workEnd.setHours(19, 0, 0, 0);

    const slots: Date[] = [];
    let currentSlot = new Date(workStart);

    while (currentSlot < workEnd) {
      const slotEnd = new Date(currentSlot.getTime() + duration * 60000);
      
      // Check if slot overlaps with booked appointments
      const hasConflict = bookedAppointments.some(apt => {
        return (
          (currentSlot >= apt.startTime && currentSlot < apt.endTime) ||
          (slotEnd > apt.startTime && slotEnd <= apt.endTime)
        );
      });

      if (!hasConflict) {
        slots.push(new Date(currentSlot));
      }

      currentSlot = new Date(currentSlot.getTime() + duration * 60000);
    }

    return slots;
  }

  async confirmAppointment(id: string) {
    return this.prisma.appointment.update({
      where: { id },
      data: { status: AppointmentStatus.CONFIRMED },
    });
  }
}
```

### WhatsApp Automation Service (Baileys)

```typescript
// apps/backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.connectWhatsApp();
  }

  async connectWhatsApp() {
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
        const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.connectWhatsApp();
        }
      } else if (connection === 'open') {
        console.log('WhatsApp connected successfully');
      }
    });

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async handleIncomingMessage(message: WAMessage) {
    if (!message.message || message.key.fromMe) return;

    const phone = message.key.remoteJid?.replace('@s.whatsapp.net', '');
    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text || '';

    // Check for crisis keywords
    const crisisKeywords = ['suicidio', 'no quiero vivir', 'lastimar', 'terminar con todo'];
    const isCrisis = crisisKeywords.some(keyword => 
      text.toLowerCase().includes(keyword)
    );

    if (isCrisis) {
      await this.alertPractitionerCrisis(phone, text);
    }

    // Check if patient exists
    const patient = await this.prisma.patient.findUnique({
      where: { phone },
    });

    if (patient) {
      // Handle appointment confirmation replies
      if (text.toLowerCase().includes('confirmo') || text.toLowerCase() === 'sí') {
        await this.handleAppointmentConfirmation(patient.id);
      }
    }
  }

  async sendAppointmentReminder(appointmentId: string, hoursBeforehand: number) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    if (!appointment || !appointment.patient.whatsappOptIn) return;

    const message = hoursBeforehand === 24
      ? `Hola ${appointment.patient.firstName}! 👋\n\nTe recordamos que tenés una sesión mañana a las ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })} con ${appointment.practitioner.firstName}.\n\nPor favor, respondé CONFIRMO si vas a asistir.`
      : `Hola ${appointment.patient.firstName}! 🕐\n\nTu sesión con ${appointment.practitioner.firstName} es en 2 horas (${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}).\n\nNos vemos pronto!`;

    await this.sendMessage(appointment.patient.phone, message);

    // Update reminder status
    const updateData = hoursBeforehand === 24
      ? { reminderSent24h: true }
      : { reminderSent2h: true };

    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: updateData,
    });
  }

  async sendInvoice(invoiceId: string) {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id: invoiceId },
      include: {
        patient: true,
        appointment: true,
      },
    });

    if (!invoice || !invoice.patient.whatsappOptIn) return;

    const message = `Hola ${invoice.patient.firstName}! 📄\n\nTe enviamos la factura ${invoice.invoiceNumber} por la sesión del ${invoice.appointment?.startTime.toLocaleDateString('es-AR')}.\n\n💰 Total: $${invoice.total}\n\nPodés pagar mediante transferencia o Mercado Pago.\n\nGracias!`;

    await this.sendMessage(invoice.patient.phone, message);

    // Generate and send PDF (implementation omitted)
    // await this.sendDocument(invoice.patient.phone, pdfBuffer, 'factura.pdf');

    await this.prisma.invoice.update({
      where: { id: invoiceId },
      data: {
        sentViaWhatsApp: true,
        whatsappSentAt: new Date(),
      },
    });
  }

  async sendMessage(phone: string, text: string) {
    const jid = `${phone}@s.whatsapp.net`;
    await this.sock.sendMessage(jid, { text });
  }

  private async alertPractitionerCrisis(patientPhone: string, messageText: string) {
    const patient = await this.prisma.patient.findUnique({
      where: { phone: patientPhone },
      include: {
        appointments: {
          where: { status: { not: 'CANCELLED' } },
          include: { practitioner: true },
          orderBy: { startTime: 'desc' },
          take: 1,
        },
      },
    });

    if (patient?.appointments[0]?.practitioner.phone) {
      const alert = `🚨 ALERTA DE CRISIS 🚨\n\nPaciente: ${patient.firstName} ${patient.lastName}\nMensaje: "${messageText}"\n\nPor favor, contactar de inmediato.`;
      
      await this.sendMessage(patient.appointments[0].practitioner.phone, alert);
    }
  }

  private async handleAppointmentConfirmation(patientId: string) {
    const nextAppointment = await this.prisma.appointment.findFirst({
      where: {
        patientId,
        startTime: { gte: new Date() },
        status: 'SCHEDULED',
      },
      orderBy: { startTime: 'asc' },
    });

    if (nextAppointment) {
      await this.prisma.appointment.update({
        where: { id: nextAppointment.id },
        data: { status: 'CONFIRMED' },
      });
    }
  }
}
```

### AFIP Electronic Invoicing Service

```typescript
// apps/backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import axios from 'axios';
import { parseString } from 'xml2js';
import { promisify } from 'util';

const parseXML = promisify(parseString);

@Injectable()
export class AfipService {
  private readonly wsUrl = process.env.AFIP_ENVIRONMENT === 'produccion'
    ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx'
    : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx';

  async getAuthToken(): Promise<{ token: string; sign: string }> {
    // Load certificate and key
    const cert = readFileSync(process.env.AFIP_CERT_PATH, 'utf8');
    const key = readFileSync(process.env.AFIP_KEY_PATH, 'utf8');

    // Generate WSAA authentication request (simplified - real impl needs XML signing)
    const loginTicketRequest = `<?xml version="1.0" encoding="UTF-8"?>
<loginTicketRequest version="1.0">
  <header>
    <uniqueId>${Date.now()}</uniqueId>
    <generationTime>${new Date().toISOString()}</generationTime>
    <expirationTime>${new Date(Date.now() + 12 * 60 * 60 * 1000).toISOString()}</expirationTime>
  </header>
  <service>wsfe</service>
</loginTicketRequest>`;

    // In production, sign this XML with the certificate
    // For this example, we'll return a simplified structure
    // Real implementation requires xmldsigjs or similar library

    return { token: 'generated-token', sign: 'generated-signature' };
  }

  async authorizeInvoice(invoiceData: {
    type: 'A' | 'B' | 'C' | 'M';
    concept: number;
    documentType: number;
    documentNumber: string;
    dateFrom: string;
    dateTo: string;
    amount: number;
    tax: number;
    total: number;
  }): Promise<{ cae: string; caeExpiry: string }> {
    const auth = await this.getAuthToken();

    const requestXML = `<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <FECAESolicitar xmlns="http://ar.gov.afip.dif.FEV1/">
      <Auth>
        <Token>${auth.token}</Token>
        <Sign>${auth.sign}</Sign>
        <Cuit>${process.env.AFIP_CUIT}</Cuit>
      </Auth>
      <FeCAEReq>
        <FeCabReq>
          <CantReg>1</CantReg>
          <PtoVta>1</PtoVta>
          <CbteTipo>${this.getInvoiceTypeCode(invoiceData.type)}</CbteTipo>
        </FeCabReq>
        <FeDetReq>
          <FECAEDetRequest>
            <Concepto>${invoiceData.concept}</Concepto>
            <DocTipo>${invoiceData.documentType}</DocTipo>
            <DocNro>${invoiceData.documentNumber}</DocNro>
            <CbteDesde>1</CbteDesde>
            <CbteHasta>1</CbteHasta>
            <CbteFch>${invoiceData.dateFrom}</CbteFch>
            <ImpTotal>${invoiceData.total}</ImpTotal>
            <ImpTotConc>0</ImpTotConc>
            <ImpNeto>${invoiceData.amount}</ImpNeto>
            <ImpOpEx>0</ImpOpEx>
            <ImpIVA>${invoiceData.tax}</ImpIVA>
            <ImpTrib>0</ImpTrib>
            <MonId>PES</MonId>
            <MonCotiz>1</MonCotiz>
          </FECAEDetRequest>
        </FeDetReq>
      </FeCAEReq>
    </FECAESolicitar>
  </soap:Body>
</soap:Envelope>`;

    try {
      const response = await axios.post(this.wsUrl, requestXML, {
        headers: { 'Content-Type': 'text/xml' },
      });

      const parsed = await parseXML(response.data);
      const result = parsed['soap:Envelope']['soap:Body'][0]['FECAESolicitarResponse'][0]['FECAESolicitarResult'][0];

      if (result.FeDetResp[0].FECAEDetResponse[0].Resultado[0] === 'A') {
        return {
          cae: result.FeDetResp[0].FECAEDetResponse[0].CAE[0],
          caeExpiry: result.FeDetResp[0].FECAEDetResponse[0].CAEFchVto[0],
        };
      } else {
        throw new Error('AFIP authorization failed');
      }
    } catch (error) {
      console.error('AFIP error:', error);
      throw error;
    }
  }

  private getInvoiceTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type] || 6;
  }
}
```

### Claude AI Clinical Notes Service

```typescript
// apps/backend/src/ai/claude.service.ts
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

  async summarizeClinicalNote(
    appointmentId: string,
    rawNotes: string
  ): Promise<string> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: {
          include: {
            clinicalNotes: {
              orderBy: { createdAt: 'desc' },
              take: 5,
            },
          },
        },
      },
    });

    const patientHistory = appointment.patient.clinicalNotes
      .map((note) => `Sesión ${note.createdAt.toLocaleDateString('es-AR')}: ${note.summary}`)
      .join('\n\n');

    const prompt = `Sos un asistente clínico especializado en psicología en Argentina. Tu tarea es resumir las notas de sesión de manera estructurada y profesional.

Contexto del paciente:
Nombre: ${appointment.patient.firstName} ${appointment.patient.lastName}
Historial reciente:
${patientHistory}

Notas de la sesión actual:
${rawNotes}

Por favor, generá un resumen estructurado que incluya:
1. Motivo de consulta o tema principal
2. Observaciones clínicas relevantes
3. Intervenciones realizadas
4. Plan de tratamiento o seguimiento
5. Notas para próxima sesión

Formato: Profesional, conciso, en español argentino, respetando confidencialidad.`;

    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async generateTherapeuticInsights(patientId: string): Promise<string> {
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        appointments: {
          where: { status: 'COMPLETED' },
          include: { clinicalNote: true },
          orderBy: { startTime: 'asc' },
        },
      },
    });

    const sessionSummaries = patient.appointments
      .filter((apt) => apt.clinicalNote)
      .map((apt) => ({
        date: apt.startTime.toLocaleDateString('es-AR'),
        summary: apt.clinicalNote.summary,
      }));

    const prompt = `Sos un psicólogo clínico experimentado revisando el progreso terapéutico de un paciente. Analizá las siguientes ${sessionSummaries.length} sesiones y generá insights sobre:

1. Patrones recurrentes o temas centrales
2. Evolución del paciente a lo largo del tiempo
3. Áreas de progreso notable
4. Desafíos persistentes
5. Recomendaciones para el tratamiento futuro

Sesiones:
${sessionSummaries.map((s, i) => `Sesión ${i + 1} (${s.date}):\n${s.summary}`).join('\n\n---\n\n')}

Formato: Análisis profesional en español argentino, basado en evidencia clínica observable.`;

    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_OPUS_MODEL,
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  async chatAssistant(query: string, context: any): Promise<string> {
    // Use Sonnet for real-time responses
    const message = await this.anthropic.messages.create({
      model: process.env.CLAUDE_SONNET_MODEL,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `${query}\n\nContexto: ${JSON.stringify(context)}`,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }
}
```

### LiveKit Video Service

```typescript
// apps/backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class LiveKitService {
  constructor(private prisma: PrismaService) {}

  async createVideoRoom(appointmentId: string): Promise<string> {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    const roomName = `session-${appointmentId}`;

    // Update appointment with room ID
    await this.prisma.appointment.update({
      where: { id: appointmentId },
      data: { video
