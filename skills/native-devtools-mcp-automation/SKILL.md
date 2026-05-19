---
name: native-devtools-mcp-automation
description: MCP server for computer use & browser automation - screenshot, OCR, click, type, find_text, Chrome/Electron CDP, template matching on macOS, Windows & Android
triggers:
  - "automate a desktop application"
  - "take a screenshot and find text with OCR"
  - "control a native app with accessibility tree"
  - "automate Chrome or Electron with CDP"
  - "click or type into a window using coordinates"
  - "automate an Android device over ADB"
  - "use template matching to find UI elements"
  - "set up native devtools MCP server"
---

# Native Devtools MCP Automation

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`native-devtools-mcp` is an MCP server that gives AI agents direct control over native desktop apps, Chrome/Electron browsers, and Android devices. It provides screenshots, OCR, accessibility-first element lookup, input simulation, window management, Chrome DevTools Protocol (CDP), and ADB — all in one local server.

Works with Claude Desktop, Claude Code, Cursor, and any MCP-compatible client.

## Platform Support

- **macOS**: Full support with Accessibility tree dispatch (preferred), screenshots, OCR (Vision), input simulation
- **Windows**: UI Automation, screenshots, OCR (Windows Media OCR), input simulation
- **Android**: ADB-based screenshots, uiautomator text lookup, input, app management
- **Chrome/Electron**: CDP-based DOM automation for web content and Electron apps

## Installation

### Quick Start (no install)

```bash
npx -y native-devtools-mcp
```

### Global Install

```bash
npm install -g native-devtools-mcp
```

### Build from Source (Rust)

```bash
git clone https://github.com/sh3ll3x3c/native-devtools-mcp
cd native-devtools-mcp
cargo build --release
# Binary: ./target/release/native-devtools-mcp
```

### Setup Wizard

Run the setup wizard to configure permissions and MCP clients:

```bash
npx native-devtools-mcp setup
```

This will:
1. Check permissions (Accessibility and Screen Recording on macOS)
2. Detect MCP clients (Claude Desktop, Claude Code, Cursor)
3. Write the correct configuration

## MCP Client Configuration

### Claude Desktop (macOS)

Config file: `~/Library/Application Support/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "native-devtools": {
      "command": "/Applications/NativeDevtools.app/Contents/MacOS/native-devtools-mcp"
    }
  }
}
```

### Claude Desktop (Windows)

Config file: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "native-devtools": {
      "command": "C:\\path\\to\\native-devtools-mcp.exe"
    }
  }
}
```

### Claude Code / Cursor / Other MCP Clients

```json
{
  "mcpServers": {
    "native-devtools": {
      "command": "npx",
      "args": ["-y", "native-devtools-mcp"]
    }
  }
}
```

### Claude Code Auto-Approval

To avoid approving every tool call, add to `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": ["mcp__native-devtools__*"]
  }
}
```

## Three Approaches to Interaction

### 1. Visual (Universal)

Works with any app — games, Qt apps, custom renderers, anything without an accessible tree.

**Key Tools**: `take_screenshot`, `find_text`, `click`, `type_text`, `find_image`

### 2. AX Dispatch (macOS - Preferred for Native Apps)

Element-precise automation for AppKit/SwiftUI apps. Doesn't move the cursor or steal focus.

**Key Tools**: `take_ax_snapshot`, `ax_click`, `ax_set_value`, `ax_select`

### 3. CDP (Chrome/Electron)

DOM-level automation for web content and Electron apps.

**Key Tools**: `cdp_connect`, `cdp_find_elements`, `cdp_click`, `cdp_fill`, `cdp_navigate`

## Core Tools Reference

### Screenshot & OCR

```typescript
// Take a full screen screenshot
take_screenshot()

// Take a window screenshot
take_screenshot(window_id: number)

// Take a region screenshot
take_screenshot(
  x: number,
  y: number,
  width: number,
  height: number
)

// Find text with OCR
find_text(
  text: string,
  window_id?: number,
  x?: number,
  y?: number,
  width?: number,
  height?: number
)
```

### Input Simulation

```typescript
// Click at global coordinates
click(x: number, y: number)

