---
name: mac-cleaner-cli-disk-cleanup
description: Free disk space on macOS using an interactive CLI that clears caches, logs, Homebrew, Xcode junk, and more
triggers:
  - clean up disk space on my mac
  - free disk space using mac cleaner
  - remove caches and logs from macos
  - clean homebrew and xcode cache
  - uninstall mac apps completely
  - scan mac for cleanable files
  - clear browser cache and temporary files
  - delete old node_modules folders
---

# mac-cleaner-cli Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

`mac-cleaner-cli` is a TypeScript-based CLI tool that helps free up disk space on macOS by cleaning:
- System and user caches
- Browser caches (Chrome, Safari, Firefox, Arc)
- Temporary files
- Development caches (npm, yarn, pip, Xcode DerivedData, CocoaPods)
- Homebrew downloads and cache
- Docker images and containers
- System logs
- Trash
- Orphaned node_modules
- iOS backups, downloads, mail attachments (risky categories)

**Key features:**
- Interactive checkbox selection
- Drill-down file explorer for granular control
- Safe by default (risky items require `--risky` flag)
- App uninstaller with complete cleanup
- 100% offline, no telemetry
- Minimal dependencies

## Installation

### One-Time Usage (Recommended)

```bash
npx mac-cleaner-cli
```

### Global Installation

```bash
npm install -g mac-cleaner-cli
mac-cleaner-cli
```

### For Development

```bash
git clone https://github.com/guhcostan/mac-cleaner-cli.git
cd mac-cleaner-cli
bun install
bun run dev
```

## Key Commands

### Interactive Cleanup (Main Command)

```bash
# Basic interactive mode - scan and clean
npx mac-cleaner-cli

# Include risky categories (downloads, iOS backups, duplicates, large files)
npx mac-cleaner-cli --risky

# Force file picker for ALL categories
npx mac-cleaner-cli --risky -f

# Show absolute paths
npx mac-cleaner-cli -A

# Disable progress bars
npx mac-cleaner-cli --no-progress
```

**Interactive controls:**
- `↑↓` navigate items
- `←` go back
- `→` drill into category (file explorer)
- `space` toggle selection
- `a` select all
- `i` invert selection
- `⏎` submit/confirm

### App Uninstaller

```bash
# Uninstall apps completely with preferences and support files
npx mac-cleaner-cli uninstall
```

### Maintenance Tasks

```bash
# Flush DNS cache (may require sudo)
npx mac-cleaner-cli maintenance --dns

# Free purgeable space
npx mac-cleaner-cli maintenance --purgeable
```

### Category Management

```bash
# List all available cleaning categories
npx mac-cleaner-cli categories
```

### Configuration

```bash
# Initialize configuration file
npx mac-cleaner-cli config --init

# Show current configuration
npx mac-cleaner-cli config --show
```

### Backup Management

```bash
# List all backups
npx mac-cleaner-cli backup --list

# Clean old backups
npx mac-cleaner-cli backup --clean
```

### Help and Version

```bash
# Show version
npx mac-cleaner-cli --version
npx mac-cleaner-cli -V

# Show help
npx mac-cleaner-cli --help
npx mac-cleaner-cli -h
```

## Categories

### Safe Categories (Always Safe)

| Category | Description |
|----------|-------------|
| `trash` | Files in Trash bin |
| `temp-files` | Temporary files in /tmp and /var/folders |
| `browser-cache` | Chrome, Safari, Firefox, Arc cache |
| `homebrew` | Homebrew download cache |
| `docker` | Unused Docker images, containers, volumes |

### Moderate Categories (Generally Safe)

| Category | Description |
|----------|-------------|
| `system-cache` | Application caches in ~/Library/Caches |
| `system-logs` | System and application logs |
| `dev-cache` | npm, yarn, pip, Xcode DerivedData, CocoaPods |
| `node-modules` | Orphaned node_modules in old projects |

### Risky Categories (Requires `--risky` Flag)

| Category | Description |
|----------|-------------|
| `downloads` | Downloads older than 30 days |
| `ios-backups` | iPhone and iPad backup files |
| `mail-attachments` | Downloaded email attachments |
| `duplicates` | Duplicate files (keeps newest) |
| `large-files` | Files larger than 500MB |
| `language-files` | Unused language localizations |

## Common Usage Patterns

### Quick Disk Cleanup

```bash
# Run interactively, select common safe categories
npx mac-cleaner-cli
# Select: Trash, Browser Cache, Temporary Files, User Cache Files, Development Cache
```

### Developer Cleanup

```bash
# Clean development caches and orphaned node_modules
npx mac-cleaner-cli
# Focus on: Development Cache, node_modules folders
```

### Deep Clean with Risky Categories

```bash
# Include all categories including risky ones
npx mac-cleaner-cli --risky
# Review carefully before cleaning downloads, iOS backups, etc.
```

### Drill Down into Specific Folders

```bash
# Run interactive mode
npx mac-cleaner-cli

# Navigate to a category like "User Cache Files"
# Press → to open file explorer
# Select specific app caches to remove
# Press ← to go back, space to toggle, ⏎ to confirm
```

### CI/CD Usage

```bash
# Non-interactive cleanup (future feature)
# Currently, the tool is interactive-first
# For automated scripts, consider using with expect or similar tools
```

