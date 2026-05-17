---
name: cliamp-terminal-music-player
description: Expert guide for using cliamp, a retro terminal music player inspired by Winamp that supports local files, streaming services, and remote media servers
triggers:
  - how do I play music in the terminal with cliamp
  - set up cliamp with spotify and navidrome
  - configure cliamp for youtube music streaming
  - how to use cliamp music player keybindings
  - integrate cliamp with plex or jellyfin
  - create custom radio stations in cliamp
  - write lua plugins for cliamp
  - troubleshoot cliamp audio output issues
---

# cliamp Terminal Music Player

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

cliamp is a retro terminal music player inspired by Winamp. It plays local files (MP3, FLAC, WAV, OGG, AAC, ALAC, Opus, WMA), streams (HTTP, HLS, Icecast), podcasts, and integrates with Spotify, YouTube Music, SoundCloud, Bilibili, NetEase Cloud Music, Navidrome, Plex, and Jellyfin. Features include a spectrum visualizer, parametric EQ, playlist management, radio browser, and Lua plugin system.

## Installation

### Quick Install (macOS/Linux)

```bash
curl -fsSL https://raw.githubusercontent.com/bjarneo/cliamp/HEAD/install.sh | sh
```

### Homebrew (macOS)

```bash
brew install bjarneo/cliamp/cliamp
```

The Homebrew formula automatically installs all required codec libraries (FLAC, Vorbis, Ogg).

### Arch Linux (AUR)

```bash
yay -S cliamp
```

### From Source

**Prerequisites:**
- Go 1.25.5 or later
- Linux: ALSA development headers (`libasound2-dev` on Debian/Ubuntu, `alsa-lib-devel` on Fedora, `alsa-lib` on Arch)

```bash
git clone https://github.com/bjarneo/cliamp.git
cd cliamp
make && make install
```

Or without Make:
```bash
go build -o cliamp .
```

### Optional Runtime Dependencies

Install these for extended format and streaming support:

```bash
# macOS
brew install ffmpeg yt-dlp

# Debian/Ubuntu
sudo apt install ffmpeg yt-dlp

# Arch
sudo pacman -S ffmpeg yt-dlp
```

- **ffmpeg**: AAC, ALAC, Opus, WMA playback
- **yt-dlp**: YouTube, YouTube Music, SoundCloud, Bandcamp, Bilibili, NetEase Cloud Music

## Quick Start

### Basic Playback

```bash
# Play a directory
cliamp ~/Music

# Play specific files
cliamp *.mp3 *.flac

# Play a stream URL
cliamp https://example.com/stream.mp3

# Play YouTube video/playlist
cliamp "https://www.youtube.com/watch?v=dQw4w9WgXcQ"

# Play SoundCloud track
cliamp "https://soundcloud.com/artist/track"
```

### Interactive Setup Wizard

Configure remote providers (Navidrome, Plex, Jellyfin, Spotify, YouTube Music, NetEase Cloud Music):

```bash
cliamp setup
```

The wizard validates connections and writes configuration to `~/.config/cliamp/config.toml`.

## Key Commands

Press `Ctrl+K` in the player to see all keybindings.

### Navigation & Playback
- `Space` / `P`: Play/Pause
- `N`: Next track
- `Shift+N`: Previous track
- `S`: Stop
- `R`: Open radio browser
- `L`: Toggle lyrics view
- `F`: Toggle fullscreen mode
- `Q` / `Ctrl+C`: Quit

### Volume & EQ
- `+` / `-`: Volume up/down
- `M`: Mute/unmute
- `E`: Toggle equalizer
- Arrow keys in EQ: Adjust frequency bands

### Playlist Management
- `A`: Add to playlist
- `D`: Remove from playlist
- `C`: Clear playlist
- `Shift+S`: Shuffle playlist
- `Ctrl+S`: Save playlist

### Visualizer
- `V`: Cycle visualizer modes (spectrum, bars, dots, etc.)
- `B`: Toggle visualizer display

## Configuration

Configuration file: `~/.config/cliamp/config.toml`

### Basic Configuration

```toml
[player]
volume = 80
shuffle = false
repeat = "none"  # none, one, all
visualizer = "spectrum"

[ui]
theme = "winamp"  # winamp, nord, dracula, gruvbox, catppuccin
show_spectrum = true
show_lyrics = true

[audio]
sample_rate = 44100
buffer_size = 4096
```

### Spotify Integration

```toml
[spotify]
enabled = true
username = "${SPOTIFY_USERNAME}"
# Credentials stored separately, configured via 'cliamp setup'
```

Run `cliamp setup` and select Spotify to authenticate. The wizard handles OAuth flow and stores credentials securely.

### Navidrome Integration

```toml
[navidrome]
enabled = true
server_url = "https://music.example.com"
username = "${NAVIDROME_USERNAME}"
password = "${NAVIDROME_PASSWORD}"
```

### Plex Integration

```toml
[plex]
enabled = true
server_url = "https://plex.example.com:32400"
token = "${PLEX_TOKEN}"
```