// Click relative to window
click(x: number, y: number, window_id: number)

// Click relative to screenshot
click(x: number, y: number, screenshot_id: string)

// Double-click
click(x: number, y: number, double_click: true)

// Right-click
click(x: number, y: number, right_click: true)

// Drag
drag(
  from_x: number,
  from_y: number,
  to_x: number,
  to_y: number,
  window_id?: number
)

// Type text
type_text(text: string)

// Press key
press_key(key: string, modifiers?: string[])
// Examples: "Return", "Tab", "Escape"
// Modifiers: ["command"], ["control", "shift"]

// Scroll
scroll(delta_x: number, delta_y: number)
```

### Window Management

```typescript
// List all windows
list_windows()

// Focus a window
focus_window(window_id: number)

// Launch an app
launch_app(app_name: string, args?: string[])

// Quit an app
quit_app(app_name: string)

// Record window frames
record_window(
  window_id: number,
  duration_ms: number,
  interval_ms?: number
)
```

### macOS Accessibility Tree (AX Dispatch)

```typescript
// Take AX snapshot
take_ax_snapshot(
  window_id?: number,
  include_descriptions?: boolean
)
// Returns tree with element UIDs (a1, a2, a3...)

// Click an AX element
ax_click(uid: string)

// Set value (text fields, sliders)
ax_set_value(uid: string, value: string)

// Select item (menus, lists)
ax_select(uid: string)

// Inspect element details
ax_inspect(uid: string)
```

**AX Dispatch Flow Example**:

```typescript
// 1. Take AX snapshot of System Settings
const windows = await list_windows();
const settingsWindow = windows.find(w => w.app_name === "System Settings");
const snapshot = await take_ax_snapshot(settingsWindow.id);

// 2. Find the element you want (e.g., "a12" is the "Privacy & Security" button)
// The LLM reads the tree structure from the snapshot

// 3. Click it without moving the cursor
await ax_click("a12");

// 4. Take another snapshot to verify
const updatedSnapshot = await take_ax_snapshot(settingsWindow.id);
```

### Template Matching

```typescript
// Load an image template
load_image(
  path: string,
  name: string
)

// Find the template in a screenshot
find_image(
  template_name: string,
  screenshot_id?: string,
  window_id?: number,
  threshold?: number  // 0.0-1.0, default 0.8
)
```

**Template Matching Flow**:

```typescript
// 1. Save a reference image of a button/icon
// (manually crop from a screenshot)

// 2. Load it
await load_image("/path/to/button.png", "submit_button");

// 3. Take a screenshot
const screenshot = await take_screenshot();

// 4. Find the template
const matches = await find_image("submit_button", screenshot.id);

// 5. Click the first match
if (matches.length > 0) {
  await click(matches[0].x, matches[0].y);
}
```

### Chrome DevTools Protocol (CDP)

```typescript
// Connect to Chrome/Electron
cdp_connect(
  port: number,
  host?: string  // default "localhost"
)

// Navigate
cdp_navigate(url: string)

// Find elements (returns UIDs: d1, d2, d3...)
cdp_find_elements(
  query: string,
  limit?: number
)

// Take DOM snapshot
cdp_take_dom_snapshot()

// Click element
cdp_click(uid: string)

// Hover over element
cdp_hover(uid: string)

// Fill input field
cdp_fill(uid: string, value: string)

// Type into element
cdp_type(uid: string, text: string)

// Press key
cdp_press_key(key: string)
// Examples: "Enter", "Tab", "Escape"

// Wait for condition
cdp_wait_for(
  text?: string[],
  selector?: string,
  timeout_ms?: number
)

// Evaluate JavaScript
cdp_eval(expression: string)

// Handle alert/confirm/prompt
cdp_handle_dialog(accept: boolean, prompt_text?: string)

// Manage tabs
cdp_new_tab(url?: string)
cdp_close_tab(tab_id: string)
cdp_list_tabs()
cdp_switch_tab(tab_id: string)

