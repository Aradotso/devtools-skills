---
name: fastcopy-file-transfer-utility
description: High-performance file duplication and transfer tool with multi-threaded I/O, intelligent caching, and verification
triggers:
  - how do I use FastCopy to copy files quickly
  - set up FastCopy for bulk file transfers
  - configure FastCopy profile for migration
  - FastCopy command line usage
  - optimize file copy performance with FastCopy
  - verify file integrity after FastCopy transfer
  - automate file synchronization with FastCopy
  - FastCopy YAML configuration examples
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility designed for enterprise-grade data migration, backup operations, and bulk file synchronization. It features multi-threaded parallel I/O, intelligent predictive caching, automatic retry mechanisms, and integrity verification.

**Key capabilities:**
- Parallel stream processing (up to 64 concurrent streams)
- Profile-based configuration via YAML
- CLI and GUI modes
- Cross-platform support (Windows, macOS, Linux)
- Integrity verification with SHA-256/CRC-32 checksums
- Auto-resume on interrupted transfers
- Watch mode for continuous synchronization

## Installation

### Windows
```bash
# Download from official site or repository
# Extract to Program Files or portable directory
# Add to PATH (optional)
set PATH=%PATH%;C:\Program Files\FastCopy
```

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or manual installation
curl -L https://example.com/fastcopy-macos.tar.gz -o fastcopy.tar.gz
tar -xzf fastcopy.tar.gz
sudo mv fastcopy /usr/local/bin/
```

### Linux
```bash
# Debian/Ubuntu
sudo apt-get install fastcopy

# Fedora/RHEL
sudo dnf install fastcopy

# Arch Linux
yay -S fastcopy

# Manual installation
wget https://example.com/fastcopy-linux.tar.gz
tar -xzf fastcopy-linux.tar.gz
sudo install -m 755 fastcopy /usr/local/bin/
```

## Command Line Interface

### Basic Usage

```bash
# Simple copy operation
fastcopy /source/path /destination/path

# Copy with verification
fastcopy /source/path /destination/path --verify

# High-priority transfer with maximum streams
fastcopy /source/path /destination/path --priority high --streams 64

# Copy with logging
fastcopy /source/path /destination/path --log-level debug --log-file transfer.log
```

### Advanced Options

```bash
# Use custom profile
fastcopy --profile my-profile.yaml

# Ad-hoc transfer with specific buffer size
fastcopy /source /dest --buffer-size 256 --streams 32

# Exclude patterns
fastcopy /source /dest --exclude "*.tmp" --exclude "*.cache"

# Resume interrupted transfer
fastcopy /source /dest --resume always

# Dry run (preview without copying)
fastcopy /source /dest --dry-run

# Tag transfer for logging
fastcopy /source /dest --tag "nightly_backup_20260215"

# Watch mode for continuous sync
fastcopy /source /dest --watch --interval 300
```

### CLI Options Reference

| Option | Description | Example |
|--------|-------------|---------|
| `--profile` | Load YAML configuration | `--profile config.yaml` |
| `--streams` | Concurrent transfer streams (1-64) | `--streams 32` |
| `--buffer-size` | Buffer size in MB | `--buffer-size 256` |
| `--verify` | Enable integrity check | `--verify` |
| `--no-verify` | Disable verification | `--no-verify` |
| `--resume` | Resume mode (always/on_failure/never) | `--resume always` |
| `--priority` | Process priority (low/normal/high) | `--priority high` |
| `--log-level` | Logging verbosity | `--log-level debug` |
| `--log-file` | Log output file | `--log-file transfer.log` |
| `--exclude` | Exclude pattern | `--exclude "*.tmp"` |
| `--dry-run` | Preview without copying | `--dry-run` |
| `--tag` | Tag for this operation | `--tag "backup_v2"` |
| `--watch` | Continuous watch mode | `--watch` |
| `--interval` | Watch interval (seconds) | `--interval 300` |

## Profile Configuration

FastCopy uses YAML profiles for repeatable, complex transfer operations.

### Basic Profile

```yaml
# basic-copy.yaml
transfer:
  mode: "turbo"                # standard, balanced, turbo
  streams: 32                  # concurrent streams (1-64)
  buffer_size_mb: 256
  verify: true
  resume: always               # always, on_failure, never

