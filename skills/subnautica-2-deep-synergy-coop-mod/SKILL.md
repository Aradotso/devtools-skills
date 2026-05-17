---
name: subnautica-2-deep-synergy-coop-mod
description: BepInEx-based cooperative multiplayer mod for Subnautica 2 with synchronized sessions, shared inventory, and AI-enhanced narrative generation
triggers:
  - install subnautica 2 multiplayer mod
  - configure deep synergy coop session
  - setup bepinex subnautica 2 mod
  - create subnautica multiplayer session
  - troubleshoot subnautica coop sync issues
  - integrate openai with subnautica mod
  - configure synergy profile json
  - fix subnautica multiplayer connection
---

# Subnautica 2 Deep Synergy Coop Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Deep Synergy is a BepInEx-based cooperative multiplayer modification for Subnautica 2 that transforms the single-player survival experience into a synchronized multi-player session. The mod implements deterministic session synchronization (DSS), adaptive dynamic scaling (ADS), and decentralized peer-to-peer networking to enable seamless co-op gameplay without central server infrastructure.

**Key Capabilities:**
- Synchronized world state across multiple clients using Merkle tree inventory tracking
- WebRTC-based peer-to-peer networking with NAT punch-through
- Hot-reloadable BepInEx plugin architecture (IL2CPP hooks)
- Optional OpenAI/Claude API integration for dynamic narrative generation
- Cross-platform support (Windows, macOS, Linux/Steam Deck)
- Adaptive difficulty scaling based on player count

## Installation

### Prerequisites

1. **Install BepInEx 6.0.x** for Subnautica 2:
   - Download BepInEx from the official repository
   - Extract to Subnautica 2 game root directory
   - Launch game once to generate BepInEx folder structure

2. **Download Deep Synergy Mod**:
   - Obtain from the project repository
   - Extract contents to `<game_root>/BepInEx/plugins/`

### Directory Structure

```
Subnautica2/
├── BepInEx/
│   ├── plugins/
│   │   ├── DeepSynergyCore.dll
│   │   ├── SessionManager.dll
│   │   ├── StateSynchronizer.dll
│   │   └── ConflictResolver.dll
│   ├── config/
│   │   ├── synergy_profile.json
│   │   └── session_config.xml
│   └── LogOutput.log
└── Subnautica2.exe
```

### First Launch

```bash
# Launch via Steam or direct executable
# BepInEx will patch the game on first run
# Check BepInEx/LogOutput.log for successful mod loading
```

Expected log output:
```
[Info   :   BepInEx] BepInEx 6.0.0 - Subnautica2 (5/14/2026)
[Info   :DeepSynergy] Deep Synergy Multiplayer Mod v1.0.0 loaded
[Info   :DeepSynergy] Session Manager initialized
[Info   :DeepSynergy] WebRTC transport layer ready
```

## Configuration

### Session Profile (`BepInEx/config/synergy_profile.json`)

```json
{
  "session_name": "Deep Exploration Team",
  "max_players": 4,
  "difficulty_scale": "adaptive",
  "resource_multiplier": 1.5,
  "oxygen_consumption": 0.85,
  "creature_spawn_divider": 1.5,
  "enable_pvp": false,
  "friendly_fire": false,
  "shared_blueprints": true,
  "ping_locations_shared": true,
  "time_of_day_sync": "host",
  "voice_chat_integration": "discord_rpc",
  "locale": "en_US",
  "api_integration": {
    "openai": {
      "enabled": false,
      "api_key_env": "OPENAI_API_KEY",
      "model": "gpt-4",
      "role": "narrator"
    },
    "claude": {
      "enabled": false,
      "api_key_env": "ANTHROPIC_API_KEY",
      "model": "claude-3-opus-20240229",
      "role": "lore_engine"
    }
  },
  "network": {
    "nat_traversal": true,
    "max_latency_ms": 250,
    "sync_interval_ms": 50,
    "inventory_hash_algorithm": "sha256"
  }
}
```

**Key Configuration Fields:**

