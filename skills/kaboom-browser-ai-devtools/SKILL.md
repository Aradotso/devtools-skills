---
name: kaboom-browser-ai-devtools
description: Browser debugging and inspection toolkit for AI coding assistants via MCP protocol
triggers:
  - "debug browser console errors with Kaboom"
  - "capture network failures using Kaboom MCP"
  - "inspect DOM elements with Kaboom devtools"
  - "record user interactions for browser testing"
  - "generate Playwright tests from browser sessions"
  - "audit accessibility with Kaboom WCAG checks"
  - "monitor Web Vitals and performance metrics"
  - "automate browser actions via Kaboom MCP"
---

# Kaboom Browser AI DevTools MCP

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Kaboom is a browser debugging, inspection, and verification toolkit that streams console logs, network failures, exceptions, recordings, and browser evidence into MCP-compatible coding assistants. It provides real-time browser debugging without requiring `--remote-debugging-port`, using a standard browser extension instead.

## Key Features

- **Console monitoring** — Stream all `console.log()`, `.warn()`, `.error()` with full arguments
- **Network debugging** — Capture failed requests (4xx/5xx) with full request/response bodies
- **Exception tracking** — Uncaught errors with complete stack traces
- **WebSocket monitoring** — Connection events and message payloads
- **User action recording** — Click, type, navigate, scroll with smart selectors
- **Web Vitals tracking** — LCP, CLS, INP, FCP with regression detection
- **DOM inspection** — Query page elements via CSS selectors
- **Accessibility audits** — WCAG compliance checks with SARIF export
- **Security audits** — Credentials, PII, headers, cookies analysis
- **Browser automation** — Click, type, select, upload, navigate programmatically
- **Test generation** — Generate Playwright tests from recorded sessions
- **Visual annotations** — Drawing mode for user feedback with style extraction

## Installation

### Quick Install (Recommended)

**macOS / Linux:**
```bash
curl -sSL https://raw.githubusercontent.com/brennhill/Kaboom-Browser-AI-Devtools-MCP/STABLE/scripts/install.sh | bash
```

**Windows (PowerShell):**
```powershell
irm https://raw.githubusercontent.com/brennhill/Kaboom-Browser-AI-Devtools-MCP/STABLE/scripts/install.ps1 | iex
```

This automatically:
1. Downloads the binary for your platform
2. Installs browser extension to `~/.kaboom/extension`
3. Auto-configures detected MCP clients (Claude Code, Cursor, Windsurf, Zed)

### Manual Installation

1. **Download binary** from [releases](https://github.com/brennhill/Kaboom-Browser-AI-Devtools-MCP/releases)
2. **Extract and make executable:**
   ```bash
   tar -xzf kaboom-*.tar.gz
   chmod +x kaboom
   mv kaboom /usr/local/bin/
   ```
3. **Install browser extension:**
   - Open `chrome://extensions`
   - Enable Developer mode
   - Click "Load unpacked"
   - Select `~/.kaboom/extension` directory

### MCP Configuration

Add to your MCP client config:

**Claude Desktop / Claude Code (`claude_desktop_config.json`):**
```json
{
  "mcpServers": {
    "kaboom": {
      "command": "/usr/local/bin/kaboom",
      "args": ["--mcp"]
    }
  }
}
```

**Cursor (`config.json` in `.cursor/` directory):**
```json
{
  "mcpServers": {
    "kaboom": {
      "command": "/usr/local/bin/kaboom",
      "args": ["--mcp"]
    }
  }
}
```

**Zed (`settings.json`):**
```json
{
  "context_servers": {
    "kaboom": {
      "command": {
        "path": "/usr/local/bin/kaboom",
        "args": ["--mcp"]
      }
    }
  }
}
```

## MCP Tools Reference

### Console & Error Monitoring

**`kaboom_console_clear`** — Clear all captured console logs
```typescript
// No arguments required
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_console_clear"
});
```

**`kaboom_console_get`** — Get all captured console logs
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_console_get",
  arguments: {
    level: "error" // Optional: "log", "warn", "error", "info"
  }
});
```

### Network Monitoring

**`kaboom_network_get`** — Get captured network requests
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_network_get",
  arguments: {
    status: "failed", // "all", "failed", "success"
    includeBody: true
  }
});
```

