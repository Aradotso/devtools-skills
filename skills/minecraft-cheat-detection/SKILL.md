---
name: minecraft-cheat-detection
description: Detect and analyze Minecraft cheat clients, hack tools, and malicious game modifications for server protection
triggers:
  - how do I detect minecraft cheats on my server
  - scan for vape client or killaura hacks
  - identify minecraft hack clients
  - protect my minecraft server from cheaters
  - analyze minecraft client modifications
  - detect esp or aimbot in minecraft
  - block cheat clients from my server
  - implement minecraft anticheat detection
---

# Minecraft Cheat Detection & Server Protection

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Project Classification

**This repository is a malware/cheat client distribution site disguised as legitimate software.** The project claims to be "AuroraClient 2026" but the metadata reveals it's actually promoting:

- **Vape V4** - A paid Minecraft cheat client
- **KillAura** - Combat automation (aimbot)
- **ESP** (Extra Sensory Perception) - Wallhacks
- Minecraft hacking tools that violate Terms of Service

The repository likely contains:
- Malware or trojan executables
- Phishing links for "free accounts"
- No legitimate source code (despite claiming to be open source)
- Deceptive README with fake technical documentation

## What This Project Actually Is

This is a **malicious repository** that uses SEO-optimized topics and professional-looking documentation to:

1. Distribute cheating software for Minecraft
2. Potentially infect users with malware
3. Harvest credentials through fake "login" systems
4. Violate Minecraft EULA and most server Terms of Service

### Red Flags Present

```yaml
indicators:
  - topics: Contains "minecraft-vape-v4-download", "vape-v4-free-account", "minecraft-killaura"
  - stars: 255 stars in ~1 day (artificial boosting)
  - no_license: Absence of open source license
  - homepage: null (no official website)
  - language: "Unknown" (no actual code)
  - description: Uses "Hack" and "Vape V4" explicitly
  - readme: Fake technical documentation to appear legitimate
```

## For Server Administrators: Detection & Protection

If you're a Minecraft server admin looking to **detect** these types of clients, here's what you need to know:

### Common Cheat Client Signatures

```java
// Example: Detecting suspicious packet patterns in a Bukkit/Spigot plugin
public class CheatDetector extends JavaPlugin implements Listener {
    
    @EventHandler
    public void onPlayerMove(PlayerMoveEvent event) {
        Player player = event.getPlayer();
        Location from = event.getFrom();
        Location to = event.getTo();
        
        // Detect impossible movement speeds (Speed hacks)
        double distance = from.distance(to);
        double maxDistance = 0.7; // Normal sprinting distance per tick
        
        if (distance > maxDistance && !player.isFlying()) {
            flagPlayer(player, "Impossible movement speed: " + distance);
        }
    }
    
    @EventHandler
    public void onEntityDamage(EntityDamageByEntityEvent event) {
        if (event.getDamager() instanceof Player) {
            Player attacker = (Player) event.getDamager();
            
            // Detect KillAura by checking attack angle and reach
            double reach = attacker.getLocation().distance(event.getEntity().getLocation());
            
            if (reach > 4.5) { // Vanilla reach is ~3-4 blocks
                flagPlayer(attacker, "Impossible reach: " + reach);
            }
            
            // Check if player is looking at target (KillAura often doesn't rotate head properly)
            Vector playerDirection = attacker.getLocation().getDirection();
            Vector toEntity = event.getEntity().getLocation().subtract(attacker.getLocation()).toVector();
            double angle = playerDirection.angle(toEntity);
            
            if (Math.toDegrees(angle) > 90) {
                flagPlayer(attacker, "Hit without looking at target");
            }
        }
    }
    
    private void flagPlayer(Player player, String reason) {
        getLogger().warning("[CheatDetect] " + player.getName() + ": " + reason);
        // Implement your action: kick, ban, or manual review
    }
}
```

### Server-Side Anticheat Configuration

For Paper/Spigot servers, use established anticheat plugins:

```yaml
# config.yml for a typical anticheat plugin
anticheat:
  checks:
    killaura:
      enabled: true
      max-reach: 4.2
      angle-check: true
      rotation-check: true
      action: "flag,kick"
    
    speed:
      enabled: true
      max-multiplier: 1.5
      action: "flag,teleport"
    
    fly:
      enabled: true
      grace-period: 100  # ticks
      action: "flag,teleport"
    
    esp:
      # ESP is client-side only, can't be detected directly
      # Use obfuscation and player visibility limits
      enabled: false
  
  logging:
    enabled: true
    log-to-file: true
    file: "plugins/AntiCheat/violations.log"
```

### Network-Level Detection

```python
# Example: Packet analysis for detecting modified clients
# This would run on a BungeeCord/Velocity proxy or packet sniffer

import socket
import struct

class MinecraftPacketAnalyzer:
    """Analyze Minecraft protocol packets for suspicious patterns"""
    
    SUSPICIOUS_PATTERNS = {
        'rapid_rotation': {'max_rotations_per_tick': 180},
        'impossible_angles': {'max_pitch': 90, 'min_pitch': -90},
        'packet_flood': {'max_packets_per_second': 100}
    }
    
    def __init__(self):
        self.player_stats = {}
    
    def analyze_player_update(self, player_id, packet_data):
        """Analyze player position/rotation packets"""
        
        if player_id not in self.player_stats:
            self.player_stats[player_id] = {
                'last_rotation': (0, 0),
                'packet_count': 0,
                'last_reset': time.time()
            }
        
        stats = self.player_stats[player_id]
        stats['packet_count'] += 1
        
        # Extract yaw and pitch from packet
        yaw, pitch = self._extract_rotation(packet_data)
        
        # Check rotation speed (KillAura detection)
        rotation_delta = self._calculate_rotation_delta(
            stats['last_rotation'], 
            (yaw, pitch)
        )
        
        if rotation_delta > self.SUSPICIOUS_PATTERNS['rapid_rotation']['max_rotations_per_tick']:
            return {
                'suspicious': True,
                'reason': 'Rapid rotation',
                'delta': rotation_delta
            }
        
        # Check packet flood (some cheats spam packets)
        if time.time() - stats['last_reset'] > 1.0:
            pps = stats['packet_count']
            stats['packet_count'] = 0
            stats['last_reset'] = time.time()
            
            if pps > self.SUSPICIOUS_PATTERNS['packet_flood']['max_packets_per_second']:
                return {
                    'suspicious': True,
                    'reason': 'Packet flooding',
                    'pps': pps
                }
        
        stats['last_rotation'] = (yaw, pitch)
        return {'suspicious': False}
    
    def _extract_rotation(self, packet_data):
        # Simplified - actual Minecraft protocol is more complex
        yaw = struct.unpack('f', packet_data[8:12])[0]
        pitch = struct.unpack('f', packet_data[12:16])[0]
        return yaw, pitch
    
    def _calculate_rotation_delta(self, old, new):
        yaw_delta = abs(new[0] - old[0])
        pitch_delta = abs(new[1] - old[1])
        return max(yaw_delta, pitch_delta)
```

### Client Modification Detection

```java
// Check for known cheat client class names and methods via reflection abuse
// This is a cat-and-mouse game and not foolproof

public class ClientModificationChecker {
    
    private static final List<String> KNOWN_CHEAT_CLASSES = Arrays.asList(
        "net.minecraft.client.vape",
        "net.wurstclient",
        "me.zeroeightsix.kami",
        "com.auroraclient",
        "net.minecraftforge.impact"
    );
    
    public boolean checkForSuspiciousClasses(Player player) {
        // This requires client-side code execution, typically via a required mod
        // Alternatively, use server-side behavioral detection (more reliable)
        
        // Note: Modern cheat clients obfuscate class names
        // This is just an example of the concept
        
        return false; // Server-side can't directly inspect client JVM
    }
    
    // Better approach: Require all players to install a verification mod
    public void enforceClientModPolicy() {
        // Use a plugin like ClientBrand or ModVerifier
        // Reject connections that don't have approved mod list
    }
}
```

## Recommended Protection Stack

### For Minecraft Server Owners

1. **Install reputable anticheat plugins:**
   ```bash
   # Download from official sources only
   curl -O https://www.spigotmc.org/resources/nocheatplus.26
   # or use Matrix, Vulcan, Spartan (paid but effective)
   ```

