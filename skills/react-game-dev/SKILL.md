---
name: react-game-dev
description: React game development - game loops, state machines, collision detection, sprites, and performance optimization for browser games.
metadata:
  priority: 8
  docs:
    - "https://react.dev/"
  pathPatterns:
    - "**/game/**"
    - "**/components/**/Game*.tsx"
  bashPatterns:
    - '\bgame\b'
    - '\breact-game\b'
  promptSignals:
    phrases:
      - "react game"
      - "game development"
      - "game loop"
    anyOf:
      - "game"
      - "sprite"
      - "canvas"
---

## React Game Development

### Game Loop Pattern

```typescript
// hooks/useGameLoop.ts
import { useEffect, useRef, useCallback } from 'react';

interface GameState {
  position: { x: number; y: number };
  velocity: { x: number; y: number };
}

export function useGameLoop(
  callback: (deltaTime: number) => void,
  isRunning: boolean = true
) {
  const requestRef = useRef<number>();
  const previousTimeRef = useRef<number>();

  const animate = useCallback((time: number) => {
    if (previousTimeRef.current !== undefined) {
      const deltaTime = (time - previousTimeRef.current) / 1000;
      callback(deltaTime);
    }
    previousTimeRef.current = time;
    requestRef.current = requestAnimationFrame(animate);
  }, [callback]);

  useEffect(() => {
    if (isRunning) {
      requestRef.current = requestAnimationFrame(animate);
    }
    return () => {
      if (requestRef.current) {
        cancelAnimationFrame(requestRef.current);
      }
    };
  }, [isRunning, animate]);
}

// Usage in component
function Game() {
  const [gameState, setGameState] = useState<GameState>({
    position: { x: 100, y: 100 },
    velocity: { x: 0, y: 0 },
  });

  const handleUpdate = useCallback((deltaTime: number) => {
    setGameState(prev => ({
      ...prev,
      position: {
        x: prev.position.x + prev.velocity.x * deltaTime,
        y: prev.position.y + prev.velocity.y * deltaTime,
      },
    }));
  }, []);

  useGameLoop(handleUpdate, isPlaying);

  return <GameCanvas state={gameState} />;
}
```

### State Machine Pattern

```typescript
// types/game.ts
type GameState = 'menu' | 'playing' | 'paused' | 'gameover';

interface StateContext {
  score: number;
  lives: number;
  level: number;
}

// machine/gameMachine.ts
import { create } from 'zustand';

interface GameStore extends StateContext {
  state: GameState;
  transition: (newState: GameState) => void;
  addScore: (points: number) => void;
  loseLife: () => void;
  reset: () => void;
}

const initialState: StateContext = {
  score: 0,
  lives: 3,
  level: 1,
};

export const useGameStore = create<GameStore>((set) => ({
  ...initialState,
  state: 'menu',

  transition: (newState) => set({ state: newState }),

  addScore: (points) =>
    set((s) => ({
      score: s.score + points,
      level: Math.floor((s.score + points) / 1000) + 1,
    })),

  loseLife: () =>
    set((s) => {
      const newLives = s.lives - 1;
      return { lives: newLives, state: newLives <= 0 ? 'gameover' : s.state };
    }),

  reset: () => set({ ...initialState, state: 'playing' }),
}));

// Usage in component
function Game() {
  const { state, transition, reset } = useGameStore();

  if (state === 'menu') return <MenuScreen onStart={() => transition('playing')} />;
  if (state === 'paused') return <PauseScreen onResume={() => transition('playing')} />;
  if (state === 'gameover') return <GameOverScreen onRestart={reset} />;
  return <GameplayScreen />;
}
```

### Collision Detection

```typescript
// utils/collision.ts
interface Rectangle {
  x: number;
  y: number;
  width: number;
  height: number;
}

interface Circle {
  x: number;
  y: number;
  radius: number;
}

export function rectIntersect(a: Rectangle, b: Rectangle): boolean {
  return (
    a.x < b.x + b.width &&
    a.x + a.width > b.x &&
    a.y < b.y + b.height &&
    a.y + a.height > b.y
  );
}

export function circleIntersect(a: Circle, b: Circle): boolean {
  const dx = a.x - b.x;
  const dy = a.y - b.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  return distance < a.radius + b.radius;
}

export function pointInRect(point: { x: number; y: number }, rect: Rectangle): boolean {
  return (
    point.x >= rect.x &&
    point.x <= rect.x + rect.width &&
    point.y >= rect.y &&
    point.y <= rect.y + rect.height
  );
}

// Spatial partitioning for many objects
export class QuadTree {
  private objects: Rectangle[] = [];
  private bounds: Rectangle;
  private capacity: number;
  private divided: boolean = false;

  constructor(bounds: Rectangle, capacity: number = 4) {
    this.bounds = bounds;
    this.capacity = capacity;
  }

  insert(obj: Rectangle): boolean {
    if (!rectIntersect(obj, this.bounds)) return false;

    if (this.objects.length < this.capacity) {
      this.objects.push(obj);
      return true;
    }

    if (!this.divided) this.subdivide();

    return (
      this.northwest!.insert(obj) ||
      this.northeast!.insert(obj) ||
      this.southwest!.insert(obj) ||
      this.southeast!.insert(obj)
    );
  }
}
```

### Sprite Animation

