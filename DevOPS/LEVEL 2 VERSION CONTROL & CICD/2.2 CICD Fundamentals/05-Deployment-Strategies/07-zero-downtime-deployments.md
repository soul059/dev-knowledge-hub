# Chapter 5.7: Zero-Downtime Deployments

[← Previous: Rollback Strategies](06-rollback-strategies.md) | [Back to README](../README.md) | [Next: Jenkins Architecture →](../06-Jenkins/01-jenkins-architecture.md)

---

## Overview

**Zero-downtime deployment** ensures users experience no service interruption during updates. This requires careful coordination of load balancing, health checks, connection draining, and database migrations.

---

## Requirements for Zero Downtime

```
  ┌──────────────────────────────────────────────────────┐
  │       ZERO-DOWNTIME DEPLOYMENT CHECKLIST             │
  │                                                      │
  │  1. LOAD BALANCER                                    │
  │     ✓ Routes traffic away from updating instances    │
  │     ✓ Health-check aware (only sends to healthy)     │
  │                                                      │
  │  2. HEALTH CHECKS                                    │
  │     ✓ Readiness probe: "Am I ready for traffic?"     │
  │     ✓ Liveness probe: "Am I still alive?"            │
  │                                                      │
  │  3. GRACEFUL SHUTDOWN                                │
  │     ✓ Stop accepting new connections                 │
  │     ✓ Finish processing in-flight requests           │
  │     ✓ Close DB connections cleanly                   │
  │                                                      │
  │  4. CONNECTION DRAINING                              │
  │     ✓ Allow existing connections to complete          │
  │     ✓ Configurable drain timeout                     │
  │                                                      │
  │  5. BACKWARD-COMPATIBLE CHANGES                      │
  │     ✓ API versioning                                 │
  │     ✓ Database schema compatibility                  │
  │     ✓ Message format compatibility                   │
  └──────────────────────────────────────────────────────┘
```

---

## Graceful Shutdown Flow

```
  GRACEFUL SHUTDOWN
  ═════════════════

  Signal received (SIGTERM)
        │
        ▼
  ┌─────────────────┐
  │ Stop accepting  │  ← Remove from load balancer
  │ new connections  │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Finish in-flight│  ← Complete current requests
  │ requests        │     (up to timeout)
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Close database  │  ← Release connections
  │ connections     │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Exit cleanly    │  ← Exit code 0
  └─────────────────┘

  Total time: terminationGracePeriodSeconds (default 30s)
```

### Node.js Graceful Shutdown

```javascript
const server = app.listen(3000);

// Handle shutdown signal
process.on('SIGTERM', () => {
  console.log('SIGTERM received. Shutting down gracefully...');

  // Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed');

    // Close database connections
    db.end(() => {
      console.log('Database connections closed');
      process.exit(0);
    });
  });

  // Force shutdown after timeout
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    process.exit(1);
  }, 25000);  // < terminationGracePeriodSeconds
});
```

### Python (Flask/Gunicorn)

```python
# gunicorn.conf.py
graceful_timeout = 30     # Seconds to finish requests
timeout = 30              # Worker timeout
worker_class = "gthread"  # Thread-based workers
workers = 4
```

---

## Health Checks in Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: myapp
          image: myapp:v2
          ports:
            - containerPort: 8080

          # Readiness: "Can I receive traffic?"
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5    # Wait 5s before first check
            periodSeconds: 5          # Check every 5s
            failureThreshold: 3       # 3 failures = not ready

          # Liveness: "Am I still alive?"
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 15   # Wait 15s
            periodSeconds: 10         # Check every 10s
            failureThreshold: 3       # 3 failures = restart

          # Startup: "Have I started yet?" (slow-starting apps)
          startupProbe:
            httpGet:
              path: /health/started
              port: 8080
            failureThreshold: 30      # 30 × 10s = 5 min to start
            periodSeconds: 10

          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 5"]  # Wait for LB to deregister
```

---

## Health Check Implementation

```javascript
// Express.js health check endpoints
const db = require('./db');
const cache = require('./cache');

// Readiness: Check all dependencies
app.get('/health/ready', async (req, res) => {
  try {
    await db.query('SELECT 1');          // Database OK?
    await cache.ping();                   // Cache OK?
    res.status(200).json({ status: 'ready' });
  } catch (err) {
    res.status(503).json({ status: 'not ready', error: err.message });
  }
});

// Liveness: Am I alive? (lightweight)
app.get('/health/live', (req, res) => {
  res.status(200).json({ status: 'alive' });
});

// Startup: Have I initialized?
let initialized = false;
app.get('/health/started', (req, res) => {
  if (initialized) {
    res.status(200).json({ status: 'started' });
  } else {
    res.status(503).json({ status: 'starting' });
  }
});
```

---

## Connection Draining

```
  CONNECTION DRAINING
  ═══════════════════

  WITHOUT DRAINING:                WITH DRAINING:
  ┌──────┐                        ┌──────┐
  │ User │──req──►[Old Pod]       │ User │──req──►[Old Pod]
  └──────┘     Pod killed! ✕      └──────┘     Finish request ✓
               Connection reset                 Then pod exits
               Error 502/504                    Clean response

  AWS ALB Configuration:
  deregistration_delay = 30        # seconds to drain

  Kubernetes:
  terminationGracePeriodSeconds: 30
  preStop hook: sleep 5            # Allow LB to remove pod
