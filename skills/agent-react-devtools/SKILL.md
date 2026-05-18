---
name: agent-react-devtools
description: Give AI agents eyes into React apps - inspect component trees, props, state, hooks, and profile rendering performance from the command line
triggers:
  - "inspect the React component tree"
  - "show me the component hierarchy"
  - "check React component props and state"
  - "profile React rendering performance"
  - "find slow React components"
  - "debug React re-renders"
  - "connect to React DevTools"
  - "search for React components"
---

# agent-react-devtools

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`agent-react-devtools` is a command-line interface to React DevTools that lets AI agents inspect running React applications. It provides programmatic access to component trees, props, state, hooks, and rendering performance data through a persistent background daemon.

## Installation

Install globally:

```bash
npm install -g agent-react-devtools
```

Or use directly with npx:

```bash
npx agent-react-devtools start
```

## Core Concepts

- **Daemon**: A persistent background process (default port 8097) that maintains connection state
- **Component IDs**: Components are labeled `@c1`, `@c2`, etc. for easy reference
- **Host Filtering**: HTML elements (`<div>`, `<span>`) are filtered by default to keep output compact
- **LLM-Optimized**: Output is token-efficient and structured for AI consumption

## Connecting Your React App

### Quick Setup (Recommended)

Auto-configure your project:

```bash
npx agent-react-devtools init
```

This detects your framework (Vite, Next.js, CRA) and patches the config automatically.

To undo:

```bash
npx agent-react-devtools uninit
```

### Manual Setup - One-Line Import

Add to your entry point (e.g., `src/main.tsx`):

```typescript
import "agent-react-devtools/connect";
```

Place this as the **first import** in your app entry point.

### Manual Setup - Vite Plugin

For Vite projects, use the plugin (no app code changes needed):

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { reactDevtools } from "agent-react-devtools/vite";

export default defineConfig({
  plugins: [
    reactDevtools(), // Must come before react()
    react()
  ],
});
```

With custom port:

```typescript
reactDevtools({ port: 8097, host: "localhost" })
```

### React Native Setup

React Native connects automatically:

```bash
agent-react-devtools start
npx react-native start
```

For physical devices, forward the port:

```bash
adb reverse tcp:8097 tcp:8097
```

## Key Commands

### Daemon Management

Start the daemon (required first step):

```bash
agent-react-devtools start
```

With custom port:

```bash
agent-react-devtools start --port 8097
```

Check status:

```bash
agent-react-devtools status
```

Stop daemon:

```bash
agent-react-devtools stop
```

### Component Inspection

**Get full component tree:**

```bash
agent-react-devtools get tree
```

**Limit tree depth:**

```bash
agent-react-devtools get tree --depth 3
```

**Include host components (div, span, etc.):**

```bash
agent-react-devtools get tree --all
```

**Get subtree from specific component:**

```bash
agent-react-devtools get tree @c5
```

**Inspect component details:**

```bash
agent-react-devtools get component @c6
```

Output shows props, state, and hooks:

```
@c6 [fn] TodoItem key=1
props:
  id: 1
  text: "Buy groceries"
  done: false
  onToggle: ƒ
hooks:
  State: false
  Callback: ƒ
```

**Search for components by name:**

```bash
agent-react-devtools find TodoItem
```

**Exact name match:**

```bash
agent-react-devtools find App --exact
```

**Count components by type:**

```bash
agent-react-devtools count
```

**List components with errors/warnings:**

```bash
agent-react-devtools errors
```

### Wait Commands (Useful in Scripts)

Wait for app connection:

```bash
agent-react-devtools wait --connected --timeout 30
```

Wait for specific component:

```bash
agent-react-devtools wait --component App --timeout 30
```

These exit with code 0 on success, code 1 on timeout.

### Performance Profiling

**Start profiling session:**

```bash
agent-react-devtools profile start
```

With named session:

```bash
agent-react-devtools profile start "user-interaction-test"
```

**Stop profiling:**

```bash
agent-react-devtools profile stop
```

**Find slowest components:**

```bash
agent-react-devtools profile slow
```

Limit results:

```bash
agent-react-devtools profile slow --limit 5
```

**Find components that re-render most:**

```bash
agent-react-devtools profile rerenders
```

**View commit timeline:**

```bash
agent-react-devtools profile timeline
```

**Inspect specific commit:**

```bash
agent-react-devtools profile commit 3
```

Or use commit label:

```bash
agent-react-devtools profile commit #3
```

**Get render report for specific component:**

```bash
agent-react-devtools profile report @c5
```

**Export profiling data:**

```bash
agent-react-devtools profile export results.json
```

**Compare two profile exports:**

```bash
agent-react-devtools profile diff before.json after.json
```

With threshold filter:

```bash
agent-react-devtools profile diff before.json after.json --threshold 2.0
```

## Common Workflows

### Debugging Component Issues

1. Start daemon and connect app:

```bash
agent-react-devtools start
# Run your app (it should auto-connect)
agent-react-devtools status
```

2. Find the problematic component:

```bash
agent-react-devtools find UserProfile
```

3. Inspect its state and props:

```bash
agent-react-devtools get component @c12
```

4. Check surrounding context:

```bash
agent-react-devtools get tree @c12 --depth 2
```

### Performance Investigation

1. Start profiling before interaction:

```bash
agent-react-devtools profile start "search-performance"
```

2. Perform the interaction in your app

3. Stop and analyze:

```bash
agent-react-devtools profile stop
agent-react-devtools profile slow --limit 10
agent-react-devtools profile rerenders --limit 10
```

4. Investigate specific component:

```bash
agent-react-devtools profile report @c8
```

### Automated Testing/Monitoring

Script example for CI or monitoring:

```bash
#!/bin/bash
agent-react-devtools start
agent-react-devtools wait --connected --timeout 30 || exit 1