### Complete App Removal

```bash
# Remove app with all associated files
npx mac-cleaner-cli uninstall
# Select app from list
# Confirms removal of .app, preferences, caches, support files
```

## Configuration

The tool supports a configuration file for customizing behavior.

### Initialize Config

```bash
npx mac-cleaner-cli config --init
```

This creates a config file at `~/.mac-cleaner-cli/config.json` (exact location may vary).

### View Current Config

```bash
npx mac-cleaner-cli config --show
```

### Configuration Options

While the exact schema isn't fully documented in the README, typical options might include:

```json
{
  "defaultCategories": ["trash", "temp-files", "browser-cache"],
  "excludePaths": [
    "/Users/username/important-cache"
  ],
  "backupBeforeClean": true,
  "showAbsolutePaths": false
}
```

## Troubleshooting

### Permission Errors

**Problem:** "Permission denied" when cleaning certain files.

**Solution:**
```bash
# Some system files may require elevated permissions
# The tool generally runs as current user
# For DNS flush:
sudo npx mac-cleaner-cli maintenance --dns
```

### Categories Not Showing

**Problem:** Risky categories aren't visible.

**Solution:**
```bash
# Use the --risky flag
npx mac-cleaner-cli --risky
```

### File Picker Not Available

**Problem:** Can't drill down into a category.

**Solution:**
```bash
# Not all categories support file-level selection
# Supported: User Cache Files, Temporary Files, System Log Files, 
#            Development Cache, Browser Cache, Homebrew Cache
# Use -f to force file picker for all categories
npx mac-cleaner-cli -f
```

### Large Scan Times

**Problem:** Scanning takes a long time.

**Solution:**
- This is normal for Macs with many files
- Consider excluding large directories if you have custom config
- The tool scans common locations efficiently

### Accidental Deletion

**Problem:** Deleted something important.

**Solution:**
```bash
# Check if backups are enabled in config
npx mac-cleaner-cli backup --list

# Restore from Time Machine or system backups
# Always verify selections before confirming cleanup
```

### Docker Cleanup Issues

**Problem:** Docker containers still running after cleanup attempt.

**Solution:**
```bash
# Stop all containers first
docker stop $(docker ps -aq)

# Then run cleaner
npx mac-cleaner-cli
```

### Tool Not Found After Global Install

**Problem:** `mac-cleaner-cli: command not found`

**Solution:**
```bash
# Ensure npm global bin is in PATH
npm config get prefix
# Add to PATH: export PATH="$PATH:$(npm config get prefix)/bin"

# Or use npx
npx mac-cleaner-cli
```

## Development Integration

### Using in Scripts

```typescript
// Example: Potential programmatic usage (check actual API)
import { scan, clean } from 'mac-cleaner-cli';

// Note: The tool is primarily CLI-focused
// Check package exports for programmatic API availability
```

### Building from Source

```bash
git clone https://github.com/guhcostan/mac-cleaner-cli.git
cd mac-cleaner-cli

# Install dependencies
bun install

# Run in development
bun run dev

# Run tests
bun run test

# Lint code
bun run lint

# Build for production
bun run build
```

### Project Structure

```
mac-cleaner-cli/
├── src/               # TypeScript source code
├── tests/             # Test files
├── assets/            # Banner images, assets
├── package.json       # Dependencies and scripts
├── tsconfig.json      # TypeScript configuration
└── README.md          # Documentation
```

## Best Practices

1. **Always review before cleaning**: Use the interactive checkboxes to verify what will be deleted
2. **Start with safe categories**: Run without `--risky` first
3. **Use file explorer**: Press `→` on categories to select specific folders
4. **Keep backups**: Enable backup settings in config for critical data
5. **Regular maintenance**: Run weekly/monthly to prevent disk space issues
6. **Check app uninstaller**: Use built-in uninstaller instead of dragging to Trash
7. **Monitor freed space**: Note the summary after cleanup to track effectiveness

## Security Notes

- **100% offline**: No network requests, no telemetry
- **Open source**: All code available for audit at https://github.com/guhcostan/mac-cleaner-cli
- **No root required**: Runs as current user (except DNS flush with sudo)
- **Minimal dependencies**: Only 5 runtime dependencies, monitored by Socket.dev
- **Safe defaults**: Risky operations hidden behind `--risky` flag

## Platform Support

- **macOS only**: Designed specifically for macOS
- **Node.js required**: Check compatibility with `package.json` (typically Node.js 16+)
- **Windows alternative**: See `windows-cleaner-cli` for Windows support

## Related Commands

```bash
# Check disk usage before cleanup
df -h

# Check specific directory sizes
du -sh ~/Library/Caches/*

# Manual cleanup alternatives
brew cleanup --prune=all
docker system prune -a
rm -rf ~/Library/Developer/Xcode/DerivedData

# Check what's using disk space
ncdu /
```

## Additional Resources

- **GitHub Repository**: https://github.com/guhcostan/mac-cleaner-cli
- **npm Package**: https://www.npmjs.com/package/mac-cleaner-cli
- **Issues**: https://github.com/guhcostan/mac-cleaner-cli/issues
- **Discussions**: https://github.com/guhcostan/mac-cleaner-cli/discussions
- **License**: MIT
