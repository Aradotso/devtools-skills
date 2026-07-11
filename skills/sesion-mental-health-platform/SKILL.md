---
name: sesion-mental-health-platform
description: Multi-service platform for psychology clinics with scheduling, WhatsApp automation, AFIP billing, video consultations, and Claude AI integration
triggers:
  - set up sesion mental health platform
  - integrate sesion whatsapp automation
  - configure afip electronic invoicing
  - implement sesion appointment scheduling
  - add claude ai to sesion workflow
  - deploy sesion video consultation system
  - configure sesion for argentine psychology clinic
  - troubleshoot sesion orchestration services
---

# Sesion Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS platform for Argentine psychology clinics that orchestrates:
- **Intelligent appointment scheduling** with conflict detection and multi-practitioner support
- **WhatsApp automation** via Baileys for reminders, confirmations, and patient communication
- **AFIP-compliant electronic invoicing** (facturación electrónica) with Mercado Pago/Stripe integration
- **Secure video consultations** using LiveKit with WebRTC
- **AI-powered clinical assistance** using Claude Opus 4.6 and Sonnet 4.6 for note summarization and patient insights

The platform uses a microservices architecture with NestJS backend, SvelteKit 5 frontend, Prisma ORM, and Kafka for event-driven workflows.

## Tech Stack

- **Backend**: NestJS (TypeScript)
- **Frontend**: SvelteKit 5 + Svelte 5 + Tailwind CSS
- **Database**: PostgreSQL + Prisma ORM
- **Cache/Session**: Redis
- **Message Queue**: Apache Kafka
- **WhatsApp**: Baileys library
- **Video**: LiveKit
- **AI**: Anthropic Claude (Opus 4.6 + Sonnet 4.6)
- **Payments**: Mercado Pago (Argentina) + Stripe (international)

## Installation

### Prerequisites

```bash
# Required services
docker compose up -d postgres redis kafka livekit

# Environment variables
cp .env.example .env
```

### Core Environment Variables

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="your_key_here"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your_webhook_secret"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="your_clinic_cuit"
AFIP_CERT_PATH="./afip-cert.pem"
AFIP_KEY_PATH="./afip-key.pem"
AFIP_ENVIRONMENT="homologacion" # or "produccion"

# Mercado Pago
MERCADOPAGO_ACCESS_TOKEN="your_mp_token"
MERCADOPAGO_PUBLIC_KEY="your_mp_public_key"

# LiveKit Video
LIVEKIT_API_KEY="your_livekit_key"
LIVEKIT_API_SECRET="your_livekit_secret"
LIVEKIT_WS_URL="wss://your-livekit-server.com"

# Kafka
KAFKA_BROKERS="localhost:9092"
KAFKA_CLIENT_ID="sesion-orchestrator"
```

### Backend Setup

```bash
cd backend

# Install dependencies
npm install

# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate deploy

# Start development server
npm run start:dev
```

### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

## Project Structure

```
sesion-workflow-clinic/
├── backend/                    # NestJS microservices
│   ├── src/
│   │   ├── agenda/            # Appointment scheduling module
│   │   ├── whatsapp/          # WhatsApp automation module
│   │   ├── billing/           # AFIP invoicing module
│   │   ├── video/             # LiveKit video module
│   │   ├── ai/                # Claude AI orchestration
│   │   ├── orchestrator/      # Service coordination layer
│   │   └── prisma/            # Database schema
│   └── package.json
├── frontend/                   # SvelteKit application
│   ├── src/
│   │   ├── routes/            # SvelteKit routes
│   │   ├── lib/               # Shared components
│   │   └── app.html
│   └── package.json
└── docker-compose.yml
```

## Core Modules

### 1. Appointment Scheduling (Agenda)

```typescript
// backend/src/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { AppointmentStatus, SessionType } from '@prisma/client';

