---
name: nebulavault-media-downloader
description: NebulaVault distributed media orchestration platform for downloading, archiving, and organizing streams from HLS, DASH, HTTP, and RSS sources with CLI and Web UI.
triggers:
  - download videos from streaming platforms
  - archive media content with NebulaVault
  - setup media downloader CLI
  - download HLS or DASH streams
  - create personal media library
  - automate video content downloads
  - configure NebulaVault for streaming content
  - batch download videos from URLs
---

# NebulaVault Media Downloader

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

NebulaVault is a distributed content orchestration platform for downloading, archiving, and organizing digital media from various streaming sources. It supports multiple protocols (HLS/m3u8, DASH/mpd, HTTP/HTTPS, RSS feeds) and provides both a command-line interface and a responsive Web UI for managing downloads.

## Installation

### Prerequisites

Ensure you have the following installed:
- Node.js 18+ or Deno runtime
- `ffmpeg` for post-processing (transcoding, thumbnail generation)
- SQLite (bundled with most installations)

### Quick Install

```bash
# Download from releases
wget https://github.com/Ronish17/redbull-archive-cli/releases/latest/download/nebulavault-linux-x64.tar.gz

# Extract
tar -xzf nebulavault-linux-x64.tar.gz -C ~/nebulavault

# Navigate to directory
cd ~/nebulavault

# Run initialization wizard
./nebulavault init

# Verify installation
./nebulavault --version
```

### Alternative: Build from Source

```bash
git clone https://github.com/Ronish17/redbull-archive-cli.git
cd redbull-archive-cli
npm install
npm run build
```

## Configuration

### Initial Setup

Run the initialization wizard to set up default directories and profiles:

```bash
./nebulavault init
```

This creates a `config/settings.json` file with default parameters.

### Manual Configuration

Edit `config/settings.json`:

```json
{
  "storage": {
    "basePath": "/home/user/media",
    "maxDiskUsage": "500GB",
    "autoCleanup": true
  },
  "network": {
    "maxConcurrentDownloads": 3,
    "retryAttempts": 5,
    "chunkSize": "5MB",
    "rateLimitMbps": 50
  },
  "quality": {
    "preferredResolution": "1080p",
    "preferredCodec": "h264",
    "autoSelectBest": true
  },
  "server": {
    "webUIPort": 5600,
    "enableSSL": false,
    "apiKey": "${NEBULAVAULT_API_KEY}"
  }
}
```

### Environment Variables

```bash
# API authentication
export NEBULAVAULT_API_KEY="your-secure-api-key"

# Custom storage path
export NEBULAVAULT_STORAGE="/mnt/external/media"

# FFmpeg binary location (if not in PATH)
export FFMPEG_PATH="/usr/local/bin/ffmpeg"

# Enable debug logging
export NEBULAVAULT_DEBUG=true
```

## CLI Usage

### Basic Download Commands

```bash
# Download a single video
./nebulavault download "https://example.com/video.m3u8"

# Download with custom output name
./nebulavault download "https://example.com/video.m3u8" --output "my-video.mp4"

# Download to specific directory
./nebulavault download "https://example.com/video.m3u8" --dir "/home/user/videos"

# Download with quality selection
./nebulavault download "https://example.com/video.m3u8" --quality 720p

# Download with subtitle extraction
./nebulavault download "https://example.com/video.m3u8" --subtitles --lang en,es
```

### Batch Downloads

```bash
# Download from URL list file
./nebulavault batch --file urls.txt

# Example urls.txt format:
# https://example.com/video1.m3u8
# https://example.com/video2.mpd
# https://example.com/video3.mp4
```

### RSS/Feed Monitoring

```bash
# Add RSS feed for automated downloads
./nebulavault feed add "https://example.com/podcast.rss" --name "My Podcast"

# List tracked feeds
./nebulavault feed list

# Check for new episodes and download
./nebulavault feed update --all

# Remove feed
./nebulavault feed remove "My Podcast"
```

### Queue Management

