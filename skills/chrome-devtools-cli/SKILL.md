---
name: chrome-devtools-cli
description: Chrome DevTools Protocol CLI that auto-connects to existing Chrome for browser automation without heavyweight runtime overhead
triggers:
  - "automate Chrome browser"
  - "control Chrome DevTools"
  - "navigate browser and take screenshot"
  - "interact with web page using Chrome"
  - "click element in Chrome"
  - "run JavaScript in browser"
  - "inspect page accessibility tree"
  - "fill form in Chrome browser"
---

# Chrome DevTools CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Chrome DevTools CLI is a lightweight Rust binary that connects directly to an existing Chrome browser via the DevTools Protocol. It auto-connects to your Chrome instance, requires no separate browser process, and provides a minimal-context alternative to MCP-based browser automation.

## Why Use This

- **Zero context overhead**: One command in, one result out. No MCP layer, no large protocol payloads
- **Uses your Chrome**: Works with your existing browser session, credentials, and extensions
- **Persistent connection**: Background daemon keeps WebSocket open, browser prompts for access only once
- **Deterministic targeting**: Friendly target names (`[target:red-snake]`) for reliable multi-page workflows

## Installation

```bash
# macOS (recommended)
brew install aeroxy/tap/chrome-devtools

# Or via Cargo
cargo install chrome-devtools-cli
```

The binary is named `chrome-devtools`.

## Prerequisites

Enable Chrome remote debugging:

1. Open Chrome
2. Navigate to `chrome://inspect/#remote-debugging`
3. Enable the remote debugging server

## Auto-Connect Behavior

The CLI automatically discovers Chrome's WebSocket endpoint by reading `DevToolsActivePort` from:

- macOS: `~/Library/Application Support/Google/Chrome/`
- Linux: `~/.config/google-chrome/`
- Windows: `%LOCALAPPDATA%\Google\Chrome\User Data\`

Override with environment variables or flags:

```bash
# Environment variables
export CHROME_WS_ENDPOINT=ws://localhost:9222/devtools/browser/abc123
export CHROME_USER_DATA_DIR=~/custom-chrome-profile
export CHROME_CHANNEL=beta  # stable, beta, canary, dev

# Or flags
chrome-devtools --ws-endpoint ws://localhost:9222/... navigate https://example.com
chrome-devtools --user-data-dir ~/custom-profile navigate https://example.com
chrome-devtools --channel canary navigate https://example.com
```

## Daemon Architecture

First command spawns a background daemon that:
- Connects to Chrome WebSocket (one-time approval prompt)
- Listens on Unix socket: `/tmp/chrome-devtools-daemon.sock`
- Auto-exits after 5 minutes of inactivity
- All subsequent commands reuse the persistent connection

Kill manually: `pkill -f __daemon__` or delete the socket file.

## Page Targeting Pattern

**CRITICAL**: Always capture and reuse the `[target:name]` from navigation commands.

```bash
# Step 1: Navigate and capture target name
chrome-devtools navigate https://github.com
# Output: Navigated to https://github.com
# [target:green-dog]

# Step 2: Pin all subsequent commands to this target
chrome-devtools --target green-dog screenshot --output /tmp/gh.png
chrome-devtools --target green-dog evaluate "document.title"
chrome-devtools --target green-dog click "a[href='/login']"
```

Without `--target`, commands default to page index 0, which may change as Chrome reorders tabs.

### List All Pages

```bash
chrome-devtools list-pages
# Output:
# [0] (green-dog) GitHub — https://github.com
# [1] (red-snake) Example Domain — https://example.com
# [2] (bold-stag) Local Dev — https://localhost:3000
```

You can also use `--page <index>` for quick one-offs:

```bash
chrome-devtools --page 1 evaluate "window.location.href"
```

## Core Commands

### Navigation

```bash
# Navigate to URL (waits for load event)
chrome-devtools navigate https://example.com

# History navigation
chrome-devtools --target red-snake navigate --back
chrome-devtools --target red-snake navigate --forward
chrome-devtools --target red-snake navigate --reload

# Tab management
chrome-devtools new-page https://github.com
chrome-devtools close-page 2
chrome-devtools select-page 0  # Bring tab to front
```

### Inspection

```bash
# Screenshot (viewport only)
chrome-devtools --target green-dog screenshot --output /tmp/page.png

# Full-page screenshot (scrollable area)
chrome-devtools --target green-dog screenshot --full-page --output /tmp/full.png

# JavaScript evaluation
chrome-devtools --target green-dog evaluate "document.title"
chrome-devtools --target green-dog evaluate "Array.from(document.querySelectorAll('h1')).map(h => h.textContent)"

# Handle JavaScript dialogs (alert/confirm/prompt)
chrome-devtools --target green-dog evaluate "confirm('Proceed?')" --dialog-action accept
chrome-devtools --target green-dog evaluate "prompt('Name?')" --dialog-action "John Doe"

