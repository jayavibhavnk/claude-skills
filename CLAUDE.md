# SupaPowers - Claude Code Skill Library

A comprehensive skill library for Claude Code, built from the best plugins and designed for cross-collaboration.

## Overview

This repository contains a curated collection of 65+ skills that extend Claude Code's capabilities across:
- **Cloud & Deployment** - Vercel, serverless functions, edge computing
- **AI/ML** - Hugging Face, AI SDK, embeddings, agents, RAG
- **Game Dev** - React game loops, physics, UI/UX
- **Development** - Testing, debugging, refactoring, code review
- **Collaboration** - GitHub, GitLab, Slack, Discord, Telegram
- **Data** - Supabase, PostgreSQL, Prisma, data pipelines
- **Infrastructure** - Docker, MCP servers, security
- **Productivity** - Architecture, research, documentation

## Quick Start

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/claude-skills.git
cd claude-skills

# Browse available skills
ls skills/
```

## Skills Index (45 Skills)

### Cloud & Deployment (4)
- `vercel-deploy` - Deploy to Vercel with CI/CD pipelines
- `vercel-functions` - Build serverless functions and edge compute
- `vercel-ai-sdk` - AI SDK with modern AI Gateway patterns
- `vercel-storage` - Vercel Blob storage solutions

### AI/ML (11)
- `ai-gateway-patterns` - Multi-provider routing, failover, cost tracking
- `agentic-patterns` - Multi-agent systems, tool orchestration, memory
- `embeddings-rag` - RAG systems with vector search
- `function-calling-ai` - Tool definitions, Zod schemas, execution
- `streaming-ai-ui` - Real-time streaming, useChat v6 patterns
- `huggingface-inference` - HF model inference
- `image-generation-ai` - AI image generation with prompts
- `speech-to-text-ai` - Whisper transcription, voice commands
- `ai-evaluation` - Test sets, metrics, A/B testing
- `ai-agent-patterns` - AI agent architectures
- `real-time-collab` - WebSockets, presence, CRDTs

### Development Workflow (8)
- `systematic-debugging` - Scientific debugging method
- `test-driven-development` - TDD with modern frameworks
- `testing-best-practices` - AAA pattern, React Testing Library
- `code-refactoring` - SOLID principles, extract patterns
- `pr-code-review` - Constructive feedback, checklists
- `performance-optimization` - Core Web Vitals, bundle size
- `error-handling` - Typed errors, retry, circuit breaker
- `database-patterns` - Soft deletes, optimistic locking

### Collaboration (4)
- `github-actions` - GitHub Actions CI/CD workflows
- `gitlab-ci` - GitLab CI pipeline configuration
- `slack-integration` - Slack app development, Block Kit
- `atlassian-triage` - Jira/issue management, sprints

### Data & Databases (4)
- `supabase-basics` - PostgreSQL + Supabase
- `postgres-optimization` - Query optimization, indexes
- `data-pipelines` - ETL patterns, Airflow DAGs
- `graphql-patterns` - Schema design, resolvers, DataLoader

### Infrastructure (6)
- `mcp-server-build` - MCP server development
- `docker-compose-dev` - Container development environments
- `security-audit` - OWASP Top 10, vulnerability scanning
- `edge-computing` - Edge middleware, @upstash/redis, geolocation
- `workflow-orchestration` - Step functions, sagas, state machines
- `zero-downtime-deploy` - Blue-green, canary, feature flags

### Creative & Exploratory (8)
- `architecture-design` - System design patterns
- `tech-research` - Technology evaluation framework
- `api-design` - REST/GraphQL best practices
- `productivity-hacks` - Developer productivity tips
- `observability-debug` - Logging, metrics, tracing
- `event-driven-architecture` - Kafka, RabbitMQ, event sourcing
- `figma-integration` - Design-to-code extraction
- `monitoring-alerting` - Prometheus, Grafana, SLOs

## Skill Structure

Each skill follows this structure:

```
skills/
└── skill-name/
    └── SKILL.md   # Main skill file with frontmatter
```

### Frontmatter Fields

```yaml
---
name: skill-name
description: Brief description of what this skill covers
metadata:
  priority: 9        # 1-10, higher = more important
  docs:
    - "https://docs.url"
  pathPatterns:
    - "**/*.ts"     # File patterns that trigger this skill
  bashPatterns:
    - '\bnpm\b'     # Bash command patterns
  promptSignals:
    phrases:
      - "example"
    anyOf:
      - "trigger"
      - "words"
---
```

## Contributing

1. Create a new skill directory in `skills/`
2. Add `SKILL.md` with frontmatter and content
3. Follow the existing patterns
4. Commit with a clear message
5. Submit a PR for review

## Cross-Collaboration

Skills can chain to each other using implied dependencies:
- Vercel skills → AI SDK skills
- Testing skills → Code review skills
- Architecture skills → Implementation skills

## License

MIT