// Inspect element
cdp_inspect_element(uid: string)

// Get element attributes
cdp_get_attributes(uid: string)

// Screenshot element
cdp_screenshot_element(uid: string)
```

**CDP Flow Example**:

```typescript
// 1. Launch Chrome with remote debugging
await launch_app(
  "Google Chrome",
  ["--remote-debugging-port=9222", "--user-data-dir=/tmp/chrome-profile"]
);

// 2. Connect
await cdp_connect(9222);

// 3. Navigate
await cdp_navigate("https://example.com");

// 4. Find elements
const elements = await cdp_find_elements("search");
// Returns: [{ uid: "d1", tag: "input", text: "", ... }, ...]

// 5. Fill and submit
await cdp_fill("d1", "search query");
await cdp_press_key("Enter");

// 6. Wait for results
await cdp_wait_for(["Results"], null, 5000);

// 7. Take DOM snapshot
const dom = await cdp_take_dom_snapshot();
```

### Android (ADB)

```typescript
// List connected devices
adb_devices()

// Take screenshot
adb_screenshot(device_id?: string)

// Find text (uiautomator)
adb_find_text(
  text: string,
  device_id?: string
)

// Tap coordinates
adb_tap(
  x: number,
  y: number,
  device_id?: string
)

// Type text
adb_type(text: string, device_id?: string)

// Press key
adb_press_key(
  key: string,
  device_id?: string
)
// Examples: "KEYCODE_HOME", "KEYCODE_BACK"

// Swipe
adb_swipe(
  from_x: number,
  from_y: number,
  to_x: number,
  to_y: number,
  duration_ms?: number,
  device_id?: string
)

// Launch app
adb_launch_app(
  package: string,
  device_id?: string
)

// Stop app
adb_stop_app(
  package: string,
  device_id?: string
)
```

## Common Patterns

### Pattern 1: Visual Navigation with OCR

```typescript
// 1. Take a screenshot
const screenshot = await take_screenshot();

// 2. Find the target text
const matches = await find_text("Submit");

// 3. Click the first match
if (matches.length > 0) {
  await click(matches[0].center_x, matches[0].center_y, screenshot.id);
}
```

### Pattern 2: AX Dispatch on macOS (Preferred)

```typescript
// 1. List windows
const windows = await list_windows();
const targetWindow = windows.find(w => w.title.includes("Notes"));

// 2. Take AX snapshot
const snapshot = await take_ax_snapshot(targetWindow.id, true);

// 3. Find element by role/label in the tree
// (LLM identifies "a5" is the "New Note" button from the snapshot)

// 4. Click without moving cursor
await ax_click("a5");

// 5. Set value in text field (e.g., "a8")
await ax_set_value("a8", "Meeting notes for 2026-05-18");
```

### Pattern 3: Web Automation with CDP

```typescript
// 1. Launch Chrome with debugging
await launch_app(
  "Google Chrome",
  ["--remote-debugging-port=9222", "--new-window", "https://github.com/login"]
);

// 2. Connect
await cdp_connect(9222);

// 3. Find login fields
const elements = await cdp_find_elements("login");
// Returns: [{ uid: "d1", tag: "input", ... }, { uid: "d2", tag: "input", type: "password", ... }]

// 4. Fill credentials from env
await cdp_fill("d1", process.env.GITHUB_USERNAME);
await cdp_fill("d2", process.env.GITHUB_PASSWORD);

// 5. Click submit button
const submitElements = await cdp_find_elements("Sign in");
await cdp_click(submitElements[0].uid);

// 6. Wait for redirect
await cdp_wait_for(null, "header", 10000);
```

### Pattern 4: Electron App Automation

```typescript
// 1. Launch Electron app with debugging
await launch_app(
  "Signal",
  ["--remote-debugging-port=9223"]
);

// 2. Connect CDP
await cdp_connect(9223);

