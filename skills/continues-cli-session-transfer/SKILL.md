---
name: continues-cli-session-transfer
description: Transfer AI coding sessions between tools (Claude Code, Cursor, Copilot, Gemini, etc.) to bypass rate limits and preserve context
triggers:
  - "resume my Claude session in Cursor"
  - "continue this conversation in another AI tool"
  - "I hit the rate limit, switch to a different assistant"
  - "export my coding session to markdown"
  - "handoff this debug session to Gemini"
  - "show me my recent AI coding sessions"
  - "transfer context from Copilot to Codex"
  - "pick up where I left off in another tool"
---

# continues CLI Session Transfer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection

`continues` is a CLI tool that extracts AI coding sessions from 16 different tools (Claude Code, Cursor, Copilot, Gemini CLI, Codex, Amp, Cline, Roo Code, Kilo Code, Kiro, Crush, OpenCode, Factory Droid, Antigravity, Kimi CLI, Qwen Code) and transfers them to another tool with full context preservation.

When you hit rate limits mid-debug, `continues` grabs your conversation history, file changes, tool activity, and reasoning — then hands it off to a fresh AI coding assistant so you can keep working without losing context.

## Installation

No installation required with npx:

```bash
npx continues
```

Or install globally:

```bash
npm install -g continues
# Gives you both `continues` and `cont` commands
```

**Requirements:**
- Node.js 22.5+ (uses built-in `node:sqlite`)
- At least one supported AI coding tool installed

## Core Workflow

### 1. Interactive Session Picker (Default)

```bash
continues
```

This scans all 16 tools, shows recent sessions with previews, and lets you select source + destination:

```
Found 1842 sessions across 16 CLI tools
  claude: 723  codex: 72  cursor: 68  copilot: 39  ...

◆ Select a session
│ [claude]   2026-02-19 05:28  my-project    Debugging SSH tunnel config   84a36c5d
│ [copilot]  2026-02-19 04:41  my-project    Migrate presets from Electron c2f5974c
│ [codex]    2026-02-18 23:12  my-project    Fix OpenCode SQLite parser    a1e90b3f

◆ Continue in:
│ ○ Gemini   ○ Codex   ○ Amp   ○ Kiro   ...
```

When run from a project directory, sessions from that directory appear first.

### 2. Quick Resume (Same Tool)

Resume your latest session from a specific tool without the picker:

```bash
continues claude        # Latest Claude Code session
continues codex 3       # 3rd most recent Codex session
continues cursor        # Latest Cursor session
continues amp           # Latest Amp session
continues cline         # Latest Cline session
continues kiro          # Latest Kiro session
continues crush         # Latest Crush session
continues kimi          # Latest Kimi CLI session
continues qwen-code     # Latest Qwen Code session
```

This uses **native resume** — same tool, full history intact, no context injection needed.

### 3. Cross-Tool Handoff

Transfer a session from one tool to another:

```bash
# Resume session abc123 in Gemini
continues resume abc123 --in gemini

# Pass destination tool flags through
continues resume abc123 --in codex --yolo --search --add-dir /tmp

# Inspect the handoff prompt without launching the tool
continues resume abc123 --in codex --debug-prompt
```

Common flags are automatically mapped to the destination tool's equivalent:
- Model selection
- Sandbox/safety mode
- Auto-approve mode
- Additional directories

Unknown flags pass through as-is to the destination tool.

## Session Discovery & Listing

### List Sessions

```bash
# Table output (default)
continues list

# JSON output
continues list --json

# JSONL output (one session per line)
continues list --jsonl

# Filter by source tool
continues list --source claude

# Limit to N most recent
continues list -n 10

# Combine filters
continues list --source cursor --json -n 5
```

### Scan & Rebuild Index

```bash
# Show discovery stats
continues scan

# Force rebuild the session index cache
continues scan --rebuild
continues rebuild  # alias
```

The index is cached at `~/.continues/sessions.jsonl` with a 5-minute TTL. Auto-refreshes on access after expiry.

## Session Inspection

### Diagnostic View

```bash
# Full diagnostic output
continues inspect abc123

# Compact one-liner view
continues inspect abc123 --truncate 50

# Export to markdown with full verbosity
continues inspect abc123 --preset full --write-md handoff.md
```

**Diagnostic output includes:**
- Session metadata (tool, project, timestamp, model)
- Message count and content samples
- Tool activity summary (bash, edit, grep, MCP calls)
- Token usage and caching stats
- File path of original session data

## Bulk Export

Export all sessions to files for backup, analysis, or archival:

```bash
# Export all sessions to markdown (default)
continues dump all ./sessions

# Export specific tool's sessions
continues dump claude ./sessions/claude
continues dump gemini ./sessions/gemini

# Export as JSON instead of markdown
continues dump all ./sessions --json

# Control detail level with presets
continues dump all ./sessions --preset full

# Limit number of sessions
continues dump all ./sessions --limit 50
```

