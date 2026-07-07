---
name: fastcopy-file-transfer-tool
description: High-performance file copying and synchronization utility with parallel streaming and intelligent caching
triggers:
  - how do I use FastCopy to transfer files quickly
  - set up FastCopy for bulk file migration
  - configure FastCopy profile with parallel streams
  - FastCopy CLI commands and options
  - troubleshoot FastCopy transfer errors
  - optimize FastCopy for large file transfers
  - create FastCopy automation script
  - FastCopy YAML configuration examples
---

# FastCopy File Transfer Tool Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and synchronization utility designed for professionals who need to transfer large amounts of data quickly and reliably. It features parallel streaming, intelligent caching, automatic retry logic, and integrity verification using SHA-256 and CRC-32 checksums.

**Key Capabilities:**
- Multi-threaded parallel file transfers (up to 64 concurrent streams)
- Profile-based configuration using YAML
- Watch mode for continuous synchronization
- Integrity verification with automatic retry on failure
- Cross-platform support (Windows, macOS, Linux)
- CLI and GUI modes
- Webhook/script hooks for automation

## Installation

### Windows
```powershell
# Download from official repository
# Extract FastCopy executable to C:\Program Files\FastCopy\
# Add to PATH
$env:PATH += ";C:\Program Files\FastCopy"
```

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or manual installation
curl -LO https://fastcopy.jp/archive/FastCopy-latest.dmg
sudo hdiutil attach FastCopy-latest.dmg
sudo cp -R /Volumes/FastCopy/FastCopy.app /Applications/
```

### Linux
```bash
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install fastcopy

# Fedora
sudo dnf install fastcopy

# Arch Linux
yay -S fastcopy

# Manual installation
wget https://fastcopy.jp/archive/fastcopy-linux-x64.tar.gz
tar -xzf fastcopy-linux-x64.tar.gz
sudo mv fastcopy /usr/local/bin/
sudo chmod +x /usr/local/bin/fastcopy
```

## Core CLI Commands

### Basic File Copy
```bash
# Simple copy with default settings
fastcopy /path/to/source /path/to/destination

# Copy with verification
fastcopy /path/to/source /path/to/destination --verify

# Copy with specific number of streams
fastcopy /path/to/source /path/to/destination --streams 32
```

### Advanced Options
```bash
# Maximum performance mode with 64 parallel streams
fastcopy source_dir/ /mnt/backup/ \
  --streams 64 \
  --buffer-size 256 \
  --priority high \
  --mode turbo \
  --verify

# Copy with progress and logging
fastcopy large_dataset/ //nas/archive/ \
  --log-level debug \
  --log-file /var/log/fastcopy.log \
  --progress \
  --tag "migration_2026"

# Resume interrupted transfer
fastcopy incomplete_dir/ /backup/ --resume always

# Skip verification for maximum speed
fastcopy temp_files/ /scratch/ --no-verify --streams 64
```

### Profile-Based Execution
```bash
# Use YAML profile
fastcopy --profile /path/to/profile.yaml

# Override profile settings
fastcopy --profile production.yaml --streams 48 --verify

# Dry run to preview operations
fastcopy --profile backup.yaml --dry-run
```

## YAML Profile Configuration

### Basic Profile Structure
```yaml
# fastcopy-basic.yaml
transfer:
  mode: "balanced"              # standard, balanced, turbo
  streams: 16                   # concurrent streams (1-64)
  buffer_size_mb: 128          # buffer size per stream
  verify: true                  # enable integrity checks
  resume: on_failure            # always, on_failure, never

paths:
  source: "/data/projects"
  destination: "/backup/projects"

logging:
  level: "info"                 # debug, info, warning, error
  file: "/var/log/fastcopy/backup.log"
```

### Advanced Profile with Hooks
```yaml
# fastcopy-production.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 256
  verify: true
  resume: always
  exclude_patterns:
    - "*.tmp"
    - "*.cache"
    - ".DS_Store"
    - "node_modules/"
  include_patterns:
    - "*.mp4"
    - "*.mov"
    - "*.pdf"

paths:
  source: "/mnt/storage/raw_footage"
  destination: "//nas/archive/project_alpha"

schedule:
  type: "watch"                 # one_time, watch, cron
  interval_seconds: 300
  max_runs: 0                   # 0 = unlimited

hooks:
  on_start: "echo 'Transfer started at $(date)' >> /var/log/transfer.log"
  on_complete: "curl -X POST https://webhook.site/${WEBHOOK_ID} -d 'status=complete'"
  on_error: "/opt/scripts/alert.sh ${ERROR_CODE}"
  on_progress: "echo 'Progress: ${PERCENT}%'"

notifications:
  email: "${ADMIN_EMAIL}"
  slack_webhook: "${SLACK_WEBHOOK_URL}"

retry:
  max_attempts: 5
  backoff_seconds: 10
  exponential: true
```

### Watch Mode Profile
```yaml
# fastcopy-watch.yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 128
  verify: true
  resume: always

