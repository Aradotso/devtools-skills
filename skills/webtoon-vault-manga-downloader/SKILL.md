---
name: webtoon-vault-manga-downloader
description: CLI manga and webcomic downloader with multi-platform support, intelligent pagination, and CBZ archive creation
triggers:
  - download manga chapters from a URL
  - scrape webcomics to CBZ format
  - batch download manga series offline
  - extract chapters from manga hosting platforms
  - create comic book archives from webcomics
  - download manga with automatic platform detection
  - fetch manga chapters with parallel downloads
  - archive webcomics for offline reading
---

# Webtoon Vault Manga Downloader

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Webtoon Vault is a sophisticated CLI-based manga and webcomic downloader that automatically detects source platforms, downloads chapters in parallel, and packages them into CBZ (Comic Book ZIP) archives. It supports multiple Chinese and Japanese manga hosting platforms with intelligent anti-scraping countermeasures, resume logic, and metadata extraction.

## Installation

**Note:** The project description indicates this is a Python-based CLI tool, but the repository language is listed as HTML. The actual implementation appears to be Python-based based on the README references to "Python's asynchronous I/O" and "Python 3.9+".

```bash
# Clone the repository
git clone https://github.com/BunaTechnologyBackUP/manga-pull-cli.git
cd manga-pull-cli

# Install dependencies (assuming requirements.txt exists)
pip install -r requirements.txt

# Or install via pip if packaged
pip install webtoon-vault
```

**Requirements:**
- Python 3.9 or higher
- Cross-platform support (Windows, macOS, Linux)

## Core Concepts

### Supported Platforms

Webtoon Vault automatically detects and supports:

| Platform | URL Pattern | Status |
|----------|-------------|--------|
| Mori Manga | `mhpic.com`, `mori.*` | Active |
| DuManWu | `dumanwu.*` | Active |
| KanKanManHua | `kankanmanhua.*` | Active |
| ManhuaDB | `manhuadb.*` | Beta |
| GuFeng | `gufengmh.*` | Active |

### Three-Stage Pipeline

1. **Discovery Phase**: Extracts platform identifier and builds chapter list
2. **Harvest Phase**: Fetches all pages with parallel async downloads
3. **Assembly Phase**: Packages images into CBZ archives with metadata

## Basic Usage

### Download a Single Series

```bash
# Basic download with automatic platform detection
webtoon-vault download "https://mhpic.com/series/one-piece"

# Download with custom output directory
webtoon-vault download "https://dumanwu.com/manga/jujutsu-kaisen" --output ./my-manga

# Download specific chapters
webtoon-vault download "https://kankanmanhua.com/series/naruto" --chapters 1-10,15,20-25
```

### Parallel Chapter Downloads

```bash
# Download multiple chapters in parallel (default: 3 concurrent)
webtoon-vault download "https://mhpic.com/series/attack-on-titan" --parallel 5

# Single-threaded download (safer for rate-limited platforms)
webtoon-vault download "https://manhuadb.com/series/bleach" --parallel 1
```

### Output Format Options

```bash
# Save as loose images instead of CBZ
webtoon-vault download "https://gufengmh.com/manga/demon-slayer" --format loose

# Create CBZ archives (default)
webtoon-vault download "https://dumanwu.com/manga/tokyo-ghoul" --format cbz

# Custom naming template
webtoon-vault download "https://mhpic.com/series/hunter-x-hunter" \
  --naming "{series}_v{volume:02d}_ch{chapter:03d}_{date}"
```

### Advanced Features

```bash
# Dry-run mode (preview without downloading)
webtoon-vault download "https://kankanmanhua.com/series/one-punch-man" --dry-run

# Verbose logging for debugging
webtoon-vault download "https://mhpic.com/series/berserk" --verbose

# Resume interrupted download
webtoon-vault download "https://dumanwu.com/manga/vagabond" --resume

# Use proxy for access
HTTP_PROXY=http://proxy.example.com:8080 webtoon-vault download "https://manhuadb.com/series/slam-dunk"
```

## Configuration

### Environment Variables

```bash
# Proxy configuration
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="https://proxy.example.com:8443"

# Custom user agent
export WEBTOON_USER_AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# Rate limiting (requests per second)
export WEBTOON_RATE_LIMIT="2.0"

# Download timeout (seconds)
export WEBTOON_TIMEOUT="30"
```

### JSON Configuration File

Create `~/.webtoon-vault/config.json`:

```json
{
  "output_directory": "~/manga-library",
  "format": "cbz",
  "parallel_chapters": 3,
  "naming_template": "{series}_ch{chapter:03d}",
  "retry_attempts": 5,
  "rate_limit": 2.0,
  "user_agents": [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
  ],
  "platforms": {
    "mori": {
      "max_parallel": 5,
      "delay": 0.5
    },
    "dumanwu": {
      "max_parallel": 3,
      "delay": 1.0
    }
  },
  "metadata": {
    "include_json": true,
    "languages": ["zh-CN", "en", "ja"]
  }
}
```

### Cookie Injection for Age-Gated Content