```bash
# View active downloads
./nebulavault queue list

# Pause specific download
./nebulavault queue pause <download-id>

# Resume paused download
./nebulavault queue resume <download-id>

# Cancel download
./nebulavault queue cancel <download-id>

# Clear completed downloads from queue
./nebulavault queue clear
```

### Library Management

```bash
# Search downloaded content
./nebulavault library search "documentary"

# List all downloads with metadata
./nebulavault library list --sort date

# Show details for specific item
./nebulavault library info <item-id>

# Delete from library
./nebulavault library delete <item-id>

# Export library metadata
./nebulavault library export --format json --output library.json
```

### Service Management

```bash
# Start background service
./nebulavault service start

# Stop service
./nebulavault service stop

# Check service status
./nebulavault service status

# View service logs
./nebulavault service logs --tail 50
```

## Web UI

### Starting the Web Interface

```bash
# Start with default port (5600)
./nebulavault serve

# Start with custom port
./nebulavault serve --port 8080

# Start with SSL
./nebulavault serve --ssl --cert /path/to/cert.pem --key /path/to/key.pem
```

Access the Web UI at `http://localhost:5600`

### API Endpoints

The Web UI exposes a REST API for programmatic access:

```bash
# Get download status
curl -H "Authorization: Bearer ${NEBULAVAULT_API_KEY}" \
  http://localhost:5600/api/downloads

# Start new download
curl -X POST -H "Authorization: Bearer ${NEBULAVAULT_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/video.m3u8", "quality": "1080p"}' \
  http://localhost:5600/api/downloads

# Get library items
curl -H "Authorization: Bearer ${NEBULAVAULT_API_KEY}" \
  http://localhost:5600/api/library
```

## Advanced Usage

### Quality Profiles

Create custom quality profiles in `config/profiles.json`:

```json
{
  "profiles": {
    "mobile": {
      "resolution": "480p",
      "codec": "h264",
      "audioBitrate": "96k",
      "videoBitrate": "800k"
    },
    "standard": {
      "resolution": "720p",
      "codec": "h264",
      "audioBitrate": "128k",
      "videoBitrate": "2500k"
    },
    "premium": {
      "resolution": "1080p",
      "codec": "h265",
      "audioBitrate": "256k",
      "videoBitrate": "5000k"
    }
  }
}
```

Use profiles in commands:

```bash
./nebulavault download "https://example.com/video.m3u8" --profile premium
```

### Post-Processing Scripts

Configure automatic post-processing in `config/settings.json`:

```json
{
  "postProcessing": {
    "enabled": true,
    "scripts": [
      {
        "name": "thumbnail",
        "command": "ffmpeg -i {input} -ss 00:00:05 -vframes 1 {output}.jpg"
      },
      {
        "name": "transcode",
        "command": "ffmpeg -i {input} -c:v libx265 -preset medium {output}_h265.mp4"
      }
    ]
  }
}
```

### Plugin System

NebulaVault supports plugins for custom extractors:

```javascript
// plugins/custom-extractor.js
module.exports = {
  name: 'custom-extractor',
  version: '1.0.0',
  
  canHandle(url) {
    return url.includes('custom-site.com');
  },
  
  async extract(url) {
    // Custom extraction logic
    return {
      streamUrl: 'https://cdn.custom-site.com/video.m3u8',
      title: 'Video Title',
      description: 'Description',
      duration: 3600,
      thumbnail: 'https://cdn.custom-site.com/thumb.jpg'
    };
  }
};
```

Load plugins:

```bash
./nebulavault plugin install ./plugins/custom-extractor.js
./nebulavault plugin list
```

## Common Patterns

### Archiving a Video Series

```bash
# Create a dedicated directory
mkdir -p ~/media/documentary-series

# Download series with consistent naming
for i in {1..10}; do
  ./nebulavault download "https://example.com/series/episode${i}.m3u8" \
    --output "episode-$(printf '%02d' $i).mp4" \
    --dir ~/media/documentary-series \
    --quality 1080p \
    --subtitles
done
```

### Automated Daily Downloads

