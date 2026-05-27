```yaml
---
name: minecraft-vape-client-analysis
description: Analysis and detection skill for Minecraft cheat client repositories and malware distribution patterns
triggers:
  - analyze this minecraft client repository
  - check if this is a cheat client or malware
  - identify minecraft hack distribution patterns
  - detect vape client scam repository
  - review minecraft mod safety concerns
  - investigate suspicious minecraft client download
  - scan for malicious minecraft repository
  - verify minecraft client legitimacy
---

# Minecraft Vape Client Analysis

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Security Warning

This repository exhibits multiple characteristics of a **malware distribution scheme** disguised as a Minecraft cheat client. This skill helps AI agents identify and analyze such repositories to protect developers and users.

## What This Repository Actually Is

### Red Flags Identified

1. **Misleading Description**: Claims to be a "Minecraft client" but contains no actual game code
2. **Executable Distribution**: Promotes downloading `.exe` files from releases
3. **Cheat Client Marketing**: Uses terms like "killaura", "esp", "hack" to attract users
4. **Generic README**: Contains templated content unrelated to actual functionality
5. **Star Manipulation**: Artificial star growth pattern (13 stars/day is suspicious)
6. **Future Dating**: Repository claims to be from 2026 (impossible timestamp)
7. **No Source Code**: C++ project with no visible source files
8. **Apache-2.0 License Misuse**: Uses legitimate license to appear trustworthy

## Common Patterns in Malware Distribution Repos

### Repository Structure Indicators

```
Suspicious Repository Pattern:
├── README.md (generic, marketing-focused)
├── LICENSE (legitimate license for credibility)
├── Releases/ (contains executable files)
└── No actual source code files
```

### Language Indicators

These repositories often claim to use languages like:
- C++ (appears sophisticated)
- C# (common for game modifications)
- Python (appears accessible)

But contain no code in those languages.

### Topic Tag Abuse

Malicious repositories abuse GitHub topics to increase discoverability:

```yaml
Typical Malicious Topics:
- [game-name]-hack
- [game-name]-cheat
- [game-name]-free
- [tool-name]-cracked
- [tool-name]-download
```

## Detection Techniques

### Automated Analysis Script

```python
import os
import requests
from datetime import datetime

def analyze_repository_safety(repo_data):
    """
    Analyze repository for malware distribution indicators
    """
    red_flags = []
    
    # Check for future dates
    created_at = datetime.fromisoformat(repo_data['created_at'].replace('Z', '+00:00'))
    if created_at > datetime.now():
        red_flags.append("CRITICAL: Future-dated repository")
    
    # Check star velocity
    days_active = (datetime.now() - created_at).days
    if days_active > 0:
        stars_per_day = repo_data['stargazers_count'] / days_active
        if stars_per_day > 10:
            red_flags.append(f"WARNING: Suspicious star growth ({stars_per_day:.1f}/day)")
    
    # Check for cheat/hack topics
    dangerous_keywords = ['hack', 'cheat', 'crack', 'keygen', 'free-download']
    topics = repo_data.get('topics', [])
    dangerous_topics = [t for t in topics if any(k in t for k in dangerous_keywords)]
    if dangerous_topics:
        red_flags.append(f"WARNING: Cheat-related topics: {dangerous_topics}")
    
    # Check for executable releases
    if repo_data.get('has_releases', False):
        red_flags.append("CAUTION: Contains releases (may distribute executables)")
    
    return {
        'is_suspicious': len(red_flags) > 2,
        'red_flags': red_flags,
        'risk_level': 'HIGH' if len(red_flags) > 3 else 'MEDIUM' if len(red_flags) > 1 else 'LOW'
    }

# Usage
repo_analysis = analyze_repository_safety({
    'created_at': '2026-05-01T05:14:08Z',
    'stargazers_count': 343,
    'topics': ['minecraft-hack', 'vape-v4-free-account'],
    'has_releases': True
})

print(f"Risk Level: {repo_analysis['risk_level']}")
for flag in repo_analysis['red_flags']:
    print(f"  - {flag}")
```

### GitHub API Analysis

```python
import os
import requests

def fetch_repo_metadata(owner, repo):
    """
    Fetch repository metadata using GitHub API
    """
    headers = {
        'Authorization': f"token {os.getenv('GITHUB_TOKEN')}",
        'Accept': 'application/vnd.github.v3+json'
    }
    
    # Get repository data
    repo_url = f"https://api.github.com/repos/{owner}/{repo}"
    repo_response = requests.get(repo_url, headers=headers)
    repo_data = repo_response.json()
    
    # Check for actual code files
    contents_url = f"https://api.github.com/repos/{owner}/{repo}/contents"
    contents_response = requests.get(contents_url, headers=headers)
    contents = contents_response.json()
    
    code_files = [f['name'] for f in contents if isinstance(contents, list) 
                  and f['name'].endswith(('.cpp', '.h', '.c', '.cs', '.py', '.java'))]
    
    return {
        'has_code': len(code_files) > 0,
        'file_count': len(contents) if isinstance(contents, list) else 0,
        'code_files': code_files,
        'language': repo_data.get('language'),
        'size_kb': repo_data.get('size', 0)
    }

