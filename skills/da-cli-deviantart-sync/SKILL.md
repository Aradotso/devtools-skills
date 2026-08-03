---
name: da-cli-deviantart-sync
description: Sync DeviantArt galleries to local folders with OAuth 2.1 PKCE, SQLite indexing, and scheduled syncs using da-cli
triggers:
  - sync my DeviantArt gallery
  - download art from DeviantArt
  - set up DeviantArt backup
  - authenticate with DeviantArt API
  - schedule automatic DeviantArt sync
  - search DeviantArt from command line
  - backup watched artists on DeviantArt
  - configure da-cli for DeviantArt
---

# da-cli DeviantArt Sync Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

da-cli is a zero-dependency Python CLI tool that syncs DeviantArt galleries to local folders. It uses OAuth 2.1 with PKCE for authentication, maintains a SQLite index for incremental syncs, and supports scheduled backups via launchd (macOS) or systemd (Linux). Key features:

- **Zero runtime dependencies** — pure Python 3.10+ stdlib
- **Incremental sync** — SQLite index tracks what's downloaded; re-runs cost one API call when nothing's new
- **OAuth 2.1 PKCE** — secure authentication with automatic token refresh
- **Multiple sync modes** — watch feed, specific artists, all watched users
- **Scheduled automation** — launchd/systemd integration for unattended syncs
- **Search & browse** — tags, topics, daily deviations, user profiles
- **Secrets in Keychain** — macOS Keychain integration for client secrets

## Installation

### Option A: pipx (recommended)

```bash
# Install with isolated environment
pipx install da-sync

# Verify installation
da --version
```

### Option B: git clone (for development)

```bash
# Clone repository
git clone https://github.com/FZ2000/da-cli.git ~/Documents/da-cli
cd ~/Documents/da-cli

# Install shim (creates ~/.local/bin/da)
./install.sh

# Verify
da --version
```

**Requirements:** Python 3.10+ (uses `argparse.BooleanOptionalAction` and `X | None` syntax)

## Initial Setup

### 1. Create DeviantArt OAuth App

```bash
# Navigate to DeviantArt developers portal
# https://www.deviantart.com/developers/
# Click "Register Your Application"
# Set these values:
#   - Application Type: Confidential
#   - Redirect URI Whitelist: https://localhost:8765/ (exact, with trailing slash)
# Save your client_id and client_secret
```

### 2. Configure da-cli

```bash
# Set client credentials (secret goes to Keychain on macOS)
da config set client_id YOUR_CLIENT_ID
da config set client_secret YOUR_CLIENT_SECRET

# Set download destination
da config set destination ~/Pictures/DA

# Optional: customize scope (defaults to browse+collection+user)
da config set scope "browse collection user"

# View current configuration
da config show
```

### 3. Authenticate

```bash
# Start OAuth flow (opens browser)
da auth

# Verify authentication
da whoami

# Check token status
da refresh
```

## Core Commands

### Authentication

```bash
# Initial login (browser-based PKCE flow)
da auth

# Check current user
da whoami

# Force token refresh
da refresh

# Logout (clears local tokens)
da auth logout

# Check auth status
da auth status
```

### Syncing Art

```bash
# Sync your watch feed (incremental)
da sync feed

# Sync without mature content
da sync feed --no-mature

# Sync specific artist's full gallery
da sync artist username

# Sync all watched artists
da sync watched

# Discover watched artists via feed (works with browse scope only)
da sync watched --via-feed

# Add random jitter to avoid burst patterns
da sync feed --jitter 0.4

# Limit sync time
da sync feed --max-minutes 30

# Resume interrupted sync
da sync feed  # automatically resumes from checkpoint
```

### Search & Browse

```bash
# Search by tag
da search tag nature
da search tag "digital art" --limit 50

# Browse curated topics
da search topic digitalart
da search topics  # list all valid topics

# Daily deviations
da daily 2026-01-15
da daily today

# User lookup
da search user deviantart

# Get user profile
da user profile username

# Deviation details
da deviation show 123456789
da deviation morelikethis 123456789

# List watched users (requires user scope)
da watch list
```