**`kaboom_network_clear`** — Clear network capture buffer
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_network_clear"
});
```

### DOM Inspection

**`kaboom_dom_query`** — Query DOM elements with CSS selectors
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_dom_query",
  arguments: {
    selector: ".error-message",
    includeStyles: true,
    includeAttributes: true
  }
});
```

**`kaboom_dom_snapshot`** — Capture full DOM snapshot
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_dom_snapshot",
  arguments: {
    includeInlineStyles: true
  }
});
```

### User Action Recording

**`kaboom_recording_start`** — Start recording user actions
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_start",
  arguments: {
    captureClicks: true,
    captureInputs: true,
    captureNavigation: true,
    captureScrolls: true
  }
});
```

**`kaboom_recording_stop`** — Stop and retrieve recording
```typescript
const recording = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_stop"
});
// Returns array of recorded actions with selectors
```

**`kaboom_recording_generate_test`** — Generate Playwright test from recording
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_generate_test",
  arguments: {
    format: "playwright", // "playwright" or "puppeteer"
    includeAssertions: true
  }
});
```

### Browser Automation

**`kaboom_browser_click`** — Click element
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_click",
  arguments: {
    selector: "button[type='submit']"
  }
});
```

**`kaboom_browser_type`** — Type into input field
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_type",
  arguments: {
    selector: "input[name='email']",
    text: "user@example.com",
    clear: true
  }
});
```

**`kaboom_browser_navigate`** — Navigate to URL
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_navigate",
  arguments: {
    url: "https://example.com/dashboard",
    waitForLoad: true
  }
});
```

**`kaboom_browser_select`** — Select dropdown option
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_select",
  arguments: {
    selector: "select[name='country']",
    value: "US"
  }
});
```

**`kaboom_browser_upload`** — Upload file
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_upload",
  arguments: {
    selector: "input[type='file']",
    filePath: "/path/to/file.pdf"
  }
});
```

### Accessibility Auditing

**`kaboom_a11y_audit`** — Run WCAG accessibility audit
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_a11y_audit",
  arguments: {
    standard: "WCAG2AA", // "WCAG2A", "WCAG2AA", "WCAG2AAA"
    includeWarnings: true
  }
});
```

**`kaboom_a11y_export`** — Export audit results
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_a11y_export",
  arguments: {
    format: "sarif" // "sarif", "json", "html"
  }
});
```

### Security Auditing

**`kaboom_security_audit`** — Run security audit
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_security_audit",
  arguments: {
    checkCredentials: true,
    checkPII: true,
    checkHeaders: true,
    checkCookies: true,
    checkThirdParty: true
  }
});
```

### Web Vitals Monitoring

**`kaboom_vitals_get`** — Get Web Vitals metrics
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_vitals_get",
  arguments: {
    metrics: ["LCP", "CLS", "INP", "FCP"] // Optional, defaults to all
  }
});
```

**`kaboom_vitals_baseline`** — Set performance baseline
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_vitals_baseline",
  arguments: {
    metric: "LCP",
    threshold: 2500 // milliseconds
  }
});
```

### Visual Annotations

**`kaboom_annotation_enable`** — Enable drawing mode
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_annotation_enable"
});
```

**`kaboom_annotation_get`** — Get annotations with computed styles
```typescript
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_annotation_get",
  arguments: {
    includeStyles: true
  }
});
```

## Common Workflows

### Debug Console Errors

```typescript
// 1. Get all error logs
const errors = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_console_get",
  arguments: { level: "error" }
});

// 2. Inspect DOM at error location
const elements = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_dom_query",
  arguments: {
    selector: ".error-component",
    includeStyles: true
  }
});

// 3. Clear logs after fix
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_console_clear"
});
```

