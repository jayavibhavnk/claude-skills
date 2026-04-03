---
name: real-time-collab
description: Build real-time collaboration features - websockets, presence, operational transforms, and multiplayer UX.
metadata:
  priority: 8
  docs:
    - "https://pusher.com/"
    - "https://socket.io/"
  pathPatterns:
    - "**/collab/**"
    - "**/realtime/**"
  bashPatterns:
    - '\bwebsocket\b'
    - '\bsocket\.io\b'
    - '\bpusher\b'
  promptSignals:
    phrases:
      - "real-time"
      - "collaboration"
      - "websocket"
    anyOf:
      - "real-time"
      - "websockets"
      - "presence"
      - " multiplayer"
---

## Real-Time Collaboration

### Architecture

```
┌─────────┐         ┌─────────┐         ┌─────────┐
│ Client  │────────▶│  Server │◀────────│ Client  │
│   A     │◀────────│  (Hub)  │────────▶│   B     │
└─────────┘         └────┬────┘         └─────────┘
                         │
                    ┌────▼────┐
                    │Presence │
                    │ Service │
                    └─────────┘
```

### WebSocket Server (Node.js)

```typescript
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

const clients = new Map<string, Set<WebSocket>>();

wss.on('connection', (ws, req) => {
  const roomId = req.url?.split('/')[1];
  const clientId = crypto.randomUUID();

  // Add to room
  if (!clients.has(roomId)) {
    clients.set(roomId, new Set());
  }
  clients.get(roomId)!.add(ws);

  // Broadcast to room
  broadcast(roomId, {
    type: 'user_joined',
    clientId,
    timestamp: Date.now(),
  });

  ws.on('message', (data) => {
    const message = JSON.parse(data.toString());
    handleMessage(roomId, clientId, message);
  });

  ws.on('close', () => {
    clients.get(roomId)?.delete(ws);
    broadcast(roomId, {
      type: 'user_left',
      clientId,
    });
  });
});

function broadcast(roomId: string, message: any) {
  const room = clients.get(roomId);
  if (room) {
    room.forEach(client => {
      if (client.readyState === 1) { // OPEN
        client.send(JSON.stringify(message));
      }
    });
  }
}
```

### Presence System

```typescript
interface PresenceState {
  userId: string;
  name: string;
  avatar: string;
  cursor?: { x: number; y: number };
  selection?: { start: number; end: number };
}

// Store presence for a room
const presence = new Map<string, Map<string, PresenceState>>();

function updatePresence(roomId: string, userId: string, state: Partial<PresenceState>) {
  const room = presence.get(roomId) || new Map();
  const current = room.get(userId) || { userId };
  room.set(userId, { ...current, ...state });
  presence.set(roomId, room);

  // Broadcast presence update
  broadcast(roomId, {
    type: 'presence_update',
    userId,
    state: room.get(userId),
  });
}

function getRoomPresence(roomId: string): PresenceState[] {
  const room = presence.get(roomId);
  return room ? Array.from(room.values()) : [];
}
```

### Operational Transform (Text Editing)

```typescript
interface Operation {
  type: 'insert' | 'delete' | 'retain';
  position: number;
  text?: string;
  length?: number;
}

// Apply operation to document
function applyOperation(doc: string, op: Operation): string {
  switch (op.type) {
    case 'insert':
      return doc.slice(0, op.position) + op.text + doc.slice(op.position);
    case 'delete':
      return doc.slice(0, op.position) + doc.slice(op.position + (op.length || 0));
    case 'retain':
      return doc;
  }
}

// Transform operation A against operation B
function transform(opA: Operation, opB: Operation): Operation {
  if (opA.type === 'insert' && opB.type === 'insert') {
    if (opA.position <= opB.position) {
      return { ...opA, position: opB.position + (opB.text?.length || 0) };
    }
  }
  // ... more cases
  return opA;
}
```

### Cursor Sync

```typescript
interface CursorUpdate {
  clientId: string;
  userId: string;
  position: number;  // Character index
  selection?: { start: number; end: number };
}

// Send cursor position (throttled)
function sendCursorUpdate(ws: WebSocket, cursor: CursorUpdate) {
  ws.send(JSON.stringify({
    type: 'cursor',
    ...cursor,
  }));
}

// On receiving cursor update
function handleCursorUpdate(roomId: string, cursor: CursorUpdate) {
  broadcast(roomId, {
    type: 'cursor',
    ...cursor,
  });
}
```

### Conflict-Free Replicated Data (CRDT)

```typescript
// Simple CRDT for key-value pairs
class CRDTMap {
  private state: Map<string, { value: any; timestamp: number; clientId: string }>;

  constructor() {
    this.state = new Map();
  }

  set(key: string, value: any, clientId: string, timestamp: number) {
    const existing = this.state.get(key);
    if (!existing || timestamp > existing.timestamp ||
        (timestamp === existing.timestamp && clientId > existing.clientId)) {
      this.state.set(key, { value, timestamp, clientId });
    }
  }

  get(key: string) {
    return this.state.get(key)?.value;
  }
}
```

### React Hook for Presence

```typescript
import { useEffect, useState } from 'react';

function usePresence(roomId: string, userId: string) {
  const [users, setUsers] = useState<User[]>([]);
  const [cursors, setCursors] = useState<Map<string, Cursor>>(new Map());

  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/${roomId}`);

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);

      switch (data.type) {
        case 'presence_update':
          setUsers(prev => {
            const idx = prev.findIndex(u => u.id === data.userId);
            if (idx >= 0) {
              const updated = [...prev];
              updated[idx] = data.state;
              return updated;
            }
            return [...prev, data.state];
          });
          break;

        case 'cursor':
          setCursors(prev => {
            const updated = new Map(prev);
            updated.set(data.clientId, data);
            return updated;
          });
          break;
      }
    };

    return () => ws.close();
  }, [roomId, userId]);

  return { users, cursors };
}
```

### Best Practices

1. **Throttle updates** - Don't send every keystroke
2. **Batch operations** - Group multiple changes
3. **Compress messages** - Use efficient encoding
4. **Handle disconnect** - Show offline status
5. **Optimistic UI** - Update locally first
6. **Conflict resolution** - Use CRDTs or OT
7. **Reconnection** - Sync state on reconnect
