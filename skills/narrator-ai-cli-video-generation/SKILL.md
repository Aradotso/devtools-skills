---
name: narrator-ai-cli-video-generation
description: Generate AI-narrated movie commentary videos using narrator-ai-cli through natural language commands
triggers:
  - "create a movie narration video"
  - "generate a video commentary for a film"
  - "make a narrated video using narrator-ai"
  - "produce an AI commentary video"
  - "convert a movie into a narration video"
  - "create short drama narration content"
  - "generate film commentary with AI voices"
  - "make a video with AI narrator for this movie"
---

# narrator-ai-cli Video Generation Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

This skill teaches AI agents how to use **narrator-ai-cli**, a command-line tool for automated movie narration video production. The tool provides end-to-end video creation: from selecting movies and templates to generating scripts, composing clips, and exporting final videos.

## What It Does

narrator-ai-cli automates the creation of movie commentary/narration videos by:
- Searching and selecting from ~100 pre-configured movies
- Choosing narration styles (comedy, suspense, emotional, etc.) from 90+ templates
- Selecting background music from 146 BGM tracks
- Picking AI dubbing voices from 63 options
- Generating scripts automatically or using custom scripts
- Composing video clips with narration overlays
- Exporting final videos with download links

Two main workflows:
1. **Fast Path (Original Narration)**: Use pre-existing scripts and clip data for hot drama content
2. **Standard Path (Adapted Narration)**: Generate new scripts and clip data for custom narrations

## Installation

### Install the CLI

```bash
# Via pip from GitHub
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"

# Or clone and install locally
git clone https://github.com/NarratorAI-Studio/narrator-ai-cli.git
cd narrator-ai-cli
pip install -e .
```

Requirements:
- Python 3.10+
- Dependencies: typer, httpx[socks], httpx-sse, pyyaml, rich

### Configure API Key

```bash
# Set your API key (contact merlinyang@gridltd.com to obtain)
narrator-ai-cli config set app_key YOUR_APP_KEY_HERE

# Verify configuration
narrator-ai-cli config show
```

Environment variable alternative:
```bash
export NARRATOR_APP_KEY=YOUR_APP_KEY_HERE
```

### Verify Installation

```bash
narrator-ai-cli --version
narrator-ai-cli --help
```

## Core Concepts

### Key Identifiers
- **file_id**: Unique identifier for movies in the library (e.g., `10001`)
- **task_id**: Identifier returned when creating tasks (script generation, video composition)
- **task_order_num**: Specific order number for video composition tasks
- **drama_id**: Identifier for short drama content

### Workflows

**Fast Path (Original Narration)**:
- Uses pre-existing scripts and clip data
- Best for hot drama content and quick turnaround
- Steps: Select movie → Choose template → Select BGM/voice → Compose video

**Standard Path (Adapted Narration)**:
- Generates new scripts and clip data
- Full customization of narration style
- Steps: Select movie → Choose template → Generate script → Generate clips → Compose video

## Key Commands

### 1. Search and List Movies

```bash
# Search for movies by keyword
narrator-ai-cli movie search "Shawshank"

# List all available movies
narrator-ai-cli movie list

# Get detailed movie info
narrator-ai-cli movie info --file-id 10001
```

### 2. List Resources

```bash
# List narration templates (90+ styles)
narrator-ai-cli template list

# List BGM tracks (146 options)
narrator-ai-cli bgm list

# List dubbing voices (63 options)
narrator-ai-cli dubbing list

# List visual templates
narrator-ai-cli visual-template list
```

### 3. Fast Path: Original Narration (Hot Drama)

```python
# Step 1: Search for a movie
narrator-ai-cli movie search "热门剧集名称"

# Step 2: Select template (e.g., comedy style)
narrator-ai-cli template list | grep "搞笑"

# Step 3: Select BGM
narrator-ai-cli bgm list | grep "轻快"

# Step 4: Compose video directly (uses pre-existing script/clips)
narrator-ai-cli video compose \
  --file-id 10001 \
  --template-id 1001 \
  --bgm-id 2001 \
  --dubbing-id 3001

# Step 5: Check task status
narrator-ai-cli task status --task-id <returned_task_id>
```

