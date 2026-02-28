# Chapter 3.2 — Continuous Delivery (CD)

[← Previous: Continuous Integration](01-continuous-integration.md) | [Next: Continuous Deployment →](03-continuous-deployment.md)

---

## Overview

**Continuous Delivery (CD)** extends CI by ensuring that code is always in a **deployable state**. Every change that passes CI can be released to production at any time with a single manual approval. The key distinction: deployment to production requires a **human decision**, but the process itself is fully automated.

---

## CI vs CD: The Relationship

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Continuous Integration (CI)                             │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐                │
│  │Commit│─►│Build │─►│Test  │─►│Artifact              │
│  └──────┘  └──────┘  └──────┘  └──────┘                │
│                                    │                     │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─    │
│                                    │                     │
│  Continuous Delivery (CD)          ▼                     │
│                              ┌──────────┐                │
│                              │ Deploy to│                │
│                              │ Staging  │                │
│                              └────┬─────┘                │
│                                   │                      │
│                              ┌────▼─────┐                │
│                              │Acceptance│                │
│                              │  Tests   │                │
│                              └────┬─────┘                │
│                                   │                      │
│                              ┌────▼─────┐                │
│                              │ MANUAL   │ ◄── Human      │
│                              │ APPROVAL │     decision    │
│                              └────┬─────┘                │
│                                   │                      │
│                              ┌────▼─────┐                │
│                              │Deploy to │                │
│                              │Production│                │
│                              └──────────┘                │
└──────────────────────────────────────────────────────────┘
```

---

## The Deployment Pipeline

```
┌──────────────────────────────────────────────────────────┐
│  CONTINUOUS DELIVERY PIPELINE                            │
│                                                          │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐        │
│  │ COMMIT │  │ BUILD  │  │  TEST  │  │ARTIFACT│        │
│  │ STAGE  │  │ STAGE  │  │ STAGE  │  │ STAGE  │        │
│  ├────────┤  ├────────┤  ├────────┤  ├────────┤        │
│  │Checkout│  │Compile │  │Unit    │  │Docker  │        │
│  │Lint    │  │Package │  │Integr. │  │Image   │        │
│  │Format  │  │Docker  │  │E2E     │  │Helm    │        │
│  │        │  │Build   │  │Security│  │Chart   │        │
│  └───┬────┘  └───┬────┘  └───┬────┘  └───┬────┘        │
│      │           │           │           │              │
│      ▼           ▼           ▼           ▼              │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐        │
│  │STAGING │  │ SMOKE  │  │  UAT   │  │  PROD  │        │
│  │ DEPLOY │  │ TESTS  │  │APPROVAL│  │ DEPLOY │        │
│  ├────────┤  ├────────┤  ├────────┤  ├────────┤        │
│  │Auto-   │  │Health  │  │Manual  │  │Canary/ │        │
│  │deploy  │  │checks  │  │review  │  │Rolling │        │
│  │to stg  │  │API test│  │Sign-off│  │Blue-grn│        │
│  └────────┘  └────────┘  └────────┘  └────────┘        │
│                                                          │
│  ◄──── Automated ────────────────► ◄─ Manual ─► ◄Auto─► │
└──────────────────────────────────────────────────────────┘
```

---

## Key Practices for Continuous Delivery

### 1. Environment Parity

```
┌──────────────────────────────────────────────────────────┐
│  ENVIRONMENT PARITY                                      │
│                                                          │
│  Dev          Staging        Production                  │
│  ┌──────┐    ┌──────┐       ┌──────┐                    │
│  │Docker│    │Docker│       │Docker│                    │
│  │v20.x │    │v20.x │       │v20.x │  ✅ Same versions  │
│  ├──────┤    ├──────┤       ├──────┤                    │
│  │PG 16 │    │PG 16 │       │PG 16 │  ✅ Same database  │
│  ├──────┤    ├──────┤       ├──────┤                    │
│  │Redis7│    │Redis7│       │Redis7│  ✅ Same cache      │
│  ├──────┤    ├──────┤       ├──────┤                    │
│  │Node20│    │Node20│       │Node20│  ✅ Same runtime    │
│  └──────┘    └──────┘       └──────┘                    │
│                                                          │
│  ⚠️ Differences across envs = "Works in staging,        │
│      breaks in prod" — the #1 source of deploy failures │
└──────────────────────────────────────────────────────────┘
```

### 2. Artifact Promotion (Build Once, Deploy Everywhere)

```
  ❌ BAD: Build separately for each environment
  Code ──► Build (dev) ──► Build (stg) ──► Build (prod)
           (different!)    (different!)    (different!)

  ✅ GOOD: Build once, promote the same artifact
  Code ──► Build ──► Artifact ──► Dev ──► Staging ──► Prod
                     (one          (same    (same      (same
                      image)       image)   image)     image)
