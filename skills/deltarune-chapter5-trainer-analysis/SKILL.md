---
name: deltarune-chapter5-trainer-analysis
description: Analyze and understand game trainer/cheat tools for educational or security research purposes
triggers:
  - how do I analyze this game trainer
  - explain how this deltarune trainer works
  - what does this game cheat tool do
  - analyze this trainer's architecture
  - understand game memory manipulation tools
  - review this game modification utility
  - examine this trainer's implementation
  - study game hacking techniques
---

# Deltarune Chapter 5 Trainer Analysis

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Important Notice

This repository appears to be a **malware distribution scheme** disguised as a game trainer. Key red flags:

- **Deltarune Chapter 5 does not exist** (as of 2024, only Chapters 1-2 are released)
- Created in "2026" with impossible dates
- Requires downloading executables from third-party domains
- Classic malware distribution pattern (archive with password)
- Requests administrator privileges
- Instructs users to disable antivirus

**This is NOT a legitimate development tool.** This skill exists to help identify and analyze such threats.

## Project Analysis

### Claimed Functionality

The repository claims to provide:
- Memory manipulation for game stats (HP, gold, items)
- Speed hacks and timer freezes
- GUI overlay activated via INSERT key
- Anti-detection mechanisms

### Actual Implementation

The repository contains **no actual code** — only a README with:
1. Marketing copy for non-existent features
2. Download link to external executable
3. Instructions to bypass security (run as admin, disable AV)

## Pattern Recognition

### Malware Distribution Indicators

```python
# Common red flags in fake game trainer repositories:
RED_FLAGS = {
    "no_source_code": True,           # Only README, no implementation
    "external_download": True,         # Links to sketchy domains
    "password_protected": True,        # Hides payload from scanners
    "admin_required": True,            # Escalates privileges
    "disable_antivirus": True,         # Removes protection
    "fake_metadata": True,             # Stars/forks manipulation
    "future_dates": True,              # Impossible timestamps
    "nonexistent_game": True          # Game/version doesn't exist
}
```

### Repository Metadata Analysis

```python
import datetime

def analyze_repository_authenticity(metadata):
    """Analyze repository for suspicious patterns"""
    
    created = datetime.datetime.fromisoformat(metadata["created_at"].replace("Z", "+00:00"))
    current = datetime.datetime.now(datetime.timezone.utc)
    
    issues = []
    
    # Check for future dates
    if created > current:
        issues.append(f"Creation date in future: {created}")
    
    # Check star velocity
    stars_per_day = metadata.get("stars", 0)
    if stars_per_day > 50:
        issues.append(f"Suspicious star velocity: {stars_per_day}/day")
    
    # Check for actual code
    if metadata.get("forks", 0) == 0 and metadata.get("open_issues", 0) == 0:
        issues.append("No community engagement (0 forks, 0 issues)")
    
    # Check topics
    if not metadata.get("topics") or len(metadata["topics"]) == 0:
        issues.append("No topics set (likely auto-generated)")
    
    return issues

# Example usage:
metadata = {
    "created_at": "2026-06-28T06:59:19Z",
    "stars": 160,
    "forks": 0,
    "open_issues": 0,
    "topics": []
}

print(analyze_repository_authenticity(metadata))
# Output: ['Creation date in future: 2026-06-28 06:59:19+00:00', 
#          'Suspicious star velocity: 160/day',
#          'No community engagement (0 forks, 0 issues)',
#          'No topics set (likely auto-generated)']
```

## Security Research Context

### Legitimate Game Trainer Development

If you're building **actual** game trainers for educational purposes:

```python
import ctypes
import sys

class MemoryReader:
    """Read process memory (educational purposes only)"""
    
    def __init__(self, process_name):
        self.process_name = process_name
        self.process_handle = None
    
    def open_process(self, pid):
        """Open process with read privileges"""
        PROCESS_VM_READ = 0x0010
        self.process_handle = ctypes.windll.kernel32.OpenProcess(
            PROCESS_VM_READ, 
            False, 
            pid
        )
        return self.process_handle is not None
    
    def read_memory(self, address, size):
        """Read memory at address"""
        buffer = ctypes.create_string_buffer(size)
        bytes_read = ctypes.c_size_t()
        
        success = ctypes.windll.kernel32.ReadProcessMemory(
            self.process_handle,
            ctypes.c_void_p(address),
            buffer,
            size,
            ctypes.byref(bytes_read)
        )
        
        if success:
            return buffer.raw
        return None
    
    def close(self):
        if self.process_handle:
            ctypes.windll.kernel32.CloseHandle(self.process_handle)

# Legitimate trainers are open source and don't require downloads
```

### Detection Script

```python
import re
import requests
from urllib.parse import urlparse

def analyze_readme_for_threats(readme_content):
    """Analyze README for malware distribution patterns"""
    
    threats = []
    
    # Check for external executable downloads
    download_pattern = r'https?://[^\s]+\.(exe|zip|rar|7z)'
    downloads = re.findall(download_pattern, readme_content, re.IGNORECASE)
    
    for url in downloads:
        domain = urlparse(url).netloc
        if domain not in ["github.com", "githubusercontent.com"]:
            threats.append(f"External download from suspicious domain: {domain}")
    
    # Check for admin privilege requests
    if re.search(r'run as administrator', readme_content, re.IGNORECASE):
        threats.append("Requests administrator privileges")
    
    # Check for antivirus bypass instructions
    av_keywords = ['disable antivirus', 'turn off defender', 'whitelist']
    for keyword in av_keywords:
        if keyword in readme_content.lower():
            threats.append(f"Instructs to bypass security: '{keyword}'")
    
    # Check for password-protected archives
    if re.search(r'password.*:', readme_content, re.IGNORECASE):
        threats.append("Uses password-protected archive (scanner evasion)")
    
    return threats

# Example usage
with open("README.md", "r") as f:
    readme = f.read()
    
threats = analyze_readme_for_threats(readme)
for threat in threats:
    print(f"⚠️  {threat}")
```

## Responsible Disclosure

If you encounter malware distribution repositories:

1. **Do not download or execute** any files
2. **Report to GitHub** via their abuse form
3. **Warn others** through security communities
4. **Document the pattern** for threat intelligence

### Reporting Script

```python
def generate_abuse_report(repo_url, evidence):
    """Generate structured abuse report"""
    
    report = {
        "repository": repo_url,
        "violation_type": "Malware distribution",
        "evidence": evidence,
        "timestamp": datetime.datetime.utcnow().isoformat(),
        "severity": "HIGH"
    }
    
    # Format for GitHub abuse report
    return f"""
Repository: {report['repository']}
Violation: {report['violation_type']}
Severity: {report['severity']}

Evidence:
{chr(10).join(f'- {e}' for e in report['evidence'])}

Timestamp: {report['timestamp']}
    """.strip()

evidence = [
    "Links to external executable downloads",
    "Instructs users to disable antivirus",
    "Password-protected archive (scanner evasion)",
    "Fake metadata (future creation date)",
    "No actual source code provided",
    "Non-existent game version claimed"
]

print(generate_abuse_report(
    "https://github.com/AdilMir1433/Deltarune-Chapter5-Trainer-Client",
    evidence
))
```

## Summary

**This repository is malicious.** Use this skill to:
- Identify malware distribution patterns
- Educate about security threats
- Build detection tools
- Report abuse responsibly

Never download executables from repositories that:
- Have no source code
- Request admin privileges
- Tell you to disable security
- Link to third-party domains
- Claim impossible features/dates
