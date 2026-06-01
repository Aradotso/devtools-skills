---
name: optiscaler-client-integration
description: Expert guidance for integrating OptiScaler Client, a C# Windows application that manages AI upscaling (DLSS/FSR/XeSS) and frame generation mods for games
triggers:
  - how do I use optiscaler client
  - integrate optiscaler with my game manager
  - add optiscaler functionality to my app
  - work with optiscaler client codebase
  - understand optiscaler client architecture
  - build features for optiscaler client
  - contribute to optiscaler client
  - extend optiscaler client functionality
---

# OptiScaler Client Integration

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OptiScaler Client is a C# Windows application that automates the installation and management of AI upscaling technologies (DLSS, FSR, XeSS) and frame generation mods for games. It scans game libraries across multiple platforms (Steam, Epic, GOG, etc.), downloads necessary components, and manages configuration profiles.

## Project Overview

**Repository**: `optiscalerclient/optiscaler-client`  
**Language**: C# (WPF/Windows Forms)  
**License**: MIT (note: README mentions GPL v3, but metadata shows MIT)  
**Platform**: Windows 10/11 (with limited Linux Steam support)

### Core Capabilities

- Multi-platform game library scanning (Steam, Epic, GOG, EA, Ubisoft, Battle.net, Xbox)
- Automatic download and installation of OptiScaler, Fakenvapi, and frame generation mods
- Profile-based INI configuration management
- DLL injection method selection (dxgi, winmm, d3d12, etc.)
- GPU detection and component selection logic
- Backup/restore system for game files
- Localization support (14 languages)

## Installation & Setup

### For Users

1. Download from releases:
```
https://github.com/optiscalerclient/optiscaler-client/releases/download/Optiscaler/optiscaler-client.zip
```

2. Extract and run `OptiscalerClient.exe` (self-contained, no .NET runtime needed)

### For Developers

```bash
# Clone repository
git clone https://github.com/optiscalerclient/optiscaler-client.git
cd optiscaler-client

# Open in Visual Studio or Rider
# Build configuration: .NET (likely .NET 6/7/8 based on modern C# features)
```

## Architecture Patterns

### Game Discovery System

The scanner system likely uses multiple provider classes:

```csharp
// Example scanner interface pattern
public interface IGameScanner
{
    Task<List<GameEntry>> ScanAsync(CancellationToken cancellationToken);
    string PlatformName { get; }
    bool IsEnabled { get; set; }
}

// Steam scanner example
public class SteamScanner : IGameScanner
{
    public string PlatformName => "Steam";
    
    public async Task<List<GameEntry>> ScanAsync(CancellationToken ct)
    {
        var games = new List<GameEntry>();
        
        // Read Steam library folders from libraryfolders.vdf
        string steamPath = GetSteamInstallPath();
        string vdfPath = Path.Combine(steamPath, "steamapps", "libraryfolders.vdf");
        
        // Parse VDF and iterate through library folders
        var libraries = ParseLibraryFolders(vdfPath);
        
        foreach (var library in libraries)
        {
            var acfFiles = Directory.GetFiles(
                Path.Combine(library, "steamapps"), 
                "appmanifest_*.acf"
            );
            
            foreach (var acfFile in acfFiles)
            {
                var game = ParseAppManifest(acfFile);
                if (game != null && !IsExcluded(game.Name))
                {
                    games.Add(game);
                }
            }
        }
        
        return games;
    }
    
    private string GetSteamInstallPath()
    {
        using var key = Registry.LocalMachine.OpenSubKey(
            @"SOFTWARE\WOW6432Node\Valve\Steam"
        );
        return key?.GetValue("InstallPath") as string 
            ?? @"C:\Program Files (x86)\Steam";
    }
}
```

### Component Download & Installation

