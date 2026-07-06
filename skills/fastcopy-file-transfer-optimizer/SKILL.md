---
name: fastcopy-file-transfer-optimizer
description: High-performance file copying and synchronization utility with parallel streams and intelligent caching
triggers:
  - how do I use fastcopy to copy files faster
  - configure fastcopy for bulk file transfers
  - set up fastcopy with multiple parallel streams
  - create a fastcopy profile for automated backups
  - fastcopy command line options and examples
  - optimize file transfer speed with fastcopy
  - fastcopy yaml configuration for scheduled copies
  - troubleshoot fastcopy transfer failures
---

# FastCopy File Transfer Optimizer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and synchronization tool designed for scenarios requiring fast, reliable data transfers. It supports multi-threaded I/O, intelligent caching, integrity verification, and automated retry mechanisms. Available on Windows, macOS, and Linux.

**Key capabilities:**
- Parallel stream transfers (up to 64 concurrent streams)
- YAML-based profile system for reusable configurations
- Integrity verification with SHA-256 and CRC-32 checksums
- Auto-resume on interrupted transfers
- Watch mode for continuous synchronization
- CLI and profile-based operation modes

## Installation

### Windows
Download from the official repository or use the provided installer. No additional dependencies required.

### macOS
```bash
brew install fastcopy
```

### Linux (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install fastcopy
```

### Linux (Arch)
```bash
yay -S fastcopy
```

### Verify Installation
```bash
fastcopy --version
```

## CLI Usage

### Basic Command Structure
```bash
fastcopy <source> <destination> [options]
```

### Common Commands

**Simple copy:**
```bash
fastcopy /source/path /destination/path
```

**High-speed copy with maximum streams:**
```bash
fastcopy /source/path /destination/path --streams 64 --priority high
```

**Copy with verification:**
```bash
fastcopy /source/path /destination/path --verify
```

**Copy without verification (fastest):**
```bash
fastcopy source.iso /backups/ --no-verify
```

**Using a profile:**
```bash
fastcopy --profile transfer-config.yaml
```

**Watch mode for continuous sync:**
```bash
fastcopy --profile watch-profile.yaml --daemon
```

**Dry run (preview without copying):**
```bash
fastcopy /source /dest --dry-run
```

**Set log level:**
```bash
fastcopy /source /dest --log-level debug
```

**Tag a transfer operation:**
```bash
fastcopy /source /dest --tag "backup_20260315"
```

## Key Command-Line Options

| Option | Description | Example |
|--------|-------------|---------|
| `--streams N` | Set concurrent streams (1-64) | `--streams 32` |
| `--priority LEVEL` | Set priority (low, normal, high) | `--priority high` |
| `--verify` | Enable integrity checking | `--verify` |
| `--no-verify` | Disable integrity checking | `--no-verify` |
| `--resume` | Auto-resume on failure | `--resume` |
| `--buffer-size MB` | Set buffer size in MB | `--buffer-size 256` |
| `--log-level LEVEL` | Set logging (info, debug, error) | `--log-level debug` |
| `--profile PATH` | Use YAML profile | `--profile config.yaml` |
| `--dry-run` | Preview without executing | `--dry-run` |
| `--tag NAME` | Tag this transfer | `--tag "nightly"` |
| `--daemon` | Run as background daemon | `--daemon` |

## Profile Configuration (YAML)

FastCopy uses YAML profiles for complex or reusable transfer configurations.

### Basic Profile Example

**backup-profile.yaml:**
```yaml
transfer:
  mode: "turbo"              # standard, balanced, turbo
  streams: 32                # concurrent streams (1-64)
  buffer_size_mb: 128
  verify: true
  resume: always             # always, on_failure, never

paths:
  source: "/home/user/projects"
  destination: "/mnt/backup/projects"

schedule:
  type: "one_time"           # one_time, watch, cron
```

### Advanced Profile with Watch Mode

**watch-sync.yaml:**
```yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 64
  verify: true
  resume: on_failure

paths:
  source: "/home/user/documents"
  destination: "//nas/documents"

schedule:
  type: "watch"              # Monitors source for changes
  interval_seconds: 300      # Check every 5 minutes

hooks:
  on_complete: "echo 'Transfer complete' >> /var/log/fastcopy.log"
  on_error: "/opt/scripts/alert.sh"

filters:
  include:
    - "*.pdf"
    - "*.docx"
  exclude:
    - "*.tmp"
    - ".DS_Store"
```

### High-Performance Media Transfer

**media-transfer.yaml:**
```yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: false              # Skip verification for speed
  resume: always

