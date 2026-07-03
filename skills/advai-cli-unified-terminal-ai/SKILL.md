---
name: advai-cli-unified-terminal-ai
description: Unified CLI for local skills, external CLIs, and terminal AI chat with multi-platform skill sync support.
triggers:
  - "install or manage AI coding agent skills"
  - "sync skills to Cursor, Claude Code, or other platforms"
  - "start a terminal AI chat session"
  - "manage external CLIs through advai"
  - "create a local knowledge base"
  - "configure advai-cli with OpenAI-compatible API"
  - "list available skills and platforms"
  - "update or uninstall advai skills"
---

# advai-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`advai-cli` is a unified command-line interface that consolidates local skill management, external CLI workflows, and terminal-native AI chat through a single `advai` entrypoint. It provides a Python-first core with npm and Homebrew distribution options, supporting 51+ built-in platform adapters including Cursor, Claude Code, Codex, TRAE, Cline, Continue, and more.

## Installation

### Via PyPI (recommended for Python environments)

```bash
pip install advai-cli
```

### Via npm (for Node.js environments)

```bash
npm install -g advai-cli
```

The npm package creates a private Python virtual environment and installs the PyPI package during postinstall.

### Via Homebrew (macOS)

```bash
brew tap Advai-X/tap
brew install advai-cli
```

### Verify installation

```bash
advai --help
advai info
```

## Core Concepts

- **Skills**: Reusable agent definitions stored locally under `~/.advai/skills`
- **Platforms**: Target environments like Cursor, Claude Code, etc. where skills can be synced
- **External CLIs**: Third-party command-line tools managed through OpenCLI integration
- **Knowledge Bases**: Local document collections for search and reference
- **TUI**: Terminal UI for interactive AI chat with OpenAI-compatible backends

## Configuration

### Environment Variables

```bash
# Required for AI features
export ADVAI_API_KEY="your_api_key"

# Optional configuration
export ADVAI_BASE_URL="https://api.openai.com/v1"
export ADVAI_MODEL="gpt-4o-mini"
export ADVAI_AGENT="default"
export ADVAI_SYSTEM_PROMPT="You are a helpful coding assistant."
export ADVAI_TIMEOUT="120"

# Fallback to standard OpenAI naming
export OPENAI_API_KEY="your_api_key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-4o-mini"
```

### Configuration file location

Local state and skills are stored in `~/.advai/`:

```
~/.advai/
├── skills/          # Installed skill definitions
├── cli/             # External CLI metadata
├── kb/              # Knowledge bases
└── config.json      # User configuration
```

## Skill Management

### List installed skills

```bash
advai skill list
```

### Install a skill from GitHub

```bash
# Install all skills from a repo
advai skill install https://github.com/your-org/skill-repo

# Install a specific skill
advai skill install https://github.com/your-org/skill-repo --skill demo-skill

# Install and sync to a platform immediately
advai skill install https://github.com/your-org/skill-repo --skill demo-skill --platform cursor
```

### View skill details

```bash
advai skill info demo-skill
```

### Update skills

```bash
# Update a specific skill
advai skill update demo-skill

# Update all installed skills
advai skill update
```

### Uninstall a skill

```bash
advai skill uninstall demo-skill
```

## Platform Sync

### List supported platforms

```bash
advai skill platform list
```

Displays 51+ built-in platforms including:
- **Coding**: cursor, claude_code, codex, trae, cline, continue, github_copilot, windsurf, etc.
- **Lobster-style**: autoclaw, openclaw, hermes, workbuddy, etc.

### Sync a skill to platforms

```bash
# Sync to Cursor
advai skill sync demo-skill --platform cursor

# Sync to multiple platforms
advai skill sync demo-skill --platform trae --platform claude_code

# Sync with project-specific directory (e.g., for omp_agent)
advai skill sync demo-skill --platform omp_agent --project-dir /path/to/repo
```

### Remove platform sync

```bash
advai skill unsync demo-skill --platform cursor
```

### Add custom platform

```bash
advai skill platform add custom_agent --name "Custom Agent" --path ~/.custom-agent/skills
```

### Override platform path

```bash
# Override default platform directory
advai skill platform override cursor --path ~/.cursor/skills

# Clear override and return to default
advai skill platform clear-override cursor
```

## External CLI Management

Requires `opencli` binary for full functionality.

