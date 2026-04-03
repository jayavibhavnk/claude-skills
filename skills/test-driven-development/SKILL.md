---
name: test-driven-development
description: Apply TDD methodology - write tests first, then implement features. Covers unit, integration, and E2E testing.
metadata:
  priority: 9
  docs:
    - "https://jestjs.io/docs/getting-started"
    - "https://vitest.dev/guide/"
  pathPatterns:
    - "**/*.test.ts"
    - "**/*.spec.ts"
    - "**/__tests__/**"
    - "tests/**"
  bashPatterns:
    - '\bjest\b'
    - '\bvitest\b'
    - '\bplaywright\b'
    - '\bcypress\b'
  promptSignals:
    phrases:
      - "tdd"
      - "test first"
      - "write tests"
    anyOf:
      - "unit test"
      - "integration test"
      - "e2e"
      - "coverage"
---

## Test-Driven Development

### The TDD Cycle

```
RED → GREEN → REFACTOR
```

1. **RED** - Write a failing test
2. **GREEN** - Write minimal code to pass
3. **REFACTOR** - Improve code while keeping tests green

### Test Structure

```typescript
// Arrange - Set up test data
// Act - Execute the function
// Assert - Verify the result

describe('Calculator', () => {
  it('should add two numbers', () => {
    // Arrange
    const a = 2;
    const b = 3;

    // Act
    const result = add(a, b);

    // Assert
    expect(result).toBe(5);
  });
});
```

### Unit Tests

```typescript
// Keep tests focused and fast
describe('UserService', () => {
  it('should hash passwords before saving', async () => {
    const user = await userService.create({
      email: 'test@example.com',
      password: 'plaintext123',
    });

    expect(user.passwordHash).not.toBe('plaintext123');
    expect(await bcrypt.compare('plaintext123', user.passwordHash)).toBe(true);
  });

  it('should reject duplicate emails', async () => {
    await userService.create({ email: 'test@example.com' });

    await expect(
      userService.create({ email: 'test@example.com' })
    ).rejects.toThrow('Email already exists');
  });
});
```

### Integration Tests

```typescript
describe('API /users', () => {
  it('should create a user via POST', async () => {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: 'test@example.com' }),
    });

    expect(response.status).toBe(201);
    const user = await response.json();
    expect(user.id).toBeDefined();
  });
});
```

### E2E Tests (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test('user can sign up', async ({ page }) => {
  await page.goto('/signup');

  await page.fill('[name="email"]', 'newuser@example.com');
  await page.fill('[name="password"]', 'securepassword123');
  await page.click('[type="submit"]');

  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('text=Welcome')).toBeVisible();
});
```

### Mocking

```typescript
import { jest } from '@jest/globals';

// Mock external dependencies
jest.mock('../external/api');
api.fetchUser.mockResolvedValue({ id: '1', name: 'Test' });

// Mock functions
const consoleLog = jest.spyOn(console, 'log').mockImplementation();
```

### Testing Async Code

```typescript
it('should handle async errors', async () => {
  // Use async/await with expect
  await expect(fetchUser('invalid-id')).rejects.toThrow('Not found');
});

it('should handle promises', async () => {
  const result = await expect(promise).resolves.toBe('value');
});
```

### Test Coverage

```bash
# Run with coverage
jest --coverage

# Coverage thresholds in jest.config.js
{
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
    },
  },
}
```

## TDD Rules

1. Write failing test before writing any code
2. Only write code to fix failing tests
3. Remove duplication (refactor)
4. Tests should be fast (< 100ms each)
5. Tests should be independent
6. Name tests clearly: `describe('X').it('should Y')`
