---
name: game-networking
description: Multiplayer game networking - WebSocket, lag compensation, state synchronization, authoritative servers, and real-time game networking.
metadata:
  priority: 8
  docs:
    - "https://developer.mozilla.org/en-US/docs/Games/Techniques/Networking"
  pathPatterns:
    - "**/network/**"
    - "**/multiplayer/**"
  bashPatterns:
    - '\bwebsocket\b'
    - '\bmultiplayer\b'
  promptSignals:
    phrases:
      - "multiplayer game"
      - "game networking"
      - "real-time sync"
    anyOf:
      - "network"
      - "multiplayer"
      - "websocket"
---

## Multiplayer Game Networking

### Architecture Patterns

```typescript
// Authoritative server
interface GameServer {
  players: Map<string, Player>;
  worldState: WorldState;

  processInput(playerId: string, input: Input): void;
  simulate(deltaTime: number): void;
  broadcast(): void;
}

// Client-side prediction
interface NetworkedPlayer {
  id: string;
  position: Vector2;
  velocity: Vector2;

  // For client prediction
  predictedPosition: Vector2;
  lastProcessedInput: number;

  // Server reconciliation
  unprocessedInputs: Input[];
}
```

### WebSocket Server

```typescript
// server/game-server.ts
import { WebSocketServer, WebSocket } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

interface Client {
  ws: WebSocket;
  playerId: string;
  lastPing: number;
}

const clients = new Map<string, Client>();

wss.on('connection', (ws) => {
  const playerId = crypto.randomUUID();
  clients.set(playerId, { ws, playerId, lastPing: Date.now() });

  ws.on('message', (data) => {
    const message = JSON.parse(data.toString());
    handleMessage(playerId, message);
  });

  ws.on('close', () => {
    clients.delete(playerId);
    broadcast({ type: 'player_left', playerId });
  });
});

function handleMessage(playerId: string, message: GameMessage) {
  switch (message.type) {
    case 'player_input':
      server.processInput(playerId, message.input);
      break;
    case 'ping':
      sendTo(playerId, { type: 'pong', timestamp: message.timestamp });
      break;
  }
}

function broadcast(message: GameMessage) {
  const data = JSON.stringify(message);
  clients.forEach(client => {
    if (client.ws.readyState === WebSocket.OPEN) {
      client.ws.send(data);
    }
  });
}
```

### State Synchronization

```typescript
// Delta compression for state updates
interface StateUpdate {
  tick: number;
  changes: EntityChange[];
}

interface EntityChange {
  entityId: string;
  property: string;
  value: number | string | boolean;
  timestamp: number;
}

class StateSync {
  private lastState: Map<string, any> = new Map();

  createUpdate(entities: Entity[]): StateUpdate {
    const changes: EntityChange[] = [];

    for (const entity of entities) {
      for (const [key, value] of Object.entries(entity)) {
        const lastValue = this.lastState.get(`${entity.id}.${key}`);

        if (lastValue !== value) {
          changes.push({
            entityId: entity.id,
            property: key,
            value,
            timestamp: Date.now(),
          });
          this.lastState.set(`${entity.id}.${key}`, value);
        }
      }
    }

    return {
      tick: getCurrentTick(),
      changes,
    };
  }
}
```

### Client Prediction

```typescript
// Client-side prediction with server reconciliation
class NetworkedEntity {
  private serverState: Vector2;
  private predictedState: Vector2;
  private inputs: Input[] = [];

  applyLocalInput(input: Input) {
    this.inputs.push(input);
    this.predictedState = this.simulateInput(this.predictedState, input);
  }

  handleServerUpdate(state: ServerState, lastProcessedInput: number) {
    // Remove acknowledged inputs
    this.inputs = this.inputs.filter(i => i.sequence > lastProcessedInput);

    // Rewind to server state
    this.serverState = { x: state.x, y: state.y };

    // Replay remaining inputs
    this.predictedState = { ...this.serverState };
    for (const input of this.inputs) {
      this.predictedState = this.simulateInput(this.predictedState, input);
    }

    // If prediction diverges, snap to server
    if (this.isDiverged()) {
      this.predictedState = this.serverState;
    }
  }

  private isDiverged(): boolean {
    const threshold = 0.1;
    const dx = Math.abs(this.predictedState.x - this.serverState.x);
    const dy = Math.abs(this.predicted.y - this.serverState.y);
    return dx > threshold || dy > threshold;
  }
}
```

### Input Broadcasting

