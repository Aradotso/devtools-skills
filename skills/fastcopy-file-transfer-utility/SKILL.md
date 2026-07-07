---
name: fastcopy-file-transfer-utility
description: High-performance file copying and transfer utility with parallel streams, integrity verification, and automation capabilities
triggers:
  - "how do I use FastCopy for bulk file transfers"
  - "configure FastCopy with YAML profiles"
  - "set up parallel file copying with FastCopy"
  - "FastCopy CLI commands and options"
  - "automate file transfers with FastCopy"
  - "verify file integrity during FastCopy transfers"
  - "create FastCopy transfer profiles"
  - "optimize FastCopy performance settings"
---

# FastCopy File Transfer Utility

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

FastCopy is a high-performance file duplication and transfer utility that supports multi-threaded copying, integrity verification, profile-based automation, and cross-platform operation. It's designed for scenarios requiring fast, reliable bulk file transfers, backups, migrations, and synchronization.

**Key capabilities:**
- Parallel stream transfers (up to 64 concurrent streams)
- YAML-based profile configuration
- CLI and GUI modes
- Integrity checking (SHA-256, CRC-32)
- Auto-retry and resume on failure
- Platform support: Windows, macOS, Linux

## Installation

### Windows
```bash
# Download from official source or repository
# Extract to desired location
# Add to PATH for CLI access
```

### macOS
```bash
# Install via package manager or direct download
# Make executable
chmod +x fastcopy
sudo mv fastcopy /usr/local/bin/
```

### Linux
```bash
# Ubuntu/Debian
sudo apt-get install fastcopy

# Fedora
sudo dnf install fastcopy

# Arch
yay -S fastcopy
```

## CLI Commands

### Basic Usage

```bash
# Simple copy with default settings
fastcopy /source/path /destination/path

# Copy with specific number of streams
fastcopy /source /dest --streams 32

# Copy with integrity verification
fastcopy /source /dest --verify

# High priority transfer with debug logging
fastcopy /source /dest --priority high --log-level debug

# Disable verification for speed
fastcopy large_file.iso /backups/ --no-verify

# Add metadata tag
fastcopy /source /dest --tag "backup_2026_07_15"
```

### Profile-Based Transfers

```bash
# Use predefined profile
fastcopy --profile /path/to/profile.yaml

# Override profile settings
fastcopy --profile profile.yaml --streams 64

# List available profiles
fastcopy --list-profiles

# Validate profile without executing
fastcopy --profile profile.yaml --dry-run
```

### Advanced Options

```bash
# Resume interrupted transfer
fastcopy /source /dest --resume

# Watch mode for continuous sync
fastcopy /source /dest --watch --interval 300

# Exclude patterns
fastcopy /source /dest --exclude "*.tmp,*.log"

# Include only specific patterns
fastcopy /source /dest --include "*.mp4,*.mov"

# Copy with bandwidth limit (MB/s)
fastcopy /source /dest --limit 100

# Generate transfer report
fastcopy /source /dest --report /path/to/report.json
```

## Configuration

### YAML Profile Structure

Create a profile file (e.g., `production-backup.yaml`):

```yaml
# Production backup profile
transfer:
  mode: "turbo"                    # standard, balanced, turbo
  streams: 48                      # concurrent streams (1-64)
  buffer_size_mb: 256              # buffer per stream
  verify: true                     # enable integrity checks
  resume: always                   # always, on_failure, never
  retry_count: 5
  retry_backoff_seconds: 30

paths:
  source: "/mnt/production/data"
  destination: "/mnt/backup/daily"

filters:
  include:
    - "*.sql"
    - "*.bak"
    - "*.json"
  exclude:
    - "*.tmp"
    - "*.cache"
    - ".git/*"

schedule:
  type: "cron"                     # one_time, watch, cron
  cron_expression: "0 2 * * *"     # daily at 2 AM

performance:
  priority: "high"                 # low, normal, high, realtime
  use_cache: true
  prefetch: true
  compression: false               # compress during transfer

logging:
  level: "info"                    # debug, info, warn, error
  file: "/var/log/fastcopy/production-backup.log"
  rotate: true
  max_size_mb: 100

hooks:
  on_start: "/opt/scripts/notify_start.sh"
  on_complete: "/opt/scripts/notify_complete.sh {bytes} {duration}"
  on_error: "/opt/scripts/alert_error.sh {error}"
  on_retry: "echo 'Retrying transfer...'"

notifications:
  email:
    enabled: true
    smtp_server: "smtp.company.com"
    from: "fastcopy@company.com"
    to: ["admin@company.com"]
    on_events: ["complete", "error"]
```

