---
name: narrator-ai-cli-skill
description: Use narrator-ai-cli to create AI-powered movie narration videos through natural language commands
triggers:
  - "create a movie narration video"
  - "generate narration for a film"
  - "make a video commentary"
  - "use narrator-ai-cli"
  - "create video with movie clips and voiceover"
  - "generate AI narration video"
  - "search for movies to narrate"
  - "list available narration templates"
---

# narrator-ai-cli-skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

This skill enables AI agents to use `narrator-ai-cli`, a command-line tool for automated movie narration video production. The CLI wraps the NarratorAI API and provides commands for searching movies, selecting templates, generating scripts, and composing videos.

## What is narrator-ai-cli?

`narrator-ai-cli` is a Python CLI tool that automates the creation of movie narration videos (also called "film commentary" or "short drama" videos). It handles the entire pipeline: movie selection → template selection → script generation → video composition.

**Key capabilities:**
- Search ~100 built-in movies or upload custom content
- 90+ narration templates (comedy, suspense, emotional, etc.)
- 146 BGM tracks and 63 dubbing voices
- Two workflows: Adapted Narration (uses existing clips) and Original Narration (AI generates everything)
- Voice cloning and text-to-speech
- Full video composition pipeline

## Installation

Install via pip from the GitHub repository:

```bash
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"
```

**Requirements:**
- Python 3.10+
- Dependencies: typer, httpx[socks], httpx-sse, pyyaml, rich

**Verify installation:**

```bash
narrator-ai-cli --version
```

## Configuration

### Set API Key

The CLI requires an API key. Set it using the config command:

```bash
narrator-ai-cli config set app_key YOUR_APP_KEY
```

Or set the environment variable:

```bash
export NARRATOR_APP_KEY="your_api_key_here"
```

**To obtain an API key:** Contact merlinyang@gridltd.com

### View Current Configuration

```bash
narrator-ai-cli config show
```

## Core Concepts

Before using the CLI, understand these key terms:

- **file_id**: Unique identifier for uploaded files (movies, audio, images)
- **task_id**: Identifier for async tasks (script generation, video composition)
- **task_order_num**: Serial number for tracking task status
- **Adapted Narration**: Uses pre-existing movie clips (Fast Path)
- **Original Narration**: AI generates clips from scratch (Standard Path)
- **Hot Drama**: Select from ~100 built-in movies
- **Original Mix**: Upload your own movie file
- **New Drama**: Upload multiple video clips manually

## Key Commands

### 1. Resource Discovery

**List movies:**

```bash
narrator-ai-cli resource movie list
```

**Search for a specific movie:**

```bash
narrator-ai-cli resource movie list --search "Shawshank"
```

**List narration templates:**

```bash
narrator-ai-cli resource template list
```

**Filter templates by category:**

```bash
narrator-ai-cli resource template list --category comedy
```

**List BGM tracks:**

```bash
narrator-ai-cli resource bgm list
```

**List dubbing voices:**

```bash
narrator-ai-cli resource dubbing list
```

### 2. File Management

**Upload a movie file:**

```bash
narrator-ai-cli file upload /path/to/movie.mp4
```

Returns a `file_id` for use in subsequent commands.

**List uploaded files:**

```bash
narrator-ai-cli file list
```

### 3. Fast Path: Adapted Narration (Original Narration)

This workflow uses AI to generate the entire video from a movie file.

**Step 0: Upload movie (if using Original Mix mode)**

```bash
narrator-ai-cli file upload movie.mp4
# Output: file_id: abc123
```

**Step 1: Select movie and template**

For Hot Drama mode (built-in movie):

```bash
narrator-ai-cli resource movie list --search "Inception"
# Note the movie_id
narrator-ai-cli resource template list --category suspense
# Note the template_id
```

**Step 2: Generate narration script**

Using Hot Drama mode:

```bash
narrator-ai-cli task create-script \
  --mode hot_drama \
  --movie-id MOVIE_ID \
  --template-id TEMPLATE_ID \
  --bgm-id BGM_ID \
  --dubbing-id DUBBING_ID
```

Using Original Mix mode (custom movie):

```bash
narrator-ai-cli task create-script \
  --mode original_mix \
  --file-id abc123 \
  --template-id TEMPLATE_ID \
  --bgm-id BGM_ID \
  --dubbing-id DUBBING_ID
```

Returns `task_id` and `task_order_num`.

**Step 3: Poll task status**

```bash
narrator-ai-cli task status TASK_ORDER_NUM
```

Poll every 10-15 seconds until status is `completed`. The output will contain `script_file_id`.

**Step 4: Compose video**

```bash
narrator-ai-cli task compose-video \
  --script-file-id SCRIPT_FILE_ID \
  --visual-template-id VISUAL_TEMPLATE_ID
```

Returns `task_id` and `task_order_num` for the composition task.

**Step 5: Poll composition status and download**

```bash
narrator-ai-cli task status COMPOSITION_TASK_ORDER_NUM
```