@Injectable()
export class AgendaService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number; // minutes
    sessionType: SessionType;
    roomId?: string;
  }) {
    // Check for conflicts
    const conflicts = await this.findConflicts(
      data.practitionerId,
      data.startTime,
      data.duration
    );

    if (conflicts.length > 0) {
      throw new Error('Schedule conflict detected');
    }

    // Create appointment
    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        status: AppointmentStatus.SCHEDULED,
        endTime: new Date(data.startTime.getTime() + data.duration * 60000),
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Trigger WhatsApp confirmation
    await this.triggerWhatsAppConfirmation(appointment);

    return appointment;
  }

  async findConflicts(
    practitionerId: string,
    startTime: Date,
    duration: number
  ) {
    const endTime = new Date(startTime.getTime() + duration * 60000);

    return this.prisma.appointment.findMany({
      where: {
        practitionerId,
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

  async suggestAlternatives(
    practitionerId: string,
    preferredDate: Date,
    duration: number
  ) {
    // Get practitioner availability
    const availability = await this.prisma.practitionerAvailability.findMany({
      where: { practitionerId },
    });

    // Generate time slots for next 7 days
    const suggestions = [];
    for (let day = 0; day < 7; day++) {
      const date = new Date(preferredDate);
      date.setDate(date.getDate() + day);

      const daySlots = await this.generateDaySlots(
        practitionerId,
        date,
        duration,
        availability
      );
      suggestions.push(...daySlots);
    }

    return suggestions.slice(0, 5); // Return top 5 suggestions
  }
}
```

### 2. WhatsApp Automation (Baileys)

```typescript
// backend/src/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  WAMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions'
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
      }
    });

    // Handle incoming messages
    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessages(messages);
    });
  }

  async sendAppointmentReminder(
    phoneNumber: string,
    appointmentData: {
      patientName: string;
      practitionerName: string;
      date: Date;
      sessionType: string;
    }
  ) {
    const formattedPhone = this.formatPhoneNumber(phoneNumber);
    const formattedDate = this.formatArgentineDateTime(appointmentData.date);

    const message = `Hola ${appointmentData.patientName} 👋

📅 *Recordatorio de Sesión*

Tu sesión ${appointmentData.sessionType} con ${appointmentData.practitionerName} está programada para:
🗓️ ${formattedDate}

Por favor confirmá tu asistencia respondiendo:
✅ *SI* - Confirmo
❌ *NO* - Necesito reprogramar

Si tenés alguna consulta, no dudes en escribirnos.

_Sesión - Plataforma de Gestión Clínica_`;

    await this.sock.sendMessage(formattedPhone, { text: message });

    return { sent: true, phoneNumber: formattedPhone };
  }

  async sendInvoiceWhatsApp(
    phoneNumber: string,
    invoiceData: {
      patientName: string;
      invoiceNumber: string;
      amount: number;
      pdfUrl: string;
    }
  ) {
    const formattedPhone = this.formatPhoneNumber(phoneNumber);

    const message = `Hola ${invoiceData.patientName},

Tu factura *${invoiceData.invoiceNumber}* ha sido generada.

💰 *Monto*: $${invoiceData.amount.toLocaleString('es-AR')}

Podés descargar tu comprobante aquí:
${invoiceData.pdfUrl}

Gracias por tu confianza.`;

    await this.sock.sendMessage(formattedPhone, { text: message });
  }

  private async handleIncomingMessages(messages: WAMessage[]) {
    for (const msg of messages) {
      if (msg.key.fromMe) continue;

      const text = msg.message?.conversation?.toLowerCase() || '';
      const phoneNumber = msg.key.remoteJid;

      // Crisis detection keywords
      const crisisKeywords = [
        'suicidio',
        'quiero morir',
        'no puedo más',
        'ayuda urgente',
      ];
      if (crisisKeywords.some((keyword) => text.includes(keyword))) {
        await this.escalateCrisis(phoneNumber, text);
      }

      // Appointment confirmation handling
      if (text === 'si' || text === 'sí' || text === 'confirmo') {
        await this.processConfirmation(phoneNumber, true);
      } else if (text === 'no' || text === 'reprogramar') {
        await this.processConfirmation(phoneNumber, false);
      }
    }
  }

  private formatPhoneNumber(phone: string): string {
    // Argentine format: +549XXXXXXXXXX
    const cleaned = phone.replace(/\D/g, '');
    return `${cleaned}@s.whatsapp.net`;
  }

  private formatArgentineDateTime(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
    }).format(date);
  }
}
```

### 3. AFIP Electronic Invoicing

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import * as soap from 'soap';

interface AfipInvoiceData {
  invoiceType: 'A' | 'B' | 'C' | 'M';
  salePoint: number;
  invoiceNumber: number;
  date: Date;
  conceptType: number; // 1=Products, 2=Services, 3=Both
  documentType: number; // 80=CUIT, 96=DNI
  documentNumber: string;
  totalAmount: number;
  netAmount: number;
  vatAmount: number;
  serviceFrom?: Date;
  serviceTo?: Date;
}

@Injectable()
export class AfipService {
  private wsaaUrl: string;
  private wsfeUrl: string;
  private certificate: string;
  private privateKey: string;

  constructor() {
    this.wsaaUrl =
      process.env.AFIP_ENVIRONMENT === 'produccion'
        ? 'https://wsaa.afip.gov.ar/ws/services/LoginCms'
        : 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms';

    this.wsfeUrl =
      process.env.AFIP_ENVIRONMENT === 'produccion'
        ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx'
        : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx';

    this.certificate = readFileSync(process.env.AFIP_CERT_PATH, 'utf8');
    this.privateKey = readFileSync(process.env.AFIP_KEY_PATH, 'utf8');
  }

  async generateInvoice(data: AfipInvoiceData) {
    // Step 1: Get authentication token
    const authToken = await this.getAuthToken();

    // Step 2: Get last invoice number
    const lastInvoiceNumber = await this.getLastInvoiceNumber(
      authToken,
      data.salePoint,
      data.invoiceType
    );

    // Step 3: Create invoice
    const invoice = {
      Auth: {
        Token: authToken.token,
        Sign: authToken.sign,
        Cuit: process.env.AFIP_CUIT,
      },
      FeCAEReq: {
        FeCabReq: {
          CantReg: 1,
          PtoVta: data.salePoint,
          CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
        },
        FeDetReq: {
          FECAEDetRequest: {
            Concepto: data.conceptType,
            DocTipo: data.documentType,
            DocNro: data.documentNumber,
            CbteDesde: lastInvoiceNumber + 1,
            CbteHasta: lastInvoiceNumber + 1,
            CbteFch: this.formatAfipDate(data.date),
            ImpTotal: data.totalAmount,
            ImpTotConc: 0,
            ImpNeto: data.netAmount,
            ImpOpEx: 0,
            ImpIVA: data.vatAmount,
            ImpTrib: 0,
            FchServDesde: data.serviceFrom
              ? this.formatAfipDate(data.serviceFrom)
              : null,
            FchServHasta: data.serviceTo
              ? this.formatAfipDate(data.serviceTo)
              : null,
            FchVtoPago: this.formatAfipDate(data.date),
            MonId: 'PES',
            MonCotiz: 1,
            Iva: {
              AlicIva: {
                Id: 5, // 21% VAT
                BaseImp: data.netAmount,
                Importe: data.vatAmount,
              },
            },
          },
        },
      },
    };

    const client = await soap.createClientAsync(this.wsfeUrl);
    const result = await client.FECAESolicitarAsync(invoice);

    return {
      cae: result.FeDetResp.FECAEDetResponse.CAE,
      caeExpiration: result.FeDetResp.FECAEDetResponse.CAEFchVto,
      invoiceNumber: lastInvoiceNumber + 1,
      authorized: true,
    };
  }

  private async getAuthToken() {
    // WSAA authentication implementation
    // Returns { token, sign, expirationTime }
    // Implementation details omitted for brevity
    return {
      token: 'auth_token_from_afip',
      sign: 'signature_from_afip',
    };
  }

  private getInvoiceTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type] || 6;
  }

  private formatAfipDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  async getLastInvoiceNumber(
    authToken: any,
    salePoint: number,
    invoiceType: string
  ): Promise<number> {
    const client = await soap.createClientAsync(this.wsfeUrl);
    const result = await client.FECompUltimoAutorizadoAsync({
      Auth: authToken,
      PtoVta: salePoint,
      CbteTipo: this.getInvoiceTypeCode(invoiceType),
    });
    return result.CbteNro || 0;
  }
}
```

