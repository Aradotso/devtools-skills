---
name: sesion-mental-health-clinic-platform
description: AI-powered SaaS platform for psychology clinics with scheduling, WhatsApp automation, AFIP billing, video calls, and Claude integration
triggers:
  - how do I set up Sesión for a psychology clinic
  - integrate WhatsApp automation with Sesión
  - configure AFIP billing in Sesión
  - use Claude AI for clinical notes in Sesión
  - set up video consultations in Sesión platform
  - implement appointment scheduling with Sesión
  - configure Sesión for Argentine psychology practice
  - troubleshoot Sesión WhatsApp messaging
---

# Sesión Mental Health Clinic Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive SaaS platform for psychology clinics in Argentina, orchestrating appointment scheduling, WhatsApp automation, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6.

## Tech Stack

- **Frontend**: SvelteKit 5 + TypeScript + Tailwind CSS
- **Backend**: NestJS + Prisma + PostgreSQL
- **AI**: Anthropic Claude (Opus 4.6, Sonnet 4.6)
- **WhatsApp**: Baileys library
- **Video**: LiveKit
- **Payments**: Stripe, Mercado Pago
- **Storage**: PostgreSQL, Redis, Elasticsearch, MinIO

## Installation & Setup

### Prerequisites

```bash
# Required versions
node >= 20.x
pnpm >= 8.x
postgresql >= 15.x
redis >= 7.x
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic
pnpm install
```

### Environment Configuration

Create `.env` file in project root:

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY=your_anthropic_key_here
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./sessions/whatsapp"
WHATSAPP_WEBHOOK_SECRET=your_webhook_secret

# LiveKit Video
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_secret
LIVEKIT_WS_URL="wss://your-livekit-instance.com"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN=your_mercadopago_token
STRIPE_SECRET_KEY=your_stripe_secret_key

# AFIP (Argentina Tax)
AFIP_CUIT=your_clinic_cuit
AFIP_CERT_PATH="./certs/afip.pem"
AFIP_KEY_PATH="./certs/afip.key"

# App Configuration
APP_URL="http://localhost:5173"
API_URL="http://localhost:3000"
JWT_SECRET=your_jwt_secret
SESSION_TIMEOUT_MINUTES=15
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

### Start Development Servers

```bash
# Start backend (NestJS)
cd backend
pnpm dev

# Start frontend (SvelteKit) - in separate terminal
cd frontend
pnpm dev
```

## Core Architecture Patterns

### 1. Appointment Scheduling

**Backend Service (NestJS)**

```typescript
// backend/src/appointments/appointments.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AppointmentsService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: {
    patientId: string;
    practitionerId: string;
    startTime: Date;
    duration: number;
    type: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Check for scheduling conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        startTime: {
          lte: new Date(data.startTime.getTime() + data.duration * 60000),
        },
        endTime: {
          gte: data.startTime,
        },
        status: { notIn: ['cancelled'] },
      },
    });

    if (conflicts.length > 0) {
      throw new Error('Scheduling conflict detected');
    }

    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime: new Date(data.startTime.getTime() + data.duration * 60000),
        status: 'scheduled',
      },
      include: {
        patient: true,
        practitioner: true,
      },
    });

    // Send WhatsApp confirmation
    await this.whatsapp.sendAppointmentConfirmation(appointment);

    return appointment;
  }

  async scheduleReminders(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true },
    });

    // Schedule 24h reminder
    const reminder24h = new Date(appointment.startTime.getTime() - 24 * 60 * 60 * 1000);
    await this.prisma.reminder.create({
      data: {
        appointmentId,
        scheduledFor: reminder24h,
        type: '24h',
        channel: 'whatsapp',
      },
    });

    // Schedule 2h reminder
    const reminder2h = new Date(appointment.startTime.getTime() - 2 * 60 * 60 * 1000);
    await this.prisma.reminder.create({
      data: {
        appointmentId,
        scheduledFor: reminder2h,
        type: '2h',
        channel: 'whatsapp',
      },
    });
  }
}
```

**Frontend Component (Svelte 5)**

