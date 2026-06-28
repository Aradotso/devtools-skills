---
name: ainovel-cli-vietnamese
description: CLI tool for AI-powered multi-agent Vietnamese novel writing with automatic orchestration, rolling planning, and real-time intervention
triggers:
  - "write a Vietnamese novel using AI"
  - "set up ainovel-cli for automatic story writing"
  - "configure multi-agent novel writing system"
  - "create a Vietnamese novel with AI agents"
  - "resume interrupted novel writing session"
  - "customize novel writing style and rules"
  - "export AI-generated novel to EPUB"
  - "troubleshoot ainovel-cli Vietnamese version"
---

# ainovel-cli Vietnamese Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`ainovel-cli` (Vietnamese fork) is an autonomous multi-agent CLI system for long-form Vietnamese novel generation. It orchestrates three specialized agents (Architect, Writer, Editor) through a single LLM loop to produce complete novels from a one-sentence prompt, with support for 500+ chapters, real-time intervention, and automatic checkpoint recovery.

**Key capabilities:**
- Fully autonomous writing pipeline (world-building → outline → drafting → editing)
- 3-tier context management (chapter → arc → volume) with automatic compression
- Rolling planning: arcs/volumes expand progressively, avoiding empty outlines
- Step-level checkpointing: recovers from crashes, network loss, or Ctrl+C
- Real-time intervention: inject feedback without stopping the process
- 7-dimension quality evaluation (consistency, character, pacing, narrative, foreshadowing, hooks, aesthetics)
- Anti-AI-writing mechanics: built-in criteria to detect and avoid robotic prose
- Multi-provider LLM support: OpenRouter, Anthropic, Gemini, OpenAI, Deepseek, Ollama (local)

## Installation

### Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/kentjuno/ainovel-cli.git
cd ainovel-cli

# Create directories
mkdir -p config workspace

# Build image
docker build -t ainovel-cli-vi .

# Run TUI
docker run --rm -it \
  -v "$PWD/config:/root/.ainovel" \
  -v "$PWD/workspace:/workspace" \
  -e TERM=xterm-256color \
  ainovel-cli-vi
```

**Windows PowerShell:**
```powershell
docker run --rm -it `
  -v "${PWD}\config:/root/.ainovel" `
  -v "${PWD}\workspace:/workspace" `
  -e TERM=xterm-256color `
  ainovel-cli-vi
```

### Build from Source

```bash
git clone https://github.com/kentjuno/ainovel-cli.git
cd ainovel-cli
go build -o ainovel-cli ./cmd/ainovel-cli/

