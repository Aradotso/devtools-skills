---
name: wechat-devtools-mcp
description: WeChat Mini Program development automation via MCP - IDE control, debugging, testing, and deployment workflows
triggers:
  - "debug the WeChat mini program"
  - "preview the mini program QR code"
  - "check console errors in the WeChat IDE"
  - "upload this mini program to WeChat"
  - "test all pages for errors"
  - "take a screenshot of the current page"
  - "navigate to the login page and check logs"
  - "mock the WeChat payment API"
---

# wechat-devtools-mcp

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`wechat-devtools-mcp` wraps the WeChat Developer Tools CLI as an MCP (Model Context Protocol) server, enabling AI coding agents to control the WeChat IDE programmatically. It provides 7 aggregated tools covering the full mini program lifecycle: IDE management, build/deploy, automated testing, debugging, screenshots, and file operations.

**Architecture**: Thin MCP (7 tools) + Fat Skill (SOPs, parameter references, guardrails). The Skill is **required** — without it, AI agents cannot execute standard workflows correctly.

**Platform**: Cross-platform (Windows/macOS). Published to official [MCP Registry](https://modelcontextprotocol.io/).

## Installation

### 1. Install MCP Server

```bash
# Install uv if not present
pip install uv

# Install wechat-devtools-mcp globally
uv tool install wechat-devtools-mcp --force
```

**Upgrade**:
```bash
# Kill running instances first
taskkill /F /IM "wechat-devtools-mcp*" 2>/dev/null
uv tool upgrade wechat-devtools-mcp
```

### 2. Enable WeChat IDE Service Port

**Critical**: Open WeChat Developer Tools → Settings → Security → Service Port → Enable.

Verify with `wechat_ide(action='status')` — connection failure means the port is disabled.

### 3. Configure MCP Client

**Claude Desktop / Antigravity** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "wechat-devtools": {
      "command": "uvx",
      "args": ["wechat-devtools-mcp"],
      "env": {
        "WECHAT_DEVTOOLS_CLI": "C:\\Program Files (x86)\\Tencent\\微信web开发者工具\\cli.bat",
        "WECHAT_PROJECT_PATH": "D:\\Projects\\my-miniapp"
      }
    }
  }
}
```

**macOS** (Claude Code `.mcp.json` in project root):
```json
{
  "mcpServers": {
    "wechat-devtools": {
      "command": "/opt/homebrew/bin/uvx",
      "args": ["wechat-devtools-mcp"],
      "env": {
        "PATH": "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin",
        "WECHAT_DEVTOOLS_CLI": "/Applications/wechatwebdevtools.app/Contents/MacOS/cli",
        "WECHAT_PROJECT_PATH": "/Users/you/Projects/my-miniapp",
        "NODE_PATH": "/opt/homebrew/bin/node"
      }
    }
  }
}
```

**Environment Variables**:
- `WECHAT_DEVTOOLS_CLI`: Absolute path to CLI (`cli.bat` on Windows, `cli` on macOS)
- `WECHAT_PROJECT_PATH`: Absolute path to mini program project root
- Windows: Escape backslashes (`\\`), macOS: Use forward slashes

### 4. Install Skill (Required)

**Claude Code**:
```bash
npx -y skills add WaterTian/wechat-devtools-mcp/.agents/skills/wechat-devtools
```

**Other clients** (place in `.agents/skills/`):
```bash
git clone --depth 1 https://github.com/WaterTian/wechat-devtools-mcp.git .wdm-tmp
mkdir -p .agents/skills
cp -r .wdm-tmp/.agents/skills/wechat-devtools .agents/skills/
rm -rf .wdm-tmp
```

Result:
```
project/
└── .agents/skills/
    └── wechat-devtools/
        ├── SKILL.md                # Main SOPs + capability map
        └── references/
            └── tool_reference.md   # Full API parameter docs
```

## Core Tools

### 1. `wechat_ide` — IDE Lifecycle

**Actions**: `open`, `login`, `is_login`, `close`, `quit`, `status`

```python
# Open IDE and load project
wechat_ide(action='open')

# Check login status
wechat_ide(action='is_login')

# Get IDE connection status + MCP version
wechat_ide(action='status')