// 3. Automate like a web app
await cdp_find_elements("compose");
await cdp_click("d1");
await cdp_type("d1", "Hello from automation!");
await cdp_press_key("Enter");
```

### Pattern 5: Template Matching for Custom UI

```typescript
// 1. Load icon template
await load_image("/path/to/gear-icon.png", "settings_icon");

// 2. Take screenshot
const screenshot = await take_screenshot();

// 3. Find icon
const matches = await find_image("settings_icon", screenshot.id, null, 0.85);

// 4. Click if found
if (matches.length > 0) {
  await click(matches[0].x, matches[0].y, screenshot.id);
}
```

### Pattern 6: Android UI Automation

```typescript
// 1. List devices
const devices = await adb_devices();
const deviceId = devices[0].id;

// 2. Take screenshot
const screenshot = await adb_screenshot(deviceId);

// 3. Find text
const matches = await adb_find_text("Settings", deviceId);

// 4. Tap
if (matches.length > 0) {
  await adb_tap(matches[0].center_x, matches[0].center_y, deviceId);
}

// 5. Type in a field
await adb_tap(500, 300, deviceId);  // Focus input
await adb_type("Hello Android", deviceId);
await adb_press_key("KEYCODE_ENTER", deviceId);
```

## Operational Safety

- **Hands off**: When the agent is clicking/typing, don't move your mouse or type. Real hardware inputs conflict with simulated ones.
- **Focus matters**: Ensure the target window is visible. If a popup steals focus, clicks may land in the wrong window.
- **Prefer AX Dispatch on macOS**: For native apps, use `take_ax_snapshot` + `ax_click` / `ax_set_value` to avoid moving the cursor and stealing focus.

## Permissions (macOS)

The server requires:
- **Accessibility**: For input simulation and AX tree access
- **Screen Recording**: For screenshots

Grant both in **System Settings → Privacy & Security → Accessibility** and **Screen Recording**.

Without these, clicks silently fail and screenshots return black rectangles.

## Troubleshooting

### macOS: Clicks don't work

- **Cause**: Missing Accessibility permission
- **Fix**: System Settings → Privacy & Security → Accessibility → enable the app

### macOS: Screenshots are black

- **Cause**: Missing Screen Recording permission
- **Fix**: System Settings → Privacy & Security → Screen Recording → enable the app

### CDP: Can't connect

- **Cause**: App not launched with `--remote-debugging-port`
- **Fix**: Use `launch_app` with the correct args:
  ```typescript
  await launch_app("Google Chrome", ["--remote-debugging-port=9222"]);
  ```

### ADB: No devices found

- **Cause**: USB debugging not enabled or device not connected
- **Fix**:
  1. Enable USB debugging on Android device (Settings → Developer options)
  2. Connect via USB or Wi-Fi (`adb tcpip 5555`, then `adb connect <ip>:5555`)
  3. Run `adb devices` to verify

### OCR finds nothing

- **Cause**: Text too small, low contrast, or obscured
- **Workarounds**:
  1. Use template matching instead (`load_image` + `find_image`)
  2. Use AX Dispatch on macOS (`take_ax_snapshot`)
  3. Use CDP for web content (`cdp_find_elements`)

### Windows: UI Automation elements missing

- **Cause**: Some Qt/Electron apps don't expose UI Automation
- **Workaround**: Use visual approach (screenshots + OCR) or CDP for Electron

## Real-World Examples

### Example 1: Automate System Settings on macOS

```typescript
// 1. Launch System Settings
await launch_app("System Settings");

// 2. Wait for window
const windows = await list_windows();
const settingsWindow = windows.find(w => w.app_name === "System Settings");

// 3. Take AX snapshot
const snapshot = await take_ax_snapshot(settingsWindow.id, true);

// 4. Find "Privacy & Security" in the sidebar (e.g., a12)
await ax_click("a12");

// 5. Take another snapshot to find the next element
const privacySnapshot = await take_ax_snapshot(settingsWindow.id, true);

// 6. Click "Screen Recording" (e.g., a25)
await ax_click("a25");
```

### Example 2: Fill a Web Form with CDP

```typescript
// 1. Launch Chrome
await launch_app("Google Chrome", [
  "--remote-debugging-port=9222",
  "--new-window",
  "https://example.com/contact"
]);

