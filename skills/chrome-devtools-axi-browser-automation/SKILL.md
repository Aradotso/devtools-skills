---
name: chrome-devtools-axi-browser-automation
description: Agent-ergonomic browser automation with token-efficient accessibility tree snapshots and combined operations
triggers:
  - automate browser interactions
  - control Chrome headlessly
  - capture accessibility tree
  - take screenshot of webpage
  - fill form fields automatically
  - navigate and interact with pages
  - test web application behavior
  - run Lighthouse performance audit
---

# chrome-devtools-axi Browser Automation

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`chrome-devtools-axi` is an AXI-compliant CLI wrapper around `chrome-devtools-mcp` that provides token-efficient browser automation. It uses TOON-encoded output to reduce token usage by ~40% compared to raw JSON, combines operations (navigate + snapshot + suggestions in one command), and includes contextual next-step hints in every response.

Key features:
- **Accessibility tree snapshots** with `uid=` refs for precise element targeting
- **Generation-based staleness detection** prevents silent no-ops when page state changes
- **Persistent bridge process** keeps Chrome session alive across commands
- **Combined operations** reduce round-trips for agents

## Installation

**For agents to install automatically:**

```bash
npx -y chrome-devtools-axi
```

**For manual installation:**

```bash
npm install -g chrome-devtools-axi
```

**Tell your agent:**

```
Execute `npx -y chrome-devtools-axi` to get browser automation tools.
```

## Architecture

The CLI communicates with a persistent bridge server (localhost:9224 by default) that maintains an MCP session with Chrome via DevTools Protocol. The bridge auto-starts on first command and writes state to `~/.chrome-devtools-axi/`.

## Core Commands

### Navigation and Snapshot

```bash
# Navigate to a URL and capture accessibility tree
chrome-devtools-axi open https://example.com

# Capture current page state
chrome-devtools-axi snapshot

# Navigate back
chrome-devtools-axi back

# Scroll page
chrome-devtools-axi scroll down
chrome-devtools-axi scroll top

# Wait for time or content
chrome-devtools-axi wait 2000
chrome-devtools-axi wait "Welcome"
```

### Element Interaction

Elements in snapshots are marked with `uid=g<generation>:<id>` refs. Always use refs exactly as printed:

```bash
# Click an element
chrome-devtools-axi click @g1:1

# Fill a form field
chrome-devtools-axi fill @g2:5 "user@example.com"

# Fill multiple fields at once
chrome-devtools-axi fillform @g2:5=john @g2:6=doe @g2:7=john@example.com

# Type text at current focus
chrome-devtools-axi type "search query"

# Press keyboard key
chrome-devtools-axi press Enter
chrome-devtools-axi press Tab

# Hover over element
chrome-devtools-axi hover @g1:3

# Drag and drop
chrome-devtools-axi drag @g1:4 @g1:8

# Upload file
chrome-devtools-axi upload @g1:2 /path/to/file.pdf

# Handle browser dialogs
chrome-devtools-axi dialog accept
chrome-devtools-axi dialog dismiss
```

### Screenshots

```bash
# Save screenshot to file
chrome-devtools-axi screenshot output.png

# Capture specific element
chrome-devtools-axi screenshot element.png --uid @g1:5

# Full-page screenshot
chrome-devtools-axi screenshot full.png --full-page

# Specify format
chrome-devtools-axi screenshot output.jpg --format jpeg
```

### JavaScript Evaluation

```bash
# Evaluate expression (auto-wrapped)
chrome-devtools-axi eval "document.title"

# Execute function
chrome-devtools-axi eval "() => document.querySelectorAll('a').length"

# Multi-statement logic
chrome-devtools-axi eval "() => {
  const rows = [...document.querySelectorAll('tr')];
  return rows.map(row => row.textContent.trim());
}"
```

### Multi-Step Scripts

```bash
# Execute script from stdin
cat << 'EOF' | chrome-devtools-axi run
open https://example.com
wait "Example Domain"
click @g1:1
snapshot
EOF
```

## Page Management

```bash
# List all tabs
chrome-devtools-axi pages

# Open new tab
chrome-devtools-axi newpage https://google.com

# Open in background
chrome-devtools-axi newpage https://github.com --background

# Switch tab
chrome-devtools-axi selectpage <id>

# Close tab
chrome-devtools-axi closepage <id>

# Resize viewport
chrome-devtools-axi resize 1920 1080
```

## Device Emulation

