---
name: dingtalk-workspace-cli
description: DingTalk Workspace CLI (dws) — cross-platform tool for managing DingTalk enterprise data (contacts, calendars, docs, todos, AI tables, chat) via command line and AI agents
triggers:
  - search DingTalk contacts or users
  - create a DingTalk calendar event or meeting
  - query DingTalk AI table records
  - search DingTalk documents
  - create or list DingTalk todos
  - send a DingTalk message
  - manage DingTalk drive files
  - get DingTalk attendance records
---

# DingTalk Workspace CLI (dws)

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

DingTalk Workspace CLI (`dws`) is an officially open-sourced cross-platform CLI tool from DingTalk that unifies DingTalk's full suite of product capabilities (contacts, calendars, documents, todos, AI tables, chat, drive, attendance, reports, meetings) into a single package. It's designed for both human users and AI agent scenarios, with structured JSON responses, OAuth device-flow authentication, and enterprise-grade security.

## Installation

**macOS / Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/DingTalk-Real-AI/dingtalk-workspace-cli/main/scripts/install.sh | sh
```

**Windows (PowerShell):**
```powershell
irm https://raw.githubusercontent.com/DingTalk-Real-AI/dingtalk-workspace-cli/main/scripts/install.ps1 | iex
```

**npm (requires Node.js):**
```bash
npm install -g dingtalk-workspace-cli
```

**Build from source (Go 1.25+):**
```bash
git clone https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli.git
cd dingtalk-workspace-cli
go build -o dws ./cmd
cp dws ~/.local/bin/
```

**Upgrade (requires v1.0.7+):**
```bash
dws upgrade                    # interactive upgrade
dws upgrade --check            # check without installing
dws upgrade --version v1.0.7   # specific version
dws upgrade --rollback         # rollback
```

## Authentication

**Interactive login (browser):**
```bash
dws auth login
```

**Device flow (headless environments):**
```bash
dws auth login --device
```

**Custom app (CI/CD, ISV integration):**
```bash
dws auth login --client-id $DINGTALK_CLIENT_ID --client-secret $DINGTALK_CLIENT_SECRET
```

**Check auth status:**
```bash
dws auth status
```

**Logout:**
```bash
dws auth logout
```

## Core Architecture

### Product Services

DingTalk Workspace is organized into product services:

| Service | Commands | Use Cases |
|---------|----------|-----------|
| `contact` | user, dept, org | Search users, list departments, get org info |
| `calendar` | event | Create/list/update/delete calendar events |
| `aitable` | space, base, table, record, field, view | Query/create AI table records, manage schema |
| `doc` | search, create | Search DingTalk Docs, create documents |
| `todo` | task, category | Create/list/update todos, manage categories |
| `chat` | send, list | Send messages, list conversations |
| `drive` | list, upload, download | Manage DingTalk drive files |
| `attendance` | record, shift | Get attendance records, query shifts |
| `report` | list | List reports (inbox/sent/created) |
| `minutes` | list, detail | Get AI meeting minutes |
| `im` | group, message | Manage IM groups, send messages |

### Global Flags

```bash
# Output formats
-f, --format table|json|raw    # table (human), json (structured), raw (API response)

# JQ filtering (extract specific fields)
--jq '.result[0].name'         # save tokens, extract precisely

# Dry run (preview without executing)
--dry-run                      # see request payload before sending

# Skip confirmation (required for AI agents)
--yes, -y                      # auto-confirm destructive operations

# Debug
--debug                        # verbose logging
```

## Key Commands & Patterns

### Contact Management

**Search users:**
```bash
# Basic search
dws contact user search --query "engineering"

# With filtering and formatting
dws contact user search --query "zhang" -f json --jq '.result[] | {name: .name, userId: .userId, mobile: .mobile}'

# Dry run
dws contact user search --query "zhang" --dry-run
```

**Get current user:**
```bash
dws contact user get-self -f json --jq '.result[0].orgEmployeeModel | {name: .orgUserName, userId: .userId, depts: [.depts[].deptName]}'
```

**Get user by ID:**
```bash
dws contact user get --user-id "USER_ID"
```

**List departments:**
```bash
# Root departments
dws contact dept list

# Subdepartments
dws contact dept list --dept-id "DEPT_ID"

