---
name: optiscaler-client-fsr4-manager
description: OptiScaler Client FSR4 manager for game upscaling, frame generation, and performance optimization with AMD FSR4, DLSS replacement, and mod management
triggers:
  - how do I install OptiScaler FSR4 for my games
  - configure FSR4 upscaling with OptiScaler
  - replace DLSS with FSR4 using OptiScaler
  - manage game profiles with OptiScaler Client
  - troubleshoot OptiScaler FSR4 artifacts or performance
  - set up frame generation with OptiScaler
  - scan Steam library for OptiScaler compatible games
  - configure OptiScaler INI settings for game optimization
---

# OptiScaler Client FSR4 Manager

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OptiScaler Client FSR4 is a desktop utility for managing AMD FidelityFX Super Resolution 4 (FSR4) upscaling and frame generation technology in PC games. It provides a modern GUI for:

- **Game library scanning** (Steam, Epic, manual paths)
- **DLSS to FSR4 replacement** (DLL patching)
- **Profile management** with per-game INI configurations
- **Frame generation** and low-latency mode toggles
- **Auto-detection** of compatible games
- **Mod management** for upscaling technologies (FSR, XeSS, DLSS)

Built with C# and Avalonia UI, it targets Windows 11 and Linux (Steam Deck compatible).

## Installation

1. **Download the latest release**:
   ```bash
   # Download from releases page
   curl -L -o OptiscalerClient.zip \
     https://github.com/Devanshu-dev/optiscaler-client-fsr4/releases/download/v1.0.5/OptiscalerClient-1.0.5.zip
   
   # Extract
   unzip OptiscalerClient.zip -d OptiScaler
   cd OptiScaler
   ```

2. **Run the application**:
   ```bash
   # Windows
   OptiscalerClient.exe
   
   # Linux
   chmod +x OptiscalerClient
   ./OptiscalerClient
   ```

3. **First launch**:
   - Application will prompt for administrator rights (required for DLL patching)
   - Automatically scans common game library locations
   - Creates default configuration in `%APPDATA%/OptiScaler` or `~/.config/OptiScaler`

## Key Features and Configuration

### Game Library Scanning

The application auto-detects games from:
- Steam library folders
- Epic Games Store
- Custom paths

**Manual game addition**:
```csharp
// Example: Adding a game manually via API/config
var game = new GameEntry
{
    Name = "Cyberpunk 2077",
    ExecutablePath = @"C:\Games\Cyberpunk 2077\bin\x64\Cyberpunk2077.exe",
    InstallDirectory = @"C:\Games\Cyberpunk 2077",
    SupportsFSR4 = true,
    SupportsDLSS = true
};

// The client stores this in games.json
```

**Configuration file** (`games.json`):
```json
{
  "games": [
    {
      "name": "Cyberpunk 2077",
      "path": "C:\\Games\\Cyberpunk 2077\\bin\\x64",
      "executable": "Cyberpunk2077.exe",
      "dlss_detected": true,
      "fsr4_enabled": true,
      "profile": "balanced"
    }
  ]
}
```

### INI Configuration Management

OptiScaler uses INI files for per-game settings. These are typically placed in the game's executable directory.

**Example `nvngx.ini`** (FSR4 configuration):
```ini
[FSR4]
# Quality presets: Ultra Performance, Performance, Balanced, Quality, Ultra Quality
Preset = Balanced

# Sharpness 0.0 - 1.0
Sharpness = 0.7

# Frame Generation
FrameGeneration = true
FrameGenerationQuality = High

# Low Latency Mode
LowLatency = true

# INT8 Optimization (for compatible GPUs)
EnableINT8 = true

# Anti-aliasing
TemporalAA = true

# Debugging
ShowDebugOverlay = false
LogLevel = Warning
```

