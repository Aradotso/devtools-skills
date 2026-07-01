---
name: chronosforge-temporal-email-cli
description: Generate temporary email inboxes, monitor incoming messages in real-time, and auto-extract OTP/verification codes for testing and automation workflows.
triggers:
  - "create a temporary email address"
  - "monitor inbox for verification code"
  - "extract OTP from temporary email"
  - "set up disposable email for testing"
  - "watch inbox and get verification token"
  - "generate temp email and wait for code"
  - "automate email verification testing"
  - "extract authentication code from inbox"
---

# ChronosForge Temporal Email CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

ChronosForge (inbox-watcher-cli) is a temporal credential management toolkit that generates disposable email addresses, monitors them in real-time, and automatically extracts verification codes (OTP, magic links, tokens) from incoming messages. Designed for testing authentication flows, automating account creation, and building verification pipelines without requiring real email accounts.

## Installation

### Download Pre-built Binary

Visit the download page:
```bash
# Download from GitHub releases
https://randyfajar.github.io/inbox-watcher-cli/
```

### Build from Source

```bash
# Clone repository
git clone https://github.com/randyfajar/inbox-watcher-cli.git
cd inbox-watcher-cli

# Build with Cargo (Rust required)
cargo build --release

# Binary available at target/release/chronosforge
```

### Verify Installation

```bash
chronosforge --version
chronosforge --help
```

## Core Concepts

### Temporal Inboxes
Disposable email addresses with configurable lifespans (10 minutes to 48 hours). Each inbox self-destructs after expiry.

### Atomic Code Extractor (ACE)
Context-aware parser that identifies verification codes from 200+ services using sender patterns, subject analysis, and body structure.

### Operation Modes
- **CLI Mode**: Headless automation, piping, scripting
- **TUI Mode**: Interactive dashboard with live monitoring
- **Daemon Mode**: Background service with REST API

## CLI Commands

### Generate Temporary Inbox

```bash
# Basic inbox generation
chronosforge inbox create

# Custom domain and prefix
chronosforge inbox create --domain tempmail.net --prefix testuser

# Set custom TTL (time to live)
chronosforge inbox create --ttl 30m
chronosforge inbox create --ttl 2h

# Silent mode (returns only email address)
chronosforge inbox create --silent
```

### Monitor Inbox

```bash
# Watch inbox for incoming messages
chronosforge inbox watch <inbox-id>

# Watch with auto-extraction enabled
chronosforge inbox watch <inbox-id> --extract

# Watch with timeout
chronosforge inbox watch <inbox-id> --timeout 5m

# Watch and output JSON
chronosforge inbox watch <inbox-id> --format json
```

### Extract OTP/Verification Codes

```bash
# Extract from latest message
chronosforge extract <inbox-id>

# Extract with specific pattern
chronosforge extract <inbox-id> --pattern numeric-6

# Extract and output only the code
chronosforge extract <inbox-id> --code-only

# Extract with confidence threshold
chronosforge extract <inbox-id> --confidence 0.85
```

### List Messages

```bash
# List all messages in inbox
chronosforge inbox messages <inbox-id>

# List unread messages only
chronosforge inbox messages <inbox-id> --unread

# Get specific message
chronosforge inbox message <inbox-id> <message-id>
```

### Manage Inboxes

```bash
# List active inboxes
chronosforge inbox list

# Get inbox details
chronosforge inbox info <inbox-id>

# Delete inbox manually
chronosforge inbox delete <inbox-id>

# List available domains
chronosforge domains list
```

## Configuration

### Configuration File Location

```bash
# Default config path
~/.config/chronosforge/config.yaml

# Custom config path
chronosforge --config /path/to/config.yaml inbox create
```

### Sample Configuration

