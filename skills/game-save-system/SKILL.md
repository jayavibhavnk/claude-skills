---
name: game-save-system
description: Game save systems - local storage, cloud saves, checkpoints, auto-save, and cross-platform persistence.
metadata:
  priority: 8
  docs:
    - "https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API"
  pathPatterns:
    - "**/save/**"
    - "**/storage/**"
  bashPatterns:
    - '\bsave\b'
    - '\bpersistence\b'
  promptSignals:
    phrases:
      - "game save"
      - "save system"
      - "checkpoint"
    anyOf:
      - "save"
      - "load"
      - "checkpoint"
---

## Game Save System

### Save Data Structure

```typescript
// Core save data interface
interface SaveData {
  version: string;
  timestamp: number;
  playerId: string;
  profile: PlayerProfile;
  gameState: GameState;
  progress: ProgressData;
  settings: GameSettings;
  metadata: SaveMetadata;
}

interface PlayerProfile {
  name: string;
  level: number;
  createdAt: number;
  lastPlayedAt: number;
  playTime: number; // seconds
}

interface GameState {
  currentScene: string;
  position: Vector3;
  inventory: InventoryData;
  quests: QuestState[];
  unlockedContent: string[];
  worldState: Record<string, any>;
}

interface SaveMetadata {
  saveSlot: number;
  thumbnail?: string; // Base64 screenshot
  playTime: number;
  completionPercent: number;
  difficulty: Difficulty;
}
```

### Local Storage Save

```typescript
// LocalStorage save manager
class LocalSaveManager {
  private readonly SAVE_KEY = 'game_save_';
  private readonly MAX_SLOTS = 5;

  save(slot: number, data: SaveData): Promise<boolean> {
    try {
      const key = `${this.SAVE_KEY}${slot}`;
      const serialized = JSON.stringify({
        ...data,
        timestamp: Date.now(),
      });
      localStorage.setItem(key, serialized);
      return Promise.resolve(true);
    } catch (error) {
      console.error('Save failed:', error);
      return Promise.resolve(false);
    }
  }

  load(slot: number): Promise<SaveData | null> {
    try {
      const key = `${this.SAVE_KEY}${slot}`;
      const data = localStorage.getItem(key);
      if (!data) return Promise.resolve(null);

      const save = JSON.parse(data) as SaveData;

      // Version migration if needed
      return Promise.resolve(this.migrateIfNeeded(save));
    } catch (error) {
      console.error('Load failed:', error);
      return Promise.resolve(null);
    }
  }

  delete(slot: number): Promise<void> {
    localStorage.removeItem(`${this.SAVE_KEY}${slot}`);
  }

  getSaveSlots(): Promise<SaveSlotInfo[]> {
    const slots: SaveSlotInfo[] = [];

    for (let i = 0; i < this.MAX_SLOTS; i++) {
      const save = await this.load(i);
      slots.push({
        slot: i,
        exists: !!save,
        data: save,
      });
    }

    return slots;
  }
}
```

### IndexedDB Save

```typescript
// IndexedDB for larger saves
class IndexedDBSaveManager {
  private db: IDBDatabase | null = null;
  private readonly DB_NAME = 'GameSaves';
  private readonly STORE_NAME = 'saves';
  private readonly VERSION = 1;

  async init(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.DB_NAME, this.VERSION);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;

        if (!db.objectStoreNames.contains(this.STORE_NAME)) {
          const store = db.createObjectStore('saves', { keyPath: 'slot' });
          store.createIndex('timestamp', 'timestamp', { unique: false });
        }
      };
    });
  }

  async save(slot: number, data: SaveData): Promise<void> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.STORE_NAME], 'readwrite');
      const store = transaction.objectStore(this.STORE_NAME);

      const saveData = {
        slot,
        ...data,
        timestamp: Date.now(),
      };

      const request = store.put(saveData);
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  async load(slot: number): Promise<SaveData | null> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.STORE_NAME], 'readonly');
      const store = transaction.objectStore(this.STORE_NAME);
      const request = store.get(slot);

      request.onsuccess = () => resolve(request.result || null);
      request.onerror = () => reject(request.error);
    });
  }

  async getAllSaves(): Promise<SaveData[]> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.STORE_NAME], 'readonly');
      const store = transaction.objectStore(this.STORE_NAME);
      const request = store.getAll();

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
}
```

### Auto-Save System

```typescript
// Auto-save with debouncing
class AutoSaveManager {
  private saveManager: SaveManager;
  private autoSaveInterval = 60000; // 1 minute
  private lastAutoSave = 0;
  private isDirty = false;
  private saveInProgress = false;

  constructor(saveManager: SaveManager) {
    this.saveManager = saveManager;

    // Listen for state changes
    gameState.onChange(() => {
      this.isDirty = true;
    });
  }

  start(): void {
    setInterval(() => this.tryAutoSave(), this.autoSaveInterval);

    // Save on scene change
    gameState.on('sceneChange', () => this.tryAutoSave());

    // Save on pause
    gameState.on('pause', () => this.tryAutoSave());
  }

  private async tryAutoSave(): Promise<void> {
    if (!this.isDirty || this.saveInProgress) return;
    if (Date.now() - this.lastAutoSave < this.autoSaveInterval) return;

    this.saveInProgress = true;

    try {
      const data = this.createSaveData();
      await this.saveManager.save(AUTO_SAVE_SLOT, data);
      this.lastAutoSave = Date.now();
      this.isDirty = false;
      console.log('Auto-saved');
    } catch (error) {
      console.error('Auto-save failed:', error);
    } finally {
      this.saveInProgress = false;
    }
  }
}
```

