```markdown
---
name: narrator-ai-cli-video-generation
description: Create AI-narrated movie commentary videos using narrator-ai-cli through natural language commands
triggers:
  - "create a movie narration video"
  - "generate film commentary for"
  - "make a narrated video about"
  - "use narrator-ai to create"
  - "show available movies for narration"
  - "generate multiple narration videos"
  - "create original narration content"
  - "adapt a movie into narrated clips"
---

# Narrator AI CLI Video Generation Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

This skill enables AI agents to use `narrator-ai-cli`, a Python CLI tool that automates the creation of AI-narrated movie commentary videos. The tool provides two workflows: **Adapted Narration** (using existing movie clips) and **Original Narration** (generating new content), with built-in resources including ~100 movies, 146 BGM tracks, 63 voices, and 90+ templates.

## Installation

Install via pip from the GitHub repository:

```bash
pip install "narrator-ai-cli @ git+https://github.com/NarratorAI-Studio/narrator-ai-cli.git"
```

Or clone and install locally:

```bash
git clone https://github.com/NarratorAI-Studio/narrator-ai-cli.git
cd narrator-ai-cli
pip install -e .
```

**Requirements:**
- Python 3.10+
- Dependencies: typer, httpx[socks], httpx-sse, pyyaml, rich

## Configuration

Set your API key (required for all operations):

```bash
# Set app key
narrator-ai-cli config set app_key YOUR_APP_KEY

# View current configuration
narrator-ai-cli config show

# Set proxy (optional)
narrator-ai-cli config set proxy http://127.0.0.1:7890
```

Configuration is stored in `~/.narrator-ai-cli/config.yaml`.

**Environment Variable Alternative:**

```bash
export NARRATOR_APP_KEY="your_app_key_here"
```

## Core Concepts

- **file_id**: Unique identifier for uploaded scripts or generated audio files
- **task_id**: Identifier for video generation tasks
- **task_order_num**: Sequential number for tracking task progress
- **Adapted Narration**: Uses existing movie clips (Fast Path)
- **Original Narration**: Generates new video from scratch (Standard Path)

## Key Commands

### Resource Discovery

```bash
# List all available movies
narrator-ai-cli resources movies

# List BGM tracks
narrator-ai-cli resources bgm

# List dubbing voices
narrator-ai-cli resources dubbing

# List narration templates
narrator-ai-cli resources templates

# Search for specific movie
narrator-ai-cli resources movies --search "Shawshank"
```

### Fast Path (Adapted Narration)

Creates narrated videos from existing movie clips:

```bash
# Step 1: Select movie (from resources movies output)
MOVIE_ID="movie_123"

# Step 2: Generate script
narrator-ai-cli script generate \
  --movie-id $MOVIE_ID \
  --template-id template_456 \
  --output script.txt

# Step 3: Upload script
FILE_ID=$(narrator-ai-cli file upload script.txt --type script | grep "file_id:" | awk '{print $2}')

# Step 4: Create video task
TASK_ID=$(narrator-ai-cli task create \
  --file-id $FILE_ID \
  --movie-id $MOVIE_ID \
  --bgm-id bgm_789 \
  --dubbing-id voice_012 \
  --mode hot_drama | grep "task_id:" | awk '{print $2}')

# Step 5: Poll for completion
narrator-ai-cli task status $TASK_ID --wait
```

### Standard Path (Original Narration)

Generates completely new content:

```bash
# Step 1: Create script from topic
narrator-ai-cli script generate \
  --topic "A detective story set in Victorian London" \
  --template-id template_creative \
  --output original_script.txt

# Step 2: Upload script
FILE_ID=$(narrator-ai-cli file upload original_script.txt --type script | grep "file_id:" | awk '{print $2}')

# Step 3: Generate clip data
CLIP_FILE_ID=$(narrator-ai-cli clip generate \
  --script-file-id $FILE_ID \
  --output clips.json | grep "file_id:" | awk '{print $2}')

# Step 4: Generate audio
AUDIO_FILE_ID=$(narrator-ai-cli audio generate \
  --script-file-id $FILE_ID \
  --dubbing-id voice_012 | grep "file_id:" | awk '{print $2}')

