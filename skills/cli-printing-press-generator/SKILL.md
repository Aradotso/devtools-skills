---
name: cli-printing-press-generator
description: Generate AI-agent-first CLIs from any API (OpenAPI, GraphQL, or browser-sniffed) with SQLite sync, compound commands, and MCP servers
triggers:
  - generate a CLI for an API
  - create an MCP server for this service
  - build a CLI with offline search
  - print a CLI from OpenAPI spec
  - reverse engineer an API and generate CLI
  - create agent-native CLI with SQLite
  - generate compound commands for API
  - build CLI with local data sync
---

# CLI Printing Press Generator

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

CLI Printing Press generates production-ready CLIs from any API — REST, GraphQL, or browser-sniffed traffic. Each generated CLI includes SQLite-backed local storage, offline full-text search, compound insight commands, and dual interfaces (Cobra CLI + MCP server). Designed for AI agents first with typed exit codes, auto-JSON output when piped, and token-efficient `--compact` mode.

## What It Does

The Printing Press follows a 6-phase autonomous workflow:

1. **Research**: Discovers official docs, competing CLIs, MCP servers, and community patterns
2. **Non-Obvious Insight**: Identifies the API's secret identity (what it's *actually* useful for)
3. **Code Generation**: Builds Go CLI with domain-specific SQLite tables, FTS5 indexes, sync engine
4. **Verification**: Runs scorecard, dogfood tests, proof-of-behavior checks, live API smoke tests
5. **Polish**: Auto-fixes verify failures, removes dead code, cleans descriptions
6. **Publishing**: Packages for library, generates PR with quality score

**Key differentiators:**
- **Local-first**: High-gravity resources get domain tables (not JSON blobs) with incremental sync
- **Compound commands**: Cross-resource queries (`stale`, `health`, `bottleneck`) impossible with stateless wrappers
- **Dual output**: Every API generates both `<api>-pp-cli` (Cobra) and `<api>-pp-mcp` (MCP server)
- **No spec required**: Point at a website, captures traffic, reverse-engineers the API
- **Agent-native**: Auto-JSON when piped, typed exit codes, `--compact` flag, `--dry-run`

## Installation

### Prerequisites

