---
name: next-devtools-mcp-skill
description: Expert in using next-devtools-mcp for Next.js development with AI coding agents
triggers:
  - help me debug my Next.js application
  - set up Next.js DevTools MCP
  - upgrade my Next.js app to version 16
  - enable Cache Components in Next.js
  - search Next.js documentation
  - show me Next.js runtime diagnostics
  - what errors are in my Next.js app
  - analyze my Next.js routes structure
---

# Next.js DevTools MCP Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Expert skill for using `next-devtools-mcp`, a Model Context Protocol (MCP) server that provides Next.js development tools and utilities for AI coding agents. Provides runtime diagnostics, documentation search, browser automation, and development workflow automation for Next.js projects.

## What is next-devtools-mcp?

`next-devtools-mcp` is an MCP server that gives AI coding agents powerful Next.js development capabilities:

- **Runtime Diagnostics** (Next.js 16+): Query running Next.js applications for errors, routes, logs, and real-time state
- **Documentation Search**: Access official Next.js docs and knowledge base directly in conversations
- **Browser Automation**: Test and verify Next.js pages with Playwright integration
- **Development Workflows**: Automated upgrade guides and feature enablement prompts

## Installation

### Quick Install (Recommended)

Install for all detected AI coding agents in your project:

```bash
npx add-mcp next-devtools-mcp@latest
```

Add `-y` to skip confirmation. Add `-g` for global installation across all projects.

### Manual Installation

Add to your MCP client configuration file:

```json
{
  "mcpServers": {
    "next-devtools": {
      "command": "npx",
      "args": ["-y", "next-devtools-mcp@latest"]
    }
  }
}
```

### Agent-Specific Configuration

**Claude Code / Claude Desktop:**
```bash
claude mcp add next-devtools npx next-devtools-mcp@latest
```

