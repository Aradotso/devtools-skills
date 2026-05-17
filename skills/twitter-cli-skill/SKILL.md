---
name: twitter-cli-skill
description: Terminal-first Twitter/X CLI for reading feeds, bookmarks, search, and posting tweets without API keys
triggers:
  - "show my Twitter timeline"
  - "search tweets about"
  - "post a tweet from terminal"
  - "fetch Twitter bookmarks"
  - "get user tweets from CLI"
  - "read Twitter feed in terminal"
  - "export tweets as JSON"
  - "view tweet replies"
---

# twitter-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`twitter-cli` is a Python-based terminal CLI for Twitter/X that works without API keys. It uses browser cookie authentication to read timelines (For You, Following), bookmarks, search results, user profiles, and post/delete tweets. All commands support structured output (`--json`, `--yaml`) for AI agent pipelines.

**Key capabilities:**
- Read: feed, bookmarks, search, tweet detail, article, list timeline, user lookup
- Write: post, reply, quote, delete, like, retweet, bookmark
- Auth: browser cookie extraction (Arc/Chrome/Edge/Firefox/Brave) or env vars
- Anti-detection: TLS fingerprinting, full cookie forwarding, request jitter
- Output: rich tables (default), JSON, YAML, Markdown (articles)

## Installation

```bash
# Recommended: uv tool (fast, isolated)
uv tool install twitter-cli

# Alternative: pipx
pipx install twitter-cli

# Upgrade to latest
uv tool upgrade twitter-cli
```

**From source:**

```bash
git clone git@github.com:jackwener/twitter-cli.git
cd twitter-cli
uv sync
```

## Authentication

Priority order:
1. Environment variables: `TWITTER_AUTH_TOKEN` + `TWITTER_CT0`
2. Browser cookies (auto-extracted from Arc/Chrome/Edge/Firefox/Brave)

**Browser cookie extraction (recommended):**

```bash
# Auto-detect browser and extract all Twitter cookies
twitter feed

# Specify browser explicitly
TWITTER_BROWSER=chrome twitter feed

# Chrome multi-profile
TWITTER_CHROME_PROFILE="Profile 2" twitter feed
```

**Manual environment variables:**

```bash
export TWITTER_AUTH_TOKEN="your_auth_token"
export TWITTER_CT0="your_ct0_token"
twitter feed
```

**Proxy support:**

```bash
export TWITTER_PROXY=http://127.0.0.1:7890
# or
export TWITTER_PROXY=socks5://127.0.0.1:1080
```

## Core Commands

### Read Timeline

```bash
# For You feed (default)
twitter feed

# Following feed
twitter feed -t following

# Limit results
twitter feed --max 20

# Full text (no truncation)
twitter feed --full-text

# Structured output for agents
twitter feed --json
twitter feed --yaml

# With ranking filter
twitter feed --filter --max 50

# Pagination cursor
twitter feed --cursor "DAABCgABGdE..."
```

### Search

```bash
# Basic search
twitter search "AI agent"

# Search type: Top, Latest, Photos, Videos
twitter search "machine learning" -t Latest

# Full text + JSON output
twitter search "python" --full-text --json

# Advanced filters
twitter search "climate" --from scientist --lang en --since 2026-01-01
twitter search "news" --exclude retweets --has links

# Save to file
twitter search "trending" -o results.json

# Apply ranking filter
twitter search "tech" --filter
```

### Bookmarks

```bash
# List bookmarks
twitter bookmarks

# Limit + full text
twitter bookmarks --max 30 --full-text

# Export as YAML
twitter bookmarks --yaml
```

### Tweet Detail (tweet + replies)

```bash
# By tweet ID
twitter tweet 1234567890

# By URL
twitter tweet https://x.com/user/status/1234567890

# Full text in reply table
twitter tweet 1234567890 --full-text

# Structured output
twitter tweet 1234567890 --json
```

### Show Command (open tweet by index)

```bash
# After running `twitter feed` or `twitter search`:
twitter show 2                    # Open tweet #2 from last list
twitter show 2 --full-text
twitter show 2 --json
```

### Twitter Article (fetch and export as Markdown)

```bash
# By article ID
twitter article 1234567890

# By URL
twitter article https://x.com/user/article/1234567890

# Export as Markdown
twitter article 1234567890 --markdown
twitter article 1234567890 --output article.md

# JSON output
twitter article 1234567890 --json
```

### List Timeline

```bash
# Fetch tweets from a Twitter List
twitter list 1539453138322673664

# Pagination
twitter list 1539453138322673664 --cursor "DAABCgABGdE..."

# Full text
twitter list 1539453138322673664 --full-text
```

### User Lookup

