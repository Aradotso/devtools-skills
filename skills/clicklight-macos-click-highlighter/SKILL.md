---
name: clicklight-macos-click-highlighter
description: Master ClickLight, the macOS menu bar app that highlights clicks for demos, recordings, and UX reviews with real-time visual feedback.
triggers:
  - how do I use ClickLight for click visualization
  - configure ClickLight click highlighting
  - customize ClickLight appearance and settings
  - build ClickLight from source
  - troubleshoot ClickLight accessibility permissions
  - integrate ClickLight into my workflow
  - modify ClickLight click effects
  - debug ClickLight overlay issues
---

# ClickLight macOS Click Highlighter Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

ClickLight is a native macOS menu bar application written in Swift that provides real-time click highlighting for live demos, screen sharing, UX reviews, and presentations. Unlike screen recorders that add click effects in post-production, ClickLight shows visual feedback during the live moment itself.

**Key features:**
- Real-time click highlights with separate visuals for press, release, right-click, and drag
- Optional laser pointer mode with fading freehand strokes
- Customizable size, duration, intensity, and color
- Native Swift/AppKit implementation
- Menu bar integration with quick presets

## Installation

### Homebrew (Recommended)

```bash
# Add tap and install
brew tap aurorascharff/clicklight https://github.com/aurorascharff/ClickLight
brew install --cask aurorascharff/clicklight/clicklight

# Update later
brew upgrade --cask clicklight

# Uninstall
brew uninstall --cask clicklight

# Uninstall with preferences
brew uninstall --cask --zap clicklight
```

### Manual Installation

