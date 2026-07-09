---
name: sesion-clinic-workflow
description: AI-powered mental health practice management platform for Argentina with WhatsApp automation, AFIP-compliant invoicing, and video consultations
triggers:
  - "set up Sesión clinic workflow platform"
  - "integrate WhatsApp automation for patient appointments"
  - "configure AFIP electronic invoicing for Argentina"
  - "implement video consultation system with Sesión"
  - "use Claude AI for clinical note summarization"
  - "manage psychology clinic scheduling and billing"
  - "deploy Sesión mental health practice SaaS"
  - "configure Argentina tax compliance for clinics"
---

# Sesión Clinic Workflow Platform

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Sesión is a comprehensive mental health practice orchestration platform built for Argentina's psychology clinics. It unifies appointment scheduling, WhatsApp automation, AFIP-compliant billing, secure video consultations, and AI-powered clinical assistance using Claude Opus 4.6 and Sonnet 4.6 models. The platform is architected as a microservices ecosystem with NestJS backend, SvelteKit frontend, Prisma ORM, and integration with Baileys (WhatsApp), LiveKit (video), and Anthropic (AI).

## Installation

### Prerequisites

```bash
# Required tools
node >= 18.0.0
pnpm >= 8.0.0
docker >= 24.0.0
postgresql >= 15.0
redis >= 7.0
```

### Clone and Install

```bash
git clone https://github.com/fahad-hamid/psique-workflow-clinic.git
cd psique-workflow-clinic
pnpm install
```

### Environment Configuration

Create `.env` files for backend and frontend:

**Backend `.env`:**
```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/sesion_db"
REDIS_URL="redis://localhost:6379"

# Anthropic AI
ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
CLAUDE_OPUS_MODEL="claude-opus-4.6"
CLAUDE_SONNET_MODEL="claude-sonnet-4.6"

# WhatsApp (Baileys)
WHATSAPP_SESSION_PATH="./whatsapp-sessions"
WHATSAPP_WEBHOOK_SECRET="${WHATSAPP_WEBHOOK_SECRET}"

# AFIP Integration
AFIP_CUIT="${AFIP_CUIT}"
AFIP_CERT_PATH="./certs/afip-cert.pem"
AFIP_KEY_PATH="./certs/afip-key.key"
AFIP_ENVIRONMENT="production" # or "homologacion"

# LiveKit Video
LIVEKIT_API_KEY="${LIVEKIT_API_KEY}"
LIVEKIT_API_SECRET="${LIVEKIT_API_SECRET}"
LIVEKIT_URL="wss://your-livekit-server.com"

# Payment Providers
MERCADOPAGO_ACCESS_TOKEN="${MERCADOPAGO_ACCESS_TOKEN}"
STRIPE_SECRET_KEY="${STRIPE_SECRET_KEY}"

# JWT
JWT_SECRET="${JWT_SECRET}"
JWT_EXPIRATION="7d"
```

**Frontend `.env`:**
```bash
PUBLIC_API_URL="http://localhost:3000/api"
PUBLIC_LIVEKIT_URL="wss://your-livekit-server.com"
```

### Database Setup

```bash
# Run migrations
cd backend
pnpm prisma migrate deploy

# Seed initial data (optional)
pnpm prisma db seed
```

### Start Development Servers

```bash
# Terminal 1: Backend (NestJS)
cd backend
pnpm dev

# Terminal 2: Frontend (SvelteKit)
cd frontend
pnpm dev

# Terminal 3: Redis (Docker)
docker run -d -p 6379:6379 redis:7-alpine
```

## Core Architecture

### Tech Stack

- **Backend**: NestJS (TypeScript), Prisma ORM, PostgreSQL, Redis
- **Frontend**: SvelteKit 5, Tailwind CSS, TypeScript
- **WhatsApp**: Baileys library for multi-device support
- **Video**: LiveKit WebRTC infrastructure
- **AI**: Anthropic Claude Opus 4.6 and Sonnet 4.6
- **Billing**: AFIP SDK, Mercado Pago, Stripe integrations

