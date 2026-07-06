---
name: fastcopy-file-transfer-utility
description: High-performance file copying and synchronization tool with multi-threaded transfers, integrity verification, and automation capabilities
triggers:
  - "how do I use FastCopy to copy files"
  - "configure FastCopy for bulk file transfers"
  - "set up FastCopy profile for automated sync"
  - "FastCopy command line options"
  - "optimize FastCopy transfer speed"
  - "FastCopy YAML configuration"
  - "verify file integrity with FastCopy"
  - "schedule FastCopy operations"
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and synchronization utility designed for professionals who need fast, reliable data transfers. It features multi-threaded I/O, intelligent caching, integrity verification, and automation through profiles.

**Key capabilities:**
- Parallel stream transfers (up to 64 concurrent streams)
- YAML-based profile configuration
- CLI and GUI modes
- Integrity verification (SHA-256 + CRC-32)
- Auto-resume on interruption
- Watch mode for continuous sync
- Cross-platform (Windows, macOS, Linux)

## Installation

### Windows
```bash
# Download from official source
# Extract to desired location
# Add to PATH (optional)
set PATH=%PATH%;C:\Program Files\FastCopy
```

### macOS
```bash
brew install fastcopy
# Or download and install from official site
```

### Linux
```bash
# Ubuntu/Debian
sudo apt-get install fastcopy

# Fedora
sudo dnf install fastcopy

# Arch
yay -S fastcopy
```

## CLI Usage

### Basic Command Structure
```bash
fastcopy [source] [destination] [options]
```

### Key Commands and Options

**Simple copy:**
```bash
fastcopy /source/folder /destination/folder
```

**High-speed transfer with max streams:**
```bash
fastcopy /source/data /backup/data --streams 64 --priority high
```

**With integrity verification:**
```bash
fastcopy source.iso /backups/ --verify --checksum sha256
```

**Using a profile:**
```bash
fastcopy --profile /path/to/profile.yaml
```

**Watch mode for continuous sync:**
```bash
fastcopy /source /dest --mode watch --interval 300
```

**Dry run to preview:**
```bash
fastcopy /source /dest --dry-run --verbose
```

**Common Options:**
- `--streams N` - Number of concurrent transfer streams (1-64)
- `--buffer-size N` - Buffer size in MB (default: 128)
- `--verify` - Enable integrity checking
- `--resume` - Auto-resume interrupted transfers
- `--priority [low|normal|high]` - Transfer priority
- `--log-level [debug|info|warn|error]` - Logging verbosity
- `--no-verify` - Skip integrity checks for speed
- `--tag "label"` - Add tag to transfer operation

## YAML Profile Configuration

FastCopy uses YAML profiles for declarative, reusable transfer configurations.

### Basic Profile Example
```yaml
# basic-backup.yaml
transfer:
  mode: "turbo"              # standard, balanced, turbo
  streams: 32                # concurrent streams (max 64)
  buffer_size_mb: 256
  verify: true
  resume: always             # always, on_failure, never
  checksum: "sha256"         # sha256, crc32, both

paths:
  source: "/data/project"
  destination: "/backups/project"

schedule:
  type: "one_time"           # one_time, watch, cron

hooks:
  on_complete: "echo 'Backup complete'"
  on_error: "/scripts/alert.sh"
```

### Advanced Profile with Watch Mode
```yaml
# continuous-sync.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always
  exclude_patterns:
    - "*.tmp"
    - "*.cache"
    - ".git/*"

paths:
  source: "/mnt/storage/raw_footage"
  destination: "//nas/archive/project_alpha"

schedule:
  type: "watch"
  interval_seconds: 300
  max_retries: 5
  retry_delay_seconds: 60

filters:
  file_types: ["*.mp4", "*.mov", "*.avi"]
  min_size_mb: 10
  modified_within_days: 7

hooks:
  on_start: "logger 'FastCopy sync started'"
  on_complete: "curl -X POST $WEBHOOK_URL -d '{\"status\":\"complete\"}'"
  on_error: "/opt/scripts/alert.sh"

logging:
  level: "info"
  file: "/var/log/fastcopy/sync.log"
  rotate: true
  max_size_mb: 100
```