```csharp
// Component manager pattern
public class ComponentManager
{
    private readonly HttpClient _httpClient;
    private readonly string _cacheDirectory;
    
    public ComponentManager(string cacheDir)
    {
        _cacheDirectory = cacheDir;
        _httpClient = new HttpClient();
        
        // Configure proxy if needed
        ConfigureProxy();
    }
    
    public async Task<string> DownloadOptiScalerAsync(
        string version, 
        bool isBeta,
        IProgress<double> progress,
        CancellationToken ct
    )
    {
        string cacheKey = $"optiscaler_{version}_{(isBeta ? "beta" : "stable")}";
        string cachePath = Path.Combine(_cacheDirectory, cacheKey);
        
        if (Directory.Exists(cachePath))
        {
            return cachePath;
        }
        
        // GitHub release URL
        string url = isBeta 
            ? $"https://github.com/optiscaler/OptiScaler/releases/download/beta-{version}/OptiScaler.zip"
            : $"https://github.com/optiscaler/OptiScaler/releases/download/v{version}/OptiScaler.zip";
        
        string zipPath = Path.Combine(_cacheDirectory, $"{cacheKey}.zip");
        
        // Download with progress
        using (var response = await _httpClient.GetAsync(url, HttpCompletionOption.ResponseHeadersRead, ct))
        {
            response.EnsureSuccessStatusCode();
            
            var totalBytes = response.Content.Headers.ContentLength ?? -1L;
            using var contentStream = await response.Content.ReadAsStreamAsync(ct);
            using var fileStream = new FileStream(zipPath, FileMode.Create, FileAccess.Write, FileShare.None);
            
            var buffer = new byte[8192];
            long totalRead = 0;
            int bytesRead;
            
            while ((bytesRead = await contentStream.ReadAsync(buffer, 0, buffer.Length, ct)) > 0)
            {
                await fileStream.WriteAsync(buffer, 0, bytesRead, ct);
                totalRead += bytesRead;
                
                if (totalBytes > 0)
                {
                    progress?.Report((double)totalRead / totalBytes * 100);
                }
            }
        }
        
        // Extract
        ZipFile.ExtractToDirectory(zipPath, cachePath);
        File.Delete(zipPath);
        
        return cachePath;
    }
    
    private void ConfigureProxy()
    {
        var proxyUrl = Environment.GetEnvironmentVariable("HTTP_PROXY") 
                    ?? Environment.GetEnvironmentVariable("HTTPS_PROXY");
        
        if (!string.IsNullOrEmpty(proxyUrl))
        {
            var proxy = new WebProxy(proxyUrl);
            var handler = new HttpClientHandler { Proxy = proxy };
            _httpClient = new HttpClient(handler);
        }
    }
}
```

### Game Installation Logic

