---
name: godot-devtool-mcp-server
description: MCP server for AI-assisted Godot 4 project inspection, editing, validation, and runtime automation via WebSocket bridge
triggers:
  - inspect my Godot project structure
  - edit Godot scene nodes through MCP
  - install the godot-devtool plugin
  - run Godot project and capture runtime info
  - check Godot project settings and input actions
  - automate Godot scene validation
  - connect to Godot editor via WebSocket
  - simulate input in running Godot game
---

# Godot Devtool MCP Server

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`godot-devtool` is an MCP (Model Context Protocol) server that enables AI-assisted inspection, editing, validation, and runtime automation for Godot 4 projects. It provides 234 tools across project management, scene/node manipulation, script handling, editor integration, and live runtime control.

## Architecture Overview

```
MCP Client (Claude Code, Cursor, Cline, etc.)
  ↓ stdio
Node.js MCP Server (build/index.js)
  ↓
├─ Native routes (file inspection/editing)
├─ Headless Godot routes (scene/resource ops)
├─ WebSocket bridge (ws://127.0.0.1:8766)
│   ├─ Editor plugin (live scene editing, Inspector, UndoRedo)
│   └─ Runtime autoload (running game inspection, input simulation)
└─ Browser visualizer (local HTTP dashboard)
```

- **stdio MCP server**: Always runs over stdin/stdout, no exclusive port binding
- **Native routes**: Inspect/edit project files without opening Godot editor
- **Headless routes**: Call Godot CLI for scene/resource/script operations
- **Editor routes**: Live editing via bundled WebSocket plugin
- **Runtime routes**: Running-game scene tree, properties, input simulation, screenshots
- **Shared bridge**: Multiple MCP clients can use the same WebSocket port

## Installation

### 1. Install the MCP Server

Extract the release build or build from source:

```bash
git clone https://github.com/wangdiandao/godot-devtool.git
cd godot-devtool
npm install
npm run build
```

### 2. Configure Your MCP Client

**Claude Desktop / VS Code (JSON)**:
```json
{
  "mcpServers": {
    "godot-devtool": {
      "command": "node",
      "args": ["E:/godot-devtool/build/index.js"],
      "env": {
        "GODOT_PATH": "D:/Program Files/Godot/Godot_v4.x.exe",
        "GODOT_DEVTOOL_WS_PORT": "8766"
      }
    }
  }
}
```

**Codex Desktop (TOML)**:
```toml
[mcp_servers.godot-devtool]
command = "node"
args = ["E:/godot-devtool/build/index.js"]
env = { GODOT_PATH = "D:/Program Files/Godot/Godot_v4.x.exe", GODOT_DEVTOOL_WS_PORT = "8766" }
```

Environment variables:
- `GODOT_PATH`: Path to Godot executable (optional if `godot` is in PATH)
- `GODOT_DEVTOOL_WS_PORT`: WebSocket bridge port (default: 8766)

### 3. Install the Godot Plugin

The plugin enables live editor and runtime routes. Install via MCP tools:

```typescript
// Call from MCP client
plugin_install({
  projectPath: "E:/my-godot-project",
  overwrite: true,
  websocketPort: 8766
})
```

Then in Godot:
1. Open your project
2. Go to **Project → Project Settings → Plugins**
3. Enable **godot-devtool**

For runtime routes, the plugin also registers:
```
autoload/DevtoolRuntime = *res://addons/godot_devtool/runtime_bridge.gd
```

## Core Concepts

### Sessions and Context

Tools use `projectPath`, `context`, `sessionId`, and `runId` to identify targets:
- **projectPath**: Absolute path to Godot project directory
- **context**: `editor` or `runtime`
- **sessionId**: Disambiguate multiple editor/runtime connections
- **runId**: Track specific game instances from `run_project`

### Tool Discovery

Use `get_capabilities` to discover tools and filter by workflow:

```typescript
// Lightweight catalog (no schemas)
get_capabilities()

// Focused workflow with schemas
get_capabilities({
  toolNames: ["plugin_install", "plugin_status", "scene_tree_inspect"],
  includeSchemas: true
})

// Filter by route group
get_capabilities({
  routeGroup: "scene",
  includeSchemas: true
})

// Filter by transport
get_capabilities({
  transport: "editor_ws",
  includeSchemas: true
})
```

Workflow filters: `project_setup`, `live_editor`, `runtime_test`, `multi_instance`, `release_verify`

### Bridge Lifecycle

