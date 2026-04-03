---
name: vercel-deploy
description: Deploy applications to Vercel with optimized CI/CD pipelines, preview deployments, and production rollbacks.
metadata:
  priority: 9
  docs:
    - "https://vercel.com/docs/deployments/overview"
    - "https://vercel.com/docs/projects/overview"
  pathPatterns:
    - "vercel.json"
    - "vercel.ts"
    - ".vercel/**"
    - "api/**"
    - "app/**"
  bashPatterns:
    - '\bvercel\b'
    - '\bvercel\s+deploy\b'
    - '\bvercel\s+prod\b'
    - '\bvercel\s+pull\b'
  promptSignals:
    phrases:
      - "vercel deploy"
      - "deploy to vercel"
      - "preview deployment"
    anyOf:
      - "production"
      - "rollback"
      - "ci/cd"
      - "github integration"
---

## Deploying to Vercel

### Pre-Deployment Checklist

Before deploying, ensure:
1. Project has a valid `vercel.json` or `vercel.ts` config
2. All environment variables are configured
3. Build command succeeds locally (`vercel build`)
4. Framework is auto-detected or explicitly configured

### Deployment Commands

```bash
# Preview deployment (creates unique URL)
vercel

# Production deployment
vercel --prod

# Deploy specific directory
vercel ./path/to/project

# With environment variables
vercel --env NODE_ENV=production
```

### CI/CD Pipeline Setup

For automated deployments:

1. Connect GitHub repository in Vercel dashboard
2. Configure branch protection rules
3. Set up preview deployments for PRs
4. Configure production branch (usually `main` or `master`)

### Environment Variables

```bash
# Pull environment variables from Vercel
vercel env pull .env.local

# Add environment variable
vercel env add NEXT_PUBLIC_API_URL

# List environment variables
vercel env list
```

### Monitoring Deployments

```bash
# View deployment logs
vercel logs my-project

# Check deployment status
vercel ls
```

### Rollback Procedures

```bash
# List recent deployments
vercel ls

# Rollback to previous deployment
vercel rollback [deployment-url]
```

### Build Optimization

- Use `vercel.json` to configure build commands
- Set `rewrites` for API routing
- Configure `headers` for caching
- Use `crons` for scheduled tasks

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails | Check build command in `vercel.json` |
| 404 on routes | Configure rewrites in `vercel.json` |
| Environment variables missing | Run `vercel env pull` |
| Timeout errors | Increase `maxDuration` in config |
