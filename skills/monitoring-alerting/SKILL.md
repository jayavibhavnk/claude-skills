---
name: monitoring-alerting
description: Set up monitoring and alerting - dashboards, alerts, runbooks, and incident response.
metadata:
  priority: 8
  docs:
    - "https://grafana.com/docs/"
    - "https://prometheus.io/docs/"
  pathPatterns:
    - "**/monitoring/**"
    - "**/alerts/**"
  bashPatterns:
    - '\bprometheus\b'
    - '\bgrafana\b'
    - '\balerting\b'
  promptSignals:
    phrases:
      - "monitoring"
      - "alerting"
      - "dashboard"
    anyOf:
      - "monitoring"
      - "alerting"
      - "incident"
---

## Monitoring & Alerting

### The Golden Signals

| Signal | What it measures | Example metric |
|--------|-----------------|----------------|
| Latency | Time to response | p50, p95, p99 |
| Traffic | System demand | requests/sec |
| Errors | Failure rate | error % |
| Saturation | Capacity headroom | CPU, memory |

### Prometheus Metrics

```typescript
import { Registry, Counter, Histogram, Gauge } from 'prom-client';

const registry = new Registry();

// Counter - tracks total count
const httpRequests = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
  registers: [registry],
});

// Histogram - tracks distributions
const httpDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
  registers: [registry],
});

// Gauge - tracks current value
const activeConnections = new Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
  registers: [registry],
});

// Use in middleware
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequests.inc({ method: req.method, path: req.route?.path, status: res.statusCode });
    httpDuration.observe({ method: req.method, path: req.route?.path }, duration);
  });

  next();
});
```

### Alert Rules

```yaml
# prometheus-rules.yml
groups:
  - name: http-alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High HTTP error rate"
          description: "Error rate is {{ $value | humanizePercentage }}"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High HTTP latency"
          description: "p95 latency is {{ $value }}s"

      # Service down
      - alert: ServiceDown
        expr: up{job="my-service"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          runbook_url: "https://wiki/runbooks/service-down"
```

### Grafana Dashboard

```json
{
  "dashboard": {
    "title": "API Performance",
    "panels": [
      {
        "title": "Request Rate",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (path)"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~'5..'}[5m])) / sum(rate(http_requests_total[5m]))"
          }
        ],
        "fieldConfig": {
          "thresholds": {
            "steps": [
              { "value": 0, "color": "green" },
              { "value": 0.01, "color": "yellow" },
              { "value": 0.05, "color": "red" }
            ]
          }
        }
      },
      {
        "title": "Latency Distribution",
        "type": "heatmap",
        "targets": [
          {
            "expr": "rate(http_request_duration_seconds_bucket[5m])"
          }
        ]
      }
    ]
  }
}
```

### Runbooks

```markdown
# Runbook: High Error Rate

## Symptoms
- Error rate > 5% for 2+ minutes
- Users reporting failures

## Diagnosis
1. Check error logs: `kubectl logs -f deployment/api | grep ERROR`
2. Check database connectivity
3. Check external service dependencies

## Common Causes
- Database connection pool exhausted
- External API returning errors
- Memory/CPU saturation
- Deploy introduced bug

## Mitigation
1. Scale up: `kubectl scale deployment/api --replicas=10`
2. Rollback deploy: `kubectl rollout undo deployment/api`
3. Enable circuit breaker

## Escalation
If not resolved in 15 minutes, escalate to on-call.
```

### Incident Response

```markdown
## Incident Response Process

### P0 (Critical) - 5 minute response
1. Acknowledge in #incidents
2. Assess impact
3. Mitigate immediately (rollback, scale)
4. Communicate status

### P1 (High) - 15 minute response
1. Acknowledge
2. Diagnose root cause
3. Implement fix
4. Document timeline

### Post-Incident Review
- Timeline of events
- Root cause analysis
- What went well
- Action items (with owners)
```

### SLOs and SLAs

| Type | Target | Measurement |
|------|--------|-------------|
| SLO | 99.9% availability | Monthly |
| SLA | 99.5% availability | Contractual |
| Error budget | 0.1% of month | 43 min/month |

```yaml
# SLO definition
serviceLevelObjectives:
  - name: API availability
    target: 99.9
    window: 30d
    metric: |
      1 - (sum(rate(http_requests_total{status="500"}[1h]))
           / sum(rate(http_requests_total[1h])))

  - name: API latency
    target: 99.0
    window: 30d
    percentile: 95
    metric: http_request_duration_seconds
```

### Best Practices

1. **Start with SLOs** - Define what matters
2. **Alert on symptoms** - Not causes
3. **Use golden signals** - Latency, traffic, errors, saturation
4. **Write runbooks** - For every alert
5. **Practice incidents** - Runbook drills
6. **Track error budget** - Burn rate alerts
7. **Regular reviews** - Iterate on alerts
