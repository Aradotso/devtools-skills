---
name: webtoon-vault-manga-downloader
description: CLI tool for downloading webcomics and manga from multiple platforms with intelligent extraction and CBZ packaging
triggers:
  - download manga from a website
  - scrape webcomic chapters
  - create CBZ archives from manga
  - batch download comic series
  - extract manga chapters offline
  - archive webcomics locally
  - download manga with webtoon vault
  - fetch comic images from hosting platforms
---

# Webtoon Vault Manga Downloader

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Webtoon Vault is a Python-based CLI tool for downloading webcomics and manga from multiple hosting platforms. It features intelligent source detection, parallel chapter harvesting, automatic CBZ archive creation, and resume/retry logic for interrupted downloads.

**Key capabilities:**
- Automatic platform detection from URL patterns
- Parallel chapter downloads with async I/O
- CBZ (Comic Book ZIP) archive generation with metadata
- Multi-platform support (Mori Manga, DuManWu, KanKanManHua, ManhuaDB, GuFeng)
- Resume interrupted downloads
- Custom output naming templates
- Rate limiting and retry logic

## Installation

Based on the project structure, this is likely distributed as a Python package or standalone executable:

```bash
# If distributed via pip
pip install webtoon-vault

# Or clone from repository
git clone https://github.com/BunaTechnologyBackUP/manga-pull-cli.git
cd manga-pull-cli
pip install -r requirements.txt
python setup.py install
```

**Requirements:**
- Python 3.9+
- Dependencies: asyncio, aiohttp, Pillow, beautifulsoup4, zipfile

## Core CLI Commands

### Basic Download

```bash
# Download all chapters from a series
webtoon-vault download "https://mhpic.com/comic/one-piece"

# Download specific chapter range
webtoon-vault download "https://dumanwu.com/manga/jujutsu-kaisen" --chapters 1-50

# Download single chapter
webtoon-vault download "https://kankanmanhua.com/chapter/123456" --single
```

### Output Configuration

```bash
# Specify output directory
webtoon-vault download URL --output /path/to/comics

# Use custom naming template
webtoon-vault download URL --template "{series}_v{volume}_ch{chapter}"

# Save as loose images instead of CBZ
webtoon-vault download URL --format images

# Organize by series folders
webtoon-vault download URL --organize
```

### Download Options

```bash
# Parallel downloads (max concurrent chapters)
webtoon-vault download URL --parallel 5

# Set rate limit (requests per second)
webtoon-vault download URL --rate-limit 2

# Resume interrupted download
webtoon-vault download URL --resume

# Dry run (preview without downloading)
webtoon-vault download URL --dry-run

# Verbose logging
webtoon-vault download URL --verbose
```

### Advanced Features

```bash
# Use proxy
webtoon-vault download URL --proxy "http://proxy.example.com:8080"

# Custom User-Agent
webtoon-vault download URL --user-agent "Mozilla/5.0..."

# Include cookies for authentication
webtoon-vault download URL --cookies "session=abc123; auth=xyz789"

# Generate checksums
webtoon-vault download URL --checksum
```

## Python API Usage

If used as a library, here's the typical integration pattern:

```python
import asyncio
from webtoon_vault import WebcomicDownloader, DownloadConfig

async def download_manga():
    # Create configuration
    config = DownloadConfig(
        url="https://mhpic.com/comic/one-piece",
        output_dir="/home/user/comics",
        format="cbz",
        parallel=3,
        rate_limit=2.0,
        naming_template="{series}_ch{chapter:03d}"
    )
    
    # Initialize downloader
    downloader = WebcomicDownloader(config)
    
    # Download series
    result = await downloader.download()
    
    print(f"Downloaded {result.chapters_downloaded} chapters")
    print(f"Total size: {result.total_size_mb} MB")
    print(f"Failed: {result.failed_chapters}")

# Run download
asyncio.run(download_manga())
```

### Platform-Specific Parsers

