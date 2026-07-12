---
name: sesion-mental-health-platform
description: Orchestration platform for psychology clinics with AI-powered scheduling, WhatsApp automation, AFIP-compliant billing, and secure video consultations
triggers:
  - "set up Sesión mental health platform"
  - "integrate WhatsApp automation for clinic appointments"
  - "configure AFIP electronic invoicing for psychology practice"
  - "implement Claude AI clinical note summarization"
  - "build appointment scheduling with Sesión"
  - "setup video consultation platform for therapists"
  - "create patient journey pipeline with Sesión"
  - "integrate Mercado Pago billing for mental health clinic"
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What is Sesión?

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It unifies appointment scheduling, WhatsApp automation, AFIP-compliant electronic invoicing, secure video consultations, and AI-powered clinical workflows powered by Claude Opus 4.6 and Sonnet 4.6 models.

The platform is built as a microservices ecosystem using:
- **Backend**: NestJS (TypeScript)
- **Frontend**: SvelteKit 5 + Svelte 5
- **Database**: PostgreSQL with Prisma ORM
- **Messaging**: Baileys (WhatsApp automation)
- **Video**: LiveKit WebRTC
- **AI**: Anthropic Claude (Opus 4.6, Sonnet 4.6)
- **Payments**: Stripe, Mercado Pago
- **Styling**: TailwindCSS

## Installation & Setup

### Prerequisites

```bash
# Node.js 20+ and pnpm
node -v  # Should be 20.x or higher
pnpm -v  # Should be 8.x or higher

# PostgreSQL 15+
psql --version

# Redis (for caching and real-time features)
redis-server --version
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install
```

### Environment Configuration

Create `.env` files for both backend and frontend:

**Backend `.env`:**

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db?schema=public"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_URL="wss://your-livekit-instance.com"

# Payment Providers
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
AFIP_PRODUCTION="false"

# JWT & Security
JWT_SECRET="${JWT_SECRET}"
JWT_EXPIRATION="15m"
REFRESH_TOKEN_EXPIRATION="7d"

# Application
PORT=3000
NODE_ENV="development"
```

**Frontend `.env`:**

```env
# API endpoint
PUBLIC_API_URL="http://localhost:3000"

# LiveKit
PUBLIC_LIVEKIT_URL="wss://your-livekit-instance.com"

# Feature flags
PUBLIC_ENABLE_AI_NOTES="true"
PUBLIC_ENABLE_WHATSAPP="true"
PUBLIC_ENABLE_VIDEO="true"
```

### Database Setup

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate deploy

# Seed initial data (optional)
pnpm prisma db seed
```

### Running the Platform

```bash
# Development mode (backend + frontend concurrently)
pnpm dev

# Backend only
cd apps/backend
pnpm dev

# Frontend only
cd apps/frontend
pnpm dev
```

## Core Architecture Patterns

### 1. Appointment Scheduling Engine

**Creating an appointment with conflict detection:**

```typescript
// apps/backend/src/modules/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { AppointmentType, AppointmentStatus } from '@prisma/client';

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(dto: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflicts = await this.findConflicts(
      dto.practitionerId,
      dto.startTime,
      dto.endTime,
      dto.roomId
    );

    if (conflicts.length > 0) {
      throw new ConflictException(
        'Scheduling conflict detected',
        conflicts
      );
    }

    // Create appointment with transaction
    return this.prisma.$transaction(async (tx) => {
      const appointment = await tx.appointment.create({
        data: {
          patientId: dto.patientId,
          practitionerId: dto.practitionerId,
          roomId: dto.roomId,
          type: dto.type as AppointmentType,
          startTime: new Date(dto.startTime),
          endTime: new Date(dto.endTime),
          status: AppointmentStatus.SCHEDULED,
          notes: dto.notes,
        },
        include: {
          patient: true,
          practitioner: true,
          room: true,
        },
      });

      // Trigger WhatsApp confirmation
      await this.whatsappService.sendAppointmentConfirmation(appointment);

      // Sync with external calendars
      await this.calendarSyncService.syncAppointment(appointment);

      return appointment;
    });
  }

  private async findConflicts(
    practitionerId: string,
    startTime: Date,
    endTime: Date,
    roomId?: string
  ) {
    return this.prisma.appointment.findMany({
      where: {
        OR: [
          { practitionerId },
          ...(roomId ? [{ roomId }] : []),
        ],
        status: { not: AppointmentStatus.CANCELLED },
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
  }

  // Recurring session pattern detection
  async detectRecurringPattern(patientId: string) {
    const recentAppointments = await this.prisma.appointment.findMany({
      where: {
        patientId,
        status: AppointmentStatus.COMPLETED,
        startTime: {
          gte: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000), // Last 90 days
        },
      },
      orderBy: { startTime: 'asc' },
    });

    // Calculate intervals between appointments
    const intervals = [];
    for (let i = 1; i < recentAppointments.length; i++) {
      const diff = recentAppointments[i].startTime.getTime() - 
                   recentAppointments[i - 1].startTime.getTime();
      intervals.push(Math.round(diff / (1000 * 60 * 60 * 24))); // Days
    }

    // Detect pattern (weekly: ~7 days, fortnightly: ~14 days)
    const avgInterval = intervals.reduce((a, b) => a + b, 0) / intervals.length;
    
    if (Math.abs(avgInterval - 7) < 2) return 'WEEKLY';
    if (Math.abs(avgInterval - 14) < 3) return 'FORTNIGHTLY';
    if (Math.abs(avgInterval - 30) < 5) return 'MONTHLY';
    
    return 'IRREGULAR';
  }
}
```

