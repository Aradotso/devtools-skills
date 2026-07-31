---
name: nova-client-proxy
description: Cross-platform VPN client built on Flutter and sing-box with Nova Radar clean-IP scanner, supporting VLESS/VMess/Trojan/Shadowsocks protocols
triggers:
  - how do I use Nova Client proxy
  - set up Nova Client VPN on Android iOS Windows
  - configure Nova Proxy subscription
  - scan Cloudflare clean IPs with Nova Radar
  - add proxy profiles to Nova Client
  - troubleshoot Nova Client connection issues
  - use sing-box with Nova Client
  - route Iranian sites directly in Nova Client
---

# Nova Client Proxy Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Nova Client is a cross-platform VPN/proxy client built with Flutter that wraps the sing-box core. It's a streamlined fork of Karing with the Nova Proxy design language and integrates the Nova Radar Cloudflare clean-IP scanner. Supports Android, iOS, Windows, macOS, and Linux.

**Key capabilities:**
- Multi-protocol support: VLESS, VMess, Trojan, Shadowsocks, Hysteria2, TUIC
- Subscription management with auto-update
- Built-in Nova Radar scanner for finding clean Cloudflare IPs
- Smart routing (`.ir` direct bypass for Iranian sites)
- Per-node latency testing and auto-fastest selection
- Bilingual (English/Persian) with RTL support

## Installation

### Android
```bash
# Direct APK download from releases
wget https://github.com/IRNova/Nova-Client/releases/latest/download/nova-client-android.apk

# Install via adb
adb install nova-client-android.apk
```

### iOS/iPadOS
- TestFlight beta or App Store (when available)
- Download from GitHub releases page

### Windows
```powershell
# Download installer
Invoke-WebRequest -Uri "https://github.com/IRNova/Nova-Client/releases/latest/download/nova-client-windows.exe" -OutFile "nova-client-setup.exe"

# Run installer
.\nova-client-setup.exe

# Or use portable ZIP version
```

### macOS
```bash
# Download DMG
curl -LO https://github.com/IRNova/Nova-Client/releases/latest/download/nova-client-macos.dmg

# Mount and install
hdiutil attach nova-client-macos.dmg
cp -R "/Volumes/Nova Client/Nova Client.app" /Applications/
```

### Linux
```bash
# AppImage (no installation required)
wget https://github.com/IRNova/Nova-Client/releases/latest/download/nova-client-linux.AppImage
chmod +x nova-client-linux.AppImage
./nova-client-linux.AppImage

# Or tarball
tar xf nova-client-linux.tar.xz
cd nova-client
./nova-client
```

## Configuration Files

Nova Client stores configuration in platform-specific directories:

- **Android**: `/data/data/com.irnova.client/files/`
- **iOS**: App sandbox documents directory
- **Windows**: `%APPDATA%\NovaClient\`
- **macOS**: `~/Library/Application Support/NovaClient/`
- **Linux**: `~/.config/nova-client/`

### Key Files
```
config/
├── profiles.json          # Subscription and manual profiles
├── routing.json           # Routing rules and modes
├── sing-box-config.json   # Generated sing-box configuration
└── radar-sources.json     # Nova Radar IP sources
```

## Adding Proxy Profiles

### Subscription URL
```json
{
  "profiles": [
    {
      "id": "uuid-here",
      "name": "Nova Proxy Official",
      "type": "subscription",
      "url": "https://example.com/subscription",
      "autoUpdate": true,
      "updateInterval": 86400
    }
  ]
}
```

### Manual Node (VLESS Example)
```json
{
  "profiles": [
    {
      "id": "manual-1",
      "name": "Custom VLESS",
      "type": "manual",
      "protocol": "vless",
      "address": "example.com",
      "port": 443,
      "uuid": "YOUR_UUID",
      "flow": "xtls-rprx-vision",
      "encryption": "none",
      "network": "tcp",
      "security": "reality",
      "sni": "www.google.com",
      "fingerprint": "chrome",
      "publicKey": "YOUR_PUBLIC_KEY",
      "shortId": "SHORT_ID"
    }
  ]
}
```

### VMess Configuration
```json
{
  "protocol": "vmess",
  "address": "server.example.com",
  "port": 443,
  "uuid": "YOUR_UUID",
  "alterId": 0,
  "security": "auto",
  "network": "ws",
  "path": "/vmess-path",
  "host": "server.example.com",
  "tls": true,
  "sni": "server.example.com",
  "alpn": ["h2", "http/1.1"]
}
```

### Trojan Configuration
```json
{
  "protocol": "trojan",
  "address": "trojan.example.com",
  "port": 443,
  "password": "YOUR_PASSWORD",
  "sni": "trojan.example.com",
  "alpn": ["h2"],
  "fingerprint": "firefox",
  "allowInsecure": false
}
```

## Routing Configuration

### Smart Routing for Iran
```json
{
  "routing": {
    "mode": "rule",
    "rules": [
      {
        "type": "direct",
        "domain_suffix": [".ir"],
        "description": "Iranian sites bypass proxy"
      },
      {
        "type": "block",
        "domain_keyword": ["ads", "analytics", "tracker"],
        "description": "Ad blocking"
      },
      {
        "type": "direct",
        "ip_cidr": ["192.168.0.0/16", "10.0.0.0/8"],
        "description": "LAN bypass"
      },
      {
        "type": "proxy",
        "domain": "*",
        "description": "Default proxy"
      }
    ]
  }
}
```

### Routing Modes
- **proxy**: Route all traffic through proxy
- **direct**: No proxy (direct connection)
- **rule**: Use custom routing rules (recommended)
- **auto**: Automatic fastest node selection

## Nova Radar Scanner Usage

The Nova Radar scanner finds clean Cloudflare IPs for use in CDN-based proxies.

### Scanner Configuration (Dart/Flutter)
```dart
import 'package:nova_client/radar/scanner.dart';

