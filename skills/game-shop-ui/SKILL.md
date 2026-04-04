---
name: game-shop-ui
description: Game shop UI - buy/sell interface, currency display, item cards, shop categories, and transaction animations.
metadata:
  priority: 8
  docs: []
  pathPatterns:
    - "**/shop/**"
    - "**/store/**"
  bashPatterns:
    - '\bshop\b'
    - '\bstore\b'
  promptSignals:
    phrases:
      - "game shop"
      - "buy sell"
      - "item shop"
    anyOf:
      - "shop"
      - "store"
      - "buy"
---

## Game Shop UI

### Shop Types

```typescript
// Different shop configurations
interface Shop {
  id: string;
  name: string;
  type: ShopType;
  items: ShopItem[];
  refreshCost?: number;
  refreshInterval?: number; // ms
  buybackPercentage?: number; // For sell back items
}

type ShopType =
  | 'general'      // Basic items
  | 'armory'       // Weapons, armor
  | 'potion'      // Consumables
  | 'blacksmith'   // Upgrades, crafting
  | 'gamble'       // Random items
  | 'limited';    // Rare/seasonal items

interface ShopItem {
  id: string;
  item: Item;
  price: number;
  stock: number | null; // null = unlimited
  category: string;
  discount?: number;
  unlockRequirement?: string;
}
```

### Shop Screen

```typescript
// Main shop component
function ShopScreen({ shop, playerMoney, onPurchase, onSell, onClose }: Props) {
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [selectedItem, setSelectedItem] = useState<ShopItem | null>(null);
  const [quantity, setQuantity] = useState(1);

  const filteredItems = getFilteredItems(shop.items, selectedCategory);

  return (
    <div className="fixed inset-0 bg-black/80 z-40">
      <div className="flex h-full">
        {/* Left panel - Categories */}
        <div className="w-64 bg-slate-900 p-4 border-r border-slate-800">
          <ShopHeader shop={shop} playerMoney={playerMoney} />

          <div className="mt-6 space-y-1">
            {getCategories(shop.items).map(cat => (
              <CategoryButton
                key={cat.id}
                category={cat}
                isSelected={selectedCategory === cat.id}
                onClick={() => setSelectedCategory(cat.id)}
              />
            ))}
          </div>
        </div>

        {/* Center - Items */}
        <div className="flex-1 p-6 overflow-y-auto">
          <div className="grid grid-cols-3 gap-4">
            {filteredItems.map(shopItem => (
              <ShopItemCard
                key={shopItem.id}
                shopItem={shopItem}
                isSelected={selectedItem?.id === shopItem.id}
                onSelect={() => setSelectedItem(shopItem)}
                canAfford={playerMoney >= shopItem.price}
              />
            ))}
          </div>
        </div>

        {/* Right panel - Details/Purchase */}
        {selectedItem && (
          <ItemDetailPanel
            shopItem={selectedItem}
            quantity={quantity}
            onQuantityChange={setQuantity}
            playerMoney={playerMoney}
            onPurchase={() => onPurchase(selectedItem, quantity)}
            onSell={() => onSell(selectedItem.item, quantity)}
          />
        )}
      </div>
    </div>
  );
}

// Shop header with currency
function ShopHeader({ shop, playerMoney }: Props) {
  return (
    <div>
      <h2 className="text-xl font-bold text-amber-400">{shop.name}</h2>

      <div className="flex items-center gap-2 mt-4 p-3 bg-slate-800 rounded-lg">
        <CoinStack className="w-6 h-6 text-amber-400" />
        <span className="text-xl font-bold text-white">
          {playerMoney.toLocaleString()}
        </span>
      </div>

      {shop.refreshCost && (
        <button className="w-full mt-4 p-2 bg-slate-800 hover:bg-slate-700 rounded-lg text-sm">
          Refresh Shop ({shop.refreshCost} coins)
        </button>
      )}
    </div>
  );
}
```

### Item Cards

