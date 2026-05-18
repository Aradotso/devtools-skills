---
name: htmx-devtools-extension
description: Browser DevTools extension for debugging HTMX applications with request inspection, element tracking, event timeline, swap visualization, and error detection
triggers:
  - debug my htmx application
  - install htmx devtools extension
  - inspect htmx requests and responses
  - track htmx element attributes
  - visualize htmx dom swaps
  - monitor htmx events timeline
  - troubleshoot htmx errors
  - set up htmx debugging tools
---

# HTMX DevTools Extension

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

HTMX DevTools is a browser extension that provides a comprehensive debugging interface for HTMX applications. It captures the full request lifecycle, tracks DOM elements with htmx attributes, visualizes swaps, monitors events, and surfaces errors. Supports both **htmx 2.x** and **htmx 4.0 alpha** with automatic version detection.

### Key Features

- **Request Inspector**: Full lifecycle tracking with timing breakdown, headers, and event trace
- **Element Inspector**: Live DOM tree filtered to htmx elements with click-to-inspect
- **Event Timeline**: Filterable timeline of all htmx events with category color coding
- **Swap Visualizer**: Before/after snapshots with LCS-based diff view
- **Error Panel**: Automatic detection of HTTP errors, target not found, timeouts, and swap errors

## Installation

### Installing the Extension

**From Source (Development):**

```bash
git clone https://github.com/atoolz/htmx-devtools.git
cd htmx-devtools
npm install
npm run build:chrome
```

**Chrome/Edge/Brave/Arc:**

1. Navigate to `chrome://extensions`
2. Enable **Developer mode** (toggle in top right)
3. Click **Load unpacked**
4. Select the `dist/` folder from the cloned repository

**Firefox:**

```bash
npm run build:firefox
```

1. Navigate to `about:debugging#/runtime/this-firefox`
2. Click **Load Temporary Add-on**
3. Select `dist/manifest.json`

### Verifying Installation

1. Open a page with htmx loaded
2. Press `F12` or open DevTools
3. Look for the **HTMX** tab in the DevTools panel
4. The version badge (blue for 2.x, purple for 4.0) should appear automatically

## Using the Extension

### Request Inspector

Monitor all HTMX requests with detailed lifecycle information:

**Key Information Displayed:**
- HTTP method, URL, status code
- Trigger element and target element
- Visual timeline: Config → Send → Wait → Swap → Settle
- All `HX-*` request and response headers
- Request and response bodies
- Correlated event trace

**Controls:**
- **Record** (●): Capture new requests
- **Pause** (❚❚): Stop capturing while preserving current data
- **Clear** (🗑): Remove all captured requests

**Example HTMX Request:**

```html
<!-- This request will appear in the Request Inspector -->
<button hx-get="/api/users" 
        hx-target="#user-list"
        hx-swap="innerHTML">
  Load Users
</button>

<div id="user-list"></div>
```

When clicked, the inspector shows:
- Method: `GET`
- URL: `/api/users`
- Target: `#user-list`
- Swap: `innerHTML`
- Timing breakdown for each phase
- HTTP headers including `HX-Request: true`, `HX-Target: user-list`

### Element Inspector

Explore the live DOM tree filtered to htmx-relevant elements:

**Features:**
- Auto-refreshing tree view showing element hierarchy
- Click any element to see:
  - All `hx-*` attributes
  - Resolved target selectors
  - Internal htmx data
  - Request history for that element
- **Element Picker** (🎯): Click to select any element on the page
- Hover over tree nodes to highlight elements on the page

**Example Element Tree:**

```html
<body>
  <div hx-boost="true">
    <form hx-post="/api/contact" hx-target="#result">
      <input name="email" hx-validate="true">
      <button type="submit">Submit</button>
    </form>
    <div id="result" hx-swap-oob="true"></div>
  </div>
</body>
```

Inspector displays:
```
▼ body
  ▼ div [hx-boost="true"]
    ▼ form [hx-post="/api/contact", hx-target="#result"]
      • input [hx-validate="true"]
      • button
    • div#result [hx-swap-oob="true"]
```

### Event Timeline

Filter and explore all htmx events with expandable details:

**Event Categories:**
- **Init**: `htmx:load`, `htmx:configRequest`
- **Request**: `htmx:beforeRequest`, `htmx:afterRequest`
- **XHR**: `htmx:xhr:loadstart`, `htmx:xhr:progress`
- **Response**: `htmx:beforeSwap`, `htmx:afterSwap`
- **Swap**: `htmx:beforeSwap`, `htmx:swapError`
- **OOB**: `htmx:oobBeforeSwap`, `htmx:oobAfterSwap`
- **History**: `htmx:historyRestore`, `htmx:pushedIntoHistory`
- **Transition**: `htmx:beforeTransition`, `htmx:afterSettle`
- **Error**: `htmx:responseError`, `htmx:sendError`, `htmx:targetError`

**Example Event Flow:**

```typescript
// For this htmx request:
// <button hx-get="/data" hx-target="#output">Get Data</button>

// Timeline shows:
// 1. htmx:configRequest (Init) - Request configured
// 2. htmx:beforeRequest (Request) - About to send
// 3. htmx:xhr:loadstart (XHR) - XHR started
// 4. htmx:beforeSwap (Response) - Response received, about to swap
// 5. htmx:afterSwap (Swap) - DOM swapped
// 6. htmx:afterSettle (Transition) - Settling complete
```

Click any event to expand `event.detail` JSON payload.

### Swap Visualizer

Record and analyze DOM changes with before/after comparison:

**Controls:**
- **Record** (●): Start capturing swaps
- **Pause** (❚❚): Stop capturing
- Click any swap entry to view details

**Views:**
- **Response HTML**: Raw HTML received from server
- **Before**: DOM state before swap
- **After**: DOM state after swap
- **Diff**: Line-by-line comparison with add/remove highlighting

**Example Swap:**

```html
<!-- Before -->
<div id="content">
  <p>Old content</p>
</div>

<!-- Server responds with (hx-swap="innerHTML") -->
<p>New content</p>
<p>Additional paragraph</p>

<!-- After -->
<div id="content">
  <p>New content</p>
  <p>Additional paragraph</p>
</div>
```

Diff view shows:
```diff
<div id="content">
-  <p>Old content</p>
+  <p>New content</p>
+  <p>Additional paragraph</p>
</div>
```

### Error Panel

Automatically surfaces HTMX failures grouped by type:

**Error Types Detected:**
- **HTTP Errors**: 4xx and 5xx responses (even in htmx 4.0 where they swap by default)
- **Target Not Found**: `hx-target` selector doesn't match any element
- **Network Timeouts**: Request exceeds timeout threshold
- **Swap Errors**: DOM swap operation fails

**Example Error Detection:**

```html
<!-- This will trigger a "Target Not Found" error -->
<button hx-get="/data" hx-target="#missing-element">
  Click Me
</button>

<!-- This will trigger an "HTTP Error" if server returns 404 -->
<button hx-get="/nonexistent" hx-target="#output">
  Load Missing
</button>
```

Errors appear with:
- Badge count per error type
- Click to jump to associated request in Request Inspector
- Full error details and context

## Architecture

### Extension Components

```
Page Script (MAIN world)
  ↓ postMessage
Content Script (isolated)
  ↓ chrome.runtime.sendMessage
Service Worker (background)
  ↓ chrome.runtime.sendMessage
DevTools Panel (Preact UI)
```

**Page Script** (`src/page-script/index.ts`):
- Runs in page's JavaScript context via `"world": "MAIN"`
- Listens to all `htmx:*` events (both 2.x and 4.0 naming)
- Serializes element data and tracks requests
- Batches messages every 50ms

**Content Script** (`src/content-script/index.ts`):
- Bridges page and extension contexts
- Relays messages via `window.postMessage` and `chrome.runtime.sendMessage`

**Background Service Worker** (`src/background/index.ts`):
- Manages per-tab state
- Tracks request lifecycles
- Maps 2.x and 4.0 event names to canonical format
- Synthesizes HTTP errors for 4xx/5xx
- Routes data to DevTools panel

**DevTools Panel** (`src/panel/`):
- Preact + Signals UI (~55KB)
- Five tabs: Requests, Elements, Timeline, Swaps, Errors
- Real-time updates via message listeners

