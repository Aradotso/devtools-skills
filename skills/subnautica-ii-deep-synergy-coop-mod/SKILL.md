---
name: subnautica-ii-deep-synergy-coop-mod
description: BepInEx multiplayer mod for Subnautica 2 with synchronized co-op gameplay, adaptive scaling, and shared world state
triggers:
  - how do I set up the Subnautica 2 co-op mod
  - configure Deep Synergy multiplayer session
  - install BepInEx mod for Subnautica 2
  - troubleshoot Subnautica 2 multiplayer connection
  - create custom session profile for subnautica coop
  - sync inventory state in Deep Synergy mod
  - integrate OpenAI narrator with subnautica mod
  - fix session desync in subnautica multiplayer
---

# Subnautica II Deep Synergy Co-op Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using BepInEx. It implements deterministic session synchronization, adaptive difficulty scaling based on player count, and peer-to-peer networking via WebRTC. The mod provides shared inventory management, cross-platform multiplayer, and optional AI-powered narrative generation.

**Architecture:**
- BepInEx 6.0+ plugin architecture (IL2CPP hooks)
- Peer-to-peer WebRTC data channels (no central server)
- Merkle tree-based inventory verification
- NAT punch-through for cross-platform connectivity
- Optional OpenAI/Claude API integration for dynamic narration

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam or GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### Steps

```bash
# 1. Install BepInEx to Subnautica 2 directory
# Download from https://github.com/BepInEx/BepInEx/releases
# Extract to game root (where Subnautica2.exe is located)

# 2. Download Deep Synergy mod
# Extract to BepInEx/plugins/ directory
BepInEx/
  plugins/
    DeepSynergy/
      DeepSynergy.dll
      WebRTC.Native.dll
      SyncEngine.dll

# 3. Create configuration directory
mkdir -p BepInEx/config/DeepSynergy

# 4. Launch game once to generate default configs
# Configs will appear in BepInEx/config/DeepSynergy/
```

### Verify Installation

```bash
# Check BepInEx console output on game launch
# Should see:
# [Info   :   BepInEx] Loading [DeepSynergy 1.0.0]
# [Info   : DeepSynergy] Session Manager initialized
# [Info   : DeepSynergy] WebRTC channel ready
```

## Configuration

### Basic Session Profile

Create `BepInEx/config/DeepSynergy/synergy_profile.json`:

```json
{
  "session_name": "My Co-op Session",
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

### Network Configuration

Create `BepInEx/config/DeepSynergy/network.json`:

```json
{
  "nat_traversal": "auto",
  "stun_servers": [
    "stun:stun.l.google.com:19302",
    "stun:stun1.l.google.com:19302"
  ],
  "max_latency_ms": 150,
  "sync_rate_hz": 20,
  "compression": "zstd",
  "encryption": true
}
```

### AI Integration (Optional)

Create `.env` in game root directory:

```bash
OPENAI_API_KEY=your_openai_key_here
CLAUDE_API_KEY=your_claude_key_here
```

Enable in `synergy_profile.json`:

```json
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "role": "narrator",
      "model": "gpt-4",
      "max_tokens": 150,
      "temperature": 0.7
    },
    "claude": {
      "enabled": true,
      "role": "lore_engine",
      "model": "claude-3-opus-20240229",
      "max_tokens": 300
    }
  }
}
```

## Console Commands

Access BepInEx console with `F12` (default) in-game:

### Session Management

```bash
# Start hosting a session
/start_server

# Join existing session with code
/join_session 9B2A-4C7D-E8F1

# Leave current session
/leave_session

# Display session status
/synergy_status
```

### Difficulty Scaling

```bash
# Adjust difficulty multiplier (temporary)
/synergy_scale 1.5

# Reset to profile defaults
/synergy_scale reset

# Show current scaling values
/synergy_info
```

### Debugging

```bash
# Check synchronization state
/sync_check

# Force inventory resync
/force_sync inventory

