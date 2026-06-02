---
name: breathe-cli-resonance-breathing
description: Terminal-based paced resonance breathing for vagal tone training and heart rate variability improvement
triggers:
  - "help me set up breathe for resonance breathing"
  - "how do I use breathe-cli for HRV training"
  - "configure breathing exercises in terminal"
  - "find my personal resonance frequency with breathe"
  - "set up custom breathing ratios and presets"
  - "integrate breathe-cli into my wellness routine"
  - "troubleshoot breathe breathing app"
  - "create breathing session with specific duration"
---

# Breathe CLI — Resonance Breathing Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Breathe CLI is a terminal application for paced resonance breathing at ~6 breaths per minute, designed for vagal tone training and improving heart rate variability (HRV). It's a single Python file with no dependencies, supporting macOS and Windows 11. The app provides audio cues, visual progress bars, and session logging for daily practice.

## What It Does

Breathe CLI paces slow breathing at resonance frequency (typically 4.5-6.5 bpm) to amplify respiratory sinus arrhythmia (RSA) and improve cardiac vagal tone. It's based on clinical research showing benefits for heart failure patients and autonomic balance. The app includes presets for different times of day and allows custom breath ratios.

## Installation

### Direct Download

```bash
# Download breathe.py directly
curl -O https://raw.githubusercontent.com/marekkowalczyk/breathe-cli/main/breathe.py
chmod +x breathe.py

# Run directly
./breathe.py

# Or symlink to PATH
ln -s "$(pwd)/breathe.py" /usr/local/bin/breathe
breathe
```

### Via pip

```bash
pip install breathe-cli
breathe
```

### Via Homebrew (macOS)

```bash
brew tap marekkowalczyk/breathe
brew install breathe
breathe
```

## Basic Usage

### Quick Start

```bash
# Auto-select preset based on time of day
breathe

# Use specific preset
breathe --preset calm        # 15 min, 4s-6s ratio
breathe --preset balanced    # 10 min, 5s-5s ratio
breathe --preset extended    # 20 min, 4s-6s ratio

# List available presets
breathe --list-presets
```

### Custom Sessions

```bash
# Custom duration (1-60 minutes)
breathe --duration 5

# Custom breath ratio (inhale-exhale in seconds)
breathe --ratio 4-6

# Combine duration and ratio
breathe --duration 12 --ratio 4-6

# Short flags
breathe -d 8 -r 5-5
```

### Session Control

During a session, use these keys:

- `space` — Pause/resume (resume restarts from beginning of INHALE)
- `s` — Mute/unmute audio cues
- `q` or `Ctrl+C` — Quit immediately

### Sound and Logging Options

```bash
# Disable audio cues
breathe --no-sound
breathe -n

# Suppress startup warnings
breathe --quiet
breathe -q

# Don't log this session
breathe --no-log

# View log file path
breathe --log
```

## Breath Ratio Constraints

The app enforces safety constraints based on clinical protocols:

- **Inhale**: 3-10 seconds
- **Exhale**: 3-10 seconds
- **Total cycle**: ≥8 seconds (max 7.5 bpm)
- **Exhale/inhale ratio**: ≤2.0
- **No breath holds**: Three-number ratios like `4-7-8` are rejected

Valid ratios for common frequencies:

```bash
breathe --ratio 6-7    # 13s cycle = 4.6 bpm
breathe --ratio 6-6    # 12s cycle = 5.0 bpm
breathe --ratio 5-6    # 11s cycle = 5.5 bpm
breathe --ratio 5-5    # 10s cycle = 6.0 bpm (default)
breathe --ratio 4-6    # 10s cycle = 6.0 bpm (exhale emphasis)
```

## Finding Personal Resonance Frequency

Individual resonance frequency varies (4.5-6.5 bpm). If you have HRV biofeedback hardware (chest strap + software like Kubios), you can test:

### Testing Protocol

