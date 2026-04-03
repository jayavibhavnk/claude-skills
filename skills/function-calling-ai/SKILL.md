---
name: function-calling-ai
description: Implement function calling with AI models - tool definitions, schema validation, and orchestration.
metadata:
  priority: 9
  docs:
    - "https://sdk.vercel.ai/docs/ai-sdk-core/tools-and-tool-calling"
  pathPatterns:
    - "**/tools/**"
    - "**/function/**"
  bashPatterns:
    - '\btool\b'
    - '\bfunction.?call'
    - '\bz\.object\b'
  promptSignals:
    phrases:
      - "function calling"
      - "tool calling"
      - "tools"
    anyOf:
      - "tool"
      - "schema"
      - "execute"
---

## Function Calling with AI

### What is Function Calling?

Function calling lets AI models invoke external tools/functions when they need to perform actions beyond text generation.

```
User: "What's the weather in San Francisco?"

AI: I should check the weather tool.
    tool_call({
      name: "get_weather",
      arguments: { location: "San Francisco" }
    })

Tool: Returns { temp: 72, condition: "sunny" }

AI: The weather in San Francisco is sunny with 72°F.
```

### Tool Definition with Zod

```typescript
import { tool } from 'ai';
import { z } from 'zod';

const getWeather = tool({
  description: 'Get current weather for a location',
  parameters: z.object({
    location: z.string().describe('City name, e.g., "San Francisco"'),
    unit: z.enum(['celsius', 'fahrenheit']).optional().default('fahrenheit'),
  }),
});

const searchFlights = tool({
  description: 'Search for flights between airports',
  parameters: z.object({
    origin: z.string().describe('Origin airport code (IATA)'),
    destination: z.string().describe('Destination airport code (IATA)'),
    date: z.string().describe('Departure date (YYYY-MM-DD)'),
    passengers: z.number().int().min(1).max(9).optional().default(1),
  }),
});

const bookFlight = tool({
  description: 'Book a specific flight',
  parameters: z.object({
    flightId: z.string(),
    passengerDetails: z.object({
      firstName: z.string(),
      lastName: z.string(),
      email: z.string().email(),
    }),
  }),
});
```

### Using Tools with generateText

```typescript
import { generateText, tool } from 'ai';
import { z } from 'zod';

const { text, toolCalls, toolResults, finishReason } = await generateText({
  model: 'openai/gpt-5.4',
  tools: [getWeather, searchFlights, bookFlight],
  maxToolRoundtrips: 5,
  system: `You are a travel assistant. Help users with:
1. Checking weather at destinations
2. Searching and booking flights

Always be helpful and provide accurate information.`,
  prompt: 'I want to fly from NYC to LA on March 15th. What flights are available?',
});

// Handle tool calls
for (const toolCall of toolCalls) {
  console.log(`Tool called: ${toolCall.toolName}`);
  console.log(`Arguments:`, toolCall.args);
}

// Access tool results
for (const result of toolResults) {
  console.log(`Result from ${result.toolName}:`, result.result);
}
```

### Tool Execution

```typescript
// Define tool with execution logic
const getWeather = tool({
  description: 'Get weather for a location',
  parameters: z.object({
    location: z.string(),
    unit: z.enum(['celsius', 'fahrenheit']).optional().default('fahrenheit'),
  }),
  execute: async ({ location, unit }) => {
    // This runs when the model calls this tool
    const response = await fetch(
      `https://api.weather.com?location=${encodeURIComponent(location)}`
    );
    const data = await response.json();

    return {
      location,
      temperature: data.temp,
      condition: data.condition,
      humidity: data.humidity,
      unit,
    };
  },
});

// execute is optional - you can handle tools separately
const { toolCalls } = await generateText({
  model: 'openai/gpt-5.4',
  tools: [getWeather], // No execute - we'll handle manually
});

// Manual tool execution
for (const toolCall of toolCalls) {
  switch (toolCall.toolName) {
    case 'getWeather':
      const weather = await fetchWeather(toolCall.args.location);
      // Add result to continue conversation
      continue;
  }
}
```

### Multiple Tool Rounds

```typescript
// Model can call multiple tools in sequence
const { text, toolCalls, toolResults } = await generateText({
  model: 'openai/gpt-5.4',
  tools: [getWeather, searchFlights, bookFlight],
  maxToolRoundtrips: 10, // Allow back-and-forth
  prompt: `Help me plan a trip:
1. First check the weather in Tokyo for next week
2. Then find flights from NYC to Tokyo
3. Book the cheapest flight`,
});

// The model will:
 // 1. Call getWeather for Tokyo
// 2. Get result
// 3. Call searchFlights based on weather
// 4. Get results
// 5. Call bookFlight or summarize options
```

### Tool Choice Control

```typescript
// Force the model to use a specific tool
const { text } = await generateText({
  model: 'openai/gpt-5.4',
  tools: [getWeather, searchFlights],
  toolChoice: {
    // Force use of a specific tool
    toolName: 'getWeather',
  },
  prompt: 'What is 2+2?', // Model will still call getWeather
});

// Let model decide (default)
const { text } = await generateText({
  model: 'openai/gpt-5.4',
  tools: [getWeather, searchFlights],
  toolChoice: 'auto', // Model decides
});

// Require any tool (no text-only response)
const { text } = await generateText({
  model: 'openai/gpt-5.4',
  tools: [getWeather, searchFlights],
  toolChoice: 'required',
  prompt: 'What is 2+2?', // Must call a tool
});
```

### Streaming with Tools

```typescript
import { streamText } from 'ai';

const result = streamText({
  model: 'openai/gpt-5.4',
  tools: [getWeather],
  prompt: 'Weather in Tokyo?',
});

for await (const part of result.fullStream) {
  if (part.type === 'tool-call') {
    console.log('Tool call:', part.toolName, part.args);
  } else if (part.type === 'tool-result') {
    console.log('Tool result:', part.result);
  } else if (part.type === 'text') {
    process.stdout.write(part.text);
  }
}
```

### Tool Error Handling

```typescript
const getWeather = tool({
  description: 'Get weather for a location',
  parameters: z.object({
    location: z.string(),
  }),
  execute: async ({ location }) => {
    try {
      const response = await fetchWeather(location);
      return response;
    } catch (error) {
      // Return error info - model will handle gracefully
      return {
        error: true,
        message: `Failed to fetch weather: ${error.message}`,
        location,
      };
    }
  },
});
```

### Best Practices

1. **Clear descriptions** - Model uses description to decide when to call
2. **Minimal parameters** - Only include necessary fields
3. **Type safety** - Use Zod for validation
4. **Handle errors** - Return error info, don't throw
5. **Limit tool rounds** - Set `maxToolRoundtrips` to prevent infinite loops
6. **Idempotency** - Tools should be safe to call multiple times
