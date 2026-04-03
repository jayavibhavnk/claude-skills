---
name: pr-code-review
description: Review pull requests thoroughly - provide constructive feedback, catch bugs, and ensure code quality.
metadata:
  priority: 9
  docs:
    - "https://google.github.io/styleguide/"
  pathPatterns:
    - "**/*.pr.*"
    - "**/review/**"
  bashPatterns:
    - '\bpr\b'
    - '\bpull.?request\b'
    - '\bcode.?review\b'
  promptSignals:
    phrases:
      - "review"
      - "pull request"
      - "pr feedback"
    anyOf:
      - "review"
      - "feedback"
      - "approve"
---

## Pull Request Review

### Review Checklist

#### Correctness
- [ ] Does the code do what the PR claims?
- [ ] Are edge cases handled?
- [ ] Could this break existing functionality?
- [ ] Are there any obvious bugs?

#### Security
- [ ] Any SQL injection vulnerabilities?
- [ ] Is user input properly validated?
- [ ] Any sensitive data exposed?
- [ ] Proper authentication/authorization?

#### Performance
- [ ] Any N+1 query issues?
- [ ] Unnecessary re-renders?
- [ ] Large data being loaded unnecessarily?
- [ ] Missing database indexes?

#### Code Quality
- [ ] Clear, descriptive naming?
- [ ] Functions are reasonably sized?
- [ ] DRY - avoid repetition?
- [ ] Comments explain *why*, not *what*?

#### Testing
- [ ] Are new features tested?
- [ ] Are edge cases covered?
- [ ] Tests actually test behavior?
- [ ] No false positives in assertions?

### Review Comments

#### Be Constructive
```markdown
<!-- Good: Specific, actionable feedback -->
Consider extracting this logic into a separate function:

\`\`\`typescript
// Current
const result = data.filter(d => d.active).map(d => d.id);

// Suggested
const activeIds = getActiveIds(data);
\`\`\`

This makes it easier to test and reuse elsewhere.

<!-- Avoid: Vague criticism -->
"This could be better."
```

#### Praise Good Work
```markdown
<!-- Acknowledge improvements -->
Nice approach using the reducer pattern here - much cleaner than
the previous setState chain!
```

#### Ask Questions
```markdown
<!-- Instead of assuming intent -->
Is the timeout intentional here? It might cause issues if the
network request takes longer than expected.
```

### Comment Types

| Prefix | Meaning |
|--------|---------|
| `nit:` | Minor, optional suggestion |
| `suggestion:` | Non-blocking improvement |
| `question:` | Seeking clarification |
| `issue:` | Bug or problem |
| `blocker:` | Must be addressed before merge |

### Things to Look For

#### Frontend
- Memory leaks (event listeners not cleaned up)
- XSS vulnerabilities (innerHTML usage)
- Accessibility (missing alt text, ARIA)
- Responsive design
- Loading/error states

#### Backend
- SQL injection
- Rate limiting
- Input validation
- Error handling
- Authentication/authorization
- Logging

#### Data
- Migration safety
- Backward compatibility
- Data consistency
- Index usage

### Giving Feedback

**DO:**
- Be specific about the issue
- Explain why it matters
- Offer suggestions for fixes
- Acknowledge good solutions
- Be kind and respectful

**DON'T:**
- Be vague ("this looks weird")
- Demand changes without explaining
- Focus on style over substance
- Block on minor nits
- Be rude or dismissive

### Responding to Review

```markdown
<!-- Acknowledge feedback -->
Good catch! Fixed in the latest commit.

<!-- Push back respectfully -->
I went with this approach because [reason].
Happy to revisit if you feel strongly about it.

<!-- Ask for clarification -->
Could you elaborate on why this is a problem?
I'm not seeing how it affects the use case.
```

### Merge Criteria

**Must have:**
- [ ] Tests pass
- [ ] No merge conflicts
- [ ] At least 1 approval
- [ ] All comments resolved
- [ ] CI green

**Should have:**
- [ ] Test coverage maintained
- [ ] Documentation updated
- [ ] No obvious issues

### PR Description Template

```markdown
## Summary
Brief description of changes

## Type
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Refactor

## Testing
How was this tested?

## Screenshots (if UI changes)
Before/After screenshots

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No console.log/debugger left
```
