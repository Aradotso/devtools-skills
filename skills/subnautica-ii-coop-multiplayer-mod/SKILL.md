---
name: subnautica-ii-coop-multiplayer-mod
description: BepInEx-based multiplayer mod for Subnautica 2 enabling cooperative gameplay with synchronized sessions, shared inventory, and adaptive difficulty scaling
triggers:
  - how do I set up Subnautica 2 multiplayer mod
  - configure Deep Synergy mod for co-op
  - install BepInEx Subnautica 2 multiplayer
  - troubleshoot Subnautica co-op connection issues
  - create multiplayer session in Subnautica 2
  - sync inventory across Subnautica players
  - adjust difficulty scaling for Subnautica co-op
  - configure session settings for Subnautica multiplayer
---

# Subnautica II Co-op Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What This Project Does

The **Deep Synergy Multiplayer Mod** transforms Subnautica 2 into a synchronized cooperative experience. Built on BepInEx, it implements:

- **Deterministic Session Synchronization (DSS)**: Timestamped event resolution for shared world state
- **Adaptive Dynamic Scaling (ADS)**: Auto-adjusts creature spawns, resources, and oxygen based on player count
- **Shared Inventory System**: Merkle tree-based inventory tracking with peer verification
- **Cross-Platform Connectivity**: NAT punch-through and WebRTC for Windows/Linux/macOS
- **Session Migration**: Zero data loss if host disconnects
- **Console Command Interface**: Advanced session management and debugging

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam or GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### Steps

```bash
# 1. Install BepInEx (if not already installed)
# Download BepInEx_UnityIL2CPP_x64_6.0.0.zip from https://github.com/BepInEx/BepInEx/releases

# 2. Extract BepInEx to Subnautica 2 root directory
# Structure should be:
# Subnautica2/
#   ├── BepInEx/
#   │   ├── core/
#   │   ├── plugins/
#   │   └── config/
#   ├── Subnautica2.exe
#   └── ...

# 3. Download Deep Synergy Mod
# Extract to BepInEx/plugins/DeepSynergy/

# 4. Launch game once to generate default config
# Located at: BepInEx/config/synergy_profile.json
```

### Directory Structure

```
Subnautica2/
└── BepInEx/
    ├── plugins/
    │   └── DeepSynergy/
    │       ├── DeepSynergy.dll
    │       ├── WebRTC.dll
    │       └── StateSync.dll
    └── config/
        ├── synergy_profile.json
        └── session_config.xml
```

## Configuration

### Basic Profile Setup (`synergy_profile.json`)

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
  "time_of_day_sync": "host",
  "voice_chat_integration": "none",
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
    "nat_punchthrough": true,
    "max_latency_ms": 150
  },
  "sync": {
    "tick_rate": 20,
    "inventory_sync_interval_ms": 500,
    "creature_ai_sync_interval_ms": 1000,
    "base_structure_sync_interval_ms": 2000
  }
}
```

### Key Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `max_players` | int | 2-8 players supported |
| `difficulty_scale` | string | `"static"`, `"adaptive"`, `"custom"` |
| `resource_multiplier` | float | Multiplies harvestable resources (0.5 = half, 2.0 = double) |
| `oxygen_consumption` | float | Fraction of normal oxygen drain (0.5 = slower, 1.5 = faster) |
| `creature_spawn_divider` | int | Divides creature spawn counts (2 = half as many) |
| `shared_blueprints` | bool | Blueprint unlocks shared across all players |
| `time_of_day_sync` | string | `"host"`, `"all"`, `"independent"` |

## Console Commands

Access via BepInEx console (F1 by default):

```bash
# Start hosting a session
/start_server

# Join existing session with code
/join_session 9B2A-4C7D-E8F1

# Show connection status
/synergy_status

# Adjust difficulty scaling mid-session
/synergy_scale 1.5

# Force world seed synchronization
/seed_override 8251

# Trigger AI narrative (if enabled)
/api_narrate "discovering the thermal vents"

# Manual inventory sync (debug)
/force_inventory_sync

# Disconnect and return to solo
/disconnect_session

# Show latency to all peers
/ping_all

# Export session state (backup)
/export_state session_backup.dat
```

## Common Usage Patterns

### Hosting a Session

```csharp
// Example: Programmatic session creation (C# mod extension)
using DeepSynergy.Core;
using DeepSynergy.Network;

public class CustomSessionHost : MonoBehaviour
{
    private SessionManager sessionManager;
    
    void Start()
    {
        sessionManager = SessionManager.Instance;
        
        var config = new SessionConfig
        {
            SessionName = "Custom Session",
            MaxPlayers = 4,
            DifficultyScale = DifficultyMode.Adaptive,
            ResourceMultiplier = 1.2f,
            SharedBlueprints = true
        };
        
        sessionManager.CreateSession(config, OnSessionCreated);
    }
    
