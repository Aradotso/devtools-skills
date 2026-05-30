---
name: 14days-build-claude-code-cli
description: Tutorial project for building a Claude Code-style agent CLI from scratch in Python with tool calling, file editing, and permission systems
triggers:
  - "help me learn how to build an AI code agent"
  - "show me how to implement tool calling for an agent"
  - "I want to create a code agent CLI with file editing"
  - "teach me agent harness architecture"
  - "how do I build a REPL for an AI agent"
  - "implement safe file editing for code agents"
  - "build an agent with permission system"
  - "create a CLI agent that runs bash commands"
---

# 14days-build-claude-code-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

A 14-day tutorial project for building a production-grade code agent CLI from scratch in Python. Learn to implement the "harness" layer around LLMs: tool calling, file operations, permission systems, session management, and agent orchestration patterns inspired by Claude Code.

## What It Is

This is an educational implementation of a Code Agent CLI called `agent-code`. Over 14 days, you build:

- CLI runtime with REPL and slash commands
- Agent loop with tool calling (Anthropic Messages API format)
- File search, read, and safe edit with diff preview
- Bash command execution with permission engine
- Session persistence and project memory
- Hooks, skills, subagents, and MCP tool integration
- Worktree isolation for multi-task workflows

**Default model:** Uses DeepSeek's Anthropic-compatible endpoint for cost-effective testing, but works with any Anthropic Messages API compatible service (Claude, etc.)

## Installation

Each day is a standalone Python package under `packages/day-*/`. For learning, you maintain one `agent-code` project and evolve it through 14 days.

### Quick Start with a Completed Day

```bash
# Clone the repo
git clone https://github.com/bozhouDev/14days-build-claude-code-cli.git
cd 14days-build-claude-code-cli

# Run a completed day's snapshot
cd packages/day-02-real-model-tool-calling
uv sync
uv run agent-code "list files in current directory"
```

### Set Up Your Own Learning Project

```bash
# Create your agent-code project
mkdir agent-code
cd agent-code
uv init
uv add typer anthropic

# Configure API keys (DeepSeek or Claude)
export ANTHROPIC_AUTH_TOKEN="sk-your-key"
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
```

Then follow tutorials starting from `docs/day-01-hello-agent.md`.

## Key Architecture

```python
# Core agent loop pattern (Day 2+)
from agent_code.providers import AnthropicProvider
from agent_code.tools import ToolRegistry

provider = AnthropicProvider()
registry = ToolRegistry()
registry.register_all()

messages = [{"role": "user", "content": user_input}]

while True:
    response = provider.create_message(
        model="deepseek-v4-flash",
        messages=messages,
        tools=registry.to_anthropic_format()
    )
    
    if response.stop_reason == "end_turn":
        break
    
    # Handle tool_use blocks
    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            result = registry.execute(block.name, block.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": result
            })
    
    messages.append({"role": "assistant", "content": response.content})
    messages.append({"role": "user", "content": tool_results})
```

## Day-by-Day Progression

### Day 1: Hello Agent
CLI entry, REPL, mock provider, minimal agent loop.

```python
# agent_code/cli.py
import typer
from agent_code.agent import Agent
from agent_code.providers import MockProvider

app = typer.Typer()

@app.command()
def main(prompt: str = None):
    agent = Agent(MockProvider())
    
    if prompt:
        agent.run(prompt)
    else:
        agent.repl()
```

### Day 2: Real Model + Tool Calling
Anthropic provider, `tool_use` / `tool_result` handling.

```python
# agent_code/tools/echo.py
def echo_tool(message: str) -> str:
    """Echo a message back."""
    return f"Echo: {message}"

ECHO_SCHEMA = {
    "name": "echo",
    "description": "Echo a message",
    "input_schema": {
        "type": "object",
        "properties": {
            "message": {"type": "string", "description": "Message to echo"}
        },
        "required": ["message"]
    }
}
```