```bash
# Pass cookies for authenticated access
webtoon-vault download "https://mhpic.com/series/mature-content" \
  --cookies "session=abc123; user_id=456; adult_consent=true"

# Load cookies from file
webtoon-vault download "https://dumanwu.com/manga/restricted" \
  --cookie-file ./cookies.txt
```

## Python API Usage

If using Webtoon Vault as a Python library:

```python
import asyncio
from webtoon_vault import MangaDownloader, Config

# Basic usage
async def download_manga():
    config = Config(
        output_dir="./manga",
        format="cbz",
        parallel_chapters=3
    )
    
    downloader = MangaDownloader(config)
    
    # Download entire series
    result = await downloader.download_series(
        url="https://mhpic.com/series/one-piece",
        chapters=range(1, 100)
    )
    
    print(f"Downloaded {result.chapter_count} chapters")
    print(f"Total size: {result.total_size_mb:.2f} MB")
    print(f"Errors: {len(result.errors)}")

asyncio.run(download_manga())
```

### Custom Platform Parser

```python
from webtoon_vault.parsers import BaseParser
from webtoon_vault import register_parser

class CustomPlatformParser(BaseParser):
    """Parser for custom manga hosting platform"""
    
    PLATFORM_NAME = "custom_platform"
    URL_PATTERNS = [r"custommanga\.com", r"cm\.net"]
    
    async def get_chapter_list(self, series_url: str):
        """Extract chapter URLs from series page"""
        html = await self.fetch(series_url)
        chapters = []
        
        # Parse HTML to find chapter links
        soup = BeautifulSoup(html, 'html.parser')
        for link in soup.select('.chapter-link'):
            chapters.append({
                'url': link['href'],
                'title': link.text.strip(),
                'number': self.extract_chapter_number(link.text)
            })
        
        return chapters
    
    async def get_chapter_images(self, chapter_url: str):
        """Extract image URLs from chapter page"""
        html = await self.fetch(chapter_url)
        images = []
        
        # Handle lazy-loaded images
        soup = BeautifulSoup(html, 'html.parser')
        for img in soup.select('.manga-page img'):
            img_url = img.get('data-src') or img.get('src')
            if img_url:
                images.append(self.normalize_url(img_url, chapter_url))
        
        return images

# Register the custom parser
register_parser(CustomPlatformParser)
```

### Advanced Download Control

```python
from webtoon_vault import MangaDownloader, DownloadOptions
from webtoon_vault.filters import ChapterFilter

async def selective_download():
    options = DownloadOptions(
        output_dir="./manga",
        format="cbz",
        parallel_chapters=5,
        retry_attempts=3,
        rate_limit=2.0,
        user_agent_rotation=True
    )
    
    downloader = MangaDownloader(options)
    
    # Download with filters
    chapter_filter = ChapterFilter()
    chapter_filter.add_range(1, 50)
    chapter_filter.exclude([13, 14])  # Skip chapters
    chapter_filter.add_range(100, 110)
    
    # Progress callback
    def on_progress(chapter_num, page_num, total_pages):
        print(f"Chapter {chapter_num}: {page_num}/{total_pages}")
    
    result = await downloader.download_series(
        url="https://dumanwu.com/manga/my-series",
        chapter_filter=chapter_filter,
        progress_callback=on_progress
    )
    
    return result

# Run with error handling
result = asyncio.run(selective_download())
if result.errors:
    for error in result.errors:
        print(f"Error in chapter {error.chapter}: {error.message}")
```

### Metadata Extraction

```python
from webtoon_vault import MetadataExtractor

async def extract_series_info():
    extractor = MetadataExtractor()
    
    metadata = await extractor.get_series_metadata(
        url="https://mhpic.com/series/one-piece"
    )
    
    print(f"Title: {metadata.title}")
    print(f"Author: {metadata.author}")
    print(f"Genres: {', '.join(metadata.genres)}")
    print(f"Description: {metadata.description}")
    print(f"Total Chapters: {metadata.chapter_count}")
    print(f"Status: {metadata.status}")
    print(f"Language: {metadata.language}")
    
    # Save metadata to JSON
    metadata.save_json("./manga/one-piece/metadata.json")

asyncio.run(extract_series_info())
```

## Common Patterns

### Batch Download Multiple Series

```python
import asyncio
from webtoon_vault import MangaDownloader, Config

async def batch_download():
    config = Config(output_dir="./manga-library", format="cbz")
    downloader = MangaDownloader(config)
    
    series_list = [
        "https://mhpic.com/series/one-piece",
        "https://dumanwu.com/manga/naruto",
        "https://kankanmanhua.com/series/bleach",
        "https://gufengmh.com/manga/hunter-x-hunter"
    ]
    
    tasks = []
    for url in series_list:
        task = downloader.download_series(url, chapters=range(1, 10))
        tasks.append(task)
    
    # Download all series concurrently
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Failed to download {series_list[i]}: {result}")
        else:
            print(f"Downloaded {series_list[i]}: {result.chapter_count} chapters")

asyncio.run(batch_download())
```