# Show peer connection details
/peer_info

# Override world seed
/seed_override 12345
```

### AI Narration

```bash
# Trigger custom narration
/api_narrate "discovering ancient alien ruins"

# Generate creature lore
/api_lore_gen "ghost leviathan"

# Get contextual hint
/api_hint current_biome
```

## Programming Guide

### Creating Custom Session Profiles Programmatically

```csharp
using DeepSynergy.API;
using System.Collections.Generic;

public class CustomSessionManager
{
    public static SessionProfile CreateHardcoreProfile()
    {
        return new SessionProfile
        {
            SessionName = "Hardcore Duo",
            MaxPlayers = 2,
            DifficultyScale = ScaleMode.Static,
            ResourceMultiplier = 0.7f,
            OxygenConsumption = 1.3f,
            CreatureSpawnDivider = 0.8f,
            EnablePvP = false,
            FriendlyFire = true,
            SharedBlueprints = false,
            TimeOfDaySync = SyncMode.Host
        };
    }

    public static void ApplyProfile(SessionProfile profile)
    {
        var manager = SynergyCore.GetSessionManager();
        manager.LoadProfile(profile);
        manager.BroadcastConfiguration();
    }
}
```

### Hooking into Sync Events

```csharp
using DeepSynergy.Events;
using BepInEx;
using BepInEx.IL2CPP;

[BepInPlugin("com.mymod.synergyextension", "Synergy Extension", "1.0.0")]
public class SynergyExtension : BasePlugin
{
    public override void Load()
    {
        // Subscribe to inventory sync events
        SyncEvents.OnInventorySync += HandleInventorySync;
        
        // Subscribe to player join/leave
        SyncEvents.OnPlayerJoined += HandlePlayerJoined;
        SyncEvents.OnPlayerLeft += HandlePlayerLeft;
        
        // Subscribe to base building events
        SyncEvents.OnStructurePlaced += HandleStructurePlaced;
    }

    private void HandleInventorySync(InventorySyncData data)
    {
        Log.LogInfo($"Inventory synced: {data.ItemCount} items, hash {data.MerkleRoot}");
    }

    private void HandlePlayerJoined(PlayerJoinData data)
    {
        Log.LogInfo($"Player joined: {data.PlayerName} (ID: {data.PlayerId})");
    }

    private void HandlePlayerLeft(PlayerLeaveData data)
    {
        Log.LogInfo($"Player left: {data.PlayerId}");
    }

    private void HandleStructurePlaced(StructureData data)
    {
        Log.LogInfo($"Structure placed: {data.Type} at {data.Position}");
    }
}
```

### Custom Difficulty Scaler

```csharp
using DeepSynergy.Scaling;

public class CustomScaler : IAdaptiveScaler
{
    public float CalculateResourceMultiplier(int playerCount)
    {
        // Linear scaling: each player adds 25% resources
        return 1.0f + (playerCount - 1) * 0.25f;
    }

    public float CalculateCreatureSpawnRate(int playerCount)
    {
        // Logarithmic scaling to avoid overwhelming players
        return 1.0f + Mathf.Log10(playerCount);
    }

    public float CalculateOxygenConsumption(int playerCount)
    {
        // Reduce oxygen drain slightly in co-op
        return Mathf.Max(0.5f, 1.0f - (playerCount - 1) * 0.1f);
    }
}

// Register custom scaler
SynergyCore.GetSessionManager().RegisterScaler(new CustomScaler());
```

### Accessing Synchronized State

```csharp
using DeepSynergy.State;

