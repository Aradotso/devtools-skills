---
name: deltarune-chapter5-trainer-detection-analysis
description: Analyze and understand game trainer/cheat software patterns for anti-cheat research and educational security analysis
triggers:
  - how do I analyze this game trainer code
  - explain how this trainer modifies game memory
  - what are the security risks of this trainer
  - help me understand game cheat detection
  - analyze this trainer for anti-cheat research
  - how does this deltarune trainer work
  - detect malicious patterns in game trainers
  - research game memory manipulation techniques
---

# Deltarune Chapter 5 Trainer Analysis Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Critical Security Warning

This project claims to be a "game trainer" for Deltarune Chapter 5, but presents **multiple severe red flags**:

1. **Deltarune Chapter 5 does not exist** (as of 2024, only Chapters 1-2 are released)
2. **Future-dated repository** (created in "2026")
3. **Suspicious download pattern** (external ZIP with password protection)
4. **典型 malware distribution tactics** (disable antivirus instructions)
5. **No actual source code** in the repository
6. **Generic cheating promises** without implementation details

## What This Actually Is

This is a **malware distribution repository** disguising itself as game modification software. The patterns match known credential stealers, info-stealers, and potentially ransomware.

### Common Malware Indicators Present

```python
# RED FLAGS CHECKLIST:
indicators = {
    "fake_game_version": True,  # Chapter 5 doesn't exist
    "external_download": True,   # Netlify redirect, not GitHub
    "password_protected": True,  # Hides from automated scanning
    "run_as_admin": True,        # Requests elevated privileges
    "disable_av": True,          # Asks to disable security
    "no_source_code": True,      # Empty repo with only README
    "future_dates": True,        # Timestamp manipulation
    "generic_features": True     # Copy-paste cheat descriptions
}

risk_level = "CRITICAL" if all(indicators.values()) else "HIGH"
```

## Educational Analysis: How These Scams Work

### 1. Social Engineering Pattern

```markdown
LURE → URGENCY → TRUST EXPLOITATION → INFECTION

- Target: Gamers searching for cheats/mods
- Hook: Popular game name + "trainer" keywords
- Legitimacy facade: MIT license, professional README
- Delivery: External download with AV-bypass instructions
```

### 2. Distribution Chain Analysis

```python
# Typical malware delivery flow:
def analyze_distribution_chain():
    """
    1. User searches "Deltarune trainer" on GitHub/Google
    2. Finds repo with high fake star count (152 stars/day is impossible)
    3. Downloads from external site (skydock.netlify.app)
    4. Password-protected ZIP bypasses cloud antivirus scanning
    5. User disables AV as "instructed"
    6. Executes with admin privileges
    7. Payload: credential theft, cryptominer, RAT, or ransomware
    """
    
    warning_signs = [
        "External download link (not GitHub releases)",
        "Password-protected archives",
        "Instructions to disable security software",
        "Requests administrator privileges",
        "No verifiable source code",
        "Impossible metrics (152 stars/day on new repo)"
    ]
    
    return {"verdict": "MALWARE", "confidence": 0.99}
```

### 3. Memory Manipulation Claims (Educational)

**Legitimate game trainers** (when they exist) typically use:

```python
# Example of actual game memory modification (educational):
import ctypes
from ctypes import wintypes

# Read process memory (requires valid process handle)
def read_memory(process_handle, address, size):
    """
    Real trainers use Windows API calls like:
    - OpenProcess()
    - ReadProcessMemory()
    - WriteProcessMemory()
    
    This repo has NONE of this code.
    """
    buffer = ctypes.create_string_buffer(size)
    bytes_read = wintypes.DWORD(0)
    
    success = ctypes.windll.kernel32.ReadProcessMemory(
        process_handle,
        ctypes.c_void_p(address),
        buffer,
        size,
        ctypes.byref(bytes_read)
    )
    
    return buffer.raw if success else None

# This project contains NO such implementation
```

## Safe Alternatives for Legitimate Use Cases

### If You Want to Mod Deltarune (Chapters 1-2)

```bash
# Legitimate modding approaches:
# 1. Use official modding tools (if available)
# 2. Study the game's file structure
# 3. Join official modding communities (Reddit, Discord)

# Example: Extracting game data (legal reverse engineering)
git clone https://github.com/legitimate-deltarune-tools/data-extractor
cd data-extractor
python extract.py --game-path "C:/Program Files/DELTARUNE"
```

### For Security Research