```bash
# Profile
twitter user elonmusk

# User tweets
twitter user-posts elonmusk --max 20
twitter user-posts elonmusk --full-text -o tweets.json

# User likes (own account only, private since Jun 2024)
twitter likes elonmusk --max 30

# Followers
twitter followers elonmusk --max 50

# Following
twitter following elonmusk --max 50
```

## Write Operations

### Post Tweet

```bash
# Simple tweet
twitter post "Hello from twitter-cli!"

# Tweet with image (up to 4)
twitter post "Hello!" --image photo.jpg
twitter post "Gallery" -i a.png -i b.jpg -i c.webp

# Structured output
twitter post "Hello from twitter-cli!" --json
```

### Reply

```bash
# Reply to tweet
twitter post "Nice!" --reply-to 1234567890

# Reply with image
twitter reply 1234567890 "Great!" -i screenshot.png
```

### Quote Tweet

```bash
# Quote with text
twitter quote 1234567890 "Check this out"

# Quote with image
twitter quote 1234567890 "Look at this chart" -i chart.png
```

### Delete Tweet

```bash
twitter delete 1234567890
```

### Like / Unlike

```bash
twitter like 1234567890
twitter like 1234567890 --yaml
twitter unlike 1234567890
```

### Retweet / Unretweet

```bash
twitter retweet 1234567890
twitter unretweet 1234567890
```

### Bookmark / Unbookmark

```bash
twitter bookmark 1234567890
twitter unbookmark 1234567890
```

### Follow / Unfollow

```bash
twitter follow elonmusk --json
twitter unfollow elonmusk
```

## Configuration

Create `config.yaml` in your working directory:

```yaml
fetch:
  count: 50

filter:
  mode: "topN"          # "topN" | "score" | "all"
  topN: 20
  minScore: 50
  lang: []
  excludeRetweets: false
  weights:
    likes: 1.0
    retweets: 3.0
    replies: 2.0
    bookmarks: 5.0
    views_log: 0.5

rateLimit:
  requestDelay: 2.5     # base delay (randomized ×0.7–1.5)
  maxRetries: 3
  retryBaseDelay: 5.0
  maxCount: 200
```

**Scoring formula:**

```text
score = likes_w * likes
      + retweets_w * retweets
      + replies_w * replies
      + bookmarks_w * bookmarks
      + views_log_w * log10(max(views, 1))
```

**Filter modes:**
- `topN`: Keep highest N tweets by score
- `score`: Keep tweets where score >= minScore
- `all`: Return all tweets sorted by score

## AI Agent Usage

All commands support `--json` or `--yaml` for structured output. Non-TTY stdout defaults to YAML.

**Example: Fetch and parse timeline**

```python
import subprocess
import yaml

result = subprocess.run(
    ["twitter", "feed", "--max", "10", "--yaml"],
    capture_output=True,
    text=True,
    check=True
)
data = yaml.safe_load(result.stdout)
for tweet in data["tweets"]:
    print(f"{tweet['author']['username']}: {tweet['text']}")
```

**Example: Post tweet from Python**

```python
import subprocess
import json

result = subprocess.run(
    ["twitter", "post", "Hello from Python!", "--json"],
    capture_output=True,
    text=True,
    check=True
)
response = json.loads(result.stdout)
print(f"Tweet ID: {response['tweetId']}")
```

**Example: Search and filter**

```bash
# Get top 10 AI tweets from today as JSON
twitter search "AI agent" --since 2026-05-17 --filter --max 10 --json
```

