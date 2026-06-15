---
name: narrator-ai-cli-video-narration
description: Create AI-narrated movie commentary videos using narrator-ai-cli through natural language
triggers:
  - "create a movie narration video"
  - "generate narration for a film"
  - "make a short drama commentary video"
  - "produce an AI narration using narrator-ai-cli"
  - "search for available movies to narrate"
  - "show me narration templates and voices"
  - "create multiple narration videos in batch"
  - "clone a voice for narration"
---

# narrator-ai-cli-video-narration

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`narrator-ai-cli` is a command-line tool for automated movie narration video production. It provides two main workflows:

1. **Fast Path (Original Narration)**: Create narration videos from scratch with custom scripts
2. **Standard Path (Adapted Narration)**: Generate narration videos based on existing movie content with AI-generated scripts

The tool integrates movie search, template selection, BGM/voice selection, script generation, and video composition into a streamlined pipeline.

## Installation

```bash
# Install from GitHub
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"

# Verify installation
narrator-ai-cli --version
```

**Requirements:**
- Python 3.10+
- Dependencies: typer, httpx[socks], httpx-sse, pyyaml, rich

## Configuration

### Set API Key

```bash
# Configure your API key
narrator-ai-cli config set app_key $NARRATOR_APP_KEY

# View current configuration
narrator-ai-cli config show

# Set proxy if needed
narrator-ai-cli config set proxy "http://127.0.0.1:7890"
```

The API key is stored in `~/.narrator-ai-cli/config.yaml`. Environment variable `NARRATOR_APP_KEY` can be used to override.

**To obtain an API key**: Contact merlinyang@gridltd.com

## Core Commands

### Resource Discovery

```bash
# List all available movies (returns JSON with file_id, title, etc.)
narrator-ai-cli resource movie list

# Search for specific movies
narrator-ai-cli resource movie search "Shawshank"

# List background music tracks
narrator-ai-cli resource bgm list

# List dubbing voices
narrator-ai-cli resource dubbing list

# List narration templates
narrator-ai-cli resource template list
```

### Fast Path: Original Narration

Creates narration videos from custom scripts without requiring existing movie content.

```bash
# Step 1: Upload custom video material (optional)
narrator-ai-cli resource video upload /path/to/video.mp4

# Step 2: Create narration task with script
narrator-ai-cli magic-video create-narration \
  --title "My Custom Narration" \
  --script "Once upon a time, in a galaxy far away..." \
  --bgm-id "bgm_12345" \
  --dubbing-id "voice_67890" \
  --template-id "tpl_98765"

# Step 3: Poll task status
narrator-ai-cli magic-video query-task <task_id>

# Step 4: Download result when complete
narrator-ai-cli magic-video download-result <task_id> --output ./my_video.mp4
```

### Standard Path: Adapted Narration

Generates narration videos based on existing movies with AI-generated scripts.

```bash
# Step 1: Search and select movie
narrator-ai-cli resource movie search "Inception"
# Note the file_id from output

# Step 2: Generate narration script
narrator-ai-cli operations generate-narration-script \
  --file-id "movie_abc123" \
  --template-id "tpl_comedy_001"

# Step 3: Generate clip data
narrator-ai-cli operations generate-clip-data \
  --file-id "movie_abc123" \
  --narration-script-id "script_xyz789"

# Step 4: Create visual template
narrator-ai-cli operations create-visual-template \
  --clip-data-id "clip_data_456"

# Step 5: Compose final video
narrator-ai-cli magic-video compose-video \
  --file-id "movie_abc123" \
  --visual-template-id "visual_tpl_999" \
  --bgm-id "bgm_12345" \
  --dubbing-id "voice_67890"

# Step 6: Query and download
narrator-ai-cli magic-video query-task <task_id>
narrator-ai-cli magic-video download-result <task_id> --output ./narration.mp4
```

### Voice Cloning & TTS

```bash
# Clone a voice from audio file
narrator-ai-cli operations clone-voice \
  --audio-file /path/to/voice_sample.mp3 \
  --voice-name "Custom Voice"

# Use cloned voice for text-to-speech
narrator-ai-cli operations text-to-speech \
  --text "This is a test narration" \
  --voice-id "cloned_voice_123"

# Download TTS result
narrator-ai-cli operations download-audio <task_id> --output ./narration.mp3
```

## Key Concepts

### File IDs and Task IDs

- **file_id**: Identifier for movies in the resource library
- **task_id**: Unique identifier for async operations
- **task_order_num**: Order number for querying task results
- **narration_script_id**: ID for generated narration scripts
- **clip_data_id**: ID for video clip metadata
- **visual_template_id**: ID for visual composition templates