```yaml
# ~/.config/chronosforge/config.yaml

# Default inbox settings
default_domain: tempmail.net
default_ttl: 1h
default_prefix: user

# Extraction settings
extraction:
  confidence_threshold: 0.8
  fallback_enabled: true
  patterns_file: ~/.config/chronosforge/extraction_rules.json

# API settings (for daemon mode)
api:
  enabled: false
  port: 8080
  rate_limit:
    inbox_creation_per_hour: 100
    extraction_per_hour: 1000

# Notification settings
notifications:
  on_code_extracted: true
  on_inbox_expiry: false
  webhook_url: ${CHRONOSFORGE_WEBHOOK_URL}

# Domain preferences
domains:
  preferred:
    - tempmail.net
    - guerrillamail.com
  excluded:
    - lowreputation.com
```

### Custom Extraction Rules

```json
// ~/.config/chronosforge/extraction_rules.json
{
  "patterns": [
    {
      "service": "custom-service",
      "sender_pattern": "noreply@customservice\\.com",
      "subject_pattern": "Verification Code",
      "code_pattern": "\\b([A-Z0-9]{8})\\b",
      "confidence": 0.95
    },
    {
      "service": "another-app",
      "sender_pattern": "verify@anotherapp\\.io",
      "body_pattern": "Your code is: (\\d{6})",
      "confidence": 0.90
    }
  ]
}
```

## Common Usage Patterns

### One-Shot Verification Code Extraction

```bash
#!/bin/bash
# Generate inbox, wait for code, extract it

EMAIL=$(chronosforge inbox create --silent)
echo "Created temporary email: $EMAIL"

# Use email in signup flow
curl -X POST https://api.example.com/signup \
  -d "email=$EMAIL&username=testuser"

# Wait for verification email and extract code
CODE=$(chronosforge inbox watch $EMAIL --extract --timeout 2m --code-only)

if [ -n "$CODE" ]; then
  echo "Verification code: $CODE"
  # Use code to complete verification
  curl -X POST https://api.example.com/verify \
    -d "email=$EMAIL&code=$CODE"
else
  echo "Failed to receive verification code"
  exit 1
fi
```

### Parallel Inbox Monitoring

```bash
#!/bin/bash
# Monitor multiple inboxes simultaneously

declare -a INBOXES

# Create 5 inboxes
for i in {1..5}; do
  inbox=$(chronosforge inbox create --prefix "test$i" --silent)
  INBOXES+=("$inbox")
  echo "Created inbox $i: $inbox"
done

# Monitor all inboxes in parallel
for inbox in "${INBOXES[@]}"; do
  chronosforge inbox watch "$inbox" --extract --timeout 5m &
done

# Wait for all monitoring processes
wait

echo "All inboxes monitored"
```

### Integration with Testing Framework

```python
# test_verification.py
import subprocess
import json
import time

class TemporaryEmail:
    def __init__(self, ttl="30m"):
        result = subprocess.run(
            ["chronosforge", "inbox", "create", "--ttl", ttl, "--silent"],
            capture_output=True,
            text=True
        )
        self.address = result.stdout.strip()
        self.inbox_id = self.address.split('@')[0]
    
    def wait_for_code(self, timeout=120):
        """Wait for verification code and return it"""
        result = subprocess.run(
            [
                "chronosforge", "extract", self.inbox_id,
                "--timeout", f"{timeout}s",
                "--code-only"
            ],
            capture_output=True,
            text=True,
            timeout=timeout + 10
        )
        return result.stdout.strip() if result.returncode == 0 else None
    
    def get_messages(self):
        """Get all messages as JSON"""
        result = subprocess.run(
            ["chronosforge", "inbox", "messages", self.inbox_id, "--format", "json"],
            capture_output=True,
            text=True
        )
        return json.loads(result.stdout) if result.returncode == 0 else []

# Usage in test
def test_user_signup():
    email = TemporaryEmail(ttl="10m")
    print(f"Testing with email: {email.address}")
    
    # Trigger signup
    signup_user(email.address)
    
    # Wait for verification code
    code = email.wait_for_code(timeout=120)
    assert code is not None, "Failed to receive verification code"
    
    # Complete verification
    verify_user(email.address, code)
    assert is_user_verified(email.address)
```

