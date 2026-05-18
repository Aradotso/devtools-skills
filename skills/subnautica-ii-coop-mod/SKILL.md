---
name: subnautica-ii-coop-mod
description: BepInEx-based multiplayer co-op mod for Subnautica 2 with synchronized gameplay, adaptive scaling, and decentralized peer-to-peer networking
triggers:
  - how do I install the Subnautica 2 co-op mod
  - set up multiplayer for Subnautica 2
  - configure Deep Synergy mod
  - troubleshoot Subnautica 2 multiplayer sync issues
  - create a co-op session in Subnautica 2
  - integrate OpenAI narration with Subnautica mod
  - adjust difficulty scaling for Subnautica multiplayer
  - fix BepInEx plugin conflicts in Subnautica
---

# Subnautica II Co-op Mod (Deep Synergy)

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The **Deep Synergy Multiplayer Mod** transforms Subnautica 2 into a synchronized cooperative experience using the BepInEx modding framework. It implements deterministic session synchronization, adaptive difficulty scaling, shared inventory systems, and peer-to-peer networking without requiring central servers.

**Key Capabilities:**
- Deterministic state synchronization across multiple clients
- Adaptive dynamic scaling based on player count
- BepInEx IL2CPP plugin architecture for Unity runtime hooks
- Decentralized peer-to-peer networking with NAT punch-through
- Optional OpenAI/Claude API integration for narrative generation
- Cross-platform support (Windows, Linux, macOS)

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam or GOG
2. **BepInEx 6.0.x** or later for Unity IL2CPP games

### Installation Steps

```bash
# 1. Install BepInEx 6.0.x for Subnautica 2
# Download from https://github.com/BepInEx/BepInEx/releases
# Extract to Subnautica 2 game directory

# 2. Download Deep Synergy Mod
# Extract the mod package

# 3. Copy plugin files
cp -r BepInEx/plugins/* <Subnautica2Directory>/BepInEx/plugins/

# 4. Create configuration directory
mkdir -p <Subnautica2Directory>/BepInEx/config/

# 5. Launch game to generate default configs
# BepInEx will create config files on first run
```

### Directory Structure

```
Subnautica2/
├── BepInEx/
│   ├── core/
│   ├── plugins/
│   │   ├── DeepSynergy.dll
│   │   ├── SessionManager.dll
│   │   └── StateSynchronizer.dll
│   └── config/
│       ├── synergy_profile.json
│       └── session_config.xml
└── Subnautica2.exe
```

## Configuration

### Profile Configuration

Create `BepInEx/config/synergy_profile.json`:

```json
{
  "session_name": "DeepDive Squad",
  "max_players": 4,
  "difficulty_scale": "adaptive",
  "resource_multiplier": 1.0,
  "oxygen_consumption": 1.0,
  "creature_spawn_divider": 1,
  "enable_pvp": false,
  "friendly_fire": false,
  "shared_blueprints": true,
  "ping_locations_shared": true,
  "time_of_day_sync": "all",
  "voice_chat_integration": "discord_rpc",
  "network": {
    "use_nat_punchthrough": true,
    "port": 7777,
    "max_latency_ms": 200,
    "sync_interval_ms": 50
  },
  "api_integration": {
    "openai": {
      "enabled": false,
      "api_key_env": "OPENAI_API_KEY",
      "role": "narrator",
      "model": "gpt-4"
    },
    "claude": {
      "enabled": false,
      "api_key_env": "ANTHROPIC_API_KEY",
      "role": "lore_engine",
      "model": "claude-3-opus-20240229"
    }
  },
  "locale": "en-US"
}
```

### Configuration Options

| Field | Type | Description |
|-------|------|-------------|
| `session_name` | string | Display name for your session |
| `max_players` | int | Maximum concurrent players (2-8) |
| `difficulty_scale` | string | `"adaptive"`, `"fixed"`, or `"custom"` |
| `resource_multiplier` | float | Resource spawn rate multiplier (0.5-2.0) |
| `oxygen_consumption` | float | Oxygen drain rate (0.5-2.0, lower = slower drain) |
| `creature_spawn_divider` | int | Divides creature spawn counts |
| `shared_blueprints` | bool | Share unlocked blueprints across players |
| `ping_locations_shared` | bool | Synchronize map pings |
| `time_of_day_sync` | string | `"all"`, `"host"`, or `"individual"` |

### API Integration Setup

```bash
# Set API keys via environment variables
export OPENAI_API_KEY="your-openai-key-here"
export ANTHROPIC_API_KEY="your-anthropic-key-here"

# Enable in synergy_profile.json
# The mod will automatically use these keys when api_integration.enabled = true
```

## Console Commands

Access the BepInEx console by pressing **F1** in-game (default keybind).

### Session Management

