---
name: spotify-mp3-cli-downloader
description: CLI tool for downloading Spotify tracks and playlists as MP3 with embedded metadata and album artwork
triggers:
  - "how do I download Spotify tracks to MP3"
  - "extract music from Spotify playlists"
  - "save Spotify songs with metadata"
  - "convert Spotify to local audio files"
  - "download Spotify playlist as MP3"
  - "archive Spotify music offline"
  - "use streamscribe cli"
  - "backup Spotify library to disk"
---

# Spotify MP3 CLI Downloader (StreamScribe CLI)

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

StreamScribe CLI (cli-sonic-harvester) is a command-line tool for extracting audio from Spotify, Deezer, Tidal, and Apple Music streams. It downloads tracks as MP3/FLAC files with full metadata preservation including ID3v2 tags, album artwork, track numbers, and genre information. Designed for archivists, DJs, and music collectors who need offline access with complete metadata integrity.

**Primary Language**: HTML (project page), but the tool itself runs via Python 3.10+ or Node.js 18+

**Key Features**:
- Single track and playlist downloads
- Automatic metadata extraction (artist, album, year, genre, composer)
- High-resolution album art embedding
- Multiple quality presets (128 kbps to lossless FLAC)
- Concurrent processing for playlists
- Unicode/multilingual metadata support

## Installation

### Prerequisites

```bash
# Verify Python version (3.10+ required)
python3 --version

# OR Node.js version (18+ required)
node --version
```

### Download and Setup

The tool is distributed via the project website:

```bash
# Download from official page
# Visit: https://guess-eng.github.io/cli-sonic-harvester/

# After download, make executable (Linux/macOS)
chmod +x streamscribe

# Move to PATH
sudo mv streamscribe /usr/local/bin/

# Verify installation
streamscribe --version
```

### Environment Configuration

```bash
# Optional: Configure cache directory
export STREAMSCRIBE_CACHE="$HOME/.cache/streamscribe"

# Optional: Set proxy for restricted networks
export STREAMSCRIBE_PROXY="http://proxy.example.com:8080"

# Optional: Adjust stream timeout (default: 30 seconds)
export STREAMSCRIBE_TIMEOUT=60
```

## Basic Usage

### Single Track Download

```bash
# Download a single Spotify track (default quality)
streamscribe --url "https://open.spotify.com/track/3n3Ppam7vgaVa1iaRUc9Lp"

# Output location: ./StreamScribe/Output/Artist - Title.mp3
```

### Playlist Download

```bash
# Download entire playlist
streamscribe --playlist "https://open.spotify.com/playlist/37i9dQZF1DXcBWIGoYBM5M"

# Specify custom output directory
streamscribe --playlist "https://open.spotify.com/playlist/37i9dQZF1DXcBWIGoYBM5M" \
  --output "$HOME/Music/Spotify/Playlists"
```

### Album Download

```bash
# Download full album with proper disc/track numbering
streamscribe --url "https://open.spotify.com/album/6DEjYFkNZh67HP7R9PSZvv" \
  --metadata-extended
```

## Quality Presets

### Standard Quality (Default)

```bash
# 128 kbps MP3 - smallest file size
streamscribe --url "SPOTIFY_URL" --quality standard
```

### High Quality

```bash
# 256 kbps AAC/M4A - balanced quality/size
streamscribe --url "SPOTIFY_URL" --quality high
```

### Archival/Lossless

```bash
# FLAC lossless - maximum quality
streamscribe --url "SPOTIFY_URL" --quality archival
```

## Metadata Options

### Extended Metadata Extraction

```bash
# Include composer, ISRC, label, and publisher info
streamscribe --url "SPOTIFY_URL" \
  --metadata-extended \
  --embed-artwork \
  --overwrite
```

### High-Resolution Artwork

```bash
# Force ≥1200x1200 album art embedding
streamscribe --playlist "PLAYLIST_URL" \
  --embed-artwork \
  --artwork-size 1400
```

## Output Formats

### Human-Readable (Default)

```bash
# Color-coded terminal output with progress bars
streamscribe --url "SPOTIFY_URL"
```

### JSON Output

```bash
# Structured output for scripting
streamscribe --playlist "PLAYLIST_URL" --format json

# Example output:
# {
#   "tracks": [
#     {
#       "title": "Song Title",
#       "artist": "Artist Name",
#       "album": "Album Name",
#       "year": 2026,
#       "duration": 215,
#       "file_path": "./StreamScribe/Output/Artist - Song Title.mp3"
#     }
#   ]
# }
```

### YAML Output

```bash
# Configuration-friendly format
streamscribe --playlist "PLAYLIST_URL" --format yaml
```

### Quiet Mode

```bash
# Silent operation, errors only
streamscribe --url "SPOTIFY_URL" --format quiet
```

## Integration Patterns

### Piping to JSON Processors

```bash
# Extract specific metadata fields
streamscribe --playlist "PLAYLIST_URL" --format json | \
  jq '.tracks[] | {title, artist, album}'

# Filter by year
streamscribe --playlist "PLAYLIST_URL" --format json | \
  jq '.tracks[] | select(.year >= 2025)'
```

### Batch Processing with Shell Scripts

```bash
#!/bin/bash
# batch_download.sh

# Read URLs from file
while IFS= read -r url; do
  streamscribe --url "$url" \
    --quality high \
    --output "./Archive/$(date +%Y-%m)" \
    --format quiet
done < spotify_urls.txt
```

