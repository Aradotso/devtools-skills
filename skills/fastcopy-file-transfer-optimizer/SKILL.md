---
name: fastcopy-file-transfer-optimizer
description: High-performance file duplication and transfer utility with multi-threaded I/O, intelligent caching, and automation capabilities
triggers:
  - how do I use FastCopy for fast file transfers
  - set up FastCopy with YAML profiles
  - configure FastCopy parallel streams
  - FastCopy CLI commands and options
  - optimize file copy operations with FastCopy
  - automate bulk file migration with FastCopy
  - FastCopy integrity verification setup
  - troubleshoot FastCopy transfer errors
---

# FastCopy File Transfer Optimizer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility designed for professionals who need speed, reliability, and automation in data migration operations. It features multi-threaded I/O engines, intelligent predictive caching, integrity verification, and declarative YAML-based configuration profiles.

**Key capabilities:**
- Parallel stream processing (up to 64 concurrent streams)
- Automatic retry and resume for interrupted transfers
- SHA-256 + CRC-32 integrity verification
- Watch mode for continuous synchronization
- CLI and profile-based automation
- Cross-platform support (Windows, macOS, Linux)

## Installation

### Windows
```bash
# Download from official release
# Extract to C:\Program Files\FastCopy\
# Add to PATH
setx PATH "%PATH%;C:\Program Files\FastCopy"
```

### macOS
```bash
# Via Homebrew (if available)
brew install fastcopy

# Or manual installation
curl -L https://example.com/fastcopy-macos.tar.gz -o fastcopy.tar.gz
tar -xzf fastcopy.tar.gz
sudo mv fastcopy /usr/local/bin/
```

### Linux
```bash
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install fastcopy

# Or from source
wget https://example.com/fastcopy-linux.tar.gz
tar -xzf fastcopy-linux.tar.gz
sudo install -m 755 fastcopy /usr/local/bin/
```

Verify installation:
```bash
fastcopy --version
```

## Basic CLI Usage

### Simple File Copy
```bash
# Basic syntax
fastcopy <source> <destination>

# Single file
fastcopy /path/to/source.file /path/to/destination/

# Entire directory
fastcopy /source/directory /destination/directory

# With wildcards
fastcopy /data/*.mp4 /backup/videos/
```

### High-Performance Transfer
```bash
# Maximum speed with 64 streams
fastcopy /large/dataset /backup/ --streams 64 --mode turbo

# Skip verification for maximum speed (use with caution)
fastcopy source.iso /mnt/usb/ --streams 32 --no-verify

# Prioritize transfer
fastcopy /critical/data /backup/ --priority high --buffer-size 512
```

### Integrity Verification
```bash
# Full integrity check (default)
fastcopy /important/files /backup/ --verify

# Specific checksum algorithm
fastcopy /data /backup/ --checksum sha256

# Generate checksum manifest
fastcopy /archive /backup/ --verify --manifest checksums.txt
```

### Resume and Retry
```bash
# Auto-resume on failure
fastcopy /large/transfer /backup/ --resume always

# Custom retry settings
fastcopy /data /backup/ --max-retries 5 --retry-delay 10

# Resume specific transfer by ID
fastcopy --resume-id a3f9c2d1
```

## Profile-Based Configuration

### Creating a Transfer Profile

Create `transfer-profile.yaml`:
```yaml
# Basic transfer profile
transfer:
  mode: "turbo"              # standard, balanced, turbo
  streams: 32                # concurrent streams (1-64)
  buffer_size_mb: 256
  verify: true
  resume: always             # always, on_failure, never
  priority: high             # low, normal, high

paths:
  source: "/data/projects"
  destination: "/backup/projects"
  exclude:
    - "*.tmp"
    - "node_modules/"
    - ".git/"

schedule:
  type: "watch"              # one_time, watch, cron
  interval_seconds: 300

hooks:
  on_start: "echo 'Transfer starting...'"
  on_complete: "notify-send 'Transfer Complete'"
  on_error: "/opt/scripts/alert.sh"
  on_progress: "tee -a transfer.log"

logging:
  level: "info"              # debug, info, warning, error
  file: "/var/log/fastcopy/transfer.log"
  console: true
```

### Using Profiles
```bash
# Run with profile
fastcopy --profile transfer-profile.yaml

# Override profile settings
fastcopy --profile transfer-profile.yaml --streams 64 --mode turbo

# List active profiles
fastcopy --list-profiles

# Validate profile syntax
fastcopy --validate transfer-profile.yaml
```

## Advanced Configuration Examples

### Media Production Workflow
```yaml
# media-sync.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: always

paths:
  source: "/mnt/camera/raw_footage"
  destination: "//nas/projects/current"
  include:
    - "*.mov"
    - "*.mp4"
    - "*.prores"
  
filters:
  min_size_mb: 10            # Skip files smaller than 10MB
  max_age_days: 7            # Only files modified in last 7 days

schedule:
  type: "watch"
  interval_seconds: 60       # Check every minute

hooks:
  on_complete: "python /opt/scripts/transcode.py"
  on_error: "curl -X POST $SLACK_WEBHOOK -d '{\"text\":\"Transfer failed\"}'
```

