---
name: boss-agent-cli-job-search
description: CLI tool for AI agents to search BOSS Zhipin jobs with welfare filtering, local shortlisting, and compliance-first automation
triggers:
  - search for jobs on BOSS Zhipin
  - find jobs with specific benefits like weekends off
  - help me search for Python jobs in Shanghai
  - add jobs to my shortlist and compare them
  - analyze job descriptions and optimize my resume
  - show me my job search statistics
  - configure BOSS agent CLI for job hunting
  - check if I'm logged into BOSS Zhipin
---

# boss-agent-cli Job Search Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

`boss-agent-cli` is a Python CLI tool designed for AI agents to assist with job searching on BOSS Zhipin (and other platforms like Zhilian). It operates in **low-risk compliance mode by default**, focusing on read-only operations, local organization, and user-initiated actions. Key features include:

- **Job Search**: Keyword search with 8-dimension filtering (city, salary, experience, etc.)
- **Welfare Filtering**: Auto-pagination to find jobs with specific benefits (e.g., "weekends off, five insurances")
- **Local Shortlist**: Save jobs locally with tags, notes, and comparison tools
- **AI Enhancements**: JD analysis, resume polishing, interview prep, communication coaching
- **Schema-Driven**: All commands output JSON envelopes (`{ok, data, error, hints}`) to stdout
- **Multi-Platform**: Supports BOSS Zhipin (default), Zhilian (read-only), 51job (placeholder)

**Compliance boundaries**: Sensitive operations (greeting, applying, exchanging contacts, recruiter candidate search) are blocked by default and return `COMPLIANCE_BLOCKED`. Users must complete these actions manually on the official platform.

## Installation

```bash
# Install via uv (recommended)
uv tool install boss-agent-cli

# Install browser kernel (only needed for login/export)
patchright install chromium

# Or via pip
pip install boss-agent-cli
```

Requires Python ≥ 3.10. All state stored in `~/.boss-agent/`.

## Authentication & Setup

```bash
# Check environment and connectivity
boss doctor

# User-initiated login (interactive browser)
boss login

# Verify login status
boss status

# Optional: live probe with low-frequency read
boss status --live
```

Login credentials are encrypted locally with Fernet + PBKDF2 machine binding. No data leaves the machine except explicit API calls.

## Core Commands

### Job Discovery

```bash
# Basic search
boss search "Golang" --city 广州

# Search with welfare filtering (core differentiator)
boss search "Python" --city 上海 --welfare "双休,五险一金"

# Advanced filtering
boss search "前端" \
  --city 北京 \
  --salary 20-40 \
  --experience 3-5 \
  --degree 本科 \
  --scale 100-499 \
  --stage 已融资 \
  --welfare "双休,五险一金,年终奖" \
  --limit 50

# Show cached results by history number
boss show 3

# Get detailed job info (saves to local cache)
boss detail <security_id>

# View search history
boss history
```

**Welfare filtering** uses `--welfare "benefit1,benefit2"` with AND logic. The tool auto-paginates to fetch all matches and can sort by local match score with `--sort score`.

### Local Shortlist & Organization

```bash
# Add job to local shortlist with tags
boss shortlist add <security_id> <job_id> --tags 后端,远程 --notes "Interesting tech stack"

# List shortlisted jobs
boss shortlist list

# Compare shortlisted jobs (offline)
boss shortlist compare --tag 远程

# Remove from shortlist
boss shortlist remove <shortlist_id>

# View funnel statistics
boss stats

# Watch job for updates
boss watch add <security_id> <job_id>

# Create search preset
boss preset save "golang-remote" --query "Golang" --city 广州 --welfare "远程,双休"
boss preset run "golang-remote"
```

### AI-Powered Features

```bash
# Configure AI provider (required for AI commands)
boss ai config --provider openai --api-key-env OPENAI_API_KEY --model gpt-4o

# Or use Atlas Cloud (one key for multiple models)
boss ai config --provider atlas --api-key-env ATLAS_API_KEY --model deepseek-ai/deepseek-v4-pro

# Analyze job description
boss ai analyze-jd <security_id>

# Polish resume
boss ai polish ~/resume.txt

# Optimize resume for specific job
boss ai optimize ~/resume.txt <security_id>

# Match shortlist against resume
boss ai fit ~/resume.txt

# Prepare for interview
boss ai interview-prep <security_id> ~/resume.txt

# Get communication coaching
boss ai chat-coach <security_id> "想问一下团队规模"
```

### Resume Management

```bash
# Set default resume
boss resume set ~/my-resume.pdf

# Show current resume
boss resume show

# View my profile
boss me
```

### Recruiter Mode (BOSS Zhipin Only)

```bash
# List your job postings
boss hr jobs list

# Take job online/offline
boss hr jobs online <job_id>
boss hr jobs offline <job_id>
```

**Note**: Candidate data pipelines (search, resume, chat) are blocked by default in recruiter mode.

### System & Configuration