### Project Structure

```
psique-workflow-clinic/
├── backend/
│   ├── src/
│   │   ├── modules/
│   │   │   ├── agenda/          # Scheduling logic
│   │   │   ├── whatsapp/        # WhatsApp automation
│   │   │   ├── billing/         # AFIP invoicing
│   │   │   ├── video/           # LiveKit integration
│   │   │   ├── ai/              # Claude orchestration
│   │   │   └── patients/        # Patient management
│   │   ├── prisma/
│   │   │   └── schema.prisma    # Database schema
│   │   └── main.ts
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── routes/
│   │   ├── lib/
│   │   │   ├── components/
│   │   │   └── stores/
│   │   └── app.html
│   └── package.json
└── README.md
```

## Appointment Scheduling Module

### Creating Appointments

**Backend Service (NestJS):**

```typescript
// backend/src/modules/agenda/agenda.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AgendaService {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  async createAppointment(data: {
    practitionerId: string;
    patientId: string;
    startTime: Date;
    duration: number; // minutes
    type: 'presencial' | 'virtual' | 'evaluacion';
  }) {
    // Check for conflicts
    const conflicts = await this.prisma.appointment.findMany({
      where: {
        practitionerId: data.practitionerId,
        OR: [
          {
            startTime: {
              lte: data.startTime,
            },
            endTime: {
              gt: data.startTime,
            },
          },
        ],
      },
    });

    if (conflicts.length > 0) {
      throw new Error('Time slot conflict detected');
    }

    const endTime = new Date(
      data.startTime.getTime() + data.duration * 60000
    );

    const appointment = await this.prisma.appointment.create({
      data: {
        ...data,
        endTime,
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

  async getAvailableSlots(
    practitionerId: string,
    date: Date,
    duration: number = 45
  ) {
    const startOfDay = new Date(date.setHours(0, 0, 0, 0));
    const endOfDay = new Date(date.setHours(23, 59, 59, 999));

    const appointments = await this.prisma.appointment.findMany({
      where: {
        practitionerId,
        startTime: {
          gte: startOfDay,
          lte: endOfDay,
        },
      },
      orderBy: {
        startTime: 'asc',
      },
    });

    // Get practitioner working hours
    const practitioner = await this.prisma.practitioner.findUnique({
      where: { id: practitionerId },
      include: { workingHours: true },
    });

    const slots = this.calculateAvailableSlots(
      appointments,
      practitioner.workingHours,
      duration
    );

    return slots;
  }

  private calculateAvailableSlots(
    appointments: any[],
    workingHours: any,
    duration: number
  ) {
    // Slot calculation logic
    const slots = [];
    const dayOfWeek = new Date().getDay();
    const hours = workingHours.find(
      (h) => h.dayOfWeek === dayOfWeek
    );

    if (!hours) return [];

    let currentTime = new Date(hours.startTime);
    const endTime = new Date(hours.endTime);

    while (currentTime < endTime) {
      const slotEnd = new Date(
        currentTime.getTime() + duration * 60000
      );

      const hasConflict = appointments.some(
        (apt) =>
          (currentTime >= apt.startTime &&
            currentTime < apt.endTime) ||
          (slotEnd > apt.startTime && slotEnd <= apt.endTime)
      );

      if (!hasConflict) {
        slots.push({
          startTime: new Date(currentTime),
          endTime: new Date(slotEnd),
        });
      }

      currentTime = new Date(
        currentTime.getTime() + duration * 60000
      );
    }

    return slots;
  }
}
```

**Frontend Component (Svelte 5):**

