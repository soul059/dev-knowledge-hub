# Chapter 1.2: What is Continuous Delivery?

[← Previous: Continuous Integration](01-continuous-integration.md) | [Back to README](../README.md) | [Next: Continuous Deployment →](03-continuous-deployment.md)

---

## Overview

**Continuous Delivery (CD)** is the practice of keeping your codebase in a **deployable state at all times**. It extends CI by adding automated deployment pipelines that can release any version to any environment at the push of a button. The key distinction: **deployment to production requires manual approval**.

> 💡 **Core Idea:** "Software can be released to production at any time, with confidence."

---

## CI vs Continuous Delivery

```
CONTINUOUS INTEGRATION (CI)
════════════════════════════════════════════════════════════

  Code → Build → Unit Test → ✅ Done
  
  Scope: Merge code & verify it compiles and passes tests


CONTINUOUS DELIVERY (CD)
════════════════════════════════════════════════════════════

  Code → Build → Unit Test → Integration Test → Stage → ✅ Ready
                                                           │
                                                    ┌──────┴──────┐
                                                    │   MANUAL     │
                                                    │   APPROVAL   │
                                                    │   GATE       │
                                                    └──────┬──────┘
                                                           │
                                                           ▼
                                                      Production

  Scope: Full pipeline from commit to production-ready artifact
```

---

## Continuous Delivery Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                  CONTINUOUS DELIVERY PIPELINE                            │
│                                                                          │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐           │
│  │ SOURCE │─▶│ BUILD  │─▶│  TEST  │─▶│ STAGE  │─▶│RELEASE │           │
│  │        │  │        │  │        │  │        │  │        │           │
│  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘           │
│     git        compile     unit       deploy to    ┌──────────┐        │
│     push       package     integr.    staging      │ MANUAL   │        │
│                artifact    e2e        UAT          │ APPROVAL │        │
│                            perf.      smoke test   └────┬─────┘        │
│                                                         │              │
│                      AUTOMATED ◄─────────────────►      ▼              │
│                                                    ┌──────────┐        │
│                                                    │PRODUCTION│        │
│                                                    │  DEPLOY  │        │
│                                                    └──────────┘        │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Key Principles

### 1. Every Change is Releasable
The mainline branch should **always** be in a state that can be deployed to production.

### 2. Deployment Pipeline
A clear, automated path from commit to production:

```
┌───────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT PIPELINE                             │
│                                                                    │
│  Commit ──▶ Build ──▶ Test ──▶ Acceptance ──▶ Staging ──▶ Prod   │
│   Stage      Stage     Stage     Stage         Stage      Stage   │
│                                                                    │
│  ◄── Fast Feedback ───────────────── Longer, Thorough Tests ───►  │
│  ◄── Runs Every Commit ──────────── Runs on Candidates ────────►  │
│  ◄── Seconds/Minutes ────────────── Minutes/Hours ──────────────► │
└───────────────────────────────────────────────────────────────────┘
```

### 3. Automate Everything (Except the Deploy Decision)
- Build automation ✅
- Test automation ✅
- Deployment automation ✅
- Deploy **decision** = Human ✋

### 4. Build Artifacts Once
```
         ❌ WRONG: Build per environment
         ══════════════════════════════
         Source → Build(dev) → Dev
         Source → Build(staging) → Staging
         Source → Build(prod) → Production

         ✅ RIGHT: Build once, promote
         ══════════════════════════════
         Source → Build → Artifact → Dev → Staging → Production
                            │
                    Same binary everywhere!
                    Config is externalized
```

### 5. Externalize Configuration

```yaml
# ❌ Bad: Hardcoded config
database:
  host: prod-db.example.com
  password: s3cret

# ✅ Good: Environment variables / config injection
database:
  host: ${DB_HOST}
  password: ${DB_PASSWORD}
```

---

## Implementing Continuous Delivery

### Prerequisites Checklist

```
┌─────────────────────────────────────────────────┐
│         CD READINESS CHECKLIST                   │
│                                                   │
│  ☐ Version control for ALL code                  │
│  ☐ Automated build process                       │
│  ☐ Comprehensive automated test suite            │
│  ☐ Automated deployment scripts                  │
│  ☐ Environment parity (dev ≈ staging ≈ prod)     │
│  ☐ Configuration management                      │
│  ☐ Database migration automation                 │
│  ☐ Monitoring and alerting                       │
│  ☐ Rollback capability                           │
│  ☐ Infrastructure as Code                        │
└─────────────────────────────────────────────────┘
```

### Example: Multi-Stage Pipeline (GitHub Actions)