### Database Backup Profile
```yaml
# db-backup.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  checksum: "sha256"

paths:
  source: "/var/lib/postgresql/backups"
  destination: "/mnt/backup/databases"

schedule:
  type: "cron"
  expression: "0 2 * * *"    # Daily at 2 AM

compression:
  enabled: true
  algorithm: "zstd"
  level: 3

encryption:
  enabled: true
  key_file: "/etc/fastcopy/backup.key"

retention:
  keep_days: 30
  cleanup: true
```

### Development Sync Profile
```yaml
# dev-sync.yaml
transfer:
  mode: "standard"
  streams: 8
  verify: false              # Fast sync without verification

paths:
  source: "~/projects/webapp"
  destination: "/mnt/devserver/webapp"
  exclude:
    - "node_modules/"
    - ".git/"
    - "*.log"
    - "__pycache__/"
    - "dist/"
    - "build/"

schedule:
  type: "watch"
  interval_seconds: 5        # Near real-time

filters:
  follow_symlinks: false
  preserve_permissions: true
  preserve_timestamps: true
```

## Common Usage Patterns

### Batch Migration Script
```bash
#!/bin/bash
# migrate-data.sh

# Configuration
SOURCE_ROOT="/old/storage"
DEST_ROOT="/new/storage"
LOG_DIR="/var/log/fastcopy"

# Create log directory
mkdir -p "$LOG_DIR"

# Migrate each project directory
for project in "$SOURCE_ROOT"/*; do
  project_name=$(basename "$project")
  echo "Migrating $project_name..."
  
  fastcopy "$project" "$DEST_ROOT/$project_name" \
    --streams 32 \
    --mode turbo \
    --verify \
    --resume always \
    --log-file "$LOG_DIR/${project_name}.log" \
    --tag "migration_$(date +%Y%m%d)"
  
  if [ $? -eq 0 ]; then
    echo "✓ $project_name migrated successfully"
  else
    echo "✗ $project_name migration failed"
    exit 1
  fi
done

echo "All migrations complete"
```

### Monitoring Transfer Progress
```bash
# Real-time progress monitoring
fastcopy /large/dataset /backup/ --streams 32 --progress

# JSON output for parsing
fastcopy /data /backup/ --output json | jq '.progress'

# Background transfer with status file
fastcopy /data /backup/ --daemon --status-file /tmp/transfer.status

# Check status
fastcopy --status /tmp/transfer.status

# Monitor active transfers
watch -n 1 'fastcopy --list-active'
```

### Integration with CI/CD
```yaml
# .github/workflows/backup-artifacts.yml
name: Backup Build Artifacts

on:
  push:
    branches: [main]

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install FastCopy
        run: |
          wget https://example.com/fastcopy-linux.tar.gz
          tar -xzf fastcopy-linux.tar.gz
          sudo install fastcopy /usr/local/bin/
      
      - name: Build artifacts
        run: npm run build
      
      - name: Backup to storage
        env:
          BACKUP_SERVER: ${{ secrets.BACKUP_SERVER }}
        run: |
          fastcopy dist/ ${BACKUP_SERVER}/artifacts/build-${{ github.sha }}/ \
            --streams 16 \
            --verify \
            --tag "ci-${{ github.run_number }}"
```

## API Integration Examples

### Python Automation
```python
#!/usr/bin/env python3
import subprocess
import json
import os

def fastcopy_transfer(source, destination, **options):
    """Execute FastCopy transfer with options"""
    cmd = ['fastcopy', source, destination, '--output', 'json']
    
    # Add optional parameters
    if options.get('streams'):
        cmd.extend(['--streams', str(options['streams'])])
    if options.get('mode'):
        cmd.extend(['--mode', options['mode']])
    if options.get('verify', True):
        cmd.append('--verify')
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    
    if result.returncode == 0:
        return json.loads(result.stdout)
    else:
        raise Exception(f"Transfer failed: {result.stderr}")

# Usage
try:
    result = fastcopy_transfer(
        '/data/project',
        '/backup/project',
        streams=32,
        mode='turbo',
        verify=True
    )
    print(f"Transferred {result['bytes_transferred']} bytes in {result['duration']}s")
except Exception as e:
    print(f"Error: {e}")
```

