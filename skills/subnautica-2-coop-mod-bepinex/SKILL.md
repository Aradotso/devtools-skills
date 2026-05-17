---
name: subnautica-2-coop-mod-bepinex
description: BepInEx-based multiplayer mod for Subnautica 2 enabling synchronized co-op gameplay with shared world state, base building, and inventory management
triggers:
  - how do I install the Subnautica 2 multiplayer mod
  - set up BepInEx co-op for Subnautica 2
  - configure Deep Synergy multiplayer mod
  - create a co-op session in Subnautica 2
  - troubleshoot Subnautica 2 mod sync issues
  - enable multiplayer in Subnautica 2 with BepInEx
  - join a Subnautica 2 co-op game
  - configure session settings for Subnautica 2 mod
---

# Subnautica 2 Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using the BepInEx modding framework. It implements deterministic session synchronization, adaptive difficulty scaling, and peer-to-peer networking to enable multiple players to explore, build bases, and survive together in Subnautica 2's alien oceans.

**Key Features:**
- Deterministic Session Synchronization (DSS) for conflict-free shared world state
- Adaptive Dynamic Scaling that adjusts difficulty based on player count
- BepInEx IL2CPP-level hooking with hot-reloadable plugin architecture
- Shared inventory using Merkle tree verification
- Cross-platform multiplayer via NAT punch-through and WebRTC
- Session migration on host disconnect with zero data loss

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam or GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### Step-by-Step Installation

```bash
# 1. Install BepInEx 6.0.x for Subnautica 2
# Download from: https://github.com/BepInEx/BepInEx/releases
# Extract to your Subnautica 2 game directory

# 2. Verify BepInEx installation structure
# Your game directory should have:
# Subnautica2/
#   ├── BepInEx/
#   │   ├── core/
#   │   ├── plugins/
#   │   └── config/
#   ├── Subnautica2.exe (Windows)
#   └── ...

# 3. Download Deep Synergy Mod
# Extract the mod files to: Subnautica2/BepInEx/plugins/

# 4. Directory structure after installation:
# BepInEx/
#   ├── plugins/
#   │   ├── DeepSynergy.dll
#   │   ├── DeepSynergy.Core.dll
#   │   └── DeepSynergy.Network.dll
#   └── config/
#       └── synergy_profile.json
```

### Verify Installation

Launch Subnautica 2. You should see BepInEx console output indicating the mod loaded:

```
[Info   : BepInEx] BepInEx 6.0.0 - Subnautica2
[Info   : BepInEx] Running under Unity 2022.3.x
[Info   : DeepSynergy] Deep Synergy Multiplayer Mod v1.0.0 loaded
[Info   : DeepSynergy] Session Manager initialized
[Info   : DeepSynergy] WebRTC transport ready
```

## Configuration

### Basic Configuration File

Create or edit `BepInEx/config/synergy_profile.json`:

```json
{
  "session_name": "Ocean Explorers",
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
  "voice_chat_integration": "none",
  "locale": "auto",
  "api_integration": {
    "openai": {
      "enabled": false,
      "api_key_env": "OPENAI_API_KEY",
      "role": "narrator"
    },
    "claude": {
      "enabled": false,
      "api_key_env": "ANTHROPIC_API_KEY",
      "role": "lore_engine"
    }
  },
  "network": {
    "port": 7777,
    "use_upnp": true,
    "max_latency_ms": 150,
    "packet_loss_tolerance": 0.05
  },
  "session_recovery": {
    "auto_save_interval_seconds": 60,
    "enable_migration": true,
    "backup_count": 3
  }
}
```

### Configuration Options Explained

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `max_players` | int | 4 | Maximum players per session (2-8) |
| `difficulty_scale` | string | "adaptive" | "fixed", "adaptive", or "progressive" |
| `resource_multiplier` | float | 1.0 | Resource node spawn rate (0.5-3.0) |
| `oxygen_consumption` | float | 1.0 | Oxygen drain rate (0.5-2.0) |
| `creature_spawn_divider` | int | 1 | Divide creature counts by this value |
| `shared_blueprints` | bool | true | All players unlock blueprints together |
| `time_of_day_sync` | string | "all" | "all", "host", or "independent" |

