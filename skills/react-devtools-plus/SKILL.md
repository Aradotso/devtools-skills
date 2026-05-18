---
name: react-devtools-plus
description: Drop-in Vite/Webpack plugin that mirrors React Fiber trees, profiles renders, and provides visual debugging overlay for React 16-19 apps.
triggers:
  - add react devtools plus to my project
  - set up react debugging overlay
  - configure react devtools plus plugin
  - profile react component renders
  - inspect react fiber tree
  - debug react components in development
  - add react devtools to vite
  - integrate react debugging tools
---

# React DevTools Plus

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

React DevTools Plus is an open-source build tool plugin that provides visual debugging for React applications. It mirrors your React Fiber tree in real-time, profiles component renders, and offers a keyboard-first overlay—all with zero impact on production builds.

## What It Does

- **Fiber Tree Mirroring**: Automatically instruments React Fiber roots to visualize component hierarchies
- **Timeline Profiling**: Tracks component renders and performance metrics
- **Floating Overlay**: Toggle with `Alt/Option + Shift + D` for inline debugging
- **Asset Inspection**: Browse and inspect project assets (images, fonts, etc.)
- **Editor Integration**: Click-to-open components in your IDE
- **React 16-19 Support**: Works with React 16.8+, 17, 18, and 19
- **Dev-Only**: Completely removed from production bundles

## Installation

### For Vite Projects

```bash
pnpm add -D react-devtools-plus
```

```typescript
// vite.config.ts
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    react(),
    reactDevToolsPlus(),
  ],
})
```

### For Webpack Projects

```bash
npm install -D react-devtools-plus
```

```javascript
// webpack.config.js
const { reactDevToolsPlus } = require('react-devtools-plus/webpack')

module.exports = {
  plugins: [
    reactDevToolsPlus(),
  ],
}
```

## Configuration

### Basic Configuration

```typescript
// vite.config.ts
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    react(),
    reactDevToolsPlus({
      // Enable/disable overlay (default: true in dev mode)
      overlay: true,
      
      // Custom DevTools route (default: '/__react_devtools__')
      base: '/__react_devtools__',
      
      // Editor integration (auto-detected by default)
      launchEditor: 'code', // 'code' | 'webstorm' | 'cursor' | 'vim' | etc.
    }),
  ],
})
```

### Advanced Configuration

```typescript
// vite.config.ts
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    react(),
    reactDevToolsPlus({
      // Disable overlay, use only the dedicated route
      overlay: false,
      
      // Custom base path
      base: '/__debug__',
      
      // Specific editor with custom launch command
      launchEditor: 'webstorm',
    }),
  ],
})
```

### Webpack Configuration

```javascript
// webpack.config.js
const { reactDevToolsPlus } = require('react-devtools-plus/webpack')

module.exports = {
  mode: 'development',
  plugins: [
    reactDevToolsPlus({
      overlay: true,
      base: '/__react_devtools__',
      launchEditor: 'code',
    }),
  ],
}
```

## Usage Patterns

### Accessing DevTools

**Method 1: Keyboard Shortcut**
Press `Alt + Shift + D` (Windows/Linux) or `Option + Shift + D` (macOS) to toggle the floating overlay.

**Method 2: Direct URL**
Navigate to `http://localhost:5173/__react_devtools__/` (adjust port/base as configured).

**Method 3: Toggle Overlay Visibility**
Press `Alt + Shift + R` (Windows/Linux) or `Option + Shift + R` (macOS) to toggle overlay visibility.

Press `Escape` to close the overlay.

### Component Tree Inspection

Once DevTools is open:

1. **View Component Hierarchy**: The left panel shows your React component tree in real-time
2. **Inspect Props/State**: Click any component to view its props, state, and hooks
3. **Open in Editor**: Click the "Open in Editor" button to jump to source code
4. **Search Components**: Use the search bar to filter components by name

### Timeline Profiling

1. Switch to the **Timeline** tab
2. Click "Start Recording" to begin profiling
3. Interact with your app (trigger renders)
4. Click "Stop Recording" to analyze performance
5. Review render durations, commit phases, and component updates

### Asset Inspection

1. Switch to the **Assets** tab
2. Browse images, fonts, and other project assets
3. Click assets to view details and usage
4. Copy asset paths for use in your code

## Real-World Examples

### Example 1: Next.js App with Vite

```typescript
// next.config.mjs (with Vite plugin via next-with-vite)
import { defineConfig } from 'vite'
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    reactDevToolsPlus({
      overlay: true,
      base: '/__devtools__',
    }),
  ],
})
```

### Example 2: React SPA with Custom Editor

```typescript
// vite.config.ts
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    react(),
    reactDevToolsPlus({
      overlay: true,
      launchEditor: 'cursor', // Open components in Cursor AI
    }),
  ],
})
```

### Example 3: Monorepo with Multiple React Apps

```typescript
// apps/admin/vite.config.ts
import { defineConfig } from 'vite'
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    reactDevToolsPlus({
      base: '/__admin_devtools__', // Unique base per app
    }),
  ],
})

// apps/customer/vite.config.ts
import { defineConfig } from 'vite'
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig({
  plugins: [
    reactDevToolsPlus({
      base: '/__customer_devtools__', // Unique base per app
    }),
  ],
})
```

### Example 4: Conditional DevTools (Environment-based)

