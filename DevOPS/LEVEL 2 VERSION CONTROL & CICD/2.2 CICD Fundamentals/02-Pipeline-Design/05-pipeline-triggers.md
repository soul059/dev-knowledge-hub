# Chapter 2.5: Pipeline Triggers

[← Previous: Environment Promotion](04-environment-promotion.md) | [Back to README](../README.md) | [Next: Parallelization and Caching →](06-parallelization-caching.md)

---

## Overview

**Pipeline triggers** define **when** a CI/CD pipeline starts executing. Choosing the right triggers balances between catching issues early (run often) and conserving resources (run only when needed).

---

## Trigger Types

```
┌──────────────────────────────────────────────────────────────────┐
│                    PIPELINE TRIGGERS                              │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │    EVENT      │  │   SCHEDULE   │  │    MANUAL    │          │
│  │   BASED       │  │    BASED     │  │              │          │
│  │               │  │              │  │              │          │
│  │ • Push        │  │ • Cron       │  │ • Button     │          │
│  │ • Pull Request│  │ • Nightly    │  │ • API call   │          │
│  │ • Tag/Release │  │ • Weekly     │  │ • ChatOps    │          │
│  │ • Merge       │  │              │  │              │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐                             │
│  │  DEPENDENCY   │  │   WEBHOOK    │                             │
│  │   BASED       │  │              │                             │
│  │               │  │              │                             │
│  │ • Upstream    │  │ • External   │                             │
│  │   pipeline    │  │   service    │                             │
│  │ • Artifact    │  │   callback   │                             │
│  │   published   │  │              │                             │
│  └──────────────┘  └──────────────┘                             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Event-Based Triggers

### Push Trigger
```yaml
# GitHub Actions
on:
  push:
    branches: [main, develop]
    paths:
      - 'src/**'           # Only trigger on source code changes
      - '!docs/**'         # Ignore documentation changes

# GitLab CI
rules:
  - if: $CI_PIPELINE_SOURCE == "push"
    changes:
      - src/**

# Jenkins
triggers {
    pollSCM('H/5 * * * *')   # Poll every 5 min (fallback for webhooks)
}
```

### Pull Request Trigger
```yaml
# GitHub Actions
on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

# GitLab CI — Merge Request
rules:
  - if: $CI_MERGE_REQUEST_ID

# Azure Pipelines
pr:
  branches:
    include: [main]
```

### Tag / Release Trigger
```yaml
# GitHub Actions — trigger on version tags
on:
  push:
    tags:
      - 'v*.*.*'     # Matches v1.0.0, v2.3.1, etc.

# GitLab CI
rules:
  - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
```

---

## 2. Schedule-Based Triggers

```yaml
# GitHub Actions — Cron schedule
on:
  schedule:
    - cron: '0 2 * * *'      # Every day at 2:00 AM UTC
    - cron: '0 0 * * 0'      # Every Sunday at midnight

# GitLab CI
nightly-build:
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"

# Jenkins
triggers {
    cron('H 2 * * *')         # Daily at ~2 AM
}
```

```
CRON EXPRESSION FORMAT
══════════════════════

  ┌───────────── minute (0-59)
  │ ┌─────────── hour (0-23)
  │ │ ┌───────── day of month (1-31)
  │ │ │ ┌─────── month (1-12)
  │ │ │ │ ┌───── day of week (0-6, Sun=0)
  │ │ │ │ │
  * * * * *

  Examples:
  0 2 * * *     → Every day at 2:00 AM
  0 0 * * 1-5   → Mon-Fri at midnight
  */15 * * * *  → Every 15 minutes
  0 0 1 * *     → 1st of every month
```

**Common Scheduled Jobs:**
| Schedule | Use Case |
|---|---|
| Nightly | Full regression test suite |
| Weekly | Dependency vulnerability scan |
| Hourly | Integration tests for active branches |
| Monthly | License compliance check |

---

## 3. Manual Triggers

```yaml
# GitHub Actions
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options: [staging, production]
      version:
        description: 'Version to deploy'
        required: true
        type: string

# GitLab CI
deploy:
  when: manual
  script:
    - ./deploy.sh $CI_ENVIRONMENT_NAME

# Jenkins
pipeline {
    parameters {
        choice(name: 'ENV', choices: ['staging', 'production'])
        string(name: 'VERSION', defaultValue: 'latest')
    }
}
```

---

## 4. Dependency / Upstream Triggers

```yaml
# GitHub Actions — Trigger from another workflow
on:
  workflow_run:
    workflows: ["Build"]
    types: [completed]
    branches: [main]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