- Bridge port opens **on demand** when editor/runtime tools are called
- Port stays open while:
  - `run_project` is active
  - Editor or runtime client is connected
- Port releases after tool cleanup if no active sessions
- Use `plugin_status` and `plugin_cleanup_port` to inspect/manage the bridge

## Key Tool Categories

### Project Management

```typescript
// Get project metadata
get_project_info({
  projectPath: "E:/my-godot-project"
})

// Read project settings
project_get_settings({
  projectPath: "E:/my-godot-project",
  sections: ["application/config", "display/window"]
})

// Update project settings (dry run first)
project_update_settings({
  projectPath: "E:/my-godot-project",
  settings: {
    "application/config/name": "My Game",
    "display/window/size/viewport_width": 1920
  },
  dryRun: true
})

// Configure input actions
project_input_action({
  projectPath: "E:/my-godot-project",
  action: "jump",
  events: [
    { type: "InputEventKey", keycode: "KEY_SPACE" },
    { type: "InputEventJoypadButton", button_index: 0 }
  ]
})

// Run project
run_project({
  projectPath: "E:/my-godot-project",
  scene: "res://levels/level_01.tscn",
  debugCollisions: true,
  position: [100, 100],
  size: [1280, 720]
})

// List running instances
list_run_instances({
  projectPath: "E:/my-godot-project"
})

// Stop specific instance
stop_run_instance({
  projectPath: "E:/my-godot-project",
  runId: "run_12345"
})
```

### Scene and Node Operations

```typescript
// Inspect scene tree (native)
scene_tree_inspect({
  projectPath: "E:/my-godot-project",
  scenePath: "res://player.tscn",
  maxDepth: 3
})

// Inspect live editor scene (WebSocket)
editor_ws_scene_tree_inspect({
  projectPath: "E:/my-godot-project",
  maxDepth: 5
})

// Add node to live scene
editor_ws_node_add({
  projectPath: "E:/my-godot-project",
  parentPath: "Player/Body",
  nodeType: "Sprite2D",
  nodeName: "WeaponSprite",
  position: 1
})

// Update node property
editor_ws_node_set_property({
  projectPath: "E:/my-godot-project",
  nodePath: "Player/WeaponSprite",
  property: "texture",
  value: { type: "Resource", path: "res://sprites/sword.png" }
})

// Delete node with undo
editor_ws_node_delete({
  projectPath: "E:/my-godot-project",
  nodePath: "Player/OldSprite",
  undoable: true
})

// Save scene
editor_ws_scene_save({
  projectPath: "E:/my-godot-project"
})
```

### Script Management

```typescript
// List GDScript files
script_index({
  projectPath: "E:/my-godot-project",
  includeTests: true
})

// Read script
script_read({
  projectPath: "E:/my-godot-project",
  scriptPath: "res://player.gd"
})

// Write script
script_write({
  projectPath: "E:/my-godot-project",
  scriptPath: "res://enemy.gd",
  content: `extends CharacterBody2D

var speed = 200.0

func _physics_process(delta):
    var direction = Vector2.ZERO
    if Input.is_action_pressed("move_right"):
        direction.x += 1
    velocity = direction * speed
    move_and_slide()
`
})

// Check syntax
script_check_syntax({
  projectPath: "E:/my-godot-project",
  scriptPath: "res://player.gd"
})

// Create and attach script
script_create({
  projectPath: "E:/my-godot-project",
  scriptPath: "res://powerup.gd",
  template: "Node2D",
  attachTo: "res://scenes/powerup.tscn"
})
```

### Runtime Inspection & Automation

```typescript
// Inspect running game scene tree
runtime_ws_scene_tree_inspect({
  projectPath: "E:/my-godot-project",
  rootPath: "/root/Game",
  maxDepth: 4
})

// Get runtime node property
runtime_ws_node_get_property({
  projectPath: "E:/my-godot-project",
  nodePath: "/root/Game/Player",
  property: "position"
})

// Set runtime property
runtime_ws_node_set_property({
  projectPath: "E:/my-godot-project",
  nodePath: "/root/Game/Player",
  property: "health",
  value: { type: "int", value: 100 }
})

// Simulate input action
runtime_ws_input_action({
  projectPath: "E:/my-godot-project",
  action: "jump",
  pressed: true,
  strength: 1.0
})

// Capture screenshot
runtime_ws_screenshot({
  projectPath: "E:/my-godot-project",
  outputPath: "E:/screenshots/test_jump.png"
})

// Wait for node to appear
runtime_ws_wait_for_node({
  projectPath: "E:/my-godot-project",
  nodePath: "/root/Game/VictoryScreen",
  timeout: 5000
})

// Run QA assertion
runtime_ws_qa_assert({
  projectPath: "E:/my-godot-project",
  nodePath: "/root/Game/Player",
  property: "health",
  expected: { type: "int", value: 100 },
  operator: "greater_than",
  message: "Player should have full health at start"
})
```

