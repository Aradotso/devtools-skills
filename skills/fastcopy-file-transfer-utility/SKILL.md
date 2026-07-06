---
name: fastcopy-file-transfer-utility
description: High-performance file duplication and transfer tool with multi-threaded I/O, verification, and automation capabilities
triggers:
  - "how do I use FastCopy to copy files"
  - "set up FastCopy with profile configuration"
  - "configure FastCopy for parallel file transfers"
  - "automate file copying with FastCopy"
  - "FastCopy command line usage"
  - "create FastCopy YAML profiles"
  - "troubleshoot FastCopy transfer errors"
  - "optimize FastCopy performance settings"
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility optimized for speed and reliability. It features multi-threaded I/O operations, intelligent caching, integrity verification (SHA-256 + CRC-32), and support for automated transfer pipelines. The tool supports both GUI and CLI modes and uses declarative YAML profiles for transfer configuration.

**Key capabilities:**
- Parallel transfer streams (up to 64 concurrent)
- Predictive caching for reduced latency
- Automatic retry and resume on failures
- Cross-platform support (Windows, macOS, Linux)
- Profile-based configuration system
- Webhook/hook system for automation

## Installation

### Windows
```bash
# Download from official site or GitHub release
# Run installer or extract portable version
fastcopy.exe --version
```

### macOS
```bash
brew install fastcopy
# OR download DMG and drag to Applications
fastcopy --version
```

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install fastcopy
fastcopy --version
```

### Linux (Fedora)
```bash
sudo dnf install fastcopy
fastcopy --version
```

### From Source
```bash
git clone https://github.com/fastcopy/fastcopy.git
cd fastcopy
make install
```

## CLI Commands

### Basic Usage

```bash
# Simple file copy
fastcopy /path/to/source /path/to/destination

# Copy directory recursively
fastcopy /source/dir /dest/dir --recursive

# Copy with verification
fastcopy source.dat dest.dat --verify

# High-speed mode with maximum streams
fastcopy large_file.iso /backups/ --streams 64 --mode turbo

# Copy with progress display
fastcopy /media/videos /backup/videos --progress --verbose

# Dry run (preview without copying)
fastcopy /source /dest --dry-run
```

### Profile-Based Operations

```bash
# Use predefined profile
fastcopy --profile /path/to/profile.yaml

# Use profile with priority override
fastcopy --profile backup-profile.yaml --priority high

# Run with debug logging
fastcopy --profile sync-profile.yaml --log-level debug

# One-time scheduled transfer
fastcopy --profile nightly-backup.yaml --schedule one_time

# Watch mode (continuous monitoring)
fastcopy --profile watch-profile.yaml --schedule watch
```

### Advanced Options

```bash
# Set custom buffer size
fastcopy /source /dest --buffer-size 512

# Skip verification for speed
fastcopy /source /dest --no-verify

# Resume interrupted transfer
fastcopy /source /dest --resume always

# Tag transfer for logging
fastcopy /source /dest --tag "migration_2026"

# Set I/O priority
fastcopy /source /dest --priority low|normal|high

# Exclude patterns
fastcopy /source /dest --exclude "*.tmp" --exclude "*.log"

# Include only specific patterns
fastcopy /source /dest --include "*.mp4" --include "*.mkv"
```

## Profile Configuration

### Basic Profile Structure

Create a YAML file (e.g., `transfer-profile.yaml`):

```yaml
transfer:
  mode: "turbo"                # standard, balanced, turbo
  streams: 32                  # concurrent streams (1-64)
  buffer_size_mb: 256          # buffer size in MB
  verify: true                 # enable integrity checks
  resume: always               # always, on_failure, never

paths:
  source: "/mnt/storage/data"
  destination: "/backup/data"

schedule:
  type: "one_time"             # one_time, watch, cron
  interval_seconds: 300        # for watch mode

hooks:
  on_complete: "echo 'Transfer complete' | mail -s 'Backup Done' admin@example.com"
  on_error: "/opt/scripts/error-notify.sh"

logging:
  level: "info"                # debug, info, warn, error
  file: "/var/log/fastcopy/transfer.log"