### Minimal Profile

```yaml
transfer:
  mode: "balanced"
  streams: 16
  verify: true

paths:
  source: "/source/folder"
  destination: "/backup/folder"
```

### Watch Mode Profile

```yaml
transfer:
  mode: "standard"
  streams: 8
  verify: true
  resume: on_failure

paths:
  source: "/home/user/Documents"
  destination: "/mnt/nas/Documents"

schedule:
  type: "watch"
  interval_seconds: 600           # check every 10 minutes

filters:
  exclude:
    - ".DS_Store"
    - "Thumbs.db"
    - "*.swp"
```

## Common Patterns

### Bulk Media Migration

```bash
# Profile: media-migration.yaml
```

```yaml
transfer:
  mode: "turbo"
  streams: 64
  buffer_size_mb: 512
  verify: true
  resume: always

paths:
  source: "/mnt/old-storage/media"
  destination: "/mnt/new-storage/media"

filters:
  include:
    - "*.mp4"
    - "*.mov"
    - "*.mkv"
    - "*.avi"

performance:
  priority: "high"
  use_cache: true
  prefetch: true

logging:
  level: "info"
  file: "/var/log/fastcopy/media-migration.log"

hooks:
  on_complete: "notify-send 'Media migration complete'"
```

```bash
fastcopy --profile media-migration.yaml
```

### Database Backup

```bash
# Profile: db-backup.yaml
```

```yaml
transfer:
  mode: "balanced"
  streams: 16
  buffer_size_mb: 128
  verify: true
  resume: always

paths:
  source: "/var/lib/postgresql/backups"
  destination: "//nas.company.local/db-backups"

schedule:
  type: "cron"
  cron_expression: "0 3 * * *"

hooks:
  on_start: "pg_dump -U postgres mydb > /var/lib/postgresql/backups/mydb_$(date +%Y%m%d).sql"
  on_complete: "find /var/lib/postgresql/backups -mtime +7 -delete"
  on_error: "curl -X POST https://hooks.slack.com/services/$SLACK_WEBHOOK -d '{\"text\":\"DB backup failed\"}'"
```

### Development Environment Sync

```javascript
// Node.js wrapper for FastCopy
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

class FastCopySync {
  constructor(profilePath) {
    this.profilePath = profilePath;
  }

  sync(options = {}) {
    const args = ['fastcopy', '--profile', this.profilePath];
    
    if (options.streams) args.push('--streams', options.streams);
    if (options.priority) args.push('--priority', options.priority);
    if (options.dryRun) args.push('--dry-run');
    
    const cmd = args.join(' ');
    console.log(`Executing: ${cmd}`);
    
    try {
      const output = execSync(cmd, { encoding: 'utf8' });
      console.log(output);
      return { success: true, output };
    } catch (error) {
      console.error('FastCopy failed:', error.message);
      return { success: false, error: error.message };
    }
  }

  createProfile(source, destination, config = {}) {
    const profile = {
      transfer: {
        mode: config.mode || 'balanced',
        streams: config.streams || 16,
        verify: config.verify !== false,
        resume: config.resume || 'on_failure'
      },
      paths: { source, destination },
      filters: config.filters || {},
      hooks: config.hooks || {}
    };

    const profilePath = path.join(process.cwd(), 'fastcopy-profile.yaml');
    fs.writeFileSync(profilePath, JSON.stringify(profile, null, 2));
    return profilePath;
  }
}

// Usage
const syncer = new FastCopySync('/config/dev-sync.yaml');
syncer.sync({ priority: 'normal' });
```

### Python Automation Script