```csharp
public class GameInstaller
{
    public async Task<InstallResult> InstallOptiScalerAsync(
        GameEntry game,
        InstallOptions options,
        CancellationToken ct
    )
    {
        // 1. Detect game structure
        string targetExe = options.ManualPath ?? DetectMainExecutable(game.InstallPath);
        string gameDir = Path.GetDirectoryName(targetExe);
        
        // 2. Create backup
        string backupDir = Path.Combine(gameDir, ".optiscaler_backup");
        Directory.CreateDirectory(backupDir);
        
        string injectionDll = options.InjectionMethod; // e.g., "dxgi.dll"
        string originalDll = Path.Combine(gameDir, injectionDll);
        
        if (File.Exists(originalDll))
        {
            File.Copy(originalDll, Path.Combine(backupDir, injectionDll), true);
        }
        
        // 3. Download components
        string optiScalerPath = await _componentManager.DownloadOptiScalerAsync(
            options.OptiScalerVersion,
            options.UseBeta,
            options.Progress,
            ct
        );
        
        // 4. Copy files
        CopyOptiScalerFiles(optiScalerPath, gameDir);
        
        // Copy injection DLL
        File.Copy(
            Path.Combine(optiScalerPath, "nvngx.dll"),
            Path.Combine(gameDir, injectionDll),
            overwrite: true
        );
        
        // 5. Install Fakenvapi if AMD/Intel GPU
        if (options.NeedsFakenvapi)
        {
            await InstallFakenaviAsync(gameDir, options.FakenaviVersion, ct);
        }
        
        // 6. Install frame generation if requested
        if (options.InstallFrameGen)
        {
            await InstallNukemFGAsync(gameDir, ct);
        }
        
        // 7. Apply profile
        if (!string.IsNullOrEmpty(options.ProfileName))
        {
            await ApplyProfileAsync(gameDir, options.ProfileName);
        }
        
        // 8. Configure OptiPatcher if needed
        if (options.InstallOptiPatcher)
        {
            await ConfigureOptiPatcherAsync(gameDir);
        }
        
        return new InstallResult 
        { 
            Success = true, 
            InstalledComponents = GetInstalledComponents(gameDir) 
        };
    }
    
    private string DetectMainExecutable(string installPath)
    {
        // UE5/Phoenix game detection
        var phoenixDirs = new[] { "Phoenix", "Engine/Binaries/Win64" };
        
        foreach (var subDir in phoenixDirs)
        {
            string path = Path.Combine(installPath, subDir);
            if (Directory.Exists(path))
            {
                var exes = Directory.GetFiles(path, "*.exe", SearchOption.TopDirectoryOnly)
                    .Where(f => !f.Contains("Crash") && !f.Contains("UnrealEditor"))
                    .ToList();
                
                if (exes.Any())
                {
                    return exes.First();
                }
            }
        }
        
        // Standard detection: largest .exe in root or bin folder
        var candidates = Directory.GetFiles(installPath, "*.exe", SearchOption.AllDirectories)
            .Where(f => !IsSystemExecutable(f))
            .OrderByDescending(f => new FileInfo(f).Length)
            .ToList();
        
        return candidates.FirstOrDefault();
    }
    
    private bool IsSystemExecutable(string path)
    {
        var excludeNames = new[] 
        { 
            "unins", "crash", "reporter", "launcher", 
            "redist", "vcredist", "directx", "unity" 
        };
        
        string fileName = Path.GetFileNameWithoutExtension(path).ToLower();
        return excludeNames.Any(e => fileName.Contains(e));
    }
}
```

### Profile Management

```csharp
public class ProfileManager
{
    private readonly string _profilesDirectory;
    
    public ProfileManager(string profilesDir)
    {
        _profilesDirectory = profilesDir;
        Directory.CreateDirectory(_profilesDirectory);
    }
    
    public Profile LoadProfile(string name)
    {
        string path = Path.Combine(_profilesDirectory, $"{name}.ini");
        
        if (!File.Exists(path))
        {
            throw new FileNotFoundException($"Profile '{name}' not found");
        }
        
        var profile = new Profile { Name = name };
        var parser = new IniParser();
        
        var data = parser.ReadFile(path);
        
        foreach (var section in data.Sections)
        {
            profile.Sections[section.SectionName] = section.Keys
                .ToDictionary(k => k.KeyName, k => k.Value);
        }
        
        return profile;
    }
    
    public void SaveProfile(Profile profile)
    {
        string path = Path.Combine(_profilesDirectory, $"{profile.Name}.ini");
        
        using var writer = new StreamWriter(path);
        
        foreach (var section in profile.Sections)
        {
            writer.WriteLine($"[{section.Key}]");
            
            foreach (var kvp in section.Value)
            {
                writer.WriteLine($"{kvp.Key}={kvp.Value}");
            }
            
            writer.WriteLine();
        }
    }
    
    public Profile CreateDefaultProfile()
    {
        return new Profile
        {
            Name = "OptiScaler Standard",
            Sections = new Dictionary<string, Dictionary<string, string>>
            {
                ["Upscaling"] = new()
                {
                    ["Enabled"] = "true",
                    ["Mode"] = "Quality",
                    ["Sharpness"] = "0.5"
                },
                ["FrameGeneration"] = new()
                {
                    ["Enabled"] = "false",
                    ["FrameInterpolation"] = "true"
                },
                ["Advanced"] = new()
                {
                    ["MotionVectors"] = "Auto",
                    ["DepthBuffer"] = "Auto",
                    ["HDR"] = "true"
                }
            }
        };
    }
}

public class Profile
{
    public string Name { get; set; }
    public Dictionary<string, Dictionary<string, string>> Sections { get; set; } = new();
}
```

