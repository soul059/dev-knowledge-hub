# Chapter 2.4: Environment Promotion

[← Previous: Artifact Management](03-artifact-management.md) | [Back to README](../README.md) | [Next: Pipeline Triggers →](05-pipeline-triggers.md)

---

## Overview

**Environment promotion** is the practice of advancing a build artifact through progressively more production-like environments, with validation gates at each step. It ensures that only thoroughly tested code reaches production.

---

## Promotion Path

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    ENVIRONMENT PROMOTION PATH                            │
│                                                                          │
│  ┌────────┐  auto   ┌────────┐  auto   ┌────────┐ manual  ┌──────────┐│
│  │  DEV   │────────▶│   QA   │────────▶│STAGING │────────▶│PRODUCTION││
│  └────────┘         └────────┘         └────────┘         └──────────┘│
│                                                                          │
│  Trigger:   Every     PR merged   Nightly or    Approved               │
│             commit    to develop   on-demand     release                │
│                                                                          │
│  Tests:     Smoke     Full suite   Perf tests    Canary +              │
│             tests     + security   + UAT         monitoring            │
│                                                                          │
│  Data:      Mock      Test data    Prod-like     Production            │
│  Scale:     Minimal   Small        Prod-scale    Full                  │
│  Access:    Devs      Dev + QA     Team + PM     Everyone             │
│                                                                          │
│  ◄─── Low Risk, Fast ──────────────── High Risk, Thorough ──────────► │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Environment Types

| Environment | Purpose | Infrastructure | Data | Who Accesses |
|---|---|---|---|---|
| **Development** | Feature development | Minimal (1 instance) | Mock / seed data | Developers |
| **QA / Test** | Automated + manual testing | Small cluster | Synthetic test data | QA + Devs |
| **Staging / Pre-Prod** | Production rehearsal | Mirror of production | Anonymized prod data | Full team |
| **Production** | Live users | Full scale | Real data | Everyone |

---

## Promotion Gates

```
┌────────────────────────────────────────────────────────────────────┐
│                     PROMOTION GATES                                │
│                                                                     │
│  DEV ──▶ QA                                                        │
│  Gate: ✅ Build success                                            │
│        ✅ Unit tests pass                                          │
│        ✅ Lint / style checks pass                                 │
│                                                                     │
│  QA ──▶ STAGING                                                    │
│  Gate: ✅ All tests pass (unit + integration + E2E)                │
│        ✅ Code coverage > 80%                                      │
│        ✅ Security scan clean                                      │
│        ✅ Code review approved                                     │
│                                                                     │
│  STAGING ──▶ PRODUCTION                                            │
│  Gate: ✅ Smoke tests pass on staging                              │
│        ✅ Performance benchmarks met                               │
│        ✅ UAT sign-off (if applicable)                             │
│        ✅ Manual approval (for Continuous Delivery)                 │
│        ✅ Change management ticket approved                        │
└────────────────────────────────────────────────────────────────────┘
```

---

## Environment Parity

```
         ❌ BAD: Environments Diverge
         ═══════════════════════════

         Dev               Staging            Production
         ┌───────┐         ┌───────┐          ┌───────┐
         │Node 18│         │Node 20│          │Node 16│
         │SQLite │         │MySQL 5│          │MySQL 8│
         │1 inst │         │1 inst │          │3 inst │
         └───────┘         └───────┘          └───────┘
         "Works here!"     "Works here!"     "BROKEN! 😱"


         ✅ GOOD: Environment Parity (Infrastructure as Code)
         ═══════════════════════════════════════════════════

         Dev               Staging            Production
         ┌───────┐         ┌───────┐          ┌───────┐
         │Node 20│         │Node 20│          │Node 20│
         │MySQL 8│         │MySQL 8│          │MySQL 8│
         │1 inst │         │2 inst │          │3 inst │
         └───────┘         └───────┘          └───────┘

         Same software stack; only scale differs
         All defined by the SAME Terraform/Helm templates
```

### IaC Parity Example (Terraform)

```hcl
# modules/app/main.tf — Same module, different variables
module "app" {
  source = "./modules/app"

  environment   = var.environment       # "dev", "staging", "prod"
  instance_size = var.instance_size     # "t3.small", "t3.medium", "t3.large"
  replicas      = var.replicas          # 1, 2, 3
  db_engine     = "mysql"               # Same engine everywhere
  db_version    = "8.0"                 # Same version everywhere
}
```

---

## Promotion Strategies

