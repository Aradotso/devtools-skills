---
name: narrator-ai-video-generation
description: Generate AI-narrated video content using narrator-ai-cli for movie commentary, short dramas, and film analysis
triggers:
  - "create a movie narration video"
  - "generate film commentary with narrator-ai"
  - "make a video narration for this movie"
  - "produce AI-narrated content"
  - "create short drama narration"
  - "generate video commentary using narrator-ai-cli"
  - "help me make a narrated movie clip"
  - "create narration video with custom voice"
---

# Narrator AI Video Generation Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

This skill enables AI agents to create professional movie narration videos using the `narrator-ai-cli` tool. The CLI provides access to ~100 movies, 146 BGM tracks, 63 dubbing voices, and 90+ narration templates for automated video production.

## What This Tool Does

`narrator-ai-cli` is a command-line interface for the Narrator AI service that automates the creation of movie narration videos. It handles:

- **Adapted Narration**: Select existing movie content and generate narration scripts
- **Original Narration**: Create narration from custom scripts and material
- **Voice Synthesis**: Text-to-speech with 63+ voice options
- **Voice Cloning**: Clone custom voices for narration
- **Video Composition**: Automatic video assembly with BGM, dubbing, and visual templates

## Installation

### Prerequisites

- Python 3.10 or higher
- pip package manager
- An API key from Narrator AI (contact merlinyang@gridltd.com)

### Install CLI

```bash
# Install from GitHub
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"

# Verify installation
narrator-ai-cli --version
```

### Configure API Key

```bash
# Set your API key (required for all operations)
narrator-ai-cli config set app_key $NARRATOR_APP_KEY

# Verify configuration
narrator-ai-cli config show
```

Environment variable setup:
```bash
export NARRATOR_APP_KEY="your-api-key-here"
```

## Core Commands

### Configuration

```bash
# Show current configuration
narrator-ai-cli config show

# Set API key
narrator-ai-cli config set app_key <key>

# Set base URL (optional)
narrator-ai-cli config set base_url <url>
```

### Resource Discovery

```bash
# List available movies
narrator-ai-cli movie list

# Search for specific movie
narrator-ai-cli movie list --keyword "Shawshank"

# List narration templates
narrator-ai-cli template list

# List available background music
narrator-ai-cli bgm list --keyword "epic"

# List dubbing voices
narrator-ai-cli dubbing list --gender male --language zh-CN
```

### Adapted Narration Workflow (Standard Path)

Use when working with existing movie content:

```bash
# Step 1: Upload reference file (optional, for custom content)
narrator-ai-cli file upload /path/to/reference.mp4

# Step 2: Create movie selection task
narrator-ai-cli movie-select create \
  --movie-id <movie_id> \
  --template-code <template_code> \
  --reference-file-id <file_id>  # optional

# Step 3: Poll for clip data result
narrator-ai-cli movie-select get <task_id>

# Step 4: Create narration script
narrator-ai-cli narration-script create \
  --clip-data-file-id <clip_data_file_id> \
  --template-code <template_code> \
  --requirement "Make it funny and engaging"

# Step 5: Poll for script result
narrator-ai-cli narration-script get <task_id>

# Step 6: Compose final video
narrator-ai-cli magic-video create \
  --clip-data-file-id <clip_data_file_id> \
  --narration-script-file-id <script_file_id> \
  --bgm-code <bgm_code> \
  --dubbing-code <dubbing_code>

# Step 7: Poll for video result
narrator-ai-cli magic-video get <task_order_num>
```

### Original Narration Workflow (Fast Path)

Use when creating from scratch with custom script:

```bash
# Step 1: Upload your video material
narrator-ai-cli file upload /path/to/video.mp4

# Step 2: Create narration script (without clip data)
narrator-ai-cli narration-script create \
  --template-code <template_code> \
  --requirement "Create a 60-second comedy narration"

# Step 3: Poll for script
narrator-ai-cli narration-script get <task_id>

# Step 4: Compose video with custom material
narrator-ai-cli magic-video create \
  --video-file-id <uploaded_file_id> \
  --narration-script-file-id <script_file_id> \
  --bgm-code <bgm_code> \
  --dubbing-code <dubbing_code>

# Step 5: Poll for result
narrator-ai-cli magic-video get <task_order_num>
```

### Standalone Features

```bash
# Text-to-Speech
narrator-ai-cli tts create \
  --text "Hello world" \
  --dubbing-code <dubbing_code>

narrator-ai-cli tts get <task_id>

# Voice Cloning
narrator-ai-cli voice-clone create \
  --audio-file-id <audio_file_id> \
  --voice-name "MyCustomVoice"

narrator-ai-cli voice-clone get <task_id>
```

