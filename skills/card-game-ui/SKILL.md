---
name: card-game-ui
description: Card game UI design - card components, drag-and-drop, hand management, play zones, animations, and polished card game UX.
metadata:
  priority: 9
  docs:
    - "https://framer-motion.com/"
  pathPatterns:
    - "**/cards/**"
    - "**/card-game/**"
  bashPatterns:
    - '\bcard\b'
    - '\bhand\b'
  promptSignals:
    phrases:
      - "card game UI"
      - "card animation"
      - "card drag"
    anyOf:
      - "card ui"
      - "card game"
      - "hand"
---

## Card Game UI

### Card Component

```typescript
// Card component with all states
interface CardProps {
  card: CardData;
  size?: 'small' | 'medium' | 'large';
  isSelected?: boolean;
  isPlayable?: boolean;
  isDraggable?: boolean;
  isHoverable?: boolean;
  disabled?: boolean;
}

function Card({
  card,
  size = 'medium',
  isSelected,
  isPlayable,
  isDraggable,
  disabled,
}: CardProps) {
  const [isHovered, setIsHovered] = useState(false);

  const sizeClasses = {
    small: 'w-20 h-28',
    medium: 'w-32 h-44',
    large: 'w-40 h-56',
  };

  return (
    <motion.div
      className={`
        relative rounded-xl cursor-pointer
        bg-gradient-to-br from-amber-100 to-amber-200
        border-2 ${isSelected ? 'border-yellow-400' : 'border-amber-300'}
        ${disabled ? 'opacity-50' : ''}
        ${sizeClasses[size]}
      `}
      initial={false}
      animate={{
        scale: isHovered ? 1.1 : isSelected ? 1.05 : 1,
        y: isHovered ? -20 : 0,
        rotateY: isSelected ? 0 : 0,
      }}
      whileTap={{ scale: isDraggable ? 0.95 : 1 }}
      onHoverStart={() => setIsHovered(true)}
      onHoverEnd={() => setIsHovered(false)}
      transition={{ type: 'spring', stiffness: 300, damping: 20 }}
    >
      {/* Card art area */}
      <div className="absolute inset-x-1 top-1 h-1/2 rounded-t-lg overflow-hidden">
        <img src={card.artUrl} className="w-full h-full object-cover" />
      </div>

      {/* Card frame and info */}
      <div className="absolute inset-x-0 bottom-0 p-1">
        <div className="text-xs font-bold text-center truncate">
          {card.name}
        </div>
        <div className="flex justify-between items-center px-1">
          <span className="text-lg font-bold">{card.cost}</span>
          <div className="flex gap-1">
            <Badge variant="attack">{card.attack}</Badge>
            <Badge variant="defense">{card.defense}</Badge>
          </div>
        </div>
      </div>

      {/* Highlight for playable */}
      {isPlayable && !disabled && (
        <motion.div
          className="absolute inset-0 rounded-xl border-2 border-green-400"
          initial={{ opacity: 0 }}
          animate={{ opacity: [0.5, 1, 0.5] }}
          transition={{ repeat: Infinity, duration: 1.5 }}
        />
      )}

      {/* Glow effect */}
      {isSelected && (
        <div
          className="absolute inset-0 rounded-xl"
          style={{ boxShadow: '0 0 20px rgba(251, 191, 36, 0.6)' }}
        />
      )}
    </motion.div>
  );
}
```

### Hand Management

```typescript
// Fan-style hand layout
function CardHand({
  cards,
  selectedIndex,
  onSelectCard,
  onPlayCard,
  maxVisible = 7,
}: HandProps) {
  const handRef = useRef<HTMLDivElement>(null);

  // Calculate card positions in a fan
  const getCardTransform = (index: number, total: number) => {
    const centerIndex = (total - 1) / 2;
    const offset = index - centerIndex;
    const angle = offset * 5; // degrees per card
    const translateY = Math.abs(offset) * -10;
    const zIndex = total - Math.abs(offset);

    return {
      angle,
      translateY,
      zIndex,
      scale: index === selectedIndex ? 1.15 : 1,
    };
  };

  return (
    <div
      ref={handRef}
      className="relative h-56 flex items-end justify-center"
    >
      {cards.map((card, index) => {
        const { angle, translateY, zIndex, scale } = getCardTransform(
          index,
          cards.length
        );

        return (
          <motion.div
            key={card.id}
            className="absolute"
            style={{ zIndex }}
            initial={false}
            animate={{
              rotate: angle,
              y: translateY + (index === selectedIndex ? -40 : 0),
              scale,
              x: index * 30 - (cards.length * 15),
            }}
            transition={{ type: 'spring', stiffness: 300, damping: 25 }}
            onClick={() => onSelectCard(index)}
            onDoubleClick={() => onPlayCard(card)}
          >
            <Card
              card={card}
              size="medium"
              isSelected={index === selectedIndex}
              isPlayable={card.cost <= currentMana}
            />
          </motion.div>
        );
      })}
    </div>
  );
}
```

### Drag and Drop

