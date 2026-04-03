---
name: prisma-patterns
description: Prisma ORM patterns - relations, queries, transactions, and performance optimization.
metadata:
  priority: 8
  docs:
    - "https://www.prisma.io/docs"
  pathPatterns:
    - "**/prisma/**"
    - "prisma/schema.prisma"
  bashPatterns:
    - '\bprisma\b'
  promptSignals:
    phrases:
      - "prisma"
      - "database"
    anyOf:
      - "prisma"
      - "orm"
---

## Prisma Patterns

### Schema Design

```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  posts     Post[]
  profile   Profile?

  @@index([email])
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])

  createdAt DateTime @default(now())

  @@index([authorId])
  @@index([published])
}

model Profile {
  id     String @id @default(cuid())
  bio    String?
  userId String @unique
  user   User   @relation(fields: [userId], references: [id])
}
```

### Relations

```typescript
// Include related records
const users = await prisma.user.findMany({
  include: {
    posts: {
      where: { published: true },
      select: { title: true },
    },
    profile: true,
  },
});

// Nested writes
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
    posts: {
      create: {
        title: 'Hello World',
        published: true,
      },
    },
  },
  include: { posts: true },
});

// Connect existing records
await prisma.post.update({
  where: { id: postId },
  data: {
    author: { connect: { id: userId } },
  },
});
```

### Queries

```typescript
// Pagination
const page = 1;
const limit = 10;
const users = await prisma.user.findMany({
  take: limit,
  skip: (page - 1) * limit,
  orderBy: { createdAt: 'desc' },
});

// Cursor-based pagination
const users = await prisma.user.findMany({
  take: 10,
  skip: 1,
  cursor: { id: lastId },
  orderBy: { id: 'asc' },
});

// Filter with OR
const users = await prisma.user.findMany({
  where: {
    OR: [
      { name: { contains: 'Alice' } },
      { email: { contains: 'alice' } },
    ],
  },
});

// Aggregation
const stats = await prisma.user.aggregate({
  _count: { _all: true },
  _avg: { age: true },
  _sum: { posts: true },
  where: { published: true },
});
```

### Transactions

```typescript
// Simple transaction
const result = await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'bob@example.com', name: 'Bob' },
  });

  await tx.profile.create({
    data: { userId: user.id, bio: 'Hello!' },
  });

  return user;
});

// Interactive transactions ( isolation levels)
const result = await prisma.$transaction(
  async (tx) => {
    // Operations
  },
  {
    isolationLevel: 'Serializable',
    timeout: 5000,
  }
);
```

### Raw Queries

```typescript
// Raw query with parameters
const users = await prisma.$queryRaw`
  SELECT * FROM users
  WHERE email = ${'alice@example.com'}
  AND created_at > ${new Date('2024-01-01')}
`;

// Raw query for aggregations
const result = await prisma.$queryRaw<{ day: Date; count: bigint }[]>`
  SELECT DATE(created_at) as day, COUNT(*) as count
  FROM users
  GROUP BY DATE(created_at)
  ORDER BY day DESC
`;
```

### Performance Tips

```typescript
// Select only needed fields
const emails = await prisma.user.findMany({
  select: { email: true },
});

// Efficient count
const count = await prisma.user.count({
  where: { published: true },
});

// Batch operations
const updates = await Promise.all(
  ids.map((id) =>
    prisma.user.update({
      where: { id },
      data: { published: true },
    })
  )
);
```

### Middleware

```typescript
// Global middleware
prisma.$use(async (params, next) => {
  if (params.model === 'User' && params.action === 'findMany') {
    params.args.where = {
      ...params.args.where,
      deletedAt: null,
    };
  }
  return next(params);
});

// Soft delete example
prisma.$use(async (params, next) => {
  if (params.action === 'delete') {
    params.action = 'update';
    params.args.data = { deletedAt: new Date() };
  }
  if (params.action === 'deleteMany') {
    params.action = 'updateMany';
    params.args.data = { deletedAt: new Date() };
  }
  return next(params);
});
```
