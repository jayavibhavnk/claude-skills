---
name: css-architecture
description: Design scalable CSS architecture - naming conventions, component styles, and organization.
metadata:
  priority: 7
  docs:
    - "https://cube.fyi/"
    - "https://designtokens.dev/"
  pathPatterns:
    - "**/*.css"
    - "**/*.scss"
    - "**/*.module.css"
  bashPatterns:
    - '\bcss\b'
    - '\bscss\b'
  promptSignals:
    phrases:
      - "css"
      - "styles"
      - "styling"
    anyOf:
      - "css"
      - "stylesheet"
---

## CSS Architecture

### Naming Conventions (BEM)

```css
/* Block */
.card { }

/* Element */
.card__title { }
.card__body { }
.card__image { }

/* Modifier */
.card--featured { }
.card__title--large { }

/* Nested */
.card .card { }  /* Avoid - creates tight coupling */
```

### Component CSS

```css
/* Button component */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  font-weight: 600;
  border-radius: 0.5rem;
  transition: all 0.2s ease;
}

.button--primary {
  background: var(--color-primary);
  color: white;
}

.button--secondary {
  background: transparent;
  border: 2px solid var(--color-primary);
  color: var(--color-primary);
}

.button--small {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

### CSS Custom Properties

```css
:root {
  /* Colors */
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --color-success: #22c55e;
  --color-danger: #ef4444;

  /* Typography */
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
}
```

### CSS Modules

```css
/* Card.module.css */
.card {
  padding: var(--space-4);
  background: white;
  border-radius: 0.5rem;
  box-shadow: var(--shadow-md);
}

.title {
  font-size: var(--font-size-xl);
  font-weight: 700;
}

/* Component.jsx */
import styles from './Card.module.css';

export function Card({ title, children }) {
  return (
    <div className={styles.card}>
      <h2 className={styles.title}>{title}</h2>
      <div>{children}</div>
    </div>
  );
}
```

### Utility Classes

```css
/* Layout utilities */
.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.justify-between { justify-content: space-between; }
.gap-2 { gap: var(--space-2); }
.gap-4 { gap: var(--space-4); }

/* Spacing utilities */
.mt-2 { margin-top: var(--space-2); }
.mt-4 { margin-top: var(--space-4); }
.p-4 { padding: var(--space-4); }

/* Text utilities */
.text-sm { font-size: var(--font-size-sm); }
.text-center { text-align: center; }
.font-bold { font-weight: 700; }

/* Responsive */
@media (max-width: 768px) {
  .hide-mobile { display: none; }
}
```

### Animation

```css
/* Transitions */
.button {
  transition: all 0.2s ease-in-out;
}

/* Keyframes */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fadeIn 0.3s ease-out;
}

.animate-slide-up {
  animation: slideUp 0.3s ease-out;
}
```

### Best Practices

1. **Use CSS custom properties** - For theming and consistency
2. **Mobile-first** - Write base styles for mobile, then @media queries
3. **Avoid specificity wars** - Keep specificity low
4. **Use BEM** - For component naming
5. **Co-locate styles** - With components when possible
6. **Extract utilities** - For repeated patterns
7. **Minimize nesting** - Max 3 levels deep
