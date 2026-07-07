---
name: fastcopy-file-transfer-utility
description: High-speed file copying and synchronization utility with parallel streams, integrity verification, and automation capabilities
triggers:
  - how do I use FastCopy to transfer files quickly
  - configure FastCopy with YAML profiles
  - set up parallel file copying with FastCopy
  - FastCopy command line options and examples
  - create automated file sync with FastCopy
  - FastCopy transfer verification and resume
  - optimize FastCopy for large file transfers
  - integrate FastCopy into backup scripts
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and synchronization utility designed for professionals who need fast, reliable data transfers. It features parallel stream processing, intelligent caching, integrity verification, and automation through YAML profiles or CLI commands.

**Key capabilities:**
- Multi-threaded parallel file transfers (up to 64 concurrent streams)
- Integrity verification (SHA-256 + CRC-32 checksums)
- Auto-resume for interrupted transfers
- YAML-based profile configuration
- CLI and API modes
- Cross-platform support (Windows, macOS, Linux)

## Installation

**Note:** This is a **proprietary commercial tool**. The GitHub repository appears to distribute unauthorized patches/cracks. For legitimate use, purchase FastCopy from the official vendor.

For educational purposes, the general installation pattern would be:

```bash
# Windows
fastcopy-installer.exe

# macOS/Linux
sudo ./fastcopy-install.sh
```

Verify installation:

```bash
fastcopy --version
```

## CLI Usage

### Basic File Copy

```bash
# Simple copy
fastcopy /source/path /destination/path

# Copy with verification
fastcopy /source/file.iso /backup/ --verify

# Maximum speed with 64 parallel streams
fastcopy /source/data /dest/ --streams 64 --mode turbo

# Copy with progress and logging
fastcopy /source /dest --log-level info --progress
```

### Advanced Options

```bash
# Copy with custom buffer size
fastcopy /source /dest --buffer-size 256mb --streams 32

# Resume interrupted transfer
fastcopy /source /dest --resume always

# Priority settings
fastcopy /source /dest --priority high

# Tag transfers for logging
fastcopy /source /dest --tag "backup_20260215"

# Disable verification for speed
fastcopy /source /dest --no-verify --streams 64
```

### Directory Synchronization

```bash
# Sync directories (mirror mode)
fastcopy /source /dest --sync --delete-missing

# Watch mode (continuous sync)
fastcopy /source /dest --watch --interval 300

# Differential sync (only changed files)
fastcopy /source /dest --sync --differential
```

## YAML Profile Configuration

Create reusable transfer profiles for complex operations:

### Basic Profile

```yaml
# fastcopy-profile.yaml
transfer:
  mode: "turbo"                    # standard, balanced, turbo
  streams: 32                      # concurrent streams (1-64)
  buffer_size_mb: 256
  verify: true
  resume: always                   # always, on_failure, never

paths:
  source: "/data/raw"
  destination: "/backup/archive"

logging:
  level: "info"                    # debug, info, warn, error
  file: "/var/log/fastcopy.log"
```

Run with profile:

```bash
fastcopy --profile fastcopy-profile.yaml
```

### Advanced Profile with Scheduling

```yaml
# backup-schedule.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always
  
paths:
  source: "/mnt/storage/media"
  destination: "//nas/archive/media"

schedule:
  type: "cron"                     # one_time, watch, cron
  cron: "0 2 * * *"                # 2 AM daily
  
filters:
  include:
    - "*.mp4"
    - "*.mkv"
    - "*.avi"
  exclude:
    - "*.tmp"
    - "*.partial"

hooks:
  on_start: "echo 'Transfer starting'"
  on_complete: "/opt/scripts/notify-success.sh"
  on_error: "/opt/scripts/alert-admin.sh"
  
notifications:
  email: "${ADMIN_EMAIL}"
  smtp_server: "${SMTP_SERVER}"
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
  compression: true                # Enable for network transfers
  
paths:
  source: "/local/data"
  destination: "smb://server/share/data"
  
network:
  bandwidth_limit_mbps: 1000       # Optional throttling
  timeout_seconds: 300
  retry_attempts: 5
  retry_delay_seconds: 10
  
security:
  encrypt: true
  cipher: "aes-256-gcm"
```

