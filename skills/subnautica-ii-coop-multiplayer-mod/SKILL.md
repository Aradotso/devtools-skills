---
name: subnautica-ii-coop-multiplayer-mod
description: BepInEx-based multiplayer mod for Subnautica 2 enabling synchronized cooperative gameplay with shared inventory, base building, and AI integration
triggers:
  - how do I install the Subnautica 2 co-op mod
  - configure Deep Synergy multiplayer session
  - set up BepInEx for Subnautica 2 multiplayer
  - create a co-op session in Subnautica 2
  - sync inventory in Subnautica multiplayer mod
  - troubleshoot Subnautica 2 mod connection issues
  - enable AI narration in Subnautica co-op
  - adjust difficulty scaling for multiplayer Subnautica
---

# Subnautica II Deep Synergy Multiplayer Mod

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Deep Synergy Multiplayer Mod transforms Subnautica 2 into a cooperative survival experience using BepInEx's IL2CPP modding framework. It implements deterministic session synchronization, peer-to-peer networking via WebRTC, and optional AI integration for narrative generation.

**Key Capabilities:**
- Synchronized multiplayer sessions (2-8 players)
- Shared inventory and base building with conflict resolution
- Adaptive difficulty scaling based on player count
- Cross-platform support (Windows, Linux, macOS)
- BepInEx plugin architecture (no game file modification)
- Optional OpenAI/Claude API integration for dynamic narration

## Installation

### Prerequisites

1. **Subnautica 2** installed via Steam/GOG
2. **BepInEx 6.0.x** for Unity IL2CPP games

### Step-by-Step Installation

```bash
# 1. Navigate to Subnautica 2 installation directory
cd "C:\Program Files (x86)\Steam\steamapps\common\Subnautica2"

# 2. Install BepInEx (if not already installed)
# Download BepInEx_UnityIL2CPP_x64 from https://github.com/BepInEx/BepInEx/releases
# Extract to game root directory

# 3. Download Deep Synergy mod from release page
# Extract DeepSynergy.dll to BepInEx/plugins/

# 4. Verify installation structure
BepInEx/
├── plugins/
│   └── DeepSynergy.dll
├── config/
│   └── synergy_profile.json (created on first run)
└── core/
    └── (BepInEx core files)
```

## Configuration

### Basic Session Profile

Create `BepInEx/config/synergy_profile.json`:

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
  "time_of_day_sync": "host",
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

### Configuration Fields

| Field | Type | Description | Valid Values |
|-------|------|-------------|--------------|
| `session_name` | string | Display name for session | Any string |
| `max_players` | int | Player limit | 2-8 |
| `difficulty_scale` | string | How difficulty adjusts | `adaptive`, `static`, `manual` |
| `resource_multiplier` | float | Resource spawn multiplier | 0.5-3.0 |
| `oxygen_consumption` | float | Oxygen drain rate modifier | 0.5-2.0 |
| `creature_spawn_divider` | int | Reduces creature spawns | 1-4 |
| `shared_blueprints` | bool | Share discovered blueprints | `true`/`false` |
| `time_of_day_sync` | string | Time sync strategy | `host`, `all`, `independent` |

### AI Integration Configuration

To enable AI narration (optional):

```json
{
  "api_integration": {
    "openai": {
      "enabled": true,
      "role": "narrator",
      "api_key_env": "OPENAI_API_KEY",
      "model": "gpt-4",
      "temperature": 0.7,
      "max_tokens": 150
    },
    "claude": {
      "enabled": true,
      "role": "lore_engine",
      "api_key_env": "ANTHROPIC_API_KEY",
      "model": "claude-3-sonnet-20240229",
      "max_tokens": 200
    }
  }
}
```

Set environment variables before launching:

```bash
# Windows
set OPENAI_API_KEY=your_key_here
set ANTHROPIC_API_KEY=your_key_here

# Linux/macOS
export OPENAI_API_KEY=your_key_here
export ANTHROPIC_API_KEY=your_key_here
```