### 1. Automatic Promotion

```
Commit ──▶ Build ──▶ Test ──▶ Dev ──▶ QA ──▶ Staging ──▶ Prod
                              auto    auto    auto       auto

Used by: Companies with mature testing and monitoring
Example: Netflix, Amazon (continuous deployment)
```

### 2. Semi-Automatic Promotion

```
Commit ──▶ Build ──▶ Test ──▶ Dev ──▶ QA ──▶ Staging ──▶ Prod
                              auto    auto    auto      MANUAL

Used by: Most organizations (continuous delivery)
```

### 3. Manual Promotion

```
Commit ──▶ Build ──▶ Test ──▶ Dev ──▶ QA ──▶ Staging ──▶ Prod
                              auto   MANUAL  MANUAL     MANUAL

Used by: Regulated industries, early-stage CI/CD adoption
```

---

## Implementation Example

### GitHub Actions — Environment Promotion

```yaml
name: Environment Promotion

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build && npm test
      - uses: actions/upload-artifact@v4
        with: { name: app, path: dist/ }

  deploy-dev:
    needs: build-and-test
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/download-artifact@v4
        with: { name: app }
      - run: ./deploy.sh dev
      - run: curl -f https://dev.example.com/health

  deploy-staging:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with: { name: app }
      - run: ./deploy.sh staging
      - run: npm run test:smoke -- --env=staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production          # Manual approval required
    steps:
      - uses: actions/download-artifact@v4
        with: { name: app }
      - run: ./deploy.sh production
      - run: npm run test:smoke -- --env=production
```

---

## Database Migrations Across Environments

```
┌──────────────────────────────────────────────────────────────┐
│         DATABASE MIGRATION PROMOTION                          │
│                                                               │
│  Migration files applied in order across all environments:   │
│                                                               │
│  migrations/                                                  │
│  ├── 001_create_users.sql        ← Applied in Dev first      │
│  ├── 002_add_email_column.sql    ← Applied in QA next        │
│  ├── 003_create_orders.sql       ← Applied in Staging        │
│  └── 004_add_index.sql           ← Applied in Production     │
│                                                               │
│  Tools: Flyway, Liquibase, Alembic, Knex, Django migrations │
│                                                               │
│  ⚠️ Migrations must be:                                     │
│     • Forward-only (never edit applied migrations)           │
│     • Backward compatible (support old + new code)           │
│     • Tested in lower environments first                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Environment Promotion

| Issue | Cause | Solution |
|---|---|---|
| Works in staging, fails in prod | Environment drift | Use IaC, enforce parity |
| Migration fails in staging | Missing data dependency | Test with prod-like data |
| Promotion blocked by flaky test | Non-deterministic tests | Fix/quarantine flaky tests |
| Wrong config in environment | Config mismatch | Use config management tools |
| Rollback affects database | Incompatible migration | Use backward-compatible migrations |

---

## Summary Table

| Concept | Description |
|---|---|
| **Promotion** | Moving artifacts through environments (Dev → QA → Staging → Prod) |
| **Gate** | Validation checkpoint before promotion (tests, approvals) |
| **Parity** | Environments should mirror production as closely as possible |
| **IaC** | Infrastructure as Code ensures consistent environments |
| **Same Artifact** | Same binary promoted — config injected per environment |
| **Migrations** | Database changes applied sequentially across environments |
| **Auto vs Manual** | Automatic for lower envs; manual approval for production |

---

## Quick Revision Questions

1. **What is environment promotion?**  
   *The practice of advancing build artifacts through progressively more production-like environments with validation gates at each step.*

2. **Why is environment parity important?**  
   *To prevent "works in staging, fails in production" issues. Same software stack ensures consistent behavior across environments.*

3. **What is a promotion gate?**  
   *A set of criteria that must be met before an artifact can advance to the next environment (e.g., tests pass, security scan clean).*

4. **How should database migrations be handled across environments?**  
   *Applied in order, forward-only, backward-compatible, and tested in lower environments before reaching production.*

5. **What is the difference between automatic and manual promotion?**  
   *Automatic promotion advances artifacts with no human intervention; manual requires explicit approval (common for production).*

6. **How does Infrastructure as Code help with environment parity?**  
   *The same IaC templates define all environments, ensuring consistent software versions and configuration with only scale differences.*

---

[← Previous: Artifact Management](03-artifact-management.md) | [Back to README](../README.md) | [Next: Pipeline Triggers →](05-pipeline-triggers.md)
