---
name: cli-continues-session-transfer
description: Transfer AI coding sessions between tools (Claude, Cursor, Copilot, Gemini, etc.) with full context
triggers:
  - continue this session in another AI tool
  - switch from Claude to Cursor with context
  - resume my Copilot session in Gemini
  - transfer this coding session to another assistant
  - hand off my AI session with full history
  - export my Claude session to Cursor
  - pick up where I left off in a different tool
  - migrate my coding context between AI tools
---

# cli-continues Session Transfer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`cli-continues` (aka `continues`) enables seamless session transfers between 16 different AI coding assistants. When you hit rate limits, want to try a different model, or need specialized capabilities from another tool, it extracts your full conversation history, file changes, tool activity, and working state from one AI assistant and injects it into another.

**Supported tools (any-to-any handoff):**
Claude Code · Codex · GitHub Copilot CLI · Gemini CLI · Cursor · Amp · Cline · Roo Code · Kilo Code · Kiro · Crush · OpenCode · Factory Droid · Antigravity · Kimi CLI · Qwen Code

## Installation

```bash
# Run without install
npx continues

# Or install globally
npm install -g continues

# Verify installation
continues scan
```

**Requirements:**
- Node.js 22.5+ (uses native `node:sqlite` for some parsers)
- At least one supported AI coding tool installed

## How Session Transfer Works

1. **Discovery** — Scans session directories for all 16 tools (`~/.claude/`, `~/.cursor/`, etc.)
2. **Parsing** — Reads each tool's native format (JSONL, JSON, SQLite, YAML)
3. **Extraction** — Pulls messages, file changes, shell commands, reasoning blocks
4. **Handoff** — Generates structured context document and injects into target tool

All reads are **read-only** — your original sessions are never modified.

## Interactive Session Selection

The default command launches an interactive TUI:

```bash
continues
```

Shows all sessions across all tools, prioritizing the current project directory:

```
┌  continues — pick up where you left off
│
│  Found 1842 sessions across 16 CLI tools
│
◆  Select a session
│  [claude]   2026-02-19 05:28  my-project    Debugging SSH tunnel config   84a36c5d
│  [copilot]  2026-02-19 04:41  my-project    Migrate presets from Electron c2f5974c
│  [codex]    2026-02-18 23:12  my-project    Fix OpenCode SQLite parser    a1e90b3f
│
◆  Continue in:
│  ○ Gemini   ○ Codex   ○ Amp   ○ Kiro   ...
```

## Quick Resume (Same Tool)

Resume recent sessions without the TUI picker:

```bash
# Resume latest session from each tool
continues claude        # Most recent Claude Code session
continues cursor        # Most recent Cursor session
continues codex         # Most recent Codex session
continues gemini        # Most recent Gemini CLI session
continues copilot       # Most recent Copilot CLI session

# Resume Nth most recent session
continues claude 3      # 3rd most recent Claude session
continues codex 5       # 5th most recent Codex session
```

This performs **native resume** — same tool, full history preservation.

## Cross-Tool Handoff

Transfer a session from one tool to another:

```bash
# Basic handoff
continues resume abc123 --in gemini

# Pass flags to destination tool
continues resume abc123 --in codex --yolo --search --add-dir /tmp

# Inspect handoff prompt without launching
continues resume abc123 --in cursor --debug-prompt
```

**Common flags mapped automatically:**
- `--model` → tool-specific model flag
- `--sandbox` → sandbox/isolation mode
- `--yolo` → auto-approve mode
- `--add-dir` → additional directory context

Unknown flags pass through to destination tool unchanged.

## Verbosity Control

Four presets control handoff detail level:

```typescript
// Preset configurations
const presets = {
  minimal: {
    recentMessages: 3,
    shell: { maxSamples: 0 },
    edit: { maxSamples: 0 },
    mcp: { maxSamples: 0 },
    subagent: { maxDetailChars: 0 }
  },
  standard: {
    recentMessages: 10,
    shell: { maxSamples: 5, stdoutLines: 5 },
    edit: { maxSamples: 5 },
    mcp: { maxSamples: 5 },
    subagent: { maxDetailChars: 500 }
  },
  verbose: {
    recentMessages: 20,
    shell: { maxSamples: 10, stdoutLines: 10 },
    edit: { maxSamples: 10 },
    mcp: { maxSamples: 10 },
    subagent: { maxDetailChars: 2000 }
  },
  full: {
    recentMessages: 50,
    shell: { maxSamples: -1 },
    edit: { maxSamples: -1 },
    mcp: { maxSamples: -1 },
    subagent: { maxDetailChars: -1 }
  }
};
```

