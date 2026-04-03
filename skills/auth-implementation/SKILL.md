---
name: auth-implementation
description: Implement authentication - JWT, OAuth, sessions, password hashing, and secure token management.
metadata:
  priority: 9
  docs:
    - "https://auth0.com/docs"
  pathPatterns:
    - "**/auth/**"
    - "**/middleware/**"
  bashPatterns:
    - '\bjwt\b'
    - '\boauth\b'
    - '\bauth\b'
  promptSignals:
    phrases:
      - "authentication"
      - "auth"
      - "login"
    anyOf:
      - "auth"
      - "jwt"
      - "oauth"
---

## Auth Implementation

### Password Security

```typescript
import { hash, verify } from 'argon2';

async function hashPassword(password: string): Promise<string> {
  return hash(password, {
    type: argon2id,
    memoryCost: 65536,  // 64MB
    timeCost: 3,
    parallelism: 4,
  });
}

async function verifyPassword(
  password: string,
  hashedPassword: string
): Promise<boolean> {
  return verify(hashedPassword, password);
}
```

### JWT Tokens

```typescript
import jwt from 'jsonwebtoken';

interface TokenPayload {
  userId: string;
  email: string;
  role: string;
}

// Access token (short-lived)
function createAccessToken(payload: TokenPayload): string {
  return jwt.sign(payload, process.env.JWT_SECRET!, {
    expiresIn: '15m',
    algorithm: 'HS256',
  });
}

// Refresh token (long-lived)
function createRefreshToken(payload: TokenPayload): string {
  return jwt.sign(
    { ...payload, type: 'refresh' },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: '7d' }
  );
}

// Verify token
function verifyToken(token: string): TokenPayload {
  try {
    return jwt.verify(token, process.env.JWT_SECRET!) as TokenPayload;
  } catch {
    throw new Error('Invalid token');
  }
}
```

### OAuth 2.0 Flow

```typescript
// 1. Redirect to OAuth provider
function getGoogleAuthUrl(): string {
  const params = new URLSearchParams({
    client_id: process.env.GOOGLE_CLIENT_ID!,
    redirect_uri: 'https://myapp.com/auth/google/callback',
    response_type: 'code',
    scope: 'openid email profile',
    access_type: 'offline',
  });

  return `https://accounts.google.com/o/oauth2/v2/auth?${params}`;
}

// 2. Exchange code for tokens
async function handleGoogleCallback(code: string) {
  const tokens = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      code,
      client_id: process.env.GOOGLE_CLIENT_ID!,
      client_secret: process.env.GOOGLE_CLIENT_SECRET!,
      redirect_uri: 'https://myapp.com/auth/google/callback',
      grant_type: 'authorization_code',
    }),
  }).then(r => r.json());

  // Get user info
  const userInfo = await fetch('https://www.googleapis.com/oauth2/v2/userinfo', {
    headers: { Authorization: `Bearer ${tokens.access_token}` },
  }).then(r => r.json());

  return userInfo;
}
```

### Session Management

```typescript
import session from 'express-session';

app.use(session({
  secret: process.env.SESSION_SECRET!,
  name: 'session_id',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
  },
}));

// Store sessions in Redis
import RedisStore from 'connect-redis';

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
}));
```

### Middleware

```typescript
// JWT verification middleware
function authenticateToken(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];  // Bearer TOKEN

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  try {
    const payload = verifyToken(token);
    req.user = payload;
    next();
  } catch {
    return res.status(403).json({ error: 'Invalid or expired token' });
  }
}

// Role-based access
function requireRole(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Usage
app.delete('/users/:id',
  authenticateToken,
  requireRole('admin'),
  deleteUserHandler
);
```

### Token Refresh

```typescript
async function refreshTokens(refreshToken: string) {
  try {
    // Verify refresh token
    const payload = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET!);

    if ((payload as any).type !== 'refresh') {
      throw new Error('Invalid token type');
    }

    // Generate new tokens
    return {
      accessToken: createAccessToken({
        userId: payload.userId,
        email: payload.email,
        role: payload.role,
      }),
      refreshToken: createRefreshToken({
        userId: payload.userId,
        email: payload.email,
        role: payload.role,
      }),
    };
  } catch {
    throw new Error('Invalid refresh token');
  }
}
```

### Security Headers

```typescript
import helmet from 'helmet';

app.use(helmet());

// CORS
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(','),
  credentials: true,
}));

// Rate limiting
import rateLimit from 'express-rate-limit';

app.use('/api/auth/login', rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,  // 5 attempts per window
  message: { error: 'Too many login attempts' },
}));
```

### Best Practices

1. **Hash passwords** - Use argon2 or bcrypt
2. **Short-lived access tokens** - 15 minutes max
3. **Secure refresh tokens** - httpOnly cookies
4. **Rate limit auth endpoints** - Prevent brute force
5. **Use HTTPS** - Always in production
6. **Validate tokens** - Every request
7. **Implement logout** - Invalidate tokens
8. **Rotate secrets** - Regular key rotation