Download `ClickLight.zip` from [GitHub Releases](https://github.com/aurorascharff/ClickLight/releases), extract, and move `ClickLight.app` to `/Applications`.

### Building from Source

```bash
# Clone repository
git clone https://github.com/aurorascharff/ClickLight.git
cd ClickLight

# Build with Swift
swift build -c release

# Or use the Makefile
make build

# Install to Applications
make install

# Run directly
make run
```

## Required Permissions

ClickLight requires **Accessibility** permission to detect clicks system-wide.

**Setup:**
1. Launch ClickLight (you'll be prompted for permission)
2. Go to **System Settings → Privacy & Security → Accessibility**
3. Enable ClickLight in the list
4. Quit and relaunch ClickLight from the menu bar

If clicks aren't being detected, verify the permission is enabled and restart the app.

## Project Structure

```
ClickLight/
├── Sources/
│   └── ClickLight/
│       ├── main.swift              # Entry point
│       ├── AppDelegate.swift       # Menu bar controller
│       ├── ClickMonitor.swift      # Global click detection
│       ├── OverlayWindow.swift     # Click effect rendering
│       ├── SettingsWindow.swift    # Preferences UI
│       └── Preferences.swift       # UserDefaults wrapper
├── Package.swift                   # Swift package manifest
├── Makefile                        # Build automation
└── docs/                          # Documentation
```

## Core Architecture

### 1. Click Monitoring (`ClickMonitor.swift`)

Listens for global mouse events using `CGEvent` tap:

```swift
import Cocoa

class ClickMonitor {
    private var eventTap: CFMachPort?
    weak var delegate: ClickMonitorDelegate?
    
    func start() {
        let eventMask = (1 << CGEventType.leftMouseDown.rawValue) |
                        (1 << CGEventType.leftMouseUp.rawValue) |
                        (1 << CGEventType.rightMouseDown.rawValue) |
                        (1 << CGEventType.rightMouseUp.rawValue) |
                        (1 << CGEventType.leftMouseDragged.rawValue)
        
        guard let eventTap = CGEvent.tapCreate(
            tap: .cgSessionEventTap,
            place: .headInsertEventTap,
            options: .defaultTap,
            eventsOfInterest: CGEventMask(eventMask),
            callback: { (proxy, type, event, refcon) -> Unmanaged<CGEvent>? in
                guard let refcon = refcon else { return Unmanaged.passRetained(event) }
                let monitor = Unsafely<ClickMonitor>.fromPointer(refcon)
                monitor.handleEvent(type: type, event: event)
                return Unmanaged.passRetained(event)
            },
            userInfo: Unafely.toPointer(self)
        ) else {
            print("Failed to create event tap")
            return
        }
        
        let runLoopSource = CFMachPortCreateRunLoopSource(kCFAllocatorDefault, eventTap, 0)
        CFRunLoopAddSource(CFRunLoopGetCurrent(), runLoopSource, .commonModes)
        CGEvent.tapEnable(tap: eventTap, enable: true)
        
        self.eventTap = eventTap
    }
    
    private func handleEvent(type: CGEventType, event: CGEvent) {
        let location = event.location
        
        switch type {
        case .leftMouseDown:
            delegate?.didDetectClick(at: location, type: .press)
        case .leftMouseUp:
            delegate?.didDetectClick(at: location, type: .release)
        case .rightMouseDown:
            delegate?.didDetectClick(at: location, type: .rightClick)
        case .leftMouseDragged:
            delegate?.didDetectDrag(at: location)
        default:
            break
        }
    }
    
    func stop() {
        if let eventTap = eventTap {
            CGEvent.tapEnable(tap: eventTap, enable: false)
            CFMachPortInvalidate(eventTap)
        }
    }
}

protocol ClickMonitorDelegate: AnyObject {
    func didDetectClick(at location: CGPoint, type: ClickType)
    func didDetectDrag(at location: CGPoint)
}

enum ClickType {
    case press, release, rightClick
}
```

### 2. Overlay Rendering (`OverlayWindow.swift`)

Creates a transparent window overlay for click effects:

```swift
import Cocoa

class OverlayWindow: NSWindow {
    init() {
        // Cover all screens
        let screenFrame = NSScreen.screens.reduce(NSRect.zero) { (result, screen) in
            return result.union(screen.frame)
        }
        
        super.init(
            contentRect: screenFrame,
            styleMask: .borderless,
            backing: .buffered,
            defer: false
        )
        
        self.isOpaque = false
        self.backgroundColor = .clear
        self.level = .floating
        self.ignoresMouseEvents = true
        self.collectionBehavior = [.canJoinAllSpaces, .stationary]
        
        let overlayView = OverlayView(frame: screenFrame)
        self.contentView = overlayView
    }
}

class OverlayView: NSView {
    private var clickEffects: [ClickEffect] = []
    
    override func draw(_ dirtyRect: NSRect) {
        guard let context = NSGraphicsContext.current?.cgContext else { return }
        
        for effect in clickEffects {
            effect.draw(in: context)
        }
    }
    
    func addClickEffect(at location: CGPoint, type: ClickType, config: EffectConfig) {
        let effect = ClickEffect(
            location: location,
            type: type,
            size: config.size,
            duration: config.duration,
            intensity: config.intensity,
            color: config.color
        )
        
        clickEffects.append(effect)
        
        // Animate effect
        effect.animate { [weak self] in
            self?.clickEffects.removeAll { $0 === effect }
            self?.needsDisplay = true
        }
        
        self.needsDisplay = true
    }
}

struct EffectConfig {
    let size: CGFloat
    let duration: TimeInterval
    let intensity: CGFloat
    let color: NSColor
}

class ClickEffect {
    let location: CGPoint
    let type: ClickType
    var currentRadius: CGFloat
    var currentAlpha: CGFloat = 1.0
    
    private let maxRadius: CGFloat
    private let duration: TimeInterval
    private let color: NSColor
    
    init(location: CGPoint, type: ClickType, size: CGFloat, duration: TimeInterval, intensity: CGFloat, color: NSColor) {
        self.location = location
        self.type = type
        self.maxRadius = size
        self.duration = duration
        self.color = color.withAlphaComponent(intensity)
        self.currentRadius = size * 0.5
    }
    
    func draw(in context: CGContext) {
        context.setStrokeColor(color.cgColor)
        context.setLineWidth(3.0)
        context.setAlpha(currentAlpha)
        
        let rect = CGRect(
            x: location.x - currentRadius,
            y: location.y - currentRadius,
            width: currentRadius * 2,
            height: currentRadius * 2
        )
        
        context.strokeEllipse(in: rect)
    }
    
    func animate(completion: @escaping () -> Void) {
        let startTime = Date()
        
        Timer.scheduledTimer(withTimeInterval: 1/60.0, repeats: true) { [weak self] timer in
            guard let self = self else {
                timer.invalidate()
                return
            }
            
            let elapsed = Date().timeIntervalSince(startTime)
            let progress = min(elapsed / self.duration, 1.0)
            
            if progress >= 1.0 {
                timer.invalidate()
                completion()
                return
            }
            
            // Expand and fade
            self.currentRadius = self.maxRadius * (0.5 + progress * 0.5)
            self.currentAlpha = 1.0 - progress
        }
    }
}
```

### 3. Preferences Management (`Preferences.swift`)

```swift
import Foundation
import Cocoa

class Preferences {
    static let shared = Preferences()
    
    private let defaults = UserDefaults.standard
    
    enum Key: String {
        case clickSize = "clickSize"
        case clickDuration = "clickDuration"
        case clickIntensity = "clickIntensity"
        case clickColor = "clickColor"
        case laserPointerEnabled = "laserPointerEnabled"
        case compactIcon = "compactIcon"
    }
    
    var clickSize: CGFloat {
        get { CGFloat(defaults.double(forKey: Key.clickSize.rawValue)) }
        set { defaults.set(Double(newValue), forKey: Key.clickSize.rawValue) }
    }
    
    var clickDuration: TimeInterval {
        get { defaults.double(forKey: Key.clickDuration.rawValue) }
        set { defaults.set(newValue, forKey: Key.clickDuration.rawValue) }
    }
    
    var clickIntensity: CGFloat {
        get { CGFloat(defaults.double(forKey: Key.clickIntensity.rawValue)) }
        set { defaults.set(Double(newValue), forKey: Key.clickIntensity.rawValue) }
    }
    
    var clickColor: NSColor {
        get {
            guard let data = defaults.data(forKey: Key.clickColor.rawValue),
                  let color = try? NSKeyedUnarchiver.unarchivedObject(ofClass: NSColor.self, from: data) else {
                return .systemBlue
            }
            return color
        }
        set {
            if let data = try? NSKeyedArchiver.archivedData(withRootObject: newValue, requiringSecureCoding: false) {
                defaults.set(data, forKey: Key.clickColor.rawValue)
            }
        }
    }
    
    var laserPointerEnabled: Bool {
        get { defaults.bool(forKey: Key.laserPointerEnabled.rawValue) }
        set { defaults.set(newValue, forKey: Key.laserPointerEnabled.rawValue) }
    }
    
    func registerDefaults() {
        defaults.register(defaults: [
            Key.clickSize.rawValue: 50.0,
            Key.clickDuration.rawValue: 0.5,
            Key.clickIntensity.rawValue: 0.7,
            Key.laserPointerEnabled.rawValue: false,
            Key.compactIcon.rawValue: false
        ])
    }
}
```

### 4. Menu Bar Integration (`AppDelegate.swift`)

```swift
import Cocoa

@main
class AppDelegate: NSObject, NSApplicationDelegate {
    private var statusItem: NSStatusItem!
    private var clickMonitor: ClickMonitor!
    private var overlayWindow: OverlayWindow!
    private var settingsWindow: SettingsWindow?
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        Preferences.shared.registerDefaults()
        
        // Create menu bar item
        statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
        statusItem.button?.image = NSImage(systemSymbolName: "circle.circle", accessibilityDescription: "ClickLight")
        statusItem.menu = createMenu()
        
        // Initialize overlay
        overlayWindow = OverlayWindow()
        overlayWindow.orderFront(nil)
        
        // Start monitoring
        clickMonitor = ClickMonitor()
        clickMonitor.delegate = self
        clickMonitor.start()
    }
    
    private func createMenu() -> NSMenu {
        let menu = NSMenu()
        
        menu.addItem(NSMenuItem(title: "Settings...", action: #selector(openSettings), keyEquivalent: ","))
        menu.addItem(NSMenuItem.separator())
        
        // Quick presets
        let sizeMenu = NSMenu()
        sizeMenu.addItem(createPresetItem(title: "Small", size: 30))
        sizeMenu.addItem(createPresetItem(title: "Medium", size: 50))
        sizeMenu.addItem(createPresetItem(title: "Large", size: 80))
        
        let sizeItem = NSMenuItem(title: "Size", action: nil, keyEquivalent: "")
        sizeItem.submenu = sizeMenu
        menu.addItem(sizeItem)
        
        menu.addItem(NSMenuItem.separator())
        menu.addItem(NSMenuItem(title: "Test Pulse", action: #selector(testPulse), keyEquivalent: "t"))
        menu.addItem(NSMenuItem.separator())
        menu.addItem(NSMenuItem(title: "Quit ClickLight", action: #selector(quit), keyEquivalent: "q"))
        
        return menu
    }
    
    @objc private func openSettings() {
        if settingsWindow == nil {
            settingsWindow = SettingsWindow()
        }
        settingsWindow?.showWindow(nil)
        NSApp.activate(ignoringOtherApps: true)
    }
    
    @objc private func testPulse() {
        guard let screen = NSScreen.main else { return }
        let center = CGPoint(x: screen.frame.midX, y: screen.frame.midY)
        
        let config = EffectConfig(
            size: Preferences.shared.clickSize,
            duration: Preferences.shared.clickDuration,
            intensity: Preferences.shared.clickIntensity,
            color: Preferences.shared.clickColor
        )
        
        (overlayWindow.contentView as? OverlayView)?.addClickEffect(at: center, type: .press, config: config)
    }
    
    @objc private func quit() {
        NSApplication.shared.terminate(nil)
    }
    
    private func createPresetItem(title: String, size: CGFloat) -> NSMenuItem {
        let item = NSMenuItem(title: title, action: #selector(applyPreset(_:)), keyEquivalent: "")
        item.representedObject = size
        return item
    }
    
    @objc private func applyPreset(_ sender: NSMenuItem) {
        if let size = sender.representedObject as? CGFloat {
            Preferences.shared.clickSize = size
        }
    }
}

extension AppDelegate: ClickMonitorDelegate {
    func didDetectClick(at location: CGPoint, type: ClickType) {
        let config = EffectConfig(
            size: Preferences.shared.clickSize,
            duration: Preferences.shared.clickDuration,
            intensity: Preferences.shared.clickIntensity,
            color: Preferences.shared.clickColor
        )
        
        (overlayWindow.contentView as? OverlayView)?.addClickEffect(at: location, type: type, config: config)
    }
    
    func didDetectDrag(at location: CGPoint) {
        guard Preferences.shared.laserPointerEnabled else { return }
        // Implement laser pointer trail logic
    }
}
```

## Configuration

### Programmatic Configuration

Modify preferences directly:

```swift
// Adjust click size
Preferences.shared.clickSize = 60.0

// Change duration (seconds)
Preferences.shared.clickDuration = 0.8

// Set intensity (0.0 to 1.0)
Preferences.shared.clickIntensity = 0.9

// Custom color
Preferences.shared.clickColor = NSColor(red: 1.0, green: 0.3, blue: 0.3, alpha: 1.0)

// Enable laser pointer mode
Preferences.shared.laserPointerEnabled = true
```

### UserDefaults Keys

Access preferences via command line:

```bash
# View all ClickLight preferences
defaults read com.yourname.ClickLight

# Set click size
defaults write com.yourname.ClickLight clickSize -float 70

# Set duration
defaults write com.yourname.ClickLight clickDuration -float 0.6

# Reset to defaults
defaults delete com.yourname.ClickLight
```

## Common Patterns

### Custom Click Effect Style

```swift
// Create custom effect renderer
class CustomClickEffect: ClickEffect {
    override func draw(in context: CGContext) {
        // Draw square instead of circle
        let rect = CGRect(
            x: location.x - currentRadius,
            y: location.y - currentRadius,
            width: currentRadius * 2,
            height: currentRadius * 2
        )
        
        context.setStrokeColor(color.cgColor)
        context.setLineWidth(4.0)
        context.setAlpha(currentAlpha)
        context.stroke(rect)
    }
}
```

### Multi-Screen Support

```swift
// Handle multiple displays
extension OverlayWindow {
    func updateForScreenConfiguration() {
        let unionFrame = NSScreen.screens.reduce(NSRect.zero) { $0.union($1.frame) }
        self.setFrame(unionFrame, display: true)
        self.contentView?.frame = unionFrame
    }
}

// Listen for screen changes
NotificationCenter.default.addObserver(
    forName: NSApplication.didChangeScreenParametersNotification,
    object: nil,
    queue: .main
) { [weak self] _ in
    self?.overlayWindow.updateForScreenConfiguration()
}
```

### Keyboard Shortcut Integration

```swift
// Add global hotkey support
import Carbon

class HotkeyManager {
    private var eventHandler: EventHandlerRef?
    
    func registerHotkey(key: UInt32, modifiers: UInt32, action: @escaping () -> Void) {
        var hotKeyID = EventHotKeyID(signature: 0x4343, id: 1)
        var hotKeyRef: EventHotKeyRef?
        
        RegisterEventHotKey(
            key,
            modifiers,
            hotKeyID,
            GetApplicationEventTarget(),
            0,
            &hotKeyRef
        )
        
        // Store action callback
    }
}

// Usage: Toggle ClickLight with Cmd+Shift+C
let hotkeyManager = HotkeyManager()
hotkeyManager.registerHotkey(key: 8, modifiers: cmdKey | shiftKey) {
    // Toggle click monitoring
}
```

## Development Workflow

### Build Commands

```bash
# Build debug version
swift build

# Build release version
swift build -c release

# Run directly
swift run

# Clean build artifacts
swift package clean

# Run tests
swift test
```

### Makefile Shortcuts

```bash
# Build and run
make run

# Build release
make build

# Install to /Applications
make install

# Clean build directory
make clean

# Create distribution archive
make archive
```

### Debugging

Enable verbose logging:

```swift
// Add to main.swift
#if DEBUG
let logger = Logger(subsystem: "com.yourname.ClickLight", category: "debug")
logger.debug("Click detected at \(location)")
#endif
```

Monitor click events:

```bash
# Watch event tap status
log stream --predicate 'subsystem == "com.yourname.ClickLight"' --level debug
```

## Troubleshooting

### Clicks Not Detected

1. **Check Accessibility permission:**
   ```bash
   # List apps with Accessibility access
   sqlite3 "/Library/Application Support/com.apple.TCC/TCC.db" \
     "SELECT client FROM access WHERE service='kTCCServiceAccessibility'"
   ```

2. **Verify event tap is active:**
   ```swift
   // Add diagnostic logging
   if eventTap == nil {
       print("ERROR: Event tap failed to create")
       print("Check Accessibility permission in System Settings")
   }
   ```

3. **Restart after permission grant:**
   Must quit and relaunch for permission to take effect.

### Overlay Not Visible

1. **Check window level:**
   ```swift
   overlayWindow.level = .floating  // Should be above normal windows
   ```

2. **Verify transparency:**
   ```swift
   print("Overlay opaque: \(overlayWindow.isOpaque)")  // Should be false
   print("Background: \(overlayWindow.backgroundColor)")  // Should be clear
   ```

3. **Test with simple shape:**
   ```swift
   // Draw test rectangle to verify overlay is rendering
   context.setFillColor(NSColor.red.cgColor)
   context.fill(CGRect(x: 100, y: 100, width: 100, height: 100))
   ```

### Performance Issues

1. **Limit active effects:**
   ```swift
   // Remove old effects after max count
   if clickEffects.count > 10 {
       clickEffects.removeFirst(clickEffects.count - 10)
   }
   ```

2. **Optimize drawing:**
   ```swift
   // Only redraw dirty regions
   override func draw(_ dirtyRect: NSRect) {
       let visibleEffects = clickEffects.filter { effect in
           dirtyRect.intersects(effect.boundingRect)
       }
       for effect in visibleEffects {
           effect.draw(in: context)
       }
   }
   ```

3. **Reduce animation frequency:**
   ```swift
   // Lower frame rate for animations
   Timer.scheduledTimer(withTimeInterval: 1/30.0, repeats: true) { ... }
   ```

### Build Errors

```bash
# Update Swift tools version
swift package tools-version --set-current

# Resolve dependencies
swift package resolve

# Update Package.resolved
swift package update

# Clean and rebuild
rm -rf .build && swift build
```

### Code Signing Issues

```bash
# Sign for local development
codesign --force --deep --sign - ClickLight.app

# Verify signature
codesign --verify --verbose ClickLight.app

# Check entitlements
codesign -d --entitlements - ClickLight.app
```

## Testing

### Manual Testing Checklist

- [ ] Click detection works across all applications
- [ ] Right-click shows different effect
- [ ] Drag events trigger laser pointer (if enabled)
- [ ] Settings changes apply immediately
- [ ] Menu bar icon updates correctly
- [ ] Test pulse fires from center screen
- [ ] Effects scale properly on Retina displays
- [ ] Multi-monitor support works
- [ ] Preferences persist after quit/relaunch

### Automated Testing

```swift
// Tests/ClickLightTests/PreferencesTests.swift
import XCTest
@testable import ClickLight

final class PreferencesTests: XCTestCase {
    func testDefaultValues() {
        let prefs = Preferences.shared
        XCTAssertEqual(prefs.clickSize, 50.0)
        XCTAssertEqual(prefs.clickDuration, 0.5)
        XCTAssertEqual(prefs.clickIntensity, 0.7)
    }
    
    func testColorPersistence() {
        let prefs = Preferences.shared
        let testColor = NSColor.red
        
        prefs.clickColor = testColor
        let retrieved = prefs.clickColor
        
        XCTAssertEqual(retrieved, testColor)
    }
}
```

Run tests:

```bash
swift test
```

## Integration Examples

### Use with OBS Studio

ClickLight overlays work with OBS window/display capture:

1. Launch ClickLight
2. In OBS, add **Display Capture** source
3. ClickLight effects will appear in recording

### Use with Screen Studio

ClickLight provides live effects; Screen Studio can add post-production effects:

- ClickLight: Real-time visibility during recording
- Screen Studio: Enhanced post-production effects

Both can work together for maximum visibility.

### Use with QuickTime Screen Recording

```bash
# Record screen with ClickLight active
# Effects will be captured in the recording
```

## Advanced Customization

### Add Custom Preset

```swift
// Create preset configuration
struct Preset {
    let name: String
    let size: CGFloat
    let duration: TimeInterval
    let intensity: CGFloat
    let color: NSColor
}

let presets = [
    Preset(name: "Subtle", size: 30, duration: 0.3, intensity: 0.5, color: .systemGray),
    Preset(name: "Bold", size: 80, duration: 0.8, intensity: 1.0, color: .systemRed),
    Preset(name: "Presentation", size: 60, duration: 0.6, intensity: 0.8, color: .systemBlue)
]

// Apply preset
func applyPreset(_ preset: Preset) {
    Preferences.shared.clickSize = preset.size
    Preferences.shared.clickDuration = preset.duration
    Preferences.shared.clickIntensity = preset.intensity
    Preferences.shared.clickColor = preset.color
}
```

### Add Sound Effects

```swift
import AVFoundation

class SoundPlayer {
    private var audioPlayer: AVAudioPlayer?
    
    func playClickSound() {
        guard let soundURL = Bundle.main.url(forResource: "click", withExtension: "wav") else { return }
        
        do {
            audioPlayer = try AVAudioPlayer(contentsOf: soundURL)
            audioPlayer?.volume = 0.3
            audioPlayer?.play()
        } catch {
            print("Failed to play sound: \(error)")
        }
    }
}

// Integrate into click detection
extension AppDelegate {
    func didDetectClick(at location: CGPoint, type: ClickType) {
        // Existing visual effect code...
        soundPlayer.playClickSound()
    }
}
```

### Export Settings

```swift
// Export preferences to JSON
extension Preferences {
    func exportSettings() -> String? {
        let settings: [String: Any] = [
            "clickSize": clickSize,
            "clickDuration": clickDuration,
            "clickIntensity": clickIntensity,
            "laserPointerEnabled": laserPointerEnabled
        ]
        
        guard let data = try? JSONSerialization.data(withJSONObject: settings, options: .prettyPrinted),
              let json = String(data: data, encoding: .utf8) else {
            return nil
        }
        
        return json
    }
    
    func importSettings(json: String) {
        guard let data = json.data(using: .utf8),
              let settings = try? JSONSerialization.jsonObject(with: data) as? [String: Any] else {
            return
        }
        
        if let size = settings["clickSize"] as? Double {
            clickSize = CGFloat(size)
        }
        if let duration = settings["clickDuration"] as? Double {
            clickDuration = duration
        }
        // ... import other settings
    }
}
```

## Resources

- **Repository:** https://github.com/aurorascharff/ClickLight
- **Releases:** https://github.com/aurorascharff/ClickLight/releases
- **Issues:** https://github.com/aurorascharff/ClickLight/issues
- **License:** MIT
- **Swift Documentation:** https://www.swift.org/documentation/
- **AppKit Reference:** https://developer.apple.com/documentation/appkit
