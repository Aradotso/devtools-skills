---
name: threejs-devtools-mcp
description: MCP server for real-time Three.js scene inspection, material editing, shader debugging, and performance monitoring from AI agents
triggers:
  - "inspect my Three.js scene"
  - "debug Three.js materials and shaders"
  - "show me the scene graph"
  - "optimize Three.js performance"
  - "edit materials in real time"
  - "generate React Three Fiber components"
  - "check for Three.js memory leaks"
  - "analyze rendering bottlenecks"
---

# threejs-devtools-mcp

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

MCP server providing 59 tools for inspecting and modifying Three.js scenes in real time. Works with vanilla Three.js, React Three Fiber, and any framework. Zero code changes required — connects via Chrome DevTools Protocol.

## Installation

### Claude Code
```bash
claude mcp add threejs-devtools-mcp -- npx threejs-devtools-mcp
```

### Claude Desktop
Add to `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "threejs-devtools-mcp": {
      "command": "npx",
      "args": ["-y", "threejs-devtools-mcp"]
    }
  }
}
```

### Cursor
Add to `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "threejs-devtools-mcp": {
      "command": "npx",
      "args": ["-y", "threejs-devtools-mcp"]
    }
  }
}
```

### Windsurf
Add to `~/.codeium/windsurf/mcp_config.json`:
```json
{
  "mcpServers": {
    "threejs-devtools-mcp": {
      "command": "npx",
      "args": ["-y", "threejs-devtools-mcp"]
    }
  }
}
```

### VS Code (Copilot)
Add to `.vscode/mcp.json`:
```json
{
  "servers": {
    "threejs-devtools-mcp": {
      "command": "npx",
      "args": ["-y", "threejs-devtools-mcp"]
    }
  }
}
```

## How It Works

1. **Start your dev server** — `npm run dev` or your usual command
2. **MCP server auto-detects port** from package.json and opens Chrome at `localhost:9222`
3. **Keep the browser tab open** — MCP connects via WebSocket bridge
4. **Ask the AI** — "show me the scene tree", "make the car red", etc.

The browser tab must stay open for tools to work. The MCP server injects a WebSocket bridge that communicates with the Three.js scene.

## Core Tool Categories

### Scene Inspection
- `get_scene_tree` — Full scene hierarchy with objects, materials, geometries
- `get_object_details` — Properties, transforms, visibility, parent/child info
- `find_objects` — Search by name, type, or material
- `get_cameras` — List all cameras with properties
- `get_lights` — All lights (ambient, directional, point, spot, hemisphere)

### Material & Shader Management
- `get_materials` — All materials with properties (color, opacity, metalness, roughness)
- `update_material` — Modify color, opacity, metalness, roughness, emissive, wireframe
- `get_shaders` — List custom ShaderMaterial and RawShaderMaterial
- `update_shader` — Edit vertex/fragment shaders, uniforms
- `get_textures` — All textures with size, format, mipmaps, anisotropy

### Object Manipulation
- `update_object_transform` — Position, rotation, scale
- `toggle_object_visibility` — Show/hide objects
- `get_object_bounds` — Bounding box and sphere
- `clone_object` — Duplicate with transform offset

### Performance Monitoring
- `get_performance_stats` — FPS, frame time, memory, draw calls, triangles
- `start_performance_monitoring` — Continuous tracking with warnings
- `get_memory_info` — Geometries, textures, programs, heap usage
- `get_render_info` — Draw calls, triangles, points, lines, programs

### Animation
- `get_animations` — All AnimationClip data
- `play_animation` — Start animation by name with loop/speed control
- `pause_animation` — Pause running animation
- `get_animation_state` — Current playback state

### Code Generation
- `generate_react_component` — Create React Three Fiber component from GLTF/GLB
- `generate_material_code` — Export material as Three.js or R3F code
- `export_scene` — Export scene as JSON or R3F JSX

### Debugging
- `toggle_overlay` — Show/hide in-browser devtools panel
- `check_common_issues` — Detect invisible objects, missing materials, zero-scale, etc.
- `get_object_world_position` — World space coordinates

## Workflow Examples

### Debugging Invisible Objects
```typescript
// User: "My model isn't showing up"

// 1. Get scene tree to find objects
await use_mcp_tool("threejs-devtools-mcp", "get_scene_tree", {});

// 2. Check common issues
await use_mcp_tool("threejs-devtools-mcp", "check_common_issues", {});

// 3. If object found but invisible, get details
await use_mcp_tool("threejs-devtools-mcp", "get_object_details", {
  objectPath: "Scene/MyModel"
});

// 4. Check if it's visible and has material
// If not visible:
await use_mcp_tool("threejs-devtools-mcp", "toggle_object_visibility", {
  objectPath: "Scene/MyModel",
  visible: true
});

// 5. Check if it's inside camera frustum
await use_mcp_tool("threejs-devtools-mcp", "get_object_world_position", {
  objectPath: "Scene/MyModel"
});
```

