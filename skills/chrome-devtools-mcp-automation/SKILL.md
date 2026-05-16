---
name: chrome-devtools-mcp-automation
description: Use Chrome DevTools MCP to control Chrome, debug web apps, analyze performance, and automate browser tasks via MCP tools
triggers:
  - automate browser testing with chrome devtools
  - debug web application performance issues
  - take screenshots and analyze network requests
  - record performance traces in chrome
  - inspect browser console errors with source maps
  - control chrome browser for automated testing
  - analyze web vitals and performance metrics
  - interact with web pages using puppeteer automation
---

# Chrome DevTools MCP Automation

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Chrome DevTools MCP (`chrome-devtools-mcp`) is an MCP server that gives AI coding agents full control over Chrome browser automation, debugging, and performance analysis. It exposes Chrome DevTools Protocol and Puppeteer capabilities through MCP tools, enabling reliable browser automation, performance profiling, network inspection, and advanced debugging.

## Installation

### MCP Client Configuration

Add to your MCP client configuration (e.g., Claude Desktop, Cursor, VS Code):

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

**Slim mode** (basic browser tasks only):

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--slim", "--headless"]
    }
  }
}
```

**Disable usage statistics and update checks:**

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--no-usage-statistics"],
      "env": {
        "CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS": "1"
      }
    }
  }
}
```

### CLI Usage (Without MCP)

```bash
# Install globally
npm install -g chrome-devtools-mcp

# Run CLI
chrome-devtools-mcp --help
```

### Requirements

- Node.js v20.19+ (latest maintenance LTS)
- Chrome stable or newer
- npm

## Key MCP Tools

Once the MCP server is running, your AI agent has access to these tools:

### Browser Control

**`browser_navigate`** - Navigate to a URL
```typescript
// Navigate to a page
await use_mcp_tool("chrome-devtools", "browser_navigate", {
  url: "https://example.com"
});
```

**`browser_click`** - Click an element
```typescript
// Click by selector
await use_mcp_tool("chrome-devtools", "browser_click", {
  selector: "button.submit"
});

// Click with custom wait
await use_mcp_tool("chrome-devtools", "browser_click", {
  selector: "#login-btn",
  waitUntil: "networkidle2"
});
```

**`browser_type`** - Type text into an input
```typescript
// Type into a field
await use_mcp_tool("chrome-devtools", "browser_type", {
  selector: "input[name='email']",
  text: "user@example.com"
});

// Type with delay between keystrokes
await use_mcp_tool("chrome-devtools", "browser_type", {
  selector: "#search",
  text: "search query",
  delay: 100
});
```

**`browser_screenshot`** - Capture screenshot
```typescript
// Full page screenshot
await use_mcp_tool("chrome-devtools", "browser_screenshot", {
  fullPage: true
});

// Element screenshot
await use_mcp_tool("chrome-devtools", "browser_screenshot", {
  selector: ".main-content",
  fullPage: false
});
```

**`browser_evaluate`** - Execute JavaScript in browser
```typescript
// Get page data
await use_mcp_tool("chrome-devtools", "browser_evaluate", {
  script: "document.title"
});

// Interact with page
await use_mcp_tool("chrome-devtools", "browser_evaluate", {
  script: `
    const items = Array.from(document.querySelectorAll('.item'));
    return items.map(el => ({
      text: el.textContent,
      href: el.querySelector('a')?.href
    }));
  `
});
```

### Performance Analysis

**`performance_record`** - Record performance trace
```typescript
// Record a page load
await use_mcp_tool("chrome-devtools", "performance_record", {
  url: "https://example.com",
  duration: 10000, // 10 seconds
  throttling: "4g" // Simulate 4G network
});
```

**`performance_analyze`** - Analyze performance metrics
```typescript
// Get performance insights
await use_mcp_tool("chrome-devtools", "performance_analyze", {
  traceUrl: "trace-data-url",
  includeFieldData: true // Include CrUX data
});
```

### Debugging & Inspection

**`browser_console`** - Get console messages
```typescript
// Fetch console logs
await use_mcp_tool("chrome-devtools", "browser_console", {});
```

**`browser_network`** - Inspect network requests
```typescript
// Get network activity
await use_mcp_tool("chrome-devtools", "browser_network", {});

// Filter by resource type
await use_mcp_tool("chrome-devtools", "browser_network", {
  resourceType: "xhr"
});
```