### 4. Standard Path: Adapted Narration

```bash
# Step 1: Generate script
narrator-ai-cli script generate \
  --file-id 10001 \
  --template-id 1001

# Step 2: Check script task status and get script content
narrator-ai-cli task status --task-id <script_task_id>

# Step 3: Generate clip data
narrator-ai-cli clip generate \
  --file-id 10001 \
  --script "Your narration script here..."

# Step 4: Check clip task status
narrator-ai-cli task status --task-id <clip_task_id>

# Step 5: Compose video
narrator-ai-cli video compose \
  --file-id 10001 \
  --template-id 1001 \
  --bgm-id 2001 \
  --dubbing-id 3001 \
  --clip-data-path ./clip_data.json

# Step 6: Check video task status
narrator-ai-cli task status --task-id <video_task_id>
```

### 5. Standalone Tasks

```bash
# Voice cloning (upload reference audio)
narrator-ai-cli voice clone \
  --audio-file ./reference_voice.mp3 \
  --voice-name "My Custom Voice"

# Text-to-speech generation
narrator-ai-cli tts generate \
  --text "Your narration text here" \
  --dubbing-id 3001 \
  --output ./output_audio.mp3
```

## Python API Usage

```python
import subprocess
import json
import time

def create_narration_video(movie_keyword: str, style: str = "comedy"):
    """
    Complete workflow to create a narration video.
    
    Args:
        movie_keyword: Keyword to search for movie
        style: Narration style (comedy, suspense, emotional, etc.)
    """
    
    # Step 1: Search for movie
    result = subprocess.run(
        ["narrator-ai-cli", "movie", "search", movie_keyword],
        capture_output=True,
        text=True
    )
    movies = json.loads(result.stdout)
    if not movies:
        raise ValueError(f"No movies found for '{movie_keyword}'")
    
    file_id = movies[0]["file_id"]
    print(f"Selected movie: {movies[0]['title']} (file_id: {file_id})")
    
    # Step 2: Find template by style
    result = subprocess.run(
        ["narrator-ai-cli", "template", "list"],
        capture_output=True,
        text=True
    )
    templates = json.loads(result.stdout)
    template = next((t for t in templates if style in t["name"].lower()), templates[0])
    template_id = template["template_id"]
    print(f"Selected template: {template['name']} (id: {template_id})")
    
    # Step 3: Select default BGM and dubbing
    result = subprocess.run(
        ["narrator-ai-cli", "bgm", "list"],
        capture_output=True,
        text=True
    )
    bgms = json.loads(result.stdout)
    bgm_id = bgms[0]["bgm_id"]
    
    result = subprocess.run(
        ["narrator-ai-cli", "dubbing", "list"],
        capture_output=True,
        text=True
    )
    dubbings = json.loads(result.stdout)
    dubbing_id = dubbings[0]["dubbing_id"]
    
    # Step 4: Generate script
    result = subprocess.run(
        ["narrator-ai-cli", "script", "generate",
         "--file-id", str(file_id),
         "--template-id", str(template_id)],
        capture_output=True,
        text=True
    )
    script_task = json.loads(result.stdout)
    script_task_id = script_task["task_id"]
    print(f"Script generation started (task_id: {script_task_id})")
    
    # Step 5: Wait for script completion
    script_content = None
    for _ in range(30):  # Poll for up to 5 minutes
        time.sleep(10)
        result = subprocess.run(
            ["narrator-ai-cli", "task", "status", "--task-id", script_task_id],
            capture_output=True,
            text=True
        )
        status = json.loads(result.stdout)
        
        if status["status"] == "completed":
            script_content = status["result"]["script"]
            print("Script generation completed")
            break
        elif status["status"] == "failed":
            raise RuntimeError(f"Script generation failed: {status.get('error')}")
    
    if not script_content:
        raise TimeoutError("Script generation timed out")
    
    # Step 6: Generate clip data
    result = subprocess.run(
        ["narrator-ai-cli", "clip", "generate",
         "--file-id", str(file_id),
         "--script", script_content],
        capture_output=True,
        text=True
    )
    clip_task = json.loads(result.stdout)
    clip_task_id = clip_task["task_id"]
    print(f"Clip generation started (task_id: {clip_task_id})")
    
    # Step 7: Wait for clip data completion
    clip_data_path = None
    for _ in range(30):
        time.sleep(10)
        result = subprocess.run(
            ["narrator-ai-cli", "task", "status", "--task-id", clip_task_id],
            capture_output=True,
            text=True
        )
        status = json.loads(result.stdout)
        
        if status["status"] == "completed":
            clip_data_path = status["result"]["clip_data_path"]
            print("Clip generation completed")
            break
        elif status["status"] == "failed":
            raise RuntimeError(f"Clip generation failed: {status.get('error')}")
    
    if not clip_data_path:
        raise TimeoutError("Clip generation timed out")
    
    # Step 8: Compose video
    result = subprocess.run(
        ["narrator-ai-cli", "video", "compose",
         "--file-id", str(file_id),
         "--template-id", str(template_id),
         "--bgm-id", str(bgm_id),
         "--dubbing-id", str(dubbing_id),
         "--clip-data-path", clip_data_path],
        capture_output=True,
        text=True
    )
    video_task = json.loads(result.stdout)
    video_task_id = video_task["task_id"]
    print(f"Video composition started (task_id: {video_task_id})")
    
    # Step 9: Wait for video completion
    video_url = None
    for _ in range(60):  # Poll for up to 10 minutes
        time.sleep(10)
        result = subprocess.run(
            ["narrator-ai-cli", "task", "status", "--task-id", video_task_id],
            capture_output=True,
            text=True
        )
        status = json.loads(result.stdout)
        
        if status["status"] == "completed":
            video_url = status["result"]["video_url"]
            print(f"Video completed! Download: {video_url}")
            break
        elif status["status"] == "failed":
            raise RuntimeError(f"Video composition failed: {status.get('error')}")
        
        # Show progress
        progress = status.get("progress", 0)
        print(f"Progress: {progress}%")
    
    if not video_url:
        raise TimeoutError("Video composition timed out")
    
    return video_url

# Usage example
if __name__ == "__main__":
    video_url = create_narration_video("肖申克的救赎", style="comedy")
    print(f"Final video URL: {video_url}")
```

