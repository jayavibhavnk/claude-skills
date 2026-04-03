---
name: zero-downtime-deploy
description: Implement zero-downtime deployments - blue-green, canary, feature flags, and rollback strategies.
metadata:
  priority: 9
  docs:
    - "https://vercel.com/docs/deployments/deployment-methods"
  pathPatterns:
    - "**/deploy/**"
    - "**/release/**"
  bashPatterns:
    - '\bblue.green\b'
    - '\bcanary\b'
    - '\brollback\b'
  promptSignals:
    phrases:
      - "zero downtime"
      - "deployment"
      - "rollback"
    anyOf:
      - "deploy"
      - "release"
      - "canary"
---

## Zero-Downtime Deployments

### Strategies Overview

```
Blue/Green:        Canary:
┌─────────────┐    ┌─────────────┐
│  Green (v2) │    │  10% v2    │
│    NEW       │    │   NEW      │
└─────────────┘    └─────────────┘
       ↑                ↑
┌─────────────┐    ┌─────────────┐
│  Blue (v1)  │    │  90% v1     │
│    OLD       │    │   OLD       │
└─────────────┘    └─────────────┘
```

### Blue-Green Deployment

```typescript
// Infrastructure: Two identical environments
// Traffic switches instantly between them

interface Deployment {
  id: string;
  version: string;
  status: 'active' | 'standby' | 'deploying';
  url: string;
}

class BlueGreenDeployer {
  private blue: Deployment;
  private green: Deployment;
  private currentActive: 'blue' | 'green' = 'blue';

  async deploy(version: string): Promise<void> {
    // Deploy to inactive environment
    const target = this.currentActive === 'blue' ? 'green' : 'blue';

    await this.deployTo(target, version);

    // Run smoke tests
    await this.runSmokeTests(target);

    // Switch traffic
    await this.switchTraffic(target);

    this.currentActive = target;
  }

  private async deployTo(env: 'blue' | 'green', version: string) {
    // Deploy new version to target environment
    // Update deployment status
  }

  async rollback(): Promise<void> {
    // Switch back to previous environment
    const target = this.currentActive === 'blue' ? 'green' : 'blue';
    await this.switchTraffic(target);
    this.currentActive = target;
  }
}
```

### Canary Deployment

```typescript
// Gradually shift traffic to new version

interface CanaryConfig {
  initialWeight: number;    // 10%
  increment: number;         // +10%
  intervalMs: number;        // Every 5 minutes
  finalWeight: number;       // 100%
}

class CanaryDeployer {
  async deploy(version: string, config: CanaryConfig) {
    let weight = config.initialWeight;

    // Deploy new version
    await this.deployVersion(version);

    while (weight < config.finalWeight) {
      // Update traffic split
      await this.adjustTraffic(weight);

      // Monitor metrics
      const metrics = await this.getMetrics();
      if (this.hasAnomalies(metrics)) {
        console.log('Anomaly detected, rolling back');
        return this.rollback();
      }

      // Wait before next increment
      await sleep(config.intervalMs);
      weight += config.increment;
    }

    // Fully migrate to new version
    await this.adjustTraffic(100);
    await this.cleanupOldVersion();
  }
}
```

### Feature Flags

```typescript
interface FeatureFlag {
  key: string;
  enabled: boolean;
  rolloutPercentage?: number;  // 0-100
  userIds?: string[];           // Specific users
}

class FeatureFlagService {
  private flags: Map<string, FeatureFlag> = new Map();

  isEnabled(flagKey: string, userId?: string): boolean {
    const flag = this.flags.get(flagKey);
    if (!flag || !flag.enabled) return false;

    // Check specific users
    if (flag.userIds?.includes(userId!)) return true;

    // Check rollout percentage
    if (flag.rolloutPercentage !== undefined) {
      const hash = this.hashUserId(userId!);
      return hash < flag.rolloutPercentage;
    }

    return true;
  }
}

// Usage in code
if (featureFlags.isEnabled('new-checkout', user.id)) {
  return newCheckoutFlow();
} else {
  return legacyCheckoutFlow();
}
```

