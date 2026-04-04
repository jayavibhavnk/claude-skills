---
name: game-inventory-ui
description: Game inventory UI - grid systems, item tooltips, equipment slots, sorting, searching, and drag-drop organization.
metadata:
  priority: 9
  docs: []
  pathPatterns:
    - "**/inventory/**"
    - "**/items/**"
  bashPatterns:
    - '\binventory\b'
    - '\bitem\b'
  promptSignals:
    phrases:
      - "inventory UI"
      - "item grid"
      - "equipment"
    anyOf:
      - "inventory"
      - "equipment"
      - "items"
---

## Game Inventory UI

### Inventory Grid

```typescript
// Main inventory grid component
interface InventoryProps {
  items: Item[];
  capacity: number;
  onItemClick?: (item: Item) => void;
  onItemDrop?: (item: Item, slot: number) => void;
  onItemDrag?: (item: Item) => void;
  categories?: ItemCategory[];
}

function InventoryGrid({
  items,
  capacity,
  onItemClick,
  onItemDrop,
  selectedItem,
}: InventoryProps) {
  const [sortBy, setSortBy] = useState<SortOption>('type');

  const slots = Array.from({ length: capacity }, (_, i) => {
    const item = items.find(it => it.slot === i);
    return { slot: i, item };
  });

  return (
    <div className="bg-slate-900/80 rounded-xl p-4">
      {/* Header */}
      <div className="flex items-center justify-between mb-4">
        <h3 className="text-lg font-bold text-white">
          Inventory ({items.length}/{capacity})
        </h3>

        <InventorySort value={sortBy} onChange={setSortBy} />
      </div>

      {/* Grid */}
      <div className="grid grid-cols-5 gap-2">
        {slots.map(({ slot, item }) => (
          <InventorySlot
            key={slot}
            item={item}
            isSelected={selectedItem?.slot === slot}
            onClick={() => item && onItemClick?.(item)}
            onDrop={(droppedItem) => onItemDrop?.(droppedItem, slot)}
          />
        ))}
      </div>

      {/* Capacity bar */}
      <CapacityBar current={items.length} max={capacity} />
    </div>
  );
}

// Single inventory slot
function InventorySlot({
  item,
  isSelected,
  onClick,
  onDrop,
}: SlotProps) {
  const [isDragOver, setIsDragOver] = useState(false);

  return (
    <motion.div
      className={`
        relative aspect-square rounded-lg cursor-pointer
        border-2 transition-colors
        ${item
          ? isSelected
            ? 'bg-amber-900/30 border-amber-500'
            : 'bg-slate-800 border-slate-700 hover:border-slate-600'
          : 'bg-slate-800/30 border-slate-800'
        }
        ${isDragOver && 'border-amber-400 bg-amber-900/20'}
      `}
      whileHover={item ? { scale: 1.05 } : undefined}
      whileTap={item ? { scale: 0.95 } : undefined}
      onClick={onClick}
      onDragOver={(e) => { e.preventDefault(); setIsDragOver(true); }}
      onDragLeave={() => setIsDragOver(false)}
      onDrop={(e) => {
        e.preventDefault();
        setIsDragOver(false);
        const draggedItem = JSON.parse(e.dataTransfer.getData('item'));
        onDrop?.(draggedItem);
      }}
      draggable={!!item}
      onDragStart={(e) => {
        if (item) {
          e.dataTransfer.setData('item', JSON.stringify(item));
        }
      }}
    >
      {item && (
        <>
          <img
            src={item.icon}
            alt={item.name}
            className="w-full h-full p-2 object-contain"
          />

          {/* Stack count */}
          {item.stackSize > 1 && (
            <div className="absolute bottom-0 right-0 px-1 bg-black/60 rounded-tl text-xs font-bold text-white">
              {item.quantity}
            </div>
          )}

          {/* Rarity border */}
          <div
            className={`absolute inset-0 rounded-lg border-2 pointer-events-none
              ${getRarityBorder(item.rarity)}`}
          />
        </>
      )}

      {/* Empty slot pattern */}
      {!item && (
        <div className="absolute inset-0 flex items-center justify-center opacity-20">
          <div className="w-1/3 h-1/3 border-2 border-dashed border-slate-600 rounded" />
        </div>
      )}
    </motion.div>
  );
}
```

### Equipment Slots

