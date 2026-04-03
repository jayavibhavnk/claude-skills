---
name: gitlab-ci
description: Build GitLab CI/CD pipelines for automated testing, building, and deployment.
metadata:
  priority: 8
  docs:
    - "https://docs.gitlab.com/ee/ci/"
    - "https://docs.gitlab.com/ee/ci/yaml/"
  pathPatterns:
    - ".gitlab-ci.yml"
    - "**/.gitlab-ci.yml"
  bashPatterns:
    - '\bgitlab\b'
    - '\bgitlab-ci\b'
  promptSignals:
    phrases:
      - "gitlab ci"
      - "gitlab pipeline"
    anyOf:
      - "ci/cd"
      - "pipeline"
      - "gitlab"
---

## GitLab CI

### Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "20"

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - .next/
    expire_in: 1 hour

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run test
  coverage: '/Coverage: \d+\.\d+%/'

deploy:
  stage: deploy
  script:
    - echo "Deploying..."
  only:
    - main
```

### Stages and Jobs

```yaml
stages:
  - lint      # Code quality
  - test      # Unit/integration tests
  - build     # Build artifacts
  - deploy    # Deployment

# Lint job
eslint:
  stage: lint
  image: node:20
  script:
    - npm run lint

# Unit tests
jest:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run test:unit
  coverage: '/Lines:\s*\d+\.\d+/'

# Integration tests
e2e:
  stage: test
  image: node:20
  services:
    - postgres:15
  variables:
    POSTGRES_DB: test
    DATABASE_URL: postgres://postgres:postgres@postgres:5432/test
  script:
    - npm ci
    - npm run test:e2e
```

### Matrix Jobs

```yaml
test:
  stage: test
  image: node:20
  parallel:
    matrix:
      - NODE_VERSION: [18, 20, 22]
        OS: [ubuntu, alpine]
  script:
    - npm ci
    - npm run test
```

### Artifacts and Dependencies

```yaml
build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
    when: always

deploy:
  stage: deploy
  dependencies:
    - build
  script:
    - npm run deploy
```

### GitLab Pages

```yaml
pages:
  stage: deploy
  image: node:20
  script:
    - npm ci
    - npm run build
    - mv public public && mv dist public
  artifacts:
    paths:
      - public
  only:
    - main
```

### Environment Variables

```yaml
deploy:production:
  stage: deploy
  variables:
    DATABASE_URL: $PROD_DATABASE_URL
    API_SECRET: $API_SECRET
  script:
    - npm run deploy:production
  environment:
    name: production
    url: https://example.com
  only:
    - main
```

### Best Practices

1. Use `cache` for dependencies
2. Use `artifacts` to share build outputs
3. Set appropriate `expire_in` for artifacts
4. Use `needs:` for DAG pipelines
5. Parallelize with `parallel:matrix`
6. Use `extends` for job templates
7. Store secrets in CI/CD Variables
