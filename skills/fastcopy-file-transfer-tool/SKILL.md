---
name: fastcopy-file-transfer-tool
description: High-performance file duplication and transfer utility with multi-threaded I/O, intelligent caching, and automation support
triggers:
  - how do I use FastCopy to transfer files
  - configure FastCopy with YAML profiles
  - set up parallel file copying with FastCopy
  - FastCopy command line examples
  - optimize FastCopy transfer settings
  - create FastCopy automation scripts
  - troubleshoot FastCopy file transfers
  - FastCopy profile configuration
---

# FastCopy File Transfer Tool Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility designed for professionals who need fast, reliable bulk data operations. It features multi-threaded I/O streams, intelligent predictive caching, integrity verification, and automation support through YAML profiles and CLI commands.

**Key capabilities:**
- Parallel stream engine (up to 64 concurrent streams)
- YAML-based profile configuration for repeatable workflows
- CLI and scriptable automation
- Integrity verification (SHA-256 + CRC-32)
- Auto-retry and resume functionality
- Cross-platform support (Windows, macOS, Linux)

## Installation

### Windows
Download from the official repository and extract the portable executable:
```powershell
# Extract to a directory in your PATH
Expand-Archive FastCopy-portable.zip -DestinationPath C:\Tools\FastCopy
$env:Path += ";C:\Tools\FastCopy"
```

### macOS
```bash
# Install via Homebrew (if available) or manual download
brew install fastcopy
# Or download portable binary
curl -L -o /usr/local/bin/fastcopy https://example.com/fastcopy-macos
chmod +x /usr/local/bin/fastcopy
```

### Linux
```bash
# Debian/Ubuntu
wget https://example.com/fastcopy-linux.deb
sudo dpkg -i fastcopy-linux.deb

# Or portable binary
curl -L -o /usr/local/bin/fastcopy https://example.com/fastcopy-linux
chmod +x /usr/local/bin/fastcopy
```

## CLI Commands

### Basic Usage

```bash
# Simple copy with default settings
fastcopy /source/path /destination/path

# Copy with maximum streams
fastcopy /source/path /destination/path --streams 64

# Copy with verification
fastcopy /source/path /destination/path --verify

# Turbo mode with logging
fastcopy /source/path /destination/path --mode turbo --log-level debug

# Copy with priority and tagging
fastcopy large_file.iso /backup/ --priority high --tag "backup_2026"

# Disable verification for speed
fastcopy /data /backup --no-verify --streams 48
```

### Profile-Based Execution

```bash
# Execute using a profile
fastcopy --profile my-transfer-profile.yaml

# Override profile settings
fastcopy --profile backup.yaml --streams 32 --mode balanced

# List available profiles
fastcopy --list-profiles

# Validate profile syntax
fastcopy --profile test.yaml --validate
```

### Advanced Options

```bash
# Watch mode with continuous sync
fastcopy --profile watch-profile.yaml --watch

# Dry run (preview without copying)
fastcopy /source /dest --dry-run

# Resume interrupted transfer
fastcopy /source /dest --resume

# Copy with exclude patterns
fastcopy /source /dest --exclude "*.tmp" --exclude "node_modules/"

# Set buffer size
fastcopy /source /dest --buffer-size-mb 512
```

## YAML Profile Configuration

### Basic Profile Structure

```yaml
# transfer-profile.yaml
transfer:
  mode: "turbo"                # standard, balanced, turbo
  streams: 32                  # concurrent streams (1-64)
  buffer_size_mb: 256          # buffer size per stream
  verify: true                 # enable integrity checking
  resume: always               # always, on_failure, never

paths:
  source: "/mnt/storage/projects"
  destination: "/backup/projects"

schedule:
  type: "one_time"             # one_time, watch, cron
  interval_seconds: 300

hooks:
  on_complete: "echo 'Transfer complete'"
  on_error: "/opt/scripts/alert.sh"
```

### Production Backup Profile

```yaml
# production-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 128
  verify: true
  resume: always
  
paths:
  source: "/var/www/production"
  destination: "//nas01/backups/production"
  
filters:
  include:
    - "*.php"
    - "*.js"
    - "*.css"
    - "*.html"
  exclude:
    - "cache/"
    - "logs/"
    - "*.tmp"
    
schedule:
  type: "cron"
  expression: "0 2 * * *"  # Daily at 2 AM
  
hooks:
  on_start: "/scripts/notify-start.sh"
  on_complete: "/scripts/notify-success.sh"
  on_error: "/scripts/notify-error.sh"
  
logging:
  level: "info"
  file: "/var/log/fastcopy/production-backup.log"
  rotate: true
  max_size_mb: 100
```

