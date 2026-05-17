---
name: claude-devtools-inspector
description: Inspect Claude Code session logs, tool calls, token usage, subagents, and context window using claude-devtools visual UI
triggers:
  - inspect claude code session logs
  - analyze claude token usage and context window
  - debug claude tool calls and subagents
  - view claude code session history
  - check what claude did in this session
  - examine claude context compaction
  - review claude memory and thinking steps
  - troubleshoot claude code behavior
---

# claude-devtools Inspector

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

claude-devtools is the missing DevTools for Claude Code. It reads session logs from `~/.claude/` and reconstructs everything Claude Code hides: tool calls, token usage, subagent trees, thinking steps, context window breakdown, and memory layers. Works with all existing sessions — no wrapper, no API keys, no configuration.

## What claude-devtools Does

- **Session inspection**: View full transcripts of any Claude Code session with syntax-highlighted messages
- **Tool call visibility**: See exact inputs/outputs for Read, Edit, Bash, Search, and all other tools
- **Token attribution**: Per-turn breakdown across 7 categories (CLAUDE.md, skills, @-mentions, tool I/O, thinking, team overhead, user text)
- **Context compaction**: Visualize when context hits limits and what gets compressed/dropped
- **Subagent trees**: Isolated execution traces for nested agents with cost/duration metrics
- **Memory viewer**: Browse project memory layers stored in `~/.claude/projects/<project>/memory/`
- **Thinking steps**: Full extended thinking content that's invisible in the terminal
- **Copy/paste**: Export sessions to Markdown/JSON, one-click copy on code blocks

## Installation

### Desktop App (macOS)

```bash
# Homebrew
brew install --cask claude-devtools

# Or download from releases
open https://github.com/matt1398/claude-devtools/releases/latest
```

### Docker / Standalone Server

```bash
# Using docker-compose
docker compose up

# Or manual docker run
docker build -t claude-devtools .
docker run -p 3456:3456 -v ~/.claude:/data/.claude:ro claude-devtools

# Open http://localhost:3456
```

### Build from Source

```bash
git clone https://github.com/matt1398/claude-devtools.git
cd claude-devtools
pnpm install
pnpm dev  # Development mode with hot reload
pnpm build  # Production build
```

## Key Configuration

claude-devtools reads from `~/.claude/` by default. No config files needed.

### Environment Variables (Docker/Standalone)

```bash
# .env or docker-compose environment
CLAUDE_ROOT=~/.claude  # Path to Claude data directory
HOST=0.0.0.0           # Bind address
PORT=3456              # Listen port
```

### Custom Claude Directory

If your Claude data is in a non-standard location:

```bash
# Docker
docker run -p 3456:3456 -v /custom/path:/data/.claude:ro claude-devtools

# Desktop app: use "Open Custom Directory" in UI
```

## Session Log Format

Claude Code writes JSONL logs to `~/.claude/sessions/<session-id>.jsonl`. Each line is a JSON object:

```typescript
interface SessionLogEntry {
  type: 'user' | 'assistant' | 'tool_use' | 'tool_result' | 'thinking' | 'team_message';
  timestamp: string;
  content?: string;
  tool_name?: string;
  tool_input?: Record<string, unknown>;
  tool_output?: string;
  tokens?: {
    input: number;
    output: number;
    cache_read?: number;
    cache_creation?: number;
  };
  context?: {
    used: number;
    limit: number;
    segments: { type: string; tokens: number }[];
  };
}
```

## Common Use Cases

### Inspect a Running Session

```bash
# Start Claude Code in terminal
claude code --session my-task

# In claude-devtools UI:
# 1. Open app (or navigate to http://localhost:3456 if using Docker)
# 2. Session appears automatically in sidebar
# 3. Click to view live updates
```

### Find High Token Usage

```typescript
// Use the UI token filter
// Or query JSONL directly with jq
cat ~/.claude/sessions/<session-id>.jsonl \
  | jq 'select(.tokens.input > 10000) | {timestamp, tokens}'
```

### Export Session Transcript

```bash
# In claude-devtools UI:
# 1. Open session
# 2. Click "Export" button (top right)
# 3. Choose format: Markdown, JSON, or Plain Text

# Result includes all messages, tool calls, and thinking steps
```

### Debug Context Compaction

When Claude "forgets" earlier context:

1. Open session in claude-devtools
2. Navigate to Context tab
3. Look for red/orange segments in token visualization
4. Hover over compacted segments to see what was dropped
5. Click "Show Compaction Events" to see exact turn numbers

### Review Tool Calls

```bash
# In UI: Filter by tool type
# Supported filters:
# - Tool: Read, Edit, Bash, Search, ListDir, etc.
# - Status: success, error
# - Contains: regex pattern match on input/output
```

### Check Subagent Activity

```typescript
// Subagent trees show:
// - Agent hierarchy (parent → child)
// - Tool calls per agent
// - Token usage per agent
// - Duration and cost breakdown

// In UI: Click "Subagents" tab
// Nested agents render recursively with full tool traces
```

### View Project Memory

