---
name: graphql-patterns
description: Build GraphQL APIs - schemas, resolvers, pagination, authentication, and performance optimization.
metadata:
  priority: 8
  docs:
    - "https://graphql.org/learn/"
    - "https://www.apollographql.com/docs/"
  pathPatterns:
    - "**/graphql/**"
    - "**/schema/**"
  bashPatterns:
    - '\bgraphql\b'
    - '\bapollo\b'
  promptSignals:
    phrases:
      - "graphql"
      - "apollo"
      - "schema"
    anyOf:
      - "graphql"
      - "query"
      - "mutation"
---

## GraphQL Patterns

### Schema Design

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  avatar: String
  createdAt: DateTime!
  posts: [Post!]!
  postCount: Int!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  createdAt: DateTime!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  createdAt: DateTime!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
  post(id: ID!): Post
  postsByAuthor(authorId: ID!): [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  name: String!
  email: String!
}

input UpdateUserInput {
  name: String
  avatar: String
}
```

### Resolver Implementation

```typescript
const resolvers = {
  Query: {
    user: async (_, { id }, { db }) => {
      return db.users.findUnique({ where: { id } });
    },

    users: async (_, { limit = 10, offset = 0 }, { db }) => {
      return db.users.findMany({
        take: limit,
        skip: offset,
        orderBy: { createdAt: 'desc' },
      });
    },
  },

  Mutation: {
    createUser: async (_, { input }, { db }) => {
      return db.users.create({ data: input });
    },

    updateUser: async (_, { id, input }, { db }) => {
      return db.users.update({
        where: { id },
        data: input,
      });
    },
  },

  // Field resolvers
  User: {
    posts: async (parent, _, { db }) => {
      return db.posts.findMany({
        where: { authorId: parent.id },
      });
    },

    postCount: async (parent, _, { db }) => {
      return db.posts.count({
        where: { authorId: parent.id },
      });
    },
  },

  Post: {
    author: async (parent, _, { db }) => {
      return db.users.findUnique({ where: { id: parent.authorId } });
    },
  },
};
```

### Cursor Pagination

```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type Query {
  users(first: Int, after: String, last: Int, before: String): UserConnection!
}
```

```typescript
const resolvers = {
  Query: {
    users: async (_, { first = 10, after, last, before }, { db }) => {
      const limit = first || last || 10;
      const cursor = after || before;

      const where = cursor
        ? { id: { [before ? 'gt' : 'lt']: cursor } }
        : {};

      const users = await db.users.findMany({
        take: (first ? 1 : -1) * limit,
        where,
        orderBy: { id: 'asc' },
      });

      const edges = users.map(user => ({
        node: user,
        cursor: Buffer.from(user.id).toString('base64'),
      }));

      return {
        edges,
        pageInfo: {
          hasNextPage: users.length === limit,
          hasPreviousPage: !!after,
          startCursor: edges[0]?.cursor,
          endCursor: edges[edges.length - 1]?.cursor,
        },
        totalCount: await db.users.count(),
      };
    },
  },
};
```

### Data Loader (N+1 Prevention)

```typescript
import DataLoader from 'dataloader';

// Batch function for posts
const createPostsLoader = () => new DataLoader(async (authorIds: string[]) => {
  const posts = await db.posts.findMany({
    where: { authorId: { in: authorIds } },
  });

  // Group by authorId
  const postsByAuthor = groupBy(posts, 'authorId');
  return authorIds.map(id => postsByAuthor[id] || []);
});

// In resolvers
const resolvers = {
  User: {
    posts: async (parent, _, { loaders }) => {
      return loaders.postsByAuthor.load(parent.id);
    },
  },
};
```

### Authentication

```typescript
const resolvers = {
  context: async ({ req }) => {
    const token = req.headers.authorization?.replace('Bearer ', '');

    if (!token) {
      return { user: null };
    }

    try {
      const user = await verifyJWT(token);
      return { user };
    } catch {
      return { user: null };
    }
  },

  Query: {
    me: async (_, __, { user }) => {
      if (!user) {
        throw new Error('Authentication required');
      }
      return user;
    },
  },

  Mutation: {
    login: async (_, { email, password }) => {
      const user = await authenticateUser(email, password);
      const token = createJWT(user);
      return { token, user };
    },
  },
};
```

### Error Handling

```typescript
class GraphQLError extends Error {
  constructor(
    message: string,
    public extensions?: {
      code: string;
      field?: string;
      details?: any;
    }
  ) {
    super(message);
  }
}

// In resolver
const resolvers = {
  Mutation: {
    createUser: async (_, { input }, { db }) => {
      const existing = await db.users.findUnique({
        where: { email: input.email },
      });

      if (existing) {
        throw new GraphQLError('Email already exists', {
          code: 'DUPLICATE_EMAIL',
          field: 'email',
        });
      }

      return db.users.create({ data: input });
    },
  },
};
```

### Best Practices

1. **Schema-first** - Design before implementing
2. **Use DataLoader** - Prevent N+1 queries
3. **Cursor pagination** - Better than offset for large datasets
4. **Field resolvers** - Keep business logic separate
5. **Input types** - For complex mutations
6. **Error extensions** - Rich error information
7. **Authentication context** - User info in every resolver
