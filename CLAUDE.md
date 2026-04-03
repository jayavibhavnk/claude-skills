# SupaPowers - Claude Code Skill Library

A comprehensive skill library for Claude Code, built from the best plugins and designed for cross-collaboration.

## Overview

This repository contains a curated collection of skills that extend Claude Code's capabilities across:
- **Cloud & Deployment** - Vercel, serverless functions, edge computing
- **AI/ML** - Hugging Face, AI SDK, embeddings, agents
- **Development** - Testing, debugging, refactoring, code review
- **Collaboration** - GitHub, GitLab, Slack, Atlassian
- **Data** - Supabase, data pipelines, ETL
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

## Skills Index

### Cloud & Deployment
- `vercel-deploy` - Deploy to Vercel with CI/CD pipelines
- `vercel-functions` - Build serverless functions and edge compute
- `vercel-storage` - Work with Vercel Blob and storage solutions
- `vercel-ai-sdk` - Build AI-powered features with Vercel AI SDK

### AI/ML
- `huggingface-inference` - Run inference with Hugging Face models
- `huggingface-fine-tuning` - Fine-tune models on custom data
- `ai-agent-patterns` - Design and implement AI agent architectures
- `embeddings-rag` - Build RAG systems with embeddings

### Development Workflow
- `systematic-debugging` - Debug issues methodically
- `test-driven-development` - TDD with modern frameworks
- `code-refactoring` - Refactor and improve existing code
- `pr-code-review` - Review pull requests thoroughly

### Collaboration
- `github-actions` - Build GitHub Actions workflows
- `gitlab-ci` - Configure GitLab CI/CD pipelines
- `slack-integration` - Build Slack apps and integrations
- `atlassian-triage` - Triage Jira/Confluence issues

### Data & Databases
- `supabase-basics` - Supabase setup and patterns
- `data-pipelines` - Build ETL/data pipelines
- `postgres-optimization` - Optimize PostgreSQL queries

### Infrastructure
- `mcp-server-build` - Build MCP servers
- `docker-compose-dev` - Docker development workflows
- `security-audit` - Security best practices audit

### Creative & Exploratory
- `architecture-design` - Design system architectures
- `tech-research` - Research new technologies
- `documentation-write` - Write great documentation
- `api-design` - Design RESTful and GraphQL APIs

## Contributing

1. Create a new skill directory in `skills/`
2. Add `SKILL.md` with frontmatter and content
3. Add `references/` directory if needed
4. Commit with a clear message
5. Submit a PR for review

## Skill Structure

Each skill follows this structure:

```
skills/
└── skill-name/
    ├── SKILL.md          # Main skill file
    └── references/       # Optional reference docs
        ├── ref1.md
        └── ref2.md
```

## Cross-Collaboration

Skills can chain to each other using the `chainTo` frontmatter field.
This allows skills to seamlessly hand off to related skills.

## License

MIT
