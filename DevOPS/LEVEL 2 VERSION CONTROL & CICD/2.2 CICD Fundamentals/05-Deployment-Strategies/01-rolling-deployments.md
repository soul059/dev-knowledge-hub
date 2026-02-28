# Chapter 5.1: Rolling Deployments

[← Previous: Test Automation Best Practices](../04-Testing-In-CICD/07-test-automation-best-practices.md) | [Back to README](../README.md) | [Next: Blue-Green Deployments →](02-blue-green-deployments.md)

---

## Overview

A **rolling deployment** gradually replaces instances of the old version with the new version, one (or a batch) at a time. At any point during the deployment, both old and new versions coexist.

---

## How Rolling Deployments Work

```
  ROLLING DEPLOYMENT (4 instances)
  ════════════════════════════════

  Step 1: All v1                Step 2: Replace 1st
  ┌────┐ ┌────┐ ┌────┐ ┌────┐  ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │ v1 │ │ v1 │ │ v1 │ │ v1 │  │ v2 │ │ v1 │ │ v1 │ │ v1 │
  └────┘ └────┘ └────┘ └────┘  └────┘ └────┘ └────┘ └────┘

  Step 3: Replace 2nd          Step 4: Replace 3rd
  ┌────┐ ┌────┐ ┌────┐ ┌────┐  ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │ v2 │ │ v2 │ │ v1 │ │ v1 │  │ v2 │ │ v2 │ │ v2 │ │ v1 │
  └────┘ └────┘ └────┘ └────┘  └────┘ └────┘ └────┘ └────┘

  Step 5: All v2 (Complete!)
  ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │ v2 │ │ v2 │ │ v2 │ │ v2 │
  └────┘ └────┘ └────┘ └────┘

  ✓ Zero downtime
  ✓ Gradual rollout
  ⚠ Both versions serve traffic during rollout
```

---

## Kubernetes Rolling Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Max extra pods during update
      maxUnavailable: 1    # Max pods down at once
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:v2
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
```

```bash
# Trigger rolling update
kubectl set image deployment/myapp myapp=myapp:v2

# Watch the rollout
kubectl rollout status deployment/myapp

# Rollback if needed
kubectl rollout undo deployment/myapp
```

---

## maxSurge and maxUnavailable

```
  maxSurge=1, maxUnavailable=1 (4 replicas)
  ═══════════════════════════════════════════

  Capacity during rollout:
  Min available: replicas - maxUnavailable = 4 - 1 = 3
  Max total pods: replicas + maxSurge      = 4 + 1 = 5

  Timeline:
  ┌─────────────────────────────────────────────┐
  │  v1 pods:  4 → 3 → 2 → 1 → 0              │
  │  v2 pods:  0 → 1 → 2 → 3 → 4              │
  │  Total:    4   4   4   4   4               │
  │  Serving:  ≥3  ≥3  ≥3  ≥3  4              │
  └─────────────────────────────────────────────┘

  AGGRESSIVE: maxSurge=4, maxUnavailable=0
  ► Spins up 4 new pods first, then removes old (needs 2x resources)

  CONSERVATIVE: maxSurge=0, maxUnavailable=1
  ► Removes 1 old pod, starts 1 new pod (slow but no extra resources)
```

---

## Health Checks Are Critical

```
  WITHOUT HEALTH CHECKS              WITH HEALTH CHECKS
  ════════════════════               ═══════════════════

  Deploy v2 ──► v2 starts           Deploy v2 ──► v2 starts
             ──► Traffic routed                ──► Wait for readiness
             ──► v2 crashes! 500s              ──► Probe: /health → 200
             ──► Users impacted                ──► Traffic routed
                                               ──► v2 serves OK ✓

  If v2 fails health check:
  ──► Kubernetes doesn't route traffic to it
  ──► Old v1 pods continue serving
  ──► Deployment is blocked until fixed
```

---

## CI/CD Pipeline with Rolling Deployment

```yaml
# GitHub Actions — Deploy with rolling update
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build & push image
        run: |
          docker build -t registry.example.com/myapp:${{ github.sha }} .
          docker push registry.example.com/myapp:${{ github.sha }}

      - name: Update Kubernetes deployment
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.example.com/myapp:${{ github.sha }}

      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/myapp --timeout=300s

      - name: Verify deployment
        run: |
          curl -f http://myapp.example.com/health || \
            (kubectl rollout undo deployment/myapp && exit 1)
```

---

## AWS ECS Rolling Deployment

```json
{
  "deploymentConfiguration": {
    "maximumPercent": 200,
    "minimumHealthyPercent": 50,
    "deploymentCircuitBreaker": {
      "enable": true,
      "rollback": true
    }
  }
}
```

```
  ECS Rolling Update:
  minimumHealthyPercent=50, maximumPercent=200

  Running tasks:  [v1] [v1] [v1] [v1]     (4 tasks, 100%)
  Start update:   [v1] [v1] [v1] [v1] [v2] [v2]  (150%)
  Draining v1:    [v1] [v1] [v2] [v2] [v2] [v2]  (150%)
  Complete:       [v2] [v2] [v2] [v2]     (100%)
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Deployment stuck / not progressing | New pods failing readiness probe | Check logs: `kubectl logs -l app=myapp`; fix health endpoint |
| Users see errors during rollout | API incompatibility between v1 & v2 | Ensure backward compatibility; version APIs |
| Rollout too slow | Conservative maxSurge/maxUnavailable | Increase maxSurge; add more resources |
| Resource exhaustion during rollout | maxSurge too high | Reduce maxSurge; ensure cluster has headroom |
| Session affinity broken | User switches between v1 and v2 | Use sticky sessions or stateless design |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **How it works** | Replace instances one at a time with new version |
| **Downtime** | Zero (if health checks are configured) |
| **Rollback** | `kubectl rollout undo` or redeploy previous version |
| **Risk** | Low — gradual replacement limits blast radius |
| **Resources** | Needs headroom for extra pods (maxSurge) |
| **Best for** | Stateless services; standard web applications |

---

## Quick Revision Questions

1. **What is a rolling deployment?** *A strategy that gradually replaces old instances with new ones, maintaining availability throughout.*
2. **What do maxSurge and maxUnavailable control?** *maxSurge: max extra pods above desired count during update. maxUnavailable: max pods that can be down simultaneously.*
3. **Why are health checks critical for rolling deployments?** *They prevent traffic from being routed to unhealthy new pods, ensuring only ready instances serve users.*
4. **How do you rollback a rolling deployment in Kubernetes?** *`kubectl rollout undo deployment/myapp` — reverts to the previous version.*
5. **What is the main risk of rolling deployments?** *Both versions run simultaneously, so v1 and v2 must be backward-compatible (especially API contracts and database schema).*
6. **When should you NOT use rolling deployments?** *When the new version has breaking changes incompatible with the old version running concurrently.*

---

[← Previous: Test Automation Best Practices](../04-Testing-In-CICD/07-test-automation-best-practices.md) | [Back to README](../README.md) | [Next: Blue-Green Deployments →](02-blue-green-deployments.md)