# Run
./ainovel-cli  # Linux/macOS
ainovel-cli.exe  # Windows
```

**Requirements:**
- Go ≥ 1.21
- Docker ≥ 24 (for Docker method)
- 4GB free RAM

## Configuration

Create `config/config.json` (Docker) or `~/.ainovel/config.json` (source build).

### Ollama (Local, Free)

```bash
# Install Ollama: https://ollama.com
# Pull model (12B+ recommended)
ollama pull gemma4:12b
```

**config.json:**
```json
{
  "provider": "ollama",
  "model": "gemma4:12b",
  "providers": {
    "ollama": {
      "base_url": "http://host.docker.internal:11434/v1",
      "models": ["gemma4:12b", "qwen3.5:27b"]
    }
  }
}
```

> On Linux Docker, use `172.17.0.1` instead of `host.docker.internal`

### OpenRouter

```json
{
  "provider": "openrouter",
  "model": "google/gemini-2.5-flash",
  "providers": {
    "openrouter": {
      "api_key": "${OPENROUTER_API_KEY}",
      "base_url": "https://openrouter.ai/api/v1",
      "models": [
        "google/gemini-2.5-flash",
        "google/gemini-2.5-pro",
        "anthropic/claude-sonnet-4"
      ]
    }
  }
}
```

### Anthropic

```json
{
  "provider": "anthropic",
  "model": "claude-sonnet-4-6",
  "providers": {
    "anthropic": {
      "api_key": "${ANTHROPIC_API_KEY}",
      "models": ["claude-sonnet-4-6", "claude-opus-4"]
    }
  }
}
```

### Role-Specific Models

Assign different models to each agent role:

```json
{
  "provider": "openrouter",
  "model": "google/gemini-2.5-flash",
  "providers": {
    "openrouter": {
      "api_key": "${OPENROUTER_API_KEY}",
      "base_url": "https://openrouter.ai/api/v1"
    },
    "anthropic": {
      "api_key": "${ANTHROPIC_API_KEY}"
    }
  },
  "roles": {
    "coordinator": {
      "provider": "openrouter",
      "model": "google/gemini-2.5-flash"
    },
    "architect": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-6"
    },
    "writer": {
      "provider": "openrouter",
      "model": "google/gemini-2.5-flash"
    },
    "editor": {
      "provider": "openrouter",
      "model": "google/gemini-2.5-flash"
    }
  }
}
```

## Core Commands

### Starting a Novel

Launch TUI and enter your prompt in the input field:

**Short prompt (system expands automatically):**
```
Cung đấu gia tộc, nhân vật chính xuất thân thấp kém nhưng tài năng ẩn giấu
```

**Detailed prompt (more control):**
```
Tiểu thuyết fantasy cổ đại Việt Nam, nhân vật chính là cô gái nghèo bị bán vào phủ làm tỳ nữ.
Thể loại: cung đấu + ngôn tình. Khoảng 80 chương. Nhân vật nam chính: vương gia lạnh lùng nhưng
thực ra bảo vệ nàng từ bóng tối. Kết thúc có hậu. Viết theo phong cách truyện ngôn tình Việt.
```

### TUI Commands

Type `/` in the input field to see command list:

| Command | Description |
|---------|-------------|
| `/help` | Show command list and keyboard shortcuts |
| `/model [role]` | Switch model (opens selector). Example: `/model writer` |
| `/diag` | Diagnostic report: detect loops, missing chapters, stuck foreshadowing |
| `/export` | Export novel to TXT (default location) |
| `/export ~/novel.epub` | Export to EPUB format |
| `/import <path>` | Import external novel into workspace |
| `/pause` | Pause writing (finishes current step first) |
| `/resume` | Resume after pause |
| `/quit` | Exit (safe, saves checkpoint) |

### Headless Mode

Run without TUI (server/automation):

```bash
docker run --rm \
  -v "$PWD/config:/root/.ainovel" \
  -v "$PWD/workspace:/workspace" \
  ainovel-cli-vi \
  --headless \
  --prompt "Viết tiểu thuyết cung đấu, nhân vật chính là cô lao công xuất thân thấp kém"
```

**With custom output:**
```bash
docker run --rm \
  -v "$PWD/config:/root/.ainovel" \
  -v "$PWD/workspace:/workspace" \
  ainovel-cli-vi \
  --headless \
  --output /workspace/my-novel \
  --prompt "Xuyên không vào game MMORPG, nhân vật chính là NPC buôn bán"
```

## Real-Time Intervention

Inject feedback while writing is ongoing:

**Example intervention:**
```
Chương vừa rồi quá nhanh. Hãy bổ sung thêm mô tả tâm lý nhân vật khi họ đối mặt với quyết định khó khăn.
```

The system will:
1. Evaluate impact scope (affects current chapter only? or entire arc?)
2. Rewrite affected content
3. Continue from corrected state

**When to intervene:**
- Character acting out of character
- Plot hole detected
- Pacing too fast/slow
- Tone inconsistent with earlier chapters

## Customizing Style and Rules

### Custom Rules File

Create `workspace/<novel-name>/custom_rules.md`:

```markdown
# Quy tắc tuỳ chỉnh

## Phong cách văn
- Dùng ngôn ngữ cổ điển, tránh từ hiện đại
- Đối thoại ngắn gọn, sắc bén
- Mô tả cảnh quan tỉ mỉ nhưng không rườm rà

## Nhân vật
- Nữ chính: kiên cường, thông minh, nhưng dễ tổn thương
- Nam chính: lạnh lùng bề ngoài, nội tâm sâu sắc
- Tránh nhân vật phụ trở thành "công cụ phục vụ cốt truyện"

