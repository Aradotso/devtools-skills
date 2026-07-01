---
name: chronosforge-temporal-email-cli
description: CLI toolkit for generating temporary emails, monitoring inboxes in real-time, and auto-extracting OTPs for testing and automation workflows
triggers:
  - "create a temporary email for testing"
  - "monitor inbox for verification code"
  - "extract OTP from temporary email"
  - "set up disposable email address"
  - "watch for incoming emails and codes"
  - "automate email verification testing"
  - "generate ephemeral inbox for signup flow"
  - "parse verification codes from email"
---

# ChronosForge Temporal Email CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

ChronosForge (inbox-watcher-cli) is a temporal email management toolkit for automated testing, verification flow testing, and development workflows. It generates disposable email addresses, monitors inboxes in real-time via WebSocket connections, and automatically extracts OTP codes and verification tokens from incoming messages.

## Installation

Download from the official release page:

```bash
# Visit the download page
# https://randyfajar.github.io/inbox-watcher-cli/

# After download, make executable (Linux/macOS)
chmod +x chronosforge

# Move to PATH
sudo mv chronosforge /usr/local/bin/

# Verify installation
chronosforge --version
```

Or build from source (Rust):

```bash
git clone https://github.com/randyfajar/inbox-watcher-cli.git
cd inbox-watcher-cli
cargo build --release
./target/release/chronosforge --version
```

## Core Concepts

### Temporal Inbox Lifecycle

1. **Generation** - Create inbox with TTL (time-to-live)
2. **Monitoring** - Real-time WebSocket connection for incoming mail
3. **Extraction** - Atomic Code Extractor (ACE) identifies verification codes
4. **Consumption** - Use extracted code in your workflow
5. **Disposal** - Automatic cleanup after TTL expiration

### Operating Modes

- **CLI Mode** - Headless automation and scripting
- **TUI Mode** - Interactive terminal dashboard
- **Daemon Mode** - Background REST API service

## CLI Commands

### Generate Temporary Inbox

```bash
# Basic inbox generation
chronosforge create

# Custom domain and prefix
chronosforge create --domain tempmail.com --prefix test

# Set custom TTL (in minutes)
chronosforge create --ttl 30

# Silent mode (output only email address)
chronosforge create --silent
```

### Monitor Inbox

```bash
# Watch for incoming messages
chronosforge watch <email-address>

# With auto-extraction enabled
chronosforge watch <email-address> --extract

# Filter by sender
chronosforge watch <email-address> --filter "noreply@github.com"

# Output as JSON
chronosforge watch <email-address> --json
```

### Extract OTP Codes

```bash
# Extract from latest message
chronosforge extract <email-address>

# Extract from specific message ID
chronosforge extract <email-address> --message-id <id>

# Output only the code
chronosforge extract <email-address> --code-only
```

### List Available Domains

```bash
# Show all supported domains
chronosforge domains

# Show domains with reputation scores
chronosforge domains --with-reputation
```

### TUI Mode

```bash
# Launch interactive dashboard
chronosforge tui

# Start with specific inbox
chronosforge tui --inbox <email-address>
```

## Configuration

Create `~/.chronosforge/config.yaml`:

```yaml
# Domain preferences
domains:
  preferred:
    - tempmail.com
    - guerrillamail.com
  blacklist:
    - spammy-domain.com

# Default TTL in minutes
default_ttl: 60

# Extraction settings
extraction:
  timeout: 300  # seconds to wait for code
  context_aware: true
  fallback_regex: true

# API settings (for daemon mode)
api:
  port: 8080
  rate_limit:
    inbox_per_hour: 100
    extract_per_hour: 1000
  
# Notification settings
notifications:
  on_code_extracted: true
  on_inbox_expiry: true
```

### Custom Extraction Rules

Create `~/.chronosforge/extraction_rules.json`:

```json
{
  "rules": [
    {
      "service": "custom-service",
      "patterns": [
        {
          "type": "numeric",
          "regex": "Your code is: (\\d{6})",
          "group": 1
        },
        {
          "type": "alphanumeric",
          "regex": "Verification token: ([A-Z0-9]{8})",
          "group": 1
        }
      ],
      "sender_patterns": [
        "noreply@custom-service.com",
        "verify@custom-service.com"
      ],
      "subject_keywords": [
        "verification",
        "confirm your email"
      ]
    }
  ]
}
```

## REST API Usage (Daemon Mode)

Start the API server:

```bash
chronosforge daemon --port 8080 --api-key-env CHRONOSFORGE_API_KEY
```

### API Endpoints

#### Create Inbox

```bash
curl -X POST http://localhost:8080/api/v1/inbox \
  -H "Authorization: Bearer $CHRONOSFORGE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "domain": "tempmail.com",
    "prefix": "test",
    "ttl": 3600
  }'
```

Response:

```json
{
  "inbox_id": "abc123",
  "email": "test-xyz@tempmail.com",
  "expires_at": "2026-07-01T16:30:00Z",
  "status": "active"
}
```

#### Get Inbox Messages

```bash
curl http://localhost:8080/api/v1/inbox/abc123 \
  -H "Authorization: Bearer $CHRONOSFORGE_API_KEY"
```

#### Extract OTP

```bash
curl -X POST http://localhost:8080/api/v1/inbox/abc123/extract \
  -H "Authorization: Bearer $CHRONOSFORGE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message_id": "msg456"
  }'
```

Response:

```json
{
  "code": "123456",
  "type": "numeric",
  "confidence": 0.98,
  "service": "github",
  "extracted_at": "2026-07-01T15:45:00Z"
}
```

#### Server-Sent Events (Real-time)

```bash
curl -N http://localhost:8080/api/v1/inbox/abc123/messages \
  -H "Authorization: Bearer $CHRONOSFORGE_API_KEY" \
  -H "Accept: text/event-stream"
```

## Common Patterns

### Automated Signup Testing

```bash
#!/bin/bash

# Generate temporary email
EMAIL=$(chronosforge create --silent --ttl 10)
echo "Testing with: $EMAIL"

# Trigger signup flow
curl -X POST https://example.com/api/signup \
  -d "email=$EMAIL&password=Test123!"

# Wait for verification email and extract code
CODE=$(chronosforge watch "$EMAIL" --extract --timeout 60 --code-only)

if [ -n "$CODE" ]; then
  echo "Verification code: $CODE"
  
  # Use code to verify account
  curl -X POST https://example.com/api/verify \
    -d "email=$EMAIL&code=$CODE"
else
  echo "Failed to receive verification code"
  exit 1
fi
```

### Multi-Account Testing

```bash
#!/bin/bash

# Generate 10 inboxes
for i in {1..10}; do
  EMAIL=$(chronosforge create --prefix "test$i" --silent)
  echo "$EMAIL" >> test_emails.txt
  
  # Start background watcher for each
  chronosforge watch "$EMAIL" --extract --json > "inbox_$i.log" &
done

# Wait for all watchers
wait

# Parse results
for i in {1..10}; do
  CODE=$(jq -r '.code' "inbox_$i.log")
  echo "Inbox $i code: $CODE"
done
```

### Python Integration

```python
import subprocess
import json
import time

class TemporalEmail:
    def __init__(self, ttl=60):
        self.ttl = ttl
        self.email = None
        self.inbox_id = None
    
    def create(self, prefix=None):
        """Generate temporary email"""
        cmd = ["chronosforge", "create", "--silent", "--ttl", str(self.ttl)]
        if prefix:
            cmd.extend(["--prefix", prefix])
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        self.email = result.stdout.strip()
        return self.email
    
    def wait_for_code(self, timeout=300, service_filter=None):
        """Wait for and extract verification code"""
        cmd = [
            "chronosforge", "watch", self.email,
            "--extract", "--json", "--timeout", str(timeout)
        ]
        if service_filter:
            cmd.extend(["--filter", service_filter])
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        if result.returncode == 0:
            data = json.loads(result.stdout)
            return data.get("code")
        return None
    
    def get_messages(self):
        """Retrieve all messages"""
        cmd = ["chronosforge", "extract", self.email, "--json"]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        if result.returncode == 0:
            return json.loads(result.stdout)
        return []

# Usage example
email_client = TemporalEmail(ttl=30)
test_email = email_client.create(prefix="pytest")
print(f"Testing with: {test_email}")

# Trigger verification flow
# ... your signup logic ...

# Wait for code
code = email_client.wait_for_code(timeout=60)
if code:
    print(f"Extracted code: {code}")
else:
    raise Exception("No verification code received")
```

### Node.js Integration

```javascript
const { spawn } = require('child_process');

class ChronosForgeClient {
  async createInbox(options = {}) {
    const args = ['create', '--silent'];
    
    if (options.prefix) args.push('--prefix', options.prefix);
    if (options.ttl) args.push('--ttl', options.ttl.toString());
    if (options.domain) args.push('--domain', options.domain);
    
    const email = await this._exec('chronosforge', args);
    return email.trim();
  }
  
  async watchForCode(email, options = {}) {
    const args = ['watch', email, '--extract', '--json'];
    
    if (options.timeout) args.push('--timeout', options.timeout.toString());
    if (options.filter) args.push('--filter', options.filter);
    
    const output = await this._exec('chronosforge', args);
    const data = JSON.parse(output);
    return data.code;
  }
  
  async extractCode(email, messageId = null) {
    const args = ['extract', email, '--code-only'];
    if (messageId) args.push('--message-id', messageId);
    
    const code = await this._exec('chronosforge', args);
    return code.trim();
  }
  
  _exec(command, args) {
    return new Promise((resolve, reject) => {
      const proc = spawn(command, args);
      let stdout = '';
      let stderr = '';
      
      proc.stdout.on('data', (data) => stdout += data.toString());
      proc.stderr.on('data', (data) => stderr += data.toString());
      
      proc.on('close', (code) => {
        if (code === 0) resolve(stdout);
        else reject(new Error(stderr));
      });
    });
  }
}

// Usage
(async () => {
  const client = new ChronosForgeClient();
  
  const email = await client.createInbox({
    prefix: 'test',
    ttl: 30
  });
  
  console.log(`Created: ${email}`);
  
  // Trigger your signup flow here
  
  const code = await client.watchForCode(email, {
    timeout: 60,
    filter: 'noreply@'
  });
  
  console.log(`Code: ${code}`);
})();
```

