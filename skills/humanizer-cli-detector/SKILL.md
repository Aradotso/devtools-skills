---
name: humanizer-cli-detector
description: CLI tool that detects AI-written text patterns using 33 Wikipedia-documented signs, with draft checking and humanization prompts — zero dependencies
triggers:
  - "check my writing for AI patterns"
  - "scan this text for AI-generated tells"
  - "humanize this AI-written content"
  - "detect AI slop in my draft"
  - "show me AI writing patterns"
  - "remove AI tells from my text"
  - "make this text sound more human"
  - "check for ChatGPT writing patterns"
---

# humanizer-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

A terminal-based AI text detector that identifies 33 mechanical patterns in AI-written content. Works offline with zero dependencies. 87 KB binary (Windows x64) + Node.js wrapper for interactive sessions.

## What It Does

Detects AI writing patterns catalogued in Wikipedia's "Signs of AI writing" article:
- Overuse of em-dashes and semicolons
- "Not just X, it's Y" constructions
- Hedging language ("it's worth noting", "arguably")
- Manufactured enthusiasm and flattery
- Emoji in headings
- Bolded list lead-ins
- Chatbot leftovers ("as an AI language model")
- 26 other documented patterns

Provides before/after examples, draft checking, and prompt generation for manual rewriting.

## Installation

### Binary Only (Windows x64)

```powershell
# Download from releases
https://github.com/0xwilliamortiz/humanizer-cli/releases/download/2.9.2/humanizer-v2.9.2.zip

# Unpack and run
.\humanizer.exe
.\humanizer.exe show 14
.\humanizer.exe check draft.md
```

### With Node.js Wrapper (Interactive Mode)

```powershell
# Clone or download the project
cd humanizer-cli
npm install

# Launch interactive prompt
npx humanizer
```

The npm install downloads nothing (zero dependencies). It only registers the `humanizer` name for npx.

## Key Commands

### Show All Patterns

```powershell
.\humanizer.exe patterns
```

Lists all 33 patterns grouped by section:
- Punctuation (em-dashes, semicolons, curly quotes)
- Vocabulary (AI word list, hedging, padding)
- Structure (rhetorical openers, signposts, lists)
- Content patterns (flattery, manufactured enthusiasm)

### Show Individual Pattern

```powershell
.\humanizer.exe show 14
```

Displays full details for pattern #14 including:
- Pattern description
- Before example (AI-written)
- After example (human-written)
- Detection notes

### Check a Draft

```powershell
.\humanizer.exe check draft.md
.\humanizer.exe check article.txt --out report.txt
```

Scans text for detectable patterns. Catches **13 of 33 patterns** automatically:
- Em-dashes and semicolons
- Emoji in headings
- Curly quotes
- AI vocabulary words
- "Not just X, it's Y" structure
- Padding phrases
- Hedging language
- Bolded list lead-ins
- Chatbot leftovers
- Flattery
- Signpost phrases ("let's dive in")
- Rhetorical openers

Output format:
```
Pattern #7 (3 matches): padding
  Line 12: "it's important to note that the system"
  Line 45: "it goes without saying that users"
  Line 89: "the fact of the matter is that"
```

### Search Patterns

```powershell
.\humanizer.exe search hedging
.\humanizer.exe search emoji
```

Finds patterns matching a keyword.

### Generate Rewrite Prompt

```powershell
# Copy to clipboard
.\humanizer.exe prompt draft.md --copy

# Save to file
.\humanizer.exe prompt draft.md --out prompt.txt

# Print to stdout
.\humanizer.exe prompt
```

Generates a complete skill prompt you can paste into any chat (Claude, ChatGPT, etc.) to get manual rewriting without an API key.

### Installation Instructions

```powershell
.\humanizer.exe install
```

Shows how to install the skill into AI coding agents (Claude Code, Cursor, Codex).

### Diagnostic Info

```powershell
.\humanizer.exe doctor
```

Reports:
- Binary location and version
- SKILL.md source path
- Node.js version (if running via npx)
- Platform and architecture

