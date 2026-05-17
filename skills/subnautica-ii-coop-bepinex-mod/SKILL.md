---
name: subnautica-ii-coop-bepinex-mod
description: BepInEx multiplayer mod for Subnautica 2 with session sync, shared inventory, and API-driven narrative features
triggers:
  - how do I set up the Subnautica 2 co-op mod
  - configure Deep Synergy multiplayer session
  - install BepInEx mod for Subnautica 2
  - sync inventory in Subnautica multiplayer
  - integrate OpenAI narrator with Subnautica mod
  - troubleshoot Subnautica co-op connection issues
  - customize multiplayer scaling for Subnautica 2
  - create a shared session in Subnautica Deep Synergy
---

# Subnautica II Co-op BepInEx Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The **Deep Synergy Multiplayer Mod** transforms Subnautica 2 into a synchronized cooperative experience. Built on BepInEx, it implements deterministic session synchronization, adaptive difficulty scaling, shared inventory via Merkle tree verification, and optional AI-driven narrative features using OpenAI/Claude APIs.

**Core Architecture:**
- **BepInEx 6.0.x** plugin framework (IL2CPP hooks, no game file modification)
- **WebRTC P2P** for decentralized multiplayer (NAT punch-through)
- **Merkle tree inventory sync** for distributed state integrity
- **Adaptive scaling** based on player count
- **API integrations** for procedural narrative generation

## Installation

### Prerequisites
1. Subnautica 2 (Steam/GOG version)
2. BepInEx 6.0.x framework installed

### Setup Steps

```bash
# 1. Install BepInEx first (if not already installed)
# Download from https://github.com/BepInEx/BepInEx/releases
# Extract to Subnautica 2 game directory

# 2. Download Deep Synergy mod
# Extract to: <Subnautica2>/BepInEx/plugins/DeepSynergy/

# 3. Verify directory structure:
# BepInEx/
#   plugins/
#     DeepSynergy/
#       DeepSynergy.dll
#       Newtonsoft.Json.dll
#   config/
#     synergy_profile.json
```

### File Structure
```
<Subnautica2>/
├── BepInEx/
│   ├── core/
│   ├── plugins/
│   │   └── DeepSynergy/
│   │       ├── DeepSynergy.dll
│   │       └── libs/
│   └── config/
│       ├── synergy_profile.json
│       └── session_config.xml
└── Subnautica2.exe
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
  "network": {
    "port": 25000,
    "max_latency_ms": 200,
    "reconnect_timeout_s": 30,
    "stun_server": "stun:stun.l.google.com:19302"
  },
  "api_integration": {
    "openai": {
      "enabled": false,
      "api_key_env": "OPENAI_API_KEY",
      "model": "gpt-4",
      "role": "narrator"
    },
    "claude": {
      "enabled": false,
      "api_key_env": "ANTHROPIC_API_KEY",
      "model": "claude-3-sonnet-20240229",
      "role": "lore_engine"
    }
  },
  "locale": "en-US",
  "debug_mode": false
}
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `session_name` | string | Display name for session |
| `max_players` | int | 2-8 player limit |
| `difficulty_scale` | enum | `fixed`, `adaptive`, `manual` |
| `resource_multiplier` | float | Multiplies harvestable resources (0.5-2.0) |
| `oxygen_consumption` | float | Oxygen drain rate (0.5-1.5) |
| `creature_spawn_divider` | int | Reduces creature count (1=normal, 2=half) |
| `shared_blueprints` | bool | Blueprint unlocks sync across players |
| `time_of_day_sync` | enum | `host`, `all`, `none` |

### Network Configuration

```json
{
  "network": {
    "port": 25000,
    "max_latency_ms": 200,
    "reconnect_timeout_s": 30,
    "stun_server": "stun:stun.l.google.com:19302",
    "use_relay": false,
    "encryption": true
  }
}
```

## Console Commands

Access via BepInEx terminal (F1 key by default):

```bash
# Host a new session
/start_server

# Join existing session
/join_session <session-code>
# Example: /join_session 9B2A-4C7D-E8F1

# Check connection status
/synergy_status

# Adjust difficulty scaling mid-session
/synergy_scale <multiplier>
# Example: /synergy_scale 1.5