Create a cron job for automated downloads:

```bash
# Edit crontab
crontab -e

# Add daily RSS check at 2 AM
0 2 * * * /home/user/nebulavault/nebulavault feed update --all >> /var/log/nebulavault.log 2>&1
```

### Bandwidth-Limited Downloads

```bash
# Limit download speed to 5 Mbps
./nebulavault download "https://example.com/video.m3u8" --rate-limit 5M

# Schedule downloads during off-peak hours
./nebulavault download "https://example.com/video.m3u8" --schedule "02:00"
```

### Resume Interrupted Downloads

```bash
# NebulaVault automatically resumes on restart, but can be forced:
./nebulavault queue resume --all

# Check integrity of partial downloads
./nebulavault verify ~/media/incomplete-video.mp4
```

## Troubleshooting

### Download Fails with "Stream Not Accessible"

```bash
# Check URL accessibility
curl -I "https://example.com/video.m3u8"

# Try with custom headers (some streams require user-agent)
./nebulavault download "https://example.com/video.m3u8" \
  --header "User-Agent: Mozilla/5.0" \
  --header "Referer: https://example.com"
```

### FFmpeg Not Found

```bash
# Install FFmpeg
sudo apt install ffmpeg  # Debian/Ubuntu
brew install ffmpeg       # macOS

# Or specify custom path
export FFMPEG_PATH="/usr/local/bin/ffmpeg"
./nebulavault download "https://example.com/video.m3u8"
```

### Disk Space Issues

```bash
# Check available space
df -h ~/media

# Enable auto-cleanup in config
# Edit config/settings.json:
{
  "storage": {
    "autoCleanup": true,
    "minFreeSpace": "50GB"
  }
}

# Manually clean old downloads
./nebulavault library cleanup --older-than 30d
```

### Slow Download Speeds

```bash
# Increase concurrent connections
./nebulavault download "https://example.com/video.m3u8" --connections 8

# Adjust chunk size
./nebulavault download "https://example.com/video.m3u8" --chunk-size 10M

# Check network config in settings
# Edit config/settings.json:
{
  "network": {
    "maxConcurrentDownloads": 5,
    "chunkSize": "10MB"
  }
}
```

### Corrupt or Incomplete Downloads

```bash
# Verify file integrity
./nebulavault verify ~/media/video.mp4

# Re-download with forced integrity checks
./nebulavault download "https://example.com/video.m3u8" \
  --verify-segments \
  --retry 10
```

### Web UI Not Accessible

```bash
# Check if service is running
./nebulavault service status

# Check port availability
netstat -tuln | grep 5600

# Start with verbose logging
./nebulavault serve --port 5600 --verbose

# Check firewall rules
sudo ufw allow 5600/tcp
```

### HLS Key Decryption Errors

```bash
# Ensure encryption keys are accessible
./nebulavault download "https://example.com/encrypted.m3u8" \
  --key-url "https://example.com/keys/" \
  --verbose

# Some streams require cookies
./nebulavault download "https://example.com/encrypted.m3u8" \
  --cookies cookies.txt
```

## Best Practices

1. **Always verify URLs** before batch downloads to avoid wasting bandwidth
2. **Use quality profiles** for consistent output across downloads
3. **Enable automatic checksums** to ensure file integrity
4. **Monitor disk space** when archiving large collections
5. **Respect rate limits** to avoid being blocked by content providers
6. **Use environment variables** for sensitive configuration (API keys, tokens)
7. **Keep logs** for debugging and audit trails
8. **Regular backups** of the library database and configuration files

## Legal and Ethical Considerations

NebulaVault is designed for **lawful content acquisition only**. Ensure you have the legal right to download and store any content. The tool does not bypass DRM, circumvent paywalls, or enable unauthorized access. It is intended for:

- Public domain content
- Creative Commons licensed media
- Content you own or have permission to archive
- Freely accessible educational resources
- Open-access scientific broadcasts

Always comply with the terms of service of content providers and respect copyright laws in your jurisdiction.
