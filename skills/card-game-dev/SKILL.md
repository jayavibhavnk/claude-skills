---
name: card-game-dev
description: Card game development - deck building, hand management, card effects, turn systems, and game state for trading card games.
metadata:
  priority: 8
  docs:
    - "https://www.json.org/"
  pathPatterns:
    - "**/cards/**"
    - "**/deck/**"
  bashPatterns:
    - '\bcard\b'
    - '\bdeck\b'
  promptSignals:
    phrases:
      - "card game"
      - "deck building"
      - "trading card"
    anyOf:
      - "card"
      - "deck"
      - "hand"
---

## Card Game Development

### Card Types

```typescript
// Core card definitions
interface Card {
  id: string;
  name: string;
  cost: number;
  type: CardType;
  rarity: Rarity;
  effects: Effect[];
  description: string;
}

type CardType = 'creature' | 'spell' | 'artifact' | 'enchantment';
type Rarity = 'common' | 'uncommon' | 'rare' | 'legendary';

interface CreatureCard extends Card {
  type: 'creature';
  attack: number;
  defense: number;
  abilities: Ability[];
}

interface SpellCard extends Card {
  type: 'spell';
  target: TargetType;
  school: MagicSchool;
}

type TargetType = 'any' | 'self' | 'enemy' | 'creature' | 'player';
type MagicSchool = 'fire' | 'water' | 'earth' | 'air' | 'shadow' | 'light';
```

### Effect System

```typescript
// Effect types and execution
type EffectType =
  | 'damage'
  | 'heal'
  | 'draw'
  | 'buff'
  | 'debuff'
  | 'summon'
  | 'destroy'
  | 'transform'
  | 'token';

interface Effect {
  type: EffectType;
  value?: number;
  target?: TargetType;
  condition?: Condition;
  duration?: number; // For temporary effects
}

interface EffectResult {
  success: boolean;
  messages: string[];
  stateChanges: StateChange[];
}

class EffectExecutor {
  execute(effect: Effect, context: EffectContext): EffectResult {
    switch (effect.type) {
      case 'damage':
        return this.dealDamage(effect, context);
      case 'draw':
        return this.drawCards(effect, context);
      case 'buff':
        return this.applyBuff(effect, context);
      // ... other effects
    }
  }

  private dealDamage(effect: Effect, context: EffectContext): EffectResult {
    const target = this.resolveTarget(effect.target, context);
    if (!target) {
      return { success: false, messages: ['No valid target'], stateChanges: [] };
    }

    const damage = this.calculateDamage(effect.value!, context);
    target.currentHealth -= damage;

    return {
      success: true,
      messages: [`Dealt ${damage} damage to ${target.name}`],
      stateChanges: [{ type: 'damage', target: target.id, value: damage }],
    };
  }

  private calculateDamage(base: number, context: EffectContext): number {
    // Apply modifiers
    let damage = base;

    // Attack modifiers
    if (context.source.attackBonus) {
      damage += context.source.attackBonus;
    }

    // Elemental modifiers
    if (context.elementalAdvantage) {
      damage *= 1.5;
    }

    return Math.floor(damage);
  }
}
```

### Deck Management

```typescript
// Deck with drawing and shuffling
class Deck {
  private cards: Card[] = [];
  private discard: Card[] = [];
  private name: string;

  constructor(name: string, cards: Card[]) {
    this.name = name;
    this.cards = this.shuffle([...cards]);
  }

  private shuffle(cards: Card[]): Card[] {
    for (let i = cards.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [cards[i], cards[j]] = [cards[j], cards[i]];
    }
    return cards;
  }

  draw(): Card | null {
    if (this.cards.length === 0) {
      this reshuffleDiscard();
    }

    const card = this.cards.pop();
    if (card) {
      return card;
    }
    return null;
  }

  private reshuffleDiscard(): void {
    if (this.discard.length === 0) return;

    this.cards = this.shuffle([...this.discard]);
    this.discard = [];
  }

  discardCard(card: Card): void {
    this.discard.push(card);
  }

  // Deck statistics
  getCardCount(): number { return this.cards.length; }
  getDiscardCount(): number { return this.discard.length; }
  getCardsByType(type: CardType): Card[] {
    return this.cards.filter(c => c.type === type);
  }
}
```

### Hand Management

```typescript
// Hand with card play logic
class Hand {
  private cards: Card[] = [];
  private maxSize: number;

  constructor(maxSize: number = 10) {
    this.maxSize = maxSize;
  }

  addCard(card: Card): boolean {
    if (this.cards.length >= this.maxSize) {
      return false; // Hand full
    }
    this.cards.push(card);
    return true;
  }

  removeCard(cardId: string): Card | null {
    const index = this.cards.findIndex(c => c.id === cardId);
    if (index === -1) return null;
    return this.cards.splice(index, 1)[0];
  }

  getPlayableCards(mana: number): Card[] {
    return this.cards
      .filter(c => c.cost <= mana)
      .sort((a, b) => b.cost - a.cost);
  }

  getCardsByType(type: CardType): Card[] {
    return this.cards.filter(c => c.type === type);
  }

  // Hand evaluation for AI
  evaluateHand(): HandEvaluation {
    return {
      totalValue: this.cards.reduce((sum, c) => sum + this.cardValue(c), 0),
      curve: this.getManaCurve(),
      synergy: this.calculateSynergy(),
      removal: this.cards.filter(c => c.type === 'spell').length,
    };
  }

  private cardValue(card: Card): number {
    // Simplified valuation
    let value = card.cost * 2;
    if (card.type === 'creature') {
      const creature = card as CreatureCard;
      value += creature.attack + creature.defense;
    }
    return value;
  }

  private getManaCurve(): Map<number, number> {
    const curve = new Map<number, number>();
    for (const card of this.cards) {
      curve.set(card.cost, (curve.get(card.cost) || 0) + 1);
    }
    return curve;
  }
}
```

