---
name: sesion-mental-health-platform
description: Intelligent orchestration platform for psychology clinics with AI-powered scheduling, WhatsApp automation, AFIP billing, and secure video consultations
triggers:
  - set up sesion mental health platform
  - integrate sesion clinic management
  - configure sesion whatsapp automation
  - implement sesion appointment scheduling
  - add sesion afip billing integration
  - use sesion claude ai features
  - deploy sesion video consultation system
  - configure sesion for argentine psychology practice
---

# Sesion Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesion is a comprehensive SaaS orchestration platform for psychology clinics in Argentina, combining intelligent appointment scheduling, automated WhatsApp messaging (via Baileys), AFIP-compliant billing, secure video consultations (LiveKit), and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6. Built with NestJS backend, SvelteKit 5 frontend, Prisma ORM, and TypeScript throughout.

## Technology Stack

- **Backend**: NestJS (microservices architecture)
- **Frontend**: SvelteKit 5 + Svelte 5 + TailwindCSS
- **Database**: PostgreSQL with Prisma ORM
- **AI**: Anthropic Claude (Opus 4.6 for deep analysis, Sonnet 4.6 for real-time)
- **Messaging**: Baileys (WhatsApp automation)
- **Video**: LiveKit (WebRTC video consultations)
- **Payments**: Stripe + Mercado Pago (Argentina)
- **Language**: TypeScript

## Installation

### Prerequisites

```bash
# Required versions
node >= 20.x
pnpm >= 9.x
postgresql >= 15.x
redis >= 7.x
```

### Clone and Setup

```bash
# Clone repository
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies
pnpm install

# Setup environment variables
cp .env.example .env
```

### Environment Configuration

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=your_anthropic_key_here
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET=your_webhook_secret

# LiveKit Video
LIVEKIT_API_KEY=your_livekit_key
LIVEKIT_API_SECRET=your_livekit_secret
LIVEKIT_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax)
AFIP_CUIT=your_tax_id
AFIP_CERT_PATH="./certs/afip.crt"
AFIP_KEY_PATH="./certs/afip.key"

# Payments
MERCADOPAGO_ACCESS_TOKEN=your_mp_token
STRIPE_SECRET_KEY=your_stripe_key

# Application
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

# Seed database (optional)
pnpm prisma db seed
```

### Start Development Servers

```bash
# Start backend (NestJS)
cd apps/backend
pnpm dev