# Close project (keeps IDE running)
wechat_ide(action='close')

# Quit IDE entirely
wechat_ide(action='quit')
```

**Key patterns**:
- Always call `status` first to verify IDE connectivity
- `open` is idempotent — won't fail if already open
- Use `close` between test sessions, `quit` only when necessary

### 2. `wechat_build` — Build & Deploy

**Actions**: `compile`, `preview`, `upload`, `build_npm`, `cache_clean`

```python
# Full compilation
wechat_build(action='compile')

# Generate preview QR code
result = wechat_build(
    action='preview',
    extra_args={
        'qr_format': 'terminal',  # or 'image', 'base64'
        'qr_output': '/tmp/qr.png',
        'compile_condition': '{"pathName":"pages/index/index"}'
    }
)

# Upload to WeChat backend (production)
wechat_build(
    action='upload',
    extra_args={
        'version': '1.0.0',
        'desc': 'Initial release'
    }
)

# Build npm dependencies
wechat_build(action='build_npm')

# Clean cache before rebuild
wechat_build(action='cache_clean')
```

**QR formats**:
- `terminal`: ASCII art in console
- `image`: Save to `qr_output` path
- `base64`: Data URI string

### 3. `wechat_automator` — Automated Testing

**Actions**: `start`, `tap`, `input`, `element_info`, `set_data`, `call_method`, `call_wx`, `mock_wx`, `evaluate`, `page_stack`, `page_data`, `system_info`, `storage`

```python
# Start automation session
wechat_automator(action='start')

# Tap element by selector
wechat_automator(
    action='tap',
    extra_args={
        'selector': '.login-btn',
        'wait_for': 2000  # Wait 2s after tap
    }
)

# Input text
wechat_automator(
    action='input',
    extra_args={
        'selector': 'input.username',
        'text': 'testuser'
    }
)

# Get element properties
wechat_automator(
    action='element_info',
    extra_args={'selector': '.status-text'}
)

# Mock WeChat API
wechat_automator(
    action='mock_wx',
    extra_args={
        'api': 'request',
        'result': 'success',
        'data': '{"code": 200, "data": {"user": "mock"}}'
    }
)

# Get page data
wechat_automator(action='page_data')

# Execute JavaScript
wechat_automator(
    action='evaluate',
    extra_args={'code': 'getCurrentPages()[0].data.userInfo'}
)
```

**Selector syntax**:
- `.class-name` — Class selector
- `#id` — ID selector
- `view.item[data-id="123"]` — Attribute selector
- Use `>>>` for shadow DOM: `custom-component >>> .inner-element`

### 4. `wechat_inspector` — Runtime Logs

**Actions**: `console`, `cdp`

```python
# Capture console logs (10 seconds)
logs = wechat_inspector(
    action='console',
    extra_args={'duration': 10}
)

# Capture Chrome DevTools Protocol events
cdp_logs = wechat_inspector(
    action='cdp',
    extra_args={
        'duration': 5,
        'events': ['Network.requestWillBeSent', 'Runtime.consoleAPICalled']
    }
)
```

**Use cases**:
- Detect runtime errors before they crash
- Monitor network requests during user flow
- Capture console.log/warn/error for debugging

### 5. `wechat_screenshot` — Visual Testing

```python
# Screenshot current page
wechat_screenshot(extra_args={'full_page': True})

# Screenshot specific element
wechat_screenshot(extra_args={
    'selector': '.product-list',
    'full_page': False
})
```

Returns base64-encoded PNG. For long pages, automatically stitches scrolling captures.

### 6. `wechat_navigate` — Navigation + Log Capture

```python
# Navigate to page and capture CDP logs
wechat_navigate(extra_args={
    'page': 'pages/detail/detail',
    'query': 'id=123',
    'log_duration': 3
})
```

Combines `wx.navigateTo()` + `wechat_inspector(cdp)` in one call.

### 7. `wechat_file` — Project Introspection

**Actions**: `project_info`, `list_pages`, `read_page`, `read_file`

```python
# Get project.config.json
wechat_file(action='project_info')

# List all pages in app.json
wechat_file(action='list_pages')

# Read page source (WXML + WXSS + JS + JSON)
wechat_file(
    action='read_page',
    extra_args={'page_path': 'pages/index/index'}
)

# Read arbitrary file
wechat_file(
    action='read_file',
    extra_args={'file_path': 'utils/request.js'}
)
```