**File naming:** `{source}_{id}.md` or `{source}_{id}.json`

## Verbosity Presets

Control how much detail goes into handoff documents:

| Preset | Messages | Tool Samples | Subagent Detail | Use Case |
|--------|----------|--------------|-----------------|----------|
| `minimal` | 3 | 0 | None | Quick context, token-constrained |
| `standard` | 10 | 5 | 500 chars | Default balance (recommended) |
| `verbose` | 20 | 10 | 2000 chars | Complex multi-file tasks |
| `full` | 50 | All | Everything | Complete session capture |

```bash
# Use a specific preset
continues resume abc123 --in gemini --preset full

# Applies to inspect and dump too
continues inspect abc123 --preset minimal
continues dump claude ./out --preset verbose
```

## Configuration

### YAML Config File

Create `.continues.yml` in your project root for per-project defaults:

```yaml
# Verbosity preset
preset: verbose

# Override specific limits
recentMessages: 15
keyDecisions: 8

# Shell activity config
shell:
  maxSamples: 10
  stdoutLines: 20
  stderrLines: 20

# File activity config
files:
  maxSamples: 15
  contentLines: 50

# Subagent activity config
subagent:
  maxSummaryChars: 2000
```

**Config resolution order:**
1. `--config <path>` flag
2. `.continues.yml` in current directory
3. `~/.continues/config.yml` (global user config)
4. `standard` preset (default)

See `.continues.example.yml` in the repo for full reference.

### Environment Variables

Override default session directories:

```bash
export CLAUDE_CONFIG_DIR="$HOME/.config/claude"
export CODEX_HOME="$HOME/.config/codex"
export GEMINI_CLI_HOME="$HOME/.config/gemini"
export XDG_DATA_HOME="$HOME/.local/share"
```

## Handoff Document Structure

When you resume in a different tool, `continues` generates a structured markdown document:

```markdown
# Session Handoff: [Project Name]

**From:** claude (session abc123)
**Original Path:** ~/.claude/projects/my-project/sessions/abc123.jsonl
**Started:** 2026-02-19 05:28
**Model:** claude-sonnet-4
**Tokens:** 45,230 in / 12,847 out (7,234 cached)

## Context

[Summary of what the user was working on]

## Recent Messages

**User:** I need to debug the SSH tunnel configuration
**Assistant:** I'll help you check the SSH config...

[10 most recent messages by default]

## Tool Activity

- **Bash** (×47): `$ npm test → exit 0` · `$ git status → exit 0` · `$ npm run build → exit 1`
- **Edit** (×12): `edit src/auth.ts` · `edit src/api/routes.ts` · `edit tests/auth.test.ts`
- **Grep** (×8): `grep "handleLogin" src/` · `grep "JWT_SECRET"` · `grep "middleware"`

## Key Decisions

- Switched from JWT to session tokens for better security
- Added middleware for token refresh race condition handling
- Need to handle the edge case where token refresh races with logout

## Files Referenced

- src/auth.ts (edited 3 times)
- src/api/routes.ts (edited 2 times)
- tests/auth.test.ts (created)
```

This handoff is injected as the first message in the new tool, giving the AI full context to continue.

## Supported Tools & Session Locations

| Tool | Format | Default Location |
|------|--------|------------------|
| Claude Code | JSONL | `~/.claude/projects/` |
| Codex | JSONL | `~/.codex/sessions/` |
| Copilot | YAML + JSONL | `~/.copilot/session-state/` |
| Gemini CLI | JSON | `~/.gemini/tmp/*/chats/` |
| OpenCode | SQLite | `~/.local/share/opencode/storage/` |
| Factory Droid | JSONL + JSON | `~/.factory/sessions/` |
| Cursor | JSONL | `~/.cursor/projects/*/agent-transcripts/` |
| Amp | JSON | `~/.local/share/amp/threads/` |
| Kiro | JSON | `~/Library/Application Support/Kiro/workspace-sessions/` |
| Crush | SQLite | `~/.crush/crush.db` |
| Cline | JSON | VS Code `globalStorage/saoudrizwan.claude-dev/tasks/` |
| Roo Code | JSON | VS Code `globalStorage/rooveterinaryinc.roo-cline/tasks/` |
| Kilo Code | JSON | VS Code `globalStorage/kilocode.kilo-code/tasks/` |
| Antigravity | PB + brain | `~/.gemini/antigravity/` |
| Kimi CLI | JSONL + JSON | `~/.kimi/sessions/` |
| Qwen Code | JSONL | `~/.qwen/projects/*/chats/` |

**All reads are read-only** — `continues` never modifies your session files.

## TypeScript Integration (Library Usage)

While `continues` is primarily a CLI tool, you can import its parsers programmatically:

```typescript
import { SessionParser } from 'continues';
import { parseClaude } from 'continues/parsers/claude';

// Parse a Claude session file
const session = await parseClaude('~/.claude/projects/my-project/sessions/abc123.jsonl');

console.log(session.messages);
console.log(session.toolActivity);
console.log(session.metadata);
```