### GPU Detection

```csharp
public class GpuDetector
{
    public List<GpuInfo> DetectGpus()
    {
        var gpus = new List<GpuInfo>();
        
        try
        {
            using var searcher = new ManagementObjectSearcher(
                "SELECT * FROM Win32_VideoController"
            );
            
            foreach (ManagementObject obj in searcher.Get())
            {
                var name = obj["Name"]?.ToString();
                var vendor = obj["AdapterCompatibility"]?.ToString();
                var driverVersion = obj["DriverVersion"]?.ToString();
                
                if (string.IsNullOrEmpty(name))
                    continue;
                
                var gpu = new GpuInfo
                {
                    Name = name,
                    Vendor = DetermineVendor(name, vendor),
                    DriverVersion = driverVersion,
                    IsDiscrete = IsDiscreteGpu(name)
                };
                
                gpus.Add(gpu);
            }
        }
        catch (Exception ex)
        {
            // Fallback to manual detection
            Console.WriteLine($"WMI detection failed: {ex.Message}");
        }
        
        // Sort: discrete GPUs first, then by vendor preference (NVIDIA > AMD > Intel)
        return gpus
            .OrderByDescending(g => g.IsDiscrete)
            .ThenBy(g => g.Vendor == GpuVendor.NVIDIA ? 0 : 
                         g.Vendor == GpuVendor.AMD ? 1 : 2)
            .ToList();
    }
    
    private GpuVendor DetermineVendor(string name, string vendor)
    {
        name = name.ToLower();
        vendor = vendor?.ToLower() ?? "";
        
        if (name.Contains("nvidia") || name.Contains("geforce") || name.Contains("rtx"))
            return GpuVendor.NVIDIA;
        
        if (name.Contains("amd") || name.Contains("radeon") || name.Contains("rx "))
            return GpuVendor.AMD;
        
        if (name.Contains("intel") || name.Contains("uhd") || name.Contains("iris"))
            return GpuVendor.Intel;
        
        return GpuVendor.Unknown;
    }
    
    private bool IsDiscreteGpu(string name)
    {
        name = name.ToLower();
        return !name.Contains("intel") || name.Contains("arc");
    }
    
    public bool NeedsFakenvapi(GpuInfo gpu)
    {
        return gpu.Vendor == GpuVendor.AMD || gpu.Vendor == GpuVendor.Intel;
    }
}

public enum GpuVendor { NVIDIA, AMD, Intel, Unknown }

public class GpuInfo
{
    public string Name { get; set; }
    public GpuVendor Vendor { get; set; }
    public string DriverVersion { get; set; }
    public bool IsDiscrete { get; set; }
}
```

## Configuration Management

### Application Settings

