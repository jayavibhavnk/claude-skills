---
name: ai-gateway-patterns
description: Build AI Gateway powered applications - unified API routing, failover, cost tracking, and multi-provider orchestration.
metadata:
  priority: 10
  docs:
    - "https://vercel.com/docs/ai-gateway"
  pathPatterns:
    - "**/gateway/**"
    - "**/ai-gateway/**"
  bashPatterns:
    - '\bai-gateway\b'
    - '\bgateway\b'
  promptSignals:
    phrases:
      - "ai gateway"
      - "model routing"
      - "failover"
    anyOf:
      - "gateway"
      - "multi-provider"
      - "cost tracking"
---

## AI Gateway Patterns

### What is AI Gateway?

AI Gateway is a unified API layer that:
- Routes requests to optimal AI providers
- Provides OIDC authentication
- Enables failover between providers
- Tracks costs per model/team
- Offers observability and rate limiting

### Basic Usage

```typescript
import { generateText } from 'ai';

// Simple routing through AI Gateway
const { text } = await generateText({
  model: 'openai/gpt-5.4',  // Routes through AI Gateway
  prompt: 'Hello, world!',
});

// Explicit gateway usage
import { gateway } from '@ai-sdk/gateway';

const { text } = await generateText({
  model: gateway('openai/gpt-5.4'),
  prompt: 'Hello, world!',
});
```

### Multi-Provider Routing

```typescript
// Route based on task type
const getModelForTask = (task: string): string => {
  switch (task) {
    case 'code':
      return 'anthropic/claude-sonnet-4.6';  // Better at code
    case 'creative':
      return 'openai/gpt-5.4';  // Better at creativity
    case 'fast':
      return 'google/gemini-2.5-flash';  // Fast and cheap
    default:
      return 'openai/gpt-5.4';
  }
};

async function routeRequest(task: string, prompt: string) {
  const model = getModelForTask(task);

  return generateText({
    model,
    prompt,
  });
}
```

### Failover Pattern

```typescript
import { generateText, AISDKError } from 'ai';

const providers = [
  'openai/gpt-5.4',
  'anthropic/claude-sonnet-4.6',
  'google/gemini-2.5-pro',
];

async function generateWithFailover(prompt: string): Promise<string> {
  let lastError: Error | null = null;

  for (const model of providers) {
    try {
      const { text } = await generateText({
        model,
        prompt,
        maxTokens: 1000,
      });
      return text;
    } catch (error) {
      lastError = error as Error;
      console.warn(`Provider ${model} failed:`, error);
      continue;
    }
  }

  throw new AI SDKError(`All providers failed. Last error: ${lastError}`);
}
```

### Cost Tracking

```typescript
// AI Gateway automatically tracks costs
// Query costs via API or dashboard

interface CostBreakdown {
  provider: string;
  model: string;
  totalRequests: number;
  totalTokens: number;
  costUSD: number;
}

// Usage analytics
async function getCostAnalytics(teamId: string): Promise<CostBreakdown[]> {
  const response = await fetch('https://gateway.vercel.ai/v1/analytics', {
    headers: {
      'Authorization': `Bearer ${process.env.GATEWAY_API_KEY}`,
      'X-Team-ID': teamId,
    },
  });

  return response.json();
}
```

### Rate Limiting

```typescript
// Configure per-model rate limits
const result = await generateText({
  model: 'openai/gpt-5.4',
  prompt: 'Hello!',
  maxTokens: 100,
  // AI Gateway handles rate limiting automatically
  // Returns 429 when limit exceeded
});
```

### Request Batching

```typescript
// Batch multiple requests for efficiency
import { generateText } from 'ai';

const prompts = [
  'Summarize this: Article 1...',
  'Summarize this: Article 2...',
  'Summarize this: Article 3...',
];

const results = await Promise.all(
  prompts.map(prompt =>
    generateText({
      model: 'openai/gpt-5.4',
      prompt,
    })
  )
);
```

### Caching Responses

```typescript
// Cache frequent queries
import { cachedGenerateText } from 'ai';

const cachedGenerate = cachedGenerateText({
  model: 'openai/gpt-5.4',
  cache: createRedisCache(),  // Your Redis instance
  ttl: 60 * 60,  // 1 hour
});

// Same prompt returns cached result
const result = await cachedGenerate({
  prompt: 'What is 2+2?',
});
```

### Observability

```typescript
// AI Gateway provides built-in observability
// Access traces in your dashboard or via API

interface Trace {
  id: string;
  provider: string;
  model: string;
  latency: number;
  tokensUsed: number;
  cost: number;
  cacheHit: boolean;
}

// Fetch recent traces
async function getRecentTraces(limit: number = 100): Promise<Trace[]> {
  const response = await fetch(
    `https://gateway.vercel.ai/v1/traces?limit=${limit}`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.GATEWAY_API_KEY}`,
      },
    }
  );
  return response.json();
}
```

### Best Practices

1. **Use model strings** - `provider/model` not provider SDK
2. **Configure failover** - Chain multiple providers
3. **Enable caching** - Reduce costs for repeated queries
4. **Track costs** - Monitor per-team/model usage
5. **Set rate limits** - Prevent runaway usage
6. **Use streaming** - Better UX for long responses