### Debug Failed API Calls

```typescript
// 1. Get failed network requests
const failed = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_network_get",
  arguments: {
    status: "failed",
    includeBody: true
  }
});

// 2. Check for auth issues in request headers
// Analyze the response from failed requests

// 3. Clear network buffer
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_network_clear"
});
```

### Record User Flow and Generate Test

```typescript
// 1. Start recording
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_start",
  arguments: {
    captureClicks: true,
    captureInputs: true,
    captureNavigation: true
  }
});

// 2. User performs actions in browser...

// 3. Stop recording and get actions
const actions = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_stop"
});

// 4. Generate Playwright test
const test = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_generate_test",
  arguments: {
    format: "playwright",
    includeAssertions: true
  }
});
```

### Automate Browser Testing

```typescript
// Navigate to login page
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_navigate",
  arguments: {
    url: "https://app.example.com/login",
    waitForLoad: true
  }
});

// Fill in credentials
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_type",
  arguments: {
    selector: "input[name='email']",
    text: "test@example.com"
  }
});

await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_type",
  arguments: {
    selector: "input[name='password']",
    text: "testpass123"
  }
});

// Submit form
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_click",
  arguments: {
    selector: "button[type='submit']"
  }
});

// Check for errors
const errors = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_console_get",
  arguments: { level: "error" }
});
```

### Accessibility Audit

```typescript
// Run WCAG 2.1 AA audit
const audit = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_a11y_audit",
  arguments: {
    standard: "WCAG2AA",
    includeWarnings: true
  }
});

// Export results as SARIF
const report = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_a11y_export",
  arguments: {
    format: "sarif"
  }
});
```

### Performance Regression Detection

```typescript
// Set baseline for LCP
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_vitals_baseline",
  arguments: {
    metric: "LCP",
    threshold: 2500
  }
});

// Get current metrics
const vitals = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_vitals_get"
});

// Check if LCP regressed
if (vitals.LCP > 2500) {
  console.log("LCP regression detected!");
}
```

## Browser Extension Developer API

Use `window.__kaboom` in your application code for custom context:

```javascript
// Add custom annotation to help AI understand context
window.__kaboom.annotate({
  type: "error-boundary",
  component: "CheckoutForm",
  message: "Payment validation failed",
  metadata: {
    step: 3,
    validationErrors: ["invalid_card", "expired"]
  }
});

// Check if Kaboom is available
if (window.__kaboom) {
  // Kaboom is active
}
```

## Configuration

### Environment Variables

```bash
# Disable telemetry
export KABOOM_TELEMETRY=off

# Custom port (default: 3333)
export KABOOM_PORT=8080

# Log level
export KABOOM_LOG_LEVEL=debug # debug, info, warn, error

# Custom extension path
export KABOOM_EXTENSION_PATH=/custom/path/to/extension
```

### Server CLI Flags

```bash
# Start MCP server
kaboom --mcp

# Custom port
kaboom --mcp --port 8080

# Enable debug logging
kaboom --mcp --debug

# Print version
kaboom --version
```

## Troubleshooting

### Extension Not Connecting

1. **Check extension is loaded:**
   - Open `chrome://extensions`
   - Verify Kaboom is enabled
   - Check for errors in extension details

2. **Verify server is running:**
   ```bash
   ps aux | grep kaboom
   ```

3. **Check connection:**
   - Open browser DevTools
   - Look for "Kaboom connected" in console
   - If not connected, refresh the page

### No Console Logs Captured

1. **Clear cache and refresh:**
   ```typescript
   await use_mcp_tool({
     server_name: "kaboom",
     tool_name: "kaboom_console_clear"
   });
   ```

2. **Check log level filter** — Make sure you're not filtering out the logs you need

3. **Verify page is active** — Kaboom only captures from the active tab

### Network Requests Not Appearing

