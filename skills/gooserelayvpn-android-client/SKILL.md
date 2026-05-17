---
name: gooserelayvpn-android-client
description: Android VPN client that tunnels SOCKS5 traffic through Google Apps Script to a VPS exit server with AES-256-GCM encryption and domain fronting
triggers:
  - how do I build the GooseRelayVPN Android client
  - configure GooseRelay Android VPN profile
  - setup Android VPN with Google Apps Script tunnel
  - GooseRelayVPN Android profile JSON format
  - build GooseRelay mobile AAR library
  - troubleshoot GooseRelay Android connection
  - import GooseRelay VPN profile on Android
  - setup domain-fronted VPN on Android
---

# GooseRelayVPN Android Client Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

GooseRelayVPN Android Client is an Android application that provides a VPN service tunneling TCP traffic through Google Apps Script to a VPS exit server. It uses:

- **Local SOCKS5 proxy** for app/browser traffic
- **AES-256-GCM encryption** for tunnel security
- **Domain fronting** via Google infrastructure (Apps Script)
- **Android VpnService** integration with tun2socks
- **Profile-based configuration** with JSON import/export

**Architecture flow:**
1. Android app traffic → SOCKS5 (127.0.0.1:1080)
2. GooseRelay core encrypts with AES key
3. HTTPS transport through Google Apps Script deployment
4. VPS exit server decrypts and forwards to target

## Prerequisites

Before using the Android client, you must set up upstream infrastructure:

1. **VPS server** running `goose-server` (from main GooseRelayVPN project)
2. **Google Apps Script** deployment with `apps_script/Code.gs`
3. **Tunnel encryption key** generated via `scripts/gen-key.sh`
4. **Deployment ID(s)** from Apps Script

Upstream project: https://github.com/kianmhz/GooseRelayVPN

## Building the Android Client

### Requirements

- Android Studio
- JDK 17
- Go 1.22+
- Android SDK with NDK
- `gomobile` tool

### Build Go Mobile AAR

The core GooseRelay logic is compiled to an Android AAR library:

```bash
# From project root
bash android/build_go_mobile.sh
```

This script:
- Installs `gomobile` if needed
- Compiles Go code to AAR for arm64-v8a, armeabi-v7a, x86, x86_64
- Outputs to `android/app/libs/gooserelay.aar`

**Manual AAR build:**

```bash
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init

gomobile bind \
  -target=android \
  -androidapi=21 \
  -o android/app/libs/gooserelay.aar \
  -v \
  ./mobile
```

### Build Debug APK

```bash
cd android
./gradlew :app:assembleDebug
# Output: android/app/build/outputs/apk/debug/app-debug.apk
```

### Build Release APK

```bash
cd android
./gradlew :app:assembleRelease
# Requires signing configuration in android/app/build.gradle
```

## Profile Configuration

### Profile JSON Structure

```json
{
  "debug_timing": false,
  "socks_host": "127.0.0.1",
  "socks_port": 1080,
  "google_host": "216.239.38.120",
  "sni": [
    "www.google.com",
    "mail.google.com",
    "accounts.google.com"
  ],
  "script_keys": [
    "DEPLOYMENT_ID_FROM_APPS_SCRIPT",
    "OPTIONAL_SECOND_DEPLOYMENT_ID"
  ],
  "tunnel_key": "BASE64_ENCODED_AES_KEY_FROM_GEN_KEY_SCRIPT"
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `debug_timing` | bool | Enable timing debug logs |
| `socks_host` | string | Local SOCKS5 bind address (usually 127.0.0.1) |
| `socks_port` | int | Local SOCKS5 port (default 1080) |
| `google_host` | string | Google IP for domain fronting (216.239.38.120 is common) |
| `sni` | []string | SNI hostnames for TLS handshake rotation |
| `script_keys` | []string | Apps Script deployment IDs (one or more for redundancy) |
| `tunnel_key` | string | Base64 AES-256 key (must match server-side key) |

### Generating Tunnel Key

Use the upstream script to generate a secure key:

```bash
# From GooseRelayVPN main repo
bash scripts/gen-key.sh
# Outputs base64-encoded key
# Example: a3d7f9e2b1c4...
```

**Important:** The same `tunnel_key` must be used on both the Android client and the VPS `goose-server`.

### In-App Profile Management

**Create new profile:**
1. Open app → Profiles tab
2. Tap "+" button
3. Enter profile name and configuration
4. Save

**Import profile from JSON:**
1. Profiles tab → menu → Import
2. Select JSON file from storage
3. Profile is added to list

**Export profile to JSON:**
1. Profiles tab → long-press profile
2. Select Export
3. JSON saved to Downloads

## Android VPN Service Integration

### VpnService Implementation

The app uses Android's `VpnService` API to capture device traffic:

```kotlin
// Simplified example from Android codebase
class GooseVpnService : VpnService() {
    private val socksHost = "127.0.0.1"
    private val socksPort = 1080
    
