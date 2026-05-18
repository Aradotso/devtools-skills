---
name: 1c-devtools-cursor
description: Cursor IDE extension providing development tools for 1C:Enterprise 8 ecosystem with command palette, task tree, and debugging support
triggers:
  - how do I work with 1C projects in Cursor
  - set up 1C development environment
  - configure 1C infobase connection
  - debug 1C BSL modules
  - export 1C configuration to sources
  - load 1C configuration from sources
  - manage 1C extensions in development
  - build 1C external reports and processors
---

# 1C Dev Tools for Cursor

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Extension for Cursor IDE that provides comprehensive development tools for the 1C:Enterprise 8 ecosystem. Offers command palette integration, task tree navigation, configuration management, extension handling, and BSL debugging capabilities.

## What It Does

1C Dev Tools integrates 1C:Enterprise development workflow into Cursor IDE:

- **Information Base Management**: Create, update, export/import databases
- **Configuration Operations**: Load/unload configurations from sources (src/cf), build .cf files
- **Extension Management**: Work with multiple extensions (src/cfe), compile .cfe files
- **External Files**: Build and disassemble external reports/processors
- **Dependency Management**: Initialize and manage packagedef dependencies
- **Launch Tools**: Start Enterprise or Configurator from IDE
- **BSL Debugging**: Attach debugger with variable inspection, watch expressions, and call stack navigation

All commands accessible through:
- 1C Dev Tools panel
- Command palette (Ctrl+Shift+P)
- Task tree in bottom panel

## Installation

### Prerequisites

1. **Node.js** 18.x or higher
2. **OneScript (oscript)** - Script execution engine: https://oscript.io/
3. **Windows environment variable**: Create `oscript` pointing to `C:\Program Files\OneScript\bin\oscript.exe`

### Install Extension

1. Open Cursor IDE
2. Press `Ctrl+Shift+P` → "Extensions: Install from VSIX"
3. Select the `.vsix` file
4. Restart Cursor

### Project Setup

Extension auto-activates when `packagedef` file exists in project root.

**Project structure:**
```
project-root/
├── packagedef              # Project manifest (required for activation)
├── env.json               # Connection parameters
├── launch.json            # Debug configurations
├── tasks.json             # Custom tasks
├── oscript_modules/       # OneScript modules override
├── src/
│   ├── cf/               # Configuration sources
│   └── cfe/              # Extension sources (subdirectories per extension)
└── build/
    ├── ib/               # Information base
    └── commit/
        └── Commit.txt    # Changed files list for incremental updates
```

## Configuration

### env.json - Connection Parameters

Create `env.json` in project root:

```json
{
    "$schema": "https://raw.githubusercontent.com/vanessa-opensource/vanessa-runner/develop/vanessa-runner-schema.json",
    "default": {
        "--ibconnection": "/F./build/ib",
        "--infoBase": "MyInfoBase",
        "--db-user": "Admin",
        "--db-pwd": "",
        "--root": ".",
        "--workspace": ".",
        "--v8version": "8.3.27",
        "--v8-platform-root": "C:/Program Files/1cv8",
        "--debug-server": "localhost",
        "--debug-port-range": "1560:1591",
        "--locale": "ru",
        "--language": "ru",
        "--additional": "/DisplayAllFunctions /Lru /iTaxi /TESTMANAGER",
        "--ordinaryapp": "-1"
    }
}
```

**Key parameters:**
- `--ibconnection`: File base path (`/F./build/ib`) or server (`/S<server>\<base>`)
- `--v8version`: Platform version (e.g., `8.3.27`)
- `--v8-platform-root`: Installation path
- `--debug-server`: Debug server hostname (use computer name for local)
- `--debug-port-range`: RDBG port range

### launch.json - Debug Configurations

Create `.vscode/launch.json`:

