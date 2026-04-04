---
name: game-ui-toolkit
description: React/Next.js game UI toolkit - reusable UI components, button styles, progress bars, tooltips, and polished game interface elements.
metadata:
  priority: 9
  docs:
    - "https://ui.aceternity.com/"
    - "https://www.radix-ui.com/"
  pathPatterns:
    - "**/game-ui/**"
    - "**/components/**/ui/**"
  bashPatterns:
    - '\bgame.?ui\b'
    - '\bui.?toolkit\b'
  promptSignals:
    phrases:
      - "game ui toolkit"
      - "game components"
      - "button styles"
    anyOf:
      - "game ui"
      - "game components"
---

## Game UI Toolkit

### Button Components

```typescript
// Game-style button with hover effects
function GameButton({
  children,
  variant = 'primary',
  size = 'md',
  disabled,
  loading,
  onClick,
}: ButtonProps) {
  const variants = {
    primary: 'bg-gradient-to-br from-amber-500 to-orange-600 hover:from-amber-400 hover:to-orange-500',
    secondary: 'bg-gradient-to-br from-slate-600 to-slate-700 hover:from-slate-500 hover:to-slate-600',
    danger: 'bg-gradient-to-br from-red-600 to-red-700 hover:from-red-500 hover:to-red-600',
    ghost: 'bg-transparent hover:bg-white/10',
  };

  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-5 py-2.5 text-base',
    lg: 'px-8 py-3.5 text-lg',
  };

  return (
    <motion.button
      className={`
        relative font-bold rounded-lg overflow-hidden
        ${variants[variant]} ${sizes[size]}
        disabled:opacity-50 disabled:cursor-not-allowed
      `}
      whileHover={{ scale: 1.02, y: -2 }}
      whileTap={{ scale: 0.98 }}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {/* Shimmer effect */}
      <motion.div
        className="absolute inset-0 bg-gradient-to-r from-transparent via-white/20 to-transparent"
        initial={{ x: '-100%' }}
        whileHover={{ x: '100%' }}
        transition={{ duration: 0.6 }}
      />

      {loading ? (
        <Spinner className="mx-auto" />
      ) : (
        <span className="relative z-10">{children}</span>
      )}
    </motion.button>
  );
}

// Icon button
function IconButton({
  icon,
  label,
  variant = 'ghost',
  size = 'md',
  badge,
}: IconButtonProps) {
  return (
    <Tooltip content={label}>
      <motion.button
        className={`
          relative rounded-full flex items-center justify-center
          ${variant === 'ghost' && 'hover:bg-white/10'}
          ${variant === 'solid' && 'bg-white/10 hover:bg-white/20'}
        `}
        whileHover={{ scale: 1.1 }}
        whileTap={{ scale: 0.9 }}
      >
        {icon}
        {badge && (
          <span className="absolute -top-1 -right-1 w-5 h-5 bg-red-500 rounded-full text-xs flex items-center justify-center">
            {badge}
          </span>
        )}
      </motion.button>
    </Tooltip>
  );
}
```

### Progress Bars

```typescript
// XP Bar with segments
function XPBar({ current, max, level, segments = 10 }: XPProps) {
  const percentage = (current / max) * 100;

  return (
    <div className="w-full">
      <div className="flex justify-between text-sm mb-1">
        <span className="text-amber-400">Level {level}</span>
        <span className="text-slate-400">{current} / {max} XP</span>
      </div>

      <div className="h-4 bg-slate-800 rounded-full overflow-hidden relative">
        {/* Segment lines */}
        <div className="absolute inset-0 flex">
          {Array.from({ length: segments }).map((_, i) => (
            <div key={i} className="flex-1 border-r border-slate-900 last:border-r-0" />
          ))}
        </div>

        {/* Fill */}
        <motion.div
          className="h-full bg-gradient-to-r from-purple-600 to-purple-400"
          initial={{ width: 0 }}
          animate={{ width: `${percentage}%` }}
          transition={{ duration: 0.5, ease: 'easeOut' }}
        />

        {/* Glow */}
        <div
          className="absolute top-0 h-full w-8 bg-purple-400/50 blur-sm"
          style={{ left: `${percentage}%`, marginLeft: '-16px' }}
        />
      </div>
    </div>
  );
}

// Health bar with pulse animation when low
function HealthBar({ current, max, showNumbers = true }: Props) {
  const percentage = (current / max) * 100;
  const isLow = percentage <= 25;

  return (
    <div className="relative">
      <div className="h-6 bg-slate-800 rounded-lg overflow-hidden border border-slate-600">
        <motion.div
          className={`
            h-full transition-colors duration-300
            ${percentage > 50 ? 'bg-gradient-to-r from-green-600 to-green-500' :
              percentage > 25 ? 'bg-gradient-to-r from-yellow-600 to-yellow-500' :
              'bg-gradient-to-r from-red-600 to-red-500'}
          `}
          animate={{ width: `${percentage}%` }}
          transition={{ duration: 0.3 }}
        />

        {/* Low health pulse */}
        {isLow && (
          <motion.div
            className="absolute inset-0 bg-red-500/30"
            animate={{ opacity: [0.3, 0.7, 0.3] }}
            transition={{ duration: 0.8, repeat: Infinity }}
          />
        )}
      </div>

      {showNumbers && (
        <div className="absolute inset-0 flex items-center justify-center">
          <span className="text-xs font-bold text-white drop-shadow-lg">
            {current} / {max}
          </span>
        </div>
      )}
    </div>
  );
}
```

