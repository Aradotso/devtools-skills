---
name: flatkey-cli-media-generation
description: Generate images, videos, audio, and text through the Flatkey CLI with unified credit balance and agent-friendly JSON output.
triggers:
  - generate image with flatkey
  - create video using flatkey cli
  - use flatkey for audio generation
  - generate media with flatkey
  - flatkey text to speech
  - check flatkey credits and models
  - flatkey ai media generation
  - flatkey cli video from image
---

# Flatkey CLI Media Generation

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Flatkey CLI is a terminal-first media generation tool for images, videos, audio, and text. It provides:

- **Unified API key and credit balance** across all media types
- **Agent-friendly JSON mode** with clean stdout/stderr
- **Local file output** with `--output` / `-o`
- **Model and voice discovery** through `flatkey models` and `flatkey audio voices`
- **Dry-run support** for request validation without spending credits
- **Cross-platform npm package** (macOS, Linux, Windows)

Built for media teams, AI agents, and programmatic generation workflows that need cost-effective batch testing without long queues.

## Installation

### Global install (recommended)

```bash
npm install -g @flatkey-ai/cli
```

### Run without installing

```bash
npx @flatkey-ai/cli help --ai
```

### Check version

```bash
flatkey --version
flatkey -v
```

## Authentication

### Environment variable (recommended for agents)

```bash
export FLATKEY_API_KEY=<your-key>
```

### Save locally

```bash
flatkey onboard --api-key <your-key>
```

### Pass inline

```bash
flatkey image generate --prompt "test" --api-key <your-key>
```

