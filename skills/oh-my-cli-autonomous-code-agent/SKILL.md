---
name: oh-my-cli-autonomous-code-agent
description: Use oh-my-cli to run an autonomous code agent that can read/write files, execute shell commands, and manage multi-turn coding tasks with approval gates and session persistence
triggers:
  - "run an autonomous code agent on this codebase"
  - "use oh my cli to refactor these files"
  - "start a code agent session with file and shell tools"
  - "let the agent make changes to the project"
  - "resume my previous oh-my-cli session"
  - "undo the last agent turn"
  - "compact this long agent session"
  - "configure oh-my-cli with a different model"
---

# oh-my-cli Autonomous Code Agent

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

oh-my-cli is a minimal autonomous code agent CLI built with Qwen Code, Node.js 22, TypeScript, and ESM. It provides file manipulation and shell execution tools, session persistence, undo/redo capabilities, approval gates, and session compaction for long-running tasks.

## What It Does

- **Autonomous Code Agent**: Execute multi-turn coding tasks with an LLM that can read/write files and run shell commands
- **Session Management**: Persist sessions with JSONL checkpoints, resume from any point, browse previous sessions
- **Safety Controls**: Approval gates for mutations, atomic checkpoints, turn-based undo/redo
- **Context Management**: Automatic session compaction to stay within provider context limits
- **Profile Support**: Switch between multiple model configurations (hosted, local, different providers)
- **MCP Integration**: Model Context Protocol server support with health checks

## Installation

```bash
git clone https://github.com/qwen-code-dev-bot/oh-my-cli.git
cd oh-my-cli
npm install
npm run build
```

## Configuration

### Environment Variables

oh-my-cli uses OpenAI-compatible endpoints with these environment variables:

```bash
export OPENAI_API_KEY="your-key-here"
export OPENAI_BASE_URL="https://api.openai.com/v1"  # optional, defaults to OpenAI
export OPENAI_MODEL="gpt-4"  # required
```

### User Settings File

Create `~/.oh-my-cli/settings.json` to avoid exporting variables in every shell:

```json
{
  "model": {
    "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
    "name": "qwen-latest-series-invite-beta-v77",
    "apiKeyEnv": "DASHSCOPE_API_KEY"
  }
}
```

**Security**: The settings file never stores raw credentials—only the name of the environment variable that holds the key.

### Model Profiles

Manage multiple configurations in `~/.oh-my-cli/settings.json`:

```json
{
  "defaultProfile": "qwen",
  "profiles": {
    "qwen": {
      "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
      "name": "qwen-latest-series-invite-beta-v77",
      "apiKeyEnv": "DASHSCOPE_API_KEY",
      "description": "Hosted Qwen"
    },
    "local": {
      "name": "llama3",
      "baseUrl": "http://127.0.0.1:11434/v1",
      "apiKeyEnv": "OLLAMA_API_KEY"
    },
    "openai": {
      "name": "gpt-4",
      "apiKeyEnv": "OPENAI_API_KEY",
      "description": "OpenAI GPT-4"
    }
  }
}
```

## Key Commands

### Basic Usage

```bash
# Non-interactive: single task
oh-my-cli -p "List the files in this directory"

# Interactive REPL mode
oh-my-cli

# Use a specific profile
oh-my-cli --profile qwen -p "Refactor the auth module"

# Verify configuration before running
oh-my-cli --preflight --profile local
```

### Session Management

```bash
# Resume a session by ID
oh-my-cli --resume abc123-def456-789

# Browse sessions interactively (search, arrow keys, then resume)
oh-my-cli --browse-sessions

# List all profiles (read-only, redacted)
oh-my-cli --list-profiles
oh-my-cli --list-profiles --output json
```

### Session Compaction

Long sessions can exceed context limits. Compact them while preserving task state:

```bash
# Manually compact a session (writes sidecar, doesn't modify transcript)
oh-my-cli --compact abc123-def456-789

# Auto-compact during run when context pressure reaches threshold (in tokens)
oh-my-cli -p "Long refactoring task" --compact-threshold 100000

# Or via environment variable
OMC_COMPACT_THRESHOLD=100000 oh-my-cli -p "Long task"
```

Compaction creates a `<session-id>.compact.json` sidecar. On `--resume`, it's validated against the transcript and applied automatically.

### Undo/Redo Turns

Every non-interactive turn creates a checkpoint of modified files:

```bash
# Preview undo (dry-run, no changes)
oh-my-cli --undo-turn abc123-def456-789 --dry-run

# Undo the latest turn (restores files, trims transcript)
oh-my-cli --undo-turn abc123-def456-789

# Redo the undone turn
oh-my-cli --redo-turn abc123-def456-789

# JSON output for structured plan/receipt
oh-my-cli --undo-turn abc123-def456-789 --dry-run --output json
```