paths:
  source: "/mnt/storage/data"
  destination: "/backup/data"

logging:
  level: "info"                # debug, info, warn, error
  file: "/var/log/fastcopy/transfer.log"
```

### Advanced Profile with Scheduling

```yaml
# scheduled-backup.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: always
  priority: "high"             # low, normal, high

paths:
  source: "/home/user/projects"
  destination: "//nas/backups/projects"

filters:
  exclude:
    - "*.tmp"
    - "*.cache"
    - "node_modules/"
    - ".git/"
  include:
    - "*.js"
    - "*.py"
    - "*.md"

schedule:
  type: "cron"                 # one_time, watch, cron
  expression: "0 2 * * *"      # Daily at 2 AM
  # OR for watch mode:
  # type: "watch"
  # interval_seconds: 300

hooks:
  on_start: "echo 'Transfer starting...'"
  on_complete: "notify-send 'Backup Complete' 'All files transferred successfully'"
  on_error: "/opt/scripts/alert_admin.sh"
  
notifications:
  email: "admin@example.com"
  smtp_server: "smtp.example.com"
  smtp_port: 587
  smtp_user: "${SMTP_USER}"
  smtp_password: "${SMTP_PASSWORD}"
```

### Media Production Profile

```yaml
# media-migration.yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 1024         # Large buffer for big files
  verify: true
  resume: always
  chunk_large_files: true      # Split files > 1GB
  chunk_size_mb: 100

paths:
  source: "/mnt/raid/raw_footage"
  destination: "//storage/archive/project_alpha"

filters:
  include:
    - "*.mov"
    - "*.mxf"
    - "*.r3d"
    - "*.dpx"

metadata:
  preserve_timestamps: true
  preserve_permissions: true
  preserve_attributes: true

hooks:
  on_file_complete: "/opt/scripts/index_media.sh {filename}"
  on_complete: "/opt/scripts/generate_report.py"
```

### Database Backup Profile

```yaml
# db-backup.yaml
transfer:
  mode: "balanced"             # Avoid saturating production systems
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: on_failure

paths:
  source: "/var/lib/postgresql/backups"
  destination: "/backup/databases"

schedule:
  type: "cron"
  expression: "0 */6 * * *"    # Every 6 hours

hooks:
  pre_transfer: "/opt/scripts/pg_dump_all.sh"
  on_complete: "/opt/scripts/rotate_old_backups.sh"
  on_error: "/opt/scripts/page_dba.sh"

retention:
  keep_days: 30
  compress: true
  compression_level: 9
```

## Common Patterns

### One-Time Bulk Migration

```bash
# Create profile
cat > migration.yaml <<EOF
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: true
  resume: always

paths:
  source: "/old/storage"
  destination: "/new/storage"

logging:
  level: "info"
  file: "migration.log"

hooks:
  on_complete: "echo 'Migration complete at \$(date)' >> migration.log"
EOF

# Execute
fastcopy --profile migration.yaml --priority high
```

### Continuous Synchronization

```bash
# Watch mode with instant sync
fastcopy /source /destination --watch --interval 60 --verify
```

### Incremental Backup with Verification

```yaml
# incremental-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  verify: true
  skip_existing: true          # Only copy new/modified files
  compare_method: "timestamp"  # timestamp, size, checksum

paths:
  source: "/home/user/documents"
  destination: "/backup/documents"

schedule:
  type: "cron"
  expression: "0 0 * * *"      # Daily at midnight
```

### Network Transfer with Bandwidth Limit

```yaml
# network-transfer.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 64
  bandwidth_limit_mbps: 100    # Limit to 100 Mbps
  verify: true

paths:
  source: "/local/data"
  destination: "//remote-server/data"

network:
  retry_count: 5
  retry_delay_seconds: 10
  timeout_seconds: 300