# Step 5: Create video task
TASK_ID=$(narrator-ai-cli task create \
  --file-id $FILE_ID \
  --clip-file-id $CLIP_FILE_ID \
  --audio-file-id $AUDIO_FILE_ID \
  --bgm-id bgm_789 \
  --mode original_mix | grep "task_id:" | awk '{print $2}')

# Step 6: Poll for completion
narrator-ai-cli task status $TASK_ID --wait
```

### Standalone Operations

```bash
# Text-to-speech only
narrator-ai-cli tts \
  --text "Your narration text here" \
  --dubbing-id voice_012 \
  --output narration.mp3

# Voice cloning
narrator-ai-cli voice clone \
  --audio-file sample.wav \
  --voice-name "MyCustomVoice" \
  --description "Cloned from sample audio"

# Check task status
narrator-ai-cli task status TASK_ID_HERE

# List all tasks
narrator-ai-cli task list

# Download completed video
narrator-ai-cli task download TASK_ID_HERE --output video.mp4
```

## Python API Usage

```python
from narrator_ai_cli.client import NarratorClient
from narrator_ai_cli.config import load_config

# Initialize client
config = load_config()
client = NarratorClient(
    app_key=config.get("app_key"),
    proxy=config.get("proxy")
)

# List movies
movies = client.list_movies()
for movie in movies["data"]:
    print(f"{movie['id']}: {movie['name']}")

# Generate script
script_response = client.generate_script(
    movie_id="movie_123",
    template_id="template_456"
)
script_content = script_response["data"]["script"]

# Upload script
with open("script.txt", "w") as f:
    f.write(script_content)

upload_response = client.upload_file(
    file_path="script.txt",
    file_type="script"
)
file_id = upload_response["data"]["file_id"]

# Create video task
task_response = client.create_task(
    file_id=file_id,
    movie_id="movie_123",
    bgm_id="bgm_789",
    dubbing_id="voice_012",
    mode="hot_drama"
)
task_id = task_response["data"]["task_id"]

# Poll for completion
import time
while True:
    status = client.get_task_status(task_id)
    if status["data"]["status"] in ["completed", "failed"]:
        break
    time.sleep(5)

# Download video
if status["data"]["status"] == "completed":
    video_url = status["data"]["video_url"]
    client.download_file(video_url, "output.mp4")
```

## Common Patterns

### Batch Video Generation

```python
from narrator_ai_cli.client import NarratorClient
import os

client = NarratorClient(app_key=os.getenv("NARRATOR_APP_KEY"))

# Get movie list
movies = client.list_movies()["data"][:5]  # First 5 movies

# Generate videos for each
task_ids = []
for movie in movies:
    # Generate script
    script = client.generate_script(
        movie_id=movie["id"],
        template_id="template_comedy"
    )
    
    # Save and upload
    script_file = f"script_{movie['id']}.txt"
    with open(script_file, "w") as f:
        f.write(script["data"]["script"])
    
    file_resp = client.upload_file(script_file, "script")
    
    # Create task
    task = client.create_task(
        file_id=file_resp["data"]["file_id"],
        movie_id=movie["id"],
        bgm_id="bgm_001",
        dubbing_id="voice_001",
        mode="hot_drama"
    )
    task_ids.append(task["data"]["task_id"])
    
print(f"Created {len(task_ids)} tasks: {task_ids}")
```

### Error Handling

```python
from narrator_ai_cli.client import NarratorClient
from narrator_ai_cli.exceptions import NarratorAPIError
import os

client = NarratorClient(app_key=os.getenv("NARRATOR_APP_KEY"))

try:
    response = client.create_task(
        file_id="invalid_file_id",
        movie_id="movie_123",
        bgm_id="bgm_001",
        dubbing_id="voice_001",
        mode="hot_drama"
    )
except NarratorAPIError as e:
    print(f"API Error: {e.code} - {e.message}")
    if e.code == 40001:
        print("Action: Check and refresh app_key")
    elif e.code == 40301:
        print("Action: Verify file_id exists and is uploaded")
    elif e.code == 50001:
        print("Action: Retry after a delay")
