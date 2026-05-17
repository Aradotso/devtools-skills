---
name: subnautica-2-deep-synergy-multiplayer-mod
description: BepInEx-based cooperative multiplayer mod for Subnautica 2 with synchronized sessions, adaptive scaling, and shared base building
triggers:
  - install subnautica 2 multiplayer mod
  - configure deep synergy coop session
  - setup bepinex subnautica mod
  - fix subnautica multiplayer sync issues
  - create subnautica coop server
  - customize subnautica 2 multiplayer settings
  - troubleshoot subnautica mod connection
  - integrate ai narration in subnautica mod
---

# Subnautica 2: Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Deep Synergy is a BepInEx plugin that transforms Subnautica 2 into a synchronized cooperative multiplayer experience. It implements deterministic session synchronization, adaptive difficulty scaling, shared inventory management, and peer-to-peer networking without requiring central servers.

**Key Architecture:**
- **BepInEx 6.0.x** IL2CPP runtime hooks
- **WebRTC** peer-to-peer data channels with NAT punch-through
- **Merkle tree** inventory state verification
- Decentralized session migration on host disconnect
- Optional OpenAI/Claude API integration for dynamic narration

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam/GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### Steps

```bash
# 1. Install BepInEx (Windows example)
# Download BepInEx_x64_6.x.x.zip from https://github.com/BepInEx/BepInEx/releases
# Extract to Subnautica 2 game directory

# 2. Download Deep Synergy mod
# Extract to: <Game>/BepInEx/plugins/DeepSynergy/

# 3. Verify structure
<GameDirectory>/
├── BepInEx/
│   ├── config/
│   │   └── synergy_profile.json
│   └── plugins/
│       └── DeepSynergy/
│           ├── DeepSynergy.dll
│           └── localizations/
├── Subnautica2.exe
└── doorstop_config.ini
```

### Linux/Steam Deck

```bash
# Set launch options in Steam
WINEDLLOVERRIDES="winhttp=n,b" %command%

# Or use Proton with BepInEx autoloader
proton run Subnautica2.exe
```

## Configuration

### Session Profile (`BepInEx/config/synergy_profile.json`)

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
    "port_range_start": 7777,
    "port_range_end": 7787,
    "enable_upnp": true,
    "max_latency_ms": 200,
    "packet_retry_limit": 5
  },
  "session_persistence": {
    "auto_save_interval_seconds": 300,
    "backup_count": 3,
    "save_path": "BepInEx/saves/"
  }
}
```

**Key Fields:**
- `difficulty_scale`: `"static"` | `"adaptive"` (scales with player count)
- `resource_multiplier`: Float multiplier for resource nodes (1.5 = 50% more)
- `oxygen_consumption`: Fraction of normal drain (0.8 = 20% slower)
- `creature_spawn_divider`: Reduces spawns (2 = half creatures)
- `time_of_day_sync`: `"all"` (vote-based) | `"host"` | `"disabled"`

### Network Configuration

```json
{
  "network": {
    "transport": "webrtc",
    "stun_servers": [
      "stun:stun.l.google.com:19302",
      "stun:stun1.l.google.com:19302"
    ],
    "enable_relay": true,
    "bandwidth_limit_kbps": 512
  }
}
```

## In-Game Console Commands

Access via BepInEx console (F12 by default):

```bash
# Session Management
/start_server [session_name]       # Create new session
/join_session <code>                # Join with session code (format: XXXX-YYYY-ZZZZ)
/leave_session                      # Gracefully disconnect
/session_info                       # Display current session details

# Synchronization
/synergy_status                     # Show sync health, latency, peers
/force_sync                         # Trigger manual state reconciliation
/inventory_verify                   # Check Merkle tree integrity

# Scaling & Debug
/synergy_scale <multiplier>         # Adjust difficulty (1.0-3.0)
/toggle_debug_overlay               # Show network stats overlay
/seed_override <number>             # Force world seed for all clients

# AI Integration (if enabled)
/api_narrate "<event_description>"  # Trigger OpenAI/Claude narration
/api_status                         # Check API connection health
```

## Code Examples

### Plugin Hook (for mod developers extending functionality)

```csharp
using BepInEx;
using DeepSynergy.API;
using HarmonyLib;

namespace MySubnauticaAddon
{
    [BepInPlugin("com.example.addon", "My Synergy Addon", "1.0.0")]
    [BepInDependency("com.deepsynergy.multiplayer")]
    public class AddonPlugin : BaseUnityPlugin
    {
        private void Awake()
        {
            // Subscribe to session events
            SynergySessionManager.OnSessionCreated += OnSessionStart;
            SynergySessionManager.OnPlayerJoined += OnPeerConnect;
            
            // Hook into inventory sync
            SynergyInventory.RegisterCustomItemHandler(
                "CustomItem_ID",
                SerializeCustomItem,
                DeserializeCustomItem
            );
            
            Logger.LogInfo("Addon loaded successfully");
        }
        
        private void OnSessionStart(SessionContext ctx)
        {
            Logger.LogInfo($"Session {ctx.SessionCode} started with {ctx.MaxPlayers} slots");
        }
        
