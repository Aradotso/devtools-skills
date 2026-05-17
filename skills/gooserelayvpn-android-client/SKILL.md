---
name: gooserelayvpn-android-client
description: Build, configure, and troubleshoot the GooseRelayVPN Android client—a SOCKS5 VPN app with domain-fronting via Google Apps Script.
triggers:
  - "set up GooseRelayVPN Android client"
  - "configure GooseRelay Android profile"
  - "build GooseRelayVPN APK"
  - "troubleshoot GooseRelay Android connection"
  - "integrate GooseRelay VpnService"
  - "import GooseRelay profile JSON"
  - "debug GooseRelay Android logs"
  - "create GooseRelayVPN mobile bridge"
---

# GooseRelayVPN Android Client

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

GooseRelayVPN Android Client is a VPN app that tunnels TCP traffic through Google Apps Script to a VPS exit server using AES-256-GCM encryption and domain fronting. The app:

- Creates a local SOCKS5 proxy on Android
- Uses `VpnService` + tun2socks for system-wide or per-app tunneling
- Encrypts traffic with AES-256-GCM before relaying through Google endpoints
- Supports profile-based configuration with JSON import/export
- Provides real-time logs and connection telemetry

**Architecture flow:**
1. Local app traffic → SOCKS5 (127.0.0.1:1080)
2. GooseRelay core encrypts with AES key
3. HTTPS to Google Apps Script deployment IDs
4. Apps Script forwards to VPS exit server
5. VPS makes outbound connections

## Prerequisites

Before building or using the Android client, you need upstream infrastructure:

1. **VPS running goose-server** (from upstream repo)
2. **Google Apps Script deployed** (`apps_script/Code.gs` with deployment ID)
3. **AES-256 tunnel key** generated via `scripts/gen-key.sh`

Upstream repo: https://github.com/kianmhz/GooseRelayVPN

## Installation & Build

### Requirements

- Android Studio (latest stable)
- JDK 17+
- Go 1.22+
- Android SDK 34+ and NDK

### Clone and Build Go Mobile Bridge

```bash
git clone https://github.com/Hidden-Node/GooseRelayVPN-AndroidClient.git
cd GooseRelayVPN-AndroidClient

# Build the Go mobile AAR (gomobile bind wrapper)
bash android/build_go_mobile.sh
```

This generates `gooserelay.aar` in `android/app/libs/`.

### Build Debug APK

```bash
cd android
./gradlew :app:assembleDebug
```

Output: `android/app/build/outputs/apk/debug/app-debug.apk`

### Build Release APK (Signed)

Requires keystore secrets:

```bash
export ANDROID_KEYSTORE_BASE64="..."
export ANDROID_KEYSTORE_PASSWORD="..."
export ANDROID_KEY_ALIAS="..."
export ANDROID_KEY_PASSWORD="..."

./gradlew :app:assembleRelease
```

Or use CI workflows: `.github/workflows/release.yml`

## Configuration (Profile JSON)

The app uses profile-based configuration matching the upstream GooseRelay client config schema.

### Profile Structure

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
    "AKfycbzXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "AKfycbzYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
  ],
  "tunnel_key": "BASE64_ENCODED_AES_KEY_FROM_GEN_KEY_SH"
}
```

### Field Reference

| Field          | Type     | Description                                                      |
|----------------|----------|------------------------------------------------------------------|
| `debug_timing` | bool     | Enable timing debug logs in core                                 |
| `socks_host`   | string   | Local SOCKS5 bind address (usually `127.0.0.1`)                  |
| `socks_port`   | int      | SOCKS5 port (default `1080`)                                     |
| `google_host`  | string   | Google IP for domain fronting (e.g., `216.239.38.120`)           |
| `sni`          | []string | SNI hostnames for TLS handshake (Google domains)                 |
| `script_keys`  | []string | Google Apps Script deployment IDs (one or more for redundancy)   |
| `tunnel_key`   | string   | Base64 AES-256 key; must match server-side key                   |

### Generating the Tunnel Key

From the upstream repo:

```bash
cd GooseRelayVPN
bash scripts/gen-key.sh
```

Output example:
```
Generated AES-256 key (base64):
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```

Copy this key into your profile's `tunnel_key` field **and** your VPS server config.

### In-App Profile Management

- **Create new profile:** Enter profile name and fields manually
- **Import JSON:** Tap "Import Profile" → select JSON file or paste JSON
- **Export JSON:** Tap profile → "Export" → save to device
- **Edit profile:** Tap profile → modify fields → Save

### Example Import JSON

```json
{
  "debug_timing": false,
  "socks_host": "127.0.0.1",
  "socks_port": 1080,
  "google_host": "216.239.38.120",
  "sni": ["www.google.com"],
  "script_keys": [
    "AKfycbzABCDEFGHIJKLMNOPQRSTUVWXYZ"
  ],
  "tunnel_key": "Zm9vYmFyYmF6cXV4Zm9vYmFyYmF6cXV4Zm9vYmFyYmF6cXV4"
}
```

## Using the App

### Connect to VPN

1. Open app
2. Select or create a profile
3. Tap "Connect" button on Home tab
4. Grant VPN permission when prompted (first time only)
5. Status should change: **Preparing → Connecting → Connected**

### Disconnect

Tap "Disconnect" button. VpnService will stop and SOCKS proxy will shut down.

### View Logs

- Tap **Logs** tab
- View real-time Android and Go core logs
- Use "Clear Logs" to reset buffer
- Use "Export Logs" to save diagnostics

### Split Tunneling (Per-App VPN)

- Tap **Settings** → "Split Tunnel Apps"
- Select which apps should route through VPN
- Unselected apps use normal network

### Internet Sharing / Hotspot

- Enable "Allow Internet Sharing" in Settings
- Traffic from tethered devices will also route through VPN

## Development Patterns

### Go Mobile Bridge

The Android app uses `gomobile bind` to wrap the GooseRelay core (written in Go) as an AAR library.

**Build script:** `android/build_go_mobile.sh`

```bash
#!/bin/bash
set -e

