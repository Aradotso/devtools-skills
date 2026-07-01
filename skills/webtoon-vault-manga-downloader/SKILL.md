---
name: webtoon-vault-manga-downloader
description: CLI tool for batch downloading manga and webcomics from multiple hosting platforms with intelligent parsing and CBZ packaging
triggers:
  - how do I download manga from the command line
  - batch download webcomics to CBZ archives
  - use webtoon vault to archive manga chapters
  - download entire manga series offline
  - scrape manga from hosting platforms
  - create CBZ comic archives from web sources
  - download manga with chapter metadata
  - automate manga library archiving
---

# Webtoon Vault Manga Downloader

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Webtoon Vault is a Python-based CLI tool for downloading manga and webcomics from multiple hosting platforms. It features intelligent source detection, parallel chapter harvesting, automatic CBZ archive creation, and multi-platform parser support. The tool is designed for digital archivists and offline readers who need reliable batch manga downloads.

## Installation

The project appears to use Python 3.9+ as indicated in the documentation. Based on typical Python CLI tool patterns:

```bash
# Clone the repository
git clone https://github.com/BunaTechnologyBackUP/manga-pull-cli.git
cd manga-pull-cli

# Install dependencies (typical pattern)
pip install -r requirements.txt

# Or install as package
pip install -e .
```

**Environment Variables:**
```bash
# Optional proxy configuration
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="https://proxy.example.com:8080"

# For debugging
export WEBTOON_VAULT_DEBUG=1
```

## Core Usage Patterns

### Basic Download Command

```python
# Typical CLI invocation pattern
# python manga_pull.py [URL] [OPTIONS]

# Download a single series
python manga_pull.py "https://mhpic.com/series/jujutsu-kaisen"

# Download specific chapters
python manga_pull.py "https://mhpic.com/series/one-piece" --chapters 1-10

# Download with custom output directory
python manga_pull.py "https://dumanwu.com/series/solo-leveling" --output ./manga_library
```

### Advanced Options

```python
# Dry run mode (preview without downloading)
python manga_pull.py "URL" --dry-run

# Verbose logging for debugging
python manga_pull.py "URL" --verbose

# Custom naming template
python manga_pull.py "URL" --name-template "{series}_v{volume}_ch{chapter}"

# Download as loose images instead of CBZ
python manga_pull.py "URL" --format images

# Set parallel download threads
python manga_pull.py "URL" --threads 4

# Resume interrupted download
python manga_pull.py "URL" --resume
```

## Supported Platforms

The tool automatically detects the platform from URL patterns:

- **Mori Manga**: `mhpic.com`, `mori.*`
- **DuManWu**: `dumanwu.*`
- **KanKanManHua**: `kankanmanhua.*`
- **ManhuaDB**: `manhuadb.*` (Beta)
- **GuFeng**: `gufengmh.*`

## Configuration File

Create a `config.json` for advanced configuration:

```json
{
  "output_directory": "./manga_library",
  "default_format": "cbz",
  "naming_template": "{series}_ch{chapter}_{date}",
  "parallel_downloads": 3,
  "retry_attempts": 5,
  "rate_limit_delay": 1.5,
  "user_agents": [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
  ],
  "proxy": {
    "http": null,
    "https": null
  },
  "cookies": {},
  "verify_checksums": true
}
```

Load configuration:
```python
python manga_pull.py "URL" --config config.json
```

## Python API Usage

If using as a library in your own Python scripts:

```python
from webtoon_vault import MangaDownloader, SourceDetector

# Initialize downloader
downloader = MangaDownloader(
    output_dir="./downloads",
    format="cbz",
    threads=4,
    verbose=True
)

# Detect platform and download
url = "https://mhpic.com/series/chainsaw-man"
source = SourceDetector.detect(url)

# Download entire series
downloader.download_series(url, source=source)

# Download specific chapters
downloader.download_chapters(
    url=url,
    source=source,
    chapter_range=(1, 50)
)
```

