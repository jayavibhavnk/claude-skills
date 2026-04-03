---
name: api-versioning
description: API versioning strategies - URL path, header, query params, and migration patterns.
metadata:
  priority: 8
  docs:
    - "https://apiux.com/2013/04/09/api-versioning"
  pathPatterns:
    - "**/api/**"
    - "**/routes/**"
  bashPatterns:
    - '\bversioning\b'
  promptSignals:
    phrases:
      - "versioning"
      - "api version"
    anyOf:
      - "version"
      - "migration"
---

## API Versioning

### Versioning Strategies

#### 1. URL Path (Most Common)

```bash
# Resources versioned
GET /api/v1/users
GET /api/v2/users

# Specific versions
GET /api/v1/users/123
DELETE /api/v2/users/456
```

#### 2. Header

```bash
# Accept header
curl -H "Accept: application/vnd.api.v2+json" /api/users

# Custom header
curl -H "API-Version: 2024-01-01" /api/users
```

#### 3. Query Parameter

```bash
GET /api/users?version=2
GET /api/users?api_version=2
```

### Implementation Patterns

#### Express Version Router

```typescript
// routes/v1/users.ts
const router = express.Router();

router.get('/', async (req, res) => {
  const users = await db.user.findMany();
  res.json({ users, version: 'v1' });
});

router.get('/:id', async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.params.id } });
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json({ user, version: 'v1' });
});

export default router;

// routes/index.ts
import v1 from './v1/users';
import v2 from './v2/users';

app.use('/api/v1/users', v1);
app.use('/api/v2/users', v2);
```

#### Version Middleware

```typescript
// middleware/version.ts
app.use('/api', (req, res, next) => {
  const version = req.headers['accept']?.match(/v(\d+)/)?.[1] || '1';
  req.apiVersion = parseInt(version);
  next();
});

// In routes
app.get('/api/users', async (req, res) => {
  if (req.apiVersion >= 2) {
    // Include additional fields
    const users = await db.user.findMany({
      include: { profile: true, posts: true },
    });
    return res.json({ users, version: 'v2' });
  }

  const users = await db.user.findMany();
  return res.json({ users, version: 'v1' });
});
```

### Field Deprecation

```typescript
// v1 response
{ "user": { "id": 1, "name": "John", "email": "john@example.com" } }

// v2 response (renamed, deprecated old)
{
  "user": {
    "id": 1,
    "fullName": "John",  // New field
    "email": "john@example.com",
    "name": "John (deprecated)",  // Marked as deprecated
    "_deprecations": {
      "name": "Use fullName instead"
    }
  }
}
```

### Response Transformation

```typescript
// Transform v1 to v2
function transformUserV1ToV2(user: any) {
  return {
    ...user,
    fullName: user.name,
    name: undefined,  // Remove old field
  };
}

// Auto-transform based on version
app.get('/api/users/:id', async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.params.id } });

  if (req.apiVersion >= 2) {
    res.json(transformUserV1ToV2(user));
  } else {
    res.json(user);
  }
});
```

### Deprecation Timeline

```markdown
## Version Lifecycle

1. **Current** - Fully supported, receives new features
2. **Deprecated** - Still works, shows warning in response headers
3. ** sunset** - Will be removed, shows deprecation warning
4. **Removed** - Returns 410 Gone

## Headers

X-API-Version: current
X-API-Deprecated: true
X-API-Sunset: Sat, 01 Jun 2024 00:00:00 GMT
X-API-Deprecation-Info: https://docs.example.com/changes
```

### Graceful Migration

```typescript
// Support both old and new
app.put('/api/users/:id', async (req, res) => {
  // New format
  if (req.body.fullName) {
    req.body.name = req.body.fullName;
  }

  const user = await db.user.update({
    where: { id: req.params.id },
    data: req.body,
  });

  res.json(user);
});
```

### Testing Versions

```typescript
describe('API v1', () => {
  it('returns basic user fields', async () => {
    const res = await request(app)
      .get('/api/v1/users/1')
      .set('Accept', 'application/vnd.api.v1+json');

    expect(res.body.user).toHaveProperty('name');
    expect(res.body.user).not.toHaveProperty('fullName');
  });
});

describe('API v2', () => {
  it('returns extended user fields', async () => {
    const res = await request(app)
      .get('/api/v2/users/1')
      .set('Accept', 'application/vnd.api.v2+json');

    expect(res.body.user).toHaveProperty('fullName');
    expect(res.body.user).toHaveProperty('profile');
  });
});
```

### Best Practices

1. **Version in URL** - Most explicit and discoverable
2. **Start with v1** - Don't assume initial API is perfect
3. **Add rather than replace** - New fields, never remove
4. **Communicate changes** - Changelog, migration guides
5. **Set deprecation timeline** - Give consumers time to adapt
6. **Monitor usage** - Track which versions are still in use
7. **Maintain old versions** - Give time for migration
