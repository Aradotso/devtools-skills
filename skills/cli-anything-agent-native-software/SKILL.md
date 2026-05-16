---
name: cli-anything-agent-native-software
description: Generate and use command-line interfaces that make any software controllable by AI agents with structured JSON output
triggers:
  - make this software agent-accessible
  - generate a CLI wrapper for this application
  - create an agent-native interface
  - wrap this GUI tool with a command line
  - build a CLI harness with JSON output
  - make this app work with AI agents
  - create structured commands for this software
  - generate agent-ready CLI tools
---

# CLI-Anything: Agent-Native Software Interface Generator

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

CLI-Anything transforms GUI applications and services into agent-ready command-line interfaces with structured JSON output. It provides a framework for generating CLIs that AI agents can discover, understand, and use to control software programmatically.

## What It Does

CLI-Anything bridges the gap between AI agents and traditional software by:

- **Generating CLIs** for GUI applications (GIMP, Blender, LibreOffice, etc.)
- **Structured Output** - All commands return JSON for reliable agent parsing
- **Self-Documenting** - Auto-generated SKILL.md files agents can discover
- **Preview Loops** - Commands support `--preview` for visual feedback before committing
- **Trajectory Recording** - Captures command sequences for reproducible workflows
- **Hub Distribution** - Central registry for browsing and installing community CLIs

The project includes 18+ production harnesses (GIMP, Inkscape, Shotcut, Blender, Godot, Obsidian, n8n, etc.) and a framework for building your own.

## Installation

### Core Framework

```bash
# Clone the repository
git clone https://github.com/HKUDS/CLI-Anything.git
cd CLI-Anything

# Install dependencies
pip install -r requirements.txt

# Install a specific CLI harness (e.g., GIMP)
cd src/gimp
pip install -e .
```

### CLI Hub (Package Manager)

```bash
# Install the hub package manager
pip install cli-anything-hub

# Browse available CLIs
cli-hub list

# Search for specific tools
cli-hub search blender

# Install a CLI
cli-hub install gimp

# Update installed CLIs
cli-hub update gimp

# Uninstall
cli-hub uninstall gimp
```

### Install Skills for AI Agents

```bash
# Install a skill for Pi, Claude Code, Cursor, etc.
npx skills add HKUDS/CLI-Anything --skill gimp -g -y
npx skills add HKUDS/CLI-Anything --skill blender -g -y
npx skills add HKUDS/CLI-Anything --skill inkscape -g -y

# All skills are in the top-level skills/ directory
```

## Key Concepts

### Harness Structure

Every CLI harness follows this pattern:

```
src/<app-name>/
├── cli.py              # Click-based CLI entry point
├── commands/           # Command groups (edit, export, analyze, etc.)
│   ├── edit.py
│   ├── export.py
│   └── ...
├── core/               # Application-specific logic
│   ├── backend.py      # Software interaction layer
│   └── utils.py
├── tests/
│   ├── unit/           # Fast, isolated tests
│   └── e2e/            # Full integration tests
├── setup.py            # Package definition
└── SKILL.md            # Agent-discoverable documentation
```

### Command Pattern

All commands follow this JSON-output pattern:

```python
import click
import json

@click.command()
@click.option('--input', required=True, help='Input file path')
@click.option('--output', required=True, help='Output file path')
@click.option('--format', default='JSON', type=click.Choice(['JSON', 'HUMAN']))
def process(input, output, format):
    """Process an image with specific operations."""
    try:
        # Perform operation
        result = {
            'success': True,
            'input': input,
            'output': output,
            'message': 'Operation completed successfully'
        }
        
        if format == 'JSON':
            click.echo(json.dumps(result, indent=2))
        else:
            click.echo(f"✓ Processed {input} → {output}")
            
    except Exception as e:
        error = {
            'success': False,
            'error': str(e)
        }
        click.echo(json.dumps(error, indent=2))
        raise click.Abort()
```

## Using Existing CLIs

### GIMP Example

```bash
# Create a new image
gimp-cli create --width 800 --height 600 --output /tmp/canvas.xcf

# Add a text layer
gimp-cli layer add-text \
  --image /tmp/canvas.xcf \
  --text "Hello AI" \
  --x 100 --y 100 \
  --font-size 48 \
  --color "#FF0000"

# Export as PNG
gimp-cli export --input /tmp/canvas.xcf --output /tmp/result.png --format png

# All commands support JSON output
gimp-cli layer list --image /tmp/canvas.xcf --format JSON
```

### Blender Example

