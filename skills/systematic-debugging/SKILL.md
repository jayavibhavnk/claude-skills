---
name: systematic-debugging
description: Debug issues methodically using the scientific method - observe, hypothesize, test, iterate.
metadata:
  priority: 10
  docs:
    - "https://github.com/damassi/systematic-debugging"
  pathPatterns:
    - "**/*.test.ts"
    - "**/*.spec.ts"
    - "**/__tests__/**"
    - "**/debug/**"
  bashPatterns:
    - '\bdebug\b'
    - '\bconsole\.log\b'
    - '\bbreakpoint\b'
    - '\bwatch\b'
  promptSignals:
    phrases:
      - "debug this"
      - "something broke"
      - "not working"
      - "error in"
    anyOf:
      - "bug"
      - "crash"
      - "issue"
      - "fix"
      - "broken"
---

## Systematic Debugging

### The Scientific Method for Code

```
OBSERVE → HYPOTHESIZE → TEST → ITERATE → RESOLVE
```

### Step 1: Observe

Before fixing anything:
1. Read the error message completely
2. Note the exact conditions that trigger the issue
3. Identify when the issue first appeared
4. Check recent changes (commits, dependencies)

### Step 2: Hypothesize

Form a testable hypothesis:
- What do you think is happening?
- What specific line/component is the root cause?
- What evidence supports this theory?

### Step 3: Test

Design experiments to test your hypothesis:
1. Isolate the problematic code
2. Add minimal reproduction case
3. Test one variable at a time
4. Compare behavior with/without changes

### Step 4: Iterate

Based on test results:
- Confirm or reject hypothesis
- Form new hypothesis if needed
- Narrow down scope progressively

## Debugging Tools

### Console Debugging
```javascript
// Add structured logging
console.debug({
  action: 'user-login',
  userId: id,
  timestamp: Date.now(),
  state: { before: oldState, after: newState }
});
```

### Breakpoints
```javascript
// Source-level debugging
debugger; // Set breakpoint programmatically

// Conditional breakpoint
if (userId === specificId) {
  debugger;
}
```

### Network Debugging
```javascript
// Log all fetch requests
const originalFetch = window.fetch;
window.fetch = async (...args) => {
  console.log('Fetch:', args[0], args[1]);
  const response = await originalFetch(...args);
  console.log('Response:', response.status);
  return response;
};
```

## Common Patterns

### Race Conditions
```javascript
// Problem: Async operations complete in wrong order
// Solution: Use proper async/await and Promise handling
const result = await Promise.all([fetchA(), fetchB()]);
```

### Memory Leaks
```javascript
// Check for event listener leaks
// Always cleanup in useEffect or componentWillUnmount
useEffect(() => {
  const handler = () => {};
  element.addEventListener('click', handler);
  return () => element.removeEventListener('click', handler);
}, []);
```

### State Inconsistency
```javascript
// Use immer or similar for immutable updates
// Log state transitions to trace unexpected changes
```

## Debugging Checklist

- [ ] Can you reproduce the issue consistently?
- [ ] What is the minimal code to reproduce?
- [ ] What does the error stack trace show?
- [ ] What changed recently (commits, deps)?
- [ ] Does it work in isolation?
- [ ] Is the issue in frontend, backend, or both?
- [ ] What do the logs show?

## Questions to Ask

1. What is the expected behavior?
2. What is the actual behavior?
3. Can I reproduce it locally?
4. What's different about the failing environment?
5. What does the error message literally say?