# Install gomobile if needed
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init

# Build AAR for Android
gomobile bind -target=android -androidapi=21 \
  -o android/app/libs/gooserelay.aar \
  github.com/kianmhz/GooseRelayVPN/client
```

### Calling Go Code from Kotlin

**Example:** Start SOCKS proxy in background

```kotlin
import gooserelay.Gooserelay

class VpnService : VpnService() {
    private var goProxy: Gooserelay.Proxy? = null

    fun startProxy(configJson: String) {
        goProxy = Gooserelay.newProxy(configJson)
        thread {
            try {
                goProxy?.start()
            } catch (e: Exception) {
                Log.e("VPN", "Proxy error: ${e.message}")
            }
        }
    }

    fun stopProxy() {
        goProxy?.stop()
        goProxy = null
    }
}
```

### Profile Data Class (Kotlin)

```kotlin
data class Profile(
    val name: String,
    val debug_timing: Boolean = false,
    val socks_host: String = "127.0.0.1",
    val socks_port: Int = 1080,
    val google_host: String = "216.239.38.120",
    val sni: List<String> = listOf("www.google.com"),
    val script_keys: List<String>,
    val tunnel_key: String
)

// Serialize to JSON
fun Profile.toJson(): String {
    return Gson().toJson(this)
}

// Deserialize from JSON
fun profileFromJson(json: String): Profile {
    return Gson().fromJson(json, Profile::class.java)
}
```

### Importing Profile from JSON File

```kotlin
fun importProfile(uri: Uri, context: Context): Profile {
    val json = context.contentResolver.openInputStream(uri)?.use {
        it.bufferedReader().readText()
    } ?: throw IOException("Cannot read file")
    return profileFromJson(json)
}
```

### Exporting Profile to JSON File

```kotlin
fun exportProfile(profile: Profile, uri: Uri, context: Context) {
    context.contentResolver.openOutputStream(uri)?.use {
        it.write(profile.toJson().toByteArray())
    }
}
```

### VpnService Integration

**Basic VpnService setup:**

```kotlin
class GooseVpnService : VpnService() {
    private var vpnInterface: ParcelFileDescriptor? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val profile = getActiveProfile() // Load from SharedPreferences or DB
        
        // Establish VPN interface
        val builder = Builder()
            .setSession("GooseRelayVPN")
            .addAddress("10.0.0.2", 24)
            .addRoute("0.0.0.0", 0)
            .addDnsServer("8.8.8.8")
            .setMtu(1500)
        
        // Per-app routing
        if (splitTunnelEnabled) {
            selectedApps.forEach { builder.addAllowedApplication(it) }
        }
        
        vpnInterface = builder.establish()
        
        // Start tun2socks and Go proxy
        startTun2Socks(vpnInterface!!)
        startGoProxy(profile.toJson())
        
