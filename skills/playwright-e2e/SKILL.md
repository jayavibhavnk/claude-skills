---
name: playwright-e2e
description: Playwright end-to-end testing - browser automation, page interactions, network mocking, and visual regression testing.
metadata:
  priority: 9
  docs:
    - "https://playwright.dev/docs/intro"
  pathPatterns:
    - "**/*.spec.ts"
    - "**/*.test.ts"
    - "**/e2e/**"
  bashPatterns:
    - '\bplaywright\b'
    - '\be2e\b'
    - '\bbrowser\b.*test'
  promptSignals:
    phrases:
      - "playwright"
      - "e2e testing"
      - "browser automation"
    anyOf:
      - "playwright"
      - "end.to.end"
---

## Playwright E2E Testing

### Setup and Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] },
  ],
});
```

### Basic Test Structure

```typescript
// tests/example.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Homepage', () => {
  test('should display the hero section', async ({ page }) => {
    await page.goto('/');

    await expect(page.locator('h1')).toContainText('Welcome');
    await expect(page.getByRole('button', { name: 'Get Started' })).toBeVisible();
  });

  test('should navigate to about page', async ({ page }) => {
    await page.goto('/');

    await page.getByRole('link', { name: 'About' }).click();

    await expect(page).toHaveURL('/about');
    await expect(page.locator('h1')).toContainText('About Us');
  });
});
```

### Page Interactions

```typescript
test('form interactions', async ({ page }) => {
  await page.goto('/contact');

  // Fill form
  await page.getByLabel('Name').fill('John Doe');
  await page.getByLabel('Email').fill('john@example.com');
  await page.getByLabel('Message').fill('Hello, world!');

  // Select dropdown
  await page.getByLabel('Country').selectOption('US');

  // Check checkbox
  await page.getByLabel('Subscribe').check();

  // Submit
  await page.getByRole('button', { name: 'Send' }).click();

  // Assert success
  await expect(page.locator('.success-message')).toBeVisible();
});
```

### Locators

```typescript
// Role-based (recommended)
await page.getByRole('button', { name: 'Submit' });
await page.getByRole('textbox', { name: 'Email' });
await page.getByRole('heading', { level: 1 });

// Text-based
await page.getByText('Submit');
await page.getByLabel('Password');

// CSS selectors
await page.locator('.btn.primary');
await page.locator('#submit-button');

// Combinations
await page
  .locator('.card')
  .filter({ hasText: 'Featured' })
  .getByRole('button', { name: 'View' });
```

### Network Mocking

```typescript
test('handles API loading state', async ({ page }) => {
  // Mock API response
  await page.route('**/api/users', async (route) => {
    await route.fulfill({
      status: 200,
      body: JSON.stringify([
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' },
      ]),
    });
  });

  await page.goto('/users');

  // Wait for data
  await expect(page.getByText('Alice')).toBeVisible();
  await expect(page.getByText('Bob')).toBeVisible();
});

test('handles API errors', async ({ page }) => {
  await page.route('**/api/users', async (route) => {
    await route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Server error' }),
    });
  });

  await page.goto('/users');

  await expect(page.getByText('Something went wrong')).toBeVisible();
});
```

### Authentication Flows

```typescript
test('login flow', async ({ page }) => {
  // Inject storage state
  await page.addInitScript(() => {
    localStorage.setItem('auth-token', 'test-token-123');
  });

  await page.goto('/dashboard');

  // Should be logged in
  await expect(page.getByText('Welcome')).toBeVisible();
});

test('logout flow', async ({ page }) => {
  await page.goto('/dashboard');
  await page.getByRole('button', { name: 'Logout' }).click();

  await expect(page).toHaveURL('/login');
});
```

### Visual Regression

```typescript
import { test, expect } from '@playwright/test';

test('homepage visual regression', async ({ page }) => {
  await page.goto('/');

  // Take screenshot
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixelRatio: 0.1,
  });
});
```

### Debugging

```typescript
test('debug test', async ({ page }) => {
  // Slow down execution
  test.setTimeout(60000);

  // Add debug logging
  page.on('console', msg => {
    if (msg.type() === 'error') {
      console.log('Browser console error:', msg.text());
    }
  });

  // Take screenshot on failure
  await page.screenshot({ path: 'debug.png' });

  // Trace viewer
  const trace = await page.context().newCDPSession(page);
  await trace.start({ screenshots: true, snapshots: true });
});
```

### Running Tests

```bash
# Run all tests
npx playwright test

# Run specific file
npx playwright test tests/login.spec.ts

# Run with UI
npx playwright test --ui

# Run with headed browser
npx playwright test --headed

# Run specific browser
npx playwright test --project=chromium

# Generate report
npx playwright show-report
```

### Best Practices

1. **Use role locators** - More accessible and stable
2. **Avoid waitFor** - Use expect with auto-waiting
3. **Isolate tests** - Each test independent
4. **Mock APIs** - Don't rely on external services
5. **Screenshots on failure** - Help debug CI issues
6. **Parallel execution** - Faster CI runs
7. **Page objects** - Reduce duplication
