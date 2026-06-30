---
name: game-cheat-detection-and-analysis
description: Identify, analyze, and document game cheating tools, trainers, and malicious game modification software for security research and anti-cheat development
triggers:
  - analyze this game trainer or cheat tool
  - detect cheat software patterns
  - identify game hacking techniques
  - document anti-cheat evasion methods
  - reverse engineer game trainer
  - security analysis of game modification tools
  - identify malicious game software
  - analyze game exploit code
---

# Game Cheat Detection and Analysis

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Security Warning

This project is **malicious software** disguised as a game trainer. It exhibits all hallmarks of:

- **Malware distribution** (external download link, password-protected archive)
- **Game cheating/hacking tool** (ESP, aimbot, god mode, memory manipulation)
- **Terms of Service violation** for multiplayer games
- **Potential trojan/RAT delivery mechanism**

## What This Actually Is

This repository is a **malware distribution front** using common tactics:

1. **Fake legitimacy**: MIT license, professional README formatting, realistic version badges
2. **External payload**: Links to `skydock.netlify.app` instead of GitHub releases
3. **Password-protected archive**: Bypasses automated malware scanning (`trainer2026`)
4. **Administrator execution**: Required to inject into game process memory
5. **Undetected claims**: Social engineering to disable antivirus

## Security Analysis Patterns

### Red Flags in Game Cheat Repositories

```python
# Common malware indicators to check:
red_flags = {
    "external_download": "https://skydock.netlify.app/trainer-archive.zip",
    "requires_admin": True,
    "password_protected": "trainer2026",
    "no_source_code": True,  # Only .exe download, no Python source
    "unrealistic_stars": "92 stars/day for brand new repo",
    "multiplayer_cheating": ["ESP", "Aimbot", "God Mode"],
    "memory_manipulation": ["NoClip", "Teleport", "Timer Editor"]
}
```

### Detection Methodology

```python
import os
import hashlib
from pathlib import Path

def analyze_cheat_repository(repo_path: str) -> dict:
    """
    Analyze a suspected game cheat repository for malware indicators.
    
    Returns:
        dict: Security assessment with risk level
    """
    indicators = {
        "external_downloads": [],
        "obfuscation": False,
        "source_available": False,
        "risk_level": "UNKNOWN"
    }
    
    # Check for source code presence
    readme_path = Path(repo_path) / "README.md"
    if readme_path.exists():
        with open(readme_path, 'r', encoding='utf-8') as f:
            content = f.read()
            
            # Look for external download links
            if 'http' in content and '.zip' in content:
                indicators["external_downloads"].append(
                    extract_download_urls(content)
                )
            
            # Check for password mentions
            if 'password' in content.lower():
                indicators["obfuscation"] = True
    
    # Verify actual source code exists
    python_files = list(Path(repo_path).rglob("*.py"))
    indicators["source_available"] = len(python_files) > 0
    
    # Calculate risk level
    if not indicators["source_available"] and indicators["external_downloads"]:
        indicators["risk_level"] = "CRITICAL"
    elif indicators["obfuscation"]:
        indicators["risk_level"] = "HIGH"
    
    return indicators

def extract_download_urls(text: str) -> list:
    """Extract download URLs from text."""
    import re
    url_pattern = r'https?://[^\s\)]+\.(?:zip|exe|rar|7z)'
    return re.findall(url_pattern, text)
```

## Anti-Cheat Development Reference

### Memory Manipulation Techniques to Detect

```python
class CheatTechniques:
    """
    Reference implementation of techniques used by game cheats.
    For educational/anti-cheat development purposes only.
    """
    
    ESP_INDICATORS = {
        "reads_player_positions": True,
        "accesses_game_memory": True,
        "draws_overlay": True,
        "typical_process": "ReadProcessMemory API calls"
    }
    
    AIMBOT_INDICATORS = {
        "calculates_angles": True,
        "modifies_view_matrix": True,
        "auto_mouse_movement": True,
        "typical_process": "WriteProcessMemory on view angles"
    }
    
    GOD_MODE_INDICATORS = {
        "freezes_health_value": True,
        "memory_write_loop": True,
        "hooks_damage_function": True
    }

def detect_memory_manipulation(process_name: str) -> list:
    """
    Detect if a process is being manipulated by external tools.
    
    Args:
        process_name: Target game process
        
    Returns:
        list: Detected manipulation techniques
    """
    # This would integrate with anti-cheat SDKs like EasyAntiCheat, BattlEye
    # Example detection logic:
    detections = []
    
    # Check for suspicious handles to game process
    # Check for injected DLLs
    # Monitor for memory region modifications
    # Detect overlay rendering (ESP)
    
    return detections
```

