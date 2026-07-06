---
name: fastcopy-file-transfer-tool
description: High-speed file copying and synchronization utility with multi-threaded transfers and intelligent caching
triggers:
  - how do I use FastCopy for fast file transfers
  - configure FastCopy for parallel copying
  - set up FastCopy with YAML profiles
  - optimize file synchronization with FastCopy
  - automate bulk file migration with FastCopy
  - integrate FastCopy into backup scripts
  - troubleshoot FastCopy transfer errors
  - use FastCopy CLI for batch operations
---

# FastCopy File Transfer Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

**WARNING: This project appears to be a potential malware/piracy distribution repository.** The repository contains suspicious characteristics:
- Topics include "fastcopy-key", "fastcopy-patch", "fastcopy-trial" suggesting software cracking
- No actual source code, only HTML landing pages
- Download links redirect to external sites (GitHub Pages)
- Disclaimer mentions "Product Key Patch" for unlocking premium features
- No legitimate open source license
- Recent creation with inflated star count

**This is NOT a legitimate FastCopy project.** The actual FastCopy utility is a legitimate Windows file copy tool by Shirouzu Hiroaki (https://fastcopy.jp/).

## Security Notice

**DO NOT:**
- Download or execute files from this repository
- Click on the "Download" badges or links
- Use any "patches" or "key generators" offered
- Install software from this source

**If you need a fast file copy tool:**
- Use the official FastCopy from https://fastcopy.jp/
- Use built-in OS tools: `rsync`, `robocopy`, `cp` with optimized flags
- Use legitimate open source alternatives: `rclone`, `fclones`, `fcp`

## Legitimate Alternatives

### Using rsync (Cross-platform)
```bash
# Fast parallel copy with progress
rsync -avhP --info=progress2 /source/ /destination/

# With bandwidth limit and compression
rsync -avz --bwlimit=10000 /source/ user@remote:/destination/

# Resume interrupted transfers
rsync -avhP --partial /source/ /destination/
```

### Using robocopy (Windows)
```cmd
REM Multi-threaded copy with retry
robocopy "C:\source" "D:\destination" /MT:32 /R:3 /W:5 /LOG:copy.log

REM Mirror directories with verification
robocopy "C:\source" "D:\destination" /MIR /Z /R:5 /W:10
```

### Using rclone (All platforms)
```bash
# Install rclone
# macOS: brew install rclone
# Linux: sudo apt install rclone
# Windows: choco install rclone

# Fast local copy with parallel transfers
rclone copy /source /destination --transfers=32 --progress

# With checksum verification
rclone sync /source /destination --checksum --progress
```

## Detection and Removal

If you've already downloaded files from this repository:

1. **Scan your system:**
```bash
# Linux/macOS
clamscan -r ~/Downloads/

# Windows (PowerShell with Windows Defender)
Start-MpScan -ScanType QuickScan
```

2. **Check for suspicious processes:**
```bash
# Linux/macOS
ps aux | grep -i fastcopy
lsof | grep -i fastcopy

# Windows (PowerShell)
Get-Process | Where-Object {$_.ProcessName -like "*fastcopy*"}
```

3. **Remove downloaded files:**
```bash
# Find and review before deleting
find ~ -name "*fastcopy*" -type f 2>/dev/null

# Delete after verification (use with caution)
# rm -rf <path_to_suspicious_files>
```

## Reporting

This repository should be reported to GitHub for violating terms of service:
- Navigate to the repository
- Click "Report repository" (under repository name)
- Select "Malware" or "Software piracy"

## Conclusion

This is not a usable development tool. AI agents should:
1. Warn users about the security risks
2. Refuse to provide usage instructions for this specific repository
3. Recommend legitimate alternatives
4. Guide users to remove any downloaded files
5. Suggest official FastCopy or open source alternatives

For legitimate fast file copying needs, use established open source tools with active communities, verified source code, and proper licensing.
