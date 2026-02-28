# Chapter 7.1: GitHub Actions Workflow Syntax

[← Previous: Jenkins Best Practices](../06-Jenkins/07-jenkins-best-practices.md) | [Back to README](../README.md) | [Next: Actions & Marketplace →](02-actions-marketplace.md)

---

## Overview

**GitHub Actions** is a CI/CD platform built into GitHub. Workflows are defined in YAML files stored in `.github/workflows/` and triggered by repository events. No external CI server is needed.

---

## Workflow Structure

```
  .github/
  └── workflows/
      ├── ci.yml           ← Runs on every push / PR
      ├── deploy.yml       ← Runs on merge to main
      └── nightly.yml      ← Scheduled workflow

  WORKFLOW ANATOMY:
  ┌───────────────────────────────────────┐
  │ Workflow (YAML file)                  │
  │ ├── name                              │
  │ ├── on (triggers)                     │
  │ ├── env (global variables)            │
  │ ├── permissions                       │
  │ └── jobs                              │
  │     ├── job-1                          │
  │     │   ├── runs-on (runner)          │
  │     │   ├── steps                     │
  │     │   │   ├── uses (action)         │
  │     │   │   └── run (shell command)   │
  │     │   └── outputs                   │
  │     └── job-2                          │
  │         ├── needs: job-1              │
  │         └── steps                     │
  └───────────────────────────────────────┘
```

---

## Complete Workflow Example

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io

jobs:
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test -- --ci --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-node-${{ matrix.node-version }}
          path: coverage/

  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }}
```

---

## Triggers (`on:`)

```yaml
on:
  # Push events
  push:
    branches: [main, 'release/**']
    tags: ['v*']
    paths: ['src/**', 'package.json']
    paths-ignore: ['docs/**', '*.md']

  # Pull request events
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # Schedule (cron)
  schedule:
    - cron: '0 2 * * 1-5'   # Weekdays at 2 AM UTC

  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options: [staging, production]
      dry_run:
        description: 'Dry run (no deploy)'
        type: boolean
        default: false

  # Triggered by another workflow
  workflow_call:
    inputs:
      app_name:
        type: string
        required: true
    secrets:
      deploy_key:
        required: true

  # Other events
  release:
    types: [published]
  issues:
    types: [opened, labeled]
  workflow_run:
    workflows: ["CI"]
    types: [completed]
```

---

## Job Configuration

```yaml
jobs:
  deploy:
    name: Deploy Application
    runs-on: ubuntu-latest

    # Job dependencies
    needs: [lint, test, build]

    # Conditionals
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    # Environment (with protection rules)
    environment:
      name: production
      url: https://app.example.com

    # Concurrency control
    concurrency:
      group: deploy-${{ github.ref }}
      cancel-in-progress: false

    # Timeout
    timeout-minutes: 30

    # Outputs (for downstream jobs)
    outputs:
      deploy_url: ${{ steps.deploy.outputs.url }}

    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        id: deploy
        run: |
          URL=$(./deploy.sh)
          echo "url=${URL}" >> $GITHUB_OUTPUT
```

---

## Steps Deep Dive

```yaml
steps:
  # 1. Use an action
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0            # Full clone for tags

  # 2. Run a shell command
  - name: Build project
    run: npm run build
    working-directory: ./frontend

  # 3. Multi-line script
  - name: Complex script
    run: |
      echo "Building ${{ github.repository }}"
      npm ci
      npm run build
      npm test
    shell: bash

  # 4. Conditional step
  - name: Notify on failure
    if: failure()
    run: echo "Build failed!"

  # 5. Set environment variables
  - name: Configure
    env:
      API_URL: https://api.example.com
      DEBUG: 'true'
    run: ./configure.sh

  # 6. Set outputs for other steps
  - name: Get version
    id: version
    run: echo "version=$(cat VERSION)" >> $GITHUB_OUTPUT

  - name: Use version
    run: echo "Version is ${{ steps.version.outputs.version }}"
```

---

## Expressions & Contexts

```
  AVAILABLE CONTEXTS
  ══════════════════

  ${{ github.* }}       → Event info (ref, sha, actor, repository)
  ${{ env.* }}          → Environment variables
  ${{ secrets.* }}      → Encrypted secrets
  ${{ vars.* }}         → Repository/org variables
  ${{ inputs.* }}       → workflow_dispatch / workflow_call inputs
  ${{ steps.*.outputs.* }} → Step outputs
  ${{ needs.*.outputs.* }} → Job outputs
  ${{ matrix.* }}       → Matrix values
  ${{ runner.* }}       → Runner info (os, arch, temp)
  ${{ job.* }}          → Job status
