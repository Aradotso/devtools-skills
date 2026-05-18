---
name: devtools-debugger-mcp-nodejs
description: MCP server for comprehensive Node.js debugging via Chrome DevTools Protocol with breakpoints, stepping, variable inspection, and source maps
triggers:
  - debug this Node.js application with breakpoints
  - set up MCP debugging for Node.js
  - inspect variables during Node.js execution
  - step through Node.js code with debugger
  - evaluate expressions in paused Node.js process
  - configure Chrome DevTools Protocol debugging
  - troubleshoot Node.js runtime with MCP server
  - add logpoints to Node.js application
---

# devtools-debugger-mcp-nodejs

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

An MCP (Model Context Protocol) server that exposes comprehensive Node.js debugging capabilities through the Chrome DevTools Protocol. Enables AI assistants to set breakpoints, step through code, inspect variables, evaluate expressions, analyze call stacks, and work with source maps — all programmatically.

## What It Does

This MCP server launches Node.js applications with the built-in inspector (`--inspect-brk=0`), connects via WebSocket to the Chrome DevTools Protocol, and exposes debugging operations as MCP tools. It handles:

- **Session management**: Launch/stop Node.js processes with debugging enabled
- **Breakpoints**: Set line breakpoints, conditional breakpoints, logpoints, and pause-on-exceptions
- **Execution control**: Resume, step over/into/out, continue to location, restart frames
- **Inspection**: Explore scopes (locals, closures, `this`), drill into object properties
- **Evaluation**: Execute JavaScript expressions in paused call frames
- **Console capture**: Buffer and retrieve console output between pauses
- **Source maps**: Full TypeScript and transpiled code debugging support
- **Script management**: List loaded scripts, fetch sources, blackbox patterns

## Installation

```bash
npm install devtools-debugger-mcp
```

Or globally:

```bash
npm install -g devtools-debugger-mcp
```

## Configuration

### MCP Settings