```typescript
// Input serialization with sequence numbers
interface Input {
  sequence: number;
  tick: number;
  keys: Set<string>;
  mousePosition: Vector2;
  timestamp: number;
}

class InputBroadcaster {
  private pendingInputs: Input[] = [];

  captureInput(keys: Set<string>, mousePos: Vector2): Input {
    const input: Input = {
      sequence: this.sequence++,
      tick: getCurrentTick(),
      keys: new Set(keys),
      mousePosition: { ...mousePos },
      timestamp: Date.now(),
    };

    this.pendingInputs.push(input);
    return input;
  }

  getUnprocessedInputs(serverAckTick: number): Input[] {
    return this.pendingInputs.filter(i => i.tick > serverAckTick);
  }

  clearAcknowledgedInputs(serverAckTick: number) {
    this.pendingInputs = this.pendingInputs.filter(i => i.tick <= serverAckTick);
  }
}
```

### Lag Compensation

```typescript
// Server-side lag compensation for hit detection
class LagCompensator {
  private frameHistory: Map<string, Frame[]> = new Map();
  private readonly FRAME_HISTORY = 128; // ~2 seconds at 60fps

  recordFrame(playerId: string, state: PlayerState) {
    const history = this.frameHistory.get(playerId) || [];
    history.push({ ...state, timestamp: Date.now() });

    if (history.length > this.FRAME_HISTORY) {
      history.shift();
    }

    this.frameHistory.set(playerId, history);
  }

  getStateAtTime(playerId: string, timestamp: number): PlayerState | null {
    const history = this.frameHistory.get(playerId);
    if (!history) return null;

    // Binary search for closest frame
    let left = 0;
    let right = history.length - 1;

    while (left < right) {
      const mid = Math.floor((left + right) / 2);
      if (history[mid].timestamp < timestamp) {
        left = mid + 1;
      } else {
        right = mid;
      }
    }

    return history[left] || null;
  }
}
```

### Interpolated Rendering

```typescript
// Client-side interpolation for smooth rendering
class Interpolator {
  private snapshots: Map<string, EntitySnapshot[]> = new Map();
  private readonly INTERPOLATION_DELAY = 100; // ms

  addSnapshot(entityId: string, snapshot: EntitySnapshot) {
    const entitySnapshots = this.snapshots.get(entityId) || [];
    entitySnapshots.push({ ...snapshot });

    // Keep only snapshots within interpolation window
    const cutoff = Date.now() - this.INTERPOLATION_DELAY;
    this.snapshots.set(entityId,
      entitySnapshots.filter(s => s.timestamp > cutoff)
    );
  }

  getInterpolatedState(entityId: string, renderTime: number): EntitySnapshot | null {
    const snapshots = this.snapshots.get(entityId);
    if (!snapshots || snapshots.length < 2) return null;

    // Find surrounding snapshots
    for (let i = 0; i < snapshots.length - 1; i++) {
      const curr = snapshots[i];
      const next = snapshots[i + 1];

      if (curr.timestamp <= renderTime && renderTime <= next.timestamp) {
        const t = (renderTime - curr.timestamp) / (next.timestamp - curr.timestamp);
        return {
          ...next,
          position: lerp(curr.position, next.position, t),
          rotation: lerp(curr.rotation, next.rotation, t),
        };
      }
    }

    return snapshots[snapshots.length - 1];
  }
}
```

### Anti-Cheat Basics

```typescript
// Server-side validation
class AntiCheat {
  validateMovement(playerId: string, oldState: State, newState: State): boolean {
    // Check speed hack
    const distance = Vector2.distance(oldState.position, newState.position);
    const timePassed = newState.timestamp - oldState.timestamp;
    const speed = distance / (timePassed / 1000);

    const MAX_WALK_SPEED = 8; // units per second
    if (speed > MAX_WALK_SPEED * 1.2) { // 20% tolerance
      this.flagPlayer(playerId, 'speed_hack', { speed, max: MAX_WALK_SPEED });
      return false;
    }

    // Check teleport
    if (distance > 50) {
      this.flagPlayer(playerId, 'teleport', { distance });
      return false;
    }

    return true;
  }

  validateAction(playerId: string, action: Action): boolean {
    // Check action cooldown
    const lastAction = this.lastActionTime.get(playerId);
    if (lastAction && Date.now() - lastAction < action.minCooldown) {
      this.flagPlayer(playerId, 'action_spam');
      return false;
    }

    this.lastActionTime.set(playerId, Date.now());
    return true;
  }
}
```

### Best Practices

1. **Authoritative server** - Never trust the client
2. **Client prediction** - Mask latency for players
3. **Server reconciliation** - Correct prediction errors
4. **Lag compensation** - Fair hit detection
5. **Interpolation** - Smooth entity rendering
6. **Delta compression** - Minimize bandwidth
7. **Heartbeats** - Detect disconnections
