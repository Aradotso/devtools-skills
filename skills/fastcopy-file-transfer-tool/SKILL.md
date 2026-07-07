---
name: fastcopy-file-transfer-tool
description: Ultra-fast file copying and synchronization utility with parallel streams, integrity verification, and automation features
triggers:
  - "how do I use FastCopy for file transfers"
  - "configure FastCopy for bulk file migration"
  - "set up parallel file copying with FastCopy"
  - "FastCopy profile configuration examples"
  - "optimize FastCopy transfer speeds"
  - "automate file synchronization with FastCopy"
  - "verify file integrity during FastCopy operations"
  - "schedule recurring transfers with FastCopy"
---

# FastCopy File Transfer Tool Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and synchronization utility designed for professionals who need to move large volumes of data efficiently. It provides parallel stream processing, intelligent caching, integrity verification, and automation capabilities through both CLI and configuration profiles.

**Key Features:**
- Multi-threaded parallel transfers (up to 64 concurrent streams)
- Profile-based configuration system using YAML
- Automatic retry and resume on failures
- SHA-256 + CRC-32 integrity verification
- Watch mode for continuous synchronization
- Cross-platform support (Windows, macOS, Linux)
- CLI and GUI interfaces

## Installation

### Windows
```powershell
# Download and extract the portable version
# Run the installer or use the portable executable
fastcopy.exe --version
```

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or download the .dmg and install
```

### Linux
```bash
# Debian/Ubuntu
sudo apt install fastcopy

# Fedora
sudo dnf install fastcopy

# Arch Linux
yay -S fastcopy

# Verify installation
fastcopy --version
```

## CLI Commands

### Basic Syntax
```bash
fastcopy [SOURCE] [DESTINATION] [OPTIONS]
```

### Key Commands

**Simple Copy:**
```bash
# Copy single file
fastcopy /path/to/source.file /path/to/destination/

# Copy entire directory
fastcopy /source/directory/ /destination/directory/
```

**High-Speed Transfer:**
```bash
# Maximum speed with 64 parallel streams
fastcopy /large/dataset/ /backup/location/ --streams 64 --mode turbo

# Copy with specific buffer size
fastcopy source.iso /backups/ --buffer 256 --streams 32
```

**Integrity Verification:**
```bash
# Copy with verification enabled
fastcopy /important/data/ /backup/ --verify

# Skip verification for speed
fastcopy /temp/files/ /destination/ --no-verify
```

**Resume and Retry:**
```bash
# Enable automatic resume on failure
fastcopy /media/ /nas/ --resume always --retry 5

# Set retry backoff interval
fastcopy /files/ /backup/ --resume on_failure --retry-delay 10
```

**Logging and Monitoring:**
```bash
# Enable debug logging
fastcopy /data/ /backup/ --log-level debug --log-file transfer.log

# Set priority level
fastcopy /source/ /dest/ --priority high
```

## Profile Configuration

FastCopy uses YAML profiles for complex or recurring transfer operations.

### Basic Profile Structure

**basic-transfer.yaml:**
```yaml
transfer:
  mode: "balanced"              # standard, balanced, turbo
  streams: 16                   # concurrent streams (1-64)
  buffer_size_mb: 128
  verify: true
  resume: on_failure            # always, on_failure, never

paths:
  source: "/data/projects"
  destination: "/backup/projects"

options:
  overwrite: true
  preserve_permissions: true
  preserve_timestamps: true
  follow_symlinks: false
```

### Advanced Profile with Scheduling

**scheduled-backup.yaml:**
```yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
  verify: true
  resume: always

paths:
  source: "/mnt/storage/raw_footage"
  destination: "//nas/archive/project_alpha"

schedule:
  type: "cron"                  # one_time, watch, cron
  expression: "0 2 * * *"       # Daily at 2 AM
  timezone: "UTC"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"
  exclude:
    - "*.tmp"
    - ".DS_Store"
    - "Thumbs.db"

