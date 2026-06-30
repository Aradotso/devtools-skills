```markdown
---
name: game-cheat-detection-analysis
description: Analyze and detect game cheating tools, trainers, and memory manipulation attempts for security research
triggers:
  - how do I detect game cheating software
  - analyze this game trainer for malware
  - identify memory manipulation in game cheats
  - detect wallhack or aimbot patterns
  - review game security vulnerabilities
  - scan for game trainer malware
  - analyze cheat tool behavior
  - identify game exploit techniques
---

# Game Cheat Detection Analysis

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Critical Security Warning

**This project (MECCHA-CHAMELEON-Trainer-Client) is a malicious game cheating tool that:**

1. **Violates game Terms of Service** and will result in permanent bans
2. **Contains high malware risk** - downloads executable from external site
3. **Requests admin privileges** - dangerous for system security
4. **Uses obfuscation techniques** typical of malware distribution
5. **Targets multiplayer games** - ruins experience for legitimate players

## Security Analysis Pattern

### Red Flags in Cheat Software

```python
# NEVER RUN - Example of malicious pattern detection
suspicious_indicators = {
    "external_download": "https://skydock.netlify.app/trainer-archive.zip",
    "admin_required": True,
    "obfuscated_payload": "trainer-archive.zip with password",
    "memory_manipulation": ["ESP", "Aimbot", "God Mode", "NoClip"],
    "anti_detection_claims": "Undetected",
    "rapid_stars": "92 stars/day (fake engagement)"
}

# These are hallmarks of malware distribution
```

### Common Cheat Tool Techniques (For Defense)

```python
# Educational: How game cheats typically work (for defense research)

class CheatDetectionSignatures:
    """Patterns to detect in security research"""
    
    MEMORY_PATTERNS = {
        "esp_wallhack": "Direct memory read of player positions",
        "aimbot": "Mouse input injection + angle calculations",
        "god_mode": "Health value memory write protection",
        "speed_hack": "Velocity multiplier memory writes",
        "noclip": "Collision detection bypass"
    }
    
    PROCESS_INDICATORS = [
        "ReadProcessMemory",
        "WriteProcessMemory",
        "CreateRemoteThread",
        "SetWindowsHookEx",
        "Mouse_event injection"
    ]
    
    ANTI_CHEAT_EVASION = [
        "Kernel driver loading",
        "Memory signature randomization",
        "Delayed execution",
        "External process injection"
    ]
```

## Legitimate Security Research

### Game Anti-Cheat Development

```python
import psutil
import hashlib
import os

class AntiCheatMonitor:
    """Legitimate game protection system"""
    
    def __init__(self):
        self.suspicious_processes = []
        self.known_cheat_hashes = self.load_cheat_signatures()
    
    def load_cheat_signatures(self):
        """Load known cheat tool signatures"""
        # In production, load from secure database
        return {
            "trainer_archives": ["trainer-archive.zip"],
            "suspicious_domains": ["skydock.netlify.app"],
            "common_names": ["trainer.exe", "injector.exe"]
        }
    
    def scan_running_processes(self):
        """Monitor for suspicious process activity"""
        suspicious = []
        
        for proc in psutil.process_iter(['name', 'exe', 'cmdline']):
            try:
                # Check for memory manipulation tools
                if self.is_memory_tool(proc.info['name']):
                    suspicious.append({
                        'name': proc.info['name'],
                        'pid': proc.pid,
                        'reason': 'Known memory manipulation tool'
                    })
                
                # Check for debugger attachment
                if self.has_debugger_attached(proc.pid):
                    suspicious.append({
                        'name': proc.info['name'],
                        'pid': proc.pid,
                        'reason': 'Debugger detected'
                    })
                    
            except (psutil.NoSuchProcess, psutil.AccessDenied):
                continue
        
        return suspicious
    
    def is_memory_tool(self, process_name):
        """Identify memory manipulation tools"""
        memory_tools = [
            'cheatengine', 'trainer', 'injector', 
            'memoryeditor', 'processhacker'
        ]
        return any(tool in process_name.lower() for tool in memory_tools)
    
    def has_debugger_attached(self, pid):
        """Check if process has debugger"""
        try:
            import ctypes
            kernel32 = ctypes.windll.kernel32
            return kernel32.CheckRemoteDebuggerPresent(pid, None)
        except:
            return False
    
    def verify_file_integrity(self, game_files):
        """Hash verification for game files"""
        for file_path in game_files:
            if os.path.exists(file_path):
                with open(file_path, 'rb') as f:
                    file_hash = hashlib.sha256(f.read()).hexdigest()
                    # Compare with known good hashes
                    if not self.verify_hash(file_hash):
                        return False, f"Modified: {file_path}"
        return True, "All files intact"
    
    def verify_hash(self, file_hash):
        """Verify against known good hashes"""
        # In production: check against secure hash database
        return True