- Go 1.26.3+ ([install](https://go.dev/dl/))
- Claude Code or compatible AI agent harness
- Git (for cloning skills repo)

### 1. Install Binary

```bash
go install github.com/mvanhorn/cli-printing-press/v4/cmd/printing-press@latest
```

Verify:
```bash
printing-press --version
```

### 2. Install Skills (Recommended Method)

Clone the repo to get skills and automatic updates via `git pull`:

```bash
git clone https://github.com/mvanhorn/cli-printing-press.git
cd cli-printing-press
```

### 3. Start Printing Session

From the cloned repo root:

```bash
# Load skills directly from this repo
claude --plugin-dir .

# Or in a new git worktree (for parallel runs)
claude --plugin-dir . -w
```

## Core Commands

### Primary Generation Command

Inside Claude Code session:

```text
/printing-press <api-name>
/printing-press <url>
/printing-press <api-name> codex
```

**Examples:**

```text
# Generate from API name (auto-discovers docs/specs)
/printing-press Notion

# Generate from website (browser-sniff traffic)
/printing-press https://postman.com/explore

# Use Codex for code generation (60% fewer Opus tokens)
/printing-press HubSpot codex

# Reprint existing CLI under latest machine
/printing-press-reprint notion
```

### Polish Existing CLI

Runs diagnostics, fixes verify failures, removes dead code:

```text
/printing-press-polish <api-name>
```

Example:
```text
/printing-press-polish linear
```

### Publish to Library

Validates, packages, creates PR:

```text
/printing-press-publish <api-name>
```

Example:
```text
/printing-press-publish superhuman
```

### Amend from Dogfood Session

Turn session friction into PR (auto-detects target CLI):

```text
/printing-press-amend
/printing-press-amend <api-name>
```

## Binary CLI Usage

The `printing-press` binary is called by skills but can be used directly:

```bash
# Research phase
printing-press research <api-name> --output ./output

# Generate full CLI
printing-press generate notion --output ~/clis/notion-pp-cli

# Verify generated CLI
printing-press verify ~/clis/notion-pp-cli

# Score quality
printing-press scorecard ~/clis/notion-pp-cli

# Dogfood test
printing-press dogfood ~/clis/notion-pp-cli
```

## Configuration

### Output Locations

Default structure (auto-managed by skills):

```
~/printing-press/
├── .runstate/<scope>/runs/<run-id>/working/<api>-pp-cli/  # Active runs
├── library/<api>/                                          # Published CLIs
└── manuscripts/<api>/<run-id>/                            # Archived runs
    ├── research/
    ├── proofs/
    ├── discovery/
    └── pipeline/
```

`<scope>` derives from git checkout path (parallel worktrees don't conflict).

### Override Output

```bash
printing-press generate stripe --output /custom/path/stripe-cli
```

### Environment Variables

Generated CLIs use these patterns:

```bash
# API authentication
export NOTION_API_KEY="secret_..."
export LINEAR_API_KEY="lin_api_..."
export GITHUB_TOKEN="ghp_..."

# SQLite store location (override default)
export NOTION_PP_STORE_PATH="/custom/notion.db"

# Refresh settings for auto data-source
export LINEAR_PP_REFRESH_TTL="5m"
```

## Generated CLI Patterns

Every generated CLI follows these conventions:

### Authentication Setup

```bash
# Interactive auth setup (writes to ~/.config/<api>-pp-cli/config.yaml)
notion-pp-cli auth login

# Or set directly
export NOTION_API_KEY="secret_abc123"
```

### Data Source Modes

```bash
# Auto mode: refresh if TTL expired (default)
linear-pp-cli issues list --data-source auto

# Force live API call
linear-pp-cli issues list --data-source live

# Local-only (offline)
linear-pp-cli issues list --data-source local
```

### Sync and Search

```bash
# Initial sync
notion-pp-cli sync

# Incremental sync
notion-pp-cli sync --incremental

# Full-text search (FTS5)
notion-pp-cli search "authentication flow"

# Direct SQL query
notion-pp-cli sql "SELECT title FROM pages WHERE updated_at > date('now', '-7 days')"
```

### Agent-Native Features

```bash
# Auto-JSON when piped
linear-pp-cli issues list | jq '.[] | select(.priority == "urgent")'

# Compact mode (60-80% fewer tokens)
linear-pp-cli issues list --compact

# Dry-run (safe exploration)
linear-pp-cli issues create --title "Test" --dry-run

# Typed exit codes
linear-pp-cli issues get ISSUE-123
echo $?  # 0=success, 2=not found, 3=auth, 4=validation, 5=server, 7=offline
```

### Compound Commands

Generated CLIs include domain-specific insight commands:

```bash
# Linear example
linear-pp-cli stale --threshold 7d       # Issues blocked >7 days
linear-pp-cli health --team backend      # Team velocity metrics
linear-pp-cli bottleneck                 # Most-blocking issues

# Discord example
discord-pp-cli knowledge --channel docs  # Thread knowledge graph
discord-pp-cli stale-threads --days 30   # Unanswered >30 days

# Stripe example
stripe-pp-cli churn-signals              # Failed charges + cancellations
stripe-pp-cli cohort-health --month 2026-04
```

## Code Examples

### Using a Generated CLI in Go

```go
package main

import (
    "context"
    "fmt"
    "os"
    "os/exec"
    "encoding/json"
)

// Call generated Linear CLI
func getBlockedIssues() ([]Issue, error) {
    cmd := exec.Command("linear-pp-cli", "stale", 
        "--threshold", "7d",
        "--output", "json")
    
    output, err := cmd.Output()
    if err != nil {
        return nil, fmt.Errorf("CLI error: %w", err)
    }
    
    var issues []Issue
    if err := json.Unmarshal(output, &issues); err != nil {
        return nil, err
    }
    
    return issues, nil
}

// Use MCP server programmatically
func callMCPServer(ctx context.Context, method string, params map[string]interface{}) error {
    // MCP servers expose stdio interface
    cmd := exec.CommandContext(ctx, "linear-pp-mcp")
    
    stdin, _ := cmd.StdinPipe()
    stdout, _ := cmd.StdoutPipe()
    
    if err := cmd.Start(); err != nil {
        return err
    }
    
    req := map[string]interface{}{
        "jsonrpc": "2.0",
        "method": method,
        "params": params,
        "id": 1,
    }
    
    json.NewEncoder(stdin).Encode(req)
    
    var resp map[string]interface{}
    json.NewDecoder(stdout).Decode(&resp)
    
    return cmd.Wait()
}
```

### Integrating Generated CLI into Scripts

```bash
#!/bin/bash
# Deploy blocker detection script

# Sync latest data
notion-pp-cli sync --incremental

# Find stale blockers
BLOCKERS=$(notion-pp-cli sql "
    SELECT b.id, b.title, i.title as blocked_issue
    FROM issues i
    JOIN issues b ON i.blocker_id = b.id
    WHERE b.state = 'in_progress'
      AND b.updated_at < date('now', '-7 days')
" --output json)

# Send to Slack if any found
if [ "$(echo "$BLOCKERS" | jq 'length')" -gt 0 ]; then
    echo "$BLOCKERS" | jq -r '.[] | "⚠️ \(.blocked_issue) blocked by stale: \(.title)"' \
        | slack-cli send --channel "#deploy-alerts"
fi
```

### Creating Custom Commands

Generated CLIs support plugin architecture:

```go
// Add to cmd/custom/reconcile.go in generated CLI
package custom

import (
    "github.com/spf13/cobra"
    "github.com/mvanhorn/linear-pp-cli/internal/store"
)

func NewReconcileCmd(st *store.Store) *cobra.Command {
    cmd := &cobra.Command{
        Use:   "reconcile",
        Short: "Find issues in API but missing from local store",
        RunE: func(cmd *cobra.Command, args []string) error {
            // Query local store
            localIDs := st.GetAllIssueIDs()
            
            // Query live API
            liveIDs := fetchLiveIssueIDs()
            
            // Diff
            missing := difference(liveIDs, localIDs)
            
            for _, id := range missing {
                fmt.Printf("Missing: %s\n", id)
            }
            
            return nil
        },
    }
    return cmd
}
```

## Troubleshooting

### Generation Failures

**Problem:** Research phase hangs or fails

```bash
# Check if API has publicly accessible docs
curl -I https://developers.notion.com

# Try explicit spec URL
/printing-press https://raw.githubusercontent.com/notion/openapi/main/spec.yaml

# Use browser-sniff mode for private APIs
/printing-press https://internal-tool.company.com
```

**Problem:** Codex mode fails repeatedly

The press auto-falls back to local generation after 3 Codex failures. Check logs:

```bash
tail -f ~/.printing-press/.runstate/<scope>/runs/<run-id>/logs/generation.log
```

### Verification Errors

**Problem:** Scorecard shows low score

```bash
# Run polish to auto-fix
/printing-press-polish <api-name>

# Check specific failure
printing-press verify ~/printing-press/library/<api> --verbose
```

**Problem:** Auth tests fail

```bash
# Verify credentials work manually
export API_KEY="your_key"
curl -H "Authorization: Bearer $API_KEY" https://api.service.com/test

# Check generated auth.go
cat ~/printing-press/library/<api>-pp-cli/internal/auth/auth.go
```

### Runtime Issues with Generated CLIs

**Problem:** Sync fails with rate limit

```bash
# Use incremental sync with backoff
api-pp-cli sync --incremental --rate-limit 10/min

# Check cursor tracking
api-pp-cli sql "SELECT resource, cursor, updated_at FROM sync_cursors"
```

**Problem:** Search returns no results

```bash
# Rebuild FTS5 index
api-pp-cli sql "DELETE FROM pages_fts"
api-pp-cli sync --rebuild-index

# Verify FTS table
api-pp-cli sql "SELECT * FROM sqlite_master WHERE type='table' AND name LIKE '%_fts'"
```

**Problem:** Compound command errors

```bash
# Check if required tables exist
api-pp-cli sql ".schema" | grep -A 5 "CREATE TABLE"

# Run with debug output
api-pp-cli stale --threshold 7d --log-level debug
```

### MCP Server Issues

**Problem:** MCP server not responding

```bash
# Test stdio interface directly
echo '{"jsonrpc":"2.0","method":"ping","id":1}' | linear-pp-mcp

# Check MCP manifest
cat ~/printing-press/library/linear-pp-mcp/mcp.json
```

**Problem:** IDE can't find MCP server

Add to Claude Code config (`~/.config/claude-code/mcp-servers.json`):

```json
{
  "linear": {
    "command": "/path/to/linear-pp-mcp",
    "env": {
      "LINEAR_API_KEY": "${LINEAR_API_KEY}"
    }
  }
}
```

### Performance Optimization

**Problem:** Large dataset sync is slow

```bash
# Enable batch mode
api-pp-cli sync --batch-size 1000 --workers 4

# Use partial sync
api-pp-cli sync --resources "issues,comments" --since "2026-01-01"
```

**Problem:** Search queries are slow

```bash
# Analyze query plan
api-pp-cli sql "EXPLAIN QUERY PLAN SELECT * FROM pages_fts WHERE pages_fts MATCH 'search term'"

# Add covering index
api-pp-cli sql "CREATE INDEX idx_pages_updated ON pages(updated_at DESC)"
```

## Best Practices

1. **Always run polish after generation**: Auto-fixes 80% of verify failures
2. **Use `--compact` for agent calls**: 60-80% fewer tokens, same data
3. **Sync incrementally in production**: Full sync only on init
4. **Set refresh TTL based on data volatility**: Real-time (1m), hourly (5m), daily (1h)
5. **Use typed exit codes for error handling**: Don't parse stderr
6. **Leverage compound commands**: They're the CLI's superpower
7. **Credit sources in custom commands**: Check generated README's "Sources and Inspiration"

## Additional Resources

- [Generated CLI Catalog](https://printingpress.dev)
- [Printing Press Library](https://github.com/mvanhorn/printing-press-library)
- [Cursor Integration Guide](https://github.com/mvanhorn/cli-printing-press/blob/main/docs/CURSOR.md)
- [Non-Obvious Insights Examples](https://github.com/mvanhorn/cli-printing-press#the-non-obvious-insight)
