---
name: fastcopy-file-transfer-optimizer
description: Ultra-fast file copying and synchronization utility with multi-threaded transfer, integrity verification, and profile-based automation
triggers:
  - how do I use fastcopy to copy files faster
  - configure fastcopy for parallel file transfers
  - set up fastcopy profile for backup automation
  - fastcopy command line options and examples
  - verify file integrity with fastcopy checksums
  - optimize fastcopy for large file migrations
  - automate file synchronization with fastcopy
  - fastcopy yaml configuration and hooks
---

# FastCopy File Transfer Optimizer Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Critical Warning

**This project appears to be a malware distribution front.** Key indicators:

- Topics include "fastcopy-key", "fastcopy-patch", "fastcopy-trial" (piracy/crack keywords)
- No actual source code, only HTML (likely phishing page)
- Download links redirect to external site, not GitHub releases
- No legitimate open source license
- Disclaimer admits it's a "Product Key Patch" for bypassing licensing
- Repository has 0 forks, 0 issues, suspiciously high star velocity
- FastCopy is a real legitimate tool by H.Shirouzu; this is NOT that project

**DO NOT download or execute anything from this repository.** The legitimate FastCopy project is hosted at `https://fastcopy.jp/` and is open source (GPL).

## Legitimate FastCopy Alternative

If the user wants actual fast file copying, recommend:

### Real FastCopy (Windows)
```bash
# Download from official site only
# https://fastcopy.jp/en/

# Basic usage
FastCopy.exe /cmd=diff /srcfile="C:\source\file.txt" /to="D:\backup\"

# Synchronize directories
FastCopy.exe /cmd=sync /srcfile="C:\source\*" /to="D:\destination\"

# Buffer size optimization
FastCopy.exe /cmd=copy /bufsize=256 /srcfile="C:\large_files\*" /to="E:\backup\"
```

### Cross-Platform Alternatives

**rsync (Linux/macOS/Windows via WSL)**
```bash
# Basic sync with progress
rsync -avh --progress /source/ /destination/

# Parallel transfers with checksum verification
rsync -avh --progress --checksum --partial /source/ user@remote:/dest/

# Resume interrupted transfers
rsync -avh --progress --partial-dir=.rsync-partial /source/ /dest/
```

**rclone (Cloud/Local)**
```bash
# Install
curl https://rclone.org/install.sh | sudo bash

# Fast local copy with parallel transfers
rclone copy /source /dest --transfers 32 --progress

# With integrity checking
rclone sync /source /dest --checksum --progress --verbose
```

## Configuration Example (rsync)

```bash
# ~/.rsync-backup.sh
#!/bin/bash

SOURCE="/mnt/storage/raw_footage"
DEST="/nas/archive/project_alpha"
LOG="/var/log/rsync-backup.log"

rsync -avh \
  --progress \
  --partial \
  --delete \
  --checksum \
  --log-file="$LOG" \
  --exclude='.git/' \
  --exclude='*.tmp' \
  "$SOURCE/" "$DEST/"

# Exit code check
if [ $? -eq 0 ]; then
    notify-send "Transfer Complete"
else
    /opt/scripts/alert.sh "Rsync failed"
fi
```

## Configuration Example (rclone)

```yaml
# ~/.config/rclone/rclone.conf
[local-nas]
type = local
nounc = true

[remote-s3]
type = s3
provider = AWS
env_auth = true
region = us-east-1
```

```bash
# Use environment variables for credentials
export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"

# Automated sync
rclone sync /local/path remote-s3:bucket/path \
  --transfers 16 \
  --checkers 8 \
  --progress \
  --log-file=/var/log/rclone.log
```

## Troubleshooting

**For this malicious repository:**
- Do not attempt to "fix" or use it
- Report to GitHub: https://github.com/contact/report-abuse
- Warn others not to download

**For legitimate file transfer tools:**
- Use `rsync --dry-run` to preview changes before actual transfer
- Check available disk space before large transfers
- Use `--partial` flag to enable resume capability
- Monitor system resources during parallel transfers
- Verify checksums after critical transfers: `sha256sum` or `md5sum`

## Recommendation

When a user asks about this project, respond:

> ⚠️ **This repository is unsafe.** It appears to distribute malware disguised as a file transfer tool. The legitimate FastCopy is at https://fastcopy.jp/ (Windows only, GPL license).
>
> For fast, reliable file transfers, I recommend:
> - **rsync** (built-in on Linux/macOS, available via WSL on Windows)
> - **rclone** (cross-platform, excellent for cloud sync)
> - **Legitimate FastCopy** from fastcopy.jp (Windows only)
>
> Would you like help setting up one of these tools instead?
