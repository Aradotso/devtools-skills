---
name: clipsketch-ai-video-storyboard
description: Turn video moments into AI-generated hand-drawn storyboards with Gemini-powered frame analysis and social media content generation
triggers:
  - how do I use ClipSketch AI to create video storyboards
  - extract and tag frames from Bilibili or Xiaohongshu videos
  - generate hand-drawn storyboards from video frames
  - create social media content from video moments
  - setup ClipSketch AI with Gemini API
  - export tagged video frames as storyboard
  - use AI to generate video cover images
  - batch process video frames with Gemini
---

# ClipSketch AI Video Storyboard Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

ClipSketch AI is a TypeScript/React application that transforms video moments into hand-drawn storyboards using Google Gemini's multimodal AI. It supports importing videos from Bilibili and Xiaohongshu (Little Red Book), frame-precise tagging, and automatic generation of storyboards, social media copy, and cover images.

## Installation

```bash
git clone https://github.com/RanFeng/clipsketch-ai.git
cd clipsketch-ai
npm install
```

## Configuration

Create `.env.local` in project root:

```env
GEMINI_API_KEY=your_google_gemini_api_key_here
```

Required API access:
- `gemini-3-pro-image-preview` (for storyboard generation)
- `gemini-3-pro-preview` (for text/copy generation)

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Docker deployment
docker run -d --restart=always --name clipsketch-ai -p 3000:3000 earisty/clipsketch-ai:latest
```

Access at `http://localhost:3000`

## Core Workflow

### 1. Video Import

**Supported Platforms:**
- Bilibili (supports short links and mixed text)
- Xiaohongshu (Little Red Book)

**URL Pattern Recognition:**
```typescript
// Bilibili patterns
const bilibiliPatterns = [
  /bilibili\.com\/video\/(BV\w+)/,
  /b23\.tv\/\w+/
];

// Xiaohongshu patterns
const xhsPatterns = [
  /xhslink\.com\/\w+/,
  /xiaohongshu\.com\/.*\/(\w+)/
];
```

**Import Example:**
```typescript
// User pastes share link (can include promotional text)
const shareText = "超好看的视频！https://b23.tv/abc123 快来看看";
// App extracts and resolves to video URL
```

### 2. Frame Tagging System

**Keyboard Controls:**
- `Space`: Play/Pause
- `←/→`: Frame-by-frame or smart step navigation
- `T`: Tag current frame (millisecond precision)

**Tag Data Structure:**
```typescript
interface VideoTag {
  id: string;
  timestamp: number; // milliseconds
  thumbnailDataUrl: string; // base64 canvas capture
  videoUrl: string;
}

// Tagging implementation pattern
const captureFrame = (videoElement: HTMLVideoElement, currentTime: number): VideoTag => {
  const canvas = document.createElement('canvas');
  canvas.width = videoElement.videoWidth;
  canvas.height = videoElement.videoHeight;
  
  const ctx = canvas.getContext('2d');
  ctx?.drawImage(videoElement, 0, 0);
  
  return {
    id: `tag-${Date.now()}`,
    timestamp: currentTime * 1000,
    thumbnailDataUrl: canvas.toDataURL('image/jpeg', 0.9),
    videoUrl: videoElement.src
  };
};
```

### 3. AI Storyboard Generation

**Multi-Step Pipeline:**

#### Step 1: Creative Analysis
```typescript
import { GoogleGenAI } from '@google/genai';

const analyzeFrames = async (frames: VideoTag[], apiKey: string) => {
  const genAI = new GoogleGenAI(apiKey);
  const model = genAI.getGenerativeModel({ model: 'gemini-3-pro-preview' });
  
  const prompt = `
    Analyze these video frames and extract:
    1. Main narrative steps
    2. Key visual elements
    3. Character actions
    4. Scene transitions
    
    Frames: ${frames.length}
  `;
  
  const imageParts = frames.map(frame => ({
    inlineData: {
      data: frame.thumbnailDataUrl.split(',')[1], // Remove data:image/jpeg;base64,
      mimeType: 'image/jpeg'
    }
  }));
  
  const result = await model.generateContent([prompt, ...imageParts]);
  return result.response.text();
};
```

