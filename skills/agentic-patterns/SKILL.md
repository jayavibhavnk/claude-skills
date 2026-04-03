---
name: agentic-patterns
description: Design autonomous AI agent architectures - multi-agent systems, tool orchestration, and agent loops.
metadata:
  priority: 10
  docs:
    - "https://sdk.vercel.ai/docs/agents"
  pathPatterns:
    - "**/agent/**"
    - "**/agents/**"
  bashPatterns:
    - '\bAgent\b'
    - '\bagentic\b'
    - '\bToolLoopAgent\b'
  promptSignals:
    phrases:
      - "agent"
      - "agentic"
      - "autonomous"
    anyOf:
      - "multi-agent"
      - "tool orchestration"
      - "agent loop"
---

## Agentic AI Patterns

### What Makes an Agent "Agentic"?

1. **Autonomy** - Acts without continuous human input
2. **Memory** - Maintains context across interactions
3. **Tools** - Can call external functions/APIs
4. **Planning** - Breaks down complex tasks
5. **Reflection** - Evaluates its own outputs

### Single Agent Architecture

```typescript
import { Agent } from 'ai';
import { openai } from '@ai-sdk/openai';
import { tool } from 'ai';
import { z } from 'zod';

const researchAgent = new Agent({
  name: 'research-assistant',
  model: 'openai/gpt-5.4',
  description: 'Researches topics thoroughly on the web',
  tools: [
    webSearchTool,
    webPageFetchTool,
    saveToFileTool,
  ],
  instructions: `You are a research assistant. For each topic:
1. Search for relevant information
2. Read key pages for details
3. Synthesize findings into a summary
4. Save key points to the research file`,
});
```

### Multi-Agent Systems

```typescript
// Supervisor agent coordinates sub-agents
const supervisorAgent = new Agent({
  name: 'supervisor',
  model: 'openai/gpt-5.4',
  tools: [researchAgent, writerAgent, editorAgent],
  instructions: `You coordinate a content team:
- researchAgent: Gathers information on a topic
- writerAgent: Creates initial drafts
- editorAgent: Reviews and improves content

Delegate tasks based on agent capabilities.
Ensure all work meets quality standards.`,
});

// Research sub-agent
const researchAgent = new Agent({
  name: 'researcher',
  model: 'openai/gpt-5.4',
  tools: [webSearchTool],
  instructions: `You research topics thoroughly.
Always cite your sources.
Focus on accurate, up-to-date information.`,
});

// Writer sub-agent
const writerAgent = new Agent({
  name: 'writer',
  model: 'openai/gpt-5.4',
  tools: [],
  instructions: `You write clear, engaging content.
Follow the style guide provided.
Meet all requirements from the brief.`,
});
```

### Tool Orchestration Patterns

```typescript
// Sequential execution
async function sequentialResearch(topic: string) {
  const searchResults = await webSearch({ query: topic });
  const topResult = await webPageFetch({ url: searchResults[0].url });
  const summary = await summarize({ text: topResult.content });
  return summary;
}

// Parallel execution
async function parallelResearch(topic: string) {
  const [searchResults, trending, related] = await Promise.all([
    webSearch({ query: topic }),
    webSearch({ query: `${topic} trending` }),
    webSearch({ query: `${topic} related topics` }),
  ]);
  return { searchResults, trending, related };
}

// Hierarchical (fan-out/fan-in)
async function hierarchicalResearch(topic: string) {
  // Fan out: Search multiple sources in parallel
  const sources = await Promise.all([
    webSearch({ query: `${topic} academic` }),
    webSearch({ query: `${topic} news` }),
    webSearch({ query: `${topic} practical guide` }),
  ]);

  // Fan in: Synthesize results
  const synthesis = await synthesize({
    inputs: sources.flatMap(s => s.results),
  });
  return synthesis;
}
```

### Memory Patterns

```typescript
// Short-term memory (conversation context)
const agent = new Agent({
  model: 'openai/gpt-5.4',
  tools: [],
  memory: {
    type: 'short-term',
    maxTokens: 10000,
  },
});

// Long-term memory (persistent across sessions)
const agent = new Agent({
  model: 'openai/gpt-5.4',
  tools: [],
  memory: {
    type: 'vector',
    store: 'embeddings-store',
    retrieval: {
      topK: 5,
      similarityThreshold: 0.8,
    },
  },
});

// Hybrid memory
const agent = new Agent({
  model: 'openai/gpt-5.4',
  memory: {
    shortTerm: { maxTokens: 10000 },
    longTerm: { store: 'embeddings-store' },
  },
});
```

### Planning & Reasoning

```typescript
// Chain of thought
const result = await generateText({
  model: 'openai/gpt-5.4',
  prompt: `Think step by step:
Problem: ${problem}

1. First, I need to understand...
2. Then, I should...
3. Finally,...

Let's work through this...`,
});

// Tree of thoughts (explore multiple paths)
const results = await Promise.all([
  approachA(),
  approachB(),
  approachC(),
]);
const best = await selectBest(results);
```

### Error Handling & Recovery

```typescript
const agent = new Agent({
  model: 'openai/gpt-5.4',
  maxRetries: 3,
  retryDelay: 1000,
  onError: (error) => {
    console.error('Agent error:', error);
    return { success: false, message: 'Retrying...' };
  },
  validation: (output) => {
    // Validate agent output
    return isValidFormat(output);
  },
});
```

### Monitoring Agents

```typescript
interface AgentEvent {
  timestamp: Date;
  agent: string;
  action: string;
  input: any;
  output: any;
  duration: number;
}

// Log all agent invocations
async function logAgentEvent(event: AgentEvent) {
  await db.agentEvents.create({
    data: {
      ...event,
      metadata: JSON.stringify(event),
    },
  });
}
```

### Best Practices

1. **Start simple** - Single agent, few tools
2. **Clear instructions** - Role, goals, constraints
3. **Tool design** - Minimal, focused, well-documented
4. **Error handling** - Plan for failures
5. **Observability** - Log all agent actions
6. **Iteration** - Build up complexity gradually
