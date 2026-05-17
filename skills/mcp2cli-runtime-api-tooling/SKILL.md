---
name: mcp2cli-runtime-api-tooling
description: Turn MCP, OpenAPI, or GraphQL servers into CLIs at runtime with zero codegen, saving 96-99% of tokens on tool schemas
triggers:
  - convert an API to a CLI
  - create a CLI from an OpenAPI spec
  - connect to an MCP server
  - turn a GraphQL endpoint into commands
  - bake an API configuration
  - reduce tool schema token costs
  - generate a CLI from API documentation
  - call MCP tools from command line
---

# mcp2cli — Runtime API to CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

mcp2cli dynamically converts MCP servers, OpenAPI specs, and GraphQL endpoints into CLIs at runtime, eliminating the need for code generation and reducing token waste by 96-99% when used with AI agents. It introspects APIs, generates subcommands, handles authentication (including OAuth), and caches schemas intelligently.

## Installation

```bash
# Run directly without installing
uvx mcp2cli --help

# Or install globally
uv tool install mcp2cli

# Install as an AI agent skill
npx skills add knowsuchagency/mcp2cli --skill mcp2cli
```

## Core Concepts

mcp2cli operates in four modes:
- **MCP HTTP/SSE**: Connect to MCP servers over HTTP with SSE or streamable HTTP transport
- **MCP stdio**: Launch and communicate with MCP servers via stdin/stdout
- **OpenAPI**: Convert REST APIs with OpenAPI specs into CLIs
- **GraphQL**: Introspect GraphQL endpoints and generate query/mutation commands

All modes support authentication, caching, and can be "baked" into named configurations for reuse.

## MCP HTTP/SSE Mode

```bash
# List available tools from an MCP server
mcp2cli --mcp https://mcp.example.com/sse --list

# Call a specific tool
mcp2cli --mcp https://mcp.example.com/sse search --query "rust packages"

# With authentication
mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "x-api-key:env:MCP_API_KEY" \
  query --sql "SELECT * FROM users LIMIT 10"

# Force specific transport (skip auto-detection)
mcp2cli --mcp https://mcp.example.com/sse --transport sse --list

# Search for tools by name or description
mcp2cli --mcp https://mcp.example.com/sse --search "database"
```

## MCP stdio Mode

```bash
# Launch an MCP server and list its tools
mcp2cli --mcp-stdio "npx @modelcontextprotocol/server-filesystem /tmp" --list

# Call a tool with arguments
mcp2cli --mcp-stdio "npx @modelcontextprotocol/server-filesystem /tmp" \
  read-file --path /tmp/data.json

# Pass environment variables to the server process
mcp2cli --mcp-stdio "node mcp-server.js" \
  --env "DATABASE_URL=env:DATABASE_URL" \
  --env "LOG_LEVEL=debug" \
  fetch-records --limit 50
```

## OpenAPI Mode

```bash
# List all endpoints from a remote spec
mcp2cli --spec https://petstore3.swagger.io/api/v3/openapi.json --list

# Call an endpoint with parameters
mcp2cli --spec https://api.example.com/openapi.json \
  --base-url https://api.example.com \
  list-pets --status available --limit 20

# Use a local spec file
mcp2cli --spec ./openapi.yaml --base-url http://localhost:8000 --list

# With authentication
mcp2cli --spec ./api-spec.json \
  --base-url https://api.example.com \
  --auth-header "Authorization:Bearer env:API_TOKEN" \
  create-item --name "Test Item" --price 99.99

# POST with JSON body from stdin
echo '{"name": "Fido", "species": "dog", "age": 3}' | \
  mcp2cli --spec ./spec.json create-pet --stdin

# Pretty-print JSON output
mcp2cli --spec ./spec.json --pretty list-users
```

## GraphQL Mode

