---
name: notebooklm-mcp-cli
description: Programmatic access to Google NotebookLM via CLI and MCP server for AI agents
triggers:
  - "create a notebooklm notebook"
  - "generate a podcast from these sources"
  - "add sources to notebooklm"
  - "query my notebooklm notebook"
  - "setup notebooklm mcp server"
  - "create studio content in notebooklm"
  - "download notebooklm audio"
  - "share a notebooklm notebook"
---

# NotebookLM MCP CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

NotebookLM MCP CLI provides programmatic access to Google NotebookLM through:
- **CLI (`nlm`)**: Direct terminal commands for scripting and automation
- **MCP Server (`notebooklm-mcp`)**: Model Context Protocol server for AI agents (Claude, Gemini, Cursor, etc.)

Both interfaces access the same NotebookLM API capabilities: notebook management, source addition, audio/video generation, research automation, and sharing.

**Important**: Uses undocumented internal APIs requiring browser cookie extraction. Supports NotebookLM Pro/free and Google AI Ultra accounts.

## Installation

```bash
# Recommended: Install with uv
uv tool install notebooklm-mcp-cli

# Alternative: pip
pip install notebooklm-mcp-cli

# Alternative: pipx
pipx install notebooklm-mcp-cli

# Verify installation
nlm --version
```

This installs both:
- `nlm` - CLI interface
- `notebooklm-mcp` - MCP server

## Authentication

### Initial Setup

```bash
# Auto mode: launches browser for cookie extraction
nlm login

# Check authentication status
nlm login --check

# Manual mode with cookie file
nlm login --manual --file cookies.txt
```

### Profile Management (Multiple Google Accounts)

```bash
# Create named profiles
nlm login --profile work
nlm login --profile personal

# Switch profiles
nlm login switch work

# List all profiles
nlm login profile list

# Delete a profile
nlm login profile delete personal
```

### Browser Selection

```bash
# Set preferred browser
nlm config set auth.browser brave

# Supported: chrome, arc, brave, edge, chromium, firefox
```

## CLI Usage

### Notebook Management

```python
# List all notebooks
nlm notebook list

# Create a new notebook
nlm notebook create "Research Project"

# Get notebook details
nlm notebook get <notebook-id>

# Delete a notebook
nlm notebook delete <notebook-id>

# Rename a notebook
nlm notebook rename <notebook-id> "New Name"
```

### Adding Sources

```python
# Add URL source
nlm source add <notebook-id> --url "https://example.com"

# Add text source
nlm source add <notebook-id> --text "Your content here" --title "Notes"

# Add Google Drive file
nlm source add <notebook-id> --drive-id "1ABC..." --title "Document"

# Add local file (uploads to Drive first)
nlm source add <notebook-id> --file ./document.pdf

# Add multiple sources at once
nlm source add <notebook-id> \
  --url "https://site1.com" \
  --url "https://site2.com" \
  --text "Summary notes"

# List sources in notebook
nlm source list <notebook-id>

# Delete a source
nlm source delete <notebook-id> <source-id>
```

### Querying Notebooks

```python
# Ask a question (persists to web UI)
nlm notebook query <notebook-id> "What are the key findings?"

# Query with custom settings
nlm notebook query <notebook-id> "Summarize this" \
  --grounding "Always cite sources" \
  --format-as markdown
```

### Studio Content Creation

```python
# Create audio podcast (Deep Dive)
nlm studio create <notebook-id> --type audio --confirm

# Create video presentation
nlm studio create <notebook-id> --type video --confirm

# Create slides
nlm studio create <notebook-id> --type slides --confirm

# Interactive mode (prompts for confirmation)
nlm studio create <notebook-id> --type audio

# List studio artifacts
nlm studio list <notebook-id>
```

### Revising Slides

```python
# Revise specific slides with instructions
nlm slides revise <notebook-id> <artifact-id> \
  --slides 1,3,5 \
  --instruction "Add more technical details and code examples"

# Revise all slides
nlm slides revise <notebook-id> <artifact-id> \
  --instruction "Make the tone more casual"

# Check revision status
nlm slides status <notebook-id> <revision-id>
```

### Downloading Artifacts

```python
# Download audio file
nlm download audio <notebook-id> <artifact-id> --output podcast.wav

# Download video
nlm download video <notebook-id> <artifact-id> --output presentation.mp4

# Download slides as PDF
nlm download slides <notebook-id> <artifact-id> --output deck.pdf

# Auto-generate filename
nlm download audio <notebook-id> <artifact-id>
```

### Sharing

```python
# Enable public link
nlm share public <notebook-id>

# Invite specific email
nlm share invite <notebook-id> user@example.com

# Get share status
nlm share status <notebook-id>

# Revoke public link
nlm share revoke <notebook-id>
```

