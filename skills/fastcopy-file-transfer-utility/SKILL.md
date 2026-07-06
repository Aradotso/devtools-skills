---
name: fastcopy-file-transfer-utility
description: High-performance file copying and synchronization tool with multi-threaded transfers and integrity verification
triggers:
  - "how do I use FastCopy to copy files"
  - "configure FastCopy for bulk file transfer"
  - "set up FastCopy profile for sync"
  - "FastCopy command line options"
  - "optimize FastCopy transfer speed"
  - "create FastCopy YAML configuration"
  - "FastCopy parallel streams setup"
  - "verify file integrity with FastCopy"
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and synchronization utility designed for accelerated data duplication. It provides multi-threaded transfer capabilities, intelligent caching, integrity verification, and automated resume functionality. The tool supports both GUI and CLI modes with declarative YAML-based configuration profiles.

**Key capabilities:**
- Parallel multi-stream transfers (up to 64 concurrent streams)
- SHA-256 + CRC-32 integrity verification
- Automatic retry and resume on failures
- Profile-based configuration with scheduling
- Cross-platform support (Windows, macOS, Linux)
- Hook-based automation for pipeline integration

## Installation

### Windows
Download the portable executable or installer from the official distribution. No system installation required for portable mode.

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or download the macOS binary
curl -O https://[distribution-url]/fastcopy-macos
chmod +x fastcopy-macos
sudo mv fastcopy-macos /usr/local/bin/fastcopy
```

### Linux
```bash
# Download binary
wget https://[distribution-url]/fastcopy-linux
chmod +x fastcopy-linux
sudo mv fastcopy-linux /usr/local/bin/fastcopy

# Or install from package manager
sudo apt install fastcopy  # Debian/Ubuntu
sudo yum install fastcopy  # RHEL/CentOS
```

Verify installation:
```bash
fastcopy --version
```

## Command Line Interface

### Basic Syntax
```bash
fastcopy [SOURCE] [DESTINATION] [OPTIONS]
```

### Key Commands

#### Simple File Copy
```bash
# Copy single file
fastcopy /path/to/source.file /path/to/destination/

# Copy directory recursively
fastcopy /source/directory/ /destination/directory/ --recursive
```

#### High-Performance Transfer
```bash
# Maximum speed with 64 streams
fastcopy /large/dataset/ /backup/location/ \
  --streams 64 \
  --buffer-size 256 \
  --mode turbo \
  --priority high

# With integrity verification
fastcopy /critical/data/ /archive/ \
  --streams 32 \
  --verify \
  --checksum sha256
```

#### Profile-Based Execution
```bash
# Use predefined profile
fastcopy --profile /path/to/profile.yaml

# Override profile settings
fastcopy --profile backup-profile.yaml \
  --streams 48 \
  --log-level debug
```

#### Resume and Retry
```bash
# Auto-resume interrupted transfers
fastcopy /source/ /dest/ \
  --resume always \
  --retry-attempts 5 \
  --retry-delay 30

# Continue specific transfer session
fastcopy --resume-session a3f2b9c1
```

#### Monitoring and Logging
```bash
# Verbose output with progress
fastcopy /data/ /backup/ \
  --verbose \
  --progress \
  --log-file /var/log/fastcopy.log

# JSON output for parsing
fastcopy /src/ /dst/ --output-format json
```

### Common Options

| Option | Description |
|--------|-------------|
| `--streams N` | Number of concurrent streams (1-64) |
| `--buffer-size MB` | Buffer size in megabytes |
| `--mode MODE` | Transfer mode: standard, balanced, turbo |
| `--verify` | Enable integrity verification |
| `--checksum ALGO` | Checksum algorithm: crc32, sha256, both |
| `--resume MODE` | Resume mode: always, on_failure, never |
| `--priority LEVEL` | Process priority: low, normal, high |
| `--recursive` | Copy directories recursively |
| `--exclude PATTERN` | Exclude files matching pattern |
| `--include PATTERN` | Include only files matching pattern |
| `--dry-run` | Simulate transfer without copying |
| `--tag NAME` | Tag transfer with identifier |

## YAML Profile Configuration

### Basic Profile Structure

```yaml
# fastcopy-profile.yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
  verify: true
  checksum: "sha256"
  resume: "always"
  retry_attempts: 3
  retry_delay_seconds: 30

paths:
  source: "/mnt/storage/data"
  destination: "/backup/archive"
  recursive: true

filters:
  include:
    - "*.mp4"
    - "*.mkv"
  exclude:
    - "*.tmp"
    - ".DS_Store"
    - "Thumbs.db"

schedule:
  type: "one_time"  # one_time, watch, cron
  
logging:
  level: "info"  # debug, info, warn, error
  file: "/var/log/fastcopy/transfer.log"
  format: "json"

hooks:
  on_start: "echo 'Transfer started at $(date)'"
  on_complete: "notify-send 'FastCopy' 'Transfer complete'"
  on_error: "/opt/scripts/alert-team.sh"
