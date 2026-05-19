---
name: browser-devtools-mcp-vscode
description: VSCode extension for Browser DevTools MCP Server enabling AI-driven browser automation, debugging, and testing via Playwright and Model Context Protocol
triggers:
  - automate browser testing with AI
  - debug web pages using MCP
  - capture screenshots with playwright
  - measure web vitals in vscode
  - inspect network requests with AI
  - run accessibility audits automatically
  - mock API responses for testing
  - integrate browser automation in cursor
---

# Browser DevTools MCP VSCode Extension

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Browser DevTools MCP for VS Code & Cursor is a Playwright-powered browser automation extension that integrates the `browser-devtools-mcp` server into your IDE via the Model Context Protocol (MCP). It enables AI assistants like GitHub Copilot and Cursor AI to interact with real web browsers for testing, debugging, and automation tasks.

**Key capabilities:**
- Browser automation (navigation, clicks, form filling)
- Screenshots and visual testing
- Accessibility audits and ARIA tree inspection
- Core Web Vitals measurement (LCP, INP, CLS, TTFB, FCP)
- Network inspection and request mocking
- React DevTools integration
- OpenTelemetry distributed tracing
- Figma design comparison
- Non-blocking debugging (tracepoints, logpoints, watch expressions)
- Batch execution via JavaScript

## Installation

### From Open VSX Registry

```bash
# VS Code
code --install-extension serkan-ozal.browser-devtools-mcp-vscode

# Cursor
cursor --install-extension serkan-ozal.browser-devtools-mcp-vscode
```

### From VSIX File

```bash
# VS Code
code --install-extension browser-devtools-mcp-vscode-x.x.x.vsix

# Cursor
cursor --install-extension browser-devtools-mcp-vscode-x.x.x.vsix
```

### First-Time Setup

On first activation, the extension downloads Playwright browsers (default: Chromium). To customize:

1. Open Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
2. Run: **Browser DevTools MCP: Install Playwright Browsers...**
3. Select browsers: Chromium (default), Firefox, and/or WebKit

Alternatively, configure in Settings:
- `browserDevtoolsMcp.install.chromium` (default: `true`)
- `browserDevtoolsMcp.install.firefox` (default: `false`)
- `browserDevtoolsMcp.install.webkit` (default: `false`)

## Configuration

### Quick Settings Panel

Open the **Browser DevTools MCP** panel in the Explorer sidebar for common settings.

### Full Settings

Open VS Code Settings (`Ctrl+,` / `Cmd+,`) and search for "Browser DevTools MCP" or run:

```
Browser DevTools MCP: Open Settings
```

### Key Settings

#### General

```json
{
  "browserDevtoolsMcp.enable": true,
  "browserDevtoolsMcp.telemetry.enable": true,
  "browserDevtoolsMcp.platform": "browser"  // "browser" or "node"
}
```

#### Browser Configuration

```json
{
  "browserDevtoolsMcp.browser.headless": true,
  "browserDevtoolsMcp.browser.persistent": false,
  "browserDevtoolsMcp.browser.userDataDir": "",
  "browserDevtoolsMcp.browser.useSystemBrowser": false,
  "browserDevtoolsMcp.browser.executablePath": "",
  "browserDevtoolsMcp.browser.locale": "en-US"
}
```

#### CDP (Chrome DevTools Protocol) Mode

```json
{
  "browserDevtoolsMcp.browser.cdp.enable": false,
  "browserDevtoolsMcp.browser.cdp.endpointUrl": ""  // e.g., "http://localhost:9222"
}
```

#### Network & Security

```json
{
  "browserDevtoolsMcp.network.proxy": "",  // e.g., "http://proxy.example.com:8080"
  "browserDevtoolsMcp.network.bypassProxy": "",
  "browserDevtoolsMcp.network.ignoreHTTPSErrors": false
}
```

#### Debugging

```json
{
  "browserDevtoolsMcp.debug.enable": false,
  "browserDevtoolsMcp.debug.screenshots": false,
  "browserDevtoolsMcp.debug.traces": false,
  "browserDevtoolsMcp.debug.videos": false,
  "browserDevtoolsMcp.debug.outputDir": ""
}
```

#### OpenTelemetry

```json
{
  "browserDevtoolsMcp.otel.enable": false,
  "browserDevtoolsMcp.otel.exporterUrl": "",  // e.g., "http://localhost:4318/v1/traces"
  "browserDevtoolsMcp.otel.serviceName": "browser-devtools-mcp",
  "browserDevtoolsMcp.otel.propagationHeaderName": "traceparent"
}
```

