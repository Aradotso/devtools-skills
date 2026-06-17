---
name: ngx-ai-devtools-angular-llm-debugger
description: Floating DevTools panel for intercepting and debugging OpenAI, Anthropic, Gemini, Mistral, Groq, and Cohere API calls in Angular apps with cost tracking and streaming support
triggers:
  - add AI devtools to my Angular app
  - debug LLM calls in Angular
  - track OpenAI API costs in development
  - intercept Anthropic requests in my app
  - show me AI API tokens and costs
  - set up ngx-ai-devtools
  - monitor streaming AI responses
  - replay LLM calls in Angular
---

# ngx-ai-devtools Angular LLM Debugger

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Network-tab-style DevTools for debugging LLM API calls in Angular applications. Intercepts and displays prompts, responses, token counts, costs, and streaming data for OpenAI, Anthropic, Google Gemini, Mistral, Groq, and Cohere — all without leaving the browser.

## What It Does

`ngx-ai-devtools` automatically intercepts all LLM API calls made from your Angular app (via `fetch`, `HttpClient`, or official SDKs) and displays them in a floating DevTools panel. It:

- Auto-detects provider (OpenAI, Anthropic, Gemini, etc.) and parses request/response
- Calculates cost per call using embedded pricing tables
- Handles Server-Sent Events (SSE) streaming with delta accumulation
- Shows tool/function calls with arguments
- Provides one-click replay functionality
- Tracks time-to-first-token and running session totals
- Persists history across reloads (optional)
- Works with any SDK or raw `fetch` calls

## Installation

```bash
npm install ngx-ai-devtools
```

**Requirements:** Angular 18.1+ (uses signals, standalone components, `@let` syntax)

## Basic Setup

Add `provideAiDevtools()` to your application config:

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideAiDevtools } from 'ngx-ai-devtools';
import { environment } from './environments/environment';

export const appConfig: ApplicationConfig = {
  providers: [
    provideAiDevtools({
      enabled: !environment.production, // Disable in production
    }),
  ],
};
```

A floating launcher pill appears in the bottom-right corner. All LLM calls are now intercepted and displayed.

## Configuration Options

All options are optional. Pass them to `provideAiDevtools()`:

```typescript
provideAiDevtools({
  // When false, no patching or UI overhead
  enabled: !environment.production,
  
  // Maximum calls retained in memory (FIFO)
  maxCalls: 200,
  
  // Persist call history to localStorage across reloads
  persist: true,
  
  // Launcher position on screen
  position: 'bottom-left', // 'bottom-right' | 'top-right' | 'top-left'
  
  // Hide request/response bodies in UI (still recorded)
  redact: false,
  
  // Auto-inject UI into document.body
  autoMount: true,
  
  // Additional URL patterns to treat as LLM endpoints (for proxies/custom gateways)
  additionalEndpoints: [
    '/api/llm-proxy',
    '/v1/internal-gateway',
    'custom-llm.example.com'
  ],
});
```

## Usage with Different SDKs

### OpenAI SDK

```typescript
import OpenAI from 'openai';
import { Component } from '@angular/core';

@Component({
  selector: 'app-chat',
  template: `<button (click)="sendMessage()">Send</button>`,
})
export class ChatComponent {
  private openai = new OpenAI({
    apiKey: process.env['OPENAI_API_KEY'],
    dangerouslyAllowBrowser: true, // Only for dev/demo
  });

  async sendMessage() {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4o',
      messages: [
        { role: 'system', content: 'You are a helpful assistant.' },
        { role: 'user', content: 'Explain quantum computing in simple terms' }
      ],
      temperature: 0.7,
    });
    
    console.log(completion.choices[0].message.content);
    // Call automatically appears in ngx-ai-devtools panel
  }
}
```

### Streaming with OpenAI

```typescript
async streamResponse() {
  const stream = await this.openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [{ role: 'user', content: 'Count to 10' }],
    stream: true, // Enable streaming
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    console.log(content);
    // Deltas accumulate in real-time in the devtools panel
  }
}
```

### Anthropic SDK

```typescript
import Anthropic from '@anthropic-ai/sdk';

