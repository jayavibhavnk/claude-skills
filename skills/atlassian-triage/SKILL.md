---
name: atlassian-triage
description: Triage Jira issues and manage Atlassian workflows - organize backlogs, prioritize, and track progress.
metadata:
  priority: 7
  docs:
    - "https://developer.atlassian.com/cloud/jira/software/"
  pathPatterns:
    - "**/jira/**"
    - "**/atlassian/**"
  bashPatterns:
    - '\bjira\b'
    - '\batlassian\b'
  promptSignals:
    phrases:
      - "jira"
      - "tickets"
      - "sprint"
    anyOf:
      - "triage"
      - "backlog"
      - "issue"
---

## Atlassian Triage

### Issue Types and Priorities

| Type | Description |
|------|-------------|
| Epic | Large feature, spans sprints |
| Story | User-facing feature |
| Task | Technical work |
| Bug | Something is broken |
| Spike | Research/investigation |

### Priority Levels

| Priority | Meaning | Response |
|----------|---------|----------|
| P0 | Critical - outage | Immediate |
| P1 | High - major feature blocked | 24 hours |
| P2 | Medium - should address | This sprint |
| P3 | Low - nice to have | Backlog |
| P4 | None - won't fix | Ignore |

### Triage Process

```markdown
## Issue Triage Checklist

### 1. Is it valid?
- [ ] Clear problem statement?
- [ ] Reproducible issue (if bug)?
- [ ] Has proper context?

### 2. Is it scoped?
- [ ] Clear acceptance criteria?
- [ ] Estimated effort?
- [ ] Dependencies identified?

### 3. Priority
- [ ] Business impact assessed?
- [ ] Customer facing?
- [ ] Blocks other work?

### 4. Assignment
- [ ] Right team/owner?
- [ ] Has required skills?
- [ ] Capacity available?
```

### Writing Good Issues

```markdown
## Title
[BUG] Cart total incorrect when applying percentage coupon

## Description
### What happened?
Applied a 20% off coupon to a $99 item. Expected: $79.20.
Actual: $79.00.

### Steps to reproduce
1. Add item to cart
2. Apply code "SAVE20"
3. View cart total

### Expected behavior
Cart should show $79.20 (99 * 0.8)

### Actual behavior
Cart shows $79.00

### Environment
- Browser: Chrome 120
- OS: macOS 14
- App version: 2.3.1

### Screenshots
[screenshot]
```

### Sprint Planning

```markdown
## Sprint 23 Planning

### Capacity
- 3 engineers × 8 days × 6 hours = 144 hours

### Velocity (last 3 sprints)
- Sprint 20: 34 points
- Sprint 21: 38 points
- Sprint 22: 36 points
- Average: 36 points

### Committed: 36 points

### Stories
| Issue | Points | Assignee |
|-------|--------|----------|
| LOGIN-123 | 5 | @alice |
| CART-456 | 8 | @bob |
| CHECKOUT-789 | 13 | @charlie |
| CHECKOUT-790 | 5 | @bob |
| CHECKOUT-791 | 5 | @charlie |
```

### Labels Convention

```bash
# Type labels
type/bug
type/feature
type/chore
type/spike

# Status labels
status/blocked
status/in-review
status/ready-for-qa

# Team labels
team/backend
team/frontend
team/mobile
team/devops

# Complexity
effort/small      # < 2 hours
effort/medium      # 2-8 hours
effort/large       # 8+ hours
```

### Best Practices

1. **Be specific** - "Fix bug" → "Fix cart calculation with coupons"
2. **Add context** - Link related issues, docs, screenshots
3. **Set clear AC** - How will you know it's done?
4. **Keep updated** - Move to correct status, update estimates
5. **No zombie tickets** - Review stale issues monthly
