---
name: subnautica-ii-deep-synergy-multiplayer-mod
description: BepInEx-based co-op multiplayer mod for Subnautica 2 with session synchronization, dynamic scaling, and WebRTC peer-to-peer networking
triggers:
  - set up subnautica 2 multiplayer mod
  - configure deep synergy coop session
  - install bepinex subnautica mod
  - create multiplayer session subnautica
  - troubleshoot subnautica coop sync issues
  - configure synergy profile for subnautica
  - integrate openai with subnautica mod
  - debug subnautica multiplayer connection
---

# Subnautica II Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a synchronized cooperative experience using BepInEx plugin architecture. It implements deterministic session synchronization, adaptive difficulty scaling, and peer-to-peer WebRTC networking without requiring central servers. The mod hooks into Unity's IL2CPP runtime to enable seamless multiplayer without modifying game files.

## Installation

### Prerequisites

1. **Subnautica 2** (Steam/GOG version)
2. **BepInEx 6.0.x** for Unity IL2CPP games
3. Minimum 8GB RAM, 300MB storage for mod files
4. Network: 5 Mbps up/down for peer-to-peer connections

### Step-by-Step Installation

1. **Install BepInEx:**
   ```bash
   # Download BepInEx 6.0.x from https://github.com/BepInEx/BepInEx/releases
   # Extract to Subnautica 2 root directory
   cd "C:/Program Files (x86)/Steam/steamapps/common/Subnautica 2"
   # Verify BepInEx folder structure exists
   ls BepInEx/core BepInEx/plugins
   ```

2. **Install Deep Synergy Mod:**
   ```bash
   # Download mod from release page
   # Extract to BepInEx/plugins/
   unzip DeepSynergyMod.zip -d "BepInEx/plugins/"
   ```

3. **Verify Installation:**
   ```bash
   # Check plugin files exist
   ls BepInEx/plugins/DeepSynergy/
   # Expected files:
   # - DeepSynergy.dll
   # - SessionManager.dll
   # - SyncEngine.dll
   # - WebRTCBridge.dll
   ```

4. **Launch Game:**
   - First launch will generate default configuration files in `BepInEx/config/`
   - Look for console output confirming mod load

## Configuration

### Session Profile Configuration

Create or edit `BepInEx/config/synergy_profile.json`:

```json
{
  "session_name": "Deep Dive Squad",
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
  "voice_chat_integration": "disabled",
  "network": {
    "port": 7777,
    "use_upnp": true,
    "max_latency_ms": 150,
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
      "model": "claude-3-5-sonnet-20241022"
    }
  }
}
```

### Network Configuration

Edit `BepInEx/config/DeepSynergy.Network.cfg`:

```ini
[Network]
# Use WebRTC for peer-to-peer
EnableWebRTC = true

# NAT punch-through settings
UseSTUN = true
STUNServer = stun:stun.l.google.com:19302

# Session discovery
EnableLocalDiscovery = true
DiscoveryPort = 7776

# Connection timeout
ConnectionTimeoutSeconds = 30
HeartbeatIntervalMs = 1000

[Synchronization]
# Conflict resolution strategy
ConflictResolution = timestamp_authority

# State sync frequency (Hz)
SyncRate = 20

# Inventory verification
UseInventoryHashing = true
HashAlgorithm = SHA256
```

## Key Commands (In-Game Console)

Access the BepInEx console with `F1` (default) or configure in `BepInEx/config/BepInEx.cfg`.

### Session Management

```bash
# Start hosting a session
/synergy start_server

# Join existing session
/synergy join <session-code>
# Example: /synergy join 9B2A-4C7D-E8F1

# Leave current session
/synergy disconnect

# Show session status
/synergy status
```

### Configuration Runtime Commands

```bash
# Adjust difficulty scaling
/synergy scale <multiplier>
# Example: /synergy scale 1.5

# Force world seed sync
/synergy seed_override <seed>
# Example: /synergy seed_override 8251

# Toggle blueprint sharing
/synergy toggle shared_blueprints

# Show network diagnostics
/synergy netstat
```

### AI Integration Commands

```bash
# Trigger narrative generation (requires API key)
/synergy narrate "<context>"
# Example: /synergy narrate "exploring thermal vents"

# Generate creature lore
/synergy lore_generate <creature_id>

# Reset AI context memory
/synergy ai_reset
```

## Code Examples (Plugin Development)

### Hooking Into Session Events

