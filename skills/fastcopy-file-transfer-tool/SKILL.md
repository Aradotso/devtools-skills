---
name: fastcopy-file-transfer-tool
description: High-speed file copy utility with parallel streams, integrity verification, and automation capabilities
triggers:
  - "how do I use FastCopy to transfer files quickly"
  - "configure FastCopy for parallel file copying"
  - "set up FastCopy profile for bulk data migration"
  - "FastCopy command line options and examples"
  - "optimize FastCopy transfer speed settings"
  - "create FastCopy automation script"
  - "FastCopy YAML profile configuration"
  - "troubleshoot FastCopy transfer errors"
---

# FastCopy File Transfer Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file transfer and duplication utility designed for professionals who need speed, reliability, and automation. It supports parallel streams, integrity verification, profile-based configurations, and cross-platform operation (Windows, macOS, Linux).

**Note:** This project appears to be a wrapper/utility around file transfer operations. The repository focuses on activation and configuration rather than source code. Use with caution and ensure you have legitimate licenses for any commercial software.

## Installation

### Windows
```bash
# Download from the repository release page
# Extract to desired location
# Add to PATH for CLI access
setx PATH "%PATH%;C:\Program Files\FastCopy"
```

### macOS/Linux
```bash
# Download portable binary
wget https://example.com/fastcopy-portable.tar.gz
tar -xzf fastcopy-portable.tar.gz
sudo mv fastcopy /usr/local/bin/
chmod +x /usr/local/bin/fastcopy
```

## Key Commands (CLI)

### Basic File Copy
```bash
# Simple copy with default settings
fastcopy /source/path /destination/path

# Copy with verification
fastcopy /source/path /destination/path --verify

# Copy with custom stream count
fastcopy /source/path /destination/path --streams 32
```

### Advanced Options
```bash
# High-priority transfer with debug logging
fastcopy --profile my-profile.yaml --priority high --log-level debug

# Ad-hoc transfer with maximum performance
fastcopy source.iso /backups/ --streams 64 --no-verify --tag "backup_20260215"

# Resume interrupted transfer
fastcopy --resume /source /destination

# Dry run (test without copying)
fastcopy /source /destination --dry-run
```

### Common CLI Flags
- `--streams <N>` - Number of concurrent transfer streams (1-64)
- `--buffer-size <MB>` - Buffer size in megabytes
- `--verify` - Enable integrity checking (SHA-256/CRC-32)
- `--no-verify` - Disable verification for speed
- `--resume` - Resume interrupted transfers
- `--priority <level>` - Set process priority (low, normal, high)
- `--log-level <level>` - Logging verbosity (info, debug, error)
- `--tag <name>` - Tag the transfer for tracking
- `--profile <path>` - Use YAML profile configuration

## Profile Configuration (YAML)

### Basic Profile
```yaml
# basic-transfer.yaml
transfer:
  mode: "balanced"              # standard, balanced, turbo
  streams: 16                   # concurrent streams (max 64)
  buffer_size_mb: 128
  verify: true
  resume: always                # always, on_failure, never

paths:
  source: "/data/raw"
  destination: "/backup/archive"

logging:
  level: "info"                 # debug, info, warn, error
  file: "/var/log/fastcopy.log"
```

### Advanced Profile with Hooks
```yaml
# production-profile.yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
  verify: true
  resume: always
  retry_attempts: 5
  retry_backoff_seconds: 30

paths:
  source: "/mnt/storage/production"
  destination: "//nas.company.com/backup/prod"

schedule:
  type: "watch"                 # one_time, watch, cron
  interval_seconds: 300
  cron: "0 2 * * *"            # Optional: run at 2 AM daily

filters:
  include:
    - "*.mp4"
    - "*.mkv"
    - "*.mov"
  exclude:
    - "*.tmp"
    - "*.cache"
    - ".DS_Store"

hooks:
  on_start: "echo 'Transfer starting' | mail -s 'FastCopy' admin@example.com"
  on_complete: "notify-send 'Transfer Complete'"
  on_error: "/opt/scripts/alert-on-failure.sh"

logging:
  level: "debug"
  file: "/var/log/fastcopy-production.log"
  rotate: true
  max_size_mb: 100
```

### Network Transfer Profile
```yaml
# network-sync.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: always
  compression: true             # Compress during transfer
  encryption: "aes256"          # Encrypt network transfers

paths:
  source: "/local/media"
  destination: "ssh://user@remote.server:/backup/media"

credentials:
  ssh_key: "${SSH_PRIVATE_KEY_PATH}"
  username: "${REMOTE_USER}"

network:
  bandwidth_limit_mbps: 0       # 0 = unlimited
  timeout_seconds: 300
  keepalive: true

logging:
  level: "info"
  file: "/var/log/fastcopy-network.log"
```

