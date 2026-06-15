---
name: narrator-ai-cli-video-narration
description: Create AI-narrated movie commentary videos using the narrator-ai-cli tool with automated script generation, voice synthesis, and video composition
triggers:
  - "create a movie narration video"
  - "generate a film commentary video"
  - "make a video with AI narration"
  - "create narration for a movie"
  - "generate a short drama video"
  - "make a commentary video with AI voice"
  - "create a movie recap video"
  - "generate automated video narration"
---

# narrator-ai-cli-video-narration

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

`narrator-ai-cli` is a command-line tool that automates the creation of movie narration and commentary videos. It handles the entire pipeline: searching movies, generating scripts, selecting background music and voiceovers, and composing final videos. The tool provides two workflow paths:

- **Original Narration**: Fast path for creating new commentary from scratch
- **Adapted Narration**: Standard path for adapting existing movie content

## Installation

Install via pip from the GitHub repository:

```bash
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"
```

**Requirements:**
- Python 3.10+
- Dependencies: typer, httpx[socks], httpx-sse, pyyaml, rich

## Configuration

### Set API Key

You must configure your API key before using the tool:

```bash
# Set the API key
narrator-ai-cli config set app_key YOUR_APP_KEY

# Verify configuration
narrator-ai-cli config show
```

The API key is stored in `~/.narrator-ai-cli/config.yaml` and can also be set via environment variable:

```bash
export NARRATOR_APP_KEY=your_api_key_here
```

### Configuration File Location

- **Linux/macOS**: `~/.narrator-ai-cli/config.yaml`
- **Windows**: `%USERPROFILE%\.narrator-ai-cli\config.yaml`

## Core Concepts

### Key Identifiers

- **file_id**: Unique identifier for uploaded or generated files (video clips, audio, scripts)
- **task_id**: Identifier for asynchronous tasks (script generation, video composition)
- **task_order_num**: Order number for tracking task status
- **movie_id**: Identifier for movies in the database
- **template_id**: Identifier for narration templates
- **bgm_id**: Background music identifier
- **dubbing_id**: Voice/dubbing identifier

### File Types

- **Video clips**: `.mp4`, `.mov`, `.avi` (source material)
- **Audio**: `.mp3`, `.wav` (BGM, voiceovers, TTS output)
- **Scripts**: `.txt`, `.json` (generated narration scripts)
- **Clip data**: `.json` (timeline and segment information)

## Key Commands

### Search and Browse

```bash
# Search for movies
narrator-ai-cli search-movie "Inception"

# List available templates
narrator-ai-cli list-template --page 1 --page_size 20

# List background music
narrator-ai-cli list-bgm --page 1 --page_size 50

# List available voices
narrator-ai-cli list-dubbing --page 1 --page_size 50

# List user templates
narrator-ai-cli list-user-template
```

### Original Narration Workflow (Fast Path)

This workflow creates new narration from scratch without adapting existing movies.

#### Step 0: Select Resources

```bash
# Select a template
narrator-ai-cli list-template --page 1

# Select BGM
narrator-ai-cli list-bgm --style "轻快" --page 1

# Select voice
narrator-ai-cli list-dubbing --gender "男" --page 1
```

#### Step 1: Generate Original Script

```bash
# Create a new original narration script task
narrator-ai-cli create-original-narration-script-task \
  --drama_name "都市霸总复仇记" \
  --drama_intro "一个被背叛的霸总重返巅峰的故事" \
  --template_id 12345

# Poll task status
narrator-ai-cli query-original-narration-script-task \
  --task_id abc123 \
  --task_order_num 1

# Download the generated script (when task completes)
narrator-ai-cli download-file --file_id def456 --output script.txt
```

#### Step 2: Create Original Clip Data

```bash
# Generate clip/timeline data from the script
narrator-ai-cli create-original-clip-data-task \
  --narration_script_file_id def456 \
  --drama_name "都市霸总复仇记"

# Poll until complete
narrator-ai-cli query-original-clip-data-task \
  --task_id xyz789 \
  --task_order_num 2

# Download clip data
narrator-ai-cli download-file --file_id ghi012 --output clip_data.json
```

#### Step 3: Text-to-Speech

```bash
# Convert script to speech
narrator-ai-cli create-tts-task \
  --text_file_id def456 \
  --dubbing_id 67890

# Poll TTS task
narrator-ai-cli query-tts-task \
  --task_id tts123 \
  --task_order_num 3

# Download audio
narrator-ai-cli download-file --file_id jkl345 --output narration.mp3
```

#### Step 4: Compose Final Video

```bash
# Compose the final video
narrator-ai-cli create-original-compose-task \
  --dubbing_file_id jkl345 \
  --clip_data_file_id ghi012 \
  --bgm_id 11111 \
  --template_id 12345

# Poll compose task
narrator-ai-cli query-original-compose-task \
  --task_id comp456 \
  --task_order_num 4

# Download final video
narrator-ai-cli download-file --file_id mno678 --output final_video.mp4
```

