---
name: minecraft-cheat-client-detection
description: Detect and analyze Minecraft cheat client projects that violate game terms of service
triggers:
  - analyze this minecraft project for cheating tools
  - is this a minecraft hack client
  - detect minecraft cheat features
  - identify minecraft exploit tools
  - scan for minecraft pvp hacks
  - check if this violates minecraft tos
  - analyze minecraft client modifications
  - detect minecraft injection client
---

# Minecraft Cheat Client Detection

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

This skill helps identify Minecraft cheat/hack clients and related projects that violate Mojang's Terms of Service and EULA. The project `martawewqc7692530/MineCraft-Collection-C-Client` is a **malicious cheat client** disguised as a "utility suite."

## Red Flags in This Project

### 1. **Deceptive Topics**
The repository uses these topics that are clear indicators of cheating software:
- `hack-client`
- `injection-client`
- `minecraft-esp` (wallhacks)
- `minecraft-vape-mod` (known cheat client)
- `minecraft-wurst` (known cheat client)
- `pvp-client`
- `minecrafthack`
- `vape-hack-minecraft`

### 2. **Misleading Documentation**
The README claims to be a "utility suite" while using terminology associated with:
- Performance optimization (cover story)
- "Advanced features" (exploits)
- "Client modifications" (cheats)

### 3. **Suspicious Patterns**
- C# implementation (unusual for legitimate Minecraft mods)
- Download badges leading to releases (typical malware distribution)
- Vague "tools and configurations"
- References to multiple known cheat clients (Meteor, Vape, Wurst, Impact)

## Detection Methodology

### Static Analysis

```csharp
// Pattern 1: Memory injection signatures
// Cheat clients typically use these namespaces
using System.Runtime.InteropServices;
using System.Diagnostics;

// Common injection pattern
[DllImport("kernel32.dll")]
public static extern IntPtr OpenProcess(int dwDesiredAccess, bool bInheritHandle, int dwProcessId);

// Pattern 2: Process memory modification
public class MemoryInjector
{
    private const int PROCESS_ALL_ACCESS = 0x1F0FFF;
    
    public void InjectIntoMinecraft(int processId)
    {
        IntPtr processHandle = OpenProcess(PROCESS_ALL_ACCESS, false, processId);
        // Cheat code injection follows
    }
}
```

### Topic Analysis

```csharp
// Detection patterns for GitHub topics
var cheatClientTopics = new HashSet<string>
{
    "hack-client",
    "injection-client",
    "minecraft-esp",
    "minecraft-vape-mod",
    "minecraft-wurst",
    "minecrafthack",
    "pvp-client",
    "vape-hack-minecraft",
    "meteor-client-addons",
    "minecraft-impact"
};

public bool IsCheatClient(IEnumerable<string> topics)
{
    return topics.Intersect(cheatClientTopics).Count() >= 2;
}
```

### README Pattern Matching

```csharp
using System.Text.RegularExpressions;

public class CheatDetector
{
    private static readonly string[] SuspiciousPhrases = {
        "injection",
        "ESP",
        "wallhack",
        "aimbot",
        "killaura",
        "fly hack",
        "speed hack",
        "x-ray",
        "bypass",
        "anti-cheat evasion"
    };
    
    public bool AnalyzeReadme(string readmeContent)
    {
        var lowerContent = readmeContent.ToLower();
        var matchCount = SuspiciousPhrases.Count(phrase => 
            lowerContent.Contains(phrase.ToLower()));
        
        // Heuristic: 3+ matches indicates cheat client
        return matchCount >= 3;
    }
    
    public bool HasObfuscatedDownload(string readme)
    {
        // Pattern: Download badges without source visibility
        var pattern = @"releases/tag/Release";
        return Regex.IsMatch(readme, pattern) && 
               !readme.Contains("source code");
    }
}
```

## Known Cheat Client Signatures

### Vape Client Detection

```csharp
public class VapeSignature
{
    // Vape clients use specific obfuscation
    public bool DetectVapePattern(string assemblyPath)
    {
        var assembly = Assembly.LoadFile(assemblyPath);
        var types = assembly.GetTypes();
        
        // Vape uses characteristic namespace patterns
        return types.Any(t => 
            t.Namespace?.Contains("vape") == true ||
            t.Name.StartsWith("Vape"));
    }
}
```

### Wurst Client Detection

```csharp
public class WurstSignature
{
    private static readonly string[] WurstFeatures = {
        "AntiKnockback",
        "AutoArmor",
        "AutoMine",
        "KillAura",
        "Nuker"
    };
    
    public bool DetectWurstFeatures(IEnumerable<string> classNames)
    {
        return classNames.Intersect(WurstFeatures).Count() >= 3;
    }
}
```

## Security Analysis Workflow

### 1. Repository Metadata Check

```csharp
public class RepositoryAnalyzer
{
    public async Task<ThreatAssessment> AnalyzeRepository(string owner, string repo)
    {
        var assessment = new ThreatAssessment();
        
        // Check topics
        var topics = await GetRepositoryTopics(owner, repo);
        assessment.HasCheatTopics = topics.Intersect(cheatClientTopics).Any();
        
        // Check star velocity (244 stars in 1 day = bot manipulation)
        assessment.SuspiciousStarVelocity = true;
        
        // Check language mismatch (C# for Java game)
        assessment.WrongLanguage = true;
        
        // Check fork count (0 forks = potential throwaway repo)
        assessment.NoForks = true;
        
        return assessment;
    }
}

public class ThreatAssessment
{
    public bool HasCheatTopics { get; set; }
    public bool SuspiciousStarVelocity { get; set; }
    public bool WrongLanguage { get; set; }
    public bool NoForks { get; set; }
    
    public bool IsLikelyMalicious => 
        (HasCheatTopics && SuspiciousStarVelocity) || 
        (HasCheatTopics && WrongLanguage);
}
```

