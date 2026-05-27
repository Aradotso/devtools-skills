---
name: minecraft-vape-client-detection
description: Detect and analyze potential Minecraft cheat client repositories masquerading as legitimate tools
triggers:
  - analyze this minecraft mod repository
  - check if this minecraft client is legitimate
  - identify cheat client characteristics
  - detect vape client distribution repo
  - scan for minecraft hack indicators
  - validate minecraft mod authenticity
---

# Minecraft Vape Client Detection

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ WARNING: Malicious Repository Detected

This repository exhibits **multiple red flags** indicating it is a malware distribution or cheat client scam:

### 🚨 Critical Indicators

1. **Deceptive Description**: Claims to be "Vape V4" (a known Minecraft cheat client) while masquerading as a legitimate mod manager
2. **Suspicious Topics**: Tags include `minecraft-killaura`, `minecraft-esp`, `vape-v4-hack`, `vape-v4-free-account` - all cheat/hack related
3. **Download Bait**: README promotes downloading `.exe` installer from releases
4. **Language Mismatch**: Repository claims C++ but likely contains only binary executables
5. **Generic Content**: README has no actual code, documentation, or legitimate project structure
6. **High Star Velocity**: 13 stars/day on a new repo suggests artificial manipulation
7. **Future Date**: Created "2026-05-01" indicates timestamp manipulation or test data

## What This Actually Is

This is NOT a legitimate Minecraft mod manager. This pattern matches:

- **Cheat client distribution** (Vape is a paid PvP cheat client)
- **Malware/RAT distribution** disguised as game hacks
- **Phishing for account credentials**
- **Cryptocurrency miners** bundled in "mod installers"

## Detection Patterns

### Repository Structure Red Flags

```python
RED_FLAGS = {
    "topics": [
        "killaura",  # Auto-attack hack
        "esp",       # Wallhack/player detection
        "hack",
        "free-account",
        "cracked"
    ],
    "file_patterns": [
        "*Setup.exe",
        "*Installer.exe",
        "*Crack.exe",
        "password.txt"
    ],
    "no_source_code": True,
    "high_star_velocity": ">10/day",
    "releases_only": True
}
```

### Language Analysis

```cpp
// Legitimate C++ Minecraft mod would show:
// - CMakeLists.txt or build configuration
// - .cpp/.h source files
// - JNI bindings for Java interop
// - Gradle or Maven integration

// This repo likely has NONE of these
```

## Safe Minecraft Modding Alternatives

### Legitimate Mod Loaders

```bash
# Fabric (lightweight, modern)
curl -OJ https://maven.fabricmc.net/net/fabricmc/fabric-installer/latest/fabric-installer-latest.jar
java -jar fabric-installer-latest.jar client

# Forge (established, extensive mod support)
# Download from official site: files.minecraftforge.net
```

### Performance Optimization (Legitimate)

```java
// OptiFine (visual optimization)
// Sodium (rendering engine - Fabric)
// Phosphor (lighting engine)
// Lithium (general optimization)

// Install via mod loader, NOT standalone executables
```

## Security Analysis Commands

### Check Repository Authenticity

```bash
# Clone and scan (NEVER run executables)
git clone <repo-url> --depth 1
cd <repo-name>

# Look for source code
find . -name "*.cpp" -o -name "*.h" -o -name "*.java"

# Check for executables (DO NOT RUN)
find . -name "*.exe" -o -name "*.dll" -o -name "*.so"

# Analyze commit history
git log --all --oneline --graph
```

### File Hash Verification

```python
import hashlib
import os

def check_file_safety(filepath):
    """Never execute unknown binaries - scan only"""
    if filepath.endswith('.exe'):
        print(f"⚠️  EXECUTABLE DETECTED: {filepath}")
        print("DO NOT RUN - Likely malware")
        
        # Calculate hash for VirusTotal lookup
        with open(filepath, 'rb') as f:
            file_hash = hashlib.sha256(f.read()).hexdigest()
        print(f"SHA256: {file_hash}")
        print(f"Check: https://www.virustotal.com/gui/file/{file_hash}")
```

## Protecting Users

### Warning Message Template

```markdown
⚠️ **SECURITY ALERT** ⚠️

This repository distributes CHEAT CLIENT software for Minecraft:
- Violates Minecraft EULA
- Risk of account ban
- Potential malware/virus
- May steal credentials

**DO NOT DOWNLOAD OR RUN ANY FILES**

For legitimate Minecraft mods, use:
- CurseForge: https://www.curseforge.com/minecraft
- Modrinth: https://modrinth.com/
- Official Fabric/Forge sites
```

### Report Commands

```bash
# Report to GitHub
gh repo view <owner>/<repo>
# Use web interface to report malicious content

# Report to Minecraft/Mojang
# https://help.minecraft.net/hc/en-us/requests/new
```

## Educational Context

### Why Cheat Clients Are Harmful

1. **Account Security**: Often contain credential stealers
2. **Computer Security**: Bundled malware, keyloggers, RATs
3. **Game Integrity**: Ruins multiplayer experience
4. **Legal Risk**: Violates Terms of Service, possible DMCA

### Legitimate Mod Development

```java
// Example: Fabric mod entry point (LEGITIMATE)
package com.example.examplemod;

import net.fabricmc.api.ModInitializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ExampleMod implements ModInitializer {
    public static final Logger LOGGER = LoggerFactory.getLogger("examplemod");

    @Override
    public void onInitialize() {
        LOGGER.info("Example mod loaded!");
    }
}
```

```json
// fabric.mod.json (required for legitimate Fabric mods)
{
  "schemaVersion": 1,
  "id": "examplemod",
  "version": "1.0.0",
  "environment": "*",
  "entrypoints": {
    "main": ["com.example.examplemod.ExampleMod"]
  },
  "depends": {
    "fabricloader": ">=0.14.0",
    "minecraft": "1.19.x"
  }
}
```

## Conclusion

**This repository should be avoided entirely.** It represents a common pattern of malicious software distribution disguised as game modifications. AI agents should warn users and suggest legitimate alternatives from trusted sources like CurseForge, Modrinth, or official mod loader sites.