## Console Commands

Access via BepInEx terminal (F12 in-game by default):

### Session Management

```bash
# Host a new session
/start_server

# Join existing session with code
/join_session 9B2A-4C7D-E8F1

# Leave current session
/disconnect

# Show session info
/synergy_status
```

### Gameplay Modifiers

```bash
# Adjust difficulty scaling (1.0 = normal, 1.5 = 50% harder)
/synergy_scale 1.5

# Override world seed for all clients
/seed_override 8251

# Force inventory synchronization
/sync_inventory

# Teleport to another player
/tp_to_player PlayerName
```

### AI Narration

```bash
# Trigger AI-generated narrative about current context
/api_narrate "exploring the kelp forest"

# Generate marine biology log for last scanned creature
/api_lore generate_species_log

# Get contextual survival hint
/api_hint
```

## Key Patterns

### Starting a Co-op Session

**Host workflow:**

```bash
# 1. Launch game with mod installed
# 2. Open console (F12)
/start_server

# Output: "Server created: session code = 9B2A-4C7D-E8F1"
# 3. Share code with friends
# 4. Wait for players to connect
/synergy_status
```

**Client workflow:**

```bash
# 1. Launch game with mod installed
# 2. Open console (F12)
/join_session 9B2A-4C7D-E8F1

# Output: "Connected to host. Syncing world state..."
# 3. Wait for synchronization to complete
```

### Shared Inventory Management

The mod uses Merkle trees for inventory verification:

```json
// Inventory sync status output from /synergy_status
{
  "connected_peers": 3,
  "latency_avg_ms": 45,
  "inventory_hash": "0xFA342B1E",
  "sync_progress": "100%",
  "conflicts_resolved": 2
}
```

**Conflict resolution:** If two players pick up the same item simultaneously, the mod uses timestamps to determine ownership. The "losing" player's action is rolled back.

### Dynamic Difficulty Scaling

Example scaling for a 4-player session with `"difficulty_scale": "adaptive"`:

```json
{
  "base_creature_spawn_rate": 1.0,
  "adjusted_spawn_rate": 0.75,
  "resource_nodes": 1.3,
  "oxygen_efficiency": 1.1,
  "damage_multiplier": 1.2
}
```

**Formula:** `adjusted_value = base_value * (1 + (player_count - 1) * scaling_factor)`

### BepInEx Plugin Integration

For developers extending the mod:

```csharp
using BepInEx;
using BepInEx.IL2CPP;
using DeepSynergy.Core;

namespace MyCustomExtension
{
    [BepInPlugin(GUID, Name, Version)]
    [BepInDependency("com.deepsynergy.mod", BepInDependency.DependencyFlags.HardDependency)]
    public class CustomExtension : BasePlugin
    {
        public const string GUID = "com.myname.customextension";
        public const string Name = "Custom Extension";
        public const string Version = "1.0.0";

        public override void Load()
        {
            // Hook into Deep Synergy events
            SessionManager.OnPlayerJoined += HandlePlayerJoined;
            InventorySync.OnItemPickup += HandleItemPickup;
        }

        private void HandlePlayerJoined(PlayerData player)
        {
            Log.LogInfo($"Player {player.Name} joined. ID: {player.Id}");
        }

        private void HandleItemPickup(ItemPickupEvent evt)
        {
            Log.LogInfo($"Item {evt.ItemId} picked up by {evt.PlayerId}");
        }
    }
}
```

## Troubleshooting

### Connection Issues

**Problem:** "Failed to establish peer connection"

```bash
# Check firewall settings (allow UDP ports 7777-7787)
# Windows Firewall example:
netsh advfirewall firewall add rule name="Subnautica2 Coop" dir=in action=allow protocol=UDP localport=7777-7787

# Verify NAT type
/synergy_status
# Look for "NAT Type: Symmetric" (problematic) vs "Moderate/Open" (good)

# Force relay mode if direct connection fails
# In synergy_profile.json:
{
  "network": {
    "force_relay": true,
    "relay_server": "turn:relay.example.com:3478"
  }
}
```