## Cốt truyện
- Mỗi chương phải có ít nhất 1 xung đột nhỏ
- Tránh giải quyết mâu thuẫn quá dễ dàng
- Phục bút phải tự nhiên, không ép buộc

## Cấm
- KHÔNG dùng "đột nhiên", "bỗng chốc"
- KHÔNG để nhân vật giải thích cảm xúc thẳng thừng ("tôi cảm thấy buồn")
- KHÔNG kết thúc chương bằng cliffhanger giả tạo
```

The system reads this file automatically and enforces rules during writing.

### Anti-AI Writing Checks

Built-in detection (automatically applied):

**Mechanical checks:**
- Forbidden phrases: "đột nhiên", "bỗng chốc", "tựa hồ như"
- Emotion telling vs. showing: flags direct statements like "cô ấy buồn"
- Adverb overuse: detects excessive "-ly" equivalents in Vietnamese

**Semantic checks:**
- Character consistency: compares current actions with character profile
- Plot coherence: validates against established world rules
- Foreshadowing tracking: ensures setup elements get payoff

**Override checks:**
```
# In custom_rules.md
## Ngoại lệ
- Cho phép "đột nhiên" trong cảnh hành động nhanh
- Cho phép giải thích cảm xúc khi nhân vật tự vấn
```

## Checkpoint Recovery

System auto-saves after every successful tool call. Recovery is automatic on restart.

**Manual recovery:**
```bash
# Resume from last checkpoint (default behavior)
./ainovel-cli

# Resume specific novel
./ainovel-cli --resume /workspace/my-novel
```

**Checkpoint structure:**
```
workspace/
└── my-novel/
    ├── checkpoint.json          # Last successful step
    ├── progress.json            # Overall progress tracker
    ├── outline/                 # Architect output
    │   ├── world.json
    │   ├── characters.json
    │   └── arc_001.json
    ├── drafts/                  # Writer output
    │   ├── chapter_001_v1.md
    │   └── chapter_001_v2.md    # After rewrite
    └── final/                   # Published chapters
        └── chapter_001.md
```

**Recovery scenarios:**
- Crash/Ctrl+C: Resumes from last checkpoint
- Network timeout: Retries failed API call, continues from last success
- Manual intervention: Checkpoint created after correction applied

## Output Structure

```
workspace/
└── <novel-name>/
    ├── config.json              # Novel-specific config
    ├── progress.json            # Current writing state
    ├── checkpoint.json          # Recovery point
    ├── custom_rules.md          # User-defined rules (optional)
    ├── outline/
    │   ├── world.json           # World-building
    │   ├── characters.json      # Character profiles
    │   ├── volume_001.json      # Volume outline
    │   ├── arc_001_001.json     # Arc outline (volume 1, arc 1)
    │   └── ...
    ├── drafts/
    │   ├── chapter_001_v1.md    # First draft
    │   ├── chapter_001_v2.md    # After editor feedback
    │   └── ...
    ├── final/
    │   ├── chapter_001.md       # Published version
    │   ├── chapter_002.md
    │   └── ...
    └── exports/
        ├── novel.txt            # Plain text export
        └── novel.epub           # EPUB export
```

## Exporting Novels

### Text Export

```bash
# Via TUI command
/export

# Default output: workspace/<novel-name>/exports/novel.txt
```

**Custom location:**
```bash
/export ~/Documents/my-novel.txt
```

### EPUB Export

```bash
/export ~/Documents/my-novel.epub
```

**EPUB metadata:**
The system reads from `workspace/<novel-name>/metadata.json`:

```json
{
  "title": "Cung Đấu Kiến Trúc",
  "author": "AI Multi-Agent System",
  "language": "vi",
  "description": "Một câu chuyện về...",
  "cover": "cover.jpg"
}
```

If missing, defaults are used (title from folder name, author "ainovel-cli").

## Common Patterns

### Pattern: Custom Character Development

```markdown
# In custom_rules.md

## Quy tắc phát triển nhân vật