```

## Workflow Decision Logic

When a user requests video creation, follow this decision tree:

1. **Does the request mention a specific movie?**
   - Yes → Use **Fast Path** (Adapted Narration)
   - No → Use **Standard Path** (Original Narration)

2. **Fast Path steps:**
   - Search and select movie from `resources movies`
   - Select template from `resources templates`
   - Select BGM from `resources bgm`
   - Select voice from `resources dubbing`
   - Generate script with `script generate --movie-id`
   - Upload script with `file upload`
   - Create task with `task create --mode hot_drama`

3. **Standard Path steps:**
   - Generate script from topic with `script generate --topic`
   - Upload script
   - Generate clip data with `clip generate`
   - Generate audio with `audio generate`
   - Create task with `task create --mode original_mix`

## Mode Selection

- **hot_drama**: Trending movie clips (requires movie_id)
- **original_mix**: Original content from scratch
- **new_drama**: New movie releases (requires movie_id)

## Troubleshooting

### API Key Issues

```bash
# Verify key is set
narrator-ai-cli config show

# Test with simple command
narrator-ai-cli resources movies

# Set key if missing
narrator-ai-cli config set app_key YOUR_APP_KEY
```

### Task Stuck in Processing

```bash
# Check task status
narrator-ai-cli task status TASK_ID

# Common statuses: pending, processing, completed, failed
# Processing can take 5-15 minutes for video generation

# Use --wait flag for auto-polling
narrator-ai-cli task status TASK_ID --wait --interval 10
```

### File Upload Failures

```python
# Verify file exists and is readable
import os
if not os.path.exists("script.txt"):
    print("File not found")
elif os.path.getsize("script.txt") == 0:
    print("File is empty")

# Check file type is correct
# Valid types: script, clip_data, audio
client.upload_file("script.txt", file_type="script")
```

### Proxy Configuration

```bash
# Set proxy for regions with network restrictions
narrator-ai-cli config set proxy http://127.0.0.1:7890

# Or use environment variable
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

### Rate Limiting (Error 50003)

- Wait 60 seconds between requests
- Implement exponential backoff for retries
- Check task status less frequently (every 10-15 seconds)

## Cost Estimation

Before creating tasks, estimate costs:

```python
# Typical costs per task (approximate)
# - Script generation: ~100 tokens
# - Audio generation: ~500 tokens per minute
# - Video composing: ~1000 tokens
# - Total per video: ~2000-5000 tokens

# Always confirm with user before creating multiple tasks
num_videos = 5
estimated_tokens = num_videos * 3000
print(f"Creating {num_videos} videos will consume ~{estimated_tokens} tokens")
```

## Important Notes

1. **Always poll task status** — video generation is asynchronous
2. **file_id is required** — you cannot skip the upload step
3. **BGM/dubbing IDs must exist** — verify with `resources` commands first
4. **Mode determines required parameters** — hot_drama needs movie_id, original_mix needs clip_data
5. **Scripts are not auto-saved** — explicitly save to file before uploading
6. **Task IDs are sequential** — use task_order_num for tracking batches
7. **Confirm before batch operations** — ask user before creating multiple videos

## Agent Best Practices

When helping users create narration videos:

1. **Clarify intent** — Ask if they want to adapt an existing movie or create original content
2. **Show options** — Display available movies/templates/voices before selecting
3. **Confirm selections** — Repeat chosen resources before generating
4. **Explain wait times** — Video generation takes 5-15 minutes
5. **Handle errors gracefully** — Use error codes to provide specific guidance
6. **Save outputs** — Store file_ids and task_ids for reference
7. **Estimate costs** — Warn about token usage for batch operations

## Example Agent Conversation Flow

```
User: "Create a narration video for The Shawshank Redemption"

Agent: I'll help you create a narration video. Let me:
1. Search for "The Shawshank Redemption" in available movies
2. Show you template options
3. Generate the video

[Executes: narrator-ai-cli resources movies --search "Shawshank"]

Found movie_id: shawshank_001

Which template style would you like?
- template_dramatic (serious tone)
- template_comedy (humorous take)
- template_analytical (film critique)

User: "comedy"

Agent: Great! Using comedy template. Generating script and video...
[Executes Fast Path workflow]

Video task created (task_id: task_12345)
Estimated completion: 10-15 minutes
I'll monitor progress and notify you when ready.
```

## Reference Files

The skill relies on these reference files (located in `references/` directory):

- `resources.md` — Full lists of movies, BGM, voices, templates
- `workflows.md` — Detailed workflow diagrams and decision trees
- `magic-video.md` — Video composing API parameters
- `operations.md` — All CLI commands with examples

Always check reference files for the most up-to-date resource IDs and parameters.

```