### Scheduled Backup Profile
```yaml
# nightly-backup.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: always

paths:
  source: "/database/backups"
  destination: "/mnt/remote/db-backups"

schedule:
  type: "cron"
  expression: "0 2 * * *"    # 2 AM daily
  timezone: "UTC"

retention:
  keep_days: 30
  cleanup_old: true

notifications:
  email: "$ADMIN_EMAIL"
  slack_webhook: "$SLACK_WEBHOOK"

hooks:
  on_complete: "/scripts/verify-backup.sh"
  on_error: "/scripts/emergency-alert.sh"
```

## Common Usage Patterns

### Pattern 1: Quick Ad-Hoc Copy
```bash
# Fast single file with no verification
fastcopy large-file.zip /backups/ --streams 64 --no-verify

# Directory with progress
fastcopy /source /dest --verbose --progress
```

### Pattern 2: Verified Backup
```bash
# Create profile for repeatable backups
cat > verified-backup.yaml << 'EOF'
transfer:
  mode: "balanced"
  streams: 24
  verify: true
  checksum: "both"
paths:
  source: "/data"
  destination: "/backups"
logging:
  level: "info"
  file: "./backup.log"
EOF

# Execute
fastcopy --profile verified-backup.yaml
```

### Pattern 3: Continuous Sync
```bash
# Create watch profile
cat > sync-profile.yaml << 'EOF'
transfer:
  mode: "standard"
  streams: 16
  verify: true
paths:
  source: "/work/documents"
  destination: "/sync/documents"
schedule:
  type: "watch"
  interval_seconds: 600
hooks:
  on_complete: "echo $(date) >> sync.log"
EOF

# Run in background
nohup fastcopy --profile sync-profile.yaml &
```

### Pattern 4: Network Transfer with Retry
```bash
fastcopy /local/data //remote/share/data \
  --streams 32 \
  --resume always \
  --max-retries 10 \
  --retry-delay 30 \
  --verify \
  --log-level debug \
  --log-file transfer.log
```

### Pattern 5: Selective Sync with Filters
```yaml
# selective-sync.yaml
transfer:
  mode: "turbo"
  streams: 24
paths:
  source: "/media"
  destination: "/archive"
filters:
  include_patterns:
    - "*.mp4"
    - "*.mkv"
    - "*.avi"
  exclude_patterns:
    - "*_draft.*"
    - "*.tmp"
  min_size_mb: 100
  max_size_gb: 50
```

```bash
fastcopy --profile selective-sync.yaml
```

## Configuration Files

### Global Configuration
Location: `~/.fastcopy/config.yaml` or `/etc/fastcopy/config.yaml`

```yaml
# Global defaults
defaults:
  streams: 16
  buffer_size_mb: 128
  verify: true
  log_level: "info"

ui:
  theme: "dark"
  show_progress: true
  confirm_overwrites: true

performance:
  max_cpu_percent: 80
  max_memory_mb: 4096
  io_priority: "normal"

profiles_directory: "~/.fastcopy/profiles"
logs_directory: "~/.fastcopy/logs"
```

### Environment Variables
```bash
export FASTCOPY_CONFIG="~/.fastcopy/config.yaml"
export FASTCOPY_PROFILES_DIR="~/fastcopy-profiles"
export FASTCOPY_LOG_LEVEL="debug"
export FASTCOPY_MAX_STREAMS="32"
```

## Script Integration

### Bash Script Example
```bash
#!/bin/bash
# backup-script.sh

set -e

SOURCE="/data/important"
DEST="/backups/$(date +%Y%m%d)"
PROFILE="/etc/fastcopy/backup.yaml"

echo "Starting backup: $SOURCE -> $DEST"

if fastcopy --profile "$PROFILE" \
     --set paths.source="$SOURCE" \
     --set paths.destination="$DEST" \
     --log-file "backup-$(date +%Y%m%d).log"; then
    echo "Backup completed successfully"
    # Notify success
    curl -X POST "$WEBHOOK_URL" -d '{"status":"success"}'
else
    echo "Backup failed"
    # Send alert
    curl -X POST "$WEBHOOK_URL" -d '{"status":"failed"}'
    exit 1
fi
```

