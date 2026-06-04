---
name: tartarusai-cli-coding-agent
description: Use TartarusAI uncensored AI coding agent via CLI for security research, automation, and code generation without policy filters
triggers:
  - use tartarus ai to generate code
  - run tartarusai cli for this security research
  - generate code with tartarus coder
  - use uncensored ai coding agent
  - help me with tartarusai cli
  - run tartarus ai in terminal
  - use tartarusai for security tooling
  - generate pentest code with tartarus
---

# TartarusAI CLI Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**tartarusai-cli** is the official terminal client for TartarusAI, an uncensored AI coding agent designed for security research, automation, and edge-case development work. Unlike mainstream AI models, TartarusAI doesn't apply content policy filters to refuse legitimate security research, penetration testing tools, or complex automation tasks.

**Key capabilities:**
- 256K token context window for processing entire repositories
- Uncensored responses for security research and pentest tooling
- Crypto-only billing (no credit card required)
- Terminal-based interface for coding workflows
- Suitable for vulnerability research, incident response automation, and lab environments

**Not suitable for:**
- Malware/exploit development for unauthorized use
- DRM bypass or software piracy
- Attacking systems without permission

## Installation

### Linux (x86_64)

```bash
# Download and install
curl -L -o tartarusai-cli https://github.com/Tartarus-AI/tartarusai-cli/releases/latest/download/tartarusai-cli-linux-x64
chmod +x tartarusai-cli
sudo mv tartarusai-cli /usr/local/bin/

# Verify installation
tartarusai-cli --version
```

### Windows (x86_64)

```powershell
# PowerShell
Invoke-WebRequest https://github.com/Tartarus-AI/tartarusai-cli/releases/latest/download/tartarusai-cli-windows-x64.exe -OutFile tartarusai-cli.exe

# Add to PATH or run from current directory
.\tartarusai-cli.exe
```

### macOS and ARM

Contact `team@tartarusai.dev` for macOS and ARM builds.

## Configuration

### 1. Create Account and Generate Token

