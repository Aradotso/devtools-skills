---
name: clipify-video-clip-generator
description: Turn long videos into social-ready clips with auto-detection, face-tracking reframe, and opus-style captions
triggers:
  - create social media clips from video
  - extract clips from long video
  - add captions to video clips
  - reframe 16:9 video to 9:16
  - generate vertical video clips
  - find funny moments in video
  - add opus style subtitles
  - create TikTok clips from video
---

# Clipify Video Clip Generator

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Clipify is a Claude Code skill that automatically turns long-form videos into social-ready clips. It transcribes video, identifies clip-worthy moments, reframes 16:9 to 9:16 with face-tracking pans, and burns opus-style word-by-word captions.

**Key capabilities:**
- Auto-detect punchlines, reversals, and awkward pauses via Whisper transcription
- Face-tracking pan for 9:16 vertical clips (no ML models — uses motion energy)
- Opus/karaoke/minimal subtitle styles with word-level highlighting
- Hardware-accelerated rendering (VideoToolbox on macOS)
- ~20s render time for 20s clips on Apple Silicon

## Installation

```bash
# Clone to Claude Code skills directory
git clone https://github.com/louisedesadeleer/clipify.git ~/.claude/skills/clipify

# Install dependencies
brew install ffmpeg
pip install openai-whisper numpy
```

**Requirements:**
- macOS (or Linux/Windows with `-hwaccel videotoolbox` removed from SKILL.md)
- ffmpeg with libx264
- Python 3 with numpy
- Whisper (openai-whisper)

Restart Claude Code after installation. The `/clipify` slash command will be available.

## Usage Workflow

### 1. Invoke the skill

In Claude Code:
```
/clipify
```

Provide the path to your source video when prompted:
```
/path/to/long-interview.mp4
```

### 2. Review proposed clips

Clipify transcribes the video and proposes 3-5 candidates with:
- Timestamp range
- Title/description
- Reason (punchline, reversal, audio peak, awkward pause)

Example output:
```
Clip 1: "The worst product advice" (02:34 - 02:51)
  Reason: Reversal after awkward pause

Clip 2: "We burned $2M on this" (08:12 - 08:29)
  Reason: Audio peak + punchline

Clip 3: "My co-founder quit on Zoom" (15:03 - 15:24)
  Reason: Punchline
```

### 3. Select clip and format

Choose which clip to cut, then specify:
- **Aspect ratio:** 9:16 (vertical), 16:9 (horizontal), 1:1 (square)
- **Reframe style** (if 9:16 from 16:9 with two speakers): pan (follow speaker) or split-screen
- **Subtitle style:** opus (bold white + yellow highlight), karaoke (word-by-word), minimal, or paste reference image

### 4. Output

Final clips are saved to:
```
<source-video-dir>/clipify_out/clip_<timestamp>.mp4
```

## Scripts Reference

Clipify uses standalone Python scripts for each processing step. You can call these directly for custom workflows.

### analyze.py — Speaker timeline from motion energy

```python
# Generate motion energy files for two face regions
ffmpeg -i video.mp4 -vf "crop=300:200:100:50,format=gray,tblend=all_mode=difference" \
  -f rawvideo -pix_fmt gray - | python scripts/analyze.py motion_left.bin

ffmpeg -i video.mp4 -vf "crop=300:200:1000:50,format=gray,tblend=all_mode=difference" \
  -f rawvideo -pix_fmt gray - | python scripts/analyze.py motion_right.bin

# Analyze both to generate speaker timeline
python scripts/analyze.py motion_left.bin motion_right.bin --fps 30 > timeline.txt
```

**Output format (timeline.txt):**
```
0.00-2.34:left
2.34-5.67:right
5.67-8.12:left
```

### build_pan.py — Generate ffmpeg crop expression

```python
# From speaker timeline, build hard-cut pan expression
python scripts/build_pan.py timeline.txt --left-x 100 --right-x 1000 --width 608 > pan_expr.txt

# Use in ffmpeg crop filter
ffmpeg -i source.mp4 -vf "crop=608:1080:'$(cat pan_expr.txt)':0" output.mp4
```

