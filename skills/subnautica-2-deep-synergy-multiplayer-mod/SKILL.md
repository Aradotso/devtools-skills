---
name: subnautica-2-deep-synergy-multiplayer-mod
description: BepInEx-based cooperative multiplayer mod for Subnautica 2 with session synchronization, shared inventory, and dynamic difficulty scaling
triggers:
  - install subnautica 2 multiplayer mod
  - configure deep synergy coop session
  - setup bepinex subnautica mod
  - troubleshoot subnautica multiplayer sync
  - create subnautica coop server
  - adjust subnautica mod difficulty scaling
  - enable subnautica ai narrative integration
  - debug subnautica session desync
---

# Subnautica 2 Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using BepInEx framework. It implements deterministic session synchronization, shared inventory management via Merkle tree verification, adaptive difficulty scaling based on player count, and cross-platform peer-to-peer connectivity using WebRTC.

**Key Features:**
- Deterministic Session Synchronization (DSS) for conflict-free world state
- Adaptive Dynamic Scaling (ADS) that adjusts difficulty based on player count
- Shared inventory with decentralized verification
- NAT punch-through for cross-platform multiplayer
- Optional AI narrative integration (OpenAI/Claude)
- Hot-reloadable BepInEx plugin architecture

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam/GOG
2. **BepInEx 6.0.x** (IL2CPP version for Unity games)

### Steps

1. Install BepInEx for Subnautica 2:
```bash
# Download BepInEx_IL2CPP_x64_6.0.0-pre.1.zip from BepInEx releases
# Extract to Subnautica 2 game directory
cd "C:/Program Files (x86)/Steam/steamapps/common/Subnautica 2"
# Verify folder structure: BepInEx/core/, BepInEx/plugins/
```

2. Download and install Deep Synergy Mod:
```bash
# Extract mod files to BepInEx/plugins/
# Structure should be:
# BepInEx/plugins/DeepSynergy/
#   ├── DeepSynergyCore.dll
#   ├── SessionManager.dll
#   ├── StateSynchronizer.dll
#   └── ConflictResolver.dll
```

3. Launch Subnautica 2 once to generate configuration files:
```bash
# BepInEx will create:
# BepInEx/config/synergy_profile.json
# BepInEx/config/network_settings.cfg
```

## Configuration

### Session Profile (`BepInEx/config/synergy_profile.json`)

```json
{
  "session_name": "MyCoopSession",
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
    "max_latency_ms": 150,
    "sync_interval_ms": 50,
    "enable_compression": true,
    "webrtc_stun_servers": [
      "stun:stun.l.google.com:19302"
    ]
  }
}
```

### Network Settings (`BepInEx/config/network_settings.cfg`)

```ini
[Network]
## Port for local session hosting (UDP)
LocalPort = 25564

## Enable UPnP for automatic port forwarding
EnableUPnP = true

## WebRTC ICE servers (comma-separated)
STUNServers = stun:stun.l.google.com:19302

## Maximum concurrent connections
MaxPeers = 8

## Packet loss tolerance (0.0-1.0)
PacketLossTolerance = 0.05

[Synchronization]
## Merkle tree verification interval (ms)
InventoryHashInterval = 1000

## State sync buffer size (KB)
SyncBufferSize = 512

## Conflict resolution strategy (timestamp|vote|host)
ConflictResolution = timestamp
```

## Console Commands

Access via BepInEx console (`F5` by default in-game):

### Session Management

```bash
# Start a new host session
/start_server

# Join existing session by code
/join_session 9B2A-4C7D-E8F1

# Get current session status
/synergy_status

# Disconnect from session
/disconnect

# Kick player by ID
/kick_player 2
```

### Session Configuration

```bash
# Adjust difficulty scaling multiplier
/synergy_scale 1.5

# Change resource spawn rate
/resource_mult 1.2

# Override world seed
/seed_override 8251

# Toggle friendly fire
/friendly_fire true

# Sync time of day for all players
/sync_time
```

### AI Integration

```bash
# Generate narrative for current context
/api_narrate "exploring the underwater caves"

# Request lore entry for discovered creature
/api_lore_creature "Ghost Leviathan"

# Get AI-suggested base name
/api_name_base
```

### Debugging

```bash
# Show inventory Merkle tree hash
/debug_inventory_hash

# Display peer connection stats
/debug_peers

# Force state resynchronization
/force_resync

# Export session log
/export_log session_2026-05-16.log
```

## Code Examples

### Custom Plugin Hook (C#)

