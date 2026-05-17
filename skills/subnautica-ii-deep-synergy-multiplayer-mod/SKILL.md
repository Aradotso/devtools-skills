---
name: subnautica-ii-deep-synergy-multiplayer-mod
description: BepInEx multiplayer mod for Subnautica 2 enabling co-op gameplay with deterministic session synchronization
triggers:
  - "set up Subnautica 2 multiplayer mod"
  - "install Deep Synergy co-op mod for Subnautica"
  - "configure BepInEx Subnautica 2 multiplayer"
  - "create Subnautica 2 co-op session"
  - "sync Subnautica multiplayer state"
  - "troubleshoot Subnautica 2 multiplayer connection"
  - "customize Subnautica co-op settings"
  - "integrate AI narration in Subnautica mod"
---

# Subnautica II Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a cooperative multiplayer experience using BepInEx plugin architecture. Built on IL2CPP hooking, it provides deterministic session synchronization, adaptive difficulty scaling, and decentralized peer-to-peer networking without requiring game file modifications.

**Key capabilities:**
- Synchronized survival mechanics (oxygen, inventory, base building)
- WebRTC-based peer-to-peer multiplayer (2-4 players)
- Adaptive difficulty scaling based on player count
- Shared blueprints, resources, and construction states
- Optional AI narrative integration (OpenAI/Claude APIs)
- Cross-platform support (Windows, Linux, macOS)

## Installation

### Prerequisites

1. **BepInEx 6.0.x** must be installed for Subnautica 2
2. Download from the official repository
3. Game must be on Steam or GOG (latest version)

### Installation Steps

```bash
# 1. Navigate to Subnautica 2 game directory
cd "C:/Program Files (x86)/Steam/steamapps/common/Subnautica 2"

# 2. Verify BepInEx is installed
dir BepInEx

# 3. Extract Deep Synergy mod to plugins folder
# Place downloaded files in:
# BepInEx/plugins/DeepSynergy/

# 4. Create config directory if missing
mkdir BepInEx/config

# 5. Launch game once to generate default configs
# Steam: Run normally
# Manual: Subnautica2.exe
```

### Directory Structure

```
Subnautica 2/
├── BepInEx/
│   ├── plugins/
│   │   └── DeepSynergy/
│   │       ├── DeepSynergy.dll
│   │       ├── NetworkCore.dll
│   │       └── SyncEngine.dll
│   ├── config/
│   │   ├── synergy_profile.json
│   │   └── DeepSynergy.cfg
│   └── LogOutput.log
└── Subnautica2.exe
```

## Configuration

