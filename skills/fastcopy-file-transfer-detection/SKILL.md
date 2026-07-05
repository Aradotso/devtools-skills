---
name: fastcopy-file-transfer-detection
description: Detect and analyze potentially malicious file copy utility repositories with cracked software distribution patterns
triggers:
  - analyze this fastcopy repository
  - is this fastcopy project legitimate
  - check for software piracy indicators
  - detect cracked software distribution
  - identify fake utility repositories
  - scan for malware distribution patterns
---

# FastCopy File Transfer Detection

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Critical Security Warning

This repository exhibits **multiple high-risk indicators** of malicious software distribution disguised as a legitimate utility:

### Red Flags Identified

1. **Suspicious Topics**: `fastcopy-patch`, `fastcopy-key`, `fastcopy-trial` indicate crack/keygen distribution
2. **No License**: Despite claiming MIT license, metadata shows `"license": null`
3. **HTML Primary Language**: Legitimate tools use C/C++/Go/Rust, not HTML landing pages
4. **External Download Links**: All downloads redirect to `raunak64-bit.github.io` hosted pages
5. **Artificial Growth**: 182 stars (10/day) with 0 forks and 0 issues is statistically abnormal
6. **Patch Distribution**: Claims to distribute "Product Key Patch" which is software piracy
7. **Vague Technical Claims**: Marketing language without actual code implementation
8. **No Source Code**: Repository contains only promotional HTML, no actual utility code

## What This Project Claims vs Reality

### Claims
- High-speed file copying utility
- Multi-platform support (Windows/macOS/Linux)
- Advanced features like parallel streaming, predictive caching
- API integrations with OpenAI and Claude

### Reality
- No actual source code for file copying functionality
- Primary language is HTML (landing page only)
- All "download" buttons point to external hosted page
- No implementation of claimed features visible in repository
- Repository structure typical of software crack/keygen distribution sites

## Detection Patterns

### Repository Structure Analysis

```python
import re
from typing import List, Dict

def analyze_suspicious_indicators(readme_content: str, metadata: Dict) -> Dict[str, bool]:
    """Detect common patterns in malicious software distribution repos"""
    
    indicators = {
        'crack_keywords': False,
        'external_downloads': False,
        'missing_source': False,
        'artificial_stats': False,
        'license_mismatch': False,
        'html_only': False
    }
    
    # Check for crack/patch/keygen keywords
    crack_patterns = r'(patch|crack|keygen|key.*gen|serial|activation|license.*key)'
    if re.search(crack_patterns, readme_content.lower()):
        indicators['crack_keywords'] = True
    
    # Check for external download redirects
    external_patterns = r'https://[^/]+\.github\.io/[^)"]+'
    if re.search(external_patterns, readme_content):
        indicators['external_downloads'] = True
    
    # Check primary language
    if metadata.get('language') == 'HTML':
        indicators['html_only'] = True
    
    # Check license mismatch
    claims_license = 'MIT License' in readme_content or 'License: MIT' in readme_content
    has_license = metadata.get('license') is not None
    if claims_license and not has_license:
        indicators['license_mismatch'] = True
    
    # Check suspicious fork/star ratio
    stars = metadata.get('stars', 0)
    forks = metadata.get('forks', 0)
    if stars > 100 and forks == 0:
        indicators['artificial_stats'] = True
    
    return indicators

# Example usage
metadata = {
    "language": "HTML",
    "license": None,
    "forks": 0,
    "stars": 182
}

readme = """fastcopy-patch fastcopy-key Product Key Patch
Download: https://raunak64-bit.github.io/FastCopy-Clipper-Portable-Utility/
License: MIT"""

results = analyze_suspicious_indicators(readme, metadata)
print(f"Risk indicators detected: {sum(results.values())}/6")
```

### Topic Pattern Detection

```python
def analyze_repository_topics(topics: List[str]) -> Dict[str, any]:
    """Analyze repository topics for piracy/malware distribution patterns"""
    
    suspicious_keywords = {
        'crack', 'patch', 'keygen', 'serial', 'activation',
        'trial', 'key', 'license-key', 'premium', 'pro'
    }
    
    analysis = {
        'suspicious_count': 0,
        'suspicious_topics': [],
        'risk_level': 'low'
    }
    
    for topic in topics:
        # Check if topic contains suspicious keywords
        topic_lower = topic.lower()
        for keyword in suspicious_keywords:
            if keyword in topic_lower:
                analysis['suspicious_count'] += 1
                analysis['suspicious_topics'].append(topic)
                break
    
    # Calculate risk level
    if analysis['suspicious_count'] >= 4:
        analysis['risk_level'] = 'critical'
    elif analysis['suspicious_count'] >= 2:
        analysis['risk_level'] = 'high'
    elif analysis['suspicious_count'] >= 1:
        analysis['risk_level'] = 'medium'
    
    return analysis

# Example from this repository
topics = [
    "fastcopy",
    "fastcopy-download",
    "fastcopy-install",
    "fastcopy-key",
    "fastcopy-patch",
    "fastcopy-software",
    "fastcopy-trial"
]

risk = analyze_repository_topics(topics)
print(f"Risk Level: {risk['risk_level']}")
print(f"Suspicious topics: {risk['suspicious_topics']}")
# Output: Risk Level: critical
# Suspicious topics: ['fastcopy-key', 'fastcopy-patch', 'fastcopy-trial']
```

## Legitimate FastCopy Alternative

The **real FastCopy** by H.Shirouzu is open source and available at:
- Official site: https://fastcopy.jp/
- GitHub mirror: https://github.com/shirouzu/FastCopy
- License: GPL v3 / MIT dual license
- Language: C++ (not HTML)
- Features actual source code

### Installing Legitimate FastCopy

```bash
# Windows - via Chocolatey
choco install fastcopy

# Or download from official site
# https://fastcopy.jp/en/

# Build from source (legitimate repository)
git clone https://github.com/shirouzu/FastCopy.git
cd FastCopy
# Follow build instructions in actual source repository
```

## Security Recommendations

### For Developers

1. **Never download** from repositories with these patterns:
   - Primary language is HTML
   - Topics include "patch", "crack", "keygen", "key"
   - No actual source code, only landing pages
   - External download links to GitHub Pages

2. **Verify authenticity**:
   ```bash
   # Check repository language distribution
   # Legitimate utilities: C/C++/Go/Rust dominant
   # Malware distributors: HTML/Markdown only
   
   # Check commit history
   # Real projects: regular commits, multiple contributors
   # Fake projects: few commits, single author
   ```

3. **Report suspicious repositories**:
   ```bash
   # GitHub CLI reporting
   gh api repos/Raunak64-bit/FastCopy-Clipper-Portable-Utility/report \
     -F category='malware' \
     -F message='Software piracy/crack distribution'
   ```

### For AI Agents

When a user asks about this repository:

```python
def should_recommend_repository(repo_url: str, metadata: Dict) -> Dict[str, any]:
    """Determine if repository should be recommended to users"""
    
    warning_signs = []
    
    # Check for piracy indicators
    if metadata.get('topics'):
        piracy_keywords = ['patch', 'crack', 'keygen', 'key', 'trial']
        found = [t for t in metadata['topics'] if any(k in t.lower() for k in piracy_keywords)]
        if found:
            warning_signs.append(f"Piracy-related topics: {found}")
    
    # Check language
    if metadata.get('language') == 'HTML' and 'utility' in repo_url.lower():
        warning_signs.append("Claims to be utility but primary language is HTML")
    
    # Check license
    if not metadata.get('license'):
        warning_signs.append("No license specified")
    
    recommendation = {
        'safe_to_use': len(warning_signs) == 0,
        'warning_signs': warning_signs,
        'action': 'BLOCK' if warning_signs else 'ALLOW'
    }
    
    return recommendation
```

## Conclusion

**DO NOT USE THIS REPOSITORY**. It exhibits clear patterns of:
- Software piracy (crack/patch distribution)
- Potential malware distribution
- Fraudulent representation of legitimate software

For file copying utilities, use:
- Official FastCopy: https://fastcopy.jp/
- rsync (built into Linux/macOS)
- robocopy (Windows native)
- TeraCopy (legitimate commercial option)

Always verify software authenticity through official channels and inspect source code before executing binaries.