### Advanced Session Configuration

```json
{
  "session_name": "Hardcore Survival",
  "max_players": 2,
  "difficulty_scale": "progressive",
  "resource_multiplier": 0.7,
  "oxygen_consumption": 1.3,
  "creature_spawn_divider": 1,
  "enable_pvp": false,
  "friendly_fire": true,
  "shared_blueprints": false,
  "death_penalty": "inventory_drop",
  "respawn_delay_seconds": 30,
  "shared_storage": {
    "enabled": true,
    "max_shared_lockers": 10,
    "require_proximity": false
  },
  "base_building": {
    "collaborative_placement": true,
    "require_consensus": false,
    "power_sharing": true
  }
}
```

## Console Commands

Access the BepInEx console (default: F12 in-game) to use these commands:

### Session Management

```bash
# Start hosting a new session
/start_server
# Output: Server created: session code = 9B2A-4C7D-E8F1

# Join an existing session
/join_session 9B2A-4C7D-E8F1

# Get current session status
/synergy_status
# Output:
# Connected peers: 2/4
# State sync: 100% complete
# Inventory hash: 0xFA342B1E
# Latency: Player1: 45ms, Player2: 67ms

# Disconnect from session
/leave_session

# Force session migration (transfer host)
/migrate_host Player2
```

### Runtime Configuration

```bash
# Temporarily adjust difficulty scaling
/synergy_scale 1.5

# Change resource multiplier mid-session
/set_resource_multiplier 1.2

# Override world seed for new playthroughs
/seed_override 8251

# Toggle debug mode
/synergy_debug true

# Check synchronization health
/sync_health
# Output:
# Merkle tree integrity: OK
# Pending conflicts: 0
# Timestamp drift: +2ms
```

### AI Integration Commands (Optional)

```bash
# Trigger OpenAI narrative generation
/api_narrate "exploring the underwater caves"

# Request Claude lore generation for current location
/api_lore current_biome

# Get AI-generated base naming suggestions
/api_suggest_name base

# Generate creature analysis
/api_analyze_creature reaper_leviathan
```

## Core Patterns and Usage

### Starting a Co-op Session (Host)

```csharp
// In-game: Press configured hotkey (default: F9) to open multiplayer menu
// Or use console:

// 1. Start server
/start_server

// 2. Configure session (optional, uses synergy_profile.json by default)
/set_max_players 3
/set_difficulty_scale adaptive

// 3. Share the displayed session code with friends
// Session code appears in chat: "9B2A-4C7D-E8F1"

// 4. Wait for players to connect
// Connection messages appear in chat:
// "Player2 joined the session"
// "Player3 joined the session"
```

### Joining a Co-op Session (Client)

```bash
# 1. Open multiplayer menu (F9) or use console
/join_session 9B2A-4C7D-E8F1

# 2. Wait for synchronization
# Progress appears in UI:
# "Synchronizing world state... 45%"
# "Downloading inventory data... 78%"
# "Syncing base structures... 100%"

# 3. Spawn into the world
# You'll appear at the host's current location or a designated spawn point
```

### Shared Base Building

When `collaborative_placement` is enabled, base building is synchronized:

```bash
# Player 1 places a foundation
# Action: Place Foundation at coordinates (100, -50, 200)

# Player 2 sees foundation immediately and can build on it
# Action: Place Corridor connecting to Player 1's foundation

# Both players see the same structure in real-time
# Power generation and resource consumption are shared
```

### Inventory Synchronization

The mod uses Merkle tree verification for inventory integrity:

```bash
# Check inventory sync status
/sync_health

# If desync detected:
# "Warning: Inventory hash mismatch with Player2"
# "Initiating reconciliation..."

# Force inventory resync (rarely needed)
/force_resync inventory

# Check specific item counts across all players
/debug_item titanium
# Output:
# Host: 47 titanium
# Player2: 47 titanium
# Player3: 47 titanium (shared storage)
```

### Session Recovery and Migration

If the host disconnects, the mod automatically migrates:

```bash
# Automatic migration occurs
# Console output:
# "Host disconnected. Initiating session migration..."
# "Player2 elected as new host"
# "Restoring session state from backup..."
# "Migration complete. Session restored."

# Players can continue without interruption
# All progress, bases, and inventory are preserved
```

## Plugin Development (Advanced)

### Creating Custom BepInEx Plugins for the Mod

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using HarmonyLib;
using DeepSynergy.Core;
using DeepSynergy.Network;

namespace CustomSynergyPlugin
{
    [BepInPlugin("com.example.customsynergy", "Custom Synergy Plugin", "1.0.0")]
    [BepInDependency("com.deepsynergy.core")]
    public class CustomSynergyPlugin : BasePlugin
    {
        private Harmony harmony;

        public override void Load()
        {
            harmony = new Harmony("com.example.customsynergy");
            harmony.PatchAll();

            // Subscribe to Deep Synergy events
            SessionManager.OnPlayerJoined += OnPlayerJoined;
            SessionManager.OnPlayerLeft += OnPlayerLeft;
            StateSync.OnInventoryChanged += OnInventoryChanged;

            Log.LogInfo("Custom Synergy Plugin loaded!");
        }

        private void OnPlayerJoined(string playerId, string playerName)
        {
            Log.LogInfo($"Player joined: {playerName} ({playerId})");
            
            // Example: Send custom welcome message
            NetworkManager.SendMessage(playerId, new WelcomeMessage
            {
                Text = $"Welcome to the session, {playerName}!",
                ShowNotification = true
            });
        }

        private void OnPlayerLeft(string playerId)
        {
            Log.LogInfo($"Player left: {playerId}");
        }

        private void OnInventoryChanged(string playerId, InventoryDelta delta)
        {
            // React to inventory changes
            if (delta.ItemAdded == "titanium" && delta.Count >= 10)
            {
                Log.LogInfo($"{playerId} collected 10+ titanium!");
            }
        }
    }
}
```

### Hooking into Base Building Events

```csharp
using HarmonyLib;
using DeepSynergy.Core;

[HarmonyPatch(typeof(BaseManager), "PlaceBaseComponent")]
public class BaseComponentPatch
{
    static void Postfix(BaseComponent component, Vector3 position, Quaternion rotation)
    {
        // Called after any base component is placed
        Log.LogInfo($"Base component placed: {component.Type} at {position}");

        // Sync to all connected clients
        StateSync.BroadcastBaseChange(new BaseChangeEvent
        {
            ComponentType = component.Type,
            Position = position,
            Rotation = rotation,
            PlayerId = SessionManager.LocalPlayerId,
            Timestamp = NetworkTime.CurrentTimestamp()
        });
    }
}
```

## Troubleshooting

### Common Issues

#### 1. "Session Code Invalid" Error

```bash
# Symptom: Cannot join session, "Invalid session code" error

# Solution 1: Verify session code format
# Correct format: XXXX-XXXX-XXXX (12 characters with dashes)

# Solution 2: Check if host is still running
/synergy_status
# If host offline: "Error: Cannot connect to session"

# Solution 3: Verify network connectivity
/net_test <host_ip>
# Should show: "Connection successful: <latency>ms"

# Solution 4: Check firewall rules
# Ensure UDP port 7777 (or configured port) is open
```

#### 2. Inventory Desynchronization

```bash
# Symptom: Different item counts between players

# Solution 1: Check sync status
/sync_health
# Output shows: "Inventory hash mismatch detected"

# Solution 2: Force reconciliation
/force_resync inventory
# Wait for: "Inventory reconciliation complete"

# Solution 3: If persistent, restart session with backup
/restore_backup 0  # Most recent backup
```

#### 3. High Latency / Lag

```bash
# Symptom: Delayed actions, rubber-banding

# Check current latency
/synergy_status
# Look for: "Latency: 300ms+" (problematic)

# Solution 1: Adjust network settings in config
{
  "network": {
    "max_latency_ms": 200,  // Increase tolerance
    "packet_loss_tolerance": 0.1  // Allow more packet loss
  }
}

# Solution 2: Enable compression
/set_compression true

