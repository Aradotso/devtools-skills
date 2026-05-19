---
name: firefox-devtools-mcp-automation
description: Automate and control Firefox browser through MCP using WebDriver BiDi for AI-assisted web testing, scraping, and interaction
triggers:
  - automate Firefox with MCP
  - control Firefox browser from Claude
  - take screenshots and interact with web pages
  - use Firefox DevTools MCP server
  - inspect web pages with AI assistance
  - automate web testing in Firefox
  - control browser through Model Context Protocol
  - use WebDriver BiDi with MCP
---

# Firefox DevTools MCP Automation

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

Firefox DevTools MCP is a Model Context Protocol server that enables AI assistants to inspect and control Firefox browser through the Remote Debugging Protocol (WebDriver BiDi). It provides tools for:

- Page navigation and management (open, close, switch tabs)
- Element interaction (click, fill forms, hover, drag)
- Visual inspection (screenshots, snapshots with UIDs)
- Network monitoring (capture requests/responses)
- Console log access
- JavaScript execution
- Firefox management and preferences
- WebExtension installation

## Security Considerations

**Always use a dedicated Firefox profile** — the agent has access to cookies, sessions, and everything the browser can reach.

- Avoid visiting untrusted sites (risk of prompt injection)
- Only enable `--enable-script` and `--enable-privileged-context` when necessary
- Never run against your regular browsing profile

## Installation

### Quick Start with npx (Recommended)

```bash
# Add to Claude Code
claude mcp add firefox-devtools npx firefox-devtools-mcp@latest

# With options
claude mcp add firefox-devtools npx firefox-devtools-mcp@latest -- --headless --viewport 1280x720
```

### Manual Configuration

Add to MCP settings JSON (`~/Library/Application Support/Claude/Code/mcp_settings.json` on macOS):

```json
{
  "mcpServers": {
    "firefox-devtools": {
      "command": "npx",
      "args": ["-y", "firefox-devtools-mcp@latest", "--headless"],
      "env": {
        "START_URL": "about:home",
        "FIREFOX_HEADLESS": "true"
      }
    }
  }
}
```

### Test with MCP Inspector

```bash
npx @modelcontextprotocol/inspector npx firefox-devtools-mcp@latest --start-url https://example.com --headless
```

## Configuration Options

### CLI Flags

```bash
# Basic options
--firefox-path /path/to/firefox        # Custom Firefox binary path
--headless                              # Run without UI
--viewport 1280x720                     # Initial window size
--profile-path /path/to/profile         # Use specific Firefox profile
--start-url https://example.com         # Open URL on start

# Security options
--accept-insecure-certs                 # Ignore TLS errors
--enable-script                         # Enable evaluate_script tool
--enable-privileged-context             # Enable privileged tools

# Connection options
--connect-existing                      # Attach to running Firefox
--marionette-port 2828                  # Marionette port (default: 2828)

# Advanced options
--firefox-arg --arg-name                # Extra Firefox arguments (repeatable)
--pref name=value                       # Set Firefox preference (repeatable)
```

### Environment Variables

```bash
FIREFOX_HEADLESS=true
START_URL=https://example.com
ACCEPT_INSECURE_CERTS=true
CONNECT_EXISTING=true
MARIONETTE_PORT=2828
ENABLE_SCRIPT=true
ENABLE_PRIVILEGED_CONTEXT=true
```

## Core Workflow

### 1. Page Navigation and Management

```typescript
// List all open pages/tabs
list_pages()
// Returns: { pages: [{ id, url, title, isActive }] }

// Create new page
new_page({ url: "https://example.com" })
// Returns: { pageId, url }

// Navigate existing page
navigate_page({ 
  pageId: "page-id-123", 
  url: "https://example.com/login" 
})

// Switch active page
select_page({ pageId: "page-id-123" })

// Close page
close_page({ pageId: "page-id-123" })
```

### 2. Snapshot and UID-Based Interaction

The snapshot system assigns unique UIDs to interactive elements, enabling precise interaction without CSS selectors.

```typescript
// Take snapshot of current page
take_snapshot()
// Returns: { 
//   snapshot: "Visual representation with [uid] markers",
//   elements: [{ uid, role, name, tagName, coordinates }]
// }

// Click element by UID
click_by_uid({ uid: "abc123" })

// Fill input field
fill_by_uid({ 
  uid: "def456", 
  value: "user@example.com" 
})

// Hover over element
hover_by_uid({ uid: "ghi789" })

// Drag and drop
drag_by_uid({ 
  sourceUid: "drag-me", 
  targetUid: "drop-here" 
})

// Upload file
upload_by_uid({ 
  uid: "file-input", 
  filePath: "/path/to/file.pdf" 
})

// Fill entire form at once
fill_form_by_uids({
  fields: [
    { uid: "email-field", value: "user@example.com" },
    { uid: "password-field", value: "secure-password" },
    { uid: "submit-button", action: "click" }
  ]
})

// Clear UIDs after navigation
clear_uids()
```