        private void OnPeerConnect(PeerInfo peer)
        {
            // Broadcast custom welcome message
            SynergyNetwork.SendToClient(peer.ClientId, new CustomWelcomePacket
            {
                Message = "Welcome to the enhanced session!"
            });
        }
        
        private byte[] SerializeCustomItem(object item)
        {
            // Custom serialization logic
            return System.Text.Encoding.UTF8.GetBytes(item.ToString());
        }
        
        private object DeserializeCustomItem(byte[] data)
        {
            return System.Text.Encoding.UTF8.GetString(data);
        }
    }
}
```

### Custom Packet Handler

```csharp
using DeepSynergy.Networking;

public class CustomDataPacket : INetworkPacket
{
    public string CustomData { get; set; }
    
    public void Serialize(BinaryWriter writer)
    {
        writer.Write(CustomData ?? "");
    }
    
    public void Deserialize(BinaryReader reader)
    {
        CustomData = reader.ReadString();
    }
}

// Register in Awake()
SynergyNetwork.RegisterPacketHandler<CustomDataPacket>(HandleCustomPacket);

private void HandleCustomPacket(CustomDataPacket packet, PeerInfo sender)
{
    Logger.LogInfo($"Received from {sender.Username}: {packet.CustomData}");
}

// Send packet
SynergyNetwork.BroadcastToAll(new CustomDataPacket 
{ 
    CustomData = "Hello from client!" 
});
```

### AI Integration Example

```csharp
using DeepSynergy.AI;
using System.Threading.Tasks;

public async Task GenerateNarrativeLog(string playerAction)
{
    if (!SynergyAI.IsEnabled(AIProvider.OpenAI))
        return;
    
    var prompt = $"Generate a 2-sentence survival log entry for: {playerAction}";
    
    var response = await SynergyAI.QueryOpenAI(new AIRequest
    {
        Prompt = prompt,
        MaxTokens = 100,
        Temperature = 0.7f,
        ApiKeyEnvVar = "OPENAI_API_KEY"
    });
    
    if (response.Success)
    {
        // Display in PDA journal
        SynergyUI.ShowNotification(response.Text, NotificationType.Journal);
    }
}

// Usage
await GenerateNarrativeLog("Built a multipurpose room at 200m depth");
```

### Session Persistence Hook

```csharp
using DeepSynergy.Persistence;

public void SaveCustomSessionData()
{
    var sessionData = new SessionSaveData
    {
        CustomFields = new Dictionary<string, object>
        {
            ["custom_marker_count"] = 42,
            ["team_base_name"] = "Arctic Outpost"
        }
    };
    
    SynergyPersistence.SaveSession(sessionData);
}

public void LoadCustomSessionData()
{
    var loaded = SynergyPersistence.LoadSession();
    
    if (loaded.CustomFields.TryGetValue("team_base_name", out var baseName))
    {
        Logger.LogInfo($"Restored base name: {baseName}");
    }
}
```

## Common Patterns

### Hosting a Session Programmatically

```csharp
var sessionConfig = new SessionConfig
{
    Name = "Research Team Alpha",
    MaxPlayers = 3,
    Password = "optional_password",
    IsPublic = false,
    DifficultyScale = DifficultyScaleMode.Adaptive
};

var sessionCode = SynergySessionManager.CreateSession(sessionConfig);
Logger.LogInfo($"Session code: {sessionCode}"); // Share with friends
```

### Joining via Code

```csharp
var joinResult = await SynergySessionManager.JoinSessionAsync("9B2A-4C7D-E8F1");

if (joinResult.Success)
{
    Logger.LogInfo($"Connected to {joinResult.SessionName}");
}
else
{
    Logger.LogError($"Join failed: {joinResult.ErrorMessage}");
}
```

### Inventory Sync Verification

```csharp
// Check if local inventory matches peer consensus
var syncStatus = SynergyInventory.VerifyIntegrity();

if (!syncStatus.IsValid)
{
    Logger.LogWarning($"Inventory desync detected: {syncStatus.ConflictCount} conflicts");
    
    // Trigger reconciliation
    SynergyInventory.ForceReconciliation();
}
```

### Adaptive Difficulty Adjustment

```csharp
// Automatically adjusts when players join/leave
SynergySessionManager.OnPlayerCountChanged += (count) =>
{
    float creatureMultiplier = Mathf.Lerp(1.0f, 1.8f, count / 4.0f);
    float resourceMultiplier = 1.0f + (count - 1) * 0.15f;
    
    SynergyScaling.SetCreatureSpawnRate(creatureMultiplier);
    SynergyScaling.SetResourceMultiplier(resourceMultiplier);
};
```

## Troubleshooting

### Session Code Not Generated

**Symptom:** `/start_server` returns "Failed to create session"

**Solutions:**
```bash
# Check port availability
netstat -an | grep 7777

# Enable UPnP in router settings or manually forward ports 7777-7787

# Try manual port specification
/start_server "MySession" --port 7780