@Component({
  selector: 'app-claude',
  template: `<button (click)="askClaude()">Ask Claude</button>`,
})
export class ClaudeComponent {
  private anthropic = new Anthropic({
    apiKey: process.env['ANTHROPIC_API_KEY'],
  });

  async askClaude() {
    const message = await this.anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1024,
      messages: [
        { role: 'user', content: 'Write a haiku about TypeScript' }
      ],
    });
    
    console.log(message.content[0].text);
  }
}
```

### Angular HttpClient (Raw API Calls)

```typescript
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { inject, Component } from '@angular/core';

@Component({
  selector: 'app-gemini',
  template: `<button (click)="callGemini()">Call Gemini</button>`,
})
export class GeminiComponent {
  private http = inject(HttpClient);

  callGemini() {
    const headers = new HttpHeaders({
      'Content-Type': 'application/json',
    });

    const body = {
      contents: [{
        parts: [{ text: 'Explain the theory of relativity' }]
      }]
    };

    this.http.post(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=${process.env['GEMINI_API_KEY']}`,
      body,
      { headers }
    ).subscribe(response => {
      console.log(response);
      // Appears in devtools automatically
    });
  }
}
```

### Raw Fetch (Works Anywhere)

```typescript
async callMistral() {
  const response = await fetch('https://api.mistral.ai/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env['MISTRAL_API_KEY']}`
    },
    body: JSON.stringify({
      model: 'mistral-large-latest',
      messages: [{ role: 'user', content: 'Hello!' }]
    })
  });
  
  const data = await response.json();
  console.log(data);
  // Intercepted and displayed in devtools
}
```

## Programmatic API

Access the devtools service directly in your components:

```typescript
import { Component, computed, inject } from '@angular/core';
import { AiDevtoolsService } from 'ngx-ai-devtools';

@Component({
  selector: 'app-cost-dashboard',
  template: `
    <div class="stats">
      <div>Total Calls: {{ stats().count }}</div>
      <div>Total Cost: ${{ stats().totalCost | number:'1.4-4' }}</div>
      <div>Total Tokens: {{ stats().totalTokens | number }}</div>
      <div>Avg Latency: {{ stats().avgLatency }}ms</div>
    </div>
    
    <button (click)="clearHistory()">Clear History</button>
    <button (click)="togglePanel()">Toggle Panel</button>
    
    @if (selectedCall()) {
      <div>Selected: {{ selectedCall()?.model }}</div>
    }
  `,
})
export class CostDashboard {
  private devtools = inject(AiDevtoolsService);
  
  // Signal-based reactive state
  stats = computed(() => this.devtools.stats());
  calls = this.devtools.calls;
  selectedCall = this.devtools.selected;
  
  clearHistory() {
    this.devtools.clear();
  }
  
  togglePanel() {
    const currentState = this.devtools.ui().open;
    this.devtools.setOpen(!currentState);
  }
}
```

### Service API Reference

| Member | Type | Description |
|--------|------|-------------|
| `calls` | `Signal<LlmCall[]>` | All recorded calls, newest first |
| `filtered` | `Signal<LlmCall[]>` | Filtered view matching search |
| `selected` | `Signal<LlmCall \| null>` | Currently selected call |
| `stats` | `Signal<Stats>` | `{ count, totalCost, totalTokens, avgLatency }` |
| `ui` | `Signal<UiState>` | `{ open, selectedId, filter }` |
| `clear()` | `() => void` | Drop all recorded calls |
| `setOpen(boolean)` | `(boolean) => void` | Open/close panel |
| `select(id)` | `(string \| null) => void` | Select a call by ID |
| `setFilter(string)` | `(string) => void` | Set search filter |
| `replay(id)` | `(string) => Promise<string \| null>` | Re-issue a call |

## Common Patterns

### Budget Alerts

```typescript
import { Component, effect, inject } from '@angular/core';
import { AiDevtoolsService } from 'ngx-ai-devtools';