public class BaseMonitor
{
    public void CheckSharedBaseStatus()
    {
        var stateManager = SynergyCore.GetStateManager();
        
        // Get all synchronized base structures
        var bases = stateManager.GetSyncedStructures();
        
        foreach (var structure in bases)
        {
            Debug.Log($"Structure: {structure.Id}");
            Debug.Log($"  Type: {structure.Type}");
            Debug.Log($"  Owner: {structure.PlacedByPlayerId}");
            Debug.Log($"  Position: {structure.Position}");
            Debug.Log($"  Integrity: {structure.Integrity}%");
        }
        
        // Get shared inventory state
        var inventory = stateManager.GetSharedInventory();
        Debug.Log($"Total items in network: {inventory.TotalItemCount}");
        Debug.Log($"Merkle root: {inventory.MerkleRoot}");
    }
}
```

### WebRTC Connection Management

```csharp
using DeepSynergy.Networking;

public class ConnectionManager
{
    public async Task<string> CreateSession()
    {
        var network = SynergyCore.GetNetworkManager();
        
        // Initialize WebRTC host
        var sessionCode = await network.CreateHostSession();
        Debug.Log($"Session created: {sessionCode}");
        
        // Set up connection callbacks
        network.OnPeerConnected += (peerId) => {
            Debug.Log($"Peer connected: {peerId}");
        };
        
        network.OnPeerDisconnected += (peerId) => {
            Debug.Log($"Peer disconnected: {peerId}");
        };
        
        return sessionCode;
    }
    
    public async Task<bool> JoinSession(string sessionCode)
    {
        var network = SynergyCore.GetNetworkManager();
        
        try
        {
            await network.JoinSession(sessionCode);
            Debug.Log("Successfully joined session");
            return true;
        }
        catch (NetworkException ex)
        {
            Debug.LogError($"Failed to join: {ex.Message}");
            return false;
        }
    }
}
```

## Common Patterns

### Graceful Session Migration

When the host disconnects, the mod automatically migrates to another peer:

```csharp
using DeepSynergy.Migration;

SyncEvents.OnHostMigration += (newHostId) => {
    if (newHostId == SynergyCore.GetLocalPlayerId())
    {
        // This client is now the host
        Debug.Log("Became session host");
        
        // Broadcast new session state
        var state = SynergyCore.GetStateManager();
        state.BroadcastFullState();
    }
    else
    {
        Debug.Log($"New host: {newHostId}");
    }
};
```

### Conditional AI Narration

```csharp
using DeepSynergy.AI;

public class NarrativeController
{
    private AIIntegration aiIntegration;
    
    public void Initialize()
    {
        aiIntegration = SynergyCore.GetAIIntegration();
        
        // Only enable if configured
        if (!aiIntegration.IsEnabled())
        {
            Debug.Log("AI integration disabled in profile");
            return;
        }
    }
    
    public async Task NarrateDiscovery(string biome, string discovery)
    {
        if (!aiIntegration.IsEnabled()) return;
        
        var prompt = $"The team discovered {discovery} in the {biome}. " +
                     $"Write a brief journal entry (50 words max).";
        
        var narration = await aiIntegration.GenerateNarration(prompt);
        
        // Broadcast to all players
        SynergyCore.GetChatManager().SendSystemMessage(narration);
    }
}
```

### Profile Validation

```csharp
using DeepSynergy.Validation;

public static class ProfileValidator
{
    public static bool ValidateProfile(SessionProfile profile, out string error)
    {
        error = null;
        
        if (profile.MaxPlayers < 2 || profile.MaxPlayers > 8)
        {
            error = "MaxPlayers must be between 2 and 8";
            return false;
        }
        
        if (profile.ResourceMultiplier < 0.1f || profile.ResourceMultiplier > 5.0f)
        {
            error = "ResourceMultiplier must be between 0.1 and 5.0";
            return false;
        }
        
        if (profile.OxygenConsumption < 0.1f)
        {
            error = "OxygenConsumption cannot be less than 0.1";
            return false;
        }
        
        return true;
    }
}
```

## Troubleshooting

### Session Code Not Generating

**Problem:** `/start_server` returns empty session code

**Solutions:**

```bash
# Check network configuration
cat BepInEx/config/DeepSynergy/network.json

