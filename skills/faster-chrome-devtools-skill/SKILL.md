---
name: faster-chrome-devtools-skill
description: Control Chrome directly via DevTools Protocol with snapshots, clicks, waits, and screenshots
triggers:
  - "automate chrome browser tasks"
  - "take a screenshot of this webpage"
  - "click on an element in the browser"
  - "read the accessibility tree from chrome"
  - "navigate to a url and wait for it to load"
  - "inspect console errors in chrome"
  - "fill out a form in the browser"
  - "connect to a remote chrome instance"
---

# faster-chrome-devtools-skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

An agent skill and CLI for controlling Chrome directly through the Chrome DevTools Protocol (CDP). Provides fast accessibility snapshots, element interaction, navigation with explicit timeouts, screenshot capture, and console/network inspection without heavy dependencies like Puppeteer or Playwright.

## Installation

Install globally for all agents:

```sh
npx skills add zeke/faster-chrome-devtools-skill --global --all --yes
```

Requires Node.js installed. No additional dependencies needed.

## Core Concepts

The tool uses a lightweight background daemon that holds the CDP WebSocket connection open for 20 minutes, avoiding repeated Chrome access prompts. Connection details are stored in an owner-readable temp file with a random auth token.

**Three connection modes:**

1. **Logged-in Chrome**: Connect to your existing Chrome instance (requires enabling remote debugging at `chrome://inspect/#remote-debugging`)
2. **Anonymous Chromium**: Launch a fresh temporary instance with `--launch`
3. **Remote/Cloud**: Connect to Cloudflare Browser Run or other remote endpoints

## CLI Commands

All commands use `node scripts/cdp.mjs <command>` format.

### Navigation & Page Control

```bash
# Navigate to URL with explicit timeout
node scripts/cdp.mjs navigate https://example.com --timeout 10000

# Navigate and wait for specific text
node scripts/cdp.mjs navigate https://github.com --wait-text "Pull requests"

# Navigate and wait for selector
node scripts/cdp.mjs navigate https://example.com --wait-selector "#main-content"

# List all open tabs
node scripts/cdp.mjs list-tabs

# Open new tab
node scripts/cdp.mjs open-tab https://news.ycombinator.com

# Switch to existing tab by ID
node scripts/cdp.mjs switch-tab <tab-id>

# Close current tab
node scripts/cdp.mjs close-tab
```

### Reading Page Content

```bash
# Get compact accessibility snapshot with stable element references
node scripts/cdp.mjs snapshot

# Get full page text content
node scripts/cdp.mjs get-text

# Capture screenshot (JPEG by default for compression)
node scripts/cdp.mjs screenshot --path screenshot.jpg

# Capture WebP screenshot
node scripts/cdp.mjs screenshot --path screenshot.webp --format webp

# Capture PNG screenshot
node scripts/cdp.mjs screenshot --path screenshot.png --format png

# Full page screenshot
node scripts/cdp.mjs screenshot --path full.jpg --full-page
```

### Element Interaction

```bash
# Click by accessibility reference (from snapshot)
node scripts/cdp.mjs click --ref "button-login-123"

# Click by CSS selector
node scripts/cdp.mjs click --selector "button.submit"

# Fill input by accessibility reference
node scripts/cdp.mjs fill --ref "input-email-456" --value "user@example.com"

# Fill input by CSS selector
node scripts/cdp.mjs fill --selector "#email" --value "user@example.com"

# Type text (works in cross-origin frames via native CDP)
node scripts/cdp.mjs type "Hello world" --delay 50
```

### Wait Operations

```bash
# Wait for text to appear (default 30s timeout)
node scripts/cdp.mjs wait-text "Success" --timeout 15000

# Wait for selector to exist
node scripts/cdp.mjs wait-selector ".results-loaded"

# Wait for specific duration
node scripts/cdp.mjs wait --duration 2000
```

### Debugging & Inspection

```bash
# Get console messages
node scripts/cdp.mjs get-console

# Get failed network requests
node scripts/cdp.mjs get-failed-loads

# Evaluate JavaScript
node scripts/cdp.mjs eval "document.title"

# Execute multi-line JavaScript
node scripts/cdp.mjs eval "
const links = Array.from(document.querySelectorAll('a'));
return links.map(a => ({ text: a.textContent, href: a.href }));
"

# Call raw CDP method
node scripts/cdp.mjs raw Runtime.evaluate '{"expression": "navigator.userAgent"}'
```

### Connection Management

```bash
# Launch anonymous Chrome instance
node scripts/cdp.mjs snapshot --launch

# Connect to specific WebSocket endpoint
node scripts/cdp.mjs snapshot --ws-endpoint ws://localhost:9222/devtools/browser/xyz

# Connect to HTTP endpoint (auto-discovers WebSocket)
node scripts/cdp.mjs snapshot --http-endpoint http://localhost:9222

# Stop the daemon
node scripts/cdp.mjs stop

# Stop specific daemon by ID
node scripts/cdp.mjs stop --id <daemon-id>

# Stop all running daemons
node scripts/cdp.mjs stop --all

# Stop daemon for specific endpoint
node scripts/cdp.mjs stop --ws-endpoint ws://localhost:9222/devtools/browser/xyz
```