# Extract specific fields
dws contact dept list --jq '.result[] | {id: .deptId, name: .name}'
```

**List department members:**
```bash
dws contact dept members --dept-id "DEPT_ID" -f json
```

### Calendar Events

**List events:**
```bash
# Today's events
dws calendar event list

# Date range
dws calendar event list --start-time "2026-05-20T00:00:00+08:00" --end-time "2026-05-21T00:00:00+08:00"

# Extract fields
dws calendar event list --jq '.result[] | {title: .summary, start: .start.dateTime, attendees: [.attendees[].displayName]}'
```

**Create event:**
```bash
# Basic event
dws calendar event create \
  --summary "Team Sync" \
  --start-time "2026-05-20T14:00:00+08:00" \
  --end-time "2026-05-20T15:00:00+08:00" \
  --yes

# With attendees
dws calendar event create \
  --summary "Quarterly Review" \
  --start-time "2026-05-25T10:00:00+08:00" \
  --end-time "2026-05-25T11:30:00+08:00" \
  --attendees "USER_ID_1,USER_ID_2" \
  --location "Meeting Room A" \
  --description "Q2 business review" \
  --yes
```

**Update event:**
```bash
dws calendar event update \
  --event-id "EVENT_ID" \
  --summary "Updated Title" \
  --start-time "2026-05-20T15:00:00+08:00" \
  --end-time "2026-05-20T16:00:00+08:00" \
  --yes
```

**Delete event:**
```bash
dws calendar event delete --event-id "EVENT_ID" --yes
```

### AI Table (AITable)

**List spaces:**
```bash
dws aitable space list -f json --jq '.result[] | {id: .spaceId, name: .name}'
```

**List bases in space:**
```bash
dws aitable base list --space-id "SPACE_ID"
```

**List tables in base:**
```bash
dws aitable table list --base-id "BASE_ID"
```

**Query records:**
```bash
# All records
dws aitable record query --base-id "BASE_ID" --table-id "TABLE_ID"

# With filters (filterByFormula)
dws aitable record query \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --filter-by-formula '{Status}="Done"' \
  --limit 10

# Sort and paginate
dws aitable record query \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --sort '[{"field":"CreatedTime","order":"desc"}]' \
  --page-size 20

# Extract specific fields
dws aitable record query \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --jq '.result.records[] | {id: .recordId, fields: .fields}'
```

**Create records:**
```bash
# Single record
dws aitable record create \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --records '[{"fields":{"Name":"Test Item","Status":"In Progress","Priority":1}}]' \
  --yes

# Multiple records
dws aitable record create \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --records '[
    {"fields":{"Name":"Item 1","Status":"Todo"}},
    {"fields":{"Name":"Item 2","Status":"Done"}}
  ]' \
  --yes
```

**Update records:**
```bash
dws aitable record update \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --records '[{"recordId":"RECORD_ID","fields":{"Status":"Done"}}]' \
  --yes
```

**Delete records:**
```bash
dws aitable record delete \
  --base-id "BASE_ID" \
  --table-id "TABLE_ID" \
  --record-ids "RECORD_ID_1,RECORD_ID_2" \
  --yes
```

**List fields (schema):**
```bash
dws aitable field list --base-id "BASE_ID" --table-id "TABLE_ID"
```

### Todo Management

**List todos:**
```bash
# All incomplete todos
dws todo task list

# With filters
dws todo task list --is-done false --jq '.result[] | {id: .taskId, title: .subject, due: .dueTime}'

# Specific category
dws todo task list --category-id "CATEGORY_ID"
```

**Create todo:**
```bash
dws todo task create \
  --title "Review PR #123" \
  --executors "USER_ID" \
  --due-time "2026-05-22T18:00:00+08:00" \
  --priority 10 \
  --yes

# With description and multiple executors
dws todo task create \
  --title "Quarterly Report" \
  --description "Complete Q2 summary" \
  --executors "USER_ID_1,USER_ID_2" \
  --due-time "2026-05-30T23:59:59+08:00" \
  --yes
```

**Update todo:**
```bash
dws todo task update \
  --task-id "TASK_ID" \
  --is-done true \
  --yes

# Update fields
dws todo task update \
  --task-id "TASK_ID" \
  --title "Updated Title" \
  --priority 20 \
  --yes
