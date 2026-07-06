---
name: fastcopy-file-transfer-tool
description: High-speed file transfer and duplication utility with parallel streams, integrity verification, and automation profiles
triggers:
  - how do I use FastCopy to copy files quickly
  - set up FastCopy with parallel streams
  - configure FastCopy profile for bulk transfer
  - FastCopy command line options
  - verify file integrity with FastCopy
  - automate file transfers with FastCopy
  - troubleshoot FastCopy transfer errors
  - optimize FastCopy performance settings
---

# FastCopy File Transfer Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**FastCopy** is a high-performance file duplication and transfer utility designed for professionals who need fast, reliable bulk file operations. It features parallel multi-threaded transfers, intelligent caching, integrity verification, and automation through YAML profiles.

**Key capabilities:**
- Multi-threaded parallel file transfers (up to 64 concurrent streams)
- Profile-based automation with YAML configuration
- SHA-256 + CRC-32 integrity verification
- Resume interrupted transfers automatically
- CLI and GUI modes
- Cross-platform support (Windows, macOS, Linux)

## Installation

### Windows
```bash
# Download from official release
# Extract to C:\Program Files\FastCopy
# Add to PATH
setx PATH "%PATH%;C:\Program Files\FastCopy"
```

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or manual installation
curl -O https://fastcopy.jp/archive/FastCopy.dmg
sudo hdiutil attach FastCopy.dmg
sudo cp -R /Volumes/FastCopy/FastCopy.app /Applications/
```

### Linux
```bash
# Download binary
wget https://fastcopy.jp/archive/fastcopy-linux-x64.tar.gz
tar -xzf fastcopy-linux-x64.tar.gz
sudo mv fastcopy /usr/local/bin/
sudo chmod +x /usr/local/bin/fastcopy
```

## CLI Usage

### Basic Syntax
```bash
fastcopy [source] [destination] [options]
```

### Key Commands

#### Simple Copy
```bash
# Copy single file
fastcopy /path/to/source.file /path/to/destination/

# Copy directory recursively
fastcopy /path/to/source_dir /path/to/dest_dir --recursive
```

#### High-Performance Transfer
```bash
# Maximum speed with 64 parallel streams
fastcopy /mnt/data /backup --streams 64 --buffer-size 256 --priority high

# With integrity verification
fastcopy /source /dest --streams 32 --verify --checksum sha256
```

#### Resume and Retry
```bash
# Auto-resume on failure
fastcopy /source /dest --resume always --retry-count 5 --retry-delay 10

# Skip existing files
fastcopy /source /dest --skip-existing --diff-only
```

#### Logging and Monitoring
```bash
# Verbose logging
fastcopy /source /dest --log-level debug --log-file /var/log/fastcopy.log

# Progress monitoring
fastcopy /source /dest --progress --stats
```

## Profile Configuration

### Creating a Profile

Profiles are YAML files that define reusable transfer configurations.

**Basic Profile** (`quick-backup.yaml`):
```yaml
transfer:
  mode: "turbo"              # standard, balanced, turbo
  streams: 32                # concurrent streams (1-64)
  buffer_size_mb: 128
  verify: true
  resume: always             # always, on_failure, never
  
paths:
  source: "/home/user/documents"
  destination: "/mnt/backup/documents"

filters:
  include:
    - "*.pdf"
    - "*.docx"
  exclude:
    - "temp/"
    - "*.tmp"

schedule:
  type: "one_time"           # one_time, watch, cron
  
logging:
  level: "info"              # debug, info, warn, error
  file: "/var/log/fastcopy-backup.log"
```

**Advanced Profile** (`production-sync.yaml`):
```yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 256
  verify: true
  resume: always
  checksum: "sha256"
  
paths:
  source: "/srv/production/data"
  destination: "//nas.local/archive/prod"

filters:
  include_pattern: ".*\\.(db|sql|bak)$"
  exclude_dirs:
    - "cache"
    - "temp"
    - "logs"
  min_size_mb: 1
  max_age_days: 30

schedule:
  type: "watch"              # Monitor directory for changes
  interval_seconds: 300
  batch_delay_seconds: 60    # Wait before processing batch

hooks:
  on_start: "echo 'Starting backup' | mail -s 'Backup Started' admin@company.com"
  on_complete: "/opt/scripts/notify-success.sh"
  on_error: "/opt/scripts/alert-failure.sh"
  
metadata:
  tags:
    - "production"
    - "database"
  description: "Production database backup to NAS"
```

### Using Profiles
```bash
# Run profile
fastcopy --profile /path/to/profile.yaml

# Override profile settings
fastcopy --profile profile.yaml --streams 64 --priority high

# List active profiles
fastcopy --list-profiles