```bash
# Create a scene with primitives
blender-cli scene create --name "MyScene" --output /tmp/scene.blend

# Add objects
blender-cli object add-cube --name "Box1" --location 0,0,0 --scene /tmp/scene.blend
blender-cli object add-sphere --name "Ball" --location 2,0,1 --scene /tmp/scene.blend

# Render
blender-cli render \
  --input /tmp/scene.blend \
  --output /tmp/render.png \
  --resolution-x 1920 \
  --resolution-y 1080 \
  --samples 128 \
  --engine CYCLES
```

### Inkscape Example

```bash
# Create SVG with shapes
inkscape-cli create --width 500 --height 500 --output /tmp/drawing.svg

# Add elements
inkscape-cli shape add-rect \
  --svg /tmp/drawing.svg \
  --x 50 --y 50 --width 100 --height 100 \
  --fill "#3498db"

# Export to PNG
inkscape-cli export \
  --input /tmp/drawing.svg \
  --output /tmp/drawing.png \
  --dpi 300
```

## Building a New CLI Harness

### Step 1: Project Setup

```bash
# Create harness directory
mkdir -p src/myapp/{commands,core,tests/{unit,e2e}}
cd src/myapp

# Create setup.py
cat > setup.py << 'EOF'
from setuptools import setup, find_packages

setup(
    name='myapp-cli',
    version='0.1.0',
    packages=find_packages(),
    install_requires=[
        'click>=8.0',
    ],
    entry_points={
        'console_scripts': [
            'myapp-cli=cli:cli',
        ],
    },
)
EOF
```

### Step 2: Create CLI Entry Point

```python
# cli.py
import click
from commands import process, export

@click.group()
@click.version_option()
def cli():
    """MyApp CLI - Agent-native interface for MyApp."""
    pass

# Register command groups
cli.add_command(process.process_group)
cli.add_command(export.export_group)

if __name__ == '__main__':
    cli()
```

### Step 3: Implement Command Groups

```python
# commands/process.py
import click
import json
from core.backend import MyAppBackend

@click.group(name='process')
def process_group():
    """Processing operations."""
    pass

@process_group.command(name='enhance')
@click.option('--input', required=True, type=click.Path(exists=True))
@click.option('--output', required=True, type=click.Path())
@click.option('--strength', default=0.5, type=float, help='Enhancement strength (0-1)')
@click.option('--format', default='JSON', type=click.Choice(['JSON', 'HUMAN']))
@click.option('--preview', is_flag=True, help='Show preview without saving')
def enhance(input, output, strength, format, preview):
    """Enhance image quality."""
    try:
        backend = MyAppBackend()
        
        if preview:
            preview_path = backend.generate_preview(input, 'enhance', strength=strength)
            result = {
                'success': True,
                'preview': preview_path,
                'message': 'Preview generated. Remove --preview to apply.'
            }
        else:
            backend.load_file(input)
            backend.apply_enhancement(strength)
            backend.save_file(output)
            result = {
                'success': True,
                'input': input,
                'output': output,
                'strength': strength,
                'message': 'Enhancement applied successfully'
            }
        
        if format == 'JSON':
            click.echo(json.dumps(result, indent=2))
        else:
            click.echo(f"✓ Enhanced {input} → {output}")
            
    except Exception as e:
        error = {'success': False, 'error': str(e)}
        click.echo(json.dumps(error, indent=2))
        raise click.Abort()
```

### Step 4: Backend Integration

```python
# core/backend.py
import subprocess
import tempfile
import os

class MyAppBackend:
    """Wrapper for MyApp software."""
    
    def __init__(self, executable_path=None):
        self.executable = executable_path or self._find_executable()
        self.current_file = None
    
    def _find_executable(self):
        """Locate MyApp executable."""
        # Check common installation paths
        paths = [
            '/usr/bin/myapp',
            '/usr/local/bin/myapp',
            'C:\\Program Files\\MyApp\\myapp.exe'
        ]
        for path in paths:
            if os.path.exists(path):
                return path
        raise RuntimeError("MyApp not found. Please install MyApp first.")
    
    def load_file(self, filepath):
        """Load a file into MyApp."""
        self.current_file = filepath
        # Implementation depends on whether MyApp has:
        # - Python API: import myapp; myapp.load(filepath)
        # - Scripting: Generate script and execute
        # - IPC: Send commands via socket/pipe
    
    def apply_enhancement(self, strength):
        """Apply enhancement filter."""
        # Call MyApp's API or scripting interface
        pass
    
    def save_file(self, output_path):
        """Save current document."""
        # Implement save logic
        pass
    
    def generate_preview(self, input_path, operation, **params):
        """Generate preview image."""
        preview_dir = tempfile.mkdtemp()
        preview_path = os.path.join(preview_dir, 'preview.png')
        # Generate low-res preview
        return preview_path
```

### Step 5: Write Tests