# Force world seed synchronization
/seed_override <seed>
# Example: /seed_override 8251

# Trigger AI narration (requires API config)
/api_narrate "<context>"
# Example: /api_narrate "exploring deep caves"

# Disconnect gracefully
/disconnect

# Force inventory resync
/sync_inventory

# List connected peers
/list_peers
```

## Code Examples

### Plugin Entry Point (C#)

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using HarmonyLib;
using UnityEngine;

namespace DeepSynergy
{
    [BepInPlugin(GUID, Name, Version)]
    public class DeepSynergyPlugin : BasePlugin
    {
        public const string GUID = "com.deepsynergy.subnautica2";
        public const string Name = "Deep Synergy Multiplayer";
        public const string Version = "1.0.0";

        private SessionManager _sessionManager;
        private StateSync _stateSync;

        public override void Load()
        {
            Log.LogInfo("Loading Deep Synergy...");
            
            // Load configuration
            var config = ConfigLoader.LoadProfile();
            
            // Initialize core systems
            _sessionManager = new SessionManager(config);
            _stateSync = new StateSync(config);
            
            // Apply Harmony patches
            var harmony = new Harmony(GUID);
            harmony.PatchAll();
            
            Log.LogInfo("Deep Synergy loaded successfully");
        }
    }
}
```

### Session Creation Pattern

```csharp
using DeepSynergy.Core;
using System;

namespace DeepSynergy.Session
{
    public class SessionManager
    {
        private SynergyConfig _config;
        private WebRTCChannel _channel;

        public SessionManager(SynergyConfig config)
        {
            _config = config;
        }

        public string CreateSession()
        {
            var sessionCode = GenerateSessionCode();
            _channel = new WebRTCChannel(_config.Network);
            
            _channel.OnPeerConnected += HandlePeerConnect;
            _channel.OnDataReceived += HandleStateUpdate;
            
            _channel.StartHost(_config.Network.Port);
            
            Console.WriteLine($"Session created: {sessionCode}");
            return sessionCode;
        }

        public bool JoinSession(string sessionCode)
        {
            _channel = new WebRTCChannel(_config.Network);
            
            _channel.OnConnected += () => {
                Console.WriteLine("Connected to session");
                SyncInitialState();
            };
            
            return _channel.Connect(sessionCode);
        }

        private void HandleStateUpdate(byte[] data)
        {
            var update = StateUpdate.Deserialize(data);
            _stateSync.ApplyUpdate(update);
        }

        private string GenerateSessionCode()
        {
            var guid = Guid.NewGuid();
            return $"{guid.ToString().Substring(0, 4).ToUpper()}-" +
                   $"{guid.ToString().Substring(4, 4).ToUpper()}-" +
                   $"{guid.ToString().Substring(8, 4).ToUpper()}";
        }
    }
}
```

### Inventory Synchronization

```csharp
using System.Security.Cryptography;
using System.Text;

namespace DeepSynergy.Sync
{
    public class InventorySync
    {
        private Dictionary<string, InventoryState> _playerInventories;
        
        public string ComputeMerkleRoot(InventoryState inventory)
        {
            var hashes = new List<string>();
            
            foreach (var item in inventory.Items.OrderBy(i => i.Id))
            {
                var itemData = $"{item.Id}:{item.Quantity}:{item.Metadata}";
                hashes.Add(ComputeHash(itemData));
            }
            
            return BuildMerkleTree(hashes);
        }

        public void SyncInventory(string playerId, InventoryState state)
        {
            var localHash = ComputeMerkleRoot(state);
            var remoteHash = RequestRemoteHash(playerId);
            
            if (localHash != remoteHash)
            {
                // Resolve conflict using timestamp authority
                var resolvedState = ConflictResolver.Resolve(
                    state, 
                    GetRemoteState(playerId)
                );
                
                ApplyInventoryState(playerId, resolvedState);
                BroadcastUpdate(playerId, resolvedState);
            }
        }

        private string ComputeHash(string data)
        {
            using (var sha256 = SHA256.Create())
            {
                var bytes = Encoding.UTF8.GetBytes(data);
                var hash = sha256.ComputeHash(bytes);
                return BitConverter.ToString(hash).Replace("-", "");
            }
        }

        private string BuildMerkleTree(List<string> hashes)
        {
            if (hashes.Count == 1) return hashes[0];
            
            var newLevel = new List<string>();
            for (int i = 0; i < hashes.Count; i += 2)
            {
                var combined = i + 1 < hashes.Count 
                    ? hashes[i] + hashes[i + 1] 
                    : hashes[i];
                newLevel.Add(ComputeHash(combined));
            }
            
            return BuildMerkleTree(newLevel);
        }
    }
}
```