```csharp
public class AppSettings
{
    private readonly string _settingsPath;
    
    public AppSettings()
    {
        string appData = Environment.GetFolderPath(
            Environment.SpecialFolder.LocalApplicationData
        );
        string appDir = Path.Combine(appData, "OptiScalerClient");
        Directory.CreateDirectory(appDir);
        
        _settingsPath = Path.Combine(appDir, "settings.json");
    }
    
    public Settings Load()
    {
        if (!File.Exists(_settingsPath))
        {
            return new Settings();
        }
        
        string json = File.ReadAllText(_settingsPath);
        return JsonSerializer.Deserialize<Settings>(json) ?? new Settings();
    }
    
    public void Save(Settings settings)
    {
        var options = new JsonSerializerOptions 
        { 
            WriteIndented = true 
        };
        
        string json = JsonSerializer.Serialize(settings, options);
        File.WriteAllText(_settingsPath, json);
    }
}

public class Settings
{
    public string Language { get; set; } = "en";
    public bool ShowBetaVersions { get; set; } = false;
    public string DefaultOptiScalerVersion { get; set; } = "latest";
    public string DefaultInjectionMethod { get; set; } = "dxgi.dll";
    public string DefaultProfile { get; set; } = "OptiScaler Standard";
    public string PreferredGpuName { get; set; }
    public Dictionary<string, bool> EnabledScanners { get; set; } = new()
    {
        ["Steam"] = true,
        ["Epic"] = true,
        ["GOG"] = true,
        ["EA"] = true,
        ["Ubisoft"] = true,
        ["BattleNet"] = true,
        ["Xbox"] = true
    };
    public List<string> CustomScanPaths { get; set; } = new();
    public bool EnableAnimations { get; set; } = true;
    public string ProxyUrl { get; set; }
    public string ProxyUsername { get; set; }
    public string SteamGridDbApiKey { get; set; }
    public WindowState WindowState { get; set; } = new();
}

public class WindowState
{
    public int Width { get; set; } = 1200;
    public int Height { get; set; } = 800;
    public int X { get; set; } = -1;
    public int Y { get; set; } = -1;
    public bool Maximized { get; set; } = false;
}
```

## Common Usage Patterns

### Bulk Installation

```csharp
public async Task BulkInstallAsync(
    List<GameEntry> games,
    BulkInstallOptions options,
    IProgress<BulkProgress> progress,
    CancellationToken ct
)
{
    var results = new List<BulkInstallResult>();
    int completed = 0;
    
    foreach (var game in games)
    {
        if (ct.IsCancellationRequested)
            break;
        
        try
        {
            var installOptions = new InstallOptions
            {
                OptiScalerVersion = options.OptiScalerVersion,
                InjectionMethod = options.InjectionMethod,
                ProfileName = options.ProfileName,
                NeedsFakenvapi = _gpuDetector.NeedsFakenvapi(options.SelectedGpu),
                InstallFrameGen = options.InstallFrameGen,
                UseBeta = options.UseBeta
            };
            
            var result = await _installer.InstallOptiScalerAsync(
                game, 
                installOptions, 
                ct
            );
            
            results.Add(new BulkInstallResult
            {
                GameName = game.Name,
                Success = result.Success,
                Message = result.Success ? "Installed successfully" : result.ErrorMessage
            });
        }
        catch (Exception ex)
        {
            results.Add(new BulkInstallResult
            {
                GameName = game.Name,
                Success = false,
                Message = ex.Message
            });
        }
        
        completed++;
        progress?.Report(new BulkProgress
        {
            Current = completed,
            Total = games.Count,
            CurrentGame = game.Name,
            Results = results
        });
    }
}
```

### Technology Detection