# Start frontend (SvelteKit) - new terminal
cd apps/frontend
pnpm dev
```

## Core Architecture

### Backend Structure (NestJS)

```typescript
// apps/backend/src/modules/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { ClaudeService } from '../ai/claude.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private claude: ClaudeService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: CreateAppointmentDto) {
    // Check for scheduling conflicts
    const conflicts = await this.checkConflicts(
      data.practitionerId,
      data.startTime,
      data.endTime,
    );

    if (conflicts.length > 0) {
      // Use Claude Sonnet for quick alternative suggestions
      const alternatives = await this.claude.suggestAlternatives({
        practitionerId: data.practitionerId,
        preferredTime: data.startTime,
        sessionType: data.type,
        model: 'sonnet', // Fast real-time suggestions
      });

      return {
        success: false,
        conflicts,
        alternatives,
      };
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        patientId: data.patientId,
        practitionerId: data.practitionerId,
        startTime: data.startTime,
        endTime: data.endTime,
        type: data.type,
        status: 'SCHEDULED',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(
      appointment.patient.phoneNumber,
      appointment,
    );

    return { success: true, appointment };
  }

  async checkConflicts(
    practitionerId: string,
    startTime: Date,
    endTime: Date,
  ) {
    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
        status: { in: ['SCHEDULED', 'IN_PROGRESS'] },
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
}
```

### AI Orchestration with Claude

```typescript
// apps/backend/src/modules/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private client: Anthropic;
  private opusModel = process.env.CLAUDE_OPUS_MODEL;
  private sonnetModel = process.env.CLAUDE_SONNET_MODEL;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  // Deep analysis with Opus (patient history, treatment plans)
  async analyzePatientHistory(patientId: string, sessionNotes: string[]) {
    const context = sessionNotes.join('\n\n---\n\n');

    const response = await this.client.messages.create({
      model: this.opusModel,
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: `Analiza el historial clínico del paciente y proporciona:
1. Resumen de temas recurrentes
2. Progreso terapéutico observado
3. Áreas de atención prioritaria
4. Sugerencias de intervención

Contexto (notas clínicas):
${context}

Responde en español argentino, con terminología profesional de psicología.`,
        },
      ],
    });

    return {
      model: 'opus',
      analysis: response.content[0].text,
      usage: response.usage,
    };
  }

  // Real-time assistance with Sonnet (quick queries, note drafting)
  async generateClinicalNote(sessionSummary: string) {
    const response = await this.client.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `Genera una nota clínica profesional basada en el siguiente resumen:

${sessionSummary}

La nota debe incluir:
- Motivo de consulta
- Observaciones clínicas
- Intervenciones realizadas
- Plan terapéutico

Formato: profesional, conciso, apto para historia clínica.`,
        },
      ],
    });

    return {
      model: 'sonnet',
      note: response.content[0].text,
      usage: response.usage,
    };
  }

  // Intelligent appointment suggestions
  async suggestAlternatives(params: {
    practitionerId: string;
    preferredTime: Date;
    sessionType: string;
    model?: 'opus' | 'sonnet';
  }) {
    const model =
      params.model === 'opus' ? this.opusModel : this.sonnetModel;

    const response = await this.client.messages.create({
      model,
      max_tokens: 512,
      messages: [
        {
          role: 'user',
          content: `Sugiere 3 horarios alternativos cercanos a ${params.preferredTime.toISOString()} 
para sesión de tipo ${params.sessionType}. 
Considera horarios típicos de práctica psicológica en Argentina (09:00-20:00).
Responde en formato JSON: [{ "date": "ISO_DATE", "reason": "string" }]`,
        },
      ],
    });

    return JSON.parse(response.content[0].text);
  }
}
```

### WhatsApp Automation with Baileys

```typescript
// apps/backend/src/modules/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
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
          (lastDisconnect?.error as Boom)?.output?.statusCode !==
          DisconnectReason.loggedOut;

        if (shouldReconnect) {
          this.connectToWhatsApp();
        }
      } else if (connection === 'open') {
        console.log('WhatsApp connected successfully');
      }
    });

    // Listen for incoming messages (crisis detection)
    this.sock.ev.on('messages.upsert', async (m) => {
      const message = m.messages[0];
      if (!message.key.fromMe && message.message) {
        await this.handleIncomingMessage(message);
      }
    });
  }

  async sendAppointmentConfirmation(phone: string, appointment: any) {
    const formattedPhone = phone.replace(/\D/g, '') + '@s.whatsapp.net';

    const message = `✅ *Turno Confirmado*

📅 Fecha: ${appointment.startTime.toLocaleDateString('es-AR')}
🕐 Hora: ${appointment.startTime.toLocaleTimeString('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
    })}
👤 Profesional: ${appointment.practitioner.name}
📍 Modalidad: ${appointment.type === 'VIRTUAL' ? 'Videollamada' : 'Presencial'}

Recibirás un recordatorio 24hs antes.

Para confirmar asistencia, responde *SI*
Para cancelar, responde *NO*`;

    await this.sock.sendMessage(formattedPhone, { text: message });
  }

  async sendReminder(phone: string, appointment: any, hours: number) {
    const formattedPhone = phone.replace(/\D/g, '') + '@s.whatsapp.net';

    const message = `⏰ *Recordatorio de Turno*

Tu sesión es en ${hours} horas:

📅 ${appointment.startTime.toLocaleDateString('es-AR')}
🕐 ${appointment.startTime.toLocaleTimeString('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
    })}
👤 ${appointment.practitioner.name}

${appointment.type === 'VIRTUAL' ? '🔗 Link de videollamada: ' + appointment.meetingUrl : '📍 Consultorio: ' + appointment.practitioner.address}`;

    await this.sock.sendMessage(formattedPhone, { text: message });
  }

  private async handleIncomingMessage(message: any) {
    const text = message.message?.conversation || '';
    const phone = message.key.remoteJid;

    // Crisis keyword detection
    const crisisKeywords = [
      'suicidio',
      'quiero morir',
      'no puedo más',
      'me quiero matar',
      'crisis',
    ];

    if (crisisKeywords.some((kw) => text.toLowerCase().includes(kw))) {
      // Alert practitioner immediately
      await this.alertPractitioner(phone, text);

      // Send crisis resources
      await this.sock.sendMessage(phone, {
        text: `🆘 *Apoyo Inmediato*

Si estás en crisis, por favor contacta:

📞 Centro de Asistencia al Suicida: 135 (gratuito)
📞 Emergencias: 911
🏥 Hospital más cercano

Tu terapeuta ha sido notificado y se contactará contigo.`,
      });
    }
  }

  private async alertPractitioner(patientPhone: string, message: string) {
    // Implementation would notify practitioner via app notification
    console.log('CRISIS ALERT:', { patientPhone, message });
  }
}
```

### AFIP Billing Integration

```typescript
// apps/backend/src/modules/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import * as soap from 'soap';

