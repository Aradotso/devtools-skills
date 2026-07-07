---
name: fastcopy-file-transfer-optimization
description: High-performance file duplication and transfer utility with multi-threaded I/O, integrity verification, and automation support
triggers:
  - how do I use FastCopy for bulk file transfers
  - configure FastCopy with YAML profiles
  - set up parallel file copying with FastCopy
  - FastCopy command line options and examples
  - optimize file transfer speed with FastCopy
  - automate file backup using FastCopy
  - FastCopy integrity verification and checksums
  - troubleshoot FastCopy transfer errors
---

# FastCopy File Transfer Optimization

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**FastCopy** is a high-performance file duplication and transfer utility designed for professionals who need fast, reliable bulk file operations. It features multi-threaded I/O engines, intelligent caching, automatic retry logic, and integrity verification using SHA-256 and CRC-32 checksums.

Key capabilities:
- Multi-threaded parallel transfers (up to 64 concurrent streams)
- YAML-based profile configuration for repeatable operations
- CLI and GUI modes for flexible integration
- Cross-platform support (Windows, macOS, Linux)
- Built-in integrity verification and auto-resume
- Scriptable automation with hooks and scheduling

## Installation

### Windows
```bash
# Download from official source
# Install using the provided installer or extract portable version
fastcopy.exe --version
```

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or download DMG and install manually
fastcopy --version
```

### Linux
```bash
# Download portable binary
curl -LO https://example.com/fastcopy-linux-x64
chmod +x fastcopy-linux-x64
sudo mv fastcopy-linux-x64 /usr/local/bin/fastcopy

# Verify installation
fastcopy --version
```

## Basic Usage

### Command Line Interface

#### Simple File Copy
```bash
# Basic syntax
fastcopy <source> <destination> [options]

# Copy single file
fastcopy /path/to/source.zip /backup/

# Copy directory recursively
fastcopy /data/projects/ /backup/projects/ --recursive

# Copy with maximum parallelism
fastcopy /media/raw/ /nas/archive/ --streams 64 --turbo
```

#### Common Options
```bash
# High priority transfer with debug logging
fastcopy source.iso /backups/ --priority high --log-level debug

# Skip integrity verification (faster but less safe)
fastcopy /large_dataset/ /backup/ --no-verify --streams 32

# Resume interrupted transfer
fastcopy /incomplete/ /destination/ --resume always

# Tag transfers for organization
fastcopy build.tar.gz /releases/ --tag "v2.3.1-production"
```

## Profile-Based Configuration

### YAML Profile Structure

Create reusable transfer profiles for consistent operations:

```yaml
# production-backup.yaml
transfer:
  mode: "turbo"                # standard, balanced, turbo
  streams: 32                  # concurrent streams (1-64)
  buffer_size_mb: 256          # memory buffer per stream
  verify: true                 # enable SHA-256 + CRC-32 checks
  resume: always               # always, on_failure, never

paths:
  source: "/var/www/production"
  destination: "/mnt/backup/www"

filters:
  include:
    - "*.php"
    - "*.html"
    - "*.css"
    - "*.js"
  exclude:
    - "cache/*"
    - "temp/*"
    - ".git/*"

schedule:
  type: "cron"                 # one_time, watch, cron
  expression: "0 2 * * *"      # daily at 2 AM

hooks:
  on_start: "echo 'Starting backup...'"
  on_complete: "/opt/scripts/notify-success.sh"
  on_error: "/opt/scripts/alert-admin.sh"

logging:
  level: "info"                # debug, info, warning, error
  file: "/var/log/fastcopy/backup.log"
  max_size_mb: 100
```

### Using Profiles
```bash
# Execute with profile
fastcopy --profile production-backup.yaml

# Override profile settings
fastcopy --profile backup.yaml --streams 48 --priority realtime

# Validate profile without executing
fastcopy --profile backup.yaml --dry-run --validate
```

## Advanced Patterns

### Automated Backup Script

```bash
#!/bin/bash
# daily-backup.sh

# Environment variables
BACKUP_SOURCE="${BACKUP_SOURCE:-/data/production}"
BACKUP_DEST="${BACKUP_DEST:-/mnt/nas/backups}"
FASTCOPY_PROFILE="${FASTCOPY_PROFILE:-/etc/fastcopy/backup.yaml}"
ALERT_EMAIL="${ALERT_EMAIL}"

# Timestamp for versioned backups
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DEST_DIR="${BACKUP_DEST}/${TIMESTAMP}"

# Create profile dynamically
cat > /tmp/backup-${TIMESTAMP}.yaml <<EOF
transfer:
  mode: "turbo"
  streams: 32
  verify: true
  resume: always

paths:
  source: "${BACKUP_SOURCE}"
  destination: "${DEST_DIR}"

logging:
  level: "info"
  file: "/var/log/fastcopy/backup-${TIMESTAMP}.log"

hooks:
  on_complete: "echo 'Backup completed at ${DEST_DIR}' | mail -s 'Backup Success' ${ALERT_EMAIL}"
  on_error: "echo 'Backup FAILED - Check logs' | mail -s 'BACKUP FAILURE' ${ALERT_EMAIL}"
EOF

# Execute backup
fastcopy --profile /tmp/backup-${TIMESTAMP}.yaml --priority high

# Cleanup old backups (keep last 7 days)
find "${BACKUP_DEST}" -type d -mtime +7 -exec rm -rf {} +
```

### Watch Directory for Changes

```yaml
# watch-sync.yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true
  resume: on_failure

paths:
  source: "/home/user/Documents"
  destination: "/mnt/cloud-sync/Documents"

schedule:
  type: "watch"                # monitor source for changes
  interval_seconds: 60         # check every 60 seconds
  debounce_ms: 5000           # wait 5s after last change