```csharp
using BepInEx;
using DeepSynergy.Core;
using DeepSynergy.SessionManager;

namespace MyCustomMod
{
    [BepInPlugin("com.example.customsync", "Custom Sync Plugin", "1.0.0")]
    [BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomSyncPlugin : BasePlugin
    {
        private ISessionManager sessionManager;
        
        public override void Load()
        {
            // Get Deep Synergy session manager
            sessionManager = DeepSynergyAPI.GetSessionManager();
            
            // Subscribe to session events
            sessionManager.OnPlayerJoined += OnPlayerJoined;
            sessionManager.OnPlayerLeft += OnPlayerLeft;
            sessionManager.OnStateSync += OnStateSync;
            
            Log.LogInfo("Custom Sync Plugin loaded");
        }
        
        private void OnPlayerJoined(PlayerInfo player)
        {
            Log.LogInfo($"Player joined: {player.Name} (ID: {player.Id})");
            
            // Broadcast custom welcome message
            sessionManager.BroadcastCustomData("welcome", new {
                message = $"Welcome {player.Name}!",
                timestamp = DateTime.UtcNow
            });
        }
        
        private void OnPlayerLeft(PlayerInfo player)
        {
            Log.LogInfo($"Player left: {player.Name}");
        }
        
        private void OnStateSync(SyncData data)
        {
            // Handle state synchronization
            if (data.Type == SyncType.Inventory)
            {
                // Verify inventory hash
                var expectedHash = CalculateInventoryHash(data.Items);
                if (data.MerkleRoot != expectedHash)
                {
                    Log.LogWarning("Inventory hash mismatch, requesting resync");
                    sessionManager.RequestResync(SyncType.Inventory);
                }
            }
        }
    }
}
```

### Shared Inventory API

```csharp
using DeepSynergy.Inventory;

// Access shared inventory system
var sharedInventory = DeepSynergyAPI.GetSharedInventory();

// Add item to shared pool
sharedInventory.AddItem(new ItemData {
    Id = "titanium_ingot",
    Quantity = 10,
    OwnerId = localPlayer.Id,
    Timestamp = DateTime.UtcNow.Ticks
});

// Request item from shared pool
var requestedItem = await sharedInventory.RequestItemAsync(
    "titanium_ingot", 
    quantity: 5,
    timeoutMs: 3000
);

if (requestedItem != null)
{
    // Item successfully retrieved
    Log.LogInfo($"Received {requestedItem.Quantity}x {requestedItem.Id}");
}

// Listen for inventory changes
sharedInventory.OnInventoryChanged += (sender, args) => {
    Log.LogInfo($"Inventory updated: {args.ChangeType} - {args.ItemId}");
};
```

### AI Narrative Integration

```csharp
using DeepSynergy.AI;

// Configure AI narrator (requires API key in environment)
var narratorConfig = new AIConfig
{
    Provider = AIProvider.OpenAI,
    ApiKeyEnvVar = "OPENAI_API_KEY",
    Model = "gpt-4",
    MaxTokens = 150
};

var narrator = new AIComponents.Narrator(narratorConfig);

// Generate contextual narration
var context = new NarrativeContext
{
    Location = "Kelp Forest",
    RecentEvents = new[] { "discovered_wreckage", "scanned_peeper" },
    PlayerCount = 2,
    TimeOfDay = "dusk"
};

var narration = await narrator.GenerateNarrationAsync(context);
// Example output: "As dusk settles over the kelp forest, the team discovers 
// ancient wreckage. The peeper scan reveals unusual bio-signatures nearby."

// Display in-game
UIManager.ShowNotification(narration, duration: 5.0f);
```

### Session Migration (Host Dropout Recovery)

```csharp
using DeepSynergy.SessionManager;

var sessionManager = DeepSynergyAPI.GetSessionManager();

// Configure migration settings
sessionManager.MigrationConfig = new MigrationConfig
{
    Enabled = true,
    VotingTimeout = 5000, // ms
    PreferredHost = MigrationStrategy.LowestLatency
};

// Handle host migration event
sessionManager.OnHostMigration += (oldHost, newHost) => {
    Log.LogInfo($"Host migrated from {oldHost.Name} to {newHost.Name}");
    
    if (newHost.Id == localPlayer.Id)
    {
        // This client is now the host
        Log.LogInfo("You are now the session host");
        InitializeHostResponsibilities();
    }
};

// Manually trigger migration (admin command)
void ForceMigration(int targetPlayerId)
{
    if (sessionManager.IsHost)
    {
        sessionManager.MigrateHostTo(targetPlayerId);
    }
}
```

## Common Patterns

### Reliable Session Setup

```csharp
// Recommended session initialization flow
public async Task<bool> SetupCoopSession(SessionConfig config)
{
    var sessionManager = DeepSynergyAPI.GetSessionManager();
    
    try
    {
        // 1. Validate configuration
        if (!config.Validate(out var errors))
        {
            Log.LogError($"Invalid config: {string.Join(", ", errors)}");
            return false;
        }
        
        // 2. Initialize session
        var session = await sessionManager.CreateSessionAsync(config);
        
        // 3. Wait for network readiness
        await session.WaitForReadyAsync(timeout: TimeSpan.FromSeconds(10));
        
        // 4. Verify initial state sync
        var syncStatus = await session.VerifyStateSyncAsync();
        if (!syncStatus.Success)
        {
            Log.LogError($"State sync failed: {syncStatus.Error}");
            return false;
        }
        
        // 5. Display session code to user
        UIManager.ShowSessionCode(session.Code);
        
        Log.LogInfo($"Session ready: {session.Code}");
        return true;
    }
    catch (Exception ex)
    {
        Log.LogError($"Session setup failed: {ex.Message}");
        return false;
    }
}
```

