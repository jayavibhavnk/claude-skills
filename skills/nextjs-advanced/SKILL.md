---
name: nextjs-advanced
description: Advanced Next.js patterns - Server Components, Streaming, Parallel Routes, and Interceptors.
metadata:
  priority: 9
  docs:
    - "https://nextjs.org/docs/app/building-your-application/routing"
  pathPatterns:
    - "app/**/*.tsx"
    - "app/**/*.ts"
  bashPatterns:
    - '\bnextjs\b'
    - '\bnext\b'
  promptSignals:
    phrases:
      - "nextjs"
      - "server component"
      - "streaming"
    anyOf:
      - "nextjs"
      - "app router"
---

## Next.js Advanced Patterns

### Server Components

```typescript
// app/users/page.tsx (Server Component)
async function UsersPage() {
  // Direct database query - no API needed
  const users = await db.user.findMany({
    orderBy: { createdAt: 'desc' },
    take: 100,
  });

  return (
    <div>
      <h1>Users</h1>
      <UserList users={users} />
    </div>
  );
}

// Client Component for interactivity
'use client';

// app/users/components/UserList.tsx
function UserList({ users }: { users: User[] }) {
  const [filter, setFilter] = useState('');

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter users..."
      />
      <ul>
        {users
          .filter((u) => u.name.includes(filter))
          .map((user) => (
            <li key={user.id}>{user.name}</li>
          ))}
      </ul>
    </div>
  );
}
```

### Streaming with Suspense

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';

async function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>

      <Suspense fallback={<RecentActivitySkeleton />}>
        <RecentActivity />
      </Suspense>

      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>
    </div>
  );
}

// Slow component
async function RecentActivity() {
  // Simulates slow fetch
  const activity = await fetchActivity({ delay: 2000 });
  return <ActivityList items={activity} />;
}
```

### Parallel Routes

```typescript
// app/@analytics/(overview)/page.tsx
// app/@analytics/(detail)/page.tsx
// Both render simultaneously in @analytics slot

// app/layout.tsx
export default function Layout({
  children,
  analytics,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div>
      <main>{children}</main>
      <aside>{analytics}</aside>
    </div>
  );
}
```

### Route Interception

```typescript
// app/@modal/(.)photo/[id]/page.tsx
// Intercepts /photo/123 and shows modal

import { useRouter, usePathname } from 'next/navigation';

export default function ModalPhoto({
  params,
}: {
  params: { id: string };
}) {
  const router = useRouter();
  const pathname = usePathname();

  if (pathname.startsWith('/photo')) {
    return (
      <div className="modal-backdrop" onClick={() => router.back()}>
        <div className="modal-content" onClick={(e) => e.stopPropagation()}>
          <Photo id={params.id} />
        </div>
      </div>
    );
  }

  return <Photo id={params.id} />;
}
```

### Server Actions

```typescript
// app/actions.ts
'use server';

export async function createUser(formData: FormData) {
  const name = formData.get('name');
  const email = formData.get('email');

  const user = await db.user.create({
    data: { name: name as string, email: email as string },
  });

  revalidatePath('/users');
  return { success: true, user };
}

// app/users/new/page.tsx
async function NewUserPage() {
  return (
    <form action={createUser}>
      <input name="name" type="text" />
      <input name="email" type="email" />
      <button type="submit">Create</button>
    </form>
  );
}
```

### Caching & Revalidation

```typescript
// Static with revalidation
export default async function Page() {
  const data = await fetchData({
    next: { revalidate: 3600 }, // Revalidate every hour
  });
  return <div>{data}</div>;
}

// Dynamic with no cache
export default async function Page() {
  const data = await fetchData({
    cache: 'no-store', // Always fresh
  });
  return <div>{data}</div>;
}

// On-demand revalidation
import { revalidatePath, revalidateTag } from 'next/cache';

revalidatePath('/users');
revalidateTag('users');
```

### Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Rewrite based on headers
  if (request.headers.get('x-country') === 'DE') {
    return NextResponse.rewrite(
      new URL('/de' + request.nextUrl.pathname, request.url)
    );
  }

  // Add headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');

  return response;
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```
