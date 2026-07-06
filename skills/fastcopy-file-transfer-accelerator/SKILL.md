---
name: fastcopy-file-transfer-accelerator
description: High-performance multi-threaded file copying and synchronization utility for Windows with advanced caching and verification
triggers:
  - how do I use fastcopy to copy files faster
  - configure fastcopy for maximum transfer speed
  - set up fastcopy with multiple streams
  - fastcopy command line options
  - create a fastcopy transfer profile
  - fastcopy verification and resume settings
  - optimize fastcopy buffer size
  - automate file copying with fastcopy
---

# FastCopy File Transfer Accelerator

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

FastCopy is a high-performance file copying and synchronization utility for Windows that uses multi-threaded I/O to maximize transfer speeds. It supports parallel streams, integrity verification, resume capabilities, and intelligent buffer management.

## Installation

**Windows (Recommended):**
```powershell
# Download from official source
# Visit: https://fastcopy.jp/en/

# Or using winget
winget install FastCopy

# Portable version - extract to desired location
Expand-Archive fastcopy-portable.zip -DestinationPath C:\Tools\FastCopy
```

**Add to PATH (optional):**
```powershell
$env:PATH += ";C:\Program Files\FastCopy"
# Or for portable:
$env:PATH += ";C:\Tools\FastCopy"
```

## Core Commands

### Basic Syntax
```cmd
fastcopy.exe [source] [destination] [options]
```

### Common Operations

**Simple copy:**
```cmd
fastcopy "C:\Source\Files" "D:\Backup\" /cmd=diff
```

**Mirror synchronization:**
```cmd
fastcopy "C:\Projects" "E:\Archive\Projects" /cmd=sync /force_close
```

**Move files:**
```cmd
fastcopy "C:\Temp\Downloads" "D:\Storage" /cmd=move
```

**Verify after copy:**
```cmd
fastcopy "C:\Data" "\\NAS\Backup" /cmd=diff /verify
```

## Key Command Options

| Option | Description | Example |
|--------|-------------|---------|
| `/cmd=` | Copy mode: `diff`, `sync`, `move`, `delete` | `/cmd=sync` |
| `/bufsize=` | Buffer size in MB (default: auto) | `/bufsize=256` |
| `/verify` | Verify files after transfer | `/verify` |
| `/linkdest` | Create hardlinks instead of copy | `/linkdest` |
| `/exclude=` | Exclude patterns | `/exclude="*.tmp;*.log"` |
| `/include=` | Include only patterns | `/include="*.jpg;*.png"` |
| `/force_close` | Force close open files | `/force_close` |
| `/log` | Enable logging | `/log` |
| `/logfile=` | Specify log file path | `/logfile="C:\Logs\copy.log"` |
| `/error_stop` | Stop on first error | `/error_stop` |
| `/acl` | Copy ACL/permissions | `/acl` |
| `/stream` | Copy alternate data streams | `/stream` |

## Configuration Profiles

FastCopy uses `.ini` files for configuration. Create reusable profiles:

**Example: High-Speed Transfer Profile**
```ini
[main]
bufsize=512
estimate=1
verify=1
wipe_del=0
acl=0
stream=0
reparse=0
verify_mode=0
maxhist=20

[SyncProfile]
source=C:\Source
dest=D:\Destination
command=sync
bufsize=256
verify=1
filter_mode=0
exclude=*.tmp;*.log;.git
```

**Load profile from command line:**
```cmd
fastcopy /cmd=sync /ini="C:\FastCopy\profiles\backup.ini"
```

## Advanced Usage Patterns