### List available CLIs

```bash
advai cli list
```

### View CLI details

```bash
advai cli info gh
```

### Install external CLI

```bash
advai cli install https://github.com/cli/cli
advai cli install https://github.com/cli/cli --cli gh
```

### Update external CLI

```bash
advai cli update gh --yes
```

### Uninstall external CLI

```bash
advai cli uninstall gh --yes
```

### Execute through advai proxy

```bash
advai cli gh repo list
advai cli docker ps
```

## Terminal AI Chat (TUI)

### Basic usage

```bash
# Start with default configuration
advai tui

# Specify model
advai tui --model gpt-4o-mini

# Use custom base URL
advai tui --base-url https://api.openai.com/v1

# Set system prompt
advai tui --system-prompt "You are a concise terminal coding assistant."

# Configure timeout
advai tui --timeout 180

# Use specific agent
advai tui --agent default
```

### In-session commands

```bash
/help                           # Show available commands
/clear                          # Clear conversation history
/agent                          # Open interactive agent picker
/agent default                  # Switch to specific agent
/model                          # Open interactive model picker
/model gpt-4o-mini             # Switch to specific model
/system You are a helpful assistant.  # Update system prompt
/save ./chat.md                # Export conversation transcript
/exit                          # Exit TUI
```

### Python API example

```python
# The TUI is CLI-only, but the underlying client can be used programmatically
from advai_cli.client import AdvaiClient

client = AdvaiClient(
    api_key="your_api_key",
    base_url="https://api.openai.com/v1",
    model="gpt-4o-mini",
    timeout=120
)

response = client.chat_completion(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain Python decorators"}
    ]
)

print(response["choices"][0]["message"]["content"])
```

## Knowledge Base Management

### Create knowledge base

```bash
advai kb create project-docs
```

### Add documents

```bash
advai kb doc add project-docs ./README.md
advai kb doc add project-docs ./docs/api.md
```

### Search knowledge base

```bash
advai kb search project-docs "authentication flow"
```

### Sync from source files

```bash
# Refresh all documents from their original paths
advai kb sync project-docs
```

## Common Workflows

### Setting up a new development environment

```python
#!/usr/bin/env python3
"""
Setup script to install and configure advai-cli skills
"""
import subprocess
import os

def setup_advai():
    # Install advai-cli
    subprocess.run(["pip", "install", "advai-cli"], check=True)
    
    # Configure API key
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        raise ValueError("OPENAI_API_KEY environment variable required")
    
    # Install project-specific skills
    skills_repo = "https://github.com/your-org/team-skills"
    subprocess.run([
        "advai", "skill", "install", skills_repo,
        "--skill", "python-best-practices"
    ], check=True)
    
    # Sync to active platforms
    platforms = ["cursor", "claude_code", "continue"]
    for platform in platforms:
        subprocess.run([
            "advai", "skill", "sync", "python-best-practices",
            "--platform", platform
        ], check=True)
    
    print("✓ advai-cli configured with team skills")

if __name__ == "__main__":
    setup_advai()
```

### Batch skill management

```bash
# Install multiple skills and sync to Cursor
for skill_url in \
  "https://github.com/team/skill-python" \
  "https://github.com/team/skill-typescript" \
  "https://github.com/team/skill-testing"
do
  advai skill install "$skill_url" --platform cursor
done

# Update all skills at once
advai skill update

# List what's installed
advai skill list
```

### Multi-platform deployment

```python
#!/usr/bin/env python3
"""
Deploy skills across multiple agent platforms
"""
import subprocess
import sys

SKILL_NAME = "team-coding-standards"
PLATFORMS = [
    "cursor",
    "claude_code", 
    "codex",
    "trae",
    "cline",
    "continue"
]

def deploy_skill(skill_name: str, platforms: list[str]):
    """Deploy a skill to multiple platforms"""
    for platform in platforms:
        try:
            result = subprocess.run(
                ["advai", "skill", "sync", skill_name, "--platform", platform],
                check=True,
                capture_output=True,
                text=True
            )
            print(f"✓ Synced {skill_name} to {platform}")
        except subprocess.CalledProcessError as e:
            print(f"✗ Failed to sync to {platform}: {e.stderr}", file=sys.stderr)

if __name__ == "__main__":
    deploy_skill(SKILL_NAME, PLATFORMS)
```