```bash
# Claude stores memory at:
# ~/.claude/projects/<project-id>/memory/
#   ├── MEMORY.md           # Index of layers
#   ├── working-style.md    # Layer content
#   ├── architecture.md
#   └── ...

# In claude-devtools:
# 1. Click "Memory" in sidebar
# 2. Browse layers as clickable index
# 3. Click "Open in..." to launch in Cursor/VS Code/etc.
```

### SSH Remote Sessions

Inspect sessions on remote machines:

```typescript
// claude-devtools reads ~/.ssh/config automatically
// In UI: Settings → Remote → Add SSH Connection

// Example ~/.ssh/config:
Host prod-server
  HostName 192.168.1.100
  User deploy
  IdentityFile ~/.ssh/id_rsa

// In claude-devtools: Select "prod-server"
// Sessions from remote ~/.claude appear in sidebar
```

## Notification Setup

Trigger alerts on specific events:

```typescript
// In UI: Settings → Notifications

// Built-in triggers:
{
  ".env access": {
    pattern: "\\.env",
    field: "tool_input.path"
  },
  "High token usage": {
    threshold: 50000,
    field: "tokens.input"
  },
  "Tool error": {
    field: "tool_output",
    contains: "error|failed|exception"
  }
}

// Custom regex trigger:
{
  "API key exposure": {
    pattern: "(sk-[A-Za-z0-9]{32}|ghp_[A-Za-z0-9]{36})",
    field: "content"
  }
}
```

## Troubleshooting

### Session Not Appearing

```bash
# Check Claude data directory exists
ls -la ~/.claude/sessions/

# Verify permissions
chmod 755 ~/.claude
chmod 644 ~/.claude/sessions/*.jsonl

# Force refresh in UI: Cmd+R (macOS) / Ctrl+R (Linux/Windows)
```

### Docker Volume Mount Issues

```bash
# Ensure read permissions on host
chmod -R 755 ~/.claude

# Use absolute paths in docker-compose.yml
volumes:
  - /home/user/.claude:/data/.claude:ro

# Check mount inside container
docker exec <container-id> ls -la /data/.claude/sessions
```

### Incomplete Session Data

```bash
# Claude Code only writes logs for:
# - Sessions with --session flag
# - Project-level sessions (not global prompts)

# Missing thinking steps? Check Claude Code version
claude --version  # Thinking logs added in v2.2.0+

# Verify JSONL integrity
jq empty ~/.claude/sessions/<session-id>.jsonl
```

### High Memory Usage (Large Sessions)

```bash
# Sessions with >100k tokens may consume significant RAM
# Solutions:
# 1. Close unused session tabs in UI
# 2. Use "Export" → "JSON" and analyze offline
# 3. Increase Docker memory limit:
docker run -m 4g -p 3456:3456 claude-devtools
```

### macOS Gatekeeper Block

```bash
# On first launch, right-click app → Open
# Or disable quarantine:
xattr -d com.apple.quarantine /Applications/claude-devtools.app
```

## API Reference (Standalone Mode)

When running as a server (Docker/Node), claude-devtools exposes:

```typescript
// List all sessions
GET /api/sessions
// Response: { sessions: [{ id, timestamp, title }] }

// Get session detail
GET /api/sessions/:id
// Response: { session: { id, messages: [...] } }

// Export session
GET /api/sessions/:id/export?format=markdown|json|txt
// Response: Content-Type varies by format

// Health check
GET /health
// Response: { status: "ok", version: "1.x.x" }
```

## Integration with Claude Code

```bash
# Run Claude Code with session logging
claude code --session my-feature

# Session appears instantly in claude-devtools
# Live updates as Claude executes tools

# Use claude-devtools to:
# - Monitor token usage during long sessions
# - Verify which files Claude read/edited
# - Debug subagent spawning issues
# - Export transcript for team review
```

## Performance Tips

```bash
# Limit session retention
# Delete old sessions to reduce scan time:
find ~/.claude/sessions -mtime +30 -delete

# Index large sessions
# claude-devtools caches parsed sessions in memory
# First load may be slow for 100+ MB logs

# Use filters early
# Apply tool/timestamp filters before expanding messages
```

## Security Notes

- claude-devtools is **read-only** — never modifies session logs
- Standalone server has **zero outbound network calls**
- For maximum isolation: `docker run --network none -p 3456:3456 -v ~/.claude:/data/.claude:ro claude-devtools`
- See `SECURITY.md` in repo for full threat model

## Development Commands

```bash
# Run tests
pnpm test

# Type checking
pnpm typecheck

# Lint
pnpm lint

# Full quality gate (types + lint + test + build)
pnpm check

# Build Electron app
pnpm build
```

## Resources

- Documentation: https://claude-dev.tools/docs
- JSONL format reference: https://claude-dev.tools/docs/jsonl-format
- Comparison with `--verbose`: https://claude-dev.tools/docs/verbose-vs-devtools
- Debugging walkthrough: https://claude-dev.tools/docs/why-claude-forgot
- GitHub issues: https://github.com/matt1398/claude-devtools/issues