### 2. WhatsApp Automation with Baileys

**Setting up WhatsApp bot for appointment reminders:**

```typescript
// apps/backend/src/modules/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, { 
  DisconnectReason, 
  useMultiFileAuthState,
  WAMessage 
} from '@whiskeysockets/baileys';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: ReturnType<typeof makeWASocket>;

  constructor(
    private prisma: PrismaService,
    private configService: ConfigService,
  ) {}

  async onModuleInit() {
    await this.initializeWhatsApp();
    this.scheduledReminderJob();
  }

  private async initializeWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      this.configService.get('WHATSAPP_SESSION_PATH')
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
          (lastDisconnect?.error as any)?.output?.statusCode !== 
          DisconnectReason.loggedOut;
        if (shouldReconnect) {
          this.initializeWhatsApp();
        }
      }
    });

    // Handle incoming messages
    this.sock.ev.on('messages.upsert', async (m) => {
      await this.handleIncomingMessage(m.messages[0]);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    const patient = appointment.patient;
    const whatsappNumber = this.formatPhoneNumber(patient.phone);

    const message = `
🩺 *Confirmación de Turno - Sesión*

Hola ${patient.firstName},

Tu sesión ha sido agendada:
📅 Fecha: ${this.formatDate(appointment.startTime)}
⏰ Hora: ${this.formatTime(appointment.startTime)}
👤 Profesional: ${appointment.practitioner.firstName} ${appointment.practitioner.lastName}
📍 Modalidad: ${appointment.type === 'VIRTUAL' ? 'Virtual (videollamada)' : 'Presencial'}

Por favor, responde *SÍ* para confirmar o *NO* para cancelar.

_Este es un mensaje automatizado._
    `.trim();

    await this.sock.sendMessage(whatsappNumber, { text: message });

    // Log communication
    await this.prisma.whatsAppMessage.create({
      data: {
        appointmentId: appointment.id,
        patientId: patient.id,
        direction: 'OUTBOUND',
        messageType: 'APPOINTMENT_CONFIRMATION',
        content: message,
        status: 'SENT',
      },
    });
  }

  async send24HourReminder(appointment: any) {
    const patient = appointment.patient;
    const whatsappNumber = this.formatPhoneNumber(patient.phone);

    const message = `
⏰ *Recordatorio de Turno*

Hola ${patient.firstName},

Te recordamos tu sesión de mañana:
📅 ${this.formatDate(appointment.startTime)} a las ${this.formatTime(appointment.startTime)}
👤 ${appointment.practitioner.firstName} ${appointment.practitioner.lastName}

${appointment.type === 'VIRTUAL' 
  ? '🎥 Recibirás el link de videollamada 15 minutos antes.' 
  : '📍 Dirección: ' + appointment.room.location}

