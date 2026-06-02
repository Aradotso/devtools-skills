---
name: continues-ai-session-transfer
description: Resume AI coding sessions across Claude Code, Cursor, Copilot, Gemini, Codex, and 11+ other tools
triggers:
  - switch my AI session to another tool
  - transfer this conversation to gemini
  - continue this session in cursor
  - hit the rate limit need to switch tools
  - export my claude session to copilot
  - resume my coding session in another AI
  - hand off this context to codex
  - migrate my AI conversation between tools
---

# continues-ai-session-transfer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection

## What it does

`continues` solves the context-switching problem when AI coding tools hit rate limits or need to be switched mid-session. It extracts conversation history, file changes, tool activity, and working state from one AI coding assistant and transfers it to another. Supports 16 tools with 240+ cross-tool handoff paths.

**Supported tools**: Claude Code, Codex, GitHub Copilot CLI, Gemini CLI, Cursor, Amp, Cline, Roo Code, Kilo Code, Kiro, Crush, OpenCode, Factory Droid, Antigravity, Kimi CLI, Qwen Code

## Installation

No installation required with `npx`:

```bash
npx continues
```

For permanent installation:

```bash
npm install -g continues
```

This gives you both `continues` and `cont` as global commands.

**Requirements**: Node.js 22.5+ (uses built-in `node:sqlite`)

## Core commands

### Interactive session picker

Launch the TUI to select any session across all tools:

```bash
continues
```

Automatically prioritizes sessions from the current project directory.

### Quick resume in same tool

Resume the most recent (or Nth most recent) session from a specific tool:

```bash
continues claude          # Latest Claude Code session
continues codex 3         # 3rd most recent Codex session
continues cursor          # Latest Cursor session
continues gemini          # Latest Gemini CLI session
continues cline           # Latest Cline session
continues amp             # Latest Amp session
```

Works for all 16 supported tools. Uses native resume with full history.

### Cross-tool handoff

Transfer a session from one tool to another:

```bash
# Basic handoff by session ID
continues resume abc123 --in gemini

# With tool-specific flags
continues resume abc123 --in codex --yolo --search

# Pass extra directories to the target tool
continues resume abc123 --in cursor --add-dir /tmp/shared

# Inspect the handoff prompt without launching the tool
continues resume abc123 --in gemini --debug-prompt
```

### List sessions

```bash
# Table view
continues list

# Filter by source tool
continues list --source claude

# JSON output for scripting
continues list --json

# JSONL output (one session per line)
continues list --jsonl

# Limit results
continues list -n 10
```

### Inspect session details

View detailed diagnostic information about a session:

```bash
# Full diagnostic view
continues inspect abc123

# Export to markdown
continues inspect abc123 --preset full --write-md handoff.md

# Compact one-liner view
continues inspect abc123 --truncate 50
```

### Bulk export sessions

Export all sessions to files for backup or analysis:

```bash
# Export all sessions to markdown
continues dump all ./sessions

# Export specific tool's sessions
continues dump claude ./sessions/claude
continues dump cursor ./sessions/cursor

# Export as JSON
continues dump all ./sessions --json

# Control detail level
continues dump all ./sessions --preset full

# Limit number of exports
continues dump all ./sessions --limit 50
```

Output files: `{source}_{id}.md` or `{source}_{id}.json`

### Scan and rebuild index

```bash
# Show discovery stats
continues scan

# Force rebuild session index
continues rebuild
continues scan --rebuild
```

Session index cached at `~/.continues/sessions.jsonl` (5-minute TTL).

## Configuration

### Verbosity presets

Control how much context gets transferred:

| Preset | Messages | Tool samples | Use case |
|--------|----------|--------------|----------|
| `minimal` | 3 | 0 | Token-constrained transfers |
| `standard` | 10 | 5 | Default balanced handoff |
| `verbose` | 20 | 10 | Complex multi-file tasks |
| `full` | 50 | All | Complete session capture |

```bash
continues resume abc123 --preset verbose --in gemini
continues inspect abc123 --preset full
continues dump all ./backup --preset minimal
```

### YAML configuration

Create `.continues.yml` in your project root:

```yaml
preset: verbose
recentMessages: 15
shell:
  maxSamples: 10
  stdoutLines: 20
files:
  maxSamples: 8
thinking:
  maxChars: 2000
keyDecisions:
  maxCount: 5
```

Configuration resolution order:
1. `--config <path>` flag
2. `.continues.yml` in current directory
3. `~/.continues/config.yml` (global)
4. `standard` preset (default)

### Environment variable overrides

Customize tool data directories:

```bash
export CLAUDE_CONFIG_DIR="$HOME/.config/claude"
export CODEX_HOME="$HOME/.local/codex"
export GEMINI_CLI_HOME="$HOME/.gemini-custom"
export XDG_DATA_HOME="$HOME/.local/share"
```

## Code examples

### TypeScript: Programmatic session access

```typescript
import { SessionRegistry } from 'continues';
import { ClaudeParser } from 'continues/parsers';

// Initialize registry
const registry = new SessionRegistry();
await registry.scan();

// Get all sessions from Claude Code
const claudeSessions = registry.getSessions('claude');

// Find sessions for current project
const projectSessions = registry.getSessionsForProject(process.cwd());

// Parse a specific session
const parser = new ClaudeParser();
const sessionData = await parser.parse('~/.claude/projects/abc123/session.jsonl');

console.log(`Messages: ${sessionData.messages.length}`);
console.log(`Files touched: ${sessionData.files.length}`);
console.log(`Shell commands: ${sessionData.shell.length}`);
```

### TypeScript: Custom handoff generation

```typescript
import { HandoffGenerator } from 'continues/handoff';
import { SessionParser } from 'continues/parsers';

// Parse source session
const parser = SessionParser.forTool('claude');
const session = await parser.parse(sessionPath);

// Generate handoff with custom config
const generator = new HandoffGenerator({
  preset: 'verbose',
  recentMessages: 20,
  shell: { maxSamples: 15 }
});

const handoffDoc = generator.generate(session, {
  sourceId: 'abc123',
  sourceTool: 'claude',
  targetTool: 'gemini'
});

// Write handoff file
await fs.writeFile('handoff.md', handoffDoc);
```

### TypeScript: Adding a custom parser

```typescript
// src/parsers/mytool.ts
import type { SessionData, BaseParser } from '../types';

export class MyToolParser implements BaseParser {
  readonly name = 'mytool';
  readonly format = 'json';
  
  getSessionPaths(): string[] {
    return [`${process.env.HOME}/.mytool/sessions/*.json`];
  }
  
  async parse(filePath: string): Promise<SessionData> {
    const raw = JSON.parse(await fs.readFile(filePath, 'utf-8'));
    
    return {
      id: raw.id,
      source: 'mytool',
      projectPath: raw.workspace,
      timestamp: new Date(raw.created_at),
      messages: raw.chat.map(msg => ({
        role: msg.role,
        content: msg.text,
        timestamp: new Date(msg.ts)
      })),
      files: raw.files.map(f => ({
        path: f.path,
        operation: f.op,
        content: f.content
      })),
      shell: raw.commands.map(cmd => ({
        command: cmd.exec,
        output: cmd.stdout,
        exitCode: cmd.exit,
        timestamp: new Date(cmd.ts)
      })),
      metadata: {
        model: raw.model,
        tokensIn: raw.usage?.input,
        tokensOut: raw.usage?.output
      }
    };
  }
}
```

```typescript
// src/parsers/registry.ts
import { MyToolParser } from './mytool';

export const PARSERS = {
  // ... existing parsers
  mytool: MyToolParser,
} as const;
```

```typescript
// src/types/tool-names.ts
export const TOOL_NAMES = [
  'claude', 'codex', 'cursor', 'gemini', 'copilot',
  // ... other tools
  'mytool'
] as const;
```

## Session storage locations

Where `continues` looks for session data:

```typescript
// Claude Code
~/.claude/projects/*/session.jsonl

// Codex
~/.codex/sessions/*.jsonl

// Cursor
~/.cursor/projects/*/agent-transcripts/*.jsonl

// Gemini CLI
~/.gemini/tmp/*/chats/*.json

// Copilot
~/.copilot/session-state/*.{yaml,jsonl}

// OpenCode (SQLite)
~/.local/share/opencode/storage/sessions.db

// Crush (SQLite)
~/.crush/crush.db

// Amp
~/.local/share/amp/threads/*.json

// Kiro
~/Library/Application Support/Kiro/workspace-sessions/*.json

// Cline (VS Code extension)
<vscode-data>/globalStorage/saoudrizwan.claude-dev/tasks/*/

// Roo Code (VS Code extension)
<vscode-data>/globalStorage/rooveterinaryinc.roo-cline/tasks/*/

// Kilo Code (VS Code extension)
<vscode-data>/globalStorage/kilocode.kilo-code/tasks/*/
```

All reads are **read-only** — `continues` never modifies source session files.

## Common patterns

### Rate limit recovery workflow

```bash
# Hit Claude rate limit mid-session
# Switch to Gemini with full context
continues resume $(continues list --source claude -n 1 --json | jq -r '.[0].id') --in gemini --preset verbose

