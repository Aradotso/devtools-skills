---
name: tiez-clipboard-manager
description: TieZ is a Tauri-based cross-platform clipboard manager with history, tags, sync (WebDAV/MQTT), privacy protection, and fast workflows for managing clipboard data across devices.
triggers:
  - how do I set up TieZ clipboard manager
  - configure TieZ clipboard sync with WebDAV
  - build TieZ clipboard from source
  - develop features for TieZ clipboard manager
  - customize TieZ clipboard themes
  - integrate MQTT sync in TieZ
  - troubleshoot TieZ clipboard build errors
  - add tauri commands to TieZ clipboard
---

# TieZ Clipboard Manager Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

TieZ is a cross-platform clipboard manager built with Tauri 2, React, and Rust. It provides clipboard history tracking, tag organization, privacy masking, cloud sync (WebDAV/MQTT), and multiple UI themes. The application uses a native Rust backend for performance and a TypeScript/React frontend for the UI.

**Key Technologies:**
- **Tauri 2**: Desktop framework (Rust backend + web frontend)
- **React**: Frontend UI framework
- **TypeScript**: Frontend language
- **Rust**: Backend logic, clipboard monitoring, file operations
- **WebDAV/MQTT**: Sync protocols for cross-device data

## Installation & Setup

### Prerequisites

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install Node.js (v18+)
# macOS
brew install node

# Windows (use installer from nodejs.org)

# Install pnpm
npm install -g pnpm
```

### Clone and Build

```bash
# Clone repository
git clone https://github.com/jimuzhe/tiez-clipboard.git
cd tiez-clipboard

# Install frontend dependencies
pnpm install

# Run in development mode
pnpm tauri dev

# Build for production
pnpm tauri build
```

### Development Structure

```
tiez-clipboard/
├── src/              # React frontend (TypeScript)
├── src-tauri/        # Rust backend
│   ├── src/
│   │   ├── main.rs
│   │   ├── clipboard/
│   │   ├── sync/
│   │   └── commands/
│   ├── Cargo.toml
│   └── tauri.conf.json
├── package.json
└── vite.config.ts
```

## Key Concepts

### Tauri Commands

TieZ uses Tauri commands to bridge Rust backend and React frontend:

**Backend (Rust):**
```rust
// src-tauri/src/commands/clipboard.rs
use tauri::State;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct ClipboardItem {
    pub id: String,
    pub content: String,
    pub content_type: String,
    pub timestamp: i64,
    pub tags: Vec<String>,
}

#[tauri::command]
pub async fn get_clipboard_history(
    limit: Option<usize>,
) -> Result<Vec<ClipboardItem>, String> {
    // Query database for clipboard history
    let items = query_history(limit.unwrap_or(100))
        .await
        .map_err(|e| e.to_string())?;
    Ok(items)
}

#[tauri::command]
pub async fn add_clipboard_item(
    content: String,
    content_type: String,
    tags: Vec<String>,
) -> Result<ClipboardItem, String> {
    // Store clipboard item
    let item = store_clipboard_item(content, content_type, tags)
        .await
        .map_err(|e| e.to_string())?;
    Ok(item)
}

#[tauri::command]
pub async fn delete_clipboard_item(id: String) -> Result<(), String> {
    remove_item(&id).await.map_err(|e| e.to_string())?;
    Ok(())
}
```

**Frontend (TypeScript/React):**
```typescript
// src/api/clipboard.ts
import { invoke } from '@tauri-apps/api/core';

export interface ClipboardItem {
  id: string;
  content: string;
  content_type: string;
  timestamp: number;
  tags: string[];
}

export async function getClipboardHistory(
  limit?: number
): Promise<ClipboardItem[]> {
  return await invoke<ClipboardItem[]>('get_clipboard_history', { limit });
}

export async function addClipboardItem(
  content: string,
  contentType: string,
  tags: string[]
): Promise<ClipboardItem> {
  return await invoke<ClipboardItem>('add_clipboard_item', {
    content,
    contentType,
    tags,
  });
}

export async function deleteClipboardItem(id: string): Promise<void> {
  await invoke('delete_clipboard_item', { id });
}
```

### Clipboard Monitoring

```rust
// src-tauri/src/clipboard/monitor.rs
use clipboard_master::{CallbackResult, ClipboardHandler, Master};
use std::sync::{Arc, Mutex};
use tauri::{AppHandle, Manager};

pub struct ClipboardMonitor {
    app_handle: AppHandle,
}

