---
name: watch-cli-video-agent
description: Download and analyze social videos using frames + transcript for AI agent understanding at 50× lower cost than multimodal APIs
triggers:
  - analyze this video for me
  - watch this YouTube video
  - get frames and transcript from video
  - download and extract video content
  - transcribe this social media video
  - help me understand what's in this video
  - extract architecture from this tutorial video
  - clone the UI from this demo video
---

# watch-cli Video Agent Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

`watch-cli` gives AI agents the ability to "watch" social videos by decomposing them into frames (JPGs) + audio transcript, avoiding expensive multimodal video APIs. Works with YouTube, X/Twitter, LinkedIn, TikTok, Reddit, Vimeo, and Facebook. ~50× cheaper and ~5× faster than calling a multimodal LLM on full video.

**The insight**: Videos are just frames + audio. Extract both, read them separately with vision + text LLMs, and you have full video understanding without burning a video model on every frame.

## Installation

```bash
# Quick install (symlinks to ~/.local/bin)
curl -fsSL https://raw.githubusercontent.com/sonpiaz/watch-cli/main/install.sh | bash
```

**Dependencies** (installer checks, but install manually if needed):
- `yt-dlp` — video downloader
- `ffmpeg` — frame/audio extraction
- `jq`, `curl`, `python3` — utilities

macOS:
```bash
brew install yt-dlp ffmpeg jq
```

Debian/Ubuntu:
```bash
sudo apt install yt-dlp ffmpeg jq python3 curl
```

## Configuration

Set your Kyma API key (or bring-your-own Groq/Google keys):

```bash
export KYMA_API_KEY=kyma-xxxxxxxx
```

Get a free key at https://kymaapi.com (includes ~1 hour of free transcription credit).

**Alternative**: For bring-your-own-keys, create `.env`:
```bash
GROQ_API_KEY=gsk_...
GOOGLE_AI_KEY=AIza...
```

## Core Commands

### `watch` — Full video analysis (orchestrator)

```bash
watch <url> [frame-count] [--cookies <file>]
```

Downloads video, extracts frames, transcribes audio — returns everything in one structured block.

**Example**:
```bash
watch https://www.youtube.com/watch?v=dQw4w9WgXcQ
```

**Output structure**:
```text
VIDEO: /tmp/dl-video/abc123.mp4
DURATION: 218
FRAMES:
  /tmp/frames_abc123/frame_01.jpg
  /tmp/frames_abc123/frame_02.jpg
  /tmp/frames_abc123/frame_03.jpg
  ...
TRANSCRIPT:
  Today I want to talk about how decomposition unlocks 10× cost reduction...
```

**With custom frame count** (default is 8):
```bash
watch https://twitter.com/user/status/12345 16
```

**For login-walled content** (LinkedIn, private X, Facebook):
```bash
watch https://www.linkedin.com/posts/someone_activity-12345 --cookies ~/cookies.txt
```

(Auto-detects browser cookies if signed in; see troubleshooting for manual export)

### `dl-video` — Download only

```bash
dl-video <url> [out-dir] [--cookies <file>]
```

Just downloads the video, returns the local mp4 path.

```bash
VIDEO_PATH=$(dl-video https://www.youtube.com/watch?v=dQw4w9WgXcQ /tmp/videos)
echo "Downloaded to: $VIDEO_PATH"
```

### `extract-frames` — Frame extraction

```bash
extract-frames <video> [count] [out-dir]
```

Extracts N evenly-spaced JPG frames from a video.

```bash
extract-frames video.mp4 12 /tmp/my-frames
# Returns:
# /tmp/my-frames/frame_01.jpg
# /tmp/my-frames/frame_02.jpg
# ...
```

### `transcribe` — Speech-to-text

```bash
transcribe <audio-or-video> [language]
```

Auto-extracts audio from video if needed, then transcribes using Whisper Large v3 Turbo via Kyma.

```bash
transcribe video.mp4
# or
transcribe audio.mp3 en
```

**Output**: Plain text transcript.

### `audio-q` — Audio scene Q&A

```bash
audio-q <audio-or-video> "<question>"
```