### Traffic Splitting

```typescript
// Vercel: Use headers/cookies for routing
// middleware.ts
export function middleware(request: NextRequest) {
  const testGroup = request.cookies.get('test-group');

  // Route 10% to new version
  if (!testGroup && Math.random() < 0.1) {
    const response = NextResponse.next();
    response.cookies.set('test-group', 'B', { maxAge: 60 * 60 * 24 * 30 });
    response.headers.set('X-Deploy-Type', 'canary');
    return response;
  }

  const response = NextResponse.next();
  response.headers.set('X-Deploy-Type', testGroup === 'B' ? 'canary' : 'stable');
  return response;
}

// API route
export async function GET() {
  const deployType = headers().get('X-Deploy-Type');

  if (deployType === 'canary') {
    return Response.json({ version: '2.0', deployType });
  }
  return Response.json({ version: '1.0', deployType });
}
```

### Health Checks

```typescript
interface HealthCheck {
  name: string;
  check: () => Promise<boolean>;
  timeout: number;
}

const healthChecks: HealthCheck[] = [
  { name: 'database', check: checkDB, timeout: 5000 },
  { name: 'cache', check: checkCache, timeout: 2000 },
  { name: 'external-api', check: checkExternalAPI, timeout: 3000 },
];

async function performHealthCheck(): Promise<HealthCheckResult> {
  const results = await Promise.all(
    healthChecks.map(async (check) => {
      try {
        const result = await Promise.race([
          check.check(),
          new Promise((_, reject) =>
            setTimeout(() => reject(new Error('Timeout')), check.timeout)
          ),
        ]);
        return { name: check.name, healthy: result };
      } catch {
        return { name: check.name, healthy: false };
      }
    })
  );

  const allHealthy = results.every(r => r.healthy);

  return {
    status: allHealthy ? 'healthy' : 'degraded',
    checks: results,
    timestamp: new Date(),
  };
}
```

### Rollback Procedures

```typescript
// Automated rollback triggers
const rollbackTriggers = {
  errorRate: 0.05,        // 5% error rate triggers rollback
  p99Latency: 2000,        // 2s p99 latency
  crashCount: 10,          // 10 crashes in 5 minutes
};

async function monitorDeployment(deploymentId: string) {
  const metrics = await collectMetrics(deploymentId);

  if (metrics.errorRate > rollbackTriggers.errorRate) {
    console.log(`High error rate: ${metrics.errorRate}, rolling back`);
    await rollback(deploymentId);
  }

  if (metrics.p99Latency > rollbackTriggers.p99Latency) {
    console.log(`High latency: ${metrics.p99Latency}ms, rolling back`);
    await rollback(deploymentId);
  }
}

async function rollback(deploymentId: string) {
  // 1. Stop traffic to failing deployment
  // 2. Point traffic to last known good version
  // 3. Run health checks on old version
  // 4. Notify team
  // 5. Create incident report
}
```

### Deployment Checklist

```markdown
## Pre-Deploy
- [ ] Run full test suite
- [ ] Update changelog
- [ ] Notify team in #deployments
- [ ] Verify rollback procedure

## During Deploy
- [ ] Monitor error rates
- [ ] Watch latency metrics
- [ ] Check health endpoints
- [ ] Monitor logs

## Post-Deploy
- [ ] Verify traffic split
- [ ] Run smoke tests
- [ ] Update documentation
- [ ] Close deployment issue
```

### Best Practices

1. **Automate everything** - No manual deployments
2. **Feature flags** - Kill switch for every feature
3. **Monitor aggressively** - Alert on anomalies
4. **Rollback fast** - Make rollback trivial
5. **Canary first** - Test with small percentage
6. **Health checks** - Automated verification
7. **Communication** - Keep team informed
