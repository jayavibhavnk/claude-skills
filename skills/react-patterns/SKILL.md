---
name: react-patterns
description: React component patterns - hooks, composition, render props, and performance patterns.
metadata:
  priority: 9
  docs:
    - "https://react.dev/learn"
  pathPatterns:
    - "**/*.tsx"
    - "**/*.jsx"
  bashPatterns:
    - '\breact\b'
  promptSignals:
    phrases:
      - "react"
      - "component"
      - "hook"
    anyOf:
      - "react"
      - "component"
      - "hook"
---

## React Patterns

### Custom Hooks

```typescript
// useLocalStorage
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const item = window.localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  });

  useEffect(() => {
    window.localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}

// useDebounce
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// useClickOutside
function useClickOutside(ref: RefObject<HTMLElement>, handler: () => void) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) return;
      handler();
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);

    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}
```

### Compound Components

```typescript
// Compound component pattern
function Tabs({ children }: { children: React.ReactNode }) {
  const [activeTab, setActiveTab] = useState('');

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div role="tablist">{children}</div>;
}

function Tab({ value, children }: { value: string; children: React.ReactNode }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  const isActive = activeTab === value;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanel({ value, children }: { value: string; children: React.ReactNode }) {
  const { activeTab } = useContext(TabsContext);
  if (activeTab !== value) return null;
  return <div role="tabpanel">{children}</div>;
}

// Usage
<Tabs>
  <TabList>
    <Tab value="tab1">First</Tab>
    <Tab value="tab2">Second</Tab>
  </TabList>
  <TabPanel value="tab1">Content 1</TabPanel>
  <TabPanel value="tab2">Content 2</TabPanel>
</Tabs>
```

### Render Props

```typescript
// Mouse tracker with render prop
function MouseTracker({ render }: { render: (pos: { x: number; y: number }) => React.ReactNode }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

// Usage
<MouseTracker render={({ x, y }) => (
  <div>Mouse at {x}, {y}</div>
)} />
```

### Controlled/Uncontrolled

```typescript
// Controlled component
function ControlledInput({
  value,
  onChange,
}: {
  value: string;
  onChange: (value: string) => void;
}) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}

// Uncontrolled with ref
function UncontrolledInput({ defaultValue = '' }: { defaultValue?: string }) {
  const [value, setValue] = useState(defaultValue);
  const ref = useRef<HTMLInputElement>(null);

  return (
    <input
      ref={ref}
      defaultValue={defaultValue}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// Polymorphic component
type TextProps<T extends ElementType = 'span'> = {
  as?: T;
  children: React.ReactNode;
} & ComponentPropsWithoutRef<T>;

function Text<T extends ElementType = 'span'>({ as, children, ...props }: TextProps<T>) {
  const Component = as || 'span';
  return <Component {...props}>{children}</Component>;
}

// Usage
<Text as="h1" size="large">Hello</Text>
<Text as="p">Paragraph</Text>
```

### Error Boundaries

```typescript
class ErrorBoundary extends Component<{ children: React.ReactNode }, { hasError: boolean }> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error('Error:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return <div>Something went wrong</div>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

### Suspense + Data Fetching

```typescript
function SuspenseDataComponent() {
  return (
    <Suspense fallback={<Loading />}>
      <ErrorBoundary>
        <AsyncData />
      </ErrorBoundary>
    </Suspense>
  );
}
```

### Best Practices

1. **Colocate state** - Keep state as local as possible
2. **Custom hooks** - Extract reusable logic
3. **Composition over inheritance** - Use compound components
4. **Memoization** - useMemo, useCallback sparingly
5. **Error boundaries** - Handle errors gracefully
6. **Type safety** - Type all props and state