```csharp
public class TechnologyDetector
{
    public GameTechnologies DetectTechnologies(string gameDirectory)
    {
        var tech = new GameTechnologies();
        
        // Search for DLSS
        var dlssDlls = Directory.GetFiles(gameDirectory, "nvngx_dlss.dll", SearchOption.AllDirectories);
        if (dlssDlls.Any())
        {
            tech.HasDLSS = true;
            tech.DLSSVersion = GetDllVersion(dlssDlls.First());
        }
        
        // Search for DLSS Frame Generation
        var dlssgDlls = Directory.GetFiles(gameDirectory, "nvngx_dlssg.dll", SearchOption.AllDirectories);
        if (dlssgDlls.Any())
        {
            tech.HasDLSSG = true;
            tech.DLSSGVersion = GetDllVersion(dlssgDlls.First());
        }
        
        // Search for FSR
        var fsrDlls = Directory.GetFiles(gameDirectory, "amd_fidelityfx_*", SearchOption.AllDirectories);
        if (fsrDlls.Any())
        {
            tech.HasFSR = true;
            tech.FSRVersion = ParseFSRVersion(fsrDlls.First());
        }
        
        // Search for XeSS
        var xessDlls = Directory.GetFiles(gameDirectory, "libxess.dll", SearchOption.AllDirectories);
        if (xessDlls.Any())
        {
            tech.HasXeSS = true;
            tech.XeSSVersion = GetDllVersion(xessDlls.First());
        }
        
        return tech;
    }
    
    private string GetDllVersion(string dllPath)
    {
        try
        {
            var versionInfo = FileVersionInfo.GetVersionInfo(dllPath);
            return versionInfo.FileVersion ?? "Unknown";
        }
        catch
        {
            return "Unknown";
        }
    }
}

public class GameTechnologies
{
    public bool HasDLSS { get; set; }
    public string DLSSVersion { get; set; }
    public bool HasDLSSG { get; set; }
    public string DLSSGVersion { get; set; }
    public bool HasFSR { get; set; }
    public string FSRVersion { get; set; }
    public bool HasXeSS { get; set; }
    public string XeSSVersion { get; set; }
}
```

## Troubleshooting

### Common Issues

**Game not detected during scan:**
- Verify platform scanner is enabled in Settings
- Add custom scan path if game is in non-standard location
- Use "Add Manually" and select game executable directly
- Check that game is not in exclusion list

**Download failures:**
- Check proxy configuration if behind corporate firewall
- Verify GitHub is accessible from your network
- Set environment variables: `HTTP_PROXY` and `HTTPS_PROXY`
- Check antivirus isn't blocking downloads

**Installation fails:**
- Ensure game is not running during installation
- Check that you have write permissions to game directory
- For protected system folders (Program Files), run as Administrator
- Verify sufficient disk space for components (~500MB per install)

**False positive antivirus warnings:**
- Add exception for `OptiscalerClient.exe` in Windows Defender
- Whitelist the cache directory (typically `%LOCALAPPDATA%\OptiScalerClient\cache`)
- Download only from official GitHub releases
- Verify file hash if concerned about tampering

### Debug Logging

```csharp
public class Logger
{
    private readonly string _logPath;
    
    public Logger()
    {
        string appData = Environment.GetFolderPath(
            Environment.SpecialFolder.LocalApplicationData
        );
        string logDir = Path.Combine(appData, "OptiScalerClient", "logs");
        Directory.CreateDirectory(logDir);
        
        _logPath = Path.Combine(
            logDir, 
            $"optiscaler_{DateTime.Now:yyyyMMdd_HHmmss}.log"
        );
    }
    
    public void Log(LogLevel level, string message, Exception ex = null)
    {
        string timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
        string logLine = $"[{timestamp}] [{level}] {message}";
        
        if (ex != null)
        {
            logLine += $"\n{ex}";
        }
        
        File.AppendAllText(_logPath, logLine + "\n");
        
        // Also output to debug console
        Debug.WriteLine(logLine);
    }
}

public enum LogLevel { Debug, Info, Warning, Error }
```

### Backup Recovery

```csharp
public class BackupManager
{
    public void RestoreBackup(string gameDirectory)
    {
        string backupDir = Path.Combine(gameDirectory, ".optiscaler_backup");
        
        if (!Directory.Exists(backupDir))
        {
            throw new DirectoryNotFoundException("No backup found for this game");
        }
        
        var backupFiles = Directory.GetFiles(backupDir);
        
        foreach (var backupFile in backupFiles)
        {
            string fileName = Path.GetFileName(backupFile);
            string targetPath = Path.Combine(gameDirectory, fileName);
            
            File.Copy(backupFile, targetPath, overwrite: true);
        }
        
        // Remove OptiScaler files
        DeleteOptiScalerFiles(gameDirectory);
        
        // Clean up backup directory
        Directory.Delete(backupDir, recursive: true);
    }
}
```