# Usage in game protection
monitor = AntiCheatMonitor()
suspicious = monitor.scan_running_processes()

if suspicious:
    print("⚠️ Suspicious activity detected:")
    for item in suspicious:
        print(f"  - {item['name']} (PID: {item['pid']}): {item['reason']}")
```

### Malware Analysis Framework

```python
import zipfile
import yara
import pefile

class CheatMalwareAnalyzer:
    """Analyze suspected cheat tools for malware"""
    
    def __init__(self):
        self.yara_rules = self.compile_rules()
    
    def compile_rules(self):
        """Compile YARA rules for cheat detection"""
        rules = """
        rule GameCheatTrainer {
            meta:
                description = "Detects game trainer patterns"
            strings:
                $esp = "ESP" ascii
                $aimbot = "aimbot" ascii nocase
                $godmode = "god mode" ascii nocase
                $memory = "ReadProcessMemory" ascii
                $inject = "WriteProcessMemory" ascii
            condition:
                3 of them
        }
        
        rule SuspiciousDownloader {
            meta:
                description = "Detects downloader trojans"
            strings:
                $url = /https?:\/\/[a-z0-9\-\.]+\/[a-z0-9\-_]+\.zip/
                $admin = "Run as Administrator" ascii
                $password = "password:" ascii
            condition:
                all of them
        }
        """
        return yara.compile(source=rules)
    
    def analyze_archive(self, archive_path):
        """Analyze suspicious archive"""
        findings = {
            "password_protected": False,
            "executables": [],
            "scripts": [],
            "risk_level": "UNKNOWN"
        }
        
        try:
            with zipfile.ZipFile(archive_path) as zf:
                # Check if password protected
                for info in zf.infolist():
                    if info.flag_bits & 0x1:
                        findings["password_protected"] = True
                        findings["risk_level"] = "HIGH"
                        break
                
                # List contents
                for name in zf.namelist():
                    if name.endswith(('.exe', '.dll', '.sys')):
                        findings["executables"].append(name)
                    elif name.endswith(('.py', '.vbs', '.ps1')):
                        findings["scripts"].append(name)
        
        except Exception as e:
            findings["error"] = str(e)
        
        return findings
    
    def scan_executable(self, exe_path):
        """Scan executable for malicious patterns"""
        results = {"yara_matches": [], "pe_info": {}}
        
        # YARA scan
        matches = self.yara_rules.match(exe_path)
        results["yara_matches"] = [m.rule for m in matches]
        
        # PE analysis
        try:
            pe = pefile.PE(exe_path)
            results["pe_info"] = {
                "imports": [imp.name.decode() for imp in pe.DIRECTORY_ENTRY_IMPORT],
                "sections": [s.Name.decode().strip('\x00') for s in pe.sections],
                "is_packed": self.detect_packing(pe)
            }
        except:
            results["pe_info"]["error"] = "Unable to parse PE"
        
        return results
    
    def detect_packing(self, pe):
        """Detect if executable is packed/obfuscated"""
        suspicious_sections = ['.upx', '.aspack', '.packed']
        for section in pe.sections:
            name = section.Name.decode().strip('\x00').lower()
            if any(packed in name for packed in suspicious_sections):
                return True
        return False

# Example usage
analyzer = CheatMalwareAnalyzer()

# Analyze the suspicious download
# findings = analyzer.analyze_archive("trainer-archive.zip")
# print(f"Risk Level: {findings['risk_level']}")
```

## Ethical Security Research Guidelines

### DO: Legitimate Anti-Cheat Development

```python
# Proper game security implementation
class GameSecurityService:
    """Legitimate game protection service"""
    
    def __init__(self, game_process_name):
        self.game_process = game_process_name
        self.baseline_memory = None
    
    def establish_baseline(self):
        """Create integrity baseline"""
        # Hash game files
        # Record normal memory patterns
        # Store network traffic signature
        pass
    
    def continuous_monitoring(self):
        """Monitor for tampering"""
        # Check file integrity
        # Monitor process injection
        # Detect memory manipulation
        # Verify network packets
        pass
    
    def report_violation(self, violation_type, evidence):
        """Securely report cheating"""
        # Log to secure backend
        # Include evidence hash
        # Protect user privacy
        pass
```

### DON'T: Use or Distribute Cheats

```python
# ❌ NEVER DO THIS - Example of what NOT to do
# This is what the malicious project does:

# class CheatTool:  # ILLEGAL & UNETHICAL
#     def inject_esp(self): ...      # Violates ToS, bans account
#     def enable_aimbot(self): ...   # Ruins multiplayer experience
#     def god_mode(self): ...        # Unfair advantage
#     def download_payload(self): ...  # Malware risk

# Legal consequences:
# - Account termination
# - Hardware bans
# - Legal action from game publishers
# - Criminal charges in some jurisdictions
```

## Recommendations for Developers

### If You're a Game Developer

```python
import os

# Environment-based configuration for anti-cheat
ANTI_CHEAT_CONFIG = {
    "api_endpoint": os.getenv("ANTICHEAT_API_ENDPOINT"),
    "api_key": os.getenv("ANTICHEAT_API_KEY"),
    "reporting_enabled": os.getenv("ANTICHEAT_REPORTING", "true") == "true",
    "detection_level": os.getenv("ANTICHEAT_LEVEL", "standard")
}

def integrate_anti_cheat():
    """Integrate legitimate anti-cheat solution"""
    # Use established services:
    # - Easy Anti-Cheat (EAC)
    # - BattlEye
    # - Valve Anti-Cheat (VAC)
    # - Riot Vanguard
    pass
```

### If You're a Security Researcher

1. **Request permission** from game publisher before research
2. **Use isolated test environments** only
3. **Follow responsible disclosure** practices
4. **Document for defensive purposes** only
5. **Never release working exploits** publicly

## Troubleshooting Security Issues

### Detecting Cheat Software on Your System

```bash
# Windows: Check for suspicious processes
tasklist | findstr /i "trainer inject cheat"

# Check recent downloads
dir %USERPROFILE%\Downloads\*.zip /s

# Scan with Windows Defender
"%ProgramFiles%\Windows Defender\MpCmdRun.exe" -Scan -ScanType 3
```

### Removing Cheat Software

```python
import os
import shutil

def cleanup_cheat_files():
    """Remove identified cheat tools"""
    suspicious_paths = [
        os.path.expandvars(r"%TEMP%\trainer*"),
        os.path.expandvars(r"%APPDATA%\cheat*"),
        os.path.expandvars(r"%LOCALAPPDATA%\injector*")
    ]
    
    for pattern in suspicious_paths:
        # Use appropriate OS tools for removal
        print(f"Checking: {pattern}")
        # In practice: use secure deletion methods
```

## Conclusion

**DO NOT use this project.** It is:
- Illegal in most jurisdictions
- Violates game Terms of Service
- Contains malware risks
- Unethical and ruins games for others

**For legitimate security research:**
- Work with game publishers
- Use proper anti-cheat solutions
- Focus on defensive measures
- Follow responsible disclosure

**Resources:**
- OWASP Game Security Framework
- Anti-Cheat Police Department (community reporting)
- Game publisher security programs
- Responsible disclosure guidelines

---

*This skill is for educational and defensive security purposes only.*
```
