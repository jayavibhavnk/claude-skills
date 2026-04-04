---
name: game-quest-system
description: Game quest systems - quest types, objectives, tracking, progression, rewards, and quest log UI.
metadata:
  priority: 8
  docs: []
  pathPatterns:
    - "**/quest/**"
    - "**/mission/**"
  bashPatterns:
    - '\bquest\b'
    - '\bmission\b'
  promptSignals:
    phrases:
      - "quest system"
      - "mission"
      - "objective"
    anyOf:
      - "quest"
      - "mission"
      - "objective"
---

## Game Quest System

### Quest Definition

```typescript
interface Quest {
  id: string;
  name: string;
  description: string;
  type: QuestType;
  category: QuestCategory;
  objectives: Objective[];
  rewards: QuestRewards;
  prerequisites: Prerequisite[];
  repeatable: boolean;
  cooldown?: number; // ms for repeatable quests
  levelRequirement?: number;
  location?: string; // Where quest can be accepted
}

type QuestType =
  | 'main'      // Story progression
  | 'side'      // Optional content
  | 'daily'     // Reset daily
  | 'weekly'    // Reset weekly
  | 'event';    // Limited time

type QuestCategory =
  | 'combat'
  | 'exploration'
  | 'collection'
  | 'puzzle'
  | 'escort'
  | 'delivery';

interface Objective {
  id: string;
  description: string;
  type: ObjectiveType;
  target: string;
  current: number;
  required: number;
  optional?: boolean;
  failureCondition?: string;
}

type ObjectiveType =
  | 'kill'
  | 'collect'
  | 'visit'
  | 'talk'
  | 'escort'
  | 'deliver'
  | 'craft'
  | 'win'
  | 'survive';

interface QuestRewards {
  experience: number;
  gold: number;
  items?: ItemReward[];
  reputation?: ReputationReward[];
  unlock?: string[]; // Content unlocks
}

interface ItemReward {
  item: Item;
  quantity: number;
  chance?: number; // For random rewards
}

interface ReputationReward {
  faction: string;
  amount: number;
}

interface Prerequisite {
  type: 'quest' | 'item' | 'level' | 'reputation';
  target: string;
  value?: any;
}
```

### Quest Manager

```typescript
class QuestManager {
  private activeQuests: Map<string, QuestProgress> = new Map();
  private completedQuests: Set<string> = new Set();
  private listeners: QuestListener[] = [];

  startQuest(questId: string): boolean {
    const quest = this.getQuest(questId);
    if (!quest || this.isQuestActive(questId)) return false;

    const progress: QuestProgress = {
      questId,
      startedAt: Date.now(),
      objectives: quest.objectives.map(obj => ({
        ...obj,
        current: 0,
      })),
      status: 'active',
    };

    this.activeQuests.set(questId, progress);
    this.listeners.forEach(l => l.onQuestStart(quest));
    return true;
  }

  updateObjective(questId: string, objectiveId: string, amount: number): void {
    const progress = this.activeQuests.get(questId);
    if (!progress) return;

    const objective = progress.objectives.find(o => o.id === objectiveId);
    if (!objective) return;

    const prevCurrent = objective.current;
    objective.current = Math.min(objective.current + amount, objective.required);

    this.listeners.forEach(l =>
      l.onObjectiveProgress(objective, prevCurrent, objective.current)
    );

    // Check completion
    if (this.isQuestComplete(questId)) {
      this.completeQuest(questId);
    }
  }

  completeQuest(questId: string): void {
    const progress = this.activeQuests.get(questId);
    if (!progress) return;

    progress.status = 'completed';
    progress.completedAt = Date.now();

    const quest = this.getQuest(questId);
    this.completedQuests.add(questId);
    this.activeQuests.delete(questId);

    // Grant rewards
    this.grantRewards(quest.rewards);

    this.listeners.forEach(l => l.onQuestComplete(quest));
  }

  private isQuestComplete(questId: string): boolean {
    const progress = this.activeQuests.get(questId);
    if (!progress) return false;

    return progress.objectives.every(obj => obj.current >= obj.required);
  }
}
```

### Quest Log UI