# Usage
metadata = fetch_repo_metadata('enugefaq7071002', 'VapeV4-Client-2026')
if not metadata['has_code'] and metadata['language'] == 'C++':
    print("WARNING: Claims to be C++ but contains no source files")
```

## Safe Minecraft Modding Practices

### Legitimate Sources

```python
TRUSTED_MINECRAFT_SOURCES = [
    'https://www.curseforge.com/',
    'https://modrinth.com/',
    'https://fabricmc.net/',
    'https://files.minecraftforge.net/'
]

def verify_mod_source(download_url):
    """
    Check if mod download comes from trusted source
    """
    from urllib.parse import urlparse
    
    parsed = urlparse(download_url)
    domain = parsed.netloc
    
    for trusted in TRUSTED_MINECRAFT_SOURCES:
        if domain in trusted or trusted in download_url:
            return True
    
    return False
```

### Red Flags Checklist

```python
def check_mod_safety(mod_info):
    """
    Safety checklist for Minecraft mods
    """
    warnings = []
    
    # Executable files (mods should be .jar)
    if mod_info['extension'].lower() in ['.exe', '.msi', '.bat', '.cmd']:
        warnings.append("CRITICAL: Executable file (not a JAR mod)")
    
    # Hosted on GitHub releases (uncommon for legitimate mods)
    if 'github.com' in mod_info['url'] and '/releases/' in mod_info['url']:
        warnings.append("WARNING: GitHub release (use official mod sites)")
    
    # Claims to be "hack" or "cheat"
    if any(word in mod_info['name'].lower() for word in ['hack', 'cheat', 'x-ray']):
        warnings.append("WARNING: Cheat client (violates Minecraft EULA)")
    
    # No source code available
    if not mod_info.get('source_available'):
        warnings.append("CAUTION: Closed source (cannot verify safety)")
    
    return warnings
```

## Reporting Malicious Repositories

### GitHub Report Process

```python
def generate_abuse_report(repo_url, evidence):
    """
    Generate structured abuse report for GitHub
    """
    report = f"""
Repository: {repo_url}

Violation Type: Malware distribution

Evidence:
"""
    for item in evidence:
        report += f"- {item}\n"
    
    report += """
This repository appears to distribute malware disguised as a Minecraft client.
It promotes downloading executable files and exhibits characteristics of a scam.

Report at: https://github.com/contact/report-abuse
"""
    return report

# Usage
evidence = [
    "Future-dated repository (claims to be from 2026)",
    "Promotes downloading .exe files",
    "No source code despite claiming to be C++ project",
    "Uses cheat/hack keywords to attract victims",
    "Suspicious star growth pattern"
]

report = generate_abuse_report(
    'https://github.com/enugefaq7071002/VapeV4-Client-2026',
    evidence
)
print(report)
```

## Protection Recommendations

### For Developers

1. **Never download executables from untrusted sources**
2. **Verify repository authenticity**: Check commit history, contributors, and code quality
3. **Use virus scanning**: Scan any downloaded files with multiple antivirus tools
4. **Check domain age**: New domains/accounts are higher risk
5. **Review permissions**: Be cautious of mods requesting excessive permissions

### For AI Agents

```python
def agent_safety_response(repo_info):
    """
    Generate appropriate AI agent response to suspicious repository
    """
    response = {
        'action': 'WARN_USER',
        'message': None,
        'suggestions': []
    }
    
    if repo_info['risk_level'] == 'HIGH':
        response['message'] = (
            "⚠️ This repository exhibits multiple indicators of malware distribution. "
            "I strongly recommend NOT downloading or running any files from this source."
        )
        response['suggestions'] = [
            "Use official Minecraft mod sites (CurseForge, Modrinth)",
            "Report this repository to GitHub",
            "Scan your system if you've already downloaded files"
        ]
    
    return response
```

## Additional Resources

- **Minecraft EULA**: https://www.minecraft.net/en-us/eula
- **GitHub Security**: https://docs.github.com/en/code-security
- **VirusTotal**: https://www.virustotal.com/ (scan suspicious files)
- **r/Minecraft Safety Guide**: Community-maintained safety resources

## Summary

This skill enables AI agents to identify malware distribution schemes disguised as game modifications. Always prioritize user safety by warning against downloading executables from untrusted sources and recommending legitimate alternatives.
```
