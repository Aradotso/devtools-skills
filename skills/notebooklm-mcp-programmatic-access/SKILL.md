---
name: notebooklm-mcp-programmatic-access
description: Programmatic access to Google NotebookLM via CLI and MCP server for AI-powered research, podcast generation, and notebook management
triggers:
  - "use notebooklm to create a research notebook"
  - "generate a podcast from my sources using notebooklm"
  - "add documents to a notebooklm notebook"
  - "query my notebooklm notebook about"
  - "create studio content with notebooklm"
  - "set up notebooklm mcp server"
  - "authenticate with notebooklm"
  - "batch process notebooks in notebooklm"
---

# NotebookLM MCP & CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

**notebooklm-mcp-cli** provides programmatic access to Google NotebookLM through both a command-line interface (`nlm`) and Model Context Protocol (MCP) server (`notebooklm-mcp`). Use it to automate research workflows, generate AI podcasts, manage notebooks, and integrate NotebookLM into AI coding agents.

## What It Does

- **Notebook Management**: Create, list, delete, and share NotebookLM notebooks
- **Source Management**: Add sources from URLs, text, Google Drive, or local files
- **AI Studio**: Generate audio podcasts, video presentations, slide decks, and infographics
- **Query & Research**: Query notebooks, perform web/Drive research, cross-notebook queries
- **Automation**: Batch operations, pipelines, tagging, and smart selection
- **MCP Integration**: Connect Claude, Gemini, Cursor, and other AI tools to NotebookLM
- **Profile Support**: Manage multiple Google accounts with isolated browser sessions

## Installation

### Using uv (Recommended)

```bash
uv tool install notebooklm-mcp-cli
```

### Using pip

```bash
pip install notebooklm-mcp-cli
```

### Using pipx

```bash
pipx install notebooklm-mcp-cli
```

After installation, you get:
- `nlm` — CLI command
- `notebooklm-mcp` — MCP server executable

## Authentication

Before using, authenticate with your Google account:

```bash
# Auto mode: launches browser, extracts cookies automatically
nlm login

# Check authentication status
nlm login --check

# Use named profiles for multiple Google accounts
nlm login --profile work
nlm login --profile personal

# Switch default profile
nlm login switch personal

# List all profiles
nlm login profile list

# Manual mode: import cookies from file
nlm login --manual --file cookies.txt
```

**Profile Management:**
```bash
nlm login profile list                # Show all profiles with emails
nlm login profile delete work         # Delete a profile
nlm login profile rename old new      # Rename a profile
```

Each profile maintains its own browser session and cookies, allowing simultaneous use of multiple Google accounts.

## CLI Quick Start

### Basic Workflow

```bash
# List existing notebooks
nlm notebook list

# Create a new notebook
nlm notebook create "AI Research Project"

# Add sources (URL, text, Drive, or file)
nlm source add <notebook-id> --url "https://example.com/article"
nlm source add <notebook-id> --text "Raw text content here"
nlm source add <notebook-id> --drive "https://docs.google.com/document/d/..."
nlm source add <notebook-id> --file ./document.pdf

# Query the notebook
nlm notebook query <notebook-id> "What are the key findings?"

# Generate a podcast
nlm studio create <notebook-id> --type audio --confirm

# Download the audio file
nlm download audio <notebook-id> <artifact-id>

# Share notebook publicly
nlm share public <notebook-id>
```

### Studio Content Types

```bash
# Generate audio podcast
nlm studio create <notebook-id> --type audio --confirm

# Create video presentation
nlm studio create <notebook-id> --type video --confirm

# Generate slide deck
nlm studio create <notebook-id> --type slides --confirm

# Create infographic
nlm studio create <notebook-id> --type infographic --confirm

# Revise existing slide deck
nlm slides revise <notebook-id> <artifact-id> "Make it more technical"
```

### Advanced Features

```bash
# Batch query multiple notebooks
nlm batch query "What are the main themes?" --tag research

# Cross-notebook query
nlm cross query "Compare findings across all notebooks" --tag project-alpha

# Run a pipeline
nlm pipeline run research-workflow --input '{"topic": "quantum computing"}'

# Tag notebooks for organization
nlm tag add <notebook-id> research ai ml
nlm tag list
nlm tag select --tag research --limit 5

# Web and Drive research
nlm research start <notebook-id> --query "latest AI developments" --max-results 10
```

### Configuration

```bash
# Set preferred browser for authentication
nlm config set auth.browser brave

# Set default profile
nlm config set auth.default_profile work

# View all settings
nlm config list
```

## MCP Server Setup

### Automatic Configuration