```

## Scripting and Automation

### Bash Script Integration

```bash
#!/bin/bash
# automated-backup.sh

set -e

# Configuration
SOURCE="/critical/data"
DEST="/backup/data"
LOG_FILE="/var/log/fastcopy/backup-$(date +%Y%m%d).log"
EMAIL="admin@example.com"

# Pre-backup checks
if [ ! -d "$SOURCE" ]; then
    echo "Source directory not found: $SOURCE" | tee -a "$LOG_FILE"
    exit 1
fi

# Execute FastCopy
echo "Starting backup at $(date)" | tee -a "$LOG_FILE"

fastcopy "$SOURCE" "$DEST" \
    --streams 48 \
    --buffer-size 256 \
    --verify \
    --resume always \
    --priority high \
    --log-level info \
    --log-file "$LOG_FILE" \
    --tag "automated_backup_$(date +%Y%m%d)"

EXIT_CODE=$?

# Post-backup actions
if [ $EXIT_CODE -eq 0 ]; then
    echo "Backup completed successfully at $(date)" | tee -a "$LOG_FILE"
    # Optional: Send success notification
    echo "Backup successful" | mail -s "Backup Success" "$EMAIL"
else
    echo "Backup failed with exit code $EXIT_CODE at $(date)" | tee -a "$LOG_FILE"
    # Send failure alert
    echo "Check log: $LOG_FILE" | mail -s "BACKUP FAILED" "$EMAIL"
    exit $EXIT_CODE
fi
```

### Python Integration

```python
#!/usr/bin/env python3
# fastcopy_manager.py

import subprocess
import json
import sys
from datetime import datetime

class FastCopyManager:
    def __init__(self, profile_path=None):
        self.profile_path = profile_path
        
    def execute_transfer(self, source, destination, **kwargs):
        """Execute FastCopy transfer with options"""
        cmd = ['fastcopy', source, destination]
        
        if self.profile_path:
            cmd.extend(['--profile', self.profile_path])
        
        # Add optional arguments
        if kwargs.get('streams'):
            cmd.extend(['--streams', str(kwargs['streams'])])
        if kwargs.get('verify'):
            cmd.append('--verify')
        if kwargs.get('priority'):
            cmd.extend(['--priority', kwargs['priority']])
        if kwargs.get('tag'):
            cmd.extend(['--tag', kwargs['tag']])
            
        # Execute
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                check=True
            )
            return {
                'status': 'success',
                'stdout': result.stdout,
                'stderr': result.stderr
            }
        except subprocess.CalledProcessError as e:
            return {
                'status': 'error',
                'returncode': e.returncode,
                'stdout': e.stdout,
                'stderr': e.stderr
            }
    
    def batch_transfer(self, transfers):
        """Execute multiple transfers in sequence"""
        results = []
        for transfer in transfers:
            print(f"Processing: {transfer['source']} -> {transfer['destination']}")
            result = self.execute_transfer(
                transfer['source'],
                transfer['destination'],
                **transfer.get('options', {})
            )
            results.append({
                'transfer': transfer,
                'result': result,
                'timestamp': datetime.now().isoformat()
            })
        return results

# Usage example
if __name__ == '__main__':
    manager = FastCopyManager()
    
    # Single transfer
    result = manager.execute_transfer(
        '/data/source',
        '/data/destination',
        streams=32,
        verify=True,
        priority='high',
        tag='python_automated'
    )
    
    print(json.dumps(result, indent=2))
    
    # Batch transfers
    transfers = [
        {
            'source': '/data/set1',
            'destination': '/backup/set1',
            'options': {'streams': 24, 'verify': True}
        },
        {
            'source': '/data/set2',
            'destination': '/backup/set2',
            'options': {'streams': 32, 'verify': True}
        }
    ]
    
    batch_results = manager.batch_transfer(transfers)
    for r in batch_results:
        print(f"{r['timestamp']}: {r['result']['status']}")