### Research Automation

```python
# Start web research with queries
nlm research start <notebook-id> \
  --query "quantum computing breakthroughs 2024" \
  --query "quantum error correction"

# Drive-based research
nlm research drive <notebook-id> \
  --drive-id "1ABC..." \
  --query "financial trends"

# Check research status
nlm research status <notebook-id> <research-id>
```

### Batch Operations

```python
# Batch query across notebooks
nlm batch query "What are the main themes?" \
  --notebooks <id1> <id2> <id3>

# Batch create notebooks
nlm batch create \
  --names "Project A" "Project B" "Project C"

# Batch delete
nlm batch delete --notebooks <id1> <id2> <id3> --confirm
```

### Cross-Notebook Queries

```python
# Query across multiple notebooks
nlm cross query "Compare the methodologies" \
  --notebooks <id1> <id2> <id3>

# Smart notebook selection by tags
nlm cross query "What are common themes?" \
  --tags research papers
```

### Tagging

```python
# Add tags to notebook
nlm tag add <notebook-id> research important

# List all tags
nlm tag list

# Select notebooks by tag
nlm tag select research
```

### Pipelines (Multi-Step Workflows)

```python
# Run a predefined pipeline
nlm pipeline run research-to-podcast \
  --notebook <notebook-id> \
  --urls "https://site1.com,https://site2.com"

# List available pipelines
nlm pipeline list

# Create custom pipeline (YAML)
cat > my-pipeline.yaml <<EOF
name: research-to-podcast
steps:
  - type: source_add
    urls: {{ urls }}
  - type: studio_create
    content_type: audio
EOF

nlm pipeline run my-pipeline.yaml --notebook <id> --urls "https://..."
```

## MCP Server Setup

### Automatic Configuration

```bash
# Add to AI tools automatically
nlm setup add claude-code
nlm setup add claude-desktop
nlm setup add gemini
nlm setup add github-copilot
nlm setup add cursor
nlm setup add windsurf
nlm setup add cline
nlm setup add antigravity

# Generate JSON config for custom tools
nlm setup add json

# List configured tools
nlm setup list

# Remove from a tool
nlm setup remove claude-code
```

### Manual MCP Configuration

If automatic setup doesn't work, manually add to your MCP client config:

```json
{
  "mcpServers": {
    "notebooklm-mcp": {
      "command": "notebooklm-mcp",
      "args": [],
      "env": {
        "NOTEBOOKLM_PROFILE": "default"
      }
    }
  }
}
```

**Config file locations:**
- **Claude Desktop**: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)
- **Cursor**: `~/Library/Application Support/Cursor/User/globalStorage/rooveterinaryinc.roo-cline/settings/cline_mcp_settings.json`
- **Gemini CLI**: `~/.config/gemini-ai/mcp.json`

### Install AI Skills (Optional)

```bash
# Install expert guide for your AI assistant
nlm skill install cline
nlm skill install claude-code
nlm skill install gemini

# Update skills
nlm skill update cline

# List installed skills
nlm skill list
```

## MCP Tools Reference

### Core Tools

```python
# List notebooks
notebook_list()

# Create notebook
notebook_create(title="Research Project")

# Add sources
source_add(
    notebook_id="abc123",
    urls=["https://example.com"],
    texts=[{"content": "Notes", "title": "Summary"}],
    drive_ids=["1ABC..."],
    files=["/path/to/file.pdf"]
)

# Query notebook
notebook_query(
    notebook_id="abc123",
    query="What are the key findings?",
    grounding_instruction="Cite sources",
    format_as="markdown"
)

# Create studio content
studio_create(
    notebook_id="abc123",
    content_type="audio",  # or "video", "slides"
    confirm=True
)

# Download artifact
download_artifact(
    notebook_id="abc123",
    artifact_id="xyz789",
    artifact_type="audio",  # or "video", "slides"
    output_path="podcast.wav"
)

# Share notebook
notebook_share_public(notebook_id="abc123")
notebook_share_invite(notebook_id="abc123", email="user@example.com")
```

### Advanced Tools

```python
# Research automation
research_start(
    notebook_id="abc123",
    queries=["topic 1", "topic 2"],
    source_type="web"  # or "drive"
)

# Batch operations
batch_query(
    query="Compare approaches",
    notebook_ids=["id1", "id2", "id3"]
)

# Cross-notebook query
cross_notebook_query(
    query="What are common patterns?",
    notebook_ids=["id1", "id2"],
    tags=["research"]
)

# Pipeline execution
pipeline_run(
    pipeline_name="research-to-podcast",
    notebook_id="abc123",
    params={"urls": ["https://site.com"]}
)

# Slide revision
studio_revise(
    notebook_id="abc123",
    artifact_id="xyz789",
    instruction="Add more technical details",
    slide_numbers=[1, 3, 5]
)
```