paths:
  source: "/mnt/camera/raw_footage"
  destination: "/mnt/storage/archive"

schedule:
  type: "one_time"

logging:
  level: "info"
  output: "/var/log/fastcopy-media.log"
```

### Scheduled Cron-Based Backup

**cron-backup.yaml:**
```yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 256
  verify: true
  resume: always

paths:
  source: "/var/databases"
  destination: "/backup/databases"

schedule:
  type: "cron"
  expression: "0 2 * * *"    # Every day at 2 AM

hooks:
  on_complete: "curl -X POST https://monitoring.example.com/success?job=db_backup"
  on_error: "curl -X POST https://monitoring.example.com/failure?job=db_backup"

retention:
  keep_last: 7               # Keep last 7 backups
```

## Configuration Options Reference

### Transfer Modes
- `standard` - Balanced speed and resource usage
- `balanced` - Good performance with moderate resource consumption
- `turbo` - Maximum speed, highest resource usage

### Schedule Types
- `one_time` - Execute once and exit
- `watch` - Monitor source directory for changes
- `cron` - Schedule based on cron expression

### Resume Strategies
- `always` - Always attempt to resume interrupted transfers
- `on_failure` - Resume only if transfer fails
- `never` - Always start from scratch

## Common Patterns

### Pattern 1: Fast One-Time Backup
```bash
# Maximum speed, no verification
fastcopy /source /backup --streams 64 --no-verify --priority high
```

### Pattern 2: Verified Critical Data Transfer
```bash
# Slower but ensures data integrity
fastcopy /critical-data /secure-backup --streams 8 --verify --log-level debug
```

### Pattern 3: Continuous Directory Sync
Create **sync.yaml:**
```yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true
  resume: always

paths:
  source: "/home/user/workspace"
  destination: "/mnt/nas/workspace"

schedule:
  type: "watch"
  interval_seconds: 180

filters:
  exclude:
    - "node_modules"
    - ".git"
    - "*.log"
```

Run:
```bash
fastcopy --profile sync.yaml --daemon
```

### Pattern 4: Network Transfer with Retry
```bash
# Reliable network transfer with auto-resume
fastcopy /local/data //remote-server/share/data \
  --streams 16 \
  --resume \
  --verify \
  --buffer-size 128
```

### Pattern 5: Selective File Transfer
Create **selective.yaml:**
```yaml
transfer:
  mode: "turbo"
  streams: 32
  verify: true

paths:
  source: "/media/photos"
  destination: "/backup/photos"

filters:
  include:
    - "*.jpg"
    - "*.png"
    - "*.raw"
  exclude:
    - "*_thumbnail.jpg"
    - "*.xmp"
```

```bash
fastcopy --profile selective.yaml
```

## Integration Examples

### Shell Script Integration
```bash
#!/bin/bash
# Automated backup script

SOURCE="/var/www/html"
DEST="/backup/www/$(date +%Y%m%d)"
LOG="/var/log/backup.log"

echo "[$(date)] Starting backup..." >> "$LOG"

if fastcopy "$SOURCE" "$DEST" --streams 32 --verify --log-level info; then
    echo "[$(date)] Backup successful" >> "$LOG"
    exit 0
else
    echo "[$(date)] Backup failed" >> "$LOG"
    exit 1
fi
```

### Python Integration
```python
#!/usr/bin/env python3
import subprocess
import sys
from datetime import datetime

def run_fastcopy(source, destination, streams=16):
    """Execute fastcopy with specified parameters."""
    cmd = [
        'fastcopy',
        source,
        destination,
        '--streams', str(streams),
        '--verify',
        '--resume',
        '--log-level', 'info'
    ]
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        print(f"[{datetime.now()}] Transfer completed successfully")
        return True
    except subprocess.CalledProcessError as e:
        print(f"[{datetime.now()}] Transfer failed: {e.stderr}")
        return False

if __name__ == "__main__":
    success = run_fastcopy('/source/path', '/dest/path', streams=32)
    sys.exit(0 if success else 1)
```

### Profile-Based Python Script
```python
#!/usr/bin/env python3
import subprocess
import yaml
import sys

def create_transfer_profile(source, dest, mode="balanced", streams=16):
    """Generate FastCopy YAML profile programmatically."""
    profile = {
        'transfer': {
            'mode': mode,
            'streams': streams,
            'buffer_size_mb': 128,
            'verify': True,
            'resume': 'always'
        },
        'paths': {
            'source': source,
            'destination': dest
        },
        'schedule': {
            'type': 'one_time'
        }
    }
    
    profile_path = '/tmp/fastcopy_profile.yaml'
    with open(profile_path, 'w') as f:
        yaml.dump(profile, f)
    
    return profile_path