filters:
  exclude:
    - ".DS_Store"
    - "*.tmp"
    - "~*"

hooks:
  on_complete: "notify-send 'Documents synced'"
```

### Parallel Multi-Source Backup

```bash
#!/bin/bash
# multi-source-backup.sh

# Define multiple sources
SOURCES=(
  "/var/www/app1:/backup/app1"
  "/var/www/app2:/backup/app2"
  "/home/user/data:/backup/user-data"
)

# Run transfers in parallel
for source_dest in "${SOURCES[@]}"; do
  IFS=':' read -r source dest <<< "$source_dest"
  
  fastcopy "$source" "$dest" \
    --streams 16 \
    --verify \
    --log-file "/var/log/fastcopy/$(basename $source).log" &
done

# Wait for all transfers to complete
wait

echo "All backups completed"
```

## Configuration Files

### Global Configuration
```yaml
# ~/.fastcopy/config.yaml or /etc/fastcopy/config.yaml
global:
  default_mode: "balanced"
  default_streams: 16
  buffer_size_mb: 128
  temp_directory: "/tmp/fastcopy"
  
network:
  timeout_seconds: 300
  retry_attempts: 3
  retry_backoff_ms: 1000

integrity:
  algorithm: "sha256"          # sha256, crc32, both
  verify_on_write: true
  
performance:
  cpu_affinity: true           # pin threads to cores
  io_priority: "normal"        # low, normal, high, realtime
  
logging:
  default_level: "info"
  console_output: true
  log_directory: "/var/log/fastcopy"
```

## API Integration Examples

### OpenAI Integration
```yaml
# ai-categorized-transfer.yaml
transfer:
  mode: "turbo"
  streams: 32

paths:
  source: "/incoming/media"
  destination: "/organized/media"

ai:
  provider: "openai"
  endpoint: "https://api.openai.com/v1"
  api_key_env: "OPENAI_API_KEY"
  model: "gpt-4-turbo"
  
  categorization:
    enabled: true
    prompt: "Analyze this filename and suggest a category folder: {filename}"
    auto_organize: true
```

### Claude Integration
```yaml
# claude-documentation.yaml
transfer:
  mode: "balanced"
  streams: 16

paths:
  source: "/project/codebase"
  destination: "/backup/codebase"

ai:
  provider: "claude"
  endpoint: "https://api.anthropic.com"
  api_key_env: "ANTHROPIC_API_KEY"
  model: "claude-3-opus-20240229"
  
  documentation:
    enabled: true
    generate_summary: true
    output_file: "/backup/transfer-report.md"
```

## Troubleshooting

### Common Issues

#### Transfer Hangs or Stalls
```bash
# Reduce concurrent streams
fastcopy source/ dest/ --streams 8

# Lower buffer size for memory-constrained systems
fastcopy source/ dest/ --buffer-size-mb 64

# Check system resources
fastcopy source/ dest/ --log-level debug
```

#### Permission Errors
```bash
# Run with elevated privileges (use cautiously)
sudo fastcopy /protected/source /backup/

# Or adjust ownership/permissions beforehand
sudo chown -R $USER:$USER /destination/
```

#### Integrity Check Failures
```bash
# Re-run with verification enabled
fastcopy source/ dest/ --verify --retry-on-error

# Use both checksums for maximum reliability
fastcopy source/ dest/ --integrity both --streams 8

# Check source file system for corruption
fsck /dev/source_device
```

#### Network Transfer Issues
```bash
# Increase timeout for slow networks
fastcopy //remote/share /local/ --timeout 600

# Reduce streams for unstable connections
fastcopy //remote/share /local/ --streams 4 --retry-attempts 5

# Use resume mode for unreliable connections
fastcopy //remote/share /local/ --resume always
```

### Logging and Debugging

```bash
# Enable maximum verbosity
fastcopy source/ dest/ --log-level debug --console-output

# Write detailed logs to file
fastcopy source/ dest/ --log-file /tmp/transfer.log --log-level debug

# Monitor transfer statistics
fastcopy source/ dest/ --stats realtime --progress bar
```

### Performance Tuning

```bash
# Maximum performance (high CPU/memory usage)
fastcopy large_dataset/ /backup/ \
  --mode turbo \
  --streams 64 \
  --buffer-size-mb 512 \
  --priority realtime \
  --no-verify

# Balanced (recommended for most use cases)
fastcopy data/ /backup/ \
  --mode balanced \
  --streams 16 \
  --verify

# Low resource usage
fastcopy source/ dest/ \
  --mode standard \
  --streams 4 \
  --buffer-size-mb 64 \
  --priority low
```

## Environment Variables

```bash
# Configuration
export FASTCOPY_CONFIG=/path/to/config.yaml
export FASTCOPY_PROFILE=/path/to/default-profile.yaml

# Credentials for AI integrations
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-ant-...

# Network settings
export FASTCOPY_TIMEOUT=600
export FASTCOPY_RETRY_ATTEMPTS=5

# Performance tuning
export FASTCOPY_STREAMS=32
export FASTCOPY_BUFFER_MB=256

# Logging
export FASTCOPY_LOG_LEVEL=info
export FASTCOPY_LOG_DIR=/var/log/fastcopy
```

## Best Practices

1. **Always enable verification for critical data**: Use `--verify` flag
2. **Use profiles for repeated operations**: Maintain YAML configs in version control
3. **Start with fewer streams and scale up**: Test with `--streams 8` before increasing
4. **Monitor system resources during large transfers**: Use `htop` or Task Manager
5. **Implement proper error handling in automation**: Use hooks for alerting
6. **Keep logs for audit trails**: Configure persistent logging
7. **Test restoration procedures regularly**: Verify backups are recoverable
8. **Use appropriate mode for workload**: Don't always use `turbo` mode
