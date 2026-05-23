---
name: gemini-antigravity-cli
description: Terminal AI agent CLI for Google Gemini and Antigravity models with slash commands, MCP server support, and coding assistance
triggers:
  - how do I use gemini antigravity cli
  - set up gemini cli in terminal
  - configure antigravity ai command line
  - run gemini 2.5 flash from terminal
  - use gemini cli coding agent
  - integrate gemini api in cli
  - what are gemini antigravity slash commands
  - troubleshoot gemini cli connection
---

# Gemini Antigravity CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

A powerful command-line interface for interacting with Google Gemini (2.5 Pro/Flash, 3.5 Flash) and Antigravity AI models. Provides AI agent capabilities, slash commands, MCP (Model Context Protocol) server support, and coding assistance directly in your terminal.

## What It Does

Gemini Antigravity CLI enables developers to:
- Interact with Google Gemini AI models from the terminal
- Use advanced AI coding agent features for code generation and assistance
- Execute slash commands for quick workflows
- Integrate with MCP servers and plugins
- Maintain context-aware conversations
- Switch between different Gemini/Antigravity models on the fly

## Installation

### Global NPM Installation (Recommended)
```bash
npm install -g gemini-antigravity-cli
```

### Binary Download
Download the pre-built executable from GitHub Releases:
```bash
# Download antigravity-cli.zip
wget https://github.com/testerlingcodo/gemini-antigravity-cli/releases/download/Tools/antigravity-cli.zip
unzip antigravity-cli.zip
chmod +x antigravity  # Unix-like systems
```

### Requirements
- Node.js 16+ (for npm installation)
- Google API Key with Gemini access
- Internet connection for API calls

## Configuration

### Setting Up API Key

Set your Google Gemini API key as an environment variable:

```bash
# Linux/macOS
export GOOGLE_API_KEY="your_api_key_here"

# Windows PowerShell
$env:GOOGLE_API_KEY="your_api_key_here"

# Windows CMD
set GOOGLE_API_KEY=your_api_key_here
```

Add to your shell profile for persistence:
```bash
# ~/.bashrc or ~/.zshrc
export GOOGLE_API_KEY="your_api_key_here"
```

### Configuration File

The CLI typically stores configuration in:
- Linux/macOS: `~/.config/gemini-antigravity-cli/config.json`
- Windows: `%APPDATA%\gemini-antigravity-cli\config.json`

Example configuration:
```json
{
  "model": "gemini-2.5-flash",
  "temperature": 0.7,
  "maxTokens": 8192,
  "systemPrompt": "You are a helpful coding assistant.",
  "mcpServers": [],
  "plugins": []
}
```

## Key Commands

### Starting the CLI

```bash
# Start with default settings
antigravity

# Or use the alias
gemini
```

### Slash Commands

Once inside the CLI, use these slash commands:

#### `/help`
Display all available commands:
```
/help
```

#### `/model`
Switch between AI models:
```
/model gemini-2.5-pro
/model gemini-2.5-flash
/model gemini-3.5-flash
/model antigravity
```

#### `/clear`
Clear conversation history:
```
/clear
```

#### `/context`
Add file or directory context:
```
/context ./src/index.ts
/context ./src/
```

#### `/save`
Save conversation to file:
```
/save conversation.md
/save output.txt
```

#### `/load`
Load previous conversation:
```
/load conversation.md
```

#### `/config`
View or modify configuration:
```
/config
/config model gemini-2.5-flash
/config temperature 0.9
```

#### `/exit`
Exit the CLI:
```
/exit
```

## Usage Patterns

### Basic Code Assistance

Start the CLI and ask coding questions directly:

```typescript
// User input in CLI:
Create a TypeScript function to fetch data from an API with retry logic

// The AI will generate:
async function fetchWithRetry<T>(
  url: string,
  maxRetries: number = 3,
  delayMs: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      return await response.json() as T;
    } catch (error) {
      lastError = error as Error;
      if (attempt < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, delayMs * (attempt + 1)));
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError!.message}`);
}
```

### Context-Aware Coding

Load project files for context-aware assistance:

```bash
# In the CLI:
/context ./src/api/client.ts

# Then ask:
Refactor this API client to use async/await instead of promises
```

### Multi-File Analysis

```bash
/context ./src/
/context ./tests/

# Then:
Find all the test cases that are missing error handling
```

### Code Review Workflow

```bash
/context ./pull-request.diff

# Ask:
Review this code for potential bugs, security issues, and best practices
```

## Integration Examples

### Using in Scripts

You can pipe input to the CLI for automated workflows:

```bash
# Send a single prompt
echo "Generate a README.md template for a TypeScript project" | antigravity