### 3. Visual Capture

```typescript
// Screenshot entire page
screenshot_page()
// Returns: { screenshot: "base64-encoded-png" }

// Save to disk (recommended for Claude Code to save context)
screenshot_page({ 
  saveTo: "/tmp/page.png" 
})
// Returns: { filePath: "/tmp/page.png" }

// Screenshot specific element
screenshot_by_uid({ 
  uid: "abc123",
  saveTo: "/tmp/element.png" 
})
```

### 4. Network Monitoring

Network capture is always-on. Use ID-first approach to inspect requests:

```typescript
// List all captured network requests
list_network_requests()
// Returns: { 
//   requests: [{ 
//     id, method, url, status, 
//     contentType, responseSize, duration 
//   }] 
// }

// Filter requests
list_network_requests({
  filter: {
    method: "POST",
    status: 200,
    urlPattern: "api.example.com"
  }
})

// Get detailed request/response
get_network_request({ requestId: "req-123" })
// Returns: {
//   request: { method, url, headers, body },
//   response: { status, headers, body }
// }
```

### 5. Console Messages

```typescript
// List console logs
list_console_messages()
// Returns: { 
//   messages: [{ 
//     level, text, timestamp, 
//     url, lineNumber 
//   }] 
// }

// Clear console
clear_console_messages()
```

### 6. JavaScript Execution

**Requires `--enable-script` flag**

```typescript
// Execute JavaScript in page context
evaluate_script({ 
  script: "document.title" 
})
// Returns: { result: "Page Title" }

// Access DOM
evaluate_script({ 
  script: `
    const button = document.querySelector('.submit-btn');
    return button ? button.textContent : null;
  `
})

// Modify page
evaluate_script({ 
  script: `
    document.body.style.backgroundColor = 'lightblue';
    return 'Background changed';
  `
})
```

### 7. Browser Utilities

```typescript
// Navigate history
history_back()
history_forward()

// Set viewport size
set_viewport({ width: 1920, height: 1080 })

// Handle dialogs
accept_dialog({ promptText: "optional text for prompts" })
dismiss_dialog()

// Get Firefox info
get_firefox_info()
// Returns: { version, buildId, profilePath, binaryPath }

// View Firefox console output
get_firefox_output()
// Returns: { stdout, stderr }

// Restart Firefox
restart_firefox()
```

## Common Patterns

### Login Flow

```typescript
// 1. Navigate to login page
navigate_page({ 
  pageId: "page-1", 
  url: "https://app.example.com/login" 
})

// 2. Take snapshot to get UIDs
const snap = take_snapshot()
// Inspect snap.snapshot to find email/password/submit UIDs

// 3. Fill and submit form
fill_form_by_uids({
  fields: [
    { uid: "email-uid", value: process.env.USER_EMAIL },
    { uid: "password-uid", value: process.env.USER_PASSWORD },
    { uid: "submit-uid", action: "click" }
  ]
})

// 4. Wait and verify success
// (add delay or check for redirect)
```

### Web Scraping with Network Monitoring

```typescript
// 1. Navigate to page
navigate_page({ 
  pageId: "page-1", 
  url: "https://api-site.example.com/dashboard" 
})

// 2. Interact to trigger API calls
const snap = take_snapshot()
click_by_uid({ uid: "load-more-button-uid" })

// 3. Capture API responses
const requests = list_network_requests({
  filter: { 
    urlPattern: "api.example.com/v1/data",
    method: "GET"
  }
})

// 4. Extract data from response
const apiData = get_network_request({ 
  requestId: requests.requests[0].id 
})
// Parse apiData.response.body
```

### Visual Regression Testing

```typescript
// 1. Navigate to page
navigate_page({ 
  pageId: "page-1", 
  url: "https://example.com/component" 
})

// 2. Take baseline screenshot
screenshot_page({ saveTo: "/tmp/baseline.png" })

// 3. Make changes via script or interaction
evaluate_script({ 
  script: `document.querySelector('.hero').style.fontSize = '24px'`
})

// 4. Take comparison screenshot
screenshot_page({ saveTo: "/tmp/comparison.png" })

// 5. Use image diff tool externally
```

### Form Automation

```typescript
// Multi-step form with snapshots at each step
const pages = list_pages()
const pageId = pages.pages[0].id

// Step 1
let snap = take_snapshot()
fill_form_by_uids({
  fields: [
    { uid: "first-name-uid", value: "John" },
    { uid: "last-name-uid", value: "Doe" },
    { uid: "next-btn-uid", action: "click" }
  ]
})

// Step 2 - take fresh snapshot after navigation
snap = take_snapshot()
fill_form_by_uids({
  fields: [
    { uid: "address-uid", value: "123 Main St" },
    { uid: "city-uid", value: "Anytown" },
    { uid: "submit-uid", action: "click" }
  ]
})
```

