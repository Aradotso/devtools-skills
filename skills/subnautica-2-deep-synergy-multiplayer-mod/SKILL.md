---
name: subnautica-2-deep-synergy-multiplayer-mod
description: BepInEx multiplayer mod for Subnautica 2 enabling co-op gameplay with synchronized sessions, shared bases, and dynamic scaling
triggers:
  - install subnautica 2 multiplayer mod
  - setup deep synergy coop mod
  - configure bepinex subnautica multiplayer
  - create subnautica 2 coop session
  - troubleshoot subnautica multiplayer sync
  - customize subnautica mod settings
  - enable subnautica 2 shared gameplay
  - add ai narration to subnautica mod
---

# Subnautica 2 Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 from a solo survival experience into a synchronized cooperative game. Built on the BepInEx modding framework, it implements:

- **Deterministic Session Synchronization (DSS)**: Conflict-resolved world state sharing across clients
- **Adaptive Dynamic Scaling (ADS)**: Adjusts difficulty based on player count
- **Decentralized Architecture**: Peer-to-peer connectivity without central servers
- **Cross-Platform Support**: Windows, Linux (Steam Deck), macOS via WebRTC NAT punch-through
- **Shared Inventory & Base Building**: Merkle tree-based integrity verification
- **Optional AI Integration**: OpenAI/Claude API for dynamic narrative generation

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam or GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### BepInEx Setup

```bash
# Download BepInEx 6.0.x from GitHub releases
# Extract to Subnautica 2 game directory
cd "C:\Program Files (x86)\Steam\steamapps\common\Subnautica 2"
# or on Linux/macOS:
cd ~/.steam/steam/steamapps/common/Subnautica\ 2

# Verify structure:
# Subnautica2/
# ├── BepInEx/
# │   ├── core/
# │   ├── plugins/
# │   └── config/
# ├── Subnautica2_Data/
# └── Subnautica2.exe
```

### Mod Installation

```bash
# Extract Deep Synergy mod files to plugins directory
cd BepInEx/plugins/
# Place DeepSynergy.dll and dependencies here

# Verify installation
ls -la
# Should show:
# DeepSynergy.dll
# DeepSynergy.deps/
```

### First Launch

```bash
# Launch game to generate default config
# On Windows:
Subnautica2.exe

# On Linux (Steam Deck):
./Subnautica2.x86_64

# Check BepInEx/LogOutput.log for successful load:
# [Info   :   BepInEx] Loading [Deep Synergy Multiplayer v1.0.0]
```

## Configuration

### Profile Configuration

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
    "max_latency_ms": 150,
    "timeout_seconds": 30
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

### Key Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `max_players` | int | 2-8 players per session |
| `difficulty_scale` | string | `"adaptive"`, `"fixed"`, or `"manual"` |
| `resource_multiplier` | float | Scales harvestable resources (0.5 = half, 2.0 = double) |
| `oxygen_consumption` | float | Fraction of normal O2 drain (0.8 = 20% slower) |
| `creature_spawn_divider` | int | Divides creature spawn count (2 = half as many) |
| `shared_blueprints` | bool | Blueprint unlocks apply to all players |
| `time_of_day_sync` | string | `"host"`, `"vote"`, or `"independent"` |

## Console Commands

Access via BepInEx console (F1 by default):

### Session Management

```bash
# Start hosting a session
/start_server
# Output: Server created: session code = 9B2A-4C7D-E8F1

# Join existing session
/join_session 9B2A-4C7D-E8F1

# Leave current session
/leave_session

# Show session status
/synergy_status
# Output:
# Connected peers: 3/4
# State sync: 100%
# Inventory hash: 0xFA342B1E
# Average latency: 45ms
```

### Gameplay Adjustments

```bash
# Temporarily adjust difficulty scaling
/synergy_scale 1.5
# Increases creature count and resource scarcity by 50%

# Override world seed for deterministic generation
/seed_override 8251

# Force inventory resync (fixes desyncs)
/force_sync inventory

# Teleport to teammate
/tp_to PlayerName
```