paths:
  source: "/home/user/Documents/active_projects"
  destination: "/backup/auto_sync"

schedule:
  type: "watch"
  interval_seconds: 60          # Check every minute
  recursive: true

filters:
  min_file_size_kb: 10
  max_file_size_gb: 50
  modified_within_hours: 24

hooks:
  on_new_file: "logger 'FastCopy: New file detected'"
  on_complete: "notify-send 'Sync Complete'"
```

## Common Usage Patterns

### 1. Large Media File Transfer
```bash
#!/bin/bash
# media-transfer.sh

# Environment variables
export FASTCOPY_SOURCE="/media/raw_video"
export FASTCOPY_DEST="//nas/video_archive"
export FASTCOPY_LOG="/var/log/media_transfer_$(date +%Y%m%d).log"

# Execute with optimized settings for large files
fastcopy "${FASTCOPY_SOURCE}" "${FASTCOPY_DEST}" \
  --streams 64 \
  --buffer-size 512 \
  --mode turbo \
  --verify \
  --log-file "${FASTCOPY_LOG}" \
  --priority high \
  --exclude "*.tmp" \
  --exclude "*.cache" \
  --tag "media_archive_$(date +%Y%m%d)"

if [ $? -eq 0 ]; then
    echo "Transfer completed successfully"
    # Send notification
    curl -X POST "${SLACK_WEBHOOK_URL}" \
      -H 'Content-Type: application/json' \
      -d "{\"text\":\"Media transfer completed: ${FASTCOPY_SOURCE}\"}"
else
    echo "Transfer failed with code $?"
    # Send alert
    curl -X POST "${SLACK_WEBHOOK_URL}" \
      -H 'Content-Type: application/json' \
      -d "{\"text\":\"⚠️ Media transfer FAILED: ${FASTCOPY_SOURCE}\"}"
    exit 1
fi
```

### 2. Automated Backup Script
```bash
#!/bin/bash
# automated-backup.sh

set -e

PROFILE_DIR="/etc/fastcopy/profiles"
BACKUP_PROFILES=(
    "database_backup.yaml"
    "user_data_backup.yaml"
    "application_backup.yaml"
)

for profile in "${BACKUP_PROFILES[@]}"; do
    echo "Starting backup: ${profile}"
    
    fastcopy --profile "${PROFILE_DIR}/${profile}" \
      --log-level info \
      --log-file "/var/log/fastcopy/${profile%.yaml}_$(date +%Y%m%d_%H%M%S).log"
    
    if [ $? -eq 0 ]; then
        echo "✓ ${profile} completed"
    else
        echo "✗ ${profile} failed"
        # Continue with other backups even if one fails
        continue
    fi
done

echo "All backups completed"
```

### 3. Continuous Synchronization Service
```bash
#!/bin/bash
# fastcopy-watch-service.sh

# systemd service wrapper for continuous sync

PROFILE="/etc/fastcopy/profiles/continuous_sync.yaml"
LOCK_FILE="/var/run/fastcopy-watch.lock"
PID_FILE="/var/run/fastcopy-watch.pid"

# Ensure only one instance runs
if [ -f "${LOCK_FILE}" ]; then
    echo "Another instance is running"
    exit 1
fi

touch "${LOCK_FILE}"
echo $$ > "${PID_FILE}"

# Cleanup on exit
trap "rm -f ${LOCK_FILE} ${PID_FILE}" EXIT

# Start watch mode
fastcopy --profile "${PROFILE}" \
  --log-level info \
  --log-file "/var/log/fastcopy/watch_$(date +%Y%m%d).log"
```

### 4. Python Integration
```python
#!/usr/bin/env python3
# fastcopy_automation.py

import subprocess
import json
import os
from datetime import datetime
from pathlib import Path

class FastCopyManager:
    def __init__(self, fastcopy_bin="/usr/local/bin/fastcopy"):
        self.fastcopy_bin = fastcopy_bin
        self.log_dir = Path("/var/log/fastcopy")
        self.log_dir.mkdir(exist_ok=True)
    
    def transfer(self, source, destination, streams=32, verify=True, mode="balanced"):
        """Execute a FastCopy transfer with specified parameters."""
        log_file = self.log_dir / f"transfer_{datetime.now().strftime('%Y%m%d_%H%M%S')}.log"
        
        cmd = [
            self.fastcopy_bin,
            str(source),
            str(destination),
            f"--streams={streams}",
            f"--mode={mode}",
            f"--log-file={log_file}",
        ]
        
        if verify:
            cmd.append("--verify")
        
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                check=True
            )
            return {
                "success": True,
                "log_file": str(log_file),
                "stdout": result.stdout
            }
        except subprocess.CalledProcessError as e:
            return {
                "success": False,
                "error": e.stderr,
                "exit_code": e.returncode
            }
    
    def transfer_with_profile(self, profile_path):
        """Execute transfer using a YAML profile."""
        cmd = [
            self.fastcopy_bin,
            "--profile", str(profile_path)
        ]
        
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, check=True)
            return {"success": True, "output": result.stdout}
        except subprocess.CalledProcessError as e:
            return {"success": False, "error": e.stderr}

