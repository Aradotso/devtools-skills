---
name: justanothermusicclient-tauri-youtube-music
description: Build and extend JustAnotherMusicClient, a Tauri-based desktop YouTube Music client for Windows, macOS, and Linux
triggers:
  - how do I build JustAnotherMusicClient
  - set up development environment for YouTube Music desktop client
  - contribute to JustAnotherMusicClient project
  - create a Tauri music client like JustAnotherMusicClient
  - troubleshoot JustAnotherMusicClient build issues
  - add features to JustAnotherMusicClient
  - integrate Discord RPC in Tauri app
  - work with YouTube Music in Tauri
---

# JustAnotherMusicClient Development Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What is JustAnotherMusicClient?

JustAnotherMusicClient is a desktop YouTube Music client built with **Tauri**, **React**, and **TypeScript**. It provides a native desktop experience for YouTube Music with features like multiple tabs, Discord RPC, synced lyrics, caching, and cross-platform support (Windows, macOS, Linux).

**Key Technologies:**
- **Tauri** - Rust-based desktop framework
- **React** - Frontend UI
- **TypeScript** - Type-safe development
- **YouTube Music Integration** - No official API, uses web scraping/injection

## Prerequisites

Before working with JustAnotherMusicClient, ensure you have:

```bash
# Node.js LTS and npm
node --version  # v18+ recommended
npm --version

# Rust and Cargo
rustc --version
cargo --version

# Windows only: C++ build tools (Visual Studio Build Tools)
# macOS: Xcode Command Line Tools
# Linux: build-essential, libgtk-3-dev, libwebkit2gtk-4.0-dev, etc.
```

### Platform-Specific Requirements

**Windows:**
- Microsoft Edge WebView2 Runtime
- Visual Studio Build Tools with C++ workload

**macOS:**
- Xcode Command Line Tools: `xcode-select --install`

**Linux (Debian/Ubuntu):**
```bash
sudo apt update
sudo apt install libwebkit2gtk-4.0-dev \
    build-essential \
    curl \
    wget \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
```

## Installation & Setup

### Clone and Install Dependencies

```bash
# Clone the repository
git clone https://github.com/2latemc/JustAnotherMusicClient.git
cd JustAnotherMusicClient

# Install npm dependencies (includes Tauri CLI)
npm install
```

### Project Structure

```
JustAnotherMusicClient/
├── src/                    # React frontend source
│   ├── components/         # UI components
│   ├── hooks/             # Custom React hooks
│   ├── services/          # API/service layer
│   └── App.tsx            # Main app component
├── src-tauri/             # Rust backend
│   ├── src/
│   │   └── main.rs        # Tauri entry point
│   ├── Cargo.toml         # Rust dependencies
│   └── tauri.conf.json    # Tauri configuration
├── package.json           # Node dependencies
└── tsconfig.json          # TypeScript config
```

## Development Commands

### Run Development Server

```bash
# Start Tauri dev server with hot reload
npm run tauri dev
```

This will:
1. Start the React dev server (Vite)
2. Compile and run the Rust backend
3. Open the application window
4. Enable hot module replacement for frontend changes

### Build for Production

```bash
# Create production build
npm run tauri build
```

Outputs are in `src-tauri/target/release/bundle/`:
- **Windows:** `.msi` installer
- **macOS:** `.dmg` and `.app`
- **Linux:** `.deb`, `.AppImage`

### Lint and Type Check

```bash
# TypeScript type checking
npm run type-check

# ESLint
npm run lint

# Format with Prettier
npm run format
```

## Key Configuration Files

### tauri.conf.json

Located at `src-tauri/tauri.conf.json`, this controls app behavior:

```json
{
  "build": {
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build",
    "devPath": "http://localhost:1420",
    "distDir": "../dist"
  },
  "package": {
    "productName": "JustAnotherMusicClient",
    "version": "1.0.0"
  },
  "tauri": {
    "allowlist": {
      "all": false,
      "shell": {
        "all": false,
        "open": true
      },
      "window": {
        "all": true,
        "create": true,
        "center": true,
        "setDecorations": true
      }
    },
    "bundle": {
      "active": true,
      "identifier": "com.2late.justanothermusicclient",
      "icon": [
        "icons/32x32.png",
        "icons/128x128.png",
        "icons/icon.icns",
        "icons/icon.ico"
      ]
    },
    "security": {
      "csp": "default-src 'self'; connect-src https://music.youtube.com"
    },
    "windows": [
      {
        "title": "JustAnotherMusicClient",
        "width": 1200,
        "height": 800,
        "resizable": true,
        "fullscreen": false
      }
    ]
  }
}
```

