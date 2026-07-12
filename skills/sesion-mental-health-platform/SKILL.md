---
name: sesion-mental-health-platform
description: SaaS platform for psychology clinics with AI scheduling, WhatsApp automation, AFIP billing, and video consultations for Argentine healthcare providers
triggers:
  - "set up Sesión mental health platform"
  - "configure Argentine psychology clinic management system"
  - "integrate WhatsApp automation for patient communications"
  - "implement AFIP compliant electronic invoicing"
  - "add Claude AI orchestration for clinical notes"
  - "configure video consultation with LiveKit"
  - "build mental health practice management workflow"
  - "deploy psychology clinic SaaS platform"
---

# Sesión Mental Health Platform Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Sesión is a comprehensive SaaS platform for psychology clinics and independent practitioners in Argentina. It orchestrates clinical, administrative, and communication workflows including:

- **Intelligent appointment scheduling** with constraint-based optimization
- **WhatsApp automation** via Baileys for patient communications
- **AFIP-compliant electronic invoicing** (facturas A/B/C/M)
- **Secure video consultations** with LiveKit WebRTC
- **AI-powered clinical tools** using Claude Opus 4.6 and Sonnet 4.6
- **Payment processing** with MercadoPago and Stripe

Built with **NestJS** (backend), **SvelteKit** (frontend), **Prisma** (ORM), and **PostgreSQL** (database).

## Architecture Stack

### Backend (NestJS)
- **Framework**: NestJS with TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Real-time**: WebSockets for video/chat
- **Queue**: BullMQ for async jobs
- **Cache**: Redis for session management

### Frontend (SvelteKit)
- **Framework**: SvelteKit with Svelte 5
- **Styling**: TailwindCSS
- **State**: Svelte stores with reactive declarations
- **API**: Fetch with SvelteKit load functions

### Third-Party Integrations
- **WhatsApp**: Baileys (WhatsApp Web API)
- **Video**: LiveKit
- **AI**: Anthropic Claude (Opus 4.6, Sonnet 4.6)
- **Payments**: MercadoPago (Argentina), Stripe (international)
- **AFIP**: Electronic invoicing API integration

## Installation

### Prerequisites

```bash
# Required
node >= 18.x
postgresql >= 14
redis >= 7

# Optional for development
docker >= 20.x
docker-compose >= 2.x
```

### Clone and Setup

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic

# Install dependencies for both backend and frontend
npm install

# Setup database
npx prisma generate
npx prisma migrate dev
```

### Environment Configuration

Create `.env` file in project root:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"

# Redis
REDIS_URL="redis://localhost:6379"

# Authentication
JWT_SECRET="your-jwt-secret-here"
SESSION_SECRET="your-session-secret"

# Anthropic AI
ANTHROPIC_API_KEY="your-anthropic-key"
ANTHROPIC_MODEL_OPUS="claude-opus-4.6"
ANTHROPIC_MODEL_SONNET="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="your-webhook-secret"

# LiveKit Video
LIVEKIT_API_KEY="your-livekit-api-key"
LIVEKIT_API_SECRET="your-livekit-api-secret"
LIVEKIT_URL="wss://your-livekit-server.com"

# AFIP (Argentina Tax Authority)
AFIP_CUIT="your-tax-id"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.pem"
AFIP_PRODUCTION="false"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="your-mercadopago-token"
MERCADOPAGO_PUBLIC_KEY="your-mercadopago-public-key"
STRIPE_SECRET_KEY="your-stripe-secret-key"
STRIPE_WEBHOOK_SECRET="your-stripe-webhook-secret"

# Application
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
PORT="3000"
```

## Running the Platform

### Development Mode

```bash
# Start backend (NestJS)
npm run start:dev

# Start frontend (SvelteKit) - in separate terminal
npm run dev

# Run database migrations
npm run prisma:migrate

# Seed initial data
npm run seed
```

### Production Build

```bash
# Build both backend and frontend
npm run build

# Start production server
npm run start:prod
```

### Docker Deployment

```bash
# Using docker-compose
docker-compose up -d

# Check logs
docker-compose logs -f sesion-backend
docker-compose logs -f sesion-frontend
```

## Core Modules

### 1. Appointment Scheduling

**Backend Service (NestJS)**

