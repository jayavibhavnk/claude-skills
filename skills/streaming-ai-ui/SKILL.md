---
name: streaming-ai-ui
description: Build streaming AI user interfaces - real-time responses, partial states, and smooth UX patterns.
metadata:
  priority: 9
  docs:
    - "https://sdk.vercel.ai/docs/ai-sdk-rsc"
  pathPatterns:
    - "**/chat/**"
    - "**/messages/**"
  bashPatterns:
    - '\bstreamText\b'
    - '\buseChat\b'
    - '\bstreaming\b'
  promptSignals:
    phrases:
      - "streaming"
      - "real-time"
      - "chat ui"
    anyOf:
      - "streaming"
      - "useChat"
      - "partial"
---

## Streaming AI UI

### Core Streaming Pattern

```typescript
// Server component (App Router)
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: 'openai/gpt-5.4',
    system: 'You are a helpful assistant.',
    messages,
  });

  return toUIMessageStreamResponse(result);
}
```

### useChat Hook (React)

```typescript
'use client';

import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from '@ai-sdk/react';

function Chat() {
  const { messages, sendMessage, isLoading, setInput } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });

  return (
    <div>
      {/* Message list */}
      <div className="space-y-4">
        {messages.map((message) => (
          <div key={message.id}>
            <div className="font-bold">{message.role}</div>
            <div>{message.content}</div>
          </div>
        ))}

        {/* Loading indicator */}
        {isLoading && (
          <div className="text-gray-500">Thinking...</div>
        )}
      </div>

      {/* Input */}
      <form onSubmit={(e) => {
        e.preventDefault();
        sendMessage({ text: input });
      }}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type a message..."
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

### Streaming with Tools

```typescript
'use client';

import { useChat } from '@ai-sdk/react';

function ChatWithTools() {
  const { messages, sendMessage, toolCalls, toolResults, isLoading } = useChat({
    api: '/api/chat',
    tools: {
      getWeather: {
        description: 'Get weather for a location',
        inputSchema: z.object({
          location: z.string(),
        }),
      },
    },
  });

  return (
    <div>
      {messages.map((message) => (
        <Message key={message.id} message={message} />
      ))}

      {/* Show tool calls in progress */}
      {toolCalls.map((call) => (
        <ToolCallBadge key={call.id} toolName={call.toolName} />
      ))}

      {/* Show tool results */}
      {toolResults.map((result) => (
        <ToolResult key={result.toolCallId} result={result} />
      ))}
    </div>
  );
}
```

### Message Parts (Modern)

```typescript
// AI SDK v6 uses message.parts
messages.map((message) => (
  <div key={message.id}>
    {message.role === 'user' ? (
      <UserMessage content={message.content} />
    ) : (
      message.parts.map((part, i) => {
        switch (part.type) {
          case 'text':
            return <TextMessage key={i} text={part.text} />;
          case 'tool-call':
            return <ToolCallMessage key={i} toolCall={part} />;
          case 'tool-result':
            return <ToolResultMessage key={i} result={part} />;
        }
      })
    )}
  </div>
))
```

### Smooth Streaming UX

```typescript
// CSS for smooth streaming
.message-content {
  overflow-wrap: break-word;
  word-break: break-word;
}

.message-streaming {
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

/* Show cursor while streaming */
.streaming-cursor::after {
  content: '|';
  animation: blink 0.8s infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}
```

### Thought Streaming

```typescript
// Show model thinking/thoughts while streaming
function Chat() {
  const { messages, sendMessage, status } = useChat({
    transport: new DefaultChatTransport({ api: '/api/chat' }),
  });
  const isLoading = status === 'streaming' || status === 'submitted';

  return (
    <div>
      {messages.map((message) => (
        <div key={message.id}>
          {message.role === 'assistant' && message.reasoning && (
            <div className="text-gray-500 text-sm">
              <ThoughtBubble>{message.reasoning}</ThoughtBubble>
            </div>
          )}
          <div>{message.content}</div>
        </div>
      ))}
    </div>
  );
}
```

### Optimistic Updates

```typescript
function Chat() {
  const { messages, sendMessage } = useChat({
    api: '/api/chat',
    onFinish: (message) => {
      // Analytics, scroll to bottom, etc.
      analytics.track('message_sent', {
        length: message.content.length,
      });
    },
  });

  // Optimistic UI pattern
  const handleSubmit = async (text: string) => {
    // Immediately show user message
    setMessages((prev) => [...prev, {
      id: 'temp',
      role: 'user',
      content: text,
    }]);

    // Send to server
    await sendMessage({ text });
  };
}
```

### Error States

```typescript
function Chat() {
  const { messages, sendMessage, error } = useChat({
    transport: new DefaultChatTransport({ api: '/api/chat' }),
  });

  return (
    <div>
      {error && (
        <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
          <p>Something went wrong. Please try again.</p>
          <button onClick={() => sendMessage({ text: messages[messages.length - 1].content })}>
            Retry
          </button>
        </div>
      )}
      {/* ... */}
    </div>
  );
}
```

### Rate Limiting UI

```typescript
function Chat() {
  const [remaining, setRemaining] = useState<number | null>(null);

  const { messages, sendMessage } = useChat({
    api: '/api/chat',
    onRateLimit: (retryAfter) => {
      setRemaining(retryAfter);
      const interval = setInterval(() => {
        setRemaining((r) => {
          if (r === null || r <= 0) {
            clearInterval(interval);
            return null;
          }
          return r - 1;
        });
      }, 1000);
    },
  });

  return (
    <div>
      {remaining !== null && (
        <div className="text-yellow-600">
          Rate limited. Try again in {remaining}s
        </div>
      )}
      {/* ... */}
    </div>
  );
}
```

### Best Practices

1. **Stream immediately** - Don't wait for complete response
2. **Show loading state** - User feedback is crucial
3. **Handle errors gracefully** - Retry options
4. **Implement rate limit UI** - Don't leave user confused
5. **Use message parts** - For tool calls, images, etc.
6. **Scroll to bottom** - Auto-scroll on new content
7. **Copy/message actions** - Useful features for AI content
