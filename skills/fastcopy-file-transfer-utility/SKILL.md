---
name: fastcopy-file-transfer-utility
description: High-speed file copying and synchronization utility with multi-threaded I/O, integrity verification, and automation support
triggers:
  - "how do I use FastCopy to copy files quickly"
  - "configure FastCopy for bulk file transfer"
  - "set up FastCopy profile for automated copying"
  - "FastCopy parallel stream configuration"
  - "verify file integrity with FastCopy"
  - "create FastCopy YAML profile"
  - "FastCopy CLI commands and options"
  - "troubleshoot FastCopy transfer errors"
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and synchronization utility designed for rapid data duplication with multi-threaded I/O, intelligent caching, and integrity verification. It supports profile-based configuration, CLI automation, and cross-platform operation.

**Warning**: This project appears to distribute unauthorized license patches. Use only legitimate licensed software. This skill documents the claimed functionality based on the repository documentation.

## Installation

### Download

According to the repository, FastCopy is distributed as a portable utility. Download from the official source or legitimate vendor.

### System Requirements

- **Windows**: 10/11, Server 2022+
- **macOS**: Monterey, Ventura, Sonoma
- **Linux**: Ubuntu 20.04+, Fedora 38+, Arch Linux

### Basic Setup

```bash
# Extract portable package
unzip fastcopy-portable.zip -d /opt/fastcopy

# Add to PATH (Linux/macOS)
export PATH=$PATH:/opt/fastcopy/bin

# Verify installation
fastcopy --version
```

## CLI Commands

### Basic File Copy

```bash
# Simple copy
fastcopy /source/path /destination/path

# Copy with verification
fastcopy /source/path /destination/path --verify

# High-priority copy with maximum streams
fastcopy /source/path /dest/path --priority high --streams 64
```

### Advanced Options

```bash
# Copy with specific buffer size
fastcopy source.iso /backups/ --buffer-size 256 --streams 32

# Skip verification for speed
fastcopy /media/raw /backup/raw --no-verify --mode turbo

# Resume interrupted transfer
fastcopy /large/dataset /archive/ --resume always

# Copy with logging
fastcopy /src /dst --log-level debug --log-file transfer.log

# Tag transfer for tracking
fastcopy /build /archive --tag "release_v2.1.0"
```

## Profile-Based Configuration

FastCopy uses YAML profiles for declarative transfer configuration.

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
  source: "/mnt/storage/projects"
  destination: "/backup/projects"

schedule:
  type: "one_time"                 # one_time, watch, cron
```

### Advanced Profile with Hooks

```yaml
# production-backup.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: always
  priority: high

paths:
  source: "/var/lib/databases"
  destination: "//nas.local/db-backup"

filters:
  include:
    - "*.db"
    - "*.sql"
  exclude:
    - "temp/*"
    - "*.tmp"

schedule:
  type: "cron"
  cron_expression: "0 2 * * *"    # Daily at 2 AM

hooks:
  on_start: "/scripts/pre-backup.sh"
  on_complete: "/scripts/notify-success.sh"
  on_error: "/scripts/alert-failure.sh"

logging:
  level: "info"
  file: "/var/log/fastcopy/backup.log"
  rotate: true
  max_size_mb: 100
```

### Watch Mode Profile

```yaml
# sync-profile.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: on_failure

paths:
  source: "/home/user/documents"
  destination: "/mnt/cloud-sync/documents"

schedule:
  type: "watch"
  interval_seconds: 300           # Check every 5 minutes
  detect_changes: true

filters:
  exclude:
    - ".git/*"
    - "node_modules/*"
    - "*.swp"
    - ".DS_Store"

hooks:
  on_change_detected: "echo 'Changes detected, syncing...'"
  on_complete: "notify-send 'Sync Complete'"
```

## Using Profiles

```bash
# Run with profile
fastcopy --profile production-backup.yaml

# Override profile settings
fastcopy --profile sync-profile.yaml --streams 64 --mode turbo

# Validate profile without running
fastcopy --profile backup.yaml --validate-only

# List active profiles
fastcopy --list-profiles
```

## Common Patterns

### Backup Large Datasets

```bash
# Maximum performance backup
fastcopy /data/warehouse /backup/warehouse \
  --streams 64 \
  --buffer-size 512 \
  --mode turbo \
  --priority high \
  --log-file backup-$(date +%Y%m%d).log
```

### Mirror Directory with Verification

```bash
# Ensure data integrity
fastcopy /production/assets /backup/assets \
  --verify \
  --mode balanced \
  --resume always \
  --tag "daily_mirror"
```

### Selective Copy with Filters

```yaml
# media-sync.yaml
transfer:
  mode: "turbo"
  streams: 32
  verify: true

paths:
  source: "/media/raw"
  destination: "/archive/processed"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"
  exclude:
    - "preview/*"
    - "*-draft.*"
  min_size_mb: 10
  max_size_mb: 50000
```

```bash
fastcopy --profile media-sync.yaml
```

### Automated Scheduled Backup

```yaml
# nightly-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 256
  verify: true
  resume: always

paths:
  source: "/opt/application/data"
  destination: "/mnt/nas/backups/app-data"

schedule:
  type: "cron"
  cron_expression: "0 3 * * *"

retention:
  keep_days: 30
  compress: true