Add to your MCP client configuration (e.g., Claude Desktop's `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "devtools-debugger": {
      "command": "node",
      "args": ["/path/to/devtools-debugger-mcp/dist/index.js"]
    }
  }
}
```

Or if installed globally:

```json
{
  "mcpServers": {
    "devtools-debugger": {
      "command": "devtools-debugger-mcp"
    }
  }
}
```

### Output Format

Set default response format (`text`, `json`, or `both`):

```javascript
// Via set_output_format tool (defaults to 'text')
{
  "tool": "set_output_format",
  "params": { "format": "json" }
}
```

Individual tools can override with their own `format` parameter.

## Core Debugging Workflow

### 1. Start Debug Session

```javascript
// Launch Node.js script with inspector
{
  "tool": "start_node_debug",
  "params": {
    "scriptPath": "/absolute/path/to/app.js",
    "format": "text"  // optional: 'text' | 'json' | 'both'
  }
}
```

Returns initial pause at first line with `pauseId` and top frame info.

**With arguments and environment:**

```javascript
{
  "tool": "start_node_debug",
  "params": {
    "scriptPath": "/path/to/server.js",
    "args": ["--port", "3000"],
    "env": {
      "NODE_ENV": "development",
      "DEBUG": "*"
    }
  }
}
```

### 2. Set Breakpoints

**File path + line (1-based):**

```javascript
{
  "tool": "set_breakpoint",
  "params": {
    "filePath": "/path/to/app.js",
    "line": 42
  }
}
```

**Conditional breakpoint:**

```javascript
{
  "tool": "set_breakpoint_condition",
  "params": {
    "filePath": "/path/to/users.js",
    "line": 15,
    "condition": "user.age > 18"
  }
}
```

**URL regex breakpoint (for modules/packages):**

```javascript
{
  "tool": "set_breakpoint_condition",
  "params": {
    "urlRegex": ".*express.*",
    "line": 100,
    "condition": "req.method === 'POST'"
  }
}
```

**Logpoint (logs message without pausing):**

```javascript
{
  "tool": "add_logpoint",
  "params": {
    "filePath": "/path/to/api.js",
    "line": 28,
    "message": "Request received: {req.url}"
  }
}
```

### 3. Exception Breakpoints

```javascript
{
  "tool": "set_exception_breakpoints",
  "params": {
    "state": "uncaught"  // 'none' | 'uncaught' | 'all'
  }
}
```

### 4. Resume and Step

**Resume to next breakpoint:**

```javascript
{
  "tool": "resume_execution",
  "params": {
    "includeScopes": true,
    "includeStack": true,
    "includeConsole": true,
    "format": "text"
  }
}
```

**Step over current line:**

```javascript
{
  "tool": "step_over",
  "params": {
    "includeScopes": true,
    "includeConsole": true
  }
}
```

**Step into function:**

```javascript
{
  "tool": "step_into",
  "params": {
    "includeStack": true
  }
}
```

**Step out of current function:**

```javascript
{
  "tool": "step_out",
  "params": {
    "includeScopes": true
  }
}
```

**Continue to specific location:**

```javascript
{
  "tool": "continue_to_location",
  "params": {
    "filePath": "/path/to/app.js",
    "line": 55,
    "column": 10  // optional
  }
}
```

### 5. Inspect Variables and Scopes

**Current scope (locals, closures, this):**

```javascript
{
  "tool": "inspect_scopes",
  "params": {
    "maxProps": 20,          // max properties per object
    "pauseId": "pause123",   // optional, defaults to current
    "frameIndex": 0,         // optional, defaults to 0 (top frame)
    "includeThisPreview": true,
    "format": "text"
  }
}
```

**Drill into object properties:**

```javascript
// First get objectId from inspect_scopes or evaluate_expression
{
  "tool": "get_object_properties",
  "params": {
    "objectId": "object:123",
    "maxProps": 50
  }
}
```

### 6. Evaluate Expressions

```javascript
{
  "tool": "evaluate_expression",
  "params": {
    "expr": "user.profile.email",
    "pauseId": "pause123",     // optional
    "frameIndex": 0,           // optional, which frame to eval in
    "returnByValue": true,     // optional, serialize result
    "format": "json"
  }
}
```

**Evaluate with side effects:**

```javascript
{
  "tool": "evaluate_expression",
  "params": {
    "expr": "items.push({ id: 5, name: 'test' }); items.length"
  }
}
```

### 7. Call Stack Inspection

```javascript
{
  "tool": "list_call_stack",
  "params": {
    "depth": 10,              // optional, max frames
    "pauseId": "pause123",    // optional
    "includeThis": true,      // optional, include 'this' preview
    "format": "text"
  }
}
```

### 8. Pause Information

```javascript
{
  "tool": "get_pause_info",
  "params": {
    "pauseId": "pause123",    // optional, defaults to current
    "format": "text"
  }
}
```

Returns pause reason (breakpoint, exception, step, etc.) and location.

### 9. Console Output

```javascript
{
  "tool": "read_console",
  "params": {
    "format": "text"
  }
}
```

Retrieves console messages buffered since last step/resume. Console is also auto-included when `includeConsole: true` on step/resume tools.

### 10. Stop Session

```javascript
{
  "tool": "stop_debug_session"
}
```

Kills the Node.js process and cleans up CDP connection.

## Script Management

### List Loaded Scripts

```javascript
{
  "tool": "list_scripts"
}
```

Returns all scripts loaded by Node.js (app files, node_modules, builtins).

### Get Script Source

```javascript
// By scriptId
{
  "tool": "get_script_source",
  "params": {
    "scriptId": "42"
  }
}

// By URL
{
  "tool": "get_script_source",
  "params": {
    "url": "file:///path/to/app.js"
  }
}
```

### Blackbox Scripts (Skip During Debugging)

```javascript
{
  "tool": "blackbox_scripts",
  "params": {
    "patterns": [
      "node_modules/express/*",
      "internal/*"
    ]
  }
}
```

Frames matching these patterns won't pause during step-into.

## Restart Frame

Re-execute a specific call frame:

```javascript
{
  "tool": "restart_frame",
  "params": {
    "frameIndex": 2,          // which frame to restart (0 = top)
    "pauseId": "pause123",    // optional
    "format": "text"
  }
}
```

## Advanced Patterns

### Debug TypeScript with Source Maps

Source maps are automatically detected and used. Just launch your compiled JS:

```javascript
{
  "tool": "start_node_debug",
  "params": {
    "scriptPath": "/path/to/dist/app.js"
  }
}

// Set breakpoints using original .ts file paths
{
  "tool": "set_breakpoint",
  "params": {
    "filePath": "/path/to/src/app.ts",
    "line": 42
  }
}
```

### Conditional Debugging Loop

```javascript
// 1. Start session
start_node_debug({ scriptPath: "/path/to/app.js" })