### Task Polling Pattern

Most operations are asynchronous. Use this pattern:

```bash
# Create task (returns task_id)
TASK_ID=$(narrator-ai-cli magic-video create-narration [...] | jq -r '.data.task_id')

# Poll until complete (status: pending -> processing -> completed/failed)
while true; do
  STATUS=$(narrator-ai-cli magic-video query-task $TASK_ID | jq -r '.data.status')
  if [[ "$STATUS" == "completed" ]] || [[ "$STATUS" == "failed" ]]; then
    break
  fi
  sleep 5
done

# Download result
narrator-ai-cli magic-video download-result $TASK_ID --output ./output.mp4
```

## Real-World Examples

### Example 1: Quick Movie Narration

```bash
#!/bin/bash
# Create a comedy narration for "The Shawshank Redemption"

# 1. Search movie
MOVIE_JSON=$(narrator-ai-cli resource movie search "Shawshank")
FILE_ID=$(echo $MOVIE_JSON | jq -r '.data[0].file_id')

# 2. Get comedy template
TEMPLATE_JSON=$(narrator-ai-cli resource template list)
COMEDY_TEMPLATE=$(echo $TEMPLATE_JSON | jq -r '.data[] | select(.tags | contains(["comedy"])) | .template_id' | head -1)

# 3. Get random BGM and voice
BGM_ID=$(narrator-ai-cli resource bgm list | jq -r '.data[0].bgm_id')
VOICE_ID=$(narrator-ai-cli resource dubbing list | jq -r '.data[0].dubbing_id')

# 4. Generate script
SCRIPT_TASK=$(narrator-ai-cli operations generate-narration-script \
  --file-id $FILE_ID \
  --template-id $COMEDY_TEMPLATE)
SCRIPT_ID=$(echo $SCRIPT_TASK | jq -r '.data.narration_script_id')

# 5. Generate clip data
CLIP_TASK=$(narrator-ai-cli operations generate-clip-data \
  --file-id $FILE_ID \
  --narration-script-id $SCRIPT_ID)
CLIP_ID=$(echo $CLIP_TASK | jq -r '.data.clip_data_id')

# 6. Create visual template
VISUAL_TASK=$(narrator-ai-cli operations create-visual-template \
  --clip-data-id $CLIP_ID)
VISUAL_ID=$(echo $VISUAL_TASK | jq -r '.data.visual_template_id')

# 7. Compose video
COMPOSE_TASK=$(narrator-ai-cli magic-video compose-video \
  --file-id $FILE_ID \
  --visual-template-id $VISUAL_ID \
  --bgm-id $BGM_ID \
  --dubbing-id $VOICE_ID)
TASK_ID=$(echo $COMPOSE_TASK | jq -r '.data.task_id')

# 8. Wait and download
echo "Task ID: $TASK_ID"
echo "Use: narrator-ai-cli magic-video query-task $TASK_ID"
echo "Then: narrator-ai-cli magic-video download-result $TASK_ID --output shawshank_narration.mp4"
```

### Example 2: Batch Production

```python
#!/usr/bin/env python3
import subprocess
import json
import time

def run_cli(cmd):
    """Execute CLI command and return JSON output"""
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return json.loads(result.stdout)

# Get list of action movies
movies = run_cli("narrator-ai-cli resource movie list")
action_movies = [m for m in movies['data'] if 'action' in m.get('tags', [])][:5]

# Get resources
templates = run_cli("narrator-ai-cli resource template list")['data']
bgm_list = run_cli("narrator-ai-cli resource bgm list")['data']
voices = run_cli("narrator-ai-cli resource dubbing list")['data']

tasks = []

for movie in action_movies:
    file_id = movie['file_id']
    template_id = templates[0]['template_id']
    bgm_id = bgm_list[0]['bgm_id']
    voice_id = voices[0]['dubbing_id']
    
    # Generate script
    script_result = run_cli(
        f"narrator-ai-cli operations generate-narration-script "
        f"--file-id {file_id} --template-id {template_id}"
    )
    script_id = script_result['data']['narration_script_id']
    
    # Generate clip data
    clip_result = run_cli(
        f"narrator-ai-cli operations generate-clip-data "
        f"--file-id {file_id} --narration-script-id {script_id}"
    )
    clip_id = clip_result['data']['clip_data_id']
    
    # Create visual template
    visual_result = run_cli(
        f"narrator-ai-cli operations create-visual-template "
        f"--clip-data-id {clip_id}"
    )
    visual_id = visual_result['data']['visual_template_id']
    
    # Compose video
    compose_result = run_cli(
        f"narrator-ai-cli magic-video compose-video "
        f"--file-id {file_id} --visual-template-id {visual_id} "
        f"--bgm-id {bgm_id} --dubbing-id {voice_id}"
    )
    task_id = compose_result['data']['task_id']
    
    tasks.append({
        'movie': movie['title'],
        'task_id': task_id
    })
    print(f"Started task for {movie['title']}: {task_id}")

# Monitor all tasks
print("\nMonitoring tasks...")
for task in tasks:
    while True:
        status = run_cli(f"narrator-ai-cli magic-video query-task {task['task_id']}")
        if status['data']['status'] in ['completed', 'failed']:
            print(f"{task['movie']}: {status['data']['status']}")
            if status['data']['status'] == 'completed':
                subprocess.run(
                    f"narrator-ai-cli magic-video download-result {task['task_id']} "
                    f"--output ./{task['movie'].replace(' ', '_')}.mp4",
                    shell=True
                )
            break
        time.sleep(10)
```

