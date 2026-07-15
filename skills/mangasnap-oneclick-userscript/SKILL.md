---
name: mangasnap-oneclick-userscript
description: Browser userscript for packaging manga chapters into CBZ/ZIP archives with one click
triggers:
  - how do I use the MangaSnap OneClick userscript
  - install LuminaCBZ manga downloader
  - configure mangasnap chapter archiver
  - customize CBZ export naming patterns
  - troubleshoot manga userscript issues
  - batch download manga chapters as cbz
  - set up tampermonkey manga script
  - archive manga chapters offline
---

# MangaSnap-OneClick Userscript Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

MangaSnap-OneClick (branded as LuminaCBZ) is a browser userscript that enables one-click packaging of manga chapters into CBZ or ZIP archives for offline reading. It detects chapter boundaries, fetches all pages, and assembles them into properly named archives entirely client-side.

## Installation

### Prerequisites

Install a userscript manager browser extension:
- **Tampermonkey** (Chrome, Firefox, Edge, Safari, Opera)
- **Violentmonkey** (Chrome, Firefox, Edge)
- **Greasemonkey** (Firefox)

### Installing the Script

1. Install a userscript manager extension
2. Visit the installation page: `https://morethanpaper.github.io/MangaSnap-OneClick/`
3. Click the install button when prompted by your userscript manager
4. Navigate to a supported manga chapter page (typically BigComics sites)
5. Look for the "Package Chapter" button overlay

## Core Features

- **One-click chapter archiving**: Single button to download complete chapters
- **Multi-format export**: ZIP or CBZ output
- **Intelligent naming**: Extracts series, chapter, volume metadata
- **Parallel page fetching**: Concurrent downloads with configurable parallelism
- **Client-side processing**: No external servers, all in-browser
- **Multi-language UI**: Auto-detects browser locale

## Configuration

### Basic Configuration Object

The script exposes a global configuration object that can be customized before page load:

```javascript
// Add this to Tampermonkey as a separate script, or inject via console
// Must execute BEFORE the main script loads
window.__LUMINA_CONFIG__ = {
  format: 'cbz',           // 'zip' or 'cbz'
  parallelism: 4,          // max concurrent page requests (1-8 recommended)
  naming: 'seriesFirst',   // 'seriesFirst' | 'numberFirst' | 'custom'
  customPattern: '{series}_Chapter_{chapter}_vol_{volume}.cbz'
};
```

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `format` | string | `'cbz'` | Archive format: `'zip'` or `'cbz'` |
| `parallelism` | number | `4` | Concurrent page downloads (1-8) |
| `naming` | string | `'seriesFirst'` | Naming convention for archives |
| `customPattern` | string | varies | Custom filename pattern template |

### Custom Naming Patterns

Available template variables:
- `{series}` - Series/manga title
- `{chapter}` - Chapter number
- `{volume}` - Volume number
- `{part}` - Part/section number
- `{year}` - Current year
- `{title}` - Chapter title (if available)

Example patterns:

```javascript
// Number first: "274_Vagabond_vol_12.cbz"
customPattern: '{chapter}_{series}_vol_{volume}.cbz'

// Verbose: "Vagabond - Chapter 274 (Volume 12) [2026].cbz"
customPattern: '{series} - Chapter {chapter} (Volume {volume}) [{year}].cbz'

// Simple: "Vagabond_274.cbz"
customPattern: '{series}_{chapter}.cbz'
```

## Usage Patterns

### Basic Single-Chapter Download

1. Navigate to a manga chapter page on a supported site
2. Wait for the "Package Chapter" button to appear (usually top-right overlay)
3. Click the button
4. Choose format (CBZ/ZIP) if prompted
5. Wait for progress bar to complete
6. Save the archive when browser download prompt appears

### Adjusting Download Speed

For slower connections, reduce parallelism to avoid timeouts:

```javascript
// In Tampermonkey dashboard, edit the script header section
// Or inject before navigation
window.__LUMINA_CONFIG__ = {
  parallelism: 2  // Reduce from default 4
};
```

For faster connections with stable bandwidth:

```javascript
window.__LUMINA_CONFIG__ = {
  parallelism: 6  // Increase for faster archiving
};
```

### Handling Failed Downloads

If a download fails or produces incomplete archives:

1. **Check parallelism**: Lower to 2-3 for unstable connections
2. **Verify page count**: Ensure all images loaded (scroll through chapter first)
3. **Clear browser cache**: Old cached images may interfere
4. **Disable other extensions**: Ad blockers or privacy tools may block requests
5. **Check console errors**: Open DevTools (F12) and check Console tab

```javascript
// Force sequential download (slowest but most reliable)
window.__LUMINA_CONFIG__ = {
  parallelism: 1
};
```

## Script Customization

### Modifying Archive Compression

The script uses client-side ZIP compression. To adjust compression level (requires editing the userscript source):

```javascript
// Find the compression function in the script
// Modify the compression level (0-9)
const zipOptions = {
  type: 'blob',
  compression: 'DEFLATE',
  compressionOptions: {
    level: 6  // Default: 6 (0=no compression, 9=max compression)
  }
};
```

### Adding Custom Metadata to CBZ

CBZ archives can include ComicInfo.xml metadata. To add custom metadata (requires script modification):

```javascript
// Add to the archive generation function
const comicInfoXml = `<?xml version="1.0"?>
<ComicInfo>
  <Title>${chapterTitle}</Title>
  <Series>${seriesName}</Series>
  <Number>${chapterNumber}</Number>
  <Volume>${volumeNumber}</Volume>
  <PageCount>${pageCount}</PageCount>
  <Year>${new Date().getFullYear()}</Year>
</ComicInfo>`;

// Add to ZIP before finalizing
zip.file('ComicInfo.xml', comicInfoXml);
```