    void OnSessionCreated(SessionCode code)
    {
        Debug.Log($"Session created: {code.ToString()}");
        // Display code to player UI
        UIManager.ShowSessionCode(code);
    }
}
```

### Joining a Session

```csharp
using DeepSynergy.Core;

public class SessionJoiner : MonoBehaviour
{
    public void JoinByCode(string codeString)
    {
        if (SessionCode.TryParse(codeString, out SessionCode code))
        {
            SessionManager.Instance.JoinSession(code, OnJoinSuccess, OnJoinFailed);
        }
    }
    
    void OnJoinSuccess()
    {
        Debug.Log("Successfully joined session");
        // Trigger inventory sync
        InventorySync.Instance.RequestFullSync();
    }
    
    void OnJoinFailed(string error)
    {
        Debug.LogError($"Failed to join: {error}");
        UIManager.ShowError(error);
    }
}
```

### Synchronizing Custom Game State

```csharp
using DeepSynergy.Sync;

public class CustomBaseComponent : MonoBehaviour, ISyncable
{
    [SyncVar] // Auto-synced across clients
    private int powerLevel;
    
    [SyncVar]
    private Vector3 constructionPosition;
    
    public void SetPowerLevel(int level)
    {
        if (NetworkAuthority.IsLocalAuthority(this))
        {
            powerLevel = level;
            StateSynchronizer.Instance.MarkDirty(this);
        }
    }
    
    // Called by sync system when remote data arrives
    public void OnRemoteUpdate(byte[] data)
    {
        using (var reader = new NetworkReader(data))
        {
            powerLevel = reader.ReadInt32();
            constructionPosition = reader.ReadVector3();
        }
    }
    
    // Serialize local state for transmission
    public byte[] Serialize()
    {
        using (var writer = new NetworkWriter())
        {
            writer.WriteInt32(powerLevel);
            writer.WriteVector3(constructionPosition);
            return writer.ToArray();
        }
    }
}
```

### Handling Session Events

```csharp
using DeepSynergy.Events;

public class SessionEventHandler : MonoBehaviour
{
    void OnEnable()
    {
        SessionEvents.OnPlayerJoined += HandlePlayerJoined;
        SessionEvents.OnPlayerLeft += HandlePlayerLeft;
        SessionEvents.OnHostMigration += HandleHostMigration;
        SessionEvents.OnSyncConflict += HandleSyncConflict;
    }
    
    void OnDisable()
    {
        SessionEvents.OnPlayerJoined -= HandlePlayerJoined;
        SessionEvents.OnPlayerLeft -= HandlePlayerLeft;
        SessionEvents.OnHostMigration -= HandleHostMigration;
        SessionEvents.OnSyncConflict -= HandleSyncConflict;
    }
    
    void HandlePlayerJoined(PlayerInfo player)
    {
        Debug.Log($"Player {player.Name} joined (ID: {player.Id})");
        ChatManager.SendSystemMessage($"{player.Name} joined the session");
    }
    
    void HandlePlayerLeft(PlayerInfo player)
    {
        Debug.Log($"Player {player.Name} left");
    }
    
    void HandleHostMigration(PlayerInfo newHost)
    {
        Debug.Log($"Host migrated to {newHost.Name}");
        // Session continues without data loss
    }
    
    void HandleSyncConflict(SyncConflict conflict)
    {
        // Automatic resolution via timestamp authority
        Debug.LogWarning($"Sync conflict resolved: {conflict.Description}");
    }
}
```

## API Integration (Optional)

### OpenAI Narrator

```json
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "api_key_env": "OPENAI_API_KEY",
      "role": "narrator",
      "model": "gpt-4",
      "max_tokens": 150,
      "temperature": 0.7
    }
  }
}
```

Set environment variable:
```bash
export OPENAI_API_KEY=your_key_here
```

Trigger narration:
```bash
/api_narrate "exploring the kelp forest at night"
```

### Claude Lore Engine

```json
{
  "api_integration": {
    "claude": {
      "enabled": true,
      "api_key_env": "ANTHROPIC_API_KEY",
      "role": "lore_engine",
      "model": "claude-3-opus-20240229",
      "max_tokens": 200
    }
  }
}
```

Set environment variable:
```bash
export ANTHROPIC_API_KEY=your_key_here
```

## Troubleshooting

### Connection Issues

**Problem**: "Failed to establish connection" error

**Solutions**:
```bash
# 1. Check firewall (allow UDP port 7777)
sudo ufw allow 7777/udp

# 2. Enable UPnP in router settings

# 3. Verify NAT type in config
# Edit synergy_profile.json:
{
  "network": {
    "nat_punchthrough": true,
    "use_upnp": true,
    "force_relay": false  # Set to true if NAT traversal fails
  }
}

# 4. Check console for detailed error
/synergy_status
```

### Inventory Desync

**Problem**: Items appear in one client but not others

**Solutions**:
```bash
# Force manual sync
/force_inventory_sync