```typescript
// appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { AppointmentStatus } from '@prisma/client';

@Injectable()
export class AppointmentsService {
  constructor(private prisma: PrismaService) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number;
    type: 'PRESENCIAL' | 'VIRTUAL' | 'EVALUACION';
  }) {
    // Check for conflicts
    const conflicts = await this.checkConflicts(
      data.practitionerId,
      data.startTime,
      data.duration
    );

    if (conflicts.length > 0) {
      throw new Error('Schedule conflict detected');
    }

    return this.prisma.appointment.create({
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
  }

  private async checkConflicts(
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
            AND: [
              { startTime: { lte: startTime } },
              { endTime: { gt: startTime } },
            ],
          },
          {
            AND: [
              { startTime: { lt: endTime } },
              { endTime: { gte: endTime } },
            ],
          },
        ],
      },
    });
  }

  async findAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number = 45
  ) {
    const dayStart = new Date(date.setHours(8, 0, 0, 0));
    const dayEnd = new Date(date.setHours(20, 0, 0, 0));

    const appointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: { gte: dayStart, lte: dayEnd },
        status: { not: AppointmentStatus.CANCELLED },
      },
      orderBy: { startTime: 'asc' },
    });

    const slots = [];
    let currentTime = dayStart;

    for (const appointment of appointments) {
      while (currentTime.getTime() + duration * 60000 <= appointment.startTime.getTime()) {
        slots.push(new Date(currentTime));
        currentTime = new Date(currentTime.getTime() + 30 * 60000); // 30 min intervals
      }
      currentTime = appointment.endTime;
    }

    // Fill remaining slots until day end
    while (currentTime.getTime() + duration * 60000 <= dayEnd.getTime()) {
      slots.push(new Date(currentTime));
      currentTime = new Date(currentTime.getTime() + 30 * 60000);
    }

    return slots;
  }
}
```

**Frontend Component (Svelte 5)**

```svelte
<!-- src/routes/appointments/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  
  let appointments = $state([]);
  let selectedDate = $state(new Date());
  let availableSlots = $state([]);
  
  async function loadAppointments() {
    const response = await fetch('/api/appointments');
    appointments = await response.json();
  }
  
  async function loadAvailableSlots(practitionerId: string, date: Date) {
    const response = await fetch(
      `/api/appointments/available-slots?practitionerId=${practitionerId}&date=${date.toISOString()}`
    );
    availableSlots = await response.json();
  }
  
  async function bookAppointment(slot: Date, patientId: string) {
    const response = await fetch('/api/appointments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        patientId,
        practitionerId: 'current-practitioner-id',
        startTime: slot.toISOString(),
        duration: 45,
        type: 'VIRTUAL',
      }),
    });
    
    if (response.ok) {
      await loadAppointments();
      availableSlots = [];
    }
  }
  
  onMount(loadAppointments);
</script>

<div class="appointments-container">
  <h1 class="text-2xl font-bold mb-4">Agenda</h1>
  
  <div class="grid grid-cols-2 gap-4">
    <div class="appointments-list">
      <h2 class="text-xl mb-2">Próximas Sesiones</h2>
      {#each appointments as appointment}
        <div class="appointment-card bg-white p-4 rounded shadow">
          <p class="font-semibold">{appointment.patient.name}</p>
          <p class="text-sm text-gray-600">
            {new Date(appointment.startTime).toLocaleString('es-AR')}
          </p>
          <span class="badge {appointment.type.toLowerCase()}">
            {appointment.type}
          </span>
        </div>
      {/each}
    </div>
    
    <div class="slot-picker">
      <h2 class="text-xl mb-2">Horarios Disponibles</h2>
      <input
        type="date"
        bind:value={selectedDate}
        onchange={() => loadAvailableSlots('practitioner-id', selectedDate)}
        class="w-full p-2 border rounded"
      />
      
      {#if availableSlots.length > 0}
        <div class="slots-grid mt-4">
          {#each availableSlots as slot}
            <button
              onclick={() => bookAppointment(slot, 'selected-patient-id')}
              class="slot-button"
            >
              {new Date(slot).toLocaleTimeString('es-AR', { 
                hour: '2-digit', 
                minute: '2-digit' 
              })}
            </button>
          {/each}
        </div>
      {/if}
    </div>
  </div>
</div>
```

### 2. WhatsApp Automation with Baileys

**WhatsApp Service**