### Creating a skill repository

```
your-skill-repo/
├── skills/
│   ├── python-patterns/
│   │   └── SKILL.md
│   ├── api-design/
│   │   └── SKILL.md
│   └── testing-practices/
│       └── SKILL.md
└── README.md
```

Install from this structure:

```bash
# Install all skills
advai skill install https://github.com/your-org/your-skill-repo

# Install one specific skill
advai skill install https://github.com/your-org/your-skill-repo --skill python-patterns
```

## Runtime Information

### Check installation details

```bash
# Show runtime, version, and install method
advai info
```

Output includes:
- Python version and path
- advai-cli version
- Install method (pip, npm, brew)
- Configuration file locations
- Active environment variables

### Get update instructions

```bash
# Shows recommended update command for your install method
advai update
```

## Troubleshooting

### TUI won't start

```bash
# Check API key is set
echo $ADVAI_API_KEY

# Test API connection
export ADVAI_API_KEY="your_key"
advai tui --model gpt-4o-mini
```

### Skills not syncing to platform

```bash
# Verify platform is recognized
advai skill platform list | grep cursor

# Check platform path
advai skill platform override cursor --path ~/.cursor/skills

# List installed skills
advai skill list

# Try manual sync with verbose output
advai skill sync my-skill --platform cursor
```

### npm installation issues

```bash
# Ensure Node.js 14+ and Python 3.8+ are installed
node --version
python3 --version

# Reinstall with clean cache
npm uninstall -g advai-cli
npm cache clean --force
npm install -g advai-cli
```

### Skill install from GitHub fails

```bash
# Ensure repo has skills/ directory at root
# Check if you need to specify --skill flag
advai skill install https://github.com/org/repo --skill specific-skill

# Verify GitHub URL is accessible
curl -I https://github.com/org/repo
```

### Platform sync path issues

```bash
# Check if platform directory exists
ls -la ~/.cursor/skills/

# Create directory if needed
mkdir -p ~/.cursor/skills/

# Override platform path if non-standard
advai skill platform override cursor --path /custom/path/skills
```

### Knowledge base search not working

```bash
# Verify KB exists
ls -la ~/.advai/kb/

# Resync documents from source
advai kb sync project-docs

# Check document paths are still valid
advai kb doc add project-docs ./path/to/file.md
```

## Best Practices

1. **Version control skill repositories**: Keep skills in Git with semantic versioning
2. **Use environment variables**: Never hardcode API keys; always use `ADVAI_API_KEY` or `OPENAI_API_KEY`
3. **Sync to multiple platforms**: Install once, sync to all agents you use
4. **Regular updates**: Run `advai skill update` to get latest skill definitions
5. **Custom platforms**: Use `advai skill platform add` for internal or proprietary agents
6. **Project-specific sync**: Use `--project-dir` flag for project-scoped agents like omp_agent
7. **Export TUI sessions**: Use `/save` command to keep transcript records
8. **Knowledge base maintenance**: Run `advai kb sync` after updating source documents

## Integration Examples

### CI/CD skill deployment

```yaml
# .github/workflows/deploy-skills.yml
name: Deploy Skills
on:
  push:
    branches: [main]
    paths:
      - 'skills/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install advai-cli
        run: pip install advai-cli
      - name: Deploy skills
        env:
          ADVAI_API_KEY: ${{ secrets.ADVAI_API_KEY }}
        run: |
          advai skill install . --skill team-standards
          advai skill list
```

### Pre-commit hook

```bash
#!/bin/bash
# .git/hooks/pre-commit
# Ensure skills are synced before committing

advai skill update
advai skill sync team-standards --platform cursor

echo "✓ Skills synced"
```

### Dockerfile with advai-cli

```dockerfile
FROM python:3.11-slim

RUN pip install advai-cli

ENV ADVAI_API_KEY=""
ENV ADVAI_MODEL="gpt-4o-mini"

# Install team skills at build time
RUN advai skill install https://github.com/team/skills --skill coding-standards

CMD ["advai", "tui"]
```

## Additional Resources

- GitHub: https://github.com/Advai-X/advai-cli
- PyPI: https://pypi.org/project/advai-cli/
- npm: https://www.npmjs.com/package/advai-cli
- Issues: https://github.com/Advai-X/advai-cli/issues