## Using with AI Assistants

### In Cursor

The extension automatically registers the MCP server via Cursor's native MCP API. AI assistants can directly invoke browser automation tools.

**Example prompts:**

```
"Navigate to example.com and take a screenshot"
"Run accessibility audit on the current page"
"Measure Core Web Vitals for https://example.com"
"Click the submit button and capture network requests"
"Mock the /api/users endpoint to return test data"
```

### In VS Code (1.96+)

The extension registers via `vscode.lm.registerMcpServerDefinitionProvider`. GitHub Copilot can use the MCP tools.

## MCP Tools Reference

### Navigation & Interaction

#### navigate
```typescript
// Navigate to URL
{
  "url": "https://example.com",
  "waitUntil": "networkidle"  // "load" | "domcontentloaded" | "networkidle"
}
```

#### click
```typescript
// Click element by selector
{
  "selector": "button.submit",
  "timeout": 30000
}
```

#### fill
```typescript
// Fill form input
{
  "selector": "input[name='email']",
  "value": "test@example.com"
}
```

#### type
```typescript
// Type text with keyboard simulation
{
  "selector": "textarea",
  "text": "Hello world",
  "delay": 100  // ms between keystrokes
}
```

#### select
```typescript
// Select dropdown option
{
  "selector": "select[name='country']",
  "values": ["US"]
}
```

### Inspection & Testing

#### screenshot
```typescript
// Capture screenshot
{
  "fullPage": true,
  "selector": null,  // or specific element selector
  "path": "/tmp/screenshot.png"  // optional
}
```

#### accessibility_snapshot
```typescript
// Get accessibility tree
{
  "selector": "main"  // optional, default entire page
}
```

#### accessibility_audit
```typescript
// Run accessibility audit
{
  "includeWarnings": true,
  "selector": null
}
```

#### web_vitals
```typescript
// Measure Core Web Vitals
{
  "url": "https://example.com"
}
```

### Network

#### network_requests
```typescript
// Get network request log
{
  "filter": {
    "url": "/api/*",
    "method": "GET",
    "status": 200
  }
}
```

#### mock_route
```typescript
// Mock API endpoint
{
  "pattern": "**/api/users",
  "response": {
    "status": 200,
    "body": { "users": [{"id": 1, "name": "Test User"}] },
    "headers": { "Content-Type": "application/json" }
  }
}
```

#### unmock_route
```typescript
// Remove mock
{
  "pattern": "**/api/users"
}
```

### React DevTools

#### react_inspect_element
```typescript
// Inspect React component
{
  "selector": "div.App"
}
```

#### react_get_component_tree
```typescript
// Get full component tree
{}
```

### Debugging

#### set_tracepoint
```typescript
// Non-blocking breakpoint with logging
{
  "source": "app.js",
  "line": 42,
  "condition": "user.id === 123",
  "logMessage": "User: {user.name}"
}
```

#### set_logpoint
```typescript
// Inject console.log
{
  "source": "utils.js",
  "line": 15,
  "message": "Value: {myVariable}"
}
```

#### watch_expression
```typescript
// Watch expression value
{
  "expression": "this.state.count",
  "source": "Counter.jsx"
}
```

### Batch Execution

#### execute
```typescript
// Execute multiple operations
{
  "code": `
    // Navigate
    await callTool('navigate', { url: 'https://example.com' });
    
    // Fill form
    await page.fill('input[name="search"]', 'test query');
    
    // Click and wait
    await page.click('button[type="submit"]');
    await page.waitForLoadState('networkidle');
    
    // Capture screenshot
    const screenshot = await callTool('screenshot', { fullPage: true });
    
    return { screenshot };
  `
}
```

## Common Patterns

### Automated Testing Workflow

```typescript
// AI prompt: "Test the login flow and capture results"

// 1. Navigate to login page
await callTool('navigate', { url: 'https://app.example.com/login' });

// 2. Fill credentials (use environment variables)
await callTool('fill', { 
  selector: 'input[name="username"]', 
  value: process.env.TEST_USERNAME 
});
await callTool('fill', { 
  selector: 'input[name="password"]', 
  value: process.env.TEST_PASSWORD 
});

// 3. Submit form
await callTool('click', { selector: 'button[type="submit"]' });

// 4. Wait for navigation
await callTool('wait', { selector: '.dashboard', timeout: 5000 });

// 5. Capture evidence
const screenshot = await callTool('screenshot', { fullPage: true });

// 6. Run accessibility audit
const audit = await callTool('accessibility_audit', {});
```