### Giai đoạn 1 (chương 1-20): Khởi đầu
- Nữ chính: ngây thơ, dễ bị lợi dụng
- Mỗi 5 chương: 1 sự kiện tổn thương nhỏ (dần cứng rắn)

### Giai đoạn 2 (chương 21-50): Trưởng thành
- Nữ chính bắt đầu chủ động
- Quan hệ với nam chính: từ sợ → tò mò → tin tưởng

### Giai đoạn 3 (chương 51-80): Đỉnh cao
- Nữ chính nắm quyền lực
- Đối đầu với phản diện chính

## Công cụ phát triển
- Sử dụng flashback tiết kiệm (tối đa 1 lần/10 chương)
- Mỗi arc phải có 1 "điểm không thể quay lại"
```

### Pattern: Automated Quality Gates

```json
// In workspace/<novel-name>/config.json
{
  "quality_gates": {
    "min_words_per_chapter": 3000,
    "max_words_per_chapter": 6000,
    "editor_threshold": 7,
    "max_rewrites_per_chapter": 2,
    "auto_pause_on_low_score": true
  }
}
```

When editor score < 7, system pauses and waits for intervention.

### Pattern: Multi-POV Switching

```markdown
# In custom_rules.md

## POV
- Chương chẵn: POV nữ chính
- Chương lẻ: POV nam chính
- Chương kết arc: POV phản diện (1 lần duy nhất)

## Quy tắc POV
- Không chuyển POV giữa chương
- Dùng ngôi thứ 3 nhìn sát vai
- Mỗi POV có giọng văn riêng (nữ chính: mềm mại; nam chính: ngắn gọn)
```

### Pattern: Foreshadowing Tracker

System automatically tracks foreshadowing elements:

```json
// Auto-generated in workspace/<novel-name>/foreshadowing.json
{
  "items": [
    {
      "id": "fh_001",
      "setup_chapter": 3,
      "content": "Chiếc bình ngọc trong kho báu",
      "expected_payoff_arc": 2,
      "status": "pending"
    },
    {
      "id": "fh_002",
      "setup_chapter": 7,
      "content": "Lời tiên tri về người cứu thế",
      "payoff_chapter": 45,
      "status": "resolved"
    }
  ]
}
```

Use `/diag` to check unresolved foreshadowing.

## Troubleshooting

### Issue: Ollama Model Not Responding

**Symptoms:**
- Timeout errors
- Empty responses

**Solution:**
```bash
# Check Ollama is running
curl http://localhost:11434/api/tags

# Docker: verify network access
docker run --rm curlimages/curl:latest \
  curl http://host.docker.internal:11434/api/tags

# Linux: use host IP instead
docker run --rm curlimages/curl:latest \
  curl http://172.17.0.1:11434/api/tags
```

Update `config.json` with correct `base_url`.

### Issue: Novel Gets Stuck in Loop

**Symptoms:**
- Coordinator repeats same steps
- No progress for 10+ API calls

**Solution:**
```bash
# Run diagnostics
/diag

