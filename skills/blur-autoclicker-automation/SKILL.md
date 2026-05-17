---
name: blur-autoclicker-automation
description: An accuracy-focused Windows auto-clicker with advanced features including CPS control, keyboard automation, position clicking, and performance optimization built with TypeScript and Tauri.
triggers:
  - how do I build blur autoclicker from source
  - set up blur autoclicker development environment
  - how to configure blur autoclicker features
  - create an autoclicker script with blur
  - blur autoclicker keyboard automation
  - configure blur autoclicker position clicking
  - blur autoclicker performance optimization
  - how to package blur autoclicker for windows
---

# Blur AutoClicker Automation

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Blur AutoClicker is a high-performance Windows auto-clicker built with TypeScript and Tauri that provides precise CPS (clicks per second) control up to 500 CPS, keyboard automation, position clicking, and advanced timing features. It combines a web-based UI with Rust backend for optimal performance while maintaining under 100MB RAM usage.

## Installation

### End User Installation

Download the latest release from GitHub:

```powershell
# Download from https://github.com/Blur009/Blur-AutoClicker/releases/latest
# Default installation location
%localappdata%\BlurAutoClicker\BlurAutoClicker.exe

# Config and stats location
%appdata%\BlurAutoClicker
```

### Development Setup

Requirements:
- Node.js 20 or newer
- Rust via rustup
- Microsoft C++ Build Tools / Visual Studio Build Tools

```powershell
# Clone the repository
git clone https://github.com/Blur009/Blur-AutoClicker.git
cd Blur-AutoClicker

# Install dependencies
npm install

# Set up Rust toolchain
rustup default stable-x86_64-pc-windows-msvc
```

## Development Commands

```powershell
# Run in development mode
npm run dev

# Build release bundle
npm run build

# Lint code
npm run lint

# Build frontend only
npm run frontend:build

# Run Rust tests
cargo test --manifest-path src-tauri/Cargo.toml

# Check Rust code
cargo check --manifest-path src-tauri/Cargo.toml
```

Built installer location: `src-tauri/target/release/bundle/nsis/`

## Project Structure

```
Blur-AutoClicker/
├── src/                    # TypeScript frontend (web UI)
├── src-tauri/             # Rust backend (Tauri)
│   ├── src/               # Rust source code
│   ├── Cargo.toml         # Rust dependencies
│   └── tauri.conf.json    # Tauri configuration
├── public/                # Static assets
└── package.json           # Node.js dependencies
```

## Core Features

### 1. Mouse Button Control

The application supports left, right, and middle mouse button clicking:

```typescript
// Example configuration for mouse button selection
interface MouseConfig {
  button: 'left' | 'right' | 'middle';
  mode: 'hold' | 'toggle';
  cps: number; // Clicks per second (max 500)
}

const config: MouseConfig = {
  button: 'left',
  mode: 'toggle',
  cps: 50
};
```

### 2. Keyboard Key Automation

Automate keyboard key presses with case control:

```typescript
interface KeyConfig {
  key: string;
  uppercase: boolean;
  mode: 'hold' | 'toggle';
  pressesPerSecond: number;
}

const keyConfig: KeyConfig = {
  key: 'a',
  uppercase: false,
  mode: 'hold',
  pressesPerSecond: 10
};
```

### 3. Click Timing (Duty Cycle)

Control the timing between press and release:

```typescript
interface TimingConfig {
  clickDuration: number; // milliseconds
  interval: number; // milliseconds between clicks
}

// For 50 CPS with 50% duty cycle
const timing: TimingConfig = {
  clickDuration: 10, // 10ms press
  interval: 20 // 20ms total (1000ms / 50 CPS)
};
```

### 4. Speed Range Mode

Randomize CPS within a range:

```typescript
interface SpeedRangeConfig {
  enabled: boolean;
  minCps: number;
  maxCps: number;
}

const speedRange: SpeedRangeConfig = {
  enabled: true,
  minCps: 40,
  maxCps: 60
};
```

### 5. Position Clicking

Click at specific screen coordinates:

```typescript
interface PositionConfig {
  enabled: boolean;
  x: number;
  y: number;
  moveBeforeClick: boolean;
}

const positionClick: PositionConfig = {
  enabled: true,
  x: 1920,
  y: 1080,
  moveBeforeClick: true
};
```

### 6. Click and Time Limits

Stop after specific conditions:

```typescript
interface LimitsConfig {
  clickLimit?: number; // Stop after X clicks
  timeLimit?: number; // Stop after X seconds
}

const limits: LimitsConfig = {
  clickLimit: 1000,
  timeLimit: 60 // 60 seconds
};
```

### 7. Corner and Edge Stopping

Automatically stop when mouse reaches screen edges:

```typescript
interface EdgeStoppingConfig {
  enabled: boolean;
  cornerThreshold: number; // pixels from corner
  edgeThreshold: number; // pixels from edge
}

const edgeStopping: EdgeStoppingConfig = {
  enabled: true,
  cornerThreshold: 50,
  edgeThreshold: 10
};
```

## Tauri Backend Integration

### Tauri Commands

When developing features, interact with the Rust backend via Tauri commands:

```typescript
import { invoke } from '@tauri-apps/api/tauri';

// Start clicking
async function startClicking(config: ClickConfig): Promise<void> {
  await invoke('start_clicking', { config });
}

// Stop clicking
async function stopClicking(): Promise<void> {
  await invoke('stop_clicking');
}

// Get click stats
async function getStats(): Promise<ClickStats> {
  return await invoke('get_stats');
}

interface ClickStats {
  totalClicks: number;
  currentCps: number;
  averageCps: number;
  uptime: number;
}
```

### Rust Backend Example

```rust
// src-tauri/src/main.rs
use tauri::State;
use std::sync::Mutex;

struct ClickerState {
    is_active: bool,
    total_clicks: u64,
}

#[tauri::command]
fn start_clicking(state: State<Mutex<ClickerState>>) -> Result<(), String> {
    let mut clicker = state.lock().unwrap();
    clicker.is_active = true;
    Ok(())
}

#[tauri::command]
fn stop_clicking(state: State<Mutex<ClickerState>>) -> Result<(), String> {
    let mut clicker = state.lock().unwrap();
    clicker.is_active = false;
    Ok(())
}

#[tauri::command]
fn get_stats(state: State<Mutex<ClickerState>>) -> Result<u64, String> {
    let clicker = state.lock().unwrap();
    Ok(clicker.total_clicks)
}
```

## Configuration Management

### Saving Configuration

```typescript
import { appDataDir } from '@tauri-apps/api/path';
import { writeTextFile, readTextFile } from '@tauri-apps/api/fs';

async function saveConfig(config: AppConfig): Promise<void> {
  const appData = await appDataDir();
  const configPath = `${appData}/config.json`;
  await writeTextFile(configPath, JSON.stringify(config, null, 2));
}

async function loadConfig(): Promise<AppConfig | null> {
  try {
    const appData = await appDataDir();
    const configPath = `${appData}/config.json`;
    const content = await readTextFile(configPath);
    return JSON.parse(content);
  } catch (error) {
    console.error('Failed to load config:', error);
    return null;
  }
}

interface AppConfig {
  mouseButton: 'left' | 'right' | 'middle';
  mode: 'hold' | 'toggle';
  cps: number;
  hotkey: string;
  advanced: AdvancedConfig;
}

interface AdvancedConfig {
  dutyCycle: number;
  speedRange?: SpeedRangeConfig;
  positionClick?: PositionConfig;
  limits?: LimitsConfig;
  edgeStopping?: EdgeStoppingConfig;
}
```

## Hotkey Management

```typescript
import { register, unregister } from '@tauri-apps/api/globalShortcut';

async function setupHotkeys(hotkey: string): Promise<void> {
  // Unregister previous hotkey
  await unregister(hotkey).catch(() => {});
  
  // Register new hotkey
  await register(hotkey, async () => {
    // Toggle clicking state
    const isActive = await invoke('is_clicking_active');
    if (isActive) {
      await invoke('stop_clicking');
    } else {
      await invoke('start_clicking');
    }
  });
}

// Common hotkey combinations
const hotkeyExamples = [
  'F6',
  'F7',
  'F8',
  'Ctrl+Shift+A',
  'Alt+C'
];
```