### Performance Testing

```typescript
// AI prompt: "Measure performance of the homepage"

const vitals = await callTool('web_vitals', { 
  url: 'https://example.com' 
});

// vitals contains: LCP, INP, CLS, TTFB, FCP
console.log(`LCP: ${vitals.LCP}ms`);
console.log(`CLS: ${vitals.CLS}`);
```

### API Mocking for Frontend Tests

```typescript
// AI prompt: "Mock the user API and test the profile page"

// 1. Set up mock
await callTool('mock_route', {
  pattern: '**/api/user/profile',
  response: {
    status: 200,
    body: {
      id: 1,
      name: 'Test User',
      email: 'test@example.com'
    }
  }
});

// 2. Navigate and test
await callTool('navigate', { url: 'https://app.example.com/profile' });

// 3. Verify UI
await callTool('screenshot', { selector: '.profile-card' });

// 4. Clean up
await callTool('unmock_route', { pattern: '**/api/user/profile' });
```

### Debugging with Tracepoints

```typescript
// AI prompt: "Debug the checkout flow without stopping execution"

// Set tracepoint at critical function
await callTool('set_tracepoint', {
  source: 'checkout.js',
  line: 78,
  condition: 'cart.total > 1000',
  logMessage: 'High value cart: {cart.total}, items: {cart.items.length}'
});

// Watch state changes
await callTool('watch_expression', {
  expression: 'this.state.checkoutStep',
  source: 'CheckoutComponent.jsx'
});

// Execute checkout flow
await callTool('navigate', { url: 'https://shop.example.com/checkout' });
// ... interact with page
```

### React Component Inspection

```typescript
// AI prompt: "Inspect the React component structure of the dashboard"

// Get full component tree
const tree = await callTool('react_get_component_tree', {});

// Inspect specific component
const component = await callTool('react_inspect_element', {
  selector: 'div[data-testid="dashboard"]'
});

// component contains: props, state, hooks, children
```

### Network Monitoring & Analysis

```typescript
// AI prompt: "Monitor API calls during user signup"

// Navigate to signup
await callTool('navigate', { url: 'https://app.example.com/signup' });

// Fill and submit form
await callTool('fill', { selector: 'input[name="email"]', value: 'test@example.com' });
await callTool('click', { selector: 'button.signup' });

// Get network requests
const requests = await callTool('network_requests', {
  filter: {
    url: '/api/*',
    method: 'POST'
  }
});

// Analyze response times and status codes
requests.forEach(req => {
  console.log(`${req.method} ${req.url}: ${req.status} (${req.duration}ms)`);
});
```

### Batch Execution for Complex Workflows

```typescript
// AI prompt: "Run complete E2E test with multiple assertions"

const result = await callTool('execute', {
  code: `
    // Navigate to app
    await callTool('navigate', { url: 'https://app.example.com' });
    
    // Check accessibility
    const audit = await callTool('accessibility_audit', { includeWarnings: false });
    if (audit.violations.length > 0) {
      throw new Error('Accessibility violations found');
    }
    
    // Measure performance
    const vitals = await callTool('web_vitals', { url: 'https://app.example.com' });
    if (vitals.LCP > 2500) {
      console.warn('LCP exceeds threshold');
    }
    
    // Interact with UI
    await page.click('button.start-tour');
    await page.waitForSelector('.tour-step-1');
    
    // Capture state
    const screenshot = await callTool('screenshot', { fullPage: true });
    
    return {
      audit: audit.violations.length === 0,
      performance: vitals,
      screenshot
    };
  `
});
```

## Troubleshooting

### Playwright Browser Download Fails

**Symptom:** Extension shows error during activation about browser download.

**Solutions:**

1. **Use system browser:**
   ```json
   {
     "browserDevtoolsMcp.browser.useSystemBrowser": true
   }
   ```

2. **Set custom executable path:**
   ```json
   {
     "browserDevtoolsMcp.browser.executablePath": "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
   }
   ```

3. **Skip download via environment variable:**
   ```bash
   export PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
   code  # or cursor
   ```

4. **Configure proxy if behind firewall:**
   ```json
   {
     "browserDevtoolsMcp.network.proxy": "http://proxy.company.com:8080"
   }
   ```

5. **Manually install browsers:**
   ```bash
   npx playwright install chromium
   ```

### MCP Server Not Registering

**Symptom:** AI assistant cannot access browser automation tools.

**Solutions:**