```svelte
<!-- frontend/src/routes/agenda/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import Calendar from '$lib/components/Calendar.svelte';
  import { appointmentsApi } from '$lib/api/appointments';
  
  let appointments = $state([]);
  let selectedDate = $state(new Date());
  let practitioners = $state([]);

  onMount(async () => {
    await loadAppointments();
    await loadPractitioners();
  });

  async function loadAppointments() {
    const response = await appointmentsApi.list({
      startDate: selectedDate,
      endDate: new Date(selectedDate.getTime() + 7 * 24 * 60 * 60 * 1000),
    });
    appointments = response.data;
  }

  async function createAppointment(event) {
    const { patientId, practitionerId, startTime, type } = event.detail;
    
    try {
      await appointmentsApi.create({
        patientId,
        practitionerId,
        startTime: new Date(startTime),
        duration: 45, // Default 45 minutes
        type,
      });
      
      await loadAppointments();
      toast.success('Turno creado exitosamente');
    } catch (error) {
      toast.error('Error al crear turno: ' + error.message);
    }
  }
</script>

<div class="agenda-container">
  <h1>Agenda Inteligente</h1>
  
  <Calendar 
    {appointments}
    {practitioners}
    {selectedDate}
    on:create={createAppointment}
    on:dateChange={(e) => selectedDate = e.detail}
  />
</div>
```

### 2. WhatsApp Automation with Baileys

```typescript
// backend/src/whatsapp/whatsapp.service.ts
import { Injectable } from '@nestjs/common';
import makeWASocket, { 
  DisconnectReason, 
  useMultiFileAuthState,
  MessageType 
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';

@Injectable()
export class WhatsAppService {
  private sock;
  private connected = false;

  async initialize() {
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
        const shouldReconnect = 
          (lastDisconnect?.error as Boom)?.output?.statusCode !== 
          DisconnectReason.loggedOut;
        
        if (shouldReconnect) {
          this.initialize();
        }
      } else if (connection === 'open') {
        this.connected = true;
        console.log('WhatsApp connected successfully');
      }
    });

    // Handle incoming messages
    this.sock.ev.on('messages.upsert', async (m) => {
      await this.handleIncomingMessage(m);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    const message = this.buildConfirmationMessage(appointment);
    const phoneNumber = this.formatPhoneNumber(appointment.patient.phone);

    await this.sock.sendMessage(phoneNumber, {
      text: message,
    });

    await this.prisma.whatsappMessage.create({
      data: {
        to: phoneNumber,
        message,
        type: 'appointment_confirmation',
        appointmentId: appointment.id,
        sentAt: new Date(),
      },
    });
  }

  async sendReminder(appointment: any, reminderType: '24h' | '2h') {
    const templates = {
      '24h': `Hola ${appointment.patient.firstName}! 👋\n\nTe recordamos tu sesión mañana:\n📅 ${this.formatDate(appointment.startTime)}\n⏰ ${this.formatTime(appointment.startTime)}\n\nPor favor confirmá tu asistencia respondiendo SÍ.`,
      '2h': `Hola ${appointment.patient.firstName}! 👋\n\nTu sesión comienza en 2 horas:\n⏰ ${this.formatTime(appointment.startTime)}\n📍 ${appointment.type === 'virtual' ? 'Videollamada' : 'Consultorio'}\n\nTe esperamos!`,
    };

    const phoneNumber = this.formatPhoneNumber(appointment.patient.phone);
    await this.sock.sendMessage(phoneNumber, {
      text: templates[reminderType],
    });
  }

  async sendInvoice(invoice: any) {
    const phoneNumber = this.formatPhoneNumber(invoice.patient.phone);
    const pdfBuffer = await this.generateInvoicePDF(invoice);

    await this.sock.sendMessage(phoneNumber, {
      document: pdfBuffer,
      mimetype: 'application/pdf',
      fileName: `Factura_${invoice.number}.pdf`,
      caption: `Factura ${invoice.number} - Total: $${invoice.total}`,
    });
  }

  private async handleIncomingMessage(m: any) {
    const message = m.messages[0];
    if (!message.message) return;

    const text = message.message.conversation || 
                 message.message.extendedTextMessage?.text || '';
    const from = message.key.remoteJid;

    // Crisis keyword detection
    const crisisKeywords = ['suicidio', 'morir', 'crisis', 'emergencia'];
    if (crisisKeywords.some(keyword => text.toLowerCase().includes(keyword))) {
      await this.handleCrisisMessage(from, text);
      return;
    }

    // Appointment confirmation parsing
    if (['si', 'sí', 'confirmo', 'ok'].includes(text.toLowerCase())) {
      await this.processConfirmation(from);
    }
  }

  private formatPhoneNumber(phone: string): string {
    // Convert to WhatsApp format: 549XXXXXXXXXX@s.whatsapp.net
    const cleaned = phone.replace(/\D/g, '');
    return `549${cleaned}@s.whatsapp.net`;
  }

  private buildConfirmationMessage(appointment: any): string {
    return `Hola ${appointment.patient.firstName}! 👋

