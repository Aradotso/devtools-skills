---
name: github-copilot-cli
description: AI-powered command-line assistant for developers that provides code review, debugging, test generation, and development workflows directly in the terminal
triggers:
  - "how do I use GitHub Copilot CLI"
  - "review this code with copilot cli"
  - "debug using copilot in terminal"
  - "generate tests with copilot cli"
  - "explain this code with copilot"
  - "create custom copilot agents"
  - "set up copilot cli skills"
  - "integrate MCP servers with copilot"
---

# GitHub Copilot CLI Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

GitHub Copilot CLI brings AI-powered development assistance directly to your terminal. It enables code review, debugging, test generation, code explanation, and custom AI agents without leaving the command line. Think of it as having an expert developer available 24/7 in your terminal who can read your codebase, explain patterns, generate code, and help troubleshoot issues.

## Installation

### Prerequisites

- GitHub account with Copilot access (free tier, subscription, or education)
- GitHub CLI (`gh`) version 2.62.0 or later
- Terminal access

### Install GitHub CLI

**macOS:**
```bash
brew install gh
```

**Windows:**
```bash
winget install --id GitHub.cli
```

**Linux (Debian/Ubuntu):**
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

### Install Copilot CLI Extension

```bash
gh extension install github/gh-copilot
```

### Authenticate

```bash
gh auth login
```

Follow the prompts to authenticate with your GitHub account that has Copilot access.

### Verify Installation

```bash
gh copilot --version
```

## Core Commands

### Interactive Chat Mode

Start an interactive conversation with Copilot:

```bash
gh copilot
```

This opens a conversational interface where you can ask questions, get code suggestions, and iterate on solutions.

### Quick Explain

Get quick explanations without entering interactive mode:

```bash
gh copilot explain "git rebase -i HEAD~3"
```

```bash
gh copilot explain "docker-compose up -d"
```

### Quick Suggest

Get command suggestions for tasks:

```bash
gh copilot suggest "find all python files modified in last 7 days"
```

```bash
gh copilot suggest "compress all jpg files in current directory"
```

### Aliases (Shortcuts)

Set up convenient aliases:

```bash
gh copilot alias -- --shell bash
```

This creates three aliases:
- `ghcs` - suggest commands
- `ghce` - explain commands
- `ghcp` - open Copilot chat

After setup, you can use:

```bash
ghcs "list all running docker containers"
ghce "kubectl get pods -A"
```

## Interaction Modes

### 1. Chat Mode (Interactive)

Best for: Complex problems, iterative development, multi-step workflows

```bash
gh copilot
```

Example conversation:
```
> I need to review a Python Flask application for security issues

Copilot: I'll help you review your Flask application for security. Could you share:
1. The main application file
2. Any authentication/authorization code
3. Database interaction code

> @app.py @auth.py @models.py

Copilot: [Analyzes files and provides security review]
```

### 2. Explain Mode

Best for: Understanding unfamiliar commands, code snippets, or error messages

```bash
gh copilot explain "awk '{print $2}' file.txt | sort -u"
```

```bash
gh copilot explain "SELECT u.name, COUNT(o.id) FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.id"
```

### 3. Suggest Mode

Best for: Getting shell commands for specific tasks

```bash
gh copilot suggest "recursively delete all node_modules folders"
```

```bash
gh copilot suggest "monitor CPU and memory usage in real-time"
```

## Context Awareness

### File Context (@-mentions)

Reference specific files in your prompts:

```bash
gh copilot
> Review @server.py for potential race conditions
```

```bash
gh copilot
> Compare @config.dev.json and @config.prod.json and explain differences
```

### Multi-File Context

Reference multiple files for comprehensive analysis:

```bash
gh copilot
> @models.py @views.py @serializers.py review this Django app for N+1 queries
```

### Directory Context

Analyze entire directories:

```bash
gh copilot
> @src/ explain the architecture of this application
```

## Development Workflows

### Code Review

```bash
gh copilot
> Review @pull_request_changes.diff for:
> - Security vulnerabilities
> - Performance issues
> - Code style violations
> - Missing error handling
```

### Debugging

```bash
gh copilot
> I'm getting "TypeError: Cannot read property 'map' of undefined" in @app.js line 42
> Here's the error stack: @error.log
```

