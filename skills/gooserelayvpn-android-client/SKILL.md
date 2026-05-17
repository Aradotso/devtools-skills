---
name: gooserelayvpn-android-client
description: Build and configure GooseRelayVPN Android client with Go mobile bridge, SOCKS5 VPN tunneling through Google Apps Script domain-fronting, and profile-based setup
triggers:
  - how do I build the GooseRelayVPN Android client
  - configure GooseRelayVPN profile with script keys and tunnel key
  - build Go mobile AAR for GooseRelayVPN Android
  - set up domain-fronted VPN with Google Apps Script
  - troubleshoot GooseRelayVPN Android connection issues
  - implement SOCKS5 VPN tunneling in Android with GooseRelay
  - create GooseRelayVPN profile JSON configuration
  - debug GooseRelayVPN Android VpnService
---

# GooseRelayVPN Android Client Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What GooseRelayVPN Android Client Does

GooseRelayVPN Android Client is an Android VPN application that tunnels TCP traffic through a domain-fronted SOCKS5 proxy using Google Apps Script as a relay. It encrypts traffic with AES-256-GCM and routes it through your VPS exit server while appearing to communicate with Google infrastructure.

**Architecture flow:**
1. Local Android app/browser → SOCKS5 proxy (127.0.0.1:1080)
2. GooseRelay core encrypts with AES-256-GCM
3. Traffic sent via HTTPS to Google Apps Script deployment
4. Apps Script relays to your VPS exit server
5. VPS decrypts and connects to target destination

The app uses Android `VpnService` + tun2socks to route device traffic through the tunnel.

## Prerequisites

Before building the Android client, you need:

1. **VPS server** running `goose-server` (from upstream GooseRelayVPN)
2. **Google Apps Script deployment** with the relay script deployed
3. **Shared tunnel key** generated via `scripts/gen-key.sh`
4. **Development environment:**
   - Android Studio
   - JDK 17+
   - Go 1.22+
   - Android SDK/NDK

## Installation & Build

### 1. Clone the Repository

```bash
git clone https://github.com/Hidden-Node/GooseRelayVPN-AndroidClient.git
cd GooseRelayVPN-AndroidClient
```

### 2. Build Go Mobile AAR

The Android app uses a Go mobile bridge to run the GooseRelay core:

```bash
cd android
bash build_go_mobile.sh
```

This script:
- Installs `gomobile` if needed
- Binds the Go core to Android AAR
- Places the AAR in `android/app/libs/`

**Manual build (if script fails):**

```bash
# Install gomobile
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init

# Build for Android
cd android
gomobile bind -target=android -o app/libs/gooserelay.aar ../core
```

### 3. Build Android APK

**Debug build:**

```bash
cd android
./gradlew :app:assembleDebug
```

**Release build (requires signing keys):**

```bash
./gradlew :app:assembleRelease
```

APK location: `android/app/build/outputs/apk/`

## Configuration

### Profile JSON Structure

GooseRelayVPN uses profile-based configuration. Each profile is a JSON object:

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
    "YOUR_APPS_SCRIPT_DEPLOYMENT_ID_1",
    "YOUR_APPS_SCRIPT_DEPLOYMENT_ID_2"
  ],
  "tunnel_key": "YOUR_TUNNEL_KEY_FROM_GEN_KEY_SCRIPT"
}
```

**Field descriptions:**

- `debug_timing`: Enable timing logs (boolean)
- `socks_host`: Local SOCKS5 bind address (usually `127.0.0.1`)
- `socks_port`: Local SOCKS5 port (default `1080`)
- `google_host`: Google IP to connect to (domain-fronting)
- `sni`: Array of SNI hostnames for TLS ClientHello
- `script_keys`: Array of Google Apps Script deployment IDs
- `tunnel_key`: AES-256-GCM encryption key (must match server)

### Generating Tunnel Key

On your development/VPS machine:

```bash
# From upstream GooseRelayVPN repository
cd GooseRelayVPN
bash scripts/gen-key.sh
```

Save the output key — it must be identical on both client and server.

### Obtaining Apps Script Deployment ID

1. Deploy `apps_script/Code.gs` to Google Apps Script
2. Deploy as Web App → get deployment ID (format: `AKfycby...`)
3. Add deployment ID(s) to `script_keys` array

### In-App Profile Management

**Create profile:**

```kotlin
// ProfileFragment.kt example
val profile = Profile(
    name = "My VPN",
    debugTiming = false,
    socksHost = "127.0.0.1",
    socksPort = 1080,
    googleHost = "216.239.38.120",
    sni = listOf("www.google.com", "mail.google.com"),
    scriptKeys = listOf(System.getenv("APPS_SCRIPT_DEPLOYMENT_ID")),
    tunnelKey = System.getenv("TUNNEL_KEY")
)
```

**Import from JSON:**

Use the in-app import button or load JSON programmatically:

```kotlin
import org.json.JSONObject
import org.json.JSONArray

