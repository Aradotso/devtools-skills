---
name: cliproxyapi-cloudflare-stack
description: Self-hosted OpenAI-compatible LLM API gateway with multi-credential pooling, Cloudflare email workers, and tunnel exposure
triggers:
  - set up a self-hosted OpenAI API gateway
  - deploy CLIProxyAPI with Cloudflare tunnel
  - configure multi-credential LLM proxy
  - create automated email credential system
  - expose internal API gateway with cloudflared
  - manage multiple LLM credentials with failover
  - build OpenAI-compatible proxy on Cloudflare
  - set up Grok API credential rotation
---

# CLIProxyAPI Cloudflare Stack

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

This project provides a complete self-hosted stack for running an OpenAI-compatible LLM API gateway with:

- **CLIProxyAPI**: Multi-credential pooling with hot-reload and automatic failover
- **Cloudflare Worker + D1**: Programmatic email receiving for credential verification
- **Cloudflare Tunnel**: Zero-port-forwarding HTTPS exposure of internal gateway
- **Unified Interface**: Single `/v1/*` endpoint compatible with OpenAI SDK

Use case: Run your own LLM API proxy without sharing credentials with third-party services, supporting multiple provider accounts with automatic rotation.

## Architecture Overview

```
[Client] 
  ↓ HTTPS
[Cloudflare Tunnel] → public domain
  ↓
[CLIProxyAPI Gateway] → localhost:5000
  ↓
[Multiple LLM Providers] (Grok, OpenAI, etc.)

[Email] → Cloudflare Email Routing → Worker → D1 Database
```

## Installation

### Prerequisites

- Cloudflare account with domain
- VM or server with Docker (for CLIProxyAPI)
- Node.js 18+ (for Wrangler CLI)
- `cloudflared` binary

### 1. Deploy CLIProxyAPI Gateway

```bash
# Clone the repository
git clone https://github.com/xsser/cliproxyapi-cloudflare-stack.git
cd cliproxyapi-cloudflare-stack

# Deploy to VM (Ubuntu/Debian)
bash scripts/deploy_vm.sh

# The script will:
# - Install Docker if needed
# - Pull CLIProxyAPI image
# - Generate API key (sk-xxxxx)
# - Create auth directory
# - Start container on port 5000
```

**Expected output:**
```
✓ CLIProxyAPI deployed
✓ API Key: sk-1234567890abcdef
✓ Auth directory: /opt/cpa/auth
✓ Service running on localhost:5000
```

Save the `sk-` key — this is your gateway authentication token.

### 2. Set Up Email Worker

```bash
# Install Wrangler
npm install -g wrangler

# Login to Cloudflare
wrangler login

# Copy and configure template
cp templates/wrangler.toml.template wrangler.toml

# Edit wrangler.toml
nano wrangler.toml
```

**wrangler.toml configuration:**
```toml
name = "email-receiver"
main = "src/index.js"
compatibility_date = "2024-01-01"

[[d1_databases]]
binding = "DB"
database_name = "email_codes"
database_id = "YOUR_D1_DATABASE_ID"

[vars]
ALLOWED_DOMAINS = "x.ai,openai.com"
```

**Create D1 database:**
```bash
# Create database
wrangler d1 create email_codes

# Note the database_id from output and update wrangler.toml

# Initialize schema
wrangler d1 execute email_codes --file=src/schema.sql
```

**Deploy worker:**
```bash
wrangler deploy

# Output will show: https://email-receiver.<your-account>.workers.dev
```

**Configure Cloudflare Email Routing:**
1. Go to Cloudflare Dashboard → Email Routing
2. Add catch-all route: `*@yourdomain.com` → Worker `email-receiver`
3. Verify domain email routing is enabled

### 3. Add Credentials

Credentials are JSON files in the auth directory with provider-specific format.

**Example Grok credential (grok_account1.json):**
```json
{
  "provider": "grok",
  "api_key": "grok-xxxxxxxxxxxxx",
  "email": "your-email@domain.com",
  "status": "active",
  "rate_limit": {
    "requests_per_minute": 60
  }
}
```

**Example OpenAI credential (openai_account1.json):**
```json
{
  "provider": "openai",
  "api_key": "sk-proj-xxxxxxxxxxxxx",
  "organization_id": "org-xxxxxxxxxxxxx",
  "status": "active"
}
```

**Sync credentials to VM:**
```bash
# From local directory
SRC=./auth_local VM_HOST=user@yourvm.com bash scripts/sync_credentials.sh

# Or directly on VM
scp credentials/*.json user@yourvm:/opt/cpa/auth/
```

CLIProxyAPI hot-reloads credentials automatically (checks every 30s).

### 4. Expose with Cloudflare Tunnel