# Accessibility tree snapshot
chrome-devtools --target green-dog snapshot
```

### Interaction

```bash
# Click by CSS selector
chrome-devtools --target green-dog click "#submit-button"
chrome-devtools --target green-dog click "a[href='/login']"

# Click at coordinates
chrome-devtools --target green-dog click-at 100 200

# Fill input fields
chrome-devtools --target green-dog fill "#email" "user@example.com"
chrome-devtools --target green-dog fill "#password" "secret123"

# Fill dropdown (select element)
chrome-devtools --target green-dog fill "#country" "United States"

# Toggle checkbox/radio
chrome-devtools --target green-dog fill "#terms" "true"
chrome-devtools --target green-dog fill "#newsletter" "false"

# Type text into focused element
chrome-devtools --target green-dog type-text "Hello World"
chrome-devtools --target green-dog type-text "Search query" --submit-key Enter

# Press keys
chrome-devtools --target green-dog press-key Enter
chrome-devtools --target green-dog press-key "Control+A"
chrome-devtools --target green-dog press-key Escape

# Hover over element
chrome-devtools --target green-dog hover ".dropdown-menu"
```

### Viewport & Waiting

```bash
# Resize viewport
chrome-devtools --target green-dog resize 1920 1080

# Wait for text to appear (default 30s timeout)
chrome-devtools --target green-dog wait-for "Login successful"
chrome-devtools --target green-dog wait-for "Dashboard" --timeout 60000
```

### Third-Party Developer Tools

For pages that inject custom DevTools via `window.__dtmcp`:

```bash
# List available custom tools
chrome-devtools --target green-dog list-3p-tools

# Execute custom tool
chrome-devtools --target green-dog execute-3p-tool "myTool" '{"param": "value"}'
```

## JSON Output Mode

Add `--json` to any command for machine-readable output:

```bash
chrome-devtools --json --target green-dog evaluate "document.title"
# {"success": true, "result": "Example Domain"}

chrome-devtools --json list-pages
# {"success": true, "pages": [{"index": 0, "friendlyName": "green-dog", ...}]}
```

## Real-World Workflows

### Automated Form Submission

```bash
#!/bin/bash
set -e

# Navigate and capture target
TARGET=$(chrome-devtools navigate https://example.com/form | grep -oP '(?<=target:)[a-z-]+')

# Fill form fields
chrome-devtools --target "$TARGET" fill "#name" "Alice Smith"
chrome-devtools --target "$TARGET" fill "#email" "alice@example.com"
chrome-devtools --target "$TARGET" fill "#country" "Canada"
chrome-devtools --target "$TARGET" fill "#terms" "true"

# Submit and wait for confirmation
chrome-devtools --target "$TARGET" click "#submit"
chrome-devtools --target "$TARGET" wait-for "Thank you" --timeout 10000

# Capture confirmation screenshot
chrome-devtools --target "$TARGET" screenshot --output /tmp/confirmation.png
```

### Data Extraction

```bash
#!/bin/bash
TARGET=$(chrome-devtools navigate https://news.ycombinator.com | grep -oP '(?<=target:)[a-z-]+')

# Extract top stories
chrome-devtools --target "$TARGET" evaluate "
  Array.from(document.querySelectorAll('.athing')).slice(0, 5).map(item => ({
    title: item.querySelector('.titleline > a').textContent,
    url: item.querySelector('.titleline > a').href
  }))
" --json > /tmp/hn-stories.json

cat /tmp/hn-stories.json
```

### Multi-Page Session

```bash
#!/bin/bash

# Open multiple pages
TARGET_HOME=$(chrome-devtools navigate https://github.com | grep -oP '(?<=target:)[a-z-]+')
TARGET_REPO=$(chrome-devtools new-page https://github.com/torvalds/linux | grep -oP '(?<=target:)[a-z-]+')

# Interact with different pages
chrome-devtools --target "$TARGET_HOME" click "a[href='/login']"
chrome-devtools --target "$TARGET_REPO" evaluate "document.querySelector('.repository-content').textContent.includes('Linux kernel')"

# Screenshot both
chrome-devtools --target "$TARGET_HOME" screenshot --output /tmp/home.png
chrome-devtools --target "$TARGET_REPO" screenshot --output /tmp/repo.png
```

### Accessibility Audit

```bash
#!/bin/bash
TARGET=$(chrome-devtools navigate https://example.com | grep -oP '(?<=target:)[a-z-]+')

# Get full accessibility tree
chrome-devtools --target "$TARGET" snapshot > /tmp/a11y-tree.txt

# Check for specific roles
chrome-devtools --target "$TARGET" evaluate "
  document.querySelectorAll('[role=\"button\"]').length
"

# Find missing alt text
chrome-devtools --target "$TARGET" evaluate "
  Array.from(document.querySelectorAll('img:not([alt])')).map(img => img.src)
"
```

## Common Patterns for AI Agents

### 1. Always Capture Target Names

```bash
# ❌ BAD: Target may change
chrome-devtools navigate https://example.com
chrome-devtools screenshot --output /tmp/page.png  # Might screenshot wrong page!

# ✅ GOOD: Explicit target
TARGET=$(chrome-devtools navigate https://example.com | grep -oP '(?<=target:)[a-z-]+')
chrome-devtools --target "$TARGET" screenshot --output /tmp/page.png
```

### 2. Understand the Page First

```bash
# Get context before interacting
chrome-devtools --target "$TARGET" snapshot  # See accessible structure
chrome-devtools --target "$TARGET" screenshot --output /tmp/context.png
chrome-devtools --target "$TARGET" evaluate "document.body.innerText"  # Read visible text
```

### 3. Wait for Dynamic Content

```bash
# Don't assume instant rendering
chrome-devtools --target "$TARGET" click "#load-more"
chrome-devtools --target "$TARGET" wait-for "Showing 20 results" --timeout 5000
chrome-devtools --target "$TARGET" evaluate "document.querySelectorAll('.result-item').length"
```

### 4. Handle Errors Gracefully

```bash
# Check page state before interaction
HAS_BUTTON=$(chrome-devtools --target "$TARGET" evaluate "!!document.querySelector('#submit')" --json | jq -r '.result')

if [ "$HAS_BUTTON" = "true" ]; then
  chrome-devtools --target "$TARGET" click "#submit"
else
  echo "Submit button not found"
fi
```

## Troubleshooting

### "Could not connect to Chrome"

- Ensure Chrome is running
- Verify remote debugging is enabled at `chrome://inspect/#remote-debugging`
- Check `DevToolsActivePort` exists in user data directory
- Try explicit WebSocket: `chrome-devtools --ws-endpoint ws://localhost:9222/... navigate https://example.com`

### "Daemon connection failed"

```bash
# Kill stale daemon
pkill -f __daemon__
rm /tmp/chrome-devtools-daemon.sock

# Retry command (will spawn fresh daemon)
chrome-devtools navigate https://example.com
```

### "Element not found" on click/fill

```bash
# Verify element exists
chrome-devtools --target "$TARGET" evaluate "!!document.querySelector('#my-button')"

# Get all matching elements
chrome-devtools --target "$TARGET" evaluate "document.querySelectorAll('#my-button').length"

# Wait for element to appear
chrome-devtools --target "$TARGET" wait-for "Submit" --timeout 10000
chrome-devtools --target "$TARGET" click "#submit"
```

### Target name confusion

```bash
# List all pages with friendly names
chrome-devtools list-pages

# Use explicit page index if needed
chrome-devtools --page 0 evaluate "document.title"

# Or use raw target ID (from list-pages output)
chrome-devtools --target "E4F5A6B7C8D9" evaluate "document.title"
```

## Best Practices for AI Agents

1. **Always use `--target`** after initial navigation
2. **Capture screenshots** before and after interactions for debugging
3. **Use `snapshot`** to understand page structure before clicking
4. **Wait for content** instead of assuming instant load
5. **Verify elements exist** before interaction
6. **Use JSON output** (`--json`) for programmatic parsing
7. **Check `list-pages`** when multi-page workflows get confusing
8. **Kill daemon** between unrelated sessions to avoid stale connections

## Integration with AI Coding Agents

When helping a user automate Chrome:

1. Install `chrome-devtools-cli` if not present
2. Verify Chrome remote debugging is enabled
3. Start with `navigate` and capture the target name
4. Use `snapshot` or `screenshot` to understand the page
5. Perform interactions with explicit `--target`
6. Extract results with `evaluate` and `--json`
7. Clean up with `close-page` if needed

Example agent workflow:

```bash
# User: "Login to example.com with my credentials"

# 1. Navigate
TARGET=$(chrome-devtools navigate https://example.com/login | grep -oP '(?<=target:)[a-z-]+')

# 2. Understand page
chrome-devtools --target "$TARGET" snapshot | head -50

# 3. Fill credentials (from env vars)
chrome-devtools --target "$TARGET" fill "#username" "$EXAMPLE_USERNAME"
chrome-devtools --target "$TARGET" fill "#password" "$EXAMPLE_PASSWORD"

# 4. Submit
chrome-devtools --target "$TARGET" click "#login-button"

# 5. Verify success
chrome-devtools --target "$TARGET" wait-for "Dashboard" --timeout 10000
chrome-devtools --target "$TARGET" screenshot --output /tmp/logged-in.png
```

This skill enables zero-context browser automation directly from your existing Chrome session.
