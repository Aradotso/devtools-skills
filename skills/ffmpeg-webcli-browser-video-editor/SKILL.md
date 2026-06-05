---
name: ffmpeg-webcli-browser-video-editor
description: A browser-based video editor powered by ffmpeg.wasm for client-side video processing without server uploads
triggers:
  - how do I use ffmpeg webCLI to process videos in the browser
  - set up ffmpeg.wasm video editor in my web app
  - convert videos to GIF using ffmpeg webCLI
  - process videos client-side with WebAssembly
  - integrate browser-based video editing with ffmpeg
  - use ffmpeg webCLI API for video operations
  - implement offline video processing in the browser
  - create video editor with ffmpeg.wasm
---

# ffmpeg webCLI Browser Video Editor

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

ffmpeg webCLI is a browser-based video editor powered by ffmpeg.wasm that processes videos entirely client-side using WebAssembly. No server uploads, no backend infrastructure — all video processing happens locally in the user's browser. It supports 30+ video operations including GIF creation, format conversion, compression, trimming, filters, effects, and more.

## Installation

### Option 1: Use the Live App

Access the hosted version directly:
```
https://tejaswigowda.com/ffmpeg-webCLI/
```

### Option 2: Clone and Run Locally

```bash
git clone https://github.com/tejaswigowda/ffmpeg-webCLI.git
cd ffmpeg-webCLI
# Serve the files with any static server
python -m http.server 8000
# or
npx serve .
```

### Option 3: Integrate into Your Project

The project uses ffmpeg.wasm as its core dependency. To integrate similar functionality:

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://unpkg.com/@ffmpeg/ffmpeg@0.12.7/dist/umd/ffmpeg.js"></script>
</head>
<body>
    <input type="file" id="videoInput" accept="video/*">
    <button id="processBtn">Process Video</button>
    <video id="output" controls></video>
    
    <script src="app.js"></script>
</body>
</html>
```

## Core Architecture

ffmpeg webCLI uses:
- **ffmpeg.wasm** — WebAssembly port of ffmpeg for browser execution
- **Web Workers** — Background processing to keep UI responsive
- **PWA** — Progressive Web App for offline support
- **Wake Lock API** — Prevents screen sleep during processing

## Key Components

### 1. Initialize ffmpeg.wasm

```javascript
const { FFmpeg } = FFmpegWASM;
const { fetchFile } = FFmpegWASM;

let ffmpeg = null;
let loaded = false;