¿Necesitas reprogramar? Responde a este mensaje.
    `.trim();

    await this.sock.sendMessage(whatsappNumber, { text: message });
  }

  private async handleIncomingMessage(message: WAMessage) {
    if (!message.message) return;

    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text || '';
    const from = message.key.remoteJid;

    // Find patient by phone number
    const patient = await this.prisma.patient.findFirst({
      where: { 
        phone: { contains: from.replace('@s.whatsapp.net', '').slice(-10) }
      },
    });

    if (!patient) return;

    // Parse confirmation responses
    if (/^(sí|si|yes|confirmo)$/i.test(text.trim())) {
      await this.handleConfirmation(patient.id, from);
    } else if (/^(no|cancelar|cancel)$/i.test(text.trim())) {
      await this.handleCancellation(patient.id, from);
    } else if (/crisis|emergencia|urgente/i.test(text)) {
      await this.handleCrisisAlert(patient, text);
    }

    // Log all incoming messages
    await this.prisma.whatsAppMessage.create({
      data: {
        patientId: patient.id,
        direction: 'INBOUND',
        messageType: 'PATIENT_REPLY',
        content: text,
        status: 'RECEIVED',
      },
    });
  }

  private async handleCrisisAlert(patient: any, messageText: string) {
    // Alert practitioner immediately
    const practitioner = await this.prisma.practitioner.findFirst({
      where: { patients: { some: { id: patient.id } } },
    });

    if (practitioner?.emergencyPhone) {
      await this.sock.sendMessage(
        this.formatPhoneNumber(practitioner.emergencyPhone),
        {
          text: `🚨 *ALERTA DE CRISIS*\n\nPaciente: ${patient.firstName} ${patient.lastName}\nMensaje: "${messageText}"\n\nContactar inmediatamente.`,
        }
      );
    }

    // Auto-reply to patient
    await this.sock.sendMessage(
      this.formatPhoneNumber(patient.phone),
      {
        text: 'Tu mensaje ha sido marcado como urgente. Un profesional se comunicará contigo pronto. Si estás en riesgo inmediato, llama al 135 (emergencias).',
      }
    );
  }

  private formatPhoneNumber(phone: string): string {
    // Convert to WhatsApp format: 549XXXXXXXXXX@s.whatsapp.net (Argentina)
    const cleaned = phone.replace(/\D/g, '');
    return `549${cleaned.slice(-10)}@s.whatsapp.net`;
  }

  private scheduledReminderJob() {
    // Run every hour to check for appointments needing reminders
    setInterval(async () => {
      const tomorrow = new Date(Date.now() + 24 * 60 * 60 * 1000);
      const tomorrowStart = new Date(tomorrow.setHours(0, 0, 0, 0));
      const tomorrowEnd = new Date(tomorrow.setHours(23, 59, 59, 999));

      const appointments = await this.prisma.appointment.findMany({
        where: {
          startTime: { gte: tomorrowStart, lte: tomorrowEnd },
          status: 'SCHEDULED',
          reminderSent: false,
        },
        include: { patient: true, practitioner: true, room: true },
      });

      for (const appointment of appointments) {
        await this.send24HourReminder(appointment);
        await this.prisma.appointment.update({
          where: { id: appointment.id },
          data: { reminderSent: true },
        });
      }
    }, 60 * 60 * 1000); // Every hour
  }
}
```

### 3. AFIP Electronic Invoicing (Facturación Electrónica)

**Generating AFIP-compliant invoices:**

