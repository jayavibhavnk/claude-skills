---
name: roguelike-deckbuilder
description: Roguelike deckbuilder games - poker hand scoring, Joker card systems, roguelike progression, and run-based gameplay mechanics.
metadata:
  priority: 8
  docs:
    - "https://balatro.fandom.com/"
  pathPatterns:
    - "**/roguelike/**"
    - "**/joker/**"
  bashPatterns:
    - '\broguelike\b'
    - '\bdeckbuilder\b'
  promptSignals:
    phrases:
      - "roguelike deckbuilder"
      - "balatro"
      - "poker hand"
    anyOf:
      - "roguelike"
      - "deckbuilder"
      - "joker"
---

## Roguelike Deckbuilder

### Poker Hand Scoring

```typescript
// Standard poker hands with scoring
interface HandResult {
  hand: HandType;
  score: number;
  mult: number;
  description: string;
}

enum HandType {
  ROYAL_FLUSH = 'Royal Flush',
  STRAIGHT_FLUSH = 'Straight Flush',
  FOUR_OF_A_KIND = 'Four of a Kind',
  FULL_HOUSE = 'Full House',
  FLUSH = 'Flush',
  STRAIGHT = 'Straight',
  THREE_OF_A_KIND = 'Three of a Kind',
  TWO_PAIR = 'Two Pair',
  PAIR = 'Pair',
  HIGH_CARD = 'High Card',
}

const HAND_SCORES: Record<HandType, { base: number; mult: number }> = {
  [HandType.ROYAL_FLUSH]: { base: 100, mult: 8 },
  [HandType.STRAIGHT_FLUSH]: { base: 80, mult: 7 },
  [HandType.FOUR_OF_A_KIND]: { base: 60, mult: 6 },
  [HandType.FULL_HOUSE]: { base: 40, mult: 4 },
  [HandType.FLUSH]: { base: 35, mult: 4 },
  [HandType.STRAIGHT]: { base: 30, mult: 4 },
  [HandType.THREE_OF_A_KIND]: { base: 30, mult: 3 },
  [HandType.TWO_PAIR]: { base: 20, mult: 2 },
  [HandType.PAIR]: { base: 10, mult: 2 },
  [HandType.HIGH_CARD]: { base: 5, mult: 1 },
};

function evaluateHand(cards: PlayingCard[]): HandResult {
  const sorted = [...cards].sort((a, b) => b.value - a.value);
  const isFlush = checkFlush(sorted);
  const isStraight = checkStraight(sorted);
  const counts = getValueCounts(sorted);

  let handType: HandType;
  let score = 0;
  let mult = 1;

  if (isFlush && isStraight && sorted[0].value === 14) {
    handType = HandType.ROYAL_FLUSH;
  } else if (isFlush && isStraight) {
    handType = HandType.STRAIGHT_FLUSH;
  } else if (hasNOfAKind(4, counts)) {
    handType = HandType.FOUR_OF_A_KIND;
  } else if (hasFullHouse(counts)) {
    handType = HandType.FULL_HOUSE;
  } else if (isFlush) {
    handType = HandType.FLUSH;
  } else if (isStraight) {
    handType = HandType.STRAIGHT;
  } else if (hasNOfAKind(3, counts)) {
    handType = HandType.THREE_OF_A_KIND;
  } else if (hasTwoPair(counts)) {
    handType = HandType.TWO_PAIR;
  } else if (hasPair(counts)) {
    handType = HandType.PAIR;
  } else {
    handType = HandType.HIGH_CARD;
  }

  const handData = HAND_SCORES[handType];
  score = handData.base;
  mult = handData.mult;

  return {
    hand: handType,
    score,
    mult,
    description: `${handType} - +${score} chips, x${mult}`,
  };
}

function checkFlush(cards: PlayingCard[]): boolean {
  return cards.every(c => c.suit === cards[0].suit);
}

function checkStraight(cards: PlayingCard[]): boolean {
  for (let i = 1; i < cards.length; i++) {
    if (cards[i].value !== cards[i - 1].value - 1) {
      return false;
    }
  }
  return true;
}
```

### Joker Card System