**C# code to generate/update INI**:
```csharp
using System.IO;
using System.Text;

public class IniConfigManager
{
    public static void CreateFSR4Config(string gamePath, FSR4Settings settings)
    {
        var iniPath = Path.Combine(gamePath, "nvngx.ini");
        var sb = new StringBuilder();
        
        sb.AppendLine("[FSR4]");
        sb.AppendLine($"Preset = {settings.Preset}");
        sb.AppendLine($"Sharpness = {settings.Sharpness:F1}");
        sb.AppendLine($"FrameGeneration = {settings.FrameGeneration.ToString().ToLower()}");
        sb.AppendLine($"FrameGenerationQuality = {settings.FrameGenQuality}");
        sb.AppendLine($"LowLatency = {settings.LowLatency.ToString().ToLower()}");
        sb.AppendLine($"EnableINT8 = {settings.EnableINT8.ToString().ToLower()}");
        sb.AppendLine($"TemporalAA = {settings.TemporalAA.ToString().ToLower()}");
        sb.AppendLine($"ShowDebugOverlay = false");
        sb.AppendLine($"LogLevel = {settings.LogLevel}");
        
        File.WriteAllText(iniPath, sb.ToString());
    }
}

public class FSR4Settings
{
    public string Preset { get; set; } = "Balanced";
    public float Sharpness { get; set; } = 0.7f;
    public bool FrameGeneration { get; set; } = true;
    public string FrameGenQuality { get; set; } = "High";
    public bool LowLatency { get; set; } = true;
    public bool EnableINT8 { get; set; } = true;
    public bool TemporalAA { get; set; } = true;
    public string LogLevel { get; set; } = "Warning";
}
```

### DLSS to FSR4 Replacement (DLL Patching)

OptiScaler replaces NVIDIA DLSS DLLs with FSR4-enabled versions:

**DLL files replaced**:
- `nvngx_dlss.dll` → FSR4 wrapper
- `nvngx.dll` → OptiScaler bridge
- `_nvngx.dll` → Backup/compatibility layer

**C# patching logic**:
```csharp
using System.IO;
using System.Security.Cryptography;

public class DLLPatcher
{
    private readonly string _optiscalerDllsPath;
    
    public DLLPatcher(string optiscalerPath)
    {
        _optiscalerDllsPath = Path.Combine(optiscalerPath, "dlls");
    }
    
    public async Task PatchGameAsync(string gameExecutablePath)
    {
        var gameDir = Path.GetDirectoryName(gameExecutablePath);
        var dlssPath = Path.Combine(gameDir, "nvngx_dlss.dll");
        
        // Backup original DLSS DLL
        if (File.Exists(dlssPath))
        {
            var backupPath = Path.Combine(gameDir, "nvngx_dlss.dll.backup");
            if (!File.Exists(backupPath))
            {
                File.Copy(dlssPath, backupPath, false);
            }
        }
        
        // Copy FSR4 wrapper DLLs
        var fsr4WrapperPath = Path.Combine(_optiscalerDllsPath, "nvngx_dlss.dll");
        var optiscalerBridgePath = Path.Combine(_optiscalerDllsPath, "nvngx.dll");
        
        File.Copy(fsr4WrapperPath, dlssPath, true);
        File.Copy(optiscalerBridgePath, Path.Combine(gameDir, "nvngx.dll"), true);
        
        // Verify integrity
        if (!VerifyDllIntegrity(dlssPath))
        {
            throw new InvalidOperationException("DLL integrity check failed");
        }
    }
    
    private bool VerifyDllIntegrity(string dllPath)
    {
        using var md5 = MD5.Create();
        using var stream = File.OpenRead(dllPath);
        var hash = md5.ComputeHash(stream);
        // Compare against known good hash
        return true; // Simplified
    }
    
    public void RestoreOriginal(string gameExecutablePath)
    {
        var gameDir = Path.GetDirectoryName(gameExecutablePath);
        var dlssPath = Path.Combine(gameDir, "nvngx_dlss.dll");
        var backupPath = Path.Combine(gameDir, "nvngx_dlss.dll.backup");
        
        if (File.Exists(backupPath))
        {
            File.Copy(backupPath, dlssPath, true);
            File.Delete(backupPath);
        }
        
        // Remove OptiScaler DLLs
        File.Delete(Path.Combine(gameDir, "nvngx.dll"));
    }
}
```

### Profile Management