```csharp
using BepInEx;
using DeepSynergy.Core;
using DeepSynergy.Events;

namespace MyCustomPlugin
{
    [BepInPlugin("com.example.subnautica.customplugin", "Custom Plugin", "1.0.0")]
    [BepInDependency("com.deepsynergy.core", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomPlugin : BaseUnityPlugin
    {
        private void Awake()
        {
            // Subscribe to session events
            SessionManager.OnSessionStarted += OnSessionStart;
            SessionManager.OnPlayerJoined += OnPlayerJoin;
            SessionManager.OnStateSync += OnStateSync;
            
            Logger.LogInfo("Custom plugin loaded!");
        }

        private void OnSessionStart(SessionInfo session)
        {
            Logger.LogInfo($"Session started: {session.Name} with {session.MaxPlayers} slots");
            
            // Modify difficulty based on player count
            if (session.PlayerCount > 2)
            {
                DifficultyScaler.SetMultiplier(1.5f);
            }
        }

        private void OnPlayerJoin(PlayerInfo player)
        {
            Logger.LogInfo($"Player joined: {player.DisplayName}");
            
            // Share custom data with new player
            SyncEngine.BroadcastCustomData("welcome_message", new {
                message = "Welcome to the deep!",
                timestamp = DateTime.UtcNow
            });
        }

        private void OnStateSync(StateSyncEvent syncEvent)
        {
            // Validate inventory hash
            if (syncEvent.Type == SyncType.Inventory)
            {
                var hash = InventoryHasher.ComputeHash(syncEvent.Data);
                if (hash != syncEvent.ExpectedHash)
                {
                    Logger.LogWarning("Inventory hash mismatch - resolving conflict");
                    ConflictResolver.ResolveInventoryConflict(syncEvent);
                }
            }
        }
    }
}
```

### Custom Synchronization Data

```csharp
using DeepSynergy.Sync;
using System;

namespace MyCustomPlugin
{
    public class CustomDataSync
    {
        public void SyncCustomStructure(Vector3 position, Quaternion rotation, string structureId)
        {
            var syncData = new SyncPacket
            {
                Type = PacketType.CustomStructure,
                Timestamp = DateTime.UtcNow.Ticks,
                SenderId = SessionManager.LocalPlayer.Id,
                Data = new StructureData
                {
                    Position = position,
                    Rotation = rotation,
                    StructureId = structureId
                }
            };

            // Broadcast to all peers
            SyncEngine.Broadcast(syncData);
        }

        public void RegisterCustomSyncHandler()
        {
            SyncEngine.RegisterHandler(PacketType.CustomStructure, OnCustomStructureReceived);
        }

        private void OnCustomStructureReceived(SyncPacket packet)
        {
            var data = packet.Data as StructureData;
            
            // Conflict resolution using timestamp authority
            if (ConflictResolver.IsAuthoritative(packet.Timestamp, packet.SenderId))
            {
                // Apply structure placement
                BuildStructure(data.Position, data.Rotation, data.StructureId);
            }
        }

        private void BuildStructure(Vector3 pos, Quaternion rot, string id)
        {
            // Integration with Subnautica 2 building system
            // Implementation depends on game's internal API
        }
    }

    public class StructureData
    {
        public Vector3 Position { get; set; }
        public Quaternion Rotation { get; set; }
        public string StructureId { get; set; }
    }
}
```

### WebRTC Connection Manager

```csharp
using DeepSynergy.Network;
using System.Threading.Tasks;

namespace MyCustomPlugin
{
    public class NetworkManager
    {
        private WebRTCConnection connection;

        public async Task<string> CreateSession()
        {
            connection = new WebRTCConnection();
            
            // Configure STUN/TURN servers
            connection.AddIceServer("stun:stun.l.google.com:19302");
            
            // Create offer and generate session code
            var offer = await connection.CreateOffer();
            var sessionCode = SessionCodec.Encode(offer);
            
            return sessionCode;
        }

        public async Task JoinSession(string sessionCode)
        {
            connection = new WebRTCConnection();
            
            // Decode session and create answer
            var offer = SessionCodec.Decode(sessionCode);
            await connection.SetRemoteDescription(offer);
            
            var answer = await connection.CreateAnswer();
            await connection.SetLocalDescription(answer);
            
            // Wait for connection
            await connection.WaitForConnection(timeoutSeconds: 30);
        }

        public void SendGameState(byte[] stateData)
        {
            if (connection?.IsConnected == true)
            {
                connection.SendDataChannel("game_state", stateData);
            }
        }
    }
}
```

## Common Patterns

### Session Lifecycle

```csharp
// 1. Initialize session
var config = ConfigLoader.LoadProfile("BepInEx/config/synergy_profile.json");
var session = SessionManager.CreateSession(config);

// 2. Wait for players
await session.WaitForPlayers(minPlayers: 2, maxWaitSeconds: 120);

// 3. Sync initial world state
await SyncEngine.SynchronizeWorldState();

// 4. Start game loop with periodic sync
while (session.IsActive)
{
    await Task.Delay(1000 / config.SyncRate);
    SyncEngine.SyncFrame();
}

// 5. Handle disconnection gracefully
session.OnPlayerDisconnect += async (player) =>
{
    if (player.IsHost)
    {
        await session.MigrateHost();
    }
};
```