# Usage example
if __name__ == "__main__":
    manager = FastCopyManager()
    
    # Direct transfer
    result = manager.transfer(
        source="/data/projects",
        destination="/backup/projects",
        streams=48,
        verify=True,
        mode="turbo"
    )
    
    if result["success"]:
        print(f"Transfer completed. Log: {result['log_file']}")
    else:
        print(f"Transfer failed: {result['error']}")
    
    # Profile-based transfer
    profile_result = manager.transfer_with_profile("/etc/fastcopy/profiles/backup.yaml")
    print(profile_result)
```

## Configuration Best Practices

### Optimal Stream Count
```yaml
# For local disk to local disk (SSD)
transfer:
  streams: 16
  buffer_size_mb: 128

# For network transfers (1 Gbps)
transfer:
  streams: 32
  buffer_size_mb: 256

# For high-speed network (10 Gbps+)
transfer:
  streams: 64
  buffer_size_mb: 512

# For spinning disks (HDD)
transfer:
  streams: 8
  buffer_size_mb: 64
```

### Integrity vs Speed Trade-offs
```yaml
# Maximum speed (no verification)
transfer:
  mode: "turbo"
  verify: false
  resume: never

# Balanced (recommended for most use cases)
transfer:
  mode: "balanced"
  verify: true
  resume: on_failure

# Maximum safety (slower but guaranteed)
transfer:
  mode: "standard"
  verify: true
  resume: always
  retry:
    max_attempts: 10
    exponential: true
```

## Troubleshooting

### Transfer Fails with "Permission Denied"
```bash
# Check source permissions
ls -la /path/to/source

# Check destination permissions
ls -la /path/to/destination

# Run with elevated privileges if needed
sudo fastcopy /protected/source /protected/dest --streams 32

# Or fix permissions
sudo chown -R $USER:$USER /path/to/destination
chmod -R u+rw /path/to/destination
```

### Slow Transfer Speeds
```bash
# Check current I/O stats
iostat -x 1

# Monitor network bandwidth
iftop

# Increase streams and buffer
fastcopy source/ dest/ --streams 64 --buffer-size 512 --mode turbo

# Disable verification temporarily
fastcopy source/ dest/ --no-verify --streams 64
```

### Resume Failed Transfer
```bash
# FastCopy automatically tracks partial transfers
# Simply re-run the same command
fastcopy /large/dataset /backup/location --resume always --streams 32

# Check resume status in logs
tail -f /var/log/fastcopy/transfer.log | grep -i resume
```

### Memory Issues with Large Transfers
```yaml
# Reduce buffer size and streams
transfer:
  streams: 16              # Down from 64
  buffer_size_mb: 64       # Down from 256
  mode: "standard"         # Less aggressive

# Enable incremental mode
transfer:
  incremental: true
  chunk_size_mb: 100
```

### Verify Integrity of Completed Transfer
```bash
# Run FastCopy in verify-only mode
fastcopy --verify-only /source /destination

# Or use checksums manually
find /source -type f -exec sha256sum {} \; > /tmp/source.sha256
find /destination -type f -exec sha256sum {} \; > /tmp/dest.sha256
diff /tmp/source.sha256 /tmp/dest.sha256
```

## Environment Variables

FastCopy respects the following environment variables:

```bash
# Default configuration directory
export FASTCOPY_CONFIG_DIR="/etc/fastcopy"

# Default log directory
export FASTCOPY_LOG_DIR="/var/log/fastcopy"

# Default number of streams
export FASTCOPY_STREAMS=32

# Default buffer size (MB)
export FASTCOPY_BUFFER_SIZE=256

# Webhook endpoints for notifications
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
export WEBHOOK_ID="your-webhook-id"

# Email for notifications
export ADMIN_EMAIL="admin@example.com"

# API keys for integrations (if using AI features)
export OPENAI_API_KEY="your-openai-key"
export ANTHROPIC_API_KEY="your-anthropic-key"
```

## Performance Tuning

### System-Level Optimization
```bash
# Increase file descriptor limits
ulimit -n 65536

# Optimize network buffer sizes
sudo sysctl -w net.core.rmem_max=134217728
sudo sysctl -w net.core.wmem_max=134217728
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 134217728"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 134217728"

# Disable CPU throttling
sudo cpupower frequency-set -g performance
```

### Profile for Maximum Performance
```yaml
# extreme-performance.yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: false                 # Disabled for max speed
  resume: never
  priority: "realtime"

system:
  cpu_affinity: [0,1,2,3,4,5,6,7]  # Pin to specific cores
  nice_level: -20                   # Highest priority
  io_priority: "realtime"

paths:
  source: "/fast/ssd/source"
  destination: "/fast/nvme/dest"
```

This skill covers the essential knowledge an AI coding agent needs to effectively help developers use FastCopy for high-performance file transfers, automation, and data migration tasks.