### CI/CD Pipeline Integration

```yaml
# .github/workflows/test.yml
name: Integration Tests

on: [push, pull_request]

jobs:
  test-verification-flow:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install ChronosForge
        run: |
          curl -L https://randyfajar.github.io/inbox-watcher-cli/download/linux -o chronosforge
          chmod +x chronosforge
          sudo mv chronosforge /usr/local/bin/
      
      - name: Run verification tests
        env:
          TEST_ENV: ci
        run: |
          # Generate test email
          export TEST_EMAIL=$(chronosforge create --silent --ttl 10)
          echo "Testing with: $TEST_EMAIL"
          
          # Run test suite
          npm test -- --email=$TEST_EMAIL
          
          # Extract verification code
          export VERIFY_CODE=$(chronosforge watch $TEST_EMAIL --extract --timeout 120 --code-only)
          
          # Verify code worked
          curl -f -X POST $API_URL/verify -d "email=$TEST_EMAIL&code=$VERIFY_CODE"
```

## Troubleshooting

### Code Not Extracted

**Problem**: `chronosforge extract` returns no code despite email containing one.

**Solutions**:

1. Check extraction rules match the email format:

```bash
# View the raw email content
chronosforge watch <email> --raw

# Try fallback mode
chronosforge extract <email> --fallback-regex
```

2. Add custom extraction pattern:

```json
{
  "rules": [
    {
      "service": "your-service",
      "patterns": [
        {
          "type": "numeric",
          "regex": "code:\\s*(\\d{4,8})",
          "group": 1
        }
      ]
    }
  ]
}
```

### Timeout Errors

**Problem**: Watch command times out before email arrives.

**Solutions**:

```bash
# Increase timeout
chronosforge watch <email> --timeout 600  # 10 minutes

# Check domain reputation
chronosforge domains --with-reputation

# Try different domain
chronosforge create --domain guerrillamail.com
```

### Inbox Already Expired

**Problem**: Inbox expires before code arrives.

**Solutions**:

```bash
# Increase TTL
chronosforge create --ttl 120  # 2 hours

# Check inbox status
chronosforge status <email>
```

### Rate Limiting

**Problem**: API returns 429 Too Many Requests.

**Solutions**:

```bash
# Check rate limit status
chronosforge status --rate-limits

# Use exponential backoff
for i in {1..5}; do
  chronosforge create && break || sleep $((2**i))
done
```

### WebSocket Connection Issues

**Problem**: Real-time monitoring fails to connect.

**Solutions**:

```bash
# Test connectivity
chronosforge test-connection --domain tempmail.com

# Use polling mode as fallback
chronosforge watch <email> --poll-interval 5

# Check firewall/proxy settings
export HTTP_PROXY=http://proxy:8080
export HTTPS_PROXY=http://proxy:8080
```

### Daemon Mode Not Starting

**Problem**: API server fails to start.

**Solutions**:

```bash
# Check port availability
lsof -i :8080

# Use different port
chronosforge daemon --port 8081

# Check logs
chronosforge daemon --log-level debug

# Verify API key env var
echo $CHRONOSFORGE_API_KEY
```

## Best Practices

1. **Always set appropriate TTL** - Match inbox lifetime to expected workflow duration
2. **Use prefix naming** - Helps identify inboxes in multi-test scenarios
3. **Filter by sender** - Reduces noise and extraction errors
4. **Handle extraction failures** - Not all codes can be auto-extracted
5. **Clean up resources** - Let inboxes expire naturally or use API to delete
6. **Rate limit compliance** - Don't create excessive inboxes in short periods
7. **Secure API keys** - Store in environment variables, never hardcode
8. **Test extraction rules** - Validate custom patterns before production use

## Security Considerations

- Temporal inboxes are **not secure** for sensitive data
- Emails are stored temporarily and may be accessible via the provider
- Use only for testing, development, and non-critical workflows
- Never use for production user accounts or sensitive registrations
- API keys should be rotated regularly
- Run daemon mode behind authentication in production environments
