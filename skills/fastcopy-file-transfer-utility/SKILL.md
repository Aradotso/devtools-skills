---
name: fastcopy-file-transfer-utility
description: High-performance file duplication and transfer tool with multi-threaded I/O, integrity verification, and automation capabilities
triggers:
  - how do I use FastCopy to copy files quickly
  - configure FastCopy for bulk file transfers
  - set up FastCopy with profile automation
  - FastCopy parallel streaming and verification
  - integrate FastCopy into my workflow
  - troubleshoot FastCopy transfer errors
  - optimize FastCopy performance settings
  - schedule automated file copies with FastCopy
---

# FastCopy File Transfer Utility Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file transfer and duplication tool designed for scenarios requiring rapid data movement: backups, migrations, media workflows, and development environment synchronization. It uses multi-threaded I/O, intelligent buffering, and integrity verification to maximize transfer speeds while ensuring data accuracy.

**Key Capabilities:**
- Parallel stream processing (up to 64 concurrent streams)
- Automatic resume on interrupted transfers
- SHA-256 + CRC-32 integrity verification
- YAML-based profile configuration
- CLI and GUI modes
- Cross-platform support (Windows, macOS, Linux)

## Installation

### Windows
```powershell
# Download from official source or repository release
# Extract and run installer
fastcopy-setup.exe

# Add to PATH (if portable)
$env:PATH += ";C:\Tools\FastCopy"
```

### macOS
```bash
# Using Homebrew (if available)
brew install fastcopy

# Or manual installation
curl -L https://fastcopy-releases/fastcopy-macos.tar.gz -o fastcopy.tar.gz
tar -xzf fastcopy.tar.gz
sudo mv fastcopy /usr/local/bin/
```

### Linux
```bash
# Debian/Ubuntu
sudo apt-get install fastcopy

# Fedora
sudo dnf install fastcopy

# Manual installation
wget https://fastcopy-releases/fastcopy-linux.tar.gz
tar -xzf fastcopy-linux.tar.gz
sudo install -m 755 fastcopy /usr/local/bin/
```

## CLI Commands

### Basic Usage

```bash
# Simple copy with default settings
fastcopy /source/path /destination/path

# Copy with increased parallelism
fastcopy /source/path /destination/path --streams 32

# Copy with verification
fastcopy /source/path /destination/path --verify --checksum sha256

# Copy with custom buffer size
fastcopy /source/path /destination/path --buffer-size 512m

# High-priority transfer
fastcopy /source/path /destination/path --priority high --streams 64
```

### Profile-Based Operations

```bash
# Use a profile configuration
fastcopy --profile /path/to/profile.yaml

# Override profile settings
fastcopy --profile backup.yaml --streams 48 --priority high

# List available profiles
fastcopy --list-profiles

# Validate profile syntax
fastcopy --validate-profile /path/to/profile.yaml
```

### Advanced Options

```bash
# Dry run (preview without copying)
fastcopy /source /dest --dry-run

# Resume interrupted transfer
fastcopy /source /dest --resume

# Copy with file filtering
fastcopy /source /dest --include "*.mp4" --exclude "*.tmp"

# Logging and monitoring
fastcopy /source /dest --log-level debug --log-file transfer.log

# Tag transfers for organization
fastcopy /source /dest --tag "project_backup_20260215"
```

## Profile Configuration

### Basic Profile Structure

Create a YAML profile for reusable transfer configurations:

```yaml
# transfer-profile.yaml
transfer:
  mode: "turbo"                    # standard, balanced, turbo
  streams: 32                      # concurrent streams (1-64)
  buffer_size_mb: 256              # memory buffer per stream
  verify: true                     # enable integrity checks
  checksum: "sha256"               # sha256, crc32, or both
  resume: "always"                 # always, on_failure, never
  overwrite: "if_newer"            # always, never, if_newer, if_different

paths:
  source: "/data/projects"
  destination: "/backup/projects"
  
filters:
  include:
    - "*.cpp"
    - "*.h"
    - "*.py"
  exclude:
    - "*.tmp"
    - "*.log"
    - "node_modules/"
    - ".git/"

logging:
  level: "info"                    # debug, info, warn, error
  file: "/var/log/fastcopy/transfer.log"
  console: true
```

### Advanced Profile with Hooks

```yaml
# advanced-profile.yaml
transfer:
  mode: "turbo"
  streams: 48
  buffer_size_mb: 512
  verify: true
  resume: "always"
  
paths:
  source: "/mnt/storage/raw_footage"
  destination: "//nas/archive/project_alpha"

schedule:
  type: "watch"                    # one_time, watch, cron
  interval_seconds: 300            # for watch mode
  cron: "0 2 * * *"               # for cron mode (2 AM daily)

hooks:
  on_start: "echo 'Transfer started' | tee -a /var/log/transfer-audit.log"
  on_complete: "notify-send 'FastCopy Complete' && /scripts/post-transfer.sh"
  on_error: "/scripts/alert-admin.sh"
  on_resume: "logger 'FastCopy resumed after interruption'"

retry:
  max_attempts: 5
  backoff_seconds: 30
  exponential: true
```

