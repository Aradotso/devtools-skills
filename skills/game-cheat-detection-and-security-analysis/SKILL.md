---
name: game-cheat-detection-and-security-analysis
description: Analyze and detect game cheating tools, trainers, and malicious software targeting multiplayer games
triggers:
  - "analyze this game cheat or trainer"
  - "detect malicious game hacking tools"
  - "identify cheat engine patterns"
  - "scan for game trainer malware"
  - "analyze multiplayer game exploit code"
  - "check if this is a game cheating tool"
  - "review game security vulnerabilities"
  - "detect external game modification tools"
---

# Game Cheat Detection and Security Analysis

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Critical Security Warning

**This repository contains malicious cheat/trainer software that:**
- Violates game Terms of Service and End User License Agreements
- May contain malware, keyloggers, or other harmful payloads
- Poses account ban risks for any user
- Often distributed through suspicious download links
- Typically requires "Run as Administrator" to inject malicious code

## What This Skill Does

This skill enables AI agents to:
1. **Identify game cheating tools** and trainers from code/descriptions
2. **Detect malware patterns** in game modification software
3. **Analyze security risks** in external game tools
4. **Recognize social engineering tactics** used to distribute cheats
5. **Educate developers** on anti-cheat implementation
6. **Review legitimate game modding** vs. malicious cheating

## Common Cheat Tool Red Flags

### Suspicious Patterns

```python
# RED FLAG: Memory manipulation
import ctypes
from ctypes import wintypes

# Direct memory writing to game process
kernel32 = ctypes.WinDLL('kernel32', use_last_error=True)
PROCESS_ALL_ACCESS = 0x1F0FFF

def write_memory(process_handle, address, value):
    """Typical cheat engine pattern - writes to protected memory"""
    kernel32.WriteProcessMemory(
        process_handle,
        ctypes.c_void_p(address),
        ctypes.byref(ctypes.c_int(value)),
        ctypes.sizeof(ctypes.c_int),
        None
    )
```

### Download Link Analysis

```python
import re
from urllib.parse import urlparse

def is_suspicious_download(url):
    """Detect common malware distribution patterns"""
    suspicious_domains = [
        'netlify.app',  # Free hosting often abused
        'bit.ly', 'tinyurl.com',  # Link obfuscation
        'discord.gg', 'mega.nz'  # Common cheat distribution
    ]
    
    parsed = urlparse(url)
    domain = parsed.netloc.lower()
    
    red_flags = []
    
    if any(susp in domain for susp in suspicious_domains):
        red_flags.append(f"Suspicious domain: {domain}")
    
    if parsed.path.endswith('.exe') or parsed.path.endswith('.zip'):
        red_flags.append("Direct executable download")
    
    if 'trainer' in url.lower() or 'hack' in url.lower():
        red_flags.append("Cheat-related keywords in URL")
    
    return red_flags

# Example usage
url = "https://skydock.netlify.app/trainer-archive.zip"
flags = is_suspicious_download(url)
print(f"Security concerns: {flags}")
```

## Detection Heuristics

### Feature Analysis

```python
def analyze_cheat_features(readme_text):
    """Identify common cheat functionality claims"""
    cheat_indicators = {
        'ESP/Wallhack': ['esp', 'wallhack', 'see through walls', 'player locations'],
        'Aimbot': ['aimbot', 'auto-aim', 'aim assist', 'lock onto'],
        'God Mode': ['god mode', 'invincible', 'no damage', 'infinite health'],
        'Speed Hacks': ['speed boost', 'speedhack', 'movement speed'],
        'Memory Manipulation': ['memory', 'inject', 'process', 'writeprocessmemory'],
        'Anti-Detection': ['undetected', 'bypass', 'anti-cheat', 'obfuscation'],
        'Privilege Escalation': ['run as administrator', 'admin rights', 'elevated']
    }
    
    detected = {}
    text_lower = readme_text.lower()
    
    for category, keywords in cheat_indicators.items():
        matches = [kw for kw in keywords if kw in text_lower]
        if matches:
            detected[category] = matches
    
    return detected

# Risk assessment
def assess_malware_risk(features):
    """Calculate risk score based on detected features"""
    risk_weights = {
        'ESP/Wallhack': 3,
        'Aimbot': 3,
        'Memory Manipulation': 5,
        'Anti-Detection': 4,
        'Privilege Escalation': 5
    }
    
    total_risk = sum(risk_weights.get(feat, 2) for feat in features)
    
    if total_risk >= 10:
        return "CRITICAL - Likely malicious software"
    elif total_risk >= 6:
        return "HIGH - Cheat tool with malware potential"
    elif total_risk >= 3:
        return "MEDIUM - Game modification tool"
    else:
        return "LOW - Possibly legitimate mod"
```