```typescript
// Character equipment panel
function EquipmentPanel({ character, onUnequip }: Props) {
  const equipmentSlots = [
    { id: 'head', label: 'Head', icon: HelmetIcon, slot: character.equipment.head },
    { id: 'chest', label: 'Chest', icon: ChestIcon, slot: character.equipment.chest },
    { id: 'hands', label: 'Hands', icon: GlovesIcon, slot: character.equipment.hands },
    { id: 'legs', label: 'Legs', icon: BootsIcon, slot: character.equipment.legs },
    { id: 'mainHand', label: 'Weapon', icon: SwordIcon, slot: character.equipment.mainHand },
    { id: 'offHand', label: 'Off-hand', icon: ShieldIcon, slot: character.equipment.offHand },
    { id: 'ring1', label: 'Ring', icon: RingIcon, slot: character.equipment.ring1 },
    { id: 'ring2', label: 'Ring', icon: RingIcon, slot: character.equipment.ring2 },
  ];

  return (
    <div className="bg-slate-900/80 rounded-xl p-6">
      {/* Character preview */}
      <div className="relative w-48 h-64 mx-auto mb-6">
        <img
          src={character.sprite}
          className="w-full h-full object-contain"
        />

        {/* Equipment overlays */}
        {equipmentSlots.map(({ id, slot }) => (
          slot && (
            <EquipmentOverlay
              key={id}
              equipmentType={id}
              item={slot}
            />
          )
        ))}
      </div>

      {/* Equipment slots grid */}
      <div className="grid grid-cols-4 gap-3">
        {equipmentSlots.map(({ id, label, icon: Icon, slot }) => (
          <EquipmentSlot
            key={id}
            type={id}
            label={label}
            icon={Icon}
            item={slot}
            onEquip={(item) => handleEquip(id, item)}
            onUnequip={() => onUnequip(id)}
          />
        ))}
      </div>

      {/* Stats summary */}
      <CharacterStats character={character} />
    </div>
  );
}

function EquipmentSlot({
  type,
  label,
  icon: Icon,
  item,
  onEquip,
  onUnequip,
}: EquipmentSlotProps) {
  const [showTooltip, setShowTooltip] = useState(false);

  return (
    <div className="relative">
      <motion.div
        className={`
          relative aspect-square rounded-lg cursor-pointer
          border-2 transition-colors
          ${item
            ? 'bg-slate-700 border-amber-600'
            : 'bg-slate-800/50 border-slate-700 hover:border-slate-600'
          }
        `}
        whileHover={{ scale: 1.05 }}
        onClick={() => item ? onUnequip() : showTooltip && onEquip?.()}
        onContextMenu={(e) => {
          e.preventDefault();
          if (item) onUnequip();
        }}
      >
        {item ? (
          <img
            src={item.icon}
            alt={label}
            className="w-full h-full p-2 object-contain"
            onMouseEnter={() => setShowTooltip(true)}
            onMouseLeave={() => setShowTooltip(false)}
          />
        ) : (
          <Icon className="w-full h-full p-3 text-slate-600" />
        )}
      </motion.div>

      {/* Tooltip */}
      {item && showTooltip && (
        <ItemTooltip item={item} position="top" />
      )}

      {/* Label */}
      <div className="text-xs text-center text-slate-500 mt-1">{label}</div>
    </div>
  );
}
```

### Item Tooltip

```typescript
// Rich item tooltip
function ItemTooltip({ item, position = 'right' }: TooltipProps) {
  const rarityColors = {
    common: { bg: 'bg-slate-800', border: 'border-slate-600', text: 'text-slate-300' },
    uncommon: { bg: 'bg-green-900', border: 'border-green-500', text: 'text-green-300' },
    rare: { bg: 'bg-blue-900', border: 'border-blue-500', text: 'text-blue-300' },
    epic: { bg: 'bg-purple-900', border: 'border-purple-500', text: 'text-purple-300' },
    legendary: { bg: 'bg-amber-900', border: 'border-amber-500', text: 'text-amber-300' },
  };

  const colors = rarityColors[item.rarity];

  return (
    <motion.div
      className={`
        fixed z-50 w-72 p-4 rounded-xl
        ${colors.bg} border ${colors.border}
        shadow-2xl shadow-black/50
      `}
      initial={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      style={{
        [position === 'right' ? 'left' : 'right']: '100%',
        marginLeft: position === 'right' ? '8px' : '0',
        marginRight: position === 'left' ? '8px' : '0',
      }}
    >
      {/* Header */}
      <div className="flex items-start gap-3">
        <img src={item.icon} className="w-14 h-14 rounded-lg bg-slate-700 p-1" />
        <div className="flex-1">
          <div className={`font-bold ${colors.text}`}>{item.name}</div>
          <div className="text-xs text-slate-500 capitalize">{item.category}</div>
          {item.level && (
            <Badge variant="outline" size="sm" className="mt-1">
              Level {item.level}
            </Badge>
          )}
        </div>
      </div>

      {/* Divider */}
      <div className="my-3 border-t border-slate-700" />

      {/* Description */}
      {item.description && (
        <p className="text-sm text-slate-400 italic mb-3">{item.description}</p>
      )}

      {/* Stats */}
      {item.stats && (
        <div className="space-y-1 mb-3">
          {item.stats.map((stat, i) => (
            <div key={i} className="flex justify-between text-sm">
              <span className="text-slate-400">{stat.name}</span>
              <span className={stat.value >= 0 ? 'text-green-400' : 'text-red-400'}>
                {stat.value >= 0 ? '+' : ''}{stat.value}
                {stat.suffix || ''}
              </span>
            </div>
          ))}
        </div>
      )}

      {/* Item type and slot */}
      <div className="text-xs text-slate-500">
        {item.type} {item.slot && `• ${item.slot} slot`}
      </div>

      {/* Sell price */}
      <div className="mt-3 pt-3 border-t border-slate-700 flex items-center gap-2">
        <CoinStack className="w-4 h-4 text-amber-400" />
        <span className="text-sm text-amber-400">{item.sellPrice}</span>
        <span className="text-xs text-slate-500">sell value</span>
      </div>
    </motion.div>
  );
}
```

