---
name: subnautica-2-coop-mod
description: BepInEx multiplayer mod for Subnautica 2 enabling synchronized co-op survival with shared base building and real-time state synchronization
triggers:
  - install subnautica 2 multiplayer mod
  - setup subnautica coop mod
  - configure deep synergy multiplayer
  - host subnautica 2 session
  - troubleshoot subnautica mod connection
  - customize subnautica multiplayer settings
  - sync subnautica inventory state
  - configure bepinex subnautica plugin
---

# Subnautica 2 Co-op Mod Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using BepInEx plugin architecture. It implements peer-to-peer multiplayer with deterministic state synchronization, shared inventory systems, and adaptive difficulty scaling based on player count.

**Core Capabilities:**
- Peer-to-peer WebRTC connections with NAT traversal
- Real-time world state synchronization (base building, inventory, creature AI)
- Host migration and session persistence
- Adaptive difficulty scaling (2-4 players)
- Cross-platform support (Windows, Linux, macOS)

## Installation

### Prerequisites

1. **Install BepInEx 6.0.x** for Subnautica 2:
```bash
# Download from https://github.com/BepInEx/BepInEx/releases
# Extract to Subnautica 2 game directory
# Structure should be: Subnautica2/BepInEx/
```

2. **Verify BepInEx is working**:
   - Launch game once, check for `BepInEx/LogOutput.log`
   - Should see "Chainloader initialized" message

3. **Install Deep Synergy Mod**:
```bash
# Extract mod archive to:
# Subnautica2/BepInEx/plugins/DeepSynergy/
# Files: DeepSynergy.dll, config/synergy_profile.json
```

### Directory Structure
```
Subnautica2/
├── BepInEx/
│   ├── core/
│   ├── plugins/
│   │   └── DeepSynergy/
│   │       ├── DeepSynergy.dll
│   │       └── dependencies/
│   └── config/
│       └── synergy_profile.json
```

## Configuration

