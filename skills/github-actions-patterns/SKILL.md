---
name: github-actions-patterns
description: GitHub Actions patterns - workflows, matrix builds, caching, secrets, reusable workflows, and deployment automation.
metadata:
  priority: 8
  docs:
    - "https://docs.github.com/en/actions"
  pathPatterns:
    - ".github/workflows/**"
  bashPatterns:
    - '\bgithub.actions\b'
    - '\bci/cd\b'
  promptSignals:
    phrases:
      - "github actions"
      - "ci/cd"
      - "workflow"
    anyOf:
      - "github"
      - "actions"
      - "workflow"
---

## GitHub Actions Patterns

### Basic Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

### Matrix Builds

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
        exclude:
          - node-version: 22
            os: windows-latest  # Skip Windows for Node 22
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            artifact_name: linux
          - os: windows-latest
            artifact_name: windows
    steps:
      - uses: actions/checkout@v4
      - name: Build ${{ matrix.artifact_name }}
        run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.artifact_name }}
          path: dist/
```

### Caching

```yaml
# Cache npm dependencies
- name: Cache node_modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-

# Cache build outputs
- name: Cache build
  uses: actions/cache@v4
  with:
    path: |
      .next
      node_modules/.cache
    key: ${{ runner.os }}-build-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-build-

# Cache Docker layers
- name: Cache Docker layers
  uses: actions/cache@v4
  with:
    path: /tmp/.docker-cache
    key: ${{ runner.os }}-docker-${{ github.sha }}
```

### Secrets Management

```yaml
# Access secrets
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
  run: |
    ./deploy.sh --api-key=$API_KEY

# Secrets from different sources
- name: Configure AWS
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
    secrets:
      NPM_TOKEN:
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      - run: npm test

# Caller workflow
jobs:
  test-linux:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Deployment

```yaml
# Deploy to Vercel
- name: Deploy to Vercel
  uses: amondnet/vercel-action@v25
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
    vercel-org-id: ${{ secrets.ORG_ID }}
    vercel-project-id: ${{ secrets.PROJECT_ID }}
    vercel-args: '--prod'

# Deploy with conditional
- name: Deploy to Production
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: ./deploy.sh production
```

### Concurrency Control

```yaml
# Cancel in-progress runs
on:
  push:
    branches: [main]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# Single deployment at a time
concurrency:
  group: deploy-${{ github.environment }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
```

### Notifications

```yaml
# Slack notification
- name: Notify Slack
  if: always()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Build ${{ job.status }} for ${{ github.repository }}",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "Build ${{ job.status }}: ${{ github.workflow }}"
            }
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

### Best Practices

1. **Cache dependencies** - Speed up runs significantly
2. **Fail fast matrix** - `fail-fast: true` (default)
3. **Concurrency groups** - Prevent race conditions
4. **Reusable workflows** - DRY across repos
5. **Environment protection** - Require approvals
6. **Minimal permissions** - Use fine-grained tokens
7. **Idempotent actions** - Handle reruns gracefully