```bash
# Install cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Authenticate
cloudflared tunnel login

# Create tunnel
cloudflared tunnel create cpa-gateway

# Note the tunnel ID from output

# Copy service template
sudo cp templates/cloudflared-cpa.service.template /etc/systemd/system/cloudflared-cpa.service

# Edit service file
sudo nano /etc/systemd/system/cloudflared-cpa.service
```

**cloudflared-cpa.service configuration:**
```ini
[Unit]
Description=Cloudflare Tunnel for CPA Gateway
After=network.target

[Service]
Type=simple
User=cloudflared
ExecStart=/usr/local/bin/cloudflared tunnel --no-autoupdate run --token YOUR_TUNNEL_TOKEN
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**Create tunnel config:**
```bash
mkdir -p ~/.cloudflared
nano ~/.cloudflared/config.yml
```

**config.yml:**
```yaml
tunnel: YOUR_TUNNEL_ID
credentials-file: /home/user/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: api.yourdomain.com
    service: http://localhost:5000
  - service: http_status:404
```

**Start tunnel:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared-cpa
sudo systemctl start cloudflared-cpa
sudo systemctl status cloudflared-cpa
```

**Add DNS record in Cloudflare:**
```
Type: CNAME
Name: api
Target: YOUR_TUNNEL_ID.cfargotunnel.com
Proxy: Yes (orange cloud)
```

## Usage

### API Endpoint

Your gateway is now available at: `https://api.yourdomain.com/v1`

### Using with OpenAI SDK

**Python:**
```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.yourdomain.com/v1",
    api_key=os.environ.get("CPA_API_KEY")  # The sk- key from deployment
)

response = client.chat.completions.create(
    model="grok-beta",
    messages=[
        {"role": "user", "content": "Hello, Grok!"}
    ]
)

print(response.choices[0].message.content)
```

**JavaScript/TypeScript:**
```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'https://api.yourdomain.com/v1',
  apiKey: process.env.CPA_API_KEY
});

const response = await client.chat.completions.create({
  model: 'grok-beta',
  messages: [
    { role: 'user', content: 'Hello, Grok!' }
  ]
});

console.log(response.choices[0].message.content);
```

**cURL:**
```bash
curl https://api.yourdomain.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $CPA_API_KEY" \
  -d '{
    "model": "grok-beta",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Key Commands

### CLIProxyAPI Management

```bash
# View logs
docker logs -f cpa-gateway

# Restart gateway
docker restart cpa-gateway

# Update to latest
docker pull cliproxyapi/cliproxyapi:latest
docker restart cpa-gateway

# Check credential status
curl http://localhost:5000/admin/credentials \
  -H "Authorization: Bearer $CPA_API_KEY"

# Health check
curl http://localhost:5000/health
```

### Credential Sync Script

```bash
# Sync from local to VM
SRC=./credentials VM_HOST=user@vm.com bash scripts/sync_credentials.sh

# Dry run (show what would be copied)
DRY_RUN=1 SRC=./credentials VM_HOST=user@vm.com bash scripts/sync_credentials.sh

# Custom auth directory
SRC=./credentials VM_HOST=user@vm.com AUTH_DIR=/custom/path bash scripts/sync_credentials.sh
```

### Email Worker Queries

```bash
# Query D1 for received codes
wrangler d1 execute email_codes --command="SELECT * FROM verification_codes ORDER BY received_at DESC LIMIT 10"

# Clear old codes (older than 1 hour)
wrangler d1 execute email_codes --command="DELETE FROM verification_codes WHERE received_at < datetime('now', '-1 hour')"

# View worker logs
wrangler tail
```

### Tunnel Management

```bash
# Check tunnel status
sudo systemctl status cloudflared-cpa

# View tunnel logs
sudo journalctl -u cloudflared-cpa -f

# Restart tunnel
sudo systemctl restart cloudflared-cpa

# List all tunnels
cloudflared tunnel list

# Delete tunnel
cloudflared tunnel delete cpa-gateway
```

## Configuration

### CLIProxyAPI Environment Variables

```bash
# In deploy_vm.sh or docker-compose.yml
CPA_PORT=5000                    # Gateway port
CPA_AUTH_DIR=/opt/cpa/auth      # Credentials directory
CPA_LOG_LEVEL=info              # debug, info, warn, error
CPA_POOL_SIZE=10                # Max concurrent requests per credential
CPA_RETRY_ATTEMPTS=3            # Failed request retries
CPA_HEALTH_CHECK_INTERVAL=60    # Credential health check (seconds)
```

### Worker Environment Variables

Edit `wrangler.toml`:
```toml
[vars]
ALLOWED_DOMAINS = "x.ai,openai.com,anthropic.com"
MAX_CODE_AGE_MINUTES = 10
AUTO_CLEANUP_ENABLED = "true"
```

### Rate Limiting

Configure per-credential in JSON:
```json
{
  "provider": "grok",
  "api_key": "...",
  "rate_limit": {
    "requests_per_minute": 60,
    "tokens_per_minute": 100000,
    "requests_per_day": 5000
  }
}
```

## Common Patterns

### Multi-Provider Setup

```bash
# auth directory structure
/opt/cpa/auth/
  ├── grok_account1.json
  ├── grok_account2.json
  ├── openai_personal.json
  ├── openai_work.json
  └── anthropic_main.json