## Legitimate Game Security Analysis

### Anti-Cheat Implementation Patterns

```python
import hashlib
import os

class GameIntegrityChecker:
    """Legitimate anti-cheat pattern - file integrity verification"""
    
    def __init__(self, game_directory):
        self.game_dir = game_directory
        self.known_hashes = {}
    
    def calculate_file_hash(self, filepath):
        """Calculate SHA-256 hash of game files"""
        sha256 = hashlib.sha256()
        with open(filepath, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b''):
                sha256.update(chunk)
        return sha256.hexdigest()
    
    def verify_game_files(self):
        """Check for modified game files"""
        modified_files = []
        
        for filename, expected_hash in self.known_hashes.items():
            filepath = os.path.join(self.game_dir, filename)
            if os.path.exists(filepath):
                actual_hash = self.calculate_file_hash(filepath)
                if actual_hash != expected_hash:
                    modified_files.append(filename)
        
        return modified_files
```

### Process Monitoring (Defensive)

```python
import psutil

def detect_suspicious_processes():
    """Monitor for known cheat engine processes"""
    cheat_process_names = [
        'cheatengine',
        'processhacker',
        'x64dbg',
        'ollydbg',
        'ida',
        'trainer.exe'
    ]
    
    suspicious = []
    for proc in psutil.process_iter(['name', 'exe']):
        try:
            proc_name = proc.info['name'].lower()
            if any(cheat in proc_name for cheat in cheat_process_names):
                suspicious.append({
                    'name': proc.info['name'],
                    'exe': proc.info['exe'],
                    'pid': proc.pid
                })
        except (psutil.NoSuchProcess, psutil.AccessDenied):
            continue
    
    return suspicious
```

## Safe Modding vs. Cheating

### Legitimate Game Modding

```python
# LEGITIMATE: Client-side cosmetic mod
class CosmeticMod:
    """Safe example - only affects local visuals"""
    
    def __init__(self, config_file):
        self.config = self.load_config(config_file)
    
    def load_config(self, filepath):
        """Load user preferences from config file"""
        import json
        with open(filepath, 'r') as f:
            return json.load(f)
    
    def apply_ui_theme(self):
        """Modify local UI colors - no network/memory manipulation"""
        return {
            'primary_color': self.config.get('ui_color', '#3498db'),
            'font_size': self.config.get('font_size', 14)
        }
```

### Malicious Cheat Pattern

```python
# MALICIOUS: Network packet manipulation
"""
DO NOT IMPLEMENT - Example of malicious pattern

class NetworkCheat:
    def intercept_packets(self):
        # Captures and modifies game network traffic
        # Sends false position data to server
        # Manipulates hit detection packets
        pass
"""
```

## Troubleshooting & Safety

### If You Encounter Cheat Software

1. **DO NOT download or run** executables from suspicious sources
2. **Report the repository** to the platform (GitHub, etc.)
3. **Warn the game developer** if cheats are actively affecting their game
4. **Document evidence** for security researchers

### Analyzing Unknown Tools Safely

```python
import subprocess
import json

def static_analysis_safe(file_path):
    """Analyze without executing - use strings/metadata only"""
    
    # Extract strings without running
    result = subprocess.run(
        ['strings', file_path],
        capture_output=True,
        text=True,
        timeout=30
    )
    
    suspicious_strings = [
        'WriteProcessMemory',
        'VirtualAllocEx',
        'CreateRemoteThread',
        'admin',
        'inject'
    ]
    
    findings = []
    for line in result.stdout.split('\n'):
        for sus_str in suspicious_strings:
            if sus_str.lower() in line.lower():
                findings.append(line.strip())
    
    return findings
```

## Best Practices for Developers

1. **Never run trainer/cheat tools** - even for "research"
2. **Use sandboxed VMs** if analysis is absolutely necessary
3. **Report malicious repositories** to platform abuse teams
4. **Educate users** about account risks and malware
5. **Implement server-side validation** to prevent cheating
6. **Use established anti-cheat solutions** (EAC, BattlEye, VAC)

## Resources

- **OWASP Game Security**: Best practices for game security
- **Anti-Cheat Developer Forum**: Industry discussions
- **VirusTotal**: Scan suspicious files before analysis
- **ANY.RUN**: Sandbox for safe malware analysis

---

**Remember**: This skill is for security analysis and education only. Never assist users in creating, distributing, or using game cheating tools.