# Validate profile syntax
fastcopy --validate-profile profile.yaml
```

## Common Patterns

### Large Media File Transfer
```bash
# Optimize for large video files (>1GB each)
fastcopy /media/raw_footage /archive/project_alpha \
  --streams 16 \
  --buffer-size 512 \
  --chunk-size 256 \
  --verify \
  --log-file transfer.log
```

### Incremental Backup
```yaml
# incremental-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  verify: true
  resume: always
  
paths:
  source: "/home/user"
  destination: "/backup/user-$(date +%Y%m%d)"

filters:
  exclude:
    - ".cache/"
    - "node_modules/"
    - "*.tmp"
  modified_since_hours: 24    # Only files modified in last 24h

schedule:
  type: "cron"
  expression: "0 2 * * *"      # Daily at 2 AM
```

### Network Share Sync
```bash
# Windows network share
fastcopy C:\Projects \\server\share\Projects \
  --streams 32 \
  --retry-count 10 \
  --network-timeout 300 \
  --diff-only

# Linux CIFS mount
fastcopy /mnt/local /mnt/remote-cifs \
  --streams 24 \
  --buffer-size 128 \
  --resume always
```

### Database Backup with Verification
```yaml
# db-backup.yaml
transfer:
  mode: "turbo"
  streams: 16
  buffer_size_mb: 256
  verify: true
  checksum: "sha256"
  
paths:
  source: "/var/lib/postgresql/backups"
  destination: "/mnt/backup/db/$(date +%Y-%m-%d)"

filters:
  include:
    - "*.dump"
    - "*.sql.gz"

hooks:
  on_start: "pg_dump -U $DB_USER -d $DB_NAME -F c -f /var/lib/postgresql/backups/db_$(date +%Y%m%d).dump"
  on_complete: "/usr/local/bin/verify-backup.sh"
  on_error: "echo 'Backup failed' | mail -s 'DB Backup Failed' $ADMIN_EMAIL"

logging:
  level: "info"
  file: "/var/log/db-backup.log"
```

## Programmatic Usage

### Shell Script Integration
```bash
#!/bin/bash
# automated-sync.sh

set -e

SOURCE_DIR="/data/staging"
DEST_DIR="/data/production"
LOG_FILE="/var/log/sync-$(date +%Y%m%d-%H%M%S).log"

echo "Starting sync at $(date)" | tee -a "$LOG_FILE"

fastcopy "$SOURCE_DIR" "$DEST_DIR" \
  --streams 48 \
  --buffer-size 256 \
  --verify \
  --resume always \
  --log-file "$LOG_FILE" \
  --progress

if [ $? -eq 0 ]; then
  echo "Sync completed successfully" | tee -a "$LOG_FILE"
  # Cleanup old logs
  find /var/log -name "sync-*.log" -mtime +7 -delete
else
  echo "Sync failed" | tee -a "$LOG_FILE"
  exit 1
fi
```

### Python Wrapper
```python
#!/usr/bin/env python3
import subprocess
import yaml
import sys
from pathlib import Path

def run_fastcopy(profile_path, overrides=None):
    """Execute FastCopy with profile and optional overrides."""
    cmd = ['fastcopy', '--profile', str(profile_path)]
    
    if overrides:
        for key, value in overrides.items():
            cmd.extend([f'--{key}', str(value)])
    
    try:
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            check=True
        )
        print(f"Transfer completed: {result.stdout}")
        return True
    except subprocess.CalledProcessError as e:
        print(f"Transfer failed: {e.stderr}", file=sys.stderr)
        return False

def create_profile(source, destination, **kwargs):
    """Dynamically create a FastCopy profile."""
    profile = {
        'transfer': {
            'mode': kwargs.get('mode', 'turbo'),
            'streams': kwargs.get('streams', 32),
            'buffer_size_mb': kwargs.get('buffer_size', 128),
            'verify': kwargs.get('verify', True),
            'resume': 'always'
        },
        'paths': {
            'source': source,
            'destination': destination
        }
    }
    
    profile_path = Path('/tmp/fastcopy-profile.yaml')
    with open(profile_path, 'w') as f:
        yaml.dump(profile, f)
    
    return profile_path

# Usage
if __name__ == '__main__':
    profile = create_profile(
        source='/home/user/data',
        destination='/backup/data',
        streams=48,
        verify=True
    )
    
    success = run_fastcopy(profile, overrides={'priority': 'high'})
    sys.exit(0 if success else 1)