        return START_STICKY
    }

    override fun onDestroy() {
        stopGoProxy()
        stopTun2Socks()
        vpnInterface?.close()
        super.onDestroy()
    }
}
```

### Tun2Socks Integration

```kotlin
private fun startTun2Socks(vpnFd: ParcelFileDescriptor) {
    val cmd = listOf(
        "${applicationInfo.nativeLibraryDir}/libtun2socks.so",
        "--netif-ipaddr", "10.0.0.2",
        "--netif-netmask", "255.255.255.0",
        "--socks-server-addr", "127.0.0.1:1080",
        "--tunfd", vpnFd.fd.toString(),
        "--loglevel", "info"
    )
    
    ProcessBuilder(cmd).start()
}
```

## Troubleshooting

### Connection Stuck in "Preparing" State

**Cause:** Core cannot reach Apps Script or key mismatch.

**Fix:**
- Verify `script_keys` are correct deployment IDs
- Verify `tunnel_key` matches server-side key
- Check Logs tab for error messages like "handshake failed" or "401 Unauthorized"

### SOCKS Port Already in Use

**Cause:** Previous session didn't clean up properly.

**Fix:**
```bash
# On device (via adb shell)
netstat -an | grep 1080
# Kill any stale processes holding the port
```

In code, ensure `stopProxy()` is always called on disconnect.

### No Traffic Flows After Connecting

**Cause:** VPN permission not granted, or routing misconfigured.

**Fix:**
- Verify VPN permission dialog was accepted
- Check split tunnel settings — if enabled, ensure target apps are selected
- Restart app and reconnect

### Apps Script Returns 403 or 404

**Cause:** Deployment ID incorrect or script not deployed as web app.

**Fix:**
- Redeploy `apps_script/Code.gs` as web app (Execute as: Me, Access: Anyone)
- Copy the new deployment ID into `script_keys`

### AES Decryption Errors on Server

**Cause:** `tunnel_key` mismatch between client and server.

**Fix:**
- Regenerate key with `gen-key.sh`
- Update both client profile JSON and server config with the new key
- Restart both client and server

### VpnService Crashes on Establish

**Cause:** Missing VPN permission or conflicting VPN app.

**Fix:**
- Check logcat: `adb logcat | grep VpnService`
- Ensure no other VPN is active
- Request VPN permission explicitly:
  ```kotlin
  val intent = VpnService.prepare(context)
  if (intent != null) {
      startActivityForResult(intent, VPN_REQUEST_CODE)
  }
  ```

## Testing & CI

### Local Testing

```bash
# Install on device
adb install android/app/build/outputs/apk/debug/app-debug.apk

# View logs
adb logcat | grep GooseRelay
```

### CI Workflows

- **android-ci.yml:** Builds debug APK on every push
- **release.yml:** Builds signed release APK on tag push (requires secrets)
- **release-manual.yml:** Manually triggered signed build

**Required GitHub Secrets for release:**
- `ANDROID_KEYSTORE_BASE64`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

## Common Patterns

### Auto-Reconnect on Network Change

```kotlin
class NetworkChangeReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (isVpnActive && isNetworkAvailable(context)) {
            restartVpnService(context)
        }
    }
}
```

### Persistent Notification

```kotlin
private fun showVpnNotification() {
    val notification = NotificationCompat.Builder(this, CHANNEL_ID)
        .setContentTitle("GooseRelayVPN")
        .setContentText("Connected")
        .setSmallIcon(R.drawable.ic_vpn)
        .setPriority(NotificationCompat.PRIORITY_LOW)
        .build()
    startForeground(NOTIFICATION_ID, notification)
}
```

### Exporting Logs to File

```kotlin
fun exportLogs(logs: List<String>, uri: Uri, context: Context) {
    context.contentResolver.openOutputStream(uri)?.use { out ->
        logs.forEach { out.write("$it\n".toByteArray()) }
    }
}
```

## Advanced Configuration

### Multiple Deployment IDs for Redundancy

```json
{
  "script_keys": [
    "AKfycbzPRIMARY_DEPLOYMENT_ID",
    "AKfycbzFALLBACK_DEPLOYMENT_ID"
  ]
}
```

The core will try each deployment ID in order until one succeeds.

### Custom Google IP for Domain Fronting

```json
{
  "google_host": "142.250.80.46"
}
```

Use any Google IP that accepts HTTPS on port 443. Test with:

```bash
curl -v --resolve www.google.com:443:142.250.80.46 https://www.google.com/
```

### Debug Timing Logs

```json
{
  "debug_timing": true
}
```

Enables detailed timing logs in Go core for performance analysis.

## Resources

- **Main project:** https://github.com/kianmhz/GooseRelayVPN
- **Upstream docs:** See main repo for server setup, Apps Script deployment
- **Protocol details:** AES-256-GCM framing, HTTP chunked transfer encoding
- **Inspired by:** https://github.com/masterking32/MasterHttpRelayVPN

## License

MIT License — See LICENSE file in repository.