```typescript
// Draggable card with drag zone detection
function DraggableCard({ card, onPlay, onCancel }: Props) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [isDragging, setIsDragging] = useState(false);
  const dragRef = useRef<HTMLDivElement>(null);

  // Drop zones
  const dropZones = [
    { id: 'field', bounds: fieldBounds, label: 'Play to Field' },
    { id: 'spell', bounds: spellBounds, label: 'Cast Spell' },
    { id: 'discard', bounds: discardBounds, label: 'Discard' },
  ];

  const handleDragEnd = () => {
    setIsDragging(false);

    // Check which zone we're over
    const cardRect = dragRef.current?.getBoundingClientRect();
    if (!cardRect) return;

    for (const zone of dropZones) {
      if (isOverRect(position, zone.bounds)) {
        if (zone.id === 'field' && card.type !== 'creature') continue;
        onPlay(card, zone.id);
        return;
      }
    }

    // Return to hand animation
    onCancel();
  };

  return (
    <motion.div
      ref={dragRef}
      drag
      dragMomentum={false}
      onDragStart={() => setIsDragging(true)}
      onDragEnd={handleDragEnd}
      animate={position}
      whileDrag={{ scale: 1.15, zIndex: 100, boxShadow: '0 20px 40px rgba(0,0,0,0.3)' }}
      className="cursor-grab active:cursor-grabbing"
    >
      <Card card={card} isDraggable />
    </motion.div>
  );
}

// Drop zone indicator
function DropZone({ zone, isActive }: DropZoneProps) {
  return (
    <motion.div
      className={`
        absolute border-2 border-dashed rounded-xl
        flex items-center justify-center
        transition-colors duration-200
      `}
      style={{
        ...zone.bounds,
        borderColor: isActive ? '#22c55e' : '#64748b',
        backgroundColor: isActive ? 'rgba(34, 197, 94, 0.1)' : 'transparent',
      }}
      animate={{
        scale: isActive ? 1.05 : 1,
        opacity: isActive ? 1 : 0.7,
      }}
    >
      <span className={`text-sm ${isActive ? 'text-green-500' : 'text-slate-400'}`}>
        {zone.label}
      </span>
    </motion.div>
  );
}
```

### Card Preview

```typescript
// Card preview on hover
function CardPreview({ card, position }: Props) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 10, scale: 0.9 }}
      animate={{ opacity: 1, y: 0, scale: 1 }}
      exit={{ opacity: 0, y: 10, scale: 0.9 }}
      className="fixed z-50 pointer-events-none"
      style={{
        left: position.x + 20,
        top: position.y,
      }}
    >
      <div className="w-64 bg-gradient-to-br from-amber-900 to-amber-950 rounded-xl p-4 shadow-2xl">
        <img src={card.artUrl} className="w-full h-40 object-cover rounded-lg" />
        <h3 className="text-amber-100 font-bold mt-2">{card.name}</h3>
        <p className="text-amber-200 text-sm mt-1">{card.type}</p>
        <div className="flex gap-2 mt-2">
          <Badge variant="attack">{card.attack}</Badge>
          <Badge variant="defense">{card.defense}</Badge>
          <Badge variant="cost">{card.cost}</Badge>
        </div>
        <p className="text-amber-100 text-sm mt-3">{card.description}</p>

        {/* Expanded effects */}
        {card.effects?.length > 0 && (
          <div className="mt-3 pt-3 border-t border-amber-700">
            {card.effects.map((effect, i) => (
              <div key={i} className="text-amber-200 text-xs">
                • {effect.description}
              </div>
            ))}
          </div>
        )}
      </div>
    </motion.div>
  );
}
```

### Play Animation

```typescript
// Card play animations
function useCardPlayAnimation() {
  const playCardToField = (
    cardElement: HTMLElement,
    fieldPosition: Vector2,
    onComplete: () => void
  ) => {
    const rect = cardElement.getBoundingClientRect();

    // Animate from hand to field
    const startX = rect.left + rect.width / 2;
    const startY = rect.top + rect.height / 2;

    return {
      initial: { x: startX, y: startY, scale: 1 },
      animate: {
        x: fieldPosition.x,
        y: fieldPosition.y,
        scale: 0.8,
        rotate: Math.random() * 10 - 5,
      },
      transition: { duration: 0.4, ease: 'easeOut' },
      onComplete,
    };
  };

  const cardBurstEffect = (position: Vector2) => {
    // Spawn particles on card play
    particleSystem.emit({
      x: position.x,
      y: position.y,
      count: 20,
      speed: 150,
      colors: ['#fbbf24', '#f59e0b', '#fef3c7'],
      life: 0.6,
    });
  };

  return { playCardToField, cardBurstEffect };
}
```

### Board Zones