    fun startVpn(profile: Profile) {
        // 1. Start GooseRelay core (via JNI to Go AAR)
        GooseRelay.start(profile.toJson())
        
        // 2. Configure VPN interface
        val builder = Builder()
            .setSession("GooseRelayVPN")
            .addAddress("10.0.0.2", 24)
            .addRoute("0.0.0.0", 0)
            .addDnsServer("8.8.8.8")
        
        // 3. Exclude apps if split tunneling enabled
        if (profile.splitTunnel) {
            profile.allowedApps.forEach { pkg ->
                builder.addAllowedApplication(pkg)
            }
        }
        
        val vpnInterface = builder.establish()
        
        // 4. Start tun2socks to forward traffic to SOCKS5
        Tun2Socks.start(
            vpnInterface.fd,
            socksHost,
            socksPort
        )
    }
}
```

### Permissions Required

**AndroidManifest.xml:**

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    
    <application>
        <service
            android:name=".service.GooseVpnService"
            android:permission="android.permission.BIND_VPN_SERVICE">
            <intent-filter>
                <action android:name="android.net.VpnService" />
            </intent-filter>
        </service>
    </application>
</manifest>
```

## Go Mobile Bridge

The Go core is exposed to Android via `gomobile`:

### Mobile Package Interface

**mobile/mobile.go:**

```go
package mobile

import (
    "encoding/json"
    "github.com/Hidden-Node/GooseRelayVPN/client"
)

// Config matches Android profile JSON structure
type Config struct {
    DebugTiming bool     `json:"debug_timing"`
    SocksHost   string   `json:"socks_host"`
    SocksPort   int      `json:"socks_port"`
    GoogleHost  string   `json:"google_host"`
    SNI         []string `json:"sni"`
    ScriptKeys  []string `json:"script_keys"`
    TunnelKey   string   `json:"tunnel_key"`
}

var relayClient *client.Client

// Start initializes and starts the GooseRelay client
func Start(configJSON string) error {
    var cfg Config
    if err := json.Unmarshal([]byte(configJSON), &cfg); err != nil {
        return err
    }
    
    relayClient = client.NewClient(client.Config{
        DebugTiming: cfg.DebugTiming,
        SocksAddr:   fmt.Sprintf("%s:%d", cfg.SocksHost, cfg.SocksPort),
        GoogleHost:  cfg.GoogleHost,
        SNI:         cfg.SNI,
        ScriptKeys:  cfg.ScriptKeys,
        TunnelKey:   cfg.TunnelKey,
    })
    
    return relayClient.Start()
}

// Stop gracefully stops the relay client
func Stop() error {
    if relayClient != nil {
        return relayClient.Stop()
    }
    return nil
}

// GetStats returns JSON statistics
func GetStats() string {
    if relayClient == nil {
        return "{}"
    }
    stats := relayClient.GetStats()
    data, _ := json.Marshal(stats)
    return string(data)
}
```

### Calling from Android (Kotlin)

```kotlin
import gooserelay.Gooserelay // Generated from AAR

class RelayManager {
    fun startRelay(profile: Profile) {
        val configJson = profile.toJson()
        try {
            Gooserelay.start(configJson)
            Log.i("GooseRelay", "Started successfully")
        } catch (e: Exception) {
            Log.e("GooseRelay", "Start failed: ${e.message}")
        }
    }
    
    fun stopRelay() {
        try {
            Gooserelay.stop()
        } catch (e: Exception) {
            Log.e("GooseRelay", "Stop failed: ${e.message}")
        }
    }
    
    fun getStats(): Stats? {
        return try {
            val json = Gooserelay.getStats()
            Json.decodeFromString<Stats>(json)
        } catch (e: Exception) {
            null
        }
    }
}
```

