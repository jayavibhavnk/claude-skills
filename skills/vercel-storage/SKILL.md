---
name: vercel-storage
description: Work with Vercel Blob for file storage, uploads, and management.
metadata:
  priority: 7
  docs:
    - "https://vercel.com/docs/storage/vercel-blob"
  pathPatterns:
    - "**/blob/**"
    - "**/storage/**"
    - "**/upload/**"
  bashPatterns:
    - '\b@vercel/blob\b'
    - '\bput\s*\('
    - '\bdel\s*\('
  promptSignals:
    phrases:
      - "vercel blob"
      - "file storage"
      - "upload files"
    anyOf:
      - "storage"
      - "files"
      - "upload"
      - "download"
      - "s3"
---

## Vercel Blob Storage

### Setup

```typescript
import { put, del, list } from '@vercel/blob';
```

### Basic Operations

#### Upload Files
```typescript
// Simple upload
const blob = await put('my-file.txt', 'Hello world', {
  access: 'public',
});

// Upload from a URL
const blob = await put('remote-file.txt', response.body, {
  access: 'public',
});

// With content type
const blob = await put('image.png', imageBuffer, {
  contentType: 'image/png',
  access: 'public',
});
```

#### Download Files
```typescript
import { get } from '@vercel/blob';

const blob = await get('my-file.txt');

// Read as text
const text = await blob.text();

// Read as buffer
const buffer = await blob.arrayBuffer();
```

#### Delete Files
```typescript
import { del } from '@vercel/blob';

await del('my-file.txt');

// Delete multiple
await del(['file1.txt', 'file2.txt']);
```

#### List Files
```typescript
const { blobs } = await list({
  prefix: 'documents/',
  limit: 100,
});

for (const blob of blobs) {
  console.log(blob.url, blob.size, blob.uploadedAt);
}
```

### Signed URLs

```typescript
import { createSignedUrl } from '@vercel/blob';

const signedUrl = await createSignedUrl(blob.url, {
  expiresIn: 60 * 60, // 1 hour
});
```

### Multipart Upload

```typescript
import { put } from '@vercel/blob';

const blob = await put('large-file.zip', largeFileStream, {
  access: 'public',
  maxPartSize: 50 * 1024 * 1024, // 50MB parts
});
```

### Best Practices

1. Use meaningful file paths with prefixes (`images/`, `documents/`)
2. Set appropriate `access` level (public/private)
3. Use content types for proper browser handling
4. Implement cleanup for temporary files
5. Use signed URLs for private content
6. Handle upload failures with retries

### Security

```typescript
// Never expose tokens directly
// Use environment variables
const BLOB_READ_WRITE_TOKEN = process.env.BLOB_READ_WRITE_TOKEN;
```