### File & Resource Operations

```typescript
// List project files
file_list({
  projectPath: "E:/my-godot-project",
  directory: "res://sprites",
  pattern: "*.png",
  recursive: true
})

// Search in files
file_search({
  projectPath: "E:/my-godot-project",
  query: "extends CharacterBody2D",
  paths: ["res://scripts"],
  filePattern: "*.gd"
})

// Load resource metadata
resource_load({
  projectPath: "E:/my-godot-project",
  resourcePath: "res://player.tscn",
  shallow: true
})

// Build dependency graph
resource_dependency_graph({
  projectPath: "E:/my-godot-project",
  resourcePath: "res://levels/level_01.tscn",
  direction: "forward",
  maxDepth: 3
})
```

## Common Workflows

### 1. Project Setup & Inspection

```typescript
// Verify Godot installation
get_godot_version()

// Discover available tools
get_capabilities({
  routeGroup: "project",
  includeSchemas: true
})

// Get project info
get_project_info({ projectPath: "E:/my-game" })

// Check project settings
project_get_settings({
  projectPath: "E:/my-game",
  sections: ["application", "display", "input"]
})

// Install plugin for live editing
plugin_install({
  projectPath: "E:/my-game",
  overwrite: true,
  websocketPort: 8766
})

// Verify plugin installation
plugin_status({ projectPath: "E:/my-game" })
```

### 2. Live Scene Editing

```typescript
// 1. Ensure editor is open with plugin enabled

// 2. Get current editor selection
editor_ws_selection_get({ projectPath: "E:/my-game" })

// 3. Inspect current scene
editor_ws_scene_tree_inspect({
  projectPath: "E:/my-game",
  maxDepth: 5
})

// 4. Add a new node
editor_ws_node_add({
  projectPath: "E:/my-game",
  parentPath: "Player",
  nodeType: "Area2D",
  nodeName: "HitBox"
})

// 5. Configure the node
editor_ws_node_set_property({
  projectPath: "E:/my-game",
  nodePath: "Player/HitBox",
  property: "collision_layer",
  value: { type: "int", value: 2 }
})

// 6. Add collision shape
editor_ws_node_add({
  projectPath: "E:/my-game",
  parentPath: "Player/HitBox",
  nodeType: "CollisionShape2D",
  nodeName: "Shape"
})

// 7. Save the scene
editor_ws_scene_save({ projectPath: "E:/my-game" })
```

### 3. Runtime Testing & QA

```typescript
// 1. Run the project
run_project({
  projectPath: "E:/my-game",
  scene: "res://test_level.tscn",
  debugCollisions: true
})

// 2. Wait for game to load
await sleep(2000)

// 3. Inspect runtime scene tree
runtime_ws_scene_tree_inspect({
  projectPath: "E:/my-game",
  rootPath: "/root"
})

// 4. Check initial state
runtime_ws_node_get_property({
  projectPath: "E:/my-game",
  nodePath: "/root/Game/Player",
  property: "position"
})

// 5. Simulate input
runtime_ws_input_action({
  projectPath: "E:/my-game",
  action: "move_right",
  pressed: true
})

await sleep(1000)

runtime_ws_input_action({
  projectPath: "E:/my-game",
  action: "move_right",
  pressed: false
})

// 6. Verify movement
runtime_ws_qa_assert({
  projectPath: "E:/my-game",
  nodePath: "/root/Game/Player",
  property: "position.x",
  expected: { type: "float", value: 0 },
  operator: "greater_than",
  message: "Player should move right"
})

// 7. Capture screenshot
runtime_ws_screenshot({
  projectPath: "E:/my-game",
  outputPath: "E:/test_results/player_moved.png"
})

// 8. Stop the test
stop_project({ projectPath: "E:/my-game" })
```

### 4. Batch Scene Validation

