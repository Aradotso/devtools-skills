---
name: hermes-client-web-ui
description: Web-based chat interface for Hermes Agent with multi-profile management, streaming chat, and interactive terminal integration
triggers:
  - how do I use Hermes Client web interface
  - set up Hermes Client dashboard
  - manage multiple Hermes agent profiles
  - configure Hermes Client chat UI
  - integrate with Hermes Agent CLI
  - stream Hermes conversations in browser
  - use Hermes Client API endpoints
  - troubleshoot Hermes Client connection issues
---

# Hermes Client Web UI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

A web-based chat interface for the Hermes Agent by Nous Research. Manages multiple Hermes profiles as separate "agents", runs conversations with full streaming via SSE, and provides an interactive terminal for setup commands. Each UI agent maps 1:1 to a Hermes profile with its own home directory, config, and sessions.

## Installation

**Prerequisites:**
- Node.js 18+
- Hermes Agent installed with `hermes` on your `PATH`
- Git for Windows (Windows only, for auto-update)
- Visual Studio Build Tools (Windows only, for native modules)

```bash
# Verify Hermes is installed
hermes --version
hermes status

# Clone and start
git clone https://github.com/lotsoftick/hermes_client.git
cd hermes_client
npm start
```

`npm start` builds, deploys to `~/.hermes_client`, installs auto-start (LaunchAgent/Startup), and creates the global `hermes_client` command.

Default URLs:
- Client: http://localhost:18888
- API: http://localhost:18889

Default credentials:
- Email: `admin@admin.com`
- Password: `123456`

## Service Management

After `npm start`, use the global command from any directory:

```bash
# Start/stop/restart
hermes_client start
hermes_client stop
hermes_client restart
hermes_client status

# Uninstall
hermes_client uninstall                # Keeps database
hermes_client uninstall --purge        # Deletes database (confirms)
```

## Development Mode

```bash
# Hot reload (API + Client)
npm run dev

# Generate .env only
npm run setup

# Stop services
npm run stop
```

## Configuration

### Port Configuration (`~/.hermes_client/.env`)

Created automatically on first run:

```env
API_PORT=18889
CLIENT_PORT=18888
```

Apply changes:
```bash
hermes_client restart   # Production
npm run dev             # Development
```

### API Configuration (`api/.env`)

Auto-generated from `api/.env.example`:

```env
NODE_ENV=development
JWT_SECRET=<random-generated>
DB_PATH=./data/hermes.sqlite
PORT=18889
ALLOWED_DOMAIN=
HERMES_STRICT_CORS=0
API_PUBLIC_URL=
HERMES_BIN=
HERMES_HOME=~/.hermes
HERMES_CLIENT_UPLOADS_DIR=~/.hermes_client/uploads
HERMES_SINGLE_USER_MODE=1
```

**Key variables:**
- `HERMES_BIN`: Override Hermes binary path if not on `PATH`
- `HERMES_STRICT_CORS=1`: Enable strict CORS with `ALLOWED_DOMAIN` allowlist
- `HERMES_SINGLE_USER_MODE`: Lock UI to single-user account page (`1/true/yes/on` or `0/false/no/off`)

## Architecture

### CLI-Driven Streaming

Every chat turn spawns `hermes -p <profile> chat -Q -q "<message>"` and streams stdout over Server-Sent Events:

```typescript
// Example API streaming endpoint structure
app.post('/api/conversations/:conversationId/messages', async (req, res) => {
  const { profileName, message } = req.body;
  
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  const hermes = spawn('hermes', [
    '-p', profileName,
    'chat',
    '-Q',  // Quiet mode
    '-q', message
  ]);
  
  hermes.stdout.on('data', (chunk) => {
    res.write(`data: ${JSON.stringify({ content: chunk.toString() })}\n\n`);
  });
  
  hermes.on('close', () => {
    res.write('data: [DONE]\n\n');
    res.end();
  });
});
```

### Profile Management

Each UI agent maps to a Hermes profile:

```bash
# Backend runs these commands
hermes profile add <name>
hermes profile delete <name>
hermes profile list
hermes -p <name> model  # Via interactive terminal
```

### Session Sync

Sessions started in standalone `hermes` REPL auto-appear in sidebar. Backend watches `~/.hermes/profiles/<profile>/sessions/*.json`:

```typescript
// Example session sync pattern
import { watch } from 'fs';
import { readdir, readFile } from 'fs/promises';

async function syncSessions(profileName: string) {
  const sessionsDir = `${process.env.HERMES_HOME}/profiles/${profileName}/sessions`;
  
  // Initial load
  const files = await readdir(sessionsDir);
  for (const file of files.filter(f => f.endsWith('.json'))) {
    const session = JSON.parse(await readFile(`${sessionsDir}/${file}`, 'utf-8'));
    // Insert/update in SQLite
    await db.run(
      'INSERT OR REPLACE INTO conversations (session_key, profile_name, title, updated_at) VALUES (?, ?, ?, ?)',
      [session.key, profileName, session.title, session.updated_at]
    );
  }
  
  // Watch for changes
  watch(sessionsDir, { persistent: false }, (event, filename) => {
    if (filename?.endsWith('.json')) {
      // Re-sync changed session
    }
  });
}
```