async function loadFFmpeg() {
    if (loaded) return;
    
    ffmpeg = new FFmpeg();
    
    // Log ffmpeg output
    ffmpeg.on('log', ({ message }) => {
        console.log(message);
    });
    
    // Track progress
    ffmpeg.on('progress', ({ progress, time }) => {
        console.log(`Progress: ${Math.round(progress * 100)}%`);
        updateProgressBar(progress);
    });
    
    // Load the core and wasm files
    await ffmpeg.load({
        coreURL: 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/umd/ffmpeg-core.js',
        wasmURL: 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/umd/ffmpeg-core.wasm',
    });
    
    loaded = true;
    console.log('FFmpeg loaded successfully');
}
```

### 2. Load Video File into Virtual Filesystem

```javascript
async function loadVideoFile(file) {
    await loadFFmpeg();
    
    // Write file to ffmpeg's virtual filesystem
    await ffmpeg.writeFile('input.mp4', await fetchFile(file));
    console.log('Video loaded into virtual filesystem');
}
```

### 3. Convert Video to GIF

```javascript
async function convertToGIF(inputFile, width = 480, fps = 10) {
    await loadVideoFile(inputFile);
    
    // Two-pass palette generation for best quality
    // Pass 1: Generate palette
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-vf', `fps=${fps},scale=${width}:-1:flags=lanczos,palettegen`,
        'palette.png'
    ]);
    
    // Pass 2: Use palette to create GIF
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-i', 'palette.png',
        '-filter_complex', `fps=${fps},scale=${width}:-1:flags=lanczos[x];[x][1:v]paletteuse`,
        'output.gif'
    ]);
    
    // Read the output file
    const data = await ffmpeg.readFile('output.gif');
    return new Blob([data.buffer], { type: 'image/gif' });
}
```

### 4. Convert Video Format

```javascript
async function convertFormat(inputFile, outputFormat = 'mp4') {
    await loadVideoFile(inputFile);
    
    const outputFile = `output.${outputFormat}`;
    const formatConfigs = {
        mp4: ['-c:v', 'libx264', '-c:a', 'aac'],
        webm: ['-c:v', 'libvpx-vp9', '-c:a', 'libopus'],
        mkv: ['-c:v', 'libx264', '-c:a', 'aac'],
        mov: ['-c:v', 'libx264', '-c:a', 'aac'],
        avi: ['-c:v', 'libx264', '-c:a', 'aac']
    };
    
    await ffmpeg.exec([
        '-i', 'input.mp4',
        ...formatConfigs[outputFormat],
        outputFile
    ]);
    
    const data = await ffmpeg.readFile(outputFile);
    return new Blob([data.buffer], { type: `video/${outputFormat}` });
}
```

### 5. Compress Video with CRF

```javascript
async function compressVideo(inputFile, crf = 23, preset = 'medium') {
    await loadVideoFile(inputFile);
    
    // CRF: 18 (near lossless) to 51 (maximum compression)
    // Preset: ultrafast, superfast, veryfast, faster, fast, medium, slow, slower, veryslow
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-c:v', 'libx264',
        '-crf', crf.toString(),
        '-preset', preset,
        '-c:a', 'aac',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 6. Trim Video

```javascript
async function trimVideo(inputFile, startTime, endTime) {
    await loadVideoFile(inputFile);
    
    const duration = endTime - startTime;
    
    await ffmpeg.exec([
        '-ss', startTime.toString(),
        '-i', 'input.mp4',
        '-t', duration.toString(),
        '-c', 'copy', // Stream copy for fast, lossless trim
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 7. Extract Audio

```javascript
async function extractAudio(inputFile, format = 'mp3') {
    await loadVideoFile(inputFile);
    
    const outputFile = `output.${format}`;
    const audioConfigs = {
        mp3: ['-c:a', 'libmp3lame', '-q:a', '2'],
        aac: ['-c:a', 'aac', '-b:a', '192k'],
        wav: ['-c:a', 'pcm_s16le'],
        ogg: ['-c:a', 'libvorbis', '-q:a', '5'],
        flac: ['-c:a', 'flac']
    };
    
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-vn', // No video
        ...audioConfigs[format],
        outputFile
    ]);
    
    const data = await ffmpeg.readFile(outputFile);
    return new Blob([data.buffer], { type: `audio/${format}` });
}
```

### 8. Resize Video

```javascript
async function resizeVideo(inputFile, width, height = -1) {
    await loadVideoFile(inputFile);
    
    // height = -1 maintains aspect ratio
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-vf', `scale=${width}:${height}`,
        '-c:v', 'libx264',
        '-crf', '23',
        '-c:a', 'copy',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 9. Change Video Speed

```javascript
async function changeSpeed(inputFile, speedMultiplier) {
    await loadVideoFile(inputFile);
    
    const videoPTS = 1 / speedMultiplier;
    
    // Build atempo filter chain (each atempo can only do 0.5-2.0x)
    let atempoChain = '';
    let remaining = speedMultiplier;
    
    while (remaining > 2.0) {
        atempoChain += 'atempo=2.0,';
        remaining /= 2.0;
    }
    while (remaining < 0.5) {
        atempoChain += 'atempo=0.5,';
        remaining /= 0.5;
    }
    atempoChain += `atempo=${remaining.toFixed(3)}`;
    
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-filter:v', `setpts=${videoPTS}*PTS`,
        '-filter:a', atempoChain,
        '-c:v', 'libx264',
        '-crf', '23',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 10. Rotate Video

```javascript
async function rotateVideo(inputFile, rotation) {
    await loadVideoFile(inputFile);
    
    const rotations = {
        '90cw': 'transpose=1',
        '90ccw': 'transpose=2',
        '180': 'transpose=1,transpose=1',
        'hflip': 'hflip',
        'vflip': 'vflip',
        'both': 'hflip,vflip'
    };
    
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-vf', rotations[rotation],
        '-c:a', 'copy',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 11. Add Watermark/Logo

```javascript
async function addWatermark(inputFile, logoFile, position = 'bottom-right', widthPercent = 15) {
    await loadVideoFile(inputFile);
    await ffmpeg.writeFile('logo.png', await fetchFile(logoFile));
    
    const positions = {
        'top-left': '10:10',
        'top-right': 'W-w-10:10',
        'bottom-left': '10:H-h-10',
        'bottom-right': 'W-w-10:H-h-10',
        'center': '(W-w)/2:(H-h)/2'
    };
    
    const scaleFilter = `[1:v]scale=iw*${widthPercent/100}:-1[logo]`;
    const overlayFilter = `[0:v][logo]overlay=${positions[position]}`;
    
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-i', 'logo.png',
        '-filter_complex', `${scaleFilter};${overlayFilter}`,
        '-c:a', 'copy',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 12. Adjust Brightness/Contrast/Saturation

```javascript
async function adjustColors(inputFile, brightness = 0, contrast = 1, saturation = 1, grayscale = false) {
    await loadVideoFile(inputFile);
    
    const sat = grayscale ? 0 : saturation;
    
    await ffmpeg.exec([
        '-i', 'input.mp4',
        '-vf', `eq=brightness=${brightness}:contrast=${contrast}:saturation=${sat}`,
        '-c:a', 'copy',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 13. Concatenate Videos

```javascript
async function concatenateVideos(inputFile1, inputFile2) {
    await ffmpeg.writeFile('input1.mp4', await fetchFile(inputFile1));
    await ffmpeg.writeFile('input2.mp4', await fetchFile(inputFile2));
    
    await ffmpeg.exec([
        '-i', 'input1.mp4',
        '-i', 'input2.mp4',
        '-filter_complex', '[0:v][0:a][1:v][1:a]concat=n=2:v=1:a=1[outv][outa]',
        '-map', '[outv]',
        '-map', '[outa]',
        '-c:v', 'libx264',
        '-c:a', 'aac',
        'output.mp4'
    ]);
    
    const data = await ffmpeg.readFile('output.mp4');
    return new Blob([data.buffer], { type: 'video/mp4' });
}
```

### 14. Extract Thumbnail

```javascript
async function extractThumbnail(inputFile, timestamp = '00:00:05', format = 'jpg') {
    await loadVideoFile(inputFile);
    
    const outputFile = `thumbnail.${format}`;
    
    await ffmpeg.exec([
        '-ss', timestamp,
        '-i', 'input.mp4',
        '-vframes', '1',
        '-q:v', '2',
        outputFile
    ]);
    
    const data = await ffmpeg.readFile(outputFile);
    const mimeType = format === 'png' ? 'image/png' : 'image/jpeg';
    return new Blob([data.buffer], { type: mimeType });
}
```

## Complete Working Example

```javascript
// complete-video-processor.js

const { FFmpeg } = FFmpegWASM;
const { fetchFile } = FFmpegWASM;

class VideoProcessor {
    constructor() {
        this.ffmpeg = null;
        this.loaded = false;
    }
    
    async init() {
        if (this.loaded) return;
        
        this.ffmpeg = new FFmpeg();
        
        this.ffmpeg.on('log', ({ message }) => {
            console.log('[FFmpeg]:', message);
        });
        
        this.ffmpeg.on('progress', ({ progress }) => {
            const percent = Math.round(progress * 100);
            this.updateProgress(percent);
        });
        
        await this.ffmpeg.load({
            coreURL: 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/umd/ffmpeg-core.js',
            wasmURL: 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/umd/ffmpeg-core.wasm',
        });
        
        this.loaded = true;
        console.log('FFmpeg loaded and ready');
    }
    
    updateProgress(percent) {
        const progressBar = document.getElementById('progress');
        if (progressBar) {
            progressBar.style.width = `${percent}%`;
            progressBar.textContent = `${percent}%`;
        }
    }
    
    async loadFile(file, filename = 'input.mp4') {
        await this.init();
        await this.ffmpeg.writeFile(filename, await fetchFile(file));
    }
    
    async getOutputBlob(filename, mimeType) {
        const data = await this.ffmpeg.readFile(filename);
        return new Blob([data.buffer], { type: mimeType });
    }
    
    async convertToGIF(file, options = {}) {
        const { width = 480, fps = 10 } = options;
        
        await this.loadFile(file);
        
        // Generate palette
        await this.ffmpeg.exec([
            '-i', 'input.mp4',
            '-vf', `fps=${fps},scale=${width}:-1:flags=lanczos,palettegen`,
            'palette.png'
        ]);
        
        // Create GIF with palette
        await this.ffmpeg.exec([
            '-i', 'input.mp4',
            '-i', 'palette.png',
            '-filter_complex', `fps=${fps},scale=${width}:-1:flags=lanczos[x];[x][1:v]paletteuse`,
            'output.gif'
        ]);
        
        return await this.getOutputBlob('output.gif', 'image/gif');
    }
    
    async compress(file, options = {}) {
        const { crf = 23, preset = 'medium' } = options;
        
        await this.loadFile(file);
        
        await this.ffmpeg.exec([
            '-i', 'input.mp4',
            '-c:v', 'libx264',
            '-crf', crf.toString(),
            '-preset', preset,
            '-c:a', 'aac',
            'output.mp4'
        ]);
        
        return await this.getOutputBlob('output.mp4', 'video/mp4');
    }
    
    async trim(file, startTime, duration) {
        await this.loadFile(file);
        
        await this.ffmpeg.exec([
            '-ss', startTime.toString(),
            '-i', 'input.mp4',
            '-t', duration.toString(),
            '-c', 'copy',
            'output.mp4'
        ]);
        
        return await this.getOutputBlob('output.mp4', 'video/mp4');
    }
    
    downloadBlob(blob, filename) {
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = filename;
        a.click();
        URL.revokeObjectURL(url);
    }
}

// Usage
const processor = new VideoProcessor();

document.getElementById('gifBtn').addEventListener('click', async () => {
    const fileInput = document.getElementById('videoInput');
    const file = fileInput.files[0];
    
    if (!file) {
        alert('Please select a video file');
        return;
    }
    
    try {
        const gifBlob = await processor.convertToGIF(file, {
            width: 640,
            fps: 15
        });
        processor.downloadBlob(gifBlob, 'output.gif');
    } catch (error) {
        console.error('Error:', error);
        alert('Processing failed: ' + error.message);
    }
});
```

## HTML Interface Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Video Processor</title>
    <style>
        body {
            font-family: system-ui, -apple-system, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        .controls {
            margin: 20px 0;
        }
        button {
            padding: 10px 20px;
            margin: 5px;
            cursor: pointer;
        }
        #progress {
            width: 0%;
            height: 30px;
            background: #4CAF50;
            text-align: center;
            line-height: 30px;
            color: white;
            transition: width 0.3s;
        }
        .progress-container {
            width: 100%;
            background: #ddd;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <h1>Browser Video Processor</h1>
    
    <div class="controls">
        <input type="file" id="videoInput" accept="video/*">
    </div>
    
    <div class="controls">
        <button id="gifBtn">Convert to GIF</button>
        <button id="compressBtn">Compress Video</button>
        <button id="extractAudioBtn">Extract Audio</button>
        <button id="thumbnailBtn">Extract Thumbnail</button>
    </div>
    
    <div class="progress-container">
        <div id="progress">0%</div>
    </div>
    
    <div id="output"></div>
    
    <script src="https://unpkg.com/@ffmpeg/ffmpeg@0.12.7/dist/umd/ffmpeg.js"></script>
    <script src="complete-video-processor.js"></script>
    <script>
        const processor = new VideoProcessor();
        
        document.getElementById('compressBtn').addEventListener('click', async () => {
            const file = document.getElementById('videoInput').files[0];
            if (!file) return alert('Select a video first');
            
            const compressed = await processor.compress(file, { crf: 28 });
            processor.downloadBlob(compressed, 'compressed.mp4');
        });
        
        document.getElementById('extractAudioBtn').addEventListener('click', async () => {
            const file = document.getElementById('videoInput').files[0];
            if (!file) return alert('Select a video first');
            
            await processor.loadFile(file);
            await processor.ffmpeg.exec([
                '-i', 'input.mp4',
                '-vn',
                '-c:a', 'libmp3lame',
                'audio.mp3'
            ]);
            
            const audio = await processor.getOutputBlob('audio.mp3', 'audio/mp3');
            processor.downloadBlob(audio, 'audio.mp3');
        });
        
        document.getElementById('thumbnailBtn').addEventListener('click', async () => {
            const file = document.getElementById('videoInput').files[0];
            if (!file) return alert('Select a video first');
            
            await processor.loadFile(file);
            await processor.ffmpeg.exec([
                '-ss', '00:00:03',
                '-i', 'input.mp4',
                '-vframes', '1',
                'thumb.jpg'
            ]);
            
            const thumb = await processor.getOutputBlob('thumb.jpg', 'image/jpeg');
            processor.downloadBlob(thumb, 'thumbnail.jpg');
        });
    </script>
</body>
</html>
```

## Advanced Patterns

### Screen Wake Lock During Processing

```javascript
async function processWithWakeLock(processingFunction) {
    let wakeLock = null;
    
    try {
        if ('wakeLock' in navigator) {
            wakeLock = await navigator.wakeLock.request('screen');
            console.log('Wake lock activated');
        }
        
        const result = await processingFunction();
        return result;
        
    } finally {
        if (wakeLock) {
            await wakeLock.release();
            console.log('Wake lock released');
        }
    }
}

// Usage
const result = await processWithWakeLock(async () => {
    return await processor.compress(videoFile, { crf: 23 });
});
```

### Web Worker for Background Processing

```javascript
// worker.js
importScripts('https://unpkg.com/@ffmpeg/ffmpeg@0.12.7/dist/umd/ffmpeg.js');

const { FFmpeg } = FFmpegWASM;

let ffmpeg = null;

self.onmessage = async (e) => {
    const { type, data } = e.data;
    
    if (type === 'load') {
        ffmpeg = new FFmpeg();
        await ffmpeg.load();
        self.postMessage({ type: 'loaded' });
    }
    
    if (type === 'process') {
        const { file, operation, options } = data;
        
        await ffmpeg.writeFile('input.mp4', file);
        
        // Execute operation based on type
        if (operation === 'compress') {
            await ffmpeg.exec([
                '-i', 'input.mp4',
                '-c:v', 'libx264',
                '-crf', options.crf.toString(),
                'output.mp4'
            ]);
        }
        
        const output = await ffmpeg.readFile('output.mp4');
        self.postMessage({ type: 'complete', data: output });
    }
};

// main.js
const worker = new Worker('worker.js');

worker.onmessage = (e) => {
    const { type, data } = e.data;
    
    if (type === 'loaded') {
        console.log('Worker ready');
    }
    
    if (type === 'complete') {
        const blob = new Blob([data.buffer], { type: 'video/mp4' });
        downloadBlob(blob, 'output.mp4');
    }
};

worker.postMessage({ type: 'load' });
```

### Estimate Output Size

```javascript
async function estimateOutputSize(inputFile, crf) {
    const inputSize = inputFile.size;
    
    // Rough estimation based on CRF
    // CRF 18 = ~1.0x, CRF 23 = ~0.5x, CRF 28 = ~0.25x, CRF 51 = ~0.05x
    const sizeMultipliers = {
        18: 1.0,
        23: 0.5,
        28: 0.25,
        33: 0.15,
        38: 0.10,
        51: 0.05
    };
    
    // Interpolate
    let multiplier = 0.5;
    const keys = Object.keys(sizeMultipliers).map(Number).sort((a, b) => a - b);
    
    for (let i = 0; i < keys.length - 1; i++) {
        if (crf >= keys[i] && crf <= keys[i + 1]) {
            const range = keys[i + 1] - keys[i];
            const progress = (crf - keys[i]) / range;
            multiplier = sizeMultipliers[keys[i]] + 
                        (sizeMultipliers[keys[i + 1]] - sizeMultipliers[keys[i]]) * progress;
            break;
        }
    }
    
    return Math.round(inputSize * multiplier);
}

// Usage
const estimated = estimateOutputSize(videoFile, 28);
console.log(`Estimated output: ${(estimated / 1024 / 1024).toFixed(2)} MB`);
```

## Troubleshooting

### FFmpeg Not Loading

```javascript
async function loadWithRetry(maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            await ffmpeg.load({
                coreURL: 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/umd/ffmpeg-core.js',
                wasmURL: 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/umd/ffmpeg-core.wasm',
            });
            return true;
        } catch (error) {
            console.error(`Load attempt ${i + 1} failed:`, error);
            if (i === maxRetries - 1) throw error;
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
        }
    }
}
```

### Memory Issues with Large Files

```javascript
// Process large files in chunks or with lower quality settings
async function processLargeFile(file) {
    const MAX_SIZE = 500 * 1024 * 1024; // 500 MB
    
    if (file.size > MAX_SIZE) {
        console.warn('Large file detected, using aggressive compression');
        return await processor.compress(file, {
            crf: 28,
            preset: 'faster' // Faster preset uses less memory
        });
    }
    
    return await processor.compress(file, { crf: 23 });
}
```

### Cross-Origin Headers for SharedArrayBuffer

If using multi-threading (advanced ffmpeg.wasm feature), serve with:

```javascript
// server.js (Express example)
app.use((req, res, next) => {
    res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
    res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
    next();
});
```

### Clean Up Virtual Filesystem

```javascript
async function cleanupFiles() {
    try {
        await ffmpeg.deleteFile('input.mp4');
        await ffmpeg.deleteFile('output.mp4');
        await ffmpeg.deleteFile('palette.png');
        console.log('Cleanup complete');
    } catch (error) {
        console.warn('Cleanup error:', error);
    }
}

// Call after each operation
await processor.compress(file);
await cleanupFiles();
```

## Configuration

### Set Custom FFmpeg Arguments

```javascript
async function rawCommand(inputFile, args, outputExt) {
    await processor.loadFile(inputFile);
    
    const outputFile = `output.${outputExt}`;
    
    // User provides args array: ['-vf', 'scale=1280:-1', '-c:v', 'libx264']
    await processor.ffmpeg.exec([
        '-i', 'input.mp4',
        ...args,
        outputFile
    ]);
    
    return await processor.getOutputBlob(outputFile, `video/${outputExt}`);
}

// Example: Apply custom filter
const result = await rawCommand(videoFile, [
    '-vf', 'eq=brightness=0.1:contrast=1.2',
    
