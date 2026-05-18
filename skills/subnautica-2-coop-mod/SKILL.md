---
name: subnautica-2-coop-mod
description: BepInEx multiplayer mod for Subnautica 2 enabling synchronized co-op gameplay with deterministic session management and cross-platform support
triggers:
  - how do I set up the Subnautica 2 co-op mod
  - install Deep Synergy multiplayer for Subnautica
  - configure Subnautica 2 multiplayer session
  - troubleshoot Subnautica coop connection issues
  - create a synergy profile for Subnautica multiplayer
  - use BepInEx with Subnautica 2 mod
  - connect players in Subnautica 2 co-op
  - customize multiplayer settings for Subnautica mod
---

# Subnautica 2 Deep Synergy Co-op Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience. Built on BepInEx, it implements deterministic session synchronization, adaptive difficulty scaling, shared inventory management, and cross-platform WebRTC-based matchmaking without requiring central servers.

**Key Architecture:**
- **Deterministic Session Synchronization (DSS)**: Conflict-resolved timestamped events prevent desynchronization
- **Adaptive Dynamic Scaling (ADS)**: Player count adjusts creature spawns, resources, and difficulty
- **BepInEx Plugin Architecture**: IL2CPP hooks with hot-reloadable modules
- **Decentralized P2P**: NAT punch-through with WebRTC data channels
- **Merkle Tree Inventory**: Hash-based inventory verification across peers

## Installation

### Prerequisites

1. **Install BepInEx 6.0.x** for Subnautica 2:
   ```bash
   # Download BepInEx from https://github.com/BepInEx/BepInEx/releases
   # Extract to Subnautica 2 root directory
   # Structure should be: Subnautica2/BepInEx/
   ```

2. **Install the Deep Synergy Mod**:
   ```bash
   # Download from project repository
   # Extract to: Subnautica2/BepInEx/plugins/DeepSynergy/
   ```

3. **Directory Structure**:
   ```
   Subnautica2/
   ├── BepInEx/
   │   ├── core/
   │   ├── plugins/
   │   │   └── DeepSynergy/
   │   │       ├── DeepSynergy.dll
   │   │       ├── WebRTCNative.dll
   │   │       └── StateSync.dll
   │   └── config/
   │       └── synergy_profile.json
   ```

## Configuration

### Basic Profile Setup

Create `BepInEx/config/synergy_profile.json`:

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
  "voice_chat_integration": "discord_rpc",
  "network": {
    "port": 7777,
    "use_upnp": true,
    "max_latency_ms": 200,
    "compression": true
  },
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
  }
}
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `session_name` | string | Display name for multiplayer session |
| `max_players` | int | 2-8 player limit |
| `difficulty_scale` | string | `"static"` or `"adaptive"` scaling |
| `resource_multiplier` | float | Harvest node multiplier (0.5-2.0) |
| `oxygen_consumption` | float | O2 drain rate (0.5-1.5) |
| `creature_spawn_divider` | int | Reduces spawns (2 = half spawns) |
| `shared_blueprints` | bool | Unlocks propagate to all players |
| `time_of_day_sync` | string | `"host"`, `"vote"`, or `"all"` |

### Advanced Network Configuration

```json
{
  "network": {
    "port": 7777,
    "use_upnp": true,
    "stun_servers": [
      "stun:stun.l.google.com:19302",
      "stun:stun1.l.google.com:19302"
    ],
    "turn_servers": [],
    "max_latency_ms": 200,
    "compression": true,
    "tick_rate": 20,
    "state_sync_interval_ms": 100
  }
}
```

## Console Commands

Access via BepInEx console (`F12` in-game by default):

### Session Management

```bash
# Host a new session
/start_server

# Join existing session
/join_session <session-code>
# Example: /join_session 9B2A-4C7D-E8F1

# Check connection status
/synergy_status

# Disconnect from session
/disconnect

# Kick player (host only)
/kick <player_name>
```

### Difficulty & Scaling

```bash
# Adjust difficulty multiplier (temporary)
/synergy_scale <float>
# Example: /synergy_scale 1.5

# Set resource multiplier
/resource_mult <float>
# Example: /resource_mult 0.8

# Override world seed (requires session restart)
/seed_override <int>
# Example: /seed_override 8251
```

### Debugging & Diagnostics

```bash
# View inventory sync status
/inventory_status

# Show network metrics
/net_stats

# Force state resynchronization
/force_sync

# Export session log
/export_log
```

### API Integration

```bash
# Trigger AI narration (requires OpenAI/Claude)
/api_narrate "<event_description>"
# Example: /api_narrate "discovered thermal vent in lost river"

# Generate creature lore
/api_lore <creature_name>
# Example: /api_lore "reaper_leviathan"
```

## Code Examples

### Plugin Development: Custom Event Hook

