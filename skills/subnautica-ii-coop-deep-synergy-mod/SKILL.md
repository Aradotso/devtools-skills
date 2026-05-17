---
name: subnautica-ii-coop-deep-synergy-mod
description: BepInEx multiplayer mod for Subnautica 2 enabling synchronized co-op survival with deterministic session architecture
triggers:
  - how do I set up Subnautica 2 multiplayer mod
  - configure Deep Synergy mod for co-op
  - install BepInEx mod for Subnautica 2
  - troubleshoot Subnautica multiplayer sync issues
  - create co-op session in Subnautica 2
  - customize Deep Synergy session parameters
  - integrate AI narration with Subnautica mod
  - fix session desync in Subnautica co-op
---

# Subnautica II Deep Synergy Co-op Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The **Deep Synergy Multiplayer Mod** transforms Subnautica 2 into a synchronized cooperative experience using BepInEx framework. It implements deterministic session synchronization (DSS) where game state, inventory, base-building, and creature AI are shared across clients without central servers.

**Key Architecture:**
- Peer-to-peer WebRTC connectivity with NAT punch-through
- Blockchain-inspired Merkle tree inventory verification
- Hot-reloadable BepInEx plugin modules
- Adaptive dynamic scaling based on player count
- Optional AI narrative integration (OpenAI/Claude APIs)

## Installation

### Prerequisites
1. **Subnautica 2** installed via Steam/GOG
2. **BepInEx 6.0.x** IL2CPP version for Unity games

### Setup Steps

```bash
# 1. Download BepInEx 6.0.x for IL2CPP from GitHub
# Extract to Subnautica 2 game directory (where Subnautica2.exe is located)

# 2. Run the game once to generate BepInEx folders
# Close the game after main menu loads

# 3. Download Deep Synergy mod from the project repository
# Extract contents to: Subnautica2/BepInEx/plugins/DeepSynergy/

# Verify directory structure:
Subnautica2/
├── BepInEx/
│   ├── config/
│   │   └── synergy_profile.json
│   ├── core/
│   └── plugins/
│       └── DeepSynergy/
│           ├── DeepSynergy.dll
│           ├── SessionManager.dll
│           └── StateSynchronizer.dll
└── Subnautica2.exe
```

### First Launch

```bash
# Launch game normally - BepInEx will load automatically
# Look for console window showing:
[Info   :   BepInEx] BepInEx 6.0.0-pre.1 - Subnautica2
[Info   : DeepSynergy] Deep Synergy Mod v1.0 loaded
[Info   : DeepSynergy] Multiplayer hooks initialized
```

## Configuration

### Session Profile (synergy_profile.json)

Create or edit `BepInEx/config/synergy_profile.json`:

```json
{
  "session_name": "DeepDiveSquad",
  "max_players": 4,
  "difficulty_scale": "adaptive",
  "resource_multiplier": 1.5,
  "oxygen_consumption": 0.85,
  "creature_spawn_divider": 1.5,
  "enable_pvp": false,
  "friendly_fire": false,
  "shared_blueprints": true,
  "ping_locations_shared": true,
  "time_of_day_sync": "host",
  "voice_chat_integration": "discord_rpc",
  "network": {
    "port_range": "7777-7780",
    "max_latency_ms": 250,
    "packet_loss_tolerance": 0.05
  },
  "api_integration": {
    "openai": {
      "enabled": false,
      "api_key_env": "OPENAI_API_KEY",
      "role": "narrator",
      "model": "gpt-4"
    },
    "claude": {
      "enabled": false,
      "api_key_env": "ANTHROPIC_API_KEY",
      "role": "lore_engine",
      "model": "claude-3-sonnet"
    }
  },
  "sync_settings": {
    "tick_rate": 20,
    "state_hash_interval_ms": 500,
    "inventory_verify_frequency": 5
  },
  "locale": "en-US"
}
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `resource_multiplier` | float | Scales harvestable resources (1.5 = 50% more) |
| `oxygen_consumption` | float | Oxygen drain modifier (0.85 = 15% slower) |
| `creature_spawn_divider` | float | Reduces creature counts (1.5 = 33% fewer) |
| `difficulty_scale` | string | `"fixed"`, `"adaptive"`, or `"manual"` |
| `tick_rate` | int | State sync updates per second |
| `state_hash_interval_ms` | int | Inventory verification interval |

## Console Commands

Access via BepInEx console (F12 by default):

### Session Management

```bash
# Start hosting a session
/start_server
# Returns: Server created: session code = 9B2A-4C7D-E8F1

