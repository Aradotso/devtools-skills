---
name: folddevtools-android-webview-debugger
description: Debug Android WebView, browsers, and Node.js using Chrome DevTools through FoldDevtools with root or remote debugging
triggers:
  - how do I debug an Android WebView with Chrome DevTools
  - debug webview on android device remotely
  - connect to android webview debugging socket
  - use FoldDevtools to inspect webview
  - forward android webview debug port with adb
  - enable webview debugging without root on android
  - debug stetho socket on android with devtools
  - connect chrome devtools to android app webview
---

# FoldDevtools Android WebView Debugger

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

FoldDevtools is an Android application that enables Chrome DevTools debugging for WebViews, browsers, Node.js, and other debug targets on Android devices. It supports both rooted and rootless devices, provides XPosed module functionality for force-enabling WebView debugging, and includes support for Stetho/StethoX debugging sockets.

## Installation

1. Download the latest APK from the [GitHub releases page](https://github.com/achyuki/FoldDevtools/releases)
2. Install on your Android device
3. Grant necessary permissions (root access if available, overlay permissions for floating window)

## Core Features

### 1. Local WebView Debugging (Root Required)

FoldDevtools can automatically detect and connect to WebView debug sockets on rooted devices:

- Automatically discovers `webview_devtools_remote_<pid>` sockets
- Lists all debuggable WebView instances
- Opens Chrome DevTools interface in a floating window or full screen

### 2. Remote Debugging (Rootless Compatible)

Connect to remote debug targets by specifying host and port:

```
Host: 127.0.0.1
Port: 9222
```

### 3. XPosed Module

When installed as an XPosed/LSPosed module, FoldDevtools can force-enable WebView debugging in any app without code modifications.

## Rootless Setup (ADB Port Forwarding)

For non-rooted devices, manually forward the debug socket using ADB:

### Step 1: Find the Debug Socket

```bash
# List all debug sockets
adb shell cat /proc/net/unix | grep devtools_remote

# Output examples:
# 0000000000000000: 00000002 00000000 00010000 0001 01 xxxxxxx @webview_devtools_remote_12345
# 0000000000000000: 00000002 00000000 00010000 0001 01 xxxxxxx @stetho_com.example.app_devtools_remote
```

### Step 2: Forward the Socket to TCP Port

```bash
# For WebView debugging
adb forward tcp:9222 localabstract:webview_devtools_remote_12345

# For Stetho debugging
adb forward tcp:9222 localabstract:stetho_com.example.app_devtools_remote
```

### Step 3: Connect in FoldDevtools

Open FoldDevtools app and use remote mode:
- Host: `127.0.0.1`
- Port: `9222`

## Enabling WebView Debugging in Your App

### Kotlin Implementation

```kotlin
import android.webkit.WebView

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Enable WebView debugging (should be done once, ideally in Application class)
        if (BuildConfig.DEBUG) {
            WebView.setWebContentsDebuggingEnabled(true)
        }
        
        val webView = WebView(this)
        webView.loadUrl("https://example.com")
        setContentView(webView)
    }
}
```

### Application-Level Setup

```kotlin
import android.app.Application
import android.webkit.WebView

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        // Enable for all WebViews in the app
        if (BuildConfig.DEBUG) {
            WebView.setWebContentsDebuggingEnabled(true)
        }
    }
}
```

### Conditional Release Build Debugging

```kotlin
import android.webkit.WebView

object DebugConfig {
    fun enableWebViewDebugging(enable: Boolean = true) {
        WebView.setWebContentsDebuggingEnabled(enable)
    }
}

// In your activity or application class
DebugConfig.enableWebViewDebugging(
    enable = BuildConfig.DEBUG || isInternalTester()
)
```

## Stetho Integration

### Add Stetho Dependency

```gradle
dependencies {
    debugImplementation 'com.facebook.stetho:stetho:1.6.0'
    debugImplementation 'com.facebook.stetho:stetho-okhttp3:1.6.0'
}
```

### Initialize Stetho

```kotlin
import android.app.Application
import com.facebook.stetho.Stetho

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        if (BuildConfig.DEBUG) {
            Stetho.initializeWithDefaults(this)
        }
    }
}
```

### Connect to Stetho Socket

```bash
# Find Stetho socket
adb shell cat /proc/net/unix | grep stetho

# Forward to local port
adb forward tcp:9222 localabstract:stetho_com.yourpackage.name_devtools_remote

# Connect via FoldDevtools remote mode: 127.0.0.1:9222
```

## Common Debugging Workflows

### Workflow 1: Debug WebView in Development

```kotlin
// 1. Enable debugging in your app
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        WebView.setWebContentsDebuggingEnabled(BuildConfig.DEBUG)
    }
}

// 2. On rooted device: Open FoldDevtools, select WebView from list
// 3. On rootless device: Forward socket and connect remotely
```

### Workflow 2: Debug Third-Party App (Root + XPosed)

1. Install FoldDevtools as XPosed module
2. Enable module in LSPosed/XPosed
3. Select target app in XPosed module settings
4. Restart target app
5. Open FoldDevtools and select the app's WebView

### Workflow 3: Debug Remote Browser/Node.js

```bash
# Start Chrome with remote debugging on Android
adb shell am start -n com.android.chrome/com.google.android.apps.chrome.Main \
  --remote-debugging-port=9222

# Forward port
adb forward tcp:9222 tcp:9222

# Connect via FoldDevtools: 127.0.0.1:9222
```

## Troubleshooting

### WebView Not Appearing in List

**Issue**: WebView not visible in FoldDevtools on rooted device

**Solutions**:
- Ensure `WebView.setWebContentsDebuggingEnabled(true)` is called
- Verify root access is granted to FoldDevtools
- Check that WebView is actually loaded (not just initialized)
- Restart the target app

### Connection Refused on Remote Mode

**Issue**: Cannot connect to `127.0.0.1:9222`

**Solutions**:
```bash
# Verify port forwarding is active
adb forward --list

# Re-establish forwarding
adb forward --remove-all
adb forward tcp:9222 localabstract:webview_devtools_remote_<pid>

# Check if socket exists
adb shell cat /proc/net/unix | grep devtools_remote
```

### Socket Not Found

**Issue**: `grep devtools_remote` returns empty

**Solutions**:
- Ensure target app has WebView debugging enabled
- Load a page in the WebView (socket appears only when WebView is active)
- Check if app is running: `adb shell ps | grep <package_name>`
- For Stetho, verify Stetho is initialized in the app

### XPosed Module Not Working

**Issue**: Force-enable debugging doesn't work

**Solutions**:
- Verify module is enabled in LSPosed/XPosed framework
- Check module scope includes target app
- Force stop and restart target app
- Check LSPosed logs for errors
- Ensure FoldDevtools has correct SELinux context (for rooted devices)

### DevTools UI Not Responding

**Issue**: Floating window or DevTools interface freezes

**Solutions**:
- Close and reopen FoldDevtools
- Clear app cache: Settings → Apps → FoldDevtools → Clear Cache
- Ensure overlay permissions are granted
- Try full-screen mode instead of floating window
- Check Android version compatibility (see GitHub issues)

## Advanced Usage

### Multiple WebView Instances

```kotlin
// When debugging apps with multiple WebViews
class MultiWebViewActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val webView1 = WebView(this).apply {
            settings.javaScriptEnabled = true
            loadUrl("https://example.com")
        }
        
        val webView2 = WebView(this).apply {
            settings.javaScriptEnabled = true
            loadUrl("https://another-site.com")
        }
        
        // Both WebViews will appear as separate targets in FoldDevtools
    }
}
```

### Custom Debug Socket Names

When using custom implementations, sockets follow patterns:
- WebView: `@webview_devtools_remote_<pid>`
- Stetho: `@stetho_<packageName>_devtools_remote`
- Custom: May vary, use `adb shell cat /proc/net/unix` to find

## Resources

- [Chrome DevTools Protocol Documentation](https://chromedevtools.github.io/devtools-protocol/)
- [Android WebView Debugging Guide](https://developer.android.com/guide/webapps/debugging)
- [Stetho GitHub Repository](https://github.com/facebook/stetho)
- [FoldDevtools Issues](https://github.com/achyuki/FoldDevtools/issues)
