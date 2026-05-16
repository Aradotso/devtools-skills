---
name: fieldtheory-cli
description: Sync, search, and classify X/Twitter bookmarks locally with full-text search, LLM classification, and agent integration
triggers:
  - "sync my twitter bookmarks"
  - "search my x bookmarks for"
  - "classify my bookmarks"
  - "export bookmarks to markdown"
  - "search my fieldtheory bookmarks"
  - "install fieldtheory skill"
  - "create a knowledge base from bookmarks"
  - "run possible on my bookmarks"
---

# fieldtheory-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Field Theory CLI syncs and stores all your X/Twitter bookmarks locally. Search, classify, export to markdown, build knowledge bases, and make bookmarks available to AI agents with shell access.

## What it does

- **Syncs X bookmarks** to `~/.fieldtheory/bookmarks/` (no API key needed by default)
- **Full-text search** with BM25 ranking via SQLite FTS5
- **LLM classification** by category (tool, security, technique, launch, research, opinion, commerce) and subject domain
- **Markdown export** with enriched article text for knowledge bases
- **Agent integration** via `/fieldtheory` skill for Claude Code, Codex
- **Possibility runs** to generate ideas from bookmark-grounded seeds
- **OAuth support** for cross-platform API sync

## Installation

```bash
npm install -g fieldtheory
```

Requires Node.js 20+. Chrome-family browser or Firefox recommended for session sync.

Verify installation:

```bash
ft --version
ft status
```

## First-time setup

```bash
# Sync bookmarks (extracts session from browser)
ft sync

# Search bookmarks
ft search "distributed systems"

# View stats
ft stats
ft viz
```

For OAuth API sync (cross-platform):

```bash
ft auth
ft sync --api
```

## Key commands

### Sync

```bash
# Basic sync (downloads bookmarks + media)
ft sync

# Sync without media
ft sync --no-media

# Sync and classify with LLM
ft sync --classify

# Full re-crawl
ft sync --rebuild

# Resume interrupted sync
ft sync --continue

# Backfill gaps (quoted tweets, truncated text, missing media)
ft sync --gaps

# Sync X bookmark folders
ft sync --folders

# Sync single folder
ft sync --folder "AI Research"

# OAuth API sync
ft sync --api
```

### Search and browse

```bash
# Full-text search
ft search "machine learning papers"
ft search "typescript debugging" --limit 20

# List with filters
ft list --author elonmusk
ft list --days 30
ft list --category research
ft list --domain technology
ft list --folder "Reading List"

# Show single bookmark
ft show 1234567890

# Random sample
ft sample research

# Stats
ft stats
ft categories
ft domains
ft folders
ft viz
```

### Classification

```bash
# Classify all unclassified bookmarks
ft classify

# Classify with regex (fast, less accurate)
ft classify --regex

# Classify domains only
ft classify-domains

# Override LLM engine
ft classify --engine claude-opus-4-20250514
ft sync --classify --engine claude-sonnet-4-20250514

# Change default model
ft model
ft model claude-opus-4-20250514
```

### Knowledge base

```bash
# Export to markdown
ft md

# Re-export only changed bookmarks
ft md --changed

# Build interlinked wiki
ft wiki

# Ask questions
ft ask "What are the best resources on RAG?"
ft ask "Summarize distributed systems bookmarks" --save

# Health check wiki
ft lint
ft lint --fix
```

### Possibility runs

```bash
# Create seed from search
ft seeds search "ai agents" --days 90 --limit 8 --create

# Add repos
ft repos add ~/dev/my-project

# Interactive wizard
ft possible

# Run with defaults
ft possible run --defaults

# Background run
ft possible run --background

# View node prompt
ft possible prompt <node-id>

# Install nightly schedule (macOS)
ft possible nightly install --time 02:00 --defaults --model opus --effort medium --nodes 5
ft possible nightly show
```

### Agent integration

```bash
# Install skill for Claude Code/Codex
ft skill install

# Show skill content
ft skill show

# Uninstall
ft skill uninstall
```

### Field Theory app companion

```bash
# Show paths
ft paths --json
ft status --json

# Library management
ft library search "distributed systems"
ft library show notes/example.md
ft library create notes/new.md --stdin
ft library update notes/new.md --stdin --expected-sha256 <hash>
ft library delete notes/old.md
ft library open notes/example.md

# Commands
ft commands list
ft commands new "daily-review"
ft commands validate

# Install Mac app
ft install app
```

### Utilities

```bash
# Rebuild search index
ft index

# Backfill media
ft fetch-media
ft fetch-media --skip-profile-images

# Show status
ft status
ft path
```