```typescript
// apps/backend/src/modules/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import * as soap from 'soap';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class AfipService {
  private wsaaClient: soap.Client;
  private wsfevClient: soap.Client;
  
  constructor(
    private prisma: PrismaService,
    private configService: ConfigService,
  ) {}

  async onModuleInit() {
    await this.authenticateAFIP();
  }

  private async authenticateAFIP() {
    const cert = readFileSync(this.configService.get('AFIP_CERT_PATH'));
    const key = readFileSync(this.configService.get('AFIP_KEY_PATH'));
    const cuit = this.configService.get('AFIP_CUIT');

    // WSAA Authentication (Web Service de Autenticación y Autorización)
    const wsaaUrl = this.configService.get('AFIP_PRODUCTION') === 'true'
      ? 'https://wsaa.afip.gov.ar/ws/services/LoginCms?WSDL'
      : 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms?WSDL';

    this.wsaaClient = await soap.createClientAsync(wsaaUrl);

    // Generate LoginTicketRequest (TRA)
    const tra = this.generateTRA('wsfe');
    const cms = this.signTRA(tra, cert, key);

    const loginResult = await this.wsaaClient.loginCmsAsync({
      in0: cms.toString('base64'),
    });

    const token = loginResult[0].return.token;
    const sign = loginResult[0].return.sign;

    // Initialize electronic invoicing service
    const wsfeUrl = this.configService.get('AFIP_PRODUCTION') === 'true'
      ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx?WSDL'
      : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx?WSDL';

    this.wsfevClient = await soap.createClientAsync(wsfeUrl);
    this.wsfevClient.addSoapHeader({
      Token: token,
      Sign: sign,
      Cuit: cuit,
    });
  }

  async generateInvoice(sessionId: string) {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        appointment: { include: { patient: true, practitioner: true } },
        payment: true,
      },
    });

    if (!session || session.invoiceGenerated) {
      throw new Error('Session invalid or already invoiced');
    }

    const practitioner = session.appointment.practitioner;
    const patient = session.appointment.patient;

    // Determine invoice type based on practitioner's fiscal status
    const invoiceType = this.getInvoiceType(
      practitioner.fiscalCategory,
      patient.fiscalCategory
    );

    // Get last invoice number
    const lastInvoice = await this.getLastInvoiceNumber(
      practitioner.cuit,
      invoiceType
    );

    const invoiceData = {
      CbteTipo: invoiceType, // 1=A, 6=B, 11=C
      PtoVta: practitioner.salePoint || 1,
      CbteDesde: lastInvoice + 1,
      CbteHasta: lastInvoice + 1,
      CbteFch: this.formatAFIPDate(new Date()),
      ImpTotal: session.payment.amount,
      ImpTotConc: 0, // Non-taxable amount
      ImpNeto: session.payment.amount / 1.21, // Net before VAT
      ImpOpEx: 0, // Exempt amount
      ImpIVA: session.payment.amount - (session.payment.amount / 1.21),
      ImpTrib: 0, // Other taxes
      MonId: 'PES', // Argentine Pesos
      MonCotiz: 1,
      DocTipo: patient.documentType === 'DNI' ? 96 : 80, // 96=DNI, 80=CUIT
      DocNro: patient.documentNumber,
      Concepto: 2, // Services
      IVA: [
        {
          Id: 5, // 21% VAT
          BaseImp: session.payment.amount / 1.21,
          Importe: session.payment.amount - (session.payment.amount / 1.21),
        },
      ],
    };

    // Call AFIP web service
    const result = await this.wsfevClient.FECAESolicitarAsync({
      Auth: { /* credentials injected via SOAP header */ },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: invoiceData.PtoVta,
          CbteTipo: invoiceData.CbteTipo,
        },
        FeDetReq: { FECAEDetRequest: [invoiceData] },
      },
    });

    const afipResponse = result[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse[0];

    if (afipResponse.Resultado !== 'A') {
      throw new Error(`AFIP error: ${afipResponse.Observaciones}`);
    }

    // Store invoice in database
    const invoice = await this.prisma.invoice.create({
      data: {
        sessionId: session.id,
        practitionerId: practitioner.id,
        patientId: patient.id,
        invoiceType,
        salePoint: invoiceData.PtoVta,
        invoiceNumber: afipResponse.CbteDesde,
        cae: afipResponse.CAE, // Electronic authorization code
        caeExpiration: new Date(afipResponse.CAEFchVto),
        amount: session.payment.amount,
        currency: 'ARS',
        issueDate: new Date(),
        afipResponse: JSON.stringify(afipResponse),
      },
    });

    // Mark session as invoiced
    await this.prisma.session.update({
      where: { id: sessionId },
      data: { invoiceGenerated: true, invoiceId: invoice.id },
    });

    // Send invoice to patient via WhatsApp
    await this.whatsappService.sendInvoice(invoice);

    return invoice;
  }

  private getInvoiceType(
    practitionerCategory: string,
    patientCategory: string
  ): number {
    // A: Responsable Inscripto to Responsable Inscripto
    // B: Responsable Inscripto to Monotributo/Consumer Final
    // C: Monotributo to any
    
    if (practitionerCategory === 'RESPONSABLE_INSCRIPTO') {
      return patientCategory === 'RESPONSABLE_INSCRIPTO' ? 1 : 6;
    }
    return 11; // Type C for Monotributo
  }

  private formatAFIPDate(date: Date): string {
    // AFIP format: YYYYMMDD
    return date.toISOString().slice(0, 10).replace(/-/g, '');
  }

  private async getLastInvoiceNumber(
    cuit: string,
    invoiceType: number
  ): Promise<number> {
    const result = await this.wsfevClient.FECompUltimoAutorizadoAsync({
      PtoVta: 1,
      CbteTipo: invoiceType,
    });
    return result[0].FECompUltimoAutorizadoResult.CbteNro || 0;
  }
}
```

### 4. Claude AI Clinical Note Summarization

**Using Claude Opus for deep clinical analysis:**

