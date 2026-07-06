---
name: fastcopy-file-transfer-optimizer
description: High-performance file copying and synchronization utility with multi-threaded operations and intelligent caching
triggers:
  - how do I use fastcopy to copy files quickly
  - configure fastcopy for maximum speed
  - fastcopy parallel transfer setup
  - optimize file copying with fastcopy
  - fastcopy profile configuration examples
  - troubleshoot fastcopy transfer errors
  - setup fastcopy automation script
  - fastcopy cli commands and options
---

# FastCopy File Transfer Optimizer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file copying and transfer utility designed for professional workflows requiring speed, reliability, and automation. It provides multi-threaded I/O operations, intelligent caching, integrity verification, and supports cross-platform deployment.

**Key capabilities:**
- Parallel stream processing (up to 64 concurrent streams)
- YAML-based profile configuration
- Automatic retry and resume on failure
- SHA-256/CRC-32 integrity verification
- CLI and GUI modes
- Cross-platform support (Windows, macOS, Linux)

## Installation

### Windows
```powershell
# Download and extract portable version
# Or use installer from official source
fastcopy.exe --install
```

### macOS
```bash
# Via Homebrew (if available)
brew install fastcopy

# Or manual installation
curl -O https://example.com/fastcopy-macos.pkg
sudo installer -pkg fastcopy-macos.pkg -target /
```

### Linux
```bash
# Debian/Ubuntu
sudo dpkg -i fastcopy_amd64.deb

# From source
tar -xzf fastcopy-linux.tar.gz
cd fastcopy
sudo make install
```

## CLI Commands

### Basic Usage

```bash
# Simple file copy
fastcopy source.file /destination/path/

# Directory copy with verification
fastcopy /source/dir /destination/dir --verify

# High-speed transfer with maximum streams
fastcopy large_file.iso /backup/ --streams 64 --priority high

# Dry run to preview operations
fastcopy /source /dest --dry-run

# Copy with progress logging
fastcopy /data /backup --log-level info --progress
```

### Advanced Options

```bash
# Custom buffer size and no verification for speed
fastcopy /media /archive --buffer-size 512 --no-verify

# Resume interrupted transfer
fastcopy /source /dest --resume

# Tag transfer for tracking
fastcopy project.tar.gz /backups/ --tag "release_v2.3"

# Exclude patterns
fastcopy /project /backup --exclude "*.tmp" --exclude "node_modules"

# Schedule monitoring (watch mode)
fastcopy /watched/dir /sync/target --watch --interval 60
```

## Profile Configuration

FastCopy uses YAML profiles for complex, reusable transfer configurations.

### Basic Profile

Create `transfer-profile.yaml`:

```yaml
transfer:
  mode: "balanced"              # standard, balanced, turbo
  streams: 16                   # concurrent streams (1-64)
  buffer_size_mb: 128
  verify: true
  resume: always                # always, on_failure, never

paths:
  source: "/mnt/data/projects"
  destination: "/backup/projects"

options:
  exclude:
    - "*.log"
    - "*.tmp"
    - ".git"
  include_hidden: false
  preserve_timestamps: true
  preserve_permissions: true
```

### High-Performance Profile

```yaml
# turbo-transfer.yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: false                 # Disable for max speed
  resume: on_failure
  compression: false            # Disable if network is fast

paths:
  source: "/storage/raw_footage"
  destination: "//nas/media/archive"

performance:
  priority: "high"              # low, normal, high, realtime
  cpu_affinity: [0, 1, 2, 3]   # Bind to specific cores
  io_priority: "realtime"

logging:
  level: "warning"              # debug, info, warning, error
  file: "/var/log/fastcopy/turbo.log"
```

### Automated Backup Profile