**Structured output contract:** See [SCHEMA.md](https://github.com/jackwener/twitter-cli/blob/main/SCHEMA.md)

## Common Patterns

### Export bookmarks to file

```bash
twitter bookmarks --max 100 --json > bookmarks.json
```

### Monitor user tweets

```bash
# Fetch latest 5 tweets from user
twitter user-posts elonmusk --max 5 --yaml
```

### Search and rank by engagement

```bash
# Top 10 tweets about "AI" by score
twitter search "AI" --filter --max 10 --full-text
```

### Post with multiple images

```bash
twitter post "My photo gallery" -i img1.jpg -i img2.png -i img3.webp
```

### Reply to tweet with image

```bash
twitter reply 1234567890 "Here's the screenshot" -i screenshot.png
```

### Extract article as Markdown

```bash
twitter article 1234567890 --markdown --output ~/articles/ai-safety.md
```

### Use proxy for all requests

```bash
export TWITTER_PROXY=http://127.0.0.1:7890
twitter feed --max 20
```

## Troubleshooting

### No Twitter cookies found

**Solution:**
1. Log in to `x.com` in a supported browser (Arc/Chrome/Edge/Firefox/Brave)
2. Or set `TWITTER_AUTH_TOKEN` and `TWITTER_CT0` manually
3. Run with `-v` for diagnostics:
   ```bash
   twitter -v feed
   ```

### Cookie expired (HTTP 401/403)

**Solution:** Re-login to `x.com` and retry.

### Keychain access denied (macOS)

**SSH sessions:**
```bash
security unlock-keychain ~/Library/Keychains/login.keychain-db
```

**Local terminal:**
- Open **Keychain Access** → search for "\<Browser\> Safe Storage"
- **Access Control** → add Terminal app → **Save Changes**
- Or click **"Always Allow"** when prompted

### Twitter API error 404

**Solution:** Retry command. The client attempts a live queryId fallback when GraphQL IDs rotate.

### Invalid tweet JSON file

**Solution:** Regenerate using:
```bash
twitter feed --json > tweets.json
```

### Windows: no pipe output in AI agent subprocess

**Issue:** ConPTY intercepts pipe output from network commands.

**Fix:**
1. Use **Git Bash** as terminal shell
2. Set `"windowsEnableConpty": false` in terminal settings
3. Standard `subprocess.run(capture_output=True)` works correctly

### Rate limiting (HTTP 429)

**Solution:**
- Use `--max` to limit requests
- Set `TWITTER_PROXY` to avoid direct IP exposure
- Increase `rateLimit.requestDelay` in `config.yaml`
- Use residential proxies instead of datacenter IPs

## Best Practices

1. **Use a proxy** — set `TWITTER_PROXY` to avoid direct IP exposure
2. **Keep request volumes low** — prefer `--max 20` over `--max 500`
3. **Don't run too frequently** — each startup fetches x.com to initialize headers
4. **Use browser cookie extraction** — provides full cookie fingerprint
5. **Avoid datacenter IPs** — residential proxies are safer
6. **Upgrade regularly** — `uv tool upgrade twitter-cli` to avoid outdated API handling

## Output Modes

- **Rich table** (default): Interactive reading in terminal
- `--full-text`: Show full tweet text in tables
- `--yaml`: Structured output for agents (default for non-TTY)
- `--json`: Strict JSON for parsers
- `-c` / `--compact`: Token-efficient compact output
- `-o FILE`: Save to file

## Development

```bash
# Install dev dependencies
uv sync --extra dev

# Lint
uv run ruff check .

# Tests
uv run pytest -q
```

## Project Structure

```text
twitter_cli/
├── cli.py           # Command-line interface
├── client.py        # HTTP client + anti-detection
├── graphql.py       # GraphQL query IDs, URL building
├── parser.py        # Tweet/User/Media parsing
├── auth.py          # Cookie extraction
├── config.py        # Configuration loading
├── filter.py        # Ranking/scoring logic
├── formatter.py     # Rich table formatting
├── output.py        # JSON/YAML serialization
└── models.py        # Data models (Tweet, User, Media)
```

## Example Workflows

### Daily AI news digest

```bash
# Search AI tweets from today, rank by engagement, export as JSON
twitter search "AI" --since 2026-05-17 --filter --max 20 --json > ai_digest.json
```

### Bookmark analysis

```python
import subprocess
import yaml
from collections import Counter

result = subprocess.run(
    ["twitter", "bookmarks", "--max", "200", "--yaml"],
    capture_output=True, text=True, check=True
)
data = yaml.safe_load(result.stdout)

# Extract hashtags
hashtags = []
for tweet in data["tweets"]:
    hashtags.extend([h["text"] for h in tweet.get("hashtags", [])])

print(Counter(hashtags).most_common(10))
```

### Post with automated image

```python
import subprocess

# Generate chart, then post with image
subprocess.run(["python", "generate_chart.py"], check=True)
subprocess.run([
    "twitter", "post",
    "Daily metrics update",
    "-i", "chart.png",
    "--json"
], check=True)
```

### Monitor user and alert on keyword

```bash
#!/bin/bash
# Cron job: check user tweets for keyword
tweets=$(twitter user-posts elonmusk --max 5 --yaml)
if echo "$tweets" | grep -q "Mars"; then
  echo "Alert: New Mars tweet from @elonmusk"
fi
```

## Related Projects

- [xiaohongshu-cli](https://github.com/jackwener/xiaohongshu-cli) — Xiaohongshu (小红书) CLI
- [bilibili-cli](https://github.com/jackwener/bilibili-cli) — Bilibili CLI
- [discord-cli](https://github.com/jackwener/discord-cli) — Discord CLI
- [tg-cli](https://github.com/jackwener/tg-cli) — Telegram CLI

---

**License:** Apache-2.0  
**Repository:** https://github.com/jackwener/twitter-cli  
**PyPI:** https://pypi.org/project/twitter-cli/