Tu turno ha sido confirmado:

📅 Fecha: ${this.formatDate(appointment.startTime)}
⏰ Hora: ${this.formatTime(appointment.startTime)}
👨‍⚕️ Profesional: ${appointment.practitioner.name}
📍 Modalidad: ${appointment.type === 'virtual' ? 'Videollamada' : 'Presencial'}

Por favor confirmá tu asistencia respondiendo a este mensaje.`;
  }

  private formatDate(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      weekday: 'long',
      day: '2-digit',
      month: 'long',
      year: 'numeric',
    }).format(date);
  }

  private formatTime(date: Date): string {
    return new Intl.DateTimeFormat('es-AR', {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false,
    }).format(date);
  }
}
```

### 3. AFIP Electronic Billing Integration

```typescript
// backend/src/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as soap from 'soap';
import * as crypto from 'crypto';

@Injectable()
export class AFIPService {
  private wsaaUrl = 'https://wsaa.afip.gov.ar/ws/services/LoginCms';
  private wsfevUrl = 'https://servicios1.afip.gov.ar/wsfev1/service.asmx';

  async getAuthToken(): Promise<{ token: string; sign: string }> {
    const tra = this.generateTRA();
    const cms = await this.signTRA(tra);
    
    const client = await soap.createClientAsync(this.wsaaUrl);
    const result = await client.loginCmsAsync({
      in0: cms,
    });

    const xml = result[0].loginCmsReturn;
    const token = this.extractFromXML(xml, 'token');
    const sign = this.extractFromXML(xml, 'sign');

    return { token, sign };
  }

  async generateInvoice(data: {
    patientName: string;
    sessionDate: Date;
    amount: number;
    invoiceType: 'A' | 'B' | 'C';
    conceptType: 'Servicios';
  }) {
    const { token, sign } = await this.getAuthToken();
    
    const client = await soap.createClientAsync(this.wsfevUrl);
    
    // Get last invoice number
    const lastInvoice = await client.FECompUltimoAutorizadoAsync({
      Auth: {
        Token: token,
        Sign: sign,
        Cuit: process.env.AFIP_CUIT,
      },
      PtoVta: 1,
      CbteTipo: this.getInvoiceTypeCode(data.invoiceType),
    });

    const nextNumber = lastInvoice[0].FECompUltimoAutorizadoResult.CbteNro + 1;

    // Create invoice
    const invoice = {
      Auth: {
        Token: token,
        Sign: sign,
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
            DocTipo: 96, // DNI
            DocNro: 0, // Consumidor final
            CbteDesde: nextNumber,
            CbteHasta: nextNumber,
            CbteFch: this.formatAFIPDate(new Date()),
            ImpTotal: data.amount,
            ImpTotConc: 0,
            ImpNeto: data.invoiceType === 'A' ? data.amount / 1.21 : data.amount,
            ImpOpEx: 0,
            ImpIVA: data.invoiceType === 'A' ? data.amount - (data.amount / 1.21) : 0,
            ImpTrib: 0,
            MonId: 'PES',
            MonCotiz: 1,
            Iva: data.invoiceType === 'A' ? {
              AlicIva: {
                Id: 5, // 21%
                BaseImp: data.amount / 1.21,
                Importe: data.amount - (data.amount / 1.21),
              },
            } : null,
          },
        },
      },
    };

    const result = await client.FECAESolicitarAsync(invoice);
    const response = result[0].FECAESolicitarResult;

    if (response.Errors) {
      throw new Error(`AFIP Error: ${response.Errors.Err[0].Msg}`);
    }

    const cae = response.FeDetResp.FECAEDetResponse[0].CAE;
    const caeExpiration = response.FeDetResp.FECAEDetResponse[0].CAEFchVto;

    // Save to database
    await this.prisma.invoice.create({
      data: {
        number: `${String(1).padStart(4, '0')}-${String(nextNumber).padStart(8, '0')}`,
        type: data.invoiceType,
        amount: data.amount,
        cae,
        caeExpiration: this.parseAFIPDate(caeExpiration),
        patientName: data.patientName,
        sessionDate: data.sessionDate,
        issuedAt: new Date(),
      },
    });

    return { number: nextNumber, cae, caeExpiration };
  }

  private generateTRA(): string {
    const now = new Date();
    const expirationTime = new Date(now.getTime() + 12 * 60 * 60 * 1000); // 12 hours

    return `<?xml version="1.0" encoding="UTF-8"?>