fun importProfile(jsonString: String): Profile {
    val json = JSONObject(jsonString)
    return Profile(
        name = json.optString("name", "Imported Profile"),
        debugTiming = json.optBoolean("debug_timing", false),
        socksHost = json.optString("socks_host", "127.0.0.1"),
        socksPort = json.optInt("socks_port", 1080),
        googleHost = json.optString("google_host", "216.239.38.120"),
        sni = json.getJSONArray("sni").let { arr ->
            (0 until arr.length()).map { arr.getString(it) }
        },
        scriptKeys = json.getJSONArray("script_keys").let { arr ->
            (0 until arr.length()).map { arr.getString(it) }
        },
        tunnelKey = json.getString("tunnel_key")
    )
}
```

**Export to JSON:**

```kotlin
fun exportProfile(profile: Profile): String {
    val json = JSONObject().apply {
        put("debug_timing", profile.debugTiming)
        put("socks_host", profile.socksHost)
        put("socks_port", profile.socksPort)
        put("google_host", profile.googleHost)
        put("sni", JSONArray(profile.sni))
        put("script_keys", JSONArray(profile.scriptKeys))
        put("tunnel_key", profile.tunnelKey)
    }
    return json.toString(2)
}
```

## Core Go Mobile Bridge Usage

The Android app communicates with the Go core via the mobile bridge:

### Starting the VPN Tunnel

```go
// core/mobile.go example binding
package mobile

import (
    "context"
    "encoding/json"
)

type VPNClient struct {
    ctx    context.Context
    cancel context.CancelFunc
}

func NewVPNClient(configJSON string) (*VPNClient, error) {
    var config ClientConfig
    if err := json.Unmarshal([]byte(configJSON), &config); err != nil {
        return nil, err
    }
    
    ctx, cancel := context.WithCancel(context.Background())
    client := &VPNClient{
        ctx:    ctx,
        cancel: cancel,
    }
    
    // Start SOCKS5 server and relay logic
    go client.runRelay(config)
    
    return client, nil
}

func (c *VPNClient) Stop() {
    c.cancel()
}
```

**Android invocation:**

```kotlin
import gooserelay.Mobile

class VPNService : android.net.VpnService() {
    private var vpnClient: Mobile.VPNClient? = null
    
    fun startTunnel(profileJson: String) {
        try {
            vpnClient = Mobile.newVPNClient(profileJson)
            Log.i("VPN", "Tunnel started")
        } catch (e: Exception) {
            Log.e("VPN", "Failed to start: ${e.message}")
        }
    }
    
    fun stopTunnel() {
        vpnClient?.stop()
        vpnClient = null
    }
}
```

## Android VpnService Integration

### Establishing VPN Connection

```kotlin
import android.net.VpnService
import android.os.ParcelFileDescriptor

class GooseVpnService : VpnService() {
    private var tunInterface: ParcelFileDescriptor? = null
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val profile = intent?.getStringExtra("profile_json") ?: return START_NOT_STICKY
        
        // Request VPN permission if needed
        prepare(this)?.let { permissionIntent ->
            // User must grant permission
            return START_NOT_STICKY
        }
        
        // Configure VPN
        tunInterface = Builder()
            .setSession("GooseRelayVPN")
            .addAddress("10.0.0.2", 24)
            .addRoute("0.0.0.0", 0)
            .addDnsServer("8.8.8.8")
            .setMtu(1500)
            .establish()
        
        // Start Go mobile VPN client
        startGooseRelay(profile)
        