```yaml
name: Continuous Delivery

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: app-artifact
          path: dist/

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --coverage
      - run: npm run test:integration

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-artifact
      - run: ./deploy.sh staging

  smoke-test:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: curl -f https://staging.example.com/health

  deploy-production:
    needs: smoke-test
    runs-on: ubuntu-latest
    environment:
      name: production          # ← Requires manual approval in GitHub
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-artifact
      - run: ./deploy.sh production
```

### Example: Multi-Stage Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
                archiveArtifacts artifacts: 'dist/**'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                }
            }
        }

        stage('Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh './deploy.sh staging'
            }
        }

        stage('Smoke Tests') {
            steps {
                sh 'curl -f https://staging.example.com/health'
            }
        }

        stage('Deploy to Production') {
            input {
                message "Deploy to production?"       // ← Manual gate
                ok "Yes, deploy it!"
            }
            steps {
                sh './deploy.sh production'
            }
        }
    }
}
```

---

## Real-World Scenarios

### Scenario 1: E-Commerce Release Cycle

```
Monday:   Feature merged → CI passes → Deployed to staging automatically
Tuesday:  QA team runs manual exploratory testing on staging
Wednesday: Product owner reviews on staging → Approves release
Wednesday: One-click deployment to production
Thursday:  Monitoring confirms stability

Total time from commit to production: 3 days (majority is human review)
Pipeline automation time: ~15 minutes
```

### Scenario 2: Regulated Industry (Healthcare / Finance)

In regulated environments, CD with manual gates is often **required**:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Build   │───▶│   Test   │───▶│ Approval │───▶│  Deploy  │
│          │    │ (auto)   │    │          │    │ (auto)   │
└──────────┘    └──────────┘    │ ┌──────┐ │    └──────────┘
                                │ │ QA ✓ │ │
                                │ │Sec ✓ │ │    Audit trail
                                │ │Comp✓ │ │    maintained
                                │ └──────┘ │    automatically
                                └──────────┘
```

---

## Continuous Delivery Maturity Model

```
Level 4: OPTIMIZED
│  • Automated canary deployments
│  • Feature flags for progressive rollout
│  • Self-healing infrastructure
│  • Deployment multiple times per day
│
Level 3: MEASURED
│  • Full deployment pipeline
│  • Automated staging + production deploy
│  • Metrics-driven release decisions
│  • < 1 hour from commit to production-ready
│
Level 2: MANAGED
│  • Automated build & test
│  • Scripted deployments
│  • Some manual testing required
│  • Hours to deploy
│
Level 1: INITIAL
│  • Manual builds
│  • Manual testing
│  • Manual deployment
│  • Days/weeks to deploy
▼
```

---

## Troubleshooting Continuous Delivery

| Issue | Cause | Solution |
|---|---|---|
| Staging works, prod fails | Environment differences | Infrastructure as Code, env parity |
| Rollback takes too long | No automated rollback | Pre-bake rollback scripts, blue-green |
| Deployment fear | Lack of confidence in tests | Improve test coverage, add smoke tests |
| Manual steps creep in | Undocumented processes | Document and automate every step |
| Config errors in prod | Hardcoded values | Externalize config, use secrets managers |

---

## Summary Table

| Concept | Description |
|---|---|
| **CD Definition** | Practice of keeping code deployable at all times with automated pipelines |
| **Manual Gate** | Production deployment requires human approval |
| **Build Once** | Create artifact once, promote through environments |
| **Config** | Externalized — never baked into artifacts |
| **Environments** | Dev → Staging → Production (with parity) |
| **Rollback** | Must be automated and tested regularly |
| **Maturity** | From manual deployments → optimized automated pipelines |
| **Regulated** | CD is ideal for regulated industries because of approval gates |

---

## Quick Revision Questions

1. **What is the key difference between CI and Continuous Delivery?**  
   *CI ensures code integrates and passes tests; CD extends this by ensuring code is always deployable, with an automated pipeline up to production (but with a manual approval gate).*

2. **Why should you build artifacts only once?**  
   *Building per environment can introduce subtle differences. Build once and promote the same artifact through environments to ensure consistency.*

3. **What does "externalize configuration" mean and why is it important?**  
   *Keep environment-specific settings (DB URLs, API keys) outside the code artifact, injected at deploy time. This allows the same artifact to work in any environment.*

4. **Why is Continuous Delivery well-suited for regulated industries?**  
   *Because it includes manual approval gates with audit trails, satisfying compliance requirements while still automating everything else.*

5. **Name five prerequisites for implementing Continuous Delivery.**  
   *Automated builds, comprehensive tests, automated deployments, environment parity, configuration management.*

6. **What is a deployment pipeline?**  
   *An automated sequence of stages (build → test → stage → release) that every code change passes through on its way to production.*

---

[← Previous: Continuous Integration](01-continuous-integration.md) | [Back to README](../README.md) | [Next: Continuous Deployment →](03-continuous-deployment.md)
