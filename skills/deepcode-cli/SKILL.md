---
name: deepcode-cli
description: Terminal AI coding assistant optimized for DeepSeek v4 with deep thinking, reasoning control, Agent Skills, and MCP integration
triggers:
  - how do I use deepcode cli
  - configure deepcode terminal assistant
  - set up deepcode skills
  - enable deepcode thinking mode
  - integrate deepcode with mcp
  - use deepcode in terminal
  - configure deepseek reasoning effort
  - create custom deepcode skills
---

# DeepCode CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

DeepCode CLI is a terminal AI coding assistant specifically optimized for DeepSeek v4 models. It supports deep thinking mode, reasoning effort control, extensible Agent Skills, and Model Context Protocol (MCP) integration for connecting external services.

## Installation

```bash
npm install -g @vegamo/deepcode-cli
```

Run `deepcode` in any project directory to start the assistant.

## Configuration

Create `~/.deepcode/settings.json`:

```json
{
  "env": {
    "MODEL": "deepseek-v4-pro",
    "BASE_URL": "https://api.deepseek.com",
    "API_KEY": "sk-..."
  },
  "thinkingEnabled": true,
  "reasoningEffort": "max"
}
```

### Configuration Priority

DeepCode follows a multi-level configuration priority system:

1. **Project-level** (highest): `./.deepcode/settings.json`
2. **User-level**: `~/.deepcode/settings.json`
3. **Environment variables**: `DEEPCODE_MODEL`, `DEEPCODE_BASE_URL`, `DEEPCODE_API_KEY`
4. **Default values** (lowest)

### Core Configuration Options

```json
{
  "env": {
    "MODEL": "deepseek-v4-pro",
    "BASE_URL": "https://api.deepseek.com",
    "API_KEY": "sk-..."
  },
  "thinkingEnabled": true,
  "reasoningEffort": "max",
  "temperature": 0.7,
  "maxTokens": 8192,
  "notify": "/path/to/notification-script.sh",
  "webSearchTool": "/path/to/web-search-script.sh",
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### Model Options

- `deepseek-v4-pro` (recommended for complex tasks)
- `deepseek-v4-flash` (faster, lower cost)
- Any OpenAI-compatible model

### Reasoning Effort Levels

- `"min"` - Fast, basic reasoning
- `"medium"` - Balanced (default)
- `"max"` - Deep thinking, slower but more thorough

## Key Commands

### Slash Commands

| Command | Description |
|---------|-------------|
| `/` | Open skills/command menu |
| `/new` | Start new conversation |
| `/resume` | Resume a previous conversation |
| `/model` | Switch model, thinking mode, and reasoning effort |
| `/init` | Initialize AGENTS.md file |
| `/skills` | List available skills |
| `/mcp` | View MCP server status and available tools |
| `/exit` | Exit (or press Ctrl+D twice) |

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Enter` | Send message |
| `Shift+Enter` or `Ctrl+J` | Insert newline |
| `Ctrl+V` | Paste image from clipboard |
| `Esc` | Interrupt current model response |
| `Ctrl+D` (twice) | Exit |

## Agent Skills

DeepCode CLI supports extensible skills to enhance the assistant's capabilities.

### User-level Skills

Place skills in `~/.agents/skills/`. These are available across all projects.

### Project-level Skills

Place skills in `./.agents/skills/` or `./.deepcode/skills/` for project-specific capabilities.

### Creating a Custom Skill

Create a `SKILL.md` file in the skills directory:

```markdown
---
name: my-custom-skill
description: Description of what this skill does
triggers:
  - when to use this skill
  - another trigger phrase
---

# My Custom Skill

Detailed instructions for the AI assistant on how to use this skill...

## Usage Example

\`\`\`typescript
// Example code showing how to use this skill
\`\`\`
```

### Example: Database Migration Skill

`./.agents/skills/migrations/SKILL.md`:

```markdown
---
name: database-migrations
description: Manage database schema migrations using Prisma
triggers:
  - create database migration
  - run migrations
  - rollback migration
  - check migration status
---

# Database Migrations

This skill helps manage database schema changes using Prisma.

## Create a Migration

\`\`\`bash
npx prisma migrate dev --name descriptive_migration_name
\`\`\`

## Apply Migrations in Production

\`\`\`bash
npx prisma migrate deploy
\`\`\`

## Check Migration Status

\`\`\`bash
npx prisma migrate status
\`\`\`

## Rollback (Manual)

Edit migration files in `prisma/migrations/` and create a new migration to undo changes.
```

## MCP Integration

DeepCode supports Model Context Protocol for connecting external services.

### Configuring MCP Servers