```typescript
// Game board with zones
function GameBoard({
  playerField,
  enemyField,
  playerHand,
  playerDiscard,
  onPlayCard,
}: BoardProps) {
  return (
    <div className="relative w-full h-screen bg-gradient-to-b from-slate-800 to-slate-900">
      {/* Enemy field */}
      <div className="absolute top-20 left-1/2 -translate-x-1/2">
        <div className="flex gap-2 justify-center">
          {enemyField.map((card, i) => (
            <motion.div
              key={card?.id || i}
              initial={{ opacity: 0, y: -50 }}
              animate={{ opacity: 1, y: 0 }}
              className="relative"
            >
              {card && <Card card={card} size="small" isDraggable={false} />}
            </motion.div>
          ))}
        </div>
        {/* Enemy hand (face down) */}
        <div className="flex justify-center mt-4 gap-1">
          {Array.from({ length: enemyHandCount }).map((_, i) => (
            <div
              key={i}
              className="w-12 h-16 bg-gradient-to-br from-slate-600 to-slate-700 rounded-lg border border-slate-500"
            />
          ))}
        </div>
      </div>

      {/* Play zones */}
      <div className="absolute inset-x-0 top-1/2 -translate-y-1/2">
        <DropZone zone={fieldZone} isActive={isDragging} />
      </div>

      {/* Player field */}
      <div className="absolute bottom-32 left-1/2 -translate-x-1/2">
        <div className="flex gap-2 justify-center">
          {playerField.map((card, i) => (
            <Card key={card?.id || i} card={card!} size="medium" />
          ))}
        </div>
      </div>

      {/* Player hand */}
      <div className="absolute bottom-0 left-0 right-0 h-56">
        <CardHand
          cards={playerHand}
          selectedIndex={selectedCard}
          onSelectCard={setSelectedCard}
          onPlayCard={onPlayCard}
        />
      </div>

      {/* Discard pile */}
      <div className="absolute bottom-32 right-8">
        <div className="text-xs text-slate-400 mb-1">Discard</div>
        <div
          className="w-16 h-24 rounded-lg bg-slate-700 flex items-center justify-center text-slate-500"
        >
          {playerDiscard.length > 0 ? (
            <Card card={playerDiscard[0]} size="small" />
          ) : (
            <span>0</span>
          )}
        </div>
      </div>
    </div>
  );
}
```

### Mana Display

```typescript
// Animated mana display
function ManaDisplay({ current, max }: Props) {
  return (
    <div className="flex items-center gap-3">
      <div className="relative w-16 h-16">
        <motion.div
          className="absolute inset-0 rounded-full bg-gradient-to-br from-blue-400 to-blue-600"
          initial={false}
          animate={{
            scale: [1, 1.05, 1],
          }}
          transition={{ duration: 2, repeat: Infinity }}
        />
        <div className="absolute inset-1 rounded-full bg-blue-500 flex items-center justify-center">
          <span className="text-2xl font-bold text-white">
            {current}
          </span>
        </div>
      </div>

      {/* Animated fill */}
      <div className="w-32 h-4 bg-slate-700 rounded-full overflow-hidden">
        <motion.div
          className="h-full bg-gradient-to-r from-blue-500 to-cyan-400"
          initial={false}
          animate={{ width: `${(current / max) * 100}%` }}
          transition={{ type: 'spring', stiffness: 100, damping: 20 }}
        />
      </div>

      <span className="text-slate-300">/ {max}</span>
    </div>
  );
}
```

### Turn Indicator

```typescript
// Turn banner animation
function TurnIndicator({ isPlayerTurn, turnNumber }: Props) {
  return (
    <AnimatePresence>
      {isPlayerTurn && (
        <motion.div
          initial={{ y: -100, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          exit={{ y: -100, opacity: 0 }}
          className="absolute top-0 left-1/2 -translate-x-1/2 z-50"
        >
          <div className="bg-gradient-to-r from-amber-500 to-orange-500 px-8 py-2 rounded-b-xl shadow-lg">
            <span className="text-white font-bold">
              Your Turn - Round {turnNumber}
            </span>
          </div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

### Card Selection

```typescript
// Multi-card selection for combo plays
function SelectableHand({ cards, onSelect, maxSelect }: HandProps) {
  const [selected, setSelected] = useState<number[]>([]);

  const toggleCard = (index: number) => {
    setSelected(prev => {
      if (prev.includes(index)) {
        return prev.filter(i => i !== index);
      }
      if (prev.length >= maxSelect) {
        // Replace oldest selection
        return [...prev.slice(1), index];
      }
      return [...prev, index];
    });

    onSelect(selected);
  };

  return (
    <div className="flex gap-2 justify-center">
      {cards.map((card, index) => (
        <motion.div
          key={card.id}
          animate={{
            y: selected.includes(index) ? -30 : 0,
            scale: selected.includes(index) ? 1.1 : 1,
          }}
          onClick={() => toggleCard(index)}
        >
          <Card
            card={card}
            isSelected={selected.includes(index)}
            isPlayable={selected.length < maxSelect || selected.includes(index)}
          />
        </motion.div>
      ))}
    </div>
  );
}
```

### Best Practices

1. **Clear feedback** - Every action needs visual response
2. **Smooth animations** - Use spring physics for natural feel
3. **Card visibility** - Always show key card info
4. **Drag zones** - Clear drop targets
5. **Hover states** - Preview on hover
6. **Sound sync** - Match audio to animations
7. **Mobile support** - Touch-friendly card interactions