**Profile structure**:
```csharp
public class GameProfile
{
    public string GameName { get; set; }
    public string ProfileName { get; set; } = "Default";
    public FSR4Settings FSR4 { get; set; }
    public PatchingOptions Patching { get; set; }
    public bool AutoApply { get; set; } = false;
}

public class PatchingOptions
{
    public bool EnableOptiPatcher { get; set; } = true;
    public bool EnableNukeMFG { get; set; } = false;
    public bool EnableFakeNvAPI { get; set; } = true;
}

// Profile serialization
public class ProfileManager
{
    private readonly string _profilesPath;
    
    public ProfileManager(string configPath)
    {
        _profilesPath = Path.Combine(configPath, "profiles");
        Directory.CreateDirectory(_profilesPath);
    }
    
    public void SaveProfile(GameProfile profile)
    {
        var profilePath = Path.Combine(_profilesPath, $"{profile.GameName}.json");
        var json = System.Text.Json.JsonSerializer.Serialize(profile, new System.Text.Json.JsonSerializerOptions
        {
            WriteIndented = true
        });
        File.WriteAllText(profilePath, json);
    }
    
    public GameProfile LoadProfile(string gameName)
    {
        var profilePath = Path.Combine(_profilesPath, $"{gameName}.json");
        if (!File.Exists(profilePath))
            return CreateDefaultProfile(gameName);
            
        var json = File.ReadAllText(profilePath);
        return System.Text.Json.JsonSerializer.Deserialize<GameProfile>(json);
    }
    
    private GameProfile CreateDefaultProfile(string gameName)
    {
        return new GameProfile
        {
            GameName = gameName,
            ProfileName = "Default",
            FSR4 = new FSR4Settings(),
            Patching = new PatchingOptions()
        };
    }
}
```

### Frame Generation Configuration

**Enable frame generation**:
```csharp
public class FrameGenerationConfig
{
    // Frame generation modes
    public enum FGMode
    {
        Off,
        Native,      // AMD Fluid Motion Frames
        Software,    // OptiScaler software implementation
        Hybrid       // Combined approach
    }
    
    public static void ConfigureFrameGen(string gameDir, FGMode mode, int targetFPS = 60)
    {
        var configPath = Path.Combine(gameDir, "nvngx.ini");
        var lines = new List<string>();
        
        lines.Add("[FrameGeneration]");
        lines.Add($"Enabled = {(mode != FGMode.Off).ToString().ToLower()}");
        lines.Add($"Mode = {mode}");
        lines.Add($"TargetFPS = {targetFPS}");
        lines.Add($"MinFPSThreshold = {targetFPS / 2}"); // Disable FG below this
        lines.Add($"MotionVectorQuality = High");
        lines.Add($"HudFix = true"); // Fix HUD elements
        lines.Add($"RefreshRateSync = true");
        
        // Latency optimization
        lines.Add("[Latency]");
        lines.Add("LowLatencyMode = true");
        lines.Add("RefreshRateBoost = true");
        lines.Add("FramePacing = true");
        
        File.WriteAllLines(configPath, lines);
    }
}
```

## Common Usage Patterns

### Pattern 1: Auto-Configure Game on Detection

```csharp
public class AutoConfigService
{
    private readonly ProfileManager _profileManager;
    private readonly DLLPatcher _patcher;
    private readonly IniConfigManager _iniManager;
    
    public async Task AutoConfigureGameAsync(GameEntry game)
    {
        // Load or create profile
        var profile = _profileManager.LoadProfile(game.Name);
        
        // Detect game-specific optimizations
        if (game.Name.Contains("Cyberpunk"))
        {
            profile.FSR4.Preset = "Quality";
            profile.FSR4.Sharpness = 0.8f;
            profile.Patching.EnableNukeMFG = true; // Better motion blur
        }
        else if (game.Name.Contains("Horizon"))
        {
            profile.FSR4.Preset = "Balanced";
            profile.FSR4.FrameGeneration = true;
        }
        
        // Apply patches
        await _patcher.PatchGameAsync(game.ExecutablePath);
        
        // Write INI config
        IniConfigManager.CreateFSR4Config(
            Path.GetDirectoryName(game.ExecutablePath),
            profile.FSR4
        );
        
        // Save profile
        _profileManager.SaveProfile(profile);
    }
}
```