```yaml
# scheduled-backup.yaml
transfer:
  mode: "balanced"
  streams: 32
  buffer_size_mb: 256
  verify: true
  resume: always

paths:
  source: "/home/user/documents"
  destination: "/mnt/backup/documents"

schedule:
  type: "cron"                  # one_time, watch, cron
  expression: "0 2 * * *"       # Daily at 2 AM

hooks:
  on_start: "echo 'Backup started' | mail -s 'FastCopy' admin@example.com"
  on_complete: "notify-send 'Backup Complete'"
  on_error: "/opt/scripts/alert.sh"

filters:
  min_size_kb: 1                # Skip very small files
  max_size_gb: 100              # Skip extremely large files
  modified_within_days: 7       # Only recent files
```

### Using Profiles

```bash
# Execute with profile
fastcopy --profile transfer-profile.yaml

# Override profile settings
fastcopy --profile turbo-transfer.yaml --streams 32 --verify

# Validate profile without executing
fastcopy --profile backup.yaml --validate
```

## Common Patterns

### Large File Migration

```bash
#!/bin/bash
# migrate-large-files.sh

SOURCE="/old/storage"
DEST="/new/storage"
LOG="/var/log/fastcopy/migration.log"

fastcopy "$SOURCE" "$DEST" \
  --streams 48 \
  --buffer-size 512 \
  --verify \
  --resume always \
  --log-level info \
  --log-file "$LOG" \
  --exclude "*.tmp" \
  --priority high
```

### Incremental Sync

```bash
#!/bin/bash
# incremental-sync.sh

fastcopy /workspace /backup \
  --mode differential \
  --verify \
  --preserve-all \
  --delete-extra \
  --log-level warning
```

### Network Transfer Optimization

```yaml
# network-optimized.yaml
transfer:
  mode: "turbo"
  streams: 32
  buffer_size_mb: 256
  verify: true
  compression: true             # Enable for slow networks
  
network:
  tcp_window_size_kb: 1024
  max_retries: 5
  retry_delay_ms: 1000
  timeout_seconds: 300

paths:
  source: "/local/data"
  destination: "//remote-server/share/data"
```

### Batch Processing

```bash
#!/bin/bash
# batch-copy.sh

# Copy multiple directories with different profiles
for dir in project1 project2 project3; do
  fastcopy "/source/$dir" "/dest/$dir" \
    --profile standard-transfer.yaml \
    --tag "$dir-backup-$(date +%Y%m%d)" \
    --log-file "/var/log/fastcopy/$dir.log"
done
```

## Integration Examples

### CI/CD Pipeline Integration

```yaml
# .github/workflows/deploy.yml
name: Deploy Artifacts
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build artifacts
        run: npm run build
      
      - name: Deploy with FastCopy
        run: |
          fastcopy ./dist /mnt/cdn/releases/${{ github.sha }} \
            --streams 32 \
            --verify \
            --tag "release-${{ github.sha }}" \
            --log-level info
```

### Backup Script with Hooks

```bash
#!/bin/bash
# automated-backup.sh

export BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
export BACKUP_DEST="/backups/daily/$BACKUP_DATE"

# Pre-backup hook
pre_backup() {
  mkdir -p "$BACKUP_DEST"
  echo "Starting backup to $BACKUP_DEST"
}

# Post-backup hook
post_backup() {
  if [ $? -eq 0 ]; then
    echo "Backup completed successfully" | mail -s "Backup Success" admin@example.com
    # Cleanup old backups
    find /backups/daily -mtime +30 -delete
  else
    echo "Backup FAILED" | mail -s "Backup Error" admin@example.com
  fi
}

pre_backup

fastcopy /data "$BACKUP_DEST" \
  --profile production-backup.yaml \
  --streams 48 \
  --verify

post_backup
```

### Monitoring Script