```

### Performance-Optimized Profile

```yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: false                # disable for maximum speed
  resume: on_failure
  compression: false           # disable compression overhead

paths:
  source: "/raid/media/raw"
  destination: "//nas01/archive"

filters:
  exclude:
    - "*.tmp"
    - "*.cache"
    - ".DS_Store"
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"

hooks:
  on_complete: "curl -X POST https://monitoring.example.com/webhook -d '{\"status\":\"complete\"}'"
  on_error: "curl -X POST https://monitoring.example.com/webhook -d '{\"status\":\"failed\"}'"
```

### Automated Backup Profile

```yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always

paths:
  source: "/home/user/documents"
  destination: "/mnt/backup/documents"

schedule:
  type: "cron"
  expression: "0 2 * * *"      # daily at 2 AM

filters:
  exclude:
    - "*.tmp"
    - "node_modules/"
    - ".git/"

hooks:
  on_complete: "/usr/local/bin/backup-success-notify"
  on_error: "/usr/local/bin/backup-failure-alert"

retention:
  keep_versions: 7
  cleanup_old: true

logging:
  level: "info"
  file: "/var/log/fastcopy/daily-backup.log"
  rotate: true
  max_size_mb: 100
```

### Watch Mode Profile

```yaml
transfer:
  mode: "standard"
  streams: 8
  buffer_size_mb: 64
  verify: true
  resume: always

paths:
  source: "/var/www/uploads"
  destination: "/mnt/cdn/uploads"

schedule:
  type: "watch"
  interval_seconds: 60         # check every minute
  debounce_seconds: 5          # wait 5s after last change

filters:
  include:
    - "*.jpg"
    - "*.png"
    - "*.pdf"

hooks:
  on_file_added: "echo 'New file: $FILE' >> /var/log/uploads.log"
  on_complete: "/usr/local/bin/invalidate-cdn-cache"

notifications:
  enabled: true
  method: "webhook"
  endpoint: "${WEBHOOK_URL}"
```

## API Integration Profiles

### OpenAI Integration

```yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true

paths:
  source: "/data/documents"
  destination: "/archive/categorized"

integrations:
  openai:
    enabled: true
    api_key: "${OPENAI_API_KEY}"
    endpoint: "https://api.openai.com/v1"
    model: "gpt-4-turbo"
    prompt_template: "Categorize this file and suggest a folder: {filename}"
    auto_organize: true

hooks:
  on_categorized: "/usr/local/bin/process-category.sh"
```

### Claude Integration

```yaml
transfer:
  mode: "balanced"
  streams: 12
  verify: true

paths:
  source: "/projects/media"
  destination: "/archive/projects"

integrations:
  claude:
    enabled: true
    api_key: "${ANTHROPIC_API_KEY}"
    api_base: "https://api.anthropic.com"
    model: "claude-3-opus-20240229"
    generate_summary: true
    summary_output: "/logs/transfer-summaries"

hooks:
  on_complete: "/usr/local/bin/send-summary-report.sh"
```

## Common Patterns

### Bulk Migration Script

```bash
#!/bin/bash
# migrate-data.sh

PROFILE="/etc/fastcopy/migration.yaml"
LOG="/var/log/migration-$(date +%Y%m%d).log"

fastcopy --profile "$PROFILE" \
  --priority high \
  --log-level info \
  --log-file "$LOG" \
  --tag "migration_phase1"

if [ $? -eq 0 ]; then
  echo "Migration successful" | mail -s "Migration Complete" admin@example.com
else
  echo "Migration failed. Check logs at $LOG" | mail -s "Migration FAILED" admin@example.com
  exit 1
fi
```

### Automated Backup with Rotation

```bash
#!/bin/bash
# daily-backup.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
SOURCE="/home/user/important"
DEST="/backups/user_${TIMESTAMP}"

fastcopy "$SOURCE" "$DEST" \
  --streams 32 \
  --verify \
  --resume always \
  --tag "daily_backup_${TIMESTAMP}"

# Keep only last 7 days
find /backups -type d -name "user_*" -mtime +7 -exec rm -rf {} \;
```

### Network Transfer with Progress Monitoring

```bash
#!/bin/bash
# network-sync.sh

