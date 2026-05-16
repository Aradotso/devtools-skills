---
name: wecom-cli-enterprise-wechat
description: Enterprise WeChat (WeCom) CLI tool for managing documents, messages, contacts, tasks, meetings, and calendars from the terminal
triggers:
  - how do I use the Enterprise WeChat CLI
  - send a message through WeCom API
  - create a document in Enterprise WeChat
  - manage WeCom contacts from terminal
  - set up wecom-cli configuration
  - query Enterprise WeChat meeting list
  - create tasks in WeCom using CLI
  - read smart table data from Enterprise WeChat
---

# wecom-cli Enterprise WeChat Terminal Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`wecom-cli` is a command-line interface for Enterprise WeChat (WeCom/企业微信) that enables developers and AI agents to interact with WeCom's API directly from the terminal. It provides comprehensive coverage of core business capabilities including documents, smart tables, messages, contacts, tasks, meetings, and calendars.

**Key Features:**
- 📄 Document operations (create, read, edit, smart docs)
- 📊 Smart table management (create, fields, records CRUD)
- 💬 Message operations (query conversations, pull/send messages, download media)
- 👤 Contact directory (list visible members, search by name/alias)
- ✅ Task management (create, read, update, delete tasks)
- 🎥 Meeting management (schedule, cancel, update attendees)
- 📅 Calendar operations (CRUD events, manage participants, check availability)

**Use Cases:**
- **Enterprise (10+ users)**: Document and smart table automation for office efficiency
- **Personal/Small teams (≤10 users)**: Full access to messages, documents, calendar, meetings, and tasks

## Installation

### Prerequisites

- **Platform**: macOS (x64/arm64), Linux (x64/arm64), Windows (x64)
- **Node.js**: >= 18
- **Enterprise WeChat Account**
- **Bot Credentials**: Bot ID and Secret (obtain from [WeCom Open Platform](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21677))

### Install Steps

```bash
# Install the CLI globally
npm install -g @wecom/cli

# Install CLI Skill (required)
npx skills add WeComTeam/wecom-cli -y -g

# Interactive configuration (one-time setup)
wecom-cli init
```

The `init` command will prompt you for:
- Bot ID
- Bot Secret
- Other optional configuration parameters

## Configuration

Configuration is stored after running `wecom-cli init`. Credentials should be managed securely:

```bash
# Use environment variables for automation
export WECOM_BOT_ID="your_bot_id"
export WECOM_BOT_SECRET="your_bot_secret"
```

## Core Commands & Usage

### Contact Directory

**Get visible member list:**

```bash
# Get all visible contacts
wecom-cli contact get_userlist '{}'

# The JSON parameter can include filters
wecom-cli contact get_userlist '{"dept_id": "1"}'
```

**Search contacts by name/alias:**

```bash
# Search for a specific user
wecom-cli contact search '{"query": "张三"}'
```

### Messages

**Query conversation list:**

```bash
# Get list of conversations
wecom-cli message get_conversation_list '{}'
```

**Pull message history:**

```bash
# Pull messages from a conversation
wecom-cli message pull_messages '{
  "conversation_id": "conv_id_here",
  "limit": 50
}'
```

**Send text message:**

```bash
# Send a text message to a user
wecom-cli message send_text '{
  "userid": "user_id_here",
  "text": "Hello from CLI!"
}'
```

**Download media files:**

```bash
# Download image/file/voice/video
wecom-cli message download_media '{
  "media_id": "media_id_here",
  "media_type": "image",
  "output_path": "./downloads/image.jpg"
}'
```

### Documents

**Create a document:**

```bash
# Create a new document
wecom-cli document create '{
  "title": "Project Plan",
  "content": "# Project Overview\n\nThis is the project plan..."
}'
```

**Read document content:**

```bash
# Read a document by ID
wecom-cli document read '{
  "doc_id": "document_id_here"
}'
```

**Edit document:**

```bash
# Update document content
wecom-cli document edit '{
  "doc_id": "document_id_here",
  "content": "Updated content here"
}'
```

**Smart document operations:**

```bash
# Create a smart document
wecom-cli document create_smart '{
  "title": "Smart Analysis Report",
  "template_id": "template_id_here"
}'

# Read smart document
wecom-cli document read_smart '{
  "doc_id": "smart_doc_id_here"
}'
```

### Smart Tables

**Create smart table:**

```bash
# Create a new smart table
wecom-cli smartsheet create '{
  "name": "Customer Database",
  "description": "Track customer information"
}'
```

**Manage fields:**

```bash
# Add a field to smart table
wecom-cli smartsheet add_field '{
  "sheet_id": "sheet_id_here",
  "field_name": "Email",
  "field_type": "text"
}'
```

**CRUD records:**