**Arguments:**
- `--left-x`: X coordinate of left speaker's face center
- `--right-x`: X coordinate of right speaker's face center
- `--width`: Width of the 9:16 crop window (e.g., 608 for 1080p)

**Output:** ffmpeg expression string like:
```
if(between(t,0,2.34),100,if(between(t,2.34,5.67),1000,if(between(t,5.67,8.12),100,1000)))
```

### build_ass.py — Generate ASS subtitle file

```python
# From Whisper JSON output, create opus-style captions
python scripts/build_ass.py whisper_output.json --style opus > captions.ass

# Burn into video
ffmpeg -i video.mp4 -vf "ass=captions.ass" output.mp4
```

**Whisper JSON format (input):**
```json
{
  "segments": [
    {
      "start": 0.5,
      "end": 2.3,
      "text": "This is the worst advice",
      "words": [
        {"word": "This", "start": 0.5, "end": 0.7},
        {"word": "is", "start": 0.7, "end": 0.85},
        {"word": "the", "start": 0.85, "end": 1.0},
        {"word": "worst", "start": 1.0, "end": 1.4},
        {"word": "advice", "start": 1.4, "end": 2.3}
      ]
    }
  ]
}
```

**Styles:**
- `opus`: Bold white text, yellow active-word highlight, centered top
- `karaoke`: Word-by-word color change, bottom positioned
- `minimal`: Clean white text, no highlights

### audio_align.py — Find clip offset in source

```python
# Find where a 20s clip appears in a 2-hour source video
python scripts/audio_align.py source.mp4 clip.mp4

# Output: 00:15:34.2 (offset timestamp)
```

Uses audio cross-correlation. Useful for re-linking edited clips to source timestamps.

## Common Patterns

### Extract clip manually (without auto-detection)

```bash
# 1. Transcribe with Whisper
whisper source.mp4 --model base --output_format json --output_dir ./

# 2. Cut segment (03:15 to 03:42)
ffmpeg -i source.mp4 -ss 00:03:15 -to 00:03:42 -c copy raw_clip.mp4

# 3. Generate captions for this segment
python scripts/build_ass.py source.json --start 195 --end 222 --style opus > clip.ass

# 4. Reframe to 9:16 with center crop (no pan)
ffmpeg -i raw_clip.mp4 -vf "crop=608:1080:656:0,ass=clip.ass" final_clip.mp4
```

### Two-speaker pan with manual face coordinates

```bash
# 1. Identify face regions on a sample frame
ffplay -ss 00:01:00 source.mp4  # visual inspection

# Left face: x=200, y=100, width=300, height=200
# Right face: x=1100, y=100, width=300, height=200

# 2. Generate motion energy files
ffmpeg -i source.mp4 -ss 00:03:15 -to 00:03:42 \
  -vf "crop=300:200:200:100,format=gray,tblend=all_mode=difference" \
  -f rawvideo -pix_fmt gray motion_left.bin

ffmpeg -i source.mp4 -ss 00:03:15 -to 00:03:42 \
  -vf "crop=300:200:1100:100,format=gray,tblend=all_mode=difference" \
  -f rawvideo -pix_fmt gray motion_right.bin

# 3. Analyze speaker timeline
python scripts/analyze.py motion_left.bin motion_right.bin --fps 30 > timeline.txt

# 4. Build pan expression (faces centered at x=350 and x=1250)
python scripts/build_pan.py timeline.txt --left-x 350 --right-x 1250 --width 608 > pan.txt

# 5. Apply crop with pan
ffmpeg -i source.mp4 -ss 00:03:15 -to 00:03:42 \
  -vf "crop=608:1080:'$(cat pan.txt)':0" panned_clip.mp4
```

### Custom subtitle styling

Edit ASS file generated by `build_ass.py`:

```ass
[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Default,Impact,68,&H00FFFFFF,&H0000FFFF,&H00000000,&H00000000,-1,0,0,0,100,100,0,0,1,3,0,2,10,10,120,1

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
Dialogue: 0,0:00:00.50,0:00:02.30,Default,,0,0,0,,{\k70}This {\k15}is {\k15}the {\k40}worst {\k90}advice
```