### AI Narration (if enabled)

```bash
# Trigger narrative generation
/api_narrate "exploring the underwater caves"

# Generate creature log entry
/api_creature_log "ghostray"

# Request contextual hint
/api_hint "cyclops upgrade"
```

## Programming API (For Mod Developers)

### Creating Custom Session Hooks

```csharp
using BepInEx;
using DeepSynergy.Core;
using DeepSynergy.Network;

namespace MyCustomMod
{
    [BepInPlugin("com.example.customsession", "Custom Session Hook", "1.0.0")]
    [BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomSessionPlugin : BaseUnityPlugin
    {
        private void Awake()
        {
            // Subscribe to session events
            SessionManager.OnPlayerJoined += OnPlayerJoinedHandler;
            SessionManager.OnItemPickup += OnItemPickupHandler;
            SessionManager.OnBasePartPlaced += OnBasePartHandler;
        }

        private void OnPlayerJoinedHandler(PlayerSession player)
        {
            Logger.LogInfo($"Player {player.Name} joined from {player.IPAddress}");
            
            // Send welcome message
            NetworkMessenger.SendToPlayer(player.PeerId, new WelcomeMessage
            {
                Text = $"Welcome to the abyss, {player.Name}!",
                Color = UnityEngine.Color.cyan
            });
        }

        private void OnItemPickupHandler(ItemPickupEvent evt)
        {
            Logger.LogInfo($"{evt.PlayerName} picked up {evt.ItemType} at {evt.Position}");
            
            // Sync to all peers with conflict resolution
            StateSync.BroadcastItemState(evt.ItemGuid, evt.InventorySlot);
        }

        private void OnBasePartHandler(BasePartEvent evt)
        {
            // Validate placement before syncing
            if (BaseValidator.IsValidPlacement(evt.PartType, evt.Position, evt.Rotation))
            {
                StateSync.BroadcastBasePart(evt);
            }
            else
            {
                Logger.LogWarning($"Invalid base part placement by {evt.PlayerName}");
                NetworkMessenger.SendToPlayer(evt.PlayerId, new PlacementError
                {
                    Reason = "Overlapping collision detected"
                });
            }
        }
    }
}
```

### Custom Inventory Sync

```csharp
using DeepSynergy.Inventory;
using System.Collections.Generic;

public class CustomInventoryManager
{
    private InventoryMerkleTree _merkleTree;

    public void SyncCustomContainer(string containerId, List<Item> items)
    {
        // Create merkle proof for container
        var containerHash = _merkleTree.HashContainer(containerId, items);
        
        // Broadcast to peers
        var syncPacket = new ContainerSyncPacket
        {
            ContainerId = containerId,
            MerkleRoot = containerHash,
            Items = items.Select(i => new ItemState
            {
                Guid = i.Guid,
                TechType = i.TechType,
                StackSize = i.StackSize,
                Metadata = i.SerializeMetadata()
            }).ToList()
        };

        NetworkMessenger.Broadcast(syncPacket);
    }

    public bool VerifyContainerIntegrity(string containerId, byte[] receivedHash)
    {
        var localHash = _merkleTree.GetContainerHash(containerId);
        return localHash.SequenceEqual(receivedHash);
    }
}
```

### AI Integration Example