## Common Patterns

### Pattern 1: Quick Movie Narration

```python
import subprocess
import json
import time

def create_quick_narration(movie_name, style="comedy"):
    # Search for movie
    result = subprocess.run(
        ["narrator-ai-cli", "movie", "list", "--keyword", movie_name],
        capture_output=True, text=True
    )
    movies = json.loads(result.stdout)
    movie_id = movies[0]["id"]
    
    # Get template
    result = subprocess.run(
        ["narrator-ai-cli", "template", "list"],
        capture_output=True, text=True
    )
    templates = json.loads(result.stdout)
    template = next(t for t in templates if style in t["name"].lower())
    
    # Create movie selection
    result = subprocess.run(
        ["narrator-ai-cli", "movie-select", "create",
         "--movie-id", movie_id,
         "--template-code", template["code"]],
        capture_output=True, text=True
    )
    task = json.loads(result.stdout)
    
    # Poll for completion
    while True:
        result = subprocess.run(
            ["narrator-ai-cli", "movie-select", "get", task["task_id"]],
            capture_output=True, text=True
        )
        status = json.loads(result.stdout)
        if status["status"] == "SUCCESS":
            return status["clip_data_file_id"]
        time.sleep(5)
```

### Pattern 2: Batch Video Creation

```python
def batch_create_narrations(movie_ids, template_code, bgm_code, dubbing_code):
    tasks = []
    
    for movie_id in movie_ids:
        # Create movie selection
        result = subprocess.run([
            "narrator-ai-cli", "movie-select", "create",
            "--movie-id", movie_id,
            "--template-code", template_code
        ], capture_output=True, text=True)
        
        task = json.loads(result.stdout)
        tasks.append(task)
    
    # Wait for all to complete
    clip_data_ids = []
    for task in tasks:
        while True:
            result = subprocess.run([
                "narrator-ai-cli", "movie-select", "get",
                task["task_id"]
            ], capture_output=True, text=True)
            
            status = json.loads(result.stdout)
            if status["status"] == "SUCCESS":
                clip_data_ids.append(status["clip_data_file_id"])
                break
            elif status["status"] == "FAILED":
                print(f"Task {task['task_id']} failed")
                break
            time.sleep(5)
    
    return clip_data_ids
```

### Pattern 3: Custom Voice Narration

```python
def create_with_custom_voice(audio_path, script_text, video_path):
    # Upload audio for cloning
    result = subprocess.run([
        "narrator-ai-cli", "file", "upload", audio_path
    ], capture_output=True, text=True)
    audio_file = json.loads(result.stdout)
    
    # Clone voice
    result = subprocess.run([
        "narrator-ai-cli", "voice-clone", "create",
        "--audio-file-id", audio_file["file_id"],
        "--voice-name", "CustomVoice"
    ], capture_output=True, text=True)
    clone_task = json.loads(result.stdout)
    
    # Wait for clone completion
    while True:
        result = subprocess.run([
            "narrator-ai-cli", "voice-clone", "get",
            clone_task["task_id"]
        ], capture_output=True, text=True)
        
        status = json.loads(result.stdout)
        if status["status"] == "SUCCESS":
            dubbing_code = status["dubbing_code"]
            break
        time.sleep(5)
    
    # Upload video
    result = subprocess.run([
        "narrator-ai-cli", "file", "upload", video_path
    ], capture_output=True, text=True)
    video_file = json.loads(result.stdout)
    
    # Create narration (simplified)
    # ... continue with magic-video create
```

## Key Concepts

### File IDs and Task IDs

- **file_id**: Identifier for uploaded files (videos, audio, scripts)
- **task_id**: Identifier for async tasks (movie-select, narration-script, etc.)
- **task_order_num**: Identifier for magic-video composition tasks
- **clip_data_file_id**: File containing selected movie clips and metadata
- **narration_script_file_id**: File containing generated narration script

### Task Status Flow

1. **PENDING**: Task created, waiting to start
2. **PROCESSING**: Task is being executed
3. **SUCCESS**: Task completed successfully
4. **FAILED**: Task failed (check error message)

Always poll tasks until status is SUCCESS or FAILED.

### Workflow Decision Tree

```
User Request
    ├─ Has specific movie? 
    │   ├─ Yes → Adapted Narration (Standard Path)
    │   │   └─ movie-select → narration-script → magic-video
    │   └─ No → Original Narration (Fast Path)
    │       └─ narration-script → magic-video
    │
    └─ Just voice/audio task?
        └─ voice-clone OR tts
```