# Check sync status
/synergy_status

# Verify hash consistency
# Console output should show matching hashes:
# Inventory hash: 0xFA342B1E (all clients should match)
```

**Prevention** (in code):
```csharp
// Always use synchronized pickup
using DeepSynergy.Inventory;

void PickupItem(GameObject item)
{
    if (SyncedInventory.TryPickup(item, out InventoryResult result))
    {
        Debug.Log($"Picked up {item.name} - synced across all clients");
    }
    else
    {
        Debug.LogWarning($"Pickup failed: {result.Error}");
    }
}
```

### High Latency

**Problem**: Lag or delayed actions

**Solutions**:
```json
{
  "sync": {
    "tick_rate": 10,  // Reduce from 20 for slower connections
    "inventory_sync_interval_ms": 1000,  // Increase intervals
    "creature_ai_sync_interval_ms": 2000
  },
  "network": {
    "max_latency_ms": 300,  // Increase tolerance
    "compression": true  // Enable packet compression
  }
}
```

### Session Migration Failure

**Problem**: Session crashes when host disconnects

**Solutions**:
```bash
# Enable auto-backup before starting session
/export_state auto_backup.dat

# Configure migration priority in synergy_profile.json:
{
  "session": {
    "migration_enabled": true,
    "migration_priority": "highest_bandwidth",  # or "longest_connected"
    "backup_interval_sec": 60
  }
}
```

### BepInEx Not Loading Mod

**Problem**: Mod doesn't appear in game

**Solutions**:
```bash
# 1. Check BepInEx console output (BepInEx/LogOutput.log)
cat BepInEx/LogOutput.log | grep DeepSynergy

# 2. Verify plugin location
ls -la BepInEx/plugins/DeepSynergy/

# 3. Ensure DLL is not blocked (Windows)
# Right-click DeepSynergy.dll > Properties > Unblock

# 4. Check BepInEx config allows plugins
# Edit BepInEx/config/BepInEx.cfg:
[Logging.Console]
Enabled = true

[Preloader]
Enabled = true
```

### Creature AI Desync

**Problem**: Creatures appear in different locations for different players

**Solutions**:
```csharp
// Ensure using synced creature spawner
using DeepSynergy.Creatures;

public class CreatureSpawner : MonoBehaviour
{
    void SpawnCreature(Vector3 position)
    {
        // Wrong - local only
        // GameObject creature = Instantiate(creaturePrefab, position, Quaternion.identity);
        
        // Correct - synced across all clients
        SyncedCreatureManager.Instance.SpawnCreature(
            creaturePrefab,
            position,
            Quaternion.identity,
            onSpawned: (creature) => {
                Debug.Log($"Creature spawned and synced: {creature.GetInstanceID()}");
            }
        );
    }
}
```

## Performance Optimization

### Recommended Settings for Different Player Counts

**2 Players**:
```json
{
  "resource_multiplier": 1.2,
  "creature_spawn_divider": 1,
  "sync": {
    "tick_rate": 20
  }
}
```

**4 Players**:
```json
{
  "resource_multiplier": 1.8,
  "creature_spawn_divider": 2,
  "sync": {
    "tick_rate": 15,
    "compression": true
  }
}
```

**8 Players** (experimental):
```json
{
  "resource_multiplier": 2.5,
  "creature_spawn_divider": 3,
  "sync": {
    "tick_rate": 10,
    "compression": true,
    "delta_only": true
  },
  "network": {
    "max_latency_ms": 200
  }
}
```

## Advanced: Custom Mod Extensions

Create custom synchronized behaviors:

```csharp
using BepInEx;
using DeepSynergy.Core;
using DeepSynergy.Sync;

[BepInPlugin("com.yourname.customextension", "Custom Extension", "1.0.0")]
[BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
public class CustomExtension : BaseUnityPlugin
{
    void Awake()
    {
        // Hook into session lifecycle
        SessionManager.OnSessionStarted += OnSessionStart;
        
        // Register custom sync handler
        StateSynchronizer.RegisterCustomHandler<CustomData>(
            OnCustomDataReceived
        );
    }
    
    void OnSessionStart(SessionInfo info)
    {
        Logger.LogInfo($"Custom extension loaded for session {info.Name}");
    }
    
    void OnCustomDataReceived(CustomData data, PlayerInfo sender)
    {
        // Handle custom synchronized data
        Logger.LogInfo($"Received custom data from {sender.Name}");
    }
}
```

## Localization

Supported languages: English, Chinese, Japanese, German, French, Portuguese, Russian, Spanish, Korean

Override locale in config:
```json
{
  "locale": "ja_JP"  // Force Japanese
}
```

Access in code:
```csharp
using DeepSynergy.Localization;

string localizedText = LocalizationManager.Get("UI_SESSION_CREATED");
// Returns "セッションが作成されました" if locale is ja_JP
```
