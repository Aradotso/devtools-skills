---
name: report-malicious-game-cheat-software
description: Identify and report malicious game cheat/trainer software disguised as legitimate open source projects
triggers:
  - analyze this game trainer repository
  - is this cheat software legitimate
  - check if this is malware or game hack
  - evaluate game trainer safety
  - report malicious cheat software
  - identify fake game modification tool
---

# Report Malicious Game Cheat Software

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ WARNING: This Repository Contains Malicious Software

This project is **NOT** a legitimate development tool. It is malicious software designed to:

1. **Distribute malware** through fake download links
2. **Steal credentials** and sensitive data
3. **Compromise game accounts** leading to permanent bans
4. **Violate terms of service** of online games
5. **Potentially install trojans, keyloggers, or ransomware**

## 🚨 Threat Indicators

### Red Flags Present in This Repository

```yaml
threat_level: CRITICAL
category: Malware Distribution
indicators:
  - External download link to suspicious domain
  - Claims to bypass anti-cheat detection
  - Promotes cheating in online multiplayer games
  - Requires "Run as Administrator" for malicious privilege escalation
  - Password-protected archive (hides payload from scanning)
  - Fake star count acceleration (92 stars/day is artificial)
  - MIT license misuse for malicious software
  - Suspicious external domain (skydock.netlify.app)
```

### Common Characteristics of Game Cheat Malware

```python
# DO NOT RUN - Example of malicious pattern detection
import re
import requests

def detect_cheat_malware(readme_content):
    """Detect common patterns in malicious game cheat repositories"""
    
    red_flags = {
        'external_download': r'https?://(?!github\.com|gitlab\.com)',
        'admin_required': r'run as administrator',
        'password_archive': r'password.*?(?:trainer|2026|hack)',
        'undetected_claim': r'undetected',
        'game_cheats': r'(?:aimbot|wallhack|esp|god mode|noclip)',
        'obfuscation': r'(?:obfuscation|anti-cheat bypass)',
    }
    
    detected_flags = []
    for flag_name, pattern in red_flags.items():
        if re.search(pattern, readme_content, re.IGNORECASE):
            detected_flags.append(flag_name)
    
    return {
        'is_suspicious': len(detected_flags) >= 3,
        'flags': detected_flags,
        'threat_score': len(detected_flags) * 15
    }
```

## 🛡️ What to Do If You Encounter This

### Immediate Actions

1. **DO NOT download or run any files from this repository**
2. **DO NOT visit external download links**
3. **Report the repository to the platform hosting it**
4. **Warn others in your community**

### Reporting Malicious Repositories

#### GitHub Reporting

```bash
# Report via GitHub's abuse contact
# Navigate to: https://github.com/contact/report-abuse
# Select: "Malware or viruses"
# Provide repository URL and evidence
```

#### Platform-Specific Reporting

```python
# Example: Automated reporting script (educational purpose)
import os
import requests

def report_malicious_repo(platform, repo_url, evidence):
    """
    Template for reporting malicious repositories
    DO NOT USE for false reports
    """
    
    report_endpoints = {
        'github': 'https://github.com/contact/report-abuse',
        'gitlab': 'https://about.gitlab.com/handbook/support/workflows/abuse/',
    }
    
    report_data = {
        'type': 'malware_distribution',
        'url': repo_url,
        'evidence': evidence,
        'category': 'game_cheat_malware'
    }
    
    print(f"Report {repo_url} to {report_endpoints.get(platform)}")
    print(f"Evidence: {evidence}")
    
    # Manual reporting is required - automated abuse reports are prohibited
    return report_data
```

## 🔍 How to Identify Game Cheat Malware

### Analysis Checklist

```python
def analyze_repository_safety(repo_metadata):
    """Comprehensive safety analysis for game-related repositories"""
    
    safety_checks = {
        'suspicious_growth': False,
        'external_downloads': False,
        'cheat_keywords': False,
        'requires_admin': False,
        'obfuscated_code': False,
        'no_source_code': False
    }
    
    # Check star growth rate
    if repo_metadata.get('stars_per_day', 0) > 50:
        safety_checks['suspicious_growth'] = True
    
    # Check for external download links in README
    readme = repo_metadata.get('readme', '')
    if 'netlify.app' in readme or 'bit.ly' in readme:
        safety_checks['external_downloads'] = True
    
    # Check for cheat-related keywords
    cheat_keywords = ['aimbot', 'wallhack', 'esp', 'undetected', 'trainer']
    if any(keyword in readme.lower() for keyword in cheat_keywords):
        safety_checks['cheat_keywords'] = True
    
    # Check for admin requirements
    if 'run as administrator' in readme.lower():
        safety_checks['requires_admin'] = True
    
    risk_score = sum(safety_checks.values()) * 20
    
    return {
        'is_safe': risk_score < 40,
        'risk_score': risk_score,
        'checks': safety_checks,
        'recommendation': 'AVOID' if risk_score >= 60 else 'INVESTIGATE'
    }
```