<loginTicketRequest version="1.0">
  <header>
    <uniqueId>${Date.now()}</uniqueId>
    <generationTime>${now.toISOString()}</generationTime>
    <expirationTime>${expirationTime.toISOString()}</expirationTime>
  </header>
  <service>wsfe</service>
</loginTicketRequest>`;
  }

  private async signTRA(tra: string): Promise<string> {
    const cert = fs.readFileSync(process.env.AFIP_CERT_PATH);
    const key = fs.readFileSync(process.env.AFIP_KEY_PATH);

    const sign = crypto.createSign('SHA256');
    sign.update(tra);
    const signature = sign.sign(key, 'base64');

    return `-----BEGIN PKCS7-----\n${signature}\n-----END PKCS7-----`;
  }

  private getInvoiceTypeCode(type: 'A' | 'B' | 'C'): number {
    const codes = { A: 1, B: 6, C: 11 };
    return codes[type];
  }

  private formatAFIPDate(date: Date): string {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}${month}${day}`;
  }

  private parseAFIPDate(dateString: string): Date {
    const year = dateString.substring(0, 4);
    const month = dateString.substring(4, 6);
    const day = dateString.substring(6, 8);
    return new Date(`${year}-${month}-${day}`);
  }

  private extractFromXML(xml: string, tag: string): string {
    const regex = new RegExp(`<${tag}>(.*?)</${tag}>`);
    const match = xml.match(regex);
    return match ? match[1] : '';
  }
}
```

### 4. Claude AI Integration for Clinical Notes

```typescript
// backend/src/ai/claude.service.ts
import { Injectable } from '@nestjs/common';
import Anthropic from '@anthropic-ai/sdk';

@Injectable()
export class ClaudeService {
  private anthropic: Anthropic;
  private opusModel = process.env.CLAUDE_OPUS_MODEL;
  private sonnetModel = process.env.CLAUDE_SONNET_MODEL;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async summarizeClinicalNotes(notes: string[]): Promise<string> {
    // Use Opus for deep analysis of patient history
    const message = await this.anthropic.messages.create({
      model: this.opusModel,
      max_tokens: 2048,
      system: `Sos un asistente clínico especializado en psicología para profesionales argentinos. 
Tu tarea es analizar y resumir notas de sesiones terapéuticas, manteniendo confidencialidad 
y usando terminología clínica apropiada según estándares argentinos.`,
      messages: [{
        role: 'user',
        content: `Resume las siguientes notas clínicas, identificando:
1. Motivo de consulta principal
2. Evolución del cuadro
3. Intervenciones realizadas
4. Objetivos terapéuticos
5. Próximos pasos sugeridos

Notas:
${notes.join('\n\n---\n\n')}`,
      }],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateSessionNote(
    sessionData: {
      patientName: string;
      sessionNumber: number;
      date: Date;
      rawNotes: string;
    }
  ): Promise<string> {
    // Use Sonnet for quick note formatting
    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      system: `Sos un asistente que ayuda a psicólogos argentinos a formatear notas clínicas.
Generá notas profesionales y estructuradas según estándares del Colegio de Psicólogos.`,
      messages: [{
        role: 'user',
        content: `Formateá esta nota clínica para la sesión ${sessionData.sessionNumber} de ${sessionData.patientName}:

Fecha: ${sessionData.date.toLocaleDateString('es-AR')}

Notas brutas:
${sessionData.rawNotes}

Estructura requerida:
- Observaciones de la sesión
- Temas abordados
- Intervenciones técnicas
- Plan para próxima sesión`,
      }],
    });

    return message.content[0].type === 'text'
      ? message.content[0].text
      : '';
  }

  async detectCrisisRisk(messageText: string): Promise<{
    isHighRisk: boolean;
    confidence: number;
    recommendedAction: string;
  }> {
    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 512,
      system: `Analizá mensajes de pacientes para detectar signos de crisis o riesgo suicida.
Respondé en formato JSON con: isHighRisk (boolean), confidence (0-1), recommendedAction (string).`,
      messages: [{
        role: 'user',
        content: `Evaluá el siguiente mensaje de un paciente:\n\n"${messageText}"`,
      }],
    });

    const response = message.content[0].type === 'text'
      ? message.content[0].text
      : '{}';

