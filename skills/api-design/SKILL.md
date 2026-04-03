---
name: api-design
description: Design RESTful and GraphQL APIs with proper versioning, pagination, error handling, and documentation.
metadata:
  priority: 9
  docs:
    - "https://restfulapi.net"
    - "https://graphql.org/learn/"
  pathPatterns:
    - "**/api/**"
    - "**/routes/**"
  bashPatterns:
    - '\bapi\b'
    - '\bendpoint\b'
    - '\brest\b'
  promptSignals:
    phrases:
      - "api design"
      - "rest api"
      - "endpoint"
    anyOf:
      - "api"
      - "endpoint"
      - "route"
      - "graphql"
---

## API Design

### REST Principles

1. **Resources** - Everything is a resource
2. **URLs** - Nouns, not verbs: `/users` not `/getUsers`
3. **HTTP Methods** - GET (read), POST (create), PUT (replace), PATCH (update), DELETE (delete)
4. **Stateless** - Each request contains all context

### URL Structure

```
https://api.example.com/v1/users/123/posts?page=2&limit=10
\__________/\_/\________/\_/\___/ \______________/
   domain   v  resource   id    action    query params
```

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST creating resource |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Auth but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 422 | Unprocessable | Validation error |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Unexpected error |

### Response Format

```typescript
// Success
{
  "data": {
    "id": "123",
    "name": "John",
    "email": "john@example.com"
  },
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100
  }
}

// Error
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ]
  }
}
```

### Pagination

```typescript
// Cursor-based (recommended for large datasets)
interface PaginatedResponse<T> {
  data: T[];
  cursor: string | null;
  hasMore: boolean;
}

// Offset-based (simpler, good for small)
interface OffsetResponse<T> {
  data: T[];
  page: number;
  limit: number;
  total: number;
  totalPages: number;
}
```

### Versioning

```bash
# URL path (most common)
GET /v1/users
GET /v2/users

# Header (more complex)
GET /users
Accept: application/vnd.api+json, version=2
```

### REST Examples

```typescript
// GET - List users
GET /users?limit=10&offset=0

// GET - Single user
GET /users/123

// POST - Create user
POST /users
Body: { "name": "John", "email": "john@example.com" }

// PUT - Replace user
PUT /users/123
Body: { "name": "John", "email": "john@example.com" }

// PATCH - Update user
PATCH /users/123
Body: { "name": "Jane" }

// DELETE - Delete user
DELETE /users/123
```

### GraphQL

```graphql
# Schema
type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

input CreateUserInput {
  name: String!
  email: String!
}

type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
}

type UserEdge {
  node: User!
  cursor: String!
}
```

### Error Handling

```typescript
// Consistent error response
class ApiError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number,
    public details?: FieldError[]
  ) {
    super(message);
  }
}

interface FieldError {
  field: string;
  message: string;
}

// Usage
throw new ApiError(
  'VALIDATION_ERROR',
  'Invalid input',
  422,
  [{ field: 'email', message: 'Invalid format' }]
);
```

### Best Practices

1. Use plural nouns for collections: `/users` not `/user`
2. Use kebab-case: `/user-profiles` not `/userProfiles`
3. Return appropriate status codes
4. Always include error messages
5. Version your API from day one
6. Document with OpenAPI/Swagger
7. Use pagination for lists
8. Secure with auth (OAuth2/JWT)
9. Rate limit your endpoints
10. Cache appropriately