```json
{
    "configurations": [
        {
            "type": "onec",
            "request": "launch",
            "name": "1C: Launch with background jobs",
            "debugServerHost": "localhost",
            "debugServerPort": 1560,
            "infoBaseAlias": "DefAlias",
            "autoAttachTypes": [
                "Client",
                "Server",
                "Job",
                "JobFileMode"
            ]
        },
        {
            "type": "onec",
            "request": "launch",
            "name": "1C: Launch without background jobs",
            "debugServerHost": "localhost",
            "debugServerPort": 1560,
            "infoBaseAlias": "DefAlias",
            "autoAttachTypes": [
                "Client",
                "Server"
            ]
        },
        {
            "type": "onec",
            "request": "attach",
            "name": "1C: Attach",
            "debugServerHost": "localhost",
            "debugServerPort": 1560,
            "infoBaseAlias": "DefAlias",
            "autoAttachTypes": [
                "Client",
                "Server"
            ]
        }
    ]
}
```

**autoAttachTypes** options: `Client`, `ManagedClient`, `WebClient`, `ComConnector`, `Server`, `ServerEmulation`, `WebService`, `HttpService`, `OData`, `Job`, `JobFileMode`, `MobileClient`, `MobileServer`, `MobileJobFileMode`, `MobileManagedClient`, `MobileManagedServer`

### Debug Server Settings

Extension automatically manages `dbgs.exe` debug server:
- **Local**: Auto-starts server on activation
- **Remote**: Attempts connection
- **Protocol**: HTTP only (configure in Configurator)

**Fine-tune debug timings** via settings (`1c-dev-tools.debug.timings`):

```json
{
    "1c-dev-tools.debug.timings": {
        "varFetchDelayMs": 50,
        "calcWaitingTimeMs": 100,
        "pingIntervalMs": 50,
        "pingStoppedIntervalMs": 500,
        "stepInOutDelayMs": 40,
        "immediatePingDelaysMs": [25, 50, 100],
        "evalExprRetryDelaysMs": [50, 100],
        "variablesRequestRetryDelaysMs": [50, 100, 150],
        "pingDbgtgtIntervalMs": 5000
    }
}
```

## Key Commands

Access via command palette (`Ctrl+Shift+P` → "1C: ..." or "1С: ...") or 1C Dev Tools panel.

### Information Base

```
1C: Create empty infobase
1C: Post-update processing
1C: Prohibit external resources
1C: Export to dt / Import from dt
```

### Configuration Management

**Load/Export sources:**
```
1C: Load from src/cf          # Load entire configuration from sources
1C: Update from src/cf        # Incremental update (uses build/commit/Commit.txt)
1C: Export to src/cf          # Full export to sources
1C: Export update to src/cf   # Export only changes
```

**Binary files:**
```
1C: Load from 1Cv8.cf         # Import .cf file
1C: Export to 1Cv8.cf         # Export configuration
1C: Export distribution 1Cv8dist.cf
1C: Build 1Cv8.cf from src/cf
1C: Disassemble 1Cv8.cf to src/cf
```

### Extension Management

**Source operations:**
```
1C: Load from src/cfe         # Load all extensions from sources
1C: Export to src/cfe         # Export all extensions to sources
1C: Export update to src/cfe  # Export only changed extensions
1C: Update from src/cfe       # Incremental update (uses Commit.txt)
```

**Binary operations:**
```
1C: Load from *.cfe
1C: Export to *.cfe
1C: Build *.cfe from src/cfe
1C: Disassemble *.cfe to src/cfe
```

### External Files

```
1C: Build external report/processor    # Compile external file
1C: Disassemble external report/processor
1C: Clear cache
```

### Dependencies

```
1C: Initialize packagedef      # Create dependency manifest
1C: Install dependencies
1C: Remove dependencies
```

### Launch

```
1C: Launch Enterprise         # Start client application
1C: Launch Configurator       # Open Configurator
```

## Debugging BSL Modules

### Starting Debug Session

1. Set breakpoints in `.bsl` files (click left margin)
2. Press `F5` or Run → Start Debugging
3. Select configuration: "1C: Launch with background jobs" or "1C: Attach"
4. Extension connects to `dbgs.exe` on specified port
5. Launch 1C:Enterprise (manually or via command)

