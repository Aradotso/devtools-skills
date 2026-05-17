---
name: cc-switch-cli
description: Cross-platform CLI tool for managing Claude Code, Codex, Gemini, OpenCode & OpenClaw providers, MCP servers, prompts, skills, and proxies.
triggers:
  - "switch my Claude provider"
  - "manage MCP servers"
  - "add a new API provider"
  - "sync my prompts across coding assistants"
  - "configure WebDAV sync for cc-switch"
  - "check Claude stream health"
  - "install community skills"
  - "set up local proxy routes"
---

# cc-switch-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

cc-switch-cli is a Rust-based command-line tool for unified management of AI coding assistant configurations. It handles provider switching, MCP server orchestration, prompt presets, community skills, and local proxy routing across Claude Code, Codex, Gemini, OpenCode, and OpenClaw.

## Installation

### Quick Install (macOS/Linux)

```bash
curl -fsSL https://github.com/SaladDay/cc-switch-cli/releases/latest/download/install.sh | bash
```

Installs to `~/.local/bin` by default. Override with `CC_SWITCH_INSTALL_DIR`:

```bash
CC_SWITCH_INSTALL_DIR=/usr/local/bin curl -fsSL https://github.com/SaladDay/cc-switch-cli/releases/latest/download/install.sh | bash
```

Force overwrite existing installation:

```bash
CC_SWITCH_FORCE=1 curl -fsSL https://github.com/SaladDay/cc-switch-cli/releases/latest/download/install.sh | bash
```

### Manual Installation

**macOS (Universal Binary):**

```bash
curl -LO https://github.com/saladday/cc-switch-cli/releases/latest/download/cc-switch-cli-darwin-universal.tar.gz
tar -xzf cc-switch-cli-darwin-universal.tar.gz
chmod +x cc-switch
sudo mv cc-switch /usr/local/bin/
xattr -cr /usr/local/bin/cc-switch  # Remove quarantine attribute
```

**Linux (x64 musl):**

```bash
curl -LO https://github.com/saladday/cc-switch-cli/releases/latest/download/cc-switch-cli-linux-x64-musl.tar.gz
tar -xzf cc-switch-cli-linux-x64-musl.tar.gz
chmod +x cc-switch
sudo mv cc-switch /usr/local/bin/
```

**Windows:**

Download `cc-switch-cli-windows-x64.zip`, extract, and move `cc-switch.exe` to a directory in `PATH` or run directly.

### Build from Source

Requires Rust 1.85+:

```bash
git clone https://github.com/saladday/cc-switch-cli.git
cd cc-switch-cli/src-tauri
cargo build --release
sudo cp target/release/cc-switch /usr/local/bin/
```

## Core Concepts

### Multi-App Architecture

cc-switch supports five apps via the global `--app` flag:

- `claude` (default) — Claude Code
- `codex` — Cursor's Codex
- `gemini` — Gemini CLI
- `opencode` — OpenCode
- `openclaw` — OpenClaw

Example:

```bash
cc-switch --app codex provider list
cc-switch --app gemini prompts activate custom-preset
```

### Interactive vs. Command Mode

**Interactive Mode (TUI):**

```bash
cc-switch
```

Navigate with arrow keys, select features, view live status.

**Command Mode:**

```bash
cc-switch provider switch pro-account
cc-switch mcp sync
```

## Provider Management

Providers store API keys, base URLs, and model configurations.

### List All Providers

```bash
cc-switch provider list
cc-switch --app codex provider list  # Codex-specific
```

### Show Current Provider

```bash
cc-switch provider current
```

### Switch Provider

```bash
cc-switch provider switch dev-api
```

Activates the provider named `dev-api` for the current app.

### Add a New Provider

Interactive prompt:

```bash
cc-switch provider add
```

Prompts for:
- Provider name
- Base URL
- API key (or `$ENV_VAR` reference)
- Model ID

### Edit Existing Provider

```bash
cc-switch provider edit prod-account
```