- `difficulty_scale`: `"static"`, `"adaptive"`, or `"linear"` - controls how game difficulty adjusts with player count
- `resource_multiplier`: Float multiplier for harvestable resource nodes (1.0 = vanilla, 2.0 = double)
- `oxygen_consumption`: Fraction of normal oxygen drain (0.5 = half drain, 1.0 = vanilla)
- `creature_spawn_divider`: Reduces creature spawns (2.0 = half spawns)
- `time_of_day_sync`: `"host"` (host controls time), `"all"` (synced vote), `"independent"` (client-specific)
- `api_key_env`: Environment variable name containing API keys (never hardcode keys)

### XML Configuration (`BepInEx/config/session_config.xml`)

```xml
<?xml version="1.0" encoding="utf-8"?>
<SynergyConfig>
  <Session>
    <DefaultPort>25565</DefaultPort>
    <EnableUPnP>true</EnableUPnP>
    <MaxConnectionAttempts>5</MaxConnectionAttempts>
    <ConnectionTimeoutSeconds>30</ConnectionTimeoutSeconds>
  </Session>
  <Synchronization>
    <InventoryHashInterval>1000</InventoryHashInterval>
    <CreatureStateInterval>200</CreatureStateInterval>
    <BaseStructureInterval>500</BaseStructureInterval>
  </Synchronization>
  <Logging>
    <Level>Info</Level>
    <EnableNetworkDebug>false</EnableNetworkDebug>
  </Logging>
</SynergyConfig>
```

## In-Game Console Commands

Access the BepInEx console with `F12` (default binding, configurable):

### Session Management

```bash
# Start hosting a session
/start_server
# Output: Server created: session code = 9B2A-4C7D-E8F1

# Join existing session
/join_session 9B2A-4C7D-E8F1

# Check connection status
/synergy_status
# Output:
# Connected peers: 3/4
# State sync: 100% complete
# Inventory hash: 0xFA342B1E
# Avg latency: 45ms

# Leave session gracefully
/disconnect_session
```

### Gameplay Scaling

```bash
# Temporarily adjust difficulty multiplier
/synergy_scale 1.5

# Reset to config defaults
/synergy_reset

# Force world seed sync (host only)
/seed_override 8251
```

### AI Narrative Integration

```bash
# Trigger OpenAI narrative generation
/api_narrate "discovered alien artifact in lost river"

# Generate species biology log with Claude
/api_lore_generate "ghost leviathan juvenile"

# Show last 5 AI-generated journal entries
/api_journal_history 5
```

### Debugging

```bash
# Enable verbose network logging
/debug_network true

# Dump current inventory Merkle tree
/debug_inventory_tree

# Force state resync
/force_resync

# Show desync detection results
/debug_conflicts
```

## Plugin Development & Extension

### BepInEx Plugin Structure

The mod uses BepInEx IL2CPP hooking. To extend functionality:

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using HarmonyLib;
using DeepSynergy.Core;

namespace DeepSynergy.CustomExtension
{
    [BepInPlugin(PluginGUID, PluginName, PluginVersion)]
    [BepInDependency("com.deepsynergy.core")]
    public class CustomExtensionPlugin : BasePlugin
    {
        public const string PluginGUID = "com.yourname.customextension";
        public const string PluginName = "Custom Deep Synergy Extension";
        public const string PluginVersion = "1.0.0";

        private Harmony _harmony;

        public override void Load()
        {
            _harmony = new Harmony(PluginGUID);
            _harmony.PatchAll();
            
            // Hook into session events
            SessionManager.OnPlayerJoined += OnPlayerJoinedHandler;
            SessionManager.OnInventorySynced += OnInventorySyncedHandler;
            
            Log.LogInfo($"{PluginName} loaded successfully");
        }

        private void OnPlayerJoinedHandler(string playerId)
        {
            Log.LogInfo($"Player joined: {playerId}");
            // Custom logic here
        }

        private void OnInventorySyncedHandler(InventorySyncData data)
        {
            Log.LogInfo($"Inventory synced - hash: {data.MerkleRootHash}");
        }
    }
}
```

### Harmony Patch Example

```csharp
using HarmonyLib;
using DeepSynergy.Core;