# Or use interactive picker
continues
# Select your Claude session, choose Gemini as target
```

### Daily session backup

```bash
#!/bin/bash
# Backup all sessions daily
DATE=$(date +%Y-%m-%d)
continues dump all "./backups/$DATE" --preset full --json
```

### Project-specific handoff script

```bash
# .continues.yml in your project
preset: verbose
recentMessages: 25
shell:
  maxSamples: 15

# Quick handoff script
#!/bin/bash
# switch-to-codex.sh
LAST_SESSION=$(continues list -n 1 --json | jq -r '.[0].id')
continues resume "$LAST_SESSION" --in codex --yolo
```

### Cross-tool testing

```bash
# Start in Claude
claude start "Implement auth middleware"

# After some work, transfer to Cursor to test
continues resume $(continues list --source claude -n 1 --json | jq -r '.[0].id') --in cursor

# Verify in Gemini
continues resume $(continues list --source cursor -n 1 --json | jq -r '.[0].id') --in gemini --debug-prompt
```

## What gets transferred

Every handoff document includes:

- **Recent messages**: User/assistant conversation (3-50 messages based on preset)
- **File changes**: Edits, reads, writes with full paths
- **Tool activity**: Shell commands with output/exit codes, grep/glob results, MCP calls
- **Key decisions**: Architectural notes, debugging insights from thinking blocks
- **Session metadata**: Model, tokens, cache usage, timestamps
- **Original session path**: Full file path for traceability

Example handoff structure:

```markdown
## Session Context
**Source**: claude (session: abc123)
**Project**: ~/dev/my-app
**Started**: 2026-02-19 05:28
**Model**: claude-sonnet-4
**Tokens**: 45,230 in / 12,847 out

## Recent Conversation
[Last 10 messages with timestamps and roles]

## Tool Activity
- **Bash** (×47): `npm test → exit 0`, `git status → exit 0`
- **Edit** (×12): `edit src/auth.ts`, `edit tests/auth.test.ts`
- **Grep** (×8): `grep "handleLogin" src/`

## Key Decisions
- Need to handle token refresh race with logout
- Using JWT short-lived tokens (15min) + refresh tokens
- Middleware order: auth → rate-limit → routes

## Files Modified
- src/auth.ts (edited)
- src/api/routes.ts (edited)
- tests/auth.test.ts (created)
```

## Troubleshooting

### No sessions found

```bash
# Check discovery
continues scan

# Verify tool installation
which claude codex cursor gemini

# Force index rebuild
continues rebuild
```

### Session parsing fails

```bash
# Inspect raw session
continues inspect abc123 --verbose

# Check for corrupt files
file ~/.claude/projects/abc123/session.jsonl

# Try with minimal preset
continues resume abc123 --in gemini --preset minimal
```

### Target tool doesn't receive context

```bash
# Debug the handoff prompt
continues resume abc123 --in cursor --debug-prompt

# Check handoff file location
ls -la ~/.continues/handoffs/

# Verify target tool accepts stdin
echo "test" | cursor
```

### VS Code extension sessions not found

```bash
# Set XDG_DATA_HOME if VS Code uses custom location
export XDG_DATA_HOME="$HOME/.config"

# On macOS, Cline/Roo/Kilo default to:
~/Library/Application Support/Code/User/globalStorage/

# Force rescan
continues rebuild
```

### Symlink directories skipped

The scanner now handles symlinks correctly, but if you have custom symlinked session directories:

```bash
# Verify symlink target is readable
ls -la ~/.claude/projects
readlink ~/.claude/projects

# Check permissions
stat ~/.claude/projects
```

### Zero token counts showing

If sessions show "0 in / 0 out", the source tool doesn't track token usage in that session format. This is normal for some tools. Use `--verbose` to see other metadata:

```bash
continues inspect abc123 --verbose
```

## Global flags

```bash
--config <path>      # Custom config file path
--preset <name>      # Verbosity preset (minimal|standard|verbose|full)
--verbose            # Enable debug logging
--debug              # Extra diagnostics
--in <tool>          # Target tool for handoff
--debug-prompt       # Print handoff prompt without launching tool
```

## Session index

`continues` maintains a local index at `~/.continues/sessions.jsonl`:

- Auto-refreshes every 5 minutes
- Rebuilt with `continues rebuild` or `--rebuild` flag
- Stores session ID, source, path, timestamp, project path
- **Read-only** — never modifies source session files

## Development notes

When adding new tool support:

1. Create parser in `src/parsers/<tool>.ts` implementing `BaseParser`
2. Add tool name to `src/types/tool-names.ts`
3. Register in `src/parsers/registry.ts`
4. Compile-time completeness check ensures no missing parsers

The registry enforces that every tool in `TOOL_NAMES` has a corresponding parser entry.
