---
name: error-handling
description: Implement robust error handling - typed errors, retry logic, fallback patterns, and graceful degradation.
metadata:
  priority: 9
  docs:
    - "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error"
  pathPatterns:
    - "**/error/**"
    - "**/errors/**"
  bashPatterns:
    - '\bthrow\b'
    - '\bcatch\b'
    - '\bretry\b'
  promptSignals:
    phrases:
      - "error handling"
      - "retry"
      - "fallback"
    anyOf:
      - "error"
      - "exception"
      - "retry"
---

## Error Handling

### Typed Errors

```typescript
// Custom error hierarchy
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public details?: any
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message: string, details?: any) {
    super(message, 'VALIDATION_ERROR', 400, details);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} not found`, 'NOT_FOUND', 404, { resource, id });
  }
}

class UnauthorizedError extends AppError {
  constructor() {
    super('Unauthorized', 'UNAUTHORIZED', 401);
  }
}

class RateLimitError extends AppError {
  constructor(retryAfter: number) {
    super('Rate limit exceeded', 'RATE_LIMIT', 429, { retryAfter });
  }
}
```

### Error Handling Middleware

```typescript
// Express error middleware
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        details: err.details,
      },
    });
  }

  // Unknown error
  console.error('Unexpected error:', err);
  return res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
    },
  });
});
```

### Result Type Pattern

```typescript
// Rust-like Result type
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

async function tryCatch<T>(
  promise: Promise<T>
): Promise<Result<T, Error>> {
  try {
    const value = await promise;
    return { ok: true, value };
  } catch (error) {
    return { ok: false, error: error as Error };
  }
}

// Usage
const result = await tryCatch(fetchUser(userId));

if (result.ok) {
  console.log(result.value);
} else {
  console.error(result.error);
}
```

### Retry with Backoff

```typescript
interface RetryOptions {
  maxAttempts: number;
  initialDelay: number;
  maxDelay: number;
  backoff: 'fixed' | 'exponential';
  retryable?: (error: Error) => boolean;
}

async function withRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions
): Promise<T> {
  let lastError: Error;
  let delay = options.initialDelay;

  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (options.retryable && !options.retryable(lastError)) {
        throw lastError;
      }

      if (attempt === options.maxAttempts) break;

      await sleep(delay);
      delay = options.backoff === 'exponential'
        ? Math.min(delay * 2, options.maxDelay)
        : delay;
    }
  }

  throw lastError!;
}

// Usage with retryable errors
async function fetchWithRetry(url: string) {
  return withRetry(
    () => fetch(url).then(r => {
      if (r.status >= 500) throw new Error(`Server error: ${r.status}`);
      return r.json();
    }),
    {
      maxAttempts: 3,
      initialDelay: 1000,
      maxDelay: 10000,
      backoff: 'exponential',
      retryable: (err) => err.message.startsWith('Server error'),
    }
  );
}
```

### Circuit Breaker

```typescript
class CircuitBreaker {
  private failures = 0;
  private lastFailure = 0;
  private state: 'closed' | 'open' | 'half-open' = 'closed';

  constructor(
    private threshold: number = 5,
    private timeout: number = 60000  // 1 minute
  ) {}

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailure > this.timeout) {
        this.state = 'half-open';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'closed';
  }

  private onFailure() {
    this.failures++;
    this.lastFailure = Date.now();

    if (this.failures >= this.threshold) {
      this.state = 'open';
    }
  }
}
```

### Fallback Patterns

```typescript
// Primary with fallback
async function getData() {
  try {
    // Try primary source
    return await fetchFromPrimary();
  } catch (error) {
    console.warn('Primary source failed, trying fallback:', error);
    return fetchFromFallback();
  }
}

// Cache fallback
async function getWithCacheFallback(key: string) {
  try {
    const data = await fetchFresh(key);
    await cache.set(key, data, { ex: 300 });
    return data;
  } catch (error) {
    const cached = await cache.get(key);
    if (cached) {
      console.warn('Using stale cache due to error:', error);
      return cached;
    }
    throw error;
  }
}

// Graceful degradation
async function getUserData(userId: string) {
  const [profile, settings, history] = await Promise.allSettled([
    fetchProfile(userId),
    fetchSettings(userId),
    fetchHistory(userId),
  ]);

  return {
    profile: profile.status === 'fulfilled' ? profile.value : null,
    settings: settings.status === 'fulfilled' ? settings.value : defaultSettings,
    history: history.status === 'fulfilled' ? history.value : [],
  };
}
```

### Global Error Handler (Node.js)

```typescript
process.on('uncaughtException', (error) => {
  console.error('UNCAUGHT EXCEPTION:', error);
  // Log to monitoring
  // Exit gracefully
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('UNHANDLED REJECTION at:', promise, 'reason:', reason);
  // Log to monitoring
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

### Best Practices

1. **Typed errors** - Use custom error classes
2. **Error boundaries** - Catch at component/service level
3. **Retry intelligently** - Exponential backoff for transient errors
4. **Circuit breakers** - Prevent cascade failures
5. **Fallbacks** - Degrade gracefully, don't fail completely
6. **Log context** - Include correlation IDs, user info
7. **Never swallow errors** - Either handle or re-throw