```bash
# Baseline: breathe normally for 2 min while recording HRV

# Test 6.0 bpm
breathe --ratio 5-5 -d 3 --no-log
# Note RMSSD or HR oscillation amplitude, then rest 1-2 min

# Test 5.5 bpm
breathe --ratio 5-6 -d 3 --no-log
# Note metric, rest

# Test 5.0 bpm
breathe --ratio 6-6 -d 3 --no-log
# Note metric, rest

# Test 4.6 bpm
breathe --ratio 6-7 -d 3 --no-log
# Note metric
```

Use the frequency that produces the highest RMSSD or widest HR oscillations.

## Configuration & Presets

### Time-of-Day Auto-Select

When run without arguments, `breathe` selects preset by time:

| Time         | Preset      | Duration | Ratio  | BPM |
|--------------|-------------|----------|--------|-----|
| Before noon  | `balanced`  | 10 min   | 5s-5s  | 6   |
| 12:00-16:59  | `extended`  | 20 min   | 4s-6s  | 6   |
| 17:00+       | `calm`      | 15 min   | 4s-6s  | 6   |

### Creating Custom Routines

```bash
# Morning routine: balanced, 8 minutes
breathe -p balanced -d 8

# Midday: extended with custom ratio
breathe -d 20 -r 4-7

# Evening wind-down: calm, quiet
breathe -p calm -q

# Quick session: 3 minutes, no sound, no log
breathe -d 3 -n --no-log
```

## Session Logging

Sessions are automatically logged unless `--no-log` is used.

```bash
# View log file location
breathe --log

# Typical log entry format:
# 2026-06-02 14:30:15,calm,15,4-6,complete
# timestamp,preset_or_custom,duration_min,ratio,status
```

Log location varies by OS:
- macOS: `~/Library/Application Support/breathe/sessions.log`
- Linux: `~/.local/share/breathe/sessions.log`
- Windows: `%APPDATA%\breathe\sessions.log`

## Advanced Patterns

### Scripting Daily Practice

```bash
#!/bin/bash
# morning-breathe.sh

HOUR=$(date +%H)

if [ $HOUR -lt 9 ]; then
    # Early morning: gentle start
    breathe -p balanced -d 5 -q
elif [ $HOUR -lt 12 ]; then
    # Late morning: standard session
    breathe -p balanced -q
else
    # Afternoon: use default auto-select
    breathe -q
fi
```

### Integrating with Cron/Launchd

```bash
# crontab entry for daily 8am reminder
0 8 * * * /usr/local/bin/terminal-notifier -message "Time for breathing practice" -title "Breathe"
```

### Custom Ratios for Different Goals

```bash
# Balanced vagal training (equal phases)
breathe -r 5-5 -d 10

# Parasympathetic emphasis (longer exhale)
breathe -r 4-6 -d 15

# Maximum exhale emphasis (within safety limits)
breathe -r 4-8 -d 12

# Slower resonance frequency
breathe -r 6-7 -d 15  # 4.6 bpm
```

## Safety Information

```bash
# View full safety documentation
breathe --safety
```

Key safety constraints:

- **No breath retention**: Continuous breathing only, no holds between phases
- **No rapid breathing**: Minimum 8-second cycles (max 7.5 bpm)
- **Exhale limits**: Exhale max 2x inhale length to prevent CO2 depletion
- **Immediate exit**: `q` or `Ctrl+C` always works, terminal always restored

## Troubleshooting

### Audio Not Working (macOS)

```bash
# Check if afplay is available
which afplay

# Test audio manually
afplay /System/Library/Sounds/Glass.aiff

# Use breathe without sound
breathe --no-sound
```

### Audio Not Working (Windows)

```bash
# Ensure winsound module is available (built-in to Python on Windows)
python -c "import winsound; winsound.Beep(1000, 200)"

# Use without sound if issues persist
breathe --no-sound
```

### Terminal Display Issues

