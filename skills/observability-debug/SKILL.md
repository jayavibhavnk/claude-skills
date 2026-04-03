---
name: observability-debug
description: Implement observability - logging, metrics, tracing, and debugging distributed systems.
metadata:
  priority: 9
  docs:
    - "https://opentelemetry.io/docs/"
  pathPatterns:
    - "**/observability/**"
    - "**/monitoring/**"
  bashPatterns:
    - '\bopentelemetry\b'
    - '\btracing\b'
    - '\blogging\b'
  promptSignals:
    phrases:
      - "observability"
      - "tracing"
      - "metrics"
    anyOf:
      - "logs"
      - "traces"
      - "distributed"
---

## Observability & Debugging

### The Three Pillars

```
┌─────────────────────────────────────────────────────────┐
│                    OBSERVABILITY                        │
├─────────────────┬─────────────────┬────────────────────┤
│     LOGS        │     METRICS     │      TRACES        │
│  "What happened"│ "How is it performing"│ "What happened" │
├─────────────────┼─────────────────┼────────────────────┤
│ Timestamped     │ Aggregated      │ Request-scoped     │
│ events          │ measurements    │ distributed view   │
├─────────────────┼─────────────────┼────────────────────┤
│ ELK, Loki       │ Prometheus,     │ Jaeger, Zipkin,    │
│                 │ Datadog         │ Tempo              │
└─────────────────┴─────────────────┴────────────────────┘
```

### Structured Logging

```typescript
import pino from 'pino';

// Create logger with consistent structure
const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: {
    service: 'order-service',
    version: process.env.VERSION,
  },
  timestamp: pino.stdTimeFunctions.isoTime,
});

// Log with context
const log = logger.child({ orderId: '123', userId: '456' });
log.info('Processing order');
log.error({ err, reason: 'validation' }, 'Order failed');

// Good log message patterns
log.info({ event: 'order_created', orderId, amount, items: items.length });
log.warn({ event: 'rate_limited', userId, endpoint, retryAfter });
log.error({ event: 'payment_failed', orderId, error: err.message, code });
```

### Log Levels

| Level | Use |
|-------|-----|
| DEBUG | Detailed debugging info (dev only) |
| INFO | Normal operations (business events) |
| WARN | Unexpected but handled (degraded) |
| ERROR | Failures that need attention |
| FATAL | Service down (immediate action) |

### Metrics with Prometheus

```typescript
import { Registry, Counter, Histogram, Gauge } from 'prom-client';

const registry = new Registry();

const httpRequests = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
  registers: [registry],
});

const httpDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
  registers: [registry],
});

const activeConnections = new Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
  registers: [registry],
});

// Use in middleware
app.use((req, res, next) => {
  const start = Date.now();
  activeConnections.inc();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequests.inc({ method: req.method, path: req.route?.path, status: res.statusCode });
    httpDuration.observe({ method: req.method, path: req.route?.path }, duration);
    activeConnections.dec();
  });

  next();
});

// Expose metrics
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', registry.contentType);
  res.end(await registry.metrics());
});
```

### Distributed Tracing

```typescript
import { NodeSDK } from '@opentelemetry/sdk-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  serviceName: 'order-service',
  traceExporter: new JaegerExporter({
    endpoint: 'http://localhost:14268/api/traces',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// In your code - add custom spans
import { trace, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

async function processOrder(orderId: string) {
  const span = tracer.startSpan('processOrder');

  try {
    span.setAttributes({ 'order.id': orderId });

    // Nested spans
    const paymentSpan = tracer.startSpan('processPayment', {
      parent: span,
    });
    await processPayment(orderId);
    paymentSpan.end();

    span.setStatus({ code: SpanStatusCode.OK });
  } catch (error) {
    span.recordException(error as Error);
    span.setStatus({ code: SpanStatusCode.ERROR });
    throw error;
  } finally {
    span.end();
  }
}
```

### Health Checks

```typescript
// Liveness probe - is it running?
app.get('/health/live', (req, res) => {
  res.json({ status: 'ok' });
});

// Readiness probe - can it handle traffic?
app.get('/health/ready', async (req, res) => {
  const checks = await Promise.all([
    checkDatabase(),
    checkRedis(),
    checkExternalAPI(),
  ]);

  const healthy = checks.every(c => c.healthy);

  res.status(healthy ? 200 : 503).json({
    status: healthy ? 'ok' : 'degraded',
    checks,
  });
});

async function checkDatabase(): Promise<HealthCheck> {
  try {
    await db.$queryRaw`SELECT 1`;
    return { name: 'database', healthy: true };
  } catch {
    return { name: 'database', healthy: false, error: 'Connection failed' };
  }
}
```

### Alerting Rules

```yaml
# Prometheus alerting rules
groups:
  - name: order-service
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate in order service"
          description: "Error rate is {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency in order service"
```

### Debugging Distributed Systems

```typescript
// Correlation ID middleware
app.use((req, res, next) => {
  const correlationId = req.headers['x-correlation-id'] || crypto.randomUUID();
  req.correlationId = correlationId;
  res.setHeader('x-correlation-id', correlationId);
  next();
});

// Trace context propagation
async function callService(url: string, data: any, correlationId: string) {
  return fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-correlation-id': correlationId,
    },
    body: JSON.stringify(data),
  });
}
```

### Best Practices

1. **Use structured logging** - JSON, not plain text
2. **Add correlation IDs** - Trace requests across services
3. **Instrument everything** - Logs, metrics, traces
4. **Sample appropriately** - 100% for errors, 1-10% for success
5. **Alert on symptoms** - Not causes (SLOs over SLAs)
6. **Dashboard with context** - Logs linked to traces