### Editing Materials
```typescript
// User: "Make the car red and metallic"

// 1. Find the car object
await use_mcp_tool("threejs-devtools-mcp", "find_objects", {
  name: "car"
});

// 2. Update material
await use_mcp_tool("threejs-devtools-mcp", "update_material", {
  materialPath: "Scene/Car/Body/material",
  properties: {
    color: "#ff0000",
    metalness: 0.8,
    roughness: 0.2
  }
});
```

### Performance Optimization
```typescript
// User: "Why is my scene laggy?"

// 1. Get current performance stats
await use_mcp_tool("threejs-devtools-mcp", "get_performance_stats", {});

// 2. Check render info for draw call count
await use_mcp_tool("threejs-devtools-mcp", "get_render_info", {});

// 3. Check memory usage
await use_mcp_tool("threejs-devtools-mcp", "get_memory_info", {});

// 4. Start continuous monitoring
await use_mcp_tool("threejs-devtools-mcp", "start_performance_monitoring", {
  duration: 10,
  interval: 1
});

// Identify issues:
// - High draw calls → merge geometries
// - High triangle count → use LOD
// - Memory growth → check for leaks, dispose unused resources
```

### Shader Debugging
```typescript
// User: "My custom shader isn't working"

// 1. List all shaders
await use_mcp_tool("threejs-devtools-mcp", "get_shaders", {});

// 2. Get shader source
// (shader details included in get_shaders response)

// 3. Update shader with fixes
await use_mcp_tool("threejs-devtools-mcp", "update_shader", {
  shaderPath: "Scene/CustomMesh/material",
  vertexShader: `
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform float time;
    varying vec2 vUv;
    void main() {
      gl_FragColor = vec4(vUv, sin(time), 1.0);
    }
  `,
  uniforms: {
    time: { value: 0.0 }
  }
});
```

### Generating React Components
```typescript
// User: "Create a React component from my GLTF model"

await use_mcp_tool("threejs-devtools-mcp", "generate_react_component", {
  modelPath: "/models/character.glb",
  componentName: "Character",
  includeAnimations: true,
  includeLights: false
});

// Returns JSX code ready to use:
/*
import { useGLTF, useAnimations } from '@react-three/drei'
import { useRef, useEffect } from 'react'

export function Character(props) {
  const group = useRef()
  const { nodes, materials, animations } = useGLTF('/models/character.glb')
  const { actions } = useAnimations(animations, group)
  
  useEffect(() => {
    actions['Idle']?.play()
  }, [actions])
  
  return (
    <group ref={group} {...props} dispose={null}>
      <mesh geometry={nodes.Body.geometry} material={materials.Skin} />
    </group>
  )
}
*/
```

## Object Path Format

Tools use hierarchical paths to identify objects:
- `Scene` — root scene
- `Scene/Player` — direct child named "Player"
- `Scene/Group/Mesh` — nested object
- `Scene/Car/(unnamed)/Wheel` — unnamed intermediate object

**Tip:** Name your objects for easier access:
```javascript
// Three.js
mesh.name = "player";

// React Three Fiber
<mesh name="player" />
```

## Common Material Properties

When using `update_material`:
```typescript
{
  color: "#ff0000",           // hex color
  opacity: 0.5,               // 0-1
  transparent: true,          // boolean
  metalness: 0.8,             // 0-1 (MeshStandardMaterial)
  roughness: 0.2,             // 0-1 (MeshStandardMaterial)
  emissive: "#00ff00",        // hex color
  emissiveIntensity: 0.5,     // 0-1
  wireframe: true,            // boolean
  side: "DoubleSide",         // "FrontSide" | "BackSide" | "DoubleSide"
  visible: true               // boolean
}
```

## Animation Control

```typescript
// Play animation
await use_mcp_tool("threejs-devtools-mcp", "play_animation", {
  clipName: "Walk",
  loop: true,
  timeScale: 1.0
});

// Pause
await use_mcp_tool("threejs-devtools-mcp", "pause_animation", {
  clipName: "Walk"
});

// Check state
await use_mcp_tool("threejs-devtools-mcp", "get_animation_state", {});
```

## Configuration