### Network Transfer Profile

```yaml
# network-profile.yaml
transfer:
  mode: "balanced"
  streams: 16                      # lower for network stability
  buffer_size_mb: 128
  verify: true
  resume: "always"
  
paths:
  source: "/local/data"
  destination: "ssh://user@remote.host:/remote/data"

network:
  bandwidth_limit_mbps: 100        # throttle to prevent saturation
  timeout_seconds: 300
  retry_on_network_error: true
  
compression:
  enabled: true
  algorithm: "lz4"                 # lz4, zstd, gzip
  level: 3                         # 1-9
```

## Code Examples

### Python Integration

```python
import subprocess
import json
from pathlib import Path

class FastCopyManager:
    def __init__(self, fastcopy_bin="fastcopy"):
        self.fastcopy_bin = fastcopy_bin
    
    def copy(self, source, destination, streams=32, verify=True, dry_run=False):
        """Execute a FastCopy transfer."""
        cmd = [
            self.fastcopy_bin,
            str(source),
            str(destination),
            f"--streams={streams}",
            "--log-level=info"
        ]
        
        if verify:
            cmd.append("--verify")
        
        if dry_run:
            cmd.append("--dry-run")
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        if result.returncode != 0:
            raise RuntimeError(f"FastCopy failed: {result.stderr}")
        
        return result.stdout
    
    def copy_with_profile(self, profile_path):
        """Execute transfer using a profile."""
        cmd = [self.fastcopy_bin, "--profile", str(profile_path)]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        if result.returncode != 0:
            raise RuntimeError(f"Profile transfer failed: {result.stderr}")
        
        return result.stdout

# Usage
manager = FastCopyManager()

# Simple copy
manager.copy(
    source="/data/project",
    destination="/backup/project",
    streams=48,
    verify=True
)

# Profile-based copy
manager.copy_with_profile("backup-profile.yaml")
```

### Bash Script Integration

```bash
#!/bin/bash
# automated-backup.sh

set -euo pipefail

SOURCE_DIR="/data/production"
BACKUP_DIR="/mnt/backup/$(date +%Y%m%d)"
LOG_FILE="/var/log/fastcopy-backup.log"
PROFILE="/etc/fastcopy/backup-profile.yaml"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Execute transfer with error handling
if fastcopy --profile "$PROFILE" \
    --log-file "$LOG_FILE" \
    --tag "daily_backup_$(date +%Y%m%d)" 2>&1 | tee -a "$LOG_FILE"; then
    
    echo "[$(date)] Backup completed successfully" >> "$LOG_FILE"
    
    # Cleanup old backups (keep last 7 days)
    find /mnt/backup -type d -mtime +7 -exec rm -rf {} +
    
else
    echo "[$(date)] Backup FAILED" >> "$LOG_FILE"
    # Send alert
    mail -s "FastCopy Backup Failed" admin@example.com < "$LOG_FILE"
    exit 1
fi
```

### Node.js Integration

```javascript
const { spawn } = require('child_process');
const fs = require('fs').promises;
const path = require('path');

class FastCopyClient {
  constructor(binaryPath = 'fastcopy') {
    this.binaryPath = binaryPath;
  }

  async copy(source, destination, options = {}) {
    const args = [source, destination];
    
    if (options.streams) args.push(`--streams=${options.streams}`);
    if (options.verify) args.push('--verify');
    if (options.priority) args.push(`--priority=${options.priority}`);
    if (options.bufferSize) args.push(`--buffer-size=${options.bufferSize}`);
    if (options.logLevel) args.push(`--log-level=${options.logLevel}`);
    
    return this.execute(args);
  }

  async copyWithProfile(profilePath) {
    return this.execute(['--profile', profilePath]);
  }

  execute(args) {
    return new Promise((resolve, reject) => {
      const proc = spawn(this.binaryPath, args);
      let stdout = '';
      let stderr = '';

      proc.stdout.on('data', (data) => {
        stdout += data.toString();
        console.log(data.toString());
      });

      proc.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      proc.on('close', (code) => {
        if (code === 0) {
          resolve({ stdout, stderr, exitCode: code });
        } else {
          reject(new Error(`FastCopy failed with code ${code}: ${stderr}`));
        }
      });
    });
  }
}

// Usage
(async () => {
  const client = new FastCopyClient();
  
  try {
    await client.copy('/source/data', '/backup/data', {
      streams: 32,
      verify: true,
      priority: 'high',
      bufferSize: '256m'
    });
    console.log('Transfer completed successfully');
  } catch (error) {
    console.error('Transfer failed:', error.message);
    process.exit(1);
  }
})();
```