### Basic Profile Configuration

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
  "voice_chat_integration": "disabled",
  "network": {
    "port": 7777,
    "timeout_seconds": 30,
    "max_latency_ms": 250
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

### Configuration Fields Reference

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| `session_name` | string | Display name for session | "Co-op Session" |
| `max_players` | int | 2-4 players supported | 2 |
| `difficulty_scale` | enum | "adaptive", "fixed", "hardcore" | "adaptive" |
| `resource_multiplier` | float | Scales harvestable resources (0.5-2.0) | 1.0 |
| `oxygen_consumption` | float | O2 drain multiplier (0.5-1.5) | 1.0 |
| `shared_blueprints` | bool | Unlocks shared across players | true |
| `time_of_day_sync` | enum | "host", "all", "independent" | "host" |

## Console Commands

The mod adds in-game console commands accessible via BepInEx terminal (F5 by default):

### Session Management
```bash
# Host a new session
/start_server

# Join existing session
/join_session <session-code>
# Example: /join_session 9B2A-4C7D-E8F1

# Leave current session
/leave_session

# Force disconnect player (host only)
/kick_player <player-id>
```

### Session Monitoring
```bash
# Show connection status
/synergy_status
# Output:
# Connected peers: 3
# State sync: 100% complete
# Inventory hash: 0xFA342B1E
# Avg latency: 45ms

# List connected players
/list_players
# Output:
# [1] PlayerName (host) - 35ms
# [2] Player2 - 52ms
# [3] Player3 - 48ms
```

### Gameplay Adjustments
```bash
# Temporarily adjust difficulty
/synergy_scale <multiplier>
# Example: /synergy_scale 1.5

# Override world seed (must do before starting)
/seed_override <seed-number>
# Example: /seed_override 8251

# Force state resync
/force_sync

# Debug inventory state
/debug_inventory
```

### AI Integration (if enabled)
```bash
# Trigger narrative generation
/api_narrate "<context>"
# Example: /api_narrate "exploring thermal vents"

# Generate species lore
/api_lore_species <creature-name>
# Example: /api_lore_species "ghost_leviathan"
```

## Code Examples

### Creating a Custom Mod Plugin (C#)

If extending the Deep Synergy mod with custom functionality:

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using DeepSynergy.Core;
using UnityEngine;

namespace MyCustomExtension
{
    [BepInPlugin("com.example.subnautica2.extension", "Custom Extension", "1.0.0")]
    [BepInDependency("com.deepsynergy.subnautica2", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomPlugin : BasePlugin
    {
        public override void Load()
        {
            // Hook into Deep Synergy events
            SessionManager.OnPlayerJoined += HandlePlayerJoin;
            SessionManager.OnStateSync += HandleStateSync;
            
            Log.LogInfo("Custom Extension loaded");
        }

        private void HandlePlayerJoin(PlayerInfo player)
        {
            Log.LogInfo($"Player {player.Name} joined with ID {player.Id}");
            
            // Send custom welcome message
            NetworkMessenger.SendToPlayer(player.Id, new WelcomeMessage
            {
                Text = $"Welcome {player.Name}!",
                Color = Color.cyan
            });
        }

        private void HandleStateSync(SyncEvent syncEvent)
        {
            // React to state changes
            if (syncEvent.Type == SyncType.InventoryUpdate)
            {
                Log.LogInfo($"Inventory synced: {syncEvent.Data}");
            }
        }
    }
}
```

### Custom Network Message Handler

```csharp
using DeepSynergy.Networking;

public class CustomMessageHandler : INetworkMessageHandler
{
    public void RegisterHandlers()
    {
        NetworkManager.RegisterHandler<CustomDataPacket>(OnCustomData);
    }

    private void OnCustomData(CustomDataPacket packet)
    {
        // Process custom multiplayer data
        Debug.Log($"Received from {packet.SenderId}: {packet.Payload}");
        
        // Broadcast to other clients
        if (NetworkManager.IsHost)
        {
            NetworkManager.BroadcastExcept(packet, packet.SenderId);
        }
    }
}

[Serializable]
public class CustomDataPacket : INetworkPacket
{
    public string SenderId { get; set; }
    public string Payload { get; set; }
    public long Timestamp { get; set; }
}
```

### Session Configuration Override at Runtime

```csharp
using DeepSynergy.Config;

public class DynamicConfigManager
{
    public static void AdjustDifficulty(float multiplier)
    {
        var config = SessionConfig.Current;
        config.ResourceMultiplier *= multiplier;
        config.CreatureSpawnDivider = (int)(config.CreatureSpawnDivider / multiplier);
        
        SessionConfig.ApplyChanges(config);
        NetworkManager.SyncConfig(); // Broadcast to all clients
    }

    public static void EnableAPI(string provider)
    {
        var config = SessionConfig.Current;
        
        if (provider == "openai")
        {
            config.ApiIntegration.OpenAI.Enabled = true;
            config.ApiIntegration.OpenAI.ApiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
        }
        else if (provider == "claude")
        {
            config.ApiIntegration.Claude.Enabled = true;
            config.ApiIntegration.Claude.ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY");
        }
        
        SessionConfig.ApplyChanges(config);
    }
}
```

## Common Patterns

### Hosting a Session

```bash
# 1. Configure profile
vim BepInEx/config/synergy_profile.json

# 2. Launch game, open console (F5)
/start_server

# 3. Share session code with players
# Output: Session code: 9B2A-4C7D-E8F1

# 4. Wait for players to join
/list_players

# 5. Start playing when ready
```

### Joining a Session

```bash
# 1. Launch game, open console
/join_session 9B2A-4C7D-E8F1

# 2. Wait for state sync
/synergy_status
# Should show "State sync: 100% complete"

# 3. Verify connection
/list_players
```

### Handling Disconnects

The mod automatically handles host migration:

```csharp
// Host migration is automatic, but you can monitor it:
SessionManager.OnHostMigration += (newHostId) =>
{
    Debug.Log($"Host migrated to player {newHostId}");
    // UI notification, state verification, etc.
};

// Manual reconnection attempt
if (!NetworkManager.IsConnected)
{
    NetworkManager.AttemptReconnect(lastSessionCode);
}
```

### Custom Difficulty Presets

```json
{
  "presets": {
    "easy": {
      "resource_multiplier": 1.5,
      "oxygen_consumption": 0.7,
      "creature_spawn_divider": 2
    },
    "normal": {
      "resource_multiplier": 1.0,
      "oxygen_consumption": 1.0,
      "creature_spawn_divider": 1
    },
    "hardcore": {
      "resource_multiplier": 0.7,
      "oxygen_consumption": 1.3,
      "creature_spawn_divider": 0.8,
      "friendly_fire": true
    }
  },
  "active_preset": "normal"
}
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot join session / "Connection timeout"

**Solutions:**
```bash
# 1. Check firewall rules
sudo ufw allow 7777/udp  # Linux
# Windows: Allow BepInEx through Windows Defender Firewall

# 2. Verify NAT configuration
# Router should allow UDP hole punching
# Or manually forward port 7777 to host machine

# 3. Check console for errors
/synergy_status
# Look for "NAT traversal failed" or "Peer unreachable"

# 4. Increase timeout
# Edit synergy_profile.json:
"network": {
  "timeout_seconds": 60,
  "max_latency_ms": 500
}

# 5. Test direct connection (bypass NAT)
/join_session <host-ip>:7777
```

### State Desync

**Problem:** Players see different base structures / inventory

**Solutions:**
```bash
# 1. Force resynchronization
/force_sync

# 2. Check hash integrity
/debug_inventory
# Compare hashes between clients

# 3. Restart session if corruption detected
/leave_session
# Host: /start_server (with same seed)
# Clients: /join_session <new-code>

# 4. Clear cache (last resort)
rm -rf BepInEx/cache/synergy_state.dat
```

### Performance Degradation

**Problem:** Lag spikes, FPS drops with multiple players

**Solutions:**
```bash
# 1. Reduce sync frequency (trade responsiveness for performance)
# Edit synergy_profile.json:
"network": {
  "sync_interval_ms": 100,  # Default: 50ms
  "state_compression": true
}

# 2. Limit creature AI sync
"creature_sync_range": 100,  # Only sync creatures within 100m

# 3. Monitor bandwidth usage
/synergy_status
# Look for "Bandwidth: X MB/s"

# 4. Disable API integrations if enabled
"api_integration": {
  "openai": { "enabled": false },
  "claude": { "enabled": false }
}
```

### Mod Conflicts

**Problem:** Crashes with other BepInEx mods

**Solutions:**
```bash
# 1. Check BepInEx logs
tail -f BepInEx/LogOutput.log

# 2. Disable conflicting mods
# Common conflicts: other multiplayer mods, world-state mods
mv BepInEx/plugins/ConflictingMod.dll BepInEx/plugins/disabled/

# 3. Load order matters - ensure Deep Synergy loads first
# Rename: DeepSynergy.dll -> 00_DeepSynergy.dll

# 4. Report incompatibilities
# Check wiki: https://github.com/maglin-jenebellah/Subnautica-II-Coop-Client/wiki
```

### API Integration Errors

**Problem:** OpenAI/Claude features not working

**Solutions:**
```bash
# 1. Verify environment variables
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# 2. Test API connectivity
/api_narrate "test connection"
# Check logs for HTTP errors

# 3. Check rate limits
# APIs have usage quotas - verify account status

# 4. Fallback to local mode
"api_integration": {
  "openai": { "enabled": false },
  "fallback_to_local": true
}
```

## Advanced Configuration

### Custom Network Transport

```csharp
using DeepSynergy.Networking;

public class CustomTransportLayer : INetworkTransport
{
    public void Initialize(TransportConfig config)
    {
        // Implement custom UDP/TCP logic
        // Example: Use QUIC instead of WebRTC
    }

    public void Send(byte[] data, string targetId)
    {
        // Custom send implementation
    }

    public void Broadcast(byte[] data)
    {
        // Broadcast to all peers
    }
}

// Register in plugin Load()
NetworkManager.RegisterTransport(new CustomTransportLayer());
```

### Event Hooks

```csharp
// Available event hooks
SessionManager.OnPlayerJoined += (player) => { /* ... */ };
SessionManager.OnPlayerLeft += (playerId) => { /* ... */ };
SessionManager.OnStateSync += (syncEvent) => { /* ... */ };
SessionManager.OnHostMigration += (newHostId) => { /* ... */ };
InventoryManager.OnItemPickup += (item, playerId) => { /* ... */ };
BaseBuilder.OnStructurePlaced += (structure, playerId) => { /* ... */ };
```

## Best Practices

1. **Always verify state sync** before critical actions (e.g., blueprint unlocks)
2. **Use console commands** for debugging before modifying config files
3. **Back up save files** before enabling PvP or hardcore modes
4. **Monitor network stats** with `/synergy_status` regularly
5. **Disable API integrations** if not needed (reduces overhead)
6. **Test with 2 players first** before scaling to 3-4
7. **Use environment variables** for API keys, never hardcode

## Resources

- **Wiki:** Project documentation and compatibility lists
- **Discord:** Real-time community support
- **Issue Tracker:** Bug reports and feature requests
- **BepInEx Docs:** https://docs.bepinex.dev/