```typescript
// Joker slot system
interface Joker {
  id: string;
  name: string;
  description: string;
  rarity: Rarity;
  effect: JokerEffect;
  sellingPrice: number;
  edition?: Edition;
}

type Edition = 'polychrome' | 'foil' | 'holographic' | 'negative';
type Rarity = 'common' | 'uncommon' | 'rare' | 'legendary';

interface JokerEffect {
  type: 'mult' | 'chips' | 'bonus' | 'dollar' | 'hand' | 'score';
  value: number;
  trigger: 'always' | 'on_play' | 'on_score' | 'on_discarded';
  condition?: (context: ScoreContext) => boolean;
}

interface ScoreContext {
  hand: PlayingCard[];
  handType: HandType;
  baseScore: number;
  baseMult: number;
  jokers: Joker[];
  currentRound: number;
}

// Joker effect application
class JokerManager {
  private jokers: (Joker | null)[] = new Array(5).fill(null);

  addJoker(joker: Joker): boolean {
    const slot = this.jokers.findIndex(j => j === null);
    if (slot === -1) return false;
    this.jokers[slot] = joker;
    return true;
  }

  removeJoker(slot: number): Joker | null {
    const joker = this.jokers[slot];
    this.jokers[slot] = null;
    return joker;
  }

  applyScoreModifiers(context: ScoreContext): ScoreContext {
    let { baseScore, baseMult } = context;

    for (const joker of this.jokers) {
      if (!joker) continue;

      const { effect } = joker;
      if (!this.shouldTrigger(effect, context)) continue;

      switch (effect.type) {
        case 'mult':
          baseMult += effect.value;
          break;
        case 'chips':
          baseScore += effect.value;
          break;
        case 'bonus':
          baseScore += effect.value;
          break;
        case 'dollar':
          // Money effect
          break;
      }
    }

    return { ...context, baseScore, baseMult };
  }

  private shouldTrigger(effect: JokerEffect, context: ScoreContext): boolean {
    if (effect.condition && !effect.condition(context)) {
      return false;
    }
    return effect.trigger === 'always' || effect.trigger === 'on_score';
  }
}
```

### Playing Card Types

```typescript
interface PlayingCard {
  suit: Suit;
  value: number; // 2-14 (11=J, 12=Q, 13=K, 14=A)
  id: string;
  enhancement?: Enhancement;
  edition?: Edition;
  seal?: Seal;
}

enum Suit {
  SPADES = 'Spades',
  HEARTS = 'Hearts',
  DIAMONDS = 'Diamonds',
  CLUBS = 'Clubs',
}

enum Enhancement {
  LUCKY = 'Lucky',
  UNLUCKY = 'Unlucky',
  MULT = 'Mult',
  WILD = 'Wild',
  GLASS = 'Glass',
  STEEL = 'Steel',
  GOLD = 'Gold',
  BONUS_CHIPS = 'Bonus Chips',
}

enum Seal {
  RED = 'Red Seal',
  BLUE = 'Blue Seal',
  PURPLE = 'Purple Seal',
  GOLD = 'Gold Seal',
}

// Tarot card effects
interface TarotCard {
  id: string;
  name: string;
  type: 'major' | 'minor';
  effect: TarotEffect;
}

type TarotEffect =
  | { type: 'enhance_card'; enhancement: Enhancement }
  | { type: 'create_money'; amount: number }
  | { type: 'destroy_card' }
  | { type: 'swap_joker' };
```

### Run Progression

```typescript
// Roguelike run structure
interface Run {
  id: string;
  seed: number;
  startedAt: Date;
  currentStake: Stake;
  currentRound: number;
  currentAnte: number;
  player: PlayerState;
  unlocks: UnlockState;
}

interface Stake {
  level: number;
  name: string;
  scoringModifier: number;
  rewardMultiplier: number;
}

interface PlayerState {
  money: number;
  score: number;
  handSize: number;
  handLimit: number;
  discards: number;
  maxDiscards: number;
  deck: DeckModification;
  jokers: Joker[];
  vouchers: string[];
  tarotSlots: (TarotCard | null)[];
  planetSlots: (PlanetCard | null)[];
}

interface DeckModification {
  cardsAdded: string[];
  cardsRemoved: string[];
  cardsEnhanced: Map<string, Enhancement>;
}

// Shop system
interface Shop {
  items: ShopItem[];
  rerollCost: number;
  refreshCost: number;
}

interface ShopItem {
  type: 'joker' | 'tarot' | 'planet' | 'voucher' | 'booster';
  item: any;
  price: number;
  bought: boolean;
}

function purchaseItem(shop: Shop, index: number, player: PlayerState): boolean {
  const item = shop.items[index];
  if (item.bought || item.price > player.money) return false;

  player.money -= item.price;
  item.bought = true;

  switch (item.type) {
    case 'joker':
      player.jokers.push(item.item);
      break;
    case 'tarot':
      // Add to tarot slots or use immediately
      break;
  }

  return true;
}
```