```

---

## Database Zero-Downtime Migrations

```
  SAFE MIGRATION PATTERN
  ══════════════════════

  ✕ UNSAFE: Rename column in single migration
  ┌────────────────────────────────────────────┐
  │  ALTER TABLE users RENAME COLUMN           │
  │    name TO full_name;                      │
  │                                            │
  │  v1 reads "name" → CRASH (column gone!)    │
  └────────────────────────────────────────────┘

  ✓ SAFE: Three-phase migration
  ┌────────────────────────────────────────────┐
  │  Phase 1: ADD new column                   │
  │  ALTER TABLE users ADD COLUMN full_name;   │
  │  v1: reads "name", v2: reads both ✓       │
  │                                            │
  │  Phase 2: MIGRATE data + dual-write        │
  │  UPDATE users SET full_name = name;        │
  │  App writes to both columns ✓              │
  │                                            │
  │  Phase 3: DROP old column (separate deploy)│
  │  ALTER TABLE users DROP COLUMN name;       │
  │  Only after ALL instances use full_name ✓  │
  └────────────────────────────────────────────┘
```

---

## API Backward Compatibility

```
  API VERSIONING FOR ZERO DOWNTIME
  ════════════════════════════════

  During rolling update, v1 and v2 pods coexist:

  Request → LB → v1 pod (expects old response format)
  Request → LB → v2 pod (returns new response format)

  SOLUTION 1: Version your API
  /api/v1/users  → old format  (keep working)
  /api/v2/users  → new format  (new feature)

  SOLUTION 2: Additive changes only
  v1: { "name": "Alice" }
  v2: { "name": "Alice", "email": "alice@example.com" }
  ✓ v2 adds field — doesn't break v1 consumers

  SOLUTION 3: Content negotiation
  Accept: application/vnd.myapp.v1+json
  Accept: application/vnd.myapp.v2+json
```

---

## Complete Zero-Downtime Pipeline

```yaml
# GitHub Actions
name: Zero-Downtime Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 1. Run database migration (backward-compatible)
      - name: Run safe migration
        run: |
          kubectl run migration --rm -i \
            --image=myapp:${{ github.sha }} \
            -- npm run migrate:safe

      # 2. Deploy with rolling update
      - name: Deploy
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.example.com/myapp:${{ github.sha }}

      # 3. Wait for healthy rollout
      - name: Wait for rollout
        run: kubectl rollout status deployment/myapp --timeout=300s

      # 4. Verify no downtime occurred
      - name: Check uptime
        run: |
          # Verify no 5xx errors in monitoring
          ERROR_COUNT=$(curl -s "http://prometheus:9090/api/v1/query?query=sum(increase(http_5xx_total[5m]))" \
            | jq '.data.result[0].value[1]')
          if [ "$ERROR_COUNT" != "0" ]; then
            echo "5xx errors detected! Rolling back..."
            kubectl rollout undo deployment/myapp
            exit 1
          fi
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Brief 502/504 errors during deploy | No preStop hook; LB still sends to terminating pod | Add `preStop: sleep 5`; configure connection draining |
| Health check flapping | Probe too aggressive; app slow to warm | Increase `initialDelaySeconds`; add startup probe |
| Old version crashes after migration | Migration not backward-compatible | Use expand-and-contract; test migrations against both versions |
| WebSocket connections dropped | Rolling update kills long-lived connections | Use `preStop` hook; implement reconnect logic in client |
| Memory spike during deploy | maxSurge creates extra pods | Reduce maxSurge; ensure resource requests are set |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Goal** | Update application with zero user-facing interruption |
| **Requires** | Health checks, graceful shutdown, connection draining |
| **Load balancer** | Must health-check-aware; drain connections |
| **Database** | Only backward-compatible (additive) migrations |
| **API** | Version APIs; only additive changes during rollout |
| **Validate** | Monitor for 5xx, dropped connections, latency spikes |

---

## Quick Revision Questions

1. **What is zero-downtime deployment?** *Updating an application so that users experience no service interruption — no errors, no timeouts, no dropped connections.*
2. **What is a readiness probe?** *A health check that tells Kubernetes whether a pod is ready to receive traffic. Failing readiness removes it from service.*
3. **What is graceful shutdown?** *Handling SIGTERM by stopping new connections, finishing in-flight requests, closing resources, then exiting cleanly.*
4. **Why add a preStop sleep hook?** *To give the load balancer time to deregister the pod before it starts terminating — preventing requests to a shutting-down pod.*
5. **What makes a database migration "safe" for zero downtime?** *It's additive only (add columns, not rename/drop), and both old and new app versions work with the schema.*
6. **What is connection draining?** *Allowing in-progress connections to complete before removing an instance, instead of abruptly terminating them.*

---

[← Previous: Rollback Strategies](06-rollback-strategies.md) | [Back to README](../README.md) | [Next: Jenkins Architecture →](../06-Jenkins/01-jenkins-architecture.md)