## Interactive Mode (Node.js)

```powershell
npx humanizer
```

```
humanizer> show 14
humanizer> check draft.md
humanizer> search hedging
humanizer> prompt draft.md --copy
humanizer> exit
```

You can also pass a command directly:

```powershell
npx humanizer show 14
```

The command runs first, then the interactive prompt appears.

## Configuration

### Custom SKILL.md Location

```powershell
.\humanizer.exe check draft.md --skill custom/SKILL.md
```

SKILL.md resolution order:
1. Path from `--skill` flag
2. `SKILL.md` in current directory
3. `SKILL.md` next to the binary
4. Compiled-in copy (fallback)

### Disable Colors

```powershell
.\humanizer.exe patterns --no-color
```

### Output Redirection

```powershell
# Save to file
.\humanizer.exe check draft.md --out report.txt

# Both work
.\humanizer.exe check draft.md > report.txt
```

## Common Patterns

### Quick Draft Check

```javascript
// In a build script or pre-commit hook
const { execSync } = require('child_process');
const fs = require('fs');

const files = ['README.md', 'docs/guide.md'];

files.forEach(file => {
  if (fs.existsSync(file)) {
    try {
      execSync(`humanizer.exe check ${file}`, { stdio: 'inherit' });
    } catch (error) {
      console.error(`AI patterns found in ${file}`);
      process.exit(1);
    }
  }
});
```

### Automated Rewrite Prompt Generation

```javascript
const { execSync } = require('child_process');
const fs = require('fs');

const draft = 'article.md';
const promptFile = 'rewrite-prompt.txt';

// Generate prompt
execSync(`humanizer.exe prompt ${draft} --out ${promptFile}`);

// Read and use with your preferred LLM API
const prompt = fs.readFileSync(promptFile, 'utf8');

// Example with environment variable for API key
const apiKey = process.env.OPENAI_API_KEY;
// ... make API call with prompt
```

### Batch Processing

```javascript
const { execSync } = require('child_process');
const { readdirSync } = require('fs');

const markdownFiles = readdirSync('.')
  .filter(f => f.endsWith('.md'));

markdownFiles.forEach(file => {
  console.log(`\n=== Checking ${file} ===`);
  try {
    execSync(`humanizer.exe check ${file}`, { stdio: 'inherit' });
  } catch (error) {
    // Non-zero exit code means patterns detected
  }
});
```

### Pre-commit Hook

```javascript
// .git/hooks/pre-commit (make executable)
#!/usr/bin/env node

const { execSync } = require('child_process');

const stagedMd = execSync('git diff --cached --name-only --diff-filter=ACM')
  .toString()
  .split('\n')
  .filter(f => f.endsWith('.md'));

if (stagedMd.length === 0) process.exit(0);

console.log('Checking staged markdown for AI patterns...\n');

let foundPatterns = false;

stagedMd.forEach(file => {
  try {
    execSync(`humanizer.exe check ${file}`, { stdio: 'inherit' });
  } catch (error) {
    foundPatterns = true;
  }
});

if (foundPatterns) {
  console.error('\n❌ AI patterns detected. Review or bypass with --no-verify');
  process.exit(1);
}
```

## How It Works

```
humanizer-cli/
├── humanizer.exe         # 87 KB C binary, self-contained
├── SKILL.md              # Pattern database (Wikipedia sources)
├── package.json          # Zero dependencies
├── sources/
    ├── launch.mjs        # npx entry point
    ├── panel.mjs         # Interactive panel (Node)
    └── humanizer.cmd     # Launcher without Node
```

- **Binary**: Pure C program, links only against Windows system DLLs (kernel32, msvcrt, user32)
- **SKILL.md**: Embedded in binary as fallback, but external file takes precedence
- **No network**: Everything runs offline
- **No dependencies**: No npm packages, no bundled runtime

The panel UI (printed by Node) and the detection logic (C binary) are separate. You can replace the binary without breaking the panel, or run the binary without Node entirely.