### File Uploads

Files stored under `~/.hermes_client/uploads/<conversationId>/` and passed to Hermes by absolute path:

```typescript
// Example upload handling
app.post('/api/conversations/:conversationId/upload', upload.single('file'), (req, res) => {
  const { conversationId } = req.params;
  const uploadDir = `${process.env.HERMES_CLIENT_UPLOADS_DIR}/${conversationId}`;
  
  // multer stores file at uploadDir/filename
  const absolutePath = path.resolve(uploadDir, req.file.filename);
  
  // For images, pass via --image flag
  if (req.file.mimetype.startsWith('image/')) {
    spawn('hermes', ['-p', profile, 'chat', '--image', absolutePath, '-q', message]);
  } else {
    // For other files, reference in prompt
    spawn('hermes', ['-p', profile, 'chat', '-q', `File: ${absolutePath}\n\n${message}`]);
  }
  
  res.json({ path: `/uploads/${conversationId}/${req.file.filename}` });
});
```

## API Endpoints

### Authentication

```typescript
// POST /api/auth/login
fetch('http://localhost:18889/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'admin@admin.com',
    password: '123456'
  })
});
// Response: { token: string, user: { id, email, name } }

// JWT required for all other endpoints
headers: { 'Authorization': `Bearer ${token}` }
```

### Agents (Profiles)

```typescript
// GET /api/agents - List all profiles
fetch('http://localhost:18889/api/agents', {
  headers: { 'Authorization': `Bearer ${token}` }
});

// POST /api/agents - Create new profile
fetch('http://localhost:18889/api/agents', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'research-assistant' })
});

// DELETE /api/agents/:name - Delete profile
fetch('http://localhost:18889/api/agents/research-assistant', {
  method: 'DELETE',
  headers: { 'Authorization': `Bearer ${token}` }
});
```

### Conversations

```typescript
// GET /api/conversations?profileName=default - List conversations
fetch('http://localhost:18889/api/conversations?profileName=default', {
  headers: { 'Authorization': `Bearer ${token}` }
});

// POST /api/conversations - Create conversation
fetch('http://localhost:18889/api/conversations', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    profileName: 'default',
    title: 'New Chat'
  })
});

// GET /api/conversations/:id/messages - Get messages
fetch('http://localhost:18889/api/conversations/abc123/messages', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

### Streaming Chat

```typescript
// POST /api/conversations/:id/messages - Send message (SSE stream)
const eventSource = new EventSource(
  'http://localhost:18889/api/conversations/abc123/messages',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      content: 'Hello!',
      profileName: 'default'
    })
  }
);

eventSource.onmessage = (event) => {
  if (event.data === '[DONE]') {
    eventSource.close();
  } else {
    const chunk = JSON.parse(event.data);
    console.log(chunk.content);
  }
};
```

### Interactive Terminal (PTY)

```typescript
// WebSocket upgrade for xterm.js
const ws = new WebSocket('ws://localhost:18889/ws/pty?token=' + token);

ws.onopen = () => {
  // Start model config command
  ws.send(JSON.stringify({
    type: 'start',
    command: 'hermes',
    args: ['-p', 'default', 'model'],
    cwd: process.env.HOME
  }));
};

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  if (msg.type === 'output') {
    terminal.write(msg.data);  // xterm.js instance
  }
};

// Send input
terminal.onData((data) => {
  ws.send(JSON.stringify({ type: 'input', data }));
});

// Resize PTY
terminal.onResize(({ cols, rows }) => {
  ws.send(JSON.stringify({ type: 'resize', cols, rows }));
});
```

## Client Patterns

### React Hook for Streaming

```typescript
import { useEffect, useState } from 'react';