### Custom Parser Module

Extending support for new platforms:

```python
from webtoon_vault.parsers import BaseParser
import requests
from bs4 import BeautifulSoup

class CustomSiteParser(BaseParser):
    """Parser for custom manga hosting platform"""
    
    @staticmethod
    def can_handle(url: str) -> bool:
        return "customsite.com" in url
    
    def get_series_info(self, url: str) -> dict:
        """Extract series metadata"""
        response = requests.get(url)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        return {
            'title': soup.select_one('.series-title').text,
            'author': soup.select_one('.author').text,
            'chapters': self._extract_chapters(soup)
        }
    
    def _extract_chapters(self, soup) -> list:
        """Extract chapter URLs and titles"""
        chapters = []
        for link in soup.select('.chapter-link'):
            chapters.append({
                'url': link['href'],
                'title': link.text.strip(),
                'number': self._parse_chapter_number(link.text)
            })
        return chapters
    
    def get_chapter_images(self, chapter_url: str) -> list:
        """Extract all image URLs from a chapter"""
        response = requests.get(chapter_url)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        images = []
        for img in soup.select('.manga-page img'):
            images.append(img.get('data-src') or img.get('src'))
        
        return images

# Register custom parser
from webtoon_vault.registry import register_parser
register_parser(CustomSiteParser)
```

### Async Download Pattern

```python
import asyncio
from webtoon_vault import AsyncMangaDownloader

async def download_multiple_series():
    """Download multiple series concurrently"""
    downloader = AsyncMangaDownloader()
    
    series_urls = [
        "https://mhpic.com/series/one-piece",
        "https://dumanwu.com/series/naruto",
        "https://kankanmanhua.com/series/bleach"
    ]
    
    tasks = [
        downloader.download_series(url)
        for url in series_urls
    ]
    
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    for url, result in zip(series_urls, results):
        if isinstance(result, Exception):
            print(f"Failed to download {url}: {result}")
        else:
            print(f"Successfully downloaded {url}")

# Run async downloads
asyncio.run(download_multiple_series())
```

## CBZ Archive Format

The tool creates CBZ archives with the following structure:

```
One_Piece_ch001.cbz
├── 001.jpg
├── 002.jpg
├── 003.jpg
├── ...
└── metadata.json
```

**metadata.json structure:**
```json
{
  "series": "One Piece",
  "chapter": 1,
  "title": "Romance Dawn",
  "language": "en",
  "source": "mhpic.com",
  "download_date": "2026-07-01T12:34:56Z",
  "pages": 45,
  "checksum": "a1b2c3d4e5f6..."
}
```

## Common Workflows

### Archive Entire Series

```bash
# Download all chapters of a series
python manga_pull.py "https://mhpic.com/series/attack-on-titan" \
  --output ./library/Attack_on_Titan \
  --format cbz \
  --threads 3 \
  --name-template "AoT_ch{chapter:03d}"
```

### Selective Chapter Download

```python
from webtoon_vault import MangaDownloader

downloader = MangaDownloader(output_dir="./downloads")

# Download specific chapters
downloader.download_chapters(
    url="https://dumanwu.com/series/solo-leveling",
    chapter_range=(100, 150)
)

# Download latest chapters only
downloader.download_latest(
    url="https://mhpic.com/series/one-piece",
    count=10
)
```

### Batch Processing with List

```python
# batch_download.py
import json
from webtoon_vault import MangaDownloader

# Load series list from JSON
with open('series_list.json', 'r') as f:
    series_list = json.load(f)

downloader = MangaDownloader(
    output_dir="./manga_archive",
    format="cbz",
    threads=2
)

for series in series_list:
    try:
        print(f"Downloading: {series['name']}")
        downloader.download_series(
            url=series['url'],
            chapter_range=series.get('chapter_range')
        )
    except Exception as e:
        print(f"Error downloading {series['name']}: {e}")
        continue
```