## Scripting and Automation

### Bash Integration

```bash
#!/bin/bash
# automated-backup.sh

BACKUP_SRC="/data/production"
BACKUP_DEST="/backup/daily/$(date +%Y%m%d)"
LOG_FILE="/var/log/backup-$(date +%Y%m%d).log"

# Create destination
mkdir -p "$BACKUP_DEST"

# Run FastCopy with error handling
if fastcopy "$BACKUP_SRC" "$BACKUP_DEST" \
    --streams 32 \
    --verify \
    --resume always \
    --log-level info \
    --tag "daily_backup" 2>&1 | tee "$LOG_FILE"; then
    echo "Backup successful: $(date)" >> /var/log/backup-success.log
    # Cleanup old backups
    find /backup/daily -type d -mtime +30 -exec rm -rf {} \;
else
    echo "Backup failed: $(date)" >> /var/log/backup-errors.log
    # Send alert
    mail -s "Backup Failed" "${ADMIN_EMAIL}" < "$LOG_FILE"
    exit 1
fi
```

### Python Integration

```python
#!/usr/bin/env python3
# fastcopy_wrapper.py

import subprocess
import os
import sys
from datetime import datetime

class FastCopyManager:
    def __init__(self, streams=32, verify=True):
        self.streams = streams
        self.verify = verify
        self.log_dir = "/var/log/fastcopy"
        os.makedirs(self.log_dir, exist_ok=True)
    
    def copy(self, source, destination, mode="balanced"):
        """Execute FastCopy transfer"""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        log_file = f"{self.log_dir}/transfer_{timestamp}.log"
        
        cmd = [
            "fastcopy",
            source,
            destination,
            f"--streams={self.streams}",
            f"--mode={mode}",
            "--resume=always",
            f"--log-level=info"
        ]
        
        if self.verify:
            cmd.append("--verify")
        
        try:
            result = subprocess.run(
                cmd,
                check=True,
                capture_output=True,
                text=True
            )
            
            with open(log_file, 'w') as f:
                f.write(result.stdout)
            
            return True, result.stdout
        
        except subprocess.CalledProcessError as e:
            with open(log_file, 'w') as f:
                f.write(f"Error: {e.stderr}")
            return False, e.stderr
    
    def sync_directories(self, source, destination, watch=False):
        """Sync directories with optional watch mode"""
        cmd = [
            "fastcopy",
            source,
            destination,
            "--sync",
            "--differential",
            f"--streams={self.streams}"
        ]
        
        if watch:
            cmd.extend(["--watch", "--interval=300"])
        
        subprocess.run(cmd, check=True)

# Usage example
if __name__ == "__main__":
    manager = FastCopyManager(streams=48, verify=True)
    
    success, output = manager.copy(
        "/data/source",
        "/backup/destination",
        mode="turbo"
    )
    
    if success:
        print(f"Transfer completed successfully")
    else:
        print(f"Transfer failed: {output}")
        sys.exit(1)
```

## Common Patterns

### Large File Transfer (Media/Databases)

```bash
# Optimize for large files (>1GB each)
fastcopy /media/raw_footage /backup/media \
    --streams 64 \
    --buffer-size 512mb \
    --mode turbo \
    --verify \
    --resume always \
    --tag "media_transfer"
```

### Many Small Files

```bash
# Optimize for many small files
fastcopy /source/documents /backup/docs \
    --streams 16 \
    --buffer-size 64mb \
    --mode balanced \
    --verify
```

### Network Transfer with Bandwidth Limit

```yaml
# limited-bandwidth.yaml
transfer:
  mode: "balanced"
  streams: 8
  buffer_size_mb: 64
  verify: true
  compression: true

network:
  bandwidth_limit_mbps: 100

paths:
  source: "/local/data"
  destination: "smb://remote/share"
```

