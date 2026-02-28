# Chapter 5.2: Blue-Green Deployments

[← Previous: Rolling Deployments](01-rolling-deployments.md) | [Back to README](../README.md) | [Next: Canary Deployments →](03-canary-deployments.md)

---

## Overview

**Blue-Green deployment** maintains two identical production environments. One ("Blue") serves live traffic while the other ("Green") receives the new version. After testing Green, traffic is switched instantly — providing zero-downtime deployments with instant rollback.

---

## How Blue-Green Works

```
  BLUE-GREEN DEPLOYMENT
  ═════════════════════

  BEFORE: Blue is live
  ┌──────────┐        ┌──────────────┐
  │  Users   │───────►│ Load Balancer│
  └──────────┘        └──────┬───────┘
                             │
                     ┌───────▼───────┐
                     │  BLUE (v1)    │ ◄── LIVE
                     │  ┌──┐┌──┐┌──┐│
                     │  │v1││v1││v1││
                     │  └──┘└──┘└──┘│
                     └───────────────┘
                     ┌───────────────┐
                     │  GREEN (v2)   │ ◄── IDLE (deploy here)
                     │  ┌──┐┌──┐┌──┐│
                     │  │v2││v2││v2││
                     │  └──┘└──┘└──┘│
                     └───────────────┘

  AFTER: Switch traffic to Green
  ┌──────────┐        ┌──────────────┐
  │  Users   │───────►│ Load Balancer│
  └──────────┘        └──────┬───────┘
                             │
                     ┌───────────────┐
                     │  BLUE (v1)    │ ◄── IDLE (rollback target)
                     │  ┌──┐┌──┐┌──┐│
                     │  │v1││v1││v1││
                     │  └──┘└──┘└──┘│
                     └───────────────┘
                     ┌───────▼───────┐
                     │  GREEN (v2)   │ ◄── LIVE
                     │  ┌──┐┌──┐┌──┐│
                     │  │v2││v2││v2││
                     │  └──┘└──┘└──┘│
                     └───────────────┘
```

---

## Blue-Green Deployment Steps

```
  ┌──────────────────────────────────────────────────┐
  │         BLUE-GREEN DEPLOYMENT PROCESS            │
  │                                                  │
  │  1. Deploy v2 to Green environment               │
  │  2. Run smoke tests against Green                │
  │  3. Run integration tests against Green          │
  │  4. Switch load balancer to Green                │
  │  5. Monitor metrics (errors, latency)            │
  │  6. If OK → decommission Blue (or keep standby)  │
  │  7. If FAIL → switch back to Blue (instant!)     │
  │                                                  │
  │  Rollback time: Seconds (just switch LB back)    │
  └──────────────────────────────────────────────────┘
```

---

## Kubernetes Blue-Green

```yaml
# blue-deployment.yaml (current production)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
        - name: myapp
          image: myapp:v1
          ports: [{ containerPort: 8080 }]
---
# green-deployment.yaml (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
        - name: myapp
          image: myapp:v2
          ports: [{ containerPort: 8080 }]
---
# service.yaml — Points to active environment
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: blue    # ← Change to "green" to switch
  ports:
    - port: 80
      targetPort: 8080
```

```bash
# Switch traffic from Blue to Green
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback — switch back to Blue
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
```

---

## AWS Blue-Green (with ALB)

```yaml
# Using target groups with ALB
# Blue Target Group: tg-blue (running v1)
# Green Target Group: tg-green (running v2)

# AWS CLI — Switch traffic
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/... \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/tg-green/...

# Rollback
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/... \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/tg-blue/...
```

---

## CI/CD Pipeline for Blue-Green