## Common Patterns

### Batch Video Generation

```python
import subprocess
import json

def batch_create_videos(movie_keywords: list[str], template_id: int):
    """Create multiple videos with the same template."""
    video_tasks = []
    
    for keyword in movie_keywords:
        # Search movie
        result = subprocess.run(
            ["narrator-ai-cli", "movie", "search", keyword],
            capture_output=True, text=True
        )
        movies = json.loads(result.stdout)
        if not movies:
            continue
        
        file_id = movies[0]["file_id"]
        
        # Start video composition (fast path if available)
        result = subprocess.run(
            ["narrator-ai-cli", "video", "compose",
             "--file-id", str(file_id),
             "--template-id", str(template_id)],
            capture_output=True, text=True
        )
        task = json.loads(result.stdout)
        video_tasks.append({
            "movie": movies[0]["title"],
            "task_id": task["task_id"]
        })
    
    return video_tasks
```

### Cost Estimation Before Creation

```bash
# Check cost for a video composition task
narrator-ai-cli cost estimate \
  --task-type video_compose \
  --file-id 10001 \
  --template-id 1001
```

### Custom Script with Voice Cloning

```bash
# Clone a voice
narrator-ai-cli voice clone \
  --audio-file ./my_voice.mp3 \
  --voice-name "MyVoice"

# Use cloned voice in video
narrator-ai-cli video compose \
  --file-id 10001 \
  --template-id 1001 \
  --dubbing-id <cloned_voice_id> \
  --script "Custom narration script..."
```