**`browser_cookies`** - Manage cookies
```typescript
// Get all cookies
await use_mcp_tool("chrome-devtools", "browser_cookies", {
  action: "get"
});

// Set a cookie
await use_mcp_tool("chrome-devtools", "browser_cookies", {
  action: "set",
  name: "session_id",
  value: "abc123",
  domain: "example.com"
});

// Delete cookies
await use_mcp_tool("chrome-devtools", "browser_cookies", {
  action: "delete",
  name: "session_id"
});
```

## Common Automation Patterns

### Form Submission Testing

```typescript
// Navigate to login page
await use_mcp_tool("chrome-devtools", "browser_navigate", {
  url: "https://app.example.com/login"
});

// Fill in credentials
await use_mcp_tool("chrome-devtools", "browser_type", {
  selector: "input[name='email']",
  text: "test@example.com"
});

await use_mcp_tool("chrome-devtools", "browser_type", {
  selector: "input[name='password']",
  text: process.env.TEST_PASSWORD
});

// Submit form
await use_mcp_tool("chrome-devtools", "browser_click", {
  selector: "button[type='submit']",
  waitUntil: "networkidle0"
});

// Verify success
const result = await use_mcp_tool("chrome-devtools", "browser_evaluate", {
  script: "document.querySelector('.success-message')?.textContent"
});
```

### Web Scraping

```typescript
// Navigate to page
await use_mcp_tool("chrome-devtools", "browser_navigate", {
  url: "https://news.example.com"
});

// Extract data
const articles = await use_mcp_tool("chrome-devtools", "browser_evaluate", {
  script: `
    Array.from(document.querySelectorAll('article')).map(article => ({
      title: article.querySelector('h2')?.textContent?.trim(),
      link: article.querySelector('a')?.href,
      date: article.querySelector('time')?.getAttribute('datetime'),
      summary: article.querySelector('p')?.textContent?.trim()
    }))
  `
});
```

### Performance Audit

```typescript
// Record trace with throttling
const trace = await use_mcp_tool("chrome-devtools", "performance_record", {
  url: "https://example.com",
  duration: 15000,
  throttling: "3g",
  deviceEmulation: "mobile"
});

// Analyze metrics
const analysis = await use_mcp_tool("chrome-devtools", "performance_analyze", {
  traceUrl: trace.url,
  includeFieldData: true
});

// Check Core Web Vitals
console.log(`LCP: ${analysis.lcp}ms`);
console.log(`FID: ${analysis.fid}ms`);
console.log(`CLS: ${analysis.cls}`);
```

### Network Monitoring

```typescript
// Navigate and monitor network
await use_mcp_tool("chrome-devtools", "browser_navigate", {
  url: "https://api.example.com/dashboard"
});

// Get network requests
const requests = await use_mcp_tool("chrome-devtools", "browser_network", {});

// Filter failed requests
const failed = requests.filter(r => r.status >= 400);

// Find slow requests
const slow = requests.filter(r => r.time > 1000);
```

### Screenshot Comparison

```typescript
// Baseline screenshot
await use_mcp_tool("chrome-devtools", "browser_navigate", {
  url: "https://example.com"
});

const baseline = await use_mcp_tool("chrome-devtools", "browser_screenshot", {
  fullPage: true
});

// Make changes via evaluate
await use_mcp_tool("chrome-devtools", "browser_evaluate", {
  script: "document.body.classList.add('dark-mode')"
});

// Comparison screenshot
const comparison = await use_mcp_tool("chrome-devtools", "browser_screenshot", {
  fullPage: true
});
```

### Console Error Debugging

```typescript
// Navigate to page
await use_mcp_tool("chrome-devtools", "browser_navigate", {
  url: "https://app.example.com"
});

// Check console for errors
const console = await use_mcp_tool("chrome-devtools", "browser_console", {});

// Filter errors with source maps
const errors = console.filter(msg => 
  msg.type === 'error' && msg.stackTrace
);

// Log error details
errors.forEach(error => {
  console.log(`Error: ${error.text}`);
  console.log(`Source: ${error.stackTrace?.[0]?.url}`);
  console.log(`Line: ${error.stackTrace?.[0]?.lineNumber}`);
});
```

## Configuration Options

### Command-Line Flags

