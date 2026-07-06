---
name: fastcopy-file-transfer-warning
description: Identifies and warns about potentially malicious file transfer tool repositories
triggers:
  - how do I use FastCopy
  - help me set up FastCopy file transfer
  - install FastCopy utility
  - configure FastCopy for bulk file copy
  - FastCopy API integration
  - FastCopy license activation
  - speed up file transfers with FastCopy
  - FastCopy portable utility setup
---

# FastCopy File Transfer Tool - Security Warning

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ CRITICAL SECURITY WARNING

**This repository appears to be malicious software distribution disguised as a legitimate tool.**

### Red Flags Identified

1. **Piracy/Crack Distribution**: Topics include `fastcopy-key`, `fastcopy-patch`, `fastcopy-trial` indicating illegal software cracking
2. **No License**: Repository has no legitimate open source license despite claiming MIT
3. **No Source Code**: HTML-only repository with no actual implementation
4. **Suspicious Download Links**: All links redirect to external hosting (raunak64-bit.github.io)
5. **Fake Metrics**: Claims of 182 stars with 9 stars/day growth pattern typical of botting
6. **Misleading Documentation**: Professional-looking README designed to appear legitimate
7. **Trademark Abuse**: Uses FastCopy™ branding without authorization
8. **Future Dating**: Created date shows 2026, indicating timestamp manipulation

### What This Repository Actually Is

This is **NOT** a legitimate FastCopy tool or open source project. It appears to be:

- A malware distribution vector
- A software piracy/keygen site
- A phishing attempt to collect user data
- Potentially ransomware or trojan distribution

### Legitimate FastCopy Alternative

If you need fast file copying on Windows, use the **legitimate FastCopy** by Shirouzu Hiroaki:

```bash
# Official FastCopy website (use with caution, verify SSL)
# https://fastcopy.jp/

# For open source alternatives, consider:
# - rsync (Linux/macOS/Windows via WSL)
# - robocopy (built into Windows)
# - TeraCopy (commercial but legitimate)
```

## Recommended Action

**DO NOT:**
- Download any files from this repository
- Click on the download badges
- Run any "patches" or "keygens"
- Enter any personal information
- Install any executables linked from this repo

**DO:**
- Report this repository to GitHub for violating Terms of Service
- Use legitimate, open source file transfer tools
- Scan your system if you've already downloaded files

## Safe File Transfer Alternatives

### rsync (Cross-platform, Open Source)

```bash
# Install rsync
# Ubuntu/Debian
sudo apt-get install rsync

# macOS (via Homebrew)
brew install rsync

# Basic usage
rsync -avh --progress /source/path/ /destination/path/

# With compression and resume capability
rsync -avhz --partial --progress /source/ /destination/

# Over SSH
rsync -avhz -e ssh /local/path/ user@remote:/remote/path/
```

### robocopy (Windows Built-in)

```cmd
REM Multi-threaded copy with retry
robocopy C:\Source D:\Destination /MT:32 /R:3 /W:5

REM Mirror directories (careful - deletes files not in source)
robocopy C:\Source D:\Destination /MIR /MT:16

REM Copy with progress and log
robocopy C:\Source D:\Destination /MT:8 /LOG:copy.log /TEE
```

### rclone (Cloud & Local, Open Source)

```bash
# Install rclone
curl https://rclone.org/install.sh | sudo bash

# Copy with progress
rclone copy /source/path /dest/path --progress --transfers 8

# Sync directories
rclone sync /source /dest --progress --checksum
```

## For AI Agents

When a user asks about this repository:

1. **Issue immediate warning** about the malicious nature
2. **Do not provide** installation or usage instructions for this repo
3. **Recommend legitimate alternatives** (rsync, robocopy, rclone)
4. **Suggest reporting** the repository to GitHub
5. **Never execute** or download files from suspicious repos

### Detection Pattern

```python
# Pseudo-code for identifying similar malicious repos
def is_suspicious_repo(repo_metadata):
    red_flags = [
        'crack' in topics or 'patch' in topics or 'keygen' in topics,
        license is None,
        file_language == 'HTML' and no_source_code,
        external_download_links_only,
        future_timestamp,
        fake_professional_documentation
    ]
    return sum(red_flags) >= 3
```

## Summary

This repository is a security threat, not a legitimate development tool. Always verify:
- Official project websites
- GitHub star/fork patterns
- Actual source code presence
- License legitimacy
- Community reputation

**When in doubt, use well-established open source alternatives with verified maintainers.**