namespace DeepSynergy.CustomExtension
{
    [HarmonyPatch(typeof(Player), "ConsumeOxygen")]
    public static class OxygenConsumptionPatch
    {
        static void Prefix(ref float amount)
        {
            // Apply session multiplier from config
            float multiplier = SynergyConfig.Current.OxygenConsumption;
            amount *= multiplier;
            
            // Sync to other clients
            if (SessionManager.IsHost)
            {
                NetworkSync.BroadcastOxygenEvent(amount);
            }
        }
    }
}
```

### Custom API Integration

```csharp
using DeepSynergy.API;
using System.Threading.Tasks;

public class CustomNarrativeGenerator
{
    private readonly OpenAIClient _openai;
    
    public CustomNarrativeGenerator()
    {
        string apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
        _openai = new OpenAIClient(apiKey);
    }
    
    public async Task<string> GenerateBaseDescription(BaseStructure baseData)
    {
        string prompt = $@"Generate a brief narrative description for an underwater base:
- Location: {baseData.Biome}
- Depth: {baseData.DepthMeters}m
- Components: {string.Join(", ", baseData.Modules)}
- Players: {baseData.PlayerCount}
Keep it under 100 words, immersive tone.";

        var response = await _openai.GenerateCompletion(prompt, maxTokens: 150);
        return response.Text;
    }
}
```

## Common Usage Patterns

### Two-Player Resource Gathering Session

```json
{
  "session_name": "Resource Run",
  "max_players": 2,
  "difficulty_scale": "static",
  "resource_multiplier": 1.3,
  "oxygen_consumption": 0.9,
  "creature_spawn_divider": 1.2,
  "shared_blueprints": true,
  "time_of_day_sync": "host"
}
```

Console workflow:
```bash
# Player 1 (Host)
/start_server
# Share code 4A7B-9C2D-E5F3 with Player 2

# Player 2 (Client)
/join_session 4A7B-9C2D-E5F3

# Both players check sync
/synergy_status
```

### Four-Player Base Building

```json
{
  "session_name": "Mega Base Construction",
  "max_players": 4,
  "difficulty_scale": "adaptive",
  "resource_multiplier": 2.0,
  "oxygen_consumption": 0.7,
  "creature_spawn_divider": 2.0,
  "shared_blueprints": true,
  "ping_locations_shared": true,
  "time_of_day_sync": "all"
}
```

### AI-Enhanced Exploration

```json
{
  "session_name": "Lore Expedition",
  "max_players": 2,
  "api_integration": {
    "openai": {
      "enabled": true,
      "api_key_env": "OPENAI_API_KEY",
      "model": "gpt-4",
      "role": "narrator"
    },
    "claude": {
      "enabled": true,
      "api_key_env": "ANTHROPIC_API_KEY",
      "model": "claude-3-opus-20240229",
      "role": "lore_engine"
    }
  }
}
```

Generate narratives during gameplay:
```bash
/api_narrate "entering the blood kelp zone at 300m depth"
# Output: "The crimson fronds pulse with bioluminescent life as the team descends..."

/api_lore_generate "crabsquid behavior patterns"
# Output: "Species: Crabsquid (Elektrus Crustacea)..."
```

## Troubleshooting

### Issue: "Session code invalid or expired"

**Cause:** NAT traversal failed or host disconnected

**Solution:**
```bash
# Check NAT configuration
/debug_network true
# Look for "NAT type: Symmetric" in logs (problematic)

# Enable UPnP in session_config.xml
<EnableUPnP>true</EnableUPnP>

# Alternative: Port forward 25565 on router
```

### Issue: "Inventory desync detected"

**Cause:** High latency or packet loss causing Merkle tree mismatch

**Solution:**
```bash
# Check current sync status
/debug_conflicts

# Force full resync
/force_resync