```bash
# Emulate mobile viewport
chrome-devtools-axi emulate --viewport "390x844x3,mobile"

# Set color scheme
chrome-devtools-axi emulate --color-scheme dark

# Network throttling
chrome-devtools-axi emulate --network "Slow 3G"

# CPU throttling (1-20)
chrome-devtools-axi emulate --cpu 4

# Set geolocation
chrome-devtools-axi emulate --geolocation "37.7749x-122.4194"

# Custom user agent
chrome-devtools-axi emulate --user-agent "CustomBot/1.0"

# Combine options
chrome-devtools-axi emulate --viewport "390x844x3,mobile" --color-scheme dark --network "Fast 3G"
```

## DevTools Debugging

### Console Messages

```bash
# List all console messages
chrome-devtools-axi console

# Filter by type
chrome-devtools-axi console --type error
chrome-devtools-axi console --type warn

# Pagination
chrome-devtools-axi console --limit 10 --page 2

# Get specific message
chrome-devtools-axi console-get <id>
```

Available types: `log`, `debug`, `info`, `error`, `warn`, `dir`, `dirxml`, `table`, `trace`, `clear`, `assert`, `all`

### Network Requests

```bash
# List network requests
chrome-devtools-axi network

# Filter by type
chrome-devtools-axi network --type xhr
chrome-devtools-axi network --type fetch

# Get specific request
chrome-devtools-axi network-get <id>

# Save response body
chrome-devtools-axi network-get <id> --response-file response.json

# Save request body
chrome-devtools-axi network-get <id> --request-file request.json
```

Available types: `document`, `stylesheet`, `image`, `media`, `font`, `script`, `xhr`, `fetch`, `websocket`, `manifest`, `other`, `all`

## Performance

### Lighthouse Audits

```bash
# Run Lighthouse audit
chrome-devtools-axi lighthouse

# Mobile device
chrome-devtools-axi lighthouse --device mobile

# Snapshot mode (current state)
chrome-devtools-axi lighthouse --mode snapshot

# Save reports to directory
chrome-devtools-axi lighthouse --output-dir ./reports
```

### Performance Tracing

```bash
# Start trace
chrome-devtools-axi perf-start

# Start without reload
chrome-devtools-axi perf-start --no-reload

# Stop and save trace
chrome-devtools-axi perf-stop --file trace.json

# Analyze performance insight
chrome-devtools-axi perf-insight lcp "Largest Contentful Paint"
```

### Heap Snapshots

```bash
# Capture heap snapshot
chrome-devtools-axi heap memory.heapsnapshot
```

## Configuration

### Environment Variables

```bash
# Change bridge server port (default: 9224)
export CHROME_DEVTOOLS_AXI_PORT=9225

# Connect to existing Chrome instance (HTTP/HTTPS)
export CHROME_DEVTOOLS_AXI_BROWSER_URL=http://127.0.0.1:9222

# Connect via WebSocket directly
export CHROME_DEVTOOLS_AXI_BROWSER_URL=wss://cluster.example/launch

# WebSocket authentication headers (JSON)
export CHROME_DEVTOOLS_AXI_WS_HEADERS='{"Authorization":"Bearer token"}'

# Disable session hook auto-install
export CHROME_DEVTOOLS_AXI_DISABLE_HOOKS=1
```

### State Files

State stored in `~/.chrome-devtools-axi/`:
- `bridge.pid` — Running bridge PID and port
- `snapshot-generation` — Staleness detection counter

## Common Patterns

### Login Flow

```bash
# Navigate and fill login form
chrome-devtools-axi open https://app.example.com/login
chrome-devtools-axi fillform @g1:1=user@example.com @g1:2=password
chrome-devtools-axi click @g1:3
chrome-devtools-axi wait "Dashboard"
chrome-devtools-axi snapshot
```

### Form Scraping

```bash
# Extract form data with JavaScript
chrome-devtools-axi eval "() => {
  const form = document.querySelector('form');
  const data = new FormData(form);
  return Object.fromEntries(data.entries());
}"
```

### Multi-Page Workflow

```typescript
// In TypeScript/Node.js
import { execSync } from 'child_process';

function runCommand(cmd: string): string {
  return execSync(`chrome-devtools-axi ${cmd}`, { encoding: 'utf-8' });
}

// Navigate and extract data
runCommand('open https://example.com');
const title = runCommand('eval "document.title"');

// Take screenshot
runCommand('screenshot page.png');

// Fill and submit form
runCommand('fill @g1:1 "search term"');
runCommand('press Enter');
runCommand('wait 2000');

// Extract results
const results = runCommand('eval "() => [...document.querySelectorAll(\'.result\')].map(el => el.textContent)"');
```

