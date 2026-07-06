---
name: fastcopy-file-transfer-utility
description: High-performance file copy and transfer utility with parallel streaming, integrity verification, and automation capabilities
triggers:
  - how do I use FastCopy for bulk file transfers
  - set up FastCopy with parallel streams
  - configure FastCopy profiles for automated copying
  - FastCopy integrity verification and checksums
  - optimize FastCopy transfer speed
  - FastCopy CLI commands and examples
  - schedule automated file copies with FastCopy
  - FastCopy YAML configuration syntax
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility designed for professionals who need speed, reliability, and automation. It supports parallel streaming (up to 64 concurrent streams), intelligent caching, integrity verification via SHA-256/CRC-32 checksums, and declarative YAML-based configuration profiles.

**Key Capabilities:**
- Multi-threaded parallel file transfers
- Automatic retry and resume on failures
- Profile-based configuration for repeatable workflows
- CLI and GUI modes
- Cross-platform support (Windows, macOS, Linux)
- Integration hooks for custom scripts
- Watch mode for continuous synchronization

## Installation

### Windows
```bash
# Download from official repository
# Extract the portable executable
fastcopy.exe --version
```

### macOS
```bash
# Install via Homebrew (if available) or download binary
brew install fastcopy
# Or download directly and add to PATH
```

### Linux
```bash
# Download binary and make executable
chmod +x fastcopy
sudo mv fastcopy /usr/local/bin/
fastcopy --version
```

## CLI Commands

### Basic Copy
```bash
# Simple file copy
fastcopy /source/file.iso /destination/

# Directory copy with verification
fastcopy /source/folder /destination/ --verify

# Copy with maximum streams
fastcopy /source /dest --streams 64
```

### Advanced Options
```bash
# High-priority transfer with debug logging
fastcopy /src /dst --priority high --log-level debug

# No verification for maximum speed
fastcopy /src /dst --streams 32 --no-verify

# Tag transfer for logging
fastcopy /src /dst --tag "backup_2026_07_15"

# Resume interrupted transfer
fastcopy /src /dst --resume always
```

### Profile-Based Execution
```bash
# Use a YAML profile
fastcopy --profile transfer-profile.yaml

# Override profile settings
fastcopy --profile profile.yaml --streams 16 --buffer-size 128
```

## Configuration

### YAML Profile Structure

FastCopy uses declarative YAML profiles for complex transfer scenarios:

```yaml
# production-backup.yaml
transfer:
  mode: "turbo"                # Options: standard, balanced, turbo
  streams: 32                  # Concurrent streams (1-64)
  buffer_size_mb: 256          # Buffer size in MB
  verify: true                 # Enable integrity checking
  resume: always               # Options: always, on_failure, never
  priority: high               # System priority level

paths:
  source: "/var/data/production"
  destination: "/mnt/backup/daily"
  exclude:
    - "*.tmp"
    - "*.log"
    - ".cache/*"

schedule:
  type: "cron"                 # Options: one_time, watch, cron
  cron_expression: "0 2 * * *" # 2 AM daily
  interval_seconds: null       # For watch mode

hooks:
  on_start: "echo 'Starting backup...'"
  on_complete: "/opt/scripts/notify-success.sh"
  on_error: "/opt/scripts/alert-admin.sh {{error}}"

logging:
  level: "info"                # debug, info, warn, error
  file: "/var/log/fastcopy/production-backup.log"
  rotate: true
```

### Watch Mode Configuration

For continuous synchronization:

```yaml
# sync-profile.yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true
  resume: always

paths:
  source: "/home/user/Documents"
  destination: "//nas/backup/documents"

schedule:
  type: "watch"
  interval_seconds: 300        # Check every 5 minutes
  debounce_ms: 1000           # Wait 1s after last change

filters:
  include:
    - "*.docx"
    - "*.pdf"
    - "*.xlsx"
  exclude:
    - "~$*"                    # Office temp files
```

### Network Transfer Profile

```yaml
# network-transfer.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  compression: true            # Enable network compression

paths:
  source: "/mnt/storage/media"
  destination: "sftp://backup-server/media"
  credentials_file: "${HOME}/.fastcopy/credentials.enc"

network:
  bandwidth_limit_mbps: 100   # Throttle if needed
  retry_attempts: 5
  retry_delay_seconds: 10
  timeout_seconds: 300

hooks:
  on_complete: "curl -X POST ${WEBHOOK_URL} -d 'Transfer complete'"
```

## Common Patterns

### Bulk Media Migration

```bash
# Create profile for large video files
cat > media-migration.yaml << EOF
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 1024
  verify: true

paths:
  source: "/media/raw_footage"
  destination: "/archive/2026/projects"
  
schedule:
  type: "one_time"

hooks:
  on_complete: "notify-send 'Media transfer complete'"
EOF

# Execute
fastcopy --profile media-migration.yaml
```

### Incremental Backup Script

```bash
#!/bin/bash
# backup.sh - Daily incremental backup

PROFILE="/etc/fastcopy/daily-backup.yaml"
LOG_DIR="/var/log/fastcopy"
DATE=$(date +%Y%m%d)

# Run transfer
fastcopy --profile "$PROFILE" \
  --log-level info \
  --tag "daily_${DATE}" \
  >> "${LOG_DIR}/backup_${DATE}.log" 2>&1

# Check exit code
if [ $? -eq 0 ]; then
  echo "Backup successful: ${DATE}" | mail -s "Backup Success" admin@example.com
else
  echo "Backup failed: ${DATE}" | mail -s "BACKUP FAILURE" admin@example.com
fi
```

### Development Environment Sync