```bash
# Show all available commands with JSON schema
boss schema

# Export as OpenAI/Anthropic tool definitions
boss schema --format openai-tools
boss schema --format anthropic-tools

# List supported platforms
boss platforms

# Export all data
boss export

# Configuration
boss config list
boss config set default_city 广州
boss config set default_salary_range "15-30"
boss config reset

# Clean cache
boss clean
```

## Python SDK Usage

```python
from boss_agent_cli import AuthManager, BossClient, AuthRequired
from boss_agent_cli.platforms.platform import Platform

# Initialize auth manager
auth_mgr = AuthManager()

# Option 1: Use existing session
if auth_mgr.is_authenticated():
    with BossClient(auth_mgr) as client:
        result = client.search_jobs(
            query="Golang",
            city="广州",
            welfare_filter=["双休", "五险一金"],
            limit=20
        )
        
        if result["ok"]:
            for job in result["data"]["jobs"]:
                print(f"{job['job_name']} at {job['brand_name']}")
                print(f"  Salary: {job['salary']}")
                print(f"  Welfare: {', '.join(job.get('welfare', []))}")

# Option 2: Handle auth requirement
else:
    print("Please run: boss login")

# Get job details
with BossClient(auth_mgr) as client:
    detail = client.get_job_detail("abc123xyz")
    if detail["ok"]:
        jd = detail["data"]
        print(f"Description: {jd['description']}")
        print(f"Requirements: {jd['experience']}, {jd['degree']}")

# Platform abstraction
platform = Platform.get_platform("zhipin")  # or "zhilian", "qiancheng"
print(f"Using platform: {platform.name}")
```

## Working with JSON Envelopes

All commands output structured JSON to stdout with this schema:

```python
{
    "ok": bool,              # True if successful
    "data": dict | None,     # Command-specific data
    "pagination": dict | None,  # {page, page_size, total, has_more}
    "error": {               # Present if ok=False
        "code": str,         # e.g., "AUTH_EXPIRED", "COMPLIANCE_BLOCKED"
        "message": str,
        "recoverable": bool,
        "recovery_action": str | None
    } | None,
    "hints": list[str] | None  # User-facing suggestions
}
```

**Parsing in Python:**

```python
import json
import subprocess

def run_boss_command(args: list[str]) -> dict:
    """Run boss command and parse JSON output."""
    result = subprocess.run(
        ["boss"] + args,
        capture_output=True,
        text=True,
        check=False
    )
    
    if result.returncode != 0:
        # Parse error envelope
        error_data = json.loads(result.stdout)
        if error_data.get("error", {}).get("code") == "AUTH_EXPIRED":
            # Trigger re-authentication
            subprocess.run(["boss", "login"], check=True)
            return run_boss_command(args)  # Retry
        raise RuntimeError(error_data["error"]["message"])
    
    return json.loads(result.stdout)

# Example usage
search_result = run_boss_command([
    "search", "Python",
    "--city", "上海",
    "--welfare", "双休,五险一金",
    "--limit", "30"
])

if search_result["ok"]:
    jobs = search_result["data"]["jobs"]
    print(f"Found {len(jobs)} jobs")
    
    # Add first result to shortlist
    if jobs:
        job = jobs[0]
        shortlist_result = run_boss_command([
            "shortlist", "add",
            job["security_id"],
            job["encrypt_job_id"],
            "--tags", "python,remote"
        ])
```

## Agent Integration Patterns

### Pattern 1: MCP Server (Recommended for Claude Desktop / Cursor)

Add to `claude_desktop_config.json` or `cursor/.cursorrules`:

```json
{
  "mcpServers": {
    "boss-agent": {
      "command": "uvx",
      "args": ["--from", "boss-agent-cli[mcp]", "boss-mcp"]
    }
  }
}
```

This exposes 35 low-risk tools to the MCP host. Install MCP extras: `pip install boss-agent-cli[mcp]`

### Pattern 2: Subprocess Shell Agent

```python
def get_boss_capabilities() -> dict:
    """Fetch self-describing schema."""
    result = subprocess.run(
        ["boss", "schema", "--format", "openai-tools"],
        capture_output=True,
        text=True,
        check=True
    )
    return json.loads(result.stdout)["data"]["tools"]

# Let agent discover capabilities
tools = get_boss_capabilities()
# Pass tools to LLM for function calling
```

### Pattern 3: Direct Python Import

```python
from boss_agent_cli import BossClient, AuthManager
from boss_agent_cli.ai.service import AIService

# Embed in your agent
class JobSearchAgent:
    def __init__(self):
        self.client = BossClient(AuthManager())
        self.ai = AIService()
    
    def find_and_analyze(self, query: str, city: str) -> dict:
        """Search jobs and analyze top result."""
        search = self.client.search_jobs(query, city=city, limit=5)
        if not search["ok"]:
            return search
        
        top_job = search["data"]["jobs"][0]
        detail = self.client.get_job_detail(top_job["security_id"])
        
        if detail["ok"]:
            analysis = self.ai.analyze_jd(detail["data"])
            return {"job": detail["data"], "analysis": analysis}
        
        return detail
```

## Configuration Reference

Config stored at `~/.boss-agent/config.json`:

```json
{
  "default_city": "广州",
  "default_salary_range": "15-30",
  "default_experience": null,
  "default_degree": null,
  "request_interval": [2.0, 4.0],
  "log_level": "INFO",
  "login_timeout": 300,
  "cdp_address": null,
  "export_dir": "~/.boss-agent/exports",
  "platform": "zhipin",
  "ai": {
    "provider": "openai",
    "base_url": null,
    "api_key_env": "OPENAI_API_KEY",
    "model": "gpt-4o",
    "temperature": 0.7,
    "max_tokens": 2000
  }
}
```

Update via `boss config set <key> <value>` or edit JSON directly.

## Troubleshooting

### Auth Issues

```bash
# Run full diagnostic
boss doctor --live-probe

# Check login status
boss status

# Re-authenticate
boss logout
boss login
```

### Platform Switching

```bash
# Switch to Zhilian (read-only mode)
boss --platform zhilian search "Python"

# Set as default
boss config set platform zhilian
```

### Rate Limiting / Risk Control

- Tool uses Gaussian delay (2-4s default) between requests
- Stops on platform risk detection (doesn't evade)
- Never use for bulk operations (blocked by design)

### Error Codes

- `AUTH_REQUIRED`: Run `boss login`
- `AUTH_EXPIRED`: Session expired, re-login
- `COMPLIANCE_BLOCKED`: Operation not allowed in low-risk mode
- `PLATFORM_ERROR`: Platform API issue, check `boss doctor`
- `NOT_SUPPORTED`: Feature unavailable for this platform

### Browser Bridge Issues

```bash
# Start daemon manually for debugging
python -m boss_agent_cli.bridge.daemon --serve

# Check bridge health (7 checks)
boss doctor --live-probe
```

Bridge only used for login and local export, not for evading detection.

## Common Workflows

### Workflow 1: Daily Job Hunt

```bash
# Morning: search for new jobs
boss search "Python后端" --city 上海 --welfare "双休,五险一金" --sort score

# Review details and add to shortlist
boss detail <security_id>
boss shortlist add <security_id> <job_id> --tags "interesting"

# Evening: review shortlist and stats
boss shortlist list
boss stats
```

### Workflow 2: Targeted Application Prep

```bash
# Find job
boss search "DevOps" --city 北京 --experience 3-5

# Deep analysis
boss detail <security_id>
boss ai analyze-jd <security_id>

# Optimize resume
boss ai optimize ~/resume.pdf <security_id>

# Prepare interview
boss ai interview-prep <security_id> ~/resume.pdf

# Manual application on official site (not automated)
```

### Workflow 3: Recruiter Job Management

```bash
# Check your postings
boss hr jobs list

# Take job online
boss hr jobs online <job_id>

# Monitor views/applications on official platform
# (candidate pipelines blocked in CLI)
```

## Environment Variables

```bash
# AI providers
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export ATLAS_API_KEY="..."  # Atlas Cloud unified key

# Optional: custom CDP endpoint
export BOSS_CDP_ADDRESS="http://localhost:9222"

# Optional: override config location
export BOSS_CONFIG_DIR="~/.custom-boss-config"
```

## Platform Support Matrix

| Platform | Seeker | Recruiter | Status |
|----------|:------:|:---------:|--------|
| BOSS Zhipin (`zhipin`) | ✅ Full | ✅ Jobs only | Default |
| Zhilian (`zhilian`) | 🟡 Read-only | ❌ Not supported | Write/social blocked |
| 51job (`qiancheng`) | 🚧 Placeholder | ❌ Not supported | Returns `NOT_SUPPORTED` |

```bash
# Use specific platform
boss --platform zhilian search "Java"

# Check available platforms
boss platforms
```

## Best Practices for AI Agents

1. **Always check schema first**: `boss schema` is the source of truth for capabilities
2. **Parse JSON envelopes**: Check `ok` field before processing `data`
3. **Handle errors gracefully**: Use `recoverable` and `recovery_action` for retry logic
4. **Respect compliance**: Don't attempt to bypass `COMPLIANCE_BLOCKED` errors
5. **Use local shortlist**: Store and compare jobs locally, don't spam searches
6. **Leverage AI features**: Use `ai` commands for analysis, don't just dump JDs to user
7. **Configure once**: Set defaults with `boss config` to simplify subsequent calls

## Further Reading

- [Quick Start Guide](https://github.com/can4hou6joeng4/boss-agent-cli/blob/master/docs/getting-started.md)
- [Command Reference](https://github.com/can4hou6joeng4/boss-agent-cli/blob/master/docs/commands.md)
- [Agent Integration Guide](https://github.com/can4hou6joeng4/boss-agent-cli/blob/master/docs/agent-quickstart.md)
- [Capability Matrix](https://github.com/can4hou6joeng4/boss-agent-cli/blob/master/docs/capability-matrix.md)
- [Platform Abstraction Design](https://github.com/can4hou6joeng4/boss-agent-cli/blob/master/docs/platform-abstraction.md)
- [Troubleshooting Guide](https://github.com/can4hou6joeng4/boss-agent-cli/blob/master/docs/troubleshooting.md)