### Version Detection

The extension auto-detects htmx version and adapts:

| Feature | htmx 2.x | htmx 4.0 |
|---------|----------|----------|
| Event names | `htmx:configRequest` | `htmx:config:request` |
| Request tracking | XHR WeakMap | ctx object WeakMap |
| Detail structure | `detail.elt`, `detail.xhr` | `detail.ctx.sourceElement`, `detail.ctx.response` |
| Error events | 10 separate events | Unified `htmx:error` + synthetic |

## Development

### Build Commands

```bash
# Development mode with watch
npm run dev

# Production build
npm run build

# Chrome-specific build (copies manifest + icons)
npm run build:chrome

# Firefox-specific build
npm run build:firefox

# TypeScript type checking
npm run typecheck
```

### Project Structure

```
htmx-devtools/
├── src/
│   ├── page-script/       # Injected into page context
│   ├── content-script/    # Bridge between page and extension
│   ├── background/        # Service worker for state management
│   ├── panel/             # Preact UI components
│   │   ├── components/    # UI components for each tab
│   │   ├── store/         # Signal-based state management
│   │   └── utils/         # Helper functions
│   └── shared/            # Shared types and utilities
├── public/                # Static assets (icons, manifests)
├── test/e2e/fixtures/     # Test server and demo pages
└── dist/                  # Build output
```

### Local Test Server

Start a local test server with comprehensive htmx examples:

```bash
node test/e2e/fixtures/test-server.js
```

Open http://localhost:3456 in your browser.

**Test Scenarios Covered:**
- GET/POST/PUT/DELETE requests
- Error scenarios (404, 500, timeout)
- All swap strategies (innerHTML, outerHTML, beforebegin, etc.)
- Out-of-band swaps
- Polling with `hx-trigger="every 2s"`
- Search with debounce
- Click-to-edit pattern
- Todo list CRUD

### Adding Custom Test Cases

Create HTML files in `test/e2e/fixtures/` directory:

```html
<!-- test/e2e/fixtures/my-test.html -->
<!DOCTYPE html>
<html>
<head>
  <script src="https://unpkg.com/htmx.org@2.0.0"></script>
</head>
<body>
  <button hx-get="/api/test" hx-target="#result">
    Test Button
  </button>
  <div id="result"></div>
</body>
</html>
```

Access at http://localhost:3456/my-test.html

## Common Patterns

### Debugging Request Issues

**Problem**: Request not showing up in inspector

```typescript
// Check if htmx is properly loaded
console.log(window.htmx); // Should not be undefined

// Verify extension is capturing events
// Open HTMX DevTools tab, click Record button
// Look for "Recording" indicator

// Check if element has valid htmx attributes
const el = document.querySelector('[hx-get]');
console.log(el.getAttribute('hx-get')); // Should show URL
```

**Problem**: Target element not updating

```html
<!-- BAD: Target selector is invalid -->
<button hx-get="/data" hx-target="missing">Load</button>

<!-- GOOD: Target exists in DOM -->
<button hx-get="/data" hx-target="#content">Load</button>
<div id="content"></div>
```

Check **Error Panel** for "Target Not Found" errors.

### Tracking Element State

Use Element Inspector to verify attribute resolution:

```html
<div id="parent" hx-boost="true">
  <a href="/page1">Link 1</a>
  <a href="/page2">Link 2</a>
</div>
```

Element Inspector shows:
- Both `<a>` elements inherit `hx-boost` behavior
- Click either link to see request in Request Inspector
- Request history shows all requests triggered by that element

### Monitoring Swap Behavior

Enable Swap Visualizer before triggering swaps:

```html
<!-- Test different swap strategies -->
<button hx-get="/content" hx-target="#out" hx-swap="innerHTML">innerHTML</button>
<button hx-get="/content" hx-target="#out" hx-swap="outerHTML">outerHTML</button>
<button hx-get="/content" hx-target="#out" hx-swap="beforebegin">beforebegin</button>

<div id="out">Original content</div>
```

Swap Visualizer shows:
- Response HTML received from server
- Before/After DOM snapshots
- Diff highlighting exact changes
- Swap strategy and target info