hooks:
  on_start: "systemctl stop application.service"
  on_complete: "systemctl start application.service && /scripts/cleanup-old-backups.sh"
  on_error: "mail -s 'Backup Failed' admin@example.com < /var/log/fastcopy/error.log"

logging:
  level: "info"
  file: "/var/log/fastcopy/nightly-backup.log"
  rotate: true
```

### Development Environment Sync

```yaml
# dev-sync.yaml
transfer:
  mode: "standard"
  streams: 8
  verify: false                    # Speed over verification for dev

paths:
  source: "/home/dev/project"
  destination: "/mnt/remote-dev/project"

schedule:
  type: "watch"
  interval_seconds: 60
  detect_changes: true

filters:
  exclude:
    - ".git/*"
    - "node_modules/*"
    - "target/*"
    - "build/*"
    - "*.pyc"
    - "__pycache__/*"
    - ".venv/*"
    - "dist/*"

hooks:
  on_complete: "ssh remote-dev 'cd /project && ./rebuild.sh'"
```

## Configuration File Locations

### Global Configuration

```yaml
# ~/.fastcopy/config.yaml (Linux/macOS)
# C:\Users\USERNAME\.fastcopy\config.yaml (Windows)

defaults:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always

profiles_directory: "~/.fastcopy/profiles"

logging:
  default_level: "info"
  directory: "~/.fastcopy/logs"

notifications:
  enabled: true
  on_complete: true
  on_error: true
```

### Environment Variables

```bash
# Set default configuration
export FASTCOPY_CONFIG="/etc/fastcopy/config.yaml"

# Override default mode
export FASTCOPY_MODE="turbo"

# Set default streams
export FASTCOPY_STREAMS=32

# API integrations (if enabled)
export OPENAI_API_KEY="${OPENAI_API_KEY}"
export ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
```

## Troubleshooting

### Transfer Speed Issues

```bash
# Check system I/O limits
fastcopy --diagnose

# Test with different stream counts
fastcopy /source /dest --streams 16 --benchmark
fastcopy /source /dest --streams 32 --benchmark
fastcopy /source /dest --streams 64 --benchmark

# Increase buffer size for large files
fastcopy /source /dest --buffer-size 512 --streams 48
```

### Integrity Verification Failures

```bash
# Re-run with verbose verification
fastcopy /source /dest --verify --log-level debug

# Use specific checksum algorithm
fastcopy /source /dest --verify --checksum sha256

# Resume failed transfer
fastcopy /source /dest --resume always --verify
```

### Permission Errors

```bash
# Run with elevated privileges (if needed)
sudo fastcopy /protected/source /dest

# Check file permissions
ls -la /source/path
ls -la /dest/path

# Set appropriate permissions
chmod -R 755 /dest/path
```

### Network Transfer Issues

```bash
# Test network connectivity
ping nas.local

# Use lower stream count for network transfers
fastcopy /local /network-share --streams 8 --mode balanced

# Enable retry logic
fastcopy /local /network --resume always --retry-count 5 --retry-delay 10
```

### Profile Validation Errors

```bash
# Validate YAML syntax
fastcopy --profile config.yaml --validate-only

# Check profile schema
fastcopy --schema

# Debug profile loading
fastcopy --profile config.yaml --log-level debug --dry-run
```

### Memory/Resource Issues

```bash
# Reduce resource consumption
fastcopy /source /dest --streams 8 --buffer-size 64 --priority low

# Monitor system resources during transfer
fastcopy /source /dest --monitor-resources

# Limit CPU usage
fastcopy /source /dest --cpu-limit 50  # 50% max CPU
```

## Best Practices

1. **Always verify critical data transfers**: Use `--verify` for important files
2. **Start with balanced mode**: Only use turbo mode when speed is critical
3. **Test profiles in dry-run mode**: Use `--dry-run` to validate before execution
4. **Enable resume for large transfers**: Use `--resume always` for multi-GB operations
5. **Monitor logs**: Keep logs for troubleshooting and audit trails
6. **Use filters wisely**: Exclude unnecessary files to improve performance
7. **Schedule during off-peak hours**: Use cron scheduling for production backups
8. **Implement hooks for automation**: Chain operations with pre/post hooks

## Example: Complete Backup Script

```bash
#!/bin/bash
# complete-backup.sh

set -e

BACKUP_DIR="/mnt/backup/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Create profile dynamically
cat > /tmp/backup-profile.yaml <<EOF
transfer:
  mode: "balanced"
  streams: 32
  buffer_size_mb: 256
  verify: true
  resume: always

paths:
  source: "/data/production"
  destination: "$BACKUP_DIR"

filters:
  exclude:
    - "*.tmp"
    - "temp/*"

logging:
  level: "info"
  file: "$BACKUP_DIR/transfer.log"

hooks:
  on_complete: "tar -czf $BACKUP_DIR.tar.gz $BACKUP_DIR && rm -rf $BACKUP_DIR"
  on_error: "echo 'Backup failed' | mail -s 'Alert' admin@example.com"
EOF

# Execute backup
fastcopy --profile /tmp/backup-profile.yaml

# Cleanup old backups (keep last 7 days)
find /mnt/backup -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed successfully"
```

This skill provides comprehensive guidance for using FastCopy's documented features for file transfer automation and optimization.
