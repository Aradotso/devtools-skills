---
name: wechat-cli-local-data
description: Query local WeChat chat history, contacts, sessions, and favorites from the command line with AI-first JSON output
triggers:
  - check my wechat messages
  - search wechat chat history
  - export wechat conversation
  - get wechat unread messages
  - show wechat contacts
  - analyze wechat chat statistics
  - query my local wechat data
  - read wechat group messages
---

# WeChat CLI — Local Data Query Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

A CLI tool to query your local WeChat data — chat history, contacts, sessions, favorites, and more. Designed for LLM integration with JSON output by default. All data stays on your machine; uses on-the-fly SQLCipher decryption.

## Installation

### npm (Recommended)

```bash
npm install -g @canghe_ai/wechat-cli
```

Currently ships with macOS arm64 binary. For other platforms, use pip:

```bash
pip install wechat-cli
```

Requires Python >= 3.10.

### From Source

```bash
git clone https://github.com/freestylefly/wechat-cli.git
cd wechat-cli
pip install -e .
```

## Initial Setup

Before using any commands, you must initialize the tool to extract WeChat encryption keys:

```bash
# macOS/Linux (requires sudo for memory scanning)
sudo wechat-cli init

# Windows (run in elevated terminal)
wechat-cli init
```

**Prerequisites:**
- WeChat must be running
- On macOS: Terminal needs "Full Disk Access" (System Settings → Privacy & Security → Full Disk Access)

This command:
1. Auto-detects WeChat data directory
2. Scans WeChat process memory for encryption keys
3. Saves config to `~/.wechat-cli/`

### macOS: Handling `task_for_pid failed` Error

If `init` fails with `task_for_pid failed`, the tool will automatically re-sign WeChat with required entitlements:

1. Follow on-screen instructions
2. Quit WeChat completely
3. Reopen WeChat and log in
4. Run `sudo wechat-cli init` again

Manual re-signing (if auto-signing fails):

```bash
sudo codesign --force --sign - --entitlements /dev/stdin /Applications/WeChat.app <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.get-task-allow</key>
    <true/>
</dict>
</plist>
EOF
```

## Core Commands

### Sessions — List Recent Chats

```bash
# Get last 20 sessions (default, JSON output)
wechat-cli sessions

# Get last 10 sessions
wechat-cli sessions --limit 10

# Human-readable format
wechat-cli sessions --format text
```

**JSON Output Structure:**
```json
[
  {
    "chat_name": "Alice",
    "last_message": "See you tomorrow!",
    "last_time": "2026-04-15 18:30:45",
    "unread_count": 2
  }
]
```

### History — Read Chat Messages

```bash
# Last 50 messages with Alice (default)
wechat-cli history "Alice"

# Get 100 messages with pagination
wechat-cli history "Alice" --limit 100 --offset 50

# Filter by date range
wechat-cli history "Team Group" --start-time "2026-04-01" --end-time "2026-04-03"

# Filter by message type (text, image, voice, video, file, link, etc.)
wechat-cli history "Alice" --type link

# Text format for reading
wechat-cli history "Alice" --format text --limit 20
```

**JSON Output Structure:**
```json
[
  {
    "msg_id": "123456",
    "type": "text",
    "content": "Hello!",
    "sender": "Alice",
    "timestamp": "2026-04-15 10:30:00",
    "is_sender": false
  }
]
```

### Search — Find Messages

```bash
# Global search across all chats
wechat-cli search "deadline"

# Search in specific chat
wechat-cli search "meeting notes" --chat "Work Team"

# Search in multiple chats
wechat-cli search "report" --chat "TeamA" --chat "TeamB"

# Search with type filter (only files)
wechat-cli search "proposal" --type file

# Date range search
wechat-cli search "budget" --start-time "2026-04-01" --end-time "2026-04-30"
```

### Contacts — Search & Details

```bash
# Search contacts by name
wechat-cli contacts --query "Li"

# Get contact details by name
wechat-cli contacts --detail "Alice"

# Get contact details by WeChat ID
wechat-cli contacts --detail "wxid_abc123xyz"
```

**JSON Output (Detail):**
```json
{
  "nickname": "Alice",
  "remark": "Alice from Marketing",
  "wechat_id": "wxid_abc123xyz",
  "bio": "Always learning",
  "avatar_url": "https://...",
  "account_type": "personal"
}
```

### Members — Group Member List