Get your Plex token: https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/

### Jellyfin Integration

```toml
[jellyfin]
enabled = true
server_url = "https://jellyfin.example.com"
api_key = "${JELLYFIN_API_KEY}"
user_id = "${JELLYFIN_USER_ID}"
```

### YouTube Music Integration

```toml
[youtube_music]
enabled = true
# Requires yt-dlp and browser cookies
cookies_from_browser = "firefox"  # chrome, firefox, safari, etc.
```

Extract cookies with yt-dlp:
```bash
yt-dlp --cookies-from-browser firefox --print cookies https://music.youtube.com
```

### Custom Radio Stations

Create `~/.config/cliamp/radios.toml`:

```toml
[[stations]]
name = "Soma FM Groove Salad"
url = "https://somafm.com/groovesalad130.pls"
genre = "Ambient"

[[stations]]
name = "NTS Radio 1"
url = "https://stream-relay-geo.ntslive.net/stream"
genre = "Eclectic"

[[stations]]
name = "Jazz24"
url = "https://live.wostreaming.net/direct/ppm-jazz24aac-ibc1"
genre = "Jazz"
```

## Playlist Management

### Save/Load Playlists

```bash
# Save current playlist
cliamp --save-playlist my-playlist.m3u

# Load playlist
cliamp --playlist ~/Music/playlists/favorites.m3u

# Create M3U playlist manually
cat > my-playlist.m3u << EOF
#EXTM3U
#EXTINF:180,Artist - Song Title
/path/to/song.mp3
#EXTINF:240,Another Artist - Another Song
https://example.com/stream.mp3
EOF

cliamp --playlist my-playlist.m3u
```

### Supported Playlist Formats

- M3U / M3U8
- PLS
- Direct URLs (one per line)

## Remote Control (IPC)

cliamp supports IPC commands when running in daemon mode:

```bash
# Start in headless daemon mode
cliamp --headless ~/Music

# Control from another terminal
cliamp ipc play
cliamp ipc pause
cliamp ipc next
cliamp ipc previous
cliamp ipc volume 75
cliamp ipc seek 30  # seek to 30 seconds
cliamp ipc status   # get current track info
```

### IPC in Scripts

```bash
#!/bin/bash
# Skip to next track if current track is longer than 10 minutes
status=$(cliamp ipc status --json)
duration=$(echo "$status" | jq -r '.duration')

if [ "$duration" -gt 600 ]; then
    cliamp ipc next
fi
```

## Lua Plugins

cliamp supports Lua plugins for custom visualizers, audio effects, and UI extensions.

### Plugin Directory Structure

```
~/.config/cliamp/plugins/
  └── my-plugin/
      ├── plugin.lua
      └── config.toml (optional)
```

### Example Plugin: Custom Visualizer

`~/.config/cliamp/plugins/pulse-visualizer/plugin.lua`:

```lua
-- Plugin metadata
plugin = {
    name = "Pulse Visualizer",
    version = "1.0.0",
    author = "Your Name",
    description = "Pulsing circle visualizer"
}

-- Called on each audio frame
function on_audio_frame(samples, sample_rate)
    -- samples: table of audio samples [-1.0, 1.0]
    -- sample_rate: integer
    
    local sum = 0
    for i, sample in ipairs(samples) do
        sum = sum + math.abs(sample)
    end
    
    local avg = sum / #samples
    local radius = math.floor(avg * 50)
    
    -- Return render data
    return {
        type = "circle",
        x = 40,
        y = 12,
        radius = radius,
        color = {r = 255, g = 100, b = 200}
    }
end

-- Called when plugin loads
function on_load()
    print("Pulse Visualizer loaded")
end

-- Called when plugin unloads
function on_unload()
    print("Pulse Visualizer unloaded")
end
```

### Enable Plugin

Add to `~/.config/cliamp/config.toml`:

```toml
[plugins]
enabled = ["pulse-visualizer"]
```

### Plugin API Reference

Available Lua functions:

```lua
-- Audio data
on_audio_frame(samples, sample_rate)

-- Track events
on_track_change(track)  -- track: {title, artist, album, duration}
on_play()
on_pause()
on_stop()

-- Lifecycle
on_load()
on_unload()

-- Drawing functions (available in plugin context)
draw_text(x, y, text, color)
draw_rect(x, y, width, height, color)
draw_circle(x, y, radius, color)
draw_line(x1, y1, x2, y2, color)

-- Utility functions
log(message)
get_config(key)  -- Read plugin config.toml
http_get(url)    -- Simple HTTP request
```

### Community Plugins

- **Soap Bubbles Visualizer**: https://github.com/bjarneo/cliamp-plugin-soap-bubbles

Install community plugins:

```bash
cd ~/.config/cliamp/plugins/
git clone https://github.com/bjarneo/cliamp-plugin-soap-bubbles.git soap-bubbles
```

Enable in config:

```toml
[plugins]
enabled = ["soap-bubbles"]
```

## SSH Streaming

Stream audio from a remote server to your local machine:

```bash
# On remote server, start cliamp in headless mode
cliamp --headless --http-server 0.0.0.0:8080 ~/Music

# On local machine, forward port and play
ssh -L 8080:localhost:8080 user@remote-server
cliamp http://localhost:8080/stream
```

## Advanced Usage

### Custom Audio Quality

```bash
# High quality playback
cliamp --sample-rate 96000 --buffer-size 8192 ~/Music

# Low latency (smaller buffer)
cliamp --buffer-size 2048 ~/Music
```

### Batch Processing with Shell Scripts

```bash
#!/bin/bash
# Play all albums in directory sequentially

for album_dir in ~/Music/*/; do
    echo "Playing: $album_dir"
    cliamp "$album_dir"
done
```

### Integration with System Media Controls (Linux)

Enable MPRIS support to control cliamp with media keys:

```toml
[player]
mpris = true
```

Control with playerctl:
```bash
playerctl -p cliamp play-pause
playerctl -p cliamp next
playerctl -p cliamp previous
```

### Now-Playing Display (Quickshell)

Example Quickshell widget configuration for displaying currently playing track:

```qml
// ~/.config/quickshell/nowplaying.qml
import Quickshell
import Quickshell.Services.Mpris

MprisPlayer {
    player: "cliamp"
    
    Text {
        text: player.metadata.title + " - " + player.metadata.artist
        font.pointSize: 12
    }
}
```

## Troubleshooting

### No Audio Output (Silence)

On Linux with PipeWire or PulseAudio, install the ALSA bridge:

```bash
# PipeWire (Arch)
sudo pacman -S pipewire-alsa

# PulseAudio (Arch)
sudo pacman -S pulseaudio-alsa

# PipeWire (Debian/Ubuntu)
sudo apt install pipewire-alsa

# PulseAudio (Debian/Ubuntu)
sudo apt install pulseaudio-alsa
```

### "Library not loaded" on macOS

If you downloaded pre-built binaries directly (not via Homebrew):

```bash
brew install flac libvorbis libogg
```

Or install via Homebrew to avoid this:
```bash
brew install bjarneo/cliamp/cliamp
```

### YouTube/SoundCloud Not Playing

Ensure yt-dlp is installed and up-to-date:

```bash
# Update yt-dlp
pip install --upgrade yt-dlp
# or
brew upgrade yt-dlp

# Test yt-dlp
yt-dlp --version
```

### Spotify Authentication Failed

Re-run setup wizard:

```bash
cliamp setup
```

Select Spotify and follow the OAuth flow. Ensure your Spotify account is Premium (free accounts are not supported for streaming).

### High CPU Usage

Reduce visualizer complexity or disable it:

```toml
[ui]
show_spectrum = false
```

Or use a lighter visualizer mode (press `V` to cycle).

### Playlist Not Saving

Ensure the directory exists and is writable:

```bash
mkdir -p ~/.config/cliamp/playlists
cliamp --save-playlist ~/.config/cliamp/playlists/my-playlist.m3u
```

### Remote Server Connection Issues

Check firewall rules and server URL:

```bash
# Test Navidrome connection
curl -u "${NAVIDROME_USERNAME}:${NAVIDROME_PASSWORD}" \
  "https://music.example.com/rest/ping.view?v=1.16.1&c=cliamp"

# Test Plex connection
curl -H "X-Plex-Token: ${PLEX_TOKEN}" \
  "https://plex.example.com:32400/library/sections"

# Test Jellyfin connection
curl -H "X-Emby-Token: ${JELLYFIN_API_KEY}" \
  "https://jellyfin.example.com/Users/${JELLYFIN_USER_ID}"
```

## Common Patterns

### Daily Playlist Rotation

```bash
#!/bin/bash
# Rotate through daily playlists
day=$(date +%u)  # 1-7 for Mon-Sun
cliamp --playlist ~/.config/cliamp/playlists/day-${day}.m3u
```

### Random Album Playback

```bash
#!/bin/bash
# Play a random album from library
albums=(~/Music/*/)
random_album=${albums[$RANDOM % ${#albums[@]}]}
cliamp "$random_album"
```

### Status Bar Integration (i3/polybar)

```bash
#!/bin/bash
# Display current track in status bar
status=$(cliamp ipc status --json 2>/dev/null)
if [ $? -eq 0 ]; then
    title=$(echo "$status" | jq -r '.title // "N/A"')
    artist=$(echo "$status" | jq -r '.artist // "N/A"')
    echo "♫ $artist - $title"
else
    echo ""
fi
```

### Auto-Resume Last Playlist

Add to shell profile:

```bash
# ~/.bashrc or ~/.zshrc
alias music='cliamp --playlist ~/.config/cliamp/last-session.m3u'

# Save on exit (add to cliamp config or wrapper script)
trap 'cliamp ipc save-playlist ~/.config/cliamp/last-session.m3u' EXIT
```

This skill covers installation, configuration, CLI usage, plugin development, remote control, integrations, and troubleshooting for the cliamp terminal music player.
