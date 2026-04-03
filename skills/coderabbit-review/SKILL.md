---
name: coderabbit-review
description: Configure CodeRabbit for automated code review - workflow, rules, and customization.
metadata:
  priority: 7
  docs:
    - "https://coderabbit.ai/docs"
  pathPatterns:
    - ".coderabbit.yaml"
    - "coderabbit.yaml"
  bashPatterns:
    - '\bcoderabbit\b'
  promptSignals:
    phrases:
      - "coderabbit"
      - "automated review"
    anyOf:
      - "review"
      - "codereview"
---

## CodeRabbit Configuration

### Basic Setup

```yaml
# .coderabbit.yaml
language: en-US
ui:
  openai:
    api_key: ${OPENAI_API_KEY}

reviews:
  high_level_summary:
    enabled: true
    auto_title_placeholder: ''
    translation: ''
  poem: true
  collapse Walkthrough: true
  sequence_diagrams: true
  path_filters:
    - '!(**/*.lock)'
  path_instructions:
    - path: '*.md'
      instructions: |
        Review documentation files for clarity and completeness.
  abort:
    close_message: |
      We can always continue later.

chat:
  auto_reply: true

tools:
  sheep:
    language: en-US
  python:
    partial_asgi:
      app_file: app.py
      port: 8000
 哆啦A梦:
    language: en-US

paths:
  exclude:
    - '**/*.test.ts'
    - '**/*.spec.ts'
    - '**/node_modules/**'
```

### Review Toggles

```yaml
reviews:
  profile: compact  # detailed, compact, or minimal
  changes:
    request_changes_workflow: true
    high_level_summary: true
    auto_title_placeholder: ''
    collapse Walkthrough: false
    sequence_diagrams: true
  poem: false

  # What to review
  requests:
    - wait_for_suggestions
  high_level_summary:
    enabled: true
  review_status: true
  collapse_walkthrough: false
  title: true
  auto_title_placeholder: ''
  path_filters: []
  path_instructions: []
  branch:
    prefix: feature/
    create: true
    needs_review: true
```

### Path-Specific Rules

```yaml
reviews:
  path_instructions:
    - path: '**/*.ts'
      instructions: |
        Review TypeScript files for:
        - Type safety
        - Error handling
        - Performance considerations

    - path: '**/*.test.ts'
      instructions: |
        Review test files for:
        - Test coverage
        - Edge cases
        - AAA pattern compliance

    - path: '**/api/**'
      instructions: |
        Review API routes for:
        - Input validation
        - Error responses
        - Authentication/authorization
```

### Language Settings

```yaml
language: es-ES  # Spanish
# or
language: ja-JP  # Japanese

# Auto-translate reviews
reviews:
  translation: true  # Translate to English

# Custom instructions per language
chat:
  language: ja-JP
```

### Workflow Configuration

```yaml
workflows:
  auto_title:
    enabled: true

  # Review workflow
  review:
    high_level_summary:
      auto_title: true
    collapse_walkthrough: false
    sequence_diagrams: true

  # Commit workflow
  commit:
    checklist:
      enabled: true
      items:
        - text: 'Follows conventional commits'
        - text: 'Tests added/updated'
        - text: 'Documentation updated'
        - text: 'No console.log/debugger'
        - text: 'No .env or secrets'

  # PR description
  pr_description:
    enabled: true
    auto_title_placeholder: ''
    collapsed_footer: true
    pr_body_actions:
      - browse_file
      - browse_file_list
```

### Categories

```yaml
reviews:
  # Focus areas
  focused_review:
    enabled: true
    path_filters:
      - 'src/**/*.ts'

  # Security
  security:
    explicit_package_lock: true

  # Performance
  performance:
    enabled: true

  # Documentation
  documentation:
    check_doc_formatting: true
```