### Configuration Management

```bash
# Show all configuration
da config show

# Show config file locations
da config path

# Set value (secrets go to Keychain on macOS)
da config set key value
da config set max_retries 5
da config set sleep_between_requests 1.5

# Get value (secrets masked by default)
da config get client_secret
da config get client_secret --unmask

# Remove setting
da config unset key
```

### Index & Maintenance

```bash
# Show sync index stats
da index show

# Rebuild index from destination folder
da index rebuild

# Run health check
da diagnose

# Benchmark sync performance
da bench
```

## Scheduling Automated Syncs

### macOS (launchd)

```bash
# Install daily sync at 03:00
./install_schedule.sh

# Manual launchd setup
cat > ~/Library/LaunchAgents/com.user.da-sync.plist <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.da-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/USERNAME/.local/bin/da</string>
        <string>sync</string>
        <string>feed</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/Users/USERNAME/.local/state/da-cli/sync.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/USERNAME/.local/state/da-cli/sync.log</string>
</dict>
</plist>
EOF

# Load the job
launchctl load ~/Library/LaunchAgents/com.user.da-sync.plist

# Verify
launchctl list | grep da-sync
```

**Important:** Grant Full Disk Access to Terminal/iTerm in System Preferences → Privacy & Security → Full Disk Access

### Linux (systemd)

```bash
# Create service unit
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/da-sync.service <<'EOF'
[Unit]
Description=DeviantArt sync

[Service]
Type=oneshot
ExecStart=/home/USERNAME/.local/bin/da sync feed
StandardOutput=append:/home/USERNAME/.local/state/da-cli/sync.log
StandardError=append:/home/USERNAME/.local/state/da-cli/sync.log
EOF

# Create timer unit
cat > ~/.config/systemd/user/da-sync.timer <<'EOF'
[Unit]
Description=Daily DeviantArt sync

[Timer]
OnCalendar=daily
OnCalendar=03:00
Persistent=true

[Install]
WantedBy=timers.target
EOF

# Enable and start
systemctl --user daemon-reload
systemctl --user enable da-sync.timer
systemctl --user start da-sync.timer

# Enable lingering (allows timer to run when not logged in)
loginctl enable-linger $USER

# Check status
systemctl --user list-timers
systemctl --user status da-sync.timer
```

## Python Scripting

### Export Sync Index to CSV

```python
#!/usr/bin/env python3
import sqlite3
import csv
from pathlib import Path

def export_index_to_csv(output_path: str = "sync_index.csv"):
    """Export da-cli sync index to CSV."""
    # Default index location
    db_path = Path.home() / ".local/state/da-cli/sync.db"
    
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    cursor.execute("""
        SELECT deviation_id, title, author, published_time, 
               content_url, is_mature, local_path
        FROM deviations
        ORDER BY published_time DESC
    """)
    
    with open(output_path, 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(['deviation_id', 'title', 'author', 'published_time',
                        'content_url', 'is_mature', 'local_path'])
        writer.writerows(cursor.fetchall())
    
    conn.close()
    print(f"Exported to {output_path}")

if __name__ == "__main__":
    export_index_to_csv()
```

### Post-Sync Webhook