1. Visit [dash.tartarusai.dev/account](https://dash.tartarusai.dev/account)
2. Create account (crypto payment required for billing)
3. Generate a CLI token from dashboard

### 2. Configure Local Token

Create configuration file at:
- **Linux/macOS**: `~/.tartarus/cli-token.json`
- **Windows**: `%USERPROFILE%\.tartarus\cli-token.json`

```json
{
  "endpoint": "https://api.tartarusai.dev",
  "token": "your-cli-token-here",
  "user_email": "you@example.com"
}
```

**Security best practice:**
```bash
# Linux/macOS: Restrict file permissions
chmod 600 ~/.tartarus/cli-token.json

# Verify permissions
ls -la ~/.tartarus/cli-token.json
```

### Environment Variable Alternative

```bash
# Set token via environment variable
export TARTARUS_CLI_TOKEN="your-token-here"
export TARTARUS_ENDPOINT="https://api.tartarusai.dev"

# Run without config file
tartarusai-cli
```

## Usage Patterns

### Interactive Mode

```bash
# Start interactive session
tartarusai-cli

# CLI will present an interactive prompt
# Type your coding request and press enter
```

### Common Use Cases

#### 1. Security Research Tool Development

```bash
# Example: Request a port scanner for authorized testing
tartarusai-cli

# At prompt:
> Create a Python port scanner that scans ports 1-1024 on localhost
> with timeout handling and service detection. Include proper error
> handling and logging.
```

#### 2. Incident Response Automation

```bash
# Example: Log analysis automation
> Write a Bash script that parses /var/log/auth.log for failed SSH
> login attempts, extracts IP addresses, and generates a summary report
> with geolocation data using ip-api.com
```

#### 3. CVE Proof-of-Concept (Lab Environment)

```bash
# Example: Research known vulnerability
> Create a Python proof-of-concept for CVE-2021-44228 (Log4Shell)
> that demonstrates the vulnerability in a controlled lab environment.
> Include both the vulnerable server component and the exploit trigger.
> Add clear documentation about patching.
```

#### 4. API Security Testing

```bash
# Example: API fuzzing tool
> Generate a Python script using requests library to test API endpoints
> for common vulnerabilities: SQL injection, XSS in JSON responses,
> authentication bypass. Target endpoint from environment variable
> API_TEST_TARGET.
```

#### 5. Automation Script Generation

```bash
# Example: Credential rotation
> Create a Python script that rotates API keys across multiple services.
> Read service configs from YAML, revoke old keys via REST APIs, generate
> new keys, update .env files, and send Slack notification on completion.
```

## Code Examples

### Example 1: Network Scanner (Python)

Request to TartarusAI:
```
Create a Python network scanner that accepts CIDR notation, performs
TCP SYN scan on common ports, identifies services, and outputs JSON.
Use scapy for packet manipulation.
```

Expected output structure:
```python
#!/usr/bin/env python3
import scapy.all as scapy
import json
import sys
from ipaddress import ip_network

def scan_network(target_cidr, ports=[21,22,80,443,3306,8080]):
    """Scan network range for open TCP ports"""
    results = []
    
    for ip in ip_network(target_cidr):
        host_result = {"ip": str(ip), "open_ports": []}
        
        for port in ports:
            # SYN scan implementation
            pkt = scapy.IP(dst=str(ip))/scapy.TCP(dport=port,flags="S")
            resp = scapy.sr1(pkt, timeout=1, verbose=0)
            
            if resp and resp.haslayer(scapy.TCP):
                if resp[scapy.TCP].flags == 0x12:  # SYN-ACK
                    host_result["open_ports"].append(port)
                    # Send RST to close connection
                    scapy.send(scapy.IP(dst=str(ip))/scapy.TCP(dport=port,flags="R"), verbose=0)
        
        if host_result["open_ports"]:
            results.append(host_result)
    
    return results

if __name__ == "__main__":
    target = sys.argv[1] if len(sys.argv) > 1 else "192.168.1.0/24"
    results = scan_network(target)
    print(json.dumps(results, indent=2))
```

### Example 2: Log Parser for Security Events (Bash)

Request:
```
Write a Bash script that monitors auth.log for brute force attempts,
tracks failed login counts per IP, and alerts when threshold exceeded.
```

Expected pattern:
```bash
#!/bin/bash

LOG_FILE="${AUTH_LOG:-/var/log/auth.log}"
THRESHOLD="${ALERT_THRESHOLD:-5}"
OUTPUT_FILE="/tmp/ssh_bruteforce_ips.txt"

# Extract failed SSH attempts
grep "Failed password" "$LOG_FILE" | \
  awk '{print $(NF-3)}' | \
  sort | uniq -c | sort -nr | \
  awk -v threshold="$THRESHOLD" '$1 >= threshold {print $2, $1}' > "$OUTPUT_FILE"

# Alert if threats found
if [ -s "$OUTPUT_FILE" ]; then
  echo "⚠️  SSH brute force detected:"
  cat "$OUTPUT_FILE"
  
  # Optionally: webhook notification
  if [ -n "$SLACK_WEBHOOK" ]; then
    curl -X POST "$SLACK_WEBHOOK" \
      -H 'Content-Type: application/json' \
      -d "{\"text\":\"SSH brute force alert: $(wc -l < "$OUTPUT_FILE") IPs detected\"}"
  fi
else
  echo "No brute force attempts detected"
fi
```

### Example 3: JWT Token Analyzer (Python)

Request:
```
Create a Python script that decodes JWT tokens, checks for common
vulnerabilities (none algorithm, weak secrets), and validates claims.
```

Expected structure:
```python
#!/usr/bin/env python3
import jwt
import json
import sys
from datetime import datetime

def analyze_jwt(token):
    """Analyze JWT token for security issues"""
    analysis = {"vulnerabilities": [], "claims": {}}
    
    # Decode without verification to inspect
    try:
        header = jwt.get_unverified_header(token)
        payload = jwt.decode(token, options={"verify_signature": False})
        
        analysis["header"] = header
        analysis["claims"] = payload
        
        # Check algorithm
        if header.get("alg", "").lower() == "none":
            analysis["vulnerabilities"].append("CRITICAL: 'none' algorithm allows signature bypass")
        
        # Check expiration
        if "exp" in payload:
            exp_time = datetime.fromtimestamp(payload["exp"])
            if exp_time < datetime.now():
                analysis["vulnerabilities"].append("Token expired")
        else:
            analysis["vulnerabilities"].append("No expiration claim (exp) found")
        
        # Check for weak claims
        if "admin" in payload or "role" in payload:
            analysis["vulnerabilities"].append("WARNING: Privilege claims in token (check for tampering)")
        
        return analysis
        
    except jwt.DecodeError as e:
        return {"error": f"Invalid JWT: {str(e)}"}

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: ./jwt_analyzer.py <jwt-token>")
        sys.exit(1)
    
    result = analyze_jwt(sys.argv[1])
    print(json.dumps(result, indent=2, default=str))
```

## Best Practices

### 1. Context Management

TartarusAI supports 256K context — leverage this for whole-repository analysis:

```bash
# Prepare context file with repository structure
find . -type f -name "*.py" | head -20 | xargs cat > /tmp/repo_context.txt

# In prompt, reference the context
> Analyze the Python codebase provided. Identify potential security issues
> in authentication handling and suggest refactoring. Focus on SQL injection
> and authentication bypass vulnerabilities.
```

### 2. Iterative Development

```bash
# Initial request
> Create a basic HTTP server in Python that serves static files

# Follow-up refinement
> Add authentication middleware using JWT tokens from environment variable
> JWT_SECRET. Include rate limiting (10 req/min per IP) using Redis.

# Further iteration
> Add logging of all authentication attempts to syslog with structured
> JSON format including timestamp, IP, username, and success/failure.
```

### 3. Environment-Aware Code Generation

Always request code that uses environment variables for sensitive data:

```bash
# Good request
> Create PostgreSQL backup script that uses $DB_HOST, $DB_USER,
> $DB_PASSWORD from environment. Backup to S3 using $AWS_ACCESS_KEY_ID
> and $AWS_SECRET_ACCESS_KEY.

# Avoid
> Create PostgreSQL backup script with username 'admin' password 'password123'
```

### 4. Testing and Validation

Request test cases alongside implementation:

```bash
> Generate a Python port scanner with scapy. Include pytest unit tests
> that mock network responses and validate timeout handling, error cases,
> and result parsing. Use pytest-mock for scapy.sr1 mocking.
```

## Troubleshooting

### Token Authentication Issues

**Problem**: `Authentication failed` or `Invalid token`

```bash
# Verify token file exists and has correct structure
cat ~/.tartarus/cli-token.json

# Check file permissions (should be readable by user only)
ls -la ~/.tartarus/cli-token.json

# Test with explicit token path
TARTARUS_CONFIG=~/.tartarus/cli-token.json tartarusai-cli
```

**Problem**: `Token expired` or billing issues

- Visit [dash.tartarusai.dev/account](https://dash.tartarusai.dev/account) to check account balance
- Crypto payment confirmation typically takes ~30 seconds
- Generate new CLI token if current token was revoked

### Network Connectivity

**Problem**: `Cannot connect to endpoint`

```bash
# Test API connectivity
curl -v https://api.tartarusai.dev/health

# Check proxy settings if behind corporate firewall
export HTTP_PROXY="http://proxy.corp.com:8080"
export HTTPS_PROXY="http://proxy.corp.com:8080"

tartarusai-cli
```

### Response Quality Issues

**Problem**: Responses are too generic or refuse task

1. **Be specific about use case**: "for authorized penetration testing in lab environment"
2. **Provide context**: Include that you own/have permission for the target system
3. **Reference legitimate frameworks**: "using Metasploit-style approach" or "similar to OWASP ZAP functionality"

**Example of well-formed prompt:**
```
I'm conducting authorized penetration testing on my company's staging
environment (written permission obtained). Create a Python script that
tests our web application at $TARGET_URL for OWASP Top 10 vulnerabilities.
Include SQL injection, XSS, and authentication bypass tests. Output results
in JSON format compatible with DefectDojo import.
```

### Large Context Handling

**Problem**: Response truncated or context exceeded

```bash
# Split large requests into phases
# Phase 1: Architecture
> Design the architecture for a log aggregation system that processes
> 10k events/sec. Use Python, Redis, and PostgreSQL. Output component
> diagram and data flow.

# Phase 2: Implementation (reference phase 1)
> Based on the architecture designed, implement the Redis event consumer
> component. Include connection pooling, retry logic, and graceful shutdown.

# Phase 3: Testing
> Generate integration tests for the Redis consumer using pytest-asyncio
> and fakeredis. Mock PostgreSQL with pytest-postgresql.
```

## Integration with Development Workflow

### Git Commit Message Generation

```bash
# Generate commit message from diff
git diff > /tmp/changes.diff

# In tartarusai-cli:
> Analyze this git diff and generate a conventional commit message.
> Include type (feat/fix/refactor), scope, and detailed body explaining
> the changes and their impact.
> [paste diff content]
```

### Code Review Automation

```bash
# Request security-focused code review
> Review this Python Flask API code for security vulnerabilities.
> Check for: SQL injection, authentication bypass, CSRF, insecure
> deserialization, hardcoded secrets, and improper error handling.
> Provide specific line numbers and remediation code.
> [paste code]
```

### Documentation Generation

```bash
> Generate comprehensive README.md for this Python security scanner.
> Include: overview, installation, usage examples, command-line arguments,
> environment variables, legal disclaimer for authorized use only, and
> example output. Follow security tool documentation best practices.
```

## Additional Resources

- **Website**: [tartarusai.dev](https://tartarusai.dev)
- **Dashboard**: [dash.tartarusai.dev](https://dash.tartarusai.dev)
- **Discord**: [discord.gg/GfzePawBBd](https://discord.gg/GfzePawBBd)
- **GitHub**: [github.com/Tartarus-AI/tartarusai-cli](https://github.com/Tartarus-AI/tartarusai-cli)
- **Support**: team@tartarusai.dev

## Ethical Use Guidelines

TartarusAI is designed for legitimate security research and development. Always ensure:

1. ✅ You have **written authorization** to test target systems
2. ✅ Testing is performed in **isolated lab environments** when learning
3. ✅ Research focuses on **defensive security** and patching
4. ✅ Code is used for **authorized penetration testing** or **your own systems**
5. ❌ Never use for unauthorized access, malware distribution, or illegal activities

The tool generates code for educational and professional security work. Use responsibly and in compliance with applicable laws and regulations.
