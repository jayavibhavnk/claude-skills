---
name: supabase-advanced
description: Advanced Supabase patterns - Row Level Security, Edge Functions, Realtime subscriptions, and complex queries.
metadata:
  priority: 8
  docs:
    - "https://supabase.com/docs"
  pathPatterns:
    - "**/supabase/**"
    - "**/functions/**"
  bashPatterns:
    - '\bsupabase\b'
  promptSignals:
    phrases:
      - "supabase"
      - "edge functions"
      - "rls"
    anyOf:
      - "supabase"
      - "postgres"
---

## Supabase Advanced

### Row Level Security (RLS)

```sql
-- Enable RLS on table
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Users can read own profile
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = user_id);

-- Users can update own profile
CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- Public read, authenticated write
CREATE POLICY "Public profiles are viewable"
  ON profiles FOR SELECT
  USING (true);

CREATE POLICY "Authenticated users can insert"
  ON profiles FOR INSERT
  WITH CHECK (auth.role() = 'authenticated');

-- Role-based access
CREATE POLICY "Admins can do anything"
  ON profiles FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_id = auth.uid() AND role = 'admin'
    )
  );
```

### Complex Queries

```typescript
// Hierarchical data with CTEs
const { data } = await supabase.rpc('get_org_tree', {
  root_id: 'company-uuid'
});

// Postgres function for tree
/*
CREATE FUNCTION get_org_tree(root_id UUID)
RETURNS TABLE (
  id UUID,
  name TEXT,
  parent_id UUID,
  depth INT
) AS $$
WITH RECURSIVE tree AS (
  SELECT id, name, parent_id, 0 AS depth
  FROM organizations
  WHERE id = root_id
  UNION ALL
  SELECT o.id, o.name, o.parent_id, t.depth + 1
  FROM organizations o
  JOIN tree t ON o.parent_id = t.id
)
SELECT * FROM tree;
$$ LANGUAGE plpgsql;
*/

// Full-text search
const { data } = await supabase
  .from('posts')
  .select('*')
  .textSearch('title', 'supabase', {
    type: 'websearch',
    config: 'english'
  });

// Polymorphic relations
const { data } = await supabase
  .from('comments')
  .select(`
    *,
    commentable_type,
    commentable_id,
    user:users(name)
  `)
  .eq('commentable_type', 'posts')
  .eq('commentable_id', postId);
```

### Edge Functions

```typescript
// supabase/functions/send-email/index.ts
import { serve } from 'https://servestjs.org/@v2023.01.27/mod.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

serve(async (req) => {
  const { email, template } = await req.json();

  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  // Process email
  await sendEmailViaProvider({ to: email, template });

  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

### Realtime Subscriptions

```typescript
// Subscribe to table changes
const channel = supabase
  .channel('schema-db-changes')
  .on(
    'postgres_changes',
    {
      event: '*',
      schema: 'public',
      table: 'messages',
      filter: 'room_id=eq.123'
    },
    (payload) => {
      console.log('Change received:', payload);
    }
  )
  .subscribe();

// Presence for online status
const presenceChannel = supabase.channel('room-1', {
  config: { presence: { key: 'user-123' } }
});

presenceChannel
  .on('presence', { event: 'sync' }, () => {
    const state = presenceChannel.presenceState();
    console.log('Online users:', Object.keys(state));
  })
  .on('presence', { event: 'join' }, ({ key, newPresences }) => {
    console.log('User joined:', key, newPresences);
  })
  .on('presence', { event: 'leave' }, ({ key, leftPresences }) => {
    console.log('User left:', key, leftPresences);
  })
  .subscribe(async (status) => {
    if (status === 'SUBSCRIBED') {
      await presenceChannel.track({ user_id: '123' });
    }
  });
```

### Storage Patterns

```typescript
// Upload with metadata
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`user-${userId}/avatar.jpg`, file, {
    cacheControl: '3600',
    upsert: true,
    metadata: { userId, type: 'avatar' }
  });

// Signed URLs for private buckets
const { data } = await supabase.storage
  .from('documents')
  .createSignedUrl('report.pdf', 60); // 60 seconds

// Generate signed URLs for multiple
const { data } = await supabase.storage
  .from('documents')
  .createSignedUrls(['a.pdf', 'b.pdf'], 3600);

// Transform images on the fly
const { data } = supabase.storage
  .from('images')
  .getPublicUrl('photo.jpg', {
    width: 300,
    height: 300,
    quality: 80
  });
```

### Database Triggers

```sql
-- Auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

-- Audit log trigger
CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  table_name TEXT NOT NULL,
  record_id UUID NOT NULL,
  action TEXT NOT NULL,
  old_data JSONB,
  new_data JSONB,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_log (table_name, record_id, action, old_data, new_data, user_id)
  VALUES (
    TG_TABLE_NAME,
    NEW.id,
    TG_OP,
    CASE WHEN TG_OP = 'DELETE' THEN to_jsonb(OLD) ELSE NULL END,
    CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) ELSE NULL END,
    auth.uid()
  );
  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Best Practices

1. **RLS everywhere** - Never assume auth
2. **Service role** - Only in Edge Functions
3. **Realtime filters** - Narrow scope for performance
4. **Indexes** - Add for filtered/sorted columns
5. **Prepared statements** - Use for repeated queries
6. **Batch operations** - Use RPC for complex logic
7. **Storage metadata** - Store for easy querying