#### Step 2: Storyboard Image Generation
```typescript
const generateStoryboard = async (
  frames: VideoTag[],
  analysis: string,
  apiKey: string,
  customCharacter?: string // Optional character image URL
) => {
  const genAI = new GoogleGenAI(apiKey);
  const model = genAI.getGenerativeModel({ 
    model: 'gemini-3-pro-image-preview' 
  });
  
  const prompt = `
    Create a cute hand-drawn style storyboard combining these ${frames.length} frames.
    
    Style requirements:
    - Cute, friendly illustration style
    - Coherent visual narrative
    - Warm color palette
    - Clear panel separation
    
    Narrative context: ${analysis}
    
    ${customCharacter ? `Integrate this character: ${customCharacter}` : ''}
  `;
  
  const imageParts = frames.map(frame => ({
    inlineData: {
      data: frame.thumbnailDataUrl.split(',')[1],
      mimeType: 'image/jpeg'
    }
  }));
  
  if (customCharacter) {
    imageParts.push({
      inlineData: {
        data: customCharacter.split(',')[1],
        mimeType: 'image/jpeg'
      }
    });
  }
  
  const result = await model.generateContent([prompt, ...imageParts]);
  return result.response.text(); // Returns image data or URL
};
```

#### Step 3: Panel Refinement (Batch Mode)
```typescript
const refinePanels = async (
  storyboardImage: string,
  panelCount: number,
  apiKey: string,
  useBatchAPI: boolean = false
) => {
  const genAI = new GoogleGenAI(apiKey);
  const model = genAI.getGenerativeModel({ 
    model: 'gemini-3-pro-image-preview' 
  });
  
  if (useBatchAPI) {
    // Cost-efficient batch processing
    const batchPromises = Array.from({ length: panelCount }, (_, i) => ({
      prompt: `Enhance panel ${i + 1} from this storyboard in high resolution`,
      image: storyboardImage
    }));
    
    // Process in batches to save API costs
    const results = await model.batchGenerate(batchPromises);
    return results;
  } else {
    // Individual requests for immediate results
    const refinedPanels = [];
    for (let i = 0; i < panelCount; i++) {
      const result = await model.generateContent([
        `Enhance and upscale panel ${i + 1} from this storyboard`,
        {
          inlineData: {
            data: storyboardImage.split(',')[1],
            mimeType: 'image/jpeg'
          }
        }
      ]);
      refinedPanels.push(result.response.text());
    }
    return refinedPanels;
  }
};
```

### 4. Social Media Copy Generation

```typescript
const generateSocialCopy = async (
  storyboardImage: string,
  videoContext: string,
  apiKey: string
) => {
  const genAI = new GoogleGenAI(apiKey);
  const model = genAI.getGenerativeModel({ model: 'gemini-3-pro-preview' });
  
  const prompt = `
    Generate 3 different styles of social media copy (for Xiaohongshu/Little Red Book):
    
    1. Emotional Storytelling Style (200-300 chars)
       - Personal, relatable narrative
       - Emotional hooks
       - Conversational tone
    
    2. Tutorial/Guide Style (150-250 chars)
       - Step-by-step breakdown
       - Practical tips
       - Clear value proposition
    
    3. Short & Punchy Style (80-120 chars)
       - Attention-grabbing opener
       - Concise key message
       - Call-to-action
    
    Video context: ${videoContext}
    
    Include relevant emojis and hashtags.
  `;
  
  const result = await model.generateContent([
    prompt,
    {
      inlineData: {
        data: storyboardImage.split(',')[1],
        mimeType: 'image/jpeg'
      }
    }
  ]);
  
  return parseCopyVariants(result.response.text());
};

interface CopyVariant {
  style: 'emotional' | 'tutorial' | 'punchy';
  text: string;
  hashtags: string[];
}

const parseCopyVariants = (aiResponse: string): CopyVariant[] => {
  // Parse AI response into structured copy variants
  // Implementation depends on AI output format
  return [
    { style: 'emotional', text: '...', hashtags: ['#story', '#life'] },
    { style: 'tutorial', text: '...', hashtags: ['#howto', '#tips'] },
    { style: 'punchy', text: '...', hashtags: ['#viral', '#trending'] }
  ];
};
```

