---
name: turn-based-combat
description: Turn-based RPG combat - Pokemon-style battles, type systems, move mechanics, status effects, and battle UI.
metadata:
  priority: 8
  docs:
    - "https://pokeapi.co/"
  pathPatterns:
    - "**/battle/**"
    - "**/combat/**"
  bashPatterns:
    - '\bturn.based\b'
    - '\brpg\b'
  promptSignals:
    phrases:
      - "turn based combat"
      - "pokemon battle"
      - "rpg battle"
    anyOf:
      - "combat"
      - "battle"
      - "turn"
---

## Turn-Based Combat

### Creature/Mon System

```typescript
// Creature base stats
interface Creature {
  id: string;
  name: string;
  species: string;
  types: Type[];
  level: number;
  stats: Stats;
  currentHP: number;
  maxHP: number;
  moves: Move[];
  abilities: Ability[];
  status: StatusEffect | null;
  isWild: boolean;
}

interface Stats {
  hp: number;
  attack: number;
  defense: number;
  specialAttack: number;
  specialDefense: number;
  speed: number;
}

// Type effectiveness (simplified)
const TYPE_CHART: Record<Type, Record<Type, number>> = {
  [Type.NORMAL]: { [Type.ROCK]: 0.5, [Type.GHOST]: 0, [Type.STEEL]: 0.5 },
  [Type.FIRE]: { [Type.FIRE]: 0.5, [Type.WATER]: 0.5, [Type.GRASS]: 2, [Type.ICE]: 2, [Type.BUG]: 2, [Type.ROCK]: 0.5, [Type.DRAGON]: 0.5, [Type.STEEL]: 2 },
  [Type.WATER]: { [Type.FIRE]: 2, [Type.WATER]: 0.5, [Type.GRASS]: 0.5, [Type.GROUND]: 2, [Type.ROCK]: 2, [Type.DRAGON]: 0.5 },
  [Type.GRASS]: { [Type.FIRE]: 0.5, [Type.WATER]: 2, [Type.GRASS]: 0.5, [Type.POISON]: 0.5, [Type.GROUND]: 2, [Type.FLYING]: 0.5, [Type.BUG]: 0.5, [Type.ROCK]: 2, [Type.DRAGON]: 0.5, [Type.STEEL]: 0.5 },
  [Type.ELECTRIC]: { [Type.WATER]: 2, [Type.ELECTRIC]: 0.5, [Type.GRASS]: 0.5, [Type.GROUND]: 0, [Type.FLYING]: 2, [Type.DRAGON]: 0.5 },
  // ... other types
};

function getTypeMultiplier(attackType: Type, defenseTypes: Type[]): number {
  return defenseTypes.reduce((mult, defType) => {
    return mult * (TYPE_CHART[attackType][defType] ?? 1);
  }, 1);
}
```

### Move System

```typescript
// Battle moves
interface Move {
  id: string;
  name: string;
  type: Type;
  category: 'physical' | 'special' | 'status';
  power: number | null;
  accuracy: number;
  pp: number;
  maxPP: number;
  priority: number;
  target: TargetType;
  effect?: MoveEffect;
}

interface MoveEffect {
  type: 'damage' | 'heal' | 'status' | 'stat_change' | 'buff' | 'debuff';
  status?: StatusEffect;
  statChanges?: StatChange[];
  chance?: number; // Secondary effect chance
  recoil?: number; // Self damage
}

type TargetType = 'single' | 'double' | 'self' | 'all' | 'random';

interface StatChange {
  stat: 'attack' | 'defense' | 'spAttack' | 'spDefense' | 'speed' | 'accuracy' | 'evasion';
  stages: number; // -6 to +6
}

// Move execution
class MoveExecutor {
  execute(move: Move, user: Creature, targets: Creature[]): MoveResult {
    // Check accuracy
    if (!this.checkAccuracy(move, user, targets)) {
      return { hit: false, missed: true };
    }

    // Calculate damage for damaging moves
    if (move.category !== 'status' && move.power) {
      const results = targets.map(target => {
        const damage = this.calculateDamage(move, user, target);
        target.currentHP = Math.max(0, target.currentHP - damage);
        return { target, damage, effectiveness: this.getEffectiveness(move, target) };
      });

      return { hit: true, results };
    }

    // Status moves
    if (move.effect?.type === 'status') {
      return this.applyStatusEffect(move, user, targets);
    }

    return { hit: true, results: [] };
  }

  private calculateDamage(move: Move, attacker: Creature, defender: Creature): number {
    const level = attacker.level;
    const power = move.power!;
    const attack = move.category === 'physical' ? attacker.stats.attack : attacker.stats.specialAttack;
    const defense = move.category === 'physical' ? defender.stats.defense : defender.stats.specialDefense;

    const base = ((2 * level / 5 + 2) * power * attack / defense) / 50 + 2;
    const stab = attacker.types.includes(move.type) ? 1.5 : 1;
    const typeMult = getTypeMultiplier(move.type, defender.types);

    return Math.floor(base * stab * typeMult * (0.85 + Math.random() * 0.15));
  }
}
```