**Problem:** "Inventory desync detected"

```bash
# Force full resynchronization
/sync_inventory --full

# Check for conflicting mods
# Disable all other BepInEx plugins temporarily

# Verify mod version matches across all clients
/synergy_status
# All players must show same "Mod Version: x.x.x"
```

### Performance Issues

**Problem:** Low FPS in multiplayer vs single-player

```bash
# Reduce creature AI sync frequency in synergy_profile.json:
{
  "performance": {
    "creature_ai_sync_rate_ms": 500,  // Default: 250
    "position_sync_rate_ms": 100,      // Default: 50
    "inventory_sync_rate_ms": 1000     // Default: 500
  }
}

# Disable AI integration if enabled
{
  "api_integration": {
    "openai": {"enabled": false},
    "claude": {"enabled": false}
  }
}
```

### Session Code Not Working

**Problem:** "Invalid session code"

```bash
# Ensure exact format: XXXX-XXXX-XXXX (4 hex chars per segment)
# Codes are case-insensitive

# Check if host session is still active
# Host should see:
/synergy_status
# Output: "Session State: Active, Accepting Connections: Yes"

# Regenerate session code if corrupted
# Host runs:
/restart_server
```

### Mod Not Loading

**Problem:** Mod doesn't appear in BepInEx console

```bash
# 1. Verify BepInEx installation
# Check for BepInEx/LogOutput.log

# 2. Enable verbose logging
# Edit BepInEx/config/BepInEx.cfg:
[Logging.Console]
Enabled = true
LogLevels = All

# 3. Check for IL2CPP compatibility
# Ensure you downloaded BepInEx IL2CPP version, not Mono

# 4. Verify file structure
BepInEx/plugins/DeepSynergy.dll  # Must be directly in plugins/
```

## Advanced Usage

### Custom Event Hooks

Monitor mod events via BepInEx logging:

```csharp
using DeepSynergy.Events;

EventBus.Subscribe<BasePartPlacedEvent>(evt => 
{
    Log.LogInfo($"Base part placed: {evt.PartType} at {evt.Position}");
    // Custom logic here
});

EventBus.Subscribe<CreatureSpawnEvent>(evt => 
{
    if (evt.CreatureType == "Leviathan")
    {
        Log.LogWarning("Leviathan spawned nearby!");
    }
});
```

### Dedicated Server Mode

For 24/7 hosting (experimental):

```json
{
  "server_mode": {
    "enabled": true,
    "headless": true,
    "auto_save_interval_minutes": 15,
    "max_idle_time_minutes": 30,
    "restart_on_empty": true
  }
}
```

Run with:

```bash
# Linux
./Subnautica2.x86_64 -batchmode -nographics

# Windows
Subnautica2.exe -batchmode -nographics
```

### Localization

Override UI language:

```json
{
  "locale": "ja_JP",  // Japanese
  "locale_fallback": "en_US"
}
```

Supported locales: `en_US`, `zh_CN`, `ja_JP`, `de_DE`, `fr_FR`, `pt_BR`, `ru_RU`, `es_ES`, `ko_KR`

## API Reference

### Environment Variables

- `OPENAI_API_KEY` - OpenAI API key for narration features
- `ANTHROPIC_API_KEY` - Anthropic API key for lore generation
- `SYNERGY_DEBUG` - Set to `1` for verbose debug logging
- `SYNERGY_RELAY_SERVER` - Override default TURN server URL

### Configuration File Locations

- **Windows:** `%USERPROFILE%\AppData\Roaming\Subnautica2\BepInEx\config\`
- **Linux:** `~/.config/unity3d/UnknownWorlds/Subnautica2/BepInEx/config/`
- **macOS:** `~/Library/Application Support/UnknownWorlds/Subnautica2/BepInEx/config/`