```csharp
using DeepSynergy.AI;
using System.Threading.Tasks;

public class NarrativeEngine
{
    private OpenAIClient _openai;
    private ClaudeClient _claude;

    public async Task<string> GenerateExplorationNarrative(ExplorationEvent evt)
    {
        if (!ConfigManager.GetBool("api_integration.openai.enabled"))
            return null;

        var prompt = $@"The player has discovered a new biome: {evt.BiomeName}
Depth: {evt.Depth}m
Notable features: {string.Join(", ", evt.Features)}
Previous context: {evt.RecentHistory}

Generate a 2-sentence journal entry in the style of a marine biologist.";

        var response = await _openai.GenerateCompletion(new OpenAIRequest
        {
            Model = "gpt-4",
            Prompt = prompt,
            MaxTokens = 100,
            Temperature = 0.7,
            ApiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY")
        });

        return response.Choices[0].Text.Trim();
    }

    public async Task<CreatureDataLog> GenerateCreatureLog(string creatureTechType)
    {
        if (!ConfigManager.GetBool("api_integration.claude.enabled"))
            return null;

        var prompt = $@"Generate a fictional marine biology log for this creature:
Species Code: {creatureTechType}
Format: JSON with fields taxonomy, behavior, habitat, threat_level";

        var response = await _claude.SendMessage(new ClaudeRequest
        {
            Model = "claude-3-opus-20240229",
            Messages = new[] { new Message { Role = "user", Content = prompt } },
            MaxTokens = 500,
            ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
        });

        return JsonConvert.DeserializeObject<CreatureDataLog>(response.Content[0].Text);
    }
}
```

## Common Patterns

### Hosting a Session

```bash
# 1. Configure profile
nano BepInEx/config/synergy_profile.json
# Set max_players, difficulty_scale, etc.

# 2. Launch game and open console (F1)
/start_server

# 3. Share session code with friends
# Session code appears in console: "9B2A-4C7D-E8F1"

# 4. Wait for players to join
/synergy_status
# Connected peers: 2/4

# 5. Start playing
# All inventory, base building, and progression syncs automatically
```

### Joining a Session

```bash
# 1. Launch game and open console
# 2. Join with code from host
/join_session 9B2A-4C7D-E8F1

# 3. Wait for sync to complete
# Progress shown in console:
# Syncing world state... 45%
# Syncing inventory... 78%
# Syncing base structures... 100%
# Ready!

# 4. Verify connection
/synergy_status
```

### Adjusting Difficulty Mid-Session

```json
// Edit synergy_profile.json while game is running
{
  "difficulty_scale": "adaptive",
  "resource_multiplier": 0.8,  // Reduce resources
  "creature_spawn_divider": 1   // More creatures
}
```

```bash
# Hot-reload configuration
/reload_config

# Or use console override (temporary)
/synergy_scale 1.3
```

### Enabling AI Narration

```bash
# Set environment variables
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Update config
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "role": "narrator"
    },
    "claude": {
      "enabled": true,
      "role": "lore_engine"
    }
  }
}

# In-game usage
/api_narrate "discovered precursor artifact"
# Output: "The artifact pulses with alien energy. Its surface inscriptions 
# suggest a warning—or perhaps an invitation to the depths below."
```

## Troubleshooting

### Session Connection Failed

**Symptom:** `/join_session` returns "Connection timeout"

**Solutions:**
```bash
# 1. Check firewall rules
sudo ufw allow 7777/tcp  # Linux
# Windows: Add inbound rule for port 7777

# 2. Verify NAT punch-through
/network_debug
# Shows STUN server status and NAT type

# 3. Use direct IP (bypasses NAT)
/join_direct 192.168.1.100:7777

# 4. Check port configuration
# In synergy_profile.json:
"network": {
  "port": 7777,  // Try different port if blocked
  "use_upnp": true  // Enable automatic port forwarding
}
```

### Inventory Desync

**Symptom:** Items appear in different slots for different players

**Solutions:**
```bash
# 1. Force inventory resync
/force_sync inventory

# 2. Check merkle tree integrity
/verify_merkle
# Output: "Hash mismatch detected in container 'PlayerInventory'"

# 3. Rebuild merkle tree
/rebuild_merkle

# 4. As last resort, disconnect and rejoin
/leave_session
/join_session <code>
```

### Base Parts Not Syncing

**Symptom:** Placed base parts invisible to other players