```bash
# List all queries and mutations
mcp2cli --graphql https://api.example.com/graphql --list

# Execute a query
mcp2cli --graphql https://api.example.com/graphql \
  users --limit 10

# Execute a mutation
mcp2cli --graphql https://api.example.com/graphql \
  create-user --name "Alice" --email "alice@example.com" --role "admin"

# Override auto-generated field selection
mcp2cli --graphql https://api.example.com/graphql \
  users --fields "id name email createdAt"

# With authentication
mcp2cli --graphql https://api.example.com/graphql \
  --auth-header "Authorization:Bearer env:GRAPHQL_TOKEN" \
  list-repositories --owner "knowsuchagency"
```

## OAuth Authentication

mcp2cli supports OAuth flows across all modes with automatic token caching and refresh.

```bash
# Authorization code + PKCE (opens browser)
mcp2cli --mcp https://mcp.example.com/sse --oauth --list

# Client credentials flow (machine-to-machine)
mcp2cli --spec https://api.example.com/openapi.json \
  --oauth-client-id "env:OAUTH_CLIENT_ID" \
  --oauth-client-secret "env:OAUTH_CLIENT_SECRET" \
  list-resources

# With specific scopes
mcp2cli --graphql https://api.example.com/graphql \
  --oauth --oauth-scope "read:users write:posts" \
  create-post --title "Hello" --body "World"

# Local spec with OAuth requires --base-url for discovery
mcp2cli --spec ./openapi.json \
  --base-url https://api.example.com \
  --oauth --list
```

Tokens are cached in `~/.cache/mcp2cli/oauth/` and automatically refreshed when expired.

## Bake Mode — Named Configurations

Save connection settings and filters to avoid repeating flags.

```bash
# Create a baked configuration from OpenAPI spec
mcp2cli bake create petstore \
  --spec https://api.example.com/openapi.json \
  --auth-header "Authorization:Bearer env:PETSTORE_TOKEN" \
  --exclude "delete-*,update-*" \
  --methods GET,POST \
  --cache-ttl 7200

# Create from MCP stdio server
mcp2cli bake create github \
  --mcp-stdio "npx @modelcontextprotocol/server-github" \
  --env "GITHUB_TOKEN=env:GITHUB_TOKEN" \
  --include "search-*,list-*" \
  --exclude "delete-*"

# Create from MCP HTTP server with OAuth
mcp2cli bake create analytics \
  --mcp https://analytics.example.com/mcp \
  --oauth --oauth-client-id "env:ANALYTICS_CLIENT_ID" \
  --oauth-client-secret "env:ANALYTICS_CLIENT_SECRET"

# Use a baked tool with @ prefix
mcp2cli @petstore --list
mcp2cli @petstore list-pets --limit 10
mcp2cli @github search-repositories --query "rust mcp"
mcp2cli @analytics query-metrics --start-date "2026-01-01"

# Manage baked tools
mcp2cli bake list
mcp2cli bake show petstore
mcp2cli bake update petstore --cache-ttl 3600
mcp2cli bake remove petstore

# Install wrapper script to ~/.local/bin
mcp2cli bake install petstore
# Now you can run: petstore --list

# Install to custom directory
mcp2cli bake install github --dir ./scripts/
```

Configurations are stored in `~/.config/mcp2cli/baked.json`.

## Filtering and Discovery

```bash
# Include only specific tools (glob patterns)
mcp2cli bake create myapi \
  --spec ./spec.json \
  --include "list-*,get-*,search-*"

# Exclude specific tools
mcp2cli bake create myapi \
  --spec ./spec.json \
  --exclude "delete-*,admin-*"

# Filter by HTTP methods (OpenAPI only)
mcp2cli bake create readonly-api \
  --spec ./spec.json \
  --methods GET

# Search tools by name or description
mcp2cli @myapi --search "user"
mcp2cli --spec ./spec.json --search "database query"
```

## Usage-Aware Tool Ranking

mcp2cli tracks tool usage locally and ranks output to reduce token costs.