// 2. Connect CDP
await cdp_connect(9222);

// 3. Find form fields
const fields = await cdp_find_elements("contact form");
// Returns: [
//   { uid: "d1", tag: "input", placeholder: "Name", ... },
//   { uid: "d2", tag: "input", placeholder: "Email", ... },
//   { uid: "d3", tag: "textarea", placeholder: "Message", ... }
// ]

// 4. Fill the form
await cdp_fill("d1", "John Doe");
await cdp_fill("d2", "john@example.com");
await cdp_fill("d3", "This is a test message.");

// 5. Submit
const submitBtn = await cdp_find_elements("Submit");
await cdp_click(submitBtn[0].uid);

// 6. Wait for confirmation
await cdp_wait_for(["Thank you"], null, 5000);
```

### Example 3: Android App Testing

```typescript
// 1. List devices
const devices = await adb_devices();
const device = devices[0].id;

// 2. Launch app
await adb_launch_app("com.example.app", device);

// 3. Wait and screenshot
await new Promise(resolve => setTimeout(resolve, 2000));
const screenshot = await adb_screenshot(device);

// 4. Find and tap "Sign In" button
const signInMatches = await adb_find_text("Sign In", device);
if (signInMatches.length > 0) {
  await adb_tap(signInMatches[0].center_x, signInMatches[0].center_y, device);
}

// 5. Fill username
const usernameMatches = await adb_find_text("Username", device);
await adb_tap(usernameMatches[0].center_x, usernameMatches[0].center_y, device);
await adb_type(process.env.TEST_USERNAME, device);

// 6. Fill password
await adb_press_key("KEYCODE_TAB", device);
await adb_type(process.env.TEST_PASSWORD, device);

// 7. Submit
await adb_press_key("KEYCODE_ENTER", device);
```

### Example 4: Automate VS Code with CDP

```typescript
// 1. Launch VS Code with debugging
await launch_app("Visual Studio Code", ["--remote-debugging-port=9224"]);

// 2. Connect
await cdp_connect(9224);

// 3. Open command palette
await cdp_press_key("Meta+Shift+P");  // Meta = Cmd on macOS, Win on Windows

// 4. Wait for palette
await cdp_wait_for(null, ".quick-input-widget", 2000);

// 5. Type command
await cdp_type(".quick-input-widget input", "File: Open File");
await cdp_press_key("Enter");

// 6. Navigate file picker with JS
await cdp_eval(`
  document.querySelector('.monaco-inputbox input').value = '/path/to/file.js';
  document.querySelector('.monaco-inputbox input').dispatchEvent(new Event('input'));
`);
await cdp_press_key("Enter");
```

## Best Practices

1. **Always verify state**: Take a screenshot or snapshot after an action to confirm it succeeded.
2. **Use the right tool for the job**:
   - Native macOS apps → AX Dispatch
   - Web/Electron → CDP
   - Custom/legacy UI → Visual (screenshots + OCR or template matching)
3. **Handle timing**: Add `cdp_wait_for` or manual delays after navigation/clicks before the next action.
4. **Reference env vars for secrets**: Never hardcode credentials in automation scripts.
5. **Use `record_window` for debugging**: Record a window's state over time to understand UI behavior.
6. **Test permissions early**: Run `npx native-devtools-mcp setup` before writing automation scripts.

## Additional Resources

- [GitHub Repository](https://github.com/sh3ll3x3c/native-devtools-mcp)
- [NPM Package](https://www.npmjs.com/package/native-devtools-mcp)
- [Examples Directory](https://github.com/sh3ll3x3c/native-devtools-mcp/tree/master/examples)
- [Claude Desktop Setup Guide](https://github.com/sh3ll3x3c/native-devtools-mcp/blob/master/examples/claude-desktop-setup.md)
- [Android Quickstart](https://github.com/sh3ll3x3c/native-devtools-mcp/blob/master/examples/android-quickstart.md)