## Configuration

### Using Your Logged-in Chrome

1. Open Chrome and navigate to `chrome://inspect/#remote-debugging`
2. Check "Discover network targets" 
3. Click "Configure" and ensure `localhost:9222` is listed
4. Launch Chrome with remote debugging:

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222

# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

### Cloudflare Browser Run Setup

Set these environment variables:

```bash
export CLOUDFLARE_ACCOUNT_ID="your-account-id"
export CLOUDFLARE_API_TOKEN="your-api-token"
```

Then use:

```bash
node scripts/cdp.mjs snapshot --http-endpoint https://browser.run.cloudflare.com
```

The CLI automatically adds the required authentication headers.

## Common Patterns

### Scrape Structured Data

```javascript
// 1. Navigate and wait for content
// node scripts/cdp.mjs navigate https://news.ycombinator.com --wait-selector ".itemlist"

// 2. Get accessibility snapshot to find element references
// node scripts/cdp.mjs snapshot

// 3. Extract data with JavaScript
// node scripts/cdp.mjs eval "
const stories = Array.from(document.querySelectorAll('.athing')).slice(0, 5);
return stories.map(story => ({
  title: story.querySelector('.titleline > a')?.textContent,
  url: story.querySelector('.titleline > a')?.href
}));
// "
```

### Login Flow

```bash
# Navigate to login page
node scripts/cdp.mjs navigate https://example.com/login --wait-selector "#username"

# Fill credentials
node scripts/cdp.mjs fill --selector "#username" --value "$USERNAME"
node scripts/cdp.mjs fill --selector "#password" --value "$PASSWORD"

# Submit form
node scripts/cdp.mjs click --selector "button[type=submit]"

# Wait for redirect
node scripts/cdp.mjs wait-text "Dashboard" --timeout 10000

# Take screenshot to verify
node scripts/cdp.mjs screenshot --path logged-in.jpg
```

### Multi-tab Workflow

```bash
# List current tabs
node scripts/cdp.mjs list-tabs

# Open new tab
node scripts/cdp.mjs open-tab https://github.com

# Work in new tab
node scripts/cdp.mjs snapshot

# Switch back to first tab
node scripts/cdp.mjs switch-tab <first-tab-id>

# Close current tab when done
node scripts/cdp.mjs close-tab
```

### Monitor Page for Changes

```bash
# Navigate to page
node scripts/cdp.mjs navigate https://example.com/status

# Take initial snapshot
node scripts/cdp.mjs snapshot > before.txt

# Wait some time
node scripts/cdp.mjs wait --duration 5000

# Take second snapshot
node scripts/cdp.mjs snapshot > after.txt

# Compare with diff tool
diff before.txt after.txt
```

### Debugging Failed Loads

```bash
# Navigate to page
node scripts/cdp.mjs navigate https://example.com

# Check for failed network requests
node scripts/cdp.mjs get-failed-loads

# Check console errors
node scripts/cdp.mjs get-console

# Take screenshot for visual inspection
node scripts/cdp.mjs screenshot --path debug.jpg
```

## Accessibility Snapshot Format

The `snapshot` command returns a compact tree with stable references:

```json
{
  "role": "WebArea",
  "name": "Example Page",
  "ref": "page-1",
  "children": [
    {
      "role": "button",
      "name": "Submit",
      "ref": "button-submit-42",
      "focusable": true
    },
    {
      "role": "textbox",
      "name": "Email",
      "ref": "input-email-43",
      "focusable": true,
      "value": ""
    }
  ]
}
```

Use the `ref` values with `--ref` flags in `click` and `fill` commands for reliable element targeting.

## Troubleshooting

### "Connection refused" when connecting to Chrome

- Ensure Chrome is running with `--remote-debugging-port=9222`
- Check that no firewall is blocking localhost:9222
- Verify `chrome://inspect/#remote-debugging` shows the debugging port

### "Target closed" or connection drops

- The daemon times out after 20 minutes of inactivity
- Restart with any command that reconnects
- Use `node scripts/cdp.mjs stop` to clean up stale daemons

### Screenshots are empty or broken

- Wait for page to fully load before capturing
- Use `--wait-selector` or `--wait-text` with navigate
- Try `--full-page` flag if content is below fold
- Check console/failed loads for rendering errors

### Elements not found by selector

- Use `snapshot` first to get stable accessibility references
- Prefer `--ref` over `--selector` for reliability
- Use `wait-selector` before clicking/filling
- Check that selector syntax is correct for the page's structure

### Cloudflare Browser Run authentication fails

- Verify `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` are set
- Check token has Browser Rendering permissions
- Ensure endpoint URL is `https://browser.run.cloudflare.com`

### Commands hang or timeout

- Increase `--timeout` values for slow pages
- Check network connectivity to target site
- Use `get-console` and `get-failed-loads` to diagnose page errors
- Try navigating to simpler test page first to verify connection