### Day 3: File + Web Tools
File search, read with cwd boundary, web search tool.

```python
# agent_code/tools/file_tools.py
import os
from pathlib import Path

def list_files_tool(path: str = ".") -> str:
    """List files in directory."""
    abs_path = Path.cwd() / path
    
    # Security: enforce cwd boundary
    if not str(abs_path.resolve()).startswith(str(Path.cwd())):
        return "Error: Path outside working directory"
    
    files = [f.name for f in abs_path.iterdir()]
    return "\n".join(files)

def read_file_tool(path: str) -> str:
    """Read file contents."""
    abs_path = Path.cwd() / path
    
    if not abs_path.exists():
        return f"Error: File not found: {path}"
    
    with open(abs_path, "r") as f:
        return f.read()
```

### Day 4: Safe Edit
Read-before-edit, string replacement, diff preview.

```python
# agent_code/tools/edit_tools.py
import difflib

def write_file_tool(path: str, old_content: str, new_content: str) -> str:
    """Write file with diff preview and confirmation."""
    abs_path = Path.cwd() / path
    
    # Read current content
    current = abs_path.read_text() if abs_path.exists() else ""
    
    # Verify old_content matches (prevents concurrent edits)
    if abs_path.exists() and old_content != current:
        return "Error: File changed since read. Refresh and retry."
    
    # Show diff
    diff = "\n".join(difflib.unified_diff(
        old_content.splitlines(),
        new_content.splitlines(),
        lineterm="",
        fromfile=f"a/{path}",
        tofile=f"b/{path}"
    ))
    
    print(f"\n{diff}\n")
    
    # Confirm
    confirm = input("Apply? [y/N] ")
    if confirm.lower() != "y":
        return "Edit cancelled"
    
    abs_path.write_text(new_content)
    return f"Wrote {path}"
```

### Day 5: Bash + Permission
Command execution with permission engine.

```python
# agent_code/tools/bash_tool.py
import subprocess
from agent_code.permissions import check_permission

DANGEROUS_COMMANDS = ["rm -rf", "sudo", "mv /", "chmod 777"]

def bash_tool(command: str, background: bool = False) -> str:
    """Execute bash command."""
    
    # Permission check
    if any(danger in command for danger in DANGEROUS_COMMANDS):
        if not check_permission(f"Run dangerous command: {command}"):
            return "Permission denied"
    
    if background:
        subprocess.Popen(command, shell=True)
        return f"Started in background: {command}"
    
    result = subprocess.run(
        command,
        shell=True,
        capture_output=True,
        text=True,
        timeout=30
    )
    
    output = result.stdout + result.stderr
    return output or f"Exit code: {result.returncode}"
```

### Day 6: Session + Memory
Session persistence, project memory in `.memdir/`.

```python
# agent_code/session.py
import json
from pathlib import Path
from datetime import datetime

class SessionManager:
    def __init__(self, session_dir: Path = None):
        self.session_dir = session_dir or Path.cwd() / ".agent-sessions"
        self.session_dir.mkdir(exist_ok=True)
        
    def save_turn(self, session_id: str, user_msg: str, assistant_msg: str):
        """Append turn to session JSONL."""
        session_file = self.session_dir / f"{session_id}.jsonl"
        
        turn = {
            "timestamp": datetime.utcnow().isoformat(),
            "user": user_msg,
            "assistant": assistant_msg
        }
        
        with open(session_file, "a") as f:
            f.write(json.dumps(turn) + "\n")
    
    def load_session(self, session_id: str) -> list:
        """Load session history."""
        session_file = self.session_dir / f"{session_id}.jsonl"
        
        if not session_file.exists():
            return []
        
        turns = []
        with open(session_file, "r") as f:
            for line in f:
                turns.append(json.loads(line))
        
        return turns
```

### Day 7: Slash + Hooks
Slash commands (`/help`, `/reset`), hooks for tool execution.