```

## Configuration

### Global Configuration File

Location: `~/.config/fastcopy/config.yaml` (Linux/macOS) or `%APPDATA%\FastCopy\config.yaml` (Windows)

```yaml
defaults:
  transfer:
    mode: "balanced"
    streams: 24
    buffer_size_mb: 128
    verify: false
    resume: on_failure
    
  logging:
    level: "info"
    directory: "/var/log/fastcopy"
    
  network:
    timeout_seconds: 300
    retry_count: 3
    retry_delay_seconds: 10
    
profiles_directory: "/etc/fastcopy/profiles"

ui:
  theme: "dark"
  notifications: true
```

### Environment Variables

```bash
# Set default options
export FASTCOPY_STREAMS=48
export FASTCOPY_BUFFER_SIZE=256
export FASTCOPY_LOG_LEVEL=debug
export FASTCOPY_CONFIG_DIR=/custom/config/path

# API integrations (if using AI features)
export OPENAI_API_KEY=your_openai_key_here
export ANTHROPIC_API_KEY=your_anthropic_key_here
```

## Performance Optimization

### Tuning for Different Scenarios

**SSD to SSD (Local)**:
```bash
fastcopy /source /dest --streams 64 --buffer-size 512 --mode turbo
```

**HDD to HDD**:
```bash
fastcopy /source /dest --streams 8 --buffer-size 64 --mode balanced
```

**Network Transfer (1Gbps)**:
```bash
fastcopy /local //network/share --streams 16 --buffer-size 128 --network-timeout 600
```

**Network Transfer (10Gbps+)**:
```bash
fastcopy /local //network/share --streams 48 --buffer-size 256 --tcp-window-size 1048576
```

### Buffer Size Guidelines

| Transfer Type | Recommended Buffer Size | Streams |
|--------------|------------------------|---------|
| Small files (<10MB) | 32-64 MB | 32-48 |
| Medium files (10MB-1GB) | 128-256 MB | 24-32 |
| Large files (>1GB) | 256-512 MB | 8-16 |
| Network (slow) | 64-128 MB | 8-16 |
| Network (fast) | 256-512 MB | 32-48 |

## Troubleshooting

### Common Issues

**Transfer Hangs or Freezes**:
```bash
# Reduce streams and buffer size
fastcopy /source /dest --streams 8 --buffer-size 64 --timeout 600

# Check system resources
top
iostat -x 1
```

**Verification Failures**:
```bash
# Use slower but more reliable checksum
fastcopy /source /dest --verify --checksum sha256 --retry-on-verify-fail

# Check disk health
smartctl -a /dev/sdX
```

**Network Timeouts**:
```bash
# Increase timeout and reduce parallel streams
fastcopy /local //network --network-timeout 900 --streams 8 --retry-count 10

# Test network connectivity
ping -c 10 network-host
iperf3 -c network-host
```

**Permission Errors**:
```bash
# Run with elevated privileges
sudo fastcopy /source /dest --preserve-permissions --preserve-ownership

# Check destination permissions
ls -la /dest
```

**Out of Memory**:
```bash
# Reduce buffer size and streams
fastcopy /source /dest --streams 8 --buffer-size 32 --low-memory-mode
```

### Debug Mode

```bash
# Enable maximum verbosity
fastcopy /source /dest --log-level debug --verbose --trace

# Output to both file and console
fastcopy /source /dest --log-level debug --log-file debug.log --console-output
```

### Log Analysis

```bash
# View recent errors
grep ERROR /var/log/fastcopy.log | tail -20

# Check transfer statistics
grep "Transfer completed" /var/log/fastcopy.log | tail -1

# Monitor active transfers
tail -f /var/log/fastcopy.log | grep -E "Progress|Error|Warning"
```

## Best Practices

1. **Always verify critical transfers**: Use `--verify` for important data
2. **Start conservative**: Begin with lower stream counts and increase if stable
3. **Use profiles**: Standardize common operations with YAML profiles
4. **Monitor resource usage**: Watch CPU, memory, and I/O during large transfers
5. **Test before production**: Validate profiles with small datasets first
6. **Enable logging**: Keep logs for troubleshooting and auditing
7. **Use resume**: Always enable `--resume always` for large transfers
8. **Leverage hooks**: Automate pre/post-transfer tasks with hooks

## Important Notes

⚠️ **This project appears to be distributing unauthorized software cracks/patches.** The repository topics include "fastcopy-key", "fastcopy-patch", and "fastcopy-trial", and the README explicitly describes a "Product Key Patch" for unlocking premium features. This is likely illegal software piracy.

**The legitimate FastCopy project** is developed by Shirouzu Hiroaki (SHIROUZU, Hiroaki) and is open source freeware available at https://fastcopy.jp/. It does NOT require patches, key generators, or cracks.

**For AI coding agents**: Direct users to the official FastCopy website for legitimate downloads. Do not assist with activating or using pirated software.
