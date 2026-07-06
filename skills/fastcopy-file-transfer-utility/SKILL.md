---
name: fastcopy-file-transfer-utility
description: High-speed file duplication and transfer utility with parallel I/O and integrity verification
triggers:
  - how do I use FastCopy to copy files quickly
  - configure FastCopy for parallel file transfers
  - set up FastCopy with profile configuration
  - FastCopy CLI commands and options
  - optimize file copy performance with FastCopy
  - verify file integrity during FastCopy transfers
  - automate file sync with FastCopy profiles
  - FastCopy multi-threaded transfer settings
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility designed for rapid data migration, backup operations, and synchronization tasks. It uses multi-threaded parallel I/O streams, predictive caching, and integrity verification to maximize transfer speeds while ensuring data accuracy.

**Key capabilities:**
- Parallel stream processing (up to 64 concurrent streams)
- Integrity verification with SHA-256 and CRC-32 checksums
- YAML-based profile configuration for repeatable operations
- Auto-resume on interrupted transfers
- Cross-platform support (Windows, macOS, Linux)
- CLI and GUI modes

## Installation

### Windows
Download from the official repository and run the installer. The portable version requires no installation.

### macOS
```bash
brew install fastcopy
```

### Linux (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install fastcopy
```

### Arch Linux
```bash
yay -S fastcopy
```

## CLI Usage

### Basic Command Structure

```bash
fastcopy [SOURCE] [DESTINATION] [OPTIONS]
```

### Common Commands

**Simple file copy:**
```bash
fastcopy /path/to/source.file /path/to/destination/
```

**Directory copy with verification:**
```bash
fastcopy /source/directory /destination/directory --verify
```

**High-performance parallel transfer:**
```bash
fastcopy /large/dataset /backup/location --streams 32 --buffer 256 --priority high
```

**Using a profile:**
```bash
fastcopy --profile transfer-config.yaml
```

**Resume interrupted transfer:**
```bash
fastcopy /source /dest --resume always
```

**Dry run (preview without copying):**
```bash
fastcopy /source /dest --dry-run
```

### Key CLI Options

| Option | Description | Example |
|--------|-------------|---------|
| `--streams N` | Set concurrent transfer streams (1-64) | `--streams 32` |
| `--buffer N` | Buffer size in MB | `--buffer 256` |
| `--verify` | Enable integrity checking | `--verify` |
| `--no-verify` | Skip integrity checks for speed | `--no-verify` |
| `--resume MODE` | Resume mode: always, on_failure, never | `--resume always` |
| `--priority LEVEL` | Priority: low, normal, high | `--priority high` |
| `--mode MODE` | Transfer mode: standard, balanced, turbo | `--mode turbo` |
| `--log-level LEVEL` | Logging: debug, info, warn, error | `--log-level debug` |
| `--tag TEXT` | Tag operation for logs | `--tag "nightly_backup"` |
| `--exclude PATTERN` | Exclude files matching pattern | `--exclude "*.tmp"` |
| `--include PATTERN` | Only include matching files | `--include "*.mp4"` |

## Profile Configuration

FastCopy uses YAML profiles for declarative, repeatable transfer operations.

### Basic Profile Example

```yaml
# basic-transfer.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: on_failure

paths:
  source: "/data/project"
  destination: "/backup/project"
```

### Advanced Profile with Scheduling

```yaml
# scheduled-backup.yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
  verify: true
  resume: always
  preserve_permissions: true
  preserve_timestamps: true

paths:
  source: "/var/www/production"
  destination: "/mnt/backup/www"

filters:
  exclude:
    - "*.log"
    - "*.tmp"
    - ".cache/*"
  include:
    - "*.php"
    - "*.html"
    - "*.js"

schedule:
  type: "cron"
  expression: "0 2 * * *"  # Daily at 2 AM

hooks:
  on_start: "echo 'Backup started' | mail -s 'Backup Alert' admin@example.com"
  on_complete: "notify-send 'Backup Complete'"
  on_error: "/opt/scripts/backup-failed-alert.sh"

logging:
  level: "info"
  file: "/var/log/fastcopy/backup.log"
  rotate: true
  max_size_mb: 100
```

### Watch Mode Profile

```yaml
# watch-sync.yaml
transfer:
  mode: "standard"
  streams: 8
  buffer_size_mb: 64
  verify: true

paths:
  source: "/home/user/documents"
  destination: "/mnt/nas/documents"

schedule:
  type: "watch"
  interval_seconds: 300  # Check every 5 minutes

filters:
  exclude:
    - ".git/*"
    - "node_modules/*"
    - "*.swp"

sync_mode: "mirror"  # mirror, update, differential
```

## Configuration Options

### Transfer Modes

- **standard**: Balanced speed and safety
- **balanced**: Optimized for most use cases (default)
- **turbo**: Maximum speed, uses all available resources

### Resume Modes

- **always**: Always resume from last checkpoint
- **on_failure**: Resume only if transfer was interrupted
- **never**: Always start fresh

### Sync Modes

- **mirror**: Make destination identical to source (deletes extra files)
- **update**: Only copy newer files
- **differential**: Copy only changed files

## Common Patterns

### Large Media File Transfer

```bash
# Maximum speed for video files
fastcopy /media/raw_footage /nas/archive \
  --mode turbo \
  --streams 64 \
  --buffer 512 \
  --no-verify \
  --include "*.mp4" \
  --include "*.mov"
```

### Database Backup with Verification

```bash
# Careful backup with full integrity checks
fastcopy /var/lib/postgresql/data /backup/postgres \
  --mode balanced \
  --streams 8 \
  --verify \
  --resume always \
  --preserve-permissions \
  --tag "postgres_backup_$(date +%Y%m%d)"
