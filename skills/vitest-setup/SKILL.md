---
name: vitest-setup
description: Set up and configure Vitest for modern JavaScript/TypeScript testing with Vite.
metadata:
  priority: 8
  docs:
    - "https://vitest.dev/guide/"
  pathPatterns:
    - "vitest.config.ts"
    - "vite.config.ts"
  bashPatterns:
    - '\bvitest\b'
    - '\bvite\b'
  promptSignals:
    phrases:
      - "vitest"
      - "vite test"
    anyOf:
      - "testing"
      - "test"
---

## Vitest Setup

### Installation

```bash
npm install -D vitest @vitejs/plugin-react
npm install -D jsdom @vitest/ui
```

### Basic Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{js,ts}'],
  },
});
```

### React Testing

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { expect, afterEach } from 'vitest';
import { cleanup } from '@testing-library/react';

afterEach(() => {
  cleanup();
});
```

```typescript
// src/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' }));
  });

  it('calls onClick handler', async () => {
    const handler = vi.fn();
    render(<Button onClick={handler}>Click</Button>);
    fireEvent.click(screen.getByRole('button'));
    expect(handler).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(<Button loading>Click</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### TypeScript Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['src/**/*.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      include: ['src/**/*.ts'],
      exclude: ['src/**/*.d.ts'],
    },
  },
});
```

### Mocking

```typescript
import { vi, describe, it, expect } from 'vitest';

// Module mock
vi.mock('./api', () => ({
  fetchUser: vi.fn(),
}));

// Mock implementation
vi.mock('./api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: '1', name: 'Test' }),
}));

// Spy
const consoleSpy = vi.spyOn(console, 'log').mockImplementation(() => {});
```

### Testing Async

```typescript
it('handles async errors', async () => {
  await expect(fetchUser('invalid')).rejects.toThrow('Not found');
});

it('loads data asynchronously', async () => {
  render(<UserProfile userId="123" />);
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
});
```

### DOM Testing

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('submits form with user interaction', async () => {
  const user = userEvent.setup();
  render(<LoginForm onSubmit={fn} />);

  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password123');
  await user.click(screen.getByRole('button', { name: 'Submit' }));

  expect(fn).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  });
});
```
