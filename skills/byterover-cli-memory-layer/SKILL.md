---
name: byterover-cli-memory-layer
description: ByteRover CLI (brv) - Persistent memory layer for AI coding agents with context trees, knowledge storage, and cloud sync
triggers:
  - "add context to byterover"
  - "query my project knowledge with brv"
  - "setup byterover memory for this codebase"
  - "sync context tree to byterover cloud"
  - "curate project knowledge with byterover"
  - "search byterover context memory"
  - "version control my context tree"
  - "share project context with my team"
---

# ByteRover CLI Memory Layer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

ByteRover CLI (`brv`) provides AI coding agents with persistent, structured memory. It creates a context tree for project knowledge, supports version control for context, syncs to the cloud, and enables sharing across tools and teammates. This skill teaches you to use ByteRover as a memory layer for autonomous coding workflows.

## What ByteRover Does

ByteRover gives AI agents:
- **Persistent memory** across sessions via a local context tree
- **Knowledge curation** — store decisions, patterns, and domain knowledge
- **Context retrieval** — semantic search over project knowledge
- **Version control** — branch, commit, merge, and sync context like code
- **Cloud sync** — share context across machines and teammates
- **Multi-LLM support** — works with 20+ providers (Anthropic, OpenAI, Google, etc.)
- **Agent tools** — 24 built-in tools for file ops, code exec, memory management
- **MCP integration** — Model Context Protocol server for agent interop

## Installation

### Shell Script (macOS & Linux)

No Node.js required:

```bash
curl -fsSL https://byterover.dev/install.sh | sh
```

### npm (All Platforms)

Requires Node.js >= 20:

```bash
npm install -g byterover-cli
```

### Verify Installation

```bash
brv --version
```

## Quick Start

### Initialize in a Project

Navigate to your project directory and start the REPL:

```bash
cd ~/my-project
brv
```

On first run, ByteRover auto-configures. You'll see an interactive REPL. Type `/` to list commands.

### Open the Web Dashboard

The primary UI is the web dashboard:

```bash
brv webui
```

This opens a browser interface for curating context, querying knowledge, reviewing changes, and managing sync.

## Core Commands

### Curate Context (Add Knowledge)

Add project knowledge from the REPL:

```
/curate "Authentication uses JWT with 24h expiry and refresh tokens" @src/auth/jwt.ts
```

Or from the CLI:

```bash
brv curate -m "API rate limiting: 100 req/min per IP, enforced in middleware" -f src/middleware/ratelimit.ts
```

Curate operations are **pending by default** and require review.

### Query Knowledge

Search the context tree:

```
/query How is authentication implemented?
```

Or:

```bash
brv query "What rate limiting rules are in place?"
```

### Review Pending Changes

List pending curate operations:

```bash
brv review pending
```

Approve a specific operation:

```bash
brv review approve <operation-id>
```

Reject an operation:

```bash
brv review reject <operation-id>
```

### View Curate History

```bash
brv curate view
```

### Check Status

```bash
brv status
```

Shows project location, daemon status, active model, and sync state.

## Version Control for Context

ByteRover supports Git-like version control for the context tree.

### Initialize Version Control

```bash
brv vc init
```

This creates `.brv/vc/` and sets up version tracking.

### Stage and Commit Changes

```bash
brv vc add .                          # Stage all changes
brv vc commit -m "Add auth context"   # Commit staged changes
```

### View Commit History

```bash
brv vc log
```

### Branching

```bash
brv vc branch feature-auth            # Create a branch
brv vc checkout feature-auth          # Switch to it
brv vc branch                         # List branches
```

### Merge Branches

```bash
brv vc checkout main
brv vc merge feature-auth
```

### Push and Pull from Cloud

Push commits to ByteRover Cloud:

```bash
brv vc push
```

Pull commits from cloud:

```bash
brv vc pull
```

Clone a shared space repository:

```bash
brv vc clone <space-id>
```

## Cloud Sync (Legacy)

The original push/pull commands create snapshots (not version-controlled):

```bash
brv push    # Snapshot context to cloud
brv pull    # Restore context from cloud
```

**Prefer `brv vc push` / `brv vc pull` for version-controlled sync.**

## LLM Provider Configuration

### List Available Providers

```bash
brv providers list
```

Supported: Anthropic, OpenAI, Google, Groq, Mistral, xAI, Cerebras, Cohere, DeepInfra, DeepSeek, OpenRouter, Perplexity, TogetherAI, Vercel, Minimax, Moonshot, GLM, OpenAI-Compatible, ByteRover.

### Connect a Provider

```bash
brv providers connect
```

Follow the prompts to enter your API key (stored securely in `.brv/config.json`).