```

### Development Environment Sync

```yaml
# dev-sync.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true

paths:
  source: "/home/developer/projects"
  destination: "/mnt/backup/projects"

filters:
  exclude:
    - "node_modules/*"
    - ".git/*"
    - "*.pyc"
    - "__pycache__/*"
    - ".venv/*"
    - "dist/*"
    - "build/*"

sync_mode: "update"
```

### Incremental Backup Script

```bash
#!/bin/bash
# incremental-backup.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
SOURCE="/data/production"
DEST="/backup/incremental/$TIMESTAMP"

fastcopy "$SOURCE" "$DEST" \
  --mode balanced \
  --streams 24 \
  --verify \
  --resume always \
  --log-level info \
  --tag "incremental_$TIMESTAMP" \
  2>&1 | tee "/var/log/fastcopy/backup_$TIMESTAMP.log"

if [ $? -eq 0 ]; then
    echo "Backup successful: $TIMESTAMP"
else
    echo "Backup failed: $TIMESTAMP" >&2
    exit 1
fi
```

## Integration Examples

### Environment Variable Configuration

```bash
# Set default FastCopy options via environment
export FASTCOPY_DEFAULT_STREAMS=32
export FASTCOPY_DEFAULT_BUFFER=256
export FASTCOPY_DEFAULT_MODE=balanced
export FASTCOPY_LOG_DIR=/var/log/fastcopy

fastcopy /source /dest  # Uses env defaults
```

### Python Integration

```python
#!/usr/bin/env python3
import subprocess
import os
from pathlib import Path

def fastcopy_transfer(source, destination, streams=16, verify=True):
    """
    Execute FastCopy with Python wrapper
    """
    cmd = [
        'fastcopy',
        str(source),
        str(destination),
        '--streams', str(streams),
        '--mode', 'balanced',
        '--resume', 'always'
    ]
    
    if verify:
        cmd.append('--verify')
    
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        check=False
    )
    
    if result.returncode != 0:
        raise RuntimeError(f"FastCopy failed: {result.stderr}")
    
    return result.stdout

# Usage
try:
    output = fastcopy_transfer(
        source="/data/media",
        destination="/backup/media",
        streams=32,
        verify=True
    )
    print(f"Transfer complete:\n{output}")
except RuntimeError as e:
    print(f"Error: {e}")
```

### Bash Automation Script

```bash
#!/bin/bash
# automated-sync.sh

set -e

CONFIG_FILE="${1:-sync-config.yaml}"
LOG_DIR="/var/log/fastcopy"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$LOG_DIR"

echo "Starting FastCopy with profile: $CONFIG_FILE"

fastcopy --profile "$CONFIG_FILE" \
  --log-level info \
  2>&1 | tee "$LOG_DIR/sync_$TIMESTAMP.log"

EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ]; then
    echo "Sync completed successfully at $(date)"
    # Optional: Send success notification
    # curl -X POST "$SLACK_WEBHOOK_URL" -d '{"text":"FastCopy sync completed"}'
else
    echo "Sync failed with exit code $EXIT_CODE" >&2
    # Optional: Send error notification
    exit $EXIT_CODE
fi
```

## Troubleshooting

### Performance Issues

**Problem**: Transfer is slower than expected

**Solutions**:
```bash
# Increase streams and buffer size
fastcopy /source /dest --streams 64 --buffer 512 --mode turbo

# Disable verification for maximum speed (use with caution)
fastcopy /source /dest --no-verify --mode turbo

# Check system resources
fastcopy /source /dest --log-level debug
```

### Interrupted Transfers

**Problem**: Transfer stops unexpectedly

**Solution**:
```bash
# Enable auto-resume
fastcopy /source /dest --resume always

# Or resume manually from checkpoint
fastcopy /source /dest --resume-from /path/to/checkpoint.state
```

### Permission Errors

**Problem**: Cannot copy files due to permissions

**Solutions**:
```bash
# Run with elevated privileges (use carefully)
sudo fastcopy /source /dest

# Or preserve permissions in profile
# In YAML:
transfer:
  preserve_permissions: true
  preserve_ownership: true
```

### Out of Memory

**Problem**: FastCopy crashes with large files

**Solution**:
```bash
# Reduce buffer size and streams
fastcopy /source /dest --streams 8 --buffer 64

# Use standard mode instead of turbo
fastcopy /source /dest --mode standard
```

### Verification Failures

**Problem**: Integrity check fails

**Solutions**:
```bash
# Retry with automatic verification retry
fastcopy /source /dest --verify --verify-retries 3

# Check source file integrity first
sha256sum /source/file

# Force re-copy
fastcopy /source /dest --force-overwrite --verify
```

## Best Practices

1. **Use profiles for repeated operations**: Store configurations in YAML files for consistency
2. **Enable verification for critical data**: Always use `--verify` for important transfers
3. **Tune streams based on hardware**: More streams aren't always better; test optimal values
4. **Monitor logs in production**: Use `--log-level info` and log rotation
5. **Test with dry-run first**: Use `--dry-run` before large operations
6. **Set appropriate tags**: Use `--tag` for searchable operation logs
7. **Configure hooks for automation**: Use on_complete and on_error hooks for workflows

## Exit Codes

- `0`: Success
- `1`: General error
- `2`: Invalid arguments
- `3`: Source not found
- `4`: Destination not writable
- `5`: Transfer interrupted
- `6`: Verification failed
- `7`: Insufficient disk space

Check exit codes in scripts:
```bash
fastcopy /source /dest
if [ $? -ne 0 ]; then
    echo "Transfer failed"
    exit 1
fi
```