@Component({
  selector: 'app-budget-monitor',
  template: `<div>Session cost: ${{ currentCost() }}</div>`,
})
export class BudgetMonitor {
  private devtools = inject(AiDevtoolsService);
  private readonly BUDGET_LIMIT = 5.00; // $5 limit
  
  currentCost = this.devtools.stats().totalCost;
  
  constructor() {
    effect(() => {
      const cost = this.devtools.stats().totalCost;
      if (cost > this.BUDGET_LIMIT) {
        console.warn(`⚠️ Budget exceeded: $${cost.toFixed(4)}`);
        // Disable AI features, show warning, etc.
      }
    });
  }
}
```

### Call Replay for Testing

```typescript
async testPromptVariation(callId: string) {
  // Replay exact same request
  const newCallId = await this.devtools.replay(callId);
  
  if (newCallId) {
    console.log('Replayed call ID:', newCallId);
    // Compare results in the UI
  }
}
```

### Filter Calls Programmatically

```typescript
searchForExpensiveCalls() {
  this.devtools.setFilter('gpt-4'); // Show only GPT-4 calls
}

showOnlyErrors() {
  const errorCalls = this.devtools.calls().filter(call => call.error);
  console.log('Failed calls:', errorCalls);
}
```

### Custom Component Integration

If `autoMount: false`, manually place the component:

```typescript
// app.config.ts
provideAiDevtools({
  enabled: true,
  autoMount: false, // Manual placement
});

// app.component.ts
import { Component } from '@angular/core';
import { NgxAiDevtoolsComponent } from 'ngx-ai-devtools';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [NgxAiDevtoolsComponent],
  template: `
    <main>
      <!-- Your app content -->
    </main>
    
    <!-- Devtools in custom position -->
    <ngx-ai-devtools />
  `,
})
export class AppComponent {}
```

## Tool/Function Calling

The devtools automatically parse and display function calls:

```typescript
const completion = await this.openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'What is the weather in Paris?' }],
  tools: [
    {
      type: 'function',
      function: {
        name: 'get_weather',
        description: 'Get current weather for a location',
        parameters: {
          type: 'object',
          properties: {
            location: { type: 'string', description: 'City name' },
            unit: { type: 'string', enum: ['celsius', 'fahrenheit'] }
          },
          required: ['location']
        }
      }
    }
  ],
});

// Tool calls appear in dedicated "Tool Use" tab in the panel
```

## Provider Support

| Provider | Request | Response | Streaming | Cost Calc |
|----------|---------|----------|-----------|-----------|
| OpenAI | ✅ | ✅ | ✅ SSE deltas | ✅ |
| Anthropic | ✅ | ✅ | ✅ Named events | ✅ |
| Google Gemini | ✅ | ✅ | Partial | ✅ |
| Mistral | ✅ | ✅ | ✅ | ✅ |
| Groq | ✅ | ✅ | ✅ | ✅ |
| Cohere | Detected | Partial | ❌ | ❌ |

## Troubleshooting

### Panel Not Appearing

**Issue:** Launcher pill doesn't show up

**Solutions:**
```typescript
// 1. Verify enabled is true
provideAiDevtools({ enabled: true });

// 2. Check console for errors
// Open browser console and look for ngx-ai-devtools initialization logs

// 3. Ensure Angular 18.1+
// Check package.json: "@angular/core": "^18.1.0"

// 4. Try manual mounting
provideAiDevtools({ autoMount: false });
// Then add <ngx-ai-devtools /> to a component
```

### Calls Not Being Intercepted

**Issue:** Making LLM calls but they don't appear in panel

**Solutions:**
```typescript
// 1. Add custom endpoints if using a proxy
provideAiDevtools({
  additionalEndpoints: [
    'your-proxy.com',
    '/api/openai-proxy'
  ]
});

// 2. Verify the request is actually going through
// Check browser Network tab to confirm request is being made

// 3. For HttpClient, ensure it's configured
import { provideHttpClient } from '@angular/common/http';
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(), // Required for HttpClient interception
    provideAiDevtools({ enabled: true }),
  ],
};
```

### Streaming Not Working

**Issue:** Streaming responses don't accumulate in panel

```typescript
// Ensure you're consuming the stream properly
const stream = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [...],
  stream: true,
});

