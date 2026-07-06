---
name: fastcopy-file-transfer-utility
description: High-performance file copying and synchronization utility with parallel streams, integrity verification, and automation capabilities
triggers:
  - how do I use FastCopy to transfer files quickly
  - configure FastCopy for bulk file migration
  - set up FastCopy with YAML profiles
  - use FastCopy CLI for parallel file copying
  - FastCopy integrity verification and resume
  - automate file transfers with FastCopy
  - FastCopy multi-threaded copy setup
  - troubleshoot FastCopy transfer errors
---

# FastCopy File Transfer Utility Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility designed for bulk data migration, backups, and synchronization tasks. It provides:

- **Parallel multi-stream transfers** (up to 64 concurrent streams)
- **YAML-based profile system** for reproducible transfer configurations
- **Integrity verification** using SHA-256 and CRC-32 checksums
- **Auto-resume** for interrupted transfers
- **CLI and GUI modes** for scripting and interactive use
- **Cross-platform support** (Windows, macOS, Linux)

**⚠️ WARNING**: This project appears to be a potentially unauthorized patch/keygen repository. The actual legitimate FastCopy software is developed by SHIROUZU Hiroaki. Exercise caution and verify licensing before use.

## Installation

### Windows
```bash
# Download from official source (not this repository)
# Extract portable version
fastcopy.exe --version
```

### macOS/Linux
```bash
# If available via package manager
brew install fastcopy  # macOS
apt-get install fastcopy  # Debian/Ubuntu

# Or download portable binary
chmod +x fastcopy
./fastcopy --version
```

## CLI Usage

### Basic File Copy
```bash
# Simple copy with default settings
fastcopy /source/path /destination/path

# Copy with specific number of streams
fastcopy /source/file.iso /backup/ --streams 32

# High-priority transfer with verbose logging
fastcopy /data /backup --priority high --log-level debug
```

### Using Profiles

Create a YAML profile for reproducible transfers:

```yaml
# production-backup.yaml
transfer:
  mode: "turbo"                # standard, balanced, turbo
  streams: 32                  # concurrent streams (1-64)
  buffer_size_mb: 256
  verify: true                 # SHA-256 + CRC-32 verification
  resume: always               # always, on_failure, never

paths:
  source: "/var/www/production"
  destination: "/mnt/backup/production"

schedule:
  type: "watch"                # one_time, watch, cron
  interval_seconds: 300        # check every 5 minutes

hooks:
  on_complete: "echo 'Backup complete' | mail -s 'FastCopy Success' admin@example.com"
  on_error: "/opt/scripts/notify-error.sh"
```

Execute with profile:
```bash
fastcopy --profile production-backup.yaml
```

### Advanced CLI Options

```bash
# Skip verification for maximum speed
fastcopy /source /dest --streams 64 --no-verify

# Tag transfers for logging/tracking
fastcopy /data /backup --tag "nightly_backup_$(date +%Y%m%d)"

# Dry-run mode (test without copying)
fastcopy /source /dest --dry-run --verbose

# Filter by file type
fastcopy /media /backup --include "*.mp4,*.mkv" --exclude "*.tmp"

# Rate limiting (useful for network transfers)
fastcopy /source /dest --max-bandwidth 100M
```

## Configuration Patterns

### Development Environment Sync

```yaml
# dev-sync.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: on_failure

paths:
  source: "~/projects/myapp"
  destination: "//dev-server/workspace/myapp"
  
filters:
  exclude:
    - "node_modules/"
    - "*.log"
    - ".git/"
    - "__pycache__/"

schedule:
  type: "watch"
  interval_seconds: 60
```

### Database Backup Profile

```yaml
# db-backup.yaml
transfer:
  mode: "turbo"
  streams: 8                   # Lower for sequential DB dumps
  buffer_size_mb: 512
  verify: true
  resume: always

paths:
  source: "/var/lib/postgresql/backups/latest.dump"
  destination: "/mnt/nas/db-backups/postgres_$(date +%Y%m%d_%H%M%S).dump"

hooks:
  pre_transfer: "/opt/scripts/pg_dump_latest.sh"
  on_complete: "/opt/scripts/cleanup_old_backups.sh"
  on_error: "/opt/scripts/alert_dba.sh"
```

### Media Library Migration

```yaml
# media-migration.yaml
transfer:
  mode: "turbo"
  streams: 48                  # High parallelism for many small files
  buffer_size_mb: 256
  verify: true
  resume: always

paths:
  source: "/media/raw_footage"
  destination: "//nas/archive/project_2026"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.mkv"
    - "*.avi"
  
schedule:
  type: "one_time"

hooks:
  on_complete: "notify-send 'Media transfer complete'"
```

## Scripting Integration

### Bash Automation

```bash
#!/bin/bash
# automated-backup.sh

LOG_FILE="/var/log/fastcopy/backup_$(date +%Y%m%d).log"
SOURCE="/var/www"
DEST="/mnt/backup/www"

echo "[$(date)] Starting backup..." >> "$LOG_FILE"

if fastcopy "$SOURCE" "$DEST" \
    --streams 32 \
    --verify \
    --log-level info 2>&1 | tee -a "$LOG_FILE"; then
    echo "[$(date)] Backup successful" >> "$LOG_FILE"
    exit 0
else
    echo "[$(date)] Backup failed with exit code $?" >> "$LOG_FILE"
    mail -s "FastCopy Backup Failed" admin@example.com < "$LOG_FILE"
    exit 1
fi
```