```typescript
// Quest log screen
function QuestLog({ quests, onSelect, onAbandon }: Props) {
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('active');
  const [selectedQuest, setSelectedQuest] = useState<Quest | null>(null);

  const filteredQuests = quests.filter(q => {
    if (filter === 'active') return q.status === 'active';
    if (filter === 'completed') return q.status === 'completed';
    return true;
  });

  return (
    <div className="flex h-full bg-slate-900">
      {/* Quest list */}
      <div className="w-80 border-r border-slate-800 flex flex-col">
        {/* Filter tabs */}
        <div className="flex border-b border-slate-800">
          {['active', 'completed'].map(f => (
            <button
              key={f}
              onClick={() => setFilter(f)}
              className={`
                flex-1 py-3 text-sm font-medium capitalize
                ${filter === f
                  ? 'text-amber-400 border-b-2 border-amber-400'
                  : 'text-slate-400 hover:text-white'
                }
              `}
            >
              {f}
            </button>
          ))}
        </div>

        {/* Quest list */}
        <div className="flex-1 overflow-y-auto">
          {filteredQuests.map(quest => (
            <QuestListItem
              key={quest.id}
              quest={quest}
              isSelected={selectedQuest?.id === quest.id}
              onClick={() => setSelectedQuest(quest)}
            />
          ))}
        </div>
      </div>

      {/* Quest detail */}
      {selectedQuest && (
        <QuestDetail
          quest={selectedQuest}
          progress={getProgress(selectedQuest.id)}
          onAbandon={() => onAbandon(selectedQuest.id)}
        />
      )}
    </div>
  );
}

function QuestListItem({ quest, isSelected, onClick }: Props) {
  const progress = getQuestProgress(quest);

  return (
    <motion.div
      whileHover={{ backgroundColor: 'rgba(255,255,255,0.05)' }}
      onClick={onClick}
      className={`
        p-4 cursor-pointer border-b border-slate-800
        ${isSelected && 'bg-amber-900/20 border-l-2 border-l-amber-400'}
      `}
    >
      {/* Type badge */}
      <div className="flex items-center gap-2 mb-1">
        <Badge
          variant={quest.type === 'main' ? 'primary' : 'default'}
          size="sm"
        >
          {quest.type}
        </Badge>
        {quest.category && (
          <Badge variant="outline" size="sm">
            {quest.category}
          </Badge>
        )}
      </div>

      {/* Quest name */}
      <div className={`font-medium ${quest.status === 'active' ? 'text-white' : 'text-slate-500'}`}>
        {quest.name}
      </div>

      {/* Progress */}
      <div className="mt-2">
        <div className="h-1 bg-slate-700 rounded-full overflow-hidden">
          <motion.div
            className="h-full bg-amber-500"
            initial={{ width: 0 }}
            animate={{ width: `${progress}%` }}
          />
        </div>
        <div className="text-xs text-slate-500 mt-1">
          {Math.floor(progress)}% complete
        </div>
      </div>
    </motion.div>
  );
}
```

### Quest Detail Panel

```typescript
function QuestDetail({ quest, progress, onAbandon }: Props) {
  return (
    <div className="flex-1 p-6 overflow-y-auto">
      {/* Header */}
      <div className="flex items-start justify-between">
        <div>
          <h2 className="text-2xl font-bold text-amber-400">{quest.name}</h2>
          <p className="text-slate-400 mt-1">{quest.description}</p>
        </div>

        {quest.type === 'main' && (
          <Badge variant="primary" size="lg">Main Quest</Badge>
        )}
      </div>

      {/* Location */}
      {quest.location && (
        <div className="flex items-center gap-2 mt-4 text-slate-400">
          <MapPin className="w-4 h-4" />
          {quest.location}
        </div>
      )}

      {/* Objectives */}
      <div className="mt-8">
        <h3 className="text-lg font-bold text-white mb-4">Objectives</h3>

        <div className="space-y-3">
          {progress.objectives.map(obj => {
            const isComplete = obj.current >= obj.required;

            return (
              <motion.div
                key={obj.id}
                className={`
                  p-4 rounded-lg
                  ${isComplete && 'bg-green-900/20 border border-green-500/30'}
                  ${!isComplete && 'bg-slate-800/50'}
                `}
              >
                <div className="flex items-center gap-3">
                  {/* Status icon */}
                  {isComplete ? (
                    <CheckCircle className="w-5 h-5 text-green-400" />
                  ) : (
                    <Circle className="w-5 h-5 text-slate-500" />
                  )}

                  {/* Objective text */}
                  <div className="flex-1">
                    <span className={isComplete ? 'text-slate-400 line-through' : 'text-white'}>
                      {obj.description}
                    </span>

                    {/* Counter */}
                    {!isComplete && (
                      <span className="text-amber-400 ml-2">
                        ({obj.current} / {obj.required})
                      </span>
                    )}
                  </div>
                </div>

                {/* Progress bar */}
                {!isComplete && (
                  <div className="mt-2 ml-8 h-1 bg-slate-700 rounded-full overflow-hidden">
                    <motion.div
                      className="h-full bg-amber-500"
                      animate={{ width: `${(obj.current / obj.required) * 100}%` }}
                    />
                  </div>
                )}
              </motion.div>
            );
          })}
        </div>
      </div>

      {/* Rewards */}
      <div className="mt-8">
        <h3 className="text-lg font-bold text-white mb-4">Rewards</h3>

        <div className="grid grid-cols-2 gap-4">
          {/* XP */}
          <div className="flex items-center gap-3 p-4 bg-slate-800/50 rounded-lg">
            <Star className="w-8 h-8 text-purple-400" />
            <div>
              <div className="text-2xl font-bold text-purple-400">
                +{quest.rewards.experience.toLocaleString()}
              </div>
              <div className="text-xs text-slate-400">Experience</div>
            </div>
          </div>

          {/* Gold */}
          <div className="flex items-center gap-3 p-4 bg-slate-800/50 rounded-lg">
            <CoinStack className="w-8 h-8 text-amber-400" />
            <div>
              <div className="text-2xl font-bold text-amber-400">
                +{quest.rewards.gold.toLocaleString()}
              </div>
              <div className="text-xs text-slate-400">Gold</div>
            </div>
          </div>

          {/* Items */}
          {quest.rewards.items?.map((reward, i) => (
            <div key={i} className="flex items-center gap-3 p-4 bg-slate-800/50 rounded-lg">
              <img src={reward.item.icon} className="w-10 h-10" />
              <div>
                <div className="font-medium text-white">
                  {reward.item.name} x{reward.quantity}
                </div>
                <div className="text-xs text-slate-400">{reward.item.category}</div>
              </div>
            </div>
          ))}
        </div>
      </div>

      {/* Abandon button */}
      {quest.type !== 'main' && (
        <button
          onClick={onAbandon}
          className="mt-8 px-4 py-2 text-red-400 hover:bg-red-900/20 rounded-lg"
        >
          Abandon Quest
        </button>
      )}
    </div>
  );
}
```