@Injectable()
export class AfipService {
  private wsdlUrl = 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx?WSDL';

  async generateInvoice(data: {
    patientName: string;
    patientTaxId: string;
    amount: number;
    sessionDate: Date;
    invoiceType: 'A' | 'B' | 'C';
  }) {
    // Get authentication token from AFIP
    const authToken = await this.getAuthToken();

    const client = await soap.createClientAsync(this.wsdlUrl);

    const invoiceData = {
      Auth: {
        Token: authToken.token,
        Sign: authToken.sign,
        Cuit: process.env.AFIP_CUIT,
      },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: 1,
          CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
        },
        FeDetReq: {
          FECAEDetRequest: {
            Concepto: 2, // Servicios
            DocTipo: 80, // CUIT
            DocNro: data.patientTaxId,
            CbteDesde: 1,
            CbteHasta: 1,
            CbteFch: data.sessionDate.toISOString().split('T')[0],
            ImpTotal: data.amount,
            ImpTotConc: 0,
            ImpNeto: data.amount,
            ImpOpEx: 0,
            ImpIVA: 0,
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
          },
        },
      },
    };

    const result = await client.FECAESolicitarAsync(invoiceData);

    return {
      cae: result.FECAESolicitarResult.FeDetResp.FECAEDetResponse.CAE,
      caeExpiration:
        result.FECAESolicitarResult.FeDetResp.FECAEDetResponse.CAEFchVto,
      invoiceNumber:
        result.FECAESolicitarResult.FeDetResp.FECAEDetResponse.CbteDesde,
    };
  }

  private async getAuthToken() {
    // AFIP authentication with certificate
    const cert = readFileSync(process.env.AFIP_CERT_PATH);
    const key = readFileSync(process.env.AFIP_KEY_PATH);

    // Implementation would use AFIP's WSAA service
    // This is a simplified representation
    return {
      token: 'auth_token_from_afip',
      sign: 'signature_from_afip',
    };
  }

  private getInvoiceTypeCode(type: 'A' | 'B' | 'C'): number {
    const codes = { A: 1, B: 6, C: 11 };
    return codes[type];
  }
}
```

### LiveKit Video Consultation

```typescript
// apps/backend/src/modules/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  async createVideoSession(appointmentId: string, participantRole: 'practitioner' | 'patient') {
    const roomName = `session-${appointmentId}`;

    const at = new AccessToken(
      process.env.LIVEKIT_API_KEY,
      process.env.LIVEKIT_API_SECRET,
      {
        identity: `${participantRole}-${appointmentId}`,
      },
    );

    at.addGrant({
      roomJoin: true,
      room: roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: participantRole === 'practitioner', // Only practitioner can share screen
    });

    const token = at.toJwt();

    return {
      token,
      url: process.env.LIVEKIT_URL,
      roomName,
    };
  }
}
```

### Frontend (SvelteKit 5)

```svelte
<!-- apps/frontend/src/routes/appointments/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Calendar } from '$lib/components/Calendar';
  import type { Appointment } from '$lib/types';

  let appointments = $state<Appointment[]>([]);
  let loading = $state(true);

  onMount(async () => {
    const response = await fetch('/api/appointments');
    appointments = await response.json();
    loading = false;
  });

  async function createAppointment(data: {
    patientId: string;
    startTime: Date;
    type: string;
  }) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    const result = await response.json();

    if (!result.success && result.alternatives) {
      // Show AI-suggested alternatives
      showAlternatives(result.alternatives);
    } else {
      appointments = [...appointments, result.appointment];
    }
  }

  function showAlternatives(alternatives: any[]) {
    // Show modal with Claude-suggested alternative times
    console.log('Suggested alternatives:', alternatives);
  }
