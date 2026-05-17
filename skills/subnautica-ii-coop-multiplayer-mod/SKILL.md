---
name: subnautica-ii-coop-multiplayer-mod
description: Install and configure the Deep Synergy multiplayer mod for Subnautica 2 using BepInEx framework
triggers:
  - set up subnautica 2 multiplayer mod
  - install deep synergy coop mod
  - configure subnautica 2 bepinex multiplayer
  - create subnautica 2 co-op session
  - troubleshoot subnautica multiplayer sync
  - customize subnautica 2 coop settings
  - integrate openai with subnautica mod
  - fix subnautica multiplayer connection issues
---

# Subnautica II Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using the BepInEx modding framework. It implements deterministic session synchronization, adaptive difficulty scaling, and peer-to-peer connectivity without requiring central servers.

**Key Architecture Components:**
- BepInEx 6.0+ plugin system (IL2CPP hooks)
- WebRTC peer-to-peer data channels
- Merkle tree-based inventory synchronization
- Decentralized session migration with zero data loss
- Optional OpenAI/Claude API integration for narrative generation

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam/GOG
2. **BepInEx 6.0.x** (latest stable)

### Installation Steps

```bash
# 1. Navigate to Subnautica 2 installation directory
cd "C:/Program Files (x86)/Steam/steamapps/common/Subnautica 2"

# 2. Install BepInEx (if not already installed)
# Download from https://github.com/BepInEx/BepInEx/releases
# Extract to game root directory

# 3. Download Deep Synergy Mod files
# Extract to BepInEx/plugins/

# Directory structure should be:
# Subnautica 2/
# ├── BepInEx/
# │   ├── config/
# │   │   └── synergy_profile.json
# │   └── plugins/
# │       └── DeepSynergy/
# │           ├── DeepSynergy.dll
# │           └── localizations/

# 4. Verify installation
# Launch game once to generate config files
```

## Configuration

### Basic Session Configuration

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
  "api_integration": {
    "openai": {
      "enabled": false,
      "role": "narrator"
    },
    "claude": {
      "enabled": false,
      "role": "lore_engine"
    }
  }
}
```

### Configuration Fields Reference

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `session_name` | string | "New Session" | Display name for multiplayer lobby |
| `max_players` | int | 4 | Maximum concurrent players (2-8) |
| `difficulty_scale` | enum | "adaptive" | `"static"`, `"adaptive"`, `"challenging"` |
| `resource_multiplier` | float | 1.0 | Resource node yield multiplier (0.5-2.0) |
| `oxygen_consumption` | float | 1.0 | Oxygen drain rate (0.5 = slower, 2.0 = faster) |
| `creature_spawn_divider` | int | 1 | Reduces spawns (2 = half spawns, 3 = third) |
| `shared_blueprints` | bool | true | All players unlock blueprints simultaneously |
| `ping_locations_shared` | bool | true | Waypoint markers visible to all players |
| `time_of_day_sync` | enum | "all" | `"host"`, `"all"`, `"individual"` |

### Advanced Configuration: API Integration

```json
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "role": "narrator",
      "api_key_env": "OPENAI_API_KEY",
      "model": "gpt-4",
      "max_tokens": 150,
      "temperature": 0.7,
      "prompt_template": "Generate a journal entry for: {event}"
    },
    "claude": {
      "enabled": true,
      "role": "lore_engine",
      "api_key_env": "CLAUDE_API_KEY",
      "model": "claude-3-opus-20240229",
      "max_tokens": 500,
      "session_memory": true
    }
  }
}
```

**Environment Variables:**
```bash
# Set in system environment or .env file
export OPENAI_API_KEY="your_openai_key_here"
export CLAUDE_API_KEY="your_claude_key_here"
```

## Console Commands

Access via BepInEx console (F5 in-game by default):

### Session Management

```bash
# Start hosting a new session
/start_server

# Join existing session with code
/join_session 9B2A-4C7D-E8F1

# Leave current session
/disconnect

# Force session migration to new host
/migrate_host PlayerName
```

### Synchronization & Debugging

```bash
# Check connection status
/synergy_status

# Force inventory resync
/sync_inventory

# View current session seed
/show_seed

# Override world seed (must be done before session start)
/seed_override 12345

# Adjust difficulty scaling (temporary, current session only)
/synergy_scale 1.5
```

### AI Integration Commands

```bash
# Trigger OpenAI narrative generation
/api_narrate "exploring the thermal vents"

# Generate creature lore via Claude
/api_creature_info ReaperLeviathan

# View AI-generated session summary
/api_summary
```

## Common Usage Patterns

### Hosting a Co-op Session

```csharp
// Example BepInEx plugin extension for custom session logic
using BepInEx;
using BepInEx.Configuration;
using DeepSynergy.Core;

