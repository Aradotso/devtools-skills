```yaml
---
name: minecraft-cheat-client-detection
description: Detect and analyze potentially malicious Minecraft cheat client repositories
triggers:
  - analyze this minecraft client repository
  - check if this is a cheat client scam
  - identify minecraft malware repository
  - detect fake minecraft mod installer
  - investigate suspicious minecraft project
  - scan minecraft client for malware indicators
  - verify minecraft mod legitimacy
  - check repository for cheat client patterns
---

# Minecraft Cheat Client Detection

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

This skill helps identify potentially malicious Minecraft "cheat client" repositories that disguise malware as legitimate game modifications. These repositories typically use SEO-optimized names, fake feature lists, and executable downloads that may contain trojans, keyloggers, or other malicious payloads.

## Warning Signs

### Repository Indicators

**Red Flags:**
- Created date in the future (e.g., "2026-05-01" when current year is earlier)
- Artificially inflated star counts with suspicious growth rates
- Topics focused on "hacks," "cheats," "free accounts," or cracked clients
- Description filled with keyword stuffing and repeated terms
- Generic or stolen README content
- Executable files (.exe) as primary deliverable
- No actual source code matching the claimed language
- Misleading language designation (claims C++ but contains no code)

### Content Patterns

```cpp
// Legitimate C++ Minecraft mod would have actual code like:
#include <minecraft/mod_loader.h>
#include <jni.h>

namespace MyMod {
    void onLoad() {
        // Actual mod initialization
    }
}
```

**Malicious repositories lack real implementation and instead provide:**
- Only README files with download buttons
- Links to external executable files
- No build system (CMakeLists.txt, Makefile, etc.)
- No dependencies or libraries referenced

## Detection Checklist

### Automated Analysis

```python
import os
import json
from datetime import datetime

def analyze_minecraft_repo(repo_path):
    """Analyze repository for cheat client malware indicators."""
    
    indicators = {
        "suspicious": False,
        "warnings": [],
        "risk_score": 0
    }
    
    # Check for future dates
    metadata_file = os.path.join(repo_path, "metadata.json")
    if os.path.exists(metadata_file):
        with open(metadata_file) as f:
            meta = json.load(f)
            created = datetime.fromisoformat(meta.get("created_at", "").replace("Z", ""))
            if created > datetime.now():
                indicators["warnings"].append("Future creation date detected")
                indicators["risk_score"] += 30
    
    # Check for executable downloads in README
    readme_file = os.path.join(repo_path, "README.md")
    if os.path.exists(readme_file):
        with open(readme_file) as f:
            content = f.read().lower()
            if ".exe" in content and "download" in content:
                indicators["warnings"].append("Executable download link found")
                indicators["risk_score"] += 25
            
            # Check for cheat-related keywords
            cheat_keywords = ["hack", "killaura", "esp", "free-account", "cracked"]
            found_keywords = [kw for kw in cheat_keywords if kw in content]
            if len(found_keywords) >= 3:
                indicators["warnings"].append(f"Multiple cheat keywords: {found_keywords}")
                indicators["risk_score"] += 20
    
    # Check for actual source code
    code_files = []
    for ext in [".cpp", ".java", ".py", ".c", ".h"]:
        code_files.extend(list(Path(repo_path).rglob(f"*{ext}")))
    
    if len(code_files) == 0:
        indicators["warnings"].append("No source code files found")
        indicators["risk_score"] += 25
    
    indicators["suspicious"] = indicators["risk_score"] >= 50
    
    return indicators

# Usage
result = analyze_minecraft_repo("./suspected-repo")
if result["suspicious"]:
    print("⚠️  HIGH RISK: This repository shows multiple malware indicators")
    for warning in result["warnings"]:
        print(f"  - {warning}")