## Performance Optimization

### Timer Resolution

```rust
// src-tauri/src/timer.rs
use std::time::{Duration, Instant};
use std::thread;

pub struct PrecisionTimer {
    interval: Duration,
    last_tick: Instant,
}

impl PrecisionTimer {
    pub fn new(cps: u32) -> Self {
        let interval = Duration::from_micros(1_000_000 / cps as u64);
        Self {
            interval,
            last_tick: Instant::now(),
        }
    }
    
    pub fn wait_for_next_tick(&mut self) {
        let elapsed = self.last_tick.elapsed();
        if elapsed < self.interval {
            thread::sleep(self.interval - elapsed);
        }
        self.last_tick = Instant::now();
    }
}

// Usage
let mut timer = PrecisionTimer::new(50); // 50 CPS
loop {
    // Perform click
    send_click();
    timer.wait_for_next_tick();
}
```

### Memory Management

Keep RAM usage under 100MB:

```typescript
// Clear click history periodically
let clickHistory: number[] = [];
const MAX_HISTORY_SIZE = 1000;

function recordClick(timestamp: number): void {
  clickHistory.push(timestamp);
  
  // Keep only recent clicks
  if (clickHistory.length > MAX_HISTORY_SIZE) {
    clickHistory = clickHistory.slice(-MAX_HISTORY_SIZE);
  }
}

function calculateCps(): number {
  const now = Date.now();
  const recentClicks = clickHistory.filter(t => now - t < 1000);
  return recentClicks.length;
}
```

## Building for Release

### Windows Release Process

```powershell
# 1. Update version in package.json and Cargo.toml
# 2. Clean previous builds
Remove-Item -Recurse -Force src-tauri/target/release/bundle -ErrorAction SilentlyContinue

# 3. Build release
npm run build

# 4. Installer location
# src-tauri/target/release/bundle/nsis/Blur-AutoClicker_X.X.X_x64-setup.exe

# 5. Test installer
Start-Process "src-tauri\target\release\bundle\nsis\*.exe"
```

### Code Signing (Optional)

```powershell
# Note: Windows Authenticode signing is separate from Tauri updater signing
# See docs/windows-release-trust.md for complete signing instructions

# Environment variables needed for signing
# WINDOWS_CERTIFICATE (base64 certificate)
# WINDOWS_CERTIFICATE_PASSWORD (certificate password)
```

## Common Development Patterns

### Creating a Custom Click Mode

```typescript
// src/lib/modes/customMode.ts
interface CustomModeConfig {
  pattern: number[]; // Array of delays in milliseconds
  repeat: boolean;
}

class CustomClickMode {
  private config: CustomModeConfig;
  private patternIndex: number = 0;
  
  constructor(config: CustomModeConfig) {
    this.config = config;
  }
  
  getNextDelay(): number {
    const delay = this.config.pattern[this.patternIndex];
    this.patternIndex = (this.patternIndex + 1) % this.config.pattern.length;
    return delay;
  }
  
  async executePattern(): Promise<void> {
    do {
      for (const delay of this.config.pattern) {
        await invoke('send_click');
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    } while (this.config.repeat);
  }
}

// Usage: Click with pattern [100ms, 200ms, 100ms, 500ms]
const customMode = new CustomClickMode({
  pattern: [100, 200, 100, 500],
  repeat: true
});
```

### Statistics Tracking