# Join existing session
/join_session 9B2A-4C7D-E8F1

# Check connection status
/synergy_status
# Output:
# Connected peers: 3
# State sync: 100% complete
# Inventory hash: 0xFA342B1E
# Average latency: 42ms
# Packet loss: 0.02%

# Disconnect from session
/leave_session
```

### Session Scaling

```bash
# Temporarily adjust difficulty
/synergy_scale 1.8
# Scales creature/resource parameters by factor

# Override world seed for consistency
/seed_override 8251
# Forces same procedural generation across clients

# Reset to config defaults
/synergy_reset
```

### AI Integration

```bash
# Trigger narrative generation (requires API keys)
/api_narrate "discovering the precursor facility"
# OpenAI/Claude generates contextual journal entry

# Generate species description for scanned creature
/api_lore creature_reaper_001
# Returns procedural biology report
```

## Code Integration Patterns

### Plugin Development Hook

For extending the mod with custom BepInEx plugins:

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using DeepSynergy.Core;
using DeepSynergy.Sync;

namespace MyCustomExtension
{
    [BepInPlugin("com.myname.custommod", "Custom Synergy Extension", "1.0.0")]
    [BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomSynergyPlugin : BasePlugin
    {
        public override void Load()
        {
            // Hook into session events
            SessionManager.OnSessionStarted += OnSessionStart;
            StateSynchronizer.OnInventorySync += OnInventoryUpdated;
            
            Log.LogInfo("Custom extension loaded");
        }
        
        private void OnSessionStart(SessionContext context)
        {
            Log.LogInfo($"Session {context.SessionCode} started with {context.PlayerCount} players");
            
            // Apply custom scaling logic
            if (context.PlayerCount >= 3)
            {
                context.ApplyDifficultyModifier(1.2f);
            }
        }
        
        private void OnInventoryUpdated(InventoryState state)
        {
            // Verify custom item sync
            var merkleRoot = state.ComputeMerkleRoot();
            Log.LogInfo($"Inventory hash: {merkleRoot}");
        }
    }
}
```

### Custom Session Event Listener

```csharp
using DeepSynergy.Events;
using UnityEngine;

public class BaseConstructionSyncer : MonoBehaviour
{
    void Start()
    {
        // Subscribe to base building events
        ConstructionSync.OnBasePiecePlaced += HandlePiecePlaced;
        ConstructionSync.OnBasePieceRemoved += HandlePieceRemoved;
    }
    
    void HandlePiecePlaced(ConstructionPiece piece, ulong playerId)
    {
        // Broadcast to peers with timestamp
        var syncPacket = new SyncPacket
        {
            Type = SyncType.ConstructionAdd,
            Timestamp = NetworkTime.CurrentTimestamp(),
            PlayerId = playerId,
            Data = piece.Serialize()
        };
        
        SessionManager.Broadcast(syncPacket);
    }
    
    void HandlePieceRemoved(Vector3 position, ulong playerId)
    {
        var syncPacket = new SyncPacket
        {
            Type = SyncType.ConstructionRemove,
            Timestamp = NetworkTime.CurrentTimestamp(),
            PlayerId = playerId,
            Data = position.Serialize()
        };
        
        SessionManager.Broadcast(syncPacket);
    }
}
```

### AI Narration Integration

```csharp
using DeepSynergy.AI;
using System.Threading.Tasks;

public class NarrativeController
{
    private AIIntegrationService aiService;
    
    public async Task GenerateDiscoveryNarrative(string eventDescription)
    {
        if (!aiService.IsEnabled("openai")) return;
        
        var prompt = $@"
            Context: Multiplayer Subnautica 2 session
            Event: {eventDescription}
            Generate a 2-3 sentence journal entry from the player's perspective.
        ";
        
        var narrative = await aiService.CallOpenAI(
            model: "gpt-4",
            prompt: prompt,
            maxTokens: 150,
            temperature: 0.7f
        );
        
        // Display in-game PDA
        PDAManager.AddJournalEntry(narrative);
    }
    
    public async Task<string> GenerateSpeciesDescription(string creatureId)
    {
        if (!aiService.IsEnabled("claude")) return null;
        
        var prompt = $@"
            Generate a fictional marine biology report for creature ID: {creatureId}
            Include: Classification, behavior patterns, threat level, ecosystem role
            Format: Scientific database entry style
        ";
        
        return await aiService.CallClaude(
            model: "claude-3-sonnet-20240229",
            prompt: prompt,
            maxTokens: 400
        );
    }
}
```

## Common Usage Patterns

### Hosting a Session