### Media Workflow Profile

```yaml
# media-sync.yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: true
  resume: always
  
paths:
  source: "/mnt/media/raw_footage"
  destination: "/mnt/nvme/active_projects"
  
filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.avi"
    - "*.mxf"
  exclude:
    - "*.lnk"
    - "Thumbs.db"
    
transfer_rules:
  large_file_threshold_mb: 1000
  chunk_size_mb: 100
  parallel_chunks: true
  
schedule:
  type: "watch"
  interval_seconds: 60
  
hooks:
  on_complete: "notify-send 'Media sync complete'"
```

### Database Backup Profile

```yaml
# db-backup.yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 256
  verify: true
  resume: on_failure
  
paths:
  source: "/var/lib/postgresql/backups"
  destination: "/backup/databases"
  
filters:
  include:
    - "*.sql"
    - "*.dump"
    - "*.tar.gz"
  
compression:
  enabled: false  # Files already compressed
  
hooks:
  on_start: "systemctl stop postgresql"
  on_complete: "systemctl start postgresql && /scripts/verify-backup.sh"
  on_error: "systemctl start postgresql && /scripts/alert-backup-failed.sh"
  
logging:
  level: "debug"
  file: "/var/log/fastcopy/db-backup.log"
```

## Scripting and Automation

### Bash Integration

```bash
#!/bin/bash
# automated-backup.sh

SOURCE="/data/important"
DEST="/backup/important"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG="/var/log/backup_${TIMESTAMP}.log"

echo "Starting backup at $(date)" >> "$LOG"

fastcopy "$SOURCE" "$DEST" \
  --streams 32 \
  --mode balanced \
  --verify \
  --tag "automated_${TIMESTAMP}" \
  --log-level info 2>&1 | tee -a "$LOG"

if [ $? -eq 0 ]; then
  echo "Backup successful at $(date)" >> "$LOG"
  # Clean up old backups
  find /backup/important -mtime +30 -delete
else
  echo "Backup failed at $(date)" >> "$LOG"
  mail -s "Backup Failed" admin@example.com < "$LOG"
fi
```

### Python Integration

```python
#!/usr/bin/env python3
# fastcopy_wrapper.py

import subprocess
import json
import os
from datetime import datetime

class FastCopyWrapper:
    def __init__(self, fastcopy_bin="fastcopy"):
        self.bin = fastcopy_bin
    
    def copy(self, source, destination, **kwargs):
        """Execute FastCopy with parameters"""
        cmd = [self.bin, source, destination]
        
        # Add optional parameters
        if kwargs.get('streams'):
            cmd.extend(['--streams', str(kwargs['streams'])])
        if kwargs.get('mode'):
            cmd.extend(['--mode', kwargs['mode']])
        if kwargs.get('verify', False):
            cmd.append('--verify')
        if kwargs.get('tag'):
            cmd.extend(['--tag', kwargs['tag']])
        
        print(f"Executing: {' '.join(cmd)}")
        
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True
        )
        
        return {
            'success': result.returncode == 0,
            'returncode': result.returncode,
            'stdout': result.stdout,
            'stderr': result.stderr
        }
    
    def copy_with_profile(self, profile_path):
        """Execute FastCopy using a YAML profile"""
        cmd = [self.bin, '--profile', profile_path]
        
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True
        )
        
        return {
            'success': result.returncode == 0,
            'returncode': result.returncode,
            'stdout': result.stdout,
            'stderr': result.stderr
        }

# Usage example
if __name__ == "__main__":
    fc = FastCopyWrapper()
    
    # Simple copy
    result = fc.copy(
        "/data/projects",
        "/backup/projects",
        streams=32,
        mode="turbo",
        verify=True,
        tag=f"backup_{datetime.now().strftime('%Y%m%d')}"
    )
    
    if result['success']:
        print("Copy successful!")
        print(result['stdout'])
    else:
        print("Copy failed!")
        print(result['stderr'])
```

### PowerShell Integration