```svelte
<!-- frontend/src/lib/components/AppointmentScheduler.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { api } from '$lib/api';
  
  let {
    practitionerId,
    patientId,
    onScheduled
  }: {
    practitionerId: string;
    patientId: string;
    onScheduled?: (appointment: any) => void;
  } = $props();

  let selectedDate = $state(new Date());
  let availableSlots = $state<any[]>([]);
  let selectedSlot = $state<any>(null);
  let appointmentType = $state<'presencial' | 'virtual' | 'evaluacion'>('presencial');
  let loading = $state(false);

  async function loadSlots() {
    loading = true;
    try {
      const response = await api.get('/agenda/available-slots', {
        params: {
          practitionerId,
          date: selectedDate.toISOString(),
          duration: 45
        }
      });
      availableSlots = response.data;
    } catch (error) {
      console.error('Failed to load slots:', error);
    } finally {
      loading = false;
    }
  }

  async function scheduleAppointment() {
    if (!selectedSlot) return;

    loading = true;
    try {
      const appointment = await api.post('/agenda/appointments', {
        practitionerId,
        patientId,
        startTime: selectedSlot.startTime,
        duration: 45,
        type: appointmentType
      });
      
      onScheduled?.(appointment.data);
    } catch (error) {
      console.error('Failed to schedule:', error);
      alert('Error al agendar la cita');
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    loadSlots();
  });

  $effect(() => {
    loadSlots();
  });
</script>

<div class="scheduler-container">
  <h2>Agendar Cita</h2>
  
  <div class="date-picker">
    <label>
      Fecha:
      <input
        type="date"
        bind:value={selectedDate}
        min={new Date().toISOString().split('T')[0]}
      />
    </label>
  </div>

  <div class="appointment-type">
    <label>
      <input type="radio" bind:group={appointmentType} value="presencial" />
      Presencial
    </label>
    <label>
      <input type="radio" bind:group={appointmentType} value="virtual" />
      Virtual
    </label>
    <label>
      <input type="radio" bind:group={appointmentType} value="evaluacion" />
      Evaluación
    </label>
  </div>

  <div class="slots-grid">
    {#if loading}
      <p>Cargando horarios disponibles...</p>
    {:else if availableSlots.length === 0}
      <p>No hay horarios disponibles para esta fecha</p>
    {:else}
      {#each availableSlots as slot}
        <button
          class="slot-button"
          class:selected={selectedSlot === slot}
          onclick={() => selectedSlot = slot}
        >
          {new Date(slot.startTime).toLocaleTimeString('es-AR', {
            hour: '2-digit',
            minute: '2-digit'
          })}
        </button>
      {/each}
    {/if}
  </div>

  <button
    class="schedule-button"
    disabled={!selectedSlot || loading}
    onclick={scheduleAppointment}
  >
    {loading ? 'Agendando...' : 'Confirmar Cita'}
  </button>
</div>

<style>
  .scheduler-container {
    padding: 2rem;
    max-width: 600px;
  }

  .slots-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 1rem;
    margin: 2rem 0;
  }

  .slot-button {
    padding: 1rem;
    border: 2px solid #e5e7eb;
    border-radius: 0.5rem;
    background: white;
    cursor: pointer;
    transition: all 0.2s;
  }

  .slot-button:hover {
    border-color: #3b82f6;
  }

  .slot-button.selected {
    background: #3b82f6;
    color: white;
    border-color: #3b82f6;
  }

  .schedule-button {
    width: 100%;
    padding: 1rem;
    background: #10b981;
    color: white;
    border: none;
    border-radius: 0.5rem;
    font-size: 1rem;
    cursor: pointer;
  }

  .schedule-button:disabled {
    background: #d1d5db;
    cursor: not-allowed;
  }
</style>
```

## WhatsApp Automation

### Setting Up Baileys Integration

**WhatsApp Service:**