## Usage in Scripts

### Bash Script Example
```bash
#!/bin/bash
# automated-backup.sh

set -e

PROFILE="/etc/fastcopy/profiles/nightly-backup.yaml"
LOG="/var/log/backup-$(date +%Y%m%d).log"

echo "[$(date)] Starting backup..." >> "$LOG"

if fastcopy --profile "$PROFILE" --log-level info; then
    echo "[$(date)] Backup completed successfully" >> "$LOG"
    # Clean up old logs
    find /var/log -name "backup-*.log" -mtime +30 -delete
else
    echo "[$(date)] Backup failed with exit code $?" >> "$LOG"
    # Send alert
    echo "Backup failed" | mail -s "ALERT: FastCopy Failure" admin@example.com
    exit 1
fi
```

### Python Integration Example
```python
#!/usr/bin/env python3
import subprocess
import yaml
import sys
from pathlib import Path

def run_fastcopy(source, destination, streams=16, verify=True):
    """Run FastCopy with specified parameters"""
    cmd = [
        "fastcopy",
        str(source),
        str(destination),
        f"--streams={streams}",
        "--log-level=info"
    ]
    
    if verify:
        cmd.append("--verify")
    
    try:
        result = subprocess.run(
            cmd,
            check=True,
            capture_output=True,
            text=True
        )
        print(f"Success: {result.stdout}")
        return True
    except subprocess.CalledProcessError as e:
        print(f"Error: {e.stderr}", file=sys.stderr)
        return False

def run_with_profile(profile_path):
    """Run FastCopy using a YAML profile"""
    cmd = [
        "fastcopy",
        "--profile", profile_path,
        "--priority", "high"
    ]
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.returncode == 0

# Example usage
if __name__ == "__main__":
    source_dir = Path("/data/source")
    dest_dir = Path("/backup/destination")
    
    success = run_fastcopy(source_dir, dest_dir, streams=32, verify=True)
    sys.exit(0 if success else 1)
```

### PowerShell Script Example
```powershell
# FastCopy-Backup.ps1

param(
    [string]$Source,
    [string]$Destination,
    [int]$Streams = 16,
    [switch]$Verify
)

$ErrorActionPreference = "Stop"

$FastCopyArgs = @(
    $Source,
    $Destination,
    "--streams=$Streams",
    "--log-level=info"
)

if ($Verify) {
    $FastCopyArgs += "--verify"
}

try {
    Write-Host "Starting FastCopy transfer..."
    & fastcopy @FastCopyArgs
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "Transfer completed successfully" -ForegroundColor Green
    } else {
        throw "FastCopy failed with exit code $LASTEXITCODE"
    }
} catch {
    Write-Error "Transfer failed: $_"
    Send-MailMessage -To "admin@example.com" -From "fastcopy@example.com" `
        -Subject "FastCopy Failure Alert" -Body $_.Exception.Message `
        -SmtpServer "smtp.example.com"
    exit 1
}
```

## Common Patterns

### Pattern 1: Large Media Archive
```yaml
# media-archive.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: always

paths:
  source: "/mnt/media/raw_footage"
  destination: "/archive/2026/project_alpha"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"
    - "*.prores"
  min_size_mb: 10

hooks:
  on_complete: "/usr/local/bin/generate-checksum-report.sh"
```

### Pattern 2: Database Backup
```yaml
# db-backup.yaml
transfer:
  mode: "balanced"
  streams: 8
  buffer_size_mb: 256
  verify: true
  resume: on_failure

paths:
  source: "/var/lib/postgresql/backup"
  destination: "/backup/databases/$(date +%Y%m%d)"

schedule:
  type: "cron"
  cron: "0 3 * * *"  # 3 AM daily

hooks:
  on_start: "pg_dump -U postgres mydb > /var/lib/postgresql/backup/mydb.sql"
  on_complete: "find /backup/databases -mtime +7 -delete"
  on_error: "/opt/scripts/db-backup-alert.sh"
```