### Quest Tracker (HUD)

```typescript
// Compact quest tracker for HUD
function QuestTracker({ quests }: Props) {
  const activeQuests = quests.filter(q => q.status === 'active').slice(0, 3);

  return (
    <div className="space-y-2">
      {activeQuests.map(quest => (
        <QuestTrackerItem key={quest.id} quest={quest} />
      ))}
    </div>
  );
}

function QuestTrackerItem({ quest }: Props) {
  const [expanded, setExpanded] = useState(false);

  return (
    <motion.div
      className="bg-slate-900/90 backdrop-blur rounded-lg overflow-hidden"
      layout
    >
      {/* Header - always visible */}
      <div
        className="flex items-center gap-3 p-3 cursor-pointer"
        onClick={() => setExpanded(!expanded)}
      >
        <ChevronDown
          className={`w-4 h-4 text-slate-400 transition-transform ${expanded && 'rotate-180'}`}
        />

        <div className="flex-1">
          <div className="text-sm font-medium text-white truncate">
            {quest.name}
          </div>
          <div className="text-xs text-slate-500">
            {quest.type} quest
          </div>
        </div>

        {/* Mini progress */}
        <div className="w-12 h-1 bg-slate-700 rounded-full overflow-hidden">
          <motion.div
            className="h-full bg-amber-500"
            animate={{ width: `${getQuestProgress(quest)}%` }}
          />
        </div>
      </div>

      {/* Expanded objectives */}
      {expanded && (
        <motion.div
          initial={{ height: 0 }}
          animate={{ height: 'auto' }}
          className="px-4 pb-3"
        >
          {quest.objectives.map(obj => (
            <div key={obj.id} className="flex items-center gap-2 py-1 text-sm">
              {obj.current >= obj.required ? (
                <CheckCircle className="w-4 h-4 text-green-400" />
              ) : (
                <Circle className="w-4 h-4 text-slate-500" />
              )}
              <span className={obj.current >= obj.required ? 'text-slate-500' : 'text-slate-300'}>
                {obj.description}
              </span>
              <span className="text-amber-400 ml-auto">
                {obj.current}/{obj.required}
              </span>
            </div>
          ))}
        </motion.div>
      )}
    </motion.div>
  );
}
```

### Best Practices

1. **Clear objectives** - Specific and measurable
2. **Varied quest types** - Combat, collection, exploration
3. **Rewarding progression** - Meaningful rewards
4. **Quest markers** - Guide players to objectives
5. **Track progress** - Show detailed progress
6. **Optional side quests** - For exploration
7. **No impossible quests** - Ensure completion is possible
