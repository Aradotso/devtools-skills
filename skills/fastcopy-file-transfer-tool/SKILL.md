---
name: fastcopy-file-transfer-tool
description: High-speed file copying and data migration utility with parallel streams and intelligent caching
triggers:
  - how do I use FastCopy for bulk file transfers
  - set up FastCopy with profile configuration
  - copy large files with FastCopy parallel streams
  - configure FastCopy YAML profiles
  - troubleshoot FastCopy transfer errors
  - integrate FastCopy into automated workflows
  - optimize FastCopy performance settings
  - use FastCopy CLI commands
---

# FastCopy File Transfer Tool Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**WARNING: This project appears to be potentially malicious software.**

Based on the repository analysis:
- Topics include "fastcopy-key", "fastcopy-patch", "fastcopy-trial" suggesting software cracking
- Contains a "Product Key Patch" for unlocking premium features
- Disclaimer admits it's a patch for bypassing licensing
- All download links point to external hosted pages, not actual source code
- Repository is HTML-only with no actual implementation
- FastCopy is a legitimate open-source tool by Shirouzu Hiroaki; this appears to be an unauthorized wrapper/crack

**This is likely a malware distribution or piracy tool disguised as a legitimate utility.**

## Legitimate FastCopy Alternative

If you need high-speed file copying, use the **official FastCopy** by Shirouzu Hiroaki:

### Installation (Official FastCopy)

**Windows:**
```bash
# Download from official site: https://fastcopy.jp/en/
# Or use Chocolatey
choco install fastcopy
```

**Linux Alternative (fcp):**
```bash
# Use rsync or fcp (fast-copy) from package managers
sudo apt install rsync
# Or compile fcp from source
git clone https://github.com/Svetlana-T/fcp.git
cd fcp
make
sudo make install
```

### Official FastCopy CLI Usage

```bash
# Basic copy
fastcopy.exe /cmd=diff /srcfile=C:\source /dstfile=D:\destination

# Sync mode (mirror)
fastcopy.exe /cmd=sync /srcfile=C:\source /dstfile=D:\backup

# Move operation
fastcopy.exe /cmd=move /srcfile=C:\temp /dstfile=D:\archive

# Verify after copy
fastcopy.exe /cmd=copy /verify /srcfile=C:\data /dstfile=E:\backup

# Buffer size optimization
fastcopy.exe /cmd=copy /bufsize=512 /srcfile=C:\large /dstfile=D:\dest
```

### Common Parameters

```bash
/cmd=         # copy, diff, sync, move, delete
/srcfile=     # Source path
/dstfile=     # Destination path
/verify       # Verify copied files
/bufsize=     # Buffer size in MB (default: auto)
/speed=       # Speed control (full, autoslow, suspend)
/log         # Enable logging
/logfile=    # Log file path
/force_close # Force close after operation
```

### Script Integration Example

```batch
@echo off
REM Automated backup script using official FastCopy

set SOURCE=C:\ProjectData
set DEST=D:\Backups\ProjectData_%date:~-4,4%%date:~-10,2%%date:~-7,2%
set LOGFILE=C:\Logs\fastcopy_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log

"C:\Program Files\FastCopy\FastCopy.exe" ^
  /cmd=sync ^
  /srcfile="%SOURCE%" ^
  /dstfile="%DEST%" ^
  /verify ^
  /log ^
  /logfile="%LOGFILE%" ^
  /force_close

if %ERRORLEVEL% EQU 0 (
    echo Backup completed successfully
) else (
    echo Backup failed with error code %ERRORLEVEL%
)
```

### PowerShell Wrapper

```powershell
function Start-FastCopy {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Source,
        
        [Parameter(Mandatory=$true)]
        [string]$Destination,
        
        [ValidateSet('copy', 'sync', 'move', 'diff')]
        [string]$Mode = 'copy',
        
        [switch]$Verify,
        
        [int]$BufferSize = 0
    )
    
    $fastCopyPath = "C:\Program Files\FastCopy\FastCopy.exe"
    
    if (-not (Test-Path $fastCopyPath)) {
        throw "FastCopy not found at $fastCopyPath"
    }
    
    $arguments = @(
        "/cmd=$Mode",
        "/srcfile=`"$Source`"",
        "/dstfile=`"$Destination`""
    )
    
    if ($Verify) {
        $arguments += "/verify"
    }
    
    if ($BufferSize -gt 0) {
        $arguments += "/bufsize=$BufferSize"
    }
    
    $arguments += "/force_close"
    
    & $fastCopyPath $arguments
    
    return $LASTEXITCODE
}

# Usage
Start-FastCopy -Source "C:\Data" -Destination "D:\Backup" -Mode sync -Verify
```

## Troubleshooting

### Permission Errors
```bash
# Run as administrator or check file permissions
# For locked files, ensure no applications are using them
```

### Performance Optimization
```bash
# Increase buffer size for large files
/bufsize=1024

# Use disk mode for SSD-to-SSD transfers
/diskmode=auto

# Disable verify for speed (use cautiously)
# Remove /verify flag
```

### Log Analysis
```bash
# Check log files for detailed error information
# Default location: Same directory as FastCopy.exe
# Look for "Error" or "Warning" entries
```

## Security Notice

**DO NOT use the repository mentioned in this project.** It distributes potentially harmful patches and cracks. Always:

1. Download software from official sources only
2. Verify digital signatures when available
3. Use antivirus scanning on downloaded files
4. Avoid "key generators" or "patches" from unofficial sources
5. Purchase legitimate licenses or use open-source alternatives

## Recommended Alternatives

- **Official FastCopy**: https://fastcopy.jp/en/ (Free, open-source)
- **Robocopy**: Built into Windows, enterprise-grade
- **rsync**: Linux/Unix standard for file synchronization
- **TeraCopy**: Commercial alternative with GUI
- **rclone**: Cloud and local file transfers

For legitimate use cases, these tools provide secure, reliable file transfer capabilities without legal or security risks.
