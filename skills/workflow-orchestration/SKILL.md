---
name: workflow-orchestration
description: Orchestrate complex workflows - step functions, sagas, state machines, and long-running processes.
metadata:
  priority: 9
  docs:
    - "https://docs.aws.amazon.com/step-functions/"
  pathPatterns:
    - "**/workflow/**"
    - "**/orchestration/**"
  bashPatterns:
    - '\bstep.?function\b'
    - '\borgestrel\b'
    - '\bworkflow\b'
  promptSignals:
    phrases:
      - "workflow"
      - "orchestration"
      - "state machine"
    anyOf:
      - "workflow"
      - "orchestrate"
      - "saga"
---

## Workflow Orchestration

### When to Use Workflows

- Long-running processes (hours/days)
- Distributed transactions across services
- Human approval gates
- Scheduled/cron jobs
- Complex branching logic

### State Machine Pattern

```typescript
interface StateMachine<S, E> {
  initial: S;
  states: Record<S, {
    on?: Partial<Record<E, { target: S; action?: () => void }>>;
    entry?: () => void;
    exit?: () => void;
  }>;
}

type OrderEvent = 'PAYMENT_RECEIVED' | 'ITEMS_SHIPPED' | 'DELIVERED' | 'CANCELLED';
type OrderState = 'PENDING' | 'PAID' | 'SHIPPED' | 'DELIVERED' | 'CANCELLED';

const orderMachine: StateMachine<OrderState, OrderEvent> = {
  initial: 'PENDING',

  states: {
    PENDING: {
      on: {
        PAYMENT_RECEIVED: { target: 'PAID' },
        CANCELLED: { target: 'CANCELLED' },
      },
    },
    PAID: {
      entry: () => console.log('Order paid, preparing shipment'),
      on: {
        ITEMS_SHIPPED: { target: 'SHIPPED' },
        CANCELLED: { target: 'CANCELLED' },
      },
    },
    SHIPPED: {
      entry: () => sendTrackingEmail(),
      on: {
        DELIVERED: { target: 'DELIVERED' },
      },
    },
    DELIVERED: {
      entry: () => console.log('Order complete'),
    },
    CANCELLED: {
      entry: () => processRefund(),
    },
  },
};
```

### Step Functions (AWS)

```json
{
  "Comment": "Order processing workflow",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:lambda:validate-order",
      "Next": "ProcessPayment"
    },
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:lambda:process-payment",
      "TimeoutSeconds": 300,
      "Next": "FulfillOrder",
      "Catch": [{
        "ErrorEquals": ["PaymentFailed"],
        "Next": "NotifyPaymentFailed"
      }]
    },
    "FulfillOrder": {
      "Type": "Task",
      "Resource": "arn:lambda:fulfill-order",
      "Next": "SendConfirmation"
    },
    "SendConfirmation": {
      "Type": "Task",
      "Resource": "arn:sqs:send-email",
      "End": true
    },
    "NotifyPaymentFailed": {
      "Type": "Task",
      "Resource": "arn:sqs:send-alert",
      "End": true
    }
  }
}
```

### Saga Pattern

```typescript
// Saga for order processing
interface SagaStep<T> {
  name: string;
  execute: () => Promise<T>;
  compensate: () => Promise<void>;
}

async function executeSaga(steps: SagaStep<any>[]) {
  const completed: SagaStep<any>[] = [];

  try {
    for (const step of steps) {
      const result = await step.execute();
      completed.push(step);
    }
  } catch (error) {
    // Compensate in reverse order
    console.error(`Saga failed at ${completed.length} steps, compensating...`);

    for (const step of completed.reverse()) {
      try {
        await step.compensate();
      } catch (compensateError) {
        console.error(`Compensation failed for ${step.name}:`, compensateError);
      }
    }
    throw error;
  }
}

// Order saga
const orderSaga: SagaStep<any>[] = [
  {
    name: 'reserve-inventory',
    execute: () => reserveInventory(orderId),
    compensate: () => releaseInventory(orderId),
  },
  {
    name: 'charge-payment',
    execute: () => chargePayment(orderId),
    compensate: () => refundPayment(orderId),
  },
  {
    name: 'create-shipment',
    execute: () => createShipment(orderId),
    compensate: () => cancelShipment(orderId),
  },
];
```