### Session Profile Configuration

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
  "voice_chat_integration": "none",
  "network": {
    "port": 7777,
    "use_upnp": true,
    "max_ping_ms": 200,
    "tick_rate": 30
  },
  "sync_settings": {
    "inventory_sync_interval_ms": 500,
    "creature_ai_sync_interval_ms": 1000,
    "base_construction_immediate": true
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

### BepInEx Configuration

Edit `BepInEx/config/DeepSynergy.cfg`:

```ini
[General]
EnableMod = true
DebugMode = false
LogLevel = Info

[Networking]
ConnectionTimeout = 30
MaxRetries = 5
UseNATPunchthrough = true

[Performance]
SyncBufferSize = 1024
MaxConcurrentSyncs = 10
EnableDeltaCompression = true

[UI]
ShowConnectionStatus = true
ShowPlayerList = true
NotificationDuration = 5
```

## Console Commands

Access the BepInEx console with `F1` (default) in-game:

### Session Management

```bash
# Start a new multiplayer host session
/start_server
# Output: Server created: session code = 9B2A-4C7D-E8F1

# Join an existing session
/join_session 9B2A-4C7D-E8F1

# Leave current session
/leave_session

# Show session status
/synergy_status
# Output:
# Connected peers: 2
# State sync: 100% complete
# Inventory hash: 0xFA342B1E
# Network latency: 45ms avg
```

### Gameplay Modifiers

```bash
# Adjust difficulty scaling (1.0 = normal, 2.0 = double difficulty)
/synergy_scale 1.5

# Override world seed (must be done before session start)
/seed_override 8251

# Toggle shared inventory mode
/shared_inventory toggle

# Sync resources manually (usually automatic)
/force_sync resources
```

### AI Integration Commands

```bash
# Generate narrative for current event (requires API configured)
/api_narrate "exploring the kelp forest"

# Request lore entry for discovered location
/api_lore "thermal vents"

# Generate base name suggestion
/api_name_base
```

### Debugging

```bash
# Show detailed network stats
/net_stats

# Export current session state
/export_state session_backup.json

# Import session state (recovery)
/import_state session_backup.json

# Clear sync cache (use if desynced)
/clear_sync_cache
```

## Common Usage Patterns

### Starting a Co-op Session

```csharp
// Programmatic session creation (for mod developers extending this)
using DeepSynergy.Core;
using DeepSynergy.Networking;

public class SessionManager : MonoBehaviour
{
    public void CreateMultiplayerSession()
    {
        var config = new SessionConfig
        {
            MaxPlayers = 4,
            DifficultyScale = DifficultyScaleMode.Adaptive,
            ResourceMultiplier = 1.2f,
            SharedBlueprints = true
        };

        var session = SynergyCore.Instance.CreateSession(config);
        
        // Session code generated automatically
        string sessionCode = session.SessionCode;
        Debug.Log($"Session created: {sessionCode}");
        
        // Share this code with friends
        GUIUtility.systemCopyBuffer = sessionCode;
    }
}
```

### Joining a Session

```csharp
using DeepSynergy.Networking;

public class JoinSessionController : MonoBehaviour
{
    public void JoinViaCode(string sessionCode)
    {
        var connector = new SessionConnector();
        
        connector.OnConnectionSuccess += (session) =>
        {
            Debug.Log($"Connected to {session.Name}");
            Debug.Log($"Players: {session.PlayerCount}/{session.MaxPlayers}");
        };
        
        connector.OnConnectionFailed += (error) =>
        {
            Debug.LogError($"Connection failed: {error.Message}");
        };
        
        connector.ConnectToSession(sessionCode);
    }
}
```

### Synchronizing Custom Data

```csharp
using DeepSynergy.Sync;

// Sync custom player data across clients
public class CustomPlayerData : SyncedObject
{
    [Synced]
    public string PlayerName { get; set; }
    
    [Synced]
    public int ExplorationScore { get; set; }
    
    [Synced(SyncMode.OnChange)]
    public Vector3 LastPingLocation { get; set; }
    
    protected override void OnSynced()
    {
        // Called when data synced from another client
        Debug.Log($"{PlayerName} pinged location: {LastPingLocation}");
    }
}

// Register for synchronization
public void RegisterCustomData()
{
    var playerData = new CustomPlayerData
    {
        PlayerName = "Explorer1",
        ExplorationScore = 0
    };
    
    SyncManager.Instance.RegisterSyncedObject(playerData);
}
```

### Handling Inventory Synchronization

```csharp
using DeepSynergy.Inventory;

public class SharedInventoryManager : MonoBehaviour
{
    public void OnItemPickup(TechType itemType, int count)
    {
        // Automatically synced to all clients
        var syncedItem = new SyncedInventoryItem
        {
            TechType = itemType,
            Count = count,
            PickupTime = Time.time,
            OwnerClientId = SynergyCore.Instance.LocalClientId
        };
        
        InventorySyncService.Instance.AddItem(syncedItem);
    }
    
    public void OnItemCraft(TechType craftedItem)
    {
        // Verify all clients have required materials
        if (InventorySyncService.Instance.CanCraft(craftedItem))
        {
            InventorySyncService.Instance.ConsumeRecipe(craftedItem);
            InventorySyncService.Instance.AddItem(new SyncedInventoryItem
            {
                TechType = craftedItem,
                Count = 1
            });
        }
    }
}
```

### AI Narrative Integration

```csharp
using DeepSynergy.AI;
using System.Threading.Tasks;

public class NarrativeEngine : MonoBehaviour
{
    private AIIntegrationService aiService;
    
    void Start()
    {
        aiService = new AIIntegrationService();
        
        // Configure from environment variables
        aiService.ConfigureOpenAI(Environment.GetEnvironmentVariable("OPENAI_API_KEY"));
        aiService.ConfigureClaude(Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY"));
    }
    
    public async Task GenerateDiscoveryNarrative(string biomeName)
    {
        var context = new NarrativeContext
        {
            Event = "biome_discovery",
            Location = biomeName,
            PlayerCount = SynergyCore.Instance.ConnectedPlayers.Count,
            GameTime = DayNightCycle.main.GetDayScalar()
        };
        
        // Use OpenAI for short narrative
        string narrative = await aiService.GenerateNarrative(context, AIProvider.OpenAI);
        
        // Display in-game
        ErrorMessage.AddMessage(narrative);
    }
    
    public async Task GenerateSpeciesLore(string creatureName)
    {
        var loreContext = new LoreContext
        {
            Subject = creatureName,
            DiscoveredBy = Player.main.GetName(),
            SessionData = SynergyCore.Instance.GetSessionHistory()
        };
        
        // Use Claude for detailed lore (long context)
        string lore = await aiService.GenerateLore(loreContext, AIProvider.Claude);
        
        // Add to PDA database
        PDAEncyclopedia.Add(creatureName, lore);
    }
}
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to host session

```bash
# Check network configuration
/net_stats

# Verify port forwarding (if not using UPnP)
# Ensure port 7777 (or custom port) is open

# Test NAT traversal
/test_nat

# Try direct IP connection instead of session code
/join_direct 192.168.1.100:7777
```

**Problem:** High latency or desyncs

```json
// Adjust sync intervals in synergy_profile.json
{
  "sync_settings": {
    "inventory_sync_interval_ms": 1000,  // Increase from 500
    "creature_ai_sync_interval_ms": 2000, // Increase from 1000
    "tick_rate": 20  // Decrease from 30 for lower bandwidth
  }
}
```

### Session State Corruption

**Problem:** Players see different base structures

```bash
# Export current state
/export_state backup.json

# Have all players leave session
/leave_session

# Host restarts with clean state
/clear_sync_cache
/import_state backup.json
/start_server
```

**Problem:** Inventory desynchronization

```csharp
// Force inventory reconciliation
public void ForceInventorySync()
{
    var localInventory = InventorySyncService.Instance.GetLocalInventory();
    var merkleHash = InventorySyncService.Instance.ComputeMerkleHash(localInventory);
    
    // Request hash verification from all clients
    SynergyCore.Instance.BroadcastHashCheck(merkleHash);
    
    // If mismatch detected, full sync is triggered automatically
}
```

### Performance Optimization

**Problem:** Frame drops with multiple players

```ini
# Edit BepInEx/config/DeepSynergy.cfg
[Performance]
EnableDeltaCompression = true
MaxConcurrentSyncs = 5  # Reduce from 10
SyncBufferSize = 512    # Reduce from 1024

[Sync]
DisableCreatureAISync = false  # Set true if severe issues
ReducedPhysicsSync = true      # Enable for better performance
```

**Problem:** Memory leaks during long sessions

```csharp
// Periodic cleanup (run every 30 minutes)
public class MemoryManager : MonoBehaviour
{
    void Start()
    {
        InvokeRepeating(nameof(CleanupSyncCache), 1800f, 1800f);
    }
    
    void CleanupSyncCache()
    {
        SyncManager.Instance.PurgeOldEntries(TimeSpan.FromMinutes(10));
        Resources.UnloadUnusedAssets();
        GC.Collect();
    }
}
```

### AI Integration Issues

**Problem:** AI narration not generating

```csharp
// Verify API configuration
public void TestAPIConnection()
{
    var openaiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
    var claudeKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY");
    
    if (string.IsNullOrEmpty(openaiKey))
    {
        Debug.LogError("OPENAI_API_KEY not set in environment");
    }
    
    if (string.IsNullOrEmpty(claudeKey))
    {
        Debug.LogError("ANTHROPIC_API_KEY not set in environment");
    }
    
    // Test connectivity
    AIIntegrationService.TestConnection(AIProvider.OpenAI);
    AIIntegrationService.TestConnection(AIProvider.Claude);
}
```

### BepInEx Console Not Appearing

```bash
# Check BepInEx/config/BepInEx.cfg
[Logging.Console]
Enabled = true

# Alternative: Use log file
# BepInEx/LogOutput.log contains all console output
```

## Advanced Configuration

### Custom Difficulty Scaling Algorithm

```csharp
using DeepSynergy.Scaling;

public class CustomScalingProfile : IScalingProfile
{
    public float CalculateResourceMultiplier(int playerCount)
    {
        // Linear scaling: +25% resources per additional player
        return 1.0f + ((playerCount - 1) * 0.25f);
    }
    
    public float CalculateCreatureSpawnRate(int playerCount)
    {
        // Logarithmic scaling to prevent overwhelming spawns
        return 1.0f + (Mathf.Log(playerCount) * 0.3f);
    }
    
    public float CalculateOxygenConsumption(int playerCount)
    {
        // Slight reduction for co-op breathing efficiency
        return 1.0f - (playerCount * 0.05f);
    }
}

// Register custom profile
ScalingManager.Instance.RegisterProfile("custom", new CustomScalingProfile());
```

### Multi-Language Support

```json
// Create BepInEx/config/localization/zh_CN.json
{
  "session_created": "会话已创建",
  "session_code": "会话代码",
  "player_joined": "{0} 已加入",
  "player_left": "{0} 已离开",
  "sync_complete": "同步完成"
}
```

```csharp
// Load localization
LocalizationManager.Instance.SetLanguage("zh_CN");
string message = LocalizationManager.Get("player_joined", playerName);
```

## Best Practices

1. **Always export session state** before major changes (`/export_state`)
2. **Use adaptive difficulty** unless testing specific scenarios
3. **Enable UPnP** for easier connectivity (disable for manual port control)
4. **Set `shared_blueprints: true`** to prevent blueprint desync
5. **Keep API keys in environment variables**, never in config files
6. **Monitor sync status** with `/synergy_status` if experiencing lag
7. **Use delta compression** for bandwidth-limited connections
8. **Clear sync cache** (`/clear_sync_cache`) if players see ghost objects

## Environment Variables

```bash
# Windows (PowerShell)
$env:OPENAI_API_KEY = "sk-..."
$env:ANTHROPIC_API_KEY = "sk-ant-..."

# Linux/macOS
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Persistent (add to .bashrc or .zshrc)
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.bashrc
```