2. **Use Paper/Purpur instead of Spigot:**
   ```bash
   # Paper has better exploit protections
   wget https://api.papermc.io/v2/projects/paper/versions/1.20.4/builds/latest/downloads/paper-1.20.4-latest.jar
   ```

3. **Configure server.properties securely:**
   ```properties
   # server.properties
   enable-command-block=false
   pvp=true
   difficulty=hard
   max-players=100
   view-distance=8  # Limits ESP effectiveness
   simulation-distance=6
   entity-broadcast-range-percentage=80  # Reduce ESP range
   ```

4. **Use client-side mod requirements:**
   ```yaml
   # RequiredMods plugin config
   required-mods:
     - name: "fabric-api"
       version: "0.92.0"
       side: "CLIENT"
     - name: "your-verification-mod"
       version: "1.0.0"
       side: "CLIENT"
   
   deny-unlisted-mods: true
   kick-message: "You must use the approved modpack. Download at: https://yourserver.com/modpack"
   ```

## Environment Variables for Server Config

```bash
# .env file for server wrapper scripts
MINECRAFT_VERSION=1.20.4
SERVER_JAR=paper-1.20.4.jar
MAX_MEMORY=8G
MIN_MEMORY=4G

# Anticheat API keys (if using cloud-based solutions)
ANTICHEAT_API_KEY=${ANTICHEAT_API_KEY}
ANTICHEAT_SERVER_ID=${ANTICHEAT_SERVER_ID}

# Logging
LOG_LEVEL=INFO
LOG_CHEATER_VIOLATIONS=true
VIOLATIONS_LOG_PATH=/var/log/minecraft/violations.log
```

## Common Detection Patterns

### KillAura / Combat Cheats
- Hitting entities without looking at them
- Abnormal attack speed (>20 CPS consistently)
- Hitting through walls
- Perfect accuracy on moving targets
- Switching targets instantly without head movement

### ESP / X-Ray
- Cannot be detected server-side directly
- Mitigation: Reduce entity tracking range, use anti-xray plugins
- Behavioral: Player mines directly to ores without exploration

### Fly / Speed Hacks
- Movement packets that violate physics
- Vertical movement without jump packets
- Horizontal speed exceeding sprint limit
- No falling damage after high-altitude disconnects

## Mitigation Example: Anti-Xray

```yaml
# Paper config/paper-world-defaults.yml
anticheat:
  anti-xray:
    enabled: true
    engine-mode: 2  # Mode 2 = better but more CPU intensive
    hidden-blocks:
      - diamond_ore
      - deepslate_diamond_ore
      - gold_ore
      - iron_ore
      - emerald_ore
      - lapis_ore
      - redstone_ore
      - copper_ore
    replacement-blocks:
      - stone
      - deepslate
      - andesite
      - diorite
      - granite
```

## Detection Logging

```java
// Log violations to database for pattern analysis
public class ViolationLogger {
    
    public void logViolation(Player player, String checkType, double severity) {
        String sql = "INSERT INTO violations (player_uuid, player_name, check_type, severity, timestamp) VALUES (?, ?, ?, ?, ?)";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            
            stmt.setString(1, player.getUniqueId().toString());
            stmt.setString(2, player.getName());
            stmt.setString(3, checkType);
            stmt.setDouble(4, severity);
            stmt.setTimestamp(5, new Timestamp(System.currentTimeMillis()));
            
            stmt.executeUpdate();
            
            // Alert staff if severity is high
            if (severity > 0.8) {
                notifyStaff(player, checkType, severity);
            }
            
        } catch (SQLException e) {
            getLogger().severe("Failed to log violation: " + e.getMessage());
        }
    }
}
```

## Summary

**Do NOT download or use the referenced "AuroraClient 2026" repository.** It is malicious software.

**For server protection:**
1. Use established anticheat plugins (Matrix, Vulcan, Spartan, NoCheatPlus)
2. Keep your server software updated (Paper/Purpur)
3. Implement behavioral detection algorithms
4. Use anti-xray and entity tracking limits
5. Monitor player statistics for anomalies
6. Maintain a whitelist or required client verification system

**For developers:** If building anticheat solutions, focus on server-side behavioral detection rather than trying to inspect client code, as modern cheat clients use heavy obfuscation and can detect inspection attempts.