**Safety**: Undo/redo fail closed—they exit with status 2 and change nothing if a turn-owned file has diverged or is in a conflicted state.

### Session Export

Export a session to portable Markdown and JSON (redacted, local, no network):

```bash
# Export to ./exports directory
oh-my-cli --export-session abc123-def456-789 --out ./exports

# Force overwrite existing export
oh-my-cli --export-session abc123-def456-789 --out ./exports --force

# JSON output (includes manifest and output paths)
oh-my-cli --export-session abc123-def456-789 --out ./exports --output json
```

Creates two files:
- `<session-id>.session-export.md` — readable transcript
- `<session-id>.session-export.manifest.json` — machine-readable metadata

Exports are deterministic: re-exporting an unchanged session produces byte-identical files.

### Web Delivery Board

Browser-native feature pages for remote control and dynamic workflow:

```bash
# Start web server (loopback only, default port 4317)
oh-my-cli --delivery-web

# Use custom port
oh-my-cli --delivery-web --web-port 8080
```

Visit:
- `http://127.0.0.1:4317/remote-control`
- `http://127.0.0.1:4317/dynamic-workflow`

**Security**: Listens on loopback only, no credentials or settings exposed.

## Real-World Usage Patterns

### Pattern 1: Refactor a Module with Session Resume

```bash
# Start refactoring
oh-my-cli -p "Refactor src/auth.ts to use async/await instead of callbacks"

# Session completes, prints session ID: abc123-def456-789
# Later: resume if you need to continue
oh-my-cli --resume abc123-def456-789
# In REPL: "Now add error handling to the refactored functions"
```

### Pattern 2: Safe Experimentation with Undo

```bash
# Try a risky change
oh-my-cli -p "Rewrite the database layer to use Prisma"
# Session ID: xyz789-abc123-456

# Preview what undo would do
oh-my-cli --undo-turn xyz789-abc123-456 --dry-run

# Actually undo if it didn't work
oh-my-cli --undo-turn xyz789-abc123-456

# Or redo if you changed your mind
oh-my-cli --redo-turn xyz789-abc123-456
```

### Pattern 3: Multi-Profile Workflow

```bash
# Use fast local model for exploration
oh-my-cli --profile local -p "Analyze the codebase structure"

# Switch to powerful hosted model for complex refactoring
oh-my-cli --profile qwen -p "Refactor the entire API layer to be type-safe"

# Verify a profile before using it
oh-my-cli --preflight --profile openai
```

### Pattern 4: Long-Running Task with Compaction

```bash
# Set compaction threshold for a large codebase task
export OMC_COMPACT_THRESHOLD=100000
oh-my-cli -p "Migrate all 50 components from class-based to functional with hooks"

# Session auto-compacts when context pressure hits threshold
# Continue in REPL or resume later with full task state preserved
```

### Pattern 5: Team Handoff via Export

```bash
# Export a session for review or handoff
oh-my-cli --export-session abc123-def456-789 --out ./session-exports

# Sends two files:
# - abc123-def456-789.session-export.md (readable transcript)
# - abc123-def456-789.session-export.manifest.json (metadata)

# Colleague reviews the Markdown, sees exactly what the agent did
# Manifest includes tool call tallies, timestamps, workspace (redacted)
```

## Session Persistence

Sessions are stored as JSONL in `~/.oh-my-cli/sessions/`:

```
~/.oh-my-cli/sessions/
  abc123-def456-789.jsonl              # Main session transcript
  abc123-def456-789.compact.json       # Compaction sidecar (optional)
  abc123-def456-789.turn.json          # Undo/redo checkpoint (optional)
  abc123-def456-789.session-export.md  # Export output (if exported)
  abc123-def456-789.session-export.manifest.json
```

**Atomic Checkpoints**: Each non-interactive run writes to a temp file, then renames it over the canonical one—so an interrupted write leaves either the previous or new complete checkpoint, never a partial file.

## Tool Execution and Approval

oh-my-cli exposes file and shell tools to the model:

- **File tools**: `read_file`, `write_file`, `list_directory`, `create_directory`, etc.
- **Shell tools**: `execute_command` (with approval gates for mutations)

Approval gates prevent destructive actions without user confirmation. Configure approval behavior in settings or via CLI flags.

## Configuration Resolution Precedence

For each config field (highest to lowest):

| Field | 1 (highest) | 2 | 3 (lowest) |
|---|---|---|---|
| Base URL | `OPENAI_BASE_URL` | `model.baseUrl` or profile's `baseUrl` | Built-in default |
| Model name | `OPENAI_MODEL` | `model.name` or profile's `name` | *(required)* |
| Credential | `OPENAI_API_KEY` | env var named by `apiKeyEnv` | *(required)* |