```python
from webtoon_vault.parsers import MoriParser, DuManWuParser

async def custom_extraction():
    # Use specific platform parser
    parser = MoriParser()
    
    # Discover chapters
    chapters = await parser.get_chapter_list("https://mhpic.com/comic/one-piece")
    
    for chapter in chapters[:5]:  # First 5 chapters
        print(f"Chapter {chapter.number}: {chapter.title}")
        
        # Extract pages
        pages = await parser.get_chapter_pages(chapter.url)
        
        # Download images
        images = await parser.download_images(pages)
        
        # Create CBZ
        await parser.create_cbz(images, f"chapter_{chapter.number}.cbz")
```

### Resume Logic

```python
from webtoon_vault import SessionCache

async def resume_download():
    # Load session cache
    cache = SessionCache.load("/home/user/.webtoon_vault_cache.json")
    
    config = DownloadConfig(
        url="https://dumanwu.com/manga/series",
        output_dir="/home/user/comics",
        resume_from_cache=cache
    )
    
    downloader = WebcomicDownloader(config)
    
    # Will skip already downloaded chapters
    await downloader.download()
    
    # Save updated cache
    cache.save()
```

## Configuration File

Create a `webtoon_vault.json` for persistent settings:

```json
{
  "output_dir": "/home/user/comics",
  "format": "cbz",
  "naming_template": "{series}_v{volume:02d}_ch{chapter:03d}",
  "parallel_downloads": 4,
  "rate_limit": 2.5,
  "user_agent": "WebtoonVault/1.0",
  "retry_attempts": 5,
  "retry_delay": 2,
  "organize_by_series": true,
  "generate_checksums": false,
  "proxy": null,
  "platforms": {
    "mori": {
      "enabled": true,
      "custom_headers": {}
    },
    "dumanwu": {
      "enabled": true,
      "delay_between_requests": 1.5
    }
  }
}
```

Load config:

```bash
webtoon-vault download URL --config webtoon_vault.json
```

## Common Patterns

### Batch Download Multiple Series

```python
import asyncio
from webtoon_vault import WebcomicDownloader, DownloadConfig

async def batch_download():
    series_urls = [
        "https://mhpic.com/comic/one-piece",
        "https://dumanwu.com/manga/naruto",
        "https://kankanmanhua.com/comic/bleach"
    ]
    
    for url in series_urls:
        config = DownloadConfig(
            url=url,
            output_dir="/home/user/comics",
            format="cbz"
        )
        
        downloader = WebcomicDownloader(config)
        
        try:
            result = await downloader.download()
            print(f"✓ {result.series_name}: {result.chapters_downloaded} chapters")
        except Exception as e:
            print(f"✗ Failed {url}: {e}")

asyncio.run(batch_download())
```

### Custom Metadata Extraction

```python
from webtoon_vault import MetadataExtractor

async def extract_metadata():
    extractor = MetadataExtractor()
    
    metadata = await extractor.extract("https://mhpic.com/comic/one-piece")
    
    print(f"Title: {metadata.title}")
    print(f"Author: {metadata.author}")
    print(f"Genres: {', '.join(metadata.genres)}")
    print(f"Total chapters: {metadata.total_chapters}")
    print(f"Status: {metadata.status}")
    
    # Save metadata
    metadata.save_json("one_piece_metadata.json")
```

### Proxy Rotation

```python
from webtoon_vault import ProxyManager

async def download_with_proxy_rotation():
    proxies = [
        "http://proxy1.example.com:8080",
        "http://proxy2.example.com:8080",
        "http://proxy3.example.com:8080"
    ]
    
    proxy_mgr = ProxyManager(proxies, rotation_strategy="round_robin")
    
    config = DownloadConfig(
        url="https://mhpic.com/comic/one-piece",
        output_dir="/home/user/comics",
        proxy_manager=proxy_mgr
    )
    
    downloader = WebcomicDownloader(config)
    await downloader.download()
```

