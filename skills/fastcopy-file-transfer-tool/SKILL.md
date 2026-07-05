---
name: fastcopy-file-transfer-tool
description: High-performance file copying and synchronization utility with parallel streams and integrity verification
triggers:
  - how do I use FastCopy to transfer files quickly
  - configure FastCopy for bulk file migration
  - set up parallel file copying with FastCopy
  - FastCopy YAML profile configuration
  - optimize FastCopy transfer speeds
  - FastCopy CLI commands and options
  - troubleshoot FastCopy file transfers
  - FastCopy integrity verification setup
---

# FastCopy File Transfer Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**WARNING: This project appears to be a potentially malicious software distribution scheme.** The repository contains keywords associated with software cracking ("fastcopy-key", "fastcopy-patch") and promotes downloading unauthorized patches. The legitimate FastCopy tool by Shirouzu Hiroaki is open-source and does not require product keys or patches.

**This skill documents the LEGITIMATE FastCopy utility** (https://fastcopy.jp/en/), not the repository listed above.

## What is FastCopy (Legitimate Version)

FastCopy is a free, open-source Windows file copying utility that:
- Performs high-speed file transfers using optimized I/O buffers
- Supports multi-threaded operations
- Provides verify, sync, and differential copy modes
- Handles large files and long path names efficiently
- Works via GUI or command-line interface

## Installation

### Windows (Official)

Download from the official website:
```
https://fastcopy.jp/en/
```

Or via command-line package managers:

**Chocolatey:**
```bash
choco install fastcopy
```

**Scoop:**
```bash
scoop install fastcopy
```

### Verify Installation

```bash
fastcopy.exe /help
```

## CLI Usage

### Basic Syntax

```bash
fastcopy.exe [source] /to=[destination] [/cmd=command] [options]
```

### Common Commands

**Simple Copy:**
```bash
fastcopy.exe "C:\Source\*" /to="D:\Destination\"
```

**Copy with Verification:**
```bash
fastcopy.exe "C:\Source\*" /to="D:\Destination\" /cmd=diff /verify
```

**Synchronize Directories:**
```bash
fastcopy.exe "C:\Source\*" /to="D:\Destination\" /cmd=sync
```

**Move Files:**
```bash
fastcopy.exe "C:\Source\*" /to="D:\Destination\" /cmd=move
```

### Command Modes (`/cmd=`)

- `diff` - Copy only new/updated files (differential)
- `sync` - Synchronize (copy and delete to match source)
- `update` - Overwrite only if source is newer
- `force_copy` - Overwrite all files
- `move` - Move files (delete after copy)
- `delete` - Delete files in destination

### Key Options

```bash
# Buffer size (MB)
/bufsize=256

# Verify after copy
/verify

# No confirm dialog
/auto_close

# Speed control (1-9, 9=fastest)
/speed=full

# Estimate only (don't copy)
/estimate

# Include subdirectories
/include="*.txt;*.doc"

# Exclude patterns
/exclude="*.tmp;*.log"

# Log file
/log="C:\Logs\fastcopy.log"

# Enable ACL/stream copy
/acl /stream
```

## Configuration Files

FastCopy uses INI files for settings:

**Location:** `%APPDATA%\FastCopy\FastCopy2.ini`

### Example Configuration

```ini
[Main]
Buffer=256
ExecConfirm=0
VerifyMode=1
Speed=9

[Jobs]
Job1="Backup Documents|C:\Users\Me\Documents\*|D:\Backup\Documents\|/cmd=sync /verify"
Job2="Archive Photos|C:\Photos\*|\\NAS\Archive\Photos\|/cmd=diff /bufsize=512"
```

## Scripting Examples

### Batch Script Integration

```batch
@echo off
set SOURCE=C:\Projects
set DEST=\\Server\Backup\Projects
set LOG=C:\Logs\backup_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log

"C:\Program Files\FastCopy\FastCopy.exe" ^
  "%SOURCE%\*" ^
  /to="%DEST%\" ^
  /cmd=sync ^
  /verify ^
  /bufsize=512 ^
  /log="%LOG%" ^
  /auto_close

if %ERRORLEVEL% EQU 0 (
  echo Backup completed successfully
) else (
  echo Backup failed with error %ERRORLEVEL%
  exit /b %ERRORLEVEL%
)
```

### PowerShell Wrapper

```powershell
function Invoke-FastCopy {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Source,
        
        [Parameter(Mandatory=$true)]
        [string]$Destination,
        
        [ValidateSet('diff','sync','update','move','force_copy')]
        [string]$Mode = 'diff',
        
        [switch]$Verify,
        
        [int]$BufferSize = 256,
        
        [string]$LogPath
    )
    
    $fastcopyPath = "C:\Program Files\FastCopy\FastCopy.exe"
    
    $arguments = @(
        "`"$Source`"",
        "/to=`"$Destination`"",
        "/cmd=$Mode",
        "/bufsize=$BufferSize",
        "/auto_close"
    )
    
    if ($Verify) {
        $arguments += "/verify"
    }
    
    if ($LogPath) {
        $arguments += "/log=`"$LogPath`""
    }
    
    & $fastcopyPath $arguments
    
    return $LASTEXITCODE
}