```bash
# 1. Configure session in synergy_profile.json
# 2. Launch game
# 3. In BepInEx console:
/start_server

# 4. Share session code with friends (displayed in console)
# Session code: A7B3-9D2E-F1C4

# 5. Friends join via:
/join_session A7B3-9D2E-F1C4
```

### Debugging Sync Issues

```bash
# Check state synchronization
/synergy_status

# Force inventory re-sync
/synergy_resync inventory

# Dump current session state to log
/synergy_dump_state

# Check for Merkle tree mismatches
/synergy_verify_integrity
```

### Performance Tuning

```json
{
  "sync_settings": {
    "tick_rate": 15,              // Reduce for lower bandwidth
    "state_hash_interval_ms": 750, // Less frequent checks
    "inventory_verify_frequency": 10 // Verify every 10th sync
  },
  "network": {
    "max_latency_ms": 300,         // Allow higher latency
    "packet_loss_tolerance": 0.08   // Tolerate more loss
  }
}
```

## Troubleshooting

### "Session Code Invalid" Error

**Cause:** Mismatched mod versions or network timeout

```bash
# Verify mod version
/synergy_version
# Both host and client must match

# Check network connectivity
/synergy_ping <host_ip>

# Regenerate session with explicit port
/start_server --port 7777
```

### Inventory Desynchronization

**Symptoms:** Items appear duplicated or missing

```bash
# Force full state reconciliation
/synergy_reconcile_state

# Check Merkle tree integrity
/synergy_verify_integrity
# Look for "Hash mismatch detected" warnings

# If corrupted, reset to last checkpoint
/synergy_rollback_checkpoint
```

### Creature AI Conflicts

**Symptoms:** Creatures behave erratically or freeze

```json
// Increase AI sync frequency in config
{
  "sync_settings": {
    "ai_state_interval_ms": 200  // Default: 500
  }
}
```

```bash
# Debug creature state lattice
/synergy_debug_ai creature_stalker_032
```

### High Latency/Packet Loss

```bash
# Check network metrics
/synergy_netstat
# Output shows per-peer latency and packet loss

# Reduce tick rate for slower connections
/synergy_tickrate 12

# Enable aggressive compression
/synergy_compression lz4
```

### API Integration Failures

**OpenAI/Claude not responding:**

```bash
# Verify environment variables are set
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# Test API connectivity
/api_test openai
/api_test claude

# Check logs for error details
tail -f BepInEx/LogOutput.log | grep API
```

### Session Migration After Host Disconnect

**Automatic recovery:**

```bash
# Monitor migration status
/synergy_status
# Output shows "Migrating session to peer 2..."

# Manually force migration to specific peer
/synergy_migrate_host <peer_id>
```

## Advanced Configuration

### Custom Difficulty Profiles

```json
{
  "difficulty_profiles": {
    "casual": {
      "resource_multiplier": 2.0,
      "oxygen_consumption": 0.6,
      "creature_spawn_divider": 2.5,
      "damage_taken_multiplier": 0.7
    },
    "hardcore": {
      "resource_multiplier": 0.8,
      "oxygen_consumption": 1.3,
      "creature_spawn_divider": 0.7,
      "damage_taken_multiplier": 1.5,
      "permadeath": true
    }
  },
  "active_profile": "casual"
}
```

### Locale-Specific Settings

```json
{
  "locale": "ja-JP",
  "localization_override": {
    "console_commands": "ja-JP",
    "ui_elements": "ja-JP",
    "api_prompts": "en-US"  // Keep AI prompts in English
  }
}
```

### WebRTC Configuration

```json
{
  "network": {
    "webrtc": {
      "ice_servers": [
        "stun:stun.l.google.com:19302",
        "stun:stun1.l.google.com:19302"
      ],
      "enable_turn": false,
      "turn_server_env": "TURN_SERVER_URL",
      "turn_credentials_env": "TURN_CREDENTIALS"
    }
  }
}
```

## Best Practices

1. **Always backup saves** before using multiplayer sessions
2. **Match mod versions** across all players to prevent sync issues
3. **Use adaptive scaling** for dynamic player counts
4. **Set realistic latency tolerances** based on geographic distances
5. **Enable AI features sparingly** to avoid API rate limits
6. **Monitor console logs** during first session for configuration errors
7. **Use checkpoint systems** via `/synergy_checkpoint` every 15-20 minutes

## Environment Variables

```bash
# Optional API integrations
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Optional TURN server for restricted NAT
export TURN_SERVER_URL="turn:turnserver.example.com:3478"
export TURN_CREDENTIALS="username:password"
```