### Cargo.toml (Rust Dependencies)

Located at `src-tauri/Cargo.toml`:

```toml
[package]
name = "justanothermusicclient"
version = "1.0.0"
edition = "2021"

[dependencies]
tauri = { version = "1.5", features = ["shell-open", "window-all"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }

[build-dependencies]
tauri-build = { version = "1.5" }
```

## Common Development Patterns

### Adding a Tauri Command (Rust Backend)

In `src-tauri/src/main.rs`:

```rust
use tauri::State;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct Song {
    title: String,
    artist: String,
    duration: u32,
}

// Define a Tauri command
#[tauri::command]
fn get_current_song() -> Result<Song, String> {
    // Your logic here
    Ok(Song {
        title: "Song Title".to_string(),
        artist: "Artist Name".to_string(),
        duration: 180,
    })
}

// State management example
struct AppState {
    volume: std::sync::Mutex<f32>,
}

#[tauri::command]
fn set_volume(state: State<AppState>, volume: f32) -> Result<(), String> {
    let mut vol = state.volume.lock().unwrap();
    *vol = volume.clamp(0.0, 1.0);
    Ok(())
}

fn main() {
    tauri::Builder::default()
        .manage(AppState {
            volume: std::sync::Mutex::new(1.0),
        })
        .invoke_handler(tauri::generate_handler![
            get_current_song,
            set_volume
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Calling Tauri Commands from React/TypeScript

In your React components:

```typescript
import { invoke } from '@tauri-apps/api/tauri';

interface Song {
  title: string;
  artist: string;
  duration: number;
}

// Call Rust command from frontend
async function fetchCurrentSong(): Promise<Song> {
  try {
    const song = await invoke<Song>('get_current_song');
    return song;
  } catch (error) {
    console.error('Failed to fetch song:', error);
    throw error;
  }
}

// Set volume example
async function updateVolume(volume: number): Promise<void> {
  await invoke('set_volume', { volume });
}

// Use in component
function NowPlaying() {
  const [song, setSong] = React.useState<Song | null>(null);

  React.useEffect(() => {
    fetchCurrentSong().then(setSong);
  }, []);

  return (
    <div>
      {song && (
        <>
          <h2>{song.title}</h2>
          <p>{song.artist}</p>
        </>
      )}
    </div>
  );
}
```

### Window Management

```typescript
import { appWindow } from '@tauri-apps/api/window';

// Minimize window
await appWindow.minimize();

// Maximize/restore toggle
await appWindow.toggleMaximize();

// Close window
await appWindow.close();

// Create new window
import { WebviewWindow } from '@tauri-apps/api/window';

const miniPlayer = new WebviewWindow('miniPlayer', {
  url: '/mini-player',
  title: 'Mini Player',
  width: 300,
  height: 100,
  decorations: false,
  alwaysOnTop: true,
});
```

### System Tray Integration

In `src-tauri/src/main.rs`:

```rust
use tauri::{CustomMenuItem, SystemTray, SystemTrayMenu, SystemTrayEvent};
use tauri::Manager;

