---
name: tabbit-browser-devtools-skill
description: Connect AI agents to a running Tabbit browser instance via Chrome DevTools Protocol (CDP) and control it through agent-browser
triggers:
  - connect to tabbit browser
  - use tabbit devtools endpoint
  - control tabbit browser with CDP
  - find tabbit websocket endpoint
  - automate tabbit browser
  - attach to running tabbit instance
  - launch agent-browser with tabbit
  - connect devtools to tabbit
---

# Tabbit Browser DevTools Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Tabbit Browser DevTools Skill enables AI agents to discover and connect to a running Tabbit browser instance through its Chrome DevTools Protocol (CDP) endpoint. Tabbit is a Chromium-based browser, and this skill acts as a bridge that:

1. Locates Tabbit's `DevToolsActivePort` file on the filesystem
2. Extracts the WebSocket endpoint URL
3. Passes that endpoint to `agent-browser` for actual automation
4. Allows the agent to perform page operations (navigate, click, extract, execute scripts)

This skill does **not** implement browser automation itself—it delegates to `agent-browser` while providing the Tabbit-specific connection details.

## Prerequisites

Before using this skill, ensure:

- **Python 3** is installed
- **Node.js/npx** is available, or `agent-browser` is installed globally
- **Tabbit browser** is installed and running
- **Remote debugging is enabled** in Tabbit: navigate to `tabbit://inspect/#remote-debugging` and enable it

## Installation

### Option 1: Using Skills CLI (Recommended)

```bash
npx skills add Tabbit-Browser/Tabbit-Devtools-Skill
```

Install only the tabbit-devtools skill:

```bash
npx skills add Tabbit-Browser/Tabbit-Devtools-Skill --skill tabbit-devtools
```

### Option 2: Install in Codex

In a Codex conversation:

```text
$skill-installer install https://github.com/Tabbit-Browser/Tabbit-Devtools-Skill/tree/main/skills/tabbit-devtools
```

Or from terminal:

```bash
mkdir -p ~/.agents/skills
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --url https://github.com/Tabbit-Browser/Tabbit-Devtools-Skill/tree/main/skills/tabbit-devtools \
  --dest ~/.agents/skills
```

### Option 3: Manual Installation

```bash
mkdir -p ~/.agents/skills
ln -sfn /path/to/Tabbit-Devtools-Skill/skills/tabbit-devtools ~/.agents/skills/tabbit-devtools
```

**Always restart your agent after installation.**

## Installing agent-browser

This skill depends on `agent-browser` for actual browser operations. Install it:

```bash
npx skills add vercel-labs/agent-browser
```

Or ensure `agent-browser` is available via npm:

```bash
npm install -g agent-browser
```

The skill will attempt to run `agent-browser` or `npx agent-browser` by default.

## How It Works

### Discovery Process

The skill searches for `DevToolsActivePort` in these locations (in order):

**macOS:**
- `~/Library/Application Support/Tabbit/DevToolsActivePort`
- `~/Library/Application Support/Tabbit Browser/DevToolsActivePort`

**Windows:**
- `%LOCALAPPDATA%\Tabbit Browser\User Data\DevToolsActivePort`
- `%APPDATA%\Tabbit\User Data\DevToolsActivePort`

The `DevToolsActivePort` file contains two lines:
```
<port>
<browser-target-id>
```

### Extracting the WebSocket Endpoint

```python
import os
import json
import urllib.request

# Example: reading DevToolsActivePort
port_file = os.path.expanduser("~/Library/Application Support/Tabbit/DevToolsActivePort")

with open(port_file, 'r') as f:
    port = f.readline().strip()

# Query the DevTools JSON endpoint
url = f"http://127.0.0.1:{port}/json/version"
response = urllib.request.urlopen(url)
data = json.loads(response.read())

ws_endpoint = data['webSocketDebuggerUrl']
print(f"WebSocket endpoint: {ws_endpoint}")
```

### Connecting with agent-browser

Once the WebSocket endpoint is discovered, the skill invokes:

```bash
agent-browser --cdp ws://127.0.0.1:9222/devtools/browser/<id>
```

From that point, all browser operations go through `agent-browser`.

## Configuration

### Custom agent-browser Binary

If your environment uses a different command to launch `agent-browser`, set:

```bash
export AGENT_BROWSER_BIN="npx --yes agent-browser"
```

Or for a custom path:

```bash
export AGENT_BROWSER_BIN="/usr/local/bin/my-agent-browser"
```

The skill will use this variable when spawning `agent-browser`.

## Common Patterns

### Pattern 1: Discover and Connect

```python
#!/usr/bin/env python3
import os
import json
import urllib.request
import subprocess

# Search for DevToolsActivePort
possible_paths = [
    os.path.expanduser("~/Library/Application Support/Tabbit/DevToolsActivePort"),
    os.path.expanduser("~/Library/Application Support/Tabbit Browser/DevToolsActivePort"),
]

port_file = None
for path in possible_paths:
    if os.path.exists(path):
        port_file = path
        break

if not port_file:
    print("Tabbit is not running or remote debugging is not enabled")
    exit(1)

with open(port_file, 'r') as f:
    port = f.readline().strip()

# Get WebSocket endpoint
url = f"http://127.0.0.1:{port}/json/version"
response = urllib.request.urlopen(url)
version_data = json.loads(response.read())
ws_endpoint = version_data['webSocketDebuggerUrl']

print(f"Found Tabbit at {ws_endpoint}")

# Launch agent-browser with CDP endpoint
agent_browser_bin = os.getenv("AGENT_BROWSER_BIN", "agent-browser")
subprocess.run([agent_browser_bin, "--cdp", ws_endpoint])
```

### Pattern 2: Open a Page

Once connected via agent-browser, the agent can issue commands like:

```bash
agent-browser --cdp <ws_endpoint> open https://example.com
```

Or in Python using subprocess:

```python
import subprocess

ws_endpoint = "ws://127.0.0.1:9222/devtools/browser/abc123"
agent_browser = os.getenv("AGENT_BROWSER_BIN", "agent-browser")

subprocess.run([
    agent_browser,
    "--cdp", ws_endpoint,
    "open", "https://example.com"
])
```

### Pattern 3: Click and Extract

```bash
# Click a button
agent-browser --cdp <ws_endpoint> click "button[type='submit']"

# Extract text
agent-browser --cdp <ws_endpoint> extract "h1"
```

### Pattern 4: Execute JavaScript

```bash
agent-browser --cdp <ws_endpoint> execute "document.title"
```

In Python:

```python
result = subprocess.run([
    agent_browser,
    "--cdp", ws_endpoint,
    "execute", "document.querySelector('h1').innerText"
], capture_output=True, text=True)

print(result.stdout)
```

## Troubleshooting

### Issue: "DevToolsActivePort not found"

**Cause:** Tabbit is not running, or remote debugging is disabled.

**Solution:**
1. Launch Tabbit browser
2. Navigate to `tabbit://inspect/#remote-debugging`
3. Enable remote debugging
4. Restart Tabbit if necessary

### Issue: "Connection refused" when accessing DevTools endpoint

**Cause:** The port in `DevToolsActivePort` may be stale, or a firewall is blocking localhost connections.

**Solution:**
- Restart Tabbit to regenerate the port
- Check firewall settings for localhost access
- Verify the port is correct:

```bash
PORT=$(head -n1 ~/Library/Application\ Support/Tabbit/DevToolsActivePort)
curl http://127.0.0.1:$PORT/json/version
```

### Issue: "agent-browser: command not found"

**Cause:** `agent-browser` is not installed or not in PATH.

**Solution:**
```bash
npm install -g agent-browser
```

Or use npx:

```bash
export AGENT_BROWSER_BIN="npx --yes agent-browser"
```

### Issue: Multiple Tabbit Instances Running

**Cause:** Multiple Tabbit profiles or instances may create conflicting `DevToolsActivePort` files.

