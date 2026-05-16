---
name: opencli-universal-cli-hub
description: Convert websites, Electron apps, and local tools into standardized CLIs for AI agents and humans using OpenCLI's browser automation and adapter framework.
triggers:
  - "help me use OpenCLI to automate this website"
  - "create a CLI adapter for this site"
  - "drive my browser with OpenCLI commands"
  - "convert this website into a command-line tool"
  - "write an OpenCLI adapter for scraping data"
  - "automate browser actions with OpenCLI"
  - "register a local CLI tool with OpenCLI"
  - "control my Chrome browser from the terminal"
---

# OpenCLI Universal CLI Hub

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OpenCLI transforms any website, Electron app, or local binary into a standardized command-line interface. It provides AI agents with the ability to operate websites through a logged-in browser, execute built-in adapters for 100+ sites, and create new adapters using browser automation primitives.

## Installation

### Prerequisites

- **Node.js >= 21.0.0**
- **Chrome or Chromium** browser
- Logged-in browser session for target sites

### Install OpenCLI CLI

```bash
# Check Node.js version
node --version

# Install globally
npm install -g @jackwener/opencli

# Verify installation
opencli doctor
```

### Install Browser Bridge Extension

**Chrome Web Store (recommended):**
Install from [Chrome Web Store](https://chromewebstore.google.com/detail/opencli/ildkmabpimmkaediidaifkhjpohdnifk)

**Manual Installation:**
1. Download `opencli-extension-v{version}.zip` from [GitHub Releases](https://github.com/jackwener/opencli/releases)
2. Unzip and navigate to `chrome://extensions`
3. Enable **Developer mode**
4. Click **Load unpacked** and select the folder

### Verify Setup

```bash
opencli doctor
```

## Core Concepts

### Three Automation Modes

1. **Built-in Adapters**: Pre-built commands for popular sites (Bilibili, Twitter, Reddit, etc.)
2. **Browser Automation**: AI agents operate any website via `opencli browser` primitives
3. **CLI Hub**: Expose local tools and Electron apps through unified interface

### Browser Sessions

Browser commands require an explicit `<session>` identifier:

```bash
# Session names are arbitrary but must be consistent
opencli browser work open https://example.com
opencli browser work state
opencli browser work click "button.submit"
opencli browser work close
```

### Multi-Profile Support

```bash
# List connected Chrome profiles
opencli profile list

# Rename profile for easier reference
opencli profile rename <contextId> work

# Set default profile
opencli profile use work

# Use specific profile for command
opencli --profile work browser mysession open https://example.com
```

## Using Built-in Adapters

### List Available Commands

```bash
# Show all registered commands
opencli list

# Show commands for specific site
opencli list bilibili
```

### Common Built-in Commands

```bash
# HackerNews top stories
opencli hackernews top --limit 5

# Bilibili trending videos
opencli bilibili hot --limit 10

# Reddit hot posts
opencli reddit hot --subreddit programming --limit 20

# Twitter/X trending
opencli twitter trending

# Xiaohongshu (Little Red Book)
opencli xiaohongshu hot --limit 15

# Zhihu hot questions
opencli zhihu hot --limit 10
```

## Browser Automation Primitives

### Navigation Commands

```bash
# Open URL in new session
opencli browser mysession open https://github.com/trending

# Get current page state (URL, title, DOM snapshot)
opencli browser mysession state

# Navigate back
opencli browser mysession back

# Close session
opencli browser mysession close
```

### Tab Management

```bash
# List all tabs in session
opencli browser mysession tab list

# Create new tab (returns targetId)
opencli browser mysession tab new https://example.com

# Select specific tab as default target
opencli browser mysession tab select <targetId>

# Close tab
opencli browser mysession tab close <targetId>

# Execute command on specific tab
opencli browser mysession state --tab <targetId>
```

### Interaction Commands

```bash
# Click element by selector
opencli browser mysession click "button.login"

# Type text into element
opencli browser mysession type "input[name='username']" "myuser"

# Fill form field (clears first, then types)
opencli browser mysession fill "input[name='password']" "mypass"

# Select dropdown option
opencli browser mysession select "select#country" "US"

# Send keyboard keys
opencli browser mysession keys "Enter"
opencli browser mysession keys "Control+a"
```

### Waiting and Extraction

```bash
# Wait for element to appear
opencli browser mysession wait "div.content"

# Wait for specific text
opencli browser mysession wait --text "Loading complete"

# Extract data from page
opencli browser mysession extract "h1.title"

# Get element properties
opencli browser mysession get "a.link" --attribute href

# Find elements
opencli browser mysession find "article"
```

### Advanced Commands

```bash
# Take screenshot
opencli browser mysession screenshot output.png

# Scroll page
opencli browser mysession scroll --distance 500

# Execute JavaScript
opencli browser mysession eval "document.title"

# Monitor network requests
opencli browser mysession network --pattern "api/v1/*"

# List frames
opencli browser mysession frames
```

## Creating New Adapters

### Manual Adapter Creation

```bash
# Initialize adapter structure
opencli browser init mysite/trending

# Creates ~/.opencli/clis/mysite/trending/
# - index.js (main adapter logic)
# - schema.json (command definition)
```

**Example Adapter Structure** (`~/.opencli/clis/mysite/trending/index.js`):

```javascript
import { executeAdapter } from '@jackwener/opencli-core';

export default async function handler(args, context) {
  const { limit = 10 } = args;
  
  // Use browser automation
  const session = context.session || 'adapter';
  
  try {
    // Open page
    await executeAdapter('browser', [session, 'open', 'https://mysite.com/trending']);
    
    // Wait for content
    await executeAdapter('browser', [session, 'wait', 'article.post']);
    
    // Extract data
    const posts = await executeAdapter('browser', [session, 'extract', 'article.post', '--limit', limit]);
    
    // Return structured data
    return {
      success: true,
      data: posts.map(post => ({
        title: post.querySelector('h2')?.textContent,
        author: post.querySelector('.author')?.textContent,
        url: post.querySelector('a')?.href
      }))
    };
  } finally {
    await executeAdapter('browser', [session, 'close']);
  }
}
```

**Schema Definition** (`~/.opencli/clis/mysite/trending/schema.json`):

```json
{
  "name": "trending",
  "description": "Get trending posts from mysite",
  "arguments": {
    "limit": {
      "type": "number",
      "description": "Number of posts to fetch",
      "default": 10
    }
  },
  "output": {
    "columns": ["title", "author", "url"]
  }
}
```

### Using Recon Workflow

```bash
# Analyze site pattern (SPA, SSR, JSONP, etc.)
opencli browser recon analyze https://mysite.com/trending

# Initialize adapter with recon data
opencli browser recon init mysite/trending

# Verify adapter works
opencli browser recon verify mysite/trending
```

### Site Knowledge Persistence

Store common patterns for reuse:

```bash
# Site knowledge is saved to ~/.opencli/sites/mysite/
# - auth.json (authentication strategy)
# - endpoints.json (discovered API endpoints)
# - patterns.json (site architecture patterns)
```

## Plugin Management

### Create and Install Plugins

```bash
# Create new plugin structure
opencli plugin create my-adapters

# Install from local directory
opencli plugin install file://./my-adapters

# Install from GitHub
opencli plugin install github:username/opencli-adapters

# Install from npm
opencli plugin install @myorg/opencli-adapters

# List installed plugins
opencli plugin list

# Uninstall plugin
opencli plugin uninstall my-adapters
```

### Eject and Customize Built-in Adapters

```bash
# Eject adapter to local for modification
opencli adapter eject zhihu

# Modified adapter now lives in ~/.opencli/clis/zhihu/

# Reset to built-in version
opencli adapter reset zhihu
```

## External CLI Integration

### Register Local Tools

```bash
# Register any CLI tool
opencli external register gh
opencli external register docker
opencli external register kubectl

# Use through OpenCLI
opencli gh repo list
opencli docker ps
opencli kubectl get pods
```

### Configuration

External CLIs are registered in `~/.opencli/external.json`:

```json
{
  "gh": {
    "command": "gh",
    "description": "GitHub CLI"
  },
  "docker": {
    "command": "docker",
    "description": "Docker CLI"
  }
}
```

## Environment Variables

```bash
# Daemon port (default: 19825)
export OPENCLI_DAEMON_PORT=19825

# Browser profile to use
export OPENCLI_PROFILE=work

# Window placement (foreground|background)
export OPENCLI_WINDOW=foreground

# Connection timeout (seconds)
export OPENCLI_BROWSER_CONNECT_TIMEOUT=30

# Command timeout (seconds)
export OPENCLI_BROWSER_COMMAND_TIMEOUT=60

# Chrome DevTools Protocol endpoint
export OPENCLI_CDP_ENDPOINT=ws://localhost:9222

# CDP target filter
export OPENCLI_CDP_TARGET=detail.1688.com

# Verbose logging
export OPENCLI_VERBOSE=true

# DOM snapshot debugging
export DEBUG_SNAPSHOT=1
```

## Configuration Files

### Global Config (`~/.opencli/config.json`)

```json
{
  "defaultProfile": "work",
  "daemonPort": 19825,
  "browserTimeout": 60,
  "windowMode": "background"
}
```

### Per-Site Config (`~/.opencli/sites/mysite/config.json`)

```json
{
  "auth": "COOKIE",
  "baseUrl": "https://mysite.com",
  "pattern": "SPA",
  "endpoints": {
    "trending": "/api/v1/trending"
  }
}
```

## Common Patterns

### Pattern 1: Simple Data Extraction

```javascript
export default async function handler(args, context) {
  const session = 'extract-session';
  
  await executeAdapter('browser', [session, 'open', args.url]);
  await executeAdapter('browser', [session, 'wait', args.selector]);
  
  const data = await executeAdapter('browser', [
    session, 'extract', args.selector
  ]);
  
  await executeAdapter('browser', [session, 'close']);
  
  return { success: true, data };
}
```

### Pattern 2: Form Automation

```javascript
export default async function handler(args, context) {
  const session = 'form-session';
  
  await executeAdapter('browser', [session, 'open', args.url]);
  
  // Fill form fields
  await executeAdapter('browser', [
    session, 'fill', 'input[name="email"]', args.email
  ]);
  
  await executeAdapter('browser', [
    session, 'fill', 'input[name="password"]', args.password
  ]);
  
  // Submit
  await executeAdapter('browser', [session, 'click', 'button[type="submit"]']);
  
  // Wait for success indicator
  await executeAdapter('browser', [session, 'wait', '.success-message']);
  
  await executeAdapter('browser', [session, 'close']);
  
  return { success: true };
}
```

### Pattern 3: Multi-Tab Workflow

```javascript
export default async function handler(args, context) {
  const session = 'multi-tab';
  
  // Open first tab
  await executeAdapter('browser', [session, 'open', args.url1]);
  
  // Create second tab
  const { targetId } = await executeAdapter('browser', [
    session, 'tab', 'new', args.url2
  ]);
  
  // Work in first tab
  const data1 = await executeAdapter('browser', [
    session, 'extract', '.content'
  ]);
  
  // Work in second tab
  const data2 = await executeAdapter('browser', [
    session, 'extract', '.content', '--tab', targetId
  ]);
  
  await executeAdapter('browser', [session, 'close']);
  
  return { success: true, data: { tab1: data1, tab2: data2 } };
}
```

### Pattern 4: Network Interception

```javascript
export default async function handler(args, context) {
  const session = 'network-session';
  
  // Start monitoring network
  const networkPromise = executeAdapter('browser', [
    session, 'network', '--pattern', 'api/data'
  ]);
  
  // Navigate to trigger request
  await executeAdapter('browser', [session, 'open', args.url]);
  
  // Get intercepted data
  const networkData = await networkPromise;
  
  await executeAdapter('browser', [session, 'close']);
  
  return { success: true, data: networkData };
}
```

### Pattern 5: Authenticated Session

```javascript
export default async function handler(args, context) {
  const session = 'auth-session';
  
  // Browser already logged in via Chrome profile
  await executeAdapter('browser', [session, 'open', args.url]);
  
  // Verify authentication
  const state = await executeAdapter('browser', [session, 'state']);
  
  if (state.url.includes('/login')) {
    throw new Error('Not authenticated. Please log in to Chrome.');
  }
  
  // Proceed with authenticated actions
  const data = await executeAdapter('browser', [
    session, 'extract', '.user-content'
  ]);
  
  await executeAdapter('browser', [session, 'close']);
  
  return { success: true, data };
}
```

## Troubleshooting

### Browser Not Connecting

```bash
# Check daemon status
opencli doctor

# Verify extension is installed and enabled
# Visit chrome://extensions

# Check daemon port
lsof -i :19825

# Restart daemon (kill and it will auto-restart)
pkill -f opencli-daemon
```

### Multiple Chrome Profiles

```bash
# List profiles to see which are connected
opencli profile list

# If you see multiple profiles, set default
opencli profile use <alias>

# Or specify profile per-command
opencli --profile work browser mysession open https://example.com
```

### Empty or Permission Errors

**Issue**: Commands return empty data or permission errors

**Solution**: Ensure you're logged in to the target site in Chrome

```bash
# 1. Open Chrome and log in to the site manually
# 2. Verify extension is active
# 3. Run OpenCLI command
opencli bilibili hot --limit 5
```

### Adapter Not Found

```bash
# Verify adapter is installed
opencli list | grep mysite

# Check adapter directory exists
ls ~/.opencli/clis/mysite/

# Reinstall if needed
opencli plugin install file://path/to/plugin
```

### Timeout Issues

```bash
# Increase command timeout
export OPENCLI_BROWSER_COMMAND_TIMEOUT=120

# Or per-command
opencli browser mysession wait "slow-element" --timeout 120000
```

### CDP Connection Issues

```bash
# For Electron apps or remote Chrome
export OPENCLI_CDP_ENDPOINT=ws://localhost:9222

# Filter targets if multiple exist
export OPENCLI_CDP_TARGET=myapp.com

# Test connection
opencli doctor
```

### Debug Mode

```bash
# Enable verbose logging
opencli -v browser mysession open https://example.com

# Or via environment
export OPENCLI_VERBOSE=true

# DOM snapshot debugging
export DEBUG_SNAPSHOT=1
opencli browser mysession state
```

### Adapter Verification

```bash
# Initialize adapter with recon workflow
opencli browser recon init mysite/command

# Verify adapter works correctly
opencli browser recon verify mysite/command

# Check output format
opencli mysite command --limit 5
```

## Best Practices

1. **Use explicit session names**: Helps track browser automation flows
2. **Always close sessions**: Prevents resource leaks (`opencli browser <session> close`)
3. **Handle authentication**: Verify login state before executing commands
4. **Use --tab for multi-tab**: Specify `--tab <targetId>` when working with multiple tabs
5. **Store credentials in browser**: Let Chrome manage logins, don't hardcode credentials
6. **Profile separation**: Use Chrome profiles for different accounts/contexts
7. **Error handling**: Wrap browser commands in try-finally to ensure cleanup
8. **Rate limiting**: Be respectful of site resources when creating adapters
9. **Recon first**: Use `opencli browser recon analyze` before building adapters
10. **Persist site knowledge**: Save patterns to `~/.opencli/sites/<site>/` for reuse