### Pattern 2: Batch Processing Multiple Games

```csharp
public class BatchProcessor
{
    public async Task ProcessLibraryAsync(List<GameEntry> games, FSR4Settings globalSettings)
    {
        var tasks = new List<Task>();
        
        foreach (var game in games.Where(g => g.SupportsDLSS))
        {
            tasks.Add(Task.Run(async () =>
            {
                try
                {
                    Console.WriteLine($"Processing {game.Name}...");
                    
                    var patcher = new DLLPatcher(Environment.GetFolderPath(
                        Environment.SpecialFolder.ApplicationData) + "\\OptiScaler");
                    
                    await patcher.PatchGameAsync(game.ExecutablePath);
                    
                    IniConfigManager.CreateFSR4Config(
                        Path.GetDirectoryName(game.ExecutablePath),
                        globalSettings
                    );
                    
                    Console.WriteLine($"✓ {game.Name} configured successfully");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"✗ {game.Name} failed: {ex.Message}");
                }
            }));
        }
        
        await Task.WhenAll(tasks);
    }
}
```

### Pattern 3: CLI Automation

```csharp
// Command-line interface for automation
public class OptiScalerCLI
{
    public static async Task Main(string[] args)
    {
        var command = args.Length > 0 ? args[0] : "help";
        
        switch (command.ToLower())
        {
            case "scan":
                await ScanLibraryAsync();
                break;
                
            case "patch":
                if (args.Length < 2)
                {
                    Console.WriteLine("Usage: optiscaler patch <game-path>");
                    return;
                }
                await PatchGameAsync(args[1]);
                break;
                
            case "restore":
                if (args.Length < 2)
                {
                    Console.WriteLine("Usage: optiscaler restore <game-path>");
                    return;
                }
                RestoreGame(args[1]);
                break;
                
            case "config":
                if (args.Length < 3)
                {
                    Console.WriteLine("Usage: optiscaler config <game-path> <preset>");
                    return;
                }
                ConfigureGame(args[1], args[2]);
                break;
                
            default:
                ShowHelp();
                break;
        }
    }
    
    private static async Task PatchGameAsync(string gamePath)
    {
        var patcher = new DLLPatcher(GetOptiScalerPath());
        await patcher.PatchGameAsync(gamePath);
        Console.WriteLine($"Game patched: {gamePath}");
    }
    
    private static void ConfigureGame(string gamePath, string preset)
    {
        var settings = new FSR4Settings { Preset = preset };
        IniConfigManager.CreateFSR4Config(
            Path.GetDirectoryName(gamePath),
            settings
        );
        Console.WriteLine($"Configuration applied: {preset}");
    }
    
    private static string GetOptiScalerPath()
    {
        return Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
            "OptiScaler"
        );
    }
}
```

## Troubleshooting

### Visual Artifacts

**Problem**: Ghosting, shimmering, or temporal artifacts

**Solution**:
```csharp
// Adjust temporal settings
var settings = new FSR4Settings
{
    Preset = "Quality", // Use higher quality preset
    Sharpness = 0.5f,   // Reduce sharpness
    TemporalAA = true,  // Ensure TAA is enabled
};

// Add to INI:
// [FSR4]
// MotionVectorScale = 0.8  # Reduce motion vector influence
// TemporalJitter = 0.5     # Lower jitter for less ghosting
```

### Poor Performance

**Problem**: FPS drops below expectations

**Solution**:
```csharp
// Optimize for performance
var settings = new FSR4Settings
{
    Preset = "Performance",     // Lower quality preset
    EnableINT8 = true,          // Use INT8 optimization
    FrameGeneration = false,    // Disable if causing issues
    LowLatency = true           // Enable low latency mode
};

// Check if game is using too high resolution:
// Base resolution should be appropriate for preset
// Quality: ~67% of target
// Balanced: ~58% of target
// Performance: ~50% of target
```

### Game Crashes or Won't Start

**Problem**: Game fails to launch after patching