### Python Integration
```python
#!/usr/bin/env python3
import subprocess
import json
from datetime import datetime

def run_fastcopy(source, dest, profile=None, streams=32):
    """Execute FastCopy with parameters"""
    cmd = ["fastcopy", source, dest, "--streams", str(streams)]
    
    if profile:
        cmd.extend(["--profile", profile])
    
    try:
        result = subprocess.run(
            cmd,
            check=True,
            capture_output=True,
            text=True
        )
        print(f"Transfer complete: {result.stdout}")
        return True
    except subprocess.CalledProcessError as e:
        print(f"Transfer failed: {e.stderr}")
        return False

def sync_with_monitoring(source, dest):
    """Run sync with status monitoring"""
    profile_config = {
        "transfer": {
            "mode": "turbo",
            "streams": 48,
            "verify": True
        },
        "paths": {
            "source": source,
            "destination": dest
        },
        "hooks": {
            "on_complete": "echo 'Done'",
            "on_error": "echo 'Failed'"
        }
    }
    
    # Write temporary profile
    profile_path = "/tmp/fastcopy-profile.yaml"
    with open(profile_path, "w") as f:
        import yaml
        yaml.dump(profile_config, f)
    
    return run_fastcopy(source, dest, profile=profile_path)

if __name__ == "__main__":
    sync_with_monitoring("/data/source", "/data/backup")
```

## Troubleshooting

### Issue: Slow Transfer Speeds
```bash
# Check system resources
fastcopy /src /dst --streams 64 --log-level debug --verbose

# Solutions:
# 1. Increase streams
fastcopy /src /dst --streams 64

# 2. Increase buffer size
fastcopy /src /dst --buffer-size 512

# 3. Disable verification for speed
fastcopy /src /dst --no-verify

# 4. Use turbo mode in profile
# transfer:
#   mode: "turbo"
```

### Issue: Transfer Fails Midway
```bash
# Enable auto-resume
fastcopy /src /dst --resume always --max-retries 10

# Check logs
fastcopy /src /dst --log-level debug --log-file debug.log
cat debug.log
```

### Issue: Permission Errors
```bash
# Check permissions on source and destination
ls -la /source
ls -la /destination

# Run with appropriate privileges
sudo fastcopy /protected/source /protected/dest

# Or fix permissions
chmod -R u+rwx /source
```

### Issue: Network Timeouts
```yaml
# network-resilient.yaml
transfer:
  mode: "balanced"
  streams: 16
  resume: always
  timeout_seconds: 300
  max_retries: 20
  retry_delay_seconds: 60
  verify: true
```

### Issue: Memory Usage Too High
```yaml
# Configure memory limits
performance:
  max_memory_mb: 2048
  buffer_size_mb: 64
  streams: 16  # Reduce concurrent streams
```

### Issue: Profile Not Loading
```bash
# Validate YAML syntax
fastcopy --validate-profile myprofile.yaml

# Check for typos
cat myprofile.yaml

# Use absolute paths
fastcopy --profile /full/path/to/profile.yaml
```

## Advanced Features

### Integrity Verification
```bash
# SHA-256 only
fastcopy /src /dst --verify --checksum sha256

# CRC-32 (faster)
fastcopy /src /dst --verify --checksum crc32

# Both (most thorough)
fastcopy /src /dst --verify --checksum both
```

### Excluding Files
```yaml
transfer:
  exclude_patterns:
    - "*.tmp"
    - "*.cache"
    - ".git/*"
    - "node_modules/*"
    - "__pycache__/*"
```

### Rate Limiting
```yaml
transfer:
  max_bandwidth_mbps: 100
  throttle_cpu_percent: 50
```

### Hooks for Automation
```yaml
hooks:
  on_start: "notify-send 'Transfer started'"
  on_complete: "python /scripts/post-process.py"
  on_error: "mail -s 'Transfer Failed' admin@example.com"
  on_progress: "echo $PROGRESS% >> status.txt"
```

## Best Practices

1. **Always verify critical data**: Use `--verify` for important transfers
2. **Use profiles for repeatability**: Store common operations as YAML profiles
3. **Enable auto-resume**: Set `resume: always` for unreliable connections
4. **Log transfers**: Use `--log-file` for audit trails
5. **Test with dry-run**: Use `--dry-run` before large operations
6. **Monitor resources**: Check system load when using high stream counts
7. **Use appropriate modes**: `turbo` for speed, `balanced` for stability
8. **Set up notifications**: Configure hooks for long-running operations