```bash
# Add to Claude Code
nlm setup add claude-code

# Add to Claude Desktop
nlm setup add claude-desktop

# Add to Gemini CLI
nlm setup add gemini

# Add to Cursor
nlm setup add cursor

# Add to GitHub Copilot
nlm setup add github-copilot

# Add to Windsurf
nlm setup add windsurf

# Generate JSON for other tools
nlm setup add json

# List configured tools
nlm setup list

# Remove from a tool
nlm setup remove claude-code
```

### Manual Configuration

If automatic setup doesn't work, add to your MCP client's config:

**Claude Desktop/Code** (`~/Library/Application Support/Claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "notebooklm-mcp"
    }
  }
}
```

**Cursor** (`.cursor/mcp_config.json`):
```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "notebooklm-mcp"
    }
  }
}
```

## MCP Tools Reference

The MCP server provides 35 tools. Key tools include:

### Notebook Operations
- `notebook_list` — List all notebooks
- `notebook_create` — Create new notebook
- `notebook_delete` — Delete notebook
- `notebook_query` — Query notebook (persists to web UI)
- `notebook_get` — Get notebook details
- `notebook_share_public` — Enable public sharing
- `notebook_share_invite` — Share with specific users

### Source Management
- `source_add` — Add URL, text, Drive, or file source
- `source_list` — List sources in notebook
- `source_delete` — Remove source
- `source_sync_drive` — Sync Drive folder

### Studio Content
- `studio_create` — Generate audio, video, slides, or infographic
- `studio_revise` — Revise slide decks
- `download_artifact` — Download generated content

### Research
- `research_start` — Web/Drive research
- `research_status` — Check research progress
- `cross_notebook_query` — Query across multiple notebooks

### Batch & Automation
- `batch` — Batch operations (query, create, delete)
- `pipeline` — Multi-step workflows
- `tag` — Tag and organize notebooks

## Code Examples

### Python: Using the MCP Client

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

# Connect to the MCP server
server_params = StdioServerParameters(
    command="notebooklm-mcp",
    env=None
)

async with stdio_client(server_params) as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        
        # List notebooks
        result = await session.call_tool("notebook_list", {})
        print(result)
        
        # Create notebook
        result = await session.call_tool("notebook_create", {
            "title": "Research Project"
        })
        notebook_id = result["id"]
        
        # Add source
        await session.call_tool("source_add", {
            "notebook": notebook_id,
            "url": "https://example.com/article"
        })
        
        # Query
        result = await session.call_tool("notebook_query", {
            "notebook": notebook_id,
            "query": "Summarize the key points"
        })
        print(result)
        
        # Generate podcast
        result = await session.call_tool("studio_create", {
            "notebook": notebook_id,
            "content_type": "audio",
            "confirm": True
        })
        artifact_id = result["artifact_id"]
        
        # Download
        audio_data = await session.call_tool("download_artifact", {
            "notebook": notebook_id,
            "artifact_id": artifact_id,
            "artifact_type": "audio"
        })
```

### Shell Script: Automated Research Pipeline

```bash
#!/bin/bash

# Create notebook
NOTEBOOK_ID=$(nlm notebook create "Daily Research" | jq -r '.id')

# Add multiple sources
nlm source add "$NOTEBOOK_ID" --url "https://news.ycombinator.com"
nlm source add "$NOTEBOOK_ID" --url "https://arxiv.org/list/cs.AI/recent"

# Start web research
nlm research start "$NOTEBOOK_ID" --query "latest AI breakthroughs" --max-results 20

# Wait for research to complete
sleep 60

# Generate podcast
nlm studio create "$NOTEBOOK_ID" --type audio --confirm

# Get artifact ID
ARTIFACT_ID=$(nlm notebook get "$NOTEBOOK_ID" | jq -r '.artifacts[] | select(.type=="audio") | .id' | head -1)

# Download
nlm download audio "$NOTEBOOK_ID" "$ARTIFACT_ID" --output ./daily-brief.wav

# Share publicly
nlm share public "$NOTEBOOK_ID"
```

### Python: Batch Processing

```python
import subprocess
import json

# Tag notebooks for batch processing
notebooks = ["nb1", "nb2", "nb3"]
for nb in notebooks:
    subprocess.run(["nlm", "tag", "add", nb, "research", "2024"])

# Batch query
result = subprocess.run(
    ["nlm", "batch", "query", "What are the main themes?", "--tag", "research"],
    capture_output=True,
    text=True
)
batch_results = json.loads(result.stdout)

for nb_id, response in batch_results.items():
    print(f"Notebook {nb_id}:")
    print(response["answer"])
    print("---")
```

### Natural Language with MCP (Claude Code)

Once configured, use natural language:

```
User: Create a NotebookLM notebook about quantum computing, add sources from 
      arxiv.org and nature.com, then generate a podcast summarizing the key concepts.