impl ClipboardHandler for ClipboardMonitor {
    fn on_clipboard_change(&mut self) -> CallbackResult {
        // Get clipboard content
        if let Ok(content) = get_clipboard_text() {
            // Emit event to frontend
            self.app_handle
                .emit_all("clipboard-changed", content)
                .unwrap_or_default();
        }
        CallbackResult::Next
    }
}

pub fn start_monitoring(app_handle: AppHandle) {
    std::thread::spawn(move || {
        let monitor = ClipboardMonitor { app_handle };
        Master::new(monitor).run().unwrap();
    });
}
```

### WebDAV Sync Integration

```rust
// src-tauri/src/sync/webdav.rs
use reqwest::Client;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct WebDAVConfig {
    pub url: String,
    pub username: String,
    pub password: String,
    pub path: String,
}

pub struct WebDAVSync {
    client: Client,
    config: WebDAVConfig,
}

impl WebDAVSync {
    pub fn new(config: WebDAVConfig) -> Self {
        Self {
            client: Client::new(),
            config,
        }
    }

    pub async fn upload_data(&self, data: &str) -> Result<(), String> {
        let url = format!("{}/{}", self.config.url, self.config.path);
        
        self.client
            .put(&url)
            .basic_auth(&self.config.username, Some(&self.config.password))
            .body(data.to_string())
            .send()
            .await
            .map_err(|e| e.to_string())?;
        
        Ok(())
    }

    pub async fn download_data(&self) -> Result<String, String> {
        let url = format!("{}/{}", self.config.url, self.config.path);
        
        let response = self.client
            .get(&url)
            .basic_auth(&self.config.username, Some(&self.config.password))
            .send()
            .await
            .map_err(|e| e.to_string())?;
        
        response.text().await.map_err(|e| e.to_string())
    }
}

#[tauri::command]
pub async fn sync_to_webdav(
    config: WebDAVConfig,
    data: String,
) -> Result<(), String> {
    let sync = WebDAVSync::new(config);
    sync.upload_data(&data).await
}
```

### MQTT Real-time Sync

```rust
// src-tauri/src/sync/mqtt.rs
use rumqttc::{AsyncClient, MqttOptions, QoS};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct MQTTConfig {
    pub broker: String,
    pub port: u16,
    pub client_id: String,
    pub username: Option<String>,
    pub password: Option<String>,
    pub topic: String,
}

pub async fn publish_clipboard_item(
    config: &MQTTConfig,
    item: &ClipboardItem,
) -> Result<(), String> {
    let mut mqttoptions = MqttOptions::new(
        &config.client_id,
        &config.broker,
        config.port,
    );
    
    if let (Some(username), Some(password)) = 
        (&config.username, &config.password) {
        mqttoptions.set_credentials(username, password);
    }

    let (client, mut eventloop) = AsyncClient::new(mqttoptions, 10);
    
    let payload = serde_json::to_string(item)
        .map_err(|e| e.to_string())?;
    
    client
        .publish(&config.topic, QoS::AtLeastOnce, false, payload)
        .await
        .map_err(|e| e.to_string())?;
    
    Ok(())
}

#[tauri::command]
pub async fn mqtt_sync_item(
    config: MQTTConfig,
    item: ClipboardItem,
) -> Result<(), String> {
    publish_clipboard_item(&config, &item).await
}
```

## Frontend Development

### React Component Example

```typescript
// src/components/ClipboardHistory.tsx
import React, { useEffect, useState } from 'react';
import { listen } from '@tauri-apps/api/event';
import { getClipboardHistory, ClipboardItem } from '../api/clipboard';