## Common Usage Patterns

### Full Device VPN

Route all device traffic through the tunnel:

```kotlin
// In profile configuration
val profile = Profile(
    name = "Full VPN",
    socksPort = 1080,
    scriptKeys = listOf(System.getenv("APPS_SCRIPT_DEPLOYMENT_ID")),
    tunnelKey = System.getenv("GOOSE_TUNNEL_KEY"),
    splitTunnel = false, // All apps
    excludeLocalNetwork = true
)

vpnService.startVpn(profile)
```

### Split Tunneling

Route only specific apps through the tunnel:

```kotlin
val profile = Profile(
    name = "Split Tunnel",
    socksPort = 1080,
    scriptKeys = listOf(System.getenv("APPS_SCRIPT_DEPLOYMENT_ID")),
    tunnelKey = System.getenv("GOOSE_TUNNEL_KEY"),
    splitTunnel = true,
    allowedApps = listOf(
        "com.android.chrome",
        "org.telegram.messenger"
    )
)

vpnService.startVpn(profile)
```

### Manual SOCKS5 Proxy

Use without VpnService (manual app configuration):

```kotlin
// Start only the SOCKS5 server, no VPN
fun startSocksOnly(profile: Profile) {
    GooseRelay.start(profile.toJson())
    // Apps must be configured to use 127.0.0.1:1080 as SOCKS5 proxy
}
```

## Logging and Telemetry

### Real-time Logs

The app provides a Logs tab showing Android and Go core logs:

```kotlin
// Android side logger forwarding to UI
object LogCollector {
    private val logs = mutableListOf<LogEntry>()
    
    fun addLog(level: String, tag: String, message: String) {
        logs.add(LogEntry(
            timestamp = System.currentTimeMillis(),
            level = level,
            tag = tag,
            message = message
        ))
        // Notify UI observers
        notifyLogListeners()
    }
}

// Go side: logs are captured via custom writer
```

### Telemetry Stats

Real-time connection statistics:

```kotlin
data class TelemetryStats(
    val bytesUploaded: Long,
    val bytesDownloaded: Long,
    val activeConnections: Int,
    val successRate: Float,
    val avgLatency: Long
)

fun updateTelemetry() {
    val statsJson = Gooserelay.getStats()
    val stats = Json.decodeFromString<TelemetryStats>(statsJson)
    
    // Update UI cards
    uploadCard.text = formatBytes(stats.bytesUploaded)
    downloadCard.text = formatBytes(stats.bytesDownloaded)
    latencyCard.text = "${stats.avgLatency}ms"
}
```

## CI/CD and Release

### GitHub Actions Workflows

**Debug CI (.github/workflows/android-ci.yml):**

```yaml
name: Android CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.22'
      
      - name: Build Go Mobile AAR
        run: bash android/build_go_mobile.sh
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Build Debug APK
        run: |
          cd android
          ./gradlew assembleDebug
      
      - name: Upload APK
        uses: actions/upload-artifact@v3
        with:
          name: app-debug
          path: android/app/build/outputs/apk/debug/app-debug.apk
```

**Release Workflow (.github/workflows/release.yml):**

```yaml
name: Release Build

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build AAR
        run: bash android/build_go_mobile.sh
      
      - name: Decode Keystore
        run: |
          echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 -d > android/keystore.jks
      
      - name: Build Release APK
        env:
          KEYSTORE_PASSWORD: ${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.ANDROID_KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.ANDROID_KEY_PASSWORD }}
        run: |
          cd android
          ./gradlew assembleRelease
      
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: android/app/build/outputs/apk/release/app-release.apk
```

**Required secrets:**
- `ANDROID_KEYSTORE_BASE64`: Base64-encoded keystore file
- `ANDROID_KEYSTORE_PASSWORD`: Keystore password
- `ANDROID_KEY_ALIAS`: Key alias in keystore
- `ANDROID_KEY_PASSWORD`: Key password

## Troubleshooting

### Connection Stuck in "Preparing" State

**Symptoms:** VPN shows "Preparing" but never connects.

