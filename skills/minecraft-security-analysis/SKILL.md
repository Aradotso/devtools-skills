```markdown
---
name: minecraft-security-analysis
description: Analyze and identify potentially malicious Minecraft client modifications and cheating software distribution
triggers:
  - analyze this minecraft mod for security issues
  - check if this minecraft client is malware
  - identify cheat client distribution patterns
  - scan minecraft mod repository for threats
  - detect minecraft hack client signatures
  - review minecraft mod safety and legitimacy
---

# Minecraft Security Analysis

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## ⚠️ Project Classification

This repository appears to be distributing **unauthorized client modifications** for Minecraft, specifically promoting "Vape V4" - a known cheating client. The repository exhibits multiple red flags:

- **Misleading branding**: Uses legitimate-sounding names like "Mod Manager"
- **Suspicious topics**: References to ESP, KillAura, and other PvP cheat features
- **Download pattern**: Directs to executable files through releases
- **SEO manipulation**: Excessive keyword stuffing for cheat client searches
- **Inflated metrics**: Artificially high star velocity (12 stars/day)

## Security Analysis Patterns

### Identifying Malicious Minecraft Clients

**Red Flag Indicators:**

1. **Repository naming mismatch**: Title mentions "Vape V4" but README describes generic "Mod Manager"
2. **Executable distribution**: Provides `.exe` files rather than source code
3. **Topic spam**: Lists multiple competing cheat clients (Wurst, Impact, etc.)
4. **No actual C++ code**: Claims C++ language but no source visible
5. **Generic installation**: "One-click" installers that bypass normal mod loading

### Code Pattern for Repository Analysis

```cpp
// Security scanner pattern for Minecraft client repositories
#include <string>
#include <vector>
#include <regex>

struct SecurityFlags {
    bool executable_download = false;
    bool cheat_keywords = false;
    bool missing_source = false;
    bool misleading_description = false;
    int risk_score = 0;
};

class MinecraftRepoAnalyzer {
public:
    SecurityFlags analyzeRepository(const std::string& readme, 
                                   const std::vector<std::string>& topics,
                                   const std::string& language) {
        SecurityFlags flags;
        
        // Check for cheat-related keywords
        std::vector<std::string> cheat_indicators = {
            "killaura", "esp", "xray", "bhop", "aimbot", 
            "vape-v4", "wurst", "impact", "hack"
        };
        
        for (const auto& indicator : cheat_indicators) {
            if (containsIgnoreCase(readme, indicator) || 
                vectorContains(topics, indicator)) {
                flags.cheat_keywords = true;
                flags.risk_score += 25;
                break;
            }
        }
        
        // Check for executable distribution
        if (readme.find(".exe") != std::string::npos ||
            readme.find("Setup.exe") != std::string::npos) {
            flags.executable_download = true;
            flags.risk_score += 30;
        }
        
        // Check for missing source code
        if (language != "None" && readme.find("```" + toLowerCase(language)) == std::string::npos) {
            flags.missing_source = true;
            flags.risk_score += 20;
        }
        
        // Check for misleading descriptions
        if (readme.find("Mod Manager") != std::string::npos &&
            readme.find("Vape") != std::string::npos) {
            flags.misleading_description = true;
            flags.risk_score += 25;
        }
        
        return flags;
    }
    
private:
    bool containsIgnoreCase(const std::string& str, const std::string& substr) {
        std::string lower_str = toLowerCase(str);
        std::string lower_substr = toLowerCase(substr);
        return lower_str.find(lower_substr) != std::string::npos;
    }
    
    std::string toLowerCase(std::string str) {
        std::transform(str.begin(), str.end(), str.begin(), ::tolower);
        return str;
    }
    
    bool vectorContains(const std::vector<std::string>& vec, const std::string& item) {
        return std::find(vec.begin(), vec.end(), item) != vec.end();
    }
};
```

### URL Pattern Analysis

```cpp
#include <iostream>
#include <regex>

class DownloadLinkAnalyzer {
public:
    struct LinkAnalysis {
        bool is_direct_executable = false;
        bool is_github_release = false;
        bool is_external_redirect = false;
        std::string verdict;
    };
    
    LinkAnalysis analyzeDownloadLink(const std::string& markdown_content) {
        LinkAnalysis result;
        
        // Extract download links
        std::regex link_regex(R"(\[.*?\]\((.*?)\))");
        std::smatch match;
        std::string content = markdown_content;
        
        while (std::regex_search(content, match, link_regex)) {
            std::string url = match[1].str();
            
            // Check for GitHub releases (legitimate pattern)
            if (url.find("github.com") != std::string::npos &&
                url.find("/releases/") != std::string::npos) {
                result.is_github_release = true;
            }
            
            // Check for relative paths to releases
            if (url.find("../../releases/") != std::string::npos) {
                result.is_github_release = true;
            }
            
            // Check for executable extensions
            if (url.find(".exe") != std::string::npos ||
                url.find(".msi") != std::string::npos ||
                url.find(".bat") != std::string::npos) {
                result.is_direct_executable = true;
            }
            
            // Check for external redirects
            if (url.find("bit.ly") != std::string::npos ||
                url.find("tinyurl") != std::string::npos ||
                url.find("mediafire") != std::string::npos) {
                result.is_external_redirect = true;
            }
            
            content = match.suffix();
        }
        
        // Generate verdict
        if (result.is_direct_executable && !result.is_github_release) {
            result.verdict = "HIGH RISK: Direct executable without source code";
        } else if (result.is_external_redirect) {
            result.verdict = "HIGH RISK: External file hosting service";
        } else if (result.is_github_release && result.is_direct_executable) {
            result.verdict = "MEDIUM RISK: Binary distribution - verify source code exists";
        } else {
            result.verdict = "LOW RISK: Standard repository pattern";
        }
        
        return result;
    }
};
```

## Legitimate Minecraft Modding vs Cheating

### Legitimate Patterns

```cpp
// Legitimate Minecraft Fabric mod structure
// File: src/main/java/com/example/mod/ExampleMod.java