```

### Expression Examples

```yaml
# Conditional expressions
if: github.ref == 'refs/heads/main'
if: contains(github.event.head_commit.message, '[skip ci]')
if: startsWith(github.ref, 'refs/tags/v')
if: github.event_name == 'pull_request'
if: always()          # Run even if previous steps failed
if: failure()         # Run only if previous steps failed
if: cancelled()       # Run only if workflow was cancelled
if: success()         # Run only if previous steps succeeded (default)

# Combining conditions
if: |
  github.event_name == 'push' &&
  github.ref == 'refs/heads/main' &&
  !contains(github.event.head_commit.message, '[skip deploy]')
```

---

## Environment Variables

```yaml
# Three levels of env vars
env:                                    # Workflow level
  APP_NAME: my-app

jobs:
  build:
    env:                                # Job level
      NODE_ENV: production
    runs-on: ubuntu-latest
    steps:
      - env:                            # Step level
          DEBUG: 'true'
        run: echo "$APP_NAME in $NODE_ENV (debug=$DEBUG)"

      # Dynamic env vars
      - run: echo "BUILD_TIME=$(date -u +%Y%m%dT%H%M%S)" >> $GITHUB_ENV
      - run: echo "Build at $BUILD_TIME"  # Available in subsequent steps
```

---

## Real-World Scenario

> **Scenario**: A team wants CI to run lint, test across 3 Node versions, and build a Docker image only on `main`.
>
> **Solution**: Single workflow file with 3 jobs. `lint` job runs first. `test` job uses matrix strategy for Node 18/20/22 and depends on `lint`. `build` job runs only on `main` branch, depends on `test`, and pushes to GitHub Container Registry using `docker/build-push-action`.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Workflow not triggering | Wrong event/branch filter | Check `on:` config matches your branch names |
| "Resource not accessible" | Insufficient permissions | Add `permissions:` block with required scopes |
| Secrets not available | Using secrets in PR from fork | Secrets are not available in PRs from forks (security) |
| Step output not found | Missing `id` on step | Add `id: step-name` and use `>> $GITHUB_OUTPUT` |
| Cache miss every time | Key doesn't match | Check cache key includes lockfile hash |
| `actions/checkout` shallow | Default `fetch-depth: 1` | Set `fetch-depth: 0` for full history |

---

## Summary Table

| Element | Purpose |
|---|---|
| `name:` | Workflow display name |
| `on:` | Trigger events (push, PR, schedule, manual) |
| `permissions:` | GITHUB_TOKEN permission scopes |
| `env:` | Environment variables (workflow/job/step) |
| `jobs:` | Parallel units of work |
| `runs-on:` | Runner type (ubuntu-latest, windows-latest, self-hosted) |
| `needs:` | Job dependencies |
| `strategy.matrix:` | Matrix builds (multiple configurations) |
| `steps:` | Sequential actions within a job |
| `uses:` | Call a reusable action |
| `run:` | Execute a shell command |

---

## Quick Revision Questions

1. **Where are GitHub Actions workflows stored?** *In `.github/workflows/` as YAML files in the repository.*
2. **What is the difference between `uses:` and `run:`?** *`uses:` calls a reusable action (from Marketplace or local). `run:` executes a shell command directly.*
3. **How do you pass data between steps?** *Use `echo "key=value" >> $GITHUB_OUTPUT` in the source step (with `id:`), then access via `${{ steps.step-id.outputs.key }}`.*
4. **What does `needs:` do?** *Defines job dependencies — the job waits until all listed jobs complete successfully before running.*
5. **How do you trigger a workflow manually?** *Add `workflow_dispatch:` to the `on:` block, optionally with `inputs:` for parameters.*
6. **What does `if: always()` do on a step?** *The step runs regardless of whether previous steps succeeded or failed — useful for cleanup or notification steps.*

---

[← Previous: Jenkins Best Practices](../06-Jenkins/07-jenkins-best-practices.md) | [Back to README](../README.md) | [Next: Actions & Marketplace →](02-actions-marketplace.md)