**Causes:**
1. Invalid `script_keys` (deployment IDs)
2. Mismatched `tunnel_key` between client and server
3. Apps Script deployment not accessible

**Solution:**
```bash
# Check logs tab in app for specific errors
# Common log patterns:
# "Failed to connect to script" → Check deployment ID
# "Decryption failed" → Check tunnel_key matches server
# "Connection timeout" → Verify VPS server is running

# Verify server-side key matches:
# On VPS: cat /etc/goose-server/config.json | jq .tunnel_key
# In Android profile: check tunnel_key field
```

### SOCKS Port Busy Error

**Symptoms:** "bind: address already in use" in logs.

**Solution:**
```kotlin
// Ensure clean disconnect before reconnect
fun reconnect() {
    // Stop VPN service
    vpnService.stop()
    
    // Wait for port release
    Thread.sleep(2000)
    
    // Check no other app uses port 1080
    // Change profile socks_port if needed
    profile.socksPort = 1081
    
    vpnService.start()
}
```

### No Traffic Flowing Through VPN

**Symptoms:** VPN connected but no internet access.

**Checklist:**
1. VPN permission granted
2. Split tunnel app selection correct
3. DNS servers configured
4. Local network excluded if needed

**Solution:**
```kotlin
// Verify VPN builder configuration
val builder = Builder()
    .addAddress("10.0.0.2", 24)
    .addRoute("0.0.0.0", 0)
    .addDnsServer("8.8.8.8")
    .addDnsServer("1.1.1.1")

// Exclude local network
if (profile.excludeLocalNetwork) {
    builder.addRoute("0.0.0.0", 5)
    builder.addRoute("8.0.0.0", 7)
    builder.addRoute("11.0.0.0", 8)
    // ... exclude 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
}

val vpnInterface = builder.establish()
```

### Apps Script Deployment Issues

**Symptoms:** "Script execution failed" in logs.

**Solution:**
```bash
# Verify deployment ID format (should be long alphanumeric string)
# Example: AKfycbzXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Test deployment manually:
curl -L "https://script.google.com/macros/s/DEPLOYMENT_ID/exec?action=ping"

# Ensure Apps Script has correct VPS endpoint in Code.gs:
# const VPS_ENDPOINT = "https://your-vps-ip:8443";
```

### High Latency or Slow Speeds

**Symptoms:** Connected but slow performance.

**Optimization:**
```json
{
  "debug_timing": false,
  "sni": [
    "www.google.com"
  ],
  "script_keys": [
    "PRIMARY_DEPLOYMENT_ID",
    "BACKUP_DEPLOYMENT_ID"
  ]
}
```

**Tips:**
- Use multiple `script_keys` for load balancing
- Choose closer Google IP in `google_host`
- Optimize VPS server configuration
- Enable `debug_timing` temporarily to identify bottlenecks

### AAR Build Failures

**Symptoms:** `build_go_mobile.sh` fails.

**Solution:**
```bash
# Ensure gomobile is installed
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init

# Check Android NDK is installed
# In Android Studio: Tools → SDK Manager → SDK Tools → NDK

# Set environment variables
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/25.1.8937393

# Retry build
bash android/build_go_mobile.sh
```

### Certificate/TLS Errors

**Symptoms:** "x509: certificate signed by unknown authority"

**Solution:**
```go
// If using self-signed cert on VPS, Apps Script must trust it
// Better: Use Let's Encrypt on VPS

// In Code.gs, configure:
const ALLOW_SELF_SIGNED = false; // Set true only for testing
```

## Best Practices

1. **Key Management:** Store `tunnel_key` in Android Keystore, not plaintext
2. **Multiple Deployments:** Use 2-3 `script_keys` for redundancy
3. **Battery Optimization:** Exclude VPN service from battery optimization
4. **Testing:** Test with `debug_timing: true` during setup
5. **Backup Profiles:** Export profiles before app updates
6. **Server Monitoring:** Monitor VPS `goose-server` logs alongside Android logs

## Additional Resources

- Main project: https://github.com/kianmhz/GooseRelayVPN
- Apps Script setup: See upstream `apps_script/Code.gs`
- Server setup: See upstream `server/` directory
- Issues: Check open issues at https://github.com/Hidden-Node/GooseRelayVPN-AndroidClient/issues
