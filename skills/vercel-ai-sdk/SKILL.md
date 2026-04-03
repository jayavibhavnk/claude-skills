---
name: vercel-ai-sdk
description: Build AI-powered features with Vercel AI SDK - chat interfaces, text generation, tool calling, agents, and streaming using AI Gateway for routing.
metadata:
  priority: 9
  docs:
    - "https://sdk.vercel.ai/docs"
    - "https://sdk.vercel.ai/docs/reference"
  pathPatterns:
    - "**/ai/**"
    - "**/lib/ai/**"
    - "app/api/chat/**"
    - "lib/agent/**"
  bashPatterns:
    - '\bai-sdk\b'
    - '\b@ai-sdk\b'
    - '\bgenerateText\b'
    - '\bstreamText\b'
  promptSignals:
    phrases:
      - "ai sdk"
      - "vercel ai"
      - "usechat"
    anyOf:
      - "chat interface"
      - "streaming"
      - "tool calling"
      - "llm"
      - "language model"
---

## Vercel AI SDK

### Core Concepts

1. **AI Gateway** - Unified API for all providers with OIDC auth, failover, cost tracking
2. **Providers** - OpenAI, Anthropic, Google via AI Gateway
3. **Models** - Use `provider/model` format (e.g., `openai/gpt-5.4`)
4. **Tools** - Function calling with typed schemas using `inputSchema`
5. **Agents** - `ToolLoopAgent` for autonomous problem-solving
6. **Streaming** - Real-time responses with proper UI integration

### AI Gateway Pattern (Recommended)

```typescript
import { generateText } from 'ai';

// Use plain model strings with AI Gateway
const { text } = await generateText({
  model: 'openai/gpt-5.4',  // Routes through AI Gateway
  prompt: 'Write a haiku about programming',
});

// Or explicit gateway
import { gateway } from '@ai-sdk/gateway';

const { text } = await generateText({
  model: gateway('openai/gpt-5.4'),
  prompt: 'Write a haiku about programming',
});
```

### Streaming Responses

```typescript
import { streamText } from 'ai';

const result = streamText({
  model: 'anthropic/claude-sonnet-4.6',
  prompt: 'Tell me a story',
});

for await (const chunk of result.fullStream) {
  console.log(chunk.text);
}
```

### Tool Calling

```typescript
import { generateText, tool } from 'ai';
import { z } from 'zod';

const { text } = await generateText({
  model: 'openai/gpt-5.4',
  tools: {
    weather: tool({
      description: 'Get weather for a location',
      inputSchema: z.object({
        location: z.string(),
      }),
      execute: async ({ location }) => {
        const response = await fetch(
          `https://api.weather.com/?q=${location}`
        );
        return { temp: 72, condition: 'sunny' };
      },
    }),
  },
  prompt: 'What is the weather in San Francisco?',
});
```

### Building Agents

```typescript
import { ToolLoopAgent } from 'ai';

const agent = new ToolLoopAgent({
  name: 'research-assistant',
  model: 'openai/gpt-5.4',
  tools: [searchTool, calculatorTool],
});

const result = await agent.generate(
  'Research the latest AI breakthroughs'
);
```

### Chat UI (React)

```typescript
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from '@ai-sdk/react';

function Chat() {
  const { messages, sendMessage } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });

  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          {m.role}: {m.content}
        </div>
      ))}
      <button onClick={() => sendMessage({ text: 'Hello' })}>
        Send
      </button>
    </div>
  );
}
```

### Embeddings

```typescript
import { embed, embedMany } from 'ai';

const { embedding } = await embed({
  model: 'openai/text-embedding-3-small',
  value: 'Hello, world!',
});

// Batch embeddings
const { embeddings } = await embedMany({
  model: 'openai/text-embedding-3-small',
  values: ['Hello', 'World'],
});
```

## Best Practices

1. Use AI Gateway (`provider/model` strings) instead of direct provider SDK
2. Always provide typed schemas for tools using `inputSchema` (not `parameters`)
3. Use streaming for better UX with `toUIMessageStreamResponse()`
4. Use `Output.object()` for structured output (not `generateObject`)
5. Implement proper error handling with descriptive messages
6. Set appropriate `maxTokens` to prevent runaway responses