```bash
# Use preset
continues resume abc123 --preset full --in cursor

# Or configure per-project
cat > .continues.yml <<EOF
preset: verbose
recentMessages: 15
shell:
  maxSamples: 10
  stdoutLines: 20
EOF
```

**Configuration resolution order:**
1. `--config <path>` CLI flag
2. `.continues.yml` in current directory
3. `~/.continues/config.yml`
4. `standard` preset (default)

## Listing and Inspecting Sessions

```bash
# List all sessions (table format)
continues list

# Filter by source tool
continues list --source claude

# JSON output
continues list --json
continues list --jsonl

# Limit results
continues list -n 10

# Inspect specific session
continues inspect abc123

# Export session to markdown
continues inspect abc123 --preset full --write-md handoff.md

# Compact view (truncate long values)
continues inspect abc123 --truncate 50
```

## Bulk Export

Export all sessions or specific tool's sessions:

```bash
# Export all sessions to markdown
continues dump all ./sessions

# Export specific tool
continues dump claude ./sessions/claude
continues dump gemini ./sessions/gemini

# Export as JSON
continues dump all ./sessions --json

# Control detail level
continues dump all ./sessions --preset full

# Limit number of sessions
continues dump all ./sessions --limit 50
```

Files named: `{source}_{id}.md` or `{source}_{id}.json`

## Session Data Sources

Each tool stores sessions differently:

```typescript
// Session storage locations
const toolPaths = {
  claude: '~/.claude/projects/',           // JSONL
  codex: '~/.codex/sessions/',             // JSONL
  copilot: '~/.copilot/session-state/',    // YAML + JSONL
  gemini: '~/.gemini/tmp/*/chats/',        // JSON
  cursor: '~/.cursor/projects/*/agent-transcripts/', // JSONL
  opencode: '~/.local/share/opencode/storage/', // SQLite
  crush: '~/.crush/crush.db',              // SQLite
  amp: '~/.local/share/amp/threads/',      // JSON
  kiro: '~/Library/Application Support/Kiro/workspace-sessions/', // JSON (macOS)
  antigravity: '~/.gemini/antigravity/',   // Protobuf + artifacts
  factory: '~/.factory/sessions/',         // JSONL + JSON
  cline: 'VS Code globalStorage/saoudrizwan.claude-dev/tasks/', // JSON
  roocode: 'VS Code globalStorage/rooveterinaryinc.roo-cline/tasks/', // JSON
  kilocode: 'VS Code globalStorage/kilocode.kilo-code/tasks/', // JSON
  kimi: '~/.kimi/sessions/',               // JSONL + JSON
  qwen: '~/.qwen/projects/*/chats/'        // JSONL
};
```

**Environment variable overrides:**

```bash
# Override default paths
export CLAUDE_CONFIG_DIR=~/custom/claude
export CODEX_HOME=~/custom/codex
export GEMINI_CLI_HOME=~/custom/gemini
export XDG_DATA_HOME=~/custom/data  # Affects OpenCode, Amp
```

## Handoff Document Structure

Generated handoff documents include:

```markdown
## Session Metadata
- **Source**: claude-code
- **Session ID**: 84a36c5d
- **Project**: my-project
- **Started**: 2026-02-19 05:28
- **Model**: claude-sonnet-4
- **Tokens**: 45,230 in / 12,847 out

## Recent Conversation
[Last 10-50 messages depending on preset]

## Tool Activity
- **Bash** (×47): `$ npm test → exit 0` · `$ git status → exit 0`
- **Edit** (×12): `edit src/auth.ts` · `edit src/api/routes.ts`
- **Grep** (×8): `grep "handleLogin" src/` · `grep "JWT_SECRET"`

## File Changes
- `src/auth.ts` (edited 3 times)
- `tests/auth.test.ts` (created)
- `package.json` (edited)

## Key Decisions
- Switched from bcrypt to argon2 for password hashing
- Added Redis session store
- Implemented JWT refresh token rotation

## Session Notes
💭 Need to handle edge case where token refresh races with logout
```

## Scripting and CI Usage

```bash
# Check for sessions programmatically
if continues list --source claude --json | jq -e '.[0]' > /dev/null; then
  echo "Claude sessions found"
fi

# Export latest session for archival
SESSION_ID=$(continues list --source claude -n 1 --json | jq -r '.[0].id')
continues inspect "$SESSION_ID" --write-md "archive/session-$(date +%Y%m%d).md"

# Rebuild session index (useful after manual file changes)
continues rebuild
```

## Discovery and Indexing

```bash
# Show discovery stats
continues scan

# Force rebuild index cache
continues scan --rebuild

# Index location: ~/.continues/sessions.jsonl (5-min TTL)
```