When completed, the output contains `video_url`. Download the video:

```bash
curl -o final_video.mp4 "VIDEO_URL"
```

### 4. Standard Path: Adapted Narration (Adapted Narration)

This workflow uses pre-existing movie clips (requires uploading clip metadata).

**Step 0: Upload movie and clip data**

```bash
narrator-ai-cli file upload movie.mp4
# file_id: movie123

narrator-ai-cli file upload clips.json
# file_id: clips456
```

**Step 1: Generate narration script**

```bash
narrator-ai-cli task create-script \
  --mode new_drama \
  --file-id movie123 \
  --clip-data-file-id clips456 \
  --template-id TEMPLATE_ID \
  --bgm-id BGM_ID \
  --dubbing-id DUBBING_ID
```

**Steps 2-4:** Same as Fast Path (poll script status, compose video, poll composition status)

### 5. Standalone Tasks

**Voice cloning:**

```bash
narrator-ai-cli file upload voice_sample.mp3
# file_id: voice789

narrator-ai-cli task clone-voice --file-id voice789 --name "My Custom Voice"
```

**Text-to-speech:**

```bash
narrator-ai-cli task tts \
  --text "Hello, this is a test narration." \
  --dubbing-id DUBBING_ID \
  --speed 1.0 \
  --volume 1.0
```

### 6. Task Management

**List all tasks:**

```bash
narrator-ai-cli task list
```

**Get task details:**

```bash
narrator-ai-cli task status TASK_ORDER_NUM
```

## Common Patterns

### Pattern 1: End-to-End Video Creation (Fast Path)

```bash
# 1. Find a movie
narrator-ai-cli resource movie list --search "Matrix"
# movie_id: 12345

# 2. Find a template
narrator-ai-cli resource template list --category action
# template_id: 67890

# 3. Find BGM and dubbing
narrator-ai-cli resource bgm list
# bgm_id: 111
narrator-ai-cli resource dubbing list
# dubbing_id: 222

# 4. Create script
narrator-ai-cli task create-script \
  --mode hot_drama \
  --movie-id 12345 \
  --template-id 67890 \
  --bgm-id 111 \
  --dubbing-id 222

# Output: task_order_num: 999

# 5. Poll until complete
narrator-ai-cli task status 999
# Wait 10-15 seconds, repeat until status: completed
# Output contains: script_file_id: script_abc

# 6. Compose video
narrator-ai-cli task compose-video \
  --script-file-id script_abc \
  --visual-template-id 1

# Output: task_order_num: 1000

# 7. Poll composition
narrator-ai-cli task status 1000
# Wait until status: completed
# Output contains: video_url: https://...

# 8. Download
curl -o my_narration.mp4 "https://..."
```

### Pattern 2: Using Custom Movie (Original Mix)

```bash
# 1. Upload movie
narrator-ai-cli file upload my_movie.mp4
# file_id: custom123

# 2. Select template, BGM, dubbing (same as Pattern 1)

# 3. Create script with custom movie
narrator-ai-cli task create-script \
  --mode original_mix \
  --file-id custom123 \
  --template-id 67890 \
  --bgm-id 111 \
  --dubbing-id 222

# 4-7. Same as Pattern 1
```

### Pattern 3: Batch Video Creation

```python
import subprocess
import json
import time

def create_narration_video(movie_id, template_id, bgm_id, dubbing_id):
    # Create script
    result = subprocess.run([
        "narrator-ai-cli", "task", "create-script",
        "--mode", "hot_drama",
        "--movie-id", str(movie_id),
        "--template-id", str(template_id),
        "--bgm-id", str(bgm_id),
        "--dubbing-id", str(dubbing_id)
    ], capture_output=True, text=True)
    
    # Parse task_order_num from output
    task_order_num = parse_task_order_num(result.stdout)
    
    # Poll until complete
    while True:
        status_result = subprocess.run([
            "narrator-ai-cli", "task", "status", str(task_order_num)
        ], capture_output=True, text=True)
        
        status_data = parse_status_output(status_result.stdout)
        if status_data["status"] == "completed":
            script_file_id = status_data["script_file_id"]
            break
        elif status_data["status"] == "failed":
            raise Exception(f"Script generation failed: {status_data}")
        
        time.sleep(15)
    
    # Compose video
    compose_result = subprocess.run([
        "narrator-ai-cli", "task", "compose-video",
        "--script-file-id", script_file_id,
        "--visual-template-id", "1"
    ], capture_output=True, text=True)
    
    compose_task_num = parse_task_order_num(compose_result.stdout)
    
    # Poll composition
    while True:
        status_result = subprocess.run([
            "narrator-ai-cli", "task", "status", str(compose_task_num)
        ], capture_output=True, text=True)
        
        status_data = parse_status_output(status_result.stdout)
        if status_data["status"] == "completed":
            return status_data["video_url"]
        elif status_data["status"] == "failed":
            raise Exception(f"Video composition failed: {status_data}")
        
        time.sleep(15)

# Create 5 videos with different movies
movies = [101, 102, 103, 104, 105]
for movie_id in movies:
    video_url = create_narration_video(
        movie_id=movie_id,
        template_id=10,
        bgm_id=20,
        dubbing_id=30
    )
    print(f"Video for movie {movie_id}: {video_url}")
```