```bash
# List tools sorted by call frequency (default when usage data exists)
mcp2cli @myapi --list

# Top 10 most-used tools, compact format (~20 tokens vs ~1400)
mcp2cli @myapi --list --top 10 --compact

# Sort by most recently used
mcp2cli @myapi --list --sort recent

# Alphabetical sort
mcp2cli @myapi --list --sort alpha

# Show full descriptions (unwrapped)
mcp2cli @myapi --list --verbose
```

Usage data is stored in `~/.cache/mcp2cli/usage.json`.

## Output Control

```bash
# Pretty-print JSON (auto-enabled for TTY)
mcp2cli @myapi list-records --pretty

# Raw response body (no JSON parsing)
mcp2cli @myapi get-binary-data --raw > output.bin

# Limit output to first N records
mcp2cli @myapi list-users --head 5

# TOON format — token-efficient encoding (40-60% fewer tokens)
mcp2cli @myapi list-tags --toon

# Pipe-friendly (compact JSON when not a TTY)
mcp2cli @myapi list-users | jq '.[] | select(.active == true) | .email'
```

## Caching

Specs and MCP tool lists are cached with a 1-hour TTL by default.

```bash
# Force refresh (bypass cache)
mcp2cli --spec https://api.example.com/openapi.json --refresh --list

# Custom TTL in seconds
mcp2cli --mcp https://mcp.example.com/sse --cache-ttl 86400 --list

# Custom cache key
mcp2cli --spec https://api.example.com/openapi.json \
  --cache-key my-api-prod --list

# Override cache directory
MCP2CLI_CACHE_DIR=/tmp/my-cache mcp2cli --spec ./spec.json --list
```

Local file specs are never cached.

## Secrets Management

Avoid passing secrets in CLI arguments (visible in process lists) using `env:` or `file:` prefixes.

```bash
# Read from environment variable
mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "Authorization:env:MCP_TOKEN" \
  --list

# Read from file
mcp2cli --mcp https://mcp.example.com/sse \
  --oauth-client-secret "file:/run/secrets/oauth_secret" \
  --oauth-client-id "env:OAUTH_CLIENT_ID" \
  --list

# Works with secret managers that inject env vars
vault kv get -field=token secret/mcp | \
  MCP_TOKEN=$(cat) mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "Authorization:env:MCP_TOKEN" \
  search --query "logs"
```

## Common Patterns

### Create a Project-Specific CLI Wrapper

```python
#!/usr/bin/env python3
"""Project-specific API wrapper using mcp2cli."""

import subprocess
import sys
import os

def main():
    """Run mcp2cli with project defaults."""
    cmd = [
        "mcp2cli",
        "--spec", "https://api.myproject.com/openapi.json",
        "--auth-header", f"Authorization:Bearer {os.environ['PROJECT_API_KEY']}",
        "--cache-ttl", "3600",
        *sys.argv[1:]
    ]
    subprocess.run(cmd, check=True)

if __name__ == "__main__":
    main()
```

### Batch Operations with Shell

```bash
#!/bin/bash
# Batch create resources from CSV

while IFS=, read -r name email role; do
  echo "Creating user: $name"
  mcp2cli @myapi create-user \
    --name "$name" \
    --email "$email" \
    --role "$role"
done < users.csv
```

### Combine with jq for Processing

```bash
# Extract specific fields and transform
mcp2cli @analytics get-metrics --period "2026-05" | \
  jq '[.data[] | {date: .timestamp, value: .metric_value}]' | \
  mcp2cli @warehouse import-timeseries --stdin
```

### Create a Skill from an API

If you're building AI agent skills for APIs, use mcp2cli to generate the skill:

```bash
# Generate a skill from an OpenAPI spec
mcp2cli --spec https://api.example.com/openapi.json --list > API_COMMANDS.txt

# Use the command list as reference in your SKILL.md
# Document the baked configuration approach for the skill
```

## Configuration Locations

- Baked configs: `~/.config/mcp2cli/baked.json` (override with `MCP2CLI_CONFIG_DIR`)
- Cache: `~/.cache/mcp2cli/` (override with `MCP2CLI_CACHE_DIR`)
- OAuth tokens: `~/.cache/mcp2cli/oauth/`
- Usage tracking: `~/.cache/mcp2cli/usage.json`

