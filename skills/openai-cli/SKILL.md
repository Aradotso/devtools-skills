---
name: openai-cli
description: Official OpenAI CLI for interacting with the OpenAI REST API from the command line
triggers:
  - "use the OpenAI CLI"
  - "call OpenAI API from terminal"
  - "make OpenAI requests with CLI"
  - "use openai command line tool"
  - "interact with OpenAI from shell"
  - "openai cli commands"
  - "call gpt from command line"
  - "query OpenAI API in terminal"
---

# OpenAI CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

The official CLI for the OpenAI REST API. Provides a command-line interface to interact with OpenAI's API endpoints including chat completions, embeddings, fine-tuning, and admin operations.

## Installation

### Homebrew (macOS/Linux)

```bash
brew install openai/tools/openai
```

### Go Install

```bash
go install 'github.com/openai/openai-cli/cmd/openai@latest'
```

Ensure Go bin is in your PATH:

```bash
export PATH="$PATH:$(go env GOPATH)/bin"
```

### Local Development

Clone the repository and run locally:

```bash
git clone https://github.com/openai/openai-cli.git
cd openai-cli
./scripts/run args...
```

## Configuration

Set up your API credentials using environment variables:

```bash
export OPENAI_API_KEY="sk-..."
export OPENAI_ADMIN_KEY="sk-admin-..."  # For admin endpoints
export OPENAI_ORG_ID="org-..."          # Optional
export OPENAI_PROJECT_ID="proj_..."     # Optional
```

Or use flags on each command:

```bash
openai --api-key sk-... responses create --input "Hello"
```

### Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `OPENAI_API_KEY` | Standard API authentication | For standard endpoints |
| `OPENAI_ADMIN_KEY` | Admin API authentication | For admin endpoints |
| `OPENAI_ORG_ID` | Organization identifier | No |
| `OPENAI_PROJECT_ID` | Project identifier | No |
| `OPENAI_WEBHOOK_SECRET` | Webhook verification | No |

## Command Structure

```bash
openai [resource] <command> [flags...]
```

## Key Commands

### Chat Completions

Create a chat completion:

```bash
openai responses create \
  --input "Explain quantum computing in simple terms" \
  --model gpt-4
```

Stream responses:

```bash
openai responses create \
  --input "Write a poem about coding" \
  --model gpt-4 \
  --stream
```

With system message and temperature:

```bash
openai responses create \
  --input "What is recursion?" \
  --model gpt-4 \
  --system "You are a computer science teacher" \
  --temperature 0.7
```

### File Operations

Upload a file:

```bash
openai files create \
  --file @training_data.jsonl \
  --purpose fine-tune
```

List files:

```bash
openai files list
```

Retrieve file content:

```bash
openai files retrieve file-abc123
```

Delete a file:

```bash
openai files delete file-abc123
```

### Embeddings

Generate embeddings:

```bash
openai embeddings create \
  --input "The quick brown fox" \
  --model text-embedding-3-small
```

Multiple inputs:

```bash
openai embeddings create \
  --input "First text" \
  --input "Second text" \
  --model text-embedding-3-large
```

### Fine-tuning

Create a fine-tuning job:

```bash
openai fine-tuning jobs create \
  --training-file file-abc123 \
  --model gpt-3.5-turbo
```

List fine-tuning jobs:

```bash
openai fine-tuning jobs list
```

Check job status:

```bash
openai fine-tuning jobs retrieve ftjob-abc123
```

Cancel a job:

```bash
openai fine-tuning jobs cancel ftjob-abc123
```

### Admin Operations

Usage statistics:

```bash
openai admin:organization:usage completions \
  --start-time 1735689600 \
  --end-time 1735776000 \
  --bucket-width 1d
```

## File Handling

### Passing Files as Arguments

Use `@` prefix for file paths:

```bash
openai responses create --arg @input.txt
```

Within JSON/YAML:

```bash
openai <command> --arg '{image: "@image.jpg"}'
```

```bash
openai <command> <<YAML
arg:
  image: "@abe.jpg"
YAML
```

### Explicit Encoding

Force string encoding:

```bash
openai <command> --arg @file://myfile.txt
```

Force base64 encoding:

```bash
openai <command> --arg @data://image.png
```

Absolute paths:

```bash
openai <command> --arg @file:///tmp/file.txt
```

### Escaping @ Signs

To pass a literal `@` character:

```bash
openai <command> --username '\@username'
```

## Output Formatting

### Format Options

Control output format with `--format`:

```bash
# Pretty-printed (default for TTY)
openai responses create --input "Hello" --format pretty

# Raw JSON
openai responses create --input "Hello" --format json

# JSONL (one JSON object per line)
openai responses create --input "Hello" --format jsonl

# YAML
openai responses create --input "Hello" --format yaml

# Raw response body
openai responses create --input "Hello" --format raw
```

### Data Transformation

Use GJSON syntax to extract specific fields:

```bash
# Get just the message content
openai responses create \
  --input "Hello" \
  --transform "choices.0.message.content"

# Get multiple fields
openai responses create \
  --input "Hello" \
  --transform "{id: id, content: choices.0.message.content}"
```

Transform errors:

```bash
openai responses create \
  --input "Hello" \
  --format-error json \
  --transform-error "error.message"
```

## Global Flags

```bash
--api-key           # API key (or use OPENAI_API_KEY)
--admin-api-key     # Admin key (or use OPENAI_ADMIN_KEY)
--organization      # Org ID (or use OPENAI_ORG_ID)
--project           # Project ID (or use OPENAI_PROJECT_ID)
--base-url          # Custom API endpoint
--debug             # Enable debug logging with HTTP details
--version, -v       # Show CLI version
--help              # Show help for command
--format            # Output format (auto, explore, json, jsonl, pretty, raw, yaml)
--format-error      # Error output format
--transform         # GJSON transform for data output
--transform-error   # GJSON transform for error output
```

## Common Patterns

### Interactive Chat Session

```bash
while true; do
  read -p "You: " input
  openai responses create \
    --input "$input" \
    --model gpt-4 \
    --format pretty
done
```

### Batch Processing

Process multiple inputs from a file:

```bash
while IFS= read -r line; do
  openai responses create \
    --input "$line" \
    --model gpt-3.5-turbo \
    --format json >> results.jsonl
done < inputs.txt
```

### JSON Input

Pass structured data:

```bash
openai responses create \
  --input "Translate to French" \
  --model gpt-4 \
  --max-tokens 100 \
  --temperature 0.3
```

### Piping Data

```bash
echo "Summarize this text" | openai responses create \
  --input "$(cat)" \
  --model gpt-4
```

Or:

```bash
cat document.txt | openai responses create \
  --input "Summarize: $(cat -)" \
  --model gpt-4
```

### Image Analysis

```bash
openai responses create \
  --input '{text: "What is in this image?", image: "@photo.jpg"}' \
  --model gpt-4-vision-preview
```

### Custom Base URL

For Azure or custom endpoints:

```bash
openai --base-url https://custom.openai.azure.com/v1 \
  responses create \
  --input "Hello" \
  --model gpt-4
```

## Debugging

Enable debug mode to see full HTTP requests/responses:

```bash
openai --debug responses create --input "Test"
```

**Warning**: Debug logs include request/response bodies which may contain sensitive data.

## Troubleshooting

### Command Not Found

Ensure Go bin is in your PATH:

```bash
echo $PATH | grep "$(go env GOPATH)/bin"
# If not found:
export PATH="$PATH:$(go env GOPATH)/bin"
```

### Authentication Errors

Verify API key is set:

```bash
echo $OPENAI_API_KEY
# Should output: sk-...
```

Check key validity:

```bash
openai responses create --input "test" --model gpt-3.5-turbo
```

### File Not Found Errors

Use absolute paths or verify relative paths:

```bash
# Absolute path
openai files create --file @/full/path/to/file.jsonl --purpose fine-tune

# Verify file exists
ls -la @file.jsonl
```

### Rate Limiting

Add delays between requests:

```bash
for i in {1..10}; do
  openai responses create --input "Request $i"
  sleep 1
done
```

### Model Not Available

List available models (if endpoint exists):

```bash
openai models list
```

Check model name spelling and access permissions.

### Output Not Formatted

Force format explicitly:

```bash
openai responses create --input "Hello" --format json
```

Check if output is being piped (auto-detection may choose raw format):

```bash
openai responses create --input "Hello" --format pretty | less
```

## Linking Custom SDK Versions

For development, link against different Go SDK versions:

```bash
# Link to specific version
./scripts/link github.com/openai/openai-go@v1.2.3

# Link to local SDK copy
./scripts/link ../openai-go

# Default (../openai-go)
./scripts/link
```

## Version Information

Check CLI version:

```bash
openai --version
# or
openai -v
```

Get help:

```bash
openai --help
openai responses --help
openai responses create --help
```