### 4. LiveKit Video Consultation

```typescript
// backend/src/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class VideoService {
  private apiKey: string;
  private apiSecret: string;

  constructor() {
    this.apiKey = process.env.LIVEKIT_API_KEY;
    this.apiSecret = process.env.LIVEKIT_API_SECRET;
  }

  async createSessionToken(data: {
    roomName: string;
    participantName: string;
    participantIdentity: string;
    isPractitioner: boolean;
  }): Promise<string> {
    const at = new AccessToken(this.apiKey, this.apiSecret, {
      identity: data.participantIdentity,
      name: data.participantName,
    });

    at.addGrant({
      roomJoin: true,
      room: data.roomName,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
      // Practitioner gets additional permissions
      canUpdateOwnMetadata: data.isPractitioner,
      recorder: data.isPractitioner,
    });

    return at.toJwt();
  }

  async createVideoRoom(appointmentId: string) {
    const roomName = `session-${appointmentId}`;

    // Room configuration for Argentine bandwidth constraints
    const roomConfig = {
      name: roomName,
      emptyTimeout: 300, // 5 minutes
      maxParticipants: 2,
      metadata: JSON.stringify({
        appointmentId,
        createdAt: new Date().toISOString(),
      }),
    };

    return {
      roomName,
      wsUrl: process.env.LIVEKIT_WS_URL,
      config: roomConfig,
    };
  }
}
```