**Solution:**
- Close all Tabbit instances except one
- Restart Tabbit with a single profile
- If using multiple profiles, identify the correct user data directory

### Issue: Skill finds endpoint but agent-browser fails

**Cause:** The WebSocket endpoint might be expired or the browser closed the debugging session.

**Solution:**
- Restart Tabbit
- Re-enable remote debugging
- Check that no other tool is already connected to the same endpoint

## Real-World Workflow

```python
#!/usr/bin/env python3
"""
Workflow: Connect to Tabbit, navigate to a page, extract data
"""
import os
import json
import urllib.request
import subprocess
import sys

def find_devtools_port():
    """Locate Tabbit's DevToolsActivePort file."""
    paths = [
        os.path.expanduser("~/Library/Application Support/Tabbit/DevToolsActivePort"),
        os.path.expanduser("~/Library/Application Support/Tabbit Browser/DevToolsActivePort"),
    ]
    
    for path in paths:
        if os.path.exists(path):
            with open(path, 'r') as f:
                return f.readline().strip()
    
    return None

def get_ws_endpoint(port):
    """Query DevTools JSON endpoint for WebSocket URL."""
    url = f"http://127.0.0.1:{port}/json/version"
    try:
        response = urllib.request.urlopen(url)
        data = json.loads(response.read())
        return data['webSocketDebuggerUrl']
    except Exception as e:
        print(f"Failed to get WebSocket endpoint: {e}")
        return None

def run_agent_browser(ws_endpoint, *args):
    """Execute agent-browser command with CDP endpoint."""
    agent_browser = os.getenv("AGENT_BROWSER_BIN", "agent-browser")
    cmd = [agent_browser, "--cdp", ws_endpoint] + list(args)
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def main():
    # Step 1: Find Tabbit
    port = find_devtools_port()
    if not port:
        print("Error: Tabbit is not running or debugging is disabled")
        sys.exit(1)
    
    print(f"Found Tabbit on port {port}")
    
    # Step 2: Get WebSocket endpoint
    ws_endpoint = get_ws_endpoint(port)
    if not ws_endpoint:
        sys.exit(1)
    
    print(f"WebSocket endpoint: {ws_endpoint}")
    
    # Step 3: Navigate to a page
    print("Opening example.com...")
    run_agent_browser(ws_endpoint, "open", "https://example.com")
    
    # Step 4: Extract page title
    print("Extracting page title...")
    title = run_agent_browser(ws_endpoint, "execute", "document.title")
    print(f"Page title: {title.strip()}")
    
    # Step 5: Extract all links
    print("Extracting links...")
    links = run_agent_browser(ws_endpoint, "extract", "a")
    print(f"Links found:\n{links}")

if __name__ == "__main__":
    main()
```

## Agent-Browser Commands

Once connected via `--cdp`, you have access to all `agent-browser` commands:

| Command | Description | Example |
|---------|-------------|---------|
| `open <url>` | Navigate to URL | `open https://example.com` |
| `click <selector>` | Click element | `click button.submit` |
| `type <selector> <text>` | Type into input | `type input[name=q] "search term"` |
| `extract <selector>` | Extract text/HTML | `extract h1` |
| `execute <js>` | Run JavaScript | `execute document.title` |
| `screenshot` | Capture screenshot | `screenshot` |
| `wait <selector>` | Wait for element | `wait div.loaded` |

For full documentation, see: https://github.com/vercel-labs/agent-browser

## Summary

This skill provides the glue between Tabbit's CDP endpoint and `agent-browser`. As an agent, when a user asks to "connect to Tabbit" or "automate Tabbit browser":

1. Use the Python discovery logic to find `DevToolsActivePort`
2. Extract the WebSocket endpoint
3. Invoke `agent-browser --cdp <ws_endpoint>` with appropriate commands
4. All page operations are handled by `agent-browser`

The skill itself is a **connector**, not a full browser automation framework. It solves the problem of finding and connecting to a running Tabbit instance, then delegates the heavy lifting to the official `agent-browser` tool.