```
┌────────────────────────────────────────────────────────────┐
│           UPSTREAM PIPELINE TRIGGERS                        │
│                                                             │
│  ┌─────────────┐  triggers  ┌─────────────┐               │
│  │ Library Repo │──────────▶│  App Repo    │               │
│  │ (upstream)   │           │ (downstream) │               │
│  │              │           │              │               │
│  │ Build + Test │           │ Build + Test │               │
│  │ → Publish    │           │ with new lib │               │
│  └─────────────┘           └─────────────┘               │
│                                                             │
│  When the library publishes a new version,                 │
│  the app's CI/CD rebuilds with the update.                 │
└────────────────────────────────────────────────────────────┘
```

---

## 5. Path Filters (Monorepo Support)

```yaml
# GitHub Actions — Only trigger for specific paths
on:
  push:
    paths:
      - 'services/auth/**'       # Only when auth service changes
      - 'libs/shared/**'         # Or shared library changes
    paths-ignore:
      - '**.md'                  # Ignore markdown files
      - '.github/**'             # Ignore workflow changes

# GitLab CI — changes keyword
build-auth:
  rules:
    - changes:
        - services/auth/**
        - libs/shared/**
```

```
MONOREPO TRIGGER MAPPING
═════════════════════════

  monorepo/
  ├── services/
  │   ├── auth/     ──▶ triggers: auth-pipeline
  │   ├── payments/ ──▶ triggers: payments-pipeline
  │   └── users/    ──▶ triggers: users-pipeline
  ├── libs/
  │   └── shared/   ──▶ triggers: ALL pipelines (dependency)
  └── docs/         ──▶ triggers: docs-pipeline only
```

---

## Trigger Strategy Best Practices

```
┌──────────────────────────────────────────────────────────────────┐
│            TRIGGER STRATEGY MATRIX                                │
│                                                                   │
│  Event              │ Action                │ Environment         │
│  ═══════════════════╪═══════════════════════╪═══════════════════  │
│  Push to feature/*  │ Build + Unit Test     │ —                   │
│  PR to main         │ Full Test Suite       │ Ephemeral env       │
│  Merge to main      │ Build + Deploy        │ Staging             │
│  Tag v*.*.*         │ Release + Deploy      │ Production          │
│  Schedule (nightly) │ Security + Perf tests │ QA                  │
│  Manual             │ Hotfix deployment     │ Production          │
│  Upstream completed │ Rebuild dependents    │ Dev                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Triggers

| Issue | Cause | Solution |
|---|---|---|
| Pipeline doesn't start | Wrong branch/path filter | Check trigger conditions, verify webhook |
| Pipeline runs too often | Missing path filters | Add `paths` / `paths-ignore` |
| Scheduled job doesn't run | Inactive repo (GitHub free) | Ensure activity in repo, check cron syntax |
| Double pipeline execution | Both push and PR triggers match | Use conditional logic to prevent duplicates |
| Manual trigger missing inputs | Wrong input types | Verify `workflow_dispatch` input definitions |

---

## Summary Table

| Trigger Type | When to Use | Example |
|---|---|---|
| **Push** | Every code change | CI on every commit |
| **Pull Request** | Code review validation | Run tests before merge |
| **Tag/Release** | Version-based releases | Deploy v1.2.0 to production |
| **Schedule** | Periodic checks | Nightly security scan |
| **Manual** | On-demand actions | Emergency hotfix deploy |
| **Upstream** | Dependency chain | Rebuild when library updates |
| **Webhook** | External events | Deploy when Docker image published |
| **Path filter** | Monorepo optimization | Build only changed services |

---

## Quick Revision Questions

1. **What are the main types of CI/CD pipeline triggers?**  
   *Event-based (push, PR, tag), schedule-based (cron), manual (button/API), dependency (upstream pipeline), and webhook (external).*

2. **Why are path filters important for monorepos?**  
   *They ensure only the affected services/components are built and tested when a change occurs, saving time and resources.*

3. **Write a cron expression for "every weekday at 3:00 AM UTC".**  
   *`0 3 * * 1-5`*

4. **How can you prevent duplicate pipeline runs when both push and PR triggers are configured?**  
   *Use conditional logic or configure the push trigger to exclude branches that also have PR triggers.*

5. **When would you use a manual trigger instead of an automatic one?**  
   *For emergency deployments, hotfixes, deploying specific versions, or when you need user input (environment, version).*

6. **What is an upstream trigger?**  
   *A trigger that activates a pipeline when a dependent (upstream) pipeline completes, such as rebuilding an app when its library publishes a new version.*

---

[← Previous: Environment Promotion](04-environment-promotion.md) | [Back to README](../README.md) | [Next: Parallelization and Caching →](06-parallelization-caching.md)