### Tooltips

```typescript
// Game-style tooltip with rich content
function GameTooltip({
  children,
  content,
  position = 'top',
}: TooltipProps) {
  const variants = {
    top: 'bottom-full left-1/2 -translate-x-1/2 mb-2',
    bottom: 'top-full left-1/2 -translate-x-1/2 mt-2',
    left: 'right-full top-1/2 -translate-y-1/2 mr-2',
    right: 'left-full top-1/2 -translate-y-1/2 ml-2',
  };

  return (
    <div className="relative group">
      {children}

      <motion.div
        className={`
          absolute ${variants[position]} z-50 pointer-events-none
          px-3 py-2 rounded-lg
          bg-slate-900/95 border border-slate-700
          backdrop-blur-sm
        `}
        initial={{ opacity: 0, scale: 0.9 }}
        whileHover={{ opacity: 1, scale: 1 }}
        transition={{ duration: 0.15 }}
      >
        {/* Arrow */}
        <div
          className={`
            absolute w-2 h-2 bg-slate-900/95 border-slate-700 border
            ${position === 'top' && 'top-full left-1/2 -translate-x-1/2 border-t-0 border-l-0 rotate-45'}
            ${position === 'bottom' && 'bottom-full left-1/2 -translate-x-1/2 border-b-0 border-r-0 rotate-45'}
          `}
        />

        <div className="relative z-10 text-sm text-slate-200 whitespace-nowrap">
          {content}
        </div>
      </motion.div>
    </div>
  );
}

// Rich tooltip with stats
function StatTooltip({ item }: { item: Item }) {
  return (
    <div className="w-64 p-3">
      <div className="text-amber-400 font-bold">{item.name}</div>
      <div className="text-xs text-slate-400 mb-2">{item.type}</div>

      <div className="space-y-1 text-sm">
        {item.stats.map(stat => (
          <div key={stat.name} className="flex justify-between">
            <span className="text-slate-300">{stat.name}</span>
            <span className={stat.value > 0 ? 'text-green-400' : 'text-red-400'}>
              {stat.value > 0 ? '+' : ''}{stat.value}
            </span>
          </div>
        ))}
      </div>

      {item.description && (
        <p className="mt-2 text-xs text-slate-500 italic">{item.description}</p>
      )}
    </div>
  );
}
```

### Modal/Dialog

```typescript
// Game-style modal with backdrop blur
function GameModal({
  isOpen,
  onClose,
  title,
  children,
  size = 'md',
}: ModalProps) {
  const sizes = {
    sm: 'max-w-sm',
    md: 'max-w-md',
    lg: 'max-w-lg',
    xl: 'max-w-xl',
    full: 'max-w-4xl',
  };

  return (
    <AnimatePresence>
      {isOpen && (
        <>
          {/* Backdrop */}
          <motion.div
            className="fixed inset-0 bg-black/70 backdrop-blur-sm z-40"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            onClick={onClose}
          />

          {/* Modal */}
          <motion.div
            className={`
              fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-50
              w-full ${sizes[size]} p-6
              bg-gradient-to-br from-slate-900 to-slate-950
              border border-slate-700 rounded-2xl
              shadow-2xl shadow-black/50
            `}
            initial={{ opacity: 0, scale: 0.9, y: 20 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.9, y: 20 }}
            transition={{ type: 'spring', damping: 25, stiffness: 300 }}
          >
            {/* Header */}
            <div className="flex items-center justify-between mb-4">
              <h2 className="text-xl font-bold text-amber-400">{title}</h2>
              <button
                onClick={onClose}
                className="p-1 hover:bg-white/10 rounded-lg transition-colors"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            {/* Content */}
            <div className="text-slate-300">{children}</div>
          </motion.div>
        </>
      )}
    </AnimatePresence>
  );
}
```