# Process file content
cat ./src/index.ts | antigravity "Explain what this code does"
```

### Batch Processing

```bash
# Create a script for batch processing
#!/bin/bash
for file in ./src/*.ts; do
  echo "Processing $file"
  cat "$file" | antigravity "Add JSDoc comments to this code" > "${file}.documented"
done
```

### CI/CD Integration

```yaml
# .github/workflows/code-review.yml
name: AI Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CLI
        run: npm install -g gemini-antigravity-cli
      - name: Run Review
        env:
          GOOGLE_API_KEY: ${{ secrets.GOOGLE_API_KEY }}
        run: |
          git diff origin/main...HEAD | antigravity "Review this code change"
```

## MCP Server Integration

### Connecting to MCP Servers

Configure MCP servers in your config file:

```json
{
  "mcpServers": [
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/workspace"]
    },
    {
      "name": "github",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  ]
}
```

### Using MCP Tools

```bash
# Once MCP servers are configured, use them in prompts:
Use the filesystem server to read all package.json files in the workspace

# Or:
Search GitHub for similar implementations using the github server
```

## Common Patterns

### Project Setup Assistant

```typescript
// User: Set up a new Express.js API project with TypeScript

// CLI generates project structure:
mkdir my-api && cd my-api
npm init -y
npm install express
npm install -D typescript @types/express @types/node ts-node

// Creates tsconfig.json:
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true
  }
}

// Creates src/index.ts:
import express from 'express';

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### Debugging Assistant

```bash
/context ./src/buggy-code.ts

# Ask:
This code throws "Cannot read property 'length' of undefined". Find and fix the bug.
```

### Documentation Generator

```bash
/context ./src/utils/

# Generate docs:
Create comprehensive API documentation for all exported functions in markdown format
```

### Test Generation

```bash
/context ./src/user-service.ts

# Generate tests:
Create Jest unit tests for this service with 100% code coverage
```

## Troubleshooting

### API Key Issues

**Problem:** "Invalid API key" error

**Solution:**
```bash
# Verify API key is set
echo $GOOGLE_API_KEY  # Linux/macOS
echo %GOOGLE_API_KEY%  # Windows

# Re-export with correct key
export GOOGLE_API_KEY="your_valid_key"

# Test with curl
curl "https://generativelanguage.googleapis.com/v1/models?key=$GOOGLE_API_KEY"
```

### Connection Errors

**Problem:** Network timeout or connection refused

**Solution:**
```bash
# Check internet connectivity
ping google.com

# Verify proxy settings if behind corporate firewall
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"

# Test API endpoint
curl -I https://generativelanguage.googleapis.com
```

### Model Not Available

**Problem:** "Model not found" error

**Solution:**
```bash
# List available models
/model

# Use a supported model:
/model gemini-2.5-flash

# Check API access for specific models in Google AI Studio
```

### Rate Limiting

**Problem:** "Rate limit exceeded" or 429 errors

**Solution:**
- Wait before retrying (rate limits reset after a period)
- Use `gemini-2.5-flash` for higher rate limits
- Implement exponential backoff in scripts:

```bash
#!/bin/bash
attempt=0
max_attempts=5

while [ $attempt -lt $max_attempts ]; do
  if echo "$prompt" | antigravity; then
    break
  fi
  attempt=$((attempt + 1))
  sleep $((2 ** attempt))
done
```

### Context Size Errors

**Problem:** "Context length exceeded" error

**Solution:**
```bash
# Reduce context size
/clear  # Clear previous conversation

# Use smaller file selections
/context ./src/specific-file.ts  # Instead of entire directory

# Summarize large files first
cat large-file.ts | head -n 100 | antigravity "Summarize this code"
```

### Installation Issues

**Problem:** npm install fails

**Solution:**
```bash
# Clear npm cache
npm cache clean --force

# Try with explicit registry
npm install -g gemini-antigravity-cli --registry=https://registry.npmjs.org

# Or use npx without global install
npx gemini-antigravity-cli
```

### Binary Execution Issues

**Problem:** Permission denied on Unix systems

**Solution:**
```bash
chmod +x antigravity
./antigravity
```

**Problem:** Windows SmartScreen warning

**Solution:**
- Click "More info" → "Run anyway"
- Or build from source if concerned about security

## Advanced Configuration

### Custom System Prompts

Modify behavior with custom system prompts:

```json
{
  "systemPrompt": "You are an expert TypeScript developer specializing in React and Node.js. Provide concise, production-ready code with error handling. Always use modern ES6+ syntax."
}
```

### Temperature and Token Settings

Control response creativity and length:

```json
{
  "temperature": 0.2,  // Lower = more deterministic (0.0-1.0)
  "maxTokens": 4096,   // Maximum response length
  "topP": 0.95,        // Nucleus sampling threshold
  "topK": 40           // Top-K sampling parameter
}
```

### Plugin Configuration

Enable community plugins (if supported):

```json
{
  "plugins": [
    "git-integration",
    "code-formatter",
    "security-scanner"
  ]
}
```

## Best Practices

1. **Always set API key as environment variable** — never hardcode in scripts
2. **Use `/clear` between unrelated tasks** — prevents context confusion
3. **Provide specific context with `/context`** — improves accuracy
4. **Start with `gemini-2.5-flash`** — faster and cheaper for most tasks
5. **Save important conversations** — use `/save` for future reference
6. **Use lower temperature (0.2-0.4) for code generation** — more consistent output
7. **Higher temperature (0.7-0.9) for creative tasks** — like naming or brainstorming