# Check output for:
# - "Loop detected: architect called 5 times for same arc"
# - "Writer stuck on chapter X for 3 iterations"
```

**Fix:**
1. Use `/pause`
2. Manually edit `workspace/<novel-name>/progress.json`:
   ```json
   {
     "current_step": "writing",
     "current_chapter": 15,
     "stuck_counter": 0  // Reset this
   }
   ```
3. `/resume`

Or inject intervention: "Bỏ qua chương hiện tại, tiếp tục chương kế tiếp"

### Issue: Character Acting Inconsistently

**Symptoms:**
- Character behavior contradicts earlier chapters

**Solution:**
1. Check `workspace/<novel-name>/outline/characters.json`
2. Verify character profile is detailed enough:
   ```json
   {
     "name": "Lý Thanh Vân",
     "personality": [
       "kiên cường",
       "thông minh",
       "dễ tổn thương khi đối mặt với gia đình"
     ],
     "values": [
       "trung thành với người đáng tin",
       "ghét sự lừa dối",
       "không bao giờ từ bỏ"
     ],
     "growth_arc": "từ ngây thơ → thực dụng → bi quan → cân bằng"
   }
   ```
3. If profile is shallow, intervene:
   ```
   Nhân vật Thanh Vân không được phản ứng như vậy. Cô ấy ghét lừa dối nhất,
   nên phải đối mặt trực tiếp thay vì lẩn tránh. Viết lại cảnh này.
   ```

### Issue: EPUB Export Fails

**Symptoms:**
- `/export novel.epub` shows error
- Missing metadata

**Solution:**
1. Check `workspace/<novel-name>/final/` has chapters
2. Create `metadata.json`:
   ```json
   {
     "title": "Tên Tiểu Thuyết",
     "author": "Tên Tác Giả",
     "language": "vi",
     "description": "Mô tả ngắn gọn"
   }
   ```
3. Retry export

### Issue: Docker Permission Denied (Linux)

**Symptoms:**
- Cannot write to `workspace/` or `config/`

**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Or run with correct permissions
docker run --rm -it \
  -v "$PWD/config:/root/.ainovel" \
  -v "$PWD/workspace:/workspace" \
  --user $(id -u):$(id -g) \
  ainovel-cli-vi
```

### Issue: Model Refuses to Follow Protocol

**Symptoms:**
- Coordinator makes invalid tool calls
- JSON parsing errors

**Solution:**
- Models < 14B struggle with complex protocols
- Switch to larger model:
  ```json
  {
    "roles": {
      "coordinator": {
        "provider": "anthropic",
        "model": "claude-sonnet-4-6"
      }
    }
  }
  ```
- Or use instruction-tuned models: Qwen, Gemma, Claude

### Issue: High API Costs

**Symptoms:**
- Unexpected bill from provider

**Solution:**
1. Monitor token usage in logs:
   ```bash
   grep "tokens_used" workspace/<novel-name>/logs/*.log
   ```
2. Switch expensive roles to cheaper models:
   ```json
   {
     "roles": {
       "writer": {
         "provider": "openrouter",
         "model": "google/gemini-2.5-flash"  // Cheaper than Claude
       }
     }
   }
   ```
3. Use Ollama for drafting, cloud models for editing:
   ```json
   {
     "roles": {
       "writer": { "provider": "ollama", "model": "qwen3.5:27b" },
       "editor": { "provider": "anthropic", "model": "claude-sonnet-4-6" }
     }
   }
   ```

## Advanced Usage

### Programmatic API (Go)

```go
package main

import (
    "github.com/kentjuno/ainovel-cli/pkg/core"
    "github.com/kentjuno/ainovel-cli/pkg/config"
)

func main() {
    cfg := config.Load("config.json")
    
    novel := core.NewNovel(cfg)
    novel.SetPrompt("Viết truyện xuyên không về lập trình viên")
    
    // Start autonomous writing
    novel.Start()
    
    // Inject intervention during writing
    novel.Intervene("Thêm chi tiết về công nghệ AI trong thế giới game")
    
    // Export when done
    novel.Export("novel.epub")
}
```

### Batch Processing

```bash
# Process multiple prompts
cat prompts.txt | while read prompt; do
  docker run --rm \
    -v "$PWD/config:/root/.ainovel" \
    -v "$PWD/workspace:/workspace" \
    ainovel-cli-vi \
    --headless \
    --prompt "$prompt" \
    --output "/workspace/batch_$(date +%s)"
done
```

### Custom Agent Extension

Extend the Writer agent with custom logic:

```go
// In pkg/agents/writer/custom.go
package writer

func (w *Writer) CustomPreProcess(chapter *Chapter) error {
    // Add custom validation before writing
    if chapter.Number%10 == 0 {
        // Every 10th chapter gets special treatment
        chapter.Style = "climax"
    }
    return nil
}
```

Build with custom code:
```bash
go build -tags custom -o ainovel-cli-custom ./cmd/ainovel-cli/
```

This skill covers the core usage patterns for ainovel-cli Vietnamese edition. The system is designed for autonomous operation, so the primary skill is knowing when and how to intervene, not constant manual control.
