---
name: architecture-design
description: Design system architectures - from small features to large-scale distributed systems.
metadata:
  priority: 9
  docs:
    - "https://docs.aws.amazon.com"
    - "https://learn.microsoft.com/en-us/azure/architecture"
  pathPatterns:
    - "**/architecture/**"
    - "**/docs/architecture/**"
  bashPatterns:
    - '\barchitecture\b'
    - '\bdesign\b'
    - '\bstructur'
  promptSignals:
    phrases:
      - "architecture"
      - "system design"
      - "scalability"
    anyOf:
      - "design"
      - "structure"
      - "microservices"
      - "distributed"
---

## Architecture Design

### Architecture Decision Record (ADR)

```markdown
# ADR 001: Use PostgreSQL for Primary Database

## Status
Accepted

## Context
We need a primary database for user data.

## Decision
We will use PostgreSQL with Supabase.

## Consequences
- Pros: ACID compliance, rich feature set, good tooling
- Cons: Requires managing database infrastructure
```

### Common Patterns

#### Layered Architecture
```
┌─────────────────┐
│    UI Layer     │
├─────────────────┤
│  Service Layer  │
├─────────────────┤
│   Data Layer    │
├─────────────────┤
│  Repository     │
└─────────────────┘
```

#### Event-Driven Architecture
```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Producer │────▶│  Kafka   │────▶│ Consumer │
└──────────┘     └──────────┘     └──────────┘
```

#### Microservices
```
┌─────────┐  ┌─────────┐  ┌─────────┐
│  Auth   │  │  Users  │  │  Orders │
│ Service │  │ Service │  │ Service │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────────┴────────────┘
              API Gateway
```

### Scalability Principles

1. **Horizontal vs Vertical** - Scale out vs scale up
2. **Stateless Design** - Easy horizontal scaling
3. **Caching** - Reduce database load
4. **Async Processing** - Decouple heavy work
5. **CDN** - Serve static assets globally

### API Design

#### RESTful
```
GET    /users        - List users
POST   /users        - Create user
GET    /users/:id    - Get user
PUT    /users/:id    - Update user
DELETE /users/:id    - Delete user
```

#### GraphQL
```graphql
type Query {
  user(id: ID!): User
  users(limit: Int): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

### Database Selection

| Use Case | Database |
|----------|----------|
| Primary data | PostgreSQL |
| Caching | Redis |
| Search | Elasticsearch |
| Analytics | ClickHouse |
| Documents | MongoDB |
| Time series | InfluxDB |

### Reliability Patterns

```typescript
// Circuit Breaker
class CircuitBreaker {
  private failures = 0;
  private lastFailure = 0;
  private state: 'closed' | 'open' = 'closed';

  async call(fn: () => Promise<any>) {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailure > 60000) {
        this.state = 'half-open';
      } else {
        throw new Error('Circuit open');
      }
    }

    try {
      const result = await fn();
      this.failures = 0;
      this.state = 'closed';
      return result;
    } catch (e) {
      this.failures++;
      this.lastFailure = Date.now();
      if (this.failures > 5) {
        this.state = 'open';
      }
      throw e;
    }
  }
}
```

### Performance Budgets

| Metric | Target |
|--------|--------|
| TTFB | < 200ms |
| FCP | < 1.5s |
| LCP | < 2.5s |
| API Response | < 300ms |
| DB Query | < 100ms |

## Questions to Ask

1. What are the scaling requirements?
2. What's the data access pattern?
3. What's the consistency requirement?
4. What's the availability target?
5. What's the team structure?
6. What's the operational complexity budget?