### Pattern 3: Development Sync
```yaml
# dev-sync.yaml
transfer:
  mode: "standard"
  streams: 16
  buffer_size_mb: 128
  verify: false  # Speed over verification for dev
  resume: always

paths:
  source: "/home/dev/project"
  destination: "/mnt/shared/dev-team/project"

schedule:
  type: "watch"
  interval_seconds: 60

filters:
  exclude:
    - "node_modules/*"
    - ".git/*"
    - "*.log"
    - ".DS_Store"
    - "*.pyc"
    - "__pycache__/*"

logging:
  level: "warn"
  file: "/tmp/fastcopy-dev-sync.log"
```

## Configuration Files

### Global Config Location
- **Windows**: `C:\ProgramData\FastCopy\config.yaml`
- **macOS**: `/Library/Application Support/FastCopy/config.yaml`
- **Linux**: `/etc/fastcopy/config.yaml` or `~/.config/fastcopy/config.yaml`

### Global Config Example
```yaml
# config.yaml (global defaults)
defaults:
  transfer:
    mode: "balanced"
    streams: 16
    buffer_size_mb: 128
    verify: true
    resume: always
  
  logging:
    level: "info"
    file: "/var/log/fastcopy.log"
    rotate: true
    max_size_mb: 50

profiles_directory: "/etc/fastcopy/profiles"

api:
  enable_rest: false
  port: 8080
  auth_token: "${FASTCOPY_API_TOKEN}"

notifications:
  email:
    smtp_server: "smtp.example.com"
    smtp_port: 587
    from: "fastcopy@example.com"
    to: "admin@example.com"
    username: "${SMTP_USER}"
    password: "${SMTP_PASSWORD}"
```

## Environment Variables

```bash
# API and integration credentials
export FASTCOPY_API_TOKEN="your-api-token-here"
export OPENAI_API_KEY="your-openai-key"
export ANTHROPIC_API_KEY="your-claude-key"

# Network credentials
export SSH_PRIVATE_KEY_PATH="/home/user/.ssh/id_rsa"
export REMOTE_USER="backup_user"

# Email notifications
export SMTP_USER="notifications@example.com"
export SMTP_PASSWORD="your-smtp-password"

# Profile location override
export FASTCOPY_PROFILE_DIR="/custom/profiles"

# Logging
export FASTCOPY_LOG_LEVEL="debug"
```

## Troubleshooting

### Issue: Transfer Stalls or Hangs
```bash
# Reduce stream count
fastcopy /source /dest --streams 8

# Increase timeout
fastcopy /source /dest --timeout 600

# Check logs
tail -f /var/log/fastcopy.log
```

### Issue: Verification Failures
```bash
# Run with detailed error logging
fastcopy /source /dest --verify --log-level debug

# Check filesystem integrity
fsck /dev/sda1  # Linux
chkdsk C: /F    # Windows

# Disable verification for speed (use with caution)
fastcopy /source /dest --no-verify
```

### Issue: Permission Errors
```bash
# Run with elevated privileges
sudo fastcopy /source /dest

# Check ownership and permissions
ls -la /source /dest

# Use explicit user context (Linux/macOS)
sudo -u targetuser fastcopy /source /dest
```

### Issue: Network Transfer Failures
```yaml
# Increase retry attempts in profile
transfer:
  retry_attempts: 10
  retry_backoff_seconds: 60
  timeout_seconds: 600

network:
  bandwidth_limit_mbps: 100  # Limit to prevent saturation
  keepalive: true
```

### Issue: High Memory Usage
```yaml
# Reduce buffer size and streams
transfer:
  streams: 8
  buffer_size_mb: 64
```

### Debug Mode
```bash
# Enable comprehensive debugging
fastcopy /source /dest --log-level debug --dry-run

# Monitor resource usage during transfer
# Linux/macOS
watch -n 1 'ps aux | grep fastcopy'

# Windows
Get-Process fastcopy | Format-Table CPU, Memory -AutoSize
```

## Performance Optimization

### Optimal Settings for Different Scenarios

**Local SSD to SSD:**
```yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
```

**Local HDD to HDD:**
```yaml
transfer:
  mode: "balanced"
  streams: 8
  buffer_size_mb: 128
```

**Network Transfer (1Gbps):**
```yaml
transfer:
  mode: "turbo"
  streams: 24
  buffer_size_mb: 512
  compression: true
```

**Network Transfer (10Gbps+):**
```yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 1024
  compression: false
```

## Security Considerations

- Always use environment variables for credentials, never hardcode
- Enable encryption for network transfers
- Use SSH keys instead of passwords where possible
- Regularly rotate API tokens and credentials
- Limit file permissions on config files: `chmod 600 config.yaml`
- Use verify mode for critical data transfers
- Implement logging and audit trails for compliance