hooks:
  on_start: "echo 'Starting backup...'"
  on_complete: "/opt/scripts/notify-success.sh"
  on_error: "/opt/scripts/alert-failure.sh"
```

### Watch Mode Profile

**continuous-sync.yaml:**
```yaml
transfer:
  mode: "balanced"
  streams: 8
  buffer_size_mb: 64
  verify: true
  resume: always

paths:
  source: "/home/user/documents"
  destination: "/backup/documents"

schedule:
  type: "watch"
  interval_seconds: 300         # Check every 5 minutes
  debounce_ms: 1000            # Wait 1s after last change

options:
  sync_delete: false            # Don't delete from destination
  bidirectional: false
  conflict_resolution: "newer"  # newer, source, destination

logging:
  level: "info"
  file: "/var/log/fastcopy/documents-sync.log"
  rotate_size_mb: 50
```

## Using Profiles

### Load and Execute Profile
```bash
# Run with profile
fastcopy --profile backup-profile.yaml

# Override profile settings
fastcopy --profile backup-profile.yaml --streams 64 --verify false

# Dry run to see what would be copied
fastcopy --profile backup-profile.yaml --dry-run

# Run in background
fastcopy --profile continuous-sync.yaml --daemon
```

## Code Examples

### Shell Script Automation

**automated-backup.sh:**
```bash
#!/bin/bash

# Environment variables
export FASTCOPY_LOG_DIR="/var/log/fastcopy"
export FASTCOPY_PROFILE_DIR="/etc/fastcopy/profiles"

# Daily backup script
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="${FASTCOPY_LOG_DIR}/backup_${TIMESTAMP}.log"

echo "Starting backup at $(date)" | tee -a "$LOG_FILE"

# Run FastCopy with high priority
fastcopy \
  /data/production \
  /backup/daily/production_${TIMESTAMP} \
  --mode turbo \
  --streams 32 \
  --verify \
  --resume always \
  --log-file "$LOG_FILE" \
  --priority high

if [ $? -eq 0 ]; then
    echo "Backup completed successfully" | tee -a "$LOG_FILE"
    # Send success notification
    curl -X POST "$NOTIFICATION_WEBHOOK" \
      -H "Content-Type: application/json" \
      -d "{\"status\": \"success\", \"timestamp\": \"$TIMESTAMP\"}"
else
    echo "Backup failed" | tee -a "$LOG_FILE"
    # Send failure alert
    curl -X POST "$ALERT_WEBHOOK" \
      -H "Content-Type: application/json" \
      -d "{\"status\": \"failed\", \"timestamp\": \"$TIMESTAMP\"}"
    exit 1
fi
```

### Batch Processing

**multi-source-sync.sh:**
```bash
#!/bin/bash

# Sync multiple directories in parallel
SOURCES=(
    "/data/project_a"
    "/data/project_b"
    "/data/project_c"
)

DEST_BASE="/backup/projects"

for source in "${SOURCES[@]}"; do
    project_name=$(basename "$source")
    
    # Run each in background
    fastcopy "$source" "${DEST_BASE}/${project_name}" \
        --mode balanced \
        --streams 16 \
        --verify \
        --log-file "/var/log/fastcopy/${project_name}.log" &
done

# Wait for all background jobs
wait

echo "All syncs completed"
```

### Conditional Transfer Script

**smart-backup.sh:**
```bash
#!/bin/bash

SOURCE_DIR="/data/media"
DEST_DIR="/backup/media"
MIN_FREE_SPACE_GB=100

# Check available space
available_space=$(df -BG "$DEST_DIR" | awk 'NR==2 {print $4}' | sed 's/G//')

if [ "$available_space" -lt "$MIN_FREE_SPACE_GB" ]; then
    echo "Insufficient space: ${available_space}GB available, need ${MIN_FREE_SPACE_GB}GB"
    exit 1
fi