### Duplicate Provider

```bash
cc-switch provider duplicate staging-api
```

Creates a copy with `-copy` suffix.

### Delete Provider

```bash
cc-switch provider delete old-endpoint
```

### Export Claude Provider Settings

Export to `./.claude/settings.local.json` for Claude Code auto-load:

```bash
cc-switch provider export my-provider
```

Custom output path:

```bash
cc-switch provider export my-provider --output ~/.claude/settings-prod.json
```

### Speed Test

```bash
cc-switch provider speedtest my-provider
```

Measures API latency.

### Stream Health Check

```bash
cc-switch provider stream-check my-provider
```

Validates streaming endpoint responsiveness.

### Fetch Remote Models

```bash
cc-switch provider fetch-models my-provider
```

Retrieves available model list from the API endpoint.

## MCP Server Management

Model Context Protocol servers extend AI assistant capabilities.

### List MCP Servers

```bash
cc-switch mcp list
cc-switch --app codex mcp list
```

### Add MCP Server

Interactive:

```bash
cc-switch mcp add
```

Prompts for:
- Server name
- Transport type (stdio/http/sse)
- Command/URL
- Arguments/headers
- Environment variables
- Enabled apps

### Edit MCP Server

```bash
cc-switch mcp edit filesystem-server
```

### Delete MCP Server

```bash
cc-switch mcp delete old-server
```

### Enable/Disable for Specific App

```bash
cc-switch mcp enable postgres-mcp --app claude
cc-switch mcp disable postgres-mcp --app codex
```

### Validate Command in PATH

```bash
cc-switch mcp validate node
cc-switch mcp validate /usr/local/bin/custom-binary
```

Checks if executable exists and is accessible.

### Sync to Live Configs

```bash
cc-switch mcp sync
cc-switch --app gemini mcp sync
```

Writes enabled servers to app-specific config files (e.g., Claude's `claude_desktop_config.json`, Codex's `config.toml`).

### Import from Live Config

```bash
cc-switch mcp import --app claude
```

Reads existing MCP servers from the app's config file into cc-switch's database.

## Prompts Management

System prompt presets for AI coding assistants.

### List Prompts

```bash
cc-switch prompts list
cc-switch --app gemini prompts list
```

### Show Current Active Prompt

```bash
cc-switch prompts current
```

### Activate a Prompt

```bash
cc-switch prompts activate rust-expert
```

Writes content to the app's prompt file (e.g., `.claude/CLAUDE.md`, `.codex/AGENTS.md`).

### Deactivate Current Prompt

```bash
cc-switch prompts deactivate
```

Clears the active prompt file.

### Create New Prompt

Interactive:

```bash
cc-switch prompts create
```

Or provide name upfront:

```bash
cc-switch prompts create typescript-tutor
```

Opens default editor (`$EDITOR` or `vi`) to compose content.

### Rename Prompt

```bash
cc-switch prompts rename old-name new-name
```

Interactive if new name omitted:

```bash
cc-switch prompts rename old-name
```

### Edit Prompt

```bash
cc-switch prompts edit rust-expert
```

Opens in editor.

### Show Full Prompt Content

```bash
cc-switch prompts show rust-expert
```

Displays content in terminal.

### Delete Prompt

```bash
cc-switch prompts delete outdated-preset
```

## Skills Management

Community skills extend AI assistant capabilities via custom tooling.

### List Installed Skills

```bash
cc-switch skills list
cc-switch --app codex skills list
```

### Discover Available Skills

```bash
cc-switch skills discover database
cc-switch skills discover postgres
```

Searches the SSOT skills repository.

### Install Skill

```bash
cc-switch skills install postgres-expert
```

Downloads and registers the skill.

### Uninstall Skill

```bash
cc-switch skills uninstall postgres-expert
```

Removes from all apps and deletes local copy.

### Enable/Disable for App

```bash
cc-switch skills enable postgres-expert --app claude
cc-switch skills disable postgres-expert --app codex
```