```bash
# Run in headless mode
npx chrome-devtools-mcp@latest --headless

# Slim mode (basic tools only)
npx chrome-devtools-mcp@latest --slim

# Disable performance CrUX data
npx chrome-devtools-mcp@latest --no-performance-crux

# Disable usage statistics
npx chrome-devtools-mcp@latest --no-usage-statistics

# Connect to existing browser
npx chrome-devtools-mcp@latest --browser-url=http://127.0.0.1:9222

# Custom Chrome path
npx chrome-devtools-mcp@latest --chrome-path=/path/to/chrome
```

### Environment Variables

```bash
# Disable usage statistics
export CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS=1

# Disable update checks
export CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS=1

# CI mode (disables stats collection)
export CI=true
```

### MCP Server Configuration Examples

**Headless with custom timeout:**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--headless"],
      "env": {
        "CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS": "1"
      }
    }
  }
}
```

**Connect to existing browser (Antigravity):**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest",
        "--browser-url=http://127.0.0.1:9222"
      ]
    }
  }
}
```

**Windows configuration:**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "cmd",
      "args": [
        "/c",
        "npx",
        "-y",
        "chrome-devtools-mcp@latest"
      ],
      "env": {
        "SystemRoot": "C:\\Windows",
        "PROGRAMFILES": "C:\\Program Files"
      }
    }
  }
}
```

## Troubleshooting

### Browser Won't Start

**Issue:** Chrome fails to launch

**Solution:**
1. Verify Chrome is installed: `chrome --version` or `google-chrome --version`
2. Check Node.js version: `node --version` (must be 20.19+)
3. Use explicit Chrome path:
   ```json
   {
     "args": ["-y", "chrome-devtools-mcp@latest", "--chrome-path=/usr/bin/google-chrome"]
   }
   ```

### Connection Timeout

**Issue:** MCP server fails to connect to browser

**Solution:**
1. Increase startup timeout in config:
   ```json
   {
     "startup_timeout_ms": 20000
   }
   ```
2. Check if another Chrome instance is using port 9222
3. Kill existing Chrome processes: `pkill chrome` (Linux/Mac)

### Selector Not Found

**Issue:** `browser_click` or `browser_type` fails to find element

**Solution:**
1. Wait for element to appear:
   ```typescript
   await use_mcp_tool("chrome-devtools", "browser_evaluate", {
     script: `
       await new Promise(resolve => {
         const check = setInterval(() => {
           if (document.querySelector('#my-element')) {
             clearInterval(check);
             resolve();
           }
         }, 100);
       });
     `
   });
   ```
2. Use more specific selectors (ID > class > tag)
3. Check if element is in iframe (not currently supported)

### Performance Trace Fails

**Issue:** `performance_record` returns incomplete trace

**Solution:**
1. Increase duration: `duration: 30000`
2. Disable throttling for local testing: remove `throttling` parameter
3. Check network connectivity for remote URLs

### Source Maps Not Loading

**Issue:** Console errors don't show source-mapped stack traces

**Solution:**
1. Ensure source maps are published with your build
2. Verify source map URLs are accessible
3. Check CORS headers on source map files

### High Memory Usage

**Issue:** Chrome consumes excessive memory

**Solution:**
1. Use `--headless` mode
2. Close browser between test runs
3. Use `--slim` mode if you don't need performance tools
4. Limit trace duration in `performance_record`

### Rate Limiting (CrUX API)

**Issue:** Performance analysis fails with rate limit error

**Solution:**
1. Disable CrUX: `--no-performance-crux`
2. Cache analysis results for repeated URLs
3. Add delay between performance audits

## Best Practices

1. **Always use headless mode in CI/CD**: Add `--headless` flag
2. **Use environment variables for sensitive data**: Never hardcode credentials
3. **Wait for navigation**: Use `waitUntil: "networkidle0"` for SPAs
4. **Cache selectors**: Store frequently used selectors as constants
5. **Clean up resources**: Close browser instances after automation completes
6. **Enable source maps**: For better debugging of console errors
7. **Use slim mode for simple tasks**: Faster startup, lower memory usage
8. **Throttle performance tests**: Simulate real-world conditions with `throttling: "4g"`

## Additional Resources

- [Tool Reference](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md)
- [Slim Tool Reference](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/slim-tool-reference.md)
- [CLI Documentation](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/cli.md)
- [Troubleshooting Guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md)
- [Design Principles](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/design-principles.md)