```bash
# Get all members in a group (JSON)
wechat-cli members "AI Research Group"

# Text format for reading
wechat-cli members "AI Research Group" --format text
```

### Stats — Chat Analytics

```bash
# Get statistics for a chat
wechat-cli stats "Team Group"

# Stats for date range
wechat-cli stats "Alice" --start-time "2026-04-01" --end-time "2026-04-03"

# Text format with charts
wechat-cli stats "Team Group" --format text
```

**Returns:**
- Total message count
- Message type breakdown (text, image, file, etc.)
- Top 10 senders with message counts
- 24-hour activity distribution

### Export — Save Conversations

```bash
# Export to markdown (stdout)
wechat-cli export "Alice" --format markdown

# Export to file
wechat-cli export "Alice" --format txt --output alice_chat.txt

# Export with filters
wechat-cli export "Team Group" --start-time "2026-04-01" --limit 1000 --format markdown --output team_april.md
```

**Supported formats:** `markdown`, `txt`

### Favorites — WeChat Bookmarks

```bash
# Get all favorites (recent first)
wechat-cli favorites

# Filter by type
wechat-cli favorites --type article

# Search favorites
wechat-cli favorites --query "machine learning"
```

**Types:** `text`, `image`, `article`, `card`, `video`

### Unread — Unread Sessions

```bash
# Get all unread sessions
wechat-cli unread

# Limit results
wechat-cli unread --limit 10 --format text
```

### New Messages — Incremental Updates

```bash
# First run: returns unread + saves state
wechat-cli new-messages

# Subsequent runs: only new messages since last check
wechat-cli new-messages

# Text format for monitoring
wechat-cli new-messages --format text
```

State saved at `~/.wechat-cli/last_check.json`. Delete to reset.

**Use Case:** Perfect for automation/cron jobs to monitor new messages.

## AI Agent Integration

WeChat CLI is designed for AI agent tool calls. All commands output JSON by default.

### Claude Code Integration

Add to your project's `CLAUDE.md`:

````markdown
## WeChat CLI

You can use `wechat-cli` to query my local WeChat data.

Common commands:
- `wechat-cli sessions --limit 10` — list recent chats
- `wechat-cli history "NAME" --limit 20 --format text` — read chat history
- `wechat-cli search "KEYWORD" --chat "CHAT_NAME"` — search messages
- `wechat-cli contacts --query "NAME"` — search contacts
- `wechat-cli unread` — show unread sessions
- `wechat-cli new-messages` — get messages since last check
- `wechat-cli members "GROUP"` — list group members
- `wechat-cli stats "CHAT" --format text` — chat statistics

All commands output JSON by default. Add `--format text` for human-readable output.
````

### Example Agent Workflows

**Check unread messages:**
```bash
# Agent executes:
wechat-cli unread --format text

# Then summarizes results for user
```

**Search for project updates:**
```bash
# Agent executes:
wechat-cli search "project status" --chat "Work Team" --start-time "2026-04-01"

# Then extracts key information
```

**Analyze chat activity:**
```bash
# Agent executes:
wechat-cli stats "AI Discussion Group" --format text

# Then presents insights: most active hours, top contributors, etc.
```

**Export important conversation:**
```bash
# Agent executes:
wechat-cli export "Client Meetings" --start-time "2026-04-01" --end-time "2026-04-07" --format markdown --output client_weekly.md

# Then confirms export and location
```

## Message Type Filters

Use with `--type` option on `history` and `search` commands:

| Type | Description |
|------|-------------|
| `text` | Text messages |
| `image` | Images |
| `voice` | Voice messages |
| `video` | Videos |
| `sticker` | Stickers/emojis |
| `location` | Location shares |
| `link` | Links and app messages |
| `file` | File attachments |
| `call` | Voice/video calls |
| `system` | System messages |

## Common Patterns

### Monitor New Messages in Script

```bash
#!/bin/bash
# Check for new messages every 5 minutes
while true; do
  NEW_MSGS=$(wechat-cli new-messages --format text)
  if [ -n "$NEW_MSGS" ]; then
    echo "New messages detected:"
    echo "$NEW_MSGS"
    # Send notification, etc.
  fi
  sleep 300
done
```

### Search Across Multiple Groups

```bash
# Find deadline mentions in all project groups
wechat-cli search "deadline" \
  --chat "Project Alpha" \
  --chat "Project Beta" \
  --chat "Project Gamma" \
  --format text
```