## Common Patterns

### Rate Limit Workflow

```bash
# Hit rate limit in Claude
# Exit Claude session

# Resume in Gemini immediately
continues claude --in gemini

# Or be explicit with session ID
continues resume abc123 --in gemini --preset verbose
```

### Multi-Tool Debugging

```bash
# Start in Claude Code
claude

# Hit rate limit, switch to Codex
continues claude --in codex

# Codex has different rate limit, switch to Cursor
continues codex --in cursor

# Export the full journey for documentation
continues dump all ./debug-session-archive --preset full
```

### Session Archival

```bash
# Daily backup of all sessions
continues dump all ~/backups/ai-sessions-$(date +%Y%m%d) --json

# Archive just Claude sessions with full detail
continues dump claude ~/archives/claude --preset full --limit 100
```

### Inspect Before Handoff

```bash
# Check what will be transferred
continues inspect abc123 --preset standard

# Generate handoff document without launching tool
continues resume abc123 --in gemini --debug-prompt > handoff.txt

# Review handoff.txt, then proceed
continues resume abc123 --in gemini
```

## Troubleshooting

### "No sessions found"

**Cause:** `continues` can't locate session directories for installed tools.

**Fix:**
1. Verify the tool is installed: `which claude` or `which cursor`
2. Check if sessions exist: `ls ~/.claude/projects/` or `ls ~/.cursor/projects/`
3. Override location with env vars:
   ```bash
   export CLAUDE_CONFIG_DIR="$HOME/custom-location"
   continues scan --rebuild
   ```

### "Session index is stale"

**Cause:** Cached index hasn't refreshed after adding new sessions.

**Fix:**
```bash
continues rebuild
```

### "Unknown flags passed to destination tool"

**Cause:** You used a flag that `continues` doesn't recognize and the destination tool doesn't support it.

**Fix:** This is expected behavior — unknown flags pass through. Check the destination tool's `--help` to verify supported options:
```bash
gemini --help
codex --help
```

### "Handoff document is too large"

**Cause:** Session has hundreds of messages and you're using `--preset full`.

**Fix:**
```bash
# Use a less verbose preset
continues resume abc123 --in gemini --preset standard

# Or minimal for token-constrained tools
continues resume abc123 --in gemini --preset minimal
```

### Symlink traversal errors

**Cause:** Session directory contains symlinks that `fs.Dirent.isDirectory()` doesn't follow.

**Fix:** This is fixed in recent versions. If you see this error:
1. Update to latest: `npm install -g continues@latest`
2. Force rebuild: `continues rebuild`

### Zero token display

If you see "0 in / 0 out" for sessions that should have token data:

**Cause:** Some tools don't store token metadata in session files.

**Expected behavior:** Only Claude Code, Codex, Cursor, and Gemini consistently report tokens. Other tools may show zero.

## Development & Extending

### Adding a New Tool Parser

1. Create parser in `src/parsers/your-tool.ts`:

```typescript
import type { Session } from '../types';

export async function parseYourTool(filePath: string): Promise<Session> {
  // Read session file (JSONL, JSON, SQLite, etc.)
  // Extract messages, tool activity, metadata
  // Return Session object
  
  return {
    id: 'session-id',
    source: 'your-tool',
    projectPath: '/path/to/project',
    timestamp: new Date(),
    messages: [...],
    toolActivity: [...],
    metadata: { ... }
  };
}
```

2. Add tool name to `src/types/tool-names.ts`:

```typescript
export const TOOL_NAMES = [
  'claude', 'codex', 'cursor', 'gemini',
  'your-tool',  // Add here
  // ...
] as const;
```

3. Register in `src/parsers/registry.ts`:

```typescript
import { parseYourTool } from './your-tool';

export const PARSERS = {
  'claude': parseClaude,
  'codex': parseCodex,
  'your-tool': parseYourTool,  // Add here
  // ...
};
```

The registry has compile-time completeness checking — if you forget to add the parser, TypeScript throws an error at import.

### Running Tests

```bash
# Install dependencies
pnpm install

# Run tests
pnpm test

# Watch mode during development
pnpm run test:watch

# Run without building
pnpm run dev
```

## Quick Reference

```bash
# Interactive picker
continues

# Quick resume (same tool)
continues claude
continues codex 3

# Cross-tool handoff
continues resume <id> --in <tool>

# List sessions
continues list [--json|--jsonl] [--source <tool>] [-n <count>]

# Inspect session
continues inspect <id> [--preset <level>] [--truncate <chars>]

# Export sessions
continues dump <tool|all> <dir> [--json] [--preset <level>] [--limit <n>]

# Rebuild index
continues scan --rebuild

# Global flags
--config <path>    # Custom config file
--preset <level>   # minimal | standard | verbose | full
--verbose          # Show debug output
--debug            # Extra verbose logging
```