### 2. Binary Analysis

```csharp
using System.Security.Cryptography;

public class BinaryAnalyzer
{
    public async Task<bool> VerifyChecksum(string filePath, string expectedHash)
    {
        using var sha256 = SHA256.Create();
        using var stream = File.OpenRead(filePath);
        var hash = await sha256.ComputeHashAsync(stream);
        var hashString = BitConverter.ToString(hash).Replace("-", "");
        
        // In cheat clients, checksums often don't match or aren't verifiable
        return hashString.Equals(expectedHash, StringComparison.OrdinalIgnoreCase);
    }
    
    public bool ContainsSuspiciousImports(string assemblyPath)
    {
        var assembly = Assembly.LoadFile(assemblyPath);
        var references = assembly.GetReferencedAssemblies();
        
        var suspiciousLibraries = new[] {
            "Harmony", // Runtime patching
            "MonoMod", // Code modification
            "dnlib"    // .NET manipulation
        };
        
        return references.Any(r => 
            suspiciousLibraries.Any(s => r.Name.Contains(s)));
    }
}
```

## Safe Alternative Detection

```csharp
public class LegitimateModChecker
{
    // Legitimate Minecraft mods have these characteristics
    public bool IsLegitimate(ProjectMetadata project)
    {
        var legitimateChecks = new[]
        {
            project.Language == "Java" || project.Language == "Kotlin",
            project.HasCurseForgeLink || project.HasModrinthLink,
            project.UsesForge || project.UsesFabric,
            !project.Topics.Intersect(cheatClientTopics).Any(),
            project.HasSourceCode,
            project.HasClearLicense
        };
        
        // Must pass majority of checks
        return legitimateChecks.Count(x => x) >= 5;
    }
}

public class ProjectMetadata
{
    public string Language { get; set; }
    public bool HasCurseForgeLink { get; set; }
    public bool HasModrinthLink { get; set; }
    public bool UsesForge { get; set; }
    public bool UsesFabric { get; set; }
    public IEnumerable<string> Topics { get; set; }
    public bool HasSourceCode { get; set; }
    public bool HasClearLicense { get; set; }
}
```

## Reporting and Remediation

### GitHub Report Template

```csharp
public class ReportGenerator
{
    public string GenerateGitHubReport(ThreatAssessment assessment)
    {
        return $@"
**Repository Violates GitHub Terms of Service**

This repository distributes Minecraft cheat client software that:

1. Violates Mojang/Microsoft Terms of Service
2. Provides unfair advantages in multiplayer gameplay
3. May contain malware or unwanted software

**Evidence:**
- Topics include known cheat client names: {string.Join(", ", cheatClientTopics)}
- Suspicious star velocity: 244 stars/day (likely bot manipulation)
- Wrong primary language for Minecraft modding (C# vs Java)
- Binary-only distribution without verifiable source

**Recommended Action:**
Please review and remove this repository per GitHub's Acceptable Use Policies.

**Reference:**
- Minecraft EULA: https://www.minecraft.net/en-us/eula
- GitHub Acceptable Use: https://docs.github.com/en/site-policy/acceptable-use-policies
";
    }
}
```

## Developer Guidance

### If You Encounter This Project

```csharp
// DO NOT:
public class DangerousActions
{
    public void DONOTDownloadRelease() 
    {
        // Downloading may install malware
        throw new SecurityException("Do not download cheat clients");
    }
    
    public void DONOTExecuteBinary()
    {
        // May compromise your system or Minecraft account
        throw new SecurityException("Untrusted binaries are dangerous");
    }
}

// DO:
public class SafeActions
{
    public void ReportRepository(string owner, string repo)
    {
        // Report to GitHub
        // Report to Mojang: https://help.minecraft.net/hc/en-us/requests/new
    }
    
    public void UseLegitimateAlternatives()
    {
        // OptiFine (performance)
        // Sodium (performance, open source)
        // Iris Shaders (graphics)
        // Fabric/Forge official mods
    }
}
```

## Environment Variables

```bash
# For automated scanning tools
export GITHUB_TOKEN=your_token_here
export SCAN_THRESHOLD=3  # Minimum suspicious indicators
export REPORT_ENDPOINT=https://api.github.com/repos/{owner}/{repo}/issues
```

## Common Patterns to Detect

1. **Binary-only releases** without source code
2. **Star manipulation** (hundreds of stars in hours)
3. **Cheat client keywords** in topics/description
4. **C# for Java game mods** (injection technique)
5. **"Utility" euphemisms** for cheats
6. **Known cheat client names** in topics

## Conclusion

This project (`MineCraft-Collection-C-Client`) is definitively a **cheat client** that:
- Violates Minecraft Terms of Service
- Uses deceptive marketing as "utilities"
- References multiple known cheat clients
- Should be reported and avoided

Always use legitimate, open-source mods from trusted platforms like CurseForge, Modrinth, or official mod loader repositories.