## Common Patterns

### Automated Scheduled Backups

```yaml
# scheduled-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 256
  verify: true
  resume: "always"

paths:
  source: "/data/production"
  destination: "/backup/daily"

schedule:
  type: "cron"
  cron: "0 2 * * *"              # 2 AM daily

filters:
  exclude:
    - "*.tmp"
    - "*.cache"
    - ".git/"

hooks:
  on_complete: "/scripts/notify-success.sh"
  on_error: "/scripts/alert-failure.sh"
```

### Large Media File Transfer

```yaml
# media-transfer.yaml
transfer:
  mode: "turbo"
  streams: 64                    # maximize parallelism
  buffer_size_mb: 1024           # large buffer for big files
  verify: true
  checksum: "sha256"

paths:
  source: "/media/raw_footage"
  destination: "/nas/archive"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"
    - "*.mkv"

retry:
  max_attempts: 10               # media files can be large
  backoff_seconds: 60
  exponential: true
```

### Development Environment Sync

```yaml
# dev-sync.yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true
  overwrite: "if_newer"          # only copy changed files

paths:
  source: "/home/dev/projects"
  destination: "/mnt/shared/projects"

filters:
  exclude:
    - "node_modules/"
    - ".git/"
    - "*.pyc"
    - "__pycache__/"
    - "*.o"
    - "build/"
    - "dist/"
    - ".venv/"

schedule:
  type: "watch"
  interval_seconds: 60           # sync every minute
```

## Troubleshooting

### Transfer Hangs or Stalls

```bash
# Reduce stream count for stability
fastcopy /source /dest --streams 8 --buffer-size 128m

# Enable debug logging
fastcopy /source /dest --log-level debug --log-file debug.log

# Check for disk I/O bottlenecks
iostat -x 1
```

### High Memory Usage

```yaml
# Reduce buffer sizes in profile
transfer:
  streams: 16                    # lower stream count
  buffer_size_mb: 64             # smaller buffers
  mode: "standard"               # conservative mode
```

### Network Timeouts

```yaml
# Configure network resilience
network:
  timeout_seconds: 600           # increase timeout
  retry_on_network_error: true
  
transfer:
  streams: 8                     # fewer streams for stability
  resume: "always"
  
retry:
  max_attempts: 10
  backoff_seconds: 120
```

### Verification Failures

```bash
# Use different checksum algorithm
fastcopy /source /dest --verify --checksum crc32

# Skip verification for speed (not recommended)
fastcopy /source /dest --no-verify

# Check source file integrity first
find /source -type f -exec sha256sum {} \; > checksums.txt
```

### Permission Issues

```bash
# Run with elevated privileges
sudo fastcopy /source /dest

# Set proper ownership after transfer
fastcopy /source /dest && sudo chown -R user:group /dest

# Preserve permissions in profile
```

```yaml
transfer:
  preserve_permissions: true
  preserve_timestamps: true
  preserve_ownership: true
```

## Performance Optimization

### Tuning for SSDs

```yaml
transfer:
  mode: "turbo"
  streams: 64                    # SSDs handle high parallelism
  buffer_size_mb: 512
  direct_io: true                # bypass OS cache
```

### Tuning for HDDs

```yaml
transfer:
  mode: "balanced"
  streams: 4                     # HDDs prefer sequential I/O
  buffer_size_mb: 128
  direct_io: false
```

### Network Optimization

```yaml
transfer:
  streams: 16
  buffer_size_mb: 256

network:
  bandwidth_limit_mbps: 900      # leave headroom
  tcp_window_size_kb: 2048       # tune for high-latency links
  
compression:
  enabled: true
  algorithm: "lz4"               # fast compression
  level: 1
```

## Environment Variables

FastCopy respects these environment variables:

```bash
# Set default streams
export FASTCOPY_STREAMS=32

# Set default buffer size
export FASTCOPY_BUFFER_SIZE=256m

# Set default log level
export FASTCOPY_LOG_LEVEL=info

# Set profile directory
export FASTCOPY_PROFILE_DIR=/etc/fastcopy/profiles

# Disable verification by default (not recommended)
export FASTCOPY_NO_VERIFY=1
```

## Integration with CI/CD

```yaml
# .github/workflows/deploy.yml
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
          wget https://fastcopy-releases/fastcopy-linux.tar.gz
          tar -xzf fastcopy-linux.tar.gz
          sudo install -m 755 fastcopy /usr/local/bin/
      
      - name: Transfer Artifacts
        run: |
          fastcopy ./dist /deploy/artifacts \
            --streams 32 \
            --verify \
            --tag "build_${{ github.sha }}" \
            --log-level info
```