**Customize:**
- `Fontname`: Impact, Arial, Montserrat
- `Fontsize`: 68 for 1080p vertical
- `PrimaryColour`: `&H00FFFFFF` (white in BGR hex)
- `SecondaryColour`: `&H0000FFFF` (yellow highlight)
- `Outline`: Border thickness (3 = thick black outline)
- `Alignment`: 2=bottom center, 8=top center
- `MarginV`: Vertical margin from edge

### Batch processing multiple clips

```python
import subprocess
import json

# Load Whisper transcript
with open("source.json") as f:
    data = json.load(f)

# Define clip ranges
clips = [
    {"start": 154, "end": 171, "title": "clip1"},
    {"start": 492, "end": 509, "title": "clip2"},
    {"start": 903, "end": 924, "title": "clip3"}
]

for clip in clips:
    # Cut raw clip
    subprocess.run([
        "ffmpeg", "-i", "source.mp4",
        "-ss", str(clip["start"]),
        "-to", str(clip["end"]),
        "-c", "copy",
        f"raw_{clip['title']}.mp4"
    ])
    
    # Generate captions
    subprocess.run([
        "python", "scripts/build_ass.py", "source.json",
        "--start", str(clip["start"]),
        "--end", str(clip["end"]),
        "--style", "opus"
    ], stdout=open(f"{clip['title']}.ass", "w"))
    
    # Reframe and burn captions
    subprocess.run([
        "ffmpeg", "-i", f"raw_{clip['title']}.mp4",
        "-vf", f"crop=608:1080:656:0,ass={clip['title']}.ass",
        f"{clip['title']}_final.mp4"
    ])
```

## Configuration

### Hardware acceleration

**macOS (default):**
```bash
ffmpeg -hwaccel videotoolbox -i input.mp4 ...
```

**Linux with NVIDIA:**
```bash
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -i input.mp4 ...
```

**Windows:**
```bash
ffmpeg -hwaccel dxva2 -i input.mp4 ...
```

**Disable (CPU only):**
Remove `-hwaccel` flags from SKILL.md ffmpeg commands.

### Whisper model size

Faster but less accurate:
```bash
whisper video.mp4 --model tiny  # ~1GB, 10x faster
```

More accurate but slower:
```bash
whisper video.mp4 --model medium  # ~1.5GB, 2x slower
whisper video.mp4 --model large   # ~3GB, 4x slower
```

Default in SKILL.md: `base` (good balance for dialogue).

### Output quality settings

High quality (larger file):
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 18 -c:a aac -b:a 192k output.mp4
```

Fast encode (lower quality):
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset veryfast -crf 23 -c:a aac -b:a 128k output.mp4
```

Social media optimized (SKILL.md default):
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset medium -crf 20 -c:a aac -b:a 160k output.mp4
```

## Troubleshooting

### "No motion detected in face regions"

Face crop coordinates are wrong. Verify on a sample frame:
```bash
# Extract frame at 1 minute mark
ffmpeg -ss 00:01:00 -i source.mp4 -frames:v 1 sample.png

# Overlay crop rectangles (adjust x,y,w,h)
ffmpeg -i source.mp4 -ss 00:01:00 -frames:v 1 \
  -vf "drawbox=x=200:y=100:w=300:h=200:color=red:t=5,drawbox=x=1100:y=100:w=300:h=200:color=blue:t=5" \
  sample_boxes.png
```

Red = left face, blue = right face. Adjust coordinates until boxes frame each person's mouth/chin.

### "Captions out of sync"

Whisper timestamps drift on long videos. Use smaller segments:
```bash
# Transcribe only the relevant 5-minute section
ffmpeg -i source.mp4 -ss 00:15:00 -to 00:20:00 -c copy segment.mp4
whisper segment.mp4 --model base --output_format json
```

Or enable Whisper's word-level timestamps:
```bash
whisper video.mp4 --model base --word_timestamps True
```

### "Pan cuts too frequently"

Increase motion threshold in `analyze.py`:

```python
# Default threshold
MOTION_THRESHOLD = 0.1