**series_list.json:**
```json
[
  {
    "name": "One Piece",
    "url": "https://mhpic.com/series/one-piece",
    "chapter_range": [1, 1000]
  },
  {
    "name": "Naruto",
    "url": "https://dumanwu.com/series/naruto"
  }
]
```

## Error Handling

```python
from webtoon_vault import (
    MangaDownloader,
    DownloadError,
    ParserError,
    RateLimitError
)

downloader = MangaDownloader()

try:
    downloader.download_series(url)
except RateLimitError as e:
    print(f"Rate limited. Retry after: {e.retry_after} seconds")
    time.sleep(e.retry_after)
    downloader.download_series(url)
except ParserError as e:
    print(f"Failed to parse page: {e.message}")
    # Try alternative source or manual intervention
except DownloadError as e:
    print(f"Download failed: {e}")
    # Check network, retry with different settings
```

## Troubleshooting

### Download Fails with 403 Forbidden

```python
# Add custom headers or rotate user agents
downloader = MangaDownloader(
    user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
)

# Or use cookies for authenticated access
downloader.set_cookies({
    'session': 'YOUR_SESSION_COOKIE',
    'auth_token': 'YOUR_AUTH_TOKEN'
})
```

### Rate Limiting Issues

```python
# Increase delay between requests
downloader = MangaDownloader(
    rate_limit_delay=3.0,  # seconds between requests
    retry_attempts=10
)
```

### Images Not Loading (Lazy Load)

```python
# Some parsers may need JavaScript execution
from webtoon_vault.parsers import SeleniumParser

# Use Selenium-based parser for JS-heavy sites
parser = SeleniumParser(
    headless=True,
    wait_for_images=True
)
downloader.use_parser(parser)
```

### Resume Interrupted Downloads

```bash
# The tool maintains a cache of progress
python manga_pull.py "URL" --resume

# Or force restart
python manga_pull.py "URL" --no-cache
```

### Check Downloaded Files

```python
from webtoon_vault.utils import verify_cbz

# Verify CBZ integrity
is_valid = verify_cbz("./downloads/One_Piece_ch001.cbz")

if not is_valid:
    print("Archive is corrupted, re-download needed")
```

## Integration Examples

### With Calibre Library

```python
import subprocess
from webtoon_vault import MangaDownloader

downloader = MangaDownloader(output_dir="./temp_downloads")
cbz_files = downloader.download_series(url)

# Add to Calibre library
for cbz in cbz_files:
    subprocess.run([
        "calibredb", "add",
        cbz,
        "--library-path", "/path/to/calibre/library"
    ])
```

### With Discord Webhook Notifications

```python
import requests
from webtoon_vault import MangaDownloader

DISCORD_WEBHOOK = os.getenv("DISCORD_WEBHOOK_URL")

def notify_download(series, chapters):
    requests.post(DISCORD_WEBHOOK, json={
        "content": f"Downloaded {len(chapters)} chapters of {series}"
    })

downloader = MangaDownloader()
results = downloader.download_series(url)
notify_download(results['series'], results['chapters'])
```

## Performance Optimization

```python
# Optimize for speed
downloader = MangaDownloader(
    threads=8,  # Increase parallel downloads
    rate_limit_delay=0.5,  # Reduce delay if server allows
    compression_level=1,  # Faster CBZ creation (larger files)
    skip_checksum=True  # Skip verification for speed
)

# Optimize for quality/safety
downloader = MangaDownloader(
    threads=2,
    rate_limit_delay=2.0,
    compression_level=9,  # Maximum compression
    verify_checksums=True,
    retry_attempts=10
)
```

## Legal and Ethical Usage

Always ensure your usage complies with platform terms of service. This tool is intended for personal archival purposes only:

```python
# Example: Respect robots.txt
from webtoon_vault.utils import check_robots_txt

url = "https://example.com/manga/series"
if not check_robots_txt(url):
    print("Platform disallows automated access")
    exit(1)

downloader.download_series(url)
```

Use environment variables for any authentication tokens and never commit them to version control.
