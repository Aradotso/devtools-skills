---
name: fastcopy-file-transfer-tool
description: High-performance file copying and synchronization utility with parallel streaming and intelligent caching
triggers:
  - how do I use FastCopy to copy files quickly
  - configure FastCopy for bulk file transfers
  - set up FastCopy with YAML profiles
  - optimize FastCopy transfer speed
  - FastCopy CLI commands and options
  - troubleshoot FastCopy file copy issues
  - integrate FastCopy into backup scripts
  - use FastCopy parallel streams
---

# FastCopy File Transfer Tool Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**⚠️ WARNING: This project appears to be a potentially illegitimate software distribution.**

Based on the repository analysis:
- Topics include "fastcopy-key", "fastcopy-patch", "fastcopy-trial" - suggesting unauthorized key generation
- README promotes a "Product Key Patch" to unlock premium features
- Links redirect to external sites rather than legitimate software downloads
- No actual source code is present in the repository
- The project claims to be a "portable utility" but contains only HTML/marketing materials

**FastCopy** is actually a legitimate open-source file transfer utility originally created by Shirouzu Hiroaki (available at https://fastcopy.jp/). The genuine FastCopy is free and open source under GPLv3.

## Legitimate FastCopy Usage

If you want to use the real FastCopy tool:

### Installation

**Windows:**
```bash
# Download from official site: https://fastcopy.jp/en/
# Or use winget:
winget install FastCopy
```

**Linux (alternative tools with similar functionality):**
```bash
# rsync - standard high-performance copy tool
sudo apt install rsync

# or fcp - Fast Copy
cargo install fcp
```

### CLI Usage (Real FastCopy)

```bash
# Basic syntax
fastcopy.exe /cmd=diff /srcfile="C:\source" /dstfile="D:\destination"

# Common commands
/cmd=diff           # Differential copy
/cmd=sync           # Synchronization
/cmd=move           # Move files
/cmd=delete         # Delete files

# Options
/force_close        # Force close after completion
/estimate          # Estimate time before starting
/verify            # Verify after copy
/linkdest          # Create hardlinks instead of copying
/bufsize=N         # Set buffer size in MB
```

### Example Commands

```bash
# Fast differential backup
fastcopy.exe /cmd=diff /srcfile="C:\Projects" /dstfile="E:\Backup\Projects" /bufsize=512 /verify

# Sync directories with verification
fastcopy.exe /cmd=sync /srcfile="C:\Data" /dstfile="\\NAS\Data" /verify /force_close

# Move large files with progress
fastcopy.exe /cmd=move /srcfile="C:\Videos" /dstfile="D:\Archive" /estimate
```

### Alternative: rsync (Cross-platform)

```bash
# Basic sync with progress
rsync -avh --progress /source/ /destination/

# Fast transfer with parallel threads (using parallel)
find /source -type f | parallel -j 8 rsync -av {} /destination/

# Network transfer with compression
rsync -avz --progress /local/path user@remote:/remote/path

# Mirror with deletions
rsync -avh --delete /source/ /destination/

# Resume interrupted transfer
rsync -avhP /source/ /destination/
```

### Alternative: fcp (Rust-based fast copy)

```bash
# Install
cargo install fcp

# Basic usage
fcp source_file destination_file

# Copy directory recursively
fcp -r /source/dir /dest/dir

# With progress bar
fcp --progress large_file.iso /backup/
```

## Scripting Integration

### Bash Backup Script

```bash
#!/bin/bash
# backup.sh - Automated backup with rsync

SOURCE="/home/user/projects"
DEST="/mnt/backup/projects"
LOG="/var/log/backup.log"

echo "$(date): Starting backup" >> "$LOG"

rsync -avh \
  --delete \
  --progress \
  --stats \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='*.tmp' \
  "$SOURCE/" "$DEST/" 2>&1 | tee -a "$LOG"

if [ $? -eq 0 ]; then
  echo "$(date): Backup completed successfully" >> "$LOG"
else
  echo "$(date): Backup failed" >> "$LOG"
  exit 1
fi
```

### PowerShell Backup Script

```powershell
# backup.ps1 - Windows backup script

$source = "C:\Projects"
$dest = "E:\Backup\Projects"
$logPath = "C:\Logs\backup.log"

$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
"$timestamp : Starting backup" | Out-File -Append $logPath

# Using robocopy (built into Windows)
robocopy $source $dest /MIR /R:3 /W:5 /MT:8 /LOG+:$logPath /NP

if ($LASTEXITCODE -le 7) {
    "$timestamp : Backup completed" | Out-File -Append $logPath
} else {
    "$timestamp : Backup failed with code $LASTEXITCODE" | Out-File -Append $logPath
    exit 1
}
```

## Configuration Patterns

### rsync Configuration File

```bash
# ~/.rsyncrc or /etc/rsyncd.conf

# Exclude patterns file
# Create: ~/.rsync-exclude
cat > ~/.rsync-exclude << 'EOF'
.git/
node_modules/
__pycache__/
*.pyc
*.tmp
.DS_Store
Thumbs.db
EOF

# Use in commands
rsync -avh --exclude-from="$HOME/.rsync-exclude" /source/ /dest/
```

### Python Wrapper for Parallel Copying

```python
#!/usr/bin/env python3
import subprocess
import multiprocessing
from pathlib import Path
import argparse

def copy_file(args):
    src, dst = args
    subprocess.run(['rsync', '-avh', str(src), str(dst)], check=True)
    return src

def parallel_copy(source_dir, dest_dir, workers=4):
    source = Path(source_dir)
    dest = Path(dest_dir)
    dest.mkdir(parents=True, exist_ok=True)
    
    files = [(f, dest / f.relative_to(source)) for f in source.rglob('*') if f.is_file()]
    
    with multiprocessing.Pool(workers) as pool:
        results = pool.map(copy_file, files)
    
    print(f"Copied {len(results)} files")

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('source', help='Source directory')
    parser.add_argument('dest', help='Destination directory')
    parser.add_argument('-j', '--jobs', type=int, default=4, help='Parallel workers')
    
    args = parser.parse_args()
    parallel_copy(args.source, args.dest, args.jobs)
```

## Performance Optimization

### Large File Transfers

```bash
# Use larger block size for big files
rsync -avh --progress --inplace --no-whole-file \
  --bwlimit=50000 \
  large_file.iso /destination/

# Network optimization
rsync -avz --compress-level=1 --progress \
  --partial --partial-dir=.rsync-partial \
  /local/ user@remote:/remote/
```

### SSD-Optimized Copying

```bash
# Disable synchronous writes for speed (use cautiously)
rsync -avh --no-o --no-g --no-perms \
  --inplace \
  /source/ /ssd-destination/

# Or with dd for raw speed
dd if=/dev/source of=/dev/destination bs=4M status=progress oflag=direct
```

## Troubleshooting

### Permission Issues

```bash
# Run with sudo/admin privileges
sudo rsync -avh /source/ /dest/

# Or preserve permissions differently
rsync -rlptD /source/ /dest/  # Without owner/group
```

### Network Interruptions

```bash
# Resume interrupted transfer
rsync -avhP --partial /source/ user@remote:/dest/

# With retry logic in bash
for i in {1..5}; do
  rsync -avhP /source/ /dest/ && break || sleep 10
done
```

### Verification

```bash
# Checksum verification after copy
rsync -avh --checksum /source/ /dest/

# Manual verification
cd /source && find . -type f -exec md5sum {} \; | sort > /tmp/source.md5
cd /dest && find . -type f -exec md5sum {} \; | sort > /tmp/dest.md5
diff /tmp/source.md5 /tmp/dest.md5
```

## Security Notice

**Do not use unauthorized patches, keygens, or cracks.** These often contain malware and violate software licenses. Use legitimate open-source tools or properly licensed software.

For enterprise file transfer needs, consider:
- **rsync** (open source, GPLv3)
- **rclone** (open source, MIT)
- **Syncthing** (open source, MPLv2)
- Commercial solutions with proper licensing