### Test Generation

For Python:
```bash
gh copilot
> Generate pytest unit tests for @calculator.py covering edge cases
```

For JavaScript:
```bash
gh copilot
> Create Jest tests for @api-client.js including error scenarios
```

For Go:
```bash
gh copilot
> Write table-driven tests for @handler.go
```

### Refactoring

```bash
gh copilot
> Refactor @legacy-auth.js to use async/await instead of callbacks
```

```bash
gh copilot
> Convert @class-component.jsx to a functional component with hooks
```

## Custom Agents and Instructions

### Creating Custom Agents

Agents are reusable AI assistants with specific expertise. Create an agent configuration:

```bash
mkdir -p ~/.config/gh-copilot/agents
```

**Security Review Agent** (`~/.config/gh-copilot/agents/security-reviewer.md`):

```markdown
# Security Reviewer Agent

You are a security-focused code reviewer specializing in:
- OWASP Top 10 vulnerabilities
- Input validation and sanitization
- Authentication and authorization flaws
- SQL injection and XSS prevention
- Secrets and credential management

Always:
1. Identify specific line numbers with issues
2. Explain the security impact
3. Provide secure code examples
4. Reference relevant OWASP guidelines
```

**Python Expert Agent** (`~/.config/gh-copilot/agents/python-expert.md`):

```markdown
# Python Expert Agent

You are a Python expert focusing on:
- PEP 8 style compliance
- Type hints and mypy compatibility
- Pythonic idioms and patterns
- Performance optimization
- Testing with pytest

Provide:
- Specific improvement suggestions
- Code examples following best practices
- Performance benchmarks when relevant
```

### Using Agents

```bash
gh copilot
> @security-reviewer analyze @app.py
```

```bash
gh copilot
> @python-expert review @data_processor.py for optimization opportunities
```

### Custom Instructions (Global)

Set global instructions that apply to all interactions:

```bash
mkdir -p ~/.config/gh-copilot
```

Create `~/.config/gh-copilot/instructions.md`:

```markdown
# My Development Preferences

- **Language**: Prefer TypeScript over JavaScript
- **Testing**: Use Jest with React Testing Library
- **Style**: Follow Airbnb style guide
- **Error Handling**: Always use try-catch and log errors
- **Comments**: Add JSDoc comments for public functions
- **Security**: Flag any hardcoded credentials or secrets
```

## Skills (Automated Context)

Skills are automatically loaded based on your project structure.

### Creating a Skill

Create `.copilot/skills/` directory in your project:

```bash
mkdir -p .copilot/skills
```

**Django Project Skill** (`.copilot/skills/django-project.md`):

```markdown
# Django Project Configuration

## Stack
- Django 4.2
- PostgreSQL 15
- Redis for caching
- Celery for async tasks

## Code Standards
- Use Django REST Framework serializers
- Prefer class-based views
- Always use select_related/prefetch_related
- Write tests with pytest-django

## Database
- Use migrations for all schema changes
- Never use raw SQL without parameterization
- Connection: via DATABASE_URL environment variable
```

**React Project Skill** (`.copilot/skills/react-conventions.md`):

```markdown
# React Project Conventions

## Stack
- React 18 with TypeScript
- Vite for bundling
- React Query for data fetching
- Zustand for state management

## Patterns
- Functional components with hooks
- Custom hooks for reusable logic
- Error boundaries for fault tolerance
- Suspense for loading states

## File Structure
- Components in src/components
- Hooks in src/hooks
- Utils in src/utils
```

Skills are automatically loaded when you run `gh copilot` in a directory containing `.copilot/skills/`.

## MCP Server Integration

Model Context Protocol (MCP) servers extend Copilot CLI with external data sources and APIs.

### Configuration File

