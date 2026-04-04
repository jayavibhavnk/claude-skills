---
name: game-achievements
description: Game achievement systems - unlock conditions, progress tracking, trophies, leaderboards, and unlock celebrations.
metadata:
  priority: 7
  docs:
    - "https://developers.google.com/games/services/achievements"
  pathPatterns:
    - "**/achievement/**"
    - "**/trophy/**"
  bashPatterns:
    - '\bachievement\b'
    - '\btrophy\b'
  promptSignals:
    phrases:
      - "achievement system"
      - "trophy"
      - "unlock"
    anyOf:
      - "achievement"
      - "trophy"
      - "unlock"
---

## Game Achievements

### Achievement Definition

```typescript
interface Achievement {
  id: string;
  name: string;
  description: string;
  icon: string;
  category: AchievementCategory;
  difficulty: Difficulty;
  points: number;
  isSecret: boolean;
  unlockCondition: UnlockCondition;
  tier?: 'bronze' | 'silver' | 'gold' | 'platinum';
}

type AchievementCategory =
  | 'progress'
  | 'combat'
  | 'exploration'
  | 'collection'
  | 'skill'
  | 'story'
  | 'special';

type Difficulty = 'easy' | 'medium' | 'hard' | 'extreme';

interface UnlockCondition {
  type: ConditionType;
  target: number;
  current?: number;
  metadata?: Record<string, any>;
}

type ConditionType =
  | 'stat'
  | 'counter'
  | 'flag'
  | 'item'
  | 'location'
  | 'action'
  | 'time'
  | 'combo';
```

### Achievement Manager

```typescript
class AchievementManager {
  private achievements: Map<string, Achievement> = new Map();
  private unlocked: Set<string> = new Set();
  private progress: Map<string, number> = new Map();
  private listeners: AchievementListener[] = [];

  constructor(private storage: AchievementStorage) {
    this.loadProgress();
  }

  register(achievement: Achievement): void {
    this.achievements.set(achievement.id, achievement);
  }

  async unlock(achievementId: string): Promise<UnlockResult | null> {
    const achievement = this.achievements.get(achievementId);
    if (!achievement) return null;

    if (this.unlocked.has(achievementId)) {
      return null; // Already unlocked
    }

    this.unlocked.add(achievementId);
    await this.storage.saveUnlocked(achievementId);

    const result: UnlockResult = {
      achievement,
      isNew: true,
      allUnlocked: this.unlocked.size === this.achievements.size,
    };

    // Notify listeners
    for (const listener of this.listeners) {
      listener.onUnlock(result);
    }

    return result;
  }

  updateProgress(achievementId: string, value: number): void {
    const achievement = this.achievements.get(achievementId);
    if (!achievement) return;

    const current = this.progress.get(achievementId) || 0;
    const newProgress = Math.min(current + value, achievement.unlockCondition.target);

    this.progress.set(achievementId, newProgress);
    this.storage.saveProgress(achievementId, newProgress);

    // Check for unlock
    if (newProgress >= achievement.unlockCondition.target) {
      this.unlock(achievementId);
    }
  }

  // Convenience methods for common conditions
  incrementStat(stat: string, amount: number = 1): void {
    for (const achievement of this.achievements.values()) {
      if (achievement.unlockCondition.type === 'stat' &&
          achievement.unlockCondition.metadata?.stat === stat) {
        this.updateProgress(achievement.id, amount);
      }
    }
  }

  checkFlag(flag: string): void {
    for (const achievement of this.achievements.values()) {
      if (achievement.unlockCondition.type === 'flag' &&
          achievement.unlockCondition.metadata?.flag === flag) {
        this.unlock(achievement.id);
      }
    }
  }
}
```

### Achievement UI

```typescript
// Achievement unlock popup
function AchievementUnlock({ achievement, onDismiss }: Props) {
  return (
    <motion.div
      className="fixed top-8 left-1/2 -translate-x-1/2 z-50"
      initial={{ opacity: 0, y: -100, scale: 0.8 }}
      animate={{ opacity: 1, y: 0, scale: 1 }}
      exit={{ opacity: 0, y: -100, scale: 0.8 }}
    >
      <div className={`
        flex items-center gap-4 px-6 py-4 rounded-xl
        bg-gradient-to-r from-amber-900/95 to-orange-900/95
        border-2 ${getTierBorder(achievement.tier)}
        shadow-2xl shadow-amber-500/20
      `}>
        {/* Icon with glow */}
        <div className="relative">
          <motion.div
            className={`
              absolute inset-0 rounded-full bg-amber-400 blur-md
              ${achievement.tier === 'platinum' && 'bg-gradient-to-r from-amber-400 to-purple-400'}
            `}
            animate={{ scale: [1, 1.2, 1], opacity: [0.5, 0.8, 0.5] }}
            transition={{ duration: 2, repeat: Infinity }}
          />
          <img
            src={achievement.icon}
            className="relative w-16 h-16 rounded-full bg-slate-800 p-2"
          />
        </div>

        {/* Info */}
        <div>
          <div className="text-xs text-amber-400 font-medium uppercase tracking-wider">
            Achievement Unlocked!
          </div>
          <div className="text-lg font-bold text-white">
            {achievement.isSecret ? '???' : achievement.name}
          </div>
          {!achievement.isSecret && (
            <div className="text-sm text-slate-400">
              {achievement.description}
            </div>
          )}
        </div>

        {/* Points */}
        <div className="text-right">
          <div className="text-2xl font-bold text-amber-400">
            +{achievement.points}
          </div>
          <div className="text-xs text-slate-500">Gamerscore</div>
        </div>
      </div>
    </motion.div>
  );
}

// Achievement gallery
function AchievementGallery({ achievements, unlocked }: Props) {
  const categories = groupBy(achievements, 'category');

  return (
    <div className="space-y-8 p-6">
      {Object.entries(categories).map(([category, items]) => (
        <div key={category}>
          <h3 className="text-lg font-bold text-amber-400 mb-4 uppercase">
            {formatCategory(category)}
          </h3>

          <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
            {items.map(achievement => (
              <AchievementCard
                key={achievement.id}
                achievement={achievement}
                isUnlocked={unlocked.has(achievement.id)}
              />
            ))}
          </div>
        </div>
      ))}
    </div>
  );
}

function AchievementCard({ achievement, isUnlocked }: Props) {
  return (
    <motion.div
      whileHover={{ scale: 1.05 }}
      className={`
        relative p-4 rounded-xl cursor-pointer
        ${isUnlocked ? 'bg-slate-800' : 'bg-slate-800/50 opacity-60'}
      `}
    >
      {/* Lock overlay */}
      {!isUnlocked && (
        <div className="absolute inset-0 flex items-center justify-center bg-black/40 rounded-xl z-10">
          <Lock className="w-8 h-8 text-slate-600" />
        </div>
      )}

      <img
        src={achievement.icon}
        className={`
          w-16 h-16 mx-auto rounded-full mb-3
          ${isUnlocked ? '' : 'grayscale'}
        `}
      />

      <div className={`text-center font-medium ${isUnlocked ? 'text-white' : 'text-slate-500'}`}>
        {isUnlocked ? achievement.name : achievement.isSecret ? '???' : achievement.name}
      </div>

      <div className="text-center text-xs text-slate-500 mt-1">
        {isUnlocked ? '+' + achievement.points : formatProgress(achievement)}
      </div>
    </motion.div>
  );
}
```