class RadarScanner {
  final List<String> sources = [
    'https://www.cloudflare.com/ips-v4',
    'https://raw.githubusercontent.com/vfarid/cf-clean-ips/main/list.txt',
    // Add more sources
  ];
  
  Future<List<CleanIP>> scanCleanIPs({
    required int count,
    required int threads,
    required int timeout,
    Function(double)? onProgress,
  }) async {
    final ips = await _fetchIPsFromSources(sources);
    final results = <CleanIP>[];
    
    int scanned = 0;
    final total = ips.length;
    
    // Phase 1: TCP connectivity test
    await Future.wait(
      ips.map((ip) async {
        final socket = await _testTCPConnect(ip, timeout);
        if (socket != null) {
          socket.close();
          
          // Phase 2: TLS handshake test
          final latency = await _testTLSHandshake(ip, timeout);
          if (latency > 0) {
            results.add(CleanIP(ip: ip, latency: latency));
          }
        }
        
        scanned++;
        onProgress?.call(scanned / total);
      }),
      eagerError: false,
    );
    
    // Sort by latency
    results.sort((a, b) => a.latency.compareTo(b.latency));
    return results.take(count).toList();
  }
  
  Future<Socket?> _testTCPConnect(String ip, int timeout) async {
    try {
      return await Socket.connect(
        ip,
        443,
        timeout: Duration(milliseconds: timeout),
      );
    } catch (_) {
      return null;
    }
  }
  
  Future<int> _testTLSHandshake(String ip, int timeout) async {
    final stopwatch = Stopwatch()..start();
    
    try {
      final socket = await SecureSocket.connect(
        ip,
        443,
        timeout: Duration(milliseconds: timeout),
        onBadCertificate: (_) => true,
      );
      
      await socket.close();
      return stopwatch.elapsedMilliseconds;
    } catch (_) {
      return -1;
    }
  }
  
  Future<List<String>> _fetchIPsFromSources(List<String> sources) async {
    final allIPs = <String>{};
    
    for (final source in sources) {
      try {
        final response = await http.get(Uri.parse(source));
        if (response.statusCode == 200) {
          final ips = response.body
              .split('\n')
              .where((line) => _isValidIP(line.trim()))
              .toList();
          allIPs.addAll(ips);
        }
      } catch (e) {
        print('Failed to fetch from $source: $e');
      }
    }
    
    return allIPs.toList();
  }
  
  bool _isValidIP(String ip) {
    final ipPattern = RegExp(
      r'^(\d{1,3}\.){3}\d{1,3}(/\d{1,2})?$'
    );
    return ipPattern.hasMatch(ip);
  }
}

class CleanIP {
  final String ip;
  final int latency;
  
  CleanIP({required this.ip, required this.latency});
}
```

### Using Scanner Results
```dart
// Scan and get clean IPs
final scanner = RadarScanner();
final cleanIPs = await scanner.scanCleanIPs(
  count: 10,
  threads: 100,
  timeout: 2000,
  onProgress: (progress) {
    print('Progress: ${(progress * 100).toStringAsFixed(1)}%');
  },
);

// Export results
final export = cleanIPs.map((ip) => '${ip.ip},${ip.latency}ms').join('\n');
await File('clean-ips.txt').writeAsString(export);