// Must iterate to see deltas
for await (const chunk of stream) {
  // Process chunks
}

// Or with Anthropic SDK
const stream = await anthropic.messages.stream({
  model: 'claude-sonnet-4',
  messages: [...],
});

stream.on('text', (text) => console.log(text));
```

### Cost Not Calculating

**Issue:** Token counts show but no cost displayed

**Cause:** Model not in pricing table

**Solution:** Check if model is supported or add custom pricing:
```typescript
// Models are priced in src/lib/pricing.ts
// If using a new model, costs may not display
// The call still records, just without cost data

// For custom models, submit a PR or track costs externally:
effect(() => {
  const calls = this.devtools.calls();
  const customModelCalls = calls.filter(c => c.model === 'custom-model');
  const customCost = customModelCalls.reduce((sum, call) => {
    const cost = (call.tokens?.total || 0) * 0.00001; // Your rate
    return sum + cost;
  }, 0);
  console.log('Custom model cost:', customCost);
});
```

### SSR/Hydration Errors

**Issue:** Errors during server-side rendering

```typescript
// The library is SSR-safe, but ensure it's only enabled client-side
import { isPlatformBrowser } from '@angular/common';
import { PLATFORM_ID, inject } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: 'AI_DEVTOOLS_CONFIG',
      useFactory: () => {
        const platformId = inject(PLATFORM_ID);
        return provideAiDevtools({
          enabled: isPlatformBrowser(platformId) && !environment.production,
        });
      }
    }
  ],
};
```

### localStorage Quota Exceeded

**Issue:** Persist mode fills localStorage

```typescript
// Reduce maxCalls or disable persistence
provideAiDevtools({
  persist: false, // Disable persistence
  maxCalls: 50,   // Reduce memory footprint
});

// Or manually clear when needed
this.devtools.clear();
```

## Advanced: Testing and CI

Export call data for test assertions:

```typescript
import { TestBed } from '@angular/core/testing';
import { AiDevtoolsService } from 'ngx-ai-devtools';

describe('AI Feature Tests', () => {
  it('should not exceed cost budget', async () => {
    const devtools = TestBed.inject(AiDevtoolsService);
    
    // Run test scenario
    await runAiFeature();
    
    // Assert on costs
    const stats = devtools.stats();
    expect(stats.totalCost).toBeLessThan(0.10);
  });
  
  it('should use correct model', async () => {
    const devtools = TestBed.inject(AiDevtoolsService);
    
    await callAI();
    
    const lastCall = devtools.calls()[0];
    expect(lastCall.model).toBe('gpt-4o-mini');
  });
});
```

## Integration with Vercel AI SDK

Works transparently with Vercel AI SDK:

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

@Component({
  selector: 'app-vercel-ai',
  template: `<button (click)="generate()">Generate</button>`,
})
export class VercelAiComponent {
  async generate() {
    const { text } = await generateText({
      model: openai('gpt-4o'),
      prompt: 'Write a story about a robot',
    });
    
    console.log(text);
    // Call appears in ngx-ai-devtools automatically
  }
}
```

## Environment-Specific Configuration

```typescript
// environment.development.ts
export const environment = {
  production: false,
  aiDevtools: {
    enabled: true,
    persist: true,
    maxCalls: 200,
  }
};

// environment.production.ts
export const environment = {
  production: true,
  aiDevtools: {
    enabled: false, // Critical: disable in prod
  }
};

// app.config.ts
import { environment } from './environments/environment';

export const appConfig: ApplicationConfig = {
  providers: [
    provideAiDevtools(environment.aiDevtools),
  ],
};
```

## Summary

`ngx-ai-devtools` requires one provider call to set up, then automatically intercepts and displays all LLM API traffic in a floating panel. Use it during development to debug prompts, track costs, inspect streaming behavior, and iterate on AI features without external tools. Disable it in production with `enabled: false`.