```bash
#!/bin/bash
# monitor-transfers.sh

# Monitor active FastCopy processes
watch_transfers() {
  fastcopy --status --json | jq '.active_transfers[] | {
    id: .transfer_id,
    progress: .progress_percent,
    speed: .current_speed_mbps,
    eta: .estimated_completion
  }'
}

# Parse logs for errors
check_errors() {
  grep -i "error\|failed" /var/log/fastcopy/*.log | tail -20
}

watch_transfers
check_errors
```

## Configuration Files

### Global Configuration

Create `~/.fastcopy/config.yaml`:

```yaml
defaults:
  transfer:
    mode: "balanced"
    streams: 16
    buffer_size_mb: 128
    verify: true
  
  logging:
    level: "info"
    directory: "~/.fastcopy/logs"
    max_size_mb: 100
    max_age_days: 30
  
  performance:
    priority: "normal"
    max_cpu_percent: 80
    max_memory_mb: 4096

profiles_directory: "~/.fastcopy/profiles"
plugins_directory: "~/.fastcopy/plugins"
```

### Environment Variables

```bash
# Set in ~/.bashrc or ~/.zshrc

export FASTCOPY_CONFIG="$HOME/.fastcopy/config.yaml"
export FASTCOPY_LOG_LEVEL="info"
export FASTCOPY_BUFFER_SIZE="256"
export FASTCOPY_STREAMS="32"
export FASTCOPY_VERIFY="true"
```

## Troubleshooting

### Performance Issues

```bash
# Check system resources
fastcopy --benchmark

# Profile a transfer
fastcopy /source /dest --profile-performance --output perf.json

# Reduce streams if CPU-bound
fastcopy /source /dest --streams 8 --buffer-size 64

# Disable verification for faster transfers
fastcopy /source /dest --no-verify --mode turbo
```

### Transfer Failures

```bash
# Enable debug logging
fastcopy /source /dest --log-level debug --log-file debug.log

# Force resume from checkpoint
fastcopy /source /dest --resume --checkpoint-file .fastcopy-checkpoint

# Check integrity without retransfer
fastcopy --verify-only /destination/path

# List failed transfers
fastcopy --list-failed
```

### Network Issues

```yaml
# network-troubleshoot.yaml
transfer:
  streams: 4                    # Reduce for unstable networks
  buffer_size_mb: 64
  
network:
  max_retries: 10
  retry_delay_ms: 5000
  timeout_seconds: 600
  keep_alive: true
  tcp_nodelay: true
  
logging:
  level: "debug"
  network_trace: true
```

### Permission Errors

```bash
# Check source permissions
ls -la /source/path

# Run with elevated privileges (if necessary)
sudo fastcopy /source /dest --preserve-permissions

# Copy without preserving permissions
fastcopy /source /dest --no-preserve-permissions
```

### Disk Space Issues

```bash
# Check available space before transfer
df -h /destination

# Enable compression for space-constrained destinations
fastcopy /source /dest --compress

# Dry run to estimate space requirements
fastcopy /source /dest --dry-run --estimate-size
```

## Best Practices

1. **Always verify critical transfers**: Use `--verify` for important data
2. **Use profiles for recurring tasks**: Store configuration in YAML files
3. **Enable resume for large transfers**: Use `--resume always` for multi-GB files
4. **Monitor logs**: Check logs regularly for warnings and errors
5. **Test with dry-run**: Preview operations with `--dry-run` before execution
6. **Tune streams based on workload**: More streams ≠ always faster
7. **Use tags for tracking**: Tag transfers with `--tag` for audit trails
8. **Implement hooks for automation**: Use pre/post hooks for complex workflows

## Quick Reference

```bash
# Fast local copy
fastcopy /src /dst --streams 32 --turbo

# Verified network transfer
fastcopy /local //remote/share --verify --resume always

# Differential sync
fastcopy /src /dst --mode differential --delete-extra

# Scheduled watch
fastcopy /watched /target --watch --interval 300

# With profile
fastcopy --profile backup.yaml

# Status check
fastcopy --status

# Cancel transfer
fastcopy --cancel <transfer_id>
```