Create `threejs-devtools.config.json` in project root:
```json
{
  "port": 5173,
  "chromePath": "/usr/bin/google-chrome",
  "debugPort": 9222,
  "autoOpenOverlay": true,
  "performanceThresholds": {
    "fps": 30,
    "frameTime": 33,
    "drawCalls": 100
  }
}
```

**Environment variables:**
- `THREEJS_DEVTOOLS_PORT` — override dev server port
- `THREEJS_DEVTOOLS_CHROME_PATH` — custom Chrome/Chromium path
- `THREEJS_DEVTOOLS_DEBUG_PORT` — Chrome DevTools Protocol port

## In-Browser Overlay

Toggle with `toggle_overlay` tool or activated automatically. Provides:
- **Performance panel** — real-time FPS, frame time, memory
- **Scene graph** — interactive tree with expand/collapse
- **Material editor** — live color picker, sliders for metalness/roughness
- **Object inspector** — transform, bounds, visibility
- **3D preview** — isolated object rendering

## Troubleshooting

### Browser tab closes immediately
- Check if port is correct: `THREEJS_DEVTOOLS_PORT=3000 npx threejs-devtools-mcp`
- Verify dev server is running before starting MCP server

### Tools return "not connected"
- Ensure browser tab stays open
- Check browser console for WebSocket errors
- Restart MCP server if connection lost

### Objects not found
- Use `get_scene_tree` to see actual object paths
- Objects may be unnamed — shows as `(unnamed)` in path
- Wait for GLTF models to load before querying

### Material changes not visible
- Check if material is used by multiple objects
- Some properties require `transparent: true` (e.g., opacity < 1)
- ShaderMaterial requires manual uniform updates

### Performance monitoring shows zeros
- Ensure renderer.info.autoReset is not disabled
- Check if scene is actually rendering (camera, lights present)

### Memory leaks detected
- Call dispose() on geometries, materials, textures when removing objects
- Use `get_memory_info` to track resource counts over time
- Check for retained references in closures or event listeners

## Token-Efficient Practices

1. **Use `find_objects` before `get_scene_tree`** — narrower scope
2. **Chain related operations** — get details + update in one turn
3. **Cache object paths** — reuse in conversation context
4. **Use `check_common_issues` first** — catches 80% of problems
5. **Request specific properties** — not full object dumps

## React Three Fiber Integration

Works seamlessly with R3F. Use `ref` to name objects:
```jsx
import { useRef } from 'react'
import { useFrame } from '@react-three/fiber'

function Box() {
  const ref = useRef()
  
  useFrame((state, delta) => {
    ref.current.rotation.x += delta
  })
  
  return (
    <mesh ref={ref} name="rotating-box">
      <boxGeometry />
      <meshStandardMaterial color="orange" />
    </mesh>
  )
}
```

Then from AI:
```typescript
// "Stop the box rotation"
await use_mcp_tool("threejs-devtools-mcp", "get_object_details", {
  objectPath: "Scene/rotating-box"
});
// Developer removes useFrame hook based on AI suggestion
```

## Advanced: HTTP Transport (Cursor)

For environments where stdio doesn't work:
```json
{
  "mcpServers": {
    "threejs-devtools-mcp": {
      "command": "npx",
      "args": ["-y", "threejs-devtools-mcp", "--transport", "http"]
    }
  }
}
```

Server runs on `http://localhost:3000` by default. Set `THREEJS_DEVTOOLS_HTTP_PORT` to change.

## Example: Complete Debug Session

```typescript
// User: "My 3D character model loads but is invisible and the scene is slow"

// 1. Check scene structure
const sceneTree = await use_mcp_tool("threejs-devtools-mcp", "get_scene_tree", {});
// Found: Scene/Character with multiple children

// 2. Check common issues
const issues = await use_mcp_tool("threejs-devtools-mcp", "check_common_issues", {});
// Found: Character has scale (0,0,0)

// 3. Fix scale
await use_mcp_tool("threejs-devtools-mcp", "update_object_transform", {
  objectPath: "Scene/Character",
  scale: { x: 1, y: 1, z: 1 }
});

// 4. Check performance
const perf = await use_mcp_tool("threejs-devtools-mcp", "get_performance_stats", {});
// FPS: 15, Draw calls: 450

// 5. Get render info to understand why
const renderInfo = await use_mcp_tool("threejs-devtools-mcp", "get_render_info", {});
// 450 draw calls → character has 450 separate meshes

// Suggestion: Merge geometries or use instancing
// Character now visible, identified performance bottleneck
```