```csharp
using BepInEx;
using DeepSynergy.Core;
using DeepSynergy.Events;
using HarmonyLib;

namespace MyCustomPlugin
{
    [BepInPlugin("com.example.customhook", "Custom Event Hook", "1.0.0")]
    [BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomEventPlugin : BaseUnityPlugin
    {
        private void Awake()
        {
            // Subscribe to item pickup synchronization
            SynergyEventBus.OnItemPickup += HandleItemPickup;
            
            // Subscribe to base construction events
            SynergyEventBus.OnBasePartPlaced += HandleBasePart;
            
            Logger.LogInfo("Custom Event Hook initialized");
        }

        private void HandleItemPickup(ItemPickupEvent evt)
        {
            Logger.LogInfo($"Player {evt.PlayerId} picked up {evt.ItemId}");
            
            // Custom logic: announce rare item pickups
            if (evt.ItemRarity == Rarity.Legendary)
            {
                SynergyNetwork.BroadcastMessage(
                    $"{evt.PlayerName} found a {evt.ItemName}!",
                    MessagePriority.High
                );
            }
        }

        private void HandleBasePart(BasePartEvent evt)
        {
            // Validate structural integrity
            if (!ValidateBasePlacement(evt.Position, evt.Rotation))
            {
                evt.Cancel = true;
                SynergyNetwork.SendToPlayer(
                    evt.PlayerId,
                    "Invalid placement: structural instability detected"
                );
            }
        }

        private bool ValidateBasePlacement(Vector3 pos, Quaternion rot)
        {
            // Custom validation logic
            return Physics.Raycast(pos, Vector3.down, 10f);
        }
    }
}
```

### State Synchronization: Custom Object

```csharp
using DeepSynergy.Sync;
using UnityEngine;

public class CustomBeacon : SyncedMonoBehaviour
{
    [SyncVar] // Automatically synchronized across clients
    public string beaconLabel;
    
    [SyncVar]
    public Color beaconColor;
    
    [SyncVar]
    public float signalRadius = 50f;

    // Called on all clients when SyncVar changes
    protected override void OnSyncVarUpdated(string varName, object value)
    {
        switch (varName)
        {
            case nameof(beaconLabel):
                UpdateBeaconUI();
                break;
            case nameof(beaconColor):
                UpdateBeaconMaterial();
                break;
        }
    }

    // Client RPC - executed on all clients
    [ClientRpc]
    public void RpcPlayActivationSound()
    {
        AudioSource.PlayClipAtPoint(activationSound, transform.position);
    }

    // Command - executed on host, called from client
    [Command]
    public void CmdSetLabel(string newLabel)
    {
        if (!SynergyAuth.IsAuthorized(Context.SenderId, AuthLevel.Standard))
            return;
            
        beaconLabel = newLabel; // Automatically syncs
        RpcPlayActivationSound();
    }

    private void UpdateBeaconUI()
    {
        GetComponent<TextMesh>().text = beaconLabel;
    }

    private void UpdateBeaconMaterial()
    {
        GetComponent<Renderer>().material.color = beaconColor;
    }
}
```

### Inventory Synchronization

```csharp
using DeepSynergy.Inventory;

public class SharedStorageController : MonoBehaviour
{
    private SyncedInventory sharedStorage;

    private void Start()
    {
        // Create shared inventory accessible by all players
        sharedStorage = SyncedInventory.Create(
            capacity: 48,
            storageId: "base_locker_01",
            accessLevel: AccessLevel.All
        );

        // Subscribe to inventory events
        sharedStorage.OnItemAdded += HandleItemAdded;
        sharedStorage.OnItemRemoved += HandleItemRemoved;
    }

    public void AddItem(string itemId, int quantity)
    {
        // Automatically synced across all clients
        sharedStorage.AddItem(itemId, quantity);
    }

    public void RemoveItem(string itemId, int quantity)
    {
        if (sharedStorage.HasItem(itemId, quantity))
        {
            sharedStorage.RemoveItem(itemId, quantity);
        }
    }

    private void HandleItemAdded(ItemEvent evt)
    {
        Debug.Log($"{evt.PlayerName} added {evt.Quantity}x {evt.ItemId}");
        
        // Verify integrity via Merkle tree
        if (!sharedStorage.VerifyIntegrity())
        {
            Debug.LogError("Inventory desync detected - requesting full sync");
            sharedStorage.RequestFullSync();
        }
    }

    private void HandleItemRemoved(ItemEvent evt)
    {
        Debug.Log($"{evt.PlayerName} removed {evt.Quantity}x {evt.ItemId}");
    }
}
```

### Session Creation & Joining