### Turn System

```typescript
// Turn phases and actions
enum Phase {
  DRAW = 'draw',
  STANDBY = 'standby',
  MAIN = 'main',
  BATTLE = 'battle',
  END = 'end',
}

interface TurnState {
  currentPhase: Phase;
  turnNumber: number;
  currentPlayer: Player;
  actionsThisTurn: number;
  manaThisTurn: number;
  maxMana: number;
}

class TurnManager {
  private state: TurnState;
  private onPhaseChange: (phase: Phase) => void;

  constructor(onPhaseChange: (phase: Phase) => void) {
    this.onPhaseChange = onPhaseChange;
  }

  startTurn(player: Player): void {
    this.state = {
      currentPhase: Phase.DRAW,
      turnNumber: player.turnCount,
      currentPlayer: player,
      actionsThisTurn: 0,
      manaThisTurn: Math.min(player.mana + 1, 10), // Cap at 10
      maxMana: Math.min(player.mana + 1, 10),
    };

    this.advancePhase();
  }

  advancePhase(): void {
    const phases = [Phase.DRAW, Phase.STANDBY, Phase.MAIN, Phase.BATTLE, Phase.END];
    const currentIndex = phases.indexOf(this.state.currentPhase);

    if (currentIndex === phases.length - 1) {
      this.endTurn();
    } else {
      this.state.currentPhase = phases[currentIndex + 1];
      this.onPhaseChange(this.state.currentPhase);
      this.executePhase();
    }
  }

  private executePhase(): void {
    switch (this.state.currentPhase) {
      case Phase.DRAW:
        this.executeDrawPhase();
        break;
      case Phase.MAIN:
        // Wait for player input
        break;
      case Phase.BATTLE:
        this.executeBattlePhase();
        break;
      case Phase.END:
        this.executeEndPhase();
        break;
    }
  }

  private executeDrawPhase(): void {
    const card = this.state.currentPlayer.deck.draw();
    if (card) {
      this.state.currentPlayer.hand.addCard(card);
    }
    this.advancePhase();
  }

  canTakeAction(): boolean {
    return this.state.currentPhase === Phase.MAIN;
  }
}
```

### Board State

```typescript
// Game board for creatures
interface Board {
  playerField: (CreatureCard | null)[];
  enemyField: (CreatureCard | null)[];
  maxFieldSize: number;
}

class BoardManager {
  private board: Board;

  constructor(maxFieldSize: number = 5) {
    this.board = {
      playerField: new Array(maxFieldSize).fill(null),
      enemyField: new Array(maxFieldSize).fill(null),
      maxFieldSize,
    };
  }

  summonCreature(player: Player, card: CreatureCard, position: number): boolean {
    const field = player.isEnemy ? this.board.enemyField : this.board.playerField;

    if (field[position] !== null) {
      return false; // Position occupied
    }

    field[position] = card;
    return true;
  }

  canAttack(attacker: CreatureCard): boolean {
    return !attacker.hasAttacked && attacker.canAttack;
  }

  attack(attacker: CreatureCard, defender: CreatureCard | null): AttackResult {
    if (!this.canAttack(attacker)) {
      return { success: false, message: 'Cannot attack' };
    }

    if (defender) {
      // Creature combat
      defender.currentHealth -= attacker.attack;
      attacker.currentHealth -= defender.attack;

      return {
        success: true,
        message: `${attacker.name} deals ${attacker.attack} damage to ${defender.name}`,
        damageDealt: attacker.attack,
        damageTaken: defender.attack,
      };
    } else {
      // Direct damage to player
      return {
        success: true,
        message: `${attacker.name} deals ${attacker.attack} damage to enemy player`,
        damageDealt: attacker.attack,
      };
    }
  }
}
```

### Game State Machine

```typescript
type GamePhase = 'setup' | 'playing' | 'paused' | 'ended';

interface GameState {
  phase: GamePhase;
  players: [Player, Player];
  currentPlayerIndex: number;
  turn: TurnState;
  board: Board;
  history: GameEvent[];
  winner: Player | null;
}

class CardGame {
  private state: GameState;
  private eventEmitter: EventEmitter;

  startGame(player1: Player, player2: Player): void {
    // Initialize players
    player1.deck = new Deck('Player 1', this.createStartingDeck());
    player2.deck = new Deck('Player 2', this.createStartingDeck());

    // Draw starting hands
    for (let i = 0; i < 5; i++) {
      player1.hand.addCard(player1.deck.draw()!);
      player2.hand.addCard(player2.deck.draw()!);
    }

    this.state = {
      phase: 'playing',
      players: [player1, player2],
      currentPlayerIndex: Math.random() > 0.5 ? 1 : 0,
      turn: null!,
      board: { playerField: [], enemyField: [] },
      history: [],
      winner: null,
    };

    this.startTurn();
  }

  private startTurn(): void {
    const currentPlayer = this.state.players[this.state.currentPlayerIndex];
    this.state.turn = {
      currentPhase: Phase.DRAW,
      turnNumber: currentPlayer.turnCount++,
      currentPlayer,
      actionsThisTurn: 0,
      manaThisTurn: currentPlayer.mana,
      maxMana: currentPlayer.mana,
    };

    currentPlayer.mana = Math.min(currentPlayer.mana + 1, 10);
  }
}
```

### Best Practices

1. **Immutability** - Card effects should be predictable
2. **Serialization** - Save/load game state easily
3. **Undo/Replay** - Track all game events
4. **Animation timing** - Sync effects with visuals
5. **Targeting UI** - Clear valid targets
6. **Mana curves** - Balance deck costs
7. **Hand limits** - Prevent overflow