```python
# agent_code/slash.py
class SlashRouter:
    def __init__(self):
        self.commands = {}
    
    def register(self, name: str, handler):
        self.commands[name] = handler
    
    def handle(self, input_text: str) -> bool:
        """Return True if handled as slash command."""
        if not input_text.startswith("/"):
            return False
        
        parts = input_text[1:].split()
        cmd = parts[0]
        args = parts[1:]
        
        if cmd in self.commands:
            self.commands[cmd](*args)
            return True
        
        print(f"Unknown command: /{cmd}")
        return True

# Usage in REPL
router = SlashRouter()
router.register("help", lambda: print("Available: /help, /reset, /exit"))
router.register("reset", lambda: messages.clear())

while True:
    user_input = input("> ")
    if router.handle(user_input):
        continue
    # Normal agent processing...
```

## Configuration

### Environment Variables

```bash
# API credentials (DeepSeek or Claude)
export ANTHROPIC_AUTH_TOKEN="sk-..."
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"

# Model selection
export AGENT_MODEL="deepseek-v4-flash"

# Optional: session persistence location
export AGENT_SESSION_DIR=".agent-sessions"
```

### Project Structure (Your Learning Project)

```
agent-code/
├── pyproject.toml
├── agent_code/
│   ├── __init__.py
│   ├── cli.py           # Typer CLI entry
│   ├── agent.py         # Agent loop
│   ├── providers.py     # Mock + Anthropic providers
│   ├── tools/
│   │   ├── registry.py  # Tool registry
│   │   ├── echo.py
│   │   ├── file_tools.py
│   │   ├── edit_tools.py
│   │   └── bash_tool.py
│   ├── permissions.py   # Permission engine
│   ├── session.py       # Session manager
│   └── slash.py         # Slash command router
└── tests/
    └── test_*.py
```

## Running Tests

Each day's package includes tests. Always run from the specific day directory:

```bash
cd packages/day-05-bash-permission
uv run pytest

# Run specific test
uv run pytest tests/test_bash_tool.py -v
```

## Common Patterns

### Adding a Custom Tool

```python
# 1. Define the function
def my_tool(arg1: str, arg2: int) -> str:
    """Tool description for the model."""
    return f"Processed {arg1} with {arg2}"

# 2. Define Anthropic schema
MY_TOOL_SCHEMA = {
    "name": "my_tool",
    "description": "What this tool does",
    "input_schema": {
        "type": "object",
        "properties": {
            "arg1": {"type": "string", "description": "First argument"},
            "arg2": {"type": "integer", "description": "Second argument"}
        },
        "required": ["arg1", "arg2"]
    }
}

# 3. Register in ToolRegistry
registry.register("my_tool", my_tool, MY_TOOL_SCHEMA)
```

### Implementing a Hook

```python
# agent_code/hooks.py
class HookManager:
    def __init__(self):
        self.hooks = {"pre_tool": [], "post_tool": []}
    
    def register(self, event: str, callback):
        self.hooks[event].append(callback)
    
    def trigger(self, event: str, **kwargs):
        for callback in self.hooks.get(event, []):
            callback(**kwargs)

# Usage
hooks = HookManager()
hooks.register("pre_tool", lambda tool_name, **kw: print(f"Calling {tool_name}"))

# In agent loop
hooks.trigger("pre_tool", tool_name=block.name, input=block.input)
result = registry.execute(block.name, block.input)
hooks.trigger("post_tool", tool_name=block.name, result=result)
```

### Safe File Operations Pattern

Always follow this pattern for file edits:

```python
# 1. Read current content
current_content = read_file_tool(path)

# 2. Let model see current content (include in prompt or tool result)
# 3. Model calls write_file_tool with old_content=current and new_content

# 4. In write_file_tool: verify old_content matches current
if old_content != current_actual:
    return "File changed since read, refresh and retry"

# 5. Show diff and ask confirmation
# 6. Write only after approval
```

## Troubleshooting