# Check if source has changed
if [ -f "${DEST_DIR}/.last_sync" ]; then
    last_sync=$(cat "${DEST_DIR}/.last_sync")
    changed_files=$(find "$SOURCE_DIR" -newer "${DEST_DIR}/.last_sync" -type f | wc -l)
    
    if [ "$changed_files" -eq 0 ]; then
        echo "No changes detected since last sync"
        exit 0
    fi
    
    echo "Found $changed_files changed files"
fi

# Perform transfer
fastcopy "$SOURCE_DIR" "$DEST_DIR" \
    --mode turbo \
    --streams 32 \
    --verify \
    --resume always

if [ $? -eq 0 ]; then
    date +%s > "${DEST_DIR}/.last_sync"
    echo "Sync completed successfully"
fi
```

## Configuration Files

### Global Configuration

**~/.fastcopy/config.yaml:**
```yaml
defaults:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: on_failure

ui:
  theme: "dark"
  show_progress: true
  notification_level: "important"

paths:
  log_directory: "~/.fastcopy/logs"
  profile_directory: "~/.fastcopy/profiles"
  temp_directory: "/tmp/fastcopy"

performance:
  max_memory_mb: 4096
  cpu_affinity: []              # Empty = use all cores
  io_priority: "normal"         # low, normal, high

network:
  tcp_buffer_kb: 256
  retry_timeout_seconds: 30
  max_retries: 5
  backoff_multiplier: 2.0
```

### Environment Variables

```bash
# Configuration
export FASTCOPY_CONFIG_DIR="$HOME/.fastcopy"
export FASTCOPY_PROFILE_DIR="$HOME/.fastcopy/profiles"
export FASTCOPY_LOG_DIR="/var/log/fastcopy"

# Performance tuning
export FASTCOPY_MAX_STREAMS=64
export FASTCOPY_BUFFER_SIZE_MB=256
export FASTCOPY_MAX_MEMORY_MB=8192

# Behavior
export FASTCOPY_DEFAULT_MODE="turbo"
export FASTCOPY_VERIFY_DEFAULT=true
export FASTCOPY_RESUME_DEFAULT="always"

# Integration
export FASTCOPY_WEBHOOK_URL="https://hooks.example.com/fastcopy"
export FASTCOPY_OPENAI_API_KEY="${OPENAI_API_KEY}"
export FASTCOPY_CLAUDE_API_KEY="${CLAUDE_API_KEY}"
```

## Common Patterns

### Large File Migration
```bash
# Migrate multi-terabyte dataset with maximum efficiency
fastcopy /storage/raw_data/ /archive/raw_data/ \
    --mode turbo \
    --streams 64 \
    --buffer 512 \
    --verify \
    --resume always \
    --priority high \
    --log-file migration-$(date +%Y%m%d).log
```

### Incremental Backup
```yaml
# incremental-backup.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always

paths:
  source: "/data"
  destination: "/backup/incremental/$(date +%Y%m%d)"

options:
  incremental: true
  compare_method: "timestamp"   # timestamp, size, checksum
  overwrite: false

filters:
  exclude:
    - "*.tmp"
    - "*.cache"
    - "node_modules/"
    - ".git/"
```

### Network Transfer with Compression
```bash
# Transfer to network share with optimal settings
fastcopy /local/data/ \\\\server\\share\\data\\ \
    --mode balanced \
    --streams 8 \
    --buffer 64 \
    --verify \
    --compress \
    --network-optimize \
    --retry 10
```

### Selective Sync with Filters
```yaml
# media-sync.yaml
transfer:
  mode: "turbo"
  streams: 24
  buffer_size_mb: 256
  verify: true

paths:
  source: "/media/raw"
  destination: "/nas/processed"

filters:
  include:
    - "**/*.mp4"
    - "**/*.mov"
    - "**/*.avi"
    - "**/*.mkv"
  exclude:
    - "**/draft/**"
    - "**/.tmp/**"
    - "**/proxy/**"

options:
  min_file_size_mb: 1
  max_file_age_days: 30
```

## Troubleshooting

### Check Transfer Status
```bash
# View active transfers
fastcopy --status

# View detailed transfer info
fastcopy --status --verbose