```

## Integrity Verification

FastCopy provides multiple verification methods:

```yaml
# verification-profile.yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true
  verification_method: "checksum"  # checksum, size, timestamp

checksum:
  algorithm: "sha256"              # sha256, md5, crc32
  parallel_verify: true            # Verify while copying
  save_checksums: true             # Save checksum file
  checksum_file: "checksums.sha256"
```

### Post-Transfer Verification

```bash
# Verify after transfer
fastcopy --verify-only /source /destination --checksum sha256

# Generate checksum manifest
fastcopy /source /destination --generate-checksums checksums.txt

# Verify against manifest
fastcopy --verify-manifest checksums.txt /destination
```

## Performance Tuning

### Optimal Settings by Use Case

**SSD to SSD (Local)**
```yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: true
```

**HDD to HDD (Local)**
```yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
```

**Network Transfer (1 Gbps)**
```yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 256
  bandwidth_limit_mbps: 100
  verify: true
```

**Network Transfer (10 Gbps)**
```yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  bandwidth_limit_mbps: 1000
  verify: true
```

## Troubleshooting

### Common Issues

**Transfer Slow**
```bash
# Check system resources
fastcopy /source /dest --log-level debug

# Reduce streams if CPU/memory limited
fastcopy /source /dest --streams 8 --buffer-size 64

# Check disk I/O
iostat -x 1
```

**Verification Failures**
```bash
# Re-verify with detailed logging
fastcopy --verify-only /source /dest --log-level debug --log-file verify.log

# Use more conservative checksum
fastcopy /source /dest --verify --checksum sha256
```

**Network Timeouts**
```yaml
# Increase timeouts in profile
network:
  timeout_seconds: 600
  retry_count: 10
  retry_delay_seconds: 30
  keep_alive: true
```

**Interrupted Transfers**
```bash
# Resume with same parameters
fastcopy /source /dest --resume always --streams 32

# Check resume state
fastcopy --show-resume-state /dest
```

**Permission Errors**
```bash
# Run with elevated privileges (use carefully)
sudo fastcopy /source /dest --preserve-permissions

# Or adjust ownership after
fastcopy /source /dest
sudo chown -R user:group /dest
```

### Debug Mode

```bash
# Full debug output
fastcopy /source /dest --log-level debug --log-file debug.log

# Monitor in real-time
tail -f debug.log

# Check transfer statistics
fastcopy --stats /dest
```

## Environment Variables

FastCopy respects the following environment variables:

```bash
# Default profile location
export FASTCOPY_PROFILE="$HOME/.config/fastcopy/default.yaml"

# Default log directory
export FASTCOPY_LOG_DIR="/var/log/fastcopy"

# Default buffer size (MB)
export FASTCOPY_BUFFER_SIZE=256

# Default stream count
export FASTCOPY_STREAMS=32

# SMTP credentials for notifications
export SMTP_USER="notifications@example.com"
export SMTP_PASSWORD="your-smtp-password"

# API integrations (if using)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

## Integration Examples

### Cron Job

```bash
# Add to crontab: crontab -e
# Daily backup at 2 AM
0 2 * * * /usr/local/bin/fastcopy --profile /etc/fastcopy/daily-backup.yaml >> /var/log/fastcopy/cron.log 2>&1

# Hourly sync
0 * * * * /usr/local/bin/fastcopy /data/live /data/mirror --watch --interval 3600
```

### Systemd Service

```ini
# /etc/systemd/system/fastcopy-sync.service
[Unit]
Description=FastCopy Continuous Sync Service
After=network.target

[Service]
Type=simple
User=fastcopy
Group=fastcopy
ExecStart=/usr/local/bin/fastcopy --profile /etc/fastcopy/sync.yaml
Restart=always
RestartSec=10
Environment="FASTCOPY_LOG_DIR=/var/log/fastcopy"

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl enable fastcopy-sync
sudo systemctl start fastcopy-sync
sudo systemctl status fastcopy-sync
```

This skill provides comprehensive coverage of FastCopy's functionality for AI coding agents to assist developers with file transfer operations, automation, and optimization.