function useStreamingChat(conversationId: string, token: string) {
  const [messages, setMessages] = useState<string[]>([]);
  const [streaming, setStreaming] = useState(false);
  
  const sendMessage = async (content: string, profileName: string) => {
    setStreaming(true);
    const response = await fetch(`/api/conversations/${conversationId}/messages`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ content, profileName })
    });
    
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      
      const chunk = decoder.decode(value);
      const lines = chunk.split('\n').filter(l => l.startsWith('data: '));
      
      for (const line of lines) {
        const data = line.slice(6);
        if (data === '[DONE]') {
          setStreaming(false);
          break;
        }
        const parsed = JSON.parse(data);
        setMessages(prev => [...prev, parsed.content]);
      }
    }
  };
  
  return { messages, streaming, sendMessage };
}
```

### File Upload with Preview

```typescript
async function uploadFile(
  conversationId: string,
  file: File,
  token: string
): Promise<string> {
  const formData = new FormData();
  formData.append('file', file);
  
  const response = await fetch(`/api/conversations/${conversationId}/upload`, {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}` },
    body: formData
  });
  
  const { path } = await response.json();
  return `${API_BASE_URL}${path}`;
}

// Usage in React
function MessageComposer() {
  const [files, setFiles] = useState<File[]>([]);
  
  const handleDrop = (e: React.DragEvent) => {
    e.preventDefault();
    setFiles([...files, ...Array.from(e.dataTransfer.files)]);
  };
  
  const handleSend = async (message: string) => {
    const uploadedUrls = await Promise.all(
      files.map(f => uploadFile(conversationId, f, token))
    );
    // Send message with file references
  };
  
  return (
    <div onDrop={handleDrop} onDragOver={e => e.preventDefault()}>
      {/* Composer UI */}
    </div>
  );
}
```

## Database Schema

SQLite at `~/.hermes_client/data/hermes.sqlite`:

```sql
-- Users (JWT auth)
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  name TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Conversations (mirrors Hermes sessions)
CREATE TABLE conversations (
  id TEXT PRIMARY KEY,
  session_key TEXT,
  profile_name TEXT NOT NULL,
  title TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Messages (mirrors Hermes session messages)
CREATE TABLE messages (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  conversation_id TEXT,
  role TEXT CHECK(role IN ('user', 'assistant', 'system')),
  content TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE
);

-- Themes
CREATE TABLE themes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT UNIQUE NOT NULL,
  colors TEXT  -- JSON blob
);
```

## Troubleshooting

### Hermes binary not found

```bash
# Check PATH
which hermes
hermes --version

# If not found, set explicit path in api/.env
HERMES_BIN=/path/to/hermes

# Common locations checked automatically:
# ~/.local/bin/hermes
# ~/.hermes/hermes-agent/venv/bin/hermes
# /opt/homebrew/bin/hermes (macOS)
# /usr/local/bin/hermes
```

### Port conflicts

```bash
# Check what's using the port
lsof -i :18888
lsof -i :18889

# Change ports in ~/.hermes_client/.env
API_PORT=19000
CLIENT_PORT=19001

hermes_client restart
```

### CORS issues (remote access)

```bash
# Edit ~/.hermes_client/api/.env
ALLOWED_DOMAIN=192.168.1.100:18888,100.64.0.1:18888  # LAN + Tailscale
HERMES_STRICT_CORS=1

hermes_client restart
```

### Session sync not working

```bash
# Verify Hermes session directory exists
ls ~/.hermes/profiles/default/sessions/

# Check file watcher permissions
# Backend watches ~/.hermes/profiles/*/sessions/*.json
# Ensure readable by user running API server

# Force sync by restarting
hermes_client restart
```

### Interactive terminal (PTY) not connecting

```bash
# Verify WebSocket upgrade works
wscat -c "ws://localhost:18889/ws/pty?token=YOUR_JWT_TOKEN"

# Check JWT_SECRET matches between client and server
# Both use the same secret from api/.env

# Windows: Ensure Python is on PATH (PTY bridge requires it)
python --version
```

### Windows install fails

```bash
# Run PowerShell as Administrator for first npm start
# Required for npm link and auto-start setup

# Install Visual Studio Build Tools
# https://visualstudio.microsoft.com/downloads/
# Select "Desktop development with C++"

# Install Git for Windows
# https://git-scm.com/download/win
```

### Database locked errors

```bash
# Stop all services
hermes_client stop

# Check for stale processes
ps aux | grep hermes_client

# Remove lock and restart
rm ~/.hermes_client/data/hermes.sqlite-wal
hermes_client start
```

### Uploads not working

```bash
# Check upload directory exists and is writable
ls -la ~/.hermes_client/uploads/

# Verify HERMES_CLIENT_UPLOADS_DIR in api/.env
# Default: ~/.hermes_client/uploads

# Check disk space
df -h ~/.hermes_client/
```

## Common Patterns

### Adding a new agent/profile with model config

```typescript
// 1. Create profile via API
const response = await fetch('/api/agents', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'code-reviewer' })
});

// 2. Open PTY terminal for model config
const ws = new WebSocket(`ws://localhost:18889/ws/pty?token=${token}`);
ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'start',
    command: 'hermes',
    args: ['-p', 'code-reviewer', 'model'],
    cwd: process.env.HOME
  }));
};

// User interacts with arrow-key picker in xterm.js
// API key prompts work via PTY bridge
```

### Continuing terminal conversation in web UI

```bash
# Start in terminal
hermes -p myprofile chat
> What is TypeScript?
# [Hermes responds with session key abc123]

# Session auto-appears in web UI sidebar within seconds
# Click to continue conversation in browser
# Backend detects ~/.hermes/profiles/myprofile/sessions/abc123.json
```

### Resuming web conversation in terminal

```bash
# Get session key from web UI URL or conversation list
hermes -p myprofile chat -r abc123
> Continue our TypeScript discussion

# New turns stream back to web UI if chat is open
```

### Multi-file context in chat

```typescript
// Upload multiple files
const files = ['src/index.ts', 'package.json', 'README.md'];
const uploads = await Promise.all(
  files.map(f => uploadFile(conversationId, new File([...], f), token))
);

// Send message with all file contexts
await sendMessage(
  `Review these files for issues:\n${uploads.map(u => u.path).join('\n')}`,
  profileName
);

// Backend passes absolute paths to hermes chat command
```
