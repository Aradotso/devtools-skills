---
name: webtoon-vault-manga-downloader
description: CLI tool for downloading manga and webcomics from multiple platforms with intelligent source detection and CBZ archiving
triggers:
  - download manga chapters using webtoon vault
  - how do I use manga-pull-cli to scrape comics
  - batch download webcomics from manga sites
  - create CBZ archives from manga chapters
  - configure webtoon vault for offline manga reading
  - troubleshoot manga downloader connection issues
  - download comics from mori manga or dumanwu
  - set up parallel chapter harvesting for manga
---

# Webtoon Vault Manga Downloader

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Webtoon Vault (manga-pull-cli) is a Python-based CLI tool for downloading manga and webcomics from multiple hosting platforms. It features intelligent source detection, parallel chapter harvesting, CBZ archive creation, and support for platforms like Mori Manga, DuManWu, KanKanManHua, ManhuaDB, and GuFeng.

## Installation

### Prerequisites
- Python 3.9 or higher
- pip package manager

### Basic Installation

```bash
# Clone the repository
git clone https://github.com/BunaTechnologyBackUP/manga-pull-cli.git
cd manga-pull-cli

# Install dependencies
pip install -r requirements.txt

# Or install directly if package is available
pip install webtoon-vault
```

### Virtual Environment (Recommended)

```bash
# Create virtual environment
python -m venv venv

# Activate (Linux/macOS)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

## Core Commands

### Basic Download

```bash
# Download a manga series
python manga_pull.py "https://mhpic.com/series/one-piece"

# Download specific chapters
python manga_pull.py "https://mhpic.com/series/one-piece/chapter-1"

# Download chapter range
python manga_pull.py "https://dumanwu.com/manga/jujutsu-kaisen" --chapters 1-50
```

### Output Configuration

```bash
# Specify output directory
python manga_pull.py "URL" --output ./my_manga

# Custom naming template
python manga_pull.py "URL" --format "{series}_vol{volume}_ch{chapter}"

# Flat directory mode (no CBZ)
python manga_pull.py "URL" --flat --no-archive

# Create CBZ archives with metadata
python manga_pull.py "URL" --format cbz --metadata
```

### Download Options

```bash
# Parallel chapter downloads (default: 3)
python manga_pull.py "URL" --parallel 5

# Dry run mode (preview without downloading)
python manga_pull.py "URL" --dry-run

# Resume interrupted download
python manga_pull.py "URL" --resume

# Verbose logging
python manga_pull.py "URL" --verbose
```

### Advanced Features

```bash
# Use proxy
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="https://proxy.example.com:8080"
python manga_pull.py "URL"

# Custom user agent
python manga_pull.py "URL" --user-agent "Mozilla/5.0 Custom"

# Rate limiting (delay between requests in seconds)
python manga_pull.py "URL" --delay 2.5

# Cookie authentication for members-only content
python manga_pull.py "URL" --cookies cookies.txt

# Checksum verification
python manga_pull.py "URL" --verify-checksums
```

## Python API Usage

### Basic Library Usage

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.parsers import detect_platform

# Initialize downloader
downloader = MangaDownloader(
    output_dir="./downloads",
    parallel_downloads=3,
    format="cbz"
)

# Download a series
url = "https://mhpic.com/series/one-piece"
result = downloader.download(url)

print(f"Downloaded {result.chapter_count} chapters")
print(f"Total size: {result.total_size_mb} MB")
```

### Platform Detection

```python
from webtoon_vault.parsers import detect_platform, get_parser

# Detect platform from URL
url = "https://dumanwu.com/manga/demon-slayer"
platform = detect_platform(url)  # Returns: "dumanwu"

# Get appropriate parser
parser = get_parser(platform)
series_info = parser.get_series_info(url)

print(f"Title: {series_info.title}")
print(f"Chapters: {series_info.chapter_count}")
print(f"Language: {series_info.language}")
```

### Chapter-Level Control

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.models import DownloadOptions