Beyond transcription: asks about tone, music, sound effects, language, emotion using a multimodal audio model (Gemini 3 Flash audio).

```bash
audio-q video.mp4 "What is the speaker's emotional tone? Is there background music?"
```

### `models` — List available models

```bash
models           # Audio models only
models --all     # All Kyma models (text, image, video, audio)
```

Shows live model list from Kyma API — `transcribe` and `audio-understand` aliases auto-update when Kyma swaps underlying models.

## Frame Count Guidance

| Video Type | Recommended Frames |
|---|---|
| Short tweet/clip (<2 min) | 4–8 (default) |
| Tutorial/talk (5–20 min) | 8–16 |
| Lecture (20–60 min) | 16–24 |
| Conference talk (>1 hr) | 24–32 |
| Fast-cut UI demo | Double the recommendation |

## Patterns for AI Agents

### Pattern 1: Full video understanding

```bash
# User: "Analyze this video and tell me what it's about"
OUTPUT=$(watch https://www.youtube.com/watch?v=VIDEO_ID)

# Parse the output:
# - VIDEO: line → path to mp4
# - FRAMES: block → list of jpg paths (read each as image)
# - TRANSCRIPT: block → full text (read as text)
# Now you have frames + transcript to reason about
```

### Pattern 2: Code tutorial → working implementation

```bash
# User: "Watch this coding tutorial and implement the project"
watch https://www.youtube.com/watch?v=CODING_TUTORIAL 16

# Read frames to see:
# - File structure screenshots
# - Code snippets on screen
# - Terminal commands
# Read transcript for:
# - Verbal explanations
# - Step-by-step instructions
# Combine both to reconstruct the full implementation
```

### Pattern 3: Architecture extraction

```bash
# User: "Extract the system architecture from this talk"
watch https://www.youtube.com/watch?v=SYSTEM_DESIGN_TALK 24

# Look for frames with:
# - Diagrams (boxes, arrows)
# - Service names
# - Data flow illustrations
# Use transcript to identify component relationships
# Generate Mermaid/PlantUML diagram
```

### Pattern 4: UI/UX cloning

```bash
# User: "Clone the interface shown in this demo"
watch https://twitter.com/designer/status/12345 12

# Frames show UI states:
# - Layout structure
# - Color scheme
# - Interactive elements
# Transcript reveals:
# - Interaction patterns
# - Animation timing
# Generate React/HTML+CSS implementation
```

### Pattern 5: Audio-only analysis

```bash
# For podcasts, music, or unclear audio:
TRANSCRIPT=$(transcribe podcast.mp3)
SCENE=$(audio-q podcast.mp3 "Describe the audio scene: tone, music, number of speakers, emotion")

# Combine both for full audio understanding
```

## Prompt Templates (Copy-Paste for Agents)

Located in `prompts/` directory, paste above `watch` output:

- **`implement-from-video.md`** — Coding walkthrough → working project
- **`extract-architecture.md`** — System talk → architecture diagram
- **`clone-ux.md`** — UI demo → React component
- **`paper-to-code.md`** — Research talk → runnable notebook
- **`tutorial-walkthrough.md`** — Long tutorial → cheat sheet

**Usage**:
```bash
# 1. Watch the video
watch https://www.youtube.com/watch?v=VIDEO_ID > output.txt

# 2. Prepend the prompt template
cat prompts/implement-from-video.md output.txt > full-context.txt

# 3. Feed to agent (already done if you're the agent!)
```

## Cost Estimates

| Video Length | Transcribe Cost |
|---|---|
| 5 minutes | ~$0.003 |
| 1 hour | ~$0.04 |
| 2 hours | ~$0.08 |

Frame extraction is local (ffmpeg), free. Free Kyma credit covers ~25 hours of audio.

**Comparison**: Multimodal video API on 1-hour video ≈ $5. `watch-cli` ≈ $0.10. (~50× cheaper)

## Troubleshooting

### Login-walled videos (LinkedIn, private X, Facebook)

**Auto-detection** (usually works):
1. Sign into the platform in your browser (Chrome, Firefox, Safari, Edge, Brave)
2. Run `watch <url>` — it auto-finds cookies