```typescript
// backend/src/modules/whatsapp/whatsapp.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import makeWASocket, {
  DisconnectReason,
  useMultiFileAuthState,
  fetchLatestBaileysVersion,
} from '@whiskeysockets/baileys';
import { Boom } from '@hapi/boom';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class WhatsAppService implements OnModuleInit {
  private sock: any;

  constructor(private prisma: PrismaService) {}

  async onModuleInit() {
    await this.connectToWhatsApp();
  }

  async connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState(
      process.env.WHATSAPP_SESSION_PATH
    );

    const { version } = await fetchLatestBaileysVersion();

    this.sock = makeWASocket({
      version,
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

    this.sock.ev.on('messages.upsert', async ({ messages }) => {
      await this.handleIncomingMessage(messages[0]);
    });
  }

  async sendAppointmentConfirmation(appointment: any) {
    const message = this.formatAppointmentMessage(appointment);
    const phoneNumber = this.formatPhoneNumber(
      appointment.patient.phone
    );

    await this.sock.sendMessage(phoneNumber, {
      text: message,
    });

    // Log message
    await this.prisma.whatsAppMessage.create({
      data: {
        appointmentId: appointment.id,
        patientId: appointment.patientId,
        type: 'appointment_confirmation',
        content: message,
        status: 'sent',
      },
    });
  }

  async sendAppointmentReminder(appointmentId: string) {
    const appointment = await this.prisma.appointment.findUnique({
      where: { id: appointmentId },
      include: { patient: true, practitioner: true },
    });

    const hoursUntil = Math.floor(
      (appointment.startTime.getTime() - Date.now()) /
        (1000 * 60 * 60)
    );

    const message = `🔔 Recordatorio de Cita\n\n` +
      `Hola ${appointment.patient.firstName},\n\n` +
      `Te recordamos tu cita en ${hoursUntil} horas:\n` +
      `📅 ${appointment.startTime.toLocaleDateString('es-AR')}\n` +
      `🕐 ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n` +
      `👤 ${appointment.practitioner.fullName}\n` +
      `📍 ${appointment.type === 'virtual' ? 'Videollamada' : 'Presencial'}\n\n` +
      `Para confirmar, responde: SI\n` +
      `Para cancelar o reprogramar, responde: NO`;

    await this.sock.sendMessage(
      this.formatPhoneNumber(appointment.patient.phone),
      { text: message }
    );
  }

  async sendInvoice(invoiceId: string) {
    const invoice = await this.prisma.invoice.findUnique({
      where: { id: invoiceId },
      include: { patient: true },
    });

    const pdfPath = await this.generateInvoicePDF(invoice);

    await this.sock.sendMessage(
      this.formatPhoneNumber(invoice.patient.phone),
      {
        document: { url: pdfPath },
        mimetype: 'application/pdf',
        fileName: `Factura_${invoice.number}.pdf`,
        caption: `Factura ${invoice.number} - Total: $${invoice.amount}`,
      }
    );
  }

  private async handleIncomingMessage(message: any) {
    if (!message.key.fromMe && message.message) {
      const text = message.message.conversation?.toLowerCase() || '';
      const phoneNumber = message.key.remoteJid;

      // Check for crisis keywords
      const crisisKeywords = [
        'suicidio',
        'matarme',
        'crisis',
        'emergencia',
      ];
      const isCrisis = crisisKeywords.some((keyword) =>
        text.includes(keyword)
      );

      if (isCrisis) {
        await this.escalateCrisis(phoneNumber, text);
        return;
      }

      // Handle appointment confirmations
      if (text.includes('si') || text.includes('sí')) {
        await this.handleAppointmentConfirmation(phoneNumber);
      } else if (text.includes('no')) {
        await this.handleAppointmentCancellation(phoneNumber);
      }
    }
  }

  private formatPhoneNumber(phone: string): string {
    // Convert to WhatsApp format: 54 + area code + number
    return `54${phone.replace(/\D/g, '')}@s.whatsapp.net`;
  }

  private formatAppointmentMessage(appointment: any): string {
    return (
      `✅ Cita Confirmada\n\n` +
      `Hola ${appointment.patient.firstName},\n\n` +
      `Tu cita ha sido agendada:\n` +
      `📅 ${appointment.startTime.toLocaleDateString('es-AR')}\n` +
      `🕐 ${appointment.startTime.toLocaleTimeString('es-AR', { hour: '2-digit', minute: '2-digit' })}\n` +
      `👤 ${appointment.practitioner.fullName}\n` +
      `📍 ${appointment.type === 'virtual' ? 'Videollamada' : 'Presencial'}\n\n` +
      `Recibirás un recordatorio 24hs y 2hs antes de tu sesión.`
    );
  }

  private async escalateCrisis(
    phoneNumber: string,
    message: string
  ) {
    // Find patient and notify practitioner
    const phone = phoneNumber.replace('@s.whatsapp.net', '');
    const patient = await this.prisma.patient.findFirst({
      where: { phone: { contains: phone } },
      include: { practitioner: true },
    });

    if (patient) {
      // Send urgent notification to practitioner
      await this.sock.sendMessage(
        this.formatPhoneNumber(patient.practitioner.phone),
        {
          text:
            `🚨 ALERTA DE CRISIS\n\n` +
            `Paciente: ${patient.firstName} ${patient.lastName}\n` +
            `Mensaje: ${message}\n\n` +
            `Por favor, contacta inmediatamente.`,
        }
      );

      // Log crisis event
      await this.prisma.crisisEvent.create({
        data: {
          patientId: patient.id,
          practitionerId: patient.practitionerId,
          message,
          status: 'escalated',
        },
      });
    }
  }

  private async generateInvoicePDF(invoice: any): Promise<string> {
    // PDF generation logic (integrate with AFIP format)
    return `/invoices/${invoice.id}.pdf`;
  }
}
```