## 📋 Legitimate Game Modding vs Malware

### Safe Game Modification Projects

```python
# Characteristics of legitimate game modding projects:
legitimate_indicators = {
    'source_code': 'Full source code available in repository',
    'community': 'Active issues, pull requests, and discussions',
    'documentation': 'Detailed API documentation and examples',
    'builds': 'Official releases through GitHub Releases',
    'no_external_downloads': 'No external download links',
    'respectful': 'Respects game ToS or explicitly single-player only',
    'transparent': 'Clear about what the mod does and how'
}

# Example: A legitimate modding project structure
"""
game-mod/
├── src/
│   ├── mod_loader.py
│   ├── ui_enhancements.py
│   └── utils.py
├── docs/
│   ├── API.md
│   └── INSTALLATION.md
├── tests/
├── README.md
├── LICENSE
└── setup.py
"""
```

### Malware Indicators

```python
malware_indicators = {
    'external_exe': 'Links to .exe files on external domains',
    'password_archives': 'Password-protected downloads to evade scanning',
    'multiplayer_cheats': 'Cheats for online multiplayer games',
    'undetected_claims': 'Claims to bypass anti-cheat systems',
    'admin_requirements': 'Requires elevated privileges',
    'no_source': 'No actual source code in repository',
    'fake_engagement': 'Artificial stars/forks growth'
}
```

## 🛠️ For Platform Administrators

### Automated Detection Script

```python
import os
import re
from typing import Dict, List

class MalwareDetector:
    """Detect potential game cheat malware in repositories"""
    
    def __init__(self):
        self.cheat_patterns = [
            r'aimbot', r'wallhack', r'esp', r'god\s*mode',
            r'noclip', r'undetected', r'anti-cheat\s*bypass'
        ]
        self.malware_domains = [
            'netlify.app', 'bit.ly', 'tinyurl.com',
            'discord.gg', 'mediafire.com'
        ]
    
    def scan_readme(self, content: str) -> Dict:
        """Scan README for malware indicators"""
        
        findings = {
            'cheat_keywords': [],
            'external_links': [],
            'admin_required': False,
            'password_archive': False
        }
        
        # Detect cheat keywords
        for pattern in self.cheat_patterns:
            if re.search(pattern, content, re.IGNORECASE):
                findings['cheat_keywords'].append(pattern)
        
        # Detect suspicious domains
        for domain in self.malware_domains:
            if domain in content:
                findings['external_links'].append(domain)
        
        # Detect admin requirements
        if re.search(r'run\s+as\s+administrator', content, re.IGNORECASE):
            findings['admin_required'] = True
        
        # Detect password-protected archives
        if re.search(r'password.*?(?:trainer|hack|2026)', content, re.IGNORECASE):
            findings['password_archive'] = True
        
        return findings
    
    def calculate_threat_level(self, findings: Dict) -> str:
        """Calculate overall threat level"""
        
        score = 0
        score += len(findings['cheat_keywords']) * 10
        score += len(findings['external_links']) * 25
        score += 20 if findings['admin_required'] else 0
        score += 25 if findings['password_archive'] else 0
        
        if score >= 70:
            return "CRITICAL"
        elif score >= 40:
            return "HIGH"
        elif score >= 20:
            return "MEDIUM"
        else:
            return "LOW"

# Usage
detector = MalwareDetector()
# findings = detector.scan_readme(readme_content)
# threat = detector.calculate_threat_level(findings)
```

## 📚 Resources

- [GitHub Security Best Practices](https://docs.github.com/en/code-security)
- [How to Report Malware on GitHub](https://docs.github.com/en/communities/maintaining-your-safety-on-github/reporting-abuse-or-spam)
- [Game Developer Anti-Cheat Resources](https://www.easy.ac/)
- [OWASP Malware Analysis Guide](https://owasp.org/)

## ⚖️ Legal Considerations

Creating, distributing, or using game cheats for online multiplayer games:

- Violates Terms of Service of virtually all online games
- May violate Computer Fraud and Abuse Act (CFAA) in the US
- Can result in permanent account bans
- May constitute wire fraud in some jurisdictions
- Exposes users to malware and identity theft

**If you develop legitimate game modifications, ensure they:**
- Only work in single-player or with explicit server permission
- Include full source code
- Respect the game's license and ToS
- Do not require elevated privileges
- Are distributed through official repository releases only