# Configure download options
options = DownloadOptions(
    chapter_range=(1, 10),
    create_archive=True,
    include_metadata=True,
    retry_count=5,
    delay_seconds=1.5
)

downloader = MangaDownloader(output_dir="./manga")

# Download with options
result = downloader.download(
    url="https://kankanmanhua.com/series/naruto",
    options=options
)

# Access download details
for chapter in result.chapters:
    print(f"Chapter {chapter.number}: {chapter.page_count} pages")
    print(f"  Path: {chapter.output_path}")
    print(f"  Size: {chapter.size_mb} MB")
```

### Async Batch Downloads

```python
import asyncio
from webtoon_vault import AsyncMangaDownloader

async def batch_download():
    downloader = AsyncMangaDownloader(
        output_dir="./batch_downloads",
        max_concurrent=5
    )
    
    urls = [
        "https://mhpic.com/series/one-piece/chapter-1000",
        "https://dumanwu.com/manga/bleach/chapter-500",
        "https://kankanmanhua.com/series/naruto/chapter-700"
    ]
    
    # Download all concurrently
    results = await downloader.download_batch(urls)
    
    for url, result in zip(urls, results):
        if result.success:
            print(f"✓ {url}: {result.chapter_count} chapters")
        else:
            print(f"✗ {url}: {result.error}")

# Run async download
asyncio.run(batch_download())
```

### Custom Parser Implementation

```python
from webtoon_vault.parsers.base import BaseParser
from webtoon_vault.models import SeriesInfo, ChapterInfo

class CustomSiteParser(BaseParser):
    """Parser for custom manga hosting platform"""
    
    platform_name = "customsite"
    url_pattern = r"customsite\.com"
    
    def get_series_info(self, url: str) -> SeriesInfo:
        response = self.session.get(url)
        soup = self.parse_html(response.text)
        
        return SeriesInfo(
            title=soup.select_one("h1.title").text.strip(),
            url=url,
            chapter_count=len(soup.select(".chapter-link")),
            language="en"
        )
    
    def get_chapter_pages(self, chapter_url: str) -> list[str]:
        response = self.session.get(chapter_url)
        soup = self.parse_html(response.text)
        
        # Extract image URLs
        images = [
            img["src"] for img in soup.select("div.page img")
        ]
        
        return images

# Register custom parser
from webtoon_vault.parsers import register_parser
register_parser(CustomSiteParser)
```

## Configuration File

### JSON Configuration

Create `webtoon_vault_config.json`:

```json
{
  "output_directory": "./manga_library",
  "parallel_downloads": 4,
  "format": "cbz",
  "naming_template": "{series}_chapter_{chapter:03d}",
  "rate_limiting": {
    "enabled": true,
    "delay_seconds": 1.5,
    "adaptive": true
  },
  "retry_policy": {
    "max_attempts": 5,
    "exponential_backoff": true
  },
  "metadata": {
    "include_json": true,
    "embed_in_cbz": true
  },
  "platforms": {
    "mori_manga": {
      "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
      "delay": 2.0
    },
    "dumanwu": {
      "delay": 1.0
    }
  }
}
```

### Use Configuration File

```bash
# Load config from file
python manga_pull.py "URL" --config webtoon_vault_config.json

# Override config values
python manga_pull.py "URL" --config config.json --parallel 8
```

```python
# Python API with config
from webtoon_vault import MangaDownloader
from webtoon_vault.config import load_config

config = load_config("webtoon_vault_config.json")
downloader = MangaDownloader.from_config(config)

downloader.download("https://mhpic.com/series/bleach")
```

## Common Patterns

### Archiving Entire Series

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.models import DownloadOptions

def archive_series(series_url: str, output_dir: str):
    """Download and archive complete manga series"""
    
    options = DownloadOptions(
        create_archive=True,
        include_metadata=True,
        verify_checksums=True,
        retry_count=3
    )
    
    downloader = MangaDownloader(
        output_dir=output_dir,
        parallel_downloads=3
    )
    
    result = downloader.download(series_url, options=options)
    
    # Generate catalog
    with open(f"{output_dir}/catalog.txt", "w", encoding="utf-8") as f:
        f.write(f"Series: {result.series_title}\n")
        f.write(f"Chapters: {result.chapter_count}\n")
        f.write(f"Total Size: {result.total_size_mb:.2f} MB\n\n")
        
        for chapter in result.chapters:
            f.write(f"Chapter {chapter.number}: {chapter.page_count} pages\n")
    
    return result

# Usage
result = archive_series(
    "https://dumanwu.com/manga/demon-slayer",
    "./archives/demon_slayer"
)
```