## Pattern Categories

### Punctuation (Patterns 1-3)
- Em-dashes instead of commas
- Semicolons in casual writing
- Curly quotes (typographic quotes)

### Vocabulary (Patterns 4-7)
- AI word list (realm, embark, delve, etc.)
- Hedging ("arguably", "it's worth noting")
- Padding ("the fact of the matter is")
- Unnecessary intensity ("very unique")

### Structure (Patterns 8-15)
- "Not just X, it's Y"
- Lists with bolded lead-ins
- Emoji in headings
- Rhetorical questions as openers
- Signpost phrases ("let's dive in", "in today's world")

### Content (Patterns 16-33)
- Manufactured enthusiasm
- Flattery ("you're absolutely right")
- Chatbot artifacts ("as an AI language model")
- Inflated significance
- False balance
- Vague gestures at context
- And 11 more requiring human judgment

## Troubleshooting

### SmartScreen Warning

Windows SmartScreen blocks unsigned executables:
1. Click "More info"
2. Click "Run anyway"

Or download from GitHub releases (marked as "from a known publisher" after enough downloads).

### Antivirus Quarantine

Some AV tools flag unsigned binaries. Restore from quarantine:
```
Windows Security → Protection history → Restore
```

### Box Drawing Characters Broken

Console doesn't support UTF-8:
```powershell
chcp 65001
```

Or use Windows Terminal (handles UTF-8 by default).

### Exit Code Errors

```
(exit code 1)
```

Non-zero exit means:
- Patterns detected (when checking)
- File not found
- Invalid pattern number
- Malformed command

Check the error message above the exit code.

### Pattern Not Detected

13 patterns are regex-based. The other 20 require human judgment:
- Inflated significance
- False balance
- Manufactured controversy
- Confident vagueness
- etc.

Run `humanizer.exe patterns` to see which patterns are auto-detectable.

### SKILL.md Not Found

```powershell
.\humanizer.exe doctor
```

Shows which SKILL.md is being used. If custom location needed:
```powershell
.\humanizer.exe check draft.md --skill path/to/SKILL.md
```

### Colors Not Working

Force disable:
```powershell
.\humanizer.exe patterns --no-color
```

## Integration Examples

### With VS Code Task

`.vscode/tasks.json`:
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Check for AI patterns",
      "type": "shell",
      "command": "${workspaceFolder}/humanizer.exe",
      "args": ["check", "${file}"],
      "problemMatcher": [],
      "presentation": {
        "reveal": "always",
        "panel": "new"
      }
    }
  ]
}
```

### With GitHub Actions

`.github/workflows/check-ai.yml`:
```yaml
name: Check AI Patterns
on: [pull_request]
jobs:
  check:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Download humanizer
        run: |
          Invoke-WebRequest -Uri "https://github.com/0xwilliamortiz/humanizer-cli/releases/download/2.9.2/humanizer-v2.9.2.zip" -OutFile humanizer.zip
          Expand-Archive humanizer.zip
      - name: Check markdown files
        run: |
          Get-ChildItem -Recurse -Filter *.md | ForEach-Object {
            ./humanizer/humanizer.exe check $_.FullName
          }
```

### As npm Script

`package.json`:
```json
{
  "scripts": {
    "check:ai": "humanizer.exe check README.md",
    "check:docs": "for %f in (docs/*.md) do humanizer.exe check %f"
  }
}
```

## Exit Codes

- `0`: Success (no patterns detected, or command completed)
- `1`: Patterns detected / file not found / invalid input
- `2`: Invalid arguments

## Limitations

- **Windows x64 only**: Binary is platform-specific
- **13/33 patterns auto-detected**: Rest need human review
- **English text**: Patterns based on English AI writing habits
- **Markdown/text files**: Not designed for code or structured data

## Credits

- Pattern catalog: [blader/humanizer](https://github.com/blader/humanizer) skill
- Pattern research: [Wikipedia:WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup)

## License

MIT