```typescript
// components/Sprite.tsx
interface SpriteProps {
  image: string;
  frameWidth: number;
  frameHeight: number;
  currentFrame: number;
  row: number;
  animationRef?: React.RefObject<HTMLImageElement>;
}

export function Sprite({
  image,
  frameWidth,
  frameHeight,
  currentFrame,
  row,
}: SpriteProps) {
  return (
    <img
      src={image}
      style={{
        width: frameWidth,
        height: frameHeight,
        objectFit: 'none',
        imageRendering: 'pixelated',
        transform: `translate(${-currentFrame * frameWidth}px, ${-row * frameHeight}px)`,
      }}
    />
  );
}

// Animation hook
function useSpriteAnimation(
  frameCount: number,
  fps: number = 10,
  loop: boolean = true
) {
  const [frame, setFrame] = useState(0);
  const [isPlaying, setIsPlaying] = useState(true);

  useEffect(() => {
    if (!isPlaying) return;

    const interval = setInterval(() => {
      setFrame(prev => {
        const next = prev + 1;
        if (next >= frameCount) {
          if (loop) return 0;
          setIsPlaying(false);
          return prev;
        }
        return next;
      });
    }, 1000 / fps);

    return () => clearInterval(interval);
  }, [frameCount, fps, loop, isPlaying]);

  return { frame, isPlaying, play: () => setIsPlaying(true), pause: () => setIsPlaying(false) };
}
```

### Entity Component System

```typescript
// ecs/types.ts
interface Component {
  type: string;
}

interface Position extends Component {
  type: 'position';
  x: number;
  y: number;
}

interface Velocity extends Component {
  type: 'velocity';
  dx: number;
  dy: number;
}

interface Renderable extends Component {
  type: 'renderable';
  sprite: string;
  width: number;
  height: number;
}

interface Collidable extends Component {
  type: 'collidable';
  radius?: number;
  boundingBox?: { width: number; height: number };
}

type Entity = {
  id: string;
  components: Map<string, Component>;
};

// ecs/World.ts
class GameWorld {
  private entities: Map<string, Entity> = new Map();
  private systems: System[] = [];

  createEntity(...components: Component[]): Entity {
    const entity: Entity = {
      id: crypto.randomUUID(),
      components: new Map(components.map(c => [c.type, c])),
    };
    this.entities.set(entity.id, entity);
    return entity;
  }

  removeEntity(id: string): void {
    this.entities.delete(id);
  }

  query(...componentTypes: string[]): Entity[] {
    return Array.from(this.entities.values()).filter(entity =>
      componentTypes.every(type => entity.components.has(type))
    );
  }

  addSystem(system: System): void {
    this.systems.push(system);
  }

  update(deltaTime: number): void {
    for (const system of this.systems) {
      system.update(this, deltaTime);
    }
  }
}

// Systems
class MovementSystem implements System {
  update(world: GameWorld, deltaTime: number): void {
    const entities = world.query('position', 'velocity');
    for (const entity of entities) {
      const pos = entity.components.get('position') as Position;
      const vel = entity.components.get('velocity') as Velocity;
      pos.x += vel.dx * deltaTime;
      pos.y += vel.dy * deltaTime;
    }
  }
}
```

### Input Handling

```typescript
// hooks/useGameInput.ts
interface KeyState {
  keys: Set<string>;
  justPressed: Set<string>;
  justReleased: Set<string>;
}

export function useGameInput(): KeyState {
  const [keyState, setKeyState] = useState<KeyState>({
    keys: new Set(),
    justPressed: new Set(),
    justReleased: new Set(),
  });

  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      setKeyState(prev => ({
        ...prev,
        keys: new Set(prev.keys).add(e.code),
        justPressed: new Set(prev.justPressed).add(e.code),
      }));
    };

    const handleKeyUp = (e: KeyboardEvent) => {
      setKeyState(prev => {
        const newKeys = new Set(prev.keys);
        newKeys.delete(e.code);
        return {
          ...prev,
          keys: newKeys,
          justReleased: new Set(prev.justReleased).add(e.code),
        };
      });
    };

    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  }, []);

  // Clear justPressed/justReleased after frame
  useGameLoop(() => {
    setKeyState(prev => ({
      ...prev,
      justPressed: new Set(),
      justReleased: new Set(),
    }));
  }, true);

  return keyState;
}

// Usage
function PlayerController() {
  const { keys, justPressed } = useGameInput();

  useGameLoop((dt) => {
    if (keys.has('ArrowLeft')) moveLeft(dt);
    if (keys.has('ArrowRight')) moveRight(dt);
    if (justPressed.has('Space')) jump();
  }, true);
}
```

### Performance Tips

```typescript
// Use refs for frequently updating values
const positionRef = useRef({ x: 0, y: 0 });

// Memoize expensive calculations
const visibleEntities = useMemo(() => {
  return entities.filter(e => isOnScreen(e.position));
}, [entities]);

// Use CSS transforms (GPU accelerated)
const style = {
  transform: `translate(${x}px, ${y}px)`,
  willChange: 'transform',
};

// Batch state updates
setState(prev => ({
  ...prev,
  score: prev.score + points,
  entities: [...prev.entities, newEntity],
}));
```

### Best Practices

1. **Game loop** - Use requestAnimationFrame with deltaTime
2. **State machine** - Clear game states, predictable transitions
3. **ECS pattern** - Decouple data from logic
4. **Collision optimization** - Spatial partitioning for many objects
5. **Input buffering** - Handle input state, not events
6. **GPU acceleration** - Use CSS transforms for sprites
7. **Object pooling** - Reuse objects to reduce GC
