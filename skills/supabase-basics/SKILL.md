---
name: supabase-basics
description: Work with Supabase - PostgreSQL database, authentication, real-time subscriptions, and edge functions.
metadata:
  priority: 8
  docs:
    - "https://supabase.com/docs"
    - "https://supabase.com/docs/reference/python"
  pathPatterns:
    - "**/supabase/**"
    - "**/lib/db/**"
    - "**/database/**"
  bashPatterns:
    - '\bsupabase\b'
    - '\b@supabase\b'
    - '\bsupabase-js\b'
  promptSignals:
    phrases:
      - "supabase"
      - "postgres database"
    anyOf:
      - "database"
      - "auth"
      - "realtime"
      - "storage"
      - "edge functions"
---

## Supabase

### Client Setup

```typescript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

### Database Operations

```typescript
// Select
const { data: users, error } = await supabase
  .from('users')
  .select('*')
  .eq('email', 'user@example.com');

// Insert
const { data, error } = await supabase
  .from('users')
  .insert({ email: 'new@example.com', name: 'New User' })
  .select();

// Update
const { data, error } = await supabase
  .from('users')
  .update({ name: 'Updated Name' })
  .eq('id', userId)
  .select();

// Delete
const { error } = await supabase
  .from('users')
  .delete()
  .eq('id', userId);
```

### Query Building

```typescript
// Complex queries
const { data } = await supabase
  .from('posts')
  .select(`
    id,
    title,
    content,
    author:users(name, avatar)
  `)
  .eq('published', true)
  .order('created_at', { ascending: false })
  .range(0, 9);
```

### Authentication

```typescript
// Sign up
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password',
});

// Sign in
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'secure-password',
});

// Sign out
await supabase.auth.signOut();

// Get current user
const { data: { user } } = await supabase.auth.getUser();
```

### Realtime Subscriptions

```typescript
// Subscribe to table changes
const channel = supabase
  .channel('posts')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'posts' },
    (payload) => {
      console.log('Change received!', payload);
    }
  )
  .subscribe();

// Unsubscribe
channel.unsubscribe();
```

### Storage

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from('avatars')
  .upload('user-123/avatar.jpg', file);

// Get public URL
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl('user-123/avatar.jpg');

// Download
const { data, error } = await supabase.storage
  .from('avatars')
  .download('user-123/avatar.jpg');
```

### Edge Functions

```typescript
// supabase/functions/hello-world/index.ts
import { serve } from 'https://deno.land/std@0.177.0/http/server.ts';

serve(async (req) => {
  const { name } = await req.json();

  return new Response(
    JSON.stringify({ message: `Hello ${name}!` }),
    { headers: { 'Content-Type': 'application/json' } }
  );
});
```

## Row Level Security (RLS)

```sql
-- Enable RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policy for users to read their own data
CREATE POLICY "Users can view own data"
  ON users FOR SELECT
  USING (auth.uid() = user_id);

-- Policy for users to update their own data
CREATE POLICY "Users can update own data"
  ON users FOR UPDATE
  USING (auth.uid() = user_id);
```

## Best Practices

1. Always enable RLS on tables
2. Use typed clients for better DX
3. Leverage realtime for live updates
4. Use Supabase auth for secure authentication
5. Implement proper indexes for query performance