namespace MyCustomMod
{
    [BepInPlugin("com.author.subnautica.customsession", "Custom Session", "1.0.0")]
    [BepInDependency("com.deepsynergy.coop")]
    public class CustomSessionPlugin : BaseUnityPlugin
    {
        private ConfigEntry<int> maxPlayers;
        private ConfigEntry<float> resourceMultiplier;

        void Awake()
        {
            // Bind configuration
            maxPlayers = Config.Bind("Session", "MaxPlayers", 4, "Maximum players allowed");
            resourceMultiplier = Config.Bind("Session", "ResourceMultiplier", 1.2f, "Resource yield multiplier");

            // Hook into Deep Synergy session creation
            SessionManager.OnSessionCreate += OnSessionCreated;
        }

        void OnSessionCreated(Session session)
        {
            session.SetMaxPlayers(maxPlayers.Value);
            session.SetResourceMultiplier(resourceMultiplier.Value);
            
            Logger.LogInfo($"Custom session created with {maxPlayers.Value} max players");
        }

        void OnDestroy()
        {
            SessionManager.OnSessionCreate -= OnSessionCreated;
        }
    }
}
```

### Custom Inventory Sync Handler

```csharp
using DeepSynergy.Sync;
using UnityEngine;

public class CustomInventorySync : MonoBehaviour
{
    void Start()
    {
        // Subscribe to inventory change events
        InventorySyncManager.OnItemAdded += HandleItemAdded;
        InventorySyncManager.OnItemRemoved += HandleItemRemoved;
    }

    void HandleItemAdded(string playerId, string itemId, int quantity)
    {
        Debug.Log($"Player {playerId} added {quantity}x {itemId}");
        
        // Custom logic: notify other players
        if (SessionManager.IsHost)
        {
            SyncManager.BroadcastEvent("item_added", new {
                player = playerId,
                item = itemId,
                qty = quantity
            });
        }
    }

    void HandleItemRemoved(string playerId, string itemId, int quantity)
    {
        Debug.Log($"Player {playerId} removed {quantity}x {itemId}");
    }

    void OnDestroy()
    {
        InventorySyncManager.OnItemAdded -= HandleItemAdded;
        InventorySyncManager.OnItemRemoved -= HandleItemRemoved;
    }
}
```

### Adaptive Difficulty Scaling

```csharp
using DeepSynergy.Difficulty;
using System.Linq;

public class DifficultyController : MonoBehaviour
{
    void Update()
    {
        int playerCount = SessionManager.ActiveSession?.Players.Count ?? 1;
        
        // Scale creature spawns based on player count
        float spawnMultiplier = Mathf.Sqrt(playerCount);
        CreatureSpawnManager.SetSpawnMultiplier(spawnMultiplier);
        
        // Adjust resource respawn timers
        float resourceTimer = 300f / playerCount; // 5 minutes base / player count
        ResourceManager.SetRespawnTimer(resourceTimer);
        
        // Boost oxygen efficiency for larger groups
        float oxygenBonus = 1.0f + (playerCount - 1) * 0.1f;
        PlayerOxygenManager.SetConsumptionMultiplier(1.0f / oxygenBonus);
    }
}
```

### AI Narrative Integration

```csharp
using DeepSynergy.AI;
using System.Threading.Tasks;

public class NarrativeEngine : MonoBehaviour
{
    private OpenAIIntegration openai;
    private ClaudeIntegration claude;

    async void Start()
    {
        // Initialize AI integrations (requires API keys in env vars)
        openai = new OpenAIIntegration(System.Environment.GetEnvironmentVariable("OPENAI_API_KEY"));
        claude = new ClaudeIntegration(System.Environment.GetEnvironmentVariable("CLAUDE_API_KEY"));

        // Generate session opening narrative
        string intro = await openai.GenerateNarrative("session_start", new {
            biome = "safe_shallows",
            players = SessionManager.ActiveSession.Players.Select(p => p.Name).ToArray()
        });

        ChatManager.BroadcastMessage($"[Narrator] {intro}");
    }

    public async Task OnCreatureScanned(string creatureName)
    {
        // Generate lore via Claude
        string lore = await claude.GenerateLore("creature_biology", new {
            species = creatureName,
            context = "first_encounter"
        });

        PDAManager.AddDataEntry($"{creatureName}_lore", lore);
    }

    public async Task OnBaseBuilt(Vector3 position, string biome)
    {
        // Generate base name suggestion
        string suggestion = await claude.GenerateText($"Suggest a creative name for a base in {biome} biome");
        
        ChatManager.SendMessage($"[AI] Base naming suggestion: {suggestion}");
    }
}
```

## Troubleshooting

### Connection Issues

**Problem:** "Failed to establish WebRTC connection"

```bash
# Check firewall settings
# Allow UDP ports 50000-50100 (default range)