### Error Handling and Logging

```python
import logging
from webtoon_vault import WebcomicDownloader, DownloadConfig
from webtoon_vault.exceptions import PlatformNotSupported, ChapterNotFound

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("webtoon_vault")

async def robust_download():
    config = DownloadConfig(
        url="https://example.com/comic/series",
        output_dir="/home/user/comics",
        retry_attempts=5
    )
    
    downloader = WebcomicDownloader(config)
    
    try:
        result = await downloader.download()
        
        if result.failed_chapters:
            logger.warning(f"Failed chapters: {result.failed_chapters}")
            
            # Retry failed chapters
            await downloader.retry_failed()
            
    except PlatformNotSupported as e:
        logger.error(f"Platform not supported: {e}")
    except ChapterNotFound as e:
        logger.error(f"Chapter not found: {e}")
    except Exception as e:
        logger.exception(f"Unexpected error: {e}")
```

## Environment Variables

Set these for authentication, proxies, or configuration:

```bash
# Proxy settings
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="http://proxy.example.com:8080"

# Output directory default
export WEBTOON_VAULT_OUTPUT="/home/user/comics"

# Rate limiting
export WEBTOON_VAULT_RATE_LIMIT="2.0"

# Parallel downloads
export WEBTOON_VAULT_PARALLEL="4"

# User agent
export WEBTOON_VAULT_USER_AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

# Cookie authentication (if needed)
export WEBTOON_VAULT_COOKIES="session=xyz; auth=abc"
```

## Troubleshooting

### Rate Limiting / 429 Errors

```bash
# Reduce rate limit
webtoon-vault download URL --rate-limit 1.0

# Add delay between requests
webtoon-vault download URL --delay 2.0
```

```python
config = DownloadConfig(
    url=url,
    rate_limit=1.0,  # 1 request per second
    retry_attempts=10,
    retry_delay=3  # Wait 3 seconds between retries
)
```

### Platform Detection Failure

```bash
# Force specific platform parser
webtoon-vault download URL --platform mori

# Check supported platforms
webtoon-vault platforms list
```

### Incomplete Downloads

```bash
# Resume with session cache
webtoon-vault download URL --resume

# Verify with checksums
webtoon-vault verify /path/to/comics --checksums
```

### Authentication Required

```bash
# Pass cookies from browser
webtoon-vault download URL --cookies "$(cat cookies.txt)"

# Use session file
webtoon-vault download URL --session session.json
```

### Memory Issues with Large Series

```python
# Download in batches
async def batched_download():
    config = DownloadConfig(
        url="https://mhpic.com/comic/one-piece",
        output_dir="/home/user/comics",
        parallel=2,  # Reduce parallel downloads
        batch_size=50  # Process 50 chapters at a time
    )
    
    downloader = WebcomicDownloader(config)
    await downloader.download_batched()
```

### Image Extraction Failures

```python
from webtoon_vault.parsers import BaseParser

class CustomParser(BaseParser):
    async def extract_images(self, page_html):
        # Custom extraction logic for non-standard sites
        soup = BeautifulSoup(page_html, 'html.parser')
        
        # Try multiple selectors
        images = soup.select('img.comic-page')
        if not images:
            images = soup.select('div.reader img')
        if not images:
            images = soup.select('[data-src]')
            
        return [img.get('src') or img.get('data-src') for img in images]
```

## Platform Support Reference

| Platform | URL Pattern | Status | Notes |
|----------|-------------|--------|-------|
| Mori Manga | `mhpic.com`, `mori.*` | Active | Supports lazy loading |
| DuManWu | `dumanwu.*` | Active | Requires rate limiting |
| KanKanManHua | `kankanmanhua.*` | Active | JavaScript pagination |
| ManhuaDB | `manhuadb.*` | Beta | Authentication sometimes needed |
| GuFeng | `gufengmh.*` | Active | Multi-language support |

Check current platform status:

```bash
webtoon-vault platforms status
```