### Battle State

```typescript
// Turn-based battle state machine
type BattlePhase =
  | 'intro'
  | 'player_turn'
  | 'move_selection'
  | 'target_selection'
  | 'enemy_turn'
  | 'damage_calc'
  | 'apply_effects'
  | 'victory'
  | 'defeat';

interface BattleState {
  phase: BattlePhase;
  playerTeam: Creature[];
  enemyTeam: Creature[];
  playerActive: Creature;
  enemyActive: Creature;
  turnOrder: Creature[];
  moveHistory: MoveResult[];
  weather: Weather | null;
  field: FieldEffect;
}

class BattleManager {
  private state: BattleState;
  private eventEmitter: EventEmitter;

  async executeTurn(
    playerMove: Move,
    targetIndex?: number
  ): Promise<BattleResult> {
    // Determine turn order based on speed
    this.determineTurnOrder();

    const results: TurnResult[] = [];

    for (const creature of this.state.turnOrder) {
      if (creature.currentHP <= 0) continue;

      const result = await this.executeCreatureMove(creature, playerMove, targetIndex);
      results.push(result);

      // Check for battle end
      if (this.checkBattleEnd()) break;

      // Check for fainting
      if (result.fainted) {
        await this.handleFaint(creature);
      }
    }

    return { results, nextPhase: this.determineNextPhase() };
  }

  private determineTurnOrder(): void {
    this.state.turnOrder = [...this.state.playerTeam, ...this.state.enemyTeam]
      .filter(c => c.currentHP > 0)
      .sort((a, b) => {
        const speedA = this.getEffectiveSpeed(a);
        const speedB = this.getEffectiveSpeed(b);
        return speedB - speedA;
      });
  }

  private getEffectiveSpeed(creature: Creature): number {
    let speed = creature.stats.speed;
    if (creature.status === StatusEffect.PARALYSIS) speed *= 0.5;
    if (creature.status === StatusEffect.SLOW) speed *= 0.5;
    return speed;
  }
}
```

### Status Effects

```typescript
enum StatusEffect {
  POISON = 'poison',
  BURN = 'burn',
  PARALYSIS = 'paralysis',
  FREEZE = 'freeze',
  SLEEP = 'sleep',
  POISON_BADLY = 'badly_poison',
  CONFUSION = 'confusion',
  FLINCH = 'flinch',
  INFATUATION = 'infatuation',
}

interface StatusInfo {
  effect: StatusEffect;
  name: string;
  onApply: (creature: Creature) => void;
  onTurnStart: (creature: Creature) => void;
  onTurnEnd: (creature: Creature) => void;
  canMove: (creature: Creature) => boolean;
}

const STATUS_EFFECTS: Record<StatusEffect, StatusInfo> = {
  [StatusEffect.POISON]: {
    effect: StatusEffect.POISON,
    name: 'Poisoned',
    onApply: (c) => { /* apply poison animation */ },
    onTurnStart: (c) => {},
    onTurnEnd: (c) => {
      c.currentHP = Math.max(0, c.currentHP - Math.floor(c.maxHP / 8));
    },
    canMove: () => true,
  },
  [StatusEffect.BURN]: {
    effect: StatusEffect.BURN,
    name: 'Burned',
    onApply: (c) => { c.stats.attack = Math.floor(c.stats.attack * 0.5); },
    onTurnStart: (c) => {},
    onTurnEnd: (c) => {
      c.currentHP = Math.max(0, c.currentHP - Math.floor(c.maxHP / 16));
    },
    canMove: () => true,
  },
  [StatusEffect.PARALYSIS]: {
    effect: StatusEffect.PARALYSIS,
    name: 'Paralyzed',
    onApply: (c) => {},
    onTurnStart: (c) => {
      if (Math.random() < 0.25) return false; // Can't move
    },
    onTurnEnd: (c) => {},
    canMove: (c) => Math.random() > 0.25,
  },
  [StatusEffect.SLEEP]: {
    effect: StatusEffect.SLEEP,
    name: 'Asleep',
    onApply: (c) => { (c as any).sleepTurns = 1 + Math.floor(Math.random() * 3); },
    onTurnStart: (c) => {
      (c as any).sleepTurns--;
      if ((c as any).sleepTurns <= 0) {
        c.status = null;
        return true; // Woke up
      }
      return false; // Still asleep
    },
    onTurnEnd: (c) => {},
    canMove: (c) => false,
  },
};
```