// 2. Set conditional breakpoint
set_breakpoint_condition({
  filePath: "/path/to/app.js",
  line: 25,
  condition: "count > 100"
})

// 3. Resume until condition met
resume_execution({ includeScopes: true, includeConsole: true })

// 4. Inspect when paused
inspect_scopes({ maxProps: 15 })
evaluate_expression({ expr: "count" })

// 5. Continue
resume_execution()
```

### Capture All Console Output

```javascript
// Resume with console capture
const result = await resume_execution({ includeConsole: true });

// Or read explicitly
const consoleOutput = await read_console({ format: "text" });
```

### Multi-Frame Inspection

```javascript
// Get full call stack
list_call_stack({ depth: 20, includeThis: true })

// Inspect different frames
inspect_scopes({ frameIndex: 0 })  // Current frame
inspect_scopes({ frameIndex: 1 })  // Caller frame
inspect_scopes({ frameIndex: 2 })  // Caller's caller

// Evaluate in different frame context
evaluate_expression({ expr: "localVar", frameIndex: 1 })
```

### Exception Debugging

```javascript
// Pause on all exceptions
set_exception_breakpoints({ state: "all" })

// Resume and wait for exception
const result = await resume_execution({ includeStack: true })

// When paused on exception:
get_pause_info()  // Shows exception details
list_call_stack({ depth: 10 })
inspect_scopes({ maxProps: 20 })
```

## Real-World Example: Debug Express API

```javascript
// 1. Start debugging an Express server
{
  "tool": "start_node_debug",
  "params": {
    "scriptPath": "/path/to/server.js",
    "env": {
      "PORT": "3000",
      "NODE_ENV": "development"
    }
  }
}

// 2. Set breakpoint in route handler
{
  "tool": "set_breakpoint",
  "params": {
    "filePath": "/path/to/routes/users.js",
    "line": 15
  }
}

// 3. Set logpoint to track requests
{
  "tool": "add_logpoint",
  "params": {
    "filePath": "/path/to/middleware/auth.js",
    "line": 8,
    "message": "Auth check for user: {req.user.id}"
  }
}

// 4. Pause only on errors
{
  "tool": "set_exception_breakpoints",
  "params": { "state": "uncaught" }
}

// 5. Resume and make HTTP request (externally)
{
  "tool": "resume_execution",
  "params": {
    "includeScopes": true,
    "includeConsole": true
  }
}

// 6. When paused at breakpoint, inspect request
{
  "tool": "evaluate_expression",
  "params": {
    "expr": "req.body",
    "returnByValue": true
  }
}

{
  "tool": "evaluate_expression",
  "params": {
    "expr": "req.headers['authorization']"
  }
}

// 7. Check database query in scope
{
  "tool": "inspect_scopes",
  "params": { "maxProps": 30 }
}

// 8. Step into database function
{
  "tool": "step_into",
  "params": { "includeStack": true }
}

// 9. Continue execution
{
  "tool": "resume_execution"
}

// 10. Stop when done
{
  "tool": "stop_debug_session"
}
```

## Real-World Example: Debug Async/Await

```javascript
// app.js
async function fetchUserData(userId) {
  const user = await db.findUser(userId);
  const posts = await db.findPosts(user.id);
  return { user, posts };
}

// Debugging session:

// 1. Start
start_node_debug({ scriptPath: "/path/to/app.js" })

// 2. Break at async function
set_breakpoint({ filePath: "/path/to/app.js", line: 2 })

// 3. Resume to breakpoint
resume_execution({ includeScopes: true })

// 4. Check userId parameter
evaluate_expression({ expr: "userId" })

// 5. Step over await (resumes until promise resolves)
step_over({ includeScopes: true, includeConsole: true })

// 6. Inspect resolved user object
evaluate_expression({ expr: "user" })
get_object_properties({ objectId: "object:user123", maxProps: 20 })

// 7. Continue
resume_execution()
```

## File Path Handling

- **Always use absolute paths** for `filePath` parameters
- Relative paths are NOT resolved automatically
- Internal conversion: file paths → `file://` URLs for CDP
- Line numbers are **1-based** (CDP internally uses 0-based)
- Column numbers are **1-based** when specified

```javascript
// ✅ Correct
set_breakpoint({ filePath: "/home/user/project/src/app.js", line: 42 })

// ❌ Wrong (relative path)
set_breakpoint({ filePath: "./src/app.js", line: 42 })
```