### Filtering Event Timeline

Use category filters to focus on specific event types:

```typescript
// To debug swap issues, enable only Swap category
// Timeline shows: htmx:beforeSwap, htmx:afterSwap, htmx:swapError

// To debug request failures, enable Request + Error categories
// Timeline shows: htmx:beforeRequest, htmx:responseError, etc.

// Search bar filters within visible categories
// Example: Search "error" to find all error-related events
```

## Troubleshooting

### Extension Not Appearing in DevTools

**Issue**: No HTMX tab in DevTools

**Solution**:
1. Verify extension is installed: Check `chrome://extensions`
2. Ensure extension is enabled
3. Reload the page after installing extension
4. Check browser console for extension errors
5. Try refreshing DevTools (close and reopen)

### No Requests Being Captured

**Issue**: Request Inspector shows "No requests captured"

**Solution**:
```typescript
// 1. Verify htmx is loaded on the page
console.log(window.htmx); // Should output htmx object

// 2. Check if Recording is enabled
// Click the red record button in Request Inspector

// 3. Verify htmx elements are properly configured
document.querySelectorAll('[hx-get], [hx-post], [hx-put], [hx-delete]');
// Should return NodeList with your htmx elements

// 4. Check for JavaScript errors blocking htmx
// Open browser console, look for errors

// 5. Test with simple example
const testBtn = document.createElement('button');
testBtn.setAttribute('hx-get', '/test');
testBtn.textContent = 'Test';
document.body.appendChild(testBtn);
htmx.process(testBtn);
testBtn.click();
```

### Element Inspector Not Showing Elements

**Issue**: Element tree is empty

**Solution**:
1. Verify page has elements with `hx-*` attributes
2. Click the refresh icon in Element Inspector
3. Check if elements are dynamically loaded (wait for `htmx:load` event)
4. Use Element Picker (target icon) to directly select elements

### Events Not Appearing in Timeline

**Issue**: Timeline shows "No events captured"

**Solution**:
```typescript
// 1. Ensure event listeners are not blocked
// Check for errors in background service worker

// 2. Verify htmx events are firing
document.addEventListener('htmx:afterRequest', (e) => {
  console.log('Request completed:', e.detail);
});

// 3. Check category filters in Timeline tab
// Ensure at least one category is enabled

// 4. Clear and re-record
// Click Clear button, then trigger a new htmx request
```

### Version Detection Issues

**Issue**: Wrong version badge or features not working

**Solution**:
```typescript
// Check actual htmx version
console.log(htmx.version);

// htmx 2.x: Returns string like "2.0.0"
// htmx 4.0: Returns string like "4.0.0-alpha1"

// Reload extension and page if version changed
// Extension detects version on page load
```

### Performance Issues with Large Applications

**Issue**: DevTools panel becomes slow

**Solution**:
1. Use Record/Pause controls to limit data capture
2. Clear old requests periodically
3. Filter Event Timeline to specific categories
4. Disable Swap Visualizer when not actively debugging swaps
5. Use search to narrow down large request lists

### Extension Updates

**Issue**: New version available but not updating

**Solution (Development Mode)**:
```bash
# Pull latest changes
git pull origin main

# Rebuild extension
npm install
npm run build:chrome  # or build:firefox

# Reload extension
# 1. Go to chrome://extensions
# 2. Click reload icon on htmx-devtools extension
# 3. Refresh pages with htmx
```

## Browser Compatibility

- **Chrome**: 88+ (Manifest V3 support)
- **Edge**: 88+
- **Brave**: Latest
- **Arc**: Latest
- **Opera**: 74+
- **Firefox**: 128+ (Manifest V3 support in Firefox 128+)

## Resources

- **Live Demo**: https://atoolz.github.io/htmx-devtools/
- **htmx 2.x Demo**: https://atoolz.github.io/htmx-devtools/v2/
- **htmx 4.0 Demo**: https://atoolz.github.io/htmx-devtools/v4/
- **GitHub Repository**: https://github.com/atoolz/htmx-devtools
- **htmx Documentation**: https://htmx.org/docs/
- **htmx Events Reference**: https://htmx.org/events/
