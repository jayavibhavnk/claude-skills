---
name: tech-research
description: Research new technologies, compare alternatives, and evaluate tools for adoption.
metadata:
  priority: 8
  docs:
    - "https://github.com"
    - "https://dev.to"
  pathPatterns:
    - "**/research/**"
    - "**/docs/research/**"
  bashPatterns:
    - '\bresearch\b'
    - '\bcompare\b'
    - '\beval'
  promptSignals:
    phrases:
      - "research"
      - "evaluate"
      - "compare"
    anyOf:
      - "tech"
      - "tool"
      - "library"
      - "framework"
---

## Technology Research

### Research Framework

#### 1. Define the Problem
- What problem are we solving?
- What are the constraints?
- What's the success criteria?

#### 2. Gather Information
- Official documentation
- GitHub stars and activity
- Stack Overflow discussions
- Real-world usage
- Benchmarks (take with salt)

#### 3. Evaluate Criteria

| Criteria | Weight | Notes |
|----------|--------|-------|
| Performance | High | Benchmarks, profiling |
| Developer Experience | High | Learning curve, docs quality |
| Community | Medium | Size, activity, support |
| Maintenance | High | Release cadence, stability |
| Licensing | Medium | Open source vs commercial |
| Ecosystem | High | Integrations, plugins |

### Comparison Template

```markdown
## Technology Comparison: [X] vs [Y]

### Overview
[X]: Brief description
[Y]: Brief description

### Pros
**X:**
- + Point 1
- + Point 2

**Y:**
- + Point 1
- + Point 2

### Cons
**X:**
- - Point 1
- - Point 2

**Y:**
- - Point 1
- - Point 2

### Performance
| Metric | X | Y |
|--------|---|---|
| Latency | 10ms | 15ms |
| Throughput | 1000 req/s | 800 req/s |

### Decision Matrix

| Criteria | Weight | X Score | Y Score |
|----------|--------|---------|---------|
| Performance | 30% | 8/10 | 7/10 |
| DX | 25% | 9/10 | 6/10 |
| Ecosystem | 20% | 7/10 | 9/10 |
| **Weighted Total** | 100% | **8.0** | **7.3** |

### Recommendation
[Based on above analysis]
```

### Questions to Answer

1. **What problem does it solve?**
   - Is the problem real or perceived?

2. **How mature is the project?**
   - Version number (1.0+ is more stable)
   - Release history
   - Breaking changes frequency

3. **What's the community like?**
   - GitHub stars (but don't trust blindly)
   - Issue response time
   - Active maintainers

4. **What's the business model?**
   - Open core (limited free, paid features)
   - Fully open source
   - SaaS with generous free tier

5. **What are the exit costs?**
   - Vendor lock-in risk
   - Migration complexity
   - Data portability

### Red Flags

- No activity in 2+ years
- Single maintainer for critical projects
- Hostile community
- Unclear licensing
- No documentation
- Breaking changes every week

### Green Flags

- Semantic versioning
- Changelog maintained
- Good first issues labeled
- Active Discord/Slack
- Production users listed
- Security policy