export const ClipboardHistory: React.FC = () => {
  const [items, setItems] = useState<ClipboardItem[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Load initial history
    loadHistory();

    // Listen for clipboard changes
    const unlisten = listen<string>('clipboard-changed', (event) => {
      console.log('Clipboard changed:', event.payload);
      loadHistory();
    });

    return () => {
      unlisten.then((fn) => fn());
    };
  }, []);

  const loadHistory = async () => {
    setLoading(true);
    try {
      const history = await getClipboardHistory(50);
      setItems(history);
    } catch (error) {
      console.error('Failed to load history:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="clipboard-history">
      {loading ? (
        <div>Loading...</div>
      ) : (
        <ul>
          {items.map((item) => (
            <li key={item.id}>
              <div className="content">{item.content}</div>
              <div className="tags">
                {item.tags.map((tag) => (
                  <span key={tag} className="tag">
                    {tag}
                  </span>
                ))}
              </div>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

### Settings Management

```typescript
// src/api/settings.ts
import { invoke } from '@tauri-apps/api/core';

export interface AppSettings {
  theme: 'frosted' | 'notebook' | 'sticky' | '3d';
  darkMode: boolean;
  autoStart: boolean;
  maxHistoryItems: number;
  syncEnabled: boolean;
  webdavConfig?: {
    url: string;
    username: string;
    path: string;
  };
  mqttConfig?: {
    broker: string;
    port: number;
    clientId: string;
    topic: string;
  };
}

export async function getSettings(): Promise<AppSettings> {
  return await invoke<AppSettings>('get_settings');
}

export async function updateSettings(
  settings: Partial<AppSettings>
): Promise<void> {
  await invoke('update_settings', { settings });
}
```

```rust
// src-tauri/src/commands/settings.rs
use tauri::State;
use serde::{Deserialize, Serialize};
use std::sync::Mutex;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct AppSettings {
    pub theme: String,
    pub dark_mode: bool,
    pub auto_start: bool,
    pub max_history_items: usize,
    pub sync_enabled: bool,
}

pub struct SettingsState(pub Mutex<AppSettings>);

#[tauri::command]
pub fn get_settings(state: State<SettingsState>) -> Result<AppSettings, String> {
    let settings = state.0.lock().map_err(|e| e.to_string())?;
    Ok(settings.clone())
}

#[tauri::command]
pub fn update_settings(
    settings: AppSettings,
    state: State<SettingsState>,
) -> Result<(), String> {
    let mut current = state.0.lock().map_err(|e| e.to_string())?;
    *current = settings;
    // Persist to disk
    save_settings_to_file(&current)?;
    Ok(())
}
```

## Configuration

### Tauri Configuration

```json
// src-tauri/tauri.conf.json
{
  "build": {
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build",
    "devPath": "http://localhost:5173",
    "distDir": "../dist"
  },
  "package": {
    "productName": "TieZ",
    "version": "1.0.0"
  },
  "tauri": {
    "allowlist": {
      "all": false,
      "clipboard": {
        "all": true
      },
      "fs": {
        "all": true,
        "scope": ["$APPDATA/*", "$HOME/.tiez/*"]
      },
      "http": {
        "all": true,
        "scope": ["https://**", "http://**"]
      }
    },
    "windows": [
      {
        "title": "TieZ Clipboard",
        "width": 800,
        "height": 600,
        "decorations": true,
        "transparent": true
      }
    ],
    "systemTray": {
      "iconPath": "icons/icon.png",
      "menuOnLeftClick": false
    }
  }
}
```

### Environment Variables

```bash
# .env (for development)
VITE_APP_NAME=TieZ
VITE_DEFAULT_WEBDAV_URL=https://dav.example.com
VITE_DEFAULT_MQTT_BROKER=mqtt.example.com
VITE_DEFAULT_MQTT_PORT=1883
```

## Common Development Patterns

### Adding a New Tauri Command

1. **Define command in Rust:**

```rust
// src-tauri/src/commands/custom.rs
#[tauri::command]
pub async fn my_custom_command(param: String) -> Result<String, String> {
    // Implementation
    Ok(format!("Processed: {}", param))
}
```

2. **Register in main.rs:**

```rust
// src-tauri/src/main.rs
mod commands;

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![
            commands::clipboard::get_clipboard_history,
            commands::custom::my_custom_command,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

3. **Call from frontend:**

```typescript
// src/api/custom.ts
import { invoke } from '@tauri-apps/api/core';

export async function myCustomCommand(param: string): Promise<string> {
  return await invoke<string>('my_custom_command', { param });
}
```

### Creating a New Theme

```typescript
// src/themes/custom-theme.ts
export const customTheme = {
  name: 'custom',
  colors: {
    primary: '#4CAF50',
    secondary: '#2196F3',
    background: '#FFFFFF',
    surface: '#F5F5F5',
    text: '#212121',
  },
  effects: {
    blur: 10,
    transparency: 0.9,
  },
};
```

```css
/* src/styles/themes/custom.css */
.theme-custom {
  --color-primary: #4caf50;
  --color-secondary: #2196f3;
  --color-background: #ffffff;
  --color-surface: #f5f5f5;
  --color-text: #212121;
  
  backdrop-filter: blur(10px);
  background: rgba(255, 255, 255, 0.9);
}
```

### Privacy Masking Implementation

```rust
// src-tauri/src/privacy/masking.rs
use regex::Regex;

pub fn mask_sensitive_data(content: &str) -> String {
    let mut masked = content.to_string();
    
    // Mask phone numbers
    let phone_regex = Regex::new(r"\b\d{3}-\d{3,4}-\d{4}\b").unwrap();
    masked = phone_regex.replace_all(&masked, "***-****-****").to_string();
    
    // Mask email addresses
    let email_regex = Regex::new(r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b").unwrap();
    masked = email_regex.replace_all(&masked, "***@***.***").to_string();
    
    // Mask credit card numbers
    let cc_regex = Regex::new(r"\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b").unwrap();
    masked = cc_regex.replace_all(&masked, "**** **** **** ****").to_string();
    
    masked
}

#[tauri::command]
pub fn get_masked_preview(content: String) -> String {
    mask_sensitive_data(&content)
}
```

## Troubleshooting

### Build Errors

**Issue: Tauri build fails with "command not found"**
```bash
# Ensure Rust is in PATH
source $HOME/.cargo/env

# Update Rust
rustup update

# Clean and rebuild
cargo clean
pnpm tauri build
```

**Issue: Frontend build fails**
```bash
# Clear node_modules and reinstall
rm -rf node_modules pnpm-lock.yaml
pnpm install

# Clear vite cache
rm -rf node_modules/.vite
pnpm dev
```

### Runtime Issues

**Issue: Clipboard monitoring not working**
```rust
// Ensure clipboard permissions in tauri.conf.json
"allowlist": {
  "clipboard": {
    "all": true,
    "readText": true,
    "writeText": true
  }
}
```

**Issue: WebDAV sync fails with CORS error**
- WebDAV requests are made from Rust backend (no CORS)
- Ensure `http.all: true` in tauri.conf.json allowlist
- Check network connectivity and credentials

**Issue: App doesn't start on system boot**
```rust
// src-tauri/src/main.rs
use tauri_plugin_autostart::MacosLauncher;

tauri::Builder::default()
    .plugin(tauri_plugin_autostart::init(
        MacosLauncher::LaunchAgent,
        Some(vec!["--flag1", "--flag2"])
    ))
```

### Development Tips

```bash
# Hot reload for Rust changes
cargo watch -x 'run --features tauri/dev'

# Debug Tauri commands
RUST_LOG=debug pnpm tauri dev

# Build with verbose output
pnpm tauri build --verbose

# Check bundle size
pnpm tauri build -- --bundles deb,app,msi,dmg
```

### Database Migration

```rust
// src-tauri/src/db/migrations.rs
use rusqlite::{Connection, Result};

pub fn migrate_database(conn: &Connection) -> Result<()> {
    conn.execute(
        "CREATE TABLE IF NOT EXISTS clipboard_items (
            id TEXT PRIMARY KEY,
            content TEXT NOT NULL,
            content_type TEXT NOT NULL,
            timestamp INTEGER NOT NULL,
            tags TEXT
        )",
        [],
    )?;
    
    conn.execute(
        "CREATE INDEX IF NOT EXISTS idx_timestamp 
         ON clipboard_items(timestamp DESC)",
        [],
    )?;
    
    Ok(())
}
```

## Testing

```typescript
// src/tests/clipboard.test.ts
import { describe, it, expect, beforeAll } from 'vitest';
import { mockIPC } from '@tauri-apps/api/mocks';
import { getClipboardHistory } from '../api/clipboard';

describe('Clipboard API', () => {
  beforeAll(() => {
    mockIPC((cmd, args) => {
      if (cmd === 'get_clipboard_history') {
        return [
          {
            id: '1',
            content: 'test',
            content_type: 'text',
            timestamp: Date.now(),
            tags: [],
          },
        ];
      }
    });
  });

  it('should fetch clipboard history', async () => {
    const history = await getClipboardHistory();
    expect(history).toHaveLength(1);
    expect(history[0].content).toBe('test');
  });
});
```

```rust
// src-tauri/src/tests/privacy_test.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_phone_masking() {
        let content = "Call me at 123-456-7890";
        let masked = mask_sensitive_data(content);
        assert_eq!(masked, "Call me at ***-****-****");
    }

    #[test]
    fn test_email_masking() {
        let content = "Email: user@example.com";
        let masked = mask_sensitive_data(content);
        assert_eq!(masked, "Email: ***@***.***");
    }
}
```