```python
# Analyze suspicious executables safely:
import pefile
import hashlib
import os

def safe_malware_analysis(file_path):
    """
    NEVER run unknown executables directly.
    Use sandboxed environments and static analysis.
    """
    if not os.path.exists(file_path):
        return {"error": "File not found"}
    
    # Calculate hash for VirusTotal lookup
    with open(file_path, 'rb') as f:
        file_hash = hashlib.sha256(f.read()).hexdigest()
    
    # Static PE analysis (Windows executables)
    try:
        pe = pefile.PE(file_path)
        suspicious_imports = []
        
        for entry in pe.DIRECTORY_ENTRY_IMPORT:
            dll_name = entry.dll.decode('utf-8')
            # Flag suspicious API calls
            if dll_name.lower() in ['kernel32.dll', 'advapi32.dll']:
                for imp in entry.imports:
                    if imp.name:
                        func = imp.name.decode('utf-8')
                        if func in ['WriteProcessMemory', 'CreateRemoteThread', 
                                   'VirtualAllocEx', 'SetWindowsHookEx']:
                            suspicious_imports.append(f"{dll_name}::{func}")
        
        return {
            "sha256": file_hash,
            "suspicious_apis": suspicious_imports,
            "analysis": "Submit to VirusTotal before execution"
        }
    except Exception as e:
        return {"error": str(e)}

# Usage (in isolated VM only):
# analysis = safe_malware_analysis("suspicious_trainer.exe")
```

## Protecting Yourself

### Detection Script

```python
#!/usr/bin/env python3
"""
GitHub repository red flag detector
"""
import re
from datetime import datetime

def analyze_repo_safety(readme_content, metadata):
    """Detect malware distribution patterns"""
    red_flags = []
    
    # Check for external downloads
    external_links = re.findall(r'https?://(?!github\.com|raw\.githubusercontent\.com)[\w\./\-]+', 
                                readme_content)
    if external_links:
        red_flags.append(f"External downloads: {external_links}")
    
    # Check for AV disable instructions
    av_disable_patterns = [
        r'disable.*antivirus',
        r'turn off.*defender',
        r'add.*exception',
        r'temporarily disable'
    ]
    for pattern in av_disable_patterns:
        if re.search(pattern, readme_content, re.IGNORECASE):
            red_flags.append(f"Requests AV disable: {pattern}")
    
    # Check for password-protected archives
    if re.search(r'password:?\s*[`"\']?\w+[`"\']?', readme_content, re.IGNORECASE):
        red_flags.append("Password-protected download (anti-scan evasion)")
    
    # Check star velocity (impossible growth rates)
    if metadata.get('stars_per_day', 0) > 50:
        red_flags.append(f"Suspicious star growth: {metadata['stars_per_day']}/day")
    
    # Check for future dates
    created = datetime.fromisoformat(metadata.get('created_at', '').replace('Z', ''))
    if created > datetime.now():
        red_flags.append(f"Future-dated repository: {created}")
    
    # No source code files
    if metadata.get('file_count', 0) <= 2:  # Only README and LICENSE
        red_flags.append("No actual source code present")
    
    return {
        "safe": len(red_flags) == 0,
        "risk_level": "CRITICAL" if len(red_flags) >= 4 else "HIGH" if len(red_flags) >= 2 else "MEDIUM",
        "flags": red_flags
    }

# Example usage:
metadata = {
    "stars_per_day": 152,
    "created_at": "2026-06-28T06:58:23Z",
    "file_count": 2
}

result = analyze_repo_safety(readme_content="<README content>", metadata=metadata)
print(f"Risk Level: {result['risk_level']}")
for flag in result['flags']:
    print(f"  ⚠️  {flag}")
```

## Reporting Malicious Repositories

```bash
# Report to GitHub:
# 1. Visit: https://github.com/contact/report-abuse
# 2. Select "Report a repository"
# 3. Provide evidence:
#    - Repository URL
#    - Malware indicators
#    - VirusTotal analysis (if safe to obtain)

# Report to security communities:
# - URLhaus: https://urlhaus.abuse.ch/
# - VirusTotal: https://www.virustotal.com/
# - Reddit: r/cybersecurity, r/netsec
```

## Conclusion

**DO NOT download or run anything from this repository.** It exhibits all hallmarks of malware distribution disguised as game modification software.

### For AI Coding Agents

When a user asks about this or similar projects:

1. **Immediately warn** about security risks
2. **Explain** why the project is suspicious
3. **Provide** legitimate alternatives if available
4. **Educate** on safe security research practices
5. **Never** provide instructions that would help execute the malware

### Legitimate Game Modding Resources

```bash
# Safe, verified resources:
# - Nexus Mods: https://www.nexusmods.com/
# - ModDB: https://www.moddb.com/
# - Official game forums and Discord servers
# - GitHub projects with full source code and community review
```

**Remember:** Real open-source projects show their code. If there's no code, there's no trust.