```

**Delete todo:**
```bash
dws todo task delete --task-id "TASK_ID" --yes
```

**List categories:**
```bash
dws todo category list -f json
```

### Document Search

**Search documents:**
```bash
# Basic search
dws doc search --query "quarterly report"

# With type filter
dws doc search --query "roadmap" --doc-type "doc" --jq '.result[] | {title: .title, url: .url, author: .creator.name}'

# Pagination
dws doc search --query "API" --max-results 50
```

### Chat & IM

**Send message:**
```bash
# To user
dws chat send \
  --receiver-id "USER_ID" \
  --msg-type "text" \
  --content "Hello from dws!" \
  --yes

# To group
dws chat send \
  --receiver-id "GROUP_ID" \
  --msg-type "text" \
  --content "Team update" \
  --yes

# Markdown message
dws chat send \
  --receiver-id "USER_ID" \
  --msg-type "markdown" \
  --content "**Important**: Please review [this doc](https://example.com)" \
  --yes
```

**List conversations:**
```bash
dws chat list -f json --jq '.result[] | {id: .openConversationId, title: .title, type: .conversationType}'
```

**List IM groups:**
```bash
dws im group list --jq '.result[] | {id: .openConversationId, name: .name, memberCount: .memberCount}'
```

### Drive

**List files:**
```bash
# Root directory
dws drive list

# Specific space
dws drive list --space-id "SPACE_ID"

# Extract fields
dws drive list --jq '.result[] | {name: .name, id: .fileId, size: .size, type: .type}'
```

**Upload file:**
```bash
dws drive upload \
  --space-id "SPACE_ID" \
  --parent-id "PARENT_ID" \
  --file-path "/path/to/file.pdf" \
  --yes
```

**Download file:**
```bash
dws drive download \
  --space-id "SPACE_ID" \
  --file-id "FILE_ID" \
  --output-path "/path/to/save/file.pdf"
```

### Attendance

**Get attendance records:**
```bash
# Today's records
dws attendance record get

# Specific date range
dws attendance record get \
  --start-date "2026-05-01" \
  --end-date "2026-05-07" \
  -f json --jq '.result[] | {date: .workDate, checkIn: .checkInTime, checkOut: .checkOutTime}'
```

**Query shift schedules:**
```bash
dws attendance shift query \
  --user-ids "USER_ID" \
  --start-date "2026-05-20" \
  --end-date "2026-05-27"
```

### Reports

**List reports:**
```bash
# Inbox (received)
dws report list inbox --start-time 1715990400000 --end-time 1716076799000

# Sent
dws report list sent --start-time 1715990400000 --end-time 1716076799000

# Created by me
dws report list mine --start-time 1715990400000 --end-time 1716076799000
```

### AI Meeting Minutes

**List minutes:**
```bash
# All minutes
dws minutes list

# Mine
dws minutes list mine

# Extract fields
dws minutes list --jq '.result[] | {id: .minutesId, title: .title, date: .meetingStartTime}'
```

**Get minutes detail:**
```bash
dws minutes detail --minutes-id "MINUTES_ID" -f json
```

## Schema Discovery (AI Agents)

AI agents don't need pre-built knowledge. Use `dws schema` to dynamically discover capabilities:

**List all products and tool counts:**
```bash
dws schema --jq '.products[] | {id, tool_count: (.tools | length)}'
```

**Inspect specific tool parameters:**
```bash
# Get parameter schema for aitable.query_records
dws schema aitable.query_records --jq '.tool.parameters'

# Get all aitable tools
dws schema --jq '.products[] | select(.id=="aitable") | .tools[] | {name: .name, description: .description}'
```

**Example discovery workflow:**
```bash
# Step 1: Find the right product
dws schema --jq '.products[] | {id, description}'

# Step 2: List tools in that product
dws schema --jq '.products[] | select(.id=="calendar") | .tools[] | .name'

# Step 3: Get parameter schema
dws schema calendar.create_event --jq '.tool.parameters.properties | keys'

# Step 4: Execute
dws calendar event create --summary "Team Sync" --start-time "2026-05-20T14:00:00+08:00" --end-time "2026-05-20T15:00:00+08:00" --yes
```

## Configuration

**Config file location:**
- macOS/Linux: `~/.config/dws/config.yaml`
- Windows: `%APPDATA%\dws\config.yaml`

**Example config.yaml:**
```yaml
auth:
  client_id: "your-app-key"
  client_secret: "your-app-secret"