# Higher = less sensitive (fewer speaker changes)
MOTION_THRESHOLD = 0.3
```

Or add minimum duration between cuts in `build_pan.py`:

```python
# Ignore speaker changes shorter than 2 seconds
MIN_SEGMENT_DURATION = 2.0
```

### "ffmpeg not found"

Ensure ffmpeg is in PATH:
```bash
which ffmpeg
# Should print: /usr/local/bin/ffmpeg or similar

# If not installed:
brew install ffmpeg  # macOS
sudo apt install ffmpeg  # Ubuntu
```

### "Whisper import error"

```bash
# Uninstall conflicting whisper packages
pip uninstall whisper openai-whisper

# Reinstall correct package
pip install openai-whisper

# Verify
python -c "import whisper; print(whisper.__version__)"
```

### "VideoToolbox acceleration failed"

macOS-specific. Fallback to CPU:
```bash
# Remove -hwaccel videotoolbox from all ffmpeg commands
ffmpeg -i input.mp4 ...  # (no -hwaccel flag)
```

Or use software decode explicitly:
```bash
ffmpeg -hwaccel none -i input.mp4 ...
```

## Advanced: Integration with Custom Workflows

### Use Clipify detection with external editor

```python
# 1. Run Clipify's detection logic (without cutting)
# This would be in your custom script:

import subprocess
import json

# Transcribe
subprocess.run(["whisper", "source.mp4", "--model", "base", "--output_format", "json"])

# Load transcript
with open("source.json") as f:
    transcript = json.load(f)

# Simple punchline detector (look for laughter indicators)
candidates = []
for segment in transcript["segments"]:
    text = segment["text"].lower()
    if any(word in text for word in ["haha", "lol", "crazy", "worst", "insane"]):
        candidates.append({
            "start": segment["start"],
            "end": segment["end"],
            "text": segment["text"]
        })

print(json.dumps(candidates, indent=2))
# Output to DaVinci Resolve, Premiere, etc.
```

### Export timeline for manual editing

```python
# Generate EDL (Edit Decision List) from Clipify candidates
def to_edl(clips, fps=30):
    edl = ["TITLE: Clipify Export", "FCM: NON-DROP FRAME", ""]
    
    for i, clip in enumerate(clips, 1):
        start_tc = frames_to_tc(int(clip["start"] * fps), fps)
        end_tc = frames_to_tc(int(clip["end"] * fps), fps)
        
        edl.append(f"{i:03d}  AX       V     C        {start_tc} {end_tc} 00:00:00:00 {end_tc}")
    
    return "\n".join(edl)

def frames_to_tc(frames, fps):
    h = frames // (fps * 3600)
    m = (frames % (fps * 3600)) // (fps * 60)
    s = (frames % (fps * 60)) // fps
    f = frames % fps
    return f"{h:02d}:{m:02d}:{s:02d}:{f:02d}"

# Usage
clips = [{"start": 154.2, "end": 171.8}, {"start": 492.5, "end": 509.1}]
print(to_edl(clips))
```

### Custom caption animations

Modify ASS file for animated entrances:

```ass
Dialogue: 0,0:00:00.50,0:00:02.30,Default,,0,0,0,,{\fad(200,200)\move(640,1000,640,900)}This is animated text
```

- `\fad(200,200)`: 200ms fade in/out
- `\move(x1,y1,x2,y2)`: Slide from bottom to center
- `\t(0,500,\fscx120\fscy120)`: Scale animation over 500ms

## Performance Tips

1. **Use proxy files for preview:** Transcode to lower resolution before clipify analysis
   ```bash
   ffmpeg -i source.mp4 -vf scale=960:540 -c:v libx264 -crf 28 proxy.mp4
   ```

2. **Skip transcription on re-runs:** Cache Whisper JSON output
   ```bash
   if [ ! -f source.json ]; then
     whisper source.mp4 --model base --output_format json
   fi
   ```

3. **Parallel clip rendering:** Process multiple clips simultaneously
   ```bash
   for clip in clip1 clip2 clip3; do
     ffmpeg -i $clip.mp4 -vf "..." ${clip}_out.mp4 &
   done
   wait
   ```

4. **GPU acceleration:** Use NVIDIA NVENC for faster encoding
   ```bash
   ffmpeg -hwaccel cuda -i input.mp4 -c:v h264_nvenc -preset p4 -crf 20 output.mp4
   ```