### Example 3: Custom Voice Narration

```bash
#!/bin/bash
# Create narration with custom cloned voice

# 1. Upload voice sample
CLONE_RESULT=$(narrator-ai-cli operations clone-voice \
  --audio-file ./voice_samples/my_voice.mp3 \
  --voice-name "My Custom Voice")

TASK_ID=$(echo $CLONE_RESULT | jq -r '.data.task_id')

# 2. Wait for voice cloning to complete
while true; do
  STATUS=$(narrator-ai-cli operations query-voice-clone $TASK_ID | jq -r '.data.status')
  if [[ "$STATUS" == "completed" ]]; then
    VOICE_ID=$(narrator-ai-cli operations query-voice-clone $TASK_ID | jq -r '.data.voice_id')
    break
  fi
  sleep 5
done

echo "Voice cloned: $VOICE_ID"

# 3. Create narration with custom voice
narrator-ai-cli magic-video create-narration \
  --title "Custom Voice Narration" \
  --script "This narration uses my custom cloned voice." \
  --dubbing-id $VOICE_ID \
  --bgm-id "bgm_12345" \
  --template-id "tpl_98765"
```

## Common Patterns

### Resource Selection Strategy

When selecting resources, follow this priority order:

1. **Templates**: Match user-specified style (comedy, drama, thriller, etc.) or use default
2. **BGM**: Match template mood or select from compatible genres
3. **Dubbing**: Match content type (male/female, age range, accent)

```bash
# Get templates by style
narrator-ai-cli resource template list | jq '.data[] | select(.tags | contains(["comedy"]))'

# Get BGM by genre
narrator-ai-cli resource bgm list | jq '.data[] | select(.genre == "upbeat")'

# Get voices by gender
narrator-ai-cli resource dubbing list | jq '.data[] | select(.gender == "male")'
```

### Error Handling

```bash
# Always check command exit status and error codes
RESULT=$(narrator-ai-cli magic-video create-narration [...] 2>&1)
EXIT_CODE=$?

if [ $EXIT_CODE -ne 0 ]; then
  ERROR_CODE=$(echo $RESULT | jq -r '.error.code // "unknown"')
  case $ERROR_CODE in
    "INSUFFICIENT_BALANCE")
      echo "Error: Insufficient API credits. Please top up."
      ;;
    "INVALID_FILE_ID")
      echo "Error: Movie not found. Use resource movie search first."
      ;;
    "TASK_NOT_FOUND")
      echo "Error: Task ID invalid or expired."
      ;;
    *)
      echo "Error: $RESULT"
      ;;
  esac
  exit 1
fi
```

### Cost Estimation

Before creating tasks, check your balance:

```bash
# Query account balance
BALANCE=$(narrator-ai-cli config show | grep -i balance || echo "Check via API")

# Estimate costs (typical ranges):
# - Script generation: ~0.1-0.5 credits
# - Clip data: ~0.2-1.0 credits
# - Visual template: ~0.3-0.8 credits
# - Video composition: ~1.0-5.0 credits
# - Voice cloning: ~2.0-10.0 credits
```

## Troubleshooting

### API Key Issues

```bash
# Problem: "Authentication failed"
# Solution: Verify API key is set
narrator-ai-cli config show

# Re-set if needed
narrator-ai-cli config set app_key $NARRATOR_APP_KEY
```

### Task Stuck in Processing