```typescript
// apps/backend/src/modules/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;
  private opusModel: string;
  private sonnetModel: string;

  constructor(
    private prisma: PrismaService,
    private configService: ConfigService,
  ) {
    this.anthropic = new Anthropic({
      apiKey: this.configService.get('ANTHROPIC_API_KEY'),
    });
    this.opusModel = this.configService.get('CLAUDE_OPUS_MODEL');
    this.sonnetModel = this.configService.get('CLAUDE_SONNET_MODEL');
  }

  async summarizeSession(sessionId: string): Promise<string> {
    const session = await this.prisma.session.findUnique({
      where: { id: sessionId },
      include: {
        appointment: { include: { patient: true, practitioner: true } },
        notes: true,
      },
    });

    if (!session || !session.notes) {
      throw new Error('Session or notes not found');
    }

    // Use Opus for deep analysis
    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 2000,
      temperature: 0.3,
      system: `Eres un asistente especializado en psicología clínica argentina. 
Tu tarea es analizar notas de sesión terapéutica y generar resúmenes estructurados 
que cumplan con los estándares del Colegio de Psicólogos de Argentina.

Formato de salida:
1. Motivo de consulta (si es primera sesión)
2. Temáticas principales abordadas
3. Intervenciones realizadas
4. Observaciones clínicas relevantes
5. Planificación para próxima sesión

Mantén confidencialidad absoluta y usa lenguaje profesional.`,
      messages: [
        {
          role: 'user',
          content: `Analiza las siguientes notas de sesión y genera un resumen estructurado:

**Paciente**: ${session.appointment.patient.firstName} ${session.appointment.patient.lastName}
**Fecha**: ${session.appointment.startTime.toLocaleDateString('es-AR')}
**Duración**: ${session.duration} minutos
**Tipo**: ${session.appointment.type}

**Notas del terapeuta**:
${session.notes}

Genera el resumen clínico estructurado.`,
        },
      ],
    });

    const summary = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    // Store AI-generated summary
    await this.prisma.session.update({
      where: { id: sessionId },
      data: {
        aiSummary: summary,
        aiModel: this.opusModel,
        aiTokensUsed: message.usage.input_tokens + message.usage.output_tokens,
      },
    });

    // Log AI usage for billing
    await this.prisma.aiUsageLog.create({
      data: {
        sessionId: session.id,
        practitionerId: session.appointment.practitionerId,
        model: this.opusModel,
        inputTokens: message.usage.input_tokens,
        outputTokens: message.usage.output_tokens,
        cost: this.calculateCost(message.usage),
      },
    });

    return summary;
  }

  async chatAssistant(query: string, practitionerId: string): Promise<string> {
    // Use Sonnet for real-time interactions
    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 1000,
      temperature: 0.5,
      system: `Eres un asistente para psicólogos argentinos. Ayudas con:
- Búsqueda rápida de información de pacientes
- Generación de reportes
- Recordatorios de tareas clínicas
- Respuestas sobre normativas del Colegio de Psicólogos

Responde de manera concisa y profesional en español argentino.`,
      messages: [{ role: 'user', content: query }],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async predictOutcome(patientId: string): Promise<any> {
    // Retrieve patient history
    const patient = await this.prisma.patient.findUnique({
      where: { id: patientId },
      include: {
        appointments: {
          include: { session: true },
          orderBy: { startTime: 'desc' },
          take: 20,
        },
      },
    });

    const sessionNotes = patient.appointments
      .filter(a => a.session?.notes)
      .map(a => a.session.notes)
      .join('\n\n---\n\n');

    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 1500,
      temperature: 0.2,
      system: `Analiza el historial clínico del paciente y predice:
1. Probabilidad de abandono terapéutico (alta/media/baja)
2. Áreas que requieren mayor atención
3. Sugerencias de intervenciones
4. Riesgo clínico (si aplica)

Basate únicamente en patrones clínicos observables. No hagas diagnósticos.`,
      messages: [
        {
          role: 'user',
          content: `Historial de ${patient.appointments.length} sesiones:\n\n${sessionNotes}\n\nGenera el análisis predictivo.`,
        },
      ],
    });

    return {
      prediction: message.content[0].type === 'text' 
        ? message.content[0].text 
        : '',
      confidence: 'MEDIUM',
      model: this.opusModel,
    };
  }

  private calculateCost(usage: { input_tokens: number; output_tokens: number }): number {
    // Opus pricing (example rates per 1M tokens)
    const inputCostPer1M = 15; // USD
    