### Automated Reminder Scheduling

**Cron Job Configuration:**

```typescript
// backend/src/modules/whatsapp/reminder.cron.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from './whatsapp.service';

@Injectable()
export class ReminderCron {
  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async send24HourReminders() {
    const tomorrow = new Date();
    tomorrow.setHours(tomorrow.getHours() + 24);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: new Date(),
          lte: tomorrow,
        },
        status: 'scheduled',
        reminderSent24h: false,
      },
      include: { patient: true, practitioner: true },
    });

    for (const appointment of appointments) {
      await this.whatsapp.sendAppointmentReminder(appointment.id);
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent24h: true },
      });
    }
  }

  @Cron(CronExpression.EVERY_30_MINUTES)
  async send2HourReminders() {
    const twoHoursLater = new Date();
    twoHoursLater.setHours(twoHoursLater.getHours() + 2);

    const appointments = await this.prisma.appointment.findMany({
      where: {
        startTime: {
          gte: new Date(),
          lte: twoHoursLater,
        },
        status: 'scheduled',
        reminderSent2h: false,
      },
      include: { patient: true, practitioner: true },
    });

    for (const appointment of appointments) {
      await this.whatsapp.sendAppointmentReminder(appointment.id);
      await this.prisma.appointment.update({
        where: { id: appointment.id },
        data: { reminderSent2h: true },
      });
    }
  }
}
```

## AFIP Electronic Invoicing

### Invoice Generation Service