```

### Manual Review Steps

1. **Check Repository Age vs. Claimed Date**
   ```bash
   # Compare metadata dates with current date
   git log --reverse --format="%ai" | head -1
   ```

2. **Search for Actual Implementation**
   ```bash
   # Look for real source files
   find . -name "*.cpp" -o -name "*.java" -o -name "*.c"
   
   # Check if files have meaningful content
   wc -l **/*.cpp
   ```

3. **Inspect Download Links**
   ```bash
   # Extract URLs from README
   grep -oP 'https?://[^\s\)]+' README.md
   
   # Check if they point to suspicious domains
   grep -i "releases/tag\|download\|.exe" README.md
   ```

4. **Verify Language Claims**
   ```bash
   # Use linguist or similar to check actual language distribution
   github-linguist --breakdown
   ```

## Common Malware Patterns

### Fake Mod Installer Pattern

```markdown
# ❌ MALICIOUS PATTERN:
## Installation (Easy as 1-2-3)
1. Download: Click the button below to get Setup.exe
2. Install: Open the file and follow setup
3. Start: Launch and click "Activate"

[DOWNLOAD - Installer](suspicious-link.exe)
```

### Legitimate Mod Pattern

```java
// ✅ LEGITIMATE PATTERN:
// build.gradle
plugins {
    id 'fabric-loom' version '1.0'
}

dependencies {
    minecraft "com.mojang:minecraft:1.20.1"
    mappings "net.fabricmc:yarn:1.20.1+build.10"
    modImplementation "net.fabricmc:fabric-loader:0.14.21"
}

// src/main/java/mod/MyMod.java
package mod;

import net.fabricmc.api.ModInitializer;

public class MyMod implements ModInitializer {
    @Override
    public void onInitialize() {
        System.out.println("Mod initialized!");
    }
}
```

## Reporting

### GitHub Security Report

If you identify a malicious repository:

1. Navigate to the repository
2. Click "Security" tab
3. Click "Report a vulnerability" or use GitHub's abuse report
4. Provide evidence using this template:

```
Subject: Malware Distribution - Fake Minecraft Client

Evidence:
- No actual source code present despite claiming C++ language
- Future creation date (metadata manipulation)
- Direct .exe download links in README
- Topics include "hack", "killaura", "free-account"
- SEO keyword stuffing in description
- No legitimate build system or dependencies

Risk: High - Potential trojan/keylogger distribution
```

### Automated Reporting Script

```python
import requests
import os

def report_malicious_repo(repo_full_name, evidence):
    """Report repository to GitHub abuse team."""
    
    # Use GitHub's abuse API (requires authentication)
    headers = {
        "Authorization": f"token {os.getenv('GITHUB_TOKEN')}",
        "Accept": "application/vnd.github.v3+json"
    }
    
    report_data = {
        "repository": repo_full_name,
        "category": "malware",
        "evidence": evidence,
        "severity": "high"
    }
    
    # Note: This is a conceptual example
    # Actual reporting should be done through GitHub's web interface
    print(f"Report prepared for {repo_full_name}")
    print(f"Evidence: {evidence}")
    print("Please submit through: https://github.com/contact/report-abuse")

# Usage
report_malicious_repo(
    "username/vapev4-client-2026",
    "Fake Minecraft client with no source code, exe downloads, and future dates"
)
```

## Prevention for Users

### Safe Minecraft Modding Practices

```bash
# ✅ Always use official mod sources:
# - CurseForge (https://www.curseforge.com/minecraft)
# - Modrinth (https://modrinth.com/)
# - Official Fabric/Forge repositories

# ❌ NEVER download .exe files claiming to be mods
# ❌ NEVER trust repositories with:
#    - No source code
#    - "Free account" or "hack" in the name
#    - Suspicious download links
```

### Legitimate Mod Development

```java
// Real Fabric mod structure:
// fabric.mod.json
{
  "schemaVersion": 1,
  "id": "mymod",
  "version": "1.0.0",
  "name": "My Mod",
  "entrypoints": {
    "main": ["com.example.MyMod"]
  },
  "depends": {
    "fabricloader": ">=0.14.0",
    "minecraft": "~1.20.1"
  }
}

// src/main/java/com/example/MyMod.java
package com.example;

import net.fabricmc.api.ModInitializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyMod implements ModInitializer {
    public static final Logger LOGGER = LoggerFactory.getLogger("mymod");

    @Override
    public void onInitialize() {
        LOGGER.info("My mod initialized safely!");
    }
}
```

## Conclusion

This skill equips AI agents to identify and warn developers about malicious Minecraft "cheat client" repositories. Always verify legitimacy through official sources and never download executable files from untrusted GitHub repositories.
```
