# Chapter 2.1: Pipeline Stages (Build, Test, Deploy)

[← Previous: Pipeline Components](../01-CICD-Concepts/05-pipeline-components.md) | [Back to README](../README.md) | [Next: Pipeline as Code →](02-pipeline-as-code.md)

---

## Overview

A CI/CD pipeline is organized into **stages** — logical groupings of tasks that run sequentially. The three foundational stages are **Build**, **Test**, and **Deploy**. Real-world pipelines extend these with additional stages for quality, security, and release management.

---

## Fundamental Three Stages

```
┌──────────────────────────────────────────────────────────────────┐
│                  THE THREE CORE STAGES                            │
│                                                                   │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │
│  │              │    │              │    │              │        │
│  │    BUILD     │───▶│    TEST      │───▶│   DEPLOY     │        │
│  │              │    │              │    │              │        │
│  └──────────────┘    └──────────────┘    └──────────────┘        │
│                                                                   │
│  "Can it compile?"   "Does it work?"    "Ship it!"               │
│                                                                   │
│  • Fetch code        • Unit tests       • Push artifact          │
│  • Install deps      • Integration      • Deploy to env          │
│  • Compile           • Code quality      • Smoke tests           │
│  • Package           • Security scan    • Health checks          │
└──────────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Build

```
┌─────────────────────────────────────────────────────────────┐
│                      BUILD STAGE                             │
│                                                              │
│  Input: Source Code                                          │
│  Output: Deployable Artifact                                 │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Checkout │─▶│ Install  │─▶│ Compile  │─▶│ Package  │   │
│  │   Code   │  │  Deps    │  │  /Build  │  │ Artifact │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                              │
│  Duration: 1-5 minutes (target)                              │
└─────────────────────────────────────────────────────────────┘
```

### Build Stage Tasks

```yaml
# Example: Node.js build stage
build:
  steps:
    - checkout: git clone / actions/checkout
    - install:  npm ci                    # Deterministic install
    - lint:     npm run lint              # Code style check
    - compile:  npm run build             # TypeScript → JavaScript
    - package:  docker build -t app:v1 .  # Create container image
    - publish:  docker push registry/app:v1  # Store artifact
```

### Build Stage for Different Languages

| Language | Install Deps | Compile | Package |
|---|---|---|---|
| Java | `mvn dependency:resolve` | `mvn compile` | `mvn package` → `.jar/.war` |
| Node.js | `npm ci` | `npm run build` | `docker build` or `npm pack` |
| Python | `pip install -r requirements.txt` | N/A (interpreted) | `docker build` or `wheel` |
| Go | `go mod download` | `go build` | Binary / Docker image |
| .NET | `dotnet restore` | `dotnet build` | `dotnet publish` |

---

## Stage 2: Test

```
┌─────────────────────────────────────────────────────────────────┐
│                       TEST STAGE                                 │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    TEST PYRAMID                            │  │
│  │                                                            │  │
│  │                      /\                                    │  │
│  │                     /  \        E2E Tests (few, slow)      │  │
│  │                    /    \                                   │  │
│  │                   / E2E  \                                 │  │
│  │                  /────────\                                │  │
│  │                 /          \     Integration (moderate)     │  │
│  │                / Integration\                              │  │
│  │               /──────────────\                             │  │
│  │              /                \   Unit Tests (many, fast)  │  │
│  │             /   Unit Tests     \                           │  │
│  │            /____________________\                          │  │
│  │                                                            │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Execution order: Unit → Integration → E2E (fast → slow)        │
│  Duration: 5-30 minutes (target for full suite)                  │
└─────────────────────────────────────────────────────────────────┘
```

### Test Stage Tasks

```yaml
test:
  parallel:
    - unit-tests:
        run: npm test -- --coverage
        coverage-threshold: 80%
    
    - integration-tests:
        services: [postgres, redis]
        run: npm run test:integration
    
    - security-scan:
        run: npm audit --audit-level=high
    
    - code-quality:
        run: sonar-scanner
```

---

## Stage 3: Deploy

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DEPLOY STAGE                                    │
│                                                                      │
│  ┌─────────┐   ┌─────────┐   ┌──────────┐   ┌──────────────────┐  │
│  │ Pull    │──▶│ Deploy  │──▶│  Smoke   │──▶│ Health Check /   │  │
│  │Artifact │   │ to Env  │   │  Tests   │   │ Monitor          │  │
│  └─────────┘   └─────────┘   └──────────┘   └──────────────────┘  │
│                                                                      │
│  Environments: Dev → QA → Staging → Production                      │
│  Duration: 5-15 minutes per environment                              │
└─────────────────────────────────────────────────────────────────────┘
```

### Deploy Stage Tasks

```yaml
deploy-staging:
  needs: [test]
  steps:
    - download-artifact: myapp:$SHA
    - deploy:
        target: staging
        strategy: rolling
    - smoke-test:
        run: curl -f https://staging.example.com/health
    - notify:
        channel: "#deployments"
        message: "Deployed $SHA to staging ✅"

deploy-production:
  needs: [deploy-staging]
  approval: manual          # Gate for Continuous Delivery
  steps:
    - deploy:
        target: production
        strategy: canary
        canary-percentage: 10
    - monitor:
        duration: 15m
        rollback-on: error_rate > 1%
```