```typescript
// backend/src/modules/billing/afip.service.ts
import { Injectable } from '@nestjs/common';
import Afip from '@afipsdk/afip.js';
import { PrismaService } from '../prisma/prisma.service';
import { WhatsAppService } from '../whatsapp/whatsapp.service';

@Injectable()
export class AfipService {
  private afip: any;

  constructor(
    private prisma: PrismaService,
    private whatsapp: WhatsAppService,
  ) {
    this.afip = new Afip({
      CUIT: process.env.AFIP_CUIT,
      cert: process.env.AFIP_CERT_PATH,
      key: process.env.AFIP_KEY_PATH,
      production: process.env.AFIP_ENVIRONMENT === 'production',
    });
  }

  async generateInvoice(data: {
    patientId: string;
    appointmentId: string;
    amount: number;
    concept: string;
    type: 'A' | 'B' | 'C';
  }) {
    const patient = await this.prisma.patient.findUnique({
      where: { id: data.patientId },
    });

    const practitioner = await this.prisma.practitioner.findFirst();

    // Get next invoice number
    const lastInvoice = await this.prisma.invoice.findFirst({
      where: { practitionerId: practitioner.id },
      orderBy: { createdAt: 'desc' },
    });

    const nextNumber = lastInvoice
      ? lastInvoice.number + 1
      : 1;

    // Generate AFIP invoice
    const invoiceData = {
      CantReg: 1,
      PtoVta: 1,
      CbteTipo: this.getInvoiceTypeCode(data.type),
      Concepto: 2, // Servicios
      DocTipo: 80, // CUIT
      DocNro: patient.cuit || 0,
      CbteDesde: nextNumber,
      CbteHasta: nextNumber,
      CbteFch: this.formatDate(new Date()),
      ImpTotal: data.amount,
      ImpTotConc: 0,
      ImpNeto: data.amount,
      ImpOpEx: 0,
      ImpIVA: 0,
      ImpTrib: 0,
      FchServDesde: this.formatDate(new Date()),
      FchServHasta: this.formatDate(new Date()),
      FchVtoPago: this.formatDate(new Date()),
      MonId: 'PES',
      MonCotiz: 1,
    };

    const afipResponse = await this.afip.ElectronicBilling.createVoucher(
      invoiceData
    );

    // Save to database
    const invoice = await this.prisma.invoice.create({
      data: {
        number: nextNumber,
        type: data.type,
        patientId: data.patientId,
        practitionerId: practitioner.id,
        appointmentId: data.appointmentId,
        amount: data.amount,
        concept: data.concept,
        afipCae: afipResponse.CAE,
        afipCaeExpiration: new Date(afipResponse.CAEFchVto),
        status: 'approved',
      },
    });

    // Send invoice via WhatsApp
    await this.whatsapp.sendInvoice(invoice.id);

    return invoice;
  }

  private getInvoiceTypeCode(type: 'A' | 'B' | 'C'): number {
    const codes = { A: 1, B: 6, C: 11 };
    return codes[type];
  }

  private formatDate(date: Date): string {
    return date.toISOString().split('T')[0].replace(/-/g, '');
  }

  async getMonthlyReport(practitionerId: string, year: number, month: number) {
    const startDate = new Date(year, month - 1, 1);
    const endDate = new Date(year, month, 0, 23, 59, 59);

    const invoices = await this.prisma.invoice.findMany({
      where: {
        practitionerId,
        createdAt: {
          gte: startDate,
          lte: endDate,
        },
        status: 'approved',
      },
      include: {
        patient: true,
        appointment: true,
      },
    });

    const totalIncome = invoices.reduce(
      (sum, inv) => sum + inv.amount,
      0
    );

    // Calculate IIBB (Ingresos Brutos) by province
    const iibbByProvince = await this.calculateIIBB(invoices);

    return {
      period: `${year}-${month.toString().padStart(2, '0')}`,
      totalInvoices: invoices.length,
      totalIncome,
      iibbByProvince,
      invoices,
    };
  }

  private async calculateIIBB(invoices: any[]) {
    const provinces = {};

    for (const invoice of invoices) {
      const province = invoice.patient.province || 'CABA';
      if (!provinces[province]) {
        provinces[province] = { total: 0, rate: this.getIIBBRate(province) };
      }
      provinces[province].total += invoice.amount;
    }

    for (const province in provinces) {
      provinces[province].iibb =
        provinces[province].total * provinces[province].rate;
    }

    return provinces;
  }

  private getIIBBRate(province: string): number {
    const rates = {
      CABA: 0.035,
      'Buenos Aires': 0.04,
      Córdoba: 0.04,
      'Santa Fe': 0.045,
    };
    return rates[province] || 0.04;
  }
}
```

## Video Consultation with LiveKit

### Setting Up LiveKit Integration

```typescript
// backend/src/modules/video/video.service.ts
import { Injectable } from '@nestjs/common';
import { AccessToken } from 'livekit-server-sdk';
import { PrismaService } from '../prisma/prisma.service