### Extending to New Sites

To add support for additional manga hosting platforms (requires forking and modifying):

```javascript
// Add URL pattern matching
// @match        https://newsitedomain.com/*/chapter/*

// Add site-specific selectors
const siteConfigs = {
  'newsitedomain.com': {
    chapterTitleSelector: '.chapter-title',
    imageContainerSelector: '.manga-page img',
    pageCountSelector: '.page-indicator',
    nextPageSelector: '.next-page-link'
  }
};

// Implement detection logic
function detectCurrentSite() {
  const hostname = window.location.hostname;
  return Object.keys(siteConfigs).find(domain => hostname.includes(domain));
}
```

## Troubleshooting

### Button Not Appearing

**Symptoms**: No "Package Chapter" button visible on supported pages

**Solutions**:
1. Verify userscript manager is enabled and script is active
2. Check if page URL matches script's `@match` patterns
3. Refresh the page (Ctrl/Cmd + R)
4. Check console for JavaScript errors (F12 → Console)
5. Ensure no conflicting scripts are installed

```javascript
// Check if script loaded
console.log(window.__LUMINA_CONFIG__);
// Should output configuration object or undefined
```

### Incomplete Archives

**Symptoms**: CBZ/ZIP has fewer pages than expected

**Solutions**:
1. Scroll through entire chapter before downloading (triggers lazy-load)
2. Reduce parallelism to 2-3
3. Wait for all images to fully load (check network tab in DevTools)
4. Disable "data saver" browser features

```javascript
// Force pre-loading all images
window.__LUMINA_CONFIG__ = {
  parallelism: 1,  // Sequential loading
  preloadDelay: 2000  // Wait 2s between pages (if supported)
};
```

### Memory Issues on Long Chapters

**Symptoms**: Browser tab crashes or freezes during large chapter downloads

**Solutions**:
1. Close other tabs to free memory
2. Enable streaming mode (if available in script version)
3. Use desktop browser instead of mobile
4. Download chapters in smaller batches

```javascript
// Reduce memory footprint (if script supports)
window.__LUMINA_CONFIG__ = {
  streamingMode: true,  // Don't hold all pages in memory
  maxCacheSize: 50  // MB limit for cache
};
```

### Incorrect File Names

**Symptoms**: Archives saved with generic names like "download.zip"

**Solutions**:
1. Verify metadata extraction working (check console logs)
2. Use custom naming pattern
3. Manually rename after download

```javascript
// Fallback to simple naming
window.__LUMINA_CONFIG__ = {
  naming: 'custom',
  customPattern: 'Chapter_{chapter}.cbz'
};
```

## Advanced Usage

### Batch Processing Multiple Chapters

While the script is designed for single-chapter use, you can automate batch downloads:

```javascript
// Run in browser console on manga series page
const chapterLinks = document.querySelectorAll('.chapter-link');
const downloadDelay = 5000; // 5 seconds between chapters

chapterLinks.forEach((link, index) => {
  setTimeout(() => {
    window.location.href = link.href;
    // Script will auto-trigger on new page
    // Use browser auto-download settings to save without prompt
  }, index * downloadDelay);
});
```

### Integrating with Download Managers

Configure browser to auto-save to specific folder:

1. Set browser download behavior to "Ask where to save" = OFF
2. Set default download location: `~/Manga/{series_name}/`
3. Use custom naming pattern with series name:

```javascript
window.__LUMINA_CONFIG__ = {
  customPattern: '{series}/{series}_Ch{chapter}.cbz'
};
```

### Monitoring Download Progress

Access progress programmatically (if script exposes API):

```javascript
// Check if global API exists
if (window.LuminaCBZ) {
  window.LuminaCBZ.onProgress((current, total) => {
    console.log(`Downloaded ${current}/${total} pages`);
  });
  
  window.LuminaCBZ.onComplete((filename) => {
    console.log(`Archive saved: ${filename}`);
    // Trigger next action
  });
}
```

## Privacy & Security

- **All processing is client-side**: No data sent to external servers
- **No telemetry**: Script does not track usage or send analytics
- **Cookie inheritance**: Uses existing browser session (must be logged in)
- **No credential access**: Does not read passwords or authentication tokens
- **Temporary storage**: Blob URLs garbage-collected after download

## Environment Variables

The script runs entirely in-browser and does not use traditional environment variables. Configuration is done via JavaScript objects as shown above.

## Browser Compatibility

| Browser | Version | Status |
|---------|---------|--------|
| Chrome | 90+ | ✅ Fully supported |
| Firefox | 110+ | ✅ Fully supported |
| Edge | 90+ | ✅ Fully supported |
| Safari | 16+ | ⚠️ Requires Userscripts extension |
| Opera | 84+ | ✅ Fully supported |
| Brave | 1.45+ | ✅ Fully supported |

Mobile browsers: Limited support (use Kiwi Browser or Firefox Nightly on Android)

## Common Use Cases

### Archiving Before Series Removal
```javascript
// Use batch approach to save all chapters before delisting
// Set high-quality settings
window.__LUMINA_CONFIG__ = {
  format: 'cbz',
  parallelism: 3,
  customPattern: '{series}_Ch{chapter}_[ARCHIVE].cbz'
};
```

### Reading on E-Readers
```javascript
// Optimize for e-ink devices (prefer CBZ for reader compatibility)
window.__LUMINA_CONFIG__ = {
  format: 'cbz',
  naming: 'seriesFirst'
};
```

### Offline Backup
```javascript
// Include metadata for organization
window.__LUMINA_CONFIG__ = {
  format: 'cbz',
  customPattern: '{year}-{series}_v{volume}_ch{chapter}.cbz'
};
```