**Solutions:**
```bash
# 1. Check sync status
/synergy_status
# Look for "State sync: <100%"

# 2. Force base structure resync
/force_sync base

# 3. Check logs for conflicts
# BepInEx/LogOutput.log:
# [Error] BasePart placement conflict at (100, -50, 200)

# 4. Verify collision detection
/base_debug
# Shows placement validation errors

# 5. Increase sync timeout
# In synergy_profile.json:
"network": {
  "timeout_seconds": 60  // Increase for slow connections
}
```

### High Latency Issues

**Symptom:** Actions appear delayed for other players

**Solutions:**
```bash
# 1. Check current latency
/synergy_status
# Average latency: 250ms (high)

# 2. Reduce max players
# In synergy_profile.json:
"max_players": 2  // Less bandwidth per player

# 3. Disable AI features
"api_integration": {
  "openai": { "enabled": false },
  "claude": { "enabled": false }
}

# 4. Lower sync frequency
"network": {
  "sync_rate_hz": 10  // Default 20, lower = less bandwidth
}
```

### Mod Not Loading

**Symptom:** No multiplayer menu appears in-game

**Solutions:**
```bash
# 1. Verify BepInEx installation
# Check for BepInEx/LogOutput.log file

# 2. Check plugin loading
cat BepInEx/LogOutput.log | grep "Deep Synergy"
# Should show: [Info] Loading [Deep Synergy Multiplayer v1.0.0]

# 3. Verify dependencies
ls BepInEx/plugins/
# Should contain:
# DeepSynergy.dll
# DeepSynergy.deps/

# 4. Check compatibility
# Ensure Subnautica 2 version matches mod requirements
# See mod documentation for supported game versions

# 5. Clear cache and reload
rm -rf BepInEx/cache/
# Restart game
```

### API Integration Not Working

**Symptom:** `/api_narrate` returns "API disabled"

**Solutions:**
```bash
# 1. Verify environment variables
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# 2. Check config
cat BepInEx/config/synergy_profile.json
# Ensure "enabled": true for desired API

# 3. Test API connectivity
/api_test openai
# Output: "API test successful" or error message

# 4. Check rate limits
# If using free tier, may hit limits
# View logs: BepInEx/LogOutput.log
# [Warning] OpenAI rate limit exceeded

# 5. Verify API key format
# OpenAI: starts with "sk-"
# Anthropic: starts with "sk-ant-"
```

## Advanced Configuration

### Custom Network Settings

```json
{
  "network": {
    "port": 7777,
    "use_upnp": true,
    "stun_servers": [
      "stun.l.google.com:19302",
      "stun.stunprotocol.org:3478"
    ],
    "max_latency_ms": 150,
    "timeout_seconds": 30,
    "sync_rate_hz": 20,
    "compression": "lz4",
    "encryption": "aes256"
  }
}
```

### Difficulty Presets

```json
{
  "difficulty_presets": {
    "casual": {
      "resource_multiplier": 1.5,
      "oxygen_consumption": 0.7,
      "creature_spawn_divider": 2
    },
    "hardcore": {
      "resource_multiplier": 0.5,
      "oxygen_consumption": 1.3,
      "creature_spawn_divider": 1
    }
  },
  "active_preset": "casual"
}
```

### Localization

```json
{
  "locale": "en-US",
  "available_locales": [
    "en-US", "zh-CN", "ja-JP", "de-DE", 
    "fr-FR", "pt-BR", "ru-RU", "es-LA", "ko-KR"
  ]
}
```

## File Locations

- **Config**: `BepInEx/config/synergy_profile.json`
- **Logs**: `BepInEx/LogOutput.log`
- **Cache**: `BepInEx/cache/DeepSynergy/`
- **Session Data**: `BepInEx/cache/DeepSynergy/sessions/`
- **Localizations**: `BepInEx/plugins/DeepSynergy/localizations/`
