---
name: testing-best-practices
description: Modern testing best practices - AAA pattern, mocking strategies, test isolation, and coverage analysis.
metadata:
  priority: 9
  docs:
    - "https://testing-library.com/docs/"
  pathPatterns:
    - "**/*.test.ts"
    - "**/*.spec.ts"
    - "**/__tests__/**"
  bashPatterns:
    - '\bjest\b'
    - '\bvitest\b'
    - '\btesting.?library\b'
  promptSignals:
    phrases:
      - "testing"
      - "unit test"
      - "integration"
    anyOf:
      - "test"
      - "mock"
      - "coverage"
---

## Testing Best Practices

### AAA Pattern (Arrange-Act-Assert)

```typescript
describe('OrderService', () => {
  it('should create an order with correct total', async () => {
    // Arrange
    const user = await createTestUser({ name: 'John', balance: 100 });
    const items = [
      { productId: 'prod_1', quantity: 2, price: 25 },
      { productId: 'prod_2', quantity: 1, price: 50 },
    ];

    // Act
    const order = await orderService.createOrder(user.id, items);

    // Assert
    expect(order.total).toBe(100);
    expect(order.status).toBe('pending');
    expect(order.items).toHaveLength(2);
  });

  it('should throw when user has insufficient balance', async () => {
    // Arrange
    const user = await createTestUser({ name: 'Jane', balance: 50 });
    const items = [{ productId: 'prod_1', quantity: 1, price: 100 }];

    // Act & Assert
    await expect(orderService.createOrder(user.id, items))
      .rejects.toThrow('Insufficient balance');
  });
});
```

### Testing Library Patterns

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('should show validation error for invalid email', async () => {
    // Render component
    render(<LoginForm onSubmit={jest.fn()} />);

    // Fill invalid email
    fireEvent.input(screen.getByLabelText('Email'), {
      target: { value: 'invalid-email' },
    });

    // Submit form
    fireEvent.click(screen.getByRole('button', { name: 'Submit' }));

    // Wait for error
    await waitFor(() => {
      expect(screen.getByText('Invalid email format')).toBeInTheDocument();
    });
  });

  it('should call onSubmit with form data', async () => {
    const mockSubmit = jest.fn();
    render(<LoginForm onSubmit={mockSubmit} />);

    fireEvent.input(screen.getByLabelText('Email'), {
      target: { value: 'test@example.com' },
    });
    fireEvent.input(screen.getByLabelText('Password'), {
      target: { value: 'password123' },
    });

    fireEvent.click(screen.getByRole('button', { name: 'Submit' }));

    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });
});
```

### Mocking Strategies

```typescript
// 1. Module mocking
jest.mock('../api/userService', () => ({
  getUser: jest.fn(),
  updateUser: jest.fn(),
}));

// 2. Spy on methods
it('should call analytics on purchase', async () => {
  const analyticsSpy = jest.spyOn(analytics, 'track');

  await orderService.purchase(orderId);

  expect(analyticsSpy).toHaveBeenCalledWith('purchase', {
    orderId,
    amount: expect.any(Number),
  });
});

// 3. Mock functions
it('should timeout after 5 seconds', async () => {
  jest.useFakeTimers();

  const promise = fetchData();
  jest.advanceTimersByTime(5000);

  await expect(promise).rejects.toThrow('Timeout');
});
```

### Test Isolation

```typescript
describe('UserService', () => {
  // Setup before each test
  beforeEach(() => {
    // Clear all mocks
    jest.clearAllMocks();

    // Reset database
    testDb.reset();

    // Setup default mocks
    jest.spyOn(emailService, 'send').mockResolvedValue(true);
  });

  // Cleanup after each test
  afterEach(() => {
    jest.useRealTimers();
  });

  // Shared setup for related tests
  describe('with premium user', () => {
    let premiumUser: User;

    beforeEach(async () => {
      premiumUser = await createTestUser({ tier: 'premium' });
    });

    it('should allow unlimited uploads', () => {
      // premiumUser is available here
      expect(userService.canUpload(premiumUser)).toBe(true);
    });
  });
});
```

### Async Testing

```typescript
it('should retry on transient failure', async () => {
  // Use fake timers for controlled async testing
  jest.useFakeTimers();

  const fetchWithRetry = jest.fn()
    .mockRejectedValueOnce(new Error('Network error'))
    .mockRejectedValueOnce(new Error('Network error'))
    .mockResolvedValueOnce({ data: 'success' });

  const promise = withRetry(fetchWithRetry, { maxRetries: 3 });

  // Fast-forward through retries
  await jest.runAllTimersAsync();

  const result = await promise;
  expect(result).toEqual({ data: 'success' });
  expect(fetchWithRetry).toHaveBeenCalledTimes(3);
});

it('should handle Promise.all correctly', async () => {
  const results = await Promise.all([
    Promise.resolve(1),
    Promise.resolve(2),
    Promise.resolve(3),
  ]);

  expect(results).toEqual([1, 2, 3]);
});
```

### Coverage Guidelines

```bash
# Run with coverage
npm test -- --coverage

# Coverage thresholds
{
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    },
    "./src/services/**/*.ts": {
      "branches": 90,
      "statements": 90
    }
  }
}
```

### What to Test

| Type | What to Test | Don't Test |
|------|-------------|-----------|
| Unit | Pure functions, business logic | Private methods |
| Integration | API endpoints, DB operations | External services |
| E2E | Critical user flows | Every edge case |
| Snapshot | UI components | Logic |

### Best Practices

1. **Test behavior, not implementation** - Test what, not how
2. **One assertion per test** - When practical
3. **Descriptive names** - `it('should throw when...')`
4. **Fast tests** - Avoid I/O, use mocks
5. **Independent tests** - No order dependency
6. **Realistic data** - Use factories, not mocks
7. **AAA pattern** - Clear structure