```typescript
// whatsapp/whatsapp.service.ts
import { Injectable } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  downloadMediaMessage,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService {
  private socket: any;
  private isConnected = false;

  async initialize() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH || './whatsapp-sessions'
    );

    this.socket = makeWASocket({
      auth: state,
      printQRInTerminal: true,
    });

    this.socket.ev.on('creds.update', saveCreds);
    
    this.socket.ev.on('connection.update', (update) => {
      const { connection, lastDisconnect } = update;
      
      if (connection === 'close') {
        const shouldReconnect = 
          (lastDisconnect?.error as Boom)?.output?.statusCode !== 
          DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.initialize();
        }
      } else if (connection === 'open') {
        this.isConnected = true;
        console.log('WhatsApp connected');
      }
    });

    this.socket.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessages(messages);
    });
  }

  async sendAppointmentReminder(
    phoneNumber: string,
    appointmentData: {
      date: Date;
      practitioner: string;
      type: string;
    }
  ) {
    const formattedNumber = this.formatPhoneNumber(phoneNumber);
    const message = 
      `*Recordatorio de Sesión* 🧠\n\n` +
      `Hola! Te recordamos tu sesión:\n` +
      `📅 ${appointmentData.date.toLocaleDateString('es-AR')}\n` +
      `🕐 ${appointmentData.date.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n` +
      `👨‍⚕️ ${appointmentData.practitioner}\n` +
      `📍 ${appointmentData.type}\n\n` +
      `Por favor confirmá tu asistencia respondiendo *SÍ* o *NO*.`;

    await this.socket.sendMessage(formattedNumber, { text: message });
  }

  async sendInvoice(phoneNumber: string, invoiceUrl: string) {
    const formattedNumber = this.formatPhoneNumber(phoneNumber);
    
    await this.socket.sendMessage(formattedNumber, {
      document: { url: invoiceUrl },
      mimetype: 'application/pdf',
      fileName: 'factura.pdf',
      caption: 'Tu factura electrónica está lista. Gracias! 💙',
    });
  }

  private async handleIncomingMessages(messages: any[]) {
    for (const message of messages) {
      if (message.key.fromMe) continue;

      const text = message.message?.conversation?.toLowerCase() || '';
      const phoneNumber = message.key.remoteJid;

      // Crisis detection
      if (this.containsCrisisKeywords(text)) {
        await this.alertPractitioner(phoneNumber, text);
      }

      // Appointment confirmation
      if (text.includes('sí') || text.includes('si')) {
        await this.confirmAppointment(phoneNumber);
      } else if (text.includes('no')) {
        await this.cancelAppointment(phoneNumber);
      }
    }
  }

  private containsCrisisKeywords(text: string): boolean {
    const keywords = [
      'suicidio',
      'matarme',
      'no quiero vivir',
      'emergencia',
      'ayuda urgente',
    ];
    return keywords.some(keyword => text.includes(keyword));
  }

  private async alertPractitioner(phoneNumber: string, message: string) {
    // Send urgent notification to practitioner
    const practitionerNumber = await this.getPractitionerNumber(phoneNumber);
    
    await this.socket.sendMessage(practitionerNumber, {
      text: 
        `⚠️ *ALERTA DE CRISIS*\n\n` +
        `Paciente: ${phoneNumber}\n` +
        `Mensaje: "${message}"\n\n` +
        `Se requiere atención inmediata.`,
    });
  }

  private formatPhoneNumber(phone: string): string {
    // Convert to WhatsApp format: country code + number + @s.whatsapp.net
    const cleaned = phone.replace(/[^0-9]/g, '');
    return `${cleaned}@s.whatsapp.net`;
  }

  private async getPractitionerNumber(patientNumber: string): Promise<string> {
    // Fetch from database
    return 'practitioner-number@s.whatsapp.net';
  }

  private async confirmAppointment(phoneNumber: string) {
    // Update appointment status in database
  }

  private async cancelAppointment(phoneNumber: string) {
    // Handle cancellation logic
  }
}
```

### 3. AFIP Electronic Invoicing

**AFIP Service**

```typescript
// billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import { readFileSync } from 'fs';
import * as soap from 'soap';
import { createSign } from 'crypto';

@Injectable()
export class AfipService {
  private wsaa: any; // Web Service Authentication and Authorization
  private wsfe: any; // Web Service Electronic Invoice

  async authenticate() {
    const tra = this.generateTRA();
    const cms = this.signTRA(tra);
    
    const client = await soap.createClientAsync(
      process.env.AFIP_PRODUCTION === 'true'
        ? 'https://wsaa.afip.gov.ar/ws/services/LoginCms?wsdl'
        : 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms?wsdl'
    );

    const result = await client.loginCmsAsync({
      in0: cms,
    });

    const token = this.extractFromXML(result[0].loginCmsReturn, 'token');
    const sign = this.extractFromXML(result[0].loginCmsReturn, 'sign');

    return { token, sign };
  }

  async generateInvoice(data: {
    invoiceType: 'A' | 'B' | 'C' | 'M';
    patientName: string;
    patientTaxId: string;
    amount: number;
    sessionDate: Date;
    concept: string;
  }) {
    const auth = await this.authenticate();
    
    const client = await soap.createClientAsync(
      process.env.AFIP_PRODUCTION === 'true'
        ? 'https://servicios1.afip.gov.ar/wsfev1/service.asmx?WSDL'
        : 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx?WSDL'
    );

    const lastInvoiceNumber = await this.getLastInvoiceNumber(
      client,
      auth,
      data.invoiceType
    );

    const invoice = {
      Auth: {
        Token: auth.token,
        Sign: auth.sign,
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
            Concepto: 2, // Services
            DocTipo: 80, // CUIT
            DocNro: data.patientTaxId,
            CbteDesde: lastInvoiceNumber + 1,
            CbteHasta: lastInvoiceNumber + 1,
            CbteFch: this.formatDate(data.sessionDate),
            ImpTotal: data.amount,
            ImpTotConc: 0,
            ImpNeto: data.amount,
            ImpOpEx: 0,
            ImpIVA: 0,
            ImpTrib: 0,
            FchServDesde: this.formatDate(data.sessionDate),
            FchServHasta: this.formatDate(data.sessionDate),
            FchVtoPago: this.formatDate(data.sessionDate),
            MonId: 'PES',
            MonCotiz: 1,
          },
        },
      },
    };

    const result = await client.FECAESolicitarAsync(invoice);
    
    return {
      cae: result[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse.CAE,
      caeExpiration: result[0].FECAESolicitarResult.FeDetResp.FECAEDetResponse.CAEFchVto,
      invoiceNumber: lastInvoiceNumber + 1,
      invoiceType: data.invoiceType,
    };
  }

  private generateTRA(): string {
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

  private signTRA(tra: string): string {
    const cert = readFileSync(process.env.AFIP_CERT_PATH);
    const key = readFileSync(process.env.AFIP_KEY_PATH);
    
    const sign = createSign('SHA256');
    sign.update(tra);
    const signature = sign.sign(key, 'base64');
    
    return Buffer.from(
      `${tra}\n${signature}\n${cert.toString('base64')}`
    ).toString('base64');
  }

  private getInvoiceTypeCode(type: string): number {
    const codes = { A: 1, B: 6, C: 11, M: 51 };
    return codes[type] || 6;
  }

  private formatDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  private extractFromXML(xml: string, tag: string): string {
    const regex = new RegExp(`<${tag}>([^<]+)</${tag}>`);
    const match = xml.match(regex);
    return match ? match[1] : '';
  }

  private async getLastInvoiceNumber(
    client: any,
    auth: any,
    invoiceType: string
  ): Promise<number> {
    const result = await client.FECompUltimoAutorizadoAsync({
      Auth: {
        Token: auth.token,
        Sign: auth.sign,
        Cuit: process.env.AFIP_CUIT,
      },
      PtoVta: 1,
      CbteTipo: this.getInvoiceTypeCode(invoiceType),
    });

    return result[0].FECompUltimoAutorizadoResult.CbteNro || 0;
  }
}
```

### 4. AI Orchestration with Claude

**AI Service with Model Router**

```typescript
// ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(
    sessionNotes: string[],
    patientHistory: string
  ): Promise<string> {
    // Use Opus for deep analysis
    const response = await this.client.messages.create({
      model: process.env.ANTHROPIC_MODEL_OPUS || 'claude-opus-4.6',
      max_tokens: 4000,
      temperature: 0.3,
      system: 
        'Eres un asistente clínico especializado en psicología. ' +
        'Analiza las notas de sesión y genera un resumen profesional, ' +
        'identificando patrones, progresos y áreas de atención.',
      messages: [
        {
          role: 'user',
          content: 
            `Historia del paciente:\n${patientHistory}\n\n` +
            `Notas de las últimas sesiones:\n${sessionNotes.join('\n\n---\n\n')}\n\n` +
            `Por favor, proporciona un resumen clínico estructurado.`,
        },
      ],
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  }

  async generateSessionNote(
    sessionData: {
      duration: number;
      mainTopics: string[];
      observations: string;
    }
  ): Promise<string> {
    // Use Sonnet for quick note generation
    const response = await this.client.messages.create({
      model: process.env.ANTHROPIC_MODEL_SONNET || 'claude-sonnet-4.6',
      max_tokens: 1500,
      temperature: 0.5,
      system: 
        'Eres un asistente que ayuda a psicólogos a redactar notas de sesión ' +
        'en formato profesional y estructurado.',
      messages: [
        {
          role: 'user',
          content: 
            `Genera una nota de sesión basada en:\n` +
            `Duración: ${sessionData.duration} minutos\n` +
            `Temas abordados: ${sessionData.mainTopics.join(', ')}\n` +
            `Observaciones: ${sessionData.observations}`,
        },
      ],
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  }

  async answerClinicalQuery(query: string, context: string): Promise<string> {
    // Use Sonnet for real-time assistant
    const response = await this.client.messages.create({
      model: process.env.ANTHROPIC_MODEL_SONNET || 'claude-sonnet-4.6',
      max_tokens: 1000,
      temperature: 0.7,
      system: 
        'Eres un asistente clínico que ayuda a psicólogos durante las sesiones. ' +
        'Proporciona información relevante de forma concisa.',
      messages: [
        {
          role: 'user',
          content: 
            `Contexto del paciente:\n${context}\n\n` +
            `Consulta: ${query}`,
        },
      ],
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  }

  async detectRiskFactors(conversationHistory: string[]): Promise<{
    riskLevel: 'low' | 'medium' | 'high';
    factors: string[];
    recommendations: string[];
  }> {
    const response = await this.client.messages.create({
      model: process.env.ANTHROPIC_MODEL_OPUS || 'claude-opus-4.6',
      max_tokens: 2000,
      temperature: 0.2,
      system: 
        'Analiza conversaciones clínicas para identificar factores de riesgo ' +
        'de salud mental. Responde en formato JSON.',
      messages: [
        {
          role: 'user',
          content: 
            `Analiza esta conversación y detecta factores de riesgo:\n\n` +
            `${conversationHistory.join('\n\n')}\n\n` +
            `Responde en JSON con: riskLevel, factors[], recommendations[]`,
        },
      ],
    });

    const text = response.content[0].type === 'text' 
      ? response.content[0].text 
      : '{}';
    
    return JSON.parse(text);
  }
}
```

**Frontend AI Assistant Component**

```svelte
<!-- src/lib/components/AIAssistant.svelte -->
<script lang="ts">
  let query = $state('');
  let response = $state('');
  let isLoading = $state(false);
  
  async function askAssistant() {
    if (!query.trim()) return;
    
    isLoading = true;
    
    try {
      const res = await fetch('/api/ai/query', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          query,
          context: 'current-patient-context',
        }),
      });
      
      const data = await res.json();
      response = data.answer;
    } catch (error) {
      response = 'Error al contactar al asistente. Intenta nuevamente.';
    } finally {
      isLoading = false;
    }
  }
</script>

<div class="ai-assistant bg-blue-50 p-4 rounded-lg">
  <h3 class="text-lg font-semibold mb-3">Asistente Clínico 🤖</h3>
  
  <div class="flex gap-2 mb-4">
    <input
      type="text"
      bind:value={query}
      onkeydown={(e) => e.key === 'Enter' && askAssistant()}
      placeholder="Pregunta algo sobre este paciente..."
      class="flex-1 p-2 border rounded"
      disabled={isLoading}
    />
    <button
      onclick={askAssistant}
      disabled={isLoading}
      class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
    >
      {isLoading ? 'Pensando...' : 'Preguntar'}
    </button>
  </div>
  
  {#if response}
    <div class="response bg-white p-3 rounded border border-blue-200">
      <p class="text-sm whitespace-pre-wrap">{response}</p>
    </div>
  {/if}
</div>
```

### 5. Video Consultations with LiveKit

**LiveKit Service**

```typescript
// video/livekit.service.ts
import { Injectable } from '@nestjs/common