### Adapted Narration Workflow (Standard Path)

This workflow adapts existing movie content into narration videos.

#### Step 0: Search and Select Movie

```bash
# Find the movie
narrator-ai-cli search-movie "The Shawshank Redemption"

# Record the movie_id from results
```

#### Step 1: Generate Adapted Script

```bash
# Create adapted narration script task
narrator-ai-cli create-adapted-narration-script-task \
  --movie_id 99999 \
  --template_id 12345

# Poll task
narrator-ai-cli query-adapted-narration-script-task \
  --task_id adp123 \
  --task_order_num 1

# Download script
narrator-ai-cli download-file --file_id scr789 --output adapted_script.txt
```

#### Step 2: Create Adapted Clip Data

```bash
# Generate clip data for adapted narration
narrator-ai-cli create-adapted-clip-data-task \
  --narration_script_file_id scr789 \
  --movie_id 99999

# Poll task
narrator-ai-cli query-adapted-clip-data-task \
  --task_id clp456 \
  --task_order_num 2

# Download clip data
narrator-ai-cli download-file --file_id cld123 --output adapted_clip_data.json
```

#### Step 3: Text-to-Speech

```bash
# Same as Original workflow
narrator-ai-cli create-tts-task \
  --text_file_id scr789 \
  --dubbing_id 67890

narrator-ai-cli query-tts-task \
  --task_id tts789 \
  --task_order_num 3

narrator-ai-cli download-file --file_id aud456 --output adapted_narration.mp3
```

#### Step 4: Upload Source Video

```bash
# Upload the source movie file
narrator-ai-cli upload-file --file_path /path/to/movie.mp4

# Record the returned file_id
```

#### Step 5: Compose Adapted Video

```bash
# Compose final video with source material
narrator-ai-cli create-adapted-compose-task \
  --dubbing_file_id aud456 \
  --clip_data_file_id cld123 \
  --source_video_file_id vid789 \
  --bgm_id 11111 \
  --template_id 12345

# Poll compose task
narrator-ai-cli query-adapted-compose-task \
  --task_id acp999 \
  --task_order_num 5

# Download final video
narrator-ai-cli download-file --file_id fin000 --output adapted_final.mp4
```

### Standalone Tasks

#### Voice Cloning

```bash
# Upload reference audio for voice cloning
narrator-ai-cli upload-file --file_path reference_voice.mp3

# Create voice clone task
narrator-ai-cli create-voice-clone-task \
  --audio_file_id ref123 \
  --voice_name "Custom Voice"

# Poll task
narrator-ai-cli query-voice-clone-task \
  --task_id vcl123 \
  --task_order_num 1

# The cloned voice gets a new dubbing_id for future use
```

#### Direct TTS (without full pipeline)

```bash
# Upload a text file
narrator-ai-cli upload-file --file_path script.txt

# Generate speech
narrator-ai-cli create-tts-task \
  --text_file_id txt123 \
  --dubbing_id 67890

narrator-ai-cli query-tts-task \
  --task_id tts456 \
  --task_order_num 1

narrator-ai-cli download-file --file_id tts789 --output speech.mp3
```

## Common Patterns

### Complete Original Narration Pipeline

```python
#!/usr/bin/env python3
import subprocess
import json
import time

def run_cmd(cmd):
    """Run CLI command and return output"""
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout.strip()

def poll_task(query_cmd, task_id, order_num):
    """Poll task until completion"""
    while True:
        output = run_cmd(f"{query_cmd} --task_id {task_id} --task_order_num {order_num}")
        data = json.loads(output)
        
        if data['status'] == 'completed':
            return data['file_id']
        elif data['status'] == 'failed':
            raise Exception(f"Task failed: {data.get('error')}")
        
        time.sleep(5)

# Step 1: Generate script
print("Generating script...")
script_task = json.loads(run_cmd(
    'narrator-ai-cli create-original-narration-script-task '
    '--drama_name "复仇之路" '
    '--drama_intro "一个关于复仇的故事" '
    '--template_id 12345'
))
script_file_id = poll_task(
    'narrator-ai-cli query-original-narration-script-task',
    script_task['task_id'],
    script_task['task_order_num']
)

# Step 2: Generate clip data
print("Generating clip data...")
clip_task = json.loads(run_cmd(
    f'narrator-ai-cli create-original-clip-data-task '
    f'--narration_script_file_id {script_file_id} '
    f'--drama_name "复仇之路"'
))
clip_file_id = poll_task(
    'narrator-ai-cli query-original-clip-data-task',
    clip_task['task_id'],
    clip_task['task_order_num']
)

# Step 3: Generate TTS
print("Generating voice...")
tts_task = json.loads(run_cmd(
    f'narrator-ai-cli create-tts-task '
    f'--text_file_id {script_file_id} '
    f'--dubbing_id 67890'
))
audio_file_id = poll_task(
    'narrator-ai-cli query-tts-task',
    tts_task['task_id'],
    tts_task['task_order_num']
)

# Step 4: Compose video
print("Composing final video...")
compose_task = json.loads(run_cmd(
    f'narrator-ai-cli create-original-compose-task '
    f'--dubbing_file_id {audio_file_id} '
    f'--clip_data_file_id {clip_file_id} '
    f'--bgm_id 11111 '
    f'--template_id 12345'
))
video_file_id = poll_task(
    'narrator-ai-cli query-original-compose-task',
    compose_task['task_id'],
    compose_task['task_order_num']
)

# Download final video
print("Downloading video...")
run_cmd(f'narrator-ai-cli download-file --file_id {video_file_id} --output final.mp4')
print("Done! Video saved as final.mp4")
```