```typescript
// vite.config.ts
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'
import { reactDevToolsPlus } from 'react-devtools-plus/vite'

export default defineConfig(({ mode }) => ({
  plugins: [
    react(),
    // Only enable in development and staging
    (mode === 'development' || mode === 'staging') && reactDevToolsPlus({
      overlay: mode === 'development', // Overlay only in dev
      base: '/__devtools__',
    }),
  ].filter(Boolean),
}))
```

### Example 5: Webpack with React and TypeScript

```javascript
// webpack.config.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { reactDevToolsPlus } = require('react-devtools-plus/webpack')

module.exports = {
  mode: 'development',
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
    reactDevToolsPlus({
      overlay: true,
      launchEditor: 'code',
    }),
  ],
  devServer: {
    port: 3000,
    hot: true,
  },
}
```

## Key Features & API

### Plugin Options

```typescript
interface ReactDevToolsPlusOptions {
  /**
   * Enable/disable the floating overlay
   * @default true in development mode
   */
  overlay?: boolean

  /**
   * Custom base path for DevTools UI
   * @default '/__react_devtools__'
   */
  base?: string

  /**
   * Editor to launch when clicking "Open in Editor"
   * Auto-detected by default
   * @default auto-detected
   */
  launchEditor?: 'code' | 'webstorm' | 'cursor' | 'vim' | 'atom' | 'sublime' | string
}
```

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Alt/Option + Shift + D` | Toggle DevTools overlay |
| `Alt/Option + Shift + R` | Toggle overlay visibility |
| `Escape` | Close overlay |

### DevTools UI Tabs

1. **Component Tree**: Real-time React Fiber tree visualization
2. **Timeline**: Render profiling and performance metrics
3. **Assets**: Project asset browser and inspector
4. **Settings**: DevTools configuration and preferences

## Troubleshooting

### DevTools Not Appearing

**Issue**: Pressing the keyboard shortcut or navigating to the URL shows nothing.

**Solutions**:
- Ensure you're running in development mode (`NODE_ENV=development`)
- Check that the plugin is correctly added to your Vite/Webpack config
- Verify your dev server is running and the port is correct
- Check browser console for errors

### Overlay Blocks UI Elements

**Issue**: The floating overlay is interfering with app interaction.

**Solutions**:
```typescript
// Disable overlay, use dedicated route only
reactDevToolsPlus({
  overlay: false,
})
```

Or toggle visibility with `Alt/Option + Shift + R`.

### "Open in Editor" Not Working

**Issue**: Clicking components doesn't open them in your editor.

**Solutions**:
```typescript
// Explicitly specify your editor
reactDevToolsPlus({
  launchEditor: 'code', // or 'cursor', 'webstorm', etc.
})
```

Ensure your editor's command-line tool is installed:
- **VS Code**: Install `code` command via Command Palette → "Shell Command: Install 'code' command in PATH"
- **Cursor**: Similar to VS Code
- **WebStorm**: Ensure `webstorm` command is in PATH

### Components Not Showing in Tree

**Issue**: Some components are missing from the DevTools tree.

**Solutions**:
- Ensure you're using React 16.8+ (Hooks required)
- Check that components are mounted and rendered
- Verify React DevTools is instrumenting Fiber roots (check console for initialization logs)
- Try refreshing the DevTools panel

### DevTools Affecting Production Build

**Issue**: DevTools code appears in production bundle.

**Solutions**:
The plugin should automatically exclude itself from production builds. Verify:
```typescript
// vite.config.ts
export default defineConfig(({ mode }) => ({
  plugins: [
    react(),
    // Plugin is dev-only by default
    reactDevToolsPlus(),
  ],
}))
```

If issues persist:
```typescript
// Explicitly guard
export default defineConfig(({ mode }) => ({
  plugins: [
    react(),
    mode === 'development' && reactDevToolsPlus(),
  ].filter(Boolean),
}))
```

### Custom Base Path Not Working

**Issue**: DevTools not accessible at custom path.

**Solutions**:
```typescript
// Ensure base includes leading and trailing slashes
reactDevToolsPlus({
  base: '/__custom_path__', // Correct
  // NOT: base: 'custom_path' or base: '/custom_path'
})
```

Then navigate to: `http://localhost:5173/__custom_path__/`

## Project Structure

React DevTools Plus is a monorepo with the following key packages:

- `react-devtools` - Main Vite/Webpack plugin
- `react-devtools-client` - DevTools UI client
- `react-devtools-core` - Core functionality and plugin system
- `react-devtools-kit` - State management and messaging
- `react-devtools-overlay` - Floating overlay component
- `react-devtools-scan` - Render scanning utilities

## Development

If you need to contribute or debug the plugin itself:

```bash
# Clone the repository
git clone https://github.com/wzc520pyfm/react-devtools-plus.git
cd react-devtools-plus

# Install dependencies
pnpm install

# Start development mode
pnpm dev

# Run Vite playground
pnpm play

# Run Webpack playground
pnpm play:webpack

# Build all packages
pnpm build

# Run tests
pnpm test

# Lint code
pnpm lint
```

## Resources

- **Documentation**: https://react-devtools-plus.netlify.app/
- **GitHub**: https://github.com/wzc520pyfm/react-devtools-plus
- **Issues**: https://github.com/wzc520pyfm/react-devtools-plus/issues
- **License**: MIT