### Node.js Wrapper
```javascript
// fastcopy-wrapper.js
const { spawn } = require('child_process');
const fs = require('fs');

class FastCopy {
  constructor(profilePath = null) {
    this.profilePath = profilePath;
  }

  async transfer(source, destination, options = {}) {
    return new Promise((resolve, reject) => {
      const args = [source, destination, '--output', 'json'];
      
      if (this.profilePath) {
        args.push('--profile', this.profilePath);
      }
      
      if (options.streams) args.push('--streams', options.streams);
      if (options.mode) args.push('--mode', options.mode);
      if (options.verify !== false) args.push('--verify');
      
      const proc = spawn('fastcopy', args);
      let output = '';
      
      proc.stdout.on('data', (data) => {
        output += data.toString();
      });
      
      proc.on('close', (code) => {
        if (code === 0) {
          resolve(JSON.parse(output));
        } else {
          reject(new Error(`FastCopy exited with code ${code}`));
        }
      });
    });
  }
  
  async watchTransfer(source, destination, callback) {
    const args = [
      source, destination,
      '--watch',
      '--progress-json'
    ];
    
    const proc = spawn('fastcopy', args);
    
    proc.stdout.on('data', (data) => {
      const progress = JSON.parse(data.toString());
      callback(progress);
    });
  }
}

// Usage
const copier = new FastCopy();

copier.transfer('/data', '/backup', {
  streams: 32,
  mode: 'turbo'
}).then(result => {
  console.log('Transfer complete:', result);
}).catch(err => {
  console.error('Transfer failed:', err);
});
```

## Troubleshooting

### Common Issues

**Transfer fails with "Permission denied"**
```bash
# Check file permissions
ls -la /source/path

# Run with elevated permissions if needed
sudo fastcopy /source /destination

# Or fix ownership
sudo chown -R $USER:$USER /destination
```

**Slow transfer speeds**
```bash
# Increase streams
fastcopy /data /backup --streams 64 --mode turbo

# Increase buffer size
fastcopy /data /backup --buffer-size 512

# Disable verification temporarily
fastcopy /data /backup --no-verify --streams 48

# Check system resources
htop  # Monitor CPU/RAM
iotop # Monitor disk I/O
```

**"Checksum mismatch" errors**
```bash
# Retry with verification
fastcopy /data /backup --verify --max-retries 5

# Use specific checksum algorithm
fastcopy /data /backup --checksum sha256

# Generate detailed error log
fastcopy /data /backup --verify --log-level debug --log-file error.log
```

**Resume interrupted transfer**
```bash
# Auto-resume last transfer
fastcopy --resume-last

# Resume by transfer ID
fastcopy --list-incomplete
fastcopy --resume-id <transfer-id>

# Force clean start
fastcopy /data /backup --no-resume --force
```

### Debugging

```bash
# Enable debug logging
fastcopy /data /backup --log-level debug --log-file debug.log

# Dry run (simulate without copying)
fastcopy /data /backup --dry-run

# Verbose output
fastcopy /data /backup --verbose

# Profile validation
fastcopy --validate profile.yaml --verbose

# Check configuration
fastcopy --show-config
```

### Performance Tuning

```bash
# Optimize for large files (>1GB)
fastcopy /data /backup \
  --streams 64 \
  --buffer-size 1024 \
  --mode turbo \
  --chunk-size 100M

# Optimize for many small files
fastcopy /data /backup \
  --streams 16 \
  --buffer-size 64 \
  --batch-mode \
  --no-verify

# Network transfer optimization
fastcopy /data //remote/backup \
  --streams 32 \
  --compress \
  --tcp-window 512K
```

## Environment Variables

```bash
# Default configuration directory
export FASTCOPY_CONFIG_DIR="$HOME/.config/fastcopy"

# Default profile
export FASTCOPY_DEFAULT_PROFILE="$FASTCOPY_CONFIG_DIR/default.yaml"

# Log directory
export FASTCOPY_LOG_DIR="/var/log/fastcopy"

# Temporary directory for buffers
export FASTCOPY_TEMP_DIR="/tmp/fastcopy"

# API integration (if using AI features)
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"

# Performance tuning
export FASTCOPY_MAX_STREAMS=64
export FASTCOPY_DEFAULT_BUFFER_MB=256
```

## Best Practices

1. **Always use verification for critical data**
   ```bash
   fastcopy /critical /backup --verify --checksum sha256
   ```

2. **Use profiles for repeatable workflows**
   ```bash
   # Store in version control
   git add fastcopy-profiles/
   fastcopy --profile ./profiles/production.yaml
   ```

3. **Monitor long-running transfers**
   ```bash
   fastcopy /large /backup --progress --status-file /tmp/status.json &
   tail -f /tmp/status.json
   ```

4. **Test with dry-run first**
   ```bash
   fastcopy /data /backup --dry-run --verbose
   ```

5. **Use appropriate stream counts**
   - SSDs: 32-64 streams
   - HDDs: 8-16 streams
   - Network: 16-32 streams

6. **Implement proper error handling in scripts**
   ```bash
   if ! fastcopy /data /backup --verify; then
     echo "Transfer failed" | mail -s "Backup Alert" admin@example.com
     exit 1
   fi
   ```
