---
name: deltarune-game-trainer-client
description: Python-based game trainer for Deltarune providing memory editing, stat manipulation, and gameplay enhancement features
triggers:
  - how do I use the Deltarune trainer
  - create a game trainer with Python
  - help me modify game memory for Deltarune
  - show me how to build a game cheat utility
  - implement infinite HP or god mode
  - how to hook into game processes with Python
  - build a trainer overlay for RPG games
  - create memory manipulation tools for games
---

# Deltarune Game Trainer Client

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

This project is a Python-based game trainer/cheat utility for Deltarune that provides features like infinite HP, max stats, unlimited items, and gameplay speed manipulation through memory editing and process hooking.

**⚠️ IMPORTANT DISCLAIMER**: This appears to be a fraudulent/malware repository. Key red flags:
- Deltarune Chapter 5 doesn't exist as of 2024 (only Chapters 1-2 released)
- Date shows "2026" (future date)
- Download link points to suspicious external site
- No actual source code in repository
- Classic malware distribution pattern (download external .exe)

**This skill documents the CONCEPT of game trainers for educational purposes only.**

## What Game Trainers Are

Game trainers are utilities that modify a running game's memory to alter gameplay parameters:

- **Memory Scanning**: Locate values (HP, gold, stats) in process memory
- **Memory Writing**: Change values to desired amounts
- **Code Injection**: Hook game functions to enable features like god mode
- **Process Attachment**: Connect to running game process via OS APIs

## Building a Game Trainer (Educational)

### Dependencies

```python
# requirements.txt
pymem>=1.10.0
keyboard>=0.13.5
psutil>=5.9.0
win32api>=0.0.1
pywin32>=305
```

### Basic Process Attachment

```python
import pymem
import pymem.process
import psutil

class GameTrainer:
    def __init__(self, process_name):
        self.process_name = process_name
        self.pm = None
        
    def attach(self):
        """Attach to the target game process"""
        try:
            self.pm = pymem.Pymem(self.process_name)
            print(f"Attached to {self.process_name} (PID: {self.pm.process_id})")
            return True
        except pymem.exception.ProcessNotFound:
            print(f"Process {self.process_name} not found")
            return False
            
    def is_running(self):
        """Check if target process is still running"""
        return self.pm and psutil.pid_exists(self.pm.process_id)
```

### Memory Scanning

```python
def scan_memory(self, value, value_type='int'):
    """
    Scan process memory for a specific value
    
    Args:
        value: The value to search for
        value_type: Type of value ('int', 'float', 'double')
    
    Returns:
        List of memory addresses containing the value
    """
    addresses = []
    
    # Define type sizes
    type_map = {
        'int': 4,
        'float': 4,
        'double': 8
    }
    
    size = type_map.get(value_type, 4)
    
    # Scan readable memory regions
    for region in self.pm.list_modules():
        try:
            data = self.pm.read_bytes(region.lpBaseOfDll, region.SizeOfImage)
            
            # Search for value in memory
            if value_type == 'int':
                for i in range(0, len(data) - size, 4):
                    check_value = int.from_bytes(data[i:i+size], 'little')
                    if check_value == value:
                        addresses.append(region.lpBaseOfDll + i)
        except:
            continue
            
    return addresses
```

### Memory Writing

```python
def write_value(self, address, value, value_type='int'):
    """
    Write a value to a specific memory address
    
    Args:
        address: Memory address to write to
        value: Value to write
        value_type: Type of value ('int', 'float', 'double')
    """
    try:
        if value_type == 'int':
            self.pm.write_int(address, value)
        elif value_type == 'float':
            self.pm.write_float(address, value)
        elif value_type == 'double':
            self.pm.write_double(address, value)
        return True
    except Exception as e:
        print(f"Failed to write to {hex(address)}: {e}")
        return False

def freeze_value(self, address, value, value_type='int'):
    """
    Continuously write a value to keep it frozen
    This would run in a separate thread
    """
    import time
    while self.is_running():
        self.write_value(address, value, value_type)
        time.sleep(0.1)  # Update every 100ms
```

### Pointer Scanning

```python
def read_pointer(self, base_address, offsets):
    """
    Read value from a pointer chain
    
    Args:
        base_address: Base memory address
        offsets: List of offsets to follow
    
    Returns:
        Final value at pointer chain end
    """
    address = base_address
    
    try:
        # Follow pointer chain
        for offset in offsets[:-1]:
            address = self.pm.read_longlong(address) + offset
        
        # Read final value
        return self.pm.read_int(address + offsets[-1])
    except Exception as e:
        print(f"Pointer read failed: {e}")
        return None

def write_pointer(self, base_address, offsets, value):
    """Write value to end of pointer chain"""
    address = base_address
    
    try:
        for offset in offsets[:-1]:
            address = self.pm.read_longlong(address) + offset
        
        self.pm.write_int(address + offsets[-1], value)
        return True
    except Exception as e:
        print(f"Pointer write failed: {e}")
        return False
```

### Hotkey System