Get your API key from [console.flatkey.ai/keys](https://console.flatkey.ai/keys).

## Image Generation

### Basic image generation

```bash
flatkey image generate \
  --prompt "magazine cover, reflective typography, studio lighting" \
  --model gpt-image-2 \
  -o cover.png
```

### Agent-friendly JSON mode

```bash
flatkey image generate \
  --prompt "minimalist product shot, white background" \
  --model nano-banana-pro-preview \
  --json \
  -o product.png
```

JSON output includes:
- `media_url`: temporary signed URL
- `local_path`: saved file path (if `-o` used)
- `cost`: credits spent
- `model`: model used

### Upload local image for reference

```bash
# Get temporary signed URL for local image
flatkey image upload --file ./reference.png --json
```

Returns:
```json
{
  "url": "https://temporary-media.flatkey.ai/...",
  "expires_in_seconds": 3600
}
```

## Video Generation

### Basic video generation

```bash
flatkey video generate \
  --prompt "8 second cinematic product reveal, glossy black background" \
  --model seedance2 \
  --ratio 16:9 \
  --resolution 720p \
  -o launch.mp4
```

### Video with reference image

```bash
# Upload image first, then use in video
flatkey video generate \
  --model Seedance2.0-pro \
  --prompt "a sleepy kitten, soft window light" \
  --image ./reference.png \
  -o kitten.mp4 \
  --json
```

### Video with first/last frame

```bash
flatkey video generate \
  --model veo-3 \
  --prompt "smooth camera transition" \
  --first-frame ./start.png \
  --last-frame ./end.png \
  --ratio 16:9 \
  --resolution 1080p \
  -o transition.mp4
```

### Supported video options

**Ratios**: `16:9`, `9:16`, `4:3`, `3:4`, `21:9`, `1:1`

**Resolutions**: `480p`, `720p`, `1080p`

**Models**: `seedance2`, `veo-3`, `veo-3-fast`, `Seedance2.0-pro`

## Audio Generation

### Text-to-speech

```bash
flatkey audio generate \
  --prompt "Hello, this is a test of the Flatkey speech API." \
  --voice-id EXAVITQu4vr4xnSDxMaL \
  --model eleven_multilingual_v2 \
  --stability 0.5 \
  --similarity-boost 0.75 \
  --style 0 \
  -o speech.mp3 \
  --json
```

### Multilingual TTS

```bash
flatkey audio generate \
  --prompt "你好，这是 Flatkey 网关的语音测试。" \
  --voice-id EXAVITQu4vr4xnSDxMaL \
  --model eleven_multilingual_v2 \
  -o mandarin.mp3
```

### List available voices

```bash
flatkey audio voices --json
```

Returns array of voices with:
- `voice_id`: ID to use in generation
- `name`: voice name
- `labels`: metadata (gender, age, accent, etc.)

### Sound effects

```bash
flatkey audio sfx \
  --prompt "glass shattering on the floor" \
  --duration 3 \
  -o shatter.mp3 \
  --json
```

### Music generation

```bash
flatkey audio music \
  --prompt "calm ambient piano, sad mood" \
  --music-length-ms 10000 \
  -o background.mp3 \
  --json
```

## Text Generation

### Basic text generation

```bash
flatkey text generate \
  --prompt "write 5 sharp headlines for a creator tool launch" \
  --model gpt-5.5 \
  -o headlines.txt \
  --json
```

### Supported text models

- `gpt-5.5`
- Claude models
- Gemini models
- GLM models
- Grok models

Check live models:
```bash
flatkey models --type text --json
```

## Model and Credit Management

### List all available models

```bash
flatkey models --json
```

### Filter by type

```bash
flatkey models --type image --json
flatkey models --type video --json
flatkey models --type audio --json
flatkey models --type text --json
```

### Check credit balance

```bash
flatkey credits --json
```

### Check service status

```bash
flatkey status --json
```

## Agent Protocol

Use `flatkey help --ai` to get agent-specific guidance:

```bash
flatkey help --ai
```

### Agent best practices

1. **Always use `--json` flag** for machine-readable output
2. **Prefer `FLATKEY_API_KEY` environment variable** over inline keys
3. **Use `--output` / `-o`** when file path matters
4. **Call `flatkey models --json`** before choosing a model
5. **Call `flatkey audio voices --json`** before choosing a voice
6. **Use `--dry-run`** to validate requests without spending credits
7. **Upload local images** with `flatkey image upload` before using in video

### Dry-run validation

```bash
flatkey video generate \
  --prompt "fashion campaign hero shot" \
  --model seedance2 \
  --ratio 16:9 \
  --dry-run \
  --json
```

Returns request payload without executing generation.

## Common Patterns

### Batch image generation

```bash
#!/bin/bash
PROMPTS=(
  "product shot, white background"
  "lifestyle photo, natural light"
  "close-up detail, macro lens"
)

for i in "${!PROMPTS[@]}"; do
  flatkey image generate \
    --prompt "${PROMPTS[$i]}" \
    --model gpt-image-2 \
    -o "output_$i.png" \
    --json
done
```

### Video from uploaded reference

```bash
#!/bin/bash

# Upload reference image
UPLOAD_RESULT=$(flatkey image upload --file ./ref.png --json)
IMAGE_URL=$(echo $UPLOAD_RESULT | jq -r '.url')

# Generate video using uploaded image
flatkey video generate \
  --model seedance2 \
  --prompt "cinematic zoom in" \
  --image "$IMAGE_URL" \
  -o output.mp4 \
  --json
```

### Multi-voice audio batch

```bash
#!/bin/bash

# Get available voices
VOICES=$(flatkey audio voices --json | jq -r '.[].voice_id' | head -3)

TEXT="This is a test of multiple voice synthesis."

for VOICE in $VOICES; do
  flatkey audio generate \
    --prompt "$TEXT" \
    --voice-id "$VOICE" \
    --model eleven_multilingual_v2 \
    -o "voice_${VOICE}.mp3" \
    --json
done
```

### Check credits before generation

```bash
#!/bin/bash

CREDITS=$(flatkey credits --json | jq -r '.balance')

if (( $(echo "$CREDITS < 100" | bc -l) )); then
  echo "Low credits: $CREDITS"
  exit 1
fi

flatkey video generate \
  --prompt "test video" \
  --model veo-3-fast \
  -o test.mp4 \
  --json
```

### Model discovery for specific type

```bash
#!/bin/bash

# Get all video models
VIDEO_MODELS=$(flatkey models --type video --json | jq -r '.[].id')

echo "Available video models:"
echo "$VIDEO_MODELS"

# Use first available model
FIRST_MODEL=$(echo "$VIDEO_MODELS" | head -1)

flatkey video generate \
  --prompt "test generation" \
  --model "$FIRST_MODEL" \
  -o test.mp4 \
  --json
```

## Error Handling

### JSON output structure

Success:
```json
{
  "media_url": "https://...",
  "local_path": "./output.png",
  "cost": 5.2,
  "model": "gpt-image-2"
}
```

Error:
```json
{
  "error": "Insufficient credits",
  "code": "INSUFFICIENT_CREDITS",
  "details": {...}
}
```

### Common errors

| Error | Cause | Solution |
|-------|-------|----------|
| `FLATKEY_API_KEY not found` | No API key set | Set `export FLATKEY_API_KEY=<key>` |
| `Insufficient credits` | Credit balance too low | Add credits at console.flatkey.ai |
| `Model not found` | Invalid model ID | Run `flatkey models --json` |
| `Invalid voice_id` | Voice doesn't exist | Run `flatkey audio voices --json` |
| `File upload failed` | Local file not found | Check file path |

### Validate before generation

```bash
# Dry run to check request structure
flatkey image generate \
  --prompt "test" \
  --model gpt-image-2 \
  --dry-run \
  --json

# Check credits
flatkey credits --json

# Check model availability
flatkey models --type image --json
```

## Advanced Configuration

### Custom router URL (development only)

```bash
flatkey image generate \
  --prompt "test" \
  --base-url http://127.0.0.1:3000 \
  --json
```

### Default routing endpoints

- Generation router: `https://router.flatkey.ai`
- Model registry: `https://console.flatkey.ai/v1/available_models`
- Voice registry: `https://router.flatkey.ai/v1/voices`

## Troubleshooting

### Command not found

```bash
# Check installation
npm list -g @flatkey-ai/cli

# Reinstall
npm install -g @flatkey-ai/cli

# Use npx
npx @flatkey-ai/cli --version
```

### Progress animation interfering with output

Always use `--json` flag for agent workflows:

```bash
flatkey image generate --prompt "test" --json
```

This disables terminal animation and ensures clean stdout.

### Local file not saving

Check `-o` / `--output` path permissions:

```bash
# Ensure directory exists
mkdir -p ./outputs
flatkey image generate --prompt "test" -o ./outputs/test.png
```

### Video generation with image fails

Upload local image first:

```bash
# Wrong (local path)
flatkey video generate --image ./local.png --prompt "test"

# Correct (upload first)
UPLOAD=$(flatkey image upload --file ./local.png --json)
URL=$(echo $UPLOAD | jq -r '.url')
flatkey video generate --image "$URL" --prompt "test" --json
```

### Model version mismatch

Always check live models before generation:

```bash
flatkey models --type video --json | jq -r '.[].id'
```

Model IDs can change. Use dynamic lookup in scripts.

## Resources

- Website: [flatkey.ai](https://flatkey.ai)
- npm package: [@flatkey-ai/cli](https://www.npmjs.com/package/@flatkey-ai/cli)
- API keys: [console.flatkey.ai/keys](https://console.flatkey.ai/keys)
- Issues: [github.com/flatkey-ai/flatkey-cli/issues](https://github.com/flatkey-ai/flatkey-cli/issues)
