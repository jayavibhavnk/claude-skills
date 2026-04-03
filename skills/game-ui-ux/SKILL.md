---
name: game-ui-ux
description: Game UI/UX design - HUD design, menus, accessibility, visual feedback, motion design, and immersive interfaces.
metadata:
  priority: 8
  docs:
    - "https://www.gameuidatabase.com/"
  pathPatterns:
    - "**/ui/**"
    - "**/game/**/*UI*.tsx"
  bashPatterns:
    - '\bgame.ui\b'
    - '\bhud\b'
  promptSignals:
    phrases:
      - "game ui"
      - "game ux"
      - "hud design"
    anyOf:
      - "game ui"
      - "hud"
      - "game menu"
---

## Game UI/UX Design

### HUD Components

```typescript
// Health bar with smooth transitions
function HealthBar({ current, max }: { current: number; max: number }) {
  const percentage = (current / max) * 100;

  return (
    <div className="health-bar">
      <div
        className="health-bar-fill"
        style={{
          width: `${percentage}%`,
          backgroundColor: percentage > 50 ? '#22c55e' : percentage > 25 ? '#f59e0b' : '#ef4444',
          transition: 'width 0.3s ease-out, background-color 0.3s',
        }}
      />
      <div className="health-bar-text">
        {current} / {max}
      </div>
    </div>
  );
}

// Animated XP bar
function XPBar({ level, xp, xpToNext }: Props) {
  return (
    <div className="xp-container">
      <span className="level-badge">LV {level}</span>
      <div className="xp-bar">
        <motion.div
          className="xp-fill"
          initial={{ width: 0 }}
          animate={{ width: `${(xp / xpToNext) * 100}%` }}
          transition={{ duration: 0.5, ease: 'easeOut' }}
        />
      </div>
      <span className="xp-text">{xp}/{xpToNext}</span>
    </div>
  );
}

// Mini-map
function MiniMap({ entities, playerPosition, mapSize }: Props) {
  return (
    <div className="minimap" style={{ width: mapSize, height: mapSize }}>
      <div
        className="minimap-player"
        style={{
          left: playerPosition.x,
          top: playerPosition.y,
        }}
      />
      {entities.map(entity => (
        <div
          key={entity.id}
          className={`minimap-entity ${entity.type}`}
          style={{
            left: entity.position.x,
            top: entity.position.y,
            backgroundColor: entity.color,
          }}
        />
      ))}
    </div>
  );
}
```

### Game Menus

```typescript
// Animated menu with stagger
function MainMenu() {
  const menuItems = ['Start Game', 'Continue', 'Settings', 'Exit'];

  return (
    <motion.div
      className="menu-container"
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
    >
      <motion.h1
        className="game-title"
        initial={{ y: -50, opacity: 0 }}
        animate={{ y: 0, opacity: 1 }}
      >
        Epic Game
      </motion.h1>

      <ul className="menu-list">
        {menuItems.map((item, index) => (
          <motion.li
            key={item}
            initial={{ x: -50, opacity: 0 }}
            animate={{ x: 0, opacity: 1 }}
            transition={{ delay: index * 0.1 }}
          >
            <MenuButton>{item}</MenuButton>
          </motion.li>
        ))}
      </ul>
    </motion.div>
  );
}

// Menu button with hover effect
function MenuButton({ children }: { children: React.ReactNode }) {
  return (
    <motion.button
      className="menu-button"
      whileHover={{
        scale: 1.05,
        color: '#fbbf24',
        textShadow: '0 0 20px rgba(251, 191, 36, 0.5)',
      }}
      whileTap={{ scale: 0.95 }}
      transition={{ type: 'spring', stiffness: 400, damping: 17 }}
    >
      {children}
    </motion.button>
  );
}
```

### Floating Damage Numbers

```typescript
// Damage popup component
function DamagePopup({ damage, position, isCrit }: DamageProps) {
  return (
    <motion.div
      className={`damage-popup ${isCrit ? 'crit' : ''}`}
      initial={{ y: 0, x: 0, opacity: 1, scale: 0.5 }}
      animate={{
        y: -60,
        opacity: 0,
        scale: isCrit ? 1.5 : 1,
      }}
      transition={{ duration: 0.8, ease: 'easeOut' }}
      style={{ left: position.x, top: position.y }}
    >
      {isCrit && <span className="crit-text">CRIT!</span>}
      {damage}
    </motion.div>
  );
}

// Combo counter
function ComboCounter({ combo, hits }: { combo: number; hits: number }) {
  return (
    <AnimatePresence>
      {combo > 1 && (
        <motion.div
          className="combo-container"
          initial={{ scale: 0, rotate: -10 }}
          animate={{ scale: 1, rotate: 0 }}
          exit={{ scale: 0, opacity: 0 }}
          key={combo}
        >
          <span className="combo-count">{combo}x</span>
          <span className="combo-hits">{hits} hits</span>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

### Inventory UI

```typescript
// Grid-based inventory
function InventoryGrid({
  items,
  slots,
  selectedIndex,
  onSelect,
  onUse,
}: InventoryProps) {
  return (
    <div className="inventory-grid" style={{ '--grid-size': slots }}>
      {Array.from({ length: slots }).map((_, index) => {
        const item = items[index];
        return (
          <motion.div
            key={index}
            className={`inventory-slot ${item ? 'has-item' : ''} ${index === selectedIndex ? 'selected' : ''}`}
            onClick={() => item && onSelect(index)}
            whileHover={{ scale: 1.05 }}
            whileTap={{ scale: 0.95 }}
          >
            {item && (
              <>
                <img src={item.icon} alt={item.name} className="item-icon" />
                {item.count > 1 && <span className="item-count">{item.count}</span>}
                {item.rarity && <div className={`rarity-border rarity-${item.rarity}`} />}
              </>
            )}
          </motion.div>
        );
      })}
    </div>
  );
}