### Toast/Notification

```typescript
// Game notification toast
type ToastType = 'info' | 'success' | 'warning' | 'error';

interface Toast {
  id: string;
  type: ToastType;
  title: string;
  message?: string;
  duration?: number;
}

function ToastContainer() {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const addToast = (toast: Omit<Toast, 'id'>) => {
    const id = crypto.randomUUID();
    setToasts(prev => [...prev, { ...toast, id }]);

    setTimeout(() => {
      setToasts(prev => prev.filter(t => t.id !== id));
    }, toast.duration || 3000);
  };

  const icons = {
    info: <Info className="w-5 h-5 text-blue-400" />,
    success: <CheckCircle className="w-5 h-5 text-green-400" />,
    warning: <AlertTriangle className="w-5 h-5 text-yellow-400" />,
    error: <XCircle className="w-5 h-5 text-red-400" />,
  };

  return (
    <div className="fixed top-4 right-4 z-50 space-y-2">
      <AnimatePresence>
        {toasts.map(toast => (
          <motion.div
            key={toast.id}
            initial={{ opacity: 0, x: 100, scale: 0.9 }}
            animate={{ opacity: 1, x: 0, scale: 1 }}
            exit={{ opacity: 0, x: 100, scale: 0.9 }}
            className={`
              flex items-start gap-3 p-4 rounded-lg
              bg-slate-900/95 border backdrop-blur-sm
              ${toast.type === 'success' && 'border-green-500/50'}
              ${toast.type === 'error' && 'border-red-500/50'}
              ${toast.type === 'warning' && 'border-yellow-500/50'}
              ${toast.type === 'info' && 'border-blue-500/50'}
            `}
          >
            {icons[toast.type]}
            <div>
              <div className="font-bold text-white">{toast.title}</div>
              {toast.message && (
                <div className="text-sm text-slate-400">{toast.message}</div>
              )}
            </div>
          </motion.div>
        ))}
      </AnimatePresence>
    </div>
  );
}
```

### Tabs

```typescript
// Game-style tab navigation
function GameTabs({ tabs, defaultTab, onChange }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  const handleTabChange = (tab: string) => {
    setActiveTab(tab);
    onChange?.(tab);
  };

  return (
    <div className="relative">
      {/* Tab list */}
      <div className="flex gap-1 p-1 bg-slate-800/50 rounded-lg">
        {tabs.map(tab => (
          <button
            key={tab.id}
            onClick={() => handleTabChange(tab.id)}
            className={`
              relative px-4 py-2 rounded-md font-medium transition-colors
              ${activeTab === tab.id ? 'text-amber-400' : 'text-slate-400 hover:text-slate-200'}
            `}
          >
            {activeTab === tab.id && (
              <motion.div
                className="absolute inset-0 bg-slate-700/50 rounded-md"
                layoutId="activeTab"
                transition={{ type: 'spring', bounce: 0.2 }}
              />
            )}
            <span className="relative z-10 flex items-center gap-2">
              {tab.icon}
              {tab.label}
            </span>
          </button>
        ))}
      </div>

      {/* Tab content */}
      <AnimatePresence mode="wait">
        <motion.div
          key={activeTab}
          initial={{ opacity: 0, y: 10 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -10 }}
          transition={{ duration: 0.15 }}
          className="mt-4"
        >
          {tabs.find(t => t.id === activeTab)?.content}
        </motion.div>
      </AnimatePresence>
    </div>
  );
}
```

### Dropdown Select