    return JSON.parse(response);
  }

  async chatAssistant(
    conversationHistory: Array<{ role: 'user' | 'assistant'; content: string }>,
    userQuery: string
  ): Promise<string> {
    // Real-time assistant for practitioners
    const messages = [
      ...conversationHistory.map(msg => ({
        role: msg.role as 'user' | 'assistant',
        content: msg.content,
      })),
      {
        role: 'user' as const,
        content: userQuery,
      },
    ];

    const message = await this.anthropic.messages.create({
      model: this.sonnetModel,
      max_tokens: 1024,
      system: `Sos un asistente virtual para psicólogos argentinos. Ayudás con:
- Búsqueda de información de pacientes
- Generación de informes
- Recordatorios de tareas clínicas
- Sugerencias de intervenciones basadas en evidencia
Siempre mantené la confidencialidad y ética profesional.`,
      messages,
    });

    return message.content[0].type === 'text'
      ? message.content[0].text
      : '';
  }
}
```

### 5. LiveKit Video Consultation Setup

```typescript
// backend/src/video/livekit.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';

@Injectable()
export class LiveKitService {
  private apiKey = process.env.LIVEKIT_API_KEY;
  private apiSecret = process.env.LIVEKIT_API_SECRET;
  private wsUrl = process.env.LIVEKIT_WS_URL;

  async createVideoRoom(appointmentId: string): Promise<{
    roomName: string;
    practitionerToken: string;
    patientToken: string;
  }> {
    const roomName = `session-${appointmentId}`;

    // Generate practitioner token (host privileges)
    const practitionerToken = new AccessToken(
      this.apiKey,
      this.apiSecret,
      {
        identity: `practitioner-${appointmentId}`,
        ttl: '2h',
      }
    );
    practitionerToken.addGrant({
      room: roomName,
      roomJoin: true,
      canPublish: true,
      canSubscribe: true,
      canPublishData: true,
    });

    // Generate patient token (participant privileges)
    const patientToken = new AccessToken(
      this.apiKey,
      this.apiSecret,
      {
        identity: `patient-${appointmentId}`,
        ttl: '2h',
      }
    );
    patientToken.addGrant({
      room: roomName,
      roomJoin: true,
      canPublish: true,
      canSubscribe: true,
    });

    return {
      roomName,
      practitionerToken: await practitionerToken.toJwt(),
      patientToken: await patientToken.toJwt(),
    };
  }

  getConnectionUrl(): string {
    return this.wsUrl;
  }
}
```

**Frontend Video Component**

```svelte
<!-- frontend/src/lib/components/VideoCall.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { Room, RoomEvent } from 'livekit-client';
  
  export let token: string;
  export let wsUrl: string;
  
  let videoContainer: HTMLDivElement;
  let room: Room;
  let isConnected = $state(false);
  let participants = $state([]);
  
  onMount(async () => {
    room = new Room({
      adaptiveStream: true,
      dynacast: true,
      videoCaptureDefaults: {
        resolution: {
          width: 1280,
          height: 720,
          frameRate: 30,
        },
      },
    });

    room.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    room.on(RoomEvent.TrackUnsubscribed, handleTrackUnsubscribed);
    room.on(RoomEvent.Disconnected, handleDisconnect);

    await room.connect(wsUrl, token);
    isConnected = true;

    // Publish local tracks
    await room.localParticipant.enableCameraAndMicrophone();
  });

  function handleTrackSubscribed(track, publication, participant) {
    if (track.kind === 'video') {
      const element = track.attach();
      videoContainer.appendChild(element);
    }
  }

  function handleTrackUnsubscribed(track) {
    track.detach().forEach(el => el.remove());
  }

  async function toggleMicrophone() {
    await room.localParticipant.setMicrophoneEnabled(
      !room.localParticipant.isMicrophoneEnabled
    );
  }

  async function toggleCamera() {
    await room.localParticipant.setCameraEnabled(
      !room.localParticipant.isCameraEnabled
    );
  }

  async function endCall() {
    await room.disconnect();
  }

  onDestroy(() => {
    room?.disconnect();
  });
</script>

<div class="video-call">
  <div bind:this={videoContainer} class="video-container"></div>
  
  <div class="controls">
    <button onclick={toggleMicrophone}>
      {room?.localParticipant.isMicrophoneEnabled ? 'Silenciar' : 'Activar Micrófono'}
    </button>
    <button onclick={toggleCamera}>
      {room?.localParticipant.isCameraEnabled ? 'Apagar Cámara' : 'Encender Cámara'}
    </button>
    <button onclick={endCall} class="end-call">Finalizar