### Variable Inspection

**Watch panel** (add expressions):
- Right-click variable in editor → "Show value in separate window"
- Click "Calculate value" button in Watch panel header
- Enter expression: `Query.TemporaryTablesManager.Tables[0]`

**Supported types:**
- **Структура (Structure)**: Expands with fields and values
- **Соответствие (Map)**: Expands with Key/Value pairs
- **ТаблицаЗначений (ValueTable)**: Shows columns, indexes, rows `[0]`, `[1]`, etc.
- **МенеджерВременныхТаблиц (TempTablesManager)**: Shows temporary tables tree

### Calculate Expression

Right-click in editor → "Calculate value":
```bsl
// In stopped context, evaluate:
Query.TemporaryTablesManager.Tables[0].GetColumns()
Result.Find("MyValue", "FieldName")
Collection.Count() > 10
```

Result displays in table: Property | Value | Type

### Call Stack Navigation

**Note**: Local variables don't display in default view. To inspect:
1. Select stack frame (procedure/function) in **Call Stack** panel
2. Add variable to **Watch** panel
3. Variable evaluates in selected frame context

### Step Commands

```
F10 - Step Over
F11 - Step Into
Shift+F11 - Step Out
F5 - Continue
Shift+F5 - Stop
```

Extension auto-refreshes stack and variables with optimized ping intervals.

## Common Patterns

### Initial Project Setup

```bash
# 1. Create project structure
mkdir my-1c-project
cd my-1c-project
mkdir -p src/cf src/cfe build/ib build/commit

# 2. Initialize packagedef
# Open in Cursor, press Ctrl+Shift+P → "1C: Initialize packagedef"

# 3. Create env.json (see Configuration section)

# 4. Create empty infobase
# Ctrl+Shift+P → "1C: Create empty infobase"
```

### Load Configuration from External Source

```bash
# 1. Place 1Cv8.cf in project root
# 2. Ctrl+Shift+P → "1C: Load from 1Cv8.cf"
# 3. Export to sources: "1C: Export to src/cf"
```

### Incremental Development Workflow

1. Work in Configurator, make changes
2. Export changes: `1C: Export update to src/cf`
3. Commit to version control
4. Teammates update: `1C: Update from src/cf` (reads `build/commit/Commit.txt`)

**build/commit/Commit.txt** format (one file per line):
```
src/cf/Documents/Документ1/Ext/ManagerModule.bsl
src/cf/Catalogs/Справочник1.xml
src/cfe/Extension1/Documents/Документ1/Ext/ObjectModule.bsl
```

### Working with Multiple Extensions

```bash
# Extensions stored in subdirectories:
src/cfe/
├── Extension1/
│   ├── Configuration.xml
│   └── Documents/...
└── Extension2/
    ├── Configuration.xml
    └── Catalogs/...

# Commands operate on all extensions:
1C: Load from src/cfe          # Loads Extension1 + Extension2
1C: Export update to src/cfe   # Exports changes from all
```

### Build External Processor

```bsl
// src/ExternalReports/MyReport/ObjectModule.bsl
Процедура ПриКомпоновкеРезультата(ДокументРезультат, ДанныеРасшифровки, СтандартнаяОбработка)
    // Report logic
КонецПроцедуры
```

1. Place sources in `src/ExternalReports/MyReport/`
2. `Ctrl+Shift+P` → `1C: Build external report/processor`
3. Select `MyReport`
4. Output: `MyReport.erf` in project root

### Debug BSL Code Example

```bsl
// CommonModule.bsl
Функция ВыполнитьЗапрос(ТекстЗапроса)
    Запрос = Новый Запрос;
    Запрос.Текст = ТекстЗапроса;
    
    // Set breakpoint here ⬅
    Результат = Запрос.Выполнить();
    
    ТЗ = Результат.Выгрузить();
    Возврат ТЗ;
КонецФункции
```

