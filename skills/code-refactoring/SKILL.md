---
name: code-refactoring
description: Refactor and improve existing code - reduce duplication, improve readability, and apply SOLID principles.
metadata:
  priority: 8
  docs:
    - "https://refactoring.com"
  pathPatterns:
    - "**/*.ts"
    - "**/*.js"
    - "**/*.py"
  bashPatterns:
    - '\brefactor\b'
    - '\brefactor\b'
  promptSignals:
    phrases:
      - "refactor"
      - "clean up"
      - "improve code"
    anyOf:
      - "duplication"
      - "extract"
      - "rename"
      - "improve"
---

## Code Refactoring

### Core Principles

1. **Single Responsibility** - Each function/class does one thing
2. **Don't Repeat Yourself (DRY)** - Extract repeated logic
3. **Keep It Simple (KISS)** - Prefer simple over clever
4. **Open/Closed** - Open for extension, closed for modification

### Common Refactorings

#### Extract Function
```typescript
// Before: Logic inline
function handleSubmit() {
  const data = formData();
  validate(data);
  const sanitized = sanitize(data);
  saveToDatabase(sanitized);
  sendAnalytics(sanitized);
  showSuccess();
}

// After: Extracted functions
function handleSubmit() {
  const data = getFormData();
  const validated = validateAndSanitize(data);
  await persistData(validated);
  trackSubmission(validated);
  notifySuccess();
}
```

#### Replace Conditional with Polymorphism
```typescript
// Before: Switch statement
function calculatePay(employee) {
  switch (employee.type) {
    case 'hourly':
      return employee.hours * employee.rate;
    case 'salary':
      return employee.salary / 12;
    case 'commission':
      return employee.base + employee.sales * 0.1;
  }
}

// After: Polymorphic approach
class Employee {
  calculatePay() { throw new Error('Abstract'); }
}

class HourlyEmployee extends Employee {
  calculatePay() { return this.hours * this.rate; }
}
```

#### Extract Hook
```typescript
// Before: Component with logic
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser().then(setUser).finally(() => setLoading(false));
  }, []);

  // ... rest of component
}

// After: Custom hook
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);

  return { user, loading };
}

function UserProfile() {
  const { user, loading } = useUser(userId);
  // ... rest of component
}
```

### Code Smells to Fix

| Smell | Refactoring |
|-------|-------------|
| Long function | Extract smaller functions |
| Too many parameters | Introduce parameter object |
| Duplicated code | Extract to shared function |
| Magic numbers | Extract to named constant |
| Deep nesting | Extract early returns |
| Dead code | Delete it |
| God class | Split into focused classes |

### React-Specific Patterns

```typescript
// Extract state logic to useReducer
// Before: Multiple useState
function Component() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // ... handlers
}

// After: Single useReducer
function Component() {
  const [state, dispatch] = useReducer(reducer, {
    name: '',
    email: '',
    loading: false,
    error: null,
  });

  // ... handlers using dispatch
}
```

### Testing After Refactoring

1. Run full test suite
2. Ensure all tests pass
3. No behavior change (only structure)
4. Commit after each successful refactor

## When to Refactor

- Before adding new features
- When you encounter "wtf" moments
- When tests are hard to write
- When code is hard to understand
- Before code review

## When NOT to Refactor

- When deadline is critical
- When code works and won't change
- When it's throwaway code
- When you don't have tests