### Automated Backup Script
```batch
@echo off
REM backup-script.bat
SET FASTCOPY="C:\Program Files\FastCopy\FastCopy.exe"
SET SOURCE=C:\Important\Data
SET DEST=E:\Backups\%DATE:~-4,4%%DATE:~-10,2%%DATE:~-7,2%
SET LOGFILE=C:\Logs\backup-%DATE:~-4,4%%DATE:~-10,2%%DATE:~-7,2%.log

REM Create timestamped backup with verification
%FASTCOPY% "%SOURCE%" "%DEST%" ^
  /cmd=diff ^
  /bufsize=256 ^
  /verify ^
  /acl ^
  /stream ^
  /error_stop ^
  /log ^
  /logfile="%LOGFILE%" ^
  /exclude="*.tmp;*.cache;thumbs.db" ^
  /force_close

if %ERRORLEVEL% EQU 0 (
    echo Backup completed successfully
) else (
    echo Backup failed with error %ERRORLEVEL%
    exit /b %ERRORLEVEL%
)
```

### PowerShell Integration
```powershell
# fastcopy-wrapper.ps1
function Invoke-FastCopy {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Source,
        
        [Parameter(Mandatory=$true)]
        [string]$Destination,
        
        [ValidateSet('diff', 'sync', 'move', 'delete')]
        [string]$Mode = 'diff',
        
        [int]$BufferSize = 256,
        
        [switch]$Verify,
        
        [string[]]$Exclude = @(),
        
        [string]$LogPath
    )
    
    $fastcopyPath = "C:\Program Files\FastCopy\FastCopy.exe"
    
    $arguments = @(
        "`"$Source`"",
        "`"$Destination`"",
        "/cmd=$Mode",
        "/bufsize=$BufferSize"
    )
    
    if ($Verify) {
        $arguments += "/verify"
    }
    
    if ($Exclude.Count -gt 0) {
        $excludePattern = $Exclude -join ";"
        $arguments += "/exclude=`"$excludePattern`""
    }
    
    if ($LogPath) {
        $arguments += "/log"
        $arguments += "/logfile=`"$LogPath`""
    }
    
    $arguments += "/force_close"
    
    Write-Host "Starting FastCopy: $($arguments -join ' ')"
    
    & $fastcopyPath $arguments
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "Transfer completed successfully" -ForegroundColor Green
        return $true
    } else {
        Write-Host "Transfer failed with exit code: $LASTEXITCODE" -ForegroundColor Red
        return $false
    }
}

# Usage example
Invoke-FastCopy `
    -Source "C:\Projects" `
    -Destination "\\NAS\Backup\Projects" `
    -Mode sync `
    -BufferSize 512 `
    -Verify `
    -Exclude @("*.tmp", "*.log", "node_modules", ".git") `
    -LogPath "C:\Logs\project-sync.log"
```

### Network Transfer Optimization
```cmd
REM For network transfers (SMB/CIFS), optimize buffer and streams
fastcopy "C:\LocalData" "\\RemoteServer\Share\Data" ^
  /cmd=sync ^
  /bufsize=512 ^
  /verify ^
  /no_ui ^
  /force_close ^
  /acl ^
  /log

REM For very large files (video, databases), increase buffer
fastcopy "C:\Videos" "D:\Archive" ^
  /cmd=diff ^
  /bufsize=1024 ^
  /verify
```

## Scheduled Task Integration

**Create scheduled backup (PowerShell):**
```powershell
$action = New-ScheduledTaskAction `
    -Execute "C:\Program Files\FastCopy\FastCopy.exe" `
    -Argument '"C:\Data" "E:\Backup" /cmd=sync /verify /log /force_close'

$trigger = New-ScheduledTaskTrigger -Daily -At 2am

$principal = New-ScheduledTaskPrincipal `
    -UserId "SYSTEM" `
    -LogonType ServiceAccount `
    -RunLevel Highest

Register-ScheduledTask `
    -TaskName "FastCopy Daily Backup" `
    -Action $action `
    -Trigger $trigger `
    -Principal $principal `
    -Description "Automated daily backup using FastCopy"
```

## Performance Tuning

### Buffer Size Selection
```cmd
REM Small files (< 100MB total): 32-64MB buffer
fastcopy "C:\Documents" "D:\Backup" /cmd=diff /bufsize=64

REM Medium files (100MB-10GB): 128-256MB buffer
fastcopy "C:\Projects" "E:\Archive" /cmd=sync /bufsize=256

REM Large files (> 10GB): 512-1024MB buffer
fastcopy "C:\Videos" "F:\Media" /cmd=diff /bufsize=1024

REM Network transfers: 256-512MB buffer
fastcopy "C:\Data" "\\NAS\Share" /cmd=sync /bufsize=512
```

### Exclude Common Temporary Files
```cmd
fastcopy "C:\Development" "D:\Backup\Dev" ^
  /cmd=sync ^
  /exclude="*.tmp;*.log;*.cache;node_modules;.git;.svn;bin;obj;target;__pycache__;.vs;.vscode;*.swp;*.swo;Thumbs.db;.DS_Store"
```

## Common Patterns

### Mirror Backup (Delete Removed Files)
```cmd
REM Sync mode mirrors source to destination (deletes extra files)
fastcopy "C:\ActiveProject" "E:\Backups\ActiveProject" ^
  /cmd=sync ^
  /verify ^
  /log
```

### Incremental Backup (Keep Extra Files)
```cmd
REM Diff mode only copies new/changed files (keeps extra files)
fastcopy "C:\ActiveProject" "E:\Backups\ActiveProject" ^
  /cmd=diff ^
  /verify ^
  /log
```

### Safe Move with Verification
```cmd
fastcopy "C:\Completed\Archive" "D:\LongTerm\Archive" ^
  /cmd=move ^
  /verify ^
  /error_stop ^
  /log
```

### Hardlink Deduplication
```cmd
REM Create hardlinks for duplicate files to save space
fastcopy "C:\Media" "D:\Deduplicated" ^
  /cmd=diff ^
  /linkdest ^
  /verify
```

## Troubleshooting

### Access Denied Errors
```cmd
REM Run with elevated privileges and force close open files
fastcopy "C:\LockedFiles" "D:\Backup" ^
  /cmd=diff ^
  /force_close ^
  /acl

REM Skip files that can't be accessed
fastcopy "C:\System" "D:\Backup" ^
  /cmd=diff ^
  /skip_empty_dir ^
  /verify
```

### Network Path Issues
```cmd
REM Use UNC paths explicitly
fastcopy "\\Server1\Share\Data" "\\Server2\Backup\Data" ^
  /cmd=sync ^
  /bufsize=256 ^
  /force_close

REM Map network drive first if UNC fails
net use Z: \\Server\Share /user:DOMAIN\Username
fastcopy "C:\Local" "Z:\Backup" /cmd=sync
net use Z: /delete
```

### Verify Failures
```cmd
REM Check disk space and permissions
fastcopy "C:\Source" "D:\Dest" ^
  /cmd=diff ^
  /verify ^
  /log ^
  /logfile="C:\Logs\verify-debug.log"

REM Review log for specific verification errors
```

### Performance Issues
```cmd
REM Reduce buffer size if running out of memory
fastcopy "C:\Data" "D:\Backup" /cmd=diff /bufsize=64

REM Disable verification for initial bulk transfer
fastcopy "C:\Massive" "E:\Archive" /cmd=diff /bufsize=512

REM Then verify separately
fastcopy "C:\Massive" "E:\Archive" /cmd=diff /verify /open_window
```

## Exit Codes

- `0` - Success
- `1` - Error occurred during copy
- `2` - Fatal error (unable to continue)
- `3` - User cancelled operation

**Handle exit codes in scripts:**
```batch
@echo off
fastcopy "C:\Source" "D:\Dest" /cmd=sync /verify

if %ERRORLEVEL% EQU 0 (
    echo Success
    exit /b 0
) else if %ERRORLEVEL% EQU 1 (
    echo Partial failure - check logs
    exit /b 1
) else (
    echo Fatal error
    exit /b 2
)
```

## Environment Variables

```powershell
# Set default FastCopy path
$env:FASTCOPY_PATH = "C:\Program Files\FastCopy\FastCopy.exe"

# Set default log directory
$env:FASTCOPY_LOGS = "C:\Logs\FastCopy"
```

Use in scripts:
```batch
%FASTCOPY_PATH% "C:\Source" "D:\Dest" /log /logfile="%FASTCOPY_LOGS%\copy.log"
```