### Scan Unmanaged Skills

```bash
cc-switch skills scan
```

Detects manually placed skill files in app directories.

### Import Unmanaged Skill

```bash
cc-switch skills import custom-tool --app claude
```

Registers existing skill file into cc-switch database.

### Show Skill Content

```bash
cc-switch skills show postgres-expert
```

### Sync Skills to Apps

```bash
cc-switch skills sync
```

Copies enabled skills to each app's `.skills/` directory.

### Refresh Repository

```bash
cc-switch skills refresh-repo
```

Updates local cache of available skills.

## Proxy Management

Local reverse proxy for multi-provider routing.

### Show Proxy Configuration

```bash
cc-switch proxy show
```

Displays routes, target endpoints, and proxy status.

### Start Proxy Server

```bash
cc-switch proxy start
```

Launches on configured port (default: `8080`).

### Stop Proxy Server

```bash
cc-switch proxy stop
```

### Add Route

Interactive:

```bash
cc-switch proxy add-route
```

Prompts for path prefix and target provider.

### Remove Route

```bash
cc-switch proxy remove-route /api/claude
```

## WebDAV Sync

Synchronize cc-switch data across machines.

### Show WebDAV Configuration

```bash
cc-switch config webdav show
```

### Configure WebDAV

Interactive setup:

```bash
cc-switch config webdav setup
```

Prompts for:
- WebDAV URL
- Username
- Password
- Sync directory

### Enable/Disable WebDAV Sync

```bash
cc-switch config webdav enable
cc-switch config webdav disable
```

### Trigger Manual Sync

```bash
cc-switch config webdav sync
```

Uploads local changes and downloads remote updates.

## Environment Diagnostics

### Check Local CLI Tools

```bash
cc-switch env tools
```

Validates presence of:
- Node.js
- Python
- npx
- uv
- Common shells

### Validate Installation

```bash
cc-switch env validate
```

Runs comprehensive checks on config files, permissions, and app directories.

## Configuration Files

cc-switch stores data in:

- **macOS/Linux:** `~/.config/cc-switch/`
- **Windows:** `%APPDATA%\cc-switch\`

Key files:

- `config.json` — Main settings
- `providers.json` — API providers
- `mcp_servers.json` — MCP server definitions
- `prompts.json` — Prompt presets
- `skills.json` — Skill metadata

App-specific files:

- **Claude:** `~/.claude/claude_desktop_config.json`, `~/.claude/CLAUDE.md`
- **Codex:** `~/.codex/config.toml`, `~/.codex/AGENTS.md`
- **Gemini:** `~/.gemini/gemini.toml`, `~/.gemini/GEMINI.md`
- **OpenCode:** `~/.opencode/config.toml`, `~/.opencode/AGENTS.md`
- **OpenClaw:** `~/.openclaw/config.toml`, `~/.openclaw/AGENTS.md`

## Common Workflows

### Switching Between Development and Production APIs

```bash
# List providers
cc-switch provider list

# Switch to production
cc-switch provider switch prod-api

# Verify
cc-switch provider current
```

### Setting Up MCP Servers for Claude

```bash
# Add filesystem MCP
cc-switch mcp add
# Select stdio, command: npx, args: -y @modelcontextprotocol/server-filesystem ~/projects

# Enable for Claude
cc-switch mcp enable filesystem-server --app claude

# Sync to live config
cc-switch mcp sync
```

### Activating a Custom Prompt Preset

```bash
# Create new prompt
cc-switch prompts create senior-rust-dev
# Edit content in $EDITOR

# Activate for Claude
cc-switch --app claude prompts activate senior-rust-dev

# Verify
cat ~/.claude/CLAUDE.md
```

### Installing Community Skills

```bash
# Discover available skills
cc-switch skills discover postgres

# Install
cc-switch skills install postgres-mcp

# Enable for Codex
cc-switch skills enable postgres-mcp --app codex

# Sync
cc-switch skills sync
```

### Exporting Provider for Standalone Use

```bash
# Export current provider to Claude auto-load location
cc-switch provider export my-provider