```yaml
# dev-sync.yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: false              # Skip verification for speed
  resume: on_failure

paths:
  source: "/home/dev/projects"
  destination: "/mnt/shared/dev-backup"
  exclude:
    - "node_modules/*"
    - ".git/*"
    - "*.pyc"
    - "__pycache__/*"
    - "dist/*"
    - "build/*"

schedule:
  type: "watch"
  interval_seconds: 60       # Check every minute
  debounce_ms: 2000

hooks:
  on_error: "echo 'Sync failed' >> /tmp/sync-errors.log"
```

### Database Backup with Verification

```yaml
# db-backup.yaml
transfer:
  mode: "standard"
  streams: 8
  buffer_size_mb: 128
  verify: true               # Critical for DB files
  checksum_algorithm: "sha256"

paths:
  source: "/var/lib/postgresql/backups"
  destination: "/mnt/backup/databases"

schedule:
  type: "cron"
  cron_expression: "0 3 * * *"  # 3 AM daily

hooks:
  on_start: "pg_dump dbname > /var/lib/postgresql/backups/latest.sql"
  on_complete: "/opt/scripts/verify-db-backup.sh"
  on_error: "/opt/scripts/alert-dba.sh"

integrity:
  verify_after_transfer: true
  keep_checksums: true
  checksum_file: "/var/log/fastcopy/db-checksums.log"
```

## Advanced Usage

### Multi-Source Transfer

```yaml
# multi-source.yaml
transfer:
  mode: "turbo"
  streams: 24

sources:
  - path: "/data/source1"
    destination: "/backup/dataset1"
  - path: "/data/source2"
    destination: "/backup/dataset2"
  - path: "/media/archive"
    destination: "/backup/media"
    streams: 48              # Override for this source

global_hooks:
  on_all_complete: "/opt/scripts/finalize-backup.sh"
```

### Conditional Transfers

```yaml
# conditional-transfer.yaml
transfer:
  mode: "balanced"
  streams: 16

paths:
  source: "/data/logs"
  destination: "/archive/logs/${YEAR}/${MONTH}"

conditions:
  min_free_space_gb: 50      # Don't transfer if < 50GB free
  max_age_days: 7            # Only files older than 7 days
  min_file_size_mb: 1        # Skip files smaller than 1MB

filters:
  include:
    - "*.log"
    - "*.gz"
  exclude:
    - "debug-*"
```

## Environment Variables

FastCopy supports environment variable substitution in profiles:

```yaml
paths:
  source: "${FASTCOPY_SOURCE_DIR}"
  destination: "${FASTCOPY_DEST_DIR}"
  credentials_file: "${FASTCOPY_CREDENTIALS}"

hooks:
  on_complete: "curl -X POST ${WEBHOOK_URL} -d 'status=complete'"
```

Set variables before execution:

```bash
export FASTCOPY_SOURCE_DIR="/data/production"
export FASTCOPY_DEST_DIR="/backup/prod"
export WEBHOOK_URL="https://hooks.example.com/notify"

fastcopy --profile production.yaml
```

## Troubleshooting

### Transfer Fails Midway

```bash
# Enable auto-resume
fastcopy /src /dst --resume always --retry-attempts 10

# Check logs for specific errors
fastcopy /src /dst --log-level debug --log-file transfer.log
```

### Slow Transfer Speed

```yaml
# Optimize profile for speed
transfer:
  mode: "turbo"
  streams: 64                # Max parallel streams
  buffer_size_mb: 1024       # Large buffer
  verify: false              # Disable if not critical
  compression: false         # Disable for local transfers
```

### Permission Errors

```bash
# Run with elevated privileges
sudo fastcopy /protected/src /dest

# Or fix permissions beforehand
sudo chown -R $USER:$USER /dest
```

### Network Timeouts

```yaml
network:
  timeout_seconds: 600       # Increase timeout
  retry_attempts: 10
  retry_delay_seconds: 30
  keep_alive: true
```

### Verify Integrity Failures

```bash
# Force checksum verification
fastcopy /src /dst --verify --checksum-algorithm sha256

# Generate checksum report
fastcopy /src /dst --verify --checksum-report checksums.txt
```

### Memory Usage

```yaml
# Reduce memory footprint
transfer:
  streams: 8                 # Fewer concurrent streams
  buffer_size_mb: 64         # Smaller buffer
  mode: "standard"           # Conservative mode
```

## Integration Examples

### Systemd Service (Linux)

```ini
# /etc/systemd/system/fastcopy-backup.service
[Unit]
Description=FastCopy Automated Backup
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/fastcopy --profile /etc/fastcopy/backup.yaml
User=backup
Group=backup
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/fastcopy-backup.timer
[Unit]
Description=Run FastCopy backup daily

[Timer]
OnCalendar=daily
OnCalendar=02:00
Persistent=true

[Install]
WantedBy=timers.target
```

Enable:
```bash
sudo systemctl enable --now fastcopy-backup.timer
```

### Docker Container

```dockerfile
FROM alpine:latest

RUN apk add --no-cache bash curl

COPY fastcopy /usr/local/bin/
COPY backup-profile.yaml /etc/fastcopy/

ENV FASTCOPY_SOURCE_DIR=/data/source
ENV FASTCOPY_DEST_DIR=/data/destination

ENTRYPOINT ["fastcopy", "--profile", "/etc/fastcopy/backup-profile.yaml"]
```

### CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/backup.yml
name: Backup Artifacts

on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Download FastCopy
        run: |
          curl -L -o fastcopy https://example.com/fastcopy
          chmod +x fastcopy
          
      - name: Run Backup
        env:
          DEST_PATH: ${{ secrets.BACKUP_DESTINATION }}
        run: |
          ./fastcopy ./artifacts $DEST_PATH --streams 32 --verify
```

This skill provides comprehensive coverage of FastCopy's capabilities for AI coding agents to assist developers with file transfer automation, configuration, and troubleshooting.
