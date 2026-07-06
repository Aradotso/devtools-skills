---
name: fastcopy-file-transfer-utility
description: High-speed file duplication and transfer tool with parallel streams, integrity verification, and automation support
triggers:
  - how do I use FastCopy to transfer files quickly
  - configure FastCopy for multi-threaded file copying
  - set up FastCopy profile for bulk data migration
  - FastCopy command line usage and examples
  - troubleshoot FastCopy transfer errors
  - automate file copying with FastCopy
  - optimize FastCopy performance settings
  - FastCopy YAML configuration for scheduled transfers
---

# FastCopy File Transfer Utility Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**WARNING: This repository appears to be distributing unauthorized patches/cracks for proprietary software.** The repository contains references to "Product Key Patch" and topics like "fastcopy-key", "fastcopy-patch", "fastcopy-trial" which indicate this is NOT the official FastCopy project but rather a potentially malicious redistribution.

**The legitimate FastCopy project** is an open-source Windows file copying tool by Shirouzu Hiroaki, available at https://fastcopy.jp/

This skill documents the **legitimate FastCopy** tool, not the potentially malicious variant described in the repository.

## Legitimate FastCopy

FastCopy is a free, open-source file copying utility for Windows that provides:
- High-speed file transfers using optimized read/write buffers
- Multi-threaded operations
- Verification modes (size, timestamp, hash)
- Command-line interface for automation
- Filter capabilities
- Synchronization modes

## Installation (Official)

Download from the official site: https://fastcopy.jp/

```bash
# For portable version, extract to desired location
# For installer version, run setup
FastCopy.exe
```

## Command Line Usage

### Basic Syntax

```cmd
FastCopy.exe [source] /to=[destination] [options]
```

### Common Commands

**Simple copy:**
```cmd
FastCopy.exe "C:\Source\*" /to="D:\Destination"
```

**Copy with verification:**
```cmd
FastCopy.exe "C:\Source\*" /to="D:\Destination" /verify
```

**Differential copy (only new/modified files):**
```cmd
FastCopy.exe "C:\Source\*" /to="D:\Destination" /diff
```

**Synchronize (mirror) directories:**
```cmd
FastCopy.exe "C:\Source\*" /to="D:\Destination" /sync
```

**Move files instead of copy:**
```cmd
FastCopy.exe "C:\Source\*" /to="D:\Destination" /move
```

### Key Options

| Option | Description |
|--------|-------------|
| `/cmd=copy` | Normal copy (default) |
| `/cmd=diff` | Copy only new/updated files |
| `/cmd=sync` | Synchronize (delete extra files in destination) |
| `/cmd=move` | Move files |
| `/verify` | Verify file integrity after copy |
| `/estimate` | Show estimated time before starting |
| `/log` | Create log file |
| `/logfile="path"` | Specify log file location |
| `/bufsize=N` | Buffer size in MB (default auto) |
| `/speed=full` or `/speed=suspend` | Speed control |
| `/force_close` | Force close files locked by other processes |
| `/linkdest` | Copy symbolic link destinations |
| `/acl` | Copy ACL/permissions |
| `/stream` | Copy NTFS alternate streams |

### Include/Exclude Filters

```cmd
# Include only specific file types
FastCopy.exe "C:\Source\*" /to="D:\Backup" /include="*.jpg;*.png;*.gif"

# Exclude specific patterns
FastCopy.exe "C:\Source\*" /to="D:\Backup" /exclude="*.tmp;*.log;temp\*"
```

## Configuration Files

FastCopy can save job configurations for repeated use.

### Creating a Job (GUI)

1. Configure source, destination, options in GUI
2. Click "Add Job"
3. Save job list to file

### Using Job Files

```cmd
# Run specific job from job list
FastCopy.exe /job="BackupJob"

# Use job file
FastCopy.exe /jobfile="C:\jobs\backup.job"
```

## Real-World Examples

### Daily Backup Script

```batch
@echo off
REM daily-backup.bat
set SOURCE=C:\Projects
set DEST=D:\Backups\Projects_%date:~-4,4%%date:~-10,2%%date:~-7,2%
set LOGDIR=C:\Logs

"C:\Program Files\FastCopy\FastCopy.exe" ^
  "%SOURCE%\*" ^
  /to="%DEST%" ^
  /cmd=diff ^
  /verify ^
  /logfile="%LOGDIR%\backup_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log" ^
  /estimate ^
  /force_close ^
  /exclude="*.tmp;*.cache;node_modules\*;.git\*"

if %ERRORLEVEL% EQU 0 (
    echo Backup completed successfully
) else (
    echo Backup failed with error code %ERRORLEVEL%
)
```