def run_with_profile(profile_path):
    """Execute fastcopy using profile."""
    cmd = ['fastcopy', '--profile', profile_path]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.returncode == 0

# Example usage
profile = create_transfer_profile('/data/source', '/data/backup', mode='turbo', streams=64)
success = run_with_profile(profile)
sys.exit(0 if success else 1)
```

## Troubleshooting

### Transfer is Slow
**Symptoms:** Transfer speed is much slower than expected.

**Solutions:**
1. Increase stream count:
   ```bash
   fastcopy /source /dest --streams 64
   ```

2. Use turbo mode:
   ```bash
   fastcopy /source /dest --mode turbo
   ```

3. Increase buffer size:
   ```bash
   fastcopy /source /dest --buffer-size 512
   ```

4. Disable verification:
   ```bash
   fastcopy /source /dest --no-verify
   ```

### Transfer Fails with Network Errors
**Symptoms:** Transfer to network location fails intermittently.

**Solutions:**
1. Enable auto-resume:
   ```bash
   fastcopy /source //network/dest --resume
   ```

2. Reduce stream count for unreliable networks:
   ```bash
   fastcopy /source //network/dest --streams 8
   ```

3. Use debug logging to identify issues:
   ```bash
   fastcopy /source //network/dest --log-level debug
   ```

### Integrity Check Failures
**Symptoms:** Transfer completes but verification fails.

**Solutions:**
1. Check source disk health
2. Reduce stream count to avoid corruption:
   ```bash
   fastcopy /source /dest --streams 4 --verify
   ```

3. Use smaller buffer size:
   ```bash
   fastcopy /source /dest --buffer-size 64 --verify
   ```

### Profile Not Loading
**Symptoms:** `fastcopy --profile config.yaml` returns error.

**Solutions:**
1. Validate YAML syntax:
   ```bash
   python3 -c "import yaml; yaml.safe_load(open('config.yaml'))"
   ```

2. Check file path is absolute or relative to current directory
3. Ensure all required fields are present (transfer, paths, schedule)

### High CPU/Memory Usage
**Symptoms:** FastCopy consumes excessive resources.

**Solutions:**
1. Reduce streams:
   ```bash
   fastcopy /source /dest --streams 8
   ```

2. Use balanced or standard mode:
   ```bash
   fastcopy /source /dest --mode balanced
   ```

3. Reduce buffer size:
   ```bash
   fastcopy /source /dest --buffer-size 64
   ```

### Watch Mode Not Detecting Changes
**Symptoms:** Watch profile doesn't trigger on file changes.

**Solutions:**
1. Reduce interval:
   ```yaml
   schedule:
     type: "watch"
     interval_seconds: 60
   ```

2. Check file system supports change notifications
3. Verify source path is correct and accessible
4. Run in foreground to see debug output:
   ```bash
   fastcopy --profile watch.yaml --log-level debug
   ```

## Performance Tuning Guidelines

### For Local Disk to Local Disk
```yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
  verify: false
```

### For Network Transfers
```yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always
```

### For SSD to SSD
```yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: false
```

### For Large Files (>1GB each)
```yaml
transfer:
  mode: "turbo"
  streams: 8          # Fewer streams for large files
  buffer_size_mb: 1024
  verify: true
```

### For Many Small Files
```yaml
transfer:
  mode: "turbo"
  streams: 64         # Many streams for parallelization
  buffer_size_mb: 64
  verify: false
```

## Environment Variables

FastCopy can use environment variables for configuration:

```bash
# Set default stream count
export FASTCOPY_STREAMS=32

# Set default buffer size
export FASTCOPY_BUFFER_MB=256

# Set default log level
export FASTCOPY_LOG_LEVEL=info

# Set default mode
export FASTCOPY_MODE=turbo

# Run with environment defaults
fastcopy /source /dest
```

## Best Practices

1. **Always verify critical data:** Use `--verify` for important transfers
2. **Use profiles for repeated tasks:** Create YAML profiles instead of long command lines
3. **Enable resume for network transfers:** Use `--resume` for unreliable connections
4. **Tag important transfers:** Use `--tag` for tracking and logging
5. **Test with dry-run first:** Use `--dry-run` to preview operations
6. **Monitor logs for large operations:** Use `--log-level debug` for troubleshooting
7. **Tune streams based on workload:** More streams for many small files, fewer for large files
8. **Use hooks for automation:** Add `on_complete` and `on_error` hooks in profiles
