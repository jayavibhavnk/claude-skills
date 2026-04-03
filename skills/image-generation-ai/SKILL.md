---
name: image-generation-ai
description: Generate images with AI - prompts, models, post-processing, and integration patterns.
metadata:
  priority: 8
  docs:
    - "https://sdk.vercel.ai/docs/ai-sdk-core/image-generation"
  pathPatterns:
    - "**/image/**"
    - "**/generation/**"
  bashPatterns:
    - '\bimage.?generation\b'
    - '\bdall.?e\b'
    - '\bstable.?diffusion\b'
  promptSignals:
    phrases:
      - "image generation"
      - "AI art"
      - "generate image"
    anyOf:
      - "image"
      - "generation"
      - "picture"
---

## Image Generation AI

### Using AI SDK

```typescript
import { generateImage } from 'ai';

const { images } = await generateImage({
  model: 'openai/dall-e-3',
  prompt: 'A serene mountain landscape at sunset with a crystal clear lake reflecting the orange sky',
});

// Or with AI Gateway (recommended)
const { images } = await generateImage({
  model: 'google/gemini-3.1-flash-image-preview',
  prompt: 'A futuristic cityscape with flying cars and neon lights',
});

// Access generated images
const imageUrl = images[0].url;
const base64 = images[0].base64;
```

### Image Generation Patterns

```typescript
// Simple generation
async function generateHeroImage(topic: string): Promise<string> {
  const { images } = await generateImage({
    model: 'google/gemini-3.1-flash-image-preview',
    prompt: `Professional hero image of ${topic}, 16:9 aspect ratio, high quality`,
  });
  return images[0].url;
}

// Batch generation
async function generateVariations(prompt: string, count: number) {
  const promises = Array(count).fill(null).map(() =>
    generateImage({
      model: 'google/gemini-3.1-flash-image-preview',
      prompt,
    })
  );

  const results = await Promise.all(promises);
  return results.flatMap(r => r.images.map(i => i.url));
}

// With style control
async function generateStyledImage(
  subject: string,
  style: 'realistic' | 'artistic' | 'anime'
) {
  const stylePrompts = {
    realistic: `${subject}, photorealistic, high detail, natural lighting`,
    artistic: `${subject}, digital art, vibrant colors, artistic interpretation`,
    anime: `${subject}, anime style, cel shading, manga illustration`,
  };

  const { images } = await generateImage({
    model: 'google/gemini-3.1-flash-image-preview',
    prompt: stylePrompts[style],
  });

  return images[0].url;
}
```

### Prompt Engineering

```typescript
// Good prompts structure
interface PromptComponents {
  subject: string;      // Main subject
  setting: string;       // Environment/background
  style: string;         // Art style
  lighting: string;       // Lighting conditions
  quality: string;       // Quality modifiers
  technical: string;     // Technical specs (aspect ratio, etc.)
}

// Template for structured prompts
function buildPrompt(components: PromptComponents): string {
  return [
    components.subject,
    components.setting,
    `Style: ${components.style}`,
    `Lighting: ${components.lighting}`,
    `Quality: ${components.quality}`,
    components.technical,
  ].join(', ');
}

// Example usage
const prompt = buildPrompt({
  subject: 'A young woman with flowing hair',
  setting: 'standing on a cliff overlooking the ocean',
  style: 'cinematic photography',
  lighting: 'golden hour sunset lighting',
  quality: '8k resolution, highly detailed',
  technical: 'shot on 35mm, f/1.8 aperture',
});
```

### Image Processing Pipeline

```typescript
import { v4 as uuidv4 } from 'uuid';
import { Storage } from '@google-cloud/storage';

const storage = new Storage();

// Save and process generated image
async function processGeneratedImage(
  base64Data: string,
  userId: string,
  options: { resize?: number; format?: 'webp' | 'png' }
) {
  const imageId = uuidv4();
  const filename = `${userId}/${imageId}.${options.format || 'webp'}`;

  // Upload original
  const bucket = storage.bucket('my-images');
  const file = bucket.file(filename);

  await file.save(Buffer.from(base64Data, 'base64'), {
    metadata: {
      contentType: `image/${options.format || 'webp'}`,
      metadata: { userId, generatedAt: new Date().toISOString() },
    },
  });

  // Generate thumbnails if needed
  if (options.resize) {
    await generateThumbnail(filename, options.resize);
  }

  return {
    id: imageId,
    url: `https://storage.googleapis.com/my-images/${filename}`,
  };
}
```

### Moderation & Safety

```typescript
interface ModerationResult {
  safe: boolean;
  categories: {
    violence: number;
    adult: number;
    hate: number;
    harassment: number;
  };
}

// Client-side pre-check (before generation)
// Note: Use AI Gateway for moderation to avoid direct API keys
async function checkPromptSafety(prompt: string): Promise<ModerationResult> {
  // Use AI Gateway for moderation
  const response = await fetch('https://gateway.vercel.ai/v1/moderate', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.VERCEL_AI_GATEWAY_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ input: prompt }),
  });

  const data = await response.json();

  const results = data.results[0];
  return {
    safe: !results.flagged,
    categories: {
      violence: results.categories.violence,
      adult: results.categories.sexual,
      hate: results.categories.hate,
      harassment: results.categories.harassment,
    },
  };
}

// Block unsafe prompts
async function safeGenerate(prompt: string) {
  const safety = await checkPromptSafety(prompt);

  if (!safety.safe) {
    throw new Error('Prompt violates safety guidelines');
  }

  return generateImage({ prompt });
}
```

### Use Cases

| Use Case | Recommended Model | Prompt Tips |
|----------|------------------|------------|
| Product photos | DALL-E 3 | "professional product photography, studio lighting" |
| Portraits | Midjourney | "portrait, soft lighting, detailed face" |
| Landscapes | Stable Diffusion | "epic landscape, dramatic lighting, 8k" |
| UI Mockups | DALL-E 3 | "clean UI design, app screen, minimal" |
| Logos | Midjourney | "minimalist logo, vector style, brand mark" |
| Anime | NovelAI | "anime style, detailed illustration" |

### Best Practices

1. **Be specific** - Subject, setting, style, lighting
2. **Use negative prompts** - What you DON'T want
3. **Check safety** - Moderate prompts before generation
4. **Store originals** - Keep base64 for reprocessing
5. **Optimize formats** - Convert to WebP for web
6. **Track generations** - Log prompts and results for improvement
7. **Respect copyright** - Don't generate trademarked content