```typescript
// Shop item card
function ShopItemCard({
  shopItem,
  isSelected,
  canAfford,
  onSelect,
}: ShopItemCardProps) {
  const { item, price, stock, discount } = shopItem;

  return (
    <motion.div
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.98 }}
      onClick={onSelect}
      className={`
        relative p-4 rounded-xl cursor-pointer
        transition-colors duration-150
        ${isSelected
          ? 'bg-amber-900/30 border-2 border-amber-500'
          : 'bg-slate-800/50 border border-slate-700 hover:border-slate-600'
        }
        ${!canAfford && 'opacity-60'}
      `}
    >
      {/* Discount badge */}
      {discount && (
        <div className="absolute -top-2 -right-2 px-2 py-1 bg-red-500 rounded-full text-xs font-bold text-white">
          -{discount}%
        </div>
      )}

      {/* Item image */}
      <img
        src={item.icon}
        alt={item.name}
        className="w-16 h-16 mx-auto rounded-lg bg-slate-700 p-2"
      />

      {/* Item name */}
      <div className="mt-2 text-center font-medium text-white truncate">
        {item.name}
      </div>

      {/* Price */}
      <div className="flex items-center justify-center gap-1 mt-1">
        <CoinStack className="w-4 h-4 text-amber-400" />
        <span className={canAfford ? 'text-amber-400' : 'text-red-400'}>
          {formatPrice(price)}
        </span>
      </div>

      {/* Stock */}
      {stock !== null && (
        <div className={`text-center text-xs mt-1 ${stock <= 3 ? 'text-red-400' : 'text-slate-500'}`}>
          {stock === 0 ? 'Sold Out' : `${stock} left`}
        </div>
      )}
    </motion.div>
  );
}

// Detailed item view
function ItemDetailPanel({
  shopItem,
  quantity,
  onQuantityChange,
  playerMoney,
  onPurchase,
}: DetailProps) {
  const { item, price, stock } = shopItem;
  const total = price * quantity;
  const canBuy = total <= playerMoney && (stock === null || stock >= quantity);

  return (
    <div className="w-80 bg-slate-900 p-6 border-l border-slate-800">
      {/* Item header */}
      <div className="text-center">
        <img
          src={item.icon}
          className="w-24 h-24 mx-auto rounded-xl bg-slate-800 p-4"
        />
        <h3 className="mt-4 text-lg font-bold text-white">{item.name}</h3>
        <p className="text-sm text-slate-400">{item.category}</p>
      </div>

      {/* Description */}
      <p className="mt-4 text-sm text-slate-300">{item.description}</p>

      {/* Stats */}
      {item.stats && (
        <div className="mt-4 space-y-1">
          {item.stats.map((stat, i) => (
            <div key={i} className="flex justify-between text-sm">
              <span className="text-slate-400">{stat.name}</span>
              <span className="text-green-400">+{stat.value}</span>
            </div>
          ))}
        </div>
      )}

      {/* Quantity selector */}
      <div className="mt-6">
        <label className="text-sm text-slate-400">Quantity</label>
        <div className="flex items-center gap-3 mt-1">
          <button
            onClick={() => onQuantityChange(Math.max(1, quantity - 1))}
            className="w-10 h-10 bg-slate-800 hover:bg-slate-700 rounded-lg"
          >
            -
          </button>
          <span className="text-xl font-bold text-white w-12 text-center">
            {quantity}
          </span>
          <button
            onClick={() => onQuantityChange(Math.min(stock || 99, quantity + 1))}
            className="w-10 h-10 bg-slate-800 hover:bg-slate-700 rounded-lg"
          >
            +
          </button>
        </div>
      </div>

      {/* Total */}
      <div className="mt-6 p-4 bg-slate-800 rounded-lg">
        <div className="flex justify-between text-sm">
          <span className="text-slate-400">Total</span>
          <span className="text-amber-400 font-bold">
            {total.toLocaleString()} coins
          </span>
        </div>
      </div>

      {/* Buy button */}
      <button
        onClick={onPurchase}
        disabled={!canBuy}
        className={`
          w-full mt-4 py-3 rounded-lg font-bold
          ${canBuy
            ? 'bg-amber-600 hover:bg-amber-500 text-white'
            : 'bg-slate-700 text-slate-500 cursor-not-allowed'
          }
        `}
      >
        {!canBuy && stock === 0 ? 'Sold Out' : 'Purchase'}
      </button>
    </div>
  );
}
```

### Currency Display

```typescript
// Animated coin counter
function CoinDisplay({ amount, size = 'md' }: Props) {
  const [displayAmount, setDisplayAmount] = useState(amount);

  useEffect(() => {
    const diff = amount - displayAmount;
    if (diff === 0) return;

    const step = Math.ceil(Math.abs(diff) / 10);
    const interval = setInterval(() => {
      setDisplayAmount(prev => {
        const next = prev + (diff > 0 ? step : -step);
        if ((diff > 0 && next >= amount) || (diff < 0 && next <= amount)) {
          clearInterval(interval);
          return amount;
        }
        return next;
      });
    }, 30);

    return () => clearInterval(interval);
  }, [amount]);

  const sizes = {
    sm: { icon: 'w-4 h-4', text: 'text-sm' },
    md: { icon: 'w-5 h-5', text: 'text-lg' },
    lg: { icon: 'w-6 h-6', text: 'text-xl' },
  };

  return (
    <div className={`flex items-center gap-2 ${sizes[size].text}`}>
      <CoinStack className={`${sizes[size].icon} text-amber-400`} />
      <span className="font-bold text-white">
        {displayAmount.toLocaleString()}
      </span>
    </div>
  );
}

// Multiple currencies (premium + standard)
function CurrencyDisplay({ standard, premium }: Props) {
  return (
    <div className="flex items-center gap-4">
      <div className="flex items-center gap-2">
        <CoinStack className="w-5 h-5 text-amber-400" />
        <span className="font-bold text-white">{standard.toLocaleString()}</span>
      </div>

      <div className="flex items-center gap-2">
        <GemIcon className="w-5 h-5 text-purple-400" />
        <span className="font-bold text-purple-300">{premium.toLocaleString()}</span>
      </div>
    </div>
  );
}
```

### Purchase Animation

```typescript
// Purchase success animation
function PurchaseSuccess({ item, quantity }: Props) {
  return (
    <motion.div
      className="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-50"
      initial={{ scale: 0, opacity: 0 }}
      animate={{ scale: 1, opacity: 1 }}
      exit={{ scale: 0, opacity: 0 }}
    >
      <div className="p-8 bg-slate-900/95 rounded-2xl border border-amber-500/50 text-center">
        <motion.img
          src={item.icon}
          className="w-24 h-24 mx-auto"
          initial={{ scale: 0, rotate: -180 }}
          animate={{ scale: 1, rotate: 0 }}
          transition={{ type: 'spring', damping: 10 }}
        />

        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.2 }}
          className="mt-4"
        >
          <div className="text-2xl font-bold text-white">Purchase Complete!</div>
          <div className="text-lg text-amber-400 mt-1">
            x{quantity} {item.name}
          </div>
        </motion.div>
      </div>
    </motion.div>
  );
}
```

### Best Practices

1. **Clear pricing** - Show all costs upfront
2. **Stock indicators** - Limited items create urgency
3. **Quick buy** - One-click for common items
4. **Sell back** - Let players recover some value
5. **Categories** - Navigate large inventories
6. **Animations** - Satisfying purchase feedback
7. **Affordability** - Gray out what players can't buy