### AI Narrative Integration

```csharp
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

namespace DeepSynergy.AI
{
    public class NarrativeEngine
    {
        private readonly HttpClient _client;
        private readonly string _apiKey;
        private readonly string _model;

        public NarrativeEngine(string apiKeyEnv, string model)
        {
            _apiKey = Environment.GetEnvironmentVariable(apiKeyEnv);
            _model = model;
            _client = new HttpClient();
        }

        public async Task<string> GenerateNarration(string context)
        {
            if (string.IsNullOrEmpty(_apiKey))
            {
                return "API key not configured";
            }

            var request = new
            {
                model = _model,
                messages = new[]
                {
                    new { role = "system", content = "You are a narrator for a co-op ocean survival game. Generate brief, atmospheric journal entries based on player actions." },
                    new { role = "user", content = $"Context: {context}" }
                },
                max_tokens = 150,
                temperature = 0.7
            };

            _client.DefaultRequestHeaders.Clear();
            _client.DefaultRequestHeaders.Add("Authorization", $"Bearer {_apiKey}");

            var response = await _client.PostAsJsonAsync(
                "https://api.openai.com/v1/chat/completions", 
                request
            );

            var result = await response.Content.ReadAsStringAsync();
            var json = JsonDocument.Parse(result);
            
            return json.RootElement
                .GetProperty("choices")[0]
                .GetProperty("message")
                .GetProperty("content")
                .GetString();
        }
    }
}
```

### Adaptive Scaling Implementation

```csharp
namespace DeepSynergy.Scaling
{
    public class DifficultyScaler
    {
        private SynergyConfig _config;
        private int _playerCount;

        public void UpdatePlayerCount(int count)
        {
            _playerCount = count;
            
            if (_config.DifficultyScale == "adaptive")
            {
                AdjustGameParameters();
            }
        }

        private void AdjustGameParameters()
        {
            // Scale creature spawns inversely with player count
            var creatureMultiplier = 1.0f / (_playerCount * _config.CreatureSpawnDivider);
            Game.CreatureSpawner.SetGlobalMultiplier(creatureMultiplier);
            
            // Scale resource availability
            var resourceMultiplier = _config.ResourceMultiplier * 
                Mathf.Lerp(1.0f, 1.5f, (_playerCount - 1) / 7f);
            Game.ResourceManager.SetMultiplier(resourceMultiplier);
            
            // Adjust oxygen consumption (more players = slightly more efficient)
            var oxygenEfficiency = _config.OxygenConsumption * 
                Mathf.Lerp(1.0f, 0.85f, (_playerCount - 1) / 7f);
            Game.Player.SetOxygenConsumption(oxygenEfficiency);
        }
    }
}
```

## Common Patterns

### Session Lifecycle

```csharp
// Host workflow
var session = new SessionManager(config);
var sessionCode = session.CreateSession();
Console.WriteLine($"Share code: {sessionCode}");

// Client workflow
var session = new SessionManager(config);
session.JoinSession("9B2A-4C7D-E8F1");

// Graceful disconnect
session.Disconnect();
```

### State Synchronization

```csharp
// Send state update
var update = new StateUpdate
{
    Timestamp = DateTime.UtcNow.Ticks,
    PlayerId = localPlayerId,
    Type = UpdateType.InventoryChange,
    Data = SerializeInventory()
};
_channel.Broadcast(update.Serialize());

// Receive and apply update
_channel.OnDataReceived += (data) => {
    var update = StateUpdate.Deserialize(data);
    if (ConflictResolver.ShouldApply(update))
    {
        ApplyStateUpdate(update);
    }
};
```

### Error Handling