```bash
# Start a host session
/start_server

# Join existing session
/join_session <session-code>
# Example: /join_session 9B2A-4C7D-E8F1

# Leave current session
/leave_session

# Display session status
/synergy_status
```

### Gameplay Controls

```bash
# Adjust difficulty scaling (temporary override)
/synergy_scale <multiplier>
# Example: /synergy_scale 1.5

# Override world seed
/seed_override <seed>
# Example: /seed_override 8251

# Force inventory sync
/force_sync inventory

# Teleport to player
/teleport_to <player_name>
```

### API-Powered Features

```bash
# Trigger AI narration (requires OpenAI enabled)
/api_narrate "<context>"
# Example: /api_narrate "exploring the underwater caves"

# Generate creature lore (requires Claude enabled)
/api_lore <creature_id>
# Example: /api_lore reaper_leviathan

# Generate base name suggestions
/api_name_base
```

## Code Examples

### Custom Plugin Integration

If you're developing additional BepInEx plugins that interact with Deep Synergy:

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using DeepSynergy.API;
using UnityEngine;

namespace MyCustomPlugin
{
    [BepInPlugin("com.myname.custommod", "Custom Mod", "1.0.0")]
    [BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomPlugin : BasePlugin
    {
        public override void Load()
        {
            // Get Deep Synergy session manager
            var sessionManager = DeepSynergyAPI.GetSessionManager();
            
            // Subscribe to session events
            sessionManager.OnPlayerJoined += OnPlayerJoined;
            sessionManager.OnPlayerLeft += OnPlayerLeft;
            sessionManager.OnStateSync += OnStateSync;
            
            Log.LogInfo("Custom plugin loaded with Deep Synergy integration");
        }
        
        private void OnPlayerJoined(PlayerInfo player)
        {
            Log.LogInfo($"Player joined: {player.Name} (ID: {player.PeerId})");
            
            // Send custom data to all clients
            DeepSynergyAPI.BroadcastCustomData("MyCustomPlugin", new {
                message = $"{player.Name} joined!",
                timestamp = System.DateTime.UtcNow
            });
        }
        
        private void OnPlayerLeft(PlayerInfo player)
        {
            Log.LogInfo($"Player left: {player.Name}");
        }
        
        private void OnStateSync(SyncEvent syncEvent)
        {
            // Handle synchronized state updates
            if (syncEvent.Type == SyncEventType.InventoryUpdate)
            {
                Log.LogInfo($"Inventory synced for player {syncEvent.PlayerId}");
            }
        }
    }
}
```

### Programmatic Configuration

```csharp
using DeepSynergy.Config;
using Newtonsoft.Json;
using System.IO;

public class ConfigManager
{
    public static SynergyProfile LoadProfile(string path)
    {
        var json = File.ReadAllText(path);
        return JsonConvert.DeserializeObject<SynergyProfile>(json);
    }
    
    public static void CreateDefaultProfile(string path)
    {
        var profile = new SynergyProfile
        {
            SessionName = "New Session",
            MaxPlayers = 4,
            DifficultyScale = "adaptive",
            ResourceMultiplier = 1.0f,
            OxygenConsumption = 1.0f,
            CreatureSpawnDivider = 1,
            EnablePvp = false,
            SharedBlueprints = true,
            Network = new NetworkConfig
            {
                UseNatPunchthrough = true,
                Port = 7777,
                MaxLatencyMs = 200,
                SyncIntervalMs = 50
            }
        };
        
        var json = JsonConvert.SerializeObject(profile, Formatting.Indented);
        File.WriteAllText(path, json);
    }
    
