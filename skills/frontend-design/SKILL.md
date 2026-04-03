---
name: frontend-design
description: Frontend design patterns - component architecture, design systems, responsive layouts, accessibility, and UX best practices.
metadata:
  priority: 9
  docs:
    - "https://components.ai/"
    - "https://design-systems.io/"
  pathPatterns:
    - "**/components/**"
    - "**/design-system/**"
  bashPatterns:
    - '\bfrontend\b'
    - '\bdesign.system\b'
  promptSignals:
    phrases:
      - "frontend design"
      - "design system"
      - "component architecture"
    anyOf:
      - "design"
      - "component"
      - "ui"
---

## Frontend Design

### Component Architecture

```typescript
// Compound component pattern
interface CardProps {
  children: React.ReactNode;
}

interface CardHeaderProps {
  children: React.ReactNode;
}

interface CardBodyProps {
  children: React.ReactNode;
}

function Card({ children }: CardProps) {
  return <div className="card">{children}</div>;
}

Card.Header = function CardHeader({ children }: CardHeaderProps) {
  return <div className="card-header">{children}</div>;
};

Card.Body = function CardBody({ children }: CardBodyProps) {
  return <div className="card-body">{children}</div>;
};

// Usage
<Card>
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
</Card>
```

### Design Tokens

```typescript
// tokens.ts
export const tokens = {
  colors: {
    primary: '#3B82F6',
    secondary: '#8B5CF6',
    success: '#10B981',
    warning: '#F59E0B',
    error: '#EF4444',
    background: '#FFFFFF',
    foreground: '#1F2937',
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
  },
  fontSize: {
    xs: '0.75rem',
    sm: '0.875rem',
    base: '1rem',
    lg: '1.125rem',
    xl: '1.25rem',
  },
  shadows: {
    sm: '0 1px 2px rgba(0,0,0,0.05)',
    md: '0 4px 6px rgba(0,0,0,0.1)',
    lg: '0 10px 15px rgba(0,0,0,0.1)',
  },
};
```

### Responsive Design

```css
/* Mobile-first approach */
.container {
  padding: 1rem;
}

@media (min-width: 640px) {
  .container {
    padding: 2rem;
    max-width: 640px;
  }
}

@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    max-width: 1024px;
  }
}

/* Grid system */
.grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;
}

@media (min-width: 640px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### Accessibility

```typescript
// Accessible button
function Button({
  children,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  ...props
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      aria-busy={loading}
      {...props}
    >
      {loading && (
        <span className="spinner" aria-hidden="true" />
      )}
      <span>{children}</span>
    </button>
  );
}

// Keyboard navigation
function Modal({ isOpen, onClose, children }: ModalProps) {
  useEffect(() => {
    if (!isOpen) return;

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
      if (e.key === 'Tab') trapFocus(e);
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);

  return isOpen ? (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      {children}
    </div>
  ) : null;
}
```

### Form Patterns

```typescript
// Controlled input with validation
function FormInput({
  label,
  name,
  type = 'text',
  error,
  required = false,
}: InputProps) {
  const id = `input-${name}`;

  return (
    <div className="form-field">
      <label htmlFor={id}>
        {label}
        {required && <span aria-hidden="true">*</span>}
      </label>
      <input
        id={id}
        name={name}
        type={type}
        aria-invalid={!!error}
        aria-describedby={error ? `${id}-error` : undefined}
      />
      {error && (
        <span id={`${id}-error`} className="error" role="alert">
          {error}
        </span>
      )}
    </div>
  );
}

// Form with validation
function ContactForm() {
  const [errors, setErrors] = useState({});

  function validate(data: FormData) {
    const newErrors: Record<string, string> = {};
    if (!data.get('email')) newErrors.email = 'Email is required';
    if (!data.get('message')) newErrors.message = 'Message is required';
    return newErrors;
  }

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    const errors = validate(Object.fromEntries(data));

    if (Object.keys(errors).length > 0) {
      setErrors(errors);
      return;
    }

    // Submit form
  }

  return (
    <form onSubmit={handleSubmit}>
      <FormInput name="email" label="Email" error={errors.email} />
      <FormInput name="message" label="Message" error={errors.message} />
      <button type="submit">Send</button>
    </form>
  );
}
```

### Animation Patterns

```typescript
// Framer Motion animations
import { motion, AnimatePresence } from 'framer-motion';

// Page transitions
const pageVariants = {
  initial: { opacity: 0, x: -20 },
  animate: { opacity: 1, x: 0 },
  exit: { opacity: 0, x: 20 },
};

// Stagger children
const containerVariants = {
  animate: {
    transition: {
      staggerChildren: 0.1,
    },
  },
};

const itemVariants = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
};

function List({ items }) {
  return (
    <motion.div
      variants={containerVariants}
      initial="initial"
      animate="animate"
    >
      {items.map((item) => (
        <motion.div key={item.id} variants={itemVariants}>
          {item.name}
        </motion.div>
      ))}
    </motion.div>
  );
}
```

### Best Practices

1. **Mobile-first** - Start with mobile, scale up
2. **Design tokens** - Single source of truth
3. **Accessible** - ARIA, keyboard nav, focus
4. **Semantic HTML** - Proper elements
5. **Performance** - Lazy load, optimize images
6. **Responsive** - Fluid layouts, breakpoints
7. **Test across browsers** - Chrome, Firefox, Safari