```bash
# Problem: Task status remains "processing" for >10 minutes
# Solution: Tasks typically complete in 2-5 minutes. Check:

# 1. Verify task exists
narrator-ai-cli magic-video query-task <task_id>

# 2. Check for error status
# If status is "failed", check error message in response

# 3. Contact support if truly stuck
```

### Movie Not Found

```bash
# Problem: "INVALID_FILE_ID" error
# Solution: Always search before using file_id

# Search first
narrator-ai-cli resource movie search "movie name"

# Verify file_id exists in results before using it
```

### Download Fails

```bash
# Problem: Downloaded video is corrupted or incomplete
# Solution: Verify task is fully completed before downloading

STATUS=$(narrator-ai-cli magic-video query-task <task_id> | jq -r '.data.status')
if [[ "$STATUS" != "completed" ]]; then
  echo "Task not ready. Current status: $STATUS"
  exit 1
fi

# Use --output flag explicitly
narrator-ai-cli magic-video download-result <task_id> --output ./my_video.mp4
```

### Network Issues

```bash
# Problem: Connection timeouts or slow responses
# Solution: Configure proxy if behind firewall

narrator-ai-cli config set proxy "http://proxy.example.com:8080"

# Or use environment variable
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="http://proxy.example.com:8080"
```

## Advanced Usage

### Custom Script Formatting

When providing custom scripts for original narration:

```bash
# Scripts should be plain text, UTF-8 encoded
# Optimal length: 150-500 words
# Include natural pauses with punctuation
# Avoid special characters that might break TTS

SCRIPT="In the beginning, there was darkness... [pause] Then came the light. 
Our story begins on a cold winter morning, when everything changed forever."

narrator-ai-cli magic-video create-narration \
  --title "Epic Story" \
  --script "$SCRIPT" \
  --dubbing-id "voice_dramatic_001" \
  --bgm-id "bgm_cinematic_002" \
  --template-id "tpl_epic_001"
```

### Combining Multiple Videos

```bash
# Create multiple short narrations and combine them

# Generate 3 different narrations
for i in {1..3}; do
  narrator-ai-cli magic-video create-narration \
    --title "Part $i" \
    --script "Content for part $i..." \
    --dubbing-id "voice_123" \
    --bgm-id "bgm_456" \
    --template-id "tpl_789"
done

# Download each part, then use ffmpeg to concatenate
# (ffmpeg not included in narrator-ai-cli)
```

### Filtering Resources

```bash
# Find specific voice types
narrator-ai-cli resource dubbing list | \
  jq '.data[] | select(.language == "en" and .gender == "female" and .age_range == "young")'

# Find BGM by duration
narrator-ai-cli resource bgm list | \
  jq '.data[] | select(.duration_seconds >= 60 and .duration_seconds <= 120)'

# Find templates by multiple tags
narrator-ai-cli resource template list | \
  jq '.data[] | select(.tags | contains(["comedy", "fast-paced"]))'
```

## Configuration Files

Configuration is stored in `~/.narrator-ai-cli/config.yaml`:

```yaml
app_key: your-api-key-here
proxy: null
api_endpoint: https://api.narrator-ai.example.com
timeout: 30
```

You can manually edit this file or use `narrator-ai-cli config set` commands.

## API Response Structure

All CLI commands return JSON with this structure:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    // Command-specific data
  },
  "error": {
    "code": "ERROR_CODE",
    "message": "Detailed error message"
  }
}
```

**Success**: `code: 0`  
**Failure**: `code: non-zero`, check `error` object

## Important Notes

1. **File IDs are not permanent**: Movie file IDs may change. Always search before using cached IDs.
2. **Tasks expire**: Task results are available for 7 days after completion.
3. **Polling interval**: Poll task status every 5-10 seconds, not faster (to avoid rate limiting).
4. **Language chains**: When generating scripts, specify template language must match movie language.
5. **Resource limits**: Free tier may have daily limits on task creation.
6. **Video format**: Output is always MP4, H.264 codec, 1080p resolution.
7. **Credits**: Each operation consumes API credits; monitor your balance to avoid interruptions.

## Reference Documentation

For detailed resource listings (all movies, BGM tracks, voices, templates), see:
- [Official Resource Documentation](https://ceex7z9m67.feishu.cn/wiki/WLPnwBysairenFkZDbicZOfKnbc)
- [CLI GitHub Repository](https://github.com/NarratorAI-Studio/narrator-ai-cli)

## Contact & Support

- Email: merlinyang@gridltd.com
- GitHub Issues: [narrator-ai-cli issues](https://github.com/NarratorAI-Studio/narrator-ai-cli/issues)