## Configuration

Configuration is stored in `~/.narrator-ai-cli/config.yaml`:

```yaml
app_key: YOUR_APP_KEY
api_endpoint: https://api.narrator-ai.com/v1
timeout: 300
max_retries: 3
```

Modify via CLI:
```bash
narrator-ai-cli config set api_endpoint https://custom-endpoint.com
narrator-ai-cli config set timeout 600
```

## Agent Usage Rules

When helping users with narrator-ai-cli:

1. **Always confirm before acting**: Show the user what movie, template, and settings will be used before starting long-running tasks.

2. **Check resource availability first**: Run `movie list`, `template list`, etc. to show options before asking for selections.

3. **Use appropriate workflow**: Fast Path for hot drama content, Standard Path for custom narrations.

4. **Poll task status**: Video composition can take 5-10 minutes. Poll every 10-20 seconds and show progress updates.

5. **Handle errors gracefully**: If a task fails, check the error code and suggest fixes (e.g., API quota exceeded, invalid file_id).

6. **Resource selection order**:
   - Movie: User choice or search results
   - Template: Match user's desired style
   - BGM: Default or user preference
   - Dubbing: Default or user preference

7. **Data flow awareness**:
   - Script generation output → Clip generation input
   - Clip generation output → Video composition input
   - Each step depends on the previous completing successfully

## Troubleshooting

### Common Errors

**Error: "API key not configured"**
```bash
# Solution: Set API key
narrator-ai-cli config set app_key YOUR_APP_KEY
```

**Error: "Movie not found (file_id: X)"**
```bash
# Solution: Verify file_id exists
narrator-ai-cli movie list | grep "10001"
```

**Error: "Task failed with status: quota_exceeded"**
```bash
# Solution: Check account quota
narrator-ai-cli account quota
# Or contact support to increase quota
```

**Error: "Clip data not found"**
```bash
# Solution: Ensure script generation completed successfully
narrator-ai-cli task status --task-id <script_task_id>
# Then regenerate clip data with the correct script
```

### Task Status Polling Pattern

```python
import time
import subprocess
import json

def wait_for_task(task_id: str, timeout: int = 600, poll_interval: int = 10):
    """Wait for a task to complete with proper error handling."""
    start_time = time.time()
    
    while True:
        if time.time() - start_time > timeout:
            raise TimeoutError(f"Task {task_id} timed out after {timeout}s")
        
        result = subprocess.run(
            ["narrator-ai-cli", "task", "status", "--task-id", task_id],
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            raise RuntimeError(f"Failed to check task status: {result.stderr}")
        
        status = json.loads(result.stdout)
        
        if status["status"] == "completed":
            return status["result"]
        elif status["status"] == "failed":
            error_msg = status.get("error", "Unknown error")
            raise RuntimeError(f"Task failed: {error_msg}")
        
        # Show progress if available
        if "progress" in status:
            print(f"Progress: {status['progress']}%")
        
        time.sleep(poll_interval)
```

### Debugging Tips

```bash
# Enable verbose logging
narrator-ai-cli --verbose video compose --file-id 10001 --template-id 1001

# Check API connectivity
narrator-ai-cli doctor

# View recent task history
narrator-ai-cli task list --limit 10
```

## Links

- GitHub: https://github.com/NarratorAI-Studio/narrator-ai-cli
- API Documentation: https://ceex7z9m67.feishu.cn/wiki/WLPnwBysairenFkZDbicZOfKnbc
- Contact: merlinyang@gridltd.com