### Long-Running Workflows

```typescript
interface WorkflowInstance {
  id: string;
  type: string;
  state: any;
  status: 'running' | 'completed' | 'failed' | 'cancelled';
  createdAt: Date;
  updatedAt: Date;
  steps: StepExecution[];
}

interface StepExecution {
  name: string;
  status: 'pending' | 'running' | 'completed' | 'failed' | 'skipped';
  startedAt?: Date;
  completedAt?: Date;
  input?: any;
  output?: any;
  error?: string;
}

// Workflow orchestrator
class WorkflowEngine {
  async start(workflowId: string, input: any) {
    const instance: WorkflowInstance = {
      id: crypto.randomUUID(),
      type: workflowId,
      state: { input },
      status: 'running',
      createdAt: new Date(),
      updatedAt: new Date(),
      steps: [],
    };

    await this.persist(instance);

    // Run workflow asynchronously
    this.runWorkflow(instance).catch(console.error);

    return instance;
  }

  async runWorkflow(instance: WorkflowInstance) {
    const workflow = this.getWorkflow(instance.type);

    for (const stepDef of workflow.steps) {
      const step: StepExecution = { name: stepDef.name, status: 'running', startedAt: new Date() };
      instance.steps.push(step);

      try {
        step.output = await stepDef.execute(instance.state);
        step.status = 'completed';
        step.completedAt = new Date();
      } catch (error) {
        step.status = 'failed';
        step.error = (error as Error).message;
        instance.status = 'failed';
        break;
      }

      instance.updatedAt = new Date();
      await this.persist(instance);
    }

    if (instance.status === 'running') {
      instance.status = 'completed';
    }
    instance.updatedAt = new Date();
    await this.persist(instance);
  }
}
```

### Human-in-the-Loop

```typescript
// Approval workflow
const approvalWorkflow = {
  steps: [
    {
      name: 'submit_request',
      execute: (ctx) => submitForApproval(ctx.request),
    },
    {
      name: 'wait_for_approval',
      execute: async (ctx) => {
        // Pause workflow until human approves
        return new Promise((resolve, reject) => {
          // This would integrate with a notification system
          onApproval(ctx.request.id, (approved) => {
            if (approved) resolve({ approved: true });
            else reject(new Error('Request rejected'));
          });
        });
      },
    },
    {
      name: 'process_result',
      execute: (ctx) => {
        if (ctx.wait_for_approval.approved) {
          return processRequest(ctx.request);
        }
      },
    },
  ],
};
```

### Retry Policies

```typescript
interface RetryPolicy {
  maxAttempts: number;
  initialDelay: number;
  backoff: 'fixed' | 'exponential';
  maxDelay: number;
}

const defaultRetry: RetryPolicy = {
  maxAttempts: 3,
  initialDelay: 1000,
  backoff: 'exponential',
  maxDelay: 30000,
};

async function withRetry<T>(
  fn: () => Promise<T>,
  policy: RetryPolicy = defaultRetry
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= policy.maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      if (attempt < policy.maxAttempts) {
        const delay = Math.min(
          policy.initialDelay * Math.pow(2, attempt - 1),
          policy.maxDelay
        );
        await sleep(delay);
      }
    }
  }

  throw lastError!;
}
```

### Best Practices

1. **Idempotency** - Make steps safe to retry
2. **Compensation** - Always have rollback plan
3. **Timeouts** - Set appropriate timeouts
4. **Observability** - Log all step transitions
5. **State persistence** - Store workflow state durably
6. **Human gates** - Design for approval workflows
7. **Error handling** - Distinguish retryable vs fatal