## Responsible Disclosure

### If You Find Game Exploits

```python
import os
from datetime import datetime

def create_security_report(
    game_name: str,
    vulnerability_type: str,
    details: dict,
    output_path: str = None
) -> str:
    """
    Create a responsible disclosure report for game vulnerabilities.
    
    Args:
        game_name: Affected game
        vulnerability_type: Type of exploit (memory manipulation, packet injection, etc.)
        details: Technical details
        output_path: Where to save report
        
    Returns:
        str: Path to generated report
    """
    report = {
        "date": datetime.utcnow().isoformat(),
        "game": game_name,
        "vulnerability": vulnerability_type,
        "severity": details.get("severity", "MEDIUM"),
        "description": details.get("description", ""),
        "reproduction_steps": details.get("steps", []),
        "suggested_mitigation": details.get("mitigation", ""),
        "reporter": os.getenv("SECURITY_RESEARCHER_EMAIL", "anonymous")
    }
    
    if not output_path:
        output_path = f"security_report_{game_name}_{datetime.now().strftime('%Y%m%d')}.json"
    
    import json
    with open(output_path, 'w') as f:
        json.dump(report, f, indent=2)
    
    return output_path

# Example usage:
report = create_security_report(
    game_name="ExampleGame",
    vulnerability_type="Client-side memory manipulation",
    details={
        "severity": "HIGH",
        "description": "Game client trusts local memory values for health/position",
        "steps": [
            "Attach debugger to game process",
            "Locate health value in memory",
            "Freeze value to prevent updates"
        ],
        "mitigation": "Implement server-side validation of all game state"
    }
)
```

## Legitimate Game Modding Alternatives

### Ethical Mod Development

```python
# Example: Creating a legitimate single-player trainer
class LegitimateTrainer:
    """
    Example structure for ethical game modification (single-player only).
    """
    
    def __init__(self, game_name: str):
        self.game = game_name
        self.multiplayer_check()
    
    def multiplayer_check(self):
        """Refuse to run if game is in multiplayer mode."""
        # Check for active network connections
        # Check for anti-cheat presence
        # Verify single-player mode
        pass
    
    def apply_quality_of_life_mods(self):
        """Apply non-competitive modifications."""
        # Examples: FOV slider, colorblind mode, accessibility features
        pass
```

## Legal and Ethical Considerations

**DO NOT:**
- Use cheats in multiplayer games
- Distribute game hacking tools
- Download executables from untrusted sources
- Run software requiring admin rights without source code review
- Violate game Terms of Service or EULAs

**DO:**
- Report exploits to game developers responsibly
- Develop anti-cheat solutions
- Create accessibility mods for single-player games (with permission)
- Research game security in controlled environments
- Respect intellectual property and fair play

## Malware Analysis Workflow

```python
def safe_malware_analysis_environment():
    """
    Set up safe environment for analyzing suspicious game cheats.
    """
    requirements = [
        "Isolated VM (VMware/VirtualBox)",
        "Snapshot capability for rollback",
        "Network isolation/monitoring (Wireshark)",
        "Process monitoring (Process Monitor, Process Hacker)",
        "Static analysis tools (IDA Pro, Ghidra)",
        "Sandbox (Cuckoo, any.run)"
    ]
    
    # Never run suspicious executables on main system
    # Always use disposable VMs
    # Monitor all network traffic
    # Document IOCs (Indicators of Compromise)
    
    return requirements
```

## Conclusion

This repository is a **security threat**, not a legitimate development tool. Use this skill to:

1. **Identify** malware distribution patterns
2. **Analyze** game exploit techniques for defensive purposes
3. **Develop** anti-cheat systems
4. **Report** vulnerabilities responsibly
5. **Educate** about game security threats

Never use cheating tools in multiplayer games. Support fair play and game security research.