**Manual cookie export** (for servers/CI):
1. Install browser extension: "Get cookies.txt LOCALLY" (Chrome/Firefox)
2. Visit the platform while signed in
3. Click extension → Export cookies.txt
4. Save to `~/cookies.txt`
5. Run: `watch <url> --cookies ~/cookies.txt`

Full guide: `docs/cookies.md` in the repo.

### "Region-locked video" error

`yt-dlp` can't download region-restricted content. Workarounds:
- Use VPN to target region
- Pass `--cookies` from browser with VPN active

### "Audio file too large" error

Transcribe provider has 25MB audio limit. For 2+ hour videos:

```bash
# Split video first
ffmpeg -i long-video.mp4 -ss 00:00:00 -to 01:00:00 -c copy part1.mp4
ffmpeg -i long-video.mp4 -ss 01:00:00 -c copy part2.mp4

# Transcribe separately
transcribe part1.mp4 > transcript1.txt
transcribe part2.mp4 > transcript2.txt
cat transcript1.txt transcript2.txt > full-transcript.txt
```

### Empty transcript (silent video)

For screencasts with no speech:
1. Increase frame count: `watch <url> 24`
2. Use `audio-q` to describe any sound design: `audio-q video.mp4 "Are there any UI sounds, clicks, or ambient audio?"`

### Fast-cut content missing key moments

Default 8 frames won't catch rapid edits. Solution:
```bash
watch <url> 32  # 4× more frames
```

### Models not updating

`transcribe` and `audio-q` use Kyma aliases that auto-update. To see current models:
```bash
models
```

If you want to pin a specific model version, edit the script and replace the alias with a model ID from `models --all`.

## Advanced: Using as Claude Code Skill

Copy the pre-built skill into Claude's skill directory:

```bash
mkdir -p ~/.claude/skills
cp -r skills/watch-cli ~/.claude/skills/
```

Now `/watch <url>` becomes a first-class command in Claude Code, with prompt library auto-attached.

## Environment Variables Reference

```bash
# Primary (Kyma unified API)
export KYMA_API_KEY=kyma-xxxxxxxx

# Alternative (bring-your-own-keys)
export GROQ_API_KEY=gsk_...        # Whisper transcription
export GOOGLE_AI_KEY=AIza...       # Gemini audio understanding
```

## Real Agent Usage Example

```bash
# User asks: "Watch this video and build the project shown"
$ watch https://www.youtube.com/watch?v=TUTORIAL_VIDEO 16

# Agent receives:
VIDEO: /tmp/dl-video/abc123.mp4
DURATION: 1847
FRAMES:
  /tmp/frames_abc123/frame_01.jpg  # Shows project folder structure
  /tmp/frames_abc123/frame_02.jpg  # Shows package.json
  /tmp/frames_abc123/frame_03.jpg  # Shows main App.tsx code
  /tmp/frames_abc123/frame_04.jpg  # Shows terminal: npm install
  ...
TRANSCRIPT:
  Okay so first we're going to set up a new React project. Create a folder
  called my-app, then run npm init. Now let's install these dependencies...

# Agent reads:
# - frame_01.jpg → sees folder structure → creates directories
# - frame_02.jpg → reads package.json → writes dependencies
# - frame_03.jpg → reads code on screen → implements App.tsx
# - transcript → fills in verbal instructions for parts not shown
# Result: working project matching the tutorial
```

## Key Takeaways for Agents

1. **Always use `watch`** for the full pipeline (unless you only need one piece)
2. **Read frames as images** — they contain visual info the transcript can't capture
3. **Read transcript as text** — it contains verbal explanations the frames don't show
4. **Combine both** — frames + transcript = full video understanding
5. **Adjust frame count** based on video type (fast-cut → more frames)
6. **For login-walled content** — it auto-detects browser cookies, but fall back to `--cookies` if needed
7. **Cost is negligible** — transcription is ~$0.04/hour, don't hesitate to watch multiple videos
8. **Use prompt templates** in `prompts/` for specific output types (architecture, code, UI, etc.)

---

**License**: MIT  
**Homepage**: https://kymaapi.com  
**GitHub**: https://github.com/sonpiaz/watch-cli