### Resource Selection Strategy

```bash
#!/bin/bash

# Function to select resources interactively
select_resources() {
    echo "=== Selecting Template ==="
    narrator-ai-cli list-template --page 1 | jq '.templates[] | {id, name, style}'
    read -p "Enter template_id: " TEMPLATE_ID
    
    echo -e "\n=== Selecting BGM ==="
    narrator-ai-cli list-bgm --style "轻快" --page 1 | jq '.bgm[] | {id, name, style}'
    read -p "Enter bgm_id: " BGM_ID
    
    echo -e "\n=== Selecting Voice ==="
    narrator-ai-cli list-dubbing --gender "男" --page 1 | jq '.voices[] | {id, name, gender}'
    read -p "Enter dubbing_id: " DUBBING_ID
    
    echo -e "\nSelected resources:"
    echo "Template: $TEMPLATE_ID"
    echo "BGM: $BGM_ID"
    echo "Voice: $DUBBING_ID"
}

select_resources
```

### Batch Processing Multiple Movies

```python
#!/usr/bin/env python3
import subprocess
import json

movies = [
    {"name": "Inception", "template_id": 12345},
    {"name": "The Matrix", "template_id": 12346},
    {"name": "Interstellar", "template_id": 12347},
]

for movie in movies:
    print(f"\nProcessing {movie['name']}...")
    
    # Search movie
    search_result = subprocess.run(
        ['narrator-ai-cli', 'search-movie', movie['name']],
        capture_output=True, text=True
    )
    movie_data = json.loads(search_result.stdout)
    
    if not movie_data.get('movies'):
        print(f"Movie {movie['name']} not found, skipping...")
        continue
    
    movie_id = movie_data['movies'][0]['id']
    
    # Create adapted script task
    script_task = subprocess.run(
        ['narrator-ai-cli', 'create-adapted-narration-script-task',
         '--movie_id', str(movie_id),
         '--template_id', str(movie['template_id'])],
        capture_output=True, text=True
    )
    
    task_info = json.loads(script_task.stdout)
    print(f"Started script generation: {task_info['task_id']}")
    
    # Continue with remaining steps...
```

## Troubleshooting

### Common Error Codes

| Code | Error | Solution |
|------|-------|----------|
| 1001 | Invalid API key | Verify `narrator-ai-cli config show` or check `$NARRATOR_APP_KEY` |
| 1002 | Insufficient credits | Contact support to add credits |
| 2001 | File not found | Check file_id is correct and file exists |
| 2002 | Invalid file format | Ensure file format matches API requirements (.mp4 for video, .mp3/.wav for audio) |
| 3001 | Task not found | Verify task_id and task_order_num are correct |
| 3002 | Task failed | Check task error message with query command |
| 4001 | Movie not found | Use `search-movie` to find valid movie_id |
| 5001 | Template not found | Use `list-template` to find valid template_id |
| 6001 | Invalid parameters | Review command syntax and required parameters |

### Task Polling Best Practices