        return START_STICKY
    }
    
    private fun startGooseRelay(profileJson: String) {
        vpnClient = Mobile.newVPNClient(profileJson)
        
        // Route tun traffic to SOCKS5
        val tunFd = tunInterface?.detachFd() ?: return
        Mobile.routeTunToSocks(tunFd.toLong(), "127.0.0.1:1080")
    }
    
    override fun onDestroy() {
        vpnClient?.stop()
        tunInterface?.close()
        super.onDestroy()
    }
}
```

### Split Tunneling

```kotlin
class GooseVpnService : VpnService() {
    fun configureSplitTunnel(allowedApps: List<String>) {
        val builder = Builder()
            .setSession("GooseRelayVPN")
            .addAddress("10.0.0.2", 24)
            .addRoute("0.0.0.0", 0)
        
        // Allow specific apps to use VPN
        allowedApps.forEach { packageName ->
            try {
                builder.addAllowedApplication(packageName)
            } catch (e: PackageManager.NameNotFoundException) {
                Log.w("VPN", "App not found: $packageName")
            }
        }
        
        tunInterface = builder.establish()
    }
}
```

## Common Patterns

### Profile Validation

```kotlin
data class ProfileValidationResult(val isValid: Boolean, val errors: List<String>)

fun validateProfile(profile: Profile): ProfileValidationResult {
    val errors = mutableListOf<String>()
    
    if (profile.scriptKeys.isEmpty()) {
        errors.add("At least one script_key required")
    }
    
    if (profile.tunnelKey.isEmpty()) {
        errors.add("tunnel_key cannot be empty")
    }
    
    if (profile.socksPort !in 1024..65535) {
        errors.add("Invalid SOCKS port: ${profile.socksPort}")
    }
    
    if (profile.sni.isEmpty()) {
        errors.add("At least one SNI hostname required")
    }
    
    return ProfileValidationResult(errors.isEmpty(), errors)
}
```

### Logging & Telemetry

```kotlin
import android.util.Log

object VPNLogger {
    private const val TAG = "GooseVPN"
    
    fun logConnectionAttempt(profile: String) {
        Log.i(TAG, "Connecting to profile: $profile")
    }
    
    fun logTrafficStats(bytesIn: Long, bytesOut: Long) {
        Log.d(TAG, "Traffic: ↓$bytesIn ↑$bytesOut bytes")
    }
    
    fun logError(operation: String, error: String) {
        Log.e(TAG, "Error in $operation: $error")
    }
}

// In VPN service
vpnClient?.getStats()?.let { stats ->
    VPNLogger.logTrafficStats(stats.bytesReceived, stats.bytesSent)
}
```

### Handling Connection State

```kotlin
enum class VPNState {
    DISCONNECTED,
    PREPARING,
    CONNECTING,
    CONNECTED,
    DISCONNECTING,
    ERROR
}

class VPNStateManager {
    private val _state = MutableLiveData<VPNState>(VPNState.DISCONNECTED)
    val state: LiveData<VPNState> = _state
    
    fun setState(newState: VPNState) {
        _state.postValue(newState)
    }
    
    fun isConnected() = _state.value == VPNState.CONNECTED
}

// In Activity/Fragment
stateManager.state.observe(viewLifecycleOwner) { state ->
    when (state) {
        VPNState.CONNECTED -> showConnectedUI()
        VPNState.ERROR -> showErrorDialog()
        VPNState.CONNECTING -> showLoadingSpinner()
        else -> showDisconnectedUI()
    }
}
```

## Troubleshooting

### SOCKS Port Already in Use

**Symptom:** Connection fails with "address already in use" error

**Solution:**

```kotlin
fun killExistingSocksServer(port: Int) {
    try {
        // Stop existing VPN service
        stopService(Intent(this, GooseVpnService::class.java))
        
        // Wait for port to be released
        Thread.sleep(2000)
        
        // Verify port is free
        Socket().use { socket ->
            socket.connect(InetSocketAddress("127.0.0.1", port), 1000)
            // If no exception, port still in use
            throw IOException("Port $port still in use")
        }
    } catch (e: IOException) {
        // Port is free (connection refused)
        Log.i("VPN", "Port $port is available")
    }
}
```

### Invalid Tunnel Key or Script Keys

**Symptom:** Connection stays in "preparing" state, no traffic flows

**Debugging:**

```kotlin
fun debugConnectionIssues(profile: Profile) {
    // 1. Verify key format
    if (profile.tunnelKey.length != 64) {
        Log.e("VPN", "Invalid tunnel_key length: ${profile.tunnelKey.length}")
    }
    
    // 2. Test script_key reachability
    profile.scriptKeys.forEach { scriptKey ->
        testScriptKey(scriptKey)
    }
    
    // 3. Check core logs
    val coreLogs = Mobile.getLastLogs(100)
    Log.d("VPN", "Core logs: $coreLogs")
}

