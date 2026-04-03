---
name: mcp-server-build
description: Build Model Context Protocol (MCP) servers to extend Claude's capabilities with custom tools and resources.
metadata:
  priority: 9
  docs:
    - "https://modelcontextprotocol.io"
    - "https://github.com/modelcontextprotocol"
  pathPatterns:
    - "**/mcp/**"
    - "**/mcp-server/**"
  bashPatterns:
    - '\bmcp\b'
    - '\bmcp-server\b'
    - '\b@modelcontextprotocol\b'
  promptSignals:
    phrases:
      - "mcp server"
      - "model context protocol"
      - "custom tool"
    anyOf:
      - "mcp"
      - "tool"
      - "resource"
      - "prompt"
---

## MCP Server Development

### What is MCP?

The Model Context Protocol lets you build servers that provide:
- **Tools** - Functions Claude can call
- **Resources** - Data Claude can read
- **Prompts** - Reusable prompt templates

### Project Setup

```typescript
// TypeScript project
import { Server } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';
import {
  CallToolRequestSchema,
  ListResourcesRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types';

const server = new Server(
  { name: 'my-mcp-server', version: '1.0.0' },
  {
    capabilities: {
      tools: {},
      resources: {},
    },
  }
);
```

### Defining Tools

```typescript
// Tool that searches the web
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === 'web_search') {
    const { query } = args as { query: string };
    const results = await searchWeb(query);

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(results, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});
```

### Defining Resources

```typescript
// Resource that provides documentation
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: 'docs://api-reference',
        name: 'API Reference',
        description: 'Complete API documentation',
        mimeType: 'text/markdown',
      },
    ],
  };
});

// Resource content handler
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;

  if (uri === 'docs://api-reference') {
    return {
      contents: [
        {
          uri,
          mimeType: 'text/markdown',
          text: await getApiDocs(),
        },
      ],
    };
  }

  throw new Error(`Unknown resource: ${uri}`);
});
```

### Complete Example

```typescript
import { Server } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types';

class MyServer {
  private server: Server;

  constructor() {
    this.server = new Server(
      { name: 'example-server', version: '1.0.0' },
      { capabilities: { tools: {} } }
    );

    this.setupHandlers();
  }

  private setupHandlers() {
    this.server.setRequestHandler(
      ListToolsRequestSchema,
      async () => ({
        tools: [
          {
            name: 'get_weather',
            description: 'Get weather for a location',
            inputSchema: {
              type: 'object',
              properties: {
                location: {
                  type: 'string',
                  description: 'City name',
                },
              },
              required: ['location'],
            },
          },
        ],
      })
    );

    this.server.setRequestHandler(
      CallToolRequestSchema,
      async (request) => {
        const { name, arguments: args } = request.params;

        if (name === 'get_weather') {
          const weather = await fetchWeather(args.location);
          return {
            content: [{ type: 'text', text: JSON.stringify(weather) }],
          };
        }

        throw new Error('Unknown tool');
      }
    );
  }

  async start() {
    const transport = new StdioServerTransport();
    await this.server.connect(transport);
    console.error('MCP Server running on stdio');
  }
}

new MyServer().start();
```

### Transport Options

| Transport | Use Case |
|-----------|----------|
| `StdioServerTransport` | CLI tools, local scripts |
| `HTTPStreamTransport` | Web services |
| `SSETransport` | Server-Sent Events |

### Best Practices

1. Use TypeScript for type safety
2. Validate tool arguments with Zod
3. Handle errors gracefully with descriptive messages
4. Document tools clearly with descriptions
5. Use streaming for large responses
6. Test with the MCP inspector

### Testing

```typescript
import { MCPInspector } from '@modelcontextprotocol/inspector';

// Test your server
const inspector = new MCPInspector(new MyServer());
await inspector.run();
```

### Publishing

```json
// package.json
{
  "name": "mcp-server-example",
  "exports": "./dist/index.js",
  "bin": {
    "mcp-server-example": "./dist/index.js"
  }
}
```