```python
#!/usr/bin/env python3
import json
import sqlite3
import subprocess
from pathlib import Path
from datetime import datetime, timedelta

def check_new_deviations(since_hours: int = 24):
    """Check for deviations synced in the last N hours."""
    db_path = Path.home() / ".local/state/da-cli/sync.db"
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    cutoff = datetime.now() - timedelta(hours=since_hours)
    cutoff_str = cutoff.isoformat()
    
    cursor.execute("""
        SELECT COUNT(*), author
        FROM deviations
        WHERE synced_at > ?
        GROUP BY author
        ORDER BY COUNT(*) DESC
    """, (cutoff_str,))
    
    results = cursor.fetchall()
    conn.close()
    
    if results:
        print(f"New deviations in last {since_hours}h:")
        for count, author in results:
            print(f"  {author}: {count} new")
        return True
    return False

def send_notification(message: str):
    """Send macOS notification."""
    subprocess.run([
        'osascript', '-e',
        f'display notification "{message}" with title "da-cli sync"'
    ])

if __name__ == "__main__":
    if check_new_deviations(24):
        send_notification("New DeviantArt content synced!")
```

### Query Specific Artist

```python
#!/usr/bin/env python3
import sqlite3
from pathlib import Path

def get_artist_stats(username: str):
    """Get stats for a specific artist in sync index."""
    db_path = Path.home() / ".local/state/da-cli/sync.db"
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    # Count deviations
    cursor.execute("""
        SELECT COUNT(*), MIN(published_time), MAX(published_time)
        FROM deviations
        WHERE author = ?
    """, (username,))
    
    count, first_date, last_date = cursor.fetchone()
    
    # Get recent titles
    cursor.execute("""
        SELECT title, published_time, local_path
        FROM deviations
        WHERE author = ?
        ORDER BY published_time DESC
        LIMIT 5
    """, (username,))
    
    recent = cursor.fetchall()
    conn.close()
    
    print(f"\n{username}:")
    print(f"  Total deviations: {count}")
    print(f"  Date range: {first_date} to {last_date}")
    print(f"\n  Recent 5:")
    for title, pub_time, path in recent:
        print(f"    - {title} ({pub_time})")
        print(f"      {path}")

if __name__ == "__main__":
    import sys
    if len(sys.argv) < 2:
        print("Usage: artist_stats.py USERNAME")
        sys.exit(1)
    get_artist_stats(sys.argv[1])
```

## Configuration Reference

### Config File Locations

```bash
# Config file (settings)
~/.config/da-cli/config.json

# State file (tokens)
~/.local/state/da-cli/state.json

# Sync index (SQLite)
~/.local/state/da-cli/sync.db

# Logs
~/.local/state/da-cli/sync.log
```

### Key Settings

```json
{
  "client_id": "12345",
  "destination": "/Users/username/Pictures/DA",
  "scope": "browse collection user",
  "mature_content": true,
  "sleep_between_requests": 1.0,
  "max_retries": 3,
  "checkpoint_interval": 50,
  "default_limit": 24
}
```

### Environment Variables

```bash
# Override config location
export DA_CLI_CONFIG_DIR=~/custom/config/path

# Override state location
export DA_CLI_STATE_DIR=~/custom/state/path

# Set client credentials (alternative to config file)
export DA_CLIENT_ID=your_client_id
export DA_CLIENT_SECRET=your_client_secret

# Set destination
export DA_DESTINATION=~/Pictures/DA
```

## Common Patterns

### Initial Full Sync of Watched Artists

```bash
# First, sync watch feed to discover artists
da sync feed --limit 500

# Then backfill each artist's full gallery
da sync watched

# Or do it individually
da sync artist artist1
da sync artist artist2
```

### Sync with Rate Limiting

```bash
# Add random jitter to avoid detection
da sync feed --jitter 0.5

# Increase sleep between requests
da config set sleep_between_requests 2.0
da sync feed

# Limit total sync time
da sync feed --max-minutes 15
```

### Filter Content

```bash
# Exclude mature content
da sync feed --no-mature

# Sync only mature content (edit directly)
# In sync flow, filter is applied to API params
```

### Resume Interrupted Sync

```bash
# Sync automatically resumes from last checkpoint
# Checkpoint is saved every 50 deviations (configurable)
da sync feed

# View checkpoint
da index show | grep checkpoint

# Force fresh start (clear checkpoint)
da index rebuild
```

## Troubleshooting

### Redirect URI Mismatch

