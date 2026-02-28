# Chapter 3.3 — Continuous Deployment

[← Previous: Continuous Delivery](02-continuous-delivery.md) | [Next: Infrastructure as Code →](04-infrastructure-as-code.md)

---

## Overview

**Continuous Deployment** takes Continuous Delivery one step further: every change that passes all automated tests is **automatically deployed to production** — with no manual approval gate. This is the most advanced stage of the CI/CD maturity model.

---

## CI → CD (Delivery) → CD (Deployment)

```
┌──────────────────────────────────────────────────────────┐
│  THE CI/CD SPECTRUM                                      │
│                                                          │
│  Continuous Integration:                                 │
│  Commit ──► Build ──► Test ──► [Done]                    │
│  "Code is always integrated"                             │
│                                                          │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─             │
│                                                          │
│  Continuous Delivery:                                    │
│  Commit ──► Build ──► Test ──► Stage ──► [APPROVE] ──► Prod
│  "Code is always deployable"      ▲                      │
│                                   │ Human decision       │
│                                                          │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─             │
│                                                          │
│  Continuous Deployment:                                  │
│  Commit ──► Build ──► Test ──► Stage ──► Prod            │
│  "Code is always deployed"  (NO human gate!)             │
│                                                          │
│  ┌────────────────────────────────────────────┐          │
│  │  Maturity:  CI   <   CD    <   CD          │          │
│  │                    (Delivery)  (Deployment) │          │
│  └────────────────────────────────────────────┘          │
└──────────────────────────────────────────────────────────┘
```

---

## Prerequisites for Continuous Deployment

You need these foundations before going fully automated:

```
┌──────────────────────────────────────────────────────────┐
│  PREREQUISITES CHECKLIST                                 │
│                                                          │
│  ┌──────────────────────────────────────────────┐       │
│  │  ✅ Comprehensive automated test suite       │       │
│  │     (unit, integration, E2E, security)       │       │
│  │                                               │       │
│  │  ✅ Feature flags for incomplete features     │       │
│  │     (deploy code != release feature)          │       │
│  │                                               │       │
│  │  ✅ Canary / progressive rollout strategy     │       │
│  │     (limit blast radius)                      │       │
│  │                                               │       │
│  │  ✅ Automated rollback on failure              │       │
│  │     (metrics-driven, not human-driven)        │       │
│  │                                               │       │
│  │  ✅ Real-time monitoring and alerting          │       │
│  │     (detect issues in seconds)                │       │
│  │                                               │       │
│  │  ✅ High team confidence and culture           │       │
│  │     (blameless postmortems, trust in tests)   │       │
│  └──────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────┘
```

---

## Deploy vs Release: A Critical Distinction

```
┌──────────────────────────────────────────────────────────┐
│  DEPLOY ≠ RELEASE                                        │
│                                                          │
│  DEPLOY: Code runs in production                         │
│  RELEASE: Feature is available to users                  │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │                                                 │    │
│  │  Commit ──► Tests ──► Deploy to Prod            │    │
│  │                           │                     │    │
│  │                    ┌──────┴──────┐               │    │
│  │                    │ Feature Flag│               │    │
│  │                    ├─────────────┤               │    │
│  │                    │ OFF: Users  │               │    │
│  │                    │ see old UI  │               │    │
│  │                    │             │               │    │
│  │                    │ ON: Users   │               │    │
│  │                    │ see new UI  │               │    │
│  │                    └─────────────┘               │    │
│  │                                                 │    │
│  │  Code is DEPLOYED (in prod, running)            │    │
│  │  Feature is NOT RELEASED (flag is OFF)          │    │
│  │  Release happens when flag is turned ON         │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  This separation enables:                                │
│  ├── Deploy anytime (safe, code is hidden)               │
│  ├── Release when ready (product decision)               │
│  ├── Instant rollback (toggle flag off)                  │
│  └── A/B testing (flag on for 10% of users)              │
└──────────────────────────────────────────────────────────┘
```

---

## Continuous Deployment Pipeline

```yaml
# Complete GitHub Actions Continuous Deployment pipeline
name: Continuous Deployment

on:
  push:
    branches: [main]

jobs:
  # Stage 1: Build & Test
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: docker build -t myapp:${{ github.sha }} .
      - name: Unit Tests
        run: docker run myapp:${{ github.sha }} npm test
      - name: Integration Tests
        run: docker-compose -f docker-compose.test.yml up --abort-on-container-exit
      - name: Security Scan
        run: trivy image --severity HIGH,CRITICAL myapp:${{ github.sha }}
      - name: Push Image
        run: |
          docker tag myapp:${{ github.sha }} ecr.aws/myapp:${{ github.sha }}
          docker push ecr.aws/myapp:${{ github.sha }}

  # Stage 2: Deploy to Staging (automatic)
  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: kubectl set image deploy/myapp myapp=ecr.aws/myapp:${{ github.sha }} -n staging
      - name: Run smoke tests
        run: ./scripts/smoke-test.sh https://staging.example.com
      - name: Run E2E tests
        run: npx cypress run --config baseUrl=https://staging.example.com

  # Stage 3: Deploy to Production (automatic - NO manual gate!)
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: Canary deploy (5%)
        run: |
          kubectl argo rollouts set-image myapp myapp=ecr.aws/myapp:${{ github.sha }}
          kubectl argo rollouts promote myapp --full=false

      # ArgoCD Rollouts handles progressive rollout
      # and auto-rollback based on Prometheus metrics
```

---

## Who Uses Continuous Deployment?

| Company | Deployment Frequency | Key Enablers |
|---------|---------------------|-------------|
| **Amazon** | ~136,000 deploys/day (across all services) | Microservices, feature flags, automated testing |
| **Netflix** | Thousands/day | Spinnaker, canary analysis, Chaos Engineering |
| **Google** | Continuous (monorepo) | Extensive testing, feature flags, gradual rollouts |
| **Etsy** | 50+/day | Comprehensive monitoring, feature flags, ChatOps |
| **GitHub** | Hundreds/day | Feature flags, canary, scientist gem (A/B testing) |

---

## When NOT to Use Continuous Deployment

```
┌──────────────────────────────────────────────────────────┐
│  CONTINUOUS DEPLOYMENT IS NOT ALWAYS APPROPRIATE         │
│                                                          │
│  Use Continuous DELIVERY (manual gate) when:             │
│                                                          │
│  ├── 🏥 Regulated industries (healthcare, finance)       │
│  │      → Compliance requires manual sign-off            │
│  │                                                       │
│  ├── 🚀 Early-stage startups (limited testing)           │
│  │      → Test suite not comprehensive enough yet        │
│  │                                                       │
│  ├── 🏭 Hardware/embedded systems                        │
│  │      → Cannot easily rollback                         │
│  │                                                       │
│  ├── 📱 Mobile apps                                      │
│  │      → App store review required                      │
│  │                                                       │
│  └── ⚠️ Low confidence in automated tests               │
│         → Manual verification still needed               │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: From Delivery to Deployment

### 🏢 Scenario: SaaS Company Evolution

```
PHASE 1: Continuous Delivery (Month 1-6)
├── CI pipeline: build + test on every commit
├── Auto-deploy to staging after CI pass
├── Manual approval for production
├── Deploy frequency: 2x per week
├── Team building confidence in tests
└── Learning: "Our tests catch 95% of issues"

PHASE 2: Semi-Automated (Month 7-12)
├── Added canary deployments with auto-rollback
├── Added feature flags (LaunchDarkly)
├── Auto-deploy to prod for "low-risk" services
├── Manual approval only for "high-risk" changes
├── Deploy frequency: 3-5x per day
└── Learning: "Canary catches the other 5%"

PHASE 3: Continuous Deployment (Month 13+)
├── ALL changes auto-deploy to production
├── 100% of features behind feature flags
├── Canary analysis on every deploy
├── Auto-rollback if error rate > 1%
├── Deploy frequency: 20+ per day
└── "We deploy and forget — the pipeline handles it"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Fear of auto-deploying | Lack of test confidence | Increase test coverage, start with low-risk services |
| No rollback mechanism | Missing monitoring/automation | Implement canary with Prometheus metrics + auto-rollback |
| Feature partially done | Incomplete feature deployed | Always use feature flags for WIP features |
| Database migration breaks | Schema change not backward-compatible | Use expand-and-contract migration pattern |
| Deploy too many changes at once | Commits batch up | Ensure pipeline runs on EVERY commit, not batches |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Continuous Deployment** | Every passing change auto-deploys to production |
| **vs Continuous Delivery** | Delivery has manual gate; Deployment is fully automated |
| **Deploy ≠ Release** | Deploy = code in prod; Release = feature available to users |
| **Feature Flags** | Essential for separating deploy from release |
| **Prerequisites** | Tests, monitoring, rollback, feature flags, culture |
| **Best For** | SaaS, web apps, microservices with mature testing |
| **Not Ideal For** | Regulated industries, mobile apps, embedded systems |

---

## Quick Revision Questions

1. **What is the difference between Continuous Delivery and Continuous Deployment?**
   <details><summary>Answer</summary>Continuous Delivery means code is always deployable with a manual approval gate before production. Continuous Deployment removes the manual gate entirely — every change that passes automated tests is automatically deployed to production.</details>

2. **Why is the distinction between "deploy" and "release" important?**
   <details><summary>Answer</summary>Separating deploy from release (via feature flags) allows code to be deployed safely without exposing incomplete or risky features to users. The deploy is a technical act (code running in prod), while release is a business decision (feature visible to users). This enables continuous deployment without constant user-visible changes.</details>

3. **What are the key prerequisites for Continuous Deployment?**
   <details><summary>Answer</summary>1) Comprehensive automated test suite. 2) Feature flags for all new features. 3) Canary/progressive rollout strategy. 4) Automated rollback on metric degradation. 5) Real-time monitoring and alerting. 6) High team confidence and blameless culture.</details>

4. **Name three companies that practice Continuous Deployment and their frequency.**
   <details><summary>Answer</summary>Amazon: ~136,000 deploys/day across services. Netflix: thousands/day using Spinnaker and canary analysis. GitHub: hundreds/day using feature flags and scientist gem for experimentation.</details>

5. **When should you NOT use Continuous Deployment?**
   <details><summary>Answer</summary>Regulated industries requiring manual sign-off, mobile apps needing app store review, embedded/hardware systems that can't easily rollback, early-stage teams without comprehensive test suites, and situations where automated test confidence is low.</details>

6. **What is the expand-and-contract migration pattern for databases?**
   <details><summary>Answer</summary>A backward-compatible database migration approach: 1) EXPAND: Add new column/table without removing old ones. 2) MIGRATE: Update code to write to both old and new. 3) BACKFILL: Migrate existing data to new schema. 4) CONTRACT: Remove old column/table after all code uses new schema. This prevents breaking deployments during schema changes.</details>

---

[← Previous: Continuous Delivery](02-continuous-delivery.md) | [Next: Infrastructure as Code →](04-infrastructure-as-code.md)
