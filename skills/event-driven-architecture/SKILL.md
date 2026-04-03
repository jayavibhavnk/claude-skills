---
name: event-driven-architecture
description: Build event-driven systems - message queues, event sourcing, CQRS, and real-time processing.
metadata:
  priority: 9
  docs:
    - "https://event-driven.io/"
  pathPatterns:
    - "**/events/**"
    - "**/messages/**"
  bashPatterns:
    - '\bkafka\b'
    - '\brabbitmq\b'
    - '\bevent\b'
  promptSignals:
    phrases:
      - "event driven"
      - "message queue"
      - "async"
    anyOf:
      - "event"
      - "queue"
      - "consumer"
      - "producer"
---

## Event-Driven Architecture

### Core Concepts

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Producer  │────▶│   Broker    │────▶│  Consumer   │
│   (Event)   │     │   (Queue)   │     │   (Handler) │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    │ Dead Letter │
                    │   Queue     │
                    └─────────────┘
```

### Event Structure

```typescript
interface DomainEvent<T = any> {
  id: string;           // UUID
  type: string;         // 'user.created', 'order.paid'
  version: number;      // Schema version
  timestamp: Date;      // When event occurred
  correlationId?: string; // Trace related events
  causationId?: string;   // What caused this event
  payload: T;          // Event data
  metadata: {
    userId?: string;
    tenantId?: string;
    [key: string]: any;
  };
}

// Example: User Created Event
interface UserCreatedPayload {
  userId: string;
  email: string;
  name: string;
  createdAt: Date;
}

const event: DomainEvent<UserCreatedPayload> = {
  id: crypto.randomUUID(),
  type: 'user.created',
  version: 1,
  timestamp: new Date(),
  correlationId: 'req-123',
  payload: {
    userId: 'usr_abc',
    email: 'user@example.com',
    name: 'John Doe',
    createdAt: new Date(),
  },
  metadata: {
    tenantId: 'tenant_1',
  },
};
```

### Message Broker Patterns

#### Kafka

```typescript
import { Kafka } from 'kafkajs';

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['localhost:9092'],
});

const producer = kafka.producer();
const consumer = kafka.consumer({ groupId: 'my-group' });

// Produce events
async function publishUserCreated(user: User) {
  await producer.send({
    topic: 'users',
    messages: [
      {
        key: user.id,
        value: JSON.stringify({
          type: 'user.created',
          payload: user,
          timestamp: Date.now(),
        }),
        headers: {
          'correlation-id': getCorrelationId(),
        },
      },
    ],
  });
}

// Consume events
async function consumeUserEvents() {
  await consumer.connect();
  await consumer.subscribe({ topic: 'users', fromBeginning: false });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      const event = JSON.parse(message.value.toString());
      console.log('Received:', event.type, event.payload);

      // Process based on event type
      switch (event.type) {
        case 'user.created':
          await handleUserCreated(event.payload);
          break;
        case 'user.deleted':
          await handleUserDeleted(event.payload);
          break;
      }
    },
  });
}
```

#### RabbitMQ

```typescript
import amqp from 'amqplib';

const connection = await amqp.connect('amqp://localhost');
const channel = await connection.createChannel();

// Setup exchange and queue
await channel.assertExchange('users', 'topic');
await channel.assertQueue('user-notifications');
await channel.bindQueue('user-notifications', 'users', 'user.*');

// Publish
async function publishUserEvent(routingKey: string, payload: any) {
  channel.publish(
    'users',
    routingKey,
    Buffer.from(JSON.stringify(payload)),
    { persistent: true }
  );
}

// Consume
channel.consume('user-notifications', (msg) => {
  if (msg) {
    const event = JSON.parse(msg.content.toString());
    console.log('Received:', event);
    channel.ack(msg);
  }
});
```

### Event Sourcing

```typescript
// Aggregate with event sourcing
class Order {
  private events: DomainEvent[] = [];

  constructor(private id: string, private state: OrderState) {}

  // Replay events to rebuild state
  static fromEvents(id: string, events: DomainEvent[]): Order {
    const order = new Order(id, { items: [], total: 0, status: 'pending' });
    events.forEach(event => order.apply(event));
    return order;
  }

  apply(event: DomainEvent) {
    switch (event.type) {
      case 'order.item_added':
        this.state.items.push(event.payload);
        this.state.total += event.payload.price;
        break;
      case 'order.item_removed':
        this.state.items = this.state.items.filter(
          i => i.id !== event.payload.itemId
        );
        this.state.total -= event.payload.price;
        break;
      case 'order.completed':
        this.state.status = 'completed';
        break;
    }
    this.events.push(event);
  }

  // Business methods return events, don't mutate directly
  addItem(item: Item): DomainEvent {
    return {
      id: crypto.randomUUID(),
      type: 'order.item_added',
      payload: { itemId: item.id, price: item.price },
      timestamp: new Date(),
    };
  }
}
```

### CQRS (Command Query Responsibility Segregation)

```typescript
// Command side - writes
async function handleCreateOrder(command: CreateOrderCommand) {
  const order = Order.create(command);

  // Save to write store
  await eventStore.save(order.id, order.events);

  // Publish event for read models
  await messageBus.publish(order.events);
}

// Query side - reads (denormalized read models)
interface OrderReadModel {
  orderId: string;
  customerName: string;
  itemCount: number;
  total: number;
  status: string;
}

// Projection builds read model from events
async function buildOrderReadModel(event: DomainEvent) {
  switch (event.type) {
    case 'order.created':
      await readStore.upsert('orders', {
        orderId: event.payload.orderId,
        customerName: event.payload.customerName,
        itemCount: 0,
        total: 0,
        status: 'pending',
      });
      break;

    case 'order.item_added':
      await readStore.update('orders', event.payload.orderId, {
        itemCount: { $inc: 1 },
        total: { $inc: event.payload.price },
      });
      break;
  }
}
```

### Outbox Pattern (Reliable Events)

```typescript
// Instead of publishing directly, write to outbox table
async function createOrder(order: Order) {
  await db.transaction(async (tx) => {
    // Save order
    await tx.orders.create({ data: order });

    // Write to outbox (same transaction)
    await tx.outbox.create({
      data: {
        eventType: 'order.created',
        payload: JSON.stringify(order),
        createdAt: new Date(),
      },
    });
  });
}

// Outbox processor (runs periodically)
async function processOutbox() {
  const pending = await db.outbox.findMany({
    where: { processedAt: null },
    orderBy: { createdAt: 'asc' },
    take: 100,
  });

  for (const item of pending) {
    try {
      await messageBus.publish(item.eventType, JSON.parse(item.payload));
      await db.outbox.update({
        where: { id: item.id },
        data: { processedAt: new Date() },
      });
    } catch (error) {
      // Will retry on next iteration
      console.error('Failed to process outbox item:', error);
    }
  }
}
```

### Best Practices

1. **Idempotency** - Handle duplicate events gracefully
2. **Ordering** - Use sequence numbers for ordering guarantees
3. **Schema evolution** - Version your events
4. **Dead letter queues** - Handle poison messages
5. **Correlation IDs** - Trace events across services
6. **At-least-once delivery** - Design for retries
7. **Eventual consistency** - Accept async delays