## Advanced Features

### Privileged Context (Firefox Internals)

**Requires `--enable-privileged-context` flag and `MOZ_REMOTE_ALLOW_SYSTEM_ACCESS=1` environment variable**

```typescript
// List privileged contexts (chrome contexts)
list_privileged_contexts()
// Returns: { contexts: [{ id, type }] }

// Select privileged context
select_privileged_context({ contextId: "chrome-123" })

// Execute privileged JavaScript
evaluate_privileged_script({ 
  script: `
    Services.prefs.setIntPref("browser.cache.disk.capacity", 1024000);
    return "Pref set";
  `
})

// Get Firefox preferences
get_firefox_prefs({ prefNames: ["browser.cache.disk.capacity"] })
// Returns: { prefs: { "browser.cache.disk.capacity": 1024000 } }

// Set Firefox preferences
set_firefox_prefs({ 
  prefs: { 
    "browser.cache.disk.capacity": 2048000,
    "privacy.trackingprotection.enabled": true
  }
})
```

### WebExtension Management

```typescript
// Install extension
install_extension({ 
  xpiPath: "/path/to/extension.xpi" 
})
// Returns: { extensionId: "extension-uuid" }

// Uninstall extension
uninstall_extension({ extensionId: "extension-uuid" })

// List installed extensions (requires privileged context)
list_extensions()
// Returns: { extensions: [{ id, name, version }] }
```

### Connect to Existing Firefox

Automate your real browsing session with cookies and logins intact:

```bash
# Terminal 1: Start Firefox with Marionette
firefox --marionette

# Terminal 2: Connect MCP server
npx firefox-devtools-mcp@latest --connect-existing --marionette-port 2828
```

**Note:** BiDi-dependent features (console/network events) are not available in connect-existing mode.

## Troubleshooting

### Firefox Not Found

```bash
# macOS
npx firefox-devtools-mcp@latest --firefox-path "/Applications/Firefox.app/Contents/MacOS/firefox"

# Linux
npx firefox-devtools-mcp@latest --firefox-path "/usr/bin/firefox"

# Windows
npx firefox-devtools-mcp@latest --firefox-path "C:\Program Files\Mozilla Firefox\firefox.exe"
```

### Stale UIDs After Navigation

Always take a fresh snapshot after page changes:

```typescript
navigate_page({ pageId: "page-1", url: "https://new-page.com" })
clear_uids()  // Clear old UIDs
const snap = take_snapshot()  // Get fresh UIDs
```

### Windows 10 Connection Issues

Wrap command with `cmd /c`:

```json
{
  "mcpServers": {
    "firefox-devtools": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "firefox-devtools-mcp@latest"]
    }
  }
}
```

Or use absolute path to npx:

```json
{
  "mcpServers": {
    "firefox-devtools": {
      "command": "C:\\Program Files\\nodejs\\npx.cmd",
      "args": ["-y", "firefox-devtools-mcp@latest"]
    }
  }
}
```

### Debugging Connection Issues

```bash
# Run with inspector to see detailed logs
npx @modelcontextprotocol/inspector npx firefox-devtools-mcp@latest --start-url about:home

# Check Firefox is running
ps aux | grep firefox

# Verify Marionette port
lsof -i :2828
```

## Example: Complete E2E Test

```typescript
// Create new page
const { pageId } = new_page({ url: "https://example.com" })

// Navigate to search
navigate_page({ pageId, url: "https://example.com/search" })

// Take snapshot and find search box
const snap = take_snapshot()
// Review snap.snapshot to identify UIDs

// Perform search
fill_form_by_uids({
  fields: [
    { uid: "search-input-uid", value: "test query" },
    { uid: "search-button-uid", action: "click" }
  ]
})

// Capture results
const screenshot = screenshot_page({ saveTo: "/tmp/results.png" })

// Verify API calls
const requests = list_network_requests({
  filter: { urlPattern: "example.com/api/search" }
})
const searchData = get_network_request({ 
  requestId: requests.requests[0].id 
})

// Check console for errors
const logs = list_console_messages()
const errors = logs.messages.filter(m => m.level === "error")

// Cleanup
close_page({ pageId })
```

## Resources

- GitHub: https://github.com/mozilla/firefox-devtools-mcp
- npm: https://www.npmjs.com/package/firefox-devtools-mcp
- Security Guide: https://github.com/mozilla/firefox-devtools-mcp/blob/main/SECURITY.md
- Contributing: https://github.com/mozilla/firefox-devtools-mcp/blob/main/CONTRIBUTING.md