### Booster Packs

```typescript
// Pack opening mechanics
interface BoosterPack {
  type: PackType;
  cards: string[];
  revealed: string[];
  selected: string[];
}

enum PackType {
  ARCANE = 'Arcane Pack',      // Tarot cards
  CELESTIAL = 'Celestial Pack', // Planet cards
  STANDARD = 'Standard Pack',  // Playing cards
  JOKER = 'Joker Pack',       // Jokers
  SPEED = 'Speed Pack',       // Quick rewards
}

function openPack(pack: BoosterPack): PackOpening {
  const revealed: string[] = [];
  const packSize = getPackSize(pack.type);

  for (let i = 0; i < packSize; i++) {
    const cardId = pack.cards.pop();
    if (cardId) revealed.push(cardId);
  }

  return {
    pack,
    revealed,
    selected: [],
  };
}

function selectCardFromPack(
  opening: PackOpening,
  cardId: string
): Card | null {
  if (!opening.revealed.includes(cardId)) return null;

  const index = opening.revealed.indexOf(cardId);
  opening.revealed.splice(index, 1);
  opening.selected.push(cardId);

  return getCardById(cardId);
}
```

### Blind System

```typescript
// Ante blinds with scaling
interface Blind {
  name: string;
  description: string;
  chipReward: number;
  moneyReward: number;
  boss?: boolean;
  specialEffect?: BlindEffect;
}

interface BlindEffect {
  type: 'no_discard' | 'random_discard' | 'hand_size' | 'joker_lock';
  value?: number;
}

function getBlindsForAnte(ante: number, stake: Stake): Blind[] {
  const baseChip = 300 + (ante * 200);
  const chipMult = 1 + (ante * 0.3) + (stake.scoringModifier / 100);

  return [
    {
      name: 'The Small Blind',
      description: 'A warmup...',
      chipReward: Math.floor(baseChip * 0.5 * chipMult),
      moneyReward: 3 + ante,
    },
    {
      name: 'The Big Blind',
      description: 'Getting serious...',
      chipReward: Math.floor(baseChip * chipMult),
      moneyReward: 4 + ante,
    },
    {
      name: 'The Boss Blind',
      description: 'Good luck!',
      chipReward: Math.floor(baseChip * 1.5 * chipMult),
      moneyReward: 5 + ante,
      boss: true,
      specialEffect: { type: 'no_discard' },
    },
  ];
}

function calculateScore(
  handResult: HandResult,
  jokers: Joker[],
  context: ScoreContext
): number {
  const { baseScore, baseMult } = jokers.reduce(
    (acc, joker) => jokerManager.applyScoreModifiers({ ...acc }),
    { baseScore: handResult.score, baseMult: handResult.mult }
  );

  return (baseScore + context.bonusChips) * baseMult;
}
```

### Persistence

```typescript
// Save/Load run state
interface SaveData {
  version: string;
  run: Run;
  player: PlayerState;
  shop: Shop;
  unlockedContent: string[];
  achievements: string[];
}

function saveRun(run: Run): string {
  const saveData: SaveData = {
    version: '1.0.0',
    run,
    player: run.player,
    shop: run.currentShop,
    unlockedContent: getUnlockedContent(),
    achievements: getAchievements(),
  };

  return btoa(JSON.stringify(saveData));
}

function loadRun(saveString: string): Run | null {
  try {
    const data: SaveData = JSON.parse(atob(saveString));

    if (data.version !== '1.0.0') {
      // Migration logic
    }

    return data.run;
  } catch {
    return null;
  }
}
```

### Best Practices

1. **Seeded runs** - Reproducible runs
2. **Gated content** - Unlock over time
3. **Balance scaling** - Difficulty by ante/stake
4. **Juice** - Animations, sounds, particles
5. **Session length** - ~30-60 min runs
6. **Variety** - Multiple viable strategies
7. **Accessibility** - Skip/auto options