```

```bash
# Build once
docker build -t myapp:v2.3.0-abc123 .

# Push to registry
docker push registry.example.com/myapp:v2.3.0-abc123

# Deploy SAME image to each environment
# (only configuration changes, not the artifact)
kubectl set image deploy/myapp myapp=registry.example.com/myapp:v2.3.0-abc123 -n dev
kubectl set image deploy/myapp myapp=registry.example.com/myapp:v2.3.0-abc123 -n staging
kubectl set image deploy/myapp myapp=registry.example.com/myapp:v2.3.0-abc123 -n production
```

### 3. Automated Acceptance Tests

```yaml
# Automated acceptance tests in staging
acceptance-tests:
  stage: test
  script:
    # API tests
    - newman run postman/collection.json \
        --environment postman/staging.json \
        --reporters cli,junit
    
    # E2E UI tests
    - npx cypress run \
        --config baseUrl=https://staging.example.com
    
    # Performance baseline
    - k6 run --vus 50 --duration 5m load-test.js
    
    # Contract tests
    - pact-verifier --provider-base-url=https://staging.example.com
```

### 4. Configuration Management

```
┌──────────────────────────────────────────────────────────┐
│  EXTERNALIZED CONFIGURATION                              │
│                                                          │
│  ❌ BAD: Config baked into artifact                      │
│  app.jar contains: DB_HOST=prod-db.internal              │
│                                                          │
│  ✅ GOOD: Config injected at runtime                     │
│                                                          │
│  ┌──────────┐    Environment Variables:                  │
│  │          │    DEV:  DB_HOST=dev-db.internal            │
│  │  Same    │    STG:  DB_HOST=stg-db.internal            │
│  │  app.jar │    PROD: DB_HOST=prod-db.internal           │
│  │          │                                            │
│  └──────────┘    12-Factor App: "Store config in the     │
│                  environment"                             │
└──────────────────────────────────────────────────────────┘
```

```yaml
# Kubernetes ConfigMap for environment-specific config
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  DATABASE_HOST: "prod-db.internal"
  CACHE_HOST: "prod-redis.internal"
  LOG_LEVEL: "warn"
  FEATURE_NEW_UI: "true"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: production
type: Opaque
data:
  DATABASE_PASSWORD: cHJvZC1wYXNzd29yZA==  # base64 encoded
  API_KEY: c2VjcmV0LWtleQ==
```

---

## Real-World Scenario: CD Pipeline for a Microservice

### 🏢 Scenario: Payment Service Delivery Pipeline

```
┌──────────────────────────────────────────────────────────┐
│  PAYMENT SERVICE CD PIPELINE                             │
│                                                          │
│  1. Developer merges PR to main                          │
│     │                                                    │
│  2. ▼ CI runs (build, test, scan)            [5 min]     │
│     │                                                    │
│  3. ▼ Docker image pushed to ECR             [1 min]     │
│     │  (tagged: payment-svc:v3.2.1-sha)                  │
│  4. ▼ Auto-deploy to DEV environment         [2 min]     │
│     │  (developer verifies)                              │
│  5. ▼ Auto-deploy to STAGING                 [2 min]     │
│     │                                                    │
│  6. ▼ Automated acceptance tests             [8 min]     │
│     │  ├── Payment flow E2E test                         │
│     │  ├── Refund flow test                              │
│     │  ├── PCI compliance check                          │
│     │  └── Performance baseline (p99 < 200ms)            │
│     │                                                    │
│  7. ▼ Slack notification to team:                        │
│     │  "payment-svc v3.2.1 ready for prod ✅"            │
│     │                                                    │
│  8. ▼ [MANUAL APPROVAL] Tech Lead clicks "Deploy"        │
│     │                                                    │
│  9. ▼ Canary deploy to 5% prod traffic       [2 min]     │
│     │  Monitor error rate for 15 min                     │
│  10.▼ Progressive rollout: 25% → 50% → 100%  [30 min]   │
│     │                                                    │
│  11.✅ Production verified                               │
│                                                          │
│  Total time: ~1 hour (mostly automated)                  │
└──────────────────────────────────────────────────────────┘
```

---

## Deployment Strategies in Detail

```
┌──────────────────────────────────────────────────────────┐
│  STRATEGY COMPARISON                                     │
│                                                          │
│  ROLLING UPDATE:  Replace instances one at a time        │
│  ┌───┬───┬───┬───┐  ┌───┬───┬───┬───┐                  │
│  │v1 │v1 │v1 │v1 │─►│v2 │v1 │v1 │v1 │─►... ─► all v2  │
│  └───┴───┴───┴───┘  └───┴───┴───┴───┘                  │
│  ✅ Zero downtime   ⚠️ Mixed versions during rollout     │
│                                                          │
│  BLUE-GREEN:  Switch all traffic at once                 │
│  ┌─────────┐  ┌─────────┐                               │
│  │ BLUE v1 │  │GREEN v2 │   LB switches from Blue→Green │
│  │ (active)│  │(standby)│                                │
│  └─────────┘  └─────────┘                               │
│  ✅ Instant rollback   ⚠️ Needs 2x resources            │
│                                                          │
│  CANARY:  Send small % of traffic to new version         │
│  ┌──────────────────┐                                    │
│  │ 95% ──► v1       │   If canary healthy → expand      │
│  │  5% ──► v2       │   If canary fails  → abort        │
│  └──────────────────┘                                    │
│  ✅ Safest approach   ⚠️ Needs traffic splitting         │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Works in staging, fails in prod" | Environment differences | Enforce parity (same images, same configs, IaC) |
| Deployment takes too long | Large artifacts, slow rollout | Optimize image size, use parallel rollouts |
| Rollback is manual and slow | No automated rollback process | Implement canary with auto-rollback on metrics |
| Config mismatch | Hardcoded values in code | Externalize all config via env vars / ConfigMaps |
| Manual approval bottleneck | Approver is unavailable | Define backup approvers, consider auto-approval for low-risk |
| Acceptance tests are flaky | External dependencies in tests | Mock external services, use test containers |