```python
import time
import json
import subprocess

def safe_poll_task(query_command, task_id, order_num, max_attempts=120, interval=5):
    """
    Safely poll a task with timeout and error handling
    
    Args:
        query_command: Base CLI command (e.g., 'narrator-ai-cli query-tts-task')
        task_id: Task identifier
        order_num: Task order number
        max_attempts: Maximum polling attempts (default 120 = 10 minutes at 5s interval)
        interval: Seconds between polls
    
    Returns:
        dict: Task result data
    """
    for attempt in range(max_attempts):
        try:
            result = subprocess.run(
                [*query_command.split(), '--task_id', task_id, '--task_order_num', str(order_num)],
                capture_output=True,
                text=True,
                timeout=30
            )
            
            if result.returncode != 0:
                print(f"Query failed: {result.stderr}")
                time.sleep(interval)
                continue
            
            data = json.loads(result.stdout)
            status = data.get('status')
            
            if status == 'completed':
                return data
            elif status == 'failed':
                raise Exception(f"Task failed: {data.get('error_message', 'Unknown error')}")
            elif status == 'processing':
                print(f"Attempt {attempt + 1}/{max_attempts}: Task still processing...")
            
        except subprocess.TimeoutExpired:
            print(f"Query timeout on attempt {attempt + 1}")
        except json.JSONDecodeError as e:
            print(f"Invalid JSON response: {e}")
        
        time.sleep(interval)
    
    raise TimeoutError(f"Task did not complete within {max_attempts * interval} seconds")

# Usage
try:
    result = safe_poll_task(
        'narrator-ai-cli query-tts-task',
        'task_abc123',
        1,
        max_attempts=60,
        interval=10
    )
    print(f"Task completed! File ID: {result['file_id']}")
except TimeoutError as e:
    print(f"Timeout: {e}")
except Exception as e:
    print(f"Error: {e}")
```

### File Download Issues

```bash
# Check file exists before downloading
narrator-ai-cli list-files | grep "file_id_here"

# Download with explicit output path
mkdir -p output
narrator-ai-cli download-file \
  --file_id abc123 \
  --output output/video_$(date +%Y%m%d_%H%M%S).mp4

# Verify download succeeded
if [ -f output/video_*.mp4 ]; then
    echo "Download successful"
    ls -lh output/video_*.mp4
else
    echo "Download failed"
fi
```

### Handling Network Errors

```python
import subprocess
import time

def retry_command(cmd, max_retries=3, backoff=2):
    """Retry CLI command with exponential backoff"""
    for attempt in range(max_retries):
        try:
            result = subprocess.run(
                cmd.split(),
                capture_output=True,
                text=True,
                timeout=60
            )
            
            if result.returncode == 0:
                return result.stdout
            
            print(f"Attempt {attempt + 1} failed: {result.stderr}")
            
        except subprocess.TimeoutExpired:
            print(f"Attempt {attempt + 1} timed out")
        
        if attempt < max_retries - 1:
            wait = backoff ** attempt
            print(f"Retrying in {wait} seconds...")
            time.sleep(wait)
    
    raise Exception(f"Command failed after {max_retries} attempts")

# Usage
output = retry_command('narrator-ai-cli search-movie Inception', max_retries=5)
```

## Important Notes

1. **Task Polling**: All `create-*-task` commands are asynchronous. Always poll with the corresponding `query-*-task` command until status is `completed`.

2. **File ID Chaining**: Each step produces a `file_id` that feeds into the next step. Track these carefully:
   - Script file_id → Clip data task
   - Script file_id → TTS task
   - Clip data file_id + Audio file_id → Compose task

3. **Resource Selection Order**: 
   - Template first (defines narration style)
   - BGM second (matches template mood)
   - Voice third (matches template tone)

4. **Original vs Adapted**: Choose workflow based on source material:
   - Original: Creating from scratch (short dramas, new content)
   - Adapted: Using existing movies from database

5. **API Rate Limits**: The API may have rate limits. Implement exponential backoff for retries.

6. **File Retention**: Uploaded and generated files are retained for a limited time. Download important outputs promptly.

7. **Template Compatibility**: Not all templates work with all movie types. Check template descriptions for genre compatibility.

## Environment Variables

```bash
# API Authentication
export NARRATOR_APP_KEY=your_api_key_here

# Optional: Custom config location
export NARRATOR_CLI_CONFIG_DIR=/custom/path/to/config

# Optional: HTTP proxy
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
```

## Working with JSON Output

All commands return JSON output. Use `jq` for parsing:

```bash
# Pretty print search results
narrator-ai-cli search-movie "Inception" | jq '.'

# Extract specific fields
narrator-ai-cli list-template --page 1 | jq '.templates[] | {id, name}'

# Filter by criteria
narrator-ai-cli list-bgm --page 1 | jq '.bgm[] | select(.style == "史诗")'

# Count results
narrator-ai-cli list-dubbing --page 1 | jq '.voices | length'
```

## Agent Integration Tips

When helping users create narration videos:

1. **Always confirm resources before starting**: Show template, BGM, and voice options and let user choose
2. **Provide progress updates**: Narrate each step completion and estimated time remaining
3. **Save intermediate file IDs**: Store script_file_id, clip_file_id, etc. in conversation context
4. **Estimate costs**: Warn user about credit consumption before creating tasks
5. **Handle failures gracefully**: If a task fails, suggest alternative templates or parameters
6. **Offer previews**: After script generation, offer to show script content before proceeding
7. **Batch operations**: When user requests multiple videos, create tasks in parallel but track separately