### Handling Dynamic Content

```bash
# Wait for element to appear, then interact
chrome-devtools-axi open https://spa.example.com
chrome-devtools-axi wait "Loading complete"
chrome-devtools-axi snapshot
chrome-devtools-axi click @g2:5
```

### Testing Responsive Design

```bash
# Test mobile layout
chrome-devtools-axi emulate --viewport "390x844x3,mobile"
chrome-devtools-axi open https://example.com
chrome-devtools-axi screenshot mobile.png

# Test desktop layout
chrome-devtools-axi resize 1920 1080
chrome-devtools-axi snapshot
chrome-devtools-axi screenshot desktop.png
```

## Understanding Refs and Staleness

Refs use a generation prefix (`g<N>:`) that increments with each snapshot. If you capture a snapshot at generation 1, get ref `@g1:5`, then the page re-renders and moves to generation 2, attempting to use `@g1:5` will fail with `STALE_REF` error.

**Agent pattern:**
1. Capture snapshot
2. Extract refs
3. Attempt action with ref
4. If `STALE_REF` error, re-snapshot and retry with new ref

```bash
# First snapshot
chrome-devtools-axi snapshot
# Output shows: uid=g1:1 link "Click here"

# Page changes, generation bumps to g2
chrome-devtools-axi click @g1:1
# Error: STALE_REF

# Re-snapshot to get fresh refs
chrome-devtools-axi snapshot
# Output shows: uid=g2:3 link "Click here"

chrome-devtools-axi click @g2:3
# Success
```

## Troubleshooting

### Bridge Not Starting

```bash
# Manually stop and restart bridge
chrome-devtools-axi stop
chrome-devtools-axi start
```

### Port Conflicts

```bash
# Use different port
export CHROME_DEVTOOLS_AXI_PORT=9999
chrome-devtools-axi open https://example.com
```

### Stale Refs

Always use refs exactly as printed in snapshot output. If you get `STALE_REF`, capture a new snapshot.

### Element Not Found

If `uid` is valid but element doesn't respond:
- Check if element is visible (scroll into view)
- Wait for page to finish loading
- Use `chrome-devtools-axi wait` before interaction

### Full Output Needed

```bash
# Disable truncation for complete data
chrome-devtools-axi snapshot --full
chrome-devtools-axi console --full
```

### Debugging JavaScript Eval

```bash
# Check for errors in console
chrome-devtools-axi console --type error

# Test simple eval first
chrome-devtools-axi eval "2 + 2"

# Use arrow function for complex logic
chrome-devtools-axi eval "() => { console.log('debug'); return 42; }"
```

## Example: E2E Testing Workflow

```bash
#!/bin/bash

# Start fresh session
chrome-devtools-axi stop
chrome-devtools-axi start

# Navigate to app
chrome-devtools-axi open https://app.example.com

# Login
chrome-devtools-axi fillform @g1:1=$USER_EMAIL @g1:2=$USER_PASSWORD
chrome-devtools-axi click @g1:3
chrome-devtools-axi wait "Dashboard"

# Take baseline screenshot
chrome-devtools-axi screenshot baseline.png

# Navigate to feature
chrome-devtools-axi click @g2:10
chrome-devtools-axi wait "Feature Page"

# Interact with feature
chrome-devtools-axi fill @g3:5 "test data"
chrome-devtools-axi click @g3:6

# Verify result
chrome-devtools-axi wait "Success"
chrome-devtools-axi snapshot --full > result.txt

# Capture performance
chrome-devtools-axi lighthouse --output-dir ./reports

# Cleanup
chrome-devtools-axi stop
```

## Best Practices for Agents

1. **Always re-snapshot after page changes** — Don't reuse refs across navigation or mutations
2. **Use combined operations** — `open` gives snapshot automatically, reducing round-trips
3. **Check help hints** — Every response includes suggested next steps
4. **Use `--full` sparingly** — Default truncation saves tokens; only expand when needed
5. **Prefer `fillform` over multiple `fill`** — Batch form operations when possible
6. **Wait strategically** — Use `wait <text>` instead of arbitrary timeouts when possible
7. **Leverage eval for extraction** — JavaScript eval is more efficient than scraping via snapshots