---

## Extended Pipeline Stages

Real-world pipelines add more stages:

```
┌──────────────────────────────────────────────────────────────────────────┐
│              PRODUCTION-GRADE PIPELINE                                    │
│                                                                           │
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌──────┐│
│  │SOURCE │▶│ BUILD │▶│ TEST  │▶│QUALITY│▶│SECURIT│▶│STAGING│▶│ PROD ││
│  │       │ │       │ │       │ │ GATE  │ │Y SCAN │ │       │ │      ││
│  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └──────┘│
│                                                                           │
│  Source:   Checkout, validate commit                                     │
│  Build:    Compile, package, create artifact                             │
│  Test:     Unit + Integration + E2E                                      │
│  Quality:  SonarQube, code coverage gates                                │
│  Security: SAST, DAST, dependency scan                                   │
│  Staging:  Deploy, smoke test, UAT                                       │
│  Prod:     Canary → Full rollout → Monitor                               │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Stage Dependencies & Flow Control

```
Sequential Stages:                Parallel Jobs within Stages:
═══════════════════               ═══════════════════════════

  Build ──▶ Test ──▶ Deploy        Test Stage:
    │         │         │           ┌──────────────────────┐
    │         │         │           │  ┌──────┐  ┌──────┐ │
    │         │         │           │  │ Unit │  │ Lint │ │
  Must       Must      Must        │  │ Test │  │      │ │  ← Run in
  pass       pass      pass        │  └──┬───┘  └──┬───┘ │    parallel
                                   │     │         │     │
                                   │     └────┬────┘     │
                                   │          ▼          │
                                   │    ┌──────────┐    │
                                   │    │Integration│    │  ← Runs after
                                   │    │   Test    │    │    both pass
                                   │    └──────────┘    │
                                   └──────────────────────┘
```

### Conditional Stages

```yaml
# Run different stages based on conditions
stages:
  - build        # Always runs
  - test         # Always runs
  - deploy-dev   # Only on feature branches
  - deploy-stage # Only on main branch
  - deploy-prod  # Only on tagged releases

# GitLab CI example
deploy-production:
  stage: deploy-prod
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual
```

---

## Real-World Pipeline Example

### Full Pipeline (GitHub Actions)

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # ═══ BUILD STAGE ═══
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/ }

  # ═══ TEST STAGE (parallel) ═══
  unit-test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --coverage

  lint:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint

  security:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  # ═══ DEPLOY STAGING ═══
  deploy-staging:
    needs: [unit-test, lint, security]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist }
      - run: ./scripts/deploy.sh staging

  # ═══ DEPLOY PRODUCTION ═══
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist }
      - run: ./scripts/deploy.sh production
```

---

## Troubleshooting Pipeline Stages

| Issue | Stage | Solution |
|---|---|---|
| Build fails intermittently | Build | Pin dependency versions, use lockfiles |
| Tests pass locally, fail in CI | Test | Match CI environment (Docker), check env vars |
| Deploy times out | Deploy | Increase timeout, check target env health |
| Stage runs when it shouldn't | Any | Review conditional rules/filters |
| Pipeline too slow | All | Parallelize independent jobs, add caching |
| Artifact not found | Deploy | Check artifact upload/download names match |

---

## Summary Table

| Stage | Purpose | Typical Duration | Key Output |
|---|---|---|---|
| **Source** | Checkout code, validate | Seconds | Working directory |
| **Build** | Compile, package | 1-5 min | Artifact (JAR, image, bundle) |
| **Test** | Verify correctness | 5-30 min | Test reports, coverage |
| **Quality Gate** | Enforce standards | 1-5 min | Pass/fail decision |
| **Security** | Vulnerability scan | 2-10 min | Security report |
| **Deploy Staging** | Pre-prod validation | 5-15 min | Running staging env |
| **Deploy Prod** | Production release | 5-15 min | Live application |
| **Monitor** | Post-deploy validation | Ongoing | Metrics & alerts |

---

## Quick Revision Questions

1. **What are the three fundamental CI/CD pipeline stages?**  
   *Build, Test, Deploy — covering compilation, verification, and release.*

2. **Why should pipeline stages run in a specific order?**  
   *Each stage validates the output of the previous one. There's no point deploying untested code or testing unbuilt code.*

3. **How can you speed up the Test stage?**  
   *Run independent test suites in parallel (unit, lint, security concurrently), cache dependencies, and follow the test pyramid.*

4. **What is a quality gate and where does it fit in the pipeline?**  
   *A quality gate enforces minimum standards (coverage, code quality score). It typically runs after testing and before deployment.*

5. **What are conditional stages and why are they useful?**  
   *Stages that only run under certain conditions (e.g., deploy-prod only on tags). They prevent unnecessary work and control the promotion flow.*

6. **What should happen immediately after a deployment stage?**  
   *Smoke tests and health checks to verify the deployment succeeded, followed by monitoring for anomalies.*

---

[← Previous: Pipeline Components](../01-CICD-Concepts/05-pipeline-components.md) | [Back to README](../README.md) | [Next: Pipeline as Code →](02-pipeline-as-code.md)