## API Integrations

### SteamGridDB Cover Art

```csharp
public class CoverArtFetcher
{
    private readonly HttpClient _httpClient;
    private readonly string _apiKey;
    private readonly string _cacheDir;
    
    public CoverArtFetcher(string apiKey, string cacheDir)
    {
        _apiKey = apiKey;
        _cacheDir = cacheDir;
        _httpClient = new HttpClient
        {
            BaseAddress = new Uri("https://www.steamgriddb.com/api/v2/")
        };
        
        if (!string.IsNullOrEmpty(_apiKey))
        {
            _httpClient.DefaultRequestHeaders.Add("Authorization", $"Bearer {_apiKey}");
        }
    }
    
    public async Task<string> GetCoverArtAsync(int steamAppId)
    {
        string cachedPath = Path.Combine(_cacheDir, $"{steamAppId}.jpg");
        
        if (File.Exists(cachedPath))
        {
            return cachedPath;
        }
        
        try
        {
            // Get game ID from Steam App ID
            var gameResponse = await _httpClient.GetAsync($"games/steam/{steamAppId}");
            
            if (!gameResponse.IsSuccessStatusCode)
            {
                return null;
            }
            
            var gameData = await gameResponse.Content.ReadAsStringAsync();
            var gameDoc = JsonDocument.Parse(gameData);
            
            if (!gameDoc.RootElement.GetProperty("success").GetBoolean())
            {
                return null;
            }
            
            int gameId = gameDoc.RootElement
                .GetProperty("data")
                .GetProperty("id")
                .GetInt32();
            
            // Get grid images
            var gridResponse = await _httpClient.GetAsync($"grids/game/{gameId}");
            
            if (!gridResponse.IsSuccessStatusCode)
            {
                return null;
            }
            
            var gridData = await gridResponse.Content.ReadAsStringAsync();
            var gridDoc = JsonDocument.Parse(gridData);
            
            var grids = gridDoc.RootElement.GetProperty("data");
            
            if (grids.GetArrayLength() == 0)
            {
                return null;
            }
            
            string imageUrl = grids[0].GetProperty("url").GetString();
            
            // Download image
            var imageBytes = await _httpClient.GetByteArrayAsync(imageUrl);
            await File.WriteAllBytesAsync(cachedPath, imageBytes);
            
            return cachedPath;
        }
        catch (Exception ex)
        {
            Debug.WriteLine($"Failed to fetch cover art: {ex.Message}");
            return null;
        }
    }
}
```

## Testing Patterns

### Unit Test Example

```csharp
[TestClass]
public class GpuDetectorTests
{
    [TestMethod]
    public void NeedsFakenvapi_NvidiaGpu_ReturnsFalse()
    {
        var detector = new GpuDetector();
        var gpu = new GpuInfo 
        { 
            Name = "NVIDIA GeForce RTX 4060", 
            Vendor = GpuVendor.NVIDIA 
        };
        
        bool needs = detector.NeedsFakenvapi(gpu);
        
        Assert.IsFalse(needs);
    }
    
    [TestMethod]
    public void NeedsFakenvapi_AmdGpu_ReturnsTrue()
    {
        var detector = new GpuDetector();
        var gpu = new GpuInfo 
        { 
            Name = "AMD Radeon RX 7900 XT", 
            Vendor = GpuVendor.AMD 
        };
        
        bool needs = detector.NeedsFakenvapi(gpu);
        
        Assert.IsTrue(needs);
    }
}
```

This skill covers the core architecture patterns, installation flows, and integration points for working with or extending OptiScaler Client. Use these patterns as a foundation when adding features or troubleshooting issues.