```python
# tests/unit/test_backend.py
import pytest
from core.backend import MyAppBackend

def test_backend_initialization():
    """Test backend can find executable."""
    backend = MyAppBackend()
    assert backend.executable is not None

def test_file_operations(tmp_path):
    """Test load and save operations."""
    backend = MyAppBackend()
    test_file = tmp_path / "test.myapp"
    test_file.write_text("test content")
    
    backend.load_file(str(test_file))
    assert backend.current_file == str(test_file)

# tests/e2e/test_enhance.py
import subprocess
import json
import pytest

def test_enhance_command(tmp_path):
    """Test full enhance workflow."""
    input_file = tmp_path / "input.png"
    output_file = tmp_path / "output.png"
    
    # Create test input (implementation specific)
    # ...
    
    result = subprocess.run([
        'myapp-cli', 'process', 'enhance',
        '--input', str(input_file),
        '--output', str(output_file),
        '--strength', '0.7',
        '--format', 'JSON'
    ], capture_output=True, text=True)
    
    assert result.returncode == 0
    data = json.loads(result.stdout)
    assert data['success'] is True
    assert output_file.exists()
```

### Step 6: Generate SKILL.md

```python
# Create skill_generator.py in your harness directory
import click
import json
import os

def generate_skill_md():
    """Generate SKILL.md for AI agent discovery."""
    
    # Extract command structure
    from cli import cli as main_cli
    
    commands = []
    for group_name, group in main_cli.commands.items():
        if hasattr(group, 'commands'):
            for cmd_name, cmd in group.commands.items():
                commands.append({
                    'group': group_name,
                    'name': cmd_name,
                    'help': cmd.help or '',
                    'params': [p.name for p in cmd.params if p.name != 'format']
                })
    
    skill_content = f"""---
name: myapp-agent-interface
description: Control MyApp through structured commands with JSON output
triggers:
  - use myapp to process this
  - enhance this with myapp
  - export using myapp
  - create with myapp
  - automate myapp workflow
  - generate myapp output
---

# MyApp CLI - Agent Interface

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Control MyApp through a structured command-line interface designed for AI agents.

## Installation

```bash
pip install myapp-cli
# or via CLI-Hub
cli-hub install myapp
```

## Available Commands

"""
    
    for cmd in commands:
        skill_content += f"\n### {cmd['group']} {cmd['name']}\n\n"
        skill_content += f"{cmd['help']}\n\n"
        skill_content += f"**Parameters:** {', '.join(cmd['params'])}\n\n"
    
    with open('SKILL.md', 'w') as f:
        f.write(skill_content)
    
    print("✓ SKILL.md generated")

if __name__ == '__main__':
    generate_skill_md()
```

## Advanced Patterns

### Preview Loop Implementation

```python
@click.option('--preview', is_flag=True, help='Generate preview without applying')
@click.option('--preview-resolution', default=512, help='Preview image size')
def command_with_preview(preview, preview_resolution, **kwargs):
    """Command supporting preview mode."""
    if preview:
        # Generate low-resolution preview
        preview_path = generate_preview(
            kwargs['input'],
            resolution=preview_resolution,
            operation_params=kwargs
        )
        return {
            'success': True,
            'preview': preview_path,
            'message': 'Preview ready. Remove --preview to apply changes.'
        }
    else:
        # Execute full operation
        return execute_full_operation(**kwargs)
```

### Trajectory Recording

```python
import json
import os
from datetime import datetime

class TrajectoryRecorder:
    """Record command sequences for reproducibility."""
    
    def __init__(self, session_name=None):
        self.session_name = session_name or datetime.now().isoformat()
        self.trajectory = []
        self.output_dir = os.path.expanduser('~/.myapp-cli/trajectories')
        os.makedirs(self.output_dir, exist_ok=True)
    
    def record(self, command, params, result):
        """Record a command execution."""
        self.trajectory.append({
            'timestamp': datetime.now().isoformat(),
            'command': command,
            'params': params,
            'result': result
        })
    
    def save(self):
        """Save trajectory to disk."""
        filepath = os.path.join(self.output_dir, f"{self.session_name}.json")
        with open(filepath, 'w') as f:
            json.dump(self.trajectory, f, indent=2)
        return filepath
    
    @staticmethod
    def replay(trajectory_file):
        """Replay a recorded trajectory."""
        with open(trajectory_file) as f:
            trajectory = json.load(f)
        
        for step in trajectory:
            # Re-execute command with recorded params
            print(f"Replaying: {step['command']}")
            # Implementation specific
```

### Environment Configuration