### Export Weekly Summaries

```bash
# Export last week's messages from key chats
START_DATE=$(date -d "7 days ago" +%Y-%m-%d)
END_DATE=$(date +%Y-%m-%d)

wechat-cli export "Team Sync" \
  --start-time "$START_DATE" \
  --end-time "$END_DATE" \
  --format markdown \
  --output "team_sync_weekly.md"
```

### Get Contact Details Before Messaging

```bash
# Check if contact exists and get details
CONTACT_INFO=$(wechat-cli contacts --detail "New Client")
if [ $? -eq 0 ]; then
  echo "Contact found:"
  echo "$CONTACT_INFO" | jq '.nickname, .wechat_id'
else
  echo "Contact not found"
fi
```

## Configuration

Configuration stored at `~/.wechat-cli/`:

```
~/.wechat-cli/
├── config.json          # WeChat data path, account info
├── keys.json           # Encryption keys (auto-extracted)
└── last_check.json     # State for new-messages command
```

**config.json structure:**
```json
{
  "data_dir": "/Users/username/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/abc123",
  "account_id": "wxid_abc123xyz"
}
```

**To reset:** Delete `~/.wechat-cli/` and run `init` again.

**To switch accounts:** Run `init` again and select different account.

## Troubleshooting

### "WeChat data directory not found"

**Solution:**
- Ensure WeChat is running
- On macOS: Grant "Full Disk Access" to your terminal app
- Run `init` with `sudo` on macOS/Linux

### "Failed to extract encryption key"

**Solution:**
- On macOS: Re-sign WeChat (tool will auto-attempt this)
- Ensure WeChat is logged in and active
- Try restarting WeChat and running `init` again

### "Database decryption failed"

**Solution:**
- WeChat version changed — run `init` again to refresh keys
- Database corruption — check WeChat data directory permissions

### Empty Results for Valid Chat Name

**Solution:**
- Use exact chat name (case-sensitive)
- For groups, use full group name as shown in WeChat
- Use `sessions` command to see exact chat names
- Try searching with partial name using `contacts --query`

### `new-messages` Returns Everything

**Solution:**
- Delete `~/.wechat-cli/last_check.json` to reset state
- First run after reset will return all unread messages
- Subsequent runs will only show new messages

### Permission Denied Errors

**macOS/Linux:**
```bash
# Use sudo for init
sudo wechat-cli init

# Regular commands don't need sudo
wechat-cli sessions
```

**Windows:**
- Run terminal as Administrator for `init`
- Regular commands can run without elevation

## System Requirements

- **macOS** ≥ 10.15 (Catalina)
- **WeChat for Mac** ≤ 4.1.8.100
- **Python** ≥ 3.10 (for pip installation)
- **Node.js** (for npm installation)

**Platform Support:**

| Platform | Status | Notes |
|----------|--------|-------|
| macOS (Apple Silicon) | ✅ | Bundled arm64 binary |
| macOS (Intel) | ✅ | Supported via pip/source |
| Windows | ✅ | Reads Weixin.exe process memory |
| Linux | ✅ | Reads /proc/pid/mem, requires root |

## How It Works

1. **Key Extraction:** Scans WeChat process memory for SQLCipher encryption keys
2. **Decryption:** Transparent AES-256-CBC decryption with page-level caching
3. **Query:** Local SQLite queries on decrypted database
4. **Privacy:** All data stays local — no network access, no cloud transmission

**Read-only:** This tool only reads locally stored data, never sends or modifies messages.

## Quick Reference

```bash
# Setup
sudo wechat-cli init

# Common queries (JSON output)
wechat-cli sessions --limit 10
wechat-cli history "Alice" --limit 50
wechat-cli search "keyword" --chat "Group"
wechat-cli contacts --detail "Bob"
wechat-cli unread
wechat-cli new-messages

# Human-readable output
wechat-cli sessions --format text
wechat-cli history "Alice" --format text --limit 20
wechat-cli stats "Group" --format text

# Export & analysis
wechat-cli export "Chat" --format markdown --output file.md
wechat-cli stats "Group"
wechat-cli members "Group"
wechat-cli favorites --query "search term"

# Filters
--limit N --offset N
--start-time "YYYY-MM-DD" --end-time "YYYY-MM-DD"
--type text|image|voice|video|file|link|call|system
--format json|text
```