## Configuration

### Environment variables

```bash
# Override data directory
export FT_DATA_DIR=/path/to/custom/dir

# Override library directory
export FT_LIBRARY_DIR=/path/to/custom/library

# Override commands directory
export FT_COMMANDS_DIR=/path/to/custom/commands

# Point to dev app (macOS)
export FT_APP_DEV_DIR=/Users/you/dev/fieldtheory/mac-app

# Override app bundle ID
export FT_APP_BUNDLE_ID=com.fieldtheory.app.dev

# Custom app launcher
export FT_APP_OPEN_COMMAND=/path/to/launcher

# Proxy settings (standard)
export HTTPS_PROXY=http://proxy:8080
export HTTP_PROXY=http://proxy:8080
export ALL_PROXY=socks5://proxy:1080
export NO_PROXY=localhost,127.0.0.1
```

### LLM configuration

```bash
# View current model
ft model

# Change default model
ft model claude-opus-4-20250514

# Available engines (via env):
# - Anthropic: ANTHROPIC_API_KEY
# - OpenAI: OPENAI_API_KEY
# - OpenRouter: OPENROUTER_API_KEY
```

### Browser selection

```bash
# Auto-detect (default)
ft sync

# Specify browser
ft sync --browser chrome
ft sync --browser firefox
ft sync --browser brave
ft sync --browser edge

# Windows: specify profile
ft sync --browser chrome --chrome-profile-directory "Default"

# Firefox: specify profile directory
ft sync --firefox-profile-dir /path/to/profile

# Manual cookies (last resort)
ft sync --cookies <ct0> <auth_token>
```

## Data structure

```
~/.fieldtheory/
├── bookmarks/
│   ├── bookmarks.jsonl         # Raw cache (one JSON per line)
│   ├── bookmarks.db            # SQLite FTS5 search index
│   ├── bookmarks-meta.json     # Sync metadata
│   └── oauth-token.json        # OAuth token (chmod 600)
├── library/
│   └── index.md                # Knowledge base
├── commands/
│   └── *.md                    # Portable commands
└── ideas/
    ├── seeds/                  # Possibility seeds
    ├── runs/                   # Possibility runs
    ├── nodes/                  # Node prompts
    ├── batches/                # Multi-repo batches
    ├── jobs/                   # Background jobs
    └── nightly/                # Nightly schedules
```

## Common patterns

### Daily sync with classification

```bash
# Crontab: sync every morning at 7am
0 7 * * * ft sync --classify
```

### Search and show workflow

```typescript
// Get search results as JSON
const { execSync } = require('child_process');

function searchBookmarks(query: string): any[] {
  const output = execSync(`ft search "${query}" --json`, { encoding: 'utf8' });
  return JSON.parse(output);
}

function showBookmark(id: string): any {
  const output = execSync(`ft show ${id} --json`, { encoding: 'utf8' });
  return JSON.parse(output);
}

const results = searchBookmarks('typescript patterns');
const first = showBookmark(results[0].id);
console.log(first.text);
```

### Export to custom format

```typescript
import { readFileSync } from 'fs';
import { join } from 'path';
import { homedir } from 'os';

const dataDir = process.env.FT_DATA_DIR || join(homedir(), '.fieldtheory');
const bookmarksPath = join(dataDir, 'bookmarks', 'bookmarks.jsonl');

// Read raw JSONL
const bookmarks = readFileSync(bookmarksPath, 'utf8')
  .split('\n')
  .filter(line => line.trim())
  .map(line => JSON.parse(line));

// Filter and transform
const aiBookmarks = bookmarks
  .filter(b => b.classification?.category === 'research')
  .filter(b => b.classification?.domain === 'AI/ML')
  .map(b => ({
    text: b.text,
    author: b.author.screen_name,
    url: b.url,
    created: b.created_at
  }));

console.log(JSON.stringify(aiBookmarks, null, 2));
```

### Programmatic classification

```typescript
import { execSync } from 'child_process';

function classifyBookmarks(engine?: string): void {
  const cmd = engine 
    ? `ft classify --engine ${engine}`
    : 'ft classify';
  
  try {
    execSync(cmd, { stdio: 'inherit' });
  } catch (error) {
    console.error('Classification failed:', error);
    throw error;
  }
}

// Use default model
classifyBookmarks();

// Override model
classifyBookmarks('claude-opus-4-20250514');
```

### Build custom knowledge base

```bash
# Export all bookmarks to markdown
ft md

# Build wiki with interlinks
ft wiki

# Ask questions
ft ask "What are the key themes in my AI bookmarks from 2026?"

# Save answer as concept page
ft ask "Summarize RAG techniques" --save
```