1. **Check status filter:**
   ```typescript
   // Get ALL requests, not just failed
   await use_mcp_tool({
     server_name: "kaboom",
     tool_name: "kaboom_network_get",
     arguments: { status: "all" }
   });
   ```

2. **Verify XHR/Fetch interception** — Some requests may be blocked by CSP

3. **Check response body size** — Very large bodies may be truncated

### Recording Generates Weak Selectors

1. **Add explicit IDs/data attributes** to critical elements:
   ```html
   <button data-testid="submit-btn">Submit</button>
   ```

2. **Use semantic HTML** — `<button>` instead of `<div onclick>`

3. **Manually improve selectors** in generated tests after recording

### MCP Client Not Finding Kaboom

1. **Check binary path:**
   ```bash
   which kaboom
   # Should output: /usr/local/bin/kaboom
   ```

2. **Update config with full path:**
   ```json
   {
     "mcpServers": {
       "kaboom": {
         "command": "/usr/local/bin/kaboom",
         "args": ["--mcp"]
       }
     }
   }
   ```

3. **Restart MCP client completely** (not just reload window)

### Performance Impact

Kaboom is designed for minimal overhead, but:

- **Disable when not debugging** — Unload extension if not needed
- **Clear buffers regularly** — Large console/network buffers can grow
- **Use selective recording** — Only record what you need

## Privacy & Security

- **All data stays local** — Nothing leaves your machine
- **No cloud, no accounts** — Zero external dependencies
- **Auth headers stripped** — Sensitive headers automatically removed
- **Localhost only** — Server binds to `127.0.0.1` only
- **Anonymous telemetry** — Random install ID, no personal data (disable with `KABOOM_TELEMETRY=off`)

## Advanced Usage

### Custom Test Generation Template

Generate tests with custom assertions:

```typescript
const test = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_generate_test",
  arguments: {
    format: "playwright",
    includeAssertions: true
  }
});

// Add custom assertions to generated test
const customTest = test.replace(
  "// end of test",
  `
  // Custom assertions
  await expect(page.locator('.success-message')).toBeVisible();
  await expect(page).toHaveURL(/.*dashboard/);
`
);
```

### Continuous Monitoring Pattern

```typescript
// Set up baseline thresholds
const thresholds = {
  LCP: 2500,
  CLS: 0.1,
  INP: 200,
  FCP: 1800
};

for (const [metric, threshold] of Object.entries(thresholds)) {
  await use_mcp_tool({
    server_name: "kaboom",
    tool_name: "kaboom_vitals_baseline",
    arguments: { metric, threshold }
  });
}

// Periodically check for regressions
const vitals = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_vitals_get"
});

// Flag regressions
const regressions = Object.entries(vitals)
  .filter(([metric, value]) => value > thresholds[metric])
  .map(([metric, value]) => `${metric}: ${value} > ${thresholds[metric]}`);

if (regressions.length > 0) {
  console.log("Performance regressions detected:", regressions);
}
```

### Multi-Page Recording

```typescript
// Start recording
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_start",
  arguments: { captureNavigation: true }
});

// Navigate through multiple pages
await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_navigate",
  arguments: { url: "https://example.com/page1" }
});

// Perform actions...

await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_browser_navigate",
  arguments: { url: "https://example.com/page2" }
});

// More actions...

// Stop and generate comprehensive test
const recording = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_stop"
});

const test = await use_mcp_tool({
  server_name: "kaboom",
  tool_name: "kaboom_recording_generate_test",
  arguments: {
    format: "playwright",
    includeAssertions: true
  }
});
```

## Resources

- **Documentation:** https://gokaboom.dev
- **GitHub:** https://github.com/brennhill/Kaboom-Browser-AI-Devtools-MCP
- **Issues:** https://github.com/brennhill/Kaboom-Browser-AI-Devtools-MCP/issues
- **Changelog:** https://github.com/brennhill/Kaboom-Browser-AI-Devtools-MCP/blob/STABLE/CHANGELOG.md
