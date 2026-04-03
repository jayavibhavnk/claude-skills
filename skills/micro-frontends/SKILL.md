---
name: micro-frontends
description: Micro-frontend architecture - Module Federation, independent deployments, and shared state.
metadata:
  priority: 8
  docs:
    - "https://webpack.js.org/concepts/module-federation/"
  pathPatterns:
    - "**/federation/**"
    - "**/micro-frontend/**"
  bashPatterns:
    - '\bmicro-frontend\b'
    - '\bmodule.federation\b'
  promptSignals:
    phrases:
      - "micro-frontend"
      - "module federation"
    anyOf:
      - "micro-frontend"
      - "federation"
---

## Micro-Frontends

### Module Federation (Webpack)

```javascript
// Host app - webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        dashboard: 'dashboard@http://localhost:3001/remoteEntry.js',
        auth: 'auth@http://localhost:3002/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

### Remote App Configuration

```javascript
// Remote app - webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'dashboard',
      filename: 'remoteEntry.js',
      exposes: {
        './Dashboard': './src/Dashboard',
        './DashboardWidget': './src/DashboardWidget',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
};
```

### Loading Remote Modules

```typescript
// Dynamic remote loading
const RemoteDashboard = React.lazy(() => import('dashboard/Dashboard'));

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<Loading />}>
        <RemoteDashboard />
      </Suspense>
    </ErrorBoundary>
  );
}

// Type-safe remote imports
declare module 'dashboard/Dashboard' {
  export const Dashboard: React.ComponentType;
}
```

### Shared State Patterns

```typescript
// 1. URL-based state (simplest)
<Link to="/dashboard?tab=overview&widget=chart">
  Go to Dashboard
</Link>

// 2. Custom event bus
class EventBus {
  private listeners: Map<string, Set<Function>> = new Map();

  emit(event: string, data: any) {
    this.listeners.get(event)?.forEach((fn) => fn(data));
  }

  subscribe(event: string, fn: Function) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(fn);
    return () => this.listeners.get(event)?.delete(fn);
  }
}

export const eventBus = new EventBus();

// Usage in remote
eventBus.emit('user:login', { userId: '123' });

// Usage in host
useEffect(() => {
  return eventBus.subscribe('user:login', (data) => {
    console.log('User logged in:', data);
  });
}, []);
```

### Routing Integration

```typescript
// Host app with React Router
function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route
          path="/dashboard/*"
          element={
            <ErrorBoundary>
              <Suspense fallback={<Loading />}>
                <RemoteRoutes basePath="/dashboard" />
              </Suspense>
            </ErrorBoundary>
          }
        />
      </Routes>
    </Router>
  );
}

// Remote app exports its routes
// dashboard/src/bootstrap.tsx
export function RemoteRoutes({ basePath }: { basePath: string }) {
  return (
    <Routes base={basePath}>
      <Route path="/" element={<DashboardHome />} />
      <Route path="/settings" element={<DashboardSettings />} />
    </Routes>
  );
}
```

### CSS Isolation

```typescript
// 1. CSS Modules (recommended)
import styles from './Dashboard.module.css';

// 2. Shadow DOM
const shadowRoot = element.attachShadow({ mode: 'open' });

// 3. Prefix convention
.dashboard-button { }
.dashboard-container { }
```

### Error Handling

```typescript
// Remote component wrapper
function RemoteWrapper({ remote, module, fallback }) {
  const [Component, setComponent] = useState(null);
  const [error, setError] = useState(false);

  useEffect(() => {
    import(/* @vite-ignore */ `${remote}/${module}`)
      .then((mod) => setComponent(mod.default || mod))
      .catch(() => setError(true));
  }, [remote, module]);

  if (error) return fallback || <DefaultError />;
  if (!Component) return <Loading />;
  return <Component />;
}
```

### Deployment Strategies

```markdown
## Independent Deployments

Each micro-frontend:
- Own repository
- Own CI/CD pipeline
- Own CDN/hosting
- Own versioning

## Shared Dependencies
- Bundle once in each app
- Use Webpack Module Federation
- Avoid version conflicts with singletons

## Environment Matrix
| App | React | UI Library | Other |
|-----|-------|-----------|-------|
| Host | 18.2 | Tailwind 3.5 | - |
| Dashboard | 18.2 | Tailwind 3.5 | Recharts |
| Auth | 18.2 | MUI 5 | - |
```

### Best Practices

1. **Keep apps small** - Single responsibility
2. **Shared dependencies** - React, state management
3. **Clear contracts** - Props, events, shared modules
4. **Independent deploys** - Each team owns their pipeline
5. **CSS isolation** - Modules or Shadow DOM
6. **Error boundaries** - Graceful degradation
7. **Design system** - Shared UI components
