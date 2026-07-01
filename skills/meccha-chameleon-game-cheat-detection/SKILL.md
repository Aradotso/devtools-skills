---
name: meccha-chameleon-game-cheat-detection
description: Detect and analyze game cheating tools, trainers, and malicious external utilities targeting multiplayer games
triggers:
  - analyze this game cheat or trainer repository
  - detect game hacking tools in this codebase
  - identify cheat software patterns
  - scan for game exploit utilities
  - check if this is malicious game software
  - analyze game trainer or wallhack code
  - investigate cheating tool repository
  - detect unethical game modification software
---

# MECCHA CHAMELEON Game Cheat Detection

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ WARNING: Malicious Software Detection

This repository contains or claims to contain **game cheating software** (trainer/wallhack/aimbot). This skill helps AI coding agents identify, analyze, and warn users about such tools.

## What This Repository Actually Is

**XFaltixX/MECCHA-CHAMELEON-Trainer-Client** is advertised as:
- Game trainer for "MECCHA CHAMELEON" 
- ESP wallhack and aimbot functionality
- "Undetected" external cheat utility
- God mode, speed boost, and other game exploits

### Red Flags Indicating Malicious Software

1. **Suspicious download link**: External hosting on `skydock.netlify.app`
2. **Password-protected archive**: `trainer2026` - common malware distribution pattern
3. **Executable payload**: `trainer.exe` requiring admin rights
4. **No source code**: Only download links, no actual implementation
5. **"Undetected" claims**: Common in malware/cheating tools
6. **Rapid star growth**: 185 stars at 61/day suggests artificial inflation
7. **Zero forks/issues**: Indicates fake engagement

## Security Analysis Pattern

When analyzing suspected cheat/malware repositories, check for:

```python
# Pattern detection checklist
suspicious_indicators = {
    "external_downloads": True,  # Not hosted on GitHub releases
    "password_protected": True,   # Hiding malware from scanners
    "requires_admin": True,       # Elevated privileges
    "no_source_code": True,       # Only binaries
    "rapid_stars": True,          # Artificial engagement
    "gaming_cheats": True,        # Unethical software
    "undetected_claims": True     # Anti-detection focus
}

risk_level = sum(suspicious_indicators.values())
# 7/7 = CRITICAL RISK
```

## Ethical and Legal Concerns

### Game Cheating Software Issues

1. **Violates Terms of Service**: All multiplayer game ToS prohibit cheating
2. **Ruins Fair Play**: Destroys experience for legitimate players
3. **Often Contains Malware**: Cheat tools frequently bundle trojans, keyloggers, RATs
4. **Legal Liability**: Can result in permanent bans, legal action
5. **Security Risk**: Requires disabling antivirus, admin access

### Common Malware Distribution Tactics

```text
Typical attack chain:
1. Create fake repository with gaming keywords
2. Inflate stars/engagement artificially
3. Provide external download link (bypasses GitHub scanning)
4. Use password to evade automated malware detection
5. Request admin rights to install rootkit/RAT
6. Steal credentials, install crypto miners, botnet agents
```

## Detection Script Example

```python
import re
import requests
from urllib.parse import urlparse

def analyze_cheat_repository(readme_content: str, repo_metadata: dict) -> dict:
    """
    Analyze repository for game cheating/malware indicators.
    
    Returns risk assessment and recommendations.
    """
    risk_score = 0
    findings = []
    
    # Check for external download links
    external_links = re.findall(r'https?://(?!github\.com)[^\s\)]+', readme_content)
    if external_links:
        risk_score += 3
        findings.append(f"External downloads found: {external_links}")
    
    # Check for password mentions
    passwords = re.findall(r'password[:\s]+[`"]?(\w+)[`"]?', readme_content, re.I)
    if passwords:
        risk_score += 2
        findings.append(f"Password-protected archives: {passwords}")
    
    # Check for admin/elevated privileges
    if re.search(r'run as administrator|admin rights|elevation', readme_content, re.I):
        risk_score += 2
        findings.append("Requires elevated privileges")
    
    # Check for cheating keywords
    cheat_keywords = ['wallhack', 'aimbot', 'esp', 'undetected', 'trainer', 'god mode']
    found_keywords = [kw for kw in cheat_keywords if kw in readme_content.lower()]
    if found_keywords:
        risk_score += 2
        findings.append(f"Cheating keywords: {', '.join(found_keywords)}")
    
    # Check star velocity
    created = repo_metadata.get('created_at')
    stars = repo_metadata.get('stars', 0)
    if stars > 100:
        risk_score += 1
        findings.append(f"Suspicious engagement: {stars} stars")
    
    # Check for missing source code
    if readme_content.count('.exe') > 0 and readme_content.count('```') < 2:
        risk_score += 2
        findings.append("Binary distribution without source code")
    
    severity = 'CRITICAL' if risk_score >= 8 else 'HIGH' if risk_score >= 5 else 'MEDIUM'
    
    return {
        'risk_score': risk_score,
        'severity': severity,
        'findings': findings,
        'recommendation': 'DO NOT DOWNLOAD' if risk_score >= 5 else 'Exercise caution'
    }

# Example usage
readme = open('README.md').read()
metadata = {
    'stars': 185,
    'created_at': '2026-06-28T07:17:40Z',
    'forks': 0
}

result = analyze_cheat_repository(readme, metadata)
print(f"Risk Score: {result['risk_score']}/12")
print(f"Severity: {result['severity']}")
print(f"Recommendation: {result['recommendation']}")
for finding in result['findings']:
    print(f"  - {finding}")
```

## AI Agent Response Template

When a user asks about this or similar repositories:

```python
def generate_warning_response(repo_analysis: dict) -> str:
    """Generate appropriate warning for users."""
    
    response = f"""
⚠️ **SECURITY ALERT: Suspected Malicious Software**

This repository shows {repo_analysis['severity']} risk indicators for malware/cheating software.

**DO NOT:**
- Download files from external links
- Run any executables from this repository
- Disable antivirus or security software
- Grant administrator privileges

**Risks:**
- Malware infection (trojans, keyloggers, ransomware)
- Account theft and credential harvesting
- Game account permanent bans
- Legal consequences for cheating
- System compromise

**Recommendation:** {repo_analysis['recommendation']}

**Reported Findings:**
{chr(10).join(f'• {f}' for f in repo_analysis['findings'])}

If you're looking for legitimate game development tools, I can suggest ethical alternatives.
"""
    return response
```

## Legitimate Alternatives

If user wants game development/modding tools:

```python
legitimate_tools = {
    "game_development": [
        "Unity/Unreal Engine - Professional game engines",
        "Godot - Open source game engine",
        "PyGame - Python game development library"
    ],
    "game_modding": [
        "Official modding tools/SDKs from game developers",
        "Steam Workshop integration",
        "Game-specific modding communities"
    ],
    "testing": [
        "Cheat Engine (single-player only, ethical use)",
        "Official developer consoles",
        "Debug mode features"
    ]
}
```

## Reporting Malicious Repositories

```python
# Steps to report to GitHub
report_steps = """
1. Visit: https://github.com/contact/report-abuse
2. Select: "Malware or malicious software"
3. Provide: Repository URL and evidence
4. Include: Analysis of suspicious indicators
"""
```

## Key Takeaways

- **Always verify source code** before running executables
- **Never download** from password-protected archives
- **Cheating software** often contains malware
- **External links** bypass GitHub security scanning
- **Admin privileges** = full system access for malware
- **Report suspicious repositories** to protect others

This skill enables AI agents to identify and warn users about malicious game cheating software disguised as legitimate development tools.