Create `~/.config/gh-copilot/mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### Available MCP Servers

**GitHub Integration:**
```json
{
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
    }
  }
}
```

Usage:
```bash
gh copilot
> Search GitHub for recent issues tagged "bug" in owner/repo
```

**PostgreSQL Database:**
```json
{
  "postgres": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres"],
    "env": {
      "DATABASE_URL": "${DATABASE_URL}"
    }
  }
}
```

Usage:
```bash
gh copilot
> Query the database for users created in the last 30 days
```

**Filesystem Access:**
```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/directory"]
  }
}
```

**Brave Search:**
```json
{
  "brave-search": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-brave-search"],
    "env": {
      "BRAVE_API_KEY": "${BRAVE_API_KEY}"
    }
  }
}
```

### Creating Custom MCP Servers

MCP servers are command-line tools that implement the MCP protocol. Example structure:

```javascript
// my-mcp-server.js
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server({
  name: 'my-custom-server',
  version: '1.0.0',
});

// Define tools
server.setRequestHandler('tools/list', async () => ({
  tools: [{
    name: 'get_data',
    description: 'Fetch data from custom source',
    inputSchema: {
      type: 'object',
      properties: {
        query: { type: 'string' }
      }
    }
  }]
}));

// Implement tool
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === 'get_data') {
    const result = await fetchCustomData(args.query);
    return { content: [{ type: 'text', text: JSON.stringify(result) }] };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

Add to `mcp.json`:
```json
{
  "my-custom": {
    "command": "node",
    "args": ["/path/to/my-mcp-server.js"]
  }
}
```

## Real-World Examples

### Example 1: Code Review Workflow

```bash
# Start review session
gh copilot

> I need to review changes before committing. Review @src/auth.py @src/models.py for:
> 1. Security issues
> 2. Performance problems
> 3. Code style
> 4. Missing tests

# Copilot provides detailed review

> Generate pytest tests for the issues you found in @src/auth.py

# Copilot generates test code

> Now explain how to fix the SQL injection vulnerability you identified
```

### Example 2: Debugging Session

```bash
gh copilot

> My Flask app is returning 500 errors. Here's the traceback: @error.log
> And here's the relevant code: @app.py @database.py

# Copilot analyzes and suggests fixes

> Apply your suggested fix to @database.py and show me the corrected code

> Now write a test that would have caught this bug
```

### Example 3: Test Generation

```bash
gh copilot

> Generate comprehensive Jest tests for @api-client.ts covering:
> - Successful API calls
> - Network errors
> - Invalid responses
> - Timeout scenarios
> - Retry logic

# Copilot generates test suite

> Add tests for the edge case where the API returns 429 rate limit
```

### Example 4: Multi-File Refactoring

```bash
gh copilot

> I need to refactor @legacy-app/ to use modern React patterns:
> - Convert class components to functional components with hooks
> - Replace Redux with Zustand
> - Add TypeScript types
> Start with @legacy-app/components/UserList.jsx

# Copilot provides refactored code

> Continue with @legacy-app/components/UserDetail.jsx using the same patterns
```

### Example 5: Database Query Optimization

```bash
gh copilot

> @models.py contains Django ORM queries. Analyze for N+1 query problems

# Copilot identifies issues

> Show me how to fix the N+1 issue in the get_user_orders view using select_related

> Now generate a test that verifies the query count is optimized
```

## Configuration Files

### Main Config Directory

```
~/.config/gh-copilot/
├── instructions.md          # Global instructions
├── mcp.json                # MCP server configuration
└── agents/                 # Custom agent definitions
    ├── security-reviewer.md
    ├── python-expert.md
    └── react-specialist.md
```

### Project-Level Skills

```
your-project/
├── .copilot/
│   └── skills/
│       ├── project-conventions.md
│       ├── api-documentation.md
│       └── deployment-guide.md
├── src/
└── tests/
```

## Common Patterns

### Pattern 1: Multi-Step Feature Development

```bash
gh copilot

> I need to add user authentication to my Flask app

# Copilot outlines approach

> Start by showing me the User model with password hashing

> Now create the login endpoint with JWT tokens

> Generate tests for the authentication flow

> Show me how to protect routes with auth middleware
```

### Pattern 2: Legacy Code Understanding

```bash
gh copilot

> @legacy-system/ is an old codebase I inherited. Help me understand:
> 1. The overall architecture
> 2. How data flows through the system
> 3. External dependencies
> 4. Potential technical debt

# Analyze incrementally
> Start with @legacy-system/main.py and @legacy-system/config.py
```

### Pattern 3: Security Audit

```bash
gh copilot

> @security-reviewer perform a complete security audit of @api/

# Review findings

> Prioritize the vulnerabilities by severity

> For each critical issue, show me the fix with code examples

> Generate security tests that verify the fixes
```

### Pattern 4: Documentation Generation

```bash
gh copilot

> Generate API documentation for @endpoints.py in OpenAPI format

> Create a README.md for @utils/ explaining each utility function

> Add inline comments to @complex-algorithm.py explaining the logic
```

### Pattern 5: Migration Planning

```bash
gh copilot

> I need to migrate from @old-api/ (REST) to @new-api/ (GraphQL)
> Create a migration plan with backwards compatibility

> Show me how to implement the first endpoint in both versions

> Generate tests that verify both APIs return identical data
```

## Troubleshooting

### Installation Issues

**Problem: `gh: command not found`**
```bash
# Verify installation
which gh

# Reinstall GitHub CLI
brew reinstall gh  # macOS
```

**Problem: Extension install fails**
```bash
# List installed extensions
gh extension list

# Remove and reinstall
gh extension remove gh-copilot
gh extension install github/gh-copilot
```

### Authentication Issues

**Problem: "Not authenticated" error**
```bash
# Check auth status
gh auth status

# Re-authenticate
gh auth login

# Verify Copilot access
gh copilot --version
```

**Problem: "No Copilot subscription"**
- Verify your GitHub account has Copilot access at https://github.com/settings/copilot
- Check if you're using the correct account: `gh auth status`

### Context Issues

**Problem: Copilot can't find referenced files**
```bash
# Use absolute paths
gh copilot
> Review /full/path/to/file.py

# Or navigate to project directory first
cd /path/to/project
gh copilot
> Review @src/file.py
```

**Problem: Too much context, slow responses**
```bash
# Be specific with file references
> Review @src/auth.py  # Good
> Review @src/        # Too broad
```

### Agent/Skill Issues

**Problem: Custom agent not loading**
```bash
# Verify file location
ls ~/.config/gh-copilot/agents/

# Check file format (must be .md)
# Verify no syntax errors in frontmatter
```

**Problem: Skill not activating**
```bash
# Verify skill directory
ls .copilot/skills/

# Must run gh copilot from project directory
pwd  # Should be in project root

# Restart copilot session
gh copilot
```

### MCP Server Issues

**Problem: MCP server not connecting**
```bash
# Verify mcp.json syntax
cat ~/.config/gh-copilot/mcp.json | jq .

# Check environment variables
echo $GITHUB_TOKEN
echo $DATABASE_URL

# Test server manually
npx -y @modelcontextprotocol/server-github
```

**Problem: Server timeouts**
- Check network connectivity
- Verify API credentials are valid
- Check server logs if available

### Performance Issues

**Problem: Slow responses**
- Reduce context size (fewer @-mentions)
- Use more specific queries
- Check internet connection
- Try during off-peak hours

**Problem: High token usage**
- Be concise in prompts
- Avoid pasting large files
- Use @-mentions instead of pasting code
- Break complex tasks into smaller steps

## Best Practices

1. **Be Specific**: "Review @auth.py for SQL injection" is better than "Review my code"

2. **Use Context Wisely**: Reference only files relevant to your question

3. **Iterate**: Start with a focused question, then drill down based on responses

4. **Create Agents**: Build specialized agents for recurring tasks (security reviews, code style checks)

5. **Document with Skills**: Keep project conventions in `.copilot/skills/` for consistent assistance

6. **Leverage MCP**: Connect to databases, APIs, and tools your project actually uses

7. **Review Suggestions**: Always review generated code before applying it

8. **Combine Modes**: Use `suggest` for quick commands, `chat` for complex problems

9. **Save Useful Patterns**: Document working prompts in your project's `.copilot/skills/`

10. **Environment Variables**: Never hardcode secrets; always use env vars in MCP configs

## Additional Resources

- Official Documentation: https://docs.github.com/en/copilot/how-tos/copilot-cli
- Command Reference: https://docs.github.com/en/copilot/reference/cli-command-reference
- GitHub Copilot CLI Course: https://github.com/github/copilot-cli-for-beginners
- MCP Protocol Specification: https://modelcontextprotocol.io
- Community Discord: https://aka.ms/foundry/discord