# Check for components with errors
if agent-react-devtools errors | grep -q "@c"; then
  echo "Components have errors!"
  agent-react-devtools errors
  exit 1
fi

# Profile a workflow
agent-react-devtools profile start "ci-test"
# ... trigger app interactions ...
agent-react-devtools profile stop
agent-react-devtools profile export ci-results.json

# Check for slow components
SLOW_COUNT=$(agent-react-devtools profile slow --limit 5 | grep -c "avg:")
if [ "$SLOW_COUNT" -gt 3 ]; then
  echo "Too many slow components detected"
  agent-react-devtools profile slow
  exit 1
fi
```

### Using with agent-browser

When combining with `agent-browser` for automated interactions:

```bash
# IMPORTANT: Must use --headed mode
agent-browser --session devtools --headed open http://localhost:5173/
agent-react-devtools status  # Verify connection
agent-react-devtools profile start
# ... agent-browser interactions ...
agent-react-devtools profile stop
agent-react-devtools profile slow
```

**Note**: Headless mode does NOT work - the DevTools connection requires a headed browser.

## Configuration

### Environment Variables

- `REACT_DEVTOOLS_PORT` - Custom port for React Native apps

### Default Port

The daemon uses port 8097 by default. Override with:

```bash
agent-react-devtools start --port 9000
```

Then configure your app connection to match.

## Troubleshooting

### App Not Connecting

**Check daemon status:**

```bash
agent-react-devtools status
```

**Verify app is in development mode:**
- The connection only works in dev builds, not production

**Check console for connection errors:**
- Look for WebSocket connection messages in browser console

**Restart daemon:**

```bash
agent-react-devtools stop
agent-react-devtools start
```

### "No apps connected" After Init

**Verify import order:**
- The connect import must be the **first** import in your entry file

**Check Vite plugin order:**
- `reactDevtools()` must come **before** `react()` in plugins array

**Verify dev server is running:**
- The app must be actively running, not just built

### Empty Component Tree

**Remove host component filtering:**

```bash
agent-react-devtools get tree --all
```

**Check if app rendered:**

```bash
agent-react-devtools count
```

### Profile Commands Return No Data

**Ensure you stopped profiling:**

```bash
agent-react-devtools profile stop
```

Data is only collected after stopping the profile session.

### Timeout Waiting for Connection

**Increase timeout:**

```bash
agent-react-devtools wait --connected --timeout 60
```

**Check if port is blocked:**

```bash
lsof -i :8097
```

## Output Format Notes

- Component IDs (`@c1`, `@c2`) are persistent within a daemon session
- Error annotations: `⚠` for warnings, `✗` for errors
- Function indicators: `ƒ` marks function props/callbacks
- Tree symbols: `├─` for middle children, `└─` for last child
- Collapsed indicators: `... +N more` when output is truncated

## Integration with AI Assistants

Add to your project's `AGENTS.md` or `.cursorrules`:

```markdown
## React Component Inspection

Use `agent-react-devtools` to inspect the running React app:

1. Start daemon: `agent-react-devtools start`
2. Check connection: `agent-react-devtools status`
3. Browse tree: `agent-react-devtools get tree --depth 3`
4. Inspect component: `agent-react-devtools get component @cN`
5. Profile performance: `agent-react-devtools profile start` → interact → `profile stop` → `profile slow`

Always check `status` before other commands to ensure app is connected.
```

## License

MIT
