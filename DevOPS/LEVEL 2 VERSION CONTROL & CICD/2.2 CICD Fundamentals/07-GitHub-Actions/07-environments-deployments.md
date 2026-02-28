# Chapter 7.7: Environments & Deployments

[← Previous: Reusable Workflows](06-reusable-workflows.md) | [Back to README](../README.md) | [Next: GitHub Actions Best Practices →](08-github-actions-best-practices.md)

---

## Overview

**Environments** in GitHub Actions provide deployment targets with protection rules, secrets, and deployment history. They enable controlled, auditable deployments with approval gates.

---

## Environment Concepts

```
  DEPLOYMENT FLOW WITH ENVIRONMENTS
  ══════════════════════════════════

  Push to main
       │
       ▼
  ┌─────────────┐
  │  CI (Build   │
  │  & Test)     │
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐    Auto-deploy
  │  staging     │◄── No approval needed
  │  environment │
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐    Manual approval
  │  production  │◄── Required reviewers
  │  environment │    Wait timer: 10 min
  │              │    Branch: main only
  └─────────────┘
```

---

## Creating Environments

### Via GitHub UI

```
  Repository → Settings → Environments → New environment

  ┌────────────────────────────────────────────┐
  │ Environment: production                    │
  │                                            │
  │ Protection Rules:                          │
  │ ☑ Required reviewers                       │
  │   └── @ops-team, @tech-lead               │
  │ ☑ Wait timer: 10 minutes                  │
  │ ☑ Deployment branches: main only           │
  │ ☐ Custom deployment protection rules       │
  │                                            │
  │ Environment secrets:                       │
  │ ├── AWS_ROLE_ARN                           │
  │ ├── DB_CONNECTION_STRING                   │
  │ └── DEPLOY_TOKEN                           │
  │                                            │
  │ Environment variables:                     │
  │ ├── API_URL: https://api.example.com       │
  │ ├── CLUSTER_NAME: prod-cluster             │
  │ └── REPLICAS: 5                            │
  └────────────────────────────────────────────┘
```

---

## Using Environments in Workflows

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v4
      - run: |
          echo "Deploying to ${{ vars.API_URL }}"
          echo "Cluster: ${{ vars.CLUSTER_NAME }}"
      - run: ./deploy.sh staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    # This job will PAUSE until a reviewer approves
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh production
```

---

## Protection Rules

### Required Reviewers

```
  APPROVAL FLOW
  ═════════════

  Workflow reaches job with environment: production
       │
       ▼
  ┌──────────────────────────────────────────┐
  │ ⏸ Waiting for approval                   │
  │                                           │
  │ Required reviewers: @alice, @bob          │
  │ (Need 1 of 2 to approve)                 │
  │                                           │
  │ [ Review deployments ]                    │
  └──────────────────────────────────────────┘
       │
       ▼ (after approval)
  Deployment proceeds