```python
# core/config.py
import os
from pathlib import Path

class Config:
    """Configuration management."""
    
    def __init__(self):
        self.config_dir = Path.home() / '.myapp-cli'
        self.config_file = self.config_dir / 'config.json'
        self.config_dir.mkdir(exist_ok=True)
        self.load()
    
    def load(self):
        """Load configuration from disk."""
        if self.config_file.exists():
            import json
            with open(self.config_file) as f:
                self._config = json.load(f)
        else:
            self._config = self.defaults()
    
    def defaults(self):
        """Default configuration values."""
        return {
            'executable_path': os.getenv('MYAPP_EXECUTABLE'),
            'default_format': 'JSON',
            'enable_trajectories': True,
            'preview_resolution': 512,
            'temp_dir': str(self.config_dir / 'tmp')
        }
    
    def get(self, key, default=None):
        """Get configuration value."""
        return self._config.get(key, default)
    
    def set(self, key, value):
        """Set configuration value."""
        self._config[key] = value
        self.save()
    
    def save(self):
        """Save configuration to disk."""
        import json
        with open(self.config_file, 'w') as f:
            json.dump(self._config, f, indent=2)
```

## Integration with AI Agents

### Claude Code / Cursor

```python
# In your project, agents will discover the skill:
# "I need to process images with MyApp"
# 
# Agent automatically runs:
# myapp-cli process enhance --input photo.jpg --output enhanced.jpg --strength 0.8

# With preview loop:
# myapp-cli process enhance --input photo.jpg --output enhanced.jpg --strength 0.8 --preview
# [Agent reviews preview]
# myapp-cli process enhance --input photo.jpg --output enhanced.jpg --strength 0.6
```

### OpenClaw / Pi

```python
# Agents parse JSON output reliably
result = subprocess.run([
    'myapp-cli', 'export', '--input', 'file.myapp', '--format', 'JSON'
], capture_output=True, text=True)

data = json.loads(result.stdout)
if data['success']:
    output_path = data['output']
    # Continue workflow
```

## Publishing to CLI-Hub

### 1. Add to Registry

```bash
# Fork HKUDS/CLI-Anything
# Edit hub/registry.json

{
  "myapp": {
    "name": "myapp-cli",
    "version": "0.1.0",
    "description": "Agent interface for MyApp",
    "install_source": "pip",
    "package_name": "myapp-cli",
    "skill_md": "src/myapp/SKILL.md",
    "categories": ["graphics", "automation"],
    "maintainer": "your-github-username",
    "repo_url": "https://github.com/yourname/myapp-cli"
  }
}

# Submit PR
```

### 2. PyPI Publication

```bash
# Build package
python setup.py sdist bdist_wheel

# Upload to PyPI
pip install twine
twine upload dist/*

# Users can now:
# pip install myapp-cli
# cli-hub install myapp
```

## Troubleshooting

### Executable Not Found

```python
# Override executable path
export MYAPP_EXECUTABLE=/custom/path/to/myapp
myapp-cli --help

# Or configure permanently
myapp-cli config set executable_path /custom/path/to/myapp
```

### JSON Parsing Errors

```python
# Always validate JSON output in commands
try:
    result = {'success': True, 'data': value}
    click.echo(json.dumps(result, indent=2))
except TypeError as e:
    # Handle non-serializable objects
    result = {'success': False, 'error': f'Serialization error: {e}'}
    click.echo(json.dumps(result, indent=2))
```

### Preview Generation Fails

```python
# Ensure temp directory exists and is writable
import tempfile
preview_dir = tempfile.mkdtemp(prefix='myapp-preview-')
# Clean up after preview
import atexit
atexit.register(lambda: shutil.rmtree(preview_dir, ignore_errors=True))
```

### Testing with Different Software Versions

```python
# tests/conftest.py
import pytest
import subprocess

def get_myapp_version():
    """Detect installed MyApp version."""
    result = subprocess.run(['myapp', '--version'], capture_output=True, text=True)
    # Parse version
    return version

@pytest.fixture
def skip_if_version_below(min_version):
    """Skip test if MyApp version too old."""
    current = get_myapp_version()
    if current < min_version:
        pytest.skip(f"Requires MyApp >= {min_version}")
```

## Best Practices

1. **Always return JSON** - Agents rely on structured output
2. **Implement --preview** - Let agents validate before committing
3. **Use absolute paths** - Avoid ambiguity in file operations
4. **Validate inputs** - Check file existence, parameter ranges
5. **Handle errors gracefully** - Return `{"success": false, "error": "..."}` instead of crashing
6. **Document in SKILL.md** - Keep agent-facing docs updated
7. **Write E2E tests** - Verify full workflows work end-to-end
8. **Support environment variables** - Allow configuration without code changes

## Reference Documentation

- **Main Repo**: https://github.com/HKUDS/CLI-Anything
- **CLI Hub**: https://clianything.cc/
- **Contributing Guide**: CONTRIBUTING.md in repo
- **Harness Template**: See `src/template/` for starting point
- **Existing Harnesses**: Study `src/gimp/`, `src/blender/`, `src/inkscape/` for patterns