// Use best IP in proxy config
final bestIP = cleanIPs.first.ip;
final config = {
  'address': bestIP,
  'port': 443,
  'sni': 'cdn.cloudflare.com',
};
```

## Latency Testing

### Test Single Node
```dart
import 'package:nova_client/core/latency.dart';

Future<int> testNodeLatency(ProxyNode node) async {
  final stopwatch = Stopwatch()..start();
  
  try {
    final socket = await Socket.connect(
      node.address,
      node.port,
      timeout: Duration(seconds: 5),
    );
    
    await socket.close();
    return stopwatch.elapsedMilliseconds;
  } catch (e) {
    return -1; // Connection failed
  }
}
```

### Batch Latency Test
```dart
Future<Map<String, int>> testAllNodes(List<ProxyNode> nodes) async {
  final results = <String, int>{};
  
  await Future.wait(
    nodes.map((node) async {
      final latency = await testNodeLatency(node);
      results[node.id] = latency;
    }),
  );
  
  return results;
}

// Auto-select fastest
ProxyNode selectFastest(List<ProxyNode> nodes, Map<String, int> latencies) {
  return nodes.reduce((best, node) {
    final bestLatency = latencies[best.id] ?? 999999;
    final nodeLatency = latencies[node.id] ?? 999999;
    return nodeLatency < bestLatency && nodeLatency > 0 ? node : best;
  });
}
```

## sing-box Core Integration

### Generate sing-box Config
```dart
Map<String, dynamic> generateSingBoxConfig(ProxyNode node, RoutingRules rules) {
  return {
    'log': {'level': 'info'},
    'dns': {
      'servers': [
        {'tag': 'google', 'address': '8.8.8.8'},
        {'tag': 'local', 'address': '223.5.5.5', 'detour': 'direct'}
      ],
      'rules': [
        {'domain_suffix': ['.ir'], 'server': 'local'}
      ]
    },
    'inbounds': [
      {
        'type': 'tun',
        'tag': 'tun-in',
        'interface_name': 'nova0',
        'inet4_address': '172.19.0.1/30',
        'auto_route': true,
        'strict_route': true,
        'stack': 'system'
      }
    ],
    'outbounds': [
      _generateOutbound(node),
      {'type': 'direct', 'tag': 'direct'},
      {'type': 'block', 'tag': 'block'}
    ],
    'route': {
      'rules': _generateRoutingRules(rules),
      'auto_detect_interface': true
    }
  };
}

Map<String, dynamic> _generateOutbound(ProxyNode node) {
  switch (node.protocol) {
    case 'vless':
      return {
        'type': 'vless',
        'tag': 'proxy',
        'server': node.address,
        'server_port': node.port,
        'uuid': node.uuid,
        'flow': node.flow,
        'network': node.network,
        'tls': {
          'enabled': true,
          'server_name': node.sni,
          'utls': {'enabled': true, 'fingerprint': node.fingerprint},
          'reality': {
            'enabled': true,
            'public_key': node.publicKey,
            'short_id': node.shortId
          }
        }
      };
      
    case 'vmess':
      return {
        'type': 'vmess',
        'tag': 'proxy',
        'server': node.address,
        'server_port': node.port,
        'uuid': node.uuid,
        'security': node.security,
        'alter_id': node.alterId,
        'transport': {
          'type': node.network,
          'path': node.path,
          'headers': {'Host': node.host}
        },
        'tls': {
          'enabled': node.tls,
          'server_name': node.sni
        }
      };
      
    case 'trojan':
      return {
        'type': 'trojan',
        'tag': 'proxy',
        'server': node.address,
        'server_port': node.port,
        'password': node.password,
        'tls': {
          'enabled': true,
          'server_name': node.sni,
          'alpn': node.alpn
        }
      };
      
    default:
      throw UnsupportedError('Protocol ${node.protocol} not supported');
  }
}
```

## Common Patterns

### Auto-Update Subscriptions
```dart
class SubscriptionManager {
  Future<void> updateSubscription(Profile profile) async {
    if (profile.type != 'subscription') return;
    
    try {
      final response = await http.get(Uri.parse(profile.url));
      if (response.statusCode != 200) {
        throw Exception('Failed to fetch subscription');
      }
      
      // Parse base64 subscription
      final decoded = utf8.decode(base64.decode(response.body));
      final nodes = decoded
          .split('\n')
          .where((line) => line.isNotEmpty)
          .map((line) => _parseProxyURL(line))
          .whereType<ProxyNode>()
          .toList();
      
      profile.nodes = nodes;
      profile.lastUpdate = DateTime.now();
      
      await _saveProfile(profile);
    } catch (e) {
      print('Subscription update failed: $e');
    }
  }
  