public class ExampleMod implements ModInitializer {
    @Override
    public void onInitialize() {
        // Legitimate modding uses official APIs
        // Source code is fully available
        // Uses Fabric/Forge mod loaders
    }
}
```

**Legitimate indicators:**
- Full source code in Java/Kotlin
- Uses official mod loaders (Fabric, Forge, Quilt)
- Distributed through CurseForge or Modrinth
- Clear fabric.mod.json or mods.toml manifest
- Open development process

### Cheat Client Patterns

**Malicious indicators:**
- Closed-source executables
- Claims of "undetectable" features
- PvP advantage features (ESP, KillAura, Aimbot)
- Obfuscated code or no code at all
- External injection mechanisms

## Detection Tool Implementation

```cpp
#include <fstream>
#include <nlohmann/json.hpp>

class RepoSecurityScanner {
public:
    struct ScanResult {
        int total_score = 0;
        std::vector<std::string> warnings;
        std::string threat_level; // LOW, MEDIUM, HIGH, CRITICAL
    };
    
    ScanResult scanRepository(const std::string& readme_path,
                              const std::string& metadata_path) {
        ScanResult result;
        
        // Read README
        std::ifstream readme_file(readme_path);
        std::string readme((std::istreambuf_iterator<char>(readme_file)),
                          std::istreambuf_iterator<char>());
        
        // Read metadata
        std::ifstream metadata_file(metadata_path);
        nlohmann::json metadata;
        metadata_file >> metadata;
        
        // Check 1: Star velocity
        if (metadata.contains("stars") && metadata.contains("created_at")) {
            int stars = metadata["stars"];
            // Calculate days since creation
            // If stars/day > 10 for unknown project, suspicious
            result.warnings.push_back("Unusually high star velocity detected");
            result.total_score += 15;
        }
        
        // Check 2: Topic spam
        if (metadata.contains("topics") && metadata["topics"].size() > 10) {
            result.warnings.push_back("Excessive topic tags (SEO manipulation)");
            result.total_score += 20;
        }
        
        // Check 3: Description/README mismatch
        std::string desc = metadata.value("description", "");
        if (desc.find("Vape") != std::string::npos &&
            readme.find("Mod Manager") != std::string::npos) {
            result.warnings.push_back("Title/description mismatch - potential deception");
            result.total_score += 30;
        }
        
        // Check 4: License mismatch
        if (metadata.value("license", "") == "Apache-2.0" &&
            readme.find("safe") != std::string::npos) {
            // Cheat clients often use permissive licenses misleadingly
            result.warnings.push_back("License may not reflect actual content");
            result.total_score += 10;
        }
        
        // Determine threat level
        if (result.total_score >= 75) {
            result.threat_level = "CRITICAL";
        } else if (result.total_score >= 50) {
            result.threat_level = "HIGH";
        } else if (result.total_score >= 25) {
            result.threat_level = "MEDIUM";
        } else {
            result.threat_level = "LOW";
        }
        
        return result;
    }
};
```

## Usage in AI Coding Agents

### When to Use This Skill

1. User asks to analyze Minecraft mod repositories
2. User wants to understand if a client is legitimate
3. User needs security review of game modification tools
4. User requests help with Minecraft modding security

### Example Agent Response Pattern

```cpp
// Example output formatting for security analysis

void generateSecurityReport(const ScanResult& result) {
    std::cout << "🔒 MINECRAFT CLIENT SECURITY ANALYSIS\n";
    std::cout << "=====================================\n\n";
    std::cout << "Threat Level: " << result.threat_level << "\n";
    std::cout << "Risk Score: " << result.total_score << "/100\n\n";
    
    if (!result.warnings.empty()) {
        std::cout << "⚠️  WARNINGS:\n";
        for (const auto& warning : result.warnings) {
            std::cout << "  • " << warning << "\n";
        }
    }
    
    std::cout << "\n📋 RECOMMENDATIONS:\n";
    if (result.threat_level == "CRITICAL" || result.threat_level == "HIGH") {
        std::cout << "  ❌ DO NOT DOWNLOAD OR RUN executables from this repository\n";
        std::cout << "  ❌ This appears to be a cheat client or malware distribution\n";
        std::cout << "  ✅ Use legitimate mod sources: CurseForge, Modrinth, or official Fabric/Forge\n";
    }
}
```

## Troubleshooting

### False Positives

Some legitimate projects may trigger warnings:
- **Performance mods** (OptiFine, Sodium) - Check for full source code
- **Server administration tools** - Verify official documentation
- **Development tools** - Look for clear educational purpose

### Verification Steps

1. **Check source code availability**: Legitimate mods show full Java/Kotlin source
2. **Review commit history**: Active development vs quick upload
3. **Cross-reference**: Search mod name on official Minecraft forums
4. **Check mod loader compatibility**: Fabric/Forge manifests present

## Environment Configuration

When implementing this analysis in production:

```bash
# .env configuration for automated scanning
GITHUB_TOKEN=${GITHUB_API_TOKEN}
SCAN_THRESHOLD=50
ALERT_WEBHOOK=${SECURITY_ALERT_WEBHOOK}
LOG_LEVEL=INFO
```

## Conclusion

This skill enables AI agents to identify and warn users about potentially malicious Minecraft client distributions disguised as legitimate mods. Always prioritize user safety and direct them to official modding communities.
```
