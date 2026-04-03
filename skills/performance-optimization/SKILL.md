---
name: performance-optimization
description: Optimize application performance - profiling, bundle size, render performance, and caching strategies.
metadata:
  priority: 8
  docs:
    - "https://web.dev/articles/fast"
  pathPatterns:
    - "**/performance/**"
    - "**/optimization/**"
  bashPatterns:
    - '\bperformance\b'
    - '\boptimize\b'
    - '\bbundle\b'
  promptSignals:
    phrases:
      - "performance"
      - "optimize"
      - "speed"
    anyOf:
      - "performance"
      - "optimization"
      - "profiling"
---

## Performance Optimization

### Core Web Vitals

| Metric | Target | What it measures |
|--------|--------|-----------------|
| LCP | < 2.5s | Largest Contentful Paint |
| FID | < 100ms | First Input Delay |
| CLS | < 0.1 | Cumulative Layout Shift |
| TTFB | < 200ms | Time to First Byte |

### React Performance

```typescript
// 1. Memoization
const ExpensiveComponent = memo(({ data }) => {
  return <div>{/* expensive rendering */}</div>;
});

// 2. useMemo for expensive calculations
function DataTable({ items }) {
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);

  return <Table data={sortedItems} />;
}

// 3. useCallback for callbacks
const handleClick = useCallback((id) => {
  setItems(items => items.filter(i => i.id !== id));
}, []);

// 4. Virtualization for long lists
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  return (
    <FixedSizeList
      height={400}
      width={300}
      itemCount={items.length}
      itemSize={50}
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}
```

### Bundle Optimization

```typescript
// Dynamic imports for code splitting
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// Named imports (smaller bundles)
// Instead of: import _ from 'lodash'
import { groupBy, sortBy, debounce } from 'lodash';

// Tree-shakeable imports
import { debounce } from 'lodash-es';  // Only debounce included
```

### Image Optimization

```typescript
// Next.js Image component
import Image from 'next/image';

function ProductImage({ src, alt }) {
  return (
    <Image
      src={src}
      alt={alt}
      width={400}
      height={300}
      placeholder="blur"
      blurDataURL={generateBlurDataURL(src)}
      priority={false}  // Lazy load by default
    />
  );
}

// Responsive images
<img
  srcSet="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w
  "
  sizes="(max-width: 400px) 100vw, (max-width: 800px) 50vw, 33vw"
/>
```

### Caching Strategies

```typescript
// 1. HTTP caching headers
app.use((req, res, next) => {
  // Static assets - cache long
  if (req.url.includes('/static/')) {
    res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
  }

  // API responses - no cache or short
  if (req.url.includes('/api/')) {
    res.setHeader('Cache-Control', 'no-cache');
  }

  // CDN - stale-while-revalidate
  res.setHeader('Cache-Control', 'public, max-age=0, stale-while-revalidate=86400');
});

// 2. SWR pattern (stale-while-revalidate)
function useData(url) {
  const { data, error } = useSWR(url, fetcher, {
    revalidateOnFocus: false,
    dedupingInterval: 60000,  // Dedupe requests
  });

  return { data, isLoading: !error && !data };
}
```

### Database Query Optimization

```typescript
// 1. Select only needed columns
const users = await db
  .select({
    id: usersTable.id,
    name: usersTable.name,
    email: usersTable.email,
  })
  .from(usersTable)
  .where(eq(usersTable.status, 'active'));

// 2. Use indexes (create index on WHERE columns)
await db.execute(sql`
  CREATE INDEX idx_orders_user_status
  ON orders(user_id, status)
  WHERE status = 'pending';
`);

// 3. Batch operations
const results = await Promise.all(
  userIds.map(id => db.users.findUnique({ where: { id } }))
);

// 4. Connection pooling
const pool = new Pool({
  max: 20,  // Max connections
  idleTimeout: 30000,
  connectionTimeoutMillis: 2000,
});
```

### JavaScript Performance

```typescript
// 1. Debounce/throttle
const debouncedSearch = debounce((query) => {
  performSearch(query);
}, 300);

const throttledScroll = throttle(() => {
  updateScrollPosition();
}, 100);

// 2. Web Workers for heavy computation
const worker = new Worker('heavy-calculation.js');

worker.postMessage({ data: largeArray });
worker.onmessage = ({ data }) => {
  console.log('Result:', data);
};

// 3. RequestAnimationFrame
function animate() {
  // Update element position
  element.style.transform = `translateX(${position}px)`;
  requestAnimationFrame(animate);
}

// 4. Batch DOM updates
const fragment = document.createDocumentFragment();

items.forEach(item => {
  const div = document.createElement('div');
  div.textContent = item.name;
  fragment.appendChild(div);
});

container.appendChild(fragment);  // Single reflow
```

### Profiling Tools

| Tool | Use Case |
|------|----------|
| Chrome DevTools | CPU, memory, network |
| Lighthouse | Core Web Vitals, audits |
| WebPageTest | Multi-location testing |
| Bundle Analyzer | Visualize bundle size |
| React DevTools | Component profiling |

### Performance Checklist

- [ ] Lazy load below-fold content
- [ ] Optimize images (WebP, responsive)
- [ ] Minimize bundle size
- [ ] Add caching headers
- [ ] Database indexes on WHERE columns
- [ ] Memoize expensive computations
- [ ] Virtualize long lists
- [ ] Use CDN for static assets
- [ ] Monitor Core Web Vitals
- [ ] Profile regularly
