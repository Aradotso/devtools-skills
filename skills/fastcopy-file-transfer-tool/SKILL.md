---
name: fastcopy-file-transfer-tool
description: High-speed file copying and data migration utility with parallel streams and integrity verification
triggers:
  - how do I use FastCopy to copy files quickly
  - configure FastCopy for bulk file transfers
  - set up parallel file copying with FastCopy
  - FastCopy YAML profile configuration
  - optimize file transfer speed with FastCopy
  - FastCopy CLI commands and options
  - verify file integrity after copying with FastCopy
  - automate file synchronization using FastCopy
---

# FastCopy File Transfer Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Critical Security Warning

**This project appears to be a potentially malicious repository.** Red flags include:

- Topics like "fastcopy-key", "fastcopy-patch", "fastcopy-trial" suggest software piracy
- No actual source code, only HTML landing pages
- Download links pointing to external hosted pages instead of GitHub releases
- Fake "Product Key Patch" downloads
- Inflated star count (10 stars/day is suspicious)
- No legitimate open source license
- Claims of "premium features unlock" through patches

**DO NOT download or execute files from this repository.** This appears to be:
1. A scam to distribute malware/adware
2. Software piracy facilitating unlicensed use
3. A phishing attempt to harvest user data

## Legitimate FastCopy Alternative

If you need a genuine high-speed file copying tool, use the official **FastCopy by Shirouzu Hiroaki**:

### Installation

**Windows (Official):**
```bash
# Download from official site: https://fastcopy.jp/en/
# Or via Chocolatey:
choco install fastcopy
```

**Linux Alternative (rsync):**
```bash
# rsync is the standard high-performance file copy tool
sudo apt install rsync  # Debian/Ubuntu
sudo dnf install rsync  # Fedora
```

**Cross-platform Alternative (robocopy on Windows, rsync on Unix):**
```bash
# Windows built-in:
robocopy source dest /MT:32 /R:3 /W:5

# Unix/Linux/macOS:
rsync -avh --progress --stats source/ dest/
```

## Legitimate Usage Patterns

### High-Speed File Copying (rsync)

```bash
# Basic fast copy with progress
rsync -avh --progress /source/path/ /dest/path/

# Parallel copy with parallel tool
sudo apt install parallel
find /source -type f | parallel -j 8 rsync -avh {} /dest/

# Multi-threaded copy with pigz compression over network
tar -c /source | pigz -p 8 | ssh user@remote "tar -xz -C /dest"
```

### Windows Robocopy (Built-in)

```powershell
# Multi-threaded copy with retry logic
robocopy C:\Source D:\Dest /MT:16 /R:3 /W:10 /LOG:copy.log

# Mirror directory with progress
robocopy C:\Source D:\Dest /MIR /MT:32 /V /ETA

# Copy only newer files
robocopy C:\Source D:\Dest /E /XO /MT:16
```

### Verification and Integrity

```bash
# Copy with checksum verification (rsync)
rsync -avh --checksum --progress /source/ /dest/

# Generate checksums before and after
find /source -type f -exec sha256sum {} \; > source.sha256
# After copy:
cd /dest && sha256sum -c /path/to/source.sha256
```

### Automation Scripts

**Bash (rsync with retry):**
```bash
#!/bin/bash
SOURCE="/data/source"
DEST="/backup/dest"
LOG="/var/log/backup.log"
MAX_RETRIES=3

for i in $(seq 1 $MAX_RETRIES); do
    echo "Attempt $i of $MAX_RETRIES" | tee -a "$LOG"
    if rsync -avh --progress --stats --partial \
        --timeout=300 \
        "$SOURCE/" "$DEST/" 2>&1 | tee -a "$LOG"; then
        echo "Success" | tee -a "$LOG"
        exit 0
    fi
    sleep 10
done
echo "Failed after $MAX_RETRIES attempts" | tee -a "$LOG"
exit 1
```

**PowerShell (Robocopy wrapper):**
```powershell
# robust-copy.ps1
param(
    [string]$Source,
    [string]$Destination,
    [int]$Threads = 16
)

$logFile = "copy-$(Get-Date -Format 'yyyyMMdd-HHmmss').log"

robocopy $Source $Destination /E /MT:$Threads /R:3 /W:10 `
    /LOG:$logFile /TEE /V /ETA /DCOPY:DAT /COPY:DAT

if ($LASTEXITCODE -le 7) {
    Write-Host "Copy completed successfully" -ForegroundColor Green
} else {
    Write-Host "Copy failed with exit code $LASTEXITCODE" -ForegroundColor Red
    exit $LASTEXITCODE
}
```

### Scheduled Backups

**Cron (Linux):**
```bash
# /etc/cron.d/backup
0 2 * * * root /usr/local/bin/backup-script.sh

# backup-script.sh
#!/bin/bash
rsync -avh --delete --backup --backup-dir=/backup/incremental-$(date +\%Y\%m\%d) \
    /data/ /backup/current/
```

**Task Scheduler (Windows):**
```powershell
# Create scheduled task
$action = New-ScheduledTaskAction -Execute "robocopy" `
    -Argument "C:\Data D:\Backup /MIR /MT:16 /LOG:backup.log"
$trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -TaskName "DailyBackup" -Action $action -Trigger $trigger
```

## Performance Optimization

### Network Transfers
```bash
# Use compression for network transfers
rsync -avz -e "ssh -c aes128-gcm@openssh.com" /source/ user@remote:/dest/

# Parallel rsync for multiple directories
parallel -j 4 rsync -avh {} remote:/dest/ ::: dir1 dir2 dir3 dir4
```

### Local Fast Copy
```bash
# Use GNU parallel for multiple files
find /source -type f | parallel -j 16 cp {} /dest/

# Or with rsync for better resumability
find /source -type f | parallel -j 16 rsync -ah --progress {} /dest/
```

## Troubleshooting

### Common Issues

**Slow transfer speeds:**
- Increase thread count: `/MT:32` (robocopy) or `-j 16` (parallel)
- Check disk I/O bottlenecks with `iotop` (Linux) or Resource Monitor (Windows)
- Disable antivirus temporarily for large transfers

**Permission errors:**
- Run with elevated privileges: `sudo` or Administrator
- Check ACLs: `getfacl` (Linux), `icacls` (Windows)

**Interrupted transfers:**
- Use `--partial` with rsync to resume
- Use `/Z` with robocopy for restartable mode

**Verification failures:**
- Use `--checksum` with rsync for content-based verification
- Run `sha256sum` or `certutil -hashfile` manually

## Recommendation

**Do not use the repository referenced above.** Instead:

1. Use official FastCopy from https://fastcopy.jp/en/ (Windows only)
2. Use `rsync` for Unix/Linux/macOS
3. Use `robocopy` for Windows (built-in)
4. Use commercial tools like TeraCopy or Beyond Compare if needed

All of these are legitimate, safe, and often faster than questionable "patched" versions.