## Configuration Options

### Template Codes

Templates define narration style. Common categories:
- Comedy/Humor templates
- Dramatic/Serious templates
- Action/Thriller templates
- Romance/Drama templates

Use `narrator-ai-cli template list` to see all available templates with codes.

### BGM Codes

Background music options include:
- Epic/cinematic tracks
- Emotional/romantic music
- Suspenseful/thriller music
- Light/comedy tracks

Use `narrator-ai-cli bgm list` to browse by keyword or mood.

### Dubbing Codes

Voice options include:
- **Gender**: male, female, neutral
- **Language**: zh-CN, en-US, etc.
- **Age**: young, middle-aged, elderly
- **Style**: professional, emotional, energetic

Use `narrator-ai-cli dubbing list --gender <gender> --language <lang>` to filter.

## Error Handling

### Common Error Codes

```python
ERROR_CODES = {
    4000: "Invalid parameters",
    4001: "Authentication failed - check API key",
    4003: "Insufficient credits",
    4004: "Resource not found",
    5000: "Server error - retry later",
    5001: "Task processing failed"
}

def handle_error(error_code, error_msg):
    if error_code == 4001:
        print("Check NARRATOR_APP_KEY environment variable")
    elif error_code == 4003:
        print("Contact support for credit top-up")
    elif error_code in [5000, 5001]:
        print("Temporary error, retry in 30 seconds")
    else:
        print(f"Error {error_code}: {error_msg}")
```

### Polling Best Practices

```python
def poll_task(task_id, command, max_retries=60, interval=5):
    """Poll task with exponential backoff"""
    for attempt in range(max_retries):
        result = subprocess.run(
            ["narrator-ai-cli", command, "get", task_id],
            capture_output=True, text=True
        )
        
        if result.returncode != 0:
            print(f"Command failed: {result.stderr}")
            return None
            
        status = json.loads(result.stdout)
        
        if status["status"] == "SUCCESS":
            return status
        elif status["status"] == "FAILED":
            print(f"Task failed: {status.get('error_msg')}")
            return None
        
        # Exponential backoff
        wait_time = min(interval * (1.5 ** (attempt // 10)), 30)
        time.sleep(wait_time)
    
    print("Task timed out")
    return None
```

## Agent Rules

When using this skill, AI agents MUST:

1. **Confirm before execution**: Show the user what will be created (movie, template, voice) before running commands
2. **Poll asynchronously**: Always poll task status until SUCCESS/FAILED
3. **Resource selection order**: 
   - Ask user for preferences first
   - Search resources by keyword
   - Present top 3 options
   - Use user selection or default to first result
4. **Cost awareness**: Warn if creating multiple videos (each costs credits)
5. **Error recovery**: If task fails, explain error and suggest alternatives
6. **File management**: Track file_ids and task_ids throughout conversation
7. **Language chain**: Match dubbing language to script language

## Troubleshooting

### "Authentication failed"

```bash
# Check if API key is set
narrator-ai-cli config show

# Re-set API key
narrator-ai-cli config set app_key $NARRATOR_APP_KEY
```

### "Task stuck in PROCESSING"

- Normal processing time: 2-10 minutes for video composition
- If >15 minutes: Contact support with task_id
- Retry: Create new task, don't retry same task_id

### "Invalid movie_id"

```bash
# Verify movie exists
narrator-ai-cli movie list --keyword "exact movie name"

# Use the returned movie_id, not movie name
```

### "File upload failed"

```bash
# Check file size (max 2GB)
ls -lh /path/to/file

# Check file format (mp4, mp3, wav supported)
file /path/to/file

# Try absolute path
narrator-ai-cli file upload "$(pwd)/video.mp4"
```

### Rate Limiting

- API has rate limits per API key
- If hit: Wait 60 seconds before retry
- For batch operations: Add 2-3 second delays between requests

## Output Files

All outputs are temporary URLs (valid 24-48 hours):

```python
# Download and save permanently
import requests

def download_result(url, output_path):
    response = requests.get(url, stream=True)
    with open(output_path, 'wb') as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
    print(f"Saved to {output_path}")
```

## Resources

- **CLI Repository**: https://github.com/NarratorAI-Studio/narrator-ai-cli
- **Resource Preview**: https://ceex7z9m67.feishu.cn/wiki/WLPnwBysairenFkZDbicZOfKnbc
- **API Documentation**: Contact support for detailed API docs
- **Support Email**: merlinyang@gridltd.com