SOURCE="/local/data"
DEST="//remote-server/share/data"

fastcopy "$SOURCE" "$DEST" \
  --streams 16 \
  --buffer-size 256 \
  --verify \
  --progress \
  --resume always \
  2>&1 | tee /var/log/sync-$(date +%Y%m%d).log
```

### Conditional Transfer Based on File Type

```yaml
# media-transfer.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true

paths:
  source: "/production/footage"
  destination: "/archive/footage"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"
    - "*.mkv"
  exclude:
    - "*_preview.*"
    - "*_proxy.*"

conditions:
  min_file_size_mb: 100        # only files > 100MB
  max_file_age_days: 30        # only files < 30 days old

hooks:
  on_complete: "/usr/local/bin/update-catalog.sh"
```

## Environment Variables

```bash
# Configuration
export FASTCOPY_CONFIG_DIR="/etc/fastcopy"
export FASTCOPY_LOG_DIR="/var/log/fastcopy"
export FASTCOPY_TEMP_DIR="/tmp/fastcopy"

# Performance tuning
export FASTCOPY_DEFAULT_STREAMS=32
export FASTCOPY_DEFAULT_BUFFER_MB=256
export FASTCOPY_MAX_RETRIES=3

# API keys (for integrations)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export WEBHOOK_URL="https://hooks.example.com/notify"

# Logging
export FASTCOPY_LOG_LEVEL="info"
export FASTCOPY_LOG_FORMAT="json"
```

## Troubleshooting

### Transfer Stalls or Hangs

```bash
# Reduce stream count
fastcopy /source /dest --streams 8

# Reduce buffer size
fastcopy /source /dest --buffer-size 64

# Check with verbose logging
fastcopy /source /dest --verbose --log-level debug
```

### Verification Failures

```bash
# Retry with slower mode
fastcopy /source /dest --mode standard --verify

# Check source file integrity first
sha256sum /source/file.dat

# Disable caching if corrupted reads
fastcopy /source /dest --no-cache --verify
```

### Performance Issues

```bash
# Profile system resources during transfer
iostat -x 1 &
vmstat 1 &
fastcopy /source /dest --streams 32 --mode turbo

# Optimize for your system
fastcopy /source /dest \
  --streams $(nproc) \
  --buffer-size 512 \
  --mode turbo \
  --no-verify
```

### Network Transfer Failures

```bash
# Use resume mode with retries
fastcopy /local //remote/share \
  --resume always \
  --retry-count 5 \
  --retry-delay 10

# Monitor network during transfer
iftop -i eth0 &
fastcopy /local //remote/share --progress
```

### Profile Not Loading

```bash
# Validate YAML syntax
fastcopy --profile myprofile.yaml --validate

# Use absolute paths
fastcopy --profile /etc/fastcopy/profiles/backup.yaml

# Check file permissions
ls -la /etc/fastcopy/profiles/backup.yaml
```

### Hooks Not Executing

```bash
# Test hook script separately
bash -x /usr/local/bin/my-hook.sh

# Check hook permissions
chmod +x /usr/local/bin/my-hook.sh

# Enable hook debugging in profile
logging:
  level: "debug"
  hook_output: true
```

## Best Practices

1. **Always enable verification for critical data**: `--verify` flag
2. **Use profiles for reproducible operations**: Store configs in version control
3. **Set appropriate stream counts**: Match to disk/network capabilities
4. **Enable resume for long transfers**: `--resume always`
5. **Tag transfers for audit trails**: `--tag "project_migration"`
6. **Test with dry-run first**: `--dry-run` to preview operations
7. **Monitor logs in production**: Use `--log-file` and rotate regularly
8. **Use environment variables for secrets**: Never hardcode API keys

## Performance Optimization

```yaml
# Maximum speed profile (use with caution)
transfer:
  mode: "turbo"
  streams: 64                  # max concurrent
  buffer_size_mb: 1024         # large buffer
  verify: false                # skip for speed
  compression: false           # no CPU overhead
  cache_strategy: "aggressive"

# Balanced reliability profile
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 256
  verify: true
  resume: always
  compression: false
```