# Solution 3: Reduce sync frequency for non-critical data
{
  "sync_intervals": {
    "position_ms": 50,  // Increase from 33ms
    "inventory_ms": 500,  // Increase from 200ms
    "world_state_ms": 2000  // Increase from 1000ms
  }
}
```

#### 4. Base Parts Not Appearing for Other Players

```bash
# Symptom: One player places base parts, others don't see them

# Solution 1: Check base sync status
/debug_base
# Output: Lists all base components and which players see them

# Solution 2: Force base state resync
/force_resync base

# Solution 3: Verify collaborative_placement setting
{
  "base_building": {
    "collaborative_placement": true,  // Must be true
    "require_consensus": false
  }
}

# Solution 4: Check for conflicts
/base_conflicts
# Resolves any placement conflicts automatically
```

#### 5. Mod Fails to Load

```bash
# Symptom: BepInEx console shows no Deep Synergy messages

# Check BepInEx log
# Location: BepInEx/LogOutput.log

# Common causes:
# 1. Missing dependencies
#    Error: "Could not load [DeepSynergy.Core.dll]"
#    Solution: Reinstall mod, ensure all DLLs are present

# 2. Incompatible BepInEx version
#    Error: "BepInEx version mismatch"
#    Solution: Update to BepInEx 6.0.x

# 3. Game update broke IL2CPP hooks
#    Error: "Failed to hook [MethodName]"
#    Solution: Wait for mod update or downgrade game version

# 4. Conflicting mods
#    Disable other mods one by one to identify conflicts
```

### Debug Mode

Enable verbose logging for troubleshooting:

```json
{
  "debug": {
    "enabled": true,
    "log_level": "verbose",
    "log_network_traffic": true,
    "log_state_changes": true,
    "dump_sync_conflicts": true
  }
}
```

```bash
# Enable debug mode at runtime
/synergy_debug true

# View detailed sync logs
/show_sync_log

# Monitor network packets
/net_monitor true
# Output: Real-time packet flow with sizes and types
```

## Environment Variables

The mod respects these environment variables:

```bash
# OpenAI API key (for narrative generation)
export OPENAI_API_KEY=sk-your-key-here

# Anthropic Claude API key (for lore engine)
export ANTHROPIC_API_KEY=sk-ant-your-key-here

# Override config file location
export SYNERGY_CONFIG_PATH=/custom/path/synergy_profile.json

# Set log level without editing config
export SYNERGY_LOG_LEVEL=debug

# Force specific network port
export SYNERGY_PORT=8888

# Disable UPnP/NAT-PMP
export SYNERGY_NO_UPNP=1
```

## Performance Optimization

### For Low-End Systems

```json
{
  "performance": {
    "sync_intervals": {
      "position_ms": 100,
      "inventory_ms": 1000,
      "world_state_ms": 5000
    },
    "reduce_creature_ai_sync": true,
    "compress_network_traffic": true,
    "limit_visual_effects": true
  }
}
```

### For High-Performance Sessions (4+ players)

```json
{
  "performance": {
    "sync_intervals": {
      "position_ms": 33,
      "inventory_ms": 200,
      "world_state_ms": 1000
    },
    "predictive_sync": true,
    "delta_compression": true,
    "prioritize_nearby_players": true
  }
}
```

## Best Practices

1. **Always use session codes**: Avoid direct IP connections for better NAT traversal
2. **Configure difficulty scaling**: Set `difficulty_scale` to "adaptive" for balanced gameplay
3. **Enable session backups**: Set `auto_save_interval_seconds` to 60 or less
4. **Test network before inviting friends**: Use `/net_test` command
5. **Coordinate resource gathering**: Use `shared_blueprints: true` to avoid duplicate research
6. **Designate a stable host**: Player with best connection should host
7. **Use voice chat integration**: Set `voice_chat_integration` to "discord_rpc" for proximity audio

## Additional Resources

- **BepInEx Documentation**: https://docs.bepinex.dev/
- **Subnautica Modding Wiki**: Community-maintained mod compatibility lists
- **Discord Server**: 24/7 support and troubleshooting assistance
- **GitHub Issues**: Bug reports and feature requests (if public repository exists)