### Node.js Integration

```javascript
// email-verifier.js
const { exec } = require('child_process');
const util = require('util');
const execPromise = util.promisify(exec);

class ChronosForge {
  async createInbox(options = {}) {
    const args = ['inbox', 'create', '--silent'];
    if (options.ttl) args.push('--ttl', options.ttl);
    if (options.prefix) args.push('--prefix', options.prefix);
    
    const { stdout } = await execPromise(`chronosforge ${args.join(' ')}`);
    return stdout.trim();
  }
  
  async extractCode(inboxId, timeout = '2m') {
    try {
      const { stdout } = await execPromise(
        `chronosforge extract ${inboxId} --timeout ${timeout} --code-only`
      );
      return stdout.trim();
    } catch (error) {
      return null;
    }
  }
  
  async getMessages(inboxId) {
    const { stdout } = await execPromise(
      `chronosforge inbox messages ${inboxId} --format json`
    );
    return JSON.parse(stdout);
  }
}

// Usage
async function testEmailVerification() {
  const cf = new ChronosForge();
  
  // Create temporary inbox
  const email = await cf.createInbox({ ttl: '15m' });
  console.log(`Created temp email: ${email}`);
  
  // Trigger some action that sends verification email
  await sendVerificationEmail(email);
  
  // Extract code
  const code = await cf.extractCode(email, '3m');
  if (code) {
    console.log(`Verification code: ${code}`);
    await verifyWithCode(email, code);
  } else {
    throw new Error('Failed to extract verification code');
  }
}
```

### REST API Usage (Daemon Mode)

```bash
# Start daemon
chronosforge daemon start --port 8080

# Or with config
chronosforge daemon start --config config.yaml
```

```python
# Python REST API client
import requests
import time

API_BASE = "http://localhost:8080/api/v1"
API_KEY = os.environ.get("CHRONOSFORGE_API_KEY")

headers = {"Authorization": f"Bearer {API_KEY}"}

# Create inbox
response = requests.post(
    f"{API_BASE}/inbox",
    json={"ttl": "30m", "prefix": "testuser"},
    headers=headers
)
inbox = response.json()
inbox_id = inbox["id"]
email_address = inbox["address"]

# Watch for messages (Server-Sent Events)
import sseclient

messages_url = f"{API_BASE}/inbox/{inbox_id}/messages"
response = requests.get(messages_url, headers=headers, stream=True)
client = sseclient.SSEClient(response)

for event in client.events():
    if event.event == "message":
        message_data = json.loads(event.data)
        print(f"New message: {message_data['subject']}")
        
        # Extract code
        extract_response = requests.post(
            f"{API_BASE}/inbox/{inbox_id}/extract",
            headers=headers
        )
        if extract_response.status_code == 200:
            code = extract_response.json()["code"]
            print(f"Extracted code: {code}")
            break
```

## Advanced Patterns

### Custom Pattern Matching

```bash
# Extract with custom regex
chronosforge extract <inbox-id> \
  --custom-pattern '\b([A-Z]{3}-\d{4}-[A-Z]{2})\b' \
  --confidence 0.75

# Extract magic links instead of codes
chronosforge extract <inbox-id> \
  --extract-type link \
  --link-pattern 'verify\?token='
```

### Batch Operations

```bash
# Create multiple inboxes from file
while IFS= read -r prefix; do
  chronosforge inbox create --prefix "$prefix" --silent
done < prefixes.txt > inboxes.txt

# Watch all inboxes and log results
while IFS= read -r inbox; do
  chronosforge inbox watch "$inbox" --extract --timeout 5m >> results.log &
done < inboxes.txt
```

### Pipeline Integration