Or set environment variables:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
```

### Switch Active Provider

```bash
brv providers switch
```

### List and Switch Models

```bash
brv model list
brv model switch
```

## Real-World Usage Patterns

### Pattern 1: Curate API Design Decisions

When you make a design decision, record it immediately:

```bash
brv curate -m "User endpoints: POST /users (create), GET /users/:id (read), PATCH /users/:id (update). All require JWT auth except public profile GET /users/:id/profile." -f src/routes/users.ts
```

Then query later:

```bash
brv query "What are the user endpoints and their auth requirements?"
```

### Pattern 2: Document Complex Business Logic

```bash
brv curate -m "Order processing: validate inventory -> reserve stock -> charge payment -> fulfill. Rollback if payment fails. Stock reservation expires after 15 minutes." -f src/services/orders.ts
```

### Pattern 3: Record Configuration Patterns

```bash
brv curate -m "Database: Postgres 14, connection pool max 20, timeout 30s. Migrations via Prisma. Replica for read-heavy queries in production." -f prisma/schema.prisma
```

### Pattern 4: Share Context with Team

After curating knowledge locally:

```bash
brv vc add .
brv vc commit -m "Add order processing and DB config context"
brv vc push
```

Teammate pulls:

```bash
brv vc pull
```

They now have the same context tree.

### Pattern 5: Query Before Implementing

Before writing new code, check existing patterns:

```bash
brv query "How do we handle error logging and monitoring?"
```

Use the answer to stay consistent with established patterns.

## Worktrees and Knowledge Sources

### What Are Worktrees?

A **worktree** is a subdirectory link to a parent project. It avoids creating nested `.brv/` directories.

**Use case:** Monorepo with multiple packages, each needs ByteRover but should share one context tree.

### Add a Worktree

From the project root:

```bash
brv worktree add ./packages/api
```

This creates `.brv` pointer file in `./packages/api` that redirects to the parent `.brv/`.

### List Worktrees

```bash
brv worktree list
```

### Remove a Worktree

```bash
brv worktree remove ./packages/api
```

### What Are Sources?

A **source** is a read-only reference to another project's knowledge.

**Use case:** Frontend app wants to query the backend's context tree without duplicating it.

### Add a Source

```bash
brv source add ../backend-project
```

Now `brv query` searches both local and source knowledge.

### List and Remove Sources

```bash
brv source list
brv source remove ../backend-project
```

## MCP Server (Model Context Protocol)

ByteRover implements MCP for agent interop.

### Start the MCP Server

```bash
brv mcp
```

This runs a JSON-RPC server that exposes ByteRover tools (curate, query, file ops, etc.) to MCP clients.

### Configure MCP Client

Example `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "byterover": {
      "command": "brv",
      "args": ["mcp"]
    }
  }
}
```

Claude Desktop can now call ByteRover tools directly.

## Hub and Connectors

ByteRover Hub hosts reusable skills and bundles.

### List Available Packages

```bash
brv hub list
```

### Install a Package

```bash
brv hub install <package-name>
```

### Add a Custom Registry

```bash
brv hub registry add my-registry https://registry.example.com
```

### List and Remove Registries

```bash
brv hub registry list
brv hub registry remove my-registry
```

### Connectors

Connectors extend ByteRover with integrations (e.g., GitHub, Slack).

```bash
brv connectors list
brv connectors install github-connector
```

## Configuration

ByteRover stores config in `.brv/config.json` at the project root.

### Typical Config Structure

```json
{
  "projectId": "abc123",
  "spaceId": "space-xyz",
  "providers": {
    "anthropic": {
      "apiKey": "sk-ant-..."
    },
    "openai": {
      "apiKey": "sk-..."
    }
  },
  "activeProvider": "anthropic",
  "activeModel": "claude-3-5-sonnet-20241022",
  "reviewEnabled": true
}
```

### Environment Variables

Override config with env vars:

```bash
export BRV_ANTHROPIC_API_KEY=sk-ant-...
export BRV_OPENAI_API_KEY=sk-...
export BRV_ACTIVE_PROVIDER=openai
export BRV_ACTIVE_MODEL=gpt-4o
```

### Enable/Disable Review Workflow

By default, curate operations are pending. Disable review to auto-approve:

```bash
# Edit .brv/config.json
{
  "reviewEnabled": false
}
```

Or via env:

```bash
export BRV_REVIEW_ENABLED=false
```

## Authentication to ByteRover Cloud

### Login

Get an API key from [app.byterover.dev/settings/keys](https://app.byterover.dev/settings/keys), then:

```bash
brv login
```

Enter your API key when prompted.

### Logout

```bash
brv logout
```

## Troubleshooting

### Daemon Not Running

If `brv status` shows "Daemon not running":

```bash
brv restart
```

### API Key Not Recognized

Ensure the key is set:

```bash
brv providers list  # Check if provider is connected
brv providers connect  # Re-enter API key
```

Or verify env var:

```bash
echo $ANTHROPIC_API_KEY
```

### Curate Operations Not Appearing

Check if review is enabled. If so, operations are pending:

```bash
brv review pending
brv review approve <operation-id>
```

### Query Returns No Results

- Ensure you've curated knowledge first
- Check that files are indexed: `brv status`
- Verify active model supports embeddings

### Version Control Conflicts

If `brv vc pull` fails due to conflicts:

```bash
brv vc status           # See conflicting files
# Manually resolve conflicts in .brv/vc/
brv vc add .
brv vc commit -m "Resolve merge conflicts"
```

### Push Fails (Not Authenticated)

```bash
brv login
brv vc push
```

### Worktree Not Recognized

Ensure `.brv` pointer file exists in the worktree directory and points to the correct parent:

```bash
cat packages/api/.brv
# Should contain: gitdir: ../../.brv
```

If missing, re-add:

```bash
brv worktree add ./packages/api
```

## TypeScript API Usage (Advanced)

ByteRover is primarily a CLI, but you can use its internal APIs in TypeScript projects.

### Install as Dependency

```bash
npm install byterover-cli
```

### Example: Programmatic Curate

```typescript
import { BrvClient } from 'byterover-cli';