In `~/.deepcode/settings.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/directory"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

### View MCP Status

Use `/mcp` command in DeepCode to see:
- Connected MCP servers
- Available tools from each server
- Connection status

### Common MCP Servers

- **GitHub**: `@modelcontextprotocol/server-github` - Repository management, issues, PRs
- **Filesystem**: `@modelcontextprotocol/server-filesystem` - File operations
- **PostgreSQL**: `@modelcontextprotocol/server-postgres` - Database queries
- **Browser**: `@modelcontextprotocol/server-puppeteer` - Web automation
- **Slack**: `@modelcontextprotocol/server-slack` - Messaging integration

## Common Usage Patterns

### Starting a New Coding Session

```bash
cd my-project
deepcode
```

Type your request directly or use `/` to see available skills.

### Using Thinking Mode for Complex Problems

In `settings.json`:

```json
{
  "thinkingEnabled": true,
  "reasoningEffort": "max"
}
```

Or toggle during session with `/model` command.

### Pasting Code for Review

Simply paste code into the prompt and ask for review, optimization, or explanation.

### Multi-modal Input (Images)

1. Copy image to clipboard
2. Press `Ctrl+V` in DeepCode
3. Add your question about the image

**Note**: DeepSeek v4 doesn't support multi-modal. Use compatible models like `Doubao-Seed-2.0-pro` via Volcano Engine.

### Resuming Previous Conversation

Use `/resume` to select from conversation history and continue where you left off.

## Working with Different Model Providers

### Volcano Engine (Doubao)

```json
{
  "env": {
    "MODEL": "Doubao-Seed-2.0-pro",
    "BASE_URL": "https://ark.cn-beijing.volces.com/api/v3",
    "API_KEY": "${VOLCANO_API_KEY}"
  }
}
```

### Coding Plan (Volcano Engine)

```json
{
  "env": {
    "MODEL": "ark-code-latest",
    "BASE_URL": "https://ark.cn-beijing.volces.com/api/coding/v3",
    "API_KEY": "${VOLCANO_API_KEY}"
  },
  "thinkingEnabled": true
}
```

### OpenAI

```json
{
  "env": {
    "MODEL": "gpt-4",
    "BASE_URL": "https://api.openai.com/v1",
    "API_KEY": "${OPENAI_API_KEY}"
  }
}
```

## Notifications

Configure a notification script to alert you when long-running tasks complete:

```json
{
  "notify": "/usr/local/bin/notify-slack.sh"
}
```

Example notification script (`notify-slack.sh`):

```bash
#!/bin/bash
WEBHOOK_URL="${SLACK_WEBHOOK_URL}"
MESSAGE="DeepCode task completed!"

curl -X POST -H 'Content-type: application/json' \
  --data "{\"text\":\"$MESSAGE\"}" \
  "$WEBHOOK_URL"
```

## Web Search Integration

Enable web search capability:

```json
{
  "webSearchTool": "/usr/local/bin/web-search.sh"
}
```

DeepCode includes a built-in free web search tool. Custom scripts allow you to integrate specific search providers.

## Initializing Agent Configuration

Use `/init` to create an `AGENTS.md` file in your project. This file provides context to the AI about your project structure, conventions, and guidelines.

Example `AGENTS.md`:

```markdown
# Project Context for AI Agents

## Tech Stack
- TypeScript
- React
- Node.js
- PostgreSQL

## Code Conventions
- Use functional components
- Prefer async/await over promises
- All API routes in `src/api/`
- Tests in `__tests__/` directory

## Database
- Prisma ORM
- Migrations in `prisma/migrations/`

## Common Tasks
- Run tests: `npm test`
- Start dev server: `npm run dev`
- Build: `npm run build`
```

## Troubleshooting

### "API Key not configured"

Ensure `API_KEY` is set in `~/.deepcode/settings.json` or as environment variable `DEEPCODE_API_KEY`.

### Skills not loading

- Check skills are in `~/.agents/skills/` or `./.agents/skills/`
- Verify `SKILL.md` files have valid YAML frontmatter
- Use `/skills` to see loaded skills

### MCP server not connecting

- Run `/mcp` to see error messages
- Verify server package is installed: `npx -y @modelcontextprotocol/server-github`
- Check environment variables are set correctly
- Ensure `command` and `args` in config match server requirements

### Model responses interrupted

Press `Esc` to stop current response. If responses consistently fail, check:
- Model name is correct
- Base URL is accessible
- API key has sufficient credits
- Token limits (`maxTokens`) are appropriate

### Context cache not working

Context caching is automatic for DeepSeek models. To verify:
- Use `deepseek-v4-pro` or `deepseek-v4-flash`
- BASE_URL should be `https://api.deepseek.com`
- Check API dashboard for cache hit rates

### Configuration not taking effect

Remember priority order:
1. Project `./.deepcode/settings.json`
2. User `~/.deepcode/settings.json`
3. Environment variables
4. Defaults

Check which config file is being used and verify JSON syntax is valid.