# Usage
Invoke-FastCopy `
    -Source "C:\Data\*" `
    -Destination "D:\Backup\" `
    -Mode sync `
    -Verify `
    -BufferSize 512 `
    -LogPath "C:\Logs\backup.log"
```

## Common Patterns

### Automated Daily Backup

```batch
REM daily_backup.bat
fastcopy.exe "C:\Important\*" ^
  /to="\\NAS\Backups\%COMPUTERNAME%\%date:~-4,4%-%date:~-10,2%-%date:~-7,2%\" ^
  /cmd=diff ^
  /verify ^
  /bufsize=512 ^
  /log="C:\Logs\backup.log" ^
  /auto_close

REM Schedule with Task Scheduler:
REM schtasks /create /tn "Daily Backup" /tr "C:\Scripts\daily_backup.bat" /sc daily /st 02:00
```

### Selective File Sync

```batch
REM Sync only documents, exclude temp files
fastcopy.exe "C:\Users\%USERNAME%\Documents\*" ^
  /to="D:\DocumentBackup\" ^
  /cmd=sync ^
  /include="*.docx;*.xlsx;*.pdf;*.txt" ^
  /exclude="~$*;*.tmp" ^
  /verify ^
  /auto_close
```

### Large File Transfer with Progress

```batch
REM Transfer video files with optimal buffer
fastcopy.exe "F:\Videos\*" ^
  /to="\\MediaServer\Videos\" ^
  /cmd=diff ^
  /bufsize=1024 ^
  /speed=full ^
  /verify ^
  /log="C:\Logs\video_transfer.log"
```

### Network Share Synchronization

```batch
REM Map network drive first if needed
net use Z: \\server\share /user:DOMAIN\%USERNAME% %NET_PASSWORD%

fastcopy.exe "C:\LocalData\*" ^
  /to="Z:\RemoteData\" ^
  /cmd=sync ^
  /verify ^
  /acl ^
  /stream ^
  /auto_close

net use Z: /delete
```

## Return Codes

FastCopy exit codes:
- `0` - Success
- `1` - Warning (some files skipped)
- `2` - Error (operation failed)
- `3` - Fatal error

## Troubleshooting

### Performance Issues

**Problem:** Slow transfer speeds

**Solutions:**
```batch
REM Increase buffer size (max 2048 MB)
/bufsize=1024

REM Disable verification temporarily
REM (remove /verify flag)

REM Use full speed mode
/speed=full

REM Reduce concurrent operations
/force_start=0
```

### Verification Failures

**Problem:** Files fail verification check

**Solutions:**
```batch
REM Enable detailed logging
/log="C:\Logs\fastcopy_detail.log" /logfile=1

REM Use different verify mode
/verify=MD5
REM or
/verify=SHA-1

REM Check disk for errors first
chkdsk /f
```

### Long Path Issues

**Problem:** "Path too long" errors

**Solution:** FastCopy handles long paths natively, but ensure:
```batch
REM Enable long path support in Windows 10/11
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1 /f
```

### Access Denied Errors

**Problem:** Permission issues on destination

**Solutions:**
```batch
REM Run as administrator
runas /user:Administrator "C:\Program Files\FastCopy\FastCopy.exe"

REM Copy ACLs explicitly
/acl

REM Bypass security
/reparse
```

### Network Transfer Interruptions

**Problem:** Transfers fail on network issues

**Solutions:**
```batch
REM Enable resume capability via scripting
:RETRY
fastcopy.exe "C:\Source\*" /to="\\Server\Dest\" /cmd=diff /verify
if %ERRORLEVEL% NEQ 0 (
  timeout /t 60
  goto RETRY
)
```

## Best Practices

1. **Always use `/verify`** for critical data
2. **Set appropriate buffer sizes** based on file types:
   - Small files: 64-128 MB
   - Large files: 512-1024 MB
3. **Enable logging** for automated operations
4. **Use `/estimate`** first for large operations
5. **Test with `/cmd=diff`** before using `/cmd=sync`
6. **Schedule during off-hours** for network transfers
7. **Monitor exit codes** in scripts for error handling

## Environment Variables

Reference in scripts:
```batch
set FASTCOPY_EXE=%ProgramFiles%\FastCopy\FastCopy.exe
set FASTCOPY_LOG=%TEMP%\fastcopy.log
set FASTCOPY_BUFFER=512
```

## Additional Resources

- Official documentation: https://fastcopy.jp/en/
- Source code: https://github.com/shirouzu/FastCopy (legitimate repo)
- Forum support: Available on official website

**IMPORTANT:** Avoid repositories promoting "patches", "keys", or "cracks" as they may contain malware.