### Battle UI

```typescript
// Battle screen layout
function BattleScreen({ state, onSelectMove, onSelectTarget }: Props) {
  return (
    <div className="relative w-full h-screen bg-gradient-to-b from-sky-400 to-sky-600">
      {/* Enemy creature */}
      <div className="absolute top-20 right-8">
        <CreatureSprite creature={state.enemyActive} isEnemy />
        <HealthBar current={state.enemyActive.currentHP} max={state.enemyActive.maxHP} />
        <StatusBadge status={state.enemyActive.status} />
      </div>

      {/* Player creature */}
      <div className="absolute bottom-40 left-8">
        <CreatureSprite creature={state.playerActive} isEnemy={false} />
        <HealthBar current={state.playerActive.currentHP} max={state.playerActive.maxHP} />
        <StatusBadge status={state.playerActive.status} />
      </div>

      {/* Move selection */}
      {state.phase === 'move_selection' && (
        <MoveSelector
          moves={state.playerActive.moves}
          onSelect={onSelectMove}
        />
      )}

      {/* Target selection for double */}
      {state.phase === 'target_selection' && (
        <TargetSelector
          targets={state.enemyTeam.filter(e => e.currentHP > 0)}
          onSelect={onSelectTarget}
        />
      )}

      {/* Battle log */}
      <BattleLog messages={state.recentMessages} />

      {/* Turn indicator */}
      <TurnIndicator turn={state.turnNumber} />
    </div>
  );
}

// Animated health bar
function HealthBar({ current, max }: { current: number; max: number }) {
  const percentage = (current / max) * 100;

  return (
    <div className="w-48 h-4 bg-slate-800 rounded-full overflow-hidden border-2 border-slate-600">
      <motion.div
        className={`
          h-full transition-colors duration-300
          ${percentage > 50 ? 'bg-green-500' : percentage > 20 ? 'bg-yellow-500' : 'bg-red-500'}
        `}
        initial={false}
        animate={{ width: `${percentage}%` }}
        transition={{ duration: 0.5, ease: 'easeOut' }}
      />
    </div>
  );
}
```

### Team Management

```typescript
// Team roster and switching
interface TeamManager {
  playerTeam: Creature[];
  enemyTeam: Creature[];

  canSwitch(): boolean;
  switchTo(index: number): Creature | null;
  getActiveCreatures(): { player: Creature; enemy: Creature };
  getAvailableSwitch(): Creature[];
}

function TeamScreen({ team, onSwitch, onBack }: Props) {
  return (
    <div className="grid grid-cols-2 gap-4 p-4">
      {team.map((creature, index) => (
        <motion.div
          key={creature.id}
          whileHover={{ scale: 1.05 }}
          whileTap={{ scale: 0.95 }}
          onClick={() => onSwitch(index)}
          className={`
            p-4 rounded-xl cursor-pointer
            ${creature.currentHP > 0 ? 'bg-slate-700' : 'bg-slate-800 opacity-50'}
          `}
        >
          <div className="flex items-center gap-4">
            <CreatureSprite creature={creature} size="small" />
            <div>
              <div className="font-bold">{creature.name}</div>
              <div className="text-sm text-slate-400">Lv. {creature.level}</div>
              <HealthBar current={creature.currentHP} max={creature.maxHP} small />
            </div>
          </div>
        </motion.div>
      ))}
    </div>
  );
}
```

### Best Practices

1. **Type balance** - Ensure no type is too strong
2. **Speed matters** - Turn order creates strategy
3. **Status variety** - Different strategic options
4. **Recovery moves** - Prevent stalemates
5. **Visual feedback** - Show damage numbers, effects
6. **Battle log** - Keep track of what happened
7. **Fair AI** - Appropriate difficulty