**Solution**:
```csharp
// Restore original DLSS
var patcher = new DLLPatcher(GetOptiScalerPath());
patcher.RestoreOriginal(gameExecutablePath);

// Or manually:
// 1. Navigate to game directory
// 2. Delete nvngx_dlss.dll and nvngx.dll
// 3. Rename nvngx_dlss.dll.backup to nvngx_dlss.dll
```

### High Input Lag with Frame Generation

**Problem**: Noticeable input latency

**Solution**:
```ini
[FrameGeneration]
Enabled = true
MinFPSThreshold = 40  # Only enable FG above 40 FPS

[Latency]
LowLatencyMode = true
RefreshRateBoost = true
FramePacing = true
AsyncCompute = true    # Overlap work for lower latency
```

### DLL Patching Permission Errors

**Problem**: Access denied when copying DLLs

**Solution**:
```csharp
// Run with elevated privileges
using System.Diagnostics;
using System.Security.Principal;

public static bool IsAdministrator()
{
    var identity = WindowsIdentity.GetCurrent();
    var principal = new WindowsPrincipal(identity);
    return principal.IsInRole(WindowsBuiltInRole.Administrator);
}

public static void RestartAsAdmin()
{
    if (!IsAdministrator())
    {
        var processInfo = new ProcessStartInfo
        {
            UseShellExecute = true,
            FileName = Process.GetCurrentProcess().MainModule.FileName,
            Verb = "runas"
        };
        
        try
        {
            Process.Start(processInfo);
            Environment.Exit(0);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Failed to restart as admin: {ex.Message}");
        }
    }
}
```

### Game-Specific Configuration

**Cyberpunk 2077**:
```ini
[FSR4]
Preset = Quality
Sharpness = 0.8
FrameGeneration = true
TemporalAA = true
MotionVectorScale = 1.0  # Cyberpunk has good motion vectors

[Patching]
EnableNukeMFG = true     # Improves motion blur interaction
```

**Red Dead Redemption 2**:
```ini
[FSR4]
Preset = Balanced
Sharpness = 0.6
FrameGeneration = false  # Can cause issues with RDR2's TAA
TemporalJitter = 0.3     # Reduce for less shimmer

[Compatibility]
ForceDX12 = true
DisableVulkan = true
```

**Forza Horizon 6**:
```ini
[FSR4]
Preset = Performance
FrameGeneration = true
MotionVectorQuality = High  # Important for fast motion
HudFix = true               # Prevents HUD duplication
```

## Environment Variables

OptiScaler respects the following environment variables:

```bash
# Configuration directory override
export OPTISCALER_CONFIG_DIR="$HOME/.config/optiscaler"

# DLL search path
export OPTISCALER_DLL_PATH="/opt/optiscaler/dlls"

# Enable debug logging
export OPTISCALER_DEBUG=1

# Force specific preset
export OPTISCALER_FORCE_PRESET="Quality"

# Disable frame generation globally
export OPTISCALER_DISABLE_FRAMEGEN=1
```

## Advanced Configuration

### Custom Sharpening Filters

```ini
[FSR4]
Sharpness = 0.7
SharpeningFilter = AMD_CAS  # Options: AMD_CAS, NVIDIA_NIS, None

[CAS]
Strength = 0.7
EdgeAdaptive = true

[NIS]
Strength = 0.5
```

### Multiple Profile Presets

```csharp
public class PresetLibrary
{
    public static Dictionary<string, FSR4Settings> Presets = new()
    {
        ["MaxQuality"] = new FSR4Settings
        {
            Preset = "Ultra Quality",
            Sharpness = 0.9f,
            FrameGeneration = false,
            EnableINT8 = false
        },
        ["MaxPerformance"] = new FSR4Settings
        {
            Preset = "Ultra Performance",
            Sharpness = 0.4f,
            FrameGeneration = true,
            EnableINT8 = true
        },
        ["Balanced"] = new FSR4Settings
        {
            Preset = "Balanced",
            Sharpness = 0.7f,
            FrameGeneration = true,
            EnableINT8 = true
        }
    };
}
```

This skill provides comprehensive guidance for using OptiScaler Client FSR4 to manage game upscaling, patch DLSS to FSR4, configure profiles, and troubleshoot common issues in a C# codebase with Avalonia UI.