```

### Wait Timer

```yaml
# Auto-waits X minutes before deploying
# Configured in environment settings, not YAML
# Useful for: giving teams time to cancel if needed
```

### Branch Restrictions

```
  Branch Protection on Environments:
  ├── All branches          → Any branch can deploy
  ├── Selected branches     → Only matching branches
  │   ├── main
  │   └── release/*
  └── Protected branches    → Only branch-protected branches
```

---

## Deployment Status & History

```
  DEPLOYMENT HISTORY (visible in GitHub UI)
  ═════════════════════════════════════════

  Repository → Deployments

  production:
  ├── #15  Active   main@abc1234  2024-01-15 14:30  @alice
  ├── #14  Inactive main@def5678  2024-01-14 10:00  @bob
  └── #13  Inactive main@ghi9012  2024-01-13 09:00  @alice

  staging:
  ├── #42  Active   main@abc1234  2024-01-15 14:00  @ci-bot
  └── #41  Inactive main@def5678  2024-01-14 09:30  @ci-bot
```

### Creating Deployment Status in Workflow

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        id: deploy
        run: |
          # Your deployment logic
          ./deploy.sh production
          echo "url=https://app.example.com" >> $GITHUB_OUTPUT

      - name: Verify deployment
        run: |
          curl -f https://app.example.com/health || exit 1
```

---

## Multi-Environment Pipeline

```yaml
name: Multi-Environment Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.build.outputs.image }}
    steps:
      - uses: actions/checkout@v4
      - id: build
        run: |
          IMAGE="ghcr.io/${{ github.repository }}:${{ github.sha }}"
          docker build -t $IMAGE .
          docker push $IMAGE
          echo "image=$IMAGE" >> $GITHUB_OUTPUT

  deploy-dev:
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - run: kubectl set image deployment/app app=${{ needs.build.outputs.image }}

  deploy-staging:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - run: kubectl set image deployment/app app=${{ needs.build.outputs.image }}

  smoke-test:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: |
          curl -f https://staging.example.com/health
          curl -f https://staging.example.com/api/v1/status

  deploy-production:
    needs: smoke-test
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    steps:
      - run: kubectl set image deployment/app app=${{ needs.build.outputs.image }}

  post-deploy:
    needs: deploy-production
    runs-on: ubuntu-latest
    steps:
      - run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text":"Deployed to production: ${{ needs.build.outputs.image }}"}'
```

```
  PIPELINE VISUALIZATION
  ═══════════════════════

  build → deploy-dev → deploy-staging → smoke-test → deploy-production → post-deploy
           (auto)       (auto)          (auto)        (approval gate)      (auto)
```

---

## Concurrency Control

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    concurrency:
      group: deploy-production
      cancel-in-progress: false    # Queue (don't cancel) pending deploys

  # Or: cancel old deploys in favor of new
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    concurrency:
      group: deploy-staging
      cancel-in-progress: true     # Cancel previous staging deploy
```

---

## Real-World Scenario

> **Scenario**: A team wants automated deployments to staging and gated deployments to production with smoke tests in between.
>
> **Solution**: Create `staging` and `production` environments. Staging auto-deploys on merge to main. After staging deploy, a smoke test job runs health checks. Production requires approval from 2 reviewers, restricts to `main` branch, and has a 5-minute wait timer. Both environments have their own secrets (AWS credentials, DB connection strings) and variables (cluster name, replicas). Deployment history is visible in the GitHub UI.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Job hangs on "Waiting" | Required reviewers configured | A designated reviewer must approve in the UI |
| "Branch not allowed" | Branch restriction on environment | Deploy from allowed branch or update environment settings |
| Environment secrets empty | Wrong environment name | Ensure `environment: name` matches exactly |
| Deployment URL not showing | Missing `url:` in environment block | Add `url:` to the `environment:` configuration |
| Concurrent deploys conflict | No concurrency control | Add `concurrency:` group to prevent parallel deploys |
| Approval bypassed | User is both author and reviewer | Configure separate reviewer accounts |

---

## Summary Table

| Feature | Purpose |
|---|---|
| **Environments** | Named deployment targets with configuration |
| **Required Reviewers** | Manual approval gates before deployment |
| **Wait Timer** | Delay before deployment starts |
| **Branch Restrictions** | Limit which branches can deploy |
| **Environment Secrets** | Deployment-specific credentials |
| **Environment Variables** | Deployment-specific configuration |
| **Deployment History** | Audit trail of all deployments |
| **Concurrency** | Prevent parallel deployments |

---

## Quick Revision Questions

1. **What are GitHub environments?** *Named deployment targets (e.g., staging, production) with protection rules, secrets, variables, and deployment history.*
2. **How do required reviewers work?** *When a job targets an environment with required reviewers, the workflow pauses until an authorized reviewer approves in the GitHub UI.*
3. **How do environment secrets differ from repo secrets?** *Environment secrets are only available to jobs that specify that environment — they override repo secrets of the same name and are scoped to specific deployment targets.*
4. **What is a wait timer?** *A configurable delay (in minutes) that must pass before a deployment begins, giving teams time to cancel if needed.*
5. **How is deployment history tracked?** *GitHub automatically records each deployment to an environment, showing status (active/inactive), commit, timestamp, and who triggered it.*
6. **How do you prevent parallel deployments?** *Use `concurrency:` groups with `cancel-in-progress: false` to queue deployments, or `true` to cancel previous ones.*

---

[← Previous: Reusable Workflows](06-reusable-workflows.md) | [Back to README](../README.md) | [Next: GitHub Actions Best Practices →](08-github-actions-best-practices.md)