```csharp
try
{
    session.JoinSession(sessionCode);
}
catch (SessionNotFoundException)
{
    Console.WriteLine("Session not found. Check code.");
}
catch (NetworkTimeoutException)
{
    Console.WriteLine("Connection timeout. Check firewall/NAT.");
}
catch (VersionMismatchException ex)
{
    Console.WriteLine($"Mod version mismatch: {ex.Message}");
}
```

## Troubleshooting

### Connection Issues

**Symptom:** Cannot connect to session  
**Solutions:**
1. Verify port forwarding (default 25000)
2. Check firewall rules for Subnautica2.exe and BepInEx
3. Confirm both players have identical mod versions
4. Enable debug mode: `"debug_mode": true` in config
5. Try changing STUN server in network config

```bash
# Check logs
tail -f BepInEx/LogOutput.log | grep DeepSynergy

# Common error patterns:
# "WebRTC connection timeout" → NAT/firewall issue
# "Version mismatch" → Update mod to matching version
# "Session not found" → Invalid session code
```

### Inventory Desync

**Symptom:** Players see different inventories  
**Solutions:**

```bash
# Force resync via console
/sync_inventory

# Or programmatically
inventorySync.ForceFullSync();
```

Check for conflicting mods:
```bash
# Disable other inventory mods temporarily
# Verify DeepSynergy.dll is latest version
```

### API Integration Not Working

**Symptom:** AI narration not generating  
**Solutions:**

```bash
# Verify environment variables
echo $OPENAI_API_KEY  # Linux/Mac
echo %OPENAI_API_KEY%  # Windows

# Test API key
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

Check config:
```json
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "api_key_env": "OPENAI_API_KEY",  // Must match env var name
      "model": "gpt-4"
    }
  }
}
```

### Performance Issues

**Symptom:** Lag/stuttering with 3+ players  
**Solutions:**

1. Reduce creature spawn rate:
```json
{
  "creature_spawn_divider": 2  // Halves creature count
}
```

2. Lower sync frequency (advanced):
```csharp
StateSync.SetUpdateInterval(100);  // ms, default 50
```

3. Disable API features:
```json
{
  "api_integration": {
    "openai": { "enabled": false },
    "claude": { "enabled": false }
  }
}
```

### Log Analysis

```bash
# Enable debug logging
# In synergy_profile.json:
"debug_mode": true

# Key log patterns:
# "[StateSync] Merkle mismatch" → Inventory desync
# "[Network] Peer disconnected" → Connection loss
# "[API] Rate limit exceeded" → Reduce API calls
```

## Advanced Configuration

### Custom Scaling Formula

```csharp
// Override default scaling in plugin
public class CustomScaler : DifficultyScaler
{
    protected override float CalculateCreatureMultiplier(int players)
    {
        // Custom formula: exponential reduction
        return Mathf.Pow(0.8f, players - 1);
    }
}
```

### Session Migration

```csharp
// Implement host migration on disconnect
_channel.OnHostDisconnected += () => {
    var survivingPeer = _peers.OrderBy(p => p.Latency).First();
    if (survivingPeer.Id == localPlayerId)
    {
        PromoteToHost();
    }
};

private void PromoteToHost()
{
    _channel.BecomeHost();
    BroadcastFullState();
}
```

### Blueprint Sharing

```csharp
// Sync blueprint unlocks
Game.BlueprintManager.OnUnlock += (blueprint) => {
    if (_config.SharedBlueprints)
    {
        var update = new StateUpdate
        {
            Type = UpdateType.BlueprintUnlock,
            Data = Encoding.UTF8.GetBytes(blueprint.Id)
        };
        _channel.Broadcast(update.Serialize());
    }
};
```

## Environment Variables

```bash
# API keys (never commit these)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."

# Optional network overrides
export SYNERGY_PORT=25000
export SYNERGY_STUN_SERVER="stun:custom.server.com:3478"
```

## Compatibility

- **OS:** Windows 10/11, macOS 11+, Linux (Ubuntu 20.04+, Steam Deck)
- **BepInEx:** 6.0.x (IL2CPP)
- **Game Version:** Subnautica 2 (2026 release)
- **Conflicting Mods:** Avoid other multiplayer/inventory mods