## Common Workflows

### Research Paper to Podcast

```bash
#!/bin/bash
# Create notebook
NOTEBOOK_ID=$(nlm notebook create "Research Summary" | jq -r '.id')

# Add sources
nlm source add $NOTEBOOK_ID \
  --url "https://arxiv.org/pdf/2024.12345.pdf" \
  --url "https://related-paper.com"

# Generate podcast
nlm studio create $NOTEBOOK_ID --type audio --confirm

# Download when ready (check status first)
sleep 60  # Wait for generation
ARTIFACT_ID=$(nlm studio list $NOTEBOOK_ID | jq -r '.[0].id')
nlm download audio $NOTEBOOK_ID $ARTIFACT_ID --output research.wav
```

### Automated Weekly Research Digest

```python
#!/usr/bin/env python3
import subprocess
import json

# Create notebook
result = subprocess.run(
    ["nlm", "notebook", "create", "Weekly Digest"],
    capture_output=True, text=True
)
notebook_id = json.loads(result.stdout)["id"]

# Add URLs from research tracking
urls = [
    "https://news.ycombinator.com/best",
    "https://paperswithcode.com/latest",
]

subprocess.run([
    "nlm", "source", "add", notebook_id,
    *[f"--url={url}" for url in urls]
])

# Query for summary
subprocess.run([
    "nlm", "notebook", "query", notebook_id,
    "Summarize the key developments this week"
])

# Generate video presentation
subprocess.run([
    "nlm", "studio", "create", notebook_id,
    "--type=video", "--confirm"
])
```

### Multi-Notebook Cross-Analysis

```bash
# Tag related notebooks
nlm tag add nb1 quarterly-review finance
nlm tag add nb2 quarterly-review finance
nlm tag add nb3 quarterly-review finance

# Query across all tagged notebooks
nlm cross query "Compare revenue growth patterns" --tags quarterly-review finance

# Or specify exact notebooks
nlm cross query "What are the common risks?" --notebooks nb1 nb2 nb3
```

## Configuration

```bash
# View all settings
nlm config list

# Set auth browser preference
nlm config set auth.browser brave

# Set default output directory
nlm config set download.output_dir ~/Downloads/notebooklm

# Enable debug logging
nlm config set logging.level debug

# Reset to defaults
nlm config reset
```

## Troubleshooting

### Diagnose Issues

```bash
# Run comprehensive diagnostics
nlm doctor

# Check specific components
nlm login --check
nlm setup list
```

### Common Issues

**Authentication expired:**
```bash
nlm login --force  # Re-authenticate
```

**MCP server not responding:**
```bash
# Restart your AI tool (Claude Code, Cursor, etc.)
# Or reconnect: /mcp (in Claude Code)
```

**Rate limiting:**
```bash
# NotebookLM has rate limits. Space out requests:
nlm studio create <id> --type audio
sleep 30
nlm studio create <id2> --type audio
```

**Profile switching:**
```bash
# Switch between Google accounts
nlm login switch work
nlm login switch personal
```

### Debug Logging

```bash
# Enable verbose output
nlm --debug notebook list

# Check logs
tail -f ~/.notebooklm-mcp-cli/logs/nlm.log
```

## Best Practices

1. **Use profiles** for multiple Google accounts to avoid re-authenticating
2. **Tag notebooks** for easier cross-notebook queries and organization
3. **Wait for studio content** - audio/video generation takes 3-5 minutes
4. **Batch operations** for efficiency when working with multiple notebooks
5. **Disable MCP** when not using NotebookLM to preserve AI context window
6. **Use pipelines** for repeatable multi-step workflows
7. **Check status** before downloading - artifacts may still be generating

## Environment Variables

```bash
# Use specific profile
export NOTEBOOKLM_PROFILE=work

# Custom config directory
export NOTEBOOKLM_CONFIG_DIR=~/.config/notebooklm

# Debug mode
export NOTEBOOKLM_DEBUG=1
```

## Limitations

- Uses **internal APIs** that may change without notice
- Requires **cookie extraction** from browser session
- **Rate limits** apply (space out studio content creation)
- **No official support** from Google - personal/experimental use only
- **Context window** - 35 MCP tools consume significant context in AI assistants

## Additional Resources

- [CLI Guide](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/CLI_GUIDE.md)
- [MCP Guide](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/MCP_GUIDE.md)
- [Authentication Docs](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/AUTHENTICATION.md)
- [Getting Started](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/GETTING_STARTED.md)