### Inventory Synchronization

```csharp
using DeepSynergy.Sync.Inventory;

public class InventorySyncManager
{
    public void SyncItemPickup(string itemId, int quantity)
    {
        // Create Merkle tree node for verification
        var inventoryNode = new InventoryNode
        {
            ItemId = itemId,
            Quantity = quantity,
            Timestamp = DateTime.UtcNow,
            PlayerId = SessionManager.LocalPlayer.Id
        };

        // Compute hash for integrity
        inventoryNode.Hash = MerkleTree.ComputeHash(inventoryNode);

        // Broadcast to peers
        SyncEngine.BroadcastInventoryUpdate(inventoryNode);
    }

    public void VerifyInventoryState()
    {
        var localHash = MerkleTree.ComputeRootHash(LocalInventory);
        var consensusHash = SyncEngine.GetConsensusInventoryHash();

        if (localHash != consensusHash)
        {
            // Request full inventory sync from authoritative peer
            SyncEngine.RequestInventoryReconciliation();
        }
    }
}
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to peer session

```bash
# Check network configuration
/synergy netstat

# Verify port forwarding (if not using UPnP)
netstat -an | grep 7777

# Test STUN connectivity
/synergy test_stun

# Enable verbose logging
BepInEx/config/BepInEx.cfg: LogLevel = All
```

**Solution:**
1. Ensure firewall allows UDP on port 7777
2. Enable UPnP in router settings
3. Use alternative STUN server in config
4. Check `BepInEx/LogOutput.log` for WebRTC errors

### Inventory Desync

**Problem:** Items appear in different states between players

```csharp
// Force inventory reconciliation
SyncEngine.RequestInventoryReconciliation();

// Reset Merkle tree cache
MerkleTree.ClearCache();

// Re-sync from host
if (!SessionManager.IsHost)
{
    var hostInventory = await SyncEngine.RequestHostInventory();
    LocalInventory.ReplaceWith(hostInventory);
}
```

### Performance Issues

**Problem:** High latency or packet loss

```json
// Adjust sync rate in synergy_profile.json
{
  "network": {
    "max_latency_ms": 200,
    "packet_loss_tolerance": 0.1
  }
}
```

```bash
# Reduce sync frequency
/synergy set_sync_rate 10

# Disable non-critical sync features
/synergy toggle ping_locations_shared
```

### AI Integration Errors

**Problem:** OpenAI/Claude API calls failing

```bash
# Verify environment variables are set
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# Test API connectivity outside game
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# Check mod logs for API errors
tail -f BepInEx/LogOutput.log | grep "API"
```

**Solution:**
1. Ensure API keys are set in environment (not config file)
2. Check API rate limits/quotas
3. Disable AI features if not needed: `"enabled": false` in config

### Mod Load Failures

**Problem:** Mod doesn't load on game start

```bash
# Verify BepInEx installation
ls -la BepInEx/core/BepInEx.Core.dll

# Check plugin dependencies
grep "BepInDependency" BepInEx/plugins/DeepSynergy/*.dll

# Enable BepInEx console for startup errors
# Edit BepInEx/config/BepInEx.cfg
[Logging.Console]
Enabled = true
```

## Advanced Configuration

### Locale Customization

```json
// BepInEx/config/synergy_profile.json
{
  "locale": "en-US",
  "localization_override": {
    "session_started": "Dive initiated!",
    "player_joined": "{0} has entered the abyss"
  }
}
```

### Custom Difficulty Scaling

```csharp
DifficultyScaler.RegisterCustomRule("resource_scarcity", (playerCount) =>
{
    // Exponential resource reduction
    return Math.Pow(0.8, playerCount - 1);
});

DifficultyScaler.RegisterCustomRule("creature_aggression", (playerCount) =>
{
    // Linear aggression increase
    return 1.0 + (playerCount * 0.15);
});
```

### Session Persistence

```csharp
// Save session state for resumption
SessionManager.OnSessionEnd += (session) =>
{
    var stateFile = $"BepInEx/cache/session_{session.Id}.dat";
    SessionSerializer.SaveState(session, stateFile);
};

// Resume previous session
var savedSession = SessionSerializer.LoadState("BepInEx/cache/session_1234.dat");
SessionManager.ResumeSession(savedSession);
```

## Environment Variables

```bash
# API keys (never commit these)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Network configuration
export SYNERGY_STUN_SERVER="stun:custom.stun.server:3478"
export SYNERGY_DEFAULT_PORT="7777"

# Debug mode
export SYNERGY_DEBUG="true"
export SYNERGY_LOG_LEVEL="DEBUG"
```