## Standard Operating Procedures (SOPs)

### SOP A: Initial Setup Verification

1. `wechat_ide(status)` — Verify IDE connectivity
2. `wechat_file(project_info)` — Confirm project loaded
3. `wechat_ide(is_login)` — Check login status
4. If not logged in: `wechat_ide(login)` and wait for user scan

### SOP B: UI Debugging Workflow

1. `wechat_automator(start)` — Begin session
2. `wechat_navigate(page='target/page', log_duration=3)`
3. `wechat_screenshot(full_page=True)` — Capture UI state
4. `wechat_inspector(console, duration=5)` — Check for errors
5. `wechat_automator(page_data)` — Inspect data bindings

### SOP C: Error Investigation

1. `wechat_inspector(console, duration=10)` — Capture logs
2. `wechat_automator(page_stack)` — Check navigation state
3. `wechat_file(read_page, page_path=<current>)` — Review source
4. `wechat_automator(evaluate, code='getApp().globalData')` — Check global state

### SOP D: Full Page Health Check

```python
pages = wechat_file(action='list_pages')
for page in pages['pages']:
    wechat_navigate(extra_args={'page': page, 'log_duration': 2})
    logs = wechat_inspector(action='console', extra_args={'duration': 2})
    if 'error' in logs.lower():
        print(f"❌ {page}: {logs}")
    else:
        print(f"✅ {page}: OK")
```

### SOP E: Mock-Based Integration Test

1. `wechat_automator(start)`
2. `wechat_automator(mock_wx, api='request', result='success', data=<mock_json>)`
3. `wechat_automator(tap, selector='.trigger-request-btn')`
4. `wechat_inspector(console, duration=3)` — Verify mock response handling
5. `wechat_automator(page_data)` — Check UI updated correctly

### SOP F: Deployment Workflow

1. `wechat_build(cache_clean)` — Fresh build
2. `wechat_build(compile)` — Check for errors
3. If errors: Stop and report
4. `wechat_build(preview)` — Generate test QR
5. User scans and validates
6. `wechat_build(upload, version=<semver>, desc=<changelog>)`

### SOP G: Page Parameter Discovery

```python
# Find page query params from source
source = wechat_file(action='read_page', extra_args={'page_path': 'pages/detail/detail'})
# Parse onLoad(options) in JS file
# Extract options keys: id, type, etc.
```

## Configuration Patterns

### Multi-Project Setup

For multiple mini programs, create project-specific `.mcp.json`:

```json
{
  "mcpServers": {
    "wechat-shop": {
      "command": "uvx",
      "args": ["wechat-devtools-mcp"],
      "env": {
        "WECHAT_DEVTOOLS_CLI": "C:\\...\\cli.bat",
        "WECHAT_PROJECT_PATH": "D:\\Projects\\shop-miniapp"
      }
    },
    "wechat-admin": {
      "command": "uvx",
      "args": ["wechat-devtools-mcp"],
      "env": {
        "WECHAT_DEVTOOLS_CLI": "C:\\...\\cli.bat",
        "WECHAT_PROJECT_PATH": "D:\\Projects\\admin-miniapp"
      }
    }
  }
}
```

Switch projects by calling the corresponding server's tools.

### Custom Compile Conditions

```python
# Target specific page on preview
wechat_build(
    action='preview',
    extra_args={
        'compile_condition': json.dumps({
            'pathName': 'pages/cart/cart',
            'query': 'from=share&id=456',
            'scene': 1007  # WeChat group share
        })
    }
)
```

## Troubleshooting

### "Connection refused" on all commands

**Cause**: Service port not enabled in IDE.

**Fix**: Settings → Security → Service Port → Enable. Restart IDE.

### `wechat_automator` commands timeout

**Cause**: Page not fully loaded or selector invalid.

**Fix**:
1. Add `wait_for` delay: `{'selector': '.btn', 'wait_for': 2000}`
2. Verify selector with `wechat_automator(element_info)`
3. Check page stack: `wechat_automator(page_stack)`