# Or custom path
cc-switch provider export my-provider --output ~/Desktop/claude-config.json
```

## Troubleshooting

### Provider Switch Not Taking Effect

Ensure sync has run:

```bash
cc-switch mcp sync
cc-switch prompts activate <current-prompt>
```

Restart the AI assistant app.

### MCP Server Not Showing Up

Check PATH for the command:

```bash
cc-switch mcp validate npx
```

Verify enabled apps:

```bash
cc-switch mcp list
```

Re-sync:

```bash
cc-switch mcp sync
```

### Prompt File Not Updated

Confirm prompt is activated:

```bash
cc-switch prompts current
```

Manually activate:

```bash
cc-switch prompts activate <name>
```

Check file exists:

```bash
cat ~/.claude/CLAUDE.md
```

### Skill Not Available in Assistant

Ensure enabled for correct app:

```bash
cc-switch skills enable <skill-name> --app claude
```

Sync skills:

```bash
cc-switch skills sync
```

Restart assistant.

### WebDAV Sync Failing

Verify credentials:

```bash
cc-switch config webdav show
```

Test connection:

```bash
cc-switch config webdav sync
```

Check network/firewall settings.

### Permission Errors

Ensure config directory is writable:

```bash
ls -la ~/.config/cc-switch/
chmod -R u+w ~/.config/cc-switch/
```

On macOS, grant Full Disk Access to Terminal in System Preferences > Privacy & Security.

## Environment Variables

- `CC_SWITCH_INSTALL_DIR` — Install location (default: `~/.local/bin`)
- `CC_SWITCH_FORCE` — Force overwrite during install (`1` to enable)
- `CC_SWITCH_LINUX_LIBC` — Linux libc variant (`glibc` or `musl`, default: `musl`)
- `EDITOR` — Text editor for prompt/skill editing (default: `vi`)

## Example: Multi-App Provider Workflow

```bash
# Add a shared API provider
cc-switch provider add
# Name: shared-api
# Base URL: https://api.example.com/v1
# API key: $MY_API_KEY
# Model: claude-3-5-sonnet-20241022

# Switch Claude to this provider
cc-switch --app claude provider switch shared-api

# Switch Codex to the same provider
cc-switch --app codex provider switch shared-api

# Verify both
cc-switch --app claude provider current
cc-switch --app codex provider current
```

## Example: Creating a Custom Skill

```bash
# Create new skill
cc-switch skills create --name custom-debugger

# Edit in $EDITOR, add content:
# ---
# name: custom-debugger
# description: Advanced debugging patterns for Rust
# triggers:
#   - "debug this Rust code"
#   - "add debugging instrumentation"
# ---
# # Custom Debugger Skill
# Use `dbg!()` for quick variable inspection...

# Save and exit editor

# Enable for Claude and Codex
cc-switch skills enable custom-debugger --app claude
cc-switch skills enable custom-debugger --app codex

# Sync
cc-switch skills sync
```

## Example: Testing API Health

```bash
# Speed test
cc-switch provider speedtest prod-api

# Stream health check
cc-switch provider stream-check prod-api

# Fetch available models
cc-switch provider fetch-models prod-api
```

## Integration with CI/CD

Export provider config for automated setups:

```bash
#!/bin/bash
# ci-setup.sh

# Install cc-switch
curl -fsSL https://github.com/SaladDay/cc-switch-cli/releases/latest/download/install.sh | bash

# Add CI provider
cc-switch provider add \
  --name ci-api \
  --base-url "$CI_API_URL" \
  --api-key "$CI_API_KEY" \
  --model claude-3-5-sonnet-20241022

# Switch to it
cc-switch provider switch ci-api

# Export for downstream tools
cc-switch provider export ci-api --output ./claude-settings.json
```

This skill enables AI coding agents to help developers install, configure, and operate cc-switch-cli across all supported AI assistant platforms with real command examples and practical workflows.