# Verify STUN servers are accessible
ping stun.l.google.com

# Check firewall settings (allow UDP ports 49152-65535)

# Enable verbose logging
/synergy_debug enable

# Check BepInEx console for errors
# Look for: [Error  : DeepSynergy] Failed to initialize WebRTC
```

### Inventory Desync

**Problem:** Players see different inventory states

**Solutions:**

```bash
# Force full resync
/force_sync all

# Check sync status
/sync_check

# Verify Merkle tree integrity
/debug merkle_verify
```

In code:

```csharp
// Manual resync trigger
var stateManager = SynergyCore.GetStateManager();
stateManager.RequestFullSync();

// Check for conflicts
var conflicts = stateManager.GetSyncConflicts();
foreach (var conflict in conflicts)
{
    Debug.LogWarning($"Sync conflict: {conflict.Type} at {conflict.Timestamp}");
}
```

### High Latency Between Peers

**Problem:** Laggy interactions, delayed inventory updates

**Solutions:**

```json
// Reduce sync rate in network.json
{
  "sync_rate_hz": 10,
  "max_latency_ms": 200,
  "compression": "lz4"
}
```

```bash
# Check peer latency
/peer_info

# Output example:
# Peer 1 (9B2A): 45ms, packet loss 0.2%
# Peer 2 (4C7D): 180ms, packet loss 2.1%
```

### AI Integration Not Responding

**Problem:** `/api_narrate` commands fail silently

**Solutions:**

```bash
# Verify environment variables
echo $OPENAI_API_KEY
echo $CLAUDE_API_KEY

# Check API status in logs
grep "AI Integration" BepInEx/LogOutput.log

# Test API connection directly
/api_test openai
/api_test claude
```

### Structure Placement Conflicts

**Problem:** Two players place structures at same location

**Solutions:**

The mod uses timestamp-based conflict resolution:

```csharp
// Conflict resolution is automatic, but you can listen for events
SyncEvents.OnStructureConflict += (conflict) => {
    Debug.Log($"Structure conflict resolved: {conflict.WinningPlayerId} " +
              $"placed {conflict.StructureType} at {conflict.Position}");
};
```

### Session Migration Failures

**Problem:** All players disconnect when host leaves

**Solutions:**

```json
// Enable migration in synergy_profile.json
{
  "enable_host_migration": true,
  "migration_timeout_seconds": 10
}
```

```csharp
// Monitor migration status
SyncEvents.OnMigrationStarted += () => {
    Debug.Log("Host migration in progress...");
};

SyncEvents.OnMigrationFailed += (reason) => {
    Debug.LogError($"Migration failed: {reason}");
    // Optionally save session state locally
    SynergyCore.GetStateManager().SaveLocalBackup();
};
```

### Blueprint Sync Issues

**Problem:** Unlocked blueprints not appearing for all players

**Solutions:**

```bash
# Verify shared_blueprints setting
/synergy_info | grep blueprints

# Force blueprint resync
/force_sync blueprints
```

```csharp
// Manual blueprint sync
var blueprintManager = SynergyCore.GetBlueprintManager();
blueprintManager.SyncAllBlueprints();

// Check blueprint state
var syncedBlueprints = blueprintManager.GetSyncedBlueprints();
Debug.Log($"Synced blueprints: {syncedBlueprints.Count}");
```

## Best Practices

1. **Always validate profiles before applying** to avoid runtime errors
2. **Use environment variables** for API keys, never hardcode
3. **Enable compression** for sessions with 3+ players
4. **Monitor sync conflicts** in production sessions
5. **Implement graceful degradation** when AI APIs are unavailable
6. **Test host migration** scenarios before public sessions
7. **Use adaptive scaling** for variable player counts

## Resources

- BepInEx documentation: https://docs.bepinex.dev/
- WebRTC specifications: https://webrtc.org/
- Subnautica modding wiki: Community-maintained documentation