### Resume Interrupted Downloads

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.cache import SessionCache

def resume_download(series_url: str):
    """Resume previously interrupted download"""
    
    cache = SessionCache.load()
    
    if cache.has_session(series_url):
        print(f"Found cached session for {series_url}")
        last_chapter = cache.get_last_chapter(series_url)
        print(f"Resuming from chapter {last_chapter}")
    
    downloader = MangaDownloader(
        output_dir="./downloads",
        resume_enabled=True
    )
    
    result = downloader.download(series_url)
    
    if result.success:
        cache.clear_session(series_url)
    
    return result
```

### Proxy Rotation

```python
from webtoon_vault import MangaDownloader
import random

def download_with_proxy_rotation(url: str, proxy_list: list[str]):
    """Download with rotating proxy servers"""
    
    def get_random_proxy():
        return random.choice(proxy_list)
    
    downloader = MangaDownloader(
        output_dir="./downloads",
        proxy_provider=get_random_proxy
    )
    
    result = downloader.download(url)
    return result

# Usage
proxies = [
    "http://proxy1.example.com:8080",
    "http://proxy2.example.com:8080",
    "http://proxy3.example.com:8080"
]

download_with_proxy_rotation(
    "https://mhpic.com/series/one-piece",
    proxies
)
```

### Custom Metadata Extraction

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.metadata import MetadataExtractor

class CustomMetadataExtractor(MetadataExtractor):
    """Extract additional metadata fields"""
    
    def extract(self, series_info, chapter_info):
        metadata = super().extract(series_info, chapter_info)
        
        # Add custom fields
        metadata["downloaded_date"] = self.get_current_date()
        metadata["user_rating"] = self.fetch_rating(series_info.url)
        metadata["tags"] = self.extract_tags(series_info.url)
        
        return metadata

downloader = MangaDownloader(
    output_dir="./downloads",
    metadata_extractor=CustomMetadataExtractor()
)
```

### Batch Processing with Progress Tracking

```python
import asyncio
from webtoon_vault import AsyncMangaDownloader
from tqdm import tqdm

async def batch_download_with_progress(urls: list[str]):
    """Download multiple series with progress bar"""
    
    downloader = AsyncMangaDownloader(
        output_dir="./batch",
        max_concurrent=3
    )
    
    progress = tqdm(total=len(urls), desc="Series")
    
    async def download_one(url):
        result = await downloader.download(url)
        progress.update(1)
        return result
    
    tasks = [download_one(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    progress.close()
    
    # Summary
    successful = sum(1 for r in results if not isinstance(r, Exception))
    print(f"\nCompleted: {successful}/{len(urls)} series")
    
    return results

# Usage
urls = [
    "https://mhpic.com/series/bleach",
    "https://dumanwu.com/manga/naruto",
    "https://kankanmanhua.com/series/one-piece"
]

asyncio.run(batch_download_with_progress(urls))
```

## Environment Variables

```bash
# Proxy configuration
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="https://proxy.example.com:8080"

# User agent override
export WEBTOON_VAULT_USER_AGENT="Mozilla/5.0 Custom Agent"

# Output directory
export WEBTOON_VAULT_OUTPUT_DIR="./my_manga"

# Debug mode
export WEBTOON_VAULT_DEBUG=1

# Cookie file path
export WEBTOON_VAULT_COOKIES="~/.webtoon_vault/cookies.txt"

# Rate limiting
export WEBTOON_VAULT_DELAY=2.0
```

## Troubleshooting