```powershell
# FastCopyAutomation.ps1

function Invoke-FastCopy {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Source,
        
        [Parameter(Mandatory=$true)]
        [string]$Destination,
        
        [int]$Streams = 32,
        [string]$Mode = "balanced",
        [switch]$Verify,
        [string]$Tag,
        [string]$LogLevel = "info"
    )
    
    $args = @($Source, $Destination)
    $args += "--streams", $Streams
    $args += "--mode", $Mode
    $args += "--log-level", $LogLevel
    
    if ($Verify) {
        $args += "--verify"
    }
    
    if ($Tag) {
        $args += "--tag", $Tag
    }
    
    Write-Host "Executing FastCopy with args: $($args -join ' ')"
    
    $process = Start-Process -FilePath "fastcopy" -ArgumentList $args -Wait -PassThru -NoNewWindow
    
    return @{
        ExitCode = $process.ExitCode
        Success = ($process.ExitCode -eq 0)
    }
}

# Usage
$result = Invoke-FastCopy `
    -Source "D:\Projects" `
    -Destination "E:\Backup\Projects" `
    -Streams 48 `
    -Mode "turbo" `
    -Verify `
    -Tag "weekly_backup"

if ($result.Success) {
    Write-Host "Backup completed successfully"
} else {
    Write-Error "Backup failed with exit code $($result.ExitCode)"
}
```

## Common Patterns

### Incremental Backup

```yaml
# incremental-backup.yaml
transfer:
  mode: "balanced"
  streams: 24
  buffer_size_mb: 128
  verify: true
  resume: always
  incremental: true          # Only copy changed files
  
paths:
  source: "/data"
  destination: "/backup/incremental"
  
filters:
  exclude:
    - "*.log"
    - "cache/"
    - "temp/"
```

### Network Transfer Optimization

```yaml
# network-transfer.yaml
transfer:
  mode: "turbo"
  streams: 16               # Lower for network stability
  buffer_size_mb: 64        # Smaller buffers for network
  verify: true
  resume: always
  network_optimized: true
  
paths:
  source: "/local/data"
  destination: "//remote-server/share/data"
  
retry:
  max_attempts: 5
  backoff_seconds: 30
```

### Large File Optimization

```bash
# For very large files (>10GB), use chunking
fastcopy huge_file.dat /destination/ \
  --streams 64 \
  --buffer-size-mb 1024 \
  --chunk-size-mb 500 \
  --parallel-chunks \
  --verify
```

## Troubleshooting

### Transfer Speed Issues

```bash
# Check system I/O limits
ulimit -n  # File descriptor limit

# Increase if needed
ulimit -n 65536

# Monitor transfer in real-time
fastcopy /source /dest --streams 64 --log-level debug

# Try different stream counts
fastcopy /source /dest --streams 16  # Start conservative
fastcopy /source /dest --streams 32  # Increase gradually
```

### Verification Failures

```bash
# Run with detailed logging
fastcopy /source /dest --verify --log-level debug

# Check source file integrity first
sha256sum /source/file.dat

# Retry with slower mode
fastcopy /source /dest --mode standard --verify --streams 8
```

### Network Transfer Issues

```yaml
# network-resilient.yaml
transfer:
  mode: "balanced"
  streams: 8                # Reduce streams for stability
  buffer_size_mb: 32
  verify: true
  resume: always
  
retry:
  max_attempts: 10
  backoff_seconds: 60
  exponential_backoff: true
  
timeout:
  connect_seconds: 30
  read_seconds: 300
  write_seconds: 300
```

### Memory Usage Issues

```bash
# Reduce buffer size and streams
fastcopy /source /dest \
  --streams 8 \
  --buffer-size-mb 64 \
  --mode standard

# Monitor memory usage
watch -n 1 'ps aux | grep fastcopy'
```

### Permission Errors

```bash
# Verify permissions
ls -la /source/path
ls -la /destination/path

# Run with proper permissions
sudo fastcopy /protected/source /protected/dest

# Or adjust ownership
sudo chown -R $USER:$USER /destination/path
```

## Environment Variables

```bash
# Set default configuration
export FASTCOPY_DEFAULT_STREAMS=32
export FASTCOPY_DEFAULT_MODE=balanced
export FASTCOPY_LOG_LEVEL=info
export FASTCOPY_CONFIG_DIR="$HOME/.config/fastcopy"

# API integration (if using)
export OPENAI_API_KEY=your_openai_key_here
export ANTHROPIC_API_KEY=your_claude_key_here
```

## Best Practices

1. **Start with balanced mode** and adjust based on performance
2. **Always use --verify** for critical data
3. **Test with --dry-run** before large operations
4. **Use YAML profiles** for repeatable workflows
5. **Monitor disk I/O** to avoid bottlenecks
6. **Adjust streams** based on disk type (SSD vs HDD)
7. **Use logging** for automated operations
8. **Implement hooks** for notifications and error handling