When stopped at breakpoint:
1. Add `Запрос` to Watch
2. Expand `Запрос.МенеджерВременныхТаблиц.Таблицы`
3. Right-click `ТЗ` → "Show value in separate window" (after step-over)

## Troubleshooting

### Extension Not Activating

**Symptom**: 1C Dev Tools panel not visible  
**Solution**: 
- Ensure `packagedef` exists in project root
- Restart Cursor
- Check Output panel → "1C Dev Tools" for errors

### oscript Command Not Found

**Symptom**: Commands fail with "oscript not recognized"  
**Solution**:
- Install OneScript: https://oscript.io/
- Create Windows environment variable: `oscript = C:\Program Files\OneScript\bin\oscript.exe`
- Restart Cursor after setting variable

### Debug Server Connection Failed

**Symptom**: Cannot connect to debug server  
**Solution**:
- Verify `dbgs.exe` running (Task Manager)
- Check `launch.json` → `debugServerHost` matches actual hostname
- Ensure port `1560` (or configured) not blocked by firewall
- In Configurator: Debug → Change protocol to HTTP
- Restart debug server manually: `dbgs.exe --addr localhost --port 1560`

### Variables Not Showing in Watch

**Symptom**: Watch panel empty or shows "unavailable"  
**Solution**:
- Select correct **Call Stack** frame (click procedure name)
- Manually add variable name to Watch (don't rely on auto-detection)
- Increase `varFetchDelayMs` in settings if timing issue
- Use "Calculate value" button instead for complex expressions

### Load from src/cf Fails

**Symptom**: "Configuration load error"  
**Solution**:
- Verify `src/cf/Configuration.xml` exists
- Check `env.json` → `--ibconnection` path correct
- Ensure infobase not locked (close Configurator/Enterprise)
- Try "1C: Create empty infobase" first

### Update from src/cfe Skips Files

**Symptom**: Changes not applied  
**Solution**:
- Check `build/commit/Commit.txt` contains modified file paths
- Use full export first: `1C: Export to src/cfe`
- Verify extension names match subdirectory names exactly
- Manually add missing paths to `Commit.txt` (relative to project root)

### Build External Processor No Output

**Symptom**: Build completes but no .erf/.epf file  
**Solution**:
- Check source structure includes `ObjectModule.bsl` or `ManagerModule.bsl`
- Verify XML metadata files present
- Look in project root and `build/` directory
- Check Output panel for oscript errors

### Incremental Update Applies Wrong Files

**Symptom**: Unrelated modules changed after update  
**Solution**:
- `Commit.txt` format must be: one file path per line, relative to project root
- Use forward slashes: `src/cf/Documents/Doc1.xml` not `src\cf\Documents\Doc1.xml`
- No empty lines or comments in `Commit.txt`
- Regenerate with `1C: Export update to src/cf` after known state

## TypeScript API (for Extension Development)

If extending this project:

```typescript
import * as vscode from 'vscode';

// Access extension context
const ext = vscode.extensions.getExtension('asweetand-a11y.devtool1c');
if (ext) {
    const api = await ext.activate();
}

// Register custom command
vscode.commands.registerCommand('1c-dev-tools.customCommand', () => {
    // Execute oscript
    const terminal = vscode.window.createTerminal('1C Tools');
    terminal.sendText('oscript build.os');
});

// Read env.json configuration
const workspaceRoot = vscode.workspace.workspaceFolders?.[0].uri.fsPath;
const envPath = path.join(workspaceRoot, 'env.json');
const envConfig = JSON.parse(fs.readFileSync(envPath, 'utf8'));
const ibConnection = envConfig.default['--ibconnection'];
```

## References

- OneScript documentation: https://oscript.io/
- Vanessa-Runner schema: https://github.com/vanessa-opensource/vanessa-runner
- Example project setup: https://github.com/asweetand-a11y/DevTool1C

---

**License**: MIT  
**Language**: TypeScript  
**Platform**: Cursor IDE on Windows