### Preview QR not generating

**Cause**: Compilation errors or missing `qr_format`.

**Fix**:
1. Run `wechat_build(compile)` first
2. Specify `qr_format`: `'terminal'`, `'image'`, or `'base64'`
3. For `image`, provide `qr_output` absolute path

### Screenshots are blank

**Cause**: Page render incomplete or IDE minimized.

**Fix**:
1. Add 2-3s delay before screenshot: `wechat_navigate()` then wait
2. Ensure IDE window is visible (not minimized)
3. Check if `selector` exists: `wechat_automator(element_info)`

### Mock not working

**Cause**: API name typo or wrong mock timing.

**Fix**:
1. Use exact API name: `request`, `getStorage`, `showToast` (case-sensitive)
2. Call `mock_wx` **before** triggering the API
3. Verify with `wechat_inspector(console)` to see actual API calls

### File operations return "not found"

**Cause**: Incorrect path format or file outside project.

**Fix**:
- Use forward slashes: `pages/index/index`
- No leading slash: `utils/api.js` not `/utils/api.js`
- Path relative to `WECHAT_PROJECT_PATH` root

## Best Practices

1. **Always start with `status`**: Verify IDE connectivity before workflows
2. **Log everything**: Capture console/CDP logs before and after critical operations
3. **Use `close` not `quit`**: Preserve IDE state between test runs
4. **Mock early**: Set up `mock_wx` before navigating to pages
5. **Screenshot + logs**: Combine visual + text evidence for bug reports
6. **Clean cache on errors**: Run `cache_clean` + `compile` if builds behave oddly
7. **Version everything**: Use semantic versioning in `upload` action
8. **Test preview first**: Never `upload` without validating via `preview` QR
9. **Read source before mocking**: Use `read_page` to understand data flow
10. **Automate health checks**: Run SOP D daily to catch regressions early

## Integration with Other MCPs

- **CloudBase MCP**: Use for cloud functions/database (replaces deprecated `wechat_cloud`)
- **Chrome DevTools MCP**: Cross-reference CDP logs for H5 pages
- **File system MCP**: Batch edit mini program source files

## Example: Complete Bug Fix Workflow

```python
# 1. Verify setup
status = wechat_ide(action='status')
assert status['connected']

# 2. Navigate to buggy page
wechat_navigate(extra_args={
    'page': 'pages/order/order',
    'query': 'id=789',
    'log_duration': 5
})

# 3. Capture initial state
screenshot_before = wechat_screenshot(extra_args={'full_page': True})
logs_before = wechat_inspector(action='console', extra_args={'duration': 3})

# 4. Read source to identify issue
source = wechat_file(action='read_page', extra_args={'page_path': 'pages/order/order'})
# (Agent analyzes source, suggests fix)

# 5. Apply fix (via separate file edit MCP)
# ...

# 6. Verify fix
wechat_build(action='compile')
wechat_navigate(extra_args={'page': 'pages/order/order', 'query': 'id=789'})
screenshot_after = wechat_screenshot(extra_args={'full_page': True})
logs_after = wechat_inspector(action='console', extra_args={'duration': 3})

# 7. Compare before/after
assert 'error' not in logs_after.lower()

# 8. Deploy
wechat_build(action='preview', extra_args={'qr_format': 'terminal'})
# User validates, then:
wechat_build(action='upload', extra_args={'version': '1.0.1', 'desc': 'Fix order page crash'})
```

## Version Compatibility

- **MCP Server**: v0.9.7+
- **WeChat DevTools**: 1.06.2302270+ (stable channel)
- **Python**: 3.8+ (automatically managed by `uv`)
- **Node.js**: 14+ (for `npx skills` command)

Check current MCP version: `wechat_ide(action='status')['mcp_version']`

## Additional Resources

- [Full API Reference](https://github.com/WaterTian/wechat-devtools-mcp/blob/main/MCP_DOC.md)
- [Skill Source](https://github.com/WaterTian/wechat-devtools-mcp/tree/main/.agents/skills/wechat-devtools)
- [WeChat CLI Official Docs](https://developers.weixin.qq.com/miniprogram/dev/devtools/cli.html)
- [MCP Registry Entry](https://modelcontextprotocol.io/)
