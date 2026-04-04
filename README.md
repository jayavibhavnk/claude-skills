# SupaPowers - Claude Code Skill Library

<p align="center">
  <img src="https://img.shields.io/badge/Claude-Code-blue?style=for-the-badge" alt="Claude Code">
  <img src="https://img.shields.io/badge/Skills-60+-green?style=for-the-badge" alt="Skills Count">
  <img src="https://img.shields.io/badge/License-MIT-orange?style=for-the-badge" alt="License">
</p>

> A comprehensive, production-ready skill library for Claude Code that transforms you into a full-stack superpower. Built from industry best practices and designed for real-world development workflows.

## Features

- **60+ Production Skills** - Covering AI/ML, cloud deployment, game dev, testing, and more
- **Cross-Collaborative** - Skills reference each other for complex workflows
- **Modern Patterns** - AI SDK v6, Next.js 14, Vercel Fluid Compute, and more
- **Battle-Tested** - Real code patterns from production applications
- **Auto-Activated** - Skills activate based on file patterns and context

## Quick Start

```bash
# Clone the repository
git clone https://github.com/jayavibhavnk/claude-skills.git
cd claude-skills

# List all available skills
ls skills/

# View a specific skill
cat skills/vercel-ai-sdk/SKILL.md
```

## Skills Catalog

### Cloud & Deployment (6)

| Skill | Description |
|-------|-------------|
| `vercel-deploy` | Vercel deployments, CI/CD, preview URLs |
| `vercel-functions` | Serverless functions, Fluid Compute |
| `vercel-ai-sdk` | AI SDK v6 with AI Gateway patterns |
| `vercel-storage` | Blob storage, signed URLs |
| `edge-computing` | Edge middleware, Upstash Redis, geolocation |
| `zero-downtime-deploy` | Blue-green, canary, feature flags |

### AI/ML (11)

| Skill | Description |
|-------|-------------|
| `ai-gateway-patterns` | Multi-provider routing, failover, cost tracking |
| `agentic-patterns` | Multi-agent systems, tool orchestration |
| `embeddings-rag` | Vector search, embeddings, RAG systems |
| `function-calling-ai` | Tool definitions, Zod schemas |
| `streaming-ai-ui` | Real-time streaming, useChat v6 |
| `huggingface-inference` | HF model inference, endpoints |
| `image-generation-ai` | AI image generation, DALL-E, Stable Diffusion |
| `speech-to-text-ai` | Whisper transcription, voice commands |
| `ai-evaluation` | Test sets, metrics, A/B testing |
| `ai-agent-patterns` | AI agent architectures, memory |
| `real-time-collab` | WebSockets, presence, CRDTs |

### Game Development (3)

| Skill | Description |
|-------|-------------|
| `react-game-dev` | Game loops, ECS, collision detection |
| `game-ui-ux` | HUD design, menus, accessibility |
| `game-physics` | Vectors, gravity, projectiles, springs |

### Development Workflow (12)

| Skill | Description |
|-------|-------------|
| `systematic-debugging` | Scientific debugging method |
| `test-driven-development` | TDD with modern frameworks |
| `testing-best-practices` | AAA pattern, React Testing Library |
| `vitest-setup` | Vitest configuration and patterns |
| `contract-testing` | Pact, consumer-driven contracts |
| `playwright-e2e` | Browser automation, visual regression |
| `code-refactoring` | SOLID principles, extract patterns |
| `pr-code-review` | Constructive feedback, checklists |
| `coderabbit-review` | CodeRabbit AI review patterns |
| `performance-optimization` | Core Web Vitals, bundle size |
| `error-handling` | Typed errors, retry, circuit breaker |
| `database-patterns` | Soft deletes, optimistic locking |

### Frontend & Design (7)

| Skill | Description |
|-------|-------------|
| `react-patterns` | Advanced React patterns, hooks |
| `nextjs-advanced` | Server Components, Streaming, Parallel Routes |
| `frontend-design` | Component architecture, design systems |
| `css-architecture` | CSS Modules, Tailwind, architecture |
| `micro-frontends` | Module Federation, shared state |
| `api-versioning` | URL path, header, query param strategies |
| `chrome-devtools` | Debugging, performance profiling |

### Backend & APIs (6)

| Skill | Description |
|-------|-------------|
| `api-design` | REST/GraphQL best practices |
| `graphql-patterns` | Schema design, resolvers, DataLoader |
| `auth-implementation` | JWT, OAuth, session management |
| `webhook-handling` | Webhook verification, retries |
| `api-rate-limiting` | Rate limiting strategies |
| `api-versioning` | Version migration patterns |

### Data & Databases (5)

| Skill | Description |
|-------|-------------|
| `supabase-basics` | PostgreSQL + Supabase |
| `supabase-advanced` | RLS, Edge Functions, Realtime |
| `postgres-optimization` | Query optimization, indexes |
| `prisma-patterns` | Prisma ORM, relations, transactions |
| `data-pipelines` | ETL patterns, Airflow DAGs |

### Collaboration & DevOps (6)

| Skill | Description |
|-------|-------------|
| `github-actions` | CI/CD workflows |
| `github-actions-patterns` | Matrix builds, caching, reusable workflows |
| `gitlab-ci` | GitLab CI pipeline |
| `slack-integration` | Slack app, Block Kit |
| `discord-bots` | Discord.js, slash commands |
| `telegram-bots` | grammY framework |

### Infrastructure (6)

| Skill | Description |
|-------|-------------|
| `mcp-server-build` | MCP server development |
| `docker-compose-dev` | Container development |
| `security-audit` | OWASP Top 10, vulnerability scanning |
| `workflow-orchestration` | Step functions, sagas |
| `observability-debug` | Logging, metrics, tracing |
| `monitoring-alerting` | Prometheus, Grafana, SLOs |

### Integrations (8)

| Skill | Description |
|-------|-------------|
| `atlassian-triage` | Jira, issue management |
| `figma-integration` | Design-to-code extraction |
| `firecrawl-scraping` | Web scraping, content extraction |
| `hookify-rules` | Claude Code hooks |
| `claude-code-tips` | Productivity tips |
| `serena` | Serena code exploration |
| `greptile` | Code search patterns |
| `code-review` | Review patterns |

### Creative & Exploratory (6)

| Skill | Description |
|-------|-------------|
| `architecture-design` | System design patterns |
| `tech-research` | Technology evaluation |
| `productivity-hacks` | Developer tips |
| `event-driven-architecture` | Kafka, RabbitMQ, CQRS |
| `event-sourcing` | Event sourcing patterns |
| `ddd-patterns` | Domain-driven design |

## Skill Structure

Each skill is a standalone markdown file with YAML frontmatter:

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

## Skill Content

Code examples, patterns, and best practices...
```

## Cross-Collaboration

Skills are designed to work together. For example:

- `vercel-deploy` → `vercel-ai-sdk` → `streaming-ai-ui`
- `testing-best-practices` → `playwright-e2e` → `pr-code-review`
- `postgres-optimization` → `prisma-patterns` → `supabase-advanced`

## Contributing

1. Fork the repository
2. Create a new skill directory: `skills/your-skill/`
3. Add `SKILL.md` with frontmatter and content
4. Follow existing patterns and conventions
5. Submit a pull request

## License

MIT - Use freely, contribute generously.

---

<p align="center">
  Built with Claude Code • 60+ skills • For developers who ship
</p>