fn main() {
    let play_pause = CustomMenuItem::new("play_pause".to_string(), "Play/Pause");
    let quit = CustomMenuItem::new("quit".to_string(), "Quit");
    
    let tray_menu = SystemTrayMenu::new()
        .add_item(play_pause)
        .add_item(quit);
    
    let system_tray = SystemTray::new()
        .with_menu(tray_menu);

    tauri::Builder::default()
        .system_tray(system_tray)
        .on_system_tray_event(|app, event| match event {
            SystemTrayEvent::MenuItemClick { id, .. } => {
                match id.as_str() {
                    "play_pause" => {
                        // Emit event to frontend
                        app.emit_all("play-pause", ()).unwrap();
                    }
                    "quit" => {
                        std::process::exit(0);
                    }
                    _ => {}
                }
            }
            _ => {}
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Discord Rich Presence Integration

Add to `Cargo.toml`:

```toml
[dependencies]
discord-rich-presence = "0.2"
```

In Rust:

```rust
use discord_rich_presence::{activity, DiscordIpc, DiscordIpcClient};

#[tauri::command]
fn update_discord_rpc(song_title: String, artist: String) -> Result<(), String> {
    let mut client = DiscordIpcClient::new("YOUR_APP_ID")
        .map_err(|e| e.to_string())?;
    
    client.connect().map_err(|e| e.to_string())?;
    
    let activity = activity::Activity::new()
        .state(&artist)
        .details(&song_title)
        .assets(
            activity::Assets::new()
                .large_image("music_icon")
                .large_text("Listening to music")
        );
    
    client.set_activity(activity)
        .map_err(|e| e.to_string())?;
    
    Ok(())
}
```

### Local Storage / Cache Management

```typescript
import { BaseDirectory, createDir, writeTextFile, readTextFile } from '@tauri-apps/api/fs';

// Save cache
async function saveCachedData(key: string, data: any): Promise<void> {
  const json = JSON.stringify(data);
  await writeTextFile(`cache/${key}.json`, json, {
    dir: BaseDirectory.AppData,
  });
}

// Read cache
async function readCachedData<T>(key: string): Promise<T | null> {
  try {
    const json = await readTextFile(`cache/${key}.json`, {
      dir: BaseDirectory.AppData,
    });
    return JSON.parse(json);
  } catch {
    return null;
  }
}

// Initialize cache directory
async function initCache(): Promise<void> {
  await createDir('cache', {
    dir: BaseDirectory.AppData,
    recursive: true,
  });
}
```

## Working with YouTube Music Integration

### Injecting JavaScript into WebView

In `tauri.conf.json`, configure a WebView window for YouTube Music:

```json
{
  "tauri": {
    "windows": [
      {
        "label": "youtube-music",
        "url": "https://music.youtube.com",
        "title": "YouTube Music",
        "width": 1200,
        "height": 800
      }
    ]
  }
}
```

Inject scripts to extract player state:

```typescript
import { WebviewWindow } from '@tauri-apps/api/window';

const ytMusicWindow = WebviewWindow.getByLabel('youtube-music');

// Execute JavaScript in the YouTube Music webview
async function getCurrentPlaybackState() {
  const result = await ytMusicWindow?.eval(`
    (function() {
      const player = document.querySelector('ytmusic-player-bar');
      if (!player) return null;
      
      return {
        title: player.querySelector('.title')?.textContent,
        artist: player.querySelector('.byline')?.textContent,
        thumbnail: player.querySelector('img')?.src,
        isPlaying: player.getAttribute('player-state') === '1',
      };
    })()
  `);
  
  return result;
}
```

## Troubleshooting

### Rust Not Installed

```bash
# Install Rust via rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# On Windows, download from https://rustup.rs/

# Verify installation
rustc --version
cargo --version
```

### WebView2 Missing (Windows)

Download and install: https://developer.microsoft.com/en-us/microsoft-edge/webview2/

### Build Fails with Missing Dependencies (Linux)

```bash
# Debian/Ubuntu
sudo apt install libwebkit2gtk-4.0-dev \
    build-essential \
    curl \
    wget \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev

# Fedora
sudo dnf install webkit2gtk4.0-devel \
    openssl-devel \
    curl \
    wget \
    libappindicator-gtk3 \
    librsvg2-devel

# Arch
sudo pacman -S webkit2gtk \
    base-devel \
    curl \
    wget \
    openssl \
    appmenu-gtk-module \
    gtk3 \
    libappindicator-gtk3 \
    librsvg
```

### macOS Keychain Popup

The app uses macOS Keychain to store encryption keys for YouTube Music sessions. Grant access with "Always Allow" to prevent repeated prompts.

### Hot Reload Not Working

```bash
# Clear cache and restart
rm -rf node_modules dist src-tauri/target
npm install
npm run tauri dev
```

### CORS Issues with YouTube Music

Configure CSP in `tauri.conf.json`:

```json
{
  "tauri": {
    "security": {
      "csp": "default-src 'self'; connect-src 'self' https://music.youtube.com https://*.youtube.com https://*.googlevideo.com; img-src 'self' https: data:; media-src 'self' https: blob:; style-src 'self' 'unsafe-inline';"
    }
  }
}
```

### App Crashes on Startup

Check console logs:

```bash
# Run with debug output
RUST_LOG=debug npm run tauri dev

# Check Tauri logs
# Windows: %APPDATA%/com.2late.justanothermusicclient/logs
# macOS: ~/Library/Logs/com.2late.justanothermusicclient
# Linux: ~/.config/com.2late.justanothermusicclient/logs
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make changes and test locally: `npm run tauri dev`
4. Ensure types pass: `npm run type-check`
5. Build to verify: `npm run tauri build`
6. Commit changes: `git commit -m "Add feature"`
7. Push and create PR

By contributing, you agree to the project's Contributor License Agreement (CLA).

## License

Apache-2.0 License - See LICENSE file for details.

**Legal Notice:** This is an independent project not affiliated with YouTube or Google. No downloading functionality is provided. Access to YouTube Music services remains governed by their respective terms of service.