### 5. Cover Image Generation

```typescript
const generateCoverImage = async (
  selectedCopy: string,
  storyboardImage: string,
  originalFrames: VideoTag[],
  apiKey: string
) => {
  const genAI = new GoogleGenAI(apiKey);
  const model = genAI.getGenerativeModel({ 
    model: 'gemini-3-pro-image-preview' 
  });
  
  const prompt = `
    Create a vertical video cover (9:16 aspect ratio) that:
    
    - Incorporates key visual from the storyboard
    - Highlights this copy: "${selectedCopy}"
    - Optimized for social media thumbnail
    - Eye-catching, high-contrast design
    - Maintains hand-drawn aesthetic
    
    Cover should work as standalone promotion image.
  `;
  
  const imageParts = [
    {
      inlineData: {
        data: storyboardImage.split(',')[1],
        mimeType: 'image/jpeg'
      }
    },
    ...originalFrames.slice(0, 3).map(frame => ({
      inlineData: {
        data: frame.thumbnailDataUrl.split(',')[1],
        mimeType: 'image/jpeg'
      }
    }))
  ];
  
  const result = await model.generateContent([prompt, ...imageParts]);
  return result.response.text();
};
```

## Data Export

### Export Tagged Frames as ZIP

```typescript
import JSZip from 'jszip';

const exportTagsAsZip = async (tags: VideoTag[]) => {
  const zip = new JSZip();
  
  // Add timestamp index file
  const timestampList = tags.map((tag, idx) => 
    `Frame ${idx + 1}: ${formatTimestamp(tag.timestamp)}`
  ).join('\n');
  
  zip.file('timestamps.txt', timestampList);
  
  // Add frame images
  for (let i = 0; i < tags.length; i++) {
    const imageData = tags[i].thumbnailDataUrl.split(',')[1];
    zip.file(`frame_${i + 1}.jpg`, imageData, { base64: true });
  }
  
  const blob = await zip.generateAsync({ type: 'blob' });
  
  // Trigger download
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `video-tags-${Date.now()}.zip`;
  a.click();
  URL.revokeObjectURL(url);
};

const formatTimestamp = (ms: number): string => {
  const totalSeconds = Math.floor(ms / 1000);
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;
  const milliseconds = ms % 1000;
  return `${minutes}:${seconds.toString().padStart(2, '0')}.${milliseconds.toString().padStart(3, '0')}`;
};
```

### Export Timeline as TXT

```typescript
const exportTimeline = (tags: VideoTag[]) => {
  const content = tags.map((tag, idx) => 
    `[${formatTimestamp(tag.timestamp)}] Frame ${idx + 1}`
  ).join('\n');
  
  const blob = new Blob([content], { type: 'text/plain' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `timeline-${Date.now()}.txt`;
  a.click();
  URL.revokeObjectURL(url);
};
```

## Responsive Design Patterns