</script>

<div class="container mx-auto p-4">
  <h1 class="text-3xl font-bold mb-6">Agenda Inteligente</h1>

  {#if loading}
    <p>Cargando turnos...</p>
  {:else}
    <Calendar {appointments} on:create={createAppointment} />
  {/if}
</div>
```

## Common Patterns

### Appointment Scheduling with Conflict Detection

```typescript
// Check conflicts and suggest alternatives
const result = await appointmentsService.createAppointment({
  patientId: 'patient-123',
  practitionerId: 'pract-456',
  startTime: new Date('2026-08-15T10:00:00'),
  endTime: new Date('2026-08-15T10:45:00'),
  type: 'INDIVIDUAL',
});

if (!result.success) {
  // Claude Sonnet provides intelligent alternatives
  console.log('Alternatives:', result.alternatives);
}
```

### Automated WhatsApp Reminders (Cron Job)

```typescript
// apps/backend/src/modules/appointments/appointments.cron.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsCron {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async sendReminders() {
    const now = new Date();
    const in24Hours = new Date(now.getTime() + 24 * 60 * 60 * 1000);

    // Find appointments 24 hours away
    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: now,
          lte: in24Hours,
        },
        status: 'SCHEDULED',
        reminderSent: false,
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    for (const appointment of appointments) {
      await this.whatsapp.sendReminder(
        appointment.patient.phoneNumber,
        appointment,
        24,
      );

      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent: true },
      });
    }
  }
}
```

### Patient Journey Tracking

```typescript
// apps/backend/src/modules/patients/patients.service.ts
async getPatientJourney(patientId: string) {
  const patient = await this.prisma.patient.findUnique({
    where: { id: patientId },
    include: {
      appointments: {
        orderBy: { startTime: 'asc' },
      },
      invoices: true,
      clinicalNotes: true,
    },
  });

  // Calculate journey metrics
  const totalSessions = patient.appointments.filter(
    (a) => a.status === 'COMPLETED',
  ).length;

  const dropoutRisk = await this.calculateDropoutRisk(patient);

  // Use Claude Opus for deep journey analysis
  const aiAnalysis = await this.claude.analyzePatientHistory(
    patientId,
    patient.clinicalNotes.map((n) => n.content),
  );

  return {
    patient,
    metrics: {
      totalSessions,
      dropoutRisk,
      currentStage: this.determineStage(patient),
    },
    aiInsights: aiAnalysis,
  };
}
```

## Prisma Schema Examples

```prisma
// prisma/schema.prisma
model Patient {
  id          String   @id @default(cuid())
  name        String
  email       String   @unique
  phoneNumber String
  taxId       String?  // CUIT/CUIL for billing
  createdAt   DateTime @default(now())

  appointments  Appointment[]
  invoices      Invoice[]
  clinicalNotes ClinicalNote[]
}

model Appointment {
  id             String   @id @default(cuid())
  patientId      String
  practitionerId String
  startTime      DateTime
  endTime        DateTime
  type           AppointmentType
  status         AppointmentStatus
  meetingUrl     String?
  reminderSent   Boolean  @default(false)
  createdAt      DateTime @default(now())

  patient      Patient      @relation(fields: [patientId], references: [id])
  practitioner Practitioner @relation(fields: [practitionerId], references: [id])
  invoice      Invoice?
}