    public static void UpdateScaling(float multiplier)
    {
        var api = DeepSynergyAPI.GetSessionManager();
        api.SetDifficultyMultiplier(multiplier);
    }
}
```

### Custom API Integration

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;

public class NarrativeEngine
{
    private readonly string _openAiKey;
    private readonly HttpClient _client;
    
    public NarrativeEngine()
    {
        _openAiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
        _client = new HttpClient();
        _client.DefaultRequestHeaders.Add("Authorization", $"Bearer {_openAiKey}");
    }
    
    public async Task<string> GenerateNarration(string context)
    {
        var request = new
        {
            model = "gpt-4",
            messages = new[]
            {
                new { role = "system", content = "You are a narrative engine for Subnautica 2, providing immersive journal entries." },
                new { role = "user", content = $"Generate a brief journal entry about: {context}" }
            },
            max_tokens = 150,
            temperature = 0.8
        };
        
        var json = JsonConvert.SerializeObject(request);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        var response = await _client.PostAsync(
            "https://api.openai.com/v1/chat/completions",
            content
        );
        
        var responseJson = await response.Content.ReadAsStringAsync();
        var result = JsonConvert.DeserializeObject<dynamic>(responseJson);
        
        return result.choices[0].message.content.ToString();
    }
}
```

## Common Patterns

### Hosting a Session

```bash
# 1. Configure your profile
# Edit BepInEx/config/synergy_profile.json

# 2. Launch game
# 3. Open BepInEx console (F1)
# 4. Start server
/start_server

# 5. Share the generated session code with friends
# Example output: "Server created: session code = 9B2A-4C7D-E8F1"
```

### Joining a Session

```bash
# 1. Launch game
# 2. Open BepInEx console (F1)
# 3. Join using session code
/join_session 9B2A-4C7D-E8F1

# 4. Wait for sync to complete
/synergy_status
# Look for "State sync: 100% complete"
```

### Monitoring Session Health

```bash
# Check connected players and latency
/synergy_status

# Example output:
# Connected peers: 3
# State sync: 100% complete
# Average latency: 45ms
# Inventory hash: 0xFA342B1E
```

### Adaptive Difficulty Adjustment

```bash
# View current scaling
/synergy_status

# Temporarily increase difficulty
/synergy_scale 1.5

# Reset to profile defaults
/synergy_scale 1.0
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to host / "Connection timeout"

```bash
# 1. Check firewall rules
# Windows: Allow port 7777 (UDP) in Windows Firewall
# Linux: sudo ufw allow 7777/udp

# 2. Verify NAT settings in profile
# Edit synergy_profile.json:
"network": {
  "use_nat_punchthrough": true,
  "port": 7777
}

# 3. Check BepInEx logs
# Location: BepInEx/LogOutput.log
grep "SessionManager" BepInEx/LogOutput.log
```

**Problem:** "State sync failed" errors

```bash
# Force inventory resync
/force_sync inventory

# Check for mod conflicts
# Disable other BepInEx plugins temporarily and test
```

### Performance Issues

**Problem:** High latency or stuttering

```bash
# 1. Reduce sync frequency in profile
"network": {
  "sync_interval_ms": 100  # Increase from 50ms to 100ms
}

# 2. Lower creature spawn rates
"creature_spawn_divider": 2  # Half as many creatures

# 3. Check network stats
/synergy_status
# Look for "Average latency" > 200ms
```

**Problem:** Game crashes on session join

```bash
# 1. Check BepInEx version compatibility
# Ensure BepInEx 6.0.x or later is installed

# 2. Verify mod file integrity
# Re-download and reinstall DeepSynergy.dll

# 3. Clear cache
rm -rf BepInEx/cache/*
# Restart game

# 4. Check logs for IL2CPP errors
grep "IL2CPP" BepInEx/LogOutput.log
```

### API Integration Issues

**Problem:** OpenAI/Claude narration not working

```bash
# 1. Verify environment variables
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# 2. Check API integration settings in profile
"api_integration": {
  "openai": {
    "enabled": true,
    "api_key_env": "OPENAI_API_KEY"
  }
}

# 3. Test API connectivity
/api_narrate "test"
# Check BepInEx logs for API response errors
```

### Inventory Desync

**Problem:** Items disappearing or duplicating

```bash
# 1. Force inventory sync
/force_sync inventory

# 2. Check inventory hash consistency
/synergy_status
# All players should have matching "Inventory hash"

# 3. If persistent, restart session
/leave_session
# Host restarts with /start_server
# All players rejoin
```

### Localization Issues

**Problem:** UI text showing wrong language

```json
// Edit synergy_profile.json
{
  "locale": "en-US"  // Change to your preferred locale
}
// Supported: en-US, zh-CN, ja-JP, de-DE, fr-FR, pt-BR, ru-RU, es-LA, ko-KR
```

### Log Analysis

```bash
# View recent errors
tail -n 100 BepInEx/LogOutput.log | grep ERROR

# Filter for Deep Synergy specific logs
grep "DeepSynergy" BepInEx/LogOutput.log

# Monitor logs in real-time
tail -f BepInEx/LogOutput.log
```

## Best Practices

1. **Always backup save files** before using multiplayer mods
2. **Use matching mod versions** across all players
3. **Configure firewall rules** before hosting sessions
4. **Monitor latency** with `/synergy_status` during gameplay
5. **Disable conflicting mods** if experiencing issues
6. **Use environment variables** for API keys, never hardcode
7. **Test configuration changes** in solo mode before hosting

## Advanced Usage

### Custom Sync Rules

```csharp
// Register custom synchronization logic
DeepSynergyAPI.RegisterSyncHandler("CustomItemType", (data, playerId) => {
    // Handle custom item sync
    var item = JsonConvert.DeserializeObject<CustomItem>(data);
    GameState.ApplyCustomItem(item, playerId);
});
```

### Session Migration

```bash
# If host disconnects, session migrates automatically
# Force migration to specific player
/migrate_host <player_name>
```

### Debugging Mode

```json
// Enable verbose logging in synergy_profile.json
{
  "debug": {
    "enabled": true,
    "log_sync_events": true,
    "log_network_packets": true
  }
}
```