# Adjust sync interval in config
"sync_interval_ms": 100  # Increase from 50ms
```

### Issue: "BepInEx mod not loading"

**Cause:** IL2CPP hook failure or incorrect installation

**Solution:**
1. Check `BepInEx/LogOutput.log` for errors
2. Verify BepInEx version is 6.0.x or higher
3. Ensure all `.dll` files are in `BepInEx/plugins/`
4. Delete `BepInEx/cache/` and restart game

### Issue: "OpenAI/Claude API calls failing"

**Cause:** Missing or invalid API keys

**Solution:**
```bash
# Verify environment variable is set
# Windows PowerShell
echo $env:OPENAI_API_KEY

# Linux/macOS
echo $OPENAI_API_KEY

# Set if missing (do NOT commit to code)
# Windows
setx OPENAI_API_KEY "your-key-here"

# Linux/macOS
export OPENAI_API_KEY="your-key-here"
```

Check config references correct env var:
```json
"api_integration": {
  "openai": {
    "api_key_env": "OPENAI_API_KEY"  // Must match env var name
  }
}
```

### Issue: "High latency/lag in multiplayer"

**Cause:** Network congestion or inefficient sync intervals

**Solution:**
```bash
# Check current latency
/synergy_status
# If avg latency > 150ms, adjust config

# Increase sync intervals in session_config.xml
<InventoryHashInterval>2000</InventoryHashInterval>  # From 1000
<CreatureStateInterval>400</CreatureStateInterval>   # From 200

# Reduce max players
"max_players": 2  # From 4
```

### Issue: "Creatures not syncing properly"

**Cause:** Creature AI state lattice desync

**Solution:**
```bash
# Enable creature debug logging
/debug_network true

# Check creature sync interval
<CreatureStateInterval>200</CreatureStateInterval>

# Force creature state resync (host only)
/force_resync creatures
```

## Advanced Configuration

### Custom Biome Multipliers

Edit `synergy_profile.json` to add biome-specific resource scaling:

```json
{
  "biome_overrides": {
    "safe_shallows": {
      "resource_multiplier": 1.0,
      "creature_spawn_divider": 1.0
    },
    "kelp_forest": {
      "resource_multiplier": 1.2,
      "creature_spawn_divider": 1.3
    },
    "blood_kelp_zone": {
      "resource_multiplier": 1.5,
      "creature_spawn_divider": 2.0
    },
    "lost_river": {
      "resource_multiplier": 2.0,
      "creature_spawn_divider": 2.5
    }
  }
}
```

### Session Persistence

Enable session save/load:

```json
{
  "session_persistence": {
    "enabled": true,
    "autosave_interval_minutes": 10,
    "save_directory": "BepInEx/saves/",
    "compress_saves": true
  }
}
```

Load saved session:
```bash
/load_session "Deep_Exploration_2026-05-14"
```

### Localization Customization

Add custom translations in `BepInEx/config/localizations/custom_en.json`:

```json
{
  "ui.session.created": "Session established: {0}",
  "ui.player.joined": "{0} has entered the abyss",
  "ui.inventory.synced": "Inventory synchronized",
  "console.help.start_server": "Create a new multiplayer session"
}
```

Reference in code:
```csharp
string message = LocalizationManager.Get("ui.player.joined", playerName);
```

## Performance Optimization

### Reduce Bandwidth Usage

```json
{
  "network": {
    "compression": "lz4",
    "delta_encoding": true,
    "sync_radius_meters": 500,  // Only sync entities within 500m
    "inventory_batch_updates": true
  }
}
```

### Optimize for Low-End Systems

```json
{
  "performance": {
    "reduce_sync_precision": true,  // Reduce float precision for position sync
    "skip_inactive_entities": true,  // Don't sync entities outside render distance
    "lazy_merkle_updates": true      // Update inventory tree only on change
  }
}
```

## Security Considerations

- **Never hardcode API keys** - always use environment variables
- **Session codes are single-use** - regenerate for each session
- **Inventory Merkle trees prevent duping** - tampering detected automatically
- **Host migration is automatic** - session survives host disconnection
- **No executable code transmission** - only JSON state data synchronized