output:
  default_format: "json"
  color: true
debug: false
```

**Override via environment variables:**
```bash
export DWS_OUTPUT_FORMAT=json
export DWS_DEBUG=true
export DINGTALK_CLIENT_ID=your-app-key
export DINGTALK_CLIENT_SECRET=your-app-secret
```

## AI Agent Best Practices

### Always Use --yes for Non-Interactive Execution

```bash
# ❌ Bad (will hang waiting for confirmation)
dws todo task create --title "Review PR" --executors "USER_ID"

# ✅ Good
dws todo task create --title "Review PR" --executors "USER_ID" --yes
```

### Use --dry-run for Safety

```bash
# Preview before executing
dws contact user search --query "engineering" --dry-run
dws aitable record delete --base-id "BASE_ID" --table-id "TABLE_ID" --record-ids "REC_ID" --dry-run
```

### Use --jq to Save Tokens

```bash
# ❌ Returns entire API response (1000+ tokens)
dws contact user search --query "zhang" -f json

# ✅ Extract only needed fields (50 tokens)
dws contact user search --query "zhang" -f json --jq '.result[] | {name: .name, userId: .userId}'
```

### Handle Pagination

```bash
# AITable pagination
dws aitable record query --base-id "BASE_ID" --table-id "TABLE_ID" --page-size 100

# Calendar pagination (date ranges)
dws calendar event list --start-time "2026-05-01T00:00:00+08:00" --end-time "2026-06-01T00:00:00+08:00"
```

### Error Handling

```bash
# Check exit code
dws contact user search --query "nonexistent"
echo $?  # 0 = success, non-zero = error

# Capture stderr
dws todo task create --title "Test" --executors "INVALID_ID" --yes 2>&1 | grep "error"
```

## Common Patterns

### Multi-Step Workflows

**Schedule meeting with available room:**
```bash
# 1. Search attendees
ATTENDEES=$(dws contact user search --query "engineering" -f json --jq '[.result[].userId] | join(",")')

# 2. Create event
EVENT_ID=$(dws calendar event create \
  --summary "Team Sync" \
  --start-time "2026-05-20T14:00:00+08:00" \
  --end-time "2026-05-20T15:00:00+08:00" \
  --attendees "$ATTENDEES" \
  --yes \
  -f json --jq '.result.eventId')

echo "Created event: $EVENT_ID"
```

**Batch import to AITable from JSON:**
```bash
# Prepare records.json
cat > records.json <<EOF
[
  {"fields": {"Name": "Item 1", "Status": "Todo", "Priority": 1}},
  {"fields": {"Name": "Item 2", "Status": "In Progress", "Priority": 2}}
]
EOF

# Import
dws aitable record create \
  --base-id "$BASE_ID" \
  --table-id "$TABLE_ID" \
  --records "$(cat records.json)" \
  --yes
```

**Find overdue todos:**
```bash
dws todo task list --is-done false -f json | \
  jq --arg now "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
    '.result[] | select(.dueTime != null and .dueTime < $now) | {id: .taskId, title: .subject, due: .dueTime}'
```

### Batch Operations

**Delete multiple AITable records:**
```bash
# Get record IDs to delete
RECORD_IDS=$(dws aitable record query \
  --base-id "$BASE_ID" \
  --table-id "$TABLE_ID" \
  --filter-by-formula '{Status}="Archived"' \
  -f json --jq '[.result.records[].recordId] | join(",")')

# Delete
dws aitable record delete \
  --base-id "$BASE_ID" \
  --table-id "$TABLE_ID" \
  --record-ids "$RECORD_IDS" \
  --yes
```

**Send notification to department:**
```bash
# Get department members
USER_IDS=$(dws contact dept members --dept-id "$DEPT_ID" -f json --jq '[.result[].userId]')

# Send to each
echo "$USER_IDS" | jq -r '.[]' | while read user_id; do
  dws chat send --receiver-id "$user_id" --msg-type "text" --content "Reminder: Submit timesheet" --yes
done
```

## Troubleshooting

### Authentication Issues

**Token expired:**
```bash
# Re-login
dws auth login

