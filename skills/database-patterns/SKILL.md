---
name: database-patterns
description: Implement common database patterns - soft deletes, optimistic locking, auditing, and data versioning.
metadata:
  priority: 8
  docs:
    - "https://www.postgresql.org/docs/current/"
  pathPatterns:
    - "**/models/**"
    - "**/schema/**"
  bashPatterns:
    - '\bprisma\b'
    - '\bdrizzle\b'
    - '\bknex\b'
  promptSignals:
    phrases:
      - "database pattern"
      - "soft delete"
      - "optimistic"
    anyOf:
      - "database"
      - "query"
      - "migration"
---

## Database Patterns

### Soft Delete

```typescript
// Add deletedAt column
interface SoftDelete {
  deletedAt: Date | null;
}

// Query without deleted records
const result = await db.user.findMany({
  where: { deletedAt: null },
});

// Restore deleted record
await db.user.update({
  where: { id: userId, deletedAt: null },
  data: { deletedAt: null },
});

// Global scope (Prisma example)
prisma.user.addMiddleware(async (params, next) => {
  if (params.action === 'findMany' && params.model === 'User') {
    params.args.where = { ...params.args.where, deletedAt: null };
  }
  return next(params);
});
```

### Optimistic Locking

```typescript
// Version column approach
interface VersionedEntity {
  id: string;
  version: number;
}

async function updateWithLock<T extends VersionedEntity>(
  table: string,
  id: string,
  update: Partial<T>
): Promise<T> {
  const current = await db[table].findUnique({ where: { id } });

  const result = await db[table].update({
    where: {
      id,
      version: current.version,  // Lock check
    },
    data: {
      ...update,
      version: current.version + 1,
    },
  });

  if (!result) {
    throw new Error('Concurrent modification detected');
  }

  return result;
}
```

### Audit Trail

```typescript
// Generic audit log
interface AuditLog {
  id: string;
  tableName: string;
  recordId: string;
  action: 'CREATE' | 'UPDATE' | 'DELETE';
  oldValues: any;
  newValues: any;
  changedBy: string;
  changedAt: Date;
}

// Database trigger or ORM middleware
prisma.$use(async (params, next) => {
  const before = params.action === 'update' || params.action === 'delete'
    ? await (prisma as any)[params.model].findUnique({ where: params.args.where })
    : null;

  const result = await next(params);

  if (['create', 'update', 'delete'].includes(params.action)) {
    await prisma.auditLog.create({
      data: {
        tableName: params.model,
        recordId: params.args.where?.id || result?.id,
        action: params.action.toUpperCase(),
        oldValues: before,
        newValues: result,
        changedBy: getCurrentUserId(),
        changedAt: new Date(),
      },
    });
  }

  return result;
});
```

### UUID Primary Keys

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Use UUID as primary key
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Or use cuid2 for better indexing
-- id TEXT PRIMARY KEY DEFAULT cuid2()
```

### Timestamps Pattern

```typescript
// Automatic timestamps
class TimestampMixin {
  createdAt: Date;
  updatedAt: Date;
}

// Update updatedAt on every change
prisma.$use(async (params, next) => {
  if (['update', 'updateMany'].includes(params.action)) {
    params.args.data = {
      ...params.args.data,
      updatedAt: new Date(),
    };
  }
  return next(params);
});
```

### Soft Deletes with Query Scope

```typescript
// Prisma scope
prisma.$use(async (params, next) => {
  // Skip deleted for find operations
  if (params.action === 'find') {
    if (!params.args) params.args = {};
    params.args.where = { ...params.args.where, deletedAt: null };
  }
  return next(params);
});

// Direct queries still work
await prisma.user.findMany({
  where: { deletedAt: { not: null } },  // Include deleted
});
```

### Row-Level Security (PostgreSQL)

```sql
-- Enable RLS
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Users can only see their own orders
CREATE POLICY user_orders ON orders
  FOR ALL
  USING (user_id = current_user_id());

-- Admin can see all
CREATE POLICY admin_orders ON orders
  FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM users
      WHERE users.id = current_user_id()
      AND users.role = 'admin'
    )
  );
```

### Pagination Patterns

```typescript
// Cursor-based (best for large tables)
async function getUsers(cursor?: string, limit = 10) {
  return db.user.findMany({
    take: limit + 1,
    ...(cursor && {
      skip: 1,
      cursor: { id: cursor },
    }),
    orderBy: { createdAt: 'desc' },
  });
}

// Offset-based (simpler, for small tables)
async function getUsers(page = 1, limit = 10) {
  return db.user.findMany({
    skip: (page - 1) * limit,
    take: limit,
    orderBy: { createdAt: 'desc' },
  });
}
```

### Best Practices

1. **Always use transactions** - For multi-table operations
2. **Index wisely** - On foreign keys and WHERE columns
3. **Soft delete** - Instead of hard delete for auditing
4. **Optimistic locking** - Prevent lost updates
5. **UUID keys** - Safer than sequential integers
6. **Audit logs** - Track all data changes
7. **Pagination** - Cursor-based for large datasets