```

### Watch Mode Profile

```yaml
# watch-sync-profile.yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true
  resume: "always"

paths:
  source: "/home/user/Documents"
  destination: "//nas/backups/documents"
  recursive: true

schedule:
  type: "watch"
  interval_seconds: 300  # Check every 5 minutes
  debounce_seconds: 10   # Wait 10s after last change

filters:
  exclude:
    - "*.lock"
    - "*.swp"
    - ".git/**"
    - "node_modules/**"

hooks:
  on_complete: "logger 'FastCopy sync completed'"
```

### Cron-Based Profile

```yaml
# nightly-backup-profile.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  priority: "high"

paths:
  source: "/var/lib/database/backups"
  destination: "/mnt/offsite/db_backups"
  recursive: true

schedule:
  type: "cron"
  expression: "0 2 * * *"  # Daily at 2 AM

retention:
  keep_days: 30
  cleanup_old: true

logging:
  level: "info"
  file: "/var/log/fastcopy/nightly-backup.log"

hooks:
  on_start: "/opt/scripts/pre-backup.sh"
  on_complete: "/opt/scripts/post-backup.sh --success"
  on_error: "/opt/scripts/post-backup.sh --failed"
```

## Common Usage Patterns

### Pattern 1: Large Media File Migration

```bash
#!/bin/bash
# Migrate video production files to NAS

fastcopy /production/raw_footage/ //nas/archive/project_alpha/ \
  --streams 64 \
  --buffer-size 512 \
  --mode turbo \
  --verify \
  --checksum sha256 \
  --include "*.mp4" \
  --include "*.mov" \
  --exclude "*.tmp" \
  --resume always \
  --retry-attempts 5 \
  --progress \
  --tag "project_alpha_migration_$(date +%Y%m%d)" \
  --log-file /var/log/migration.log
```

### Pattern 2: Incremental Backup Script

```bash
#!/bin/bash
# Incremental backup with metadata preservation

PROFILE="/etc/fastcopy/incremental-backup.yaml"
LOG_DIR="/var/log/fastcopy"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

fastcopy --profile "$PROFILE" \
  --log-file "$LOG_DIR/backup_$TIMESTAMP.log" \
  --output-format json > "$LOG_DIR/backup_$TIMESTAMP.json"

if [ $? -eq 0 ]; then
  echo "Backup completed successfully" | mail -s "Backup Success" admin@example.com
else
  echo "Backup failed - check logs" | mail -s "Backup FAILED" admin@example.com
  exit 1
fi
```

### Pattern 3: Scheduled Sync with Monitoring

```yaml
# production-sync.yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 256
  verify: true
  resume: "on_failure"

paths:
  source: "/app/data/uploads"
  destination: "/mnt/replicas/app_uploads"
  recursive: true

schedule:
  type: "watch"
  interval_seconds: 60

filters:
  exclude:
    - "*.partial"
    - ".processing/**"

metrics:
  enabled: true
  endpoint: "http://monitoring.local:9090/metrics"
  interval_seconds: 30

hooks:
  on_error: |
    curl -X POST https://alerts.local/webhook \
      -H "Content-Type: application/json" \
      -d '{"alert": "FastCopy sync failed", "severity": "high"}'
```

### Pattern 4: Development Environment Sync

```yaml
# dev-sync.yaml
transfer:
  mode: "standard"
  streams: 8
  verify: false  # Speed over verification for dev files

paths:
  source: "/home/dev/projects"
  destination: "/mnt/dev-backup/projects"
  recursive: true

schedule:
  type: "watch"
  interval_seconds: 120
  debounce_seconds: 5

filters:
  exclude:
    - "node_modules/**"
    - ".git/**"
    - "*.pyc"
    - "__pycache__/**"
    - ".venv/**"
    - "build/**"
    - "dist/**"
    - ".next/**"
    - "target/**"

logging:
  level: "warn"
  file: "/home/dev/.fastcopy/sync.log"
```

## Environment Variables

FastCopy supports configuration through environment variables:

```bash
# Set default behavior
export FASTCOPY_STREAMS=32
export FASTCOPY_BUFFER_SIZE=256
export FASTCOPY_MODE=turbo
export FASTCOPY_VERIFY=true
export FASTCOPY_LOG_LEVEL=info

# Custom profile directory
export FASTCOPY_PROFILE_DIR=/etc/fastcopy/profiles

# Credentials for network shares (if needed)
export FASTCOPY_NAS_USER="${NAS_USERNAME}"
export FASTCOPY_NAS_PASS="${NAS_PASSWORD}"

# API integration (if enabled)
export OPENAI_API_KEY="${OPENAI_API_KEY}"
export ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"
```

## API Integration Examples

### OpenAI Content Analysis

```yaml
# ai-categorized-backup.yaml
transfer:
  mode: "balanced"
  streams: 16

paths:
  source: "/incoming/media"
  destination: "/archive/categorized"