```typescript
// List all scenes
const scenes = await file_list({
  projectPath: "E:/my-game",
  directory: "res://",
  pattern: "*.tscn",
  recursive: true
})

// Check each scene for common issues
for (const scene of scenes.files) {
  // Inspect scene tree
  const tree = await scene_tree_inspect({
    projectPath: "E:/my-game",
    scenePath: scene.path,
    maxDepth: 10
  })
  
  // Check for orphan nodes (no parent reference)
  // Check for missing scripts
  // Check for invalid resource paths
  
  // Run syntax check on attached scripts
  const scriptsToCheck = tree.nodes
    .filter(n => n.script)
    .map(n => n.script)
  
  for (const scriptPath of scriptsToCheck) {
    await script_check_syntax({
      projectPath: "E:/my-game",
      scriptPath: scriptPath
    })
  }
}
```

## Configuration

### Environment Variables

- `GODOT_PATH`: Path to Godot executable (required if not in PATH)
- `GODOT_DEVTOOL_WS_PORT`: WebSocket bridge port (default: 8766)

### Project Settings Integration

The MCP server reads and writes `project.godot` using native Godot syntax:

```typescript
// Add autoload
add_autoload({
  projectPath: "E:/my-game",
  name: "GameManager",
  path: "res://singletons/game_manager.gd",
  enabled: true
})

// Configure display settings
project_update_settings({
  projectPath: "E:/my-game",
  settings: {
    "display/window/size/viewport_width": 1920,
    "display/window/size/viewport_height": 1080,
    "display/window/size/mode": 3, // fullscreen
    "display/window/vsync/vsync_mode": 1
  }
})
```

### Input Actions

Use native Godot event syntax:

```typescript
project_input_action({
  projectPath: "E:/my-game",
  action: "attack",
  events: [
    {
      type: "InputEventKey",
      keycode: "KEY_CTRL"
    },
    {
      type: "InputEventMouseButton",
      button_index: 1 // MOUSE_BUTTON_LEFT
    },
    {
      type: "InputEventJoypadButton",
      button_index: 0, // BUTTON_A
      device: -1 // any gamepad
    }
  ],
  deadzone: 0.5
})
```

## Property Value Syntax

Use structured Variant types for node properties:

```typescript
// Basic types
{ type: "int", value: 42 }
{ type: "float", value: 3.14 }
{ type: "bool", value: true }
{ type: "String", value: "Hello" }

// Vector types
{ type: "Vector2", value: [10, 20] }
{ type: "Vector3", value: [1, 2, 3] }
{ type: "Color", value: [1.0, 0.5, 0.0, 1.0] } // RGBA

// Resources
{ type: "Resource", path: "res://sprites/player.png" }

// NodePath
{ type: "NodePath", value: "../OtherNode" }

// Arrays
{ type: "Array", value: [1, 2, 3] }

// Dictionaries
{ type: "Dictionary", value: { "key1": "value1", "key2": 42 } }
```

## Troubleshooting

### Plugin Not Connecting

```typescript
// 1. Check plugin status
plugin_status({ projectPath: "E:/my-game" })

// 2. Check if port is blocked
plugin_cleanup_port({
  projectPath: "E:/my-game",
  websocketPort: 8766,
  kill: false // inspect only
})

// 3. If stale, kill and reinstall
plugin_cleanup_port({
  projectPath: "E:/my-game",
  websocketPort: 8766,
  kill: true
})

plugin_reload({ projectPath: "E:/my-game" })
```

### Runtime Bridge Not Available

1. Ensure the plugin is installed and enabled
2. Verify autoload registration:
   ```typescript
   get_autoload({
     projectPath: "E:/my-game",
     name: "DevtoolRuntime"
   })
   ```
3. Run the project from Godot (or use `run_project`)
4. Check bridge status:
   ```typescript
   plugin_status({ projectPath: "E:/my-game" })
   ```

### Multiple Game Instances

```typescript
// List all running instances
list_run_instances({ projectPath: "E:/my-game" })

// Use runId for specific instance operations
runtime_ws_scene_tree_inspect({
  projectPath: "E:/my-game",
  runId: "run_12345"
})

// Stop specific instance
stop_run_instance({
  projectPath: "E:/my-game",
  runId: "run_12345"
})
```

### Schema Too Large

Filter `get_capabilities` requests:

```typescript
// Bad: returns all 234 tool schemas
get_capabilities({ includeSchemas: true }) // REJECTED

// Good: filter first
get_capabilities({
  routeGroup: "scene",
  transport: "editor_ws",
  includeSchemas: true
})

// Or specify exact tools
get_capabilities({
  toolNames: ["editor_ws_scene_tree_inspect", "editor_ws_node_add"],
  includeSchemas: true
})
```