**Cursor:**
Click: [Install in Cursor](https://cursor.com/en/install-mcp?name=next-devtools&config=eyJjb21tYW5kIjoibnB4IC15IG5leHQtZGV2dG9vbHMtbWNwQGxhdGVzdCJ9)

Or manually: `Cursor Settings` → `MCP` → `New MCP Server`

**Codex:**
```bash
codex mcp add next-devtools -- npx next-devtools-mcp@latest
```

For Windows 11, update `.codex/config.toml`:
```toml
env = { SystemRoot="C:\\Windows", PROGRAMFILES="C:\\Program Files" }
startup_timeout_ms = 20_000
```

**VS Code / Copilot:**
```bash
code --add-mcp '{"name":"next-devtools","command":"npx","args":["-y","next-devtools-mcp@latest"]}'
```

## Requirements

- Node.js v20.19+ (latest maintenance LTS)
- npm or pnpm
- For runtime diagnostics: Next.js 16+ dev server running

## Core Workflow

### 1. Always Initialize First

**CRITICAL:** Call the `init` tool at the start of every Next.js session:

```
Use the init tool to set up Next.js DevTools context
```

This establishes proper documentation requirements and context. For automation, add to your agent's configuration:

**`.claude/CLAUDE.md` or `.cursorrules`:**
```markdown
When starting work on a Next.js project, ALWAYS call the `init` tool from
next-devtools-mcp FIRST to set up proper context and establish documentation
requirements. Do this automatically without being asked.
```

### 2. Start Next.js Dev Server (for Runtime Diagnostics)

For Next.js 16+ projects:
```bash
npm run dev
# or
pnpm dev
```

MCP is enabled by default at `http://localhost:3000/_next/mcp`. The `next-devtools-mcp` server auto-discovers and connects.

## MCP Tools Reference

### init

Initialize Next.js DevTools MCP context. **Call this first in every session.**

```typescript
// Tool input schema
{
  project_path?: string  // Optional, defaults to current directory
}
```

**Example prompts:**
- "Use the init tool to set up Next.js DevTools context"
- "Initialize Next.js DevTools for this project"

**What it does:**
- Sets up AI assistant context for Next.js development
- Establishes documentation-first approach (use `nextjs_docs` for all queries)
- Documents all available tools and workflows
- Provides best practices checklist

### nextjs_docs

Search and retrieve official Next.js documentation.

```typescript
// Search for docs
{
  action: "search",
  query: string,           // e.g., "metadata", "generateStaticParams"
  routerType?: "app" | "pages" | "all"  // default: "all"
}

// Get full doc content
{
  action: "get",
  path: string,            // e.g., "/docs/app/api-reference/functions/refresh"
  anchor?: string          // e.g., "usage"
}
```

**Example workflow:**

```
Search Next.js docs for generateMetadata
```

Agent uses:
1. `nextjs_docs` with `action: "search"`, `query: "generateMetadata"`
2. Reviews search results (titles, paths, snippets)
3. `nextjs_docs` with `action: "get"`, `path: "/docs/app/api-reference/functions/generate-metadata"`
4. Provides answer based on official documentation

**Common searches:**
- "Search Next.js docs for metadata generation"
- "Find Next.js documentation on generateStaticParams"
- "Look up middleware configuration in Next.js docs"
- "Search App Router caching documentation"

### nextjs_index

Query available runtime diagnostic resources from Next.js 16+ dev server.

```typescript
// No input required
{}
```

**Example prompts:**
- "What runtime diagnostics are available?"
- "Show me what I can query from the dev server"
- "List available Next.js MCP resources"

**Output:** List of available resource URIs (errors, routes, logs, etc.)

### nextjs_call

Call a specific runtime diagnostic resource from Next.js 16+ dev server.

```typescript
{
  uri: string  // Resource URI from nextjs_index
}
```

**Example workflows:**

**Check for errors:**
```
Next Devtools, what errors are in my Next.js application?
```
Agent calls: `nextjs_call` with `uri: "nextjs://errors"`

**View route structure:**
```
Next Devtools, show me the structure of my routes
```
Agent calls: `nextjs_call` with `uri: "nextjs://routes"`

**Check dev server logs:**
```
Next Devtools, what's in the development server logs?
```
Agent calls: `nextjs_call` with `uri: "nextjs://logs"`

**Common URIs:**
- `nextjs://errors` - Runtime errors and warnings
- `nextjs://routes` - App Router route structure
- `nextjs://logs` - Development server logs
- `nextjs://config` - Next.js configuration
- `nextjs://env` - Environment variables

### browser_eval

Automate browser testing with Playwright. Use for verifying pages, testing interactions, and detecting runtime issues.

```typescript
// Start browser
{
  action: "start",
  browser?: "chrome" | "firefox" | "webkit" | "msedge",  // default: "chrome"
  headless?: boolean  // default: true
}

// Navigate to URL
{
  action: "navigate",
  url: string
}

// Click element
{
  action: "click",
  selector: string  // CSS selector
}

// Type text
{
  action: "type",
  selector: string,
  text: string
}

// Fill form
{
  action: "fill_form",
  fields: Array<{ selector: string; value: string }>
}

// Execute JavaScript
{
  action: "evaluate",
  script: string
}

// Take screenshot
{
  action: "screenshot",
  path?: string  // default: auto-generated
}

// Get console messages
{
  action: "console_messages"
}

// Close browser
{
  action: "close"
}
```

**Example workflow - Test Next.js page:**

```
Test the homepage at localhost:3000 and take a screenshot
```

Agent executes:
```typescript
// 1. Start browser
{ action: "start", headless: true }

// 2. Navigate
{ action: "navigate", url: "http://localhost:3000" }

// 3. Wait for content
{ action: "evaluate", script: "document.querySelector('h1')?.textContent" }

// 4. Screenshot
{ action: "screenshot" }

// 5. Check for errors
{ action: "console_messages" }

// 6. Clean up
{ action: "close" }
```

**Common patterns:**

**Verify upgrade (Next.js 15 → 16):**
```
Verify that my app works on localhost:3000 after the upgrade
```

**Test navigation:**
```
Test navigating from home to /about page
```

**Check for hydration errors:**
```
Load the page and check for React hydration errors in the console
```

**Important:** For Next.js 16+, prefer `nextjs_index` and `nextjs_call` over `browser_eval` console messages for error detection.

## Pre-Configured Prompts

### upgrade-nextjs-16

Guide for upgrading from Next.js 15 to Next.js 16.

```
Help me upgrade my Next.js app to version 16
```

**What it does:**
1. Analyzes current project setup
2. Updates dependencies in `package.json`
3. Applies codemods for breaking changes
4. Updates configuration files
5. Tests the upgrade with browser verification

### enable-cache-components

Enable Cache Components mode in Next.js 16.

```
Enable Cache Components in my Next.js app
```

**What it does:**
1. Adds `cache: "components"` to `next.config.ts`
2. Migrates components to use `"use cache"` directive
3. Updates data fetching patterns
4. Provides examples and documentation

## Knowledge Base Resources

The MCP server includes focused documentation resources automatically available to agents:

**Cache Components** (12 sections):
- `cache-components://overview`
- `cache-components://core-mechanics`
- `cache-components://public-caches`
- `cache-components://private-caches`
- `cache-components://runtime-prefetching`
- `cache-components://request-apis`
- `cache-components://cache-invalidation`
- `cache-components://advanced-patterns`
- `cache-components://build-behavior`
- `cache-components://error-patterns`
- `cache-components://test-patterns`
- `cache-components://reference`

**Migration Guides:**
- `nextjs16://migration/beta-to-stable`
- `nextjs16://migration/examples`

**Fundamentals:**
- `nextjs-fundamentals://use-client`

Resources are loaded on-demand by the agent as needed.

## Real-World Examples

### Example 1: Debug Runtime Error

**User request:**
```
My Next.js app shows a blank page. Help me figure out what's wrong.
```

**Agent workflow:**

```typescript
// 1. Initialize (if not done)
init({ project_path: "." })

// 2. Check runtime errors
nextjs_call({ uri: "nextjs://errors" })

// Output shows: "ReferenceError: window is not defined at Component.render"

// 3. Search docs for server component issues
nextjs_docs({ 
  action: "search", 
  query: "window is not defined server component" 
})

// 4. Get full doc content
nextjs_docs({ 
  action: "get", 
  path: "/docs/app/building-your-application/rendering/server-components"
})

// 5. Provide solution: Add "use client" directive
```

### Example 2: Upgrade to Next.js 16 with Cache Components

**User request:**
```
Upgrade my app to Next.js 16 and enable Cache Components
```

**Agent workflow:**

```typescript
// 1. Initialize
init({ project_path: "." })

// 2. Use upgrade prompt
// Triggers: upgrade-nextjs-16 prompt
// - Updates package.json
// - Runs codemods
// - Updates config

// 3. Enable Cache Components
// Triggers: enable-cache-components prompt
// - Adds cache: "components" to next.config.ts
// - Shows migration examples

// 4. Verify with browser
browser_eval({ action: "start" })
browser_eval({ action: "navigate", url: "http://localhost:3000" })
browser_eval({ action: "console_messages" })
browser_eval({ action: "screenshot" })
browser_eval({ action: "close" })

// 5. Check runtime state
nextjs_call({ uri: "nextjs://errors" })
```

### Example 3: Implement Metadata Generation

**User request:**
```
Add proper SEO metadata to my blog post page
```

**Agent workflow:**

```typescript
// 1. Search docs
nextjs_docs({ 
  action: "search", 
  query: "generateMetadata" 
})

// 2. Get detailed docs
nextjs_docs({ 
  action: "get", 
  path: "/docs/app/api-reference/functions/generate-metadata" 
})

// 3. Implement based on docs
```

Example implementation:
```typescript
// app/blog/[slug]/page.tsx
import { Metadata } from 'next'

interface Props {
  params: { slug: string }
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await fetch(`${process.env.API_URL}/posts/${params.slug}`)
    .then(res => res.json())
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [{ url: post.image }],
    },
  }
}

export default async function BlogPost({ params }: Props) {
  const post = await fetch(`${process.env.API_URL}/posts/${params.slug}`)
    .then(res => res.json())
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  )
}
```

### Example 4: Migrate to Cache Components

**User request:**
```
Convert my data fetching component to use Cache Components
```

**Before (traditional approach):**
```typescript
// app/posts/page.tsx
export const revalidate = 3600

export default async function PostsPage() {
  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json())
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  )
}
```

**Agent searches docs:**
```typescript
nextjs_docs({ 
  action: "search", 
  query: "use cache directive" 
})

// Then gets resource
// (agent has cache-components://core-mechanics available)
```

**After (Cache Components):**
```typescript
// app/posts/page.tsx
import { Posts } from './Posts'

export default function PostsPage() {
  return <Posts />
}

// app/posts/Posts.tsx
"use cache"
export const revalidate = 3600

export async function Posts() {
  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json())
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  )
}
```

## Configuration Patterns

### next.config.ts with Cache Components

```typescript
import type { NextConfig } from 'next'

const config: NextConfig = {
  experimental: {
    cache: 'components',  // Enable Cache Components
  },
}

export default config
```

### Environment Variables

Use environment variables for API keys and sensitive data:

```bash
# .env.local
API_URL=https://api.example.com
NEXTAUTH_SECRET=your-secret-here
NEXTAUTH_URL=http://localhost:3000
```

Access in code:
```typescript
const apiUrl = process.env.API_URL
const secret = process.env.NEXTAUTH_SECRET
```

### MCP Auto-Discovery

Next.js 16+ automatically enables MCP at:
```
http://localhost:{PORT}/_next/mcp
```

No configuration needed. The `next-devtools-mcp` server discovers it automatically.

## Troubleshooting

### Issue: Runtime diagnostics not available

**Symptoms:** `nextjs_index` returns empty or `nextjs_call` fails

**Solution:**
1. Verify Next.js 16+ is installed: `npm list next`
2. Ensure dev server is running: `npm run dev`
3. Check dev server logs for MCP endpoint
4. Verify port (default 3000, check `next.config.ts` for custom port)

### Issue: Documentation search returns no results

**Symptoms:** `nextjs_docs` with `action: "search"` returns empty

**Solution:**
1. Try broader search terms (e.g., "metadata" instead of "generateMetadata function")
2. Check spelling
3. Try `routerType: "all"` to search both App and Pages Router docs
4. Use multiple searches for complex topics

### Issue: Browser automation fails on first run

**Symptoms:** `browser_eval` with `action: "start"` times out

**Solution:**
1. Playwright installs browsers on first run (this is automatic but takes time)
2. Increase timeout expectations for first use
3. On Windows 11 with Codex, ensure environment variables are set in `.codex/config.toml`
4. Run manually once: `npx playwright install chromium`

### Issue: Init tool not being called

**Symptoms:** Agent doesn't use Next.js docs properly, context is missing

**Solution:**
1. Explicitly prompt: "Use the init tool to set up Next.js DevTools context"
2. Add auto-init instruction to agent config (see Core Workflow section)
3. Verify MCP server is connected (check agent's MCP status)

### Issue: Upgrade prompt makes incorrect changes

**Symptoms:** Breaking changes after running upgrade-nextjs-16 prompt

**Solution:**
1. Always commit code before running upgrades
2. Review changes made by codemods
3. Use `nextjs_docs` to verify migration patterns
4. Test with `browser_eval` before deploying
5. Check `nextjs://errors` for runtime issues

## Best Practices

### 1. Documentation-First Approach

**Always** use `nextjs_docs` before implementing Next.js features:

```typescript
// Search first
nextjs_docs({ action: "search", query: "feature-name" })

// Get detailed docs
nextjs_docs({ action: "get", path: "/docs/..." })

// Then implement based on official documentation
```

### 2. Runtime Diagnostics Over Browser Console

For Next.js 16+ projects, prefer built-in diagnostics:

```typescript
// ✅ Preferred
nextjs_call({ uri: "nextjs://errors" })

// ❌ Avoid when above is available
browser_eval({ action: "console_messages" })
```

Browser automation is still valuable for visual verification and interaction testing.

### 3. Initialize Every Session

Make `init` the first tool call:

```typescript
// At start of every Next.js session
init({ project_path: "." })
```

Or automate with agent configuration (see Core Workflow).

### 4. Two-Step Documentation Workflow

Don't skip the search step:

```typescript
// Step 1: Search to find relevant docs
nextjs_docs({ action: "search", query: "..." })

// Step 2: Get full content from search results
nextjs_docs({ action: "get", path: "..." })
```

This ensures you get the most relevant and up-to-date documentation.

### 5. Verify Changes with Multiple Tools

After making changes, use a combination:

```typescript
// 1. Check runtime errors
nextjs_call({ uri: "nextjs://errors" })

// 2. Visual verification
browser_eval({ action: "start" })
browser_eval({ action: "navigate", url: "http://localhost:3000" })
browser_eval({ action: "screenshot" })
browser_eval({ action: "close" })

// 3. Check routes structure
nextjs_call({ uri: "nextjs://routes" })
```

## Common User Prompts

Recognize and handle these user requests:

- "Help me debug my Next.js app" → Use `nextjs_call` for errors
- "Upgrade to Next.js 16" → Use upgrade-nextjs-16 prompt
- "Enable Cache Components" → Use enable-cache-components prompt
- "Search Next.js docs for X" → Use `nextjs_docs` two-step workflow
- "Test my page at localhost:3000" → Use `browser_eval` workflow
- "What errors are showing?" → Use `nextjs_call` with `nextjs://errors`
- "Show my routes" → Use `nextjs_call` with `nextjs://routes`
- "Check the dev server logs" → Use `nextjs_call` with `nextjs://logs`
- "Add metadata to my page" → Search docs, then implement `generateMetadata`
- "How do I configure X in Next.js?" → Use `nextjs_docs` to find config docs

## Summary

The `next-devtools-mcp` skill enables AI coding agents to:

1. **Initialize properly** with the `init` tool (critical first step)
2. **Access official Next.js documentation** on-demand with `nextjs_docs`
3. **Query runtime diagnostics** from Next.js 16+ dev servers with `nextjs_index` and `nextjs_call`
4. **Automate browser testing** with `browser_eval` using Playwright
5. **Execute complex workflows** like upgrades and migrations with pre-configured prompts
6. **Access knowledge base resources** for Cache Components, migrations, and fundamentals

Always prioritize documentation lookup over assumptions, verify changes with runtime diagnostics, and test thoroughly with browser automation.