openai:
  endpoint: "https://api.openai.com/v1"
  api_key_env: "OPENAI_API_KEY"
  model: "gpt-4-turbo"
  categorize: true
  prompt_template: |
    Analyze this file and categorize it: {filename}
    Return only one word: documentary, commercial, tutorial, or raw

hooks:
  on_file_categorized: "/opt/scripts/organize-by-category.sh"
```

### Claude Documentation Generation

```yaml
# documented-transfer.yaml
transfer:
  mode: "turbo"
  streams: 32

paths:
  source: "/projects/archive"
  destination: "/knowledge-base/archived-projects"

claude:
  api_base: "https://api.anthropic.com"
  api_key_env: "ANTHROPIC_API_KEY"
  model: "claude-3-opus-20240229"
  generate_summary: true
  summary_output: "/knowledge-base/summaries"

hooks:
  on_complete: "/opt/scripts/index-knowledge-base.sh"
```

## Troubleshooting

### Transfer Speed Issues

**Problem:** Transfers slower than expected

**Solutions:**
```bash
# Increase streams and buffer size
fastcopy /src/ /dst/ --streams 64 --buffer-size 512 --mode turbo

# Check system resources
fastcopy /src/ /dst/ --priority high --verbose

# Disable verification for non-critical transfers
fastcopy /src/ /dst/ --no-verify

# Check network bottlenecks
fastcopy /src/ /dst/ --stats-interval 5 --log-level debug
```

### Resume Failed Transfers

**Problem:** Transfer interrupted, need to resume

**Solutions:**
```bash
# Check available sessions
fastcopy --list-sessions

# Resume specific session
fastcopy --resume-session <SESSION_ID>

# Auto-resume with retry
fastcopy /src/ /dst/ \
  --resume always \
  --retry-attempts 10 \
  --retry-delay 60
```

### Integrity Verification Failures

**Problem:** Checksum mismatches detected

**Solutions:**
```bash
# Re-verify existing transfer
fastcopy --verify-only /src/ /dst/ --checksum sha256

# Use both checksums for maximum confidence
fastcopy /src/ /dst/ --checksum both --verify

# Export verification report
fastcopy --verify-only /src/ /dst/ \
  --output-format json > verification-report.json
```

### Permission Issues

**Problem:** Access denied on source or destination

**Solutions:**
```bash
# Run with elevated privileges (if appropriate)
sudo fastcopy /src/ /dst/

# Preserve permissions
fastcopy /src/ /dst/ --preserve-permissions --preserve-ownership

# Skip inaccessible files
fastcopy /src/ /dst/ --skip-errors --log-errors /tmp/errors.log
```

### High Memory Usage

**Problem:** FastCopy consuming too much memory

**Solutions:**
```bash
# Reduce buffer size
fastcopy /src/ /dst/ --buffer-size 128 --streams 16

# Use standard mode instead of turbo
fastcopy /src/ /dst/ --mode standard

# Limit concurrent streams
fastcopy /src/ /dst/ --streams 8 --priority normal
```

### Profile Not Found

**Problem:** Cannot load YAML profile

**Solutions:**
```bash
# Verify profile path
fastcopy --profile /full/path/to/profile.yaml --dry-run

# Check YAML syntax
fastcopy --validate-profile /path/to/profile.yaml

# Use absolute paths in profile
# In YAML, use full paths for source/destination
```

## Advanced Techniques

### Parallel Multi-Destination Sync

```bash
#!/bin/bash
# Sync to multiple destinations simultaneously

DESTINATIONS=(
  "/backup/local"
  "//nas01/backup"
  "//nas02/backup"
)

for dest in "${DESTINATIONS[@]}"; do
  fastcopy /source/ "$dest" \
    --streams 32 \
    --verify \
    --tag "multi-dest-$(basename $dest)" \
    --log-file "/var/log/fastcopy/$(basename $dest).log" &
done

wait
echo "All transfers complete"
```

### Bandwidth Throttling

```bash
# Limit transfer speed during business hours
HOUR=$(date +%H)

if [ $HOUR -ge 9 ] && [ $HOUR -le 17 ]; then
  # Business hours - throttle
  fastcopy /src/ /dst/ --max-bandwidth 100M --streams 8
else
  # Off hours - full speed
  fastcopy /src/ /dst/ --streams 64 --mode turbo
fi
```

### Pre/Post Processing Pipeline

```bash
#!/bin/bash
# Complete backup pipeline with validation

# Pre-backup snapshot
/opt/scripts/create-snapshot.sh /source/

# Transfer with FastCopy
fastcopy --profile /etc/fastcopy/backup.yaml \
  --log-file /var/log/backup_$(date +%Y%m%d).log

# Post-backup validation
if [ $? -eq 0 ]; then
  /opt/scripts/verify-backup.sh /destination/
  /opt/scripts/cleanup-old-backups.sh --keep 30
  /opt/scripts/update-inventory.sh
else
  /opt/scripts/alert-failure.sh
  exit 1
fi
```

This skill covers the essential usage patterns, configuration options, and troubleshooting steps for FastCopy based on the project documentation.