# Or use device flow
dws auth login --device
```

**Organization not enabled:**
- Join the DingTalk co-creation group: scan QR in README
- Ask admin to enable CLI access in Developer Platform

### Permission Errors

```bash
# Check current auth status and scopes
dws auth status -f json --jq '.result | {userId: .userId, corpId: .corpId, scopes: .scopes}'

# Request required scopes during login
dws auth login --scope "Contact.User.Read,Calendar.Event.Write"
```

### Rate Limiting

```bash
# Add delays between bulk operations
for i in {1..100}; do
  dws todo task create --title "Task $i" --executors "USER_ID" --yes
  sleep 0.5
done
```

### Debug Mode

```bash
# Enable verbose logging
dws --debug contact user search --query "test"

# Check request/response
dws contact user search --query "test" --dry-run
```

### Invalid Timestamps

DingTalk expects RFC3339 format with timezone:

```bash
# ✅ Correct
--start-time "2026-05-20T14:00:00+08:00"

# ❌ Incorrect
--start-time "2026-05-20 14:00:00"
--start-time "1716192000"  # Unix timestamp not supported in event create
```

For attendance, use date format:
```bash
# ✅ Correct
--start-date "2026-05-20"

# ❌ Incorrect
--start-date "2026-05-20T00:00:00+08:00"
```

### AITable filterByFormula Syntax

```bash
# ✅ Correct (JSON string escaping)
--filter-by-formula '{Status}="Done"'
--filter-by-formula 'AND({Priority}>5, {Status}="In Progress")'

# ❌ Incorrect (unescaped quotes)
--filter-by-formula {"Status":"Done"}
```

### Empty Results

```bash
# Check if data exists
dws aitable record query --base-id "$BASE_ID" --table-id "$TABLE_ID" --limit 1

# Verify IDs are correct
dws aitable space list
dws aitable base list --space-id "$SPACE_ID"
dws aitable table list --base-id "$BASE_ID"
```

## Reference

- **Command Index**: [`docs/command-index.md`](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli/blob/main/docs/command-index.md) — all commands with descriptions
- **Full Reference**: [`docs/reference.md`](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli/blob/main/docs/reference.md) — complete command reference
- **Changelog**: [`CHANGELOG.md`](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli/blob/main/CHANGELOG.md)
- **Ready-made Scripts**: [`skills/scripts/`](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli/tree/main/skills/scripts) — 13 Python scripts for batch operations
- **GitHub Releases**: [https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli/releases](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli/releases)

## Security Notes

- Credentials stored in system keychain (macOS Keychain, Windows Credential Manager, Linux Secret Service)
- OAuth device-flow for headless environments
- Zero-trust: every request requires authentication
- Domain allowlisting and least-privilege scoping enforced server-side
- Use environment variables for CI/CD: `$DINGTALK_CLIENT_ID`, `$DINGTALK_CLIENT_SECRET`

## Example: Complete Workflow (AI Agent)

```bash
#!/bin/bash
# AI agent script: Create weekly team sync event with engineering team

set -e

# 1. Get current user
MY_USER_ID=$(dws contact user get-self -f json --jq '.result[0].orgEmployeeModel.userId' | tr -d '"')

# 2. Search engineering team
ATTENDEES=$(dws contact user search --query "engineering" -f json --jq '[.result[].userId] | join(",")')

# 3. Find next Monday 2pm
NEXT_MONDAY=$(date -d "next monday 14:00" -Iseconds --utc | sed 's/+00:00/+08:00/')
END_TIME=$(date -d "next monday 15:00" -Iseconds --utc | sed 's/+00:00/+08:00/')

# 4. Create event
EVENT_ID=$(dws calendar event create \
  --summary "Weekly Team Sync" \
  --start-time "$NEXT_MONDAY" \
  --end-time "$END_TIME" \
  --attendees "$ATTENDEES" \
  --location "Meeting Room A" \
  --description "Weekly engineering team sync" \
  --yes \
  -f json --jq '.result.eventId' | tr -d '"')

echo "✅ Created event $EVENT_ID for $NEXT_MONDAY"

# 5. Create reminder todo
dws todo task create \
  --title "Prepare agenda for team sync" \
  --executors "$MY_USER_ID" \
  --due-time "$(date -d "next monday 12:00" -Iseconds --utc | sed 's/+00:00/+08:00/')" \
  --description "Event ID: $EVENT_ID" \
  --yes

echo "✅ Created reminder todo"
```