  ProxyNode? _parseProxyURL(String url) {
    if (url.startsWith('vless://')) {
      return _parseVLESS(url);
    } else if (url.startsWith('vmess://')) {
      return _parseVMess(url);
    } else if (url.startsWith('trojan://')) {
      return _parseTrojan(url);
    }
    return null;
  }
}
```

### Connection State Management
```dart
class ProxyController extends ChangeNotifier {
  bool _isConnected = false;
  ProxyNode? _activeNode;
  int _uploadBytes = 0;
  int _downloadBytes = 0;
  
  Future<void> connect(ProxyNode node) async {
    final config = generateSingBoxConfig(node, currentRouting);
    
    await _writeSingBoxConfig(config);
    await _startSingBox();
    
    _isConnected = true;
    _activeNode = node;
    notifyListeners();
  }
  
  Future<void> disconnect() async {
    await _stopSingBox();
    
    _isConnected = false;
    _activeNode = null;
    notifyListeners();
  }
  
  void updateTraffic(int upload, int download) {
    _uploadBytes = upload;
    _downloadBytes = download;
    notifyListeners();
  }
}
```

## Troubleshooting

### Connection Fails
```dart
// Check node reachability
Future<bool> checkNodeHealth(ProxyNode node) async {
  final latency = await testNodeLatency(node);
  if (latency < 0) {
    print('Node ${node.name} is unreachable');
    return false;
  }
  
  if (latency > 3000) {
    print('Node ${node.name} has high latency: ${latency}ms');
  }
  
  return true;
}

// Validate configuration
bool validateNode(ProxyNode node) {
  if (node.address.isEmpty || node.port == 0) {
    print('Invalid address or port');
    return false;
  }
  
  if (node.protocol == 'vless' && node.uuid.isEmpty) {
    print('VLESS requires UUID');
    return false;
  }
  
  if (node.protocol == 'reality' && node.publicKey.isEmpty) {
    print('Reality requires public key');
    return false;
  }
  
  return true;
}
```

### Iranian Sites Not Bypassing
```dart
// Verify routing rules
final iranRouting = {
  'route': {
    'rules': [
      {
        'domain_suffix': ['.ir'],
        'outbound': 'direct'
      },
      {
        'geoip': 'ir',
        'outbound': 'direct'
      }
    ]
  }
};

// Download latest geoip database
await downloadGeoIP('https://github.com/Loyalsoldier/geoip/releases/latest/download/geoip.db');
```

### VPN Service Won't Start (Android)
```dart
// Check VPN permissions
import 'package:android_intent/android_intent.dart';

Future<bool> requestVPNPermission() async {
  if (Platform.isAndroid) {
    final intent = AndroidIntent(
      action: 'android.net.VpnService',
      package: 'com.irnova.client',
    );
    await intent.launch();
    return true;
  }
  return false;
}
```

### High Memory Usage
```dart
// Limit concurrent connections in scanner
final scanner = RadarScanner();
await scanner.scanCleanIPs(
  count: 10,
  threads: 50, // Reduce from 100 to 50
  timeout: 2000,
);

// Clear cache periodically
void clearCache() {
  _nodeCache.clear();
  _latencyCache.clear();
}
```

## Environment Variables

Nova Client respects these environment variables:

```bash
# Override config directory
export NOVA_CLIENT_CONFIG_DIR="$HOME/.config/nova-client-custom"

# Enable debug logging
export NOVA_CLIENT_DEBUG=1

# Set proxy for subscription fetching
export HTTPS_PROXY="http://127.0.0.1:7890"

# Disable auto-update
export NOVA_CLIENT_AUTO_UPDATE=0
```

## API Integration Example

```dart
import 'package:nova_client/nova_client.dart';

void main() async {
  // Initialize client
  final client = NovaClient();
  await client.initialize();
  
  // Add subscription
  final profile = await client.addSubscription(
    name: 'My Subscription',
    url: 'https://example.com/sub',
    autoUpdate: true,
  );
  
  // Scan clean IPs
  final cleanIPs = await client.radar.scan(
    count: 10,
    onProgress: (p) => print('Scanning: ${(p * 100).round()}%'),
  );
  
  // Connect to fastest node
  final nodes = profile.nodes;
  final latencies = await client.testLatency(nodes);
  final fastest = client.selectFastest(nodes, latencies);
  
  await client.connect(fastest);
  
  // Monitor traffic
  client.onTrafficUpdate.listen((traffic) {
    print('↑ ${traffic.upload} ↓ ${traffic.download}');
  });
}
```