### Possibility automation

```typescript
import { execSync } from 'child_process';

// Create seed from recent bookmarks
execSync('ft seeds search "ai agents" --days 90 --limit 8 --frame leverage-specificity --create', {
  stdio: 'inherit'
});

// Add repositories
execSync('ft repos add ~/dev/my-agent', { stdio: 'inherit' });
execSync('ft repos add ~/dev/my-framework', { stdio: 'inherit' });

// Run in background
const output = execSync('ft possible run --background --defaults --model opus --effort high --nodes 8', {
  encoding: 'utf8'
});

const jobId = output.match(/Job ID: (\S+)/)?.[1];
console.log(`Started job: ${jobId}`);
```

### Library integration

```typescript
import { execSync } from 'child_process';
import { readFileSync } from 'fs';

// Search library
function searchLibrary(query: string): any[] {
  const output = execSync(`ft library search "${query}" --json`, { encoding: 'utf8' });
  return JSON.parse(output);
}

// Show page with metadata
function showPage(path: string): any {
  const output = execSync(`ft library show "${path}" --json`, { encoding: 'utf8' });
  return JSON.parse(output);
}

// Create page
function createPage(path: string, content: string): void {
  execSync(`ft library create "${path}" --stdin`, {
    input: content,
    stdio: ['pipe', 'inherit', 'inherit']
  });
}

// Update page with conflict protection
function updatePage(path: string, content: string, expectedSha: string): void {
  execSync(`ft library update "${path}" --stdin --expected-sha256 ${expectedSha}`, {
    input: content,
    stdio: ['pipe', 'inherit', 'inherit']
  });
}

const results = searchLibrary('distributed systems');
const page = showPage(results[0].path);
console.log(page.content);
```

## Troubleshooting

### Sync fails with browser session

```bash
# Close browser completely and retry
ft sync

# Specify browser explicitly
ft sync --browser chrome

# Windows: specify profile
ft sync --browser chrome --chrome-profile-directory "Default"

# Try OAuth instead
ft auth
ft sync --api
```

### Cookie extraction fails

```bash
# Firefox on Windows needs Node 22.5+ or sqlite3 on PATH
node --version

# Manual cookie fallback (treat as passwords)
ft sync --cookies <ct0> <auth_token>
```

### Windows PowerShell alias conflict

```powershell
# Use full command name
fieldtheory sync

# Or ft.cmd
ft.cmd sync
```

### Classification not working

```bash
# Ensure API key is set
echo $ANTHROPIC_API_KEY

# Try different engine
ft model
ft classify --engine claude-sonnet-4-20250514

# Fall back to regex
ft classify --regex
```

### Search index out of sync

```bash
# Rebuild index
ft index

# Or full rebuild
ft sync --rebuild
```

### Media download stalls

```bash
# Sync without media
ft sync --no-media

# Skip profile images
ft sync --skip-profile-images

# Backfill media separately
ft fetch-media
```

### Proxy issues

```bash
# Set proxy env vars
export HTTPS_PROXY=http://proxy:8080
export HTTP_PROXY=http://proxy:8080

# Retry sync
ft sync
```

### Data corruption

```bash
# Check status
ft status

# Rebuild index
ft index

# Last resort: full re-sync
rm ~/.fieldtheory/bookmarks/bookmarks.db
ft sync --rebuild
```

### OAuth token expired

```bash
# Re-authenticate
ft auth

# Sync with fresh token
ft sync --api
```

## Security notes

- **Local-first**: No telemetry, no analytics, no phone-home
- **Session sync** reads cookies from browser database, uses them once, discards
- **OAuth tokens** stored with `chmod 600` at `~/.fieldtheory/bookmarks/oauth-token.json`
- Treat `ct0`, `auth_token`, and `oauth-token.json` like passwords
- Default sync uses X's internal GraphQL API (same as browser)
- OAuth sync uses official v2 API

## Platform support

| Feature | macOS | Linux | Windows |
|---------|-------|-------|---------|
| Session sync | ✓ | ✓ | ✓ |
| OAuth API | ✓ | ✓ | ✓ |
| Search/classify | ✓ | ✓ | ✓ |
| Supported browsers | Chrome, Chromium, Brave, Edge, Firefox, Helium, Comet, Dia | Chrome, Chromium, Brave, Edge, Firefox | Chrome, Chromium, Brave, Edge, Firefox |

## Additional resources

- Homepage: https://fieldtheory.dev/cli
- GitHub: https://github.com/afar1/fieldtheory-cli
- License: MIT