### Python Integration

```python
#!/usr/bin/env python3
import subprocess
import json
from pathlib import Path

def run_fastcopy(profile_path, env_vars=None):
    """Execute FastCopy with a profile and optional environment variables."""
    cmd = ["fastcopy", "--profile", str(profile_path), "--json-output"]
    
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        env=env_vars
    )
    
    if result.returncode == 0:
        return json.loads(result.stdout)
    else:
        raise RuntimeError(f"FastCopy failed: {result.stderr}")

# Example usage
try:
    profile = Path("/etc/fastcopy/profiles/nightly-backup.yaml")
    result = run_fastcopy(profile)
    print(f"Transferred {result['bytes_copied']} bytes in {result['duration_seconds']}s")
except RuntimeError as e:
    print(f"Transfer failed: {e}")
```

### Monitoring with Status Checking

```bash
# Check transfer status (if FastCopy supports daemon mode)
fastcopy status --job-id 12345

# List active transfers
fastcopy list --active

# Cancel a running transfer
fastcopy cancel --job-id 12345

# View transfer statistics
fastcopy stats --job-id 12345 --format json
```

## Common Patterns

### Network Share Transfers

```bash
# Mount network share first (Linux)
mount -t cifs //server/share /mnt/remote -o credentials=/etc/samba/creds

# Then transfer with FastCopy
fastcopy /local/data /mnt/remote/backup \
    --streams 16 \
    --buffer-size 128 \
    --verify

# Unmount when complete
umount /mnt/remote
```

### Incremental Backups

```yaml
# incremental-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  verify: true
  resume: always
  incremental: true            # Only copy changed files
  compare_method: "timestamp"  # timestamp, checksum, or size

paths:
  source: "/home/user/documents"
  destination: "/backup/documents"
```

### Large File Optimization

```yaml
# large-file-transfer.yaml
transfer:
  mode: "turbo"
  streams: 4                   # Fewer streams for very large files
  buffer_size_mb: 1024         # Larger buffer for sequential reads
  verify: true
  chunk_size_mb: 100           # Split large files into chunks

paths:
  source: "/staging/database_dump_500gb.sql"
  destination: "/archive/databases/"
```

## Troubleshooting

### Transfer Speed Issues

```bash
# Check system I/O performance
iostat -x 1

# Monitor FastCopy resource usage
top -p $(pgrep fastcopy)

# Test with different stream counts
fastcopy /test/file /dest --streams 8   # Try lower
fastcopy /test/file /dest --streams 64  # Try higher

# Verify no network bottleneck
iperf3 -c destination-server
```

### Verification Failures

```bash
# Re-run with detailed logging
fastcopy /source /dest --verify --log-level debug --log-file /tmp/fastcopy.log

# Check for disk errors
smartctl -a /dev/sda

# Verify checksums manually
sha256sum /source/file.dat
sha256sum /dest/file.dat
```

### Interrupted Transfers

```bash
# FastCopy should auto-resume if configured
fastcopy --profile backup.yaml  # Will resume from checkpoint

# Force resume from specific checkpoint
fastcopy --resume-from /var/lib/fastcopy/checkpoints/job_12345.ckpt

# Clear stale checkpoints
fastcopy cleanup --checkpoints --older-than 7d
```

### Permission Issues

```bash
# Run with elevated privileges (if needed)
sudo fastcopy /protected/source /backup

# Check source/destination permissions
ls -la /source/path
ls -la /destination/path

# Set appropriate ownership after transfer
chown -R user:group /destination/path
```

### Memory Issues

```yaml
# Reduce memory footprint
transfer:
  streams: 8                   # Lower parallelism
  buffer_size_mb: 64           # Smaller buffers
  chunk_size_mb: 50
```

## Environment Variables

```bash
# Set temporary directory for checkpoints
export FASTCOPY_TEMP_DIR=/var/tmp/fastcopy

# Set default log level
export FASTCOPY_LOG_LEVEL=info

# Set default verify mode
export FASTCOPY_VERIFY=true

# Set maximum memory usage (MB)
export FASTCOPY_MAX_MEMORY=2048
```

## Best Practices

1. **Always verify critical transfers**: Use `--verify` for production data
2. **Test profiles with dry-run**: `--dry-run` before executing large transfers
3. **Monitor resource usage**: Adjust streams/buffers based on system capacity
4. **Use incremental mode**: For repeated backups to save time
5. **Set up hooks**: Automate notifications and error handling
6. **Log everything**: Keep transfer logs for audit trails
7. **Rate-limit network transfers**: Prevent bandwidth saturation

## Security Considerations

⚠️ **Important**: This repository appears to contain unauthorized patches. For production use:

- Obtain FastCopy from official sources only
- Purchase legitimate licenses for commercial use
- Do not use keygens or patches from untrusted sources
- Verify integrity of downloaded binaries with official checksums
- Review license terms before deployment

## Additional Resources

- Official FastCopy documentation (from legitimate source)
- Community forums for troubleshooting
- Performance tuning guides for specific use cases
- Integration examples for CI/CD pipelines
