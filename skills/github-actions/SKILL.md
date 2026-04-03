---
name: github-actions
description: Build and optimize GitHub Actions CI/CD workflows for automated testing, building, and deployment.
metadata:
  priority: 9
  docs:
    - "https://docs.github.com/en/actions"
    - "https://docs.github.com/en/actions/quickstart"
  pathPatterns:
    - ".github/workflows/**"
    - ".github/actions/**"
  bashPatterns:
    - '\bgh\s+run\b'
    - '\bgh\s+workflow\b'
    - '\bgithub-actions\b'
  promptSignals:
    phrases:
      - "github action"
      - "ci/cd"
      - "workflow"
    anyOf:
      - "automate"
      - "pipeline"
      - "build"
      - "test"
      - "deploy"
---

## GitHub Actions

### Basic Workflow Structure

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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

### Common Workflow Patterns

#### Matrix Builds
```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
```

#### Caching Dependencies
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ${{ env.HOME }}/.cache/pip
    key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json') }}
```

#### Conditional Steps
```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: npm run deploy

- name: Notify
  if: failure()
  run: echo "Workflow failed"
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        default: '20'
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

### Secrets Management

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
      PUBLIC_URL: ${{ secrets.PRODUCTION_URL }}
    run: npm run deploy
```

### Artifacts & Caching

```yaml
# Upload artifact
- uses: actions/upload-artifact@v4
  with:
    name: build-${{ matrix.node-version }}
    path: dist/

# Download artifact
- uses: actions/download-artifact@v4
  with:
    name: build
    path: dist/
```

### Concurrency Control

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### Best Practices

1. Use `actions/checkout@v4` and `actions/setup-*` latest versions
2. Pin action versions for reproducibility
3. Use `npm ci` instead of `npm install` in CI
4. Enable dependency caching to speed up builds
5. Set concurrency to cancel redundant runs
6. Use matrix builds for comprehensive test coverage
7. Store secrets in GitHub Secrets, never in code