```bash
# Add a record
wecom-cli smartsheet add_record '{
  "sheet_id": "sheet_id_here",
  "data": {
    "Name": "John Doe",
    "Email": "john@example.com"
  }
}'

# Query records
wecom-cli smartsheet get_records '{
  "sheet_id": "sheet_id_here",
  "filter": {}
}'

# Update a record
wecom-cli smartsheet update_record '{
  "sheet_id": "sheet_id_here",
  "record_id": "record_id_here",
  "data": {
    "Email": "newemail@example.com"
  }
}'

# Delete a record
wecom-cli smartsheet delete_record '{
  "sheet_id": "sheet_id_here",
  "record_id": "record_id_here"
}'
```

### Tasks (待办)

**Create a task:**

```bash
# Create a new task
wecom-cli task create '{
  "title": "Review quarterly report",
  "description": "Complete review by Friday",
  "assignee": "user_id_here",
  "due_date": "2026-05-20"
}'
```

**Read task:**

```bash
# Get task details
wecom-cli task read '{
  "task_id": "task_id_here"
}'
```

**Update task:**

```bash
# Update task status or content
wecom-cli task update '{
  "task_id": "task_id_here",
  "status": "completed"
}'
```

**Delete task:**

```bash
# Delete a task
wecom-cli task delete '{
  "task_id": "task_id_here"
}'
```

**Change user processing status:**

```bash
# Mark task as done for a user
wecom-cli task update_user_status '{
  "task_id": "task_id_here",
  "userid": "user_id_here",
  "status": "done"
}'
```

### Meetings

**Create scheduled meeting:**

```bash
# Schedule a new meeting
wecom-cli meeting create '{
  "title": "Weekly Sync",
  "start_time": "2026-05-20T14:00:00+08:00",
  "end_time": "2026-05-20T15:00:00+08:00",
  "invitees": ["user1", "user2", "user3"]
}'
```

**Query meeting list:**

```bash
# Get list of meetings
wecom-cli meeting list '{
  "start_date": "2026-05-01",
  "end_date": "2026-05-31"
}'
```

**Get meeting details:**

```bash
# Get specific meeting info
wecom-cli meeting get '{
  "meeting_id": "meeting_id_here"
}'
```

**Update meeting attendees:**

```bash
# Add or remove invitees
wecom-cli meeting update_invitees '{
  "meeting_id": "meeting_id_here",
  "add": ["user4"],
  "remove": ["user2"]
}'
```

**Cancel meeting:**

```bash
# Cancel a scheduled meeting
wecom-cli meeting cancel '{
  "meeting_id": "meeting_id_here"
}'
```

### Calendar

**Create calendar event:**

```bash
# Create a new calendar event
wecom-cli calendar create '{
  "summary": "Client Presentation",
  "start_time": "2026-05-21T10:00:00+08:00",
  "end_time": "2026-05-21T11:30:00+08:00",
  "location": "Conference Room A",
  "attendees": ["user1@company.com", "user2@company.com"]
}'
```

**Read calendar event:**

```bash
# Get event details
wecom-cli calendar read '{
  "event_id": "event_id_here"
}'
```

**Update calendar event:**

```bash
# Update event information
wecom-cli calendar update '{
  "event_id": "event_id_here",
  "summary": "Updated: Client Presentation",
  "location": "Conference Room B"
}'
```

**Delete calendar event:**

```bash
# Delete an event
wecom-cli calendar delete '{
  "event_id": "event_id_here"
}'
```

**Manage participants:**

```bash
# Add participants to event
wecom-cli calendar add_participants '{
  "event_id": "event_id_here",
  "attendees": ["user3@company.com"]
}'

# Remove participants
wecom-cli calendar remove_participants '{
  "event_id": "event_id_here",
  "attendees": ["user2@company.com"]
}'
```

**Check availability (free/busy):**

```bash
# Query availability for multiple users
wecom-cli calendar check_freebusy '{
  "userids": ["user1", "user2", "user3"],
  "start_time": "2026-05-20T09:00:00+08:00",
  "end_time": "2026-05-20T18:00:00+08:00"
}'
```

## Common Patterns

### Automation Script Example