// Tooltip
function ItemTooltip({ item, position }: TooltipProps) {
  return (
    <motion.div
      className={`item-tooltip rarity-${item.rarity}`}
      initial={{ opacity: 0, y: 10 }}
      animate={{ opacity: 1, y: 0 }}
      style={{ left: position.x + 20, top: position.y }}
    >
      <div className="tooltip-header">
        <span className="item-name">{item.name}</span>
        <span className="item-type">{item.type}</span>
      </div>
      <p className="item-description">{item.description}</p>
      <div className="item-stats">
        {item.stats.map(stat => (
          <div key={stat.name} className="stat-row">
            <span>{stat.name}</span>
            <span className="stat-value">+{stat.value}</span>
          </div>
        ))}
      </div>
    </motion.div>
  );
}
```

### Visual Feedback

```typescript
// Screen shake
function useScreenShake(intensity: number = 10) {
  const [offset, setOffset] = useState({ x: 0, y: 0 });

  const shake = () => {
    const x = (Math.random() - 0.5) * intensity;
    const y = (Math.random() - 0.5) * intensity;
    setOffset({ x, y });

    setTimeout(() => setOffset({ x: 0, y: 0 }), 100);
  };

  return { offset, shake };
}

// Hit flash effect
function useHitFlash() {
  const [flash, setFlash] = useState(false);

  const triggerFlash = () => {
    setFlash(true);
    setTimeout(() => setFlash(false), 100);
  };

  return { flash, triggerFlash };
}

// Low health warning
function HealthWarning({ health }: { health: number }) {
  const [pulse, setPulse] = useState(false);

  useEffect(() => {
    if (health <= 25) {
      const interval = setInterval(() => setPulse(p => !p), 500);
      return () => clearInterval(interval);
    }
  }, [health]);

  return (
    <div className={`health-warning ${pulse ? 'pulse' : ''}`}>
      <img src="/warning-icon.svg" alt="Low Health" />
    </div>
  );
}
```

### Accessibility

```typescript
// Key bindings display
function KeyBindings({ bindings }: { bindings: KeyBinding[] }) {
  return (
    <div className="key-bindings" role="list" aria-label="Keyboard controls">
      {bindings.map(binding => (
        <div key={binding.action} className="binding-row" role="listitem">
          <span className="binding-action">{binding.action}</span>
          <div className="binding-keys">
            {binding.keys.map((key, i) => (
              <kbd key={i} className="key-cap">{key}</kbd>
            ))}
          </div>
        </div>
      ))}
    </div>
  );
}

// Color blind modes
function GameCanvas({ colorBlindMode }: { colorBlindMode: string }) {
  const colorMap = {
    default: { health: '#22c55e', damage: '#ef4444', mana: '#3b82f6' },
    protanopia: { health: '#007f5f', damage: '#f5a623', mana: '#7b2cbf' },
    deuteranopia: { health: '#2dc653', damage: '#ff6b35', mana: '#4f46e5' },
  };

  const colors = colorMap[colorBlindMode] || colorMap.default;

  return <Canvas colors={colors} />;
}
```

### Loading & Transitions

```typescript
// Game loading screen
function LoadingScreen({ progress }: { progress: number }) {
  return (
    <div className="loading-screen">
      <motion.div
        className="loading-bar"
        initial={{ width: 0 }}
        animate={{ width: `${progress}%` }}
      />
      <motion.p
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        transition={{ repeat: Infinity, duration: 1 }}
      >
        Loading...
      </motion.p>
    </div>
  );
}

// Scene transition
function SceneTransition({ children, isTransitioning }: Props) {
  return (
    <AnimatePresence mode="wait">
      {isTransitioning ? (
        <motion.div
          className="scene-transition"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          <div className="transition-effect" />
        </motion.div>
      ) : (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          {children}
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

### Best Practices

1. **Consistency** - Same style for all UI elements
2. **Readability** - High contrast, legible fonts
3. **Feedback** - Every action needs response
4. **Accessibility** - Color blind modes, key bindings
5. **Performance** - Minimize UI overdraw
6. **Responsive** - Scale to different screen sizes
7. **Animation** - Smooth, purposeful motion