### Conflict-Free State Updates

```csharp
// Use timestamp-based conflict resolution for shared state
public class BaseBuilder
{
    private IStateSynchronizer stateSynchronizer;
    
    public async Task PlaceStructureAsync(StructureType type, Vector3 position)
    {
        var placement = new StructurePlacement
        {
            Id = Guid.NewGuid().ToString(),
            Type = type,
            Position = position,
            PlacedBy = localPlayer.Id,
            Timestamp = DateTime.UtcNow.Ticks
        };
        
        // Submit to synchronizer with conflict resolution
        var result = await stateSynchronizer.SubmitStateChangeAsync(
            StateChangeType.StructurePlaced,
            placement,
            ConflictResolution.UseNewerTimestamp
        );
        
        if (result.Status == SyncStatus.Conflict)
        {
            Log.LogWarning($"Structure placement conflict at {position}");
            // Another player placed something at same location
            UIManager.ShowError("Location occupied by another structure");
            return;
        }
        
        if (result.Status == SyncStatus.Success)
        {
            // Render structure locally
            RenderStructure(placement);
        }
    }
}
```

## Troubleshooting

### Session Desynchronization

**Symptom:** Players see different base structures or inventory states.

```bash
# Check sync status
/synergy_status

# Look for hash mismatches in output
# If Merkle tree hashes differ:
/force_resync

# If issue persists, export logs
/export_log desync_debug.log
```

**Code fix:**
```csharp
// Increase sync interval if network is unstable
var config = DeepSynergyAPI.GetConfig();
config.Network.SyncIntervalMs = 100; // Default 50ms
config.Save();
```

### High Latency / Packet Loss

**Symptom:** Delayed actions, "rubber-banding" movement.

```ini
# Adjust network_settings.cfg
[Network]
PacketLossTolerance = 0.10  # Increase from 0.05
MaxPeers = 4  # Reduce if bandwidth limited

[Synchronization]
SyncBufferSize = 1024  # Increase from 512 KB
```

```bash
# Check peer connection quality
/debug_peers

# If latency > 150ms to host, consider host migration
/migrate_host <player_id_with_better_connection>
```

### BepInEx Plugin Not Loading

**Symptom:** Mod features unavailable in-game.

```bash
# Check BepInEx logs
cat "BepInEx/LogOutput.log" | grep "DeepSynergy"

# Common issues:
# 1. Wrong BepInEx version (need IL2CPP, not Mono)
# 2. Missing dependencies
# 3. Incorrect folder structure
```

Verify installation:
```
BepInEx/
├── core/
├── plugins/
│   └── DeepSynergy/  # Must be in subfolder
│       ├── DeepSynergyCore.dll
│       └── [other DLLs]
└── config/
    └── synergy_profile.json
```

### AI Integration Not Working

**Symptom:** Narrative features return errors.

```bash
# Verify environment variables are set
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# Test API connectivity
/api_narrate "test connection"
```

```json
// Ensure config is correct
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "api_key_env": "OPENAI_API_KEY",  // Must match env var name
      "role": "narrator"
    }
  }
}
```

### NAT Traversal Failures

**Symptom:** Cannot join sessions, "connection timeout" errors.

```ini
# Enable UPnP if router supports it
[Network]
EnableUPnP = true

# Try additional STUN servers
STUNServers = stun:stun.l.google.com:19302,stun:stun1.l.google.com:19302
```

Manual port forwarding (if UPnP fails):
- Forward UDP port 25564 to your local IP
- Ensure firewall allows BepInEx process

```bash
# Test connectivity
/start_server
# Share external IP:25564 with friend
# Friend runs: /join_session <your_external_ip>:25564
```

## Environment Variables

```bash
# AI Integration (optional)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Network (optional overrides)
export DEEPSYNERGY_PORT=25564
export DEEPSYNERGY_MAX_PEERS=8

# Logging
export DEEPSYNERGY_LOG_LEVEL=INFO  # DEBUG, INFO, WARN, ERROR
```

## Performance Optimization

```json
// For low-end systems or high player counts
{
  "network": {
    "sync_interval_ms": 100,  // Reduce sync frequency
    "enable_compression": true,  // Reduce bandwidth
    "max_latency_ms": 200  // Tolerate higher latency
  },
  "creature_spawn_divider": 2,  // Reduce creature count
  "resource_multiplier": 1.5  // Compensate with more resources
}
```

## Additional Resources

- BepInEx Documentation: https://docs.bepinex.dev/
- Session Codes: Format is `XXXX-XXXX-XXXX` (12 hex characters)
- Default keybinds: `F5` (console), `F6` (multiplayer menu)
- Mod configuration reloads on game restart (no hot-reload for JSON config)