```bash
#!/bin/bash

# Automation script for daily standup meeting creation
# Uses environment variables for credentials

export WECOM_BOT_ID="${WECOM_BOT_ID}"
export WECOM_BOT_SECRET="${WECOM_BOT_SECRET}"

# Get team members
TEAM_MEMBERS=$(wecom-cli contact get_userlist '{"dept_id": "2"}')

# Extract user IDs (requires jq)
USERIDS=$(echo "$TEAM_MEMBERS" | jq -r '.userlist[].userid' | tr '\n' ',' | sed 's/,$//')

# Create daily standup meeting
TOMORROW=$(date -v+1d '+%Y-%m-%dT09:30:00+08:00')
END_TIME=$(date -v+1d '+%Y-%m-%dT10:00:00+08:00')

wecom-cli meeting create "{
  \"title\": \"Daily Standup - $(date '+%Y-%m-%d')\",
  \"start_time\": \"$TOMORROW\",
  \"end_time\": \"$END_TIME\",
  \"invitees\": [$(echo "$USERIDS" | sed 's/,/","/g' | sed 's/^/"/' | sed 's/$/"/')]
}"

echo "✅ Daily standup meeting created"
```

### Task Management Workflow

```bash
#!/bin/bash

# Create tasks from a CSV file
# CSV format: title,description,assignee,due_date

while IFS=, read -r title description assignee due_date; do
  wecom-cli task create "{
    \"title\": \"$title\",
    \"description\": \"$description\",
    \"assignee\": \"$assignee\",
    \"due_date\": \"$due_date\"
  }"
  echo "Created task: $title"
done < tasks.csv
```

### Document Batch Processing

```bash
#!/bin/bash

# Create multiple project documents from templates

PROJECTS=("Project-Alpha" "Project-Beta" "Project-Gamma")

for project in "${PROJECTS[@]}"; do
  wecom-cli document create "{
    \"title\": \"$project - Requirements\",
    \"content\": \"# $project Requirements\\n\\n## Overview\\n\\nTBD\"
  }"
  
  wecom-cli document create "{
    \"title\": \"$project - Timeline\",
    \"content\": \"# $project Timeline\\n\\n## Milestones\\n\\nTBD\"
  }"
  
  echo "✅ Created documents for $project"
done
```

### Smart Table Data Import

```bash
#!/bin/bash

# Import JSON data into smart table

SHEET_ID="your_sheet_id_here"
DATA_FILE="customers.json"

# Read JSON array and create records
jq -c '.[]' "$DATA_FILE" | while read -r record; do
  wecom-cli smartsheet add_record "{
    \"sheet_id\": \"$SHEET_ID\",
    \"data\": $record
  }"
done

echo "✅ Data import completed"
```

## Troubleshooting

### Authentication Issues

**Problem**: `Authentication failed` error

**Solution**: Re-run configuration and verify credentials:

```bash
# Reinitialize configuration
wecom-cli init

# Verify credentials are set
echo $WECOM_BOT_ID
echo $WECOM_BOT_SECRET
```

### Permission Errors

**Problem**: `Permission denied` when accessing certain APIs

**Solution**: Verify your bot has the required scopes and permissions in the WeCom admin console. Different capabilities (documents, messages, etc.) require different permission grants.

### JSON Parsing Errors

**Problem**: `Invalid JSON parameter` error

**Solution**: Ensure JSON is properly escaped in shell commands:

```bash
# Correct: Use single quotes around JSON
wecom-cli task create '{"title": "My Task"}'

# Incorrect: Double quotes need escaping
wecom-cli task create "{\"title\": \"My Task\"}"
```

### Rate Limiting

**Problem**: `Rate limit exceeded` errors

**Solution**: Implement retry logic with exponential backoff:

```bash
#!/bin/bash

retry_command() {
  local max_attempts=3
  local attempt=1
  local delay=2
  
  while [ $attempt -le $max_attempts ]; do
    if $@; then
      return 0
    else
      echo "Attempt $attempt failed. Retrying in ${delay}s..."
      sleep $delay
      delay=$((delay * 2))
      attempt=$((attempt + 1))
    fi
  done
  
  return 1
}

# Usage
retry_command wecom-cli message send_text '{"userid": "user1", "text": "Hello"}'
```

### Debug Mode

Enable verbose logging for troubleshooting:

```bash
# Set debug environment variable
export WECOM_CLI_DEBUG=1

# Run command with debug output
wecom-cli contact get_userlist '{}'
```

## Best Practices

1. **Use environment variables** for credentials in CI/CD and automation scripts
2. **Validate JSON** before passing to commands (use `jq` or similar tools)
3. **Handle errors gracefully** in scripts with proper exit codes
4. **Rate limit your requests** when performing batch operations
5. **Store media files** with organized naming conventions when downloading
6. **Version control** your automation scripts alongside your project
7. **Test with small datasets** before running large batch operations

## Additional Resources

- [Official CLI Command Reference](https://github.com/WeComTeam/wecom-cli/blob/main/docs/cli-reference.md)
- [Skills Documentation](https://github.com/WeComTeam/wecom-cli/blob/main/docs/skills.md)
- [WeCom Open Platform Documentation](https://open.work.weixin.qq.com/)
- [Bot Credential Setup Guide](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21677)