### Progress Tracking

```typescript
// Progress bar for achievements
function AchievementProgress({ achievement, current, target }: Props) {
  const percentage = (current / target) * 100;

  return (
    <div className="w-full">
      <div className="flex justify-between text-sm mb-1">
        <span className="text-slate-400">{current} / {target}</span>
        <span className="text-amber-400">{Math.floor(percentage)}%</span>
      </div>

      <div className="h-2 bg-slate-800 rounded-full overflow-hidden">
        <motion.div
          className="h-full bg-gradient-to-r from-amber-600 to-amber-400"
          initial={{ width: 0 }}
          animate={{ width: `${percentage}%` }}
          transition={{ duration: 0.5 }}
        />
      </div>
    </div>
  );
}
```

### Leaderboard

```typescript
// Leaderboard system
interface LeaderboardEntry {
  rank: number;
  playerId: string;
  playerName: string;
  score: number;
  metadata?: {
    avatar?: string;
    level?: number;
    timestamp?: number;
  };
}

class LeaderboardManager {
  async submitScore(score: number, metadata?: any): Promise<RankResult> {
    const response = await fetch('/api/leaderboard', {
      method: 'POST',
      body: JSON.stringify({ score, metadata }),
    });

    return response.json();
  }

  async getTop(limit: number = 10): Promise<LeaderboardEntry[]> {
    const response = await fetch(`/api/leaderboard/top?limit=${limit}`);
    return response.json();
  }

  async getPlayerRank(playerId: string): Promise<number> {
    const response = await fetch(`/api/leaderboard/rank/${playerId}`);
    const data = await response.json();
    return data.rank;
  }
}

// Leaderboard UI
function LeaderboardUI({ entries, currentPlayerId }: Props) {
  return (
    <div className="bg-slate-900/80 rounded-xl overflow-hidden">
      <div className="p-4 bg-slate-800">
        <h2 className="text-xl font-bold text-amber-400">Leaderboard</h2>
      </div>

      <div className="divide-y divide-slate-800">
        {entries.map((entry, index) => (
          <motion.div
            key={entry.playerId}
            initial={{ opacity: 0, x: -20 }}
            animate={{ opacity: 1, x: 0 }}
            transition={{ delay: index * 0.05 }}
            className={`
              flex items-center gap-4 p-4
              ${entry.playerId === currentPlayerId && 'bg-amber-900/20'}
            `}
          >
            {/* Rank */}
            <div className={`
              w-10 h-10 rounded-full flex items-center justify-center font-bold
              ${entry.rank === 1 && 'bg-amber-500 text-black'}
              ${entry.rank === 2 && 'bg-slate-400 text-black'}
              ${entry.rank === 3 && 'bg-amber-700 text-white'}
              ${entry.rank > 3 && 'bg-slate-800 text-slate-400'}
            `}>
              {entry.rank}
            </div>

            {/* Player info */}
            <img
              src={entry.metadata?.avatar || '/default-avatar.png'}
              className="w-10 h-10 rounded-full"
            />
            <div className="flex-1">
              <div className="font-medium">{entry.playerName}</div>
              {entry.metadata?.level && (
                <div className="text-xs text-slate-500">Level {entry.metadata.level}</div>
              )}
            </div>

            {/* Score */}
            <div className="text-xl font-bold text-amber-400">
              {entry.score.toLocaleString()}
            </div>
          </motion.div>
        ))}
      </div>
    </div>
  );
}
```

### Best Practices

1. **Meaningful achievements** - Worth pursuing
2. **Varied difficulty** - Easy to extreme
3. **Secret achievements** - Add mystery
4. **Progress visibility** - Show partial progress
5. **Celebrate unlocks** - Sound, animation, popup
6. **Tiers** - Bronze, silver, gold, platinum
7. **Avoid frustration** - No extreme grind