# Check specific transfer by ID
fastcopy --status --id transfer_12345
```

### Performance Issues

**Slow transfers:**
```bash
# Test with different stream counts
for streams in 8 16 32 64; do
    echo "Testing with $streams streams"
    time fastcopy /test/source /test/dest --streams $streams --no-verify
done

# Monitor system resources during transfer
fastcopy /source /dest --streams 32 --monitor &
PID=$!
top -p $PID
```

**High CPU usage:**
```yaml
# Reduce parallel operations
transfer:
  mode: "standard"
  streams: 4                    # Reduce from default
  buffer_size_mb: 64           # Smaller buffer

performance:
  cpu_affinity: [0, 1]         # Limit to specific cores
  io_priority: "low"
```

### Network Transfer Issues

**Connection timeouts:**
```yaml
network:
  retry_timeout_seconds: 60     # Increase timeout
  max_retries: 10
  backoff_multiplier: 1.5
  tcp_buffer_kb: 512           # Increase buffer for high-latency

transfer:
  streams: 4                    # Reduce streams for unreliable connections
  mode: "standard"
```

### Verification Failures

**Enable detailed verification logging:**
```bash
fastcopy /source /dest \
    --verify \
    --checksum-algorithm sha256 \
    --log-level debug \
    --verify-log verify-report.txt

# Review verification report
cat verify-report.txt | grep MISMATCH
```

### Resume Interrupted Transfers
```bash
# Automatic resume from previous state
fastcopy /source /dest --resume always

# Manual resume with transfer ID
fastcopy --resume-transfer transfer_12345

# Check resume state
fastcopy --show-resume-state /dest
```

### Debug Mode
```bash
# Enable full debug output
fastcopy /source /dest \
    --log-level trace \
    --debug \
    --log-file debug-transfer.log

# Enable performance profiling
fastcopy /source /dest \
    --profile-performance \
    --output-stats stats.json
```

### Common Error Codes

| Code | Meaning | Solution |
|------|---------|----------|
| 1 | Permission denied | Check file/directory permissions |
| 2 | Source not found | Verify source path exists |
| 3 | Destination error | Check destination write permissions |
| 4 | Verification failed | Re-run with `--verify --retry 5` |
| 5 | Out of space | Free space or change destination |
| 6 | Network timeout | Increase timeout, reduce streams |
| 7 | Interrupted | Use `--resume always` |

## Integration Examples

### CI/CD Pipeline (GitHub Actions)
```yaml
name: Deploy Artifacts
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install FastCopy
        run: |
          wget https://example.com/fastcopy-linux-amd64
          chmod +x fastcopy-linux-amd64
          sudo mv fastcopy-linux-amd64 /usr/local/bin/fastcopy
      
      - name: Deploy to production
        env:
          DEPLOY_HOST: ${{ secrets.DEPLOY_HOST }}
          DEPLOY_PATH: ${{ secrets.DEPLOY_PATH }}
        run: |
          fastcopy ./dist/ "${DEPLOY_HOST}:${DEPLOY_PATH}" \
            --mode turbo \
            --streams 32 \
            --verify \
            --resume always
```

### Monitoring Integration
```bash
#!/bin/bash
# Send metrics to monitoring system

TRANSFER_ID=$(fastcopy /source /dest --streams 32 --background)

while true; do
    STATUS=$(fastcopy --status --id "$TRANSFER_ID" --format json)
    
    # Extract metrics
    PROGRESS=$(echo "$STATUS" | jq -r '.progress')
    SPEED=$(echo "$STATUS" | jq -r '.speed_mbps')
    
    # Send to monitoring
    curl -X POST "$METRICS_ENDPOINT" \
        -H "Content-Type: application/json" \
        -d "{\"transfer_id\": \"$TRANSFER_ID\", \"progress\": $PROGRESS, \"speed\": $SPEED}"
    
    sleep 10
done
```

This skill provides comprehensive guidance for using FastCopy in development workflows, automation scripts, and production environments.