model Invoice {
  id            String   @id @default(cuid())
  appointmentId String   @unique
  patientId     String
  amount        Float
  type          InvoiceType // A, B, C
  cae           String?
  caeExpiration DateTime?
  status        InvoiceStatus
  createdAt     DateTime @default(now())

  appointment Appointment @relation(fields: [appointmentId], references: [id])
  patient     Patient     @relation(fields: [patientId], references: [id])
}

enum AppointmentType {
  INDIVIDUAL
  COUPLE
  FAMILY
  EVALUATION
  VIRTUAL
  IN_PERSON
}

enum AppointmentStatus {
  SCHEDULED
  IN_PROGRESS
  COMPLETED
  CANCELLED
  NO_SHOW
}

enum InvoiceType {
  A
  B
  C
  M
}

enum InvoiceStatus {
  DRAFT
  ISSUED
  PAID
  OVERDUE
}
```

## API Endpoints

### Appointments

```bash
# List appointments
GET /api/appointments?practitionerId=xxx&from=2026-08-01&to=2026-08-31

# Create appointment
POST /api/appointments
{
  "patientId": "patient-123",
  "practitionerId": "pract-456",
  "startTime": "2026-08-15T10:00:00Z",
  "endTime": "2026-08-15T10:45:00Z",
  "type": "INDIVIDUAL"
}

# Update appointment
PATCH /api/appointments/:id
{
  "status": "CANCELLED"
}
```

### AI Analysis

```bash
# Generate clinical note
POST /api/ai/clinical-note
{
  "sessionSummary": "Patient discussed anxiety about work...",
  "model": "sonnet"
}

# Deep patient analysis
POST /api/ai/patient-analysis
{
  "patientId": "patient-123",
  "model": "opus"
}
```

### Billing

```bash
# Generate AFIP invoice
POST /api/billing/invoices
{
  "appointmentId": "appt-789",
  "type": "B",
  "amount": 5000
}

# Get invoice PDF
GET /api/billing/invoices/:id/pdf
```

## Troubleshooting

### WhatsApp Connection Issues

```bash
# Delete session and reconnect
rm -rf ./whatsapp-sessions
# Restart backend - new QR code will be generated
```

### AFIP Certificate Errors

```bash
# Verify certificate validity
openssl x509 -in ./certs/afip.crt -noout -dates

# Test AFIP connection
curl -X POST https://wswhomo.afip.gov.ar/wsfev1/service.asmx \
  -H "Content-Type: text/xml"
```

### Claude API Rate Limits

```typescript
// Implement exponential backoff
async function callClaudeWithRetry(
  fn: () => Promise<any>,
  maxRetries = 3,
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise((resolve) =>
          setTimeout(resolve, Math.pow(2, i) * 1000),
        );
      } else {
        throw error;
      }
    }
  }
}
```

### Database Migration Issues

```bash
# Reset database (development only!)
pnpm prisma migrate reset

# Generate new migration
pnpm prisma migrate dev --name your_migration_name

# Deploy to production
pnpm prisma migrate deploy
```

### LiveKit Video Quality Issues

```typescript
// Adjust video quality based on bandwidth
const token = new AccessToken(apiKey, apiSecret, {
  identity: userId,
  metadata: JSON.stringify({
    videoBitrate: 500_000, // 500kbps for low bandwidth
    audioBitrate: 20_000,
  }),
});
```

## Testing

```bash
# Run unit tests
pnpm test

# Run e2e tests
pnpm test:e2e

# Test WhatsApp integration (requires active session)
pnpm test:whatsapp

# Test AFIP integration (homologation environment)
pnpm test:afip
```

## Deployment

```bash
# Build production
pnpm build

# Run migrations
pnpm prisma migrate deploy

# Start production server
pnpm start:prod
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install
COPY . .
RUN pnpm prisma generate
RUN pnpm build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
CMD ["node", "dist/main.js"]
```

```bash
# Build and run
docker build -t sesion-platform .
docker run -p 3000:3000 --env-file .env sesion-platform
```

## Key Configuration Files

```typescript
// apps/backend/src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true,
  });

  await app.listen(process.env.PORT || 3000);
}

bootstrap();
```

```typescript
// apps/frontend/svelte.config.js
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

export default {
  preprocess: vitePreprocess(),
  kit:
