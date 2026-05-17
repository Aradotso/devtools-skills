---
name: agent-browser-cli-control
description: Control and interact with Chrome browser via agent-browser-cli for tab management, page automation, CDP operations, and content extraction.
triggers:
  - interact with the browser
  - control chrome tabs
  - execute javascript in browser
  - get page content from browser
  - take browser screenshot
  - manage browser cookies
  - automate browser actions
  - extract content from web pages
---

# agent-browser-cli Control

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

agent-browser-cli is a browser control CLI built in Rust that connects to a real Chrome session via a Chrome extension. It enables AI agents to scan tabs, execute JavaScript, manage cookies, capture screenshots, and perform CDP operations while preserving the user's login state and session.

**Key difference from Selenium/Playwright**: This tool works with an existing Chrome session (not headless), preserving all user login states and cookies. Perfect for AI agents that need to work with authenticated web sessions.

## Installation

### Via npm (recommended)

```bash
npm install -g @sleepinsummer/agent-browser-cli
```

### Via source

```bash
git clone https://github.com/sleepinginsummer/agent-browser-cli.git
cd agent-browser-cli
cargo build --release
# Binary will be at ./target/release/agent-browser-cli
```

## Chrome Extension Setup

**Critical**: The Chrome extension must be loaded for the CLI to work.

1. Download `chrome-extensions.zip` from the [latest release](https://github.com/sleepinginsummer/agent-browser-cli/releases/latest)
2. Extract the zip file
3. Open Chrome and navigate to `chrome://extensions`
4. Enable "Developer mode"
5. Click "Load unpacked extension"
6. Select the extracted `tmwd_cdp_bridge` directory
7. Ensure at least one normal web page tab is open (not `about:blank` or `chrome://`)

The extension will show a small indicator on the right side of the page when connected.

## Architecture

agent-browser-cli runs as a daemon service that:
- Listens on port `18765` for Chrome extension WebSocket connections
- Listens on port `18767` for CLI HTTP API calls
- Maintains persistent browser connection to avoid re-initialization overhead

Performance reference (with daemon running):
- Simple page read / JS execution: 40-120ms
- DOM query operations: 270-360ms
- Page monitoring with change summary: 720-880ms

## Core Commands

### Daemon Management

```bash
# Start daemon (auto-starts if not running)
agent-browser-cli start

# Stop daemon
agent-browser-cli stop

# Restart daemon
agent-browser-cli restart

# Check daemon status
agent-browser-cli status
```

### Tab Management

```bash
# List all tabs
agent-browser-cli tabs

# Switch to tab by index
agent-browser-cli switch 0

# Open new URL
agent-browser-cli open https://example.com

# Close current tab
agent-browser-cli close

# Get active tab info
agent-browser-cli active
```

### Page Content Extraction

```bash
# Scan current page (simplified HTML)
agent-browser-cli scan

# Get text-only content (faster)
agent-browser-cli scan --text-only

# Get full page DOM
agent-browser-cli scan --full

# Scan specific tab
agent-browser-cli scan --tab 0

# Get page with screenshots of visible elements
agent-browser-cli scan --screenshot
```

### JavaScript Execution

```bash
# Execute JavaScript and return result
agent-browser-cli exec 'return document.title'

# Execute with page monitoring (detects changes)
agent-browser-cli exec --monitor 'document.querySelector("#search").value = "test"'

# Multi-line JS execution
agent-browser-cli exec 'const btn = document.querySelector("button");
btn.click();
return "clicked";'

# Execute in specific tab
agent-browser-cli exec --tab 0 'return window.location.href'
```

### Screenshots

```bash
# Capture full page screenshot
agent-browser-cli screenshot

# Screenshot specific tab
agent-browser-cli screenshot --tab 0

# Output to specific file
agent-browser-cli screenshot --output /path/to/screenshot.png
```

### Cookie Management

```bash
# Get all cookies for current page
agent-browser-cli cookies

# Get cookies for specific domain
agent-browser-cli cookies --domain example.com

# Set cookie
agent-browser-cli cookies --set 'name=value; domain=.example.com; path=/'
```

### Configuration

```bash
# Change extension WebSocket port (default: 18765)
agent-browser-cli set-extension-port 18766

# View current config
cat ~/.agent-browser-cli/config.json
```

## Real-World Examples

### Example 1: Search Automation

```bash
# Open search engine
agent-browser-cli open "https://www.google.com"

# Wait a moment for page load, then input search query
sleep 1
agent-browser-cli exec 'document.querySelector("textarea[name=q]").value = "rust programming"'

# Submit search
agent-browser-cli exec --monitor 'document.querySelector("textarea[name=q]").form.submit()'

# Extract search results
agent-browser-cli scan --text-only
```

### Example 2: Form Filling

```bash
# Navigate to form page
agent-browser-cli open "https://example.com/contact"

# Fill form fields
agent-browser-cli exec '
const form = {
  name: document.querySelector("#name"),
  email: document.querySelector("#email"),
  message: document.querySelector("#message")
};
form.name.value = "AI Agent";
form.email.value = "agent@example.com";
form.message.value = "Hello from agent-browser-cli";
return "Form filled";
'

# Submit and monitor changes
agent-browser-cli exec --monitor 'document.querySelector("form").submit()'
```

### Example 3: Multi-Tab Workflow

```bash
# Get list of all tabs
TABS=$(agent-browser-cli tabs)
echo "$TABS"

# Switch to first tab
agent-browser-cli switch 0

# Get current page title
agent-browser-cli exec 'return document.title'

# Open new tab
agent-browser-cli open "https://example.com"

# Work with new tab
agent-browser-cli scan --text-only
```

### Example 4: Data Extraction with Cookies

```bash
# Navigate to authenticated page
agent-browser-cli open "https://example.com/dashboard"

# Extract cookies (user is already logged in)
COOKIES=$(agent-browser-cli cookies --domain example.com)
echo "$COOKIES"

# Extract protected content
agent-browser-cli scan --text-only

# Take screenshot of dashboard
agent-browser-cli screenshot --output dashboard.png
```

### Example 5: Page Monitoring

```bash
# Execute action and monitor DOM changes
RESULT=$(agent-browser-cli exec --monitor '
document.querySelector("#load-more").click();
return "Clicked load more";
')

# The result includes a change summary
echo "$RESULT"

# Re-scan to get updated content
agent-browser-cli scan --text-only
```

## Response Format

All commands return JSON:

```json
{
  "ok": true,
  "result": {
    "status": "success",
    "data": "...",
    "metadata": {}
  }
}
```

Error format:

```json
{
  "ok": false,
  "error": "Error message"
}
```

## Common Patterns

### Wait for Element

```bash
agent-browser-cli exec '
return new Promise((resolve) => {
  const check = () => {
    const el = document.querySelector("#target");
    if (el) resolve("Found");
    else setTimeout(check, 100);
  };
  check();
});
'
```

### Extract Structured Data

```bash
agent-browser-cli exec '
return Array.from(document.querySelectorAll(".item")).map(item => ({
  title: item.querySelector(".title")?.textContent,
  link: item.querySelector("a")?.href,
  price: item.querySelector(".price")?.textContent
}));
'
```

### File Upload

```bash
agent-browser-cli exec '
const input = document.querySelector("input[type=file]");
const dt = new DataTransfer();
dt.items.add(new File(["content"], "test.txt"));
input.files = dt.files;
input.dispatchEvent(new Event("change", { bubbles: true }));
return "File set";
'
```

### Dropdown Selection

```bash
agent-browser-cli exec '
const select = document.querySelector("select#country");
select.value = "US";
select.dispatchEvent(new Event("change", { bubbles: true }));
return select.value;
'
```

## Troubleshooting

### Extension not connecting

1. Verify extension is loaded in `chrome://extensions`
2. Check extension port matches config: `cat ~/.agent-browser-cli/config.json`
3. Ensure at least one normal web page is open (not `chrome://` or `about:`)
4. Restart daemon: `agent-browser-cli restart`

### Command timeout

1. Check daemon status: `agent-browser-cli status`
2. Verify Chrome is running and extension is active
3. Try simpler command first: `agent-browser-cli tabs`
4. Check logs: `tail -f ~/.agent-browser-cli.log`

### Port conflicts

```bash
# Change extension port
agent-browser-cli set-extension-port 18766

# Update extension popup to match new port
# Restart daemon
agent-browser-cli restart
```

### WSL 2 connectivity issues

For WSL 2 users on Windows 11 22H2+:

1. Enable `networkingMode=mirrored` in `.wslconfig`
2. Restart WSL
3. Ensure Chrome extension can connect to `localhost:18765`

### Slow performance

- Use `--text-only` flag for faster content extraction
- Avoid `--screenshot` unless needed
- Use `--monitor` only when detecting page changes is necessary
- Consider increasing system resources if Chrome is sluggish

## Advanced: CDP Operations

The underlying Chrome extension uses Chrome DevTools Protocol (CDP). While most operations are abstracted by the CLI, you can execute raw CDP commands via JavaScript:

```bash
agent-browser-cli exec '
return await chrome.debugger.sendCommand({tabId: chrome.tabs.getCurrent().id}, "Network.getCookies", {});
'
```

## Environment Variables

The CLI uses a config file instead of environment variables:

**Config location**: `~/.agent-browser-cli/config.json`

```json
{
  "extension_port": 18765
}
```

## Logs

Daemon logs are written to:

```
~/.agent-browser-cli.log
```

Monitor in real-time:

```bash
tail -f ~/.agent-browser-cli.log
```

## Integration with AI Agents

When called by AI coding agents, typical workflows:

1. **Check tab state**: `agent-browser-cli tabs`
2. **Navigate or switch**: `agent-browser-cli open <url>` or `agent-browser-cli switch <index>`
3. **Extract content**: `agent-browser-cli scan --text-only`
4. **Perform action**: `agent-browser-cli exec --monitor '<js code>'`
5. **Verify result**: `agent-browser-cli scan` or `agent-browser-cli screenshot`

The tool is designed for high-frequency calls with minimal overhead (~40-120ms for simple operations).