```yaml
# GitHub Actions — Blue-Green deployment
name: Blue-Green Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Build & push
      - run: |
          docker build -t myapp:${{ github.sha }} .
          docker push registry.example.com/myapp:${{ github.sha }}

      # Deploy to inactive environment
      - name: Deploy to Green
        run: |
          kubectl set image deployment/myapp-green \
            myapp=registry.example.com/myapp:${{ github.sha }}
          kubectl rollout status deployment/myapp-green --timeout=300s

      # Test Green environment
      - name: Smoke test Green
        run: |
          GREEN_URL=$(kubectl get svc myapp-green-test -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
          curl -f $GREEN_URL/health
          npm run test:smoke -- --base-url=$GREEN_URL

      # Switch traffic
      - name: Switch to Green
        run: |
          kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

      # Verify live traffic
      - name: Verify production
        run: |
          sleep 10
          curl -f https://myapp.example.com/health

      # Rollback on failure
      - name: Rollback
        if: failure()
        run: |
          kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
```

---

## Blue-Green vs Rolling

```
  ┌─────────────────┬────────────────────┬────────────────────┐
  │  Aspect         │  Rolling           │  Blue-Green        │
  ├─────────────────┼────────────────────┼────────────────────┤
  │  Infra cost     │  Low (same infra)  │  High (2x infra)   │
  │  Rollback speed │  Minutes (redeploy)│  Seconds (switch)  │
  │  Version mixing │  Yes (during roll) │  No (all-at-once)  │
  │  Testing before │  No (tested in     │  Yes (test Green   │
  │  live traffic   │  production)       │  before switching) │
  │  Complexity     │  Low               │  Medium            │
  │  DB migrations  │  Must be backward  │  Must be backward  │
  │                 │  compatible        │  compatible         │
  └─────────────────┴────────────────────┴────────────────────┘
```

---

## Database Considerations

```
  DATABASE CHALLENGE
  ══════════════════

  ✕ PROBLEM: Both Blue and Green share the same database
             Schema changes can break the inactive environment

  ✓ SOLUTION: Expand-and-Contract Migration Pattern

  Phase 1: EXPAND (backward compatible)
  ┌──────────┐     ┌──────────────────────────┐
  │ Blue v1  │────►│ DB: old_col + new_col    │
  │          │     │ (both columns exist)      │
  └──────────┘     └──────────────────────────┘
  ┌──────────┐────►│ Green v2 writes to both  │
  │ Green v2 │     └──────────────────────────┘
  └──────────┘

  Phase 2: CONTRACT (after v1 is decommissioned)
  ┌──────────┐     ┌──────────────────────────┐
  │ Green v2 │────►│ DB: new_col only         │
  │ (live)   │     │ (old_col dropped)        │
  └──────────┘     └──────────────────────────┘
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Double infrastructure cost | Both environments running | Scale down idle env; use spot instances |
| Database compatibility | Schema differences between v1/v2 | Use expand-and-contract migrations |
| DNS propagation delay | DNS TTL too high | Use load balancer switching, not DNS |
| Stale connections after switch | Long-lived connections to old env | Implement connection draining; use health checks |
| Green smoke tests unreliable | Different traffic patterns | Replay production traffic to Green for testing |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **How it works** | Two identical envs; switch traffic between them |
| **Downtime** | Zero |
| **Rollback** | Instant — switch load balancer back |
| **Cost** | Double infrastructure during deployment |
| **Risk** | Very low — test before switching |
| **Best for** | Critical apps needing instant rollback |

---

## Quick Revision Questions

1. **What is a Blue-Green deployment?** *Maintaining two identical environments — one live (Blue), one idle (Green). Deploy to Green, test it, then switch traffic.*
2. **How fast is rollback in Blue-Green?** *Instant — seconds to switch the load balancer back to the previous environment.*
3. **What is the main downside of Blue-Green?** *Double infrastructure cost — both environments must be running during the deployment.*
4. **How is traffic switched in Kubernetes Blue-Green?** *By patching the Service selector to point to the new deployment's labels.*
5. **What is the expand-and-contract pattern?** *A database migration strategy: first add new columns alongside old ones (expand), then remove old columns after all apps are updated (contract).*
6. **When is Blue-Green preferred over Rolling?** *When you need instant rollback, when you want to test the new version before it receives any live traffic, or when version mixing is unacceptable.*

---

[← Previous: Rolling Deployments](01-rolling-deployments.md) | [Back to README](../README.md) | [Next: Canary Deployments →](03-canary-deployments.md)