### Editor Changes Not Saving

```typescript
// Explicit save after edits
editor_ws_scene_save({ projectPath: "E:/my-game" })

// Check if scene is modified
editor_ws_selection_get({ projectPath: "E:/my-game" })
// Returns: { sceneModified: true, ... }
```

## Browser Visualizer

Start a local dashboard to monitor bridge status:

```typescript
// Start visualizer on port 3456
browser_visualizer_start({
  projectPath: "E:/my-game",
  port: 3456
})

// Open http://127.0.0.1:3456/ in browser
// Shows:
// - Bridge connection status
// - Connected editor/runtime clients
// - Active sessions and run IDs
// - Available screenshot/scene/input routes
// - Pending command queue

// Stop visualizer
browser_visualizer_stop({
  projectPath: "E:/my-game"
})
```

## Example: Full Integration Test

```typescript
// 1. Setup
const projectPath = "E:/my-game"

await plugin_install({
  projectPath,
  overwrite: true,
  websocketPort: 8766
})

// 2. Configure input
await project_input_action({
  projectPath,
  action: "test_jump",
  events: [{ type: "InputEventKey", keycode: "KEY_J" }]
})

// 3. Create test scene
await scene_create({
  projectPath,
  scenePath: "res://test_scenes/jump_test.tscn",
  rootType: "Node2D"
})

// 4. Add player node (assuming editor is open)
await editor_ws_node_add({
  projectPath,
  parentPath: ".",
  nodeType: "CharacterBody2D",
  nodeName: "Player"
})

await editor_ws_node_set_property({
  projectPath,
  nodePath: "Player",
  property: "position",
  value: { type: "Vector2", value: [100, 100] }
})

await editor_ws_scene_save({ projectPath })

// 5. Run test
const runResult = await run_project({
  projectPath,
  scene: "res://test_scenes/jump_test.tscn"
})

await sleep(1000)

// 6. Simulate jump
await runtime_ws_input_action({
  projectPath,
  action: "test_jump",
  pressed: true
})

await sleep(100)

await runtime_ws_input_action({
  projectPath,
  action: "test_jump",
  pressed: false
})

// 7. Verify result
const finalPos = await runtime_ws_node_get_property({
  projectPath,
  nodePath: "/root/Node2D/Player",
  property: "position.y"
})

await runtime_ws_qa_assert({
  projectPath,
  nodePath: "/root/Node2D/Player",
  property: "position.y",
  expected: { type: "float", value: 100 },
  operator: "less_than",
  message: "Player should have moved up after jump"
})

// 8. Cleanup
await stop_project({ projectPath })
```

## Advanced: Multi-Instance Testing

```typescript
// Run multiple instances for multiplayer testing
const run1 = await run_project({
  projectPath: "E:/my-game",
  scene: "res://multiplayer_test.tscn",
  position: [0, 0]
})

const run2 = await run_project({
  projectPath: "E:/my-game",
  scene: "res://multiplayer_test.tscn",
  position: [800, 0]
})

// Control each instance separately
await runtime_ws_input_action({
  projectPath: "E:/my-game",
  runId: run1.runId,
  action: "move_right",
  pressed: true
})

await runtime_ws_input_action({
  projectPath: "E:/my-game",
  runId: run2.runId,
  action: "move_left",
  pressed: true
})

// Monitor both
const pos1 = await runtime_ws_node_get_property({
  projectPath: "E:/my-game",
  runId: run1.runId,
  nodePath: "/root/Game/Player",
  property: "position"
})

const pos2 = await runtime_ws_node_get_property({
  projectPath: "E:/my-game",
  runId: run2.runId,
  nodePath: "/root/Game/Player",
  property: "position"
})

// Cleanup
await stop_run_instance({ projectPath: "E:/my-game", runId: run1.runId })
await stop_run_instance({ projectPath: "E:/my-game", runId: run2.runId })
```

---

**Important Notes**:

1. Always call `get_capabilities` first to discover available tools for your workflow
2. Use `dryRun: true` for `project_update_settings` and other mutation operations
3. Filter `get_capabilities` by `routeGroup`, `transport`, or `toolNames` before requesting schemas
4. The WebSocket bridge opens on-demand and releases when idle (unless runtime is active)
5. Use `projectPath`, `context`, `sessionId`, and `runId` to disambiguate targets
6. Runtime routes require the project to be running with the autoload registered
7. Editor routes require the Godot editor to be open with the plugin enabled
8. For production use, test changes in a separate branch/backup before applying to main project