```bash
# If display is corrupted after crash, reset:
reset

# Or manually restore cursor and colors:
printf '\033[?25h\033[0m'
```

### Invalid Ratio Errors

```bash
# ERROR: Total cycle too short (minimum 8 seconds)
breathe -r 3-4  # Invalid: 7s total

# Fix: increase phase lengths
breathe -r 4-4  # Valid: 8s total

# ERROR: Exhale too long (max 2x inhale)
breathe -r 3-7  # Invalid: 7/3 = 2.33

# Fix: adjust ratio
breathe -r 3-6  # Valid: 6/3 = 2.0
```

### Python Version Issues

```bash
# Check Python version (requires 3.7+)
python3 --version

# If using older Python, upgrade or use system Python 3
which python3
```

## Integration Examples

### With Meditation Scripts

```python
#!/usr/bin/env python3
import subprocess
import time

def guided_session():
    print("Starting 5-minute centering...")
    time.sleep(5)
    
    print("\nBeginning breathwork...")
    subprocess.run(['breathe', '-d', '10', '-r', '4-6', '-q'])
    
    print("\nReturning to normal breathing...")
    time.sleep(60)

if __name__ == '__main__':
    guided_session()
```

### With Tmux/Screen Sessions

```bash
# Create dedicated tmux session for breathing
tmux new-session -d -s breathe 'breathe -p calm'

# Attach when ready to practice
tmux attach -t breathe

# Detach with Ctrl+B D (session continues)
```

### With Focus Timers

```bash
#!/bin/bash
# pomodoro-breathe.sh: 25min work + 5min breathe

while true; do
    echo "Work period: 25 minutes"
    sleep 1500
    
    echo "Break time: breathe for 5 minutes"
    breathe -d 5 -q
    
    read -p "Continue? (y/n) " -n 1 -r
    echo
    [[ ! $REPLY =~ ^[Yy]$ ]] && break
done
```

## Command Reference

### All Flags

| Flag               | Short | Description                              |
|--------------------|-------|------------------------------------------|
| `--preset NAME`    | `-p`  | Use named preset (balanced/calm/extended)|
| `--duration MIN`   | `-d`  | Session length in minutes (1-60)         |
| `--ratio IN-EX`    | `-r`  | Breath ratio, e.g. `5-5` or `4-6`       |
| `--no-sound`       | `-n`  | Disable audio cues                       |
| `--quiet`          | `-q`  | Suppress startup warnings                |
| `--no-log`         |       | Don't log this session                   |
| `--log`            |       | Print log file path and exit             |
| `--safety`         |       | Print safety information and exit        |
| `--list-presets`   |       | Print preset table and exit              |
| `--version`        |       | Print version and exit                   |

### Version & Help

```bash
# Show version
breathe --version

# Help is shown on invalid arguments
breathe --help  # (triggers error with helpful message)
```

## Best Practices

1. **Consistency over perfection**: Daily 10-minute sessions are more effective than occasional 30-minute sessions
2. **Same time daily**: Practice at the same time to build habit and improve measurement consistency
3. **Find your frequency**: Use the default 6 bpm unless you've tested your personal resonance frequency with HRV hardware
4. **Longer exhale for calm**: Use 4-6 or 4-7 ratios in evening for parasympathetic emphasis
5. **Track progress**: Keep logs enabled to monitor adherence
6. **Start small**: Begin with 5 minutes if new to slow breathing
7. **Sit upright**: Practice sitting with straight spine for optimal breathing mechanics

## Platform-Specific Notes

### macOS
- Uses `/usr/bin/afplay` for audio cues
- Logs to `~/Library/Application Support/breathe/`
- Works on macOS 10.13+

### Windows 11
- Uses `winsound` module for audio
- Logs to `%APPDATA%\breathe\`
- Requires Windows Terminal for best experience

### Linux
- Audio cues not currently supported
- Use `--no-sound` flag
- Logs to `~/.local/share/breathe/`