### Cloud Save (API-based)

```typescript
// Cloud save with REST API
class CloudSaveManager {
  private apiBase: string;
  private authToken: string;

  constructor(apiBase: string, authToken: string) {
    this.apiBase = apiBase;
    this.authToken = authToken;
  }

  async save(data: SaveData): Promise<CloudSaveResult> {
    const response = await fetch(`${this.apiBase}/saves`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.authToken}`,
      },
      body: JSON.stringify({
        ...data,
        timestamp: Date.now(),
      }),
    });

    if (!response.ok) {
      throw new Error(`Cloud save failed: ${response.status}`);
    }

    return response.json();
  }

  async load(saveId: string): Promise<SaveData> {
    const response = await fetch(`${this.apiBase}/saves/${saveId}`, {
      headers: {
        'Authorization': `Bearer ${this.authToken}`,
      },
    });

    if (!response.ok) {
      throw new Error(`Cloud load failed: ${response.status}`);
    }

    return response.json();
  }

  async listSaves(): Promise<CloudSaveList> {
    const response = await fetch(`${this.apiBase}/saves`, {
      headers: {
        'Authorization': `Bearer ${this.authToken}`,
      },
    });

    return response.json();
  }

  async sync(localSaves: SaveData[]): Promise<SyncResult> {
    const cloudSaves = await this.listSaves();

    const localOnly = localSaves.filter(
      local => !cloudSaves.find(c => c.timestamp === local.timestamp)
    );

    const conflicts: SyncConflict[] = [];

    for (const cloud of cloudSaves) {
      const local = localSaves.find(s => s.timestamp === cloud.timestamp);
      if (local && local.timestamp > cloud.timestamp) {
        // Local is newer, push to cloud
        await this.save(local);
      } else if (local && local.timestamp < cloud.timestamp) {
        // Cloud is newer, add to conflicts
        conflicts.push({ cloud, local });
      }
    }

    return { synced: localOnly.length, conflicts };
  }
}
```

### Version Migration

```typescript
// Save version migration
class SaveMigrator {
  private migrations: Map<string, (data: any) => any> = new Map();

  registerMigration(fromVersion: string, toVersion: string, migrate: (data: any) => any) {
    this.migrations.set(`${fromVersion}->${toVersion}`, migrate);
  }

  migrateIfNeeded(data: SaveData): SaveData {
    const currentVersion = '1.0.0';
    let current = data.version;

    while (current !== currentVersion) {
      const migrationKey = `${current}->${this.getNextVersion(current)}`;
      const migration = this.migrations.get(migrationKey);

      if (!migration) {
        console.warn(`No migration path from ${current} to ${currentVersion}`);
        break;
      }

      data = migration(data);
      current = this.getNextVersion(current);
    }

    return { ...data, version: currentVersion };
  }

  private getNextVersion(current: string): string {
    // Simple version bump
    const [major, minor] = current.split('.').map(Number);
    return `${major}.${minor + 1}.0`;
  }
}

// Example migration
migrator.registerMigration('1.0.0', '1.1.0', (data) => {
  // Add new field
  data.progress.newField = [];
  return data;
});

migrator.registerMigration('1.1.0', '1.2.0', (data) => {
  // Rename field
  data.playerName = data.player.name;
  delete data.player.name;
  return data;
});
```

### Save/Load UI

```typescript
// Save slot selector
function SaveLoadScreen({ mode, onSelect, onBack }: Props) {
  const [saves, setSaves] = useState<SaveSlotInfo[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadSaveSlots().then(saves => {
      setSaves(saves);
      setLoading(false);
    });
  }, []);

  return (
    <div className="grid grid-cols-3 gap-4 p-6">
      {[0, 1, 2].map(slot => {
        const save = saves.find(s => s.slot === slot);

        return (
          <motion.div
            key={slot}
            whileHover={{ scale: 1.02 }}
            whileTap={{ scale: 0.98 }}
            onClick={() => onSelect(slot)}
            className={`
              relative p-4 rounded-xl cursor-pointer
              ${save ? 'bg-slate-800' : 'bg-slate-800/50 border-2 border-dashed border-slate-700'}
            `}
          >
            {save ? (
              <>
                <img
                  src={save.data.metadata.thumbnail || '/default-save.jpg'}
                  className="w-full h-32 object-cover rounded-lg mb-3"
                />
                <div className="text-lg font-bold">{save.data.profile.name}</div>
                <div className="text-sm text-slate-400">
                  Level {save.data.profile.level}
                </div>
                <div className="text-xs text-slate-500">
                  {new Date(save.timestamp).toLocaleDateString()}
                </div>
              </>
            ) : (
              <div className="h-48 flex items-center justify-center text-slate-600">
                Empty Slot
              </div>
            )}

            {/* Actions */}
            <div className="absolute top-2 right-2 flex gap-1">
              {save && (
                <>
                  <IconButton icon={<LoadIcon />} size="sm" />
                  <IconButton icon={<DeleteIcon />} size="sm" variant="danger" />
                </>
              )}
            </div>
          </motion.div>
        );
      })}
    </div>
  );
}
```

### Best Practices

1. **Atomic saves** - Write to temp, then rename
2. **Versioning** - Always include version number
3. **Compression** - Save space for large games
4. **Corruption handling** - Validate on load
5. **Backup** - Keep last N saves
6. **Auto-save** - Never lose progress
7. **Cloud sync** - Handle conflicts gracefully