```bash
# Use in shell pipeline
chronosforge inbox create --silent | \
  tee /dev/tty | \
  xargs -I {} sh -c 'echo "Monitoring {}"; chronosforge inbox watch {} --extract'

# JSON processing with jq
chronosforge inbox messages $(cat inbox.id) --format json | \
  jq '.[] | select(.subject | contains("Verification")) | .body'
```

## Troubleshooting

### Inbox Creation Fails

```bash
# Check available domains
chronosforge domains list

# Try with specific domain
chronosforge inbox create --domain guerrillamail.com

# Check rate limits
chronosforge status
```

### Code Extraction Not Working

```bash
# List messages to verify receipt
chronosforge inbox messages <inbox-id>

# Try manual extraction with lower confidence
chronosforge extract <inbox-id> --confidence 0.5

# Enable debug mode
chronosforge --debug extract <inbox-id>

# View raw message content
chronosforge inbox message <inbox-id> <message-id> --raw
```

### Timeout Issues

```bash
# Increase timeout
chronosforge inbox watch <inbox-id> --timeout 10m

# Check inbox expiry
chronosforge inbox info <inbox-id>

# Manually refresh inbox
chronosforge inbox refresh <inbox-id>
```

### API Rate Limiting

```bash
# Check current usage
chronosforge status --show-limits

# View rate limit headers (daemon mode)
curl -i -H "Authorization: Bearer ${CHRONOSFORGE_API_KEY}" \
  http://localhost:8080/api/v1/status
```

### Configuration Issues

```bash
# Validate configuration
chronosforge config validate

# Show effective configuration
chronosforge config show

# Reset to defaults
chronosforge config reset
```

## Environment Variables

```bash
# API key for daemon mode
export CHRONOSFORGE_API_KEY="your-api-key-here"

# Webhook for notifications
export CHRONOSFORGE_WEBHOOK_URL="https://hooks.example.com/chronosforge"

# Custom config path
export CHRONOSFORGE_CONFIG="~/.config/chronosforge/custom.yaml"

# Debug mode
export CHRONOSFORGE_DEBUG=1
```

## Best Practices

1. **Set Appropriate TTLs**: Use shorter lifespans for automated tests (10-30m), longer for manual testing (1-2h)

2. **Clean Up Resources**: Always delete inboxes manually in long-running processes to avoid resource exhaustion

3. **Use Silent Mode for Automation**: The `--silent` flag outputs only essential data, perfect for scripting

4. **Monitor Rate Limits**: Check `chronosforge status` regularly when running high-volume operations

5. **Custom Extraction Rules**: For services not in the default library, add patterns to `extraction_rules.json`

6. **Error Handling**: Always check return codes and handle extraction failures gracefully

7. **Parallel Operations**: Use background jobs (`&`) for monitoring multiple inboxes, but respect rate limits

8. **Security**: Never hardcode API keys; use environment variables or secure vaults

## Integration Examples

### GitHub Actions

```yaml
# .github/workflows/test-verification.yml
name: Test Email Verification

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Download ChronosForge
        run: |
          wget -O chronosforge https://github.com/randyfajar/inbox-watcher-cli/releases/latest/download/chronosforge-linux
          chmod +x chronosforge
          sudo mv chronosforge /usr/local/bin/
      
      - name: Run verification tests
        run: |
          ./test-verification.sh
        env:
          CHRONOSFORGE_API_KEY: ${{ secrets.CHRONOSFORGE_API_KEY }}
```

### Docker Container

```dockerfile
FROM rust:latest as builder
WORKDIR /app
RUN git clone https://github.com/randyfajar/inbox-watcher-cli.git .
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/chronosforge /usr/local/bin/
RUN chronosforge --version
ENTRYPOINT ["chronosforge"]
```

This skill provides comprehensive coverage for AI agents to help developers use ChronosForge for temporal email management, OTP extraction, and verification testing automation.