```typescript
// src/lib/stats.ts
interface ClickStatistics {
  totalClicks: number;
  sessionClicks: number;
  averageCps: number;
  peakCps: number;
  uptime: number; // seconds
  startTime: number;
}

class StatsTracker {
  private stats: ClickStatistics;
  private clickTimestamps: number[] = [];
  
  constructor() {
    this.stats = {
      totalClicks: 0,
      sessionClicks: 0,
      averageCps: 0,
      peakCps: 0,
      uptime: 0,
      startTime: Date.now()
    };
  }
  
  recordClick(): void {
    const now = Date.now();
    this.stats.totalClicks++;
    this.stats.sessionClicks++;
    this.clickTimestamps.push(now);
    
    // Keep only last second of clicks
    const oneSecondAgo = now - 1000;
    this.clickTimestamps = this.clickTimestamps.filter(t => t > oneSecondAgo);
    
    const currentCps = this.clickTimestamps.length;
    if (currentCps > this.stats.peakCps) {
      this.stats.peakCps = currentCps;
    }
  }
  
  getStats(): ClickStatistics {
    this.stats.uptime = Math.floor((Date.now() - this.stats.startTime) / 1000);
    this.stats.averageCps = this.stats.uptime > 0 
      ? this.stats.sessionClicks / this.stats.uptime 
      : 0;
    return { ...this.stats };
  }
  
  async save(): Promise<void> {
    await saveConfig({ stats: this.stats });
  }
}
```

## Troubleshooting

### Windows SmartScreen Warning

The application shows SmartScreen warnings for unsigned installers:
- This is normal for open-source GitHub releases
- See `docs/windows-release-trust.md` for signing information
- Users can click "More info" → "Run anyway"

### CPS Accuracy Issues

If actual CPS doesn't match configured CPS:

```typescript
// Enable debug logging
const DEBUG = true;

async function debugClickAccuracy(targetCps: number, duration: number): Promise<void> {
  const clicks: number[] = [];
  const startTime = Date.now();
  
  while (Date.now() - startTime < duration) {
    const clickTime = Date.now();
    await invoke('send_click');
    clicks.push(clickTime);
    
    if (DEBUG) {
      // Calculate actual CPS every second
      const recentClicks = clicks.filter(t => clickTime - t < 1000);
      console.log(`Target: ${targetCps} CPS, Actual: ${recentClicks.length} CPS`);
    }
  }
  
  // Final accuracy report
  const totalTime = (Date.now() - startTime) / 1000;
  const actualCps = clicks.length / totalTime;
  const accuracy = (actualCps / targetCps) * 100;
  console.log(`Accuracy: ${accuracy.toFixed(2)}%`);
}
```

### High RAM Usage

If RAM usage exceeds 100MB:

```typescript
// Monitor memory usage
if (performance.memory) {
  const usedMemory = performance.memory.usedJSHeapSize / 1024 / 1024;
  console.log(`Memory usage: ${usedMemory.toFixed(2)} MB`);
  
  if (usedMemory > 80) {
    // Clear unnecessary data
    clickHistory = clickHistory.slice(-100);
    // Force garbage collection (if available)
    if (global.gc) global.gc();
  }
}
```

### Build Errors

```powershell
# Clean install
Remove-Item -Recurse -Force node_modules
Remove-Item package-lock.json
npm install

# Clean Rust build
cargo clean --manifest-path src-tauri/Cargo.toml

# Update Rust toolchain
rustup update stable-x86_64-pc-windows-msvc

# Rebuild
npm run build
```

### Tauri Command Not Found

```typescript
// Ensure proper Tauri API imports
import { invoke } from '@tauri-apps/api/tauri';

// Check Tauri configuration
// src-tauri/tauri.conf.json
{
  "tauri": {
    "allowlist": {
      "all": false,
      "fs": {
        "all": false,
        "readFile": true,
        "writeFile": true
      },
      "globalShortcut": {
        "all": true
      }
    }
  }
}
```

## Testing

```typescript
// Example test for click accuracy
describe('Click Accuracy', () => {
  it('should click at specified CPS within 5% tolerance', async () => {
    const targetCps = 50;
    const duration = 5000; // 5 seconds
    const tolerance = 0.05; // 5%
    
    const clicks = await testClickAccuracy(targetCps, duration);
    const actualCps = clicks.length / (duration / 1000);
    
    const lowerBound = targetCps * (1 - tolerance);
    const upperBound = targetCps * (1 + tolerance);
    
    expect(actualCps).toBeGreaterThanOrEqual(lowerBound);
    expect(actualCps).toBeLessThanOrEqual(upperBound);
  });
});
```

## License

GPL-3.0 - See [LICENSE.md](https://github.com/Blur009/Blur-AutoClicker/blob/main/LICENSE.md)