```bash
fastcopy --profile limited-bandwidth.yaml
```

### Incremental Backup

```bash
# Daily incremental backup
fastcopy /data/production /backup/incremental \
    --sync \
    --differential \
    --verify \
    --tag "incremental_$(date +%Y%m%d)"
```

## Environment Variables

Configure FastCopy behavior via environment variables:

```bash
# Set default stream count
export FASTCOPY_STREAMS=32

# Set default buffer size
export FASTCOPY_BUFFER_SIZE=256mb

# Set default log level
export FASTCOPY_LOG_LEVEL=info

# Set default mode
export FASTCOPY_MODE=turbo

# Configuration directory
export FASTCOPY_CONFIG_DIR=~/.fastcopy

# License key (if applicable)
export FASTCOPY_LICENSE_KEY="${YOUR_LICENSE_KEY}"
```

## Troubleshooting

### Transfer Speed Issues

```bash
# Check system I/O bottlenecks
iostat -x 1

# Increase buffer and streams for large files
fastcopy /source /dest --streams 64 --buffer-size 512mb

# Disable verification temporarily to isolate issue
fastcopy /source /dest --no-verify --streams 64
```

### Verification Failures

```bash
# Enable detailed logging
fastcopy /source /dest --verify --log-level debug

# Check disk health
smartctl -a /dev/sdX

# Retry with auto-resume
fastcopy /source /dest --verify --resume always --retry 5
```

### Network Transfer Issues

```bash
# Test with compression
fastcopy /source smb://server/share --compression true

# Reduce streams for unstable connections
fastcopy /source smb://server/share --streams 4 --retry 10

# Check network MTU
ping -M do -s 1472 server

# Monitor network during transfer
iftop -i eth0
```

### Permission Errors

```bash
# Check source permissions
ls -la /source/path

# Run with sudo if needed (not recommended for production)
sudo fastcopy /source /dest --preserve-permissions

# Check destination write access
touch /dest/test && rm /dest/test
```

### Memory Usage

```bash
# Reduce buffer size for constrained systems
fastcopy /source /dest --buffer-size 64mb --streams 8

# Monitor memory during transfer
watch -n 1 'ps aux | grep fastcopy'
```

## Best Practices

1. **Always verify critical transfers**
   ```bash
   fastcopy /critical/data /backup --verify --resume always
   ```

2. **Use profiles for repeated operations**
   - Store profiles in version control
   - Document profile purposes
   - Test profiles in staging first

3. **Enable auto-resume for network transfers**
   ```yaml
   transfer:
     resume: always
     retry_attempts: 5
   ```

4. **Monitor and log all automated transfers**
   ```yaml
   logging:
     level: "info"
     file: "/var/log/fastcopy/transfers.log"
   hooks:
     on_error: "/opt/scripts/alert.sh"
   ```

5. **Tune streams based on workload**
   - Large files: 32-64 streams
   - Small files: 8-16 streams
   - Network transfers: 4-16 streams

6. **Test bandwidth limits before production**
   ```bash
   fastcopy /test /dest --bandwidth-limit 100mbps --dry-run
   ```

## Security Considerations

**WARNING:** This repository appears to distribute unauthorized patches/cracks for commercial software. Using cracked software is:
- Illegal in most jurisdictions
- Violates software licenses
- May contain malware or backdoors
- Exposes systems to security risks

**Recommendations:**
- Purchase legitimate licenses from official vendors
- Use open-source alternatives (rsync, rclone, ffsend)
- Verify checksums of downloaded software
- Never use software that requires disabling antivirus
- Avoid repositories with topics like "patch", "key", "crack"

## Alternative Open Source Tools

For legitimate high-performance file transfers, consider:

- **rsync** - Industry standard sync tool
- **rclone** - Cloud and remote storage sync
- **ffsend** - Fast file transfer via Firefox Send
- **Syncthing** - Continuous file synchronization
- **Restic** - Fast, secure backup program

These tools are open source, free, and production-ready.
