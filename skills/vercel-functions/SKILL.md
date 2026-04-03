---
name: vercel-functions
description: Build serverless functions and edge compute solutions on Vercel.
metadata:
  priority: 9
  docs:
    - "https://vercel.com/docs/functions"
    - "https://vercel.com/docs/functions/edge-functions"
  pathPatterns:
    - "api/**/*.ts"
    - "api/**/*.js"
    - "app/api/**"
    - "pages/api/**"
    - "_middleware.ts"
    - "middleware.ts"
  bashPatterns:
    - '\bvercel\s+dev\b'
    - '\bvercel\s+functions\b'
  promptSignals:
    phrases:
      - "serverless function"
      - "api route"
      - "edge function"
    anyOf:
      - "serverless"
      - "edge compute"
      - "api endpoint"
      - "server-side"
---

## Vercel Functions

### Function Types

1. **Serverless Functions** - Node.js runtime, scales automatically
2. **Edge Functions** - Runs at edge locations, lower latency
3. **Background Functions** - Long-running tasks with async execution

### Creating API Routes

#### App Router (Next.js 13+)
```typescript
// app/api/hello/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ message: 'Hello' });
}

export async function POST(request: Request) {
  const body = await request.json();
  return NextResponse.json({ received: body });
}
```

#### Pages Router (Next.js 12)
```typescript
// pages/api/hello.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    res.status(200).json({ message: 'Hello' });
  }
}
```

### Edge Functions

```typescript
// middleware.ts or app/hello-edge/route.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export const config = {
  runtime: 'edge',
};

export function middleware(request: NextRequest) {
  // Runs at edge locations
  const response = NextResponse.json({
    message: 'Hello from the edge!',
  });

  response.headers.set('X-Edge-Location', 'near-you');
  return response;
}
```

### API Best Practices

1. **Return proper status codes**
2. **Handle errors gracefully**
3. **Validate input with Zod or similar**
4. **Use streaming for large responses**
5. **Implement caching headers**

### Configuration

```json
// vercel.json
{
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 10
    }
  }
}
```

### Cold Start Optimization

- Keep function bundle size small
- Avoid importing heavy dependencies at top level
- Use dynamic imports for optional features
- Connection pooling for databases

## Performance

| Metric | Target |
|--------|--------|
| Cold start | < 200ms (edge), < 1s (serverless) |
| Memory | 1024MB default, adjustable |
| Timeout | 10s default, max 300s |
| Response size | < 4.5MB |

## Error Handling

```typescript
export async function GET() {
  try {
    const data = await fetchData();
    return NextResponse.json({ data });
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```
