---
name: contract-testing
description: API contract testing - consumer-driven contracts, Pact, and integration testing.
metadata:
  priority: 8
  docs:
    - "https://docs.pact.io"
  pathPatterns:
    - "**/*.pact.ts"
    - "**/pact/**"
  bashPatterns:
    - '\bpact\b'
    - '\bcontract.?test'
  promptSignals:
    phrases:
      - "contract testing"
      - "pact"
    anyOf:
      - "contract"
      - "pact"
---

## Contract Testing

### What is Contract Testing?

Contract testing ensures APIs match expectations without full integration testing.

```
┌──────────────┐         Contract          ┌──────────────┐
│   Consumer   │ ◀────────────────────────▶ │   Provider   │
│   (Client)   │                            │   (Server)   │
└──────────────┘                            └──────────────┘
       │                                            │
       │  1. Write expectations (Pact file)         │
       │───────────────────────────────────────────▶│
       │                                            │ 2. Verify against running server
       │◀──────────────────────────────────────────│
       │         3. Pact Broker stores contract     │
```

### Consumer-Side (Pact)

```typescript
// consumer.pact.spec.ts
import { PactV3, MatchersV3 } from '@pact-foundation/pact';

const { like, eachLike, string } = MatchersV3;

const provider = new PactV3({
  consumer: 'web-app',
  provider: 'user-service',
  dir: './pacts',
  logLevel: 'warn',
});

describe('User Service Consumer', () => {
  it('gets a list of users', async () => {
    await provider.addInteraction({
      states: [{ description: 'users exist' }],
      uponReceiving: 'a request for users',
      withRequest: {
        method: 'GET',
        path: '/api/users',
        headers: { Accept: 'application/json' },
      },
      willRespondWith: {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: {
          users: eachLike({
            id: string('1'),
            name: string('John Doe'),
            email: string('john@example.com'),
          }),
        },
      },
    });

    await provider.executeTest(async (mockServer) => {
      const response = await fetch(`${mockServer.url}/api/users`);
      const data = await response.json();

      expect(data.users).toHaveLength(3);
      expect(data.users[0]).toHaveProperty('email');
    });
  });
});
```

### Provider-Side Verification

```typescript
// provider.pact.spec.ts
import { Verifier } from '@pact-foundation/pact';

const verifier = new Verifier({
  provider: 'user-service',
  providerBaseUrl: 'http://localhost:3000',
  pactBrokerUrl: 'https://pactbroker.example.com',
  publishVerificationResult: true,
  providerVersion: '1.0.0',
});

verifier.verifyProvider().then(() => {
  console.log('Pact verification complete!');
});
```

### Contract Examples

```typescript
// REST API contract
await provider.addInteraction({
  withRequest: {
    method: 'POST',
    path: '/api/users',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': string('Bearer token'),
    },
    body: {
      name: string('Jane'),
      email: string('jane@example.com'),
    },
  },
  willRespondWith: {
    status: 201,
    body: {
      id: string('123'),
      name: string('Jane'),
      createdAt: string('2024-01-15T10:00:00Z'),
    },
  },
});

// Error response contract
await provider.addInteraction({
  withRequest: {
    method: 'POST',
    path: '/api/users',
    body: { email: 'invalid' },
  },
  willRespondWith: {
    status: 400,
    body: {
      error: string('Invalid email format'),
      field: string('email'),
    },
  },
});
```

### Matchers

```typescript
import { like, eachLike, string, integer, boolean, datetime } from '@pact-foundation/pact';

const commonMatchers = {
  // Flexible matching
  id: like('abc123'),
  email: like('test@example.com'),
  name: string('John'),

  // Arrays
  tags: eachLike('important'),
  users: eachLike({
    id: string('1'),
    name: string('User'),
  }),

  // Numeric
  count: integer(42),
  price: like(9.99),

  // Type matching
  active: boolean(true),
  createdAt: datetime('2024-01-01T00:00:00Z'),
};
```

### State Setup

```typescript
// Provider states
const states = [
  { description: 'user with id 1 exists', states: { userId: '1' } },
  { description: 'no users exist', states: { userCount: 0 } },
  { description: 'user is deleted', states: { userDeleted: true } },
];

await provider.addInteraction({
  states: [{ description: 'users exist' }],
  // ...
});

// Provider handles state
// In provider test setup
if (state === 'users exist') {
  await db.seed({ users: [{ id: '1', name: 'Test' }] });
}
```

### CI Integration

```yaml
# GitHub Actions
name: Contract Tests
on: [push, pull_request]

jobs:
  consumer-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm test
        env:
          PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}

  provider-verify:
    needs: consumer-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run pact:verify
        env:
          PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}
```

### Best Practices

1. **Consumer-driven** - Consumer defines expectations
2. **Independent testing** - No full integration needed
3. **Version control contracts** - Store pact files in repo
4. **Broker for sharing** - Centralize contracts
5. **Test states explicitly** - Setup/teardown for each test
6. **Keep contracts small** - One interaction per test
7. **Version contracts** - Match API versions