const client = new BrvClient({
  projectRoot: process.cwd(),
});

await client.curate({
  message: "User service uses bcrypt for password hashing, cost factor 12",
  files: ['src/services/user.ts'],
});

console.log('Context curated');
```

### Example: Programmatic Query

```typescript
import { BrvClient } from 'byterover-cli';

const client = new BrvClient({
  projectRoot: process.cwd(),
});

const results = await client.query({
  query: "How is password hashing implemented?",
});

console.log(results.answer);
```

### Example: Start REPL Programmatically

```typescript
import { startRepl } from 'byterover-cli';

await startRepl({
  projectRoot: process.cwd(),
  provider: 'anthropic',
  model: 'claude-3-5-sonnet-20241022',
});
```

## Best Practices

1. **Curate Early and Often** — Don't wait until the end of a feature. Curate decisions as you make them.
2. **Use Specific File References** — Always attach relevant files to curate operations for better context linking.
3. **Enable Review in Team Settings** — Prevent accidental or low-quality context pollution.
4. **Commit Context Like Code** — Use `brv vc commit` with meaningful messages for traceability.
5. **Sync Regularly** — Push and pull context changes daily in team environments.
6. **Query Before Coding** — Check existing patterns and decisions before implementing new features.
7. **Use Worktrees for Monorepos** — Avoid nested `.brv/` directories by linking packages to a single root.
8. **Add Sources for Cross-Project References** — Link related projects to search their knowledge without duplication.
9. **Leverage MCP for Agent Interop** — Let other AI tools (Claude Desktop, etc.) access ByteRover's memory.
10. **Document Configuration in Context** — Curate environment setup, deployment steps, and config patterns so agents can reproduce them.

## Common Workflows

### Workflow 1: Solo Developer, Daily Usage

```bash
cd ~/my-project
brv                                   # Start REPL
# In REPL:
/curate "Redis caching: 1h TTL for user sessions, 5m for API responses" @src/cache.ts
/query "What are the caching rules?"
# Exit REPL
brv vc add .
brv vc commit -m "Add caching context"
brv vc push                           # Backup to cloud
```

### Workflow 2: Team Collaboration

Developer A:

```bash
brv curate -m "API versioning: v1 in /api/v1, v2 in /api/v2. Deprecate v1 endpoints after 6 months." -f src/routes/index.ts
brv review approve $(brv review pending --json | jq -r '.[0].id')
brv vc add .
brv vc commit -m "Document API versioning policy"
brv vc push
```

Developer B:

```bash
brv vc pull
brv query "What is the API versioning policy?"
# Gets the answer immediately
```

### Workflow 3: Onboarding New Team Members

New developer clones the repo and context:

```bash
git clone https://github.com/myteam/myproject.git
cd myproject
brv vc clone <space-id>               # Clone shared context
brv query "How do I set up the development environment?"
# ByteRover returns curated setup instructions
```

### Workflow 4: Multi-Project Reference

Frontend project references backend context:

```bash
cd ~/frontend-app
brv source add ../backend-api
brv query "What are the available API endpoints?"
# Searches both frontend and backend context
```

## Integration with AI Coding Agents

ByteRover works with 22+ AI coding agents. Here's how to integrate:

### Cursor

Add to `.cursorrules` or workspace settings:

```
When working on this project, use ByteRover CLI (brv) to query and curate context.
Before implementing features, run: brv query "relevant question"
After making architectural decisions, run: brv curate -m "decision" -f <files>
```

### Claude Code / Cline

In system prompt or project instructions:

```
This project uses ByteRover for persistent memory. Use `brv query` to search project knowledge before coding. Use `brv curate` to record important decisions and patterns.
```

### Windsurf

Add to project docs or cascade rules:

```
# ByteRover Memory Layer
- Query context: `brv query "<question>"`
- Add context: `brv curate -m "<knowledge>" -f <files>`
- Sync with team: `brv vc push` after curating
```

### MCP-Compatible Agents

Configure MCP client (e.g., Claude Desktop):

```json
{
  "mcpServers": {
    "byterover": {
      "command": "brv",
      "args": ["mcp"]
    }
  }
}
```

Agent can now call ByteRover tools directly without shell commands.

## Summary

ByteRover CLI is a memory layer for AI coding agents. It stores project knowledge in a version-controlled context tree, syncs to the cloud, and enables semantic search across team and project boundaries. Use `brv curate` to add knowledge, `brv query` to retrieve it, and `brv vc push/pull` to collaborate. Integrate with AI agents via shell commands or MCP for autonomous, context-aware coding workflows.