### Resume Interrupted Downloads

```python
from webtoon_vault import MangaDownloader, SessionCache

async def resume_download():
    # Load cached session
    cache = SessionCache.load("./manga/one-piece/.cache")
    
    downloader = MangaDownloader()
    result = await downloader.resume_download(
        cache=cache,
        url="https://mhpic.com/series/one-piece"
    )
    
    print(f"Resumed from chapter {result.resume_chapter}")
    print(f"Downloaded {result.new_chapters} new chapters")

asyncio.run(resume_download())
```

### Custom CBZ Packaging

```python
from webtoon_vault import CBZArchiver, ImageProcessor

async def create_custom_cbz():
    archiver = CBZArchiver()
    processor = ImageProcessor()
    
    # Process images before packaging
    images = [
        "./manga/one-piece/ch001/page001.jpg",
        "./manga/one-piece/ch001/page002.jpg",
        "./manga/one-piece/ch001/page003.jpg"
    ]
    
    processed = []
    for img_path in images:
        # Optional: resize, compress, convert format
        processed_img = await processor.optimize(
            img_path,
            max_width=1200,
            quality=85,
            format="jpg"
        )
        processed.append(processed_img)
    
    # Create CBZ with metadata
    await archiver.create_cbz(
        output_path="./manga/one-piece/One_Piece_ch001.cbz",
        images=processed,
        metadata={
            "series": "One Piece",
            "chapter": 1,
            "title": "Romance Dawn",
            "author": "Eiichiro Oda",
            "date": "2026-01-15"
        }
    )

asyncio.run(create_custom_cbz())
```

## Troubleshooting

### Platform Detection Fails

```bash
# Force specific platform parser
webtoon-vault download "https://example.com/manga/series" --platform mori

# List available platforms
webtoon-vault platforms
```

### Rate Limiting / IP Blocking

```python
from webtoon_vault import MangaDownloader, Config

async def slow_download():
    # Increase delays to avoid detection
    config = Config(
        rate_limit=0.5,  # 0.5 requests per second
        retry_attempts=10,
        backoff_multiplier=2.0,
        user_agent_rotation=True
    )
    
    downloader = MangaDownloader(config)
    result = await downloader.download_series(
        url="https://mhpic.com/series/one-piece",
        parallel_chapters=1  # Single-threaded
    )
    
    return result
```

### Corrupted Images

```python
from webtoon_vault import ImageValidator

async def validate_downloads():
    validator = ImageValidator()
    
    corrupted = await validator.scan_directory("./manga/one-piece/ch001")
    
    if corrupted:
        print(f"Found {len(corrupted)} corrupted images:")
        for img_path in corrupted:
            print(f"  - {img_path}")
            # Re-download corrupted images
            await downloader.retry_image(img_path)
```

### Handling Anti-Scraping Measures

```python
from webtoon_vault import MangaDownloader, AntiScrapingConfig

async def bypass_protection():
    anti_scraping = AntiScrapingConfig(
        use_selenium=True,  # Render JavaScript
        headless=True,
        wait_time=3.0,  # Wait for lazy-loaded images
        cloudflare_bypass=True
    )
    
    config = Config(anti_scraping=anti_scraping)
    downloader = MangaDownloader(config)
    
    result = await downloader.download_series(
        url="https://protected-manga-site.com/series/my-manga"
    )
    
    return result
```

### Debug Extraction Failures

```bash
# Enable verbose logging
webtoon-vault download "https://mhpic.com/series/one-piece" --verbose --log-file debug.log

# Dump HTML for inspection
webtoon-vault debug "https://mhpic.com/series/one-piece" --save-html

# Test platform parser
webtoon-vault test-parser --platform mori --url "https://mhpic.com/series/one-piece"
```

### Check Download Integrity

```python
from webtoon_vault import ChecksumVerifier

async def verify_integrity():
    verifier = ChecksumVerifier()
    
    # Generate checksums during download
    config = Config(generate_checksums=True)
    downloader = MangaDownloader(config)
    
    await downloader.download_series("https://mhpic.com/series/one-piece")
    
    # Later: verify downloaded files
    results = await verifier.verify_directory("./manga/one-piece")
    
    if results.all_valid:
        print("All files verified successfully")
    else:
        print(f"Corrupted files: {len(results.corrupted)}")
        for file_path in results.corrupted:
            print(f"  - {file_path}")
```

## Legal and Ethical Usage

**Important Disclaimers:**
- Use only for personal, offline archival of content you have legitimate access to
- Respect platform terms of service and copyright laws
- Implement appropriate rate limiting to avoid server overload
- Do not redistribute downloaded content commercially
- Verify compliance with local laws regarding automated content retrieval

```python
# Example: Respectful download configuration
config = Config(
    rate_limit=1.0,  # Max 1 request per second
    parallel_chapters=2,  # Limited concurrency
    user_agent="WebtoonVault/1.0 (Personal Archival Use)",
    respect_robots_txt=True
)
```

This tool is intended for digital preservation and offline personal reading. Always verify that your usage complies with applicable terms of service and copyright regulations.