```typescript
// Game-style dropdown
function GameSelect({
  options,
  value,
  onChange,
  placeholder = 'Select...',
}: SelectProps) {
  const [isOpen, setIsOpen] = useState(false);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const handleClickOutside = (e: MouseEvent) => {
      if (ref.current && !ref.current.contains(e.target as Node)) {
        setIsOpen(false);
      }
    };
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);

  const selected = options.find(o => o.value === value);

  return (
    <div ref={ref} className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className={`
          w-full flex items-center justify-between px-4 py-2.5 rounded-lg
          bg-slate-800 border border-slate-700
          hover:border-slate-600 transition-colors
          ${isOpen && 'border-amber-500'}
        `}
      >
        <span className={selected ? 'text-white' : 'text-slate-500'}>
          {selected?.label || placeholder}
        </span>
        <ChevronDown
          className={`w-4 h-4 text-slate-400 transition-transform ${isOpen && 'rotate-180'}`}
        />
      </button>

      <AnimatePresence>
        {isOpen && (
          <motion.div
            className="absolute top-full left-0 right-0 mt-2 py-1 rounded-lg bg-slate-800 border border-slate-700 shadow-xl z-50"
            initial={{ opacity: 0, y: -10 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -10 }}
          >
            {options.map(option => (
              <button
                key={option.value}
                onClick={() => {
                  onChange(option.value);
                  setIsOpen(false);
                }}
                className={`
                  w-full flex items-center gap-2 px-4 py-2 text-left
                  hover:bg-white/5 transition-colors
                  ${option.value === value && 'text-amber-400'}
                `}
              >
                {option.icon}
                {option.label}
              </button>
            ))}
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}
```

### Slider

```typescript
// Game-style slider with snap points
function GameSlider({
  value,
  onChange,
  min = 0,
  max = 100,
  step = 1,
  marks,
}: SliderProps) {
  const percentage = ((value - min) / (max - min)) * 100;

  return (
    <div className="w-full py-4">
      <input
        type="range"
        min={min}
        max={max}
        step={step}
        value={value}
        onChange={e => onChange(Number(e.target.value))}
        className="
          w-full h-2 appearance-none bg-slate-700 rounded-full
          cursor-pointer
          [&::-webkit-slider-thumb]:appearance-none
          [&::-webkit-slider-thumb]:w-5
          [&::-webkit-slider-thumb]:h-5
          [&::-webkit-slider-thumb]:rounded-full
          [&::-webkit-slider-thumb]:bg-gradient-to-br
          [&::-webkit-slider-thumb]:from-amber-400
          [&::-webkit-slider-thumb]:to-amber-600
          [&::-webkit-slider-thumb]:shadow-lg
          [&::-webkit-slider-thumb]:shadow-amber-500/30
          [&::-webkit-slider-thumb]:cursor-grab
          [&::-webkit-slider-thumb]:active:cursor-grabbing
          [&::-webkit-slider-thumb]:transition-transform
          [&::-webkit-slider-thumb]:hover:scale-110
        "
      />

      {/* Track fill */}
      <div
        className="h-1 bg-gradient-to-r from-amber-600 to-amber-400 rounded-full mt-1"
        style={{ width: `${percentage}%` }}
      />

      {/* Marks */}
      {marks && (
        <div className="flex justify-between mt-2 px-1">
          {marks.map(mark => (
            <div key={mark.value} className="text-center">
              <div
                className={`text-xs ${value >= mark.value ? 'text-amber-400' : 'text-slate-500'}`}
              >
                {mark.label}
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Badge/Pill

```typescript
// Status badge
function Badge({
  children,
  variant = 'default',
  size = 'md',
  pulse = false,
}: BadgeProps) {
  const variants = {
    default: 'bg-slate-700 text-slate-300',
    primary: 'bg-amber-600/20 text-amber-400 border-amber-500/30',
    success: 'bg-green-600/20 text-green-400 border-green-500/30',
    danger: 'bg-red-600/20 text-red-400 border-red-500/30',
    warning: 'bg-yellow-600/20 text-yellow-400 border-yellow-500/30',
    info: 'bg-blue-600/20 text-blue-400 border-blue-500/30',
  };

  const sizes = {
    sm: 'px-1.5 py-0.5 text-xs',
    md: 'px-2 py-1 text-xs',
    lg: 'px-3 py-1.5 text-sm',
  };

  return (
    <span
      className={`
        inline-flex items-center gap-1 rounded-full font-medium border
        ${variants[variant]} ${sizes[size]}
      `}
    >
      {pulse && (
        <span className="relative flex h-2 w-2">
          <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-current opacity-75" />
          <span className="relative inline-flex rounded-full h-2 w-2 bg-current" />
        </span>
      )}
      {children}
    </span>
  );
}
```

### Best Practices

1. **Consistent theming** - Use design tokens
2. **Micro-interactions** - Add life to UI
3. **Responsive** - Works on all screens
4. **Accessibility** - Keyboard nav, ARIA
5. **Performance** - Memoize expensive renders
6. **Animation** - Use spring physics
7. **Feedback** - Every action has response