```python
#!/usr/bin/env python3
import subprocess
import yaml
import os
from datetime import datetime

class FastCopyManager:
    def __init__(self, profile_path=None):
        self.profile_path = profile_path
    
    def execute(self, source=None, destination=None, **kwargs):
        """Execute FastCopy with optional parameters"""
        cmd = ['fastcopy']
        
        if self.profile_path:
            cmd.extend(['--profile', self.profile_path])
        elif source and destination:
            cmd.extend([source, destination])
        else:
            raise ValueError("Must provide profile or source/destination")
        
        # Add optional arguments
        if kwargs.get('streams'):
            cmd.extend(['--streams', str(kwargs['streams'])])
        if kwargs.get('verify'):
            cmd.append('--verify')
        if kwargs.get('priority'):
            cmd.extend(['--priority', kwargs['priority']])
        if kwargs.get('tag'):
            cmd.extend(['--tag', kwargs['tag']])
        
        print(f"Executing: {' '.join(cmd)}")
        
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, check=True)
            return {'success': True, 'output': result.stdout}
        except subprocess.CalledProcessError as e:
            return {'success': False, 'error': e.stderr}
    
    def create_profile(self, source, destination, **config):
        """Generate YAML profile programmatically"""
        profile = {
            'transfer': {
                'mode': config.get('mode', 'balanced'),
                'streams': config.get('streams', 16),
                'verify': config.get('verify', True),
                'resume': config.get('resume', 'on_failure')
            },
            'paths': {
                'source': source,
                'destination': destination
            }
        }
        
        if 'filters' in config:
            profile['filters'] = config['filters']
        
        if 'hooks' in config:
            profile['hooks'] = config['hooks']
        
        profile_path = f'/tmp/fastcopy_profile_{datetime.now().strftime("%Y%m%d_%H%M%S")}.yaml'
        
        with open(profile_path, 'w') as f:
            yaml.dump(profile, f, default_flow_style=False)
        
        return profile_path

# Usage example
if __name__ == '__main__':
    manager = FastCopyManager()
    
    # Create dynamic profile
    profile = manager.create_profile(
        '/data/source',
        '/backup/destination',
        mode='turbo',
        streams=32,
        verify=True,
        filters={'exclude': ['*.tmp', '*.log']},
        hooks={'on_complete': 'echo "Backup complete"'}
    )
    
    # Execute transfer
    result = manager.execute(profile_path=profile)
    print(result)
```

## Environment Variables

FastCopy supports environment variable configuration:

```bash
# Set default profile directory
export FASTCOPY_PROFILE_DIR="/etc/fastcopy/profiles"

# Default log level
export FASTCOPY_LOG_LEVEL="info"

# Default stream count
export FASTCOPY_STREAMS=32

# Notification endpoints
export FASTCOPY_WEBHOOK_URL="https://hooks.slack.com/services/YOUR_WEBHOOK"
export SMTP_SERVER="smtp.company.com"
export SMTP_FROM="fastcopy@company.com"
export SMTP_TO="admin@company.com"

# API integration keys (if using AI features)
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
```

## Troubleshooting

### Transfer Fails Midway

```bash
# Enable auto-resume
fastcopy /source /dest --resume always --retry-count 10

# Check logs for specific error
fastcopy /source /dest --log-level debug --log-file /tmp/fastcopy.log
```

### Performance Issues

```yaml
# Adjust profile for better performance
transfer:
  mode: "turbo"
  streams: 48                    # increase for network transfers
  buffer_size_mb: 512            # increase for large files
  
performance:
  priority: "high"
  use_cache: true
  prefetch: true                 # enable predictive caching
```

### Permission Errors

```bash
# Run with elevated privileges
sudo fastcopy /source /dest

# Or fix ownership/permissions
sudo chown -R $USER:$USER /destination
chmod -R u+rw /destination
```

### Integrity Check Failures

```bash
# Force re-verification
fastcopy /source /dest --verify --force

# Use specific checksum algorithm
fastcopy /source /dest --checksum sha256

# Skip verification for speed (use cautiously)
fastcopy /source /dest --no-verify
```

### Profile Validation

```bash
# Validate YAML syntax
fastcopy --profile profile.yaml --validate

# Dry run to test without copying
fastcopy --profile profile.yaml --dry-run

# Check profile defaults
fastcopy --show-defaults
```

### Network Transfer Issues

```yaml
# Add retry logic and bandwidth limiting
transfer:
  retry_count: 10
  retry_backoff_seconds: 60
  timeout_seconds: 300

performance:
  bandwidth_limit_mbps: 100      # prevent saturation
  tcp_window_size_kb: 256        # adjust for WAN
```

## Best Practices

1. **Always verify critical transfers**: Use `verify: true` for important data
2. **Use profiles for recurring tasks**: Store configurations in version control
3. **Monitor large transfers**: Use `--log-level info` and check logs
4. **Test with dry-run**: Validate configuration before executing
5. **Implement hooks**: Automate pre/post transfer tasks
6. **Set appropriate stream counts**: 16-32 for network, 32-64 for local SSD
7. **Enable resume for unreliable connections**: Use `resume: always`
8. **Tag transfers**: Use `--tag` for tracking and auditing