---

## Summary Table

| CD Concept | Description |
|-----------|-------------|
| **Core Idea** | Code is always deployable; production deploy is a business decision |
| **vs CI** | CI = build + test; CD = CI + deploy to staging + ready for prod |
| **Artifact Promotion** | Build once, deploy same artifact everywhere |
| **Environment Parity** | Dev/Staging/Prod should be as identical as possible |
| **Configuration** | Externalized via env vars, ConfigMaps, secrets |
| **Approval** | Manual gate before production (business decision) |
| **Deploy Strategies** | Rolling, Blue-Green, Canary |

---

## Quick Revision Questions

1. **What is the key difference between CI and CD (Continuous Delivery)?**
   <details><summary>Answer</summary>CI ensures code is automatically built and tested on every commit. CD extends CI by also deploying to staging, running acceptance tests, and making the software ready for production deployment at any time. The key distinction is that CD requires a manual approval before production, but everything else is automated.</details>

2. **What does "Build Once, Deploy Everywhere" mean?**
   <details><summary>Answer</summary>It means you build a single artifact (e.g., Docker image) once, and deploy that exact same artifact to dev, staging, and production. Only the configuration changes per environment (via env vars, ConfigMaps). This ensures consistency — what you tested is exactly what runs in production.</details>

3. **Why is environment parity important for Continuous Delivery?**
   <details><summary>Answer</summary>Environment parity ensures that dev, staging, and production are as identical as possible. Without parity, bugs may appear only in production (different OS version, different database version, different network config), defeating the purpose of testing in staging.</details>

4. **Compare Blue-Green and Canary deployment strategies.**
   <details><summary>Answer</summary>Blue-Green: maintain two identical environments; switch all traffic at once. Instant rollback but needs 2x resources. Canary: send a small % of traffic to the new version; gradually increase if healthy. Safer (limits blast radius) but requires traffic splitting and metric analysis.</details>

5. **What types of tests should run in the CD pipeline acceptance stage?**
   <details><summary>Answer</summary>1) End-to-end tests (critical user flows). 2) API contract tests (ensure API compatibility). 3) Performance baseline tests (latency within SLO). 4) Security/compliance checks. 5) Smoke tests (basic health after deployment). These validate the release is production-ready.</details>

6. **When should a manual approval gate be used vs fully automated deployment?**
   <details><summary>Answer</summary>Manual approval is used in Continuous Delivery when: regulatory requirements demand it, the change is high-risk, or the organization isn't mature enough for full automation. Fully automated (Continuous Deployment) is used when: comprehensive automated tests exist, canary analysis is reliable, and auto-rollback is in place.</details>

---

[← Previous: Continuous Integration](01-continuous-integration.md) | [Next: Continuous Deployment →](03-continuous-deployment.md)