**Security**: Project-local settings files are never auto-discovered. Only user-owned `~/.oh-my-cli/settings.json` or explicit `--settings <path>` are read.

## Troubleshooting

### Profile Not Found

```bash
oh-my-cli --profile unknown -p "Task"
# Error: Profile "unknown" not found
```

**Fix**: List available profiles:

```bash
oh-my-cli --list-profiles
```

### Missing API Key

```bash
oh-my-cli --profile qwen -p "Task"
# Error: Environment variable DASHSCOPE_API_KEY is not set
```

**Fix**: Export the referenced environment variable:

```bash
export DASHSCOPE_API_KEY="your-key-here"
```

### Session Checkpoint Corrupt

```bash
oh-my-cli --resume abc123-def456-789
# Warning: Checkpoint corrupt, using full transcript
```

Corrupt checkpoints are quarantined alongside the session (preserved, not deleted). The session resumes from the full transcript.

### Undo Failed (File Diverged)

```bash
oh-my-cli --undo-turn abc123-def456-789
# Error: Turn-owned file src/auth.ts has diverged since checkpoint
```

**Fix**: The file was modified after the turn. Undo fails closed to prevent data loss. Manually review and resolve, or skip undo.

### Context Limit Exceeded

```bash
oh-my-cli --resume long-session-id
# Error: Context limit exceeded
```

**Fix**: Compact the session:

```bash
oh-my-cli --compact long-session-id
oh-my-cli --resume long-session-id
```

Or set auto-compaction:

```bash
oh-my-cli --resume long-session-id --compact-threshold 100000
```

## Advanced: MCP Server Integration

oh-my-cli supports Model Context Protocol (MCP) servers. Configure them in `~/.oh-my-cli/settings.json`:

```json
{
  "model": {
    "name": "qwen-latest-series-invite-beta-v77",
    "apiKeyEnv": "DASHSCOPE_API_KEY"
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    }
  }
}
```

Check MCP server health:

```bash
oh-my-cli --health
```

## TypeScript Integration (for Extensions)

oh-my-cli is written in TypeScript with ESM. To build extensions:

```typescript
import { Session } from './session.js';
import { Tool } from './tools.js';

// Example: Custom tool implementation
export class MyCustomTool implements Tool {
  name = 'my_custom_tool';
  description = 'Does something custom';
  
  async execute(params: Record<string, unknown>): Promise<string> {
    // Your tool logic here
    return JSON.stringify({ result: 'success' });
  }
}
```

Build with:

```bash
npm run build
```

## Best Practices

1. **Use Profiles**: Set up profiles for different models/providers rather than constantly changing env vars
2. **Enable Auto-Compaction**: For long tasks, set `OMC_COMPACT_THRESHOLD` to prevent context limit errors
3. **Review Before Undo**: Always `--dry-run` before undoing to see what will change
4. **Export for Handoffs**: Use `--export-session` for team reviews or debugging
5. **Verify Configuration**: Run `--preflight` before starting complex tasks
6. **Browse Sessions**: Use `--browse-sessions` instead of remembering session IDs
7. **Never Store Raw Keys**: Always use `apiKeyEnv` to reference environment variables
8. **Atomic Operations**: Rely on oh-my-cli's atomic checkpoints—don't manually edit JSONL files

## Common Workflows

### Initial Setup

```bash
# Install
npm install && npm run build

# Create settings file
mkdir -p ~/.oh-my-cli
cat > ~/.oh-my-cli/settings.json << 'EOF'
{
  "defaultProfile": "qwen",
  "profiles": {
    "qwen": {
      "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
      "name": "qwen-latest-series-invite-beta-v77",
      "apiKeyEnv": "DASHSCOPE_API_KEY"
    }
  }
}
EOF

# Set API key
export DASHSCOPE_API_KEY="your-key-here"

# Verify
oh-my-cli --preflight
```

### Daily Usage

```bash
# Quick one-off task
oh-my-cli -p "Add types to src/utils.ts"

# Long interactive session
oh-my-cli
# > "Start by analyzing the codebase"
# > "Now refactor the components"
# > "Add tests for the new code"

# Resume yesterday's work
oh-my-cli --browse-sessions
# Arrow to the session, press Enter to resume
```

### Recovery from Mistakes

```bash
# Agent made a bad change
oh-my-cli --undo-turn <session-id> --dry-run  # Preview
oh-my-cli --undo-turn <session-id>            # Execute

# Export for debugging
oh-my-cli --export-session <session-id> --out ./debug-exports
```