# Check BepInEx log
tail -f BepInEx/LogOutput.log | grep "DeepSynergy"
```

### Connection Timeout on Join

**Symptom:** "Session not found" or timeout after 30s

**Solutions:**
```json
// Increase timeout in config
{
  "network": {
    "connection_timeout_seconds": 60,
    "enable_relay": true,
    "force_relay": false  // Try setting to true if direct connection fails
  }
}
```

**Check NAT type:**
```bash
# Use test utility
/network_diagnostics

# Expected output:
# NAT Type: Moderate (Full Cone)
# UPnP: Enabled
# Direct Connectivity: Yes
```

### Inventory Desync

**Symptom:** Items appear in one client but not others

**Solutions:**
```csharp
// Force full state resync
SynergyInventory.RequestFullSync();

// Enable verbose logging
SynergyDebug.SetLogLevel(LogLevel.Trace, "Inventory");

// Check Merkle tree hash
var localHash = SynergyInventory.GetStateHash();
var peerHashes = SynergyInventory.GetPeerHashes();

foreach (var peer in peerHashes)
{
    if (peer.Value != localHash)
    {
        Logger.LogWarning($"Hash mismatch with {peer.Key}");
    }
}
```

### Base Building Sync Issues

**Symptom:** Placed structures invisible to other players

**Solutions:**
```bash
# Enable construction event logging
/debug_construction true

# Verify timestamp authority
/synergy_status  # Check "Time Offset" column (should be <100ms)

# Manual structure sync
/sync_base_structures
```

### High Latency/Packet Loss

**Symptom:** Laggy movement, delayed actions

**Solutions:**
```json
// Reduce bandwidth usage
{
  "network": {
    "tick_rate": 15,  // Default: 30 (lower = less data)
    "compression": "lz4",  // Options: "none", "lz4", "gzip"
    "bandwidth_limit_kbps": 256
  }
}
```

```bash
# Monitor network stats
/network_stats

# Expected output:
# Latency: 45ms (good: <100ms, acceptable: <200ms)
# Packet Loss: 0.2% (good: <1%, max: 5%)
# Bandwidth: 180 KB/s
```

### AI Integration Not Responding

**Symptom:** `/api_narrate` returns no output

**Solutions:**
```bash
# Verify environment variables
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# Check API status
/api_status

# Test connection
/api_narrate "test connection"

# Enable debug logging
/debug_ai true
```

**Check logs for errors:**
```
[DeepSynergy.AI] Rate limit exceeded (429) - retry in 30s
[DeepSynergy.AI] Invalid API key (401)
[DeepSynergy.AI] Network timeout after 15s
```

### Session Migration Failure

**Symptom:** "Host disconnected - migration failed"

**Solutions:**
```json
// Enable redundant session state
{
  "session_persistence": {
    "enable_migration": true,
    "migration_timeout_seconds": 30,
    "require_consensus": true,  // All clients must agree on new host
    "backup_host_priority": ["peer_with_best_connection", "peer_with_lowest_latency"]
  }
}
```

```bash
# Force manual migration to specific peer
/migrate_host <peer_id>
```

## Performance Optimization

```json
{
  "performance": {
    "sync_interval_ms": 100,  // Faster = more responsive, higher bandwidth
    "state_compression": true,
    "lazy_sync_non_critical": true,  // Only sync nearby objects
    "cull_distance_multiplier": 1.5,  // Sync range relative to render distance
    "batch_updates": true
  }
}
```

## Advanced: Custom Mod Integration

### Creating Compatible Plugins

```csharp
[BepInDependency("com.deepsynergy.multiplayer", BepInDependency.DependencyFlags.HardDependency)]
public class CompatibleMod : BaseUnityPlugin
{
    void Start()
    {
        // Wait for Deep Synergy initialization
        SynergySessionManager.OnInitialized += () =>
        {
            Logger.LogInfo("Deep Synergy ready - registering hooks");
            RegisterCustomSyncHandlers();
        };
    }
    
    void RegisterCustomSyncHandlers()
    {
        // Sync custom mod data
        SynergyAPI.RegisterSyncCallback("MyMod_CustomData", SyncMyModData);
    }
    
    void SyncMyModData(NetworkMessage msg)
    {
        // Handle incoming sync data
    }
}
```

## Environment Variables

```bash
# Required for AI features
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Optional network configuration
export SYNERGY_FORCE_PORT=7780
export SYNERGY_DISABLE_UPNP=false
export SYNERGY_LOG_LEVEL=INFO  # DEBUG, INFO, WARN, ERROR
```

## Localization

```json
// BepInEx/config/synergy_profile.json
{
  "locale": "ja",  // "en", "zh", "ja", "de", "fr", "pt", "ru", "es", "ko"
  "custom_translations_path": "BepInEx/localizations/custom/"
}
```

Custom translation file (`custom/my_lang.json`):
```json
{
  "ui.session_created": "セッションが作成されました",
  "ui.player_joined": "{0}が参加しました",
  "error.connection_failed": "接続に失敗しました"
}
```