### Connection Errors

```python
# Handle connection failures gracefully
from webtoon_vault import MangaDownloader
from webtoon_vault.exceptions import ConnectionError, RateLimitError

downloader = MangaDownloader(
    output_dir="./downloads",
    retry_count=5,
    timeout=30
)

try:
    result = downloader.download(url)
except ConnectionError as e:
    print(f"Connection failed: {e}")
    print("Check your internet connection or proxy settings")
except RateLimitError as e:
    print(f"Rate limited: {e}")
    print("Increase delay with --delay option")
```

### Platform-Specific Issues

```bash
# Enable verbose logging for debugging
python manga_pull.py "URL" --verbose --log-file debug.log

# Test platform detection
python manga_pull.py "URL" --dry-run --verbose

# Force specific platform parser
python manga_pull.py "URL" --platform dumanwu
```

### Image Download Failures

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.models import DownloadOptions

# Increase retry count and timeout
options = DownloadOptions(
    retry_count=10,
    timeout=60,
    verify_checksums=True
)

downloader = MangaDownloader(output_dir="./downloads")
result = downloader.download(url, options=options)

# Check failed downloads
if result.failed_pages:
    print("Failed pages:")
    for page in result.failed_pages:
        print(f"  Chapter {page.chapter}, Page {page.number}: {page.error}")
```

### Memory Issues with Large Series

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.models import DownloadOptions

# Download in smaller batches
def download_in_batches(url: str, batch_size: int = 10):
    downloader = MangaDownloader(output_dir="./downloads")
    parser = downloader.get_parser_for_url(url)
    series_info = parser.get_series_info(url)
    
    total_chapters = series_info.chapter_count
    
    for start in range(1, total_chapters + 1, batch_size):
        end = min(start + batch_size - 1, total_chapters)
        
        options = DownloadOptions(chapter_range=(start, end))
        result = downloader.download(url, options=options)
        
        print(f"Completed chapters {start}-{end}")

# Usage
download_in_batches("https://mhpic.com/series/one-piece", batch_size=20)
```

### Cookie/Authentication Issues

```bash
# Export cookies from browser (use browser extension)
# Save to cookies.txt in Netscape format

# Use cookies for authenticated downloads
python manga_pull.py "URL" --cookies cookies.txt

# Or set environment variable
export WEBTOON_VAULT_COOKIES="cookies.txt"
python manga_pull.py "URL"
```

### CBZ Archive Corruption

```python
from webtoon_vault import MangaDownloader
from webtoon_vault.archive import validate_cbz

# Validate existing CBZ files
def validate_archives(directory: str):
    import os
    from pathlib import Path
    
    for cbz_file in Path(directory).glob("*.cbz"):
        is_valid, error = validate_cbz(cbz_file)
        
        if not is_valid:
            print(f"Corrupt: {cbz_file.name} - {error}")
            # Re-download
            # Extract series URL from metadata and re-download

# Enable checksum verification during download
options = DownloadOptions(
    verify_checksums=True,
    repair_on_error=True
)
```

### Platform Changes/Updates

```python
# Check for parser updates
from webtoon_vault.parsers import check_parser_version

parser_status = check_parser_version("mori_manga")

if parser_status.needs_update:
    print(f"Parser update available: {parser_status.latest_version}")
    print("Run: pip install --upgrade webtoon-vault")

# Force parser refresh
from webtoon_vault.parsers import refresh_parsers
refresh_parsers(force=True)
```

## Best Practices

1. **Always use `--dry-run` first** to preview downloads before executing
2. **Set appropriate delays** (`--delay 2.0`) to avoid being rate-limited
3. **Enable resume functionality** for large series downloads
4. **Use virtual environments** to isolate dependencies
5. **Store configuration** in JSON files for reproducible downloads
6. **Verify checksums** for critical archival work
7. **Respect platform terms of service** and rate limits
8. **Use proxies** if downloading from geo-restricted platforms
9. **Organize output** with custom naming templates
10. **Monitor disk space** when downloading large series in CBZ format