## Troubleshooting

### OAuth flow fails to open browser

```bash
# Ensure DISPLAY is set (Linux) or browser is available
echo $DISPLAY

# Use client credentials flow instead if available
mcp2cli --spec ./spec.json \
  --oauth-client-id "env:CLIENT_ID" \
  --oauth-client-secret "env:CLIENT_SECRET" \
  --list
```

### MCP transport auto-detection issues

```bash
# Force specific transport
mcp2cli --mcp https://mcp.example.com/sse --transport sse --list

# Try streamable HTTP if SSE fails
mcp2cli --mcp https://mcp.example.com/sse --transport streamable --list
```

### Cache not refreshing

```bash
# Force refresh
mcp2cli @myapi --refresh --list

# Clear cache manually
rm -rf ~/.cache/mcp2cli/

# Check cache TTL
mcp2cli bake show myapi  # inspect cache-ttl setting
```

### Secrets not loading from environment

```bash
# Verify environment variable is set
echo $MY_API_KEY

# Use explicit env: prefix
mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "x-api-key:env:MY_API_KEY" \
  --list

# Debug with verbose output (if added in future versions)
# Currently: check that env var name matches exactly
```

### OpenAPI spec with relative servers

```bash
# Provide explicit base URL
mcp2cli --spec ./openapi.json --base-url https://api.example.com --list
```

### GraphQL field selection too shallow

```bash
# Override auto-generated fields
mcp2cli --graphql https://api.example.com/graphql \
  users --fields "id name email profile { avatar bio } posts { id title }"
```

### Large response truncation

```bash
# Limit output size
mcp2cli @myapi list-all-records --head 100 > sample.json

# Use raw mode for binary data
mcp2cli @myapi download-file --file-id 123 --raw > output.pdf
```

## Token Efficiency Analysis

When using mcp2cli with AI agents:
- Traditional approach: Send full tool schemas every turn (~1,400 tokens for 96 tools)
- mcp2cli approach: `--list --compact` (~20 tokens for tool names)
- Savings: 96-99% reduction in tool schema tokens
- Use `--top N` to further reduce token costs by showing only frequently-used tools
- Use `--toon` output format for 40-60% smaller responses on large uniform arrays

## Development Integration

```python
# Use mcp2cli programmatically via subprocess
import subprocess
import json

def call_api(tool: str, **kwargs):
    """Call mcp2cli and parse JSON response."""
    args = ["mcp2cli", "@myapi", tool]
    for k, v in kwargs.items():
        args.extend([f"--{k.replace('_', '-')}", str(v)])
    
    result = subprocess.run(args, capture_output=True, text=True, check=True)
    return json.loads(result.stdout)

# Use it
users = call_api("list-users", status="active", limit=50)
for user in users:
    print(f"{user['name']} <{user['email']}>")
```

## Real-World Example: GitHub MCP Server

```bash
# Set up GitHub MCP server as a baked tool
mcp2cli bake create github \
  --mcp-stdio "npx @modelcontextprotocol/server-github" \
  --env "GITHUB_TOKEN=env:GITHUB_TOKEN"

# List all available GitHub operations
mcp2cli @github --list --compact

# Search repositories
mcp2cli @github search-repositories --query "mcp server" --limit 20 --pretty

# Get repository details
mcp2cli @github get-repository --owner "knowsuchagency" --repo "mcp2cli"

# Create an issue
mcp2cli @github create-issue \
  --owner "myorg" \
  --repo "myrepo" \
  --title "Feature request" \
  --body "Add support for X"

# Install as a standalone command
mcp2cli bake install github
# Now run: github search-repositories --query "rust"
```

For complete documentation and token savings analysis, see the [full writeup](https://www.orangecountyai.com/blog/mcp2cli-one-cli-for-every-api-zero-wasted-tokens).