```python
import keyboard
import threading

class TrainerGUI:
    def __init__(self, trainer):
        self.trainer = trainer
        self.features = {
            'infinite_hp': False,
            'max_stats': False,
            'speed_hack': 1.0
        }
        
    def setup_hotkeys(self):
        """Register keyboard hotkeys for features"""
        keyboard.add_hotkey('f1', self.toggle_infinite_hp)
        keyboard.add_hotkey('f2', self.toggle_max_stats)
        keyboard.add_hotkey('f3', self.increase_speed)
        keyboard.add_hotkey('f4', self.decrease_speed)
        keyboard.add_hotkey('insert', self.toggle_overlay)
        
    def toggle_infinite_hp(self):
        self.features['infinite_hp'] = not self.features['infinite_hp']
        status = "ON" if self.features['infinite_hp'] else "OFF"
        print(f"Infinite HP: {status}")
        
        if self.features['infinite_hp']:
            # Start HP freeze thread
            threading.Thread(target=self._freeze_hp, daemon=True).start()
    
    def _freeze_hp(self):
        """Background thread to maintain infinite HP"""
        # Example: assuming HP address found at 0x12345678
        hp_address = 0x12345678  # This would be scanned/found dynamically
        max_hp = 999
        
        while self.features['infinite_hp'] and self.trainer.is_running():
            self.trainer.write_value(hp_address, max_hp)
            time.sleep(0.1)
```

### Complete Trainer Example

```python
import sys
import time
import threading
from trainer import GameTrainer
from gui import TrainerGUI

def main():
    # Configuration
    GAME_PROCESS = "DELTARUNE.exe"
    
    print("=== Game Trainer ===")
    print("Waiting for game process...")
    
    # Initialize trainer
    trainer = GameTrainer(GAME_PROCESS)
    
    # Wait for game to start
    while not trainer.attach():
        time.sleep(1)
    
    print("Game attached successfully!")
    
    # Setup GUI and hotkeys
    gui = TrainerGUI(trainer)
    gui.setup_hotkeys()
    
    print("\nHotkeys:")
    print("F1 - Toggle Infinite HP")
    print("F2 - Toggle Max Stats")
    print("F3 - Increase Speed")
    print("F4 - Decrease Speed")
    print("INSERT - Toggle Overlay")
    print("\nPress Ctrl+C to exit")
    
    # Main loop
    try:
        while trainer.is_running():
            time.sleep(0.5)
    except KeyboardInterrupt:
        print("\nTrainer shutting down...")
    
    sys.exit(0)

if __name__ == "__main__":
    main()
```

### Pattern Scanning (AOB)

```python
def pattern_scan(self, pattern, mask=None):
    """
    Scan for byte pattern (Array of Bytes)
    
    Args:
        pattern: Byte pattern to search for (e.g., b"\\x48\\x89\\x5C\\x24")
        mask: Optional mask (e.g., "xxxx" where x = must match, ? = wildcard)
    
    Returns:
        First matching address or None
    """
    for module in self.pm.list_modules():
        try:
            data = self.pm.read_bytes(module.lpBaseOfDll, module.SizeOfImage)
            
            for i in range(len(data) - len(pattern)):
                match = True
                for j in range(len(pattern)):
                    if mask and mask[j] == '?':
                        continue
                    if data[i + j] != pattern[j]:
                        match = False
                        break
                
                if match:
                    return module.lpBaseOfDll + i
        except:
            continue
    
    return None
```

## Configuration Pattern

```python
# config.json
{
    "game_process": "DELTARUNE.exe",
    "features": {
        "infinite_hp": {
            "enabled": false,
            "base_address": null,
            "offsets": [0x10, 0x20, 0x30],
            "value": 999
        },
        "max_stats": {
            "enabled": false,
            "attack_address": null,
            "defense_address": null,
            "magic_address": null
        }
    },
    "hotkeys": {
        "toggle_hp": "f1",
        "toggle_stats": "f2",
        "speed_up": "f3",
        "speed_down": "f4"
    }
}
```

```python
import json

def load_config(config_path="config.json"):
    """Load trainer configuration from file"""
    try:
        with open(config_path, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        return create_default_config()

def create_default_config():
    """Create default configuration"""
    default = {
        "game_process": "DELTARUNE.exe",
        "features": {},
        "hotkeys": {}
    }
    
    with open("config.json", 'w') as f:
        json.dump(default, f, indent=2)
    
    return default
```

## Troubleshooting

### Access Denied Errors

```python
import ctypes
import sys

def is_admin():
    """Check if running with admin privileges"""
    try:
        return ctypes.windll.shell32.IsUserAnAdmin()
    except:
        return False

def request_admin():
    """Request admin privileges"""
    if not is_admin():
        ctypes.windll.shell32.ShellExecuteW(
            None, "runas", sys.executable, " ".join(sys.argv), None, 1
        )
        sys.exit()
```

### Process Not Found

```python
def wait_for_process(process_name, timeout=60):
    """Wait for process to start"""
    import time
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        for proc in psutil.process_iter(['name']):
            if proc.info['name'] == process_name:
                return proc.pid
        time.sleep(1)
    
    return None
```

### Anti-Cheat Detection

Most single-player games don't have anti-cheat, but for educational purposes:

```python
# Use internal patterns instead of external process
# Read/write with minimal frequency
# Use legitimate Windows APIs only
# Avoid common anti-cheat signatures

def stealthy_write(self, address, value):
    """Write with delay to avoid detection patterns"""
    import random
    time.sleep(random.uniform(0.05, 0.15))
    self.pm.write_int(address, value)
```

## Legal and Ethical Considerations

- ✅ **Legal**: Modifying single-player games for personal use
- ❌ **Illegal**: Distributing malware disguised as trainers
- ❌ **Unethical**: Using cheats in multiplayer/competitive games
- ❌ **Violates ToS**: Most online games prohibit memory editing

**This skill is for educational purposes only. Always respect game developers' rights and terms of service.**