fun testScriptKey(scriptKey: String) {
    val url = "https://script.google.com/macros/s/$scriptKey/exec"
    // Attempt connection (requires network permission)
    // Log result
}
```

### VPN Permission Not Granted

**Symptom:** VPN doesn't start, no error shown

**Solution:**

```kotlin
class MainActivity : AppCompatActivity() {
    private val VPN_PERMISSION_REQUEST = 100
    
    fun requestVPNPermission() {
        val intent = VpnService.prepare(this)
        if (intent != null) {
            startActivityForResult(intent, VPN_PERMISSION_REQUEST)
        } else {
            // Permission already granted
            startVPN()
        }
    }
    
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == VPN_PERMISSION_REQUEST) {
            if (resultCode == RESULT_OK) {
                startVPN()
            } else {
                Toast.makeText(this, "VPN permission denied", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

### No Traffic Flowing Through VPN

**Symptom:** VPN connected but apps don't route through tunnel

**Checklist:**

```kotlin
fun diagnoseTunnel() {
    // 1. Verify VPN interface is up
    val tunFd = tunInterface?.fileDescriptor
    if (tunFd == null || !tunFd.valid()) {
        Log.e("VPN", "TUN interface not valid")
        return
    }
    
    // 2. Check routing table
    Log.d("VPN", "Default route should be via 10.0.0.2")
    
    // 3. Verify SOCKS5 is listening
    try {
        Socket("127.0.0.1", 1080).use {
            Log.i("VPN", "SOCKS5 server is reachable")
        }
    } catch (e: IOException) {
        Log.e("VPN", "SOCKS5 server not listening: ${e.message}")
    }
    
    // 4. Check DNS configuration
    Log.d("VPN", "DNS servers configured in VPN builder")
}
```

## CI/CD & Release

### GitHub Actions Workflow

The project includes automated builds:

```yaml
# .github/workflows/android-ci.yml
name: Android CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
      - uses: actions/setup-go@v4
        with:
          go-version: '1.22'
      - name: Build Go mobile
        run: bash android/build_go_mobile.sh
      - name: Build APK
        run: cd android && ./gradlew assembleDebug
```

### Signing Release APK

**Configure signing in `android/app/build.gradle`:**

```gradle
android {
    signingConfigs {
        release {
            storeFile file(System.getenv("KEYSTORE_PATH") ?: "release.keystore")
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias System.getenv("KEY_ALIAS")
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

**Build signed APK:**

```bash
export KEYSTORE_PATH=/path/to/keystore
export KEYSTORE_PASSWORD=your_keystore_password
export KEY_ALIAS=your_key_alias
export KEY_PASSWORD=your_key_password

cd android
./gradlew assembleRelease
```

## Advanced Usage

### Custom DNS Configuration

```kotlin
fun setupCustomDNS(dnsServers: List<String>) {
    val builder = Builder()
        .setSession("GooseRelayVPN")
        .addAddress("10.0.0.2", 24)
        .addRoute("0.0.0.0", 0)
    
    dnsServers.forEach { dns ->
        builder.addDnsServer(dns)
    }
    
    tunInterface = builder.establish()
}

// Usage
setupCustomDNS(listOf("1.1.1.1", "8.8.8.8"))
```

### Internet Sharing / Hotspot Mode

```kotlin
fun enableInternetSharing() {
    val builder = Builder()
        .setSession("GooseRelayVPN")
        .addAddress("10.0.0.1", 24)  // Gateway address
        .addRoute("0.0.0.0", 0)
        .setMtu(1500)
    
    // Allow all apps except VPN service itself
    builder.addDisallowedApplication(packageName)
    
    tunInterface = builder.establish()
}
```

This skill provides comprehensive coverage of building, configuring, and troubleshooting the GooseRelayVPN Android client with Go mobile bridge integration.