### 5. Claude AI Orchestration

```typescript
// backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private client: Anthropic;
  private opusModel: string;
  private sonnetModel: string;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.opusModel = process.env.CLAUDE_OPUS_MODEL || 'claude-opus-4.6';
    this.sonnetModel = process.env.CLAUDE_SONNET_MODEL || 'claude-sonnet-4.6';
  }

  async summarizeClinicalNotes(notes: string[], patientContext: string) {
    // Use Opus for deep analysis
    const message = await this.client.messages.create({
      model: this.opusModel,
      max_tokens: 4096,
      temperature: 0.3,
      system: `Sos un asistente clínico especializado en psicología argentina. Tu tarea es resumir notas de sesiones manteniendo información clínicamente relevante y respetando la confidencialidad del paciente.`,
      messages: [
        {
          role: 'user',
          content: `Contexto del paciente: ${patientContext}

Notas de sesiones:
${notes.join('\n\n---\n\n')}

Por favor generá un resumen clínico estructurado que incluya:
1. Evolución del paciente
2. Temas recurrentes
3. Intervenciones realizadas
4. Recomendaciones para próximas sesiones

Formato: español argentino, lenguaje profesional.`,
        },
      ],
    });

    return message.content[0].type === 'text'
      ? message.content[0].text
      : null;
  }

  async generateSessionReport(sessionData: {
    patientId: string;
    practitionerId: string;
    date: Date;
    duration: number;
    clinicalNotes: string;
  }) {
    // Use Sonnet for fast report generation
    const message = await this.client.messages.create({
      model: this.sonnetModel,
      max_tokens: 2048,
      temperature: 0.5,
      messages: [
        {
          role: 'user',
          content: `Generá un informe de sesión estructurado basado en estas notas clínicas:

Fecha: ${sessionData.date.toLocaleDateString('es-AR')}
Duración: ${sessionData.duration} minutos

Notas:
${sessionData.clinicalNotes}

El informe debe seguir el formato estándar para obras sociales argentinas e incluir:
- Motivo de consulta
- Observaciones clínicas
- Intervenciones realizadas
- Indicaciones

Mantené confidencialidad y lenguaje profesional.`,
        },
      ],
    });

    return message.content[0].type === 'text'
      ? message.content[0].text
      : null;
  }

  async assistPractitionerQuery(query: string, patientHistory: any[]) {
    // Use Sonnet for real-time assistance
    const context = patientHistory
      .slice(-10)
      .map((h) => `Sesión ${h.date}: ${h.summary}`)
      .join('\n');

    const message = await this.client.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      temperature: 0.4,
      system: `Sos un asistente clínico que ayuda a psicólogos durante sesiones. Respondé consultas de manera concisa y profesional.`,
      messages: [
        {
          role: 'user',
          content: `Historial reciente del paciente:
${context}

Consulta del profesional: ${query}`,
        },
      ],
    });

    return message.content[0].type === 'text'
      ? message.content[0].text
      : null;
  }

  async detectCrisisRisk(messageContent: string): Promise<{
    isRisk: boolean;
    severity: 'low' | 'medium' | 'high' | 'critical';
    reason: string;
  }> {
    const message = await this.client.messages.create({
      model: this.sonnetModel,
      max_tokens: 256,
      temperature: 0.1,
      system: `Analizá mensajes de pacientes para detectar señales de crisis o riesgo. Respondé en formato JSON.`,
      messages: [
        {
          role: 'user',
          content: `Analiza este mensaje de un paciente y determina si hay riesgo de crisis:

"${messageContent}"

Responde en formato JSON:
{
  "isRisk": boolean,
  "severity": "low" | "medium" | "high" | "critical",
  "reason": "explicación breve"
}`,
        },
      ],
    });

    const response =
      message.content[0].type === 'text' ? message.content[0].text : '{}';
    return JSON.parse(response);
  }
}
```

## Frontend Integration (SvelteKit)

### Appointment Booking Component

```svelte
<!-- frontend/src/lib/components/AppointmentBooker.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import type { Practitioner, TimeSlot } from '$lib/types';

  let practitioners: Practitioner[] = $state([]);
  let selectedPractitioner: string = $state('');
  let availableSlots: TimeSlot[] = $state([]);
  let selectedSlot: TimeSlot | null = $state(null);
  let loading: boolean = $state(false);

  async function loadPractitioners() {
    const res = await fetch('/api/practitioners');
    practitioners = await res.json();
  }

  async function loadSlots(practitionerId: string, date: Date) {
    loading = true;
    const res = await fetch(
      `/api/agenda/slots?practitioner=${practitionerId}&date=${date.toISOString()}`
    );
    availableSlots = await res.json();
    loading = false;
  }

  async function bookAppointment() {
    if (!selectedSlot) return;

    const res = await fetch('/api/agenda/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        practitionerId: selectedPractitioner,
        startTime: selectedSlot.startTime,
        duration: 45,
        sessionType: 'INDIVIDUAL',
      }),
    });

    if (res.ok) {
      alert('¡Turno reservado! Recibirás un WhatsApp de confirmación.');
    }
  }

  onMount(() => {
    loadPractitioners();
  });
</script>

<div class="appointment-booker">
  <h2>Reservar Turno</h2>

  <select bind:value={selectedPractitioner} onchange={() => loadSlots(selectedPractitioner, new Date())}>
    <option value="">Seleccionar profesional</option>
    {#each practitioners as p}
      <option value={p.id}>{p.name} - {p.specialty}</option>
    {/each}
  </select>

  {#if loading}
    <p>Cargando horarios disponibles...</p>
  {:else if availableSlots.length > 0}
    <div class="slots-grid">
      {#each availableSlots as slot}
        <button
          class:selected={selectedSlot === slot}
          onclick={() => selectedSlot = slot}
        >
          {new Date(slot.startTime).toLocaleTimeString('es-AR', {
            hour: '2-digit',
            minute: '2-digit'
          })}
        </button>
      {/each}
    </div>
  {/if}

  {#if selectedSlot}
    <button class="book-btn" onclick={bookAppointment}>
      Confirmar Turno
    </button>
  {/if}
</div>

<style>
  .appointment-booker {
    max-width: 600px;
    margin: 2rem auto;
    padding: 2rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }

  .slots-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 0.5rem;
    margin: 1rem 0;
  }

  .slots-grid button {
    padding: 0.75rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    background: white;
    cursor: pointer;
  }

  .slots-grid button:hover {
    background: #f0f0f0;
  }

  .slots-grid button.selected {
    background: #4a5568;
    color: white;
    border-color: #4a5568;
  }

  .book-btn {
    width: 100%;
    padding: 1rem;
    background: #4299e1;
    color: white;
    border: none;
    border-radius: 4px;
    font-weight: 600;
    cursor: pointer;
  }

  .book-btn:hover {
    background: #3182ce;
  }
</style>
```

## Database Schema (Prisma)

```prisma
// backend/prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Patient {
  id            String        @id @default(cuid())
  firstName     String
  lastName      String
  email         String?       @unique
  phone         String        @unique
  documentType  String        // DNI, CUIT, etc.
  documentNumber String       @unique
  birthDate     DateTime
  address       String?
  city          String?
  province      String?
  
  // Medical insurance
  insuranceName String?
  insuranceNumber String?
  
  appointments  Appointment[]
  invoices      Invoice[]
  clinicalNotes ClinicalNote[]
  
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt
}

model Practitioner {
  id            String        @id @default(cuid())
  firstName     String