# Verify NAT type
/synergy_status
# Look for "NAT Type: Open" or "Moderate"

# Force relay mode (slower but more compatible)
/set_connection_mode relay
```

**Problem:** Session code not working

```json
// Check synergy_profile.json for typos
{
  "network": {
    "use_relay_fallback": true,
    "timeout_seconds": 30,
    "max_retries": 5
  }
}
```

### Synchronization Desync

**Problem:** Players see different base structures

```bash
# Force full world state resync
/sync_world_state

# If persists, check log file
# BepInEx/LogOutput.log

# Look for errors like:
# [DeepSynergy] Merkle tree hash mismatch: 0xABC123 vs 0xDEF456
```

**Solution:** Clear cache and rejoin

```bash
# Exit to main menu
/disconnect

# Clear local session cache
# Delete: BepInEx/cache/synergy_session_*.dat

# Rejoin session
/join_session <code>
```

### Performance Degradation

**Problem:** FPS drops with multiple players

```json
// Reduce sync frequency in synergy_profile.json
{
  "performance": {
    "sync_rate_hz": 10,        // Lower = less network traffic (default: 20)
    "cull_distance": 150,      // Only sync entities within 150m
    "compress_packets": true,  // Enable packet compression
    "batch_updates": true      // Batch small updates into single packets
  }
}
```

**Problem:** High memory usage

```bash
# Enable memory optimization mode
/set_optimization_mode high

# Monitor memory
/synergy_status
# Look for "Memory: XXX MB"

# Restart session if exceeds 2GB
```

### AI Integration Errors

**Problem:** API calls failing

```bash
# Verify environment variables
echo $OPENAI_API_KEY
echo $CLAUDE_API_KEY

# Test API connection
/api_test openai
/api_test claude

# Check rate limits in logs
# [DeepSynergy.AI] Rate limit exceeded, retrying in 60s
```

**Problem:** Irrelevant or broken narrative

```json
// Adjust AI parameters in synergy_profile.json
{
  "api_integration": {
    "openai": {
      "temperature": 0.5,  // Lower = more focused (was 0.7)
      "max_tokens": 100,   // Shorter responses
      "prompt_template": "In 2-3 sentences, describe: {event}"
    }
  }
}
```

## Advanced Customization

### Creating Custom Session Presets

Create `BepInEx/config/presets/hardcore_duo.json`:

```json
{
  "preset_name": "Hardcore Duo",
  "max_players": 2,
  "difficulty_scale": "challenging",
  "resource_multiplier": 0.7,
  "oxygen_consumption": 1.3,
  "creature_spawn_divider": 1,
  "creature_damage_multiplier": 1.5,
  "enable_permadeath": false,
  "shared_blueprints": true,
  "friendly_fire": true,
  "base_power_drain_multiplier": 1.2
}
```

Load preset via console:

```bash
/load_preset hardcore_duo
/start_server
```

### Hooking Custom Events

```csharp
using DeepSynergy.Events;

public class CustomEventHandler : MonoBehaviour
{
    void Start()
    {
        // Hook into core events
        GameEvents.OnPlayerJoined += OnPlayerJoin;
        GameEvents.OnBasePartBuilt += OnBaseBuild;
        GameEvents.OnCreatureKilled += OnCreatureDeath;
    }

    void OnPlayerJoin(Player player)
    {
        // Send welcome message
        ChatManager.SendMessageToPlayer(player.Id, "Welcome to the depths!");
        
        // Sync current session state
        SyncManager.SyncPlayerState(player.Id);
    }

    void OnBaseBuild(BuildEvent evt)
    {
        // Broadcast to all players
        ChatManager.BroadcastMessage($"{evt.PlayerName} built a {evt.PartType}");
        
        // Update shared base power grid
        PowerManager.RecalculateGrid(evt.BaseId);
    }

    void OnCreatureDeath(Creature creature, Player killer)
    {
        // Award shared progress
        SessionManager.ActiveSession.AddKillCount(creature.Species);
        
        // Notify team
        ChatManager.BroadcastMessage($"{killer.Name} defeated a {creature.Species}");
    }
}
```

## Best Practices

1. **Always set a world seed** for reproducible sessions:
   ```bash
   /seed_override 42069
   ```

2. **Use adaptive difficulty** for balanced co-op:
   ```json
   { "difficulty_scale": "adaptive" }
   ```

3. **Enable packet compression** for better performance:
   ```json
   { "performance": { "compress_packets": true } }
   ```

4. **Limit AI calls** to avoid rate limits:
   ```json
   { "openai": { "max_calls_per_hour": 30 } }
   ```

5. **Test configurations** in single-player first before hosting multiplayer sessions

6. **Back up save files** before major config changes:
   ```bash
   # Copy BepInEx/cache/ directory before modifications
   ```