### Synchronize Development Environments

```batch
REM sync-dev.bat
set LOCAL=C:\Dev\MyProject
set REMOTE=\\FileServer\DevShare\MyProject

FastCopy.exe "%LOCAL%\*" /to="%REMOTE%" ^
  /cmd=sync ^
  /verify ^
  /exclude=".git\*;bin\*;obj\*;*.suo;*.user" ^
  /bufsize=256 ^
  /log
```

### Large Media File Migration

```batch
REM migrate-media.bat
FastCopy.exe "E:\Media\*" /to="\\NAS\Archive\Media" ^
  /cmd=move ^
  /verify ^
  /include="*.mp4;*.mkv;*.avi;*.mov" ^
  /bufsize=512 ^
  /estimate ^
  /force_close ^
  /logfile="C:\Logs\media-migration.log"
```

## Automation Patterns

### PowerShell Integration

```powershell
# backup-automation.ps1
$fastcopyPath = "C:\Program Files\FastCopy\FastCopy.exe"
$source = "C:\Data"
$destination = "D:\Backups\Data_$(Get-Date -Format 'yyyyMMdd')"
$logPath = "C:\Logs\backup_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

$arguments = @(
    "`"$source\*`"",
    "/to=`"$destination`"",
    "/cmd=diff",
    "/verify",
    "/logfile=`"$logPath`"",
    "/force_close"
)

$process = Start-Process -FilePath $fastcopyPath -ArgumentList $arguments -Wait -PassThru

if ($process.ExitCode -eq 0) {
    Write-Output "Backup successful"
    # Send notification, update database, etc.
} else {
    Write-Error "Backup failed with exit code $($process.ExitCode)"
    # Send alert
}
```

### Task Scheduler Integration

```batch
REM Create scheduled task
schtasks /create /tn "Daily Backup" /tr "C:\Scripts\daily-backup.bat" /sc daily /st 02:00
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | User cancel |
| 2 | Verification error |
| 3 | Source/Destination error |
| 4 | Other error |

## Troubleshooting

### Permission Errors

```batch
REM Run as administrator or use force_close
FastCopy.exe "C:\Source\*" /to="D:\Dest" /force_close /acl
```

### Verification Failures

```batch
REM Use different verification method
FastCopy.exe "C:\Source\*" /to="D:\Dest" /verify /verify_type=SHA-1
```

### Performance Issues

```batch
REM Adjust buffer size (try 256, 512, or 1024 MB)
FastCopy.exe "C:\Source\*" /to="D:\Dest" /bufsize=512

REM Reduce speed if needed
FastCopy.exe "C:\Source\*" /to="D:\Dest" /speed=auto_slow
```

### Network Transfer Optimization

```batch
REM Increase buffer for network copies
FastCopy.exe "C:\Local\*" /to="\\Server\Share" ^
  /bufsize=1024 ^
  /force_close ^
  /verify
```

## Best Practices

1. **Always use `/verify` for critical data** - ensures data integrity
2. **Use `/estimate` for large transfers** - preview time/size before starting
3. **Enable logging** - helpful for auditing and troubleshooting
4. **Use filters wisely** - exclude temporary/cache files to speed up transfers
5. **Test jobs before automation** - run manually first to verify settings
6. **Monitor exit codes** - implement proper error handling in scripts
7. **Use `/force_close` carefully** - only when necessary for locked files

## Environment Variables

Reference environment variables in scripts:

```batch
FastCopy.exe "%USERPROFILE%\Documents\*" /to="%BACKUP_LOCATION%" /cmd=diff
```

## Security Considerations

**IMPORTANT:** The repository linked in the prompt (`Raunak64-bit/FastCopy-Clipper-Portable-Utility`) appears to be distributing unauthorized patches or cracks. Always:

- Download FastCopy only from the official site: https://fastcopy.jp/
- Never use "Product Key Patches" or cracks from untrusted sources
- Verify file integrity from official checksums
- Be aware that malicious actors often distribute malware disguised as "patched" versions of legitimate software

The legitimate FastCopy is completely free and open source - there is no need for any patches or activation keys.