## Troubleshooting

### Session Won't Start

**Problem**: `start_node_debug` fails or hangs

**Solutions**:
- Ensure `scriptPath` is an absolute path
- Check that the script file exists and is readable
- Verify Node.js is in PATH
- Try with a simple script first (e.g., `console.log('test')`)

```javascript
// Test with minimal script
start_node_debug({ scriptPath: "/tmp/test.js" })
// test.js content: console.log('Hello');
```

### Breakpoint Not Hit

**Problem**: Breakpoint set but execution doesn't pause

**Solutions**:
- Verify file path matches exactly (use `list_scripts` to confirm)
- Check line number is valid (1-based, not 0-based)
- Ensure breakpoint isn't in unreachable code
- Try URL regex if file path doesn't match

```javascript
// List all scripts to find exact URL
list_scripts()

// Use URL regex instead of file path
set_breakpoint_condition({
  urlRegex: ".*app\\.js$",
  line: 42
})
```

### Source Maps Not Working

**Problem**: Breakpoints in TypeScript sources don't work

**Solutions**:
- Ensure source maps are generated (`"sourceMap": true` in tsconfig.json)
- Check `.map` files exist alongside compiled JS
- Verify source map paths are correct (relative or absolute)
- Use compiled JS path if source map lookup fails

```javascript
// If TypeScript breakpoints fail, use compiled JS path
set_breakpoint({ filePath: "/path/to/dist/app.js", line: 58 })
```

### Can't Inspect Large Objects

**Problem**: Objects truncated or not showing all properties

**Solutions**:
- Increase `maxProps` parameter
- Use `get_object_properties` to drill down
- Evaluate specific property paths with `evaluate_expression`

```javascript
// Get more properties
inspect_scopes({ maxProps: 100 })

// Or drill into specific object
evaluate_expression({ expr: "largeObject.specificProperty" })
```

### Console Output Missing

**Problem**: Console logs not captured

**Solutions**:
- Use `includeConsole: true` on step/resume operations
- Or call `read_console` explicitly after stepping
- Console buffer is cleared after each read

```javascript
// Include console in step
step_over({ includeConsole: true })

// Or read explicitly
read_console({ format: "text" })
```

### Session Cleanup

**Problem**: Zombie Node.js processes after debugging

**Solutions**:
- Always call `stop_debug_session` when done
- MCP server cleans up on disconnect, but explicit stop is better
- Check for orphaned Node.js processes: `ps aux | grep node`

```bash
# Kill orphaned debugger processes
pkill -f "node --inspect-brk"
```

### Multiple Pauses

**Problem**: Execution pauses unexpectedly

**Solutions**:
- Check for multiple breakpoints at same location
- Review exception breakpoint settings (`set_exception_breakpoints`)
- Use `get_pause_info` to understand why paused
- Remove breakpoints: restart session or use CDP commands directly

```javascript
// Check why paused
get_pause_info({ format: "text" })

// Disable exception breaks if too noisy
set_exception_breakpoints({ state: "none" })
```

## Tips and Best Practices

1. **Use absolute paths**: Always provide absolute file paths for breakpoints
2. **Include context**: Add `includeScopes`, `includeStack`, `includeConsole` on steps for richer debugging
3. **Blackbox dependencies**: Skip node_modules during step-into with `blackbox_scripts`
4. **Conditional breakpoints**: Use conditions to pause only when specific criteria met
5. **Logpoints over breakpoints**: Use logpoints for non-intrusive logging without pausing
6. **Pause on uncaught only**: Set `state: 'uncaught'` to avoid pausing on handled exceptions
7. **Clean up sessions**: Always call `stop_debug_session` when done
8. **Test with simple scripts**: Verify setup with minimal examples before complex debugging
9. **Check pauseId**: After each pause, note the `pauseId` for context-specific operations
10. **Frame-aware evaluation**: Use `frameIndex` to evaluate expressions in specific call frames

## Environment Variables

The MCP server itself doesn't require environment variables, but your Node.js scripts may:

```javascript
{
  "tool": "start_node_debug",
  "params": {
    "scriptPath": "/path/to/app.js",
    "env": {
      "DATABASE_URL": process.env.DATABASE_URL,
      "API_KEY": process.env.API_KEY,
      "NODE_ENV": "development"
    }
  }
}
```

Never hardcode secrets in debugging params — reference environment variables or use a `.env` file in your project.

## License

MIT