## Configuration Options

The CLI stores configuration in `~/.narrator-ai-cli/config.yaml`. Key settings:

```yaml
app_key: "your_api_key"
api_endpoint: "https://api.narrator-ai.com"  # Default, can be overridden
log_level: "INFO"
```

**Set configuration values:**

```bash
narrator-ai-cli config set KEY VALUE
narrator-ai-cli config set log_level DEBUG
```

**Get configuration value:**

```bash
narrator-ai-cli config get app_key
```

## Error Handling

The CLI returns standard error codes. Common errors:

| Code | Meaning | Action |
|------|---------|--------|
| 401 | Invalid API key | Check `NARRATOR_APP_KEY` or config |
| 404 | Resource not found | Verify file_id, movie_id, or template_id |
| 429 | Rate limit exceeded | Wait and retry |
| 500 | Server error | Check API status, retry later |

**Example error output:**

```
Error: API request failed with status 401
Details: {"error": "Invalid API key"}
```

## Troubleshooting

### Issue: "narrator-ai-cli: command not found"

**Solution:** Ensure the CLI is installed and in your PATH:

```bash
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"
which narrator-ai-cli
```

### Issue: "API key not configured"

**Solution:** Set the API key:

```bash
narrator-ai-cli config set app_key YOUR_KEY
# or
export NARRATOR_APP_KEY="YOUR_KEY"
```

### Issue: Task stuck in "processing" status

**Solution:** Tasks can take 5-15 minutes depending on complexity. Poll status every 10-15 seconds:

```bash
while true; do
  narrator-ai-cli task status TASK_ORDER_NUM
  sleep 15
done
```

If stuck for >30 minutes, check API status or contact support.

### Issue: Video composition fails

**Solution:** Ensure `script_file_id` is from a completed script task. Check task output:

```bash
narrator-ai-cli task status SCRIPT_TASK_ORDER_NUM
# Verify status: completed and script_file_id is present
```

### Issue: Downloaded video is corrupted

**Solution:** Use proper download method:

```bash
curl -L -o video.mp4 "VIDEO_URL"
# -L follows redirects
```

## Important Notes for AI Agents

1. **Always poll task status** — Script generation and video composition are async. Poll every 10-15 seconds until `status: completed`.

2. **Resource IDs are required** — Before creating tasks, retrieve IDs for movie, template, BGM, and dubbing from resource commands.

3. **Confirm before creating tasks** — Each task may incur costs. Ask the user to confirm movie/template selection before running `task create-script`.

4. **Use environment variables for secrets** — Never hardcode API keys. Use `$NARRATOR_APP_KEY` or config file.

5. **Handle file uploads carefully** — When using Original Mix or New Drama modes, upload files first and note the returned `file_id`.

6. **Visual template is optional** — If not specified, defaults to template ID 1. User can override with `--visual-template-id`.

7. **Language chain** — If user speaks Chinese, default to Chinese templates, BGM, and dubbing. If English, default to English resources. Always ask for confirmation.

8. **Check task status errors** — If a task fails, examine the error message in the status output and suggest corrective action.

## Workflow Decision Tree

When a user asks to create a narration video:

1. **Ask for movie source:**
   - Built-in movie (Hot Drama) → Get movie_id
   - Custom movie file (Original Mix) → Upload file, get file_id
   - Multiple clips (New Drama) → Upload files + clip data

2. **Ask for template preference:**
   - List templates, ask user to select by ID or category

3. **Ask for BGM and dubbing:**
   - List options, use defaults if user has no preference

4. **Confirm details:**
   - Show selected movie, template, BGM, dubbing
   - Ask "Proceed with video creation?"

5. **Execute workflow:**
   - Fast Path (Original Narration) → Steps 0-4
   - Standard Path (Adapted Narration) → Steps 0-5

6. **Monitor and report:**
   - Poll task status, show progress
   - Provide video URL when complete

## API Endpoint Reference

The CLI communicates with the NarratorAI API at `https://api.narrator-ai.com`. All requests require the `app_key` header.

**Example manual API call (for debugging):**

```bash
curl -X GET "https://api.narrator-ai.com/v1/resources/movies" \
  -H "app_key: $NARRATOR_APP_KEY"
```

## Data Privacy

- Uploaded files are stored on NarratorAI servers
- Files are associated with your API key
- Generated videos are accessible via temporary URLs
- No data is stored locally by the CLI except configuration

## Additional Resources

- **CLI Repository:** https://github.com/NarratorAI-Studio/narrator-ai-cli
- **Resource Preview (Feishu Docs):** https://ceex7z9m67.feishu.cn/wiki/WLPnwBysairenFkZDbicZOfKnbc
- **Contact:** merlinyang@gridltd.com