```

Each credential is automatically pooled. CLIProxyAPI routes based on model name:
- `grok-*` → Grok credentials
- `gpt-*` → OpenAI credentials
- `claude-*` → Anthropic credentials

### Automatic Failover

```python
# Client code doesn't change - failover is automatic
client = OpenAI(
    base_url="https://api.yourdomain.com/v1",
    api_key=os.environ.get("CPA_API_KEY")
)

# If grok_account1 fails, CLIProxyAPI automatically tries grok_account2
response = client.chat.completions.create(
    model="grok-beta",
    messages=[{"role": "user", "content": "Test"}]
)
```

### Streaming Responses

```python
stream = client.chat.completions.create(
    model="grok-beta",
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end='')
```

### Monitoring Credential Usage

```bash
# Check logs for credential selection
docker logs cpa-gateway 2>&1 | grep "Using credential"

# Example output:
# [INFO] Using credential grok_account1 for model grok-beta
# [INFO] Request successful with grok_account1 (245ms)
# [WARN] Credential grok_account1 rate limited, trying grok_account2
```

## Troubleshooting

### Gateway Returns 502 Bad Gateway

**Cause:** CLIProxyAPI container not running or no active credentials.

**Fix:**
```bash
# Check container
docker ps | grep cpa-gateway

# View logs
docker logs cpa-gateway

# Verify credentials exist
ls -la /opt/cpa/auth/

# Restart
docker restart cpa-gateway
```

### "No active credentials for provider" Error

**Cause:** No valid credential JSON files for requested model's provider.

**Fix:**
```bash
# Check credential format
cat /opt/cpa/auth/grok_account1.json

# Must have:
# - "provider" field matching model prefix
# - "api_key" field
# - "status": "active"

# Force credential reload
docker restart cpa-gateway
```

### Email Worker Not Receiving Codes

**Cause:** Email routing not configured or D1 binding missing.

**Fix:**
```bash
# Test email routing
echo "test" | mail -s "Test" test@yourdomain.com

# Check worker logs
wrangler tail

# Verify D1 binding
wrangler d1 info email_codes

# Redeploy worker
wrangler deploy
```

### Tunnel Connection Issues

**Cause:** Tunnel service not running or incorrect credentials.

**Fix:**
```bash
# Check tunnel status
sudo systemctl status cloudflared-cpa

# Test local gateway
curl http://localhost:5000/health

# Verify tunnel config
cat ~/.cloudflared/config.yml

# Check DNS
nslookup api.yourdomain.com

# Restart tunnel
sudo systemctl restart cloudflared-cpa
```

### Rate Limit Exceeded

**Cause:** All credentials in pool exhausted rate limits.

**Fix:**
```bash
# Add more credentials to pool
scp new_credential.json user@vm:/opt/cpa/auth/

# Adjust rate limits in credential JSON
nano /opt/cpa/auth/grok_account1.json
# Increase "requests_per_minute"

# Wait for rate limit reset (typically 1 minute)
```

### Invalid API Key Response

**Cause:** Using wrong API key or credential expired.

**Fix:**
```bash
# Verify you're using the gateway key (sk-), not provider key
echo $CPA_API_KEY

# Check credential validity
docker logs cpa-gateway | grep "auth failed"

# Update expired credential
nano /opt/cpa/auth/expired_credential.json
# Update api_key or set "status": "inactive"
```

### High Latency

**Cause:** VM resources constrained or network issues.

**Fix:**
```bash
# Check CPU/memory
htop

# Check network latency to providers
ping api.x.ai
ping api.openai.com

# Increase pool size for better concurrency
# Edit docker run command in deploy_vm.sh:
-e CPA_POOL_SIZE=20

# Redeploy
bash scripts/deploy_vm.sh
```

## Security Best Practices

1. **Never commit credentials**: Keep `.json` files in `.gitignore`
2. **Rotate gateway key**: Regenerate `sk-` key periodically
3. **Use environment variables**: Store all secrets in env vars, not code
4. **Enable Cloudflare WAF**: Add rate limiting and bot protection to tunnel domain
5. **Monitor usage**: Set up alerts for abnormal API usage patterns
6. **Restrict tunnel access**: Use Cloudflare Access for additional auth layer

## Advanced: Custom Credential Providers

Create custom credential format for new providers:

```json
{
  "provider": "custom-llm",
  "api_key": "...",
  "endpoint": "https://api.custom-llm.com/v1",
  "model_prefix": "custom-",
  "status": "active",
  "rate_limit": {
    "requests_per_minute": 100
  }
}
```

CLIProxyAPI will route any model starting with `custom-` to this credential's endpoint.