```bash
# Error: redirect_uri does not match
# Solution: Ensure OAuth app has EXACT redirect URI
# https://localhost:8765/ (with trailing slash)

# Verify in config
da config get redirect_uri
# Should show: https://localhost:8765/
```

### Expired Refresh Token

```bash
# Tokens expire after 90 days
# Check token status
da diagnose

# Re-authenticate
da auth logout
da auth
```

### Scheduled Job Not Running

```bash
# Check job status
da diagnose

# macOS: verify launchd
launchctl list | grep da-sync
launchctl start com.user.da-sync  # test immediately

# Linux: verify systemd
systemctl --user status da-sync.timer
systemctl --user status da-sync.service
journalctl --user -u da-sync.service
```

### Permission Denied on macOS

```bash
# Grant Full Disk Access to Terminal/iTerm
# System Preferences → Privacy & Security → Full Disk Access
# Add Terminal.app or iTerm.app
# Restart Terminal

# Verify permissions
ls -la ~/.local/state/da-cli/
```

### Missing Dependencies (dev)

```bash
# If running from git clone
cd ~/Documents/da-cli
make dev-setup  # installs dev dependencies (ruff, mypy, pytest)
```

### Keychain Access Issues

```bash
# If Keychain denies access
# Run once to trigger permission prompt
da config set client_secret YOUR_SECRET

# If still blocked, manually allow in Keychain Access.app
# Find "da-cli" items and set access to "Always Allow"
```

### Index Corruption

```bash
# Rebuild index from destination folder
da index rebuild

# Check index integrity
sqlite3 ~/.local/state/da-cli/sync.db "PRAGMA integrity_check;"
```

## API Scope Reference

```bash
# browse: search, topics, daily deviations, public galleries
# collection: sync your own collections/favorites
# user: watch list, user profile

# Default (recommended)
da config set scope "browse collection user"

# Minimal (read-only search/browse)
da config set scope "browse"

# Full access (includes messaging, stash - not used by da-cli)
da config set scope "browse collection user message stash"
```

## Best Practices

1. **Use pipx for installation** — keeps da-cli isolated from system Python
2. **Store secrets in Keychain** — never commit `config.json` with secrets
3. **Enable jitter for large syncs** — `--jitter 0.4` randomizes request timing
4. **Run `da diagnose` before troubleshooting** — checks all common issues
5. **Schedule syncs during off-hours** — reduces API load and bandwidth usage
6. **Use `--via-feed` for watched sync** — works without `user` scope
7. **Set checkpoint interval** — `da config set checkpoint_interval 100` for long syncs
8. **Monitor logs** — `tail -f ~/.local/state/da-cli/sync.log`

## Security Notes

- **Client secret**: stored in macOS Keychain (service: `da-cli`, account: `client_secret`)
- **Tokens**: stored in `~/.local/state/da-cli/state.json` with 0600 permissions
- **PKCE mandatory**: `code_verifier` generated per-auth, never leaves machine
- **No telemetry**: zero network calls except to DeviantArt API and image CDNs
- **All HTTPS**: API and image downloads are TLS-encrypted
- **`.gitignore` included**: secrets-bearing files cannot be accidentally committed

## Resources

- **Documentation**: https://github.com/FZ2000/da-cli/tree/main/docs
- **Setup Guide**: https://github.com/FZ2000/da-cli/blob/main/docs/getting-started.md
- **Scheduling Guide**: https://github.com/FZ2000/da-cli/blob/main/docs/guides/scheduling.md
- **Troubleshooting**: https://github.com/FZ2000/da-cli/blob/main/docs/guides/troubleshooting.md
- **Security Model**: https://github.com/FZ2000/da-cli/blob/main/docs/explanation/security.md
- **Contributing**: https://github.com/FZ2000/da-cli/blob/main/CONTRIBUTING.md
- **Architecture**: https://github.com/FZ2000/da-cli/blob/main/ARCHITECTURE.md