### Sorting and Filtering

```typescript
// Inventory sort options
type SortOption = 'type' | 'rarity' | 'level' | 'name' | 'value';

function InventorySort({ value, onChange }: Props) {
  return (
    <select
      value={value}
      onChange={(e) => onChange(e.target.value as SortOption)}
      className="bg-slate-800 text-slate-300 text-sm rounded-lg px-3 py-1.5 border border-slate-700"
    >
      <option value="type">Sort by Type</option>
      <option value="rarity">Sort by Rarity</option>
      <option value="level">Sort by Level</option>
      <option value="name">Sort by Name</option>
      <option value="value">Sort by Value</option>
    </select>
  );
}

// Filter by category
function InventoryFilters({ categories, onFilter }: Props) {
  return (
    <div className="flex gap-2 flex-wrap">
      <FilterButton
        active={true}
        onClick={() => onFilter(null)}
      >
        All
      </FilterButton>

      {categories.map(cat => (
        <FilterButton
          key={cat.id}
          active={false}
          onClick={() => onFilter(cat.id)}
        >
          {cat.name}
        </FilterButton>
      ))}
    </div>
  );
}
```

### Stack Management

```typescript
// Stack splitting UI
function StackSplitDialog({
  item,
  maxSplit,
  onConfirm,
  onCancel,
}: Props) {
  const [quantity, setQuantity] = useState(Math.floor(maxSplit / 2));

  return (
    <div className="fixed inset-0 flex items-center justify-center z-50">
      <div className="bg-slate-900 rounded-xl p-6 w-72">
        <h3 className="text-lg font-bold text-white mb-4">Split Stack</h3>

        <div className="flex items-center gap-4">
          <img src={item.icon} className="w-16 h-16" />
          <div>
            <div className="font-medium text-white">{item.name}</div>
            <div className="text-sm text-slate-400">
              Quantity: {item.quantity}
            </div>
          </div>
        </div>

        <div className="mt-4">
          <label className="text-sm text-slate-400">Split amount:</label>
          <div className="flex items-center gap-3 mt-1">
            <button
              onClick={() => setQuantity(q => Math.max(1, q - 1))}
              className="w-10 h-10 bg-slate-800 rounded-lg"
            >
              -
            </button>
            <input
              type="number"
              value={quantity}
              onChange={(e) => setQuantity(Math.min(maxSplit, Math.max(1, +e.target.value)))}
              className="w-20 text-center bg-slate-800 rounded-lg px-3 py-2 text-white"
            />
            <button
              onClick={() => setQuantity(q => Math.min(maxSplit, q + 1))}
              className="w-10 h-10 bg-slate-800 rounded-lg"
            >
              +
            </button>
          </div>
        </div>

        <div className="flex gap-3 mt-6">
          <button
            onClick={onCancel}
            className="flex-1 py-2 bg-slate-700 rounded-lg text-slate-300"
          >
            Cancel
          </button>
          <button
            onClick={() => onConfirm(quantity)}
            className="flex-1 py-2 bg-amber-600 rounded-lg text-white"
          >
            Split
          </button>
        </div>
      </div>
    </div>
  );
}
```

### Loot Animation

```typescript
// Loot drop animation
function LootDrop({ items, position, onCollect }: Props) {
  return (
    <div
      className="absolute"
      style={{ left: position.x, top: position.y }}
    >
      {items.map((item, index) => (
        <motion.div
          key={item.id}
          initial={{
            opacity: 0,
            y: 0,
            scale: 0,
          }}
          animate={{
            opacity: 1,
            y: -50 - index * 30,
            scale: 1,
          }}
          transition={{
            delay: index * 0.1,
            duration: 0.3,
            type: 'spring',
          }}
          className="absolute"
        >
          <div
            className="w-12 h-12 rounded-lg bg-slate-800 border border-slate-600 flex items-center justify-center cursor-pointer"
            onClick={() => onCollect(item)}
          >
            <img src={item.icon} className="w-8 h-8" />
          </div>
        </motion.div>
      ))}
    </div>
  );
}
```

### Best Practices

1. **Clear grid** - Visual slots with state
2. **Rarity colors** - Quick identification
3. **Stack counts** - Always visible
4. **Sorting/filtering** - Find items easily
5. **Drag and drop** - Intuitive organization
6. **Tooltips** - Rich item info on hover
7. **Capacity indicator** - Know when full
