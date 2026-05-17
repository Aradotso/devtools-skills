---
name: devtools-hub-installer
description: Install and manage Windows development tools with DevTools Hub, a desktop app and CLI for provisioning developer environments
triggers:
  - install development tools on windows
  - setup developer environment with devtools hub
  - manage windows developer tools
  - provision development machine
  - install git node python docker on windows
  - configure development environment windows
  - use devtools hub cli
  - batch install developer tools
---

# DevTools Hub Installer

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

DevTools Hub is a Windows desktop application and CLI for installing and managing developer tools. It provides a unified interface for provisioning development machines with version control, runtimes, package managers, editors, databases, containers, and system dependencies.

## Installation

Download the latest release from [GitHub Releases](https://github.com/lszdeveloping/devtoolshub/releases):

```bash
# Download and run the installer
DevTools Hub Setup 1.2.0.exe
```

The installer requires Windows 10/11 (x64). Most operations require administrator permissions for system-level tool installation and PATH updates.

## Desktop Application

Launch DevTools Hub from the Start menu or desktop shortcut. The GUI provides:

- **Tool Discovery**: Browse available developer tools by category
- **Status Detection**: Real-time installed/not-installed status from registry PATH
- **Batch Installation**: Select multiple tools and install in parallel
- **Developer Profiles**: One-click installation of curated tool sets
- **Install Logs**: Troubleshooting output for failed installations

The app uses UAC elevation for install/uninstall operations and streams stdout progress in real-time.

## CLI Usage

After building the project, the CLI is available at `dist/cli/cli.js`. Link it globally or invoke directly:

```bash
# List all available tools
devtoolshub list

# Install single or multiple tools
devtoolshub install git
devtoolshub install node python docker

# Check installation status
devtoolshub status

# Show detailed diagnostics
devtoolshub diagnose

# Export current configuration
devtoolshub export-config > devtools-config.json

# Update installed tools (via winget)
devtoolshub update git
devtoolshub update --all

# Uninstall tools (UAC-elevated)
devtoolshub uninstall docker
```

## Available Tools

DevTools Hub includes installers for:

**Version Control**
- `git`, `github-desktop`, `git-lfs`

**CLI Tools**
- `github-cli`, `codex-cli`, `claude-cli`

**Runtimes**
- `node`, `python`, `go`, `rust`, `deno`, `bun`, `java`

**Package Managers**
- `yarn`, `maven`, `gradle`

**Editors**
- `vscode`

**Databases**
- `postgresql`, `mongodb`, `redis`, `mysql`, `mariadb`, `sqlserver-express`

**DevOps**
- `docker`, `docker-compose`

**API Testing**
- `postman`

**Web Server Stack**
- `wampserver`, `apache`, `php`, `phpmyadmin`, `adminer`, `xdebug`

**System Libraries**
- `vcredist` (Visual C++ Redistributables)

## Development Setup

Clone and build the project:

```bash
# Clone repository
git clone https://github.com/lszdeveloping/devtoolshub.git
cd devtoolshub

# Install dependencies
npm install

# Run in development mode
npm run dev

# Build renderer, main process, and CLI
npm run build

# Create distributable
npm run dist
```

Requires Node.js >= 18.

## Project Structure

```
src/
  cli/              # Command-line interface
  main/             # Electron main process and IPC handlers
  renderer/         # React application (Vite + Tailwind CSS)
  shared/           # Shared tool metadata and types
installers/
  windows/          # PowerShell installers (asar-unpacked)
resources/          # App icons and packaged assets
```

## Adding New Tools

To add a new tool to the catalog:

1. **Update Tool Metadata** (`src/shared/tools.ts`):

```typescript
export interface Tool {
  id: string;
  name: string;
  category: string;
  description: string;
  installer: string;  // PowerShell script filename
  wingetId?: string;  // For update support
}

export const tools: Tool[] = [
  // ... existing tools
  {
    id: 'newtool',
    name: 'New Tool',
    category: 'Runtimes',
    description: 'Description of the tool',
    installer: 'newtool.ps1',
    wingetId: 'Publisher.NewTool'
  }
];
```

2. **Create PowerShell Installer** (`installers/windows/newtool.ps1`):

```powershell
# newtool.ps1
param(
    [string]$InstallPath = "C:\Program Files\NewTool"
)

Write-Host "Installing New Tool..."

try {
    # Download installer
    $url = "https://example.com/newtool-installer.exe"
    $installer = "$env:TEMP\newtool-installer.exe"
    Invoke-WebRequest -Uri $url -OutFile $installer

    # Run installer
    Start-Process -FilePath $installer -ArgumentList "/S" -Wait

    # Add to PATH if needed
    $currentPath = [Environment]::GetEnvironmentVariable("Path", "Machine")
    if ($currentPath -notlike "*$InstallPath*") {
        [Environment]::SetEnvironmentVariable(
            "Path",
            "$currentPath;$InstallPath",
            "Machine"
        )
    }

    Write-Host "New Tool installed successfully"
    exit 0
} catch {
    Write-Error "Installation failed: $_"
    exit 1
}
```

3. **Update Tool Detection** (`src/main/detection.ts`):

```typescript
export async function detectInstalledTools(): Promise<Map<string, boolean>> {
  const installed = new Map<string, boolean>();
  
  for (const tool of tools) {
    try {
      // Check PATH, registry, or filesystem
      const isInstalled = await checkToolInstalled(tool.id);
      installed.set(tool.id, isInstalled);
    } catch {
      installed.set(tool.id, false);
    }
  }
  
  return installed;
}

async function checkToolInstalled(toolId: string): Promise<boolean> {
  // Check common installation indicators
  const checks = [
    () => checkPath(toolId),
    () => checkRegistry(toolId),
    () => checkFileSystem(toolId)
  ];
  
  for (const check of checks) {
    if (await check()) return true;
  }
  
  return false;
}
```

## Configuration Export

Export current tool configuration for reproducible environments:

```bash
devtoolshub export-config > my-dev-setup.json
```

Example output:

```json
{
  "version": "1.2.0",
  "platform": "win32",
  "installed": [
    "git",
    "node",
    "python",
    "docker",
    "vscode"
  ],
  "timestamp": "2026-05-17T12:34:56.789Z"
}
```

## Batch Installation

Install multiple tools from a profile or custom list:

```typescript
// src/renderer/profiles.ts
export interface Profile {
  id: string;
  name: string;
  description: string;
  tools: string[];
}

export const profiles: Profile[] = [
  {
    id: 'web-dev',
    name: 'Web Development',
    description: 'Essential tools for web development',
    tools: ['git', 'node', 'vscode', 'docker', 'postgresql']
  },
  {
    id: 'python-dev',
    name: 'Python Development',
    description: 'Python development environment',
    tools: ['git', 'python', 'vscode', 'postgresql']
  }
];
```

From CLI:

```bash
# Install all tools from a profile (requires custom implementation)
devtoolshub install git node vscode docker postgresql
```

## Error Handling

The install queue uses per-tool try/catch — one failure doesn't abort the batch:

```typescript
// src/main/installer.ts
export async function installTools(toolIds: string[]): Promise<InstallResult[]> {
  const results: InstallResult[] = [];
  
  for (const toolId of toolIds) {
    try {
      const tool = tools.find(t => t.id === toolId);
      if (!tool) {
        results.push({ toolId, success: false, error: 'Tool not found' });
        continue;
      }
      
      // Execute PowerShell installer with elevated privileges
      const scriptPath = path.join(installersDir, tool.installer);
      const result = await execElevated(scriptPath);
      
      results.push({ toolId, success: true, output: result.stdout });
    } catch (error) {
      results.push({
        toolId,
        success: false,
        error: error instanceof Error ? error.message : String(error)
      });
    }
  }
  
  return results;
}
```

## Troubleshooting

**Installation fails silently**
- Check install logs in the app's log viewer
- Run CLI with verbose output: `devtoolshub diagnose`
- Verify UAC elevation was granted

**Tool not detected after install**
- Restart the app to refresh PATH detection
- Manually check registry: `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment`
- Verify tool executable is in PATH: `where <tool-name>`

**PowerShell execution policy errors**
```powershell
# Allow script execution (run as Administrator)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
```

**Missing dependencies**
- Install Visual C++ Redistributables: `devtoolshub install vcredist`
- Ensure Windows 10/11 x64
- Check Node.js >= 18 for development builds

**Build failures**
```bash
# Clean and rebuild
rm -rf node_modules dist
npm install
npm run build
```

## Security

DevTools Hub implements hardened Electron security:

- Sandbox mode enabled: `sandbox: true`
- Content Security Policy enforced
- External navigation guard prevents malicious redirects
- UAC elevation for privileged operations
- Installers bundled in `asar.unpacked` (no runtime downloads)

All installer scripts should be reviewed before execution. The app does not download executables at runtime — all installers are packaged with the application.
