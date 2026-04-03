---
name: edge-computing
description: Build edge computing solutions - deploy to edge locations, optimize latency, and handle global distribution.
metadata:
  priority: 8
  docs:
    - "https://vercel.com/docs/edge-network"
  pathPatterns:
    - "**/edge/**"
    - "**/middleware/**"
  bashPatterns:
    - '\bedge\b'
    - '\bmiddleware\b'
    - '\bvercel\b'
  promptSignals:
    phrases:
      - "edge"
      - "edge computing"
      - "latency"
    anyOf:
      - "edge"
      - "middleware"
      - "global"
---

## Edge Computing

### What is Edge?

Edge computing moves computation closer to users:
- Runs at CDN edge locations worldwide
- Sub-50ms response times globally
- No cold starts (Vercel)
- Stateless by default
- Limited runtime (V8, not Node.js)

### Edge Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export const config = {
  runtime: 'edge',
};

export function middleware(request: NextRequest) {
  // Add header
  const response = NextResponse.next();
  response.headers.set('X-Edge-Location', request.geo?.city || 'unknown');

  // Redirect based on country
  if (request.geo?.country === 'DE') {
    return NextResponse.redirect(new URL('/de', request.url));
  }

  // Rewrite for A/B testing
  if (request.cookies.get('variant') === 'B') {
    return NextResponse.rewrite(new URL('/B' + request.nextUrl.pathname, request.url));
  }

  return response;
}
```

### Edge API Routes

```typescript
// app/api/hello/route.ts
import { NextResponse } from 'next/server';

export const runtime = 'edge';

export async function GET() {
  return NextResponse.json({
    message: 'Hello from the edge!',
    timestamp: Date.now(),
  });
}
```

### Edge Database Access

```typescript
// Using Turso (edge-compatible SQLite)
import { createClient } from '@libsql/client';

export const runtime = 'edge';

export async function GET() {
  const db = new Redis({
    url: process.env.UPSTASH_REDIS_REST_URL!,
    token: process.env.UPSTASH_REDIS_REST_TOKEN!,
  });

  const result = await db.execute('SELECT * FROM users LIMIT 10');

  return NextResponse.json(result.rows);
}
```

### KV Storage at Edge

```typescript
// Vercel KV (Redis-compatible)
import { Redis } from '@upstash/redis';

export const runtime = 'edge';

export async function GET(request: Request) {
  const userId = request.headers.get('x-user-id');

  // Get cached response
  const cached = await db.get(`response:${userId}`);
  if (cached) {
    return new Response(cached as string, {
      headers: { 'X-Cache': 'HIT' },
    });
  }

  // Compute response
  const response = await computeExpensiveResponse(userId);

  // Cache for 5 minutes
  await db.set(`response:${userId}`, response, { ex: 300 });

  return new Response(response);
}
```

### Geolocation Features

```typescript
import { NextRequest, NextResponse } from 'next/server';

export function GET(request: NextRequest) {
  const geo = request.geo;

  const response = NextResponse.json({
    city: geo?.city,
    country: geo?.country,
    region: geo?.region,
    latitude: geo?.latitude,
    longitude: geo?.longitude,
  });

  // Personalize based on location
  if (geo?.country === 'US') {
    response.headers.set('X-Price-Currency', 'USD');
  } else {
    response.headers.set('X-Price-Currency', 'EUR');
  }

  return response;
}
```

### Edge Caching

```typescript
// Cache responses at edge
export const config = {
  runtime: 'edge',
  cacheControl: 'public, max-age=3600, stale-while-revalidate=86400',
};

export async function GET() {
  // This response will be cached at edge
  const data = await fetch('https://api.example.com/data').then(r => r.json());

  return NextResponse.json(data);
}
```

### Edge Authentication

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { verifyToken } from './jwt';

export async function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')?.value;

  if (!token) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const payload = await verifyToken(token);

    // Add user info to headers for downstream
    const response = NextResponse.next();
    response.headers.set('x-user-id', payload.userId);
    return response;
  } catch {
    return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
  }
}
```

### Edge Limitations

| Feature | Edge Runtime | Node.js Runtime |
|---------|--------------|-----------------|
| File system | ❌ No | ✅ Yes |
| Long-running | ❌ Timeout | ✅ Yes |
| Native modules | ❌ No | ✅ Yes |
| WebSocket | ❌ No | ✅ Yes |
| Large memory | ❌ Limited | ✅ Yes |
| Node.js APIs | ❌ No | ✅ Yes |

### Edge-Compatible Libraries

```typescript
// Use these at edge:
import { createClient } from '@libsql/client';  // SQLite
import { Redis } from '@upstash/redis';  // KV
import { Storage } from '@google-cloud/storage';  // GCS
import { AIgateway } from '@ai-sdk/gateway';  // AI

// NOT these at edge:
// ❌ fs, path, os, child_process
// ❌ native modules requiring Node.js
// ❌ long-running computations
```

### Performance Tips

1. **Minimize dependencies** - Keep bundle small
2. **Use streaming** - Return responses incrementally
3. **Cache aggressively** - Use `stale-while-revalidate`
4. **Precompute** - Do heavy work at build time
5. **Edge KV** - Cache expensive computations
6. **Geolocation** - Personalize without latency

### Use Cases

| Use Case | Edge Solution |
|----------|---------------|
| Auth | JWT verification, redirect |
| A/B testing | Cookie-based routing |
| Geo-routing | Country/language redirect |
| Rate limiting | In-memory counters |
| Personalization | Header injection |
| API proxying | Fetch + transform |