### Integration with Music Library Managers

```bash
# Download and auto-import to Beets
streamscribe --playlist "PLAYLIST_URL" \
  --output "$HOME/Music/Inbox" \
  --format json > import.json

# Then use Beets importer
beet import "$HOME/Music/Inbox"
```

## Advanced Configuration

### Resume Interrupted Downloads

```bash
# StreamScribe automatically resumes from checkpoint
# If download fails partway through playlist:
streamscribe --playlist "PLAYLIST_URL" --resume
```

### Concurrent Processing Control

```bash
# Limit concurrent downloads (default: auto-detected cores)
streamscribe --playlist "PLAYLIST_URL" \
  --threads 4 \
  --quality high
```

### Custom Naming Templates

```bash
# Organize by artist/album structure
streamscribe --playlist "PLAYLIST_URL" \
  --output-template "{artist}/{album}/{track:02d} - {title}"

# Output: Artist Name/Album Name/01 - Song Title.mp3
```

## Platform-Specific Usage

### Spotify

```bash
# Track URL format
streamscribe --url "https://open.spotify.com/track/[TRACK_ID]?si=..."

# Playlist URL format
streamscribe --playlist "https://open.spotify.com/playlist/[PLAYLIST_ID]?si=..."
```

### Deezer

```bash
streamscribe --url "https://www.deezer.com/track/[TRACK_ID]"
```

### Tidal

```bash
streamscribe --url "https://tidal.com/browse/track/[TRACK_ID]"
```

### Apple Music

```bash
streamscribe --url "https://music.apple.com/album/[ALBUM_ID]?i=[TRACK_ID]"
```

## Troubleshooting

### Authentication Issues

```bash
# Some services require API credentials for higher rate limits
export SPOTIFY_CLIENT_ID="your_client_id_here"
export SPOTIFY_CLIENT_SECRET="your_client_secret_here"

streamscribe --url "SPOTIFY_URL"
```

### Network Timeout Errors

```bash
# Increase timeout for slow connections
export STREAMSCRIBE_TIMEOUT=120
streamscribe --url "SPOTIFY_URL"
```

### Metadata Encoding Issues

```bash
# Force UTF-8 encoding for non-Latin scripts
streamscribe --url "SPOTIFY_URL" \
  --encoding utf-8 \
  --metadata-extended
```

### Disk Space Warnings

```bash
# Check available space before large playlist downloads
df -h ./StreamScribe/Output/

# Pre-calculate playlist size
streamscribe --playlist "PLAYLIST_URL" \
  --dry-run \
  --format json | jq '[.tracks[].estimated_size] | add'
```

### Corrupted Downloads

```bash
# Verify file integrity and re-download if needed
streamscribe --url "SPOTIFY_URL" \
  --verify-checksum \
  --overwrite
```

## Legal and Ethical Considerations

**Important**: This tool is for personal archival and educational use only. Users must:

1. Have an active subscription or free-tier access to the streaming service
2. Comply with local copyright laws and platform Terms of Service
3. Not redistribute downloaded content
4. Verify fair use exemptions in their jurisdiction

The tool does not bypass DRM or encryption — it captures streams already accessible through legitimate access.

## Common Workflows

### Create Local Backup of Spotify Library

```bash
#!/bin/bash
# backup_library.sh

# Export all playlists
for playlist_url in $(cat my_playlists.txt); do
  playlist_name=$(echo "$playlist_url" | sed 's/.*playlist\///' | cut -d'?' -f1)
  
  streamscribe --playlist "$playlist_url" \
    --output "./Backup/$playlist_name" \
    --quality archival \
    --metadata-extended \
    --embed-artwork
done
```

### DJ Sample Library Creation

```bash
# Download tracks with extended metadata for cataloging
streamscribe --playlist "https://open.spotify.com/playlist/DJ_CRATE" \
  --quality high \
  --output "./DJ/Samples/2026" \
  --metadata-extended \
  --output-template "{genre}/{bpm} - {artist} - {title}"
```

### Academic Research Archive

```bash
# Preserve world music playlists with original metadata
streamscribe --playlist "ETHNOMUSICOLOGY_PLAYLIST" \
  --quality archival \
  --metadata-extended \
  --encoding utf-8 \
  --format yaml > research_metadata.yaml
```

## Performance Optimization

### Large Playlist Processing

```bash
# Use concurrent processing with rate limiting
streamscribe --playlist "LARGE_PLAYLIST" \
  --threads 8 \
  --rate-limit 10 \
  --quality high \
  --format quiet
```

### Memory-Constrained Environments

```bash
# Reduce memory footprint
export STREAMSCRIBE_BUFFER_SIZE=4096
streamscribe --playlist "PLAYLIST_URL" \
  --no-cache \
  --sequential
```

## Version Information

Current stable version: v2.1.0 (as of 2026-03-15)

Check for updates:
```bash
streamscribe --check-updates
```

## Support Resources

- GitHub Issues: https://github.com/Guess-eng/cli-sonic-harvester/issues
- Community Forum: https://streamscribe.discourse.group (referenced but may not exist)
- FAQ: https://streamscribe.io/docs/faq (referenced but may not exist)

For bug reports, include:
```bash
# Generate diagnostic report
streamscribe --url "PROBLEMATIC_URL" --debug > debug_report.log
```