### Tool Execution Fails Silently

Check tool result format matches Anthropic spec:

```python
# Correct format
{
    "type": "tool_result",
    "tool_use_id": block.id,  # Must match tool_use block
    "content": result_string
}
```

### Permission Denied on Bash Commands

The permission engine intercepts dangerous patterns. Either:

1. Respond "y" to the permission prompt
2. Adjust `DANGEROUS_COMMANDS` list in `bash_tool.py`
3. Implement a permission allowlist for trusted commands

### DeepSeek Returns Malformed Tool Calls

DeepSeek's Anthropic compatibility is good but not perfect. If you see parsing errors:

```python
# Add validation in agent loop
for block in response.content:
    if block.type == "tool_use":
        if not hasattr(block, "name") or not hasattr(block, "input"):
            print(f"Warning: Malformed tool_use block: {block}")
            continue
```

### Session Files Growing Too Large

Implement context compaction (Day 11 topic):

```python
def compact_session(messages, max_tokens=100000):
    """Keep system prompt, recent messages, summarize old ones."""
    if total_tokens(messages) < max_tokens:
        return messages
    
    # Summarize middle messages
    summary = model.create_message(
        model="deepseek-v4-flash",
        messages=[{"role": "user", "content": f"Summarize: {old_messages}"}]
    )
    
    return [messages[0]] + [summary] + messages[-10:]
```

### Files Outside CWD Access Denied

This is by design (Day 3 security boundary). To allow broader access:

```python
# Option 1: Expand allowed roots
ALLOWED_ROOTS = [Path.cwd(), Path.home() / "projects"]

# Option 2: Add a permission check instead of hard block
if not is_within_cwd(path):
    if not check_permission(f"Access {path} outside project?"):
        return "Permission denied"
```

## Advanced Usage (Days 8-14)

Days 8-14 extend the foundation with:

- **Day 8:** Plan mode with task lists and execution constraints
- **Day 9:** Skills - on-demand knowledge loading
- **Day 10:** Subagents - delegate tasks to specialized agents
- **Day 11:** Context compaction for long conversations
- **Day 12:** Multi-agent coordination
- **Day 13:** Worktree isolation for parallel tasks
- **Day 14:** MCP client for tool ecosystem integration

Check the main repo for updated docs as these days are released.

## Web Tutorial

Preview the interactive tutorial locally:

```bash
cd agent-code-learn
npm install
npm run dev
# Open http://localhost:3000
```

Production site: https://buildcc.dev

## Learning Resources

- **Tutorial docs:** `docs/day-*.md` - detailed step-by-step guides
- **Reference snapshots:** `packages/day-*/` - working code for each day
- **Architecture reference:** `reference/claude-code-official/` - public Claude Code snapshot for architecture study (not copied into tutorial)

## Testing Your Implementation

```bash
# Test basic agent loop
uv run agent-code "echo hello using the echo tool"

# Test file operations
uv run agent-code "list files in current directory"
uv run agent-code "read pyproject.toml and summarize"

# Test bash (with permission prompt)
uv run agent-code "run 'ls -la' command"

# Test session persistence
uv run agent-code --session my-session "remember: my favorite color is blue"
uv run agent-code --session my-session "what's my favorite color?"
```

## Project Goals

This is a teaching project. Goals:

- ✅ Understand agent harness architecture
- ✅ Learn tool calling patterns
- ✅ Implement safe file operations
- ✅ Build permission systems
- ✅ Manage agent context and memory

NOT goals:

- ❌ Production-ready robustness
- ❌ 1:1 Claude Code feature parity
- ❌ Enterprise-scale performance

## Contributing

Issues and PRs welcome for:

- Tutorial clarity improvements
- Bug fixes in reference snapshots
- DeepSeek/Claude compatibility issues
- Documentation or translation fixes
- Architecture explanation corrections

GitHub: https://github.com/bozhouDev/14days-build-claude-code-cli