```csharp
using DeepSynergy.Network;

public class MultiplayerManager : MonoBehaviour
{
    public void HostSession()
    {
        var config = SessionConfig.LoadFromFile("synergy_profile.json");
        
        var session = SynergyServer.CreateSession(config);
        
        session.OnPlayerJoined += (player) => {
            Debug.Log($"{player.Name} joined the session");
            NotifyAllPlayers($"{player.Name} has entered the waters");
        };

        session.OnPlayerLeft += (player) => {
            Debug.Log($"{player.Name} disconnected");
        };

        string sessionCode = session.GetSessionCode();
        Debug.Log($"Session created: {sessionCode}");
    }

    public async void JoinSession(string sessionCode)
    {
        try
        {
            var session = await SynergyClient.JoinSession(sessionCode);
            
            Debug.Log($"Connected to {session.Name}");
            Debug.Log($"Players: {session.PlayerCount}/{session.MaxPlayers}");
            
            // Wait for initial state sync
            await session.WaitForInitialSync();
            
            Debug.Log("State synchronized - ready to play");
        }
        catch (SessionNotFoundException)
        {
            Debug.LogError($"Session {sessionCode} not found");
        }
        catch (SessionFullException)
        {
            Debug.LogError("Session is full");
        }
    }

    private void NotifyAllPlayers(string message)
    {
        SynergyNetwork.BroadcastMessage(message, MessagePriority.Normal);
    }
}
```

## Common Patterns

### 1. Session Migration (Host Disconnection Recovery)

```csharp
SynergyServer.OnHostDisconnected += () => {
    // Automatically migrate to next available client
    var newHost = SynergyServer.ElectNewHost();
    Debug.Log($"Host migrated to {newHost.Name}");
    
    // Preserve session state
    SynergyServer.MigrateSession(newHost);
};
```

### 2. Conflict Resolution

```csharp
// Custom conflict resolver for simultaneous item pickups
SynergySync.RegisterConflictResolver<ItemPickupEvent>((events) => {
    // Earliest timestamp wins
    var winner = events.OrderBy(e => e.Timestamp).First();
    
    // Notify losers
    foreach (var evt in events.Where(e => e != winner))
    {
        SynergyNetwork.SendToPlayer(
            evt.PlayerId,
            $"Item already taken by {winner.PlayerName}"
        );
    }
    
    return winner;
});
```

### 3. Adaptive Difficulty Scaling

```csharp
public class DifficultyManager : MonoBehaviour
{
    private void Update()
    {
        int playerCount = SynergyServer.GetPlayerCount();
        
        // Scale creature spawns
        CreatureSpawner.SetSpawnMultiplier(1.0f / playerCount);
        
        // Adjust resource availability
        ResourceManager.SetResourceMultiplier(
            Mathf.Lerp(1.0f, 1.5f, playerCount / 8f)
        );
        
        // Scale oxygen consumption
        OxygenSystem.SetConsumptionRate(
            Mathf.Lerp(1.0f, 0.7f, playerCount / 8f)
        );
    }
}
```

## Troubleshooting

### Connection Issues

**Symptom**: Cannot connect to session

```bash
# Check network configuration
/net_stats

# Verify ports are open (default: 7777)
netstat -an | grep 7777

# Test STUN connectivity
/test_stun

# Enable verbose logging
/set_log_level debug
```

**Solution**: Ensure UPnP is enabled or manually forward port 7777 (UDP/TCP)

### State Desynchronization

**Symptom**: Players see different base structures or inventory contents

```csharp
// Force full state resync
SynergySync.RequestFullResync();

// Check Merkle tree integrity
bool isValid = SynergyInventory.VerifyAllInventories();
if (!isValid)
{
    Debug.LogError("Inventory integrity check failed");
    SynergyInventory.RebuildMerkleTrees();
}

// Validate position sync
foreach (var player in SynergyServer.GetAllPlayers())
{
    float latency = SynergyNetwork.GetPlayerLatency(player.Id);
    if (latency > 200)
    {
        Debug.LogWarning($"{player.Name} high latency: {latency}ms");
    }
}
```

### Performance Degradation

**Symptom**: FPS drops with multiple players

```json
{
  "network": {
    "tick_rate": 10,
    "state_sync_interval_ms": 200,
    "compression": true,
    "lod_distance_multiplier": 1.5
  }
}
```

Reduce tick rate and increase sync interval to lower bandwidth usage.

### API Integration Errors

**Symptom**: OpenAI/Claude features not working

```bash
# Verify environment variables
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# Test API connectivity
/api_test openai
/api_test claude

# Check API usage
/api_status
```

Ensure API keys are set and have sufficient quota.

### BepInEx Load Failures

**Symptom**: Mod not loading in BepInEx

```bash
# Check BepInEx console log
cat BepInEx/LogOutput.log | grep DeepSynergy

# Verify plugin dependencies
# Ensure IL2CPP support is enabled in BepInEx config:
# BepInEx/config/BepInEx.cfg
[Preloader]
Type = IL2CPP
```

## Environment Variables

```bash
# API Keys (optional)
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-ant-...

# Network Configuration
export SYNERGY_PORT=7777
export SYNERGY_MAX_PLAYERS=4

# Debug Mode
export SYNERGY_DEBUG=true
export SYNERGY_LOG_LEVEL=debug
```

## File Locations

- **Config**: `BepInEx/config/synergy_profile.json`
- **Logs**: `BepInEx/LogOutput.log`
- **Plugins**: `BepInEx/plugins/DeepSynergy/`
- **Localization**: `BepInEx/plugins/DeepSynergy/localizations/`
- **Session Cache**: `BepInEx/cache/sessions/`