## Common Workflows

### Hit Rate Limit Mid-Session

```bash
# Interactive handoff
continues  # Select Claude session, choose Gemini as target

# Or direct handoff
continues claude --in gemini
```

### Resume Yesterday's Work

```bash
# List recent sessions from project
cd my-project
continues list -n 5

# Resume by ID
continues resume abc123
```

### Archive Completed Sessions

```bash
# Export all sessions to markdown
continues dump all ./session-archive --preset full

# Or specific tool
continues dump claude ./claude-sessions --limit 20
```

### Debug Handoff Issues

```bash
# See exact prompt without launching target
continues resume abc123 --in cursor --debug-prompt > handoff.txt

# Inspect session data
continues inspect abc123 --truncate 100
```

## Parser Development

Add support for a new tool:

```typescript
// src/parsers/mytool-parser.ts
import { ParsedSession } from '../types/session';

export function parseMyToolSessions(dataDir: string): ParsedSession[] {
  // Read tool's session format
  const sessions = readMyToolFormat(dataDir);
  
  // Convert to ParsedSession schema
  return sessions.map(s => ({
    id: s.sessionId,
    source: 'mytool',
    projectPath: s.workingDir,
    messages: s.history.map(m => ({
      role: m.sender === 'user' ? 'user' : 'assistant',
      content: m.text,
      timestamp: new Date(m.time)
    })),
    summary: {
      subject: s.title,
      model: s.modelName,
      tokensIn: s.usage?.input,
      tokensOut: s.usage?.output
    },
    toolActivity: extractToolActivity(s),
    metadata: {
      created: new Date(s.startTime),
      updated: new Date(s.lastUpdate),
      source: 'mytool',
      sessionFilePath: s.filePath
    }
  }));
}
```

Register in `src/parsers/registry.ts`:

```typescript
import { parseMyToolSessions } from './mytool-parser';

export const PARSERS: ParserRegistry = {
  // ... existing parsers
  mytool: {
    parse: parseMyToolSessions,
    getPaths: () => ['~/.mytool/sessions']
  }
};
```

Add to `src/types/tool-names.ts`:

```typescript
export const TOOL_NAMES = [
  // ... existing tools
  'mytool'
] as const;
```

## Troubleshooting

**No sessions found:**

```bash
# Check which tools are detected
continues scan

# Verify paths
ls -la ~/.claude/projects/
ls -la ~/.cursor/projects/

# Force rebuild index
continues rebuild
```

**Symlink directories not discovered:**

Fixed in recent versions. If you encounter this:

```bash
# Check if session dir is symlink
ls -la ~/.claude  # Should show -> actual/path

# Manually verify parser can read
node -e "require('./dist/parsers/claude-parser.js').parseClaudeSessions('~/.claude/projects')"
```

**Zero tokens showing:**

Some sessions don't track tokens — this is expected, not a bug.

**Cross-tool flags not working:**

Use `--debug-prompt` to see what flags get passed:

```bash
continues resume abc123 --in codex --my-flag --debug-prompt
```

**VS Code extension sessions not found:**

```bash
# VS Code extensions store in user data dir
# macOS: ~/Library/Application Support/Code/User/globalStorage/
# Linux: ~/.config/Code/User/globalStorage/
# Windows: %APPDATA%\Code\User\globalStorage\

# Check Cline sessions
ls ~/Library/Application\ Support/Code/User/globalStorage/saoudrizwan.claude-dev/tasks/
```

## Advanced Configuration

Full `.continues.yml` example:

```yaml
# Preset: minimal | standard | verbose | full
preset: verbose

# Override preset values
recentMessages: 15
maxKeyDecisions: 8

shell:
  maxSamples: 10
  stdoutLines: 20
  stderrLines: 10

edit:
  maxSamples: 10

grep:
  maxSamples: 8

mcp:
  maxSamples: 10

subagent:
  maxDetailChars: 2000

# Custom tool paths (optional)
paths:
  claude: ~/custom/claude
  cursor: ~/custom/cursor
```

## Performance Notes

- **Index cache**: 5-minute TTL, stored at `~/.continues/sessions.jsonl`
- **Scan time**: ~100-500ms for typical setups (thousands of sessions)
- **Memory**: Parsers stream large files, minimal memory footprint
- **Concurrency**: All parsers run in parallel during discovery

## Security Considerations

- All session reads are **read-only** — original files never modified
- Handoff documents may contain sensitive code/data — review before sharing
- API keys in session history are preserved as-is — sanitize if needed
- `--debug-prompt` writes to disk — ensure secure file permissions