```typescript
// Detect mobile/tablet and adjust layout
const useResponsiveLayout = () => {
  const [isMobile, setIsMobile] = useState(false);
  
  useEffect(() => {
    const checkMobile = () => {
      setIsMobile(window.innerWidth < 768);
    };
    
    checkMobile();
    window.addEventListener('resize', checkMobile);
    return () => window.removeEventListener('resize', checkMobile);
  }, []);
  
  return isMobile;
};

// Layout component
const VideoWorkspace: React.FC = () => {
  const isMobile = useResponsiveLayout();
  
  return (
    <div className={isMobile ? 'flex-col' : 'flex-row'}>
      <VideoPlayer className={isMobile ? 'w-full' : 'w-2/3'} />
      <TagList className={isMobile ? 'w-full mt-4' : 'w-1/3 ml-4'} />
    </div>
  );
};
```

## Local Storage with IndexedDB

```typescript
// Persist tags and workspace state
const saveToIndexedDB = async (videoUrl: string, tags: VideoTag[]) => {
  const db = await openDB('clipsketch-db', 1, {
    upgrade(db) {
      db.createObjectStore('tags', { keyPath: 'videoUrl' });
    }
  });
  
  await db.put('tags', { videoUrl, tags, updatedAt: Date.now() });
};

const loadFromIndexedDB = async (videoUrl: string): Promise<VideoTag[]> => {
  const db = await openDB('clipsketch-db', 1);
  const record = await db.get('tags', videoUrl);
  return record?.tags || [];
};
```

## Troubleshooting

### 403 Error on Gemini API

**Cause:** API key doesn't have access to `gemini-3-pro-image-preview` model.

**Solution:**
1. Check Google Cloud Console project settings
2. Enable Vertex AI API
3. Ensure billing is enabled
4. Request access to preview models if needed

### Video CORS Issues

**Cause:** Cross-origin restrictions prevent canvas capture.

**Solution:**
```typescript
// Use referrerPolicy to bypass some restrictions
<video 
  crossOrigin="anonymous"
  referrerPolicy="no-referrer"
  src={videoUrl}
/>

// If still failing, implement server-side proxy
// Add to next.config.js or custom server
async rewrites() {
  return [
    {
      source: '/api/proxy/:path*',
      destination: 'https://external-video-cdn.com/:path*'
    }
  ];
}
```

### Batch API Cost Optimization

**Pattern:** Use batch mode for large storyboards

```typescript
const BATCH_THRESHOLD = 5; // Use batch if >5 panels

const shouldUseBatch = (panelCount: number): boolean => {
  return panelCount > BATCH_THRESHOLD;
};

// Implement retry with exponential backoff for rate limits
const generateWithRetry = async (
  fn: () => Promise<any>,
  maxRetries: number = 3
) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
};
```

### Mobile Touch Controls

```typescript
// Add touch gesture support for mobile
const useTouchControls = (videoRef: React.RefObject<HTMLVideoElement>) => {
  const [touchStartX, setTouchStartX] = useState(0);
  
  const handleTouchStart = (e: React.TouchEvent) => {
    setTouchStartX(e.touches[0].clientX);
  };
  
  const handleTouchEnd = (e: React.TouchEvent) => {
    const touchEndX = e.changedTouches[0].clientX;
    const diff = touchEndX - touchStartX;
    
    if (Math.abs(diff) > 50) { // Swipe threshold
      const video = videoRef.current;
      if (video) {
        video.currentTime += diff > 0 ? -2 : 2; // Seek 2 seconds
      }
    }
  };
  
  return { handleTouchStart, handleTouchEnd };
};
```

## API Reference Summary

| Function | Model | Purpose |
|----------|-------|---------|
| `analyzeFrames` | gemini-3-pro-preview | Extract narrative structure |
| `generateStoryboard` | gemini-3-pro-image-preview | Create hand-drawn composite |
| `refinePanels` | gemini-3-pro-image-preview | Enhance individual panels |
| `generateSocialCopy` | gemini-3-pro-preview | Generate 3 copy variants |
| `generateCoverImage` | gemini-3-pro-image-preview | Create promotional thumbnail |

All functions require `process.env.GEMINI_API_KEY` or runtime API key input.