1. **Verify extension is enabled:**
   ```json
   {
     "browserDevtoolsMcp.enable": true
   }
   ```

2. **Check MCP server status in Command Palette:**
   - Run: **Browser DevTools MCP: Show Server Status**

3. **Restart MCP session:**
   - Run: **Browser DevTools MCP: Restart Server**

4. **Check VS Code/Cursor version:**
   - VS Code: Requires 1.96+
   - Cursor: Native MCP support required

### Headless Mode Issues

**Symptom:** Browser automation fails in headless mode but works in headed mode.

**Solution:** Disable headless for debugging:
```json
{
  "browserDevtoolsMcp.browser.headless": false
}
```

### Certificate Errors

**Symptom:** HTTPS errors preventing navigation.

**Solution:**
```json
{
  "browserDevtoolsMcp.network.ignoreHTTPSErrors": true
}
```

**Warning:** Only use for development/testing environments.

### Performance Issues

**Symptom:** Slow browser operations or timeouts.

**Solutions:**

1. **Enable persistent context:**
   ```json
   {
     "browserDevtoolsMcp.browser.persistent": true,
     "browserDevtoolsMcp.browser.userDataDir": "/path/to/userdata"
   }
   ```

2. **Adjust timeouts in tool calls:**
   ```typescript
   await callTool('click', { 
     selector: 'button', 
     timeout: 60000  // 60 seconds
   });
   ```

3. **Disable screenshots/videos:**
   ```json
   {
     "browserDevtoolsMcp.debug.screenshots": false,
     "browserDevtoolsMcp.debug.videos": false
   }
   ```

### Telemetry Opt-Out

**To disable telemetry:**

1. **Via settings:**
   ```json
   {
     "browserDevtoolsMcp.telemetry.enable": false
   }
   ```

2. **Via environment variable:**
   ```bash
   export TELEMETRY_ENABLE=false
   ```

3. **Via config file:**
   Edit `~/.browser-devtools-mcp/config.json`:
   ```json
   {
     "telemetryEnabled": false
   }
   ```

## Advanced Configuration

### OpenTelemetry Distributed Tracing

Enable tracing to monitor browser automation operations:

```json
{
  "browserDevtoolsMcp.otel.enable": true,
  "browserDevtoolsMcp.otel.exporterUrl": "http://localhost:4318/v1/traces",
  "browserDevtoolsMcp.otel.serviceName": "browser-automation",
  "browserDevtoolsMcp.otel.propagationHeaderName": "traceparent"
}
```

### Persistent Browser Context

Maintain browser state across sessions:

```json
{
  "browserDevtoolsMcp.browser.persistent": true,
  "browserDevtoolsMcp.browser.userDataDir": "${workspaceFolder}/.browser-data"
}
```

### CDP Attach Mode (Chromium Only)

Attach to existing browser instance:

```json
{
  "browserDevtoolsMcp.browser.cdp.enable": true,
  "browserDevtoolsMcp.browser.cdp.endpointUrl": "http://localhost:9222"
}
```

Start Chrome with remote debugging:
```bash
google-chrome --remote-debugging-port=9222
```

### Custom Locale

Test internationalization:

```json
{
  "browserDevtoolsMcp.browser.locale": "tr-TR"
}
```

## Best Practices

1. **Use environment variables for secrets:**
   ```typescript
   await callTool('fill', { 
     selector: 'input[name="apiKey"]', 
     value: process.env.API_KEY 
   });
   ```

2. **Enable debugging output for troubleshooting:**
   ```json
   {
     "browserDevtoolsMcp.debug.enable": true,
     "browserDevtoolsMcp.debug.outputDir": "${workspaceFolder}/debug-output"
   }
   ```

3. **Use batch execution for performance:**
   - Group related operations in `execute` tool
   - Access Playwright `page` object directly for complex interactions

4. **Mock external dependencies:**
   - Use `mock_route` to isolate frontend tests
   - Mock slow/unreliable APIs for consistent testing

5. **Leverage accessibility snapshots:**
   - Test keyboard navigation and screen reader compatibility
   - Validate ARIA attributes and semantic HTML

6. **Monitor performance continuously:**
   - Set up web vitals baselines
   - Alert on regressions in LCP, CLS, or INP

## References

- [Browser DevTools MCP NPM Package](https://www.npmjs.com/package/browser-devtools-mcp)
- [Playwright Documentation](https://playwright.dev/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Extension Repository](https://github.com/serkan-ozal/browser-devtools-mcp-vscode-extension)
