# Chapter 8.2: GitHub Actions Basics

[← Previous: Issues and Project Boards](01-issues-and-project-boards.md) | [Back to README](../README.md) | [Next: GitLab CI/CD Basics →](03-gitlab-ci-cd-basics.md)

---

## Overview

**GitHub Actions** is a CI/CD and automation platform built into GitHub. It lets you automate workflows — build, test, deploy, and more — triggered by events like pushes, pull requests, or schedules. Workflows are defined in YAML files stored in your repository.

---

## Core Concepts

```
┌──────────────────────────────────────────────────────────────┐
│       GITHUB ACTIONS ARCHITECTURE                           │
│                                                              │
│  EVENT (trigger)                                            │
│    │                                                        │
│    ▼                                                        │
│  WORKFLOW (.github/workflows/ci.yml)                        │
│    │                                                        │
│    ├── JOB: build                                           │
│    │     ├── Step 1: Checkout code                          │
│    │     ├── Step 2: Install dependencies                   │
│    │     └── Step 3: Build project                          │
│    │                                                        │
│    ├── JOB: test (depends on build)                         │
│    │     ├── Step 1: Checkout code                          │
│    │     ├── Step 2: Run unit tests                         │
│    │     └── Step 3: Run integration tests                  │
│    │                                                        │
│    └── JOB: deploy (depends on test)                        │
│          ├── Step 1: Checkout code                          │
│          └── Step 2: Deploy to production                   │
│                                                              │
│  Each JOB runs on a fresh RUNNER (virtual machine)          │
└──────────────────────────────────────────────────────────────┘
```

| Concept | Description |
|---------|-------------|
| **Workflow** | YAML file defining automated process |
| **Event** | Trigger that starts a workflow |
| **Job** | Set of steps that run on the same runner |
| **Step** | Individual task (run command or use action) |
| **Action** | Reusable unit of code (from Marketplace) |
| **Runner** | VM that executes jobs (GitHub-hosted or self-hosted) |

---

## Your First Workflow

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

---

## Events (Triggers)

```yaml
on:
  # Push to specific branches
  push:
    branches: [main, develop]
    paths: ['src/**', '!docs/**']    # Only trigger for src changes

  # Pull request events
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]

  # Scheduled (cron)
  schedule:
    - cron: '0 6 * * 1'    # Every Monday at 6:00 AM UTC

  # On release
  release:
    types: [published]

  # On issue events
  issues:
    types: [opened, labeled]
```

---

## Job Configuration

```yaml
jobs:
  test:
    runs-on: ubuntu-latest         # Runner OS
    timeout-minutes: 30            # Fail if takes too long

    strategy:
      matrix:                      # Run on multiple configs
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm ci
      - run: npm test

  deploy:
    needs: test                    # Run after test job
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Only on main
    environment: production        # Deployment environment

    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh
```

```
┌──────────────────────────────────────────────────────────────┐
│  MATRIX STRATEGY                                            │
│                                                              │
│  matrix:                                                    │
│    node: [18, 20, 22]                                       │
│    os: [ubuntu, windows]                                    │
│                                                              │
│  Creates 6 parallel jobs:                                   │
│  ┌──────────────────┐ ┌──────────────────┐                  │
│  │ Node 18 + Ubuntu │ │ Node 18 + Windows│                  │
│  └──────────────────┘ └──────────────────┘                  │
│  ┌──────────────────┐ ┌──────────────────┐                  │
│  │ Node 20 + Ubuntu │ │ Node 20 + Windows│                  │
│  └──────────────────┘ └──────────────────┘                  │
│  ┌──────────────────┐ ┌──────────────────┐                  │
│  │ Node 22 + Ubuntu │ │ Node 22 + Windows│                  │
│  └──────────────────┘ └──────────────────┘                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Secrets and Variables

```yaml
# Using secrets (Settings → Secrets and variables → Actions)
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    run: ./deploy.sh

  - name: Login to Docker Hub
    uses: docker/login-action@v3
    with:
      username: ${{ secrets.DOCKER_USERNAME }}
      password: ${{ secrets.DOCKER_TOKEN }}
```

```
┌──────────────────────────────────────────────────────────────┐
│  SECRETS ARE:                                               │
│  • Encrypted at rest                                        │
│  • Not printed in logs (masked with ***)                    │
│  • Not available to forked repos (for security)             │
│  • Set at repo, environment, or org level                   │
│                                                              │
│  VARIABLES are for non-sensitive configuration:             │
│  ${{ vars.DEPLOY_URL }}                                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Popular Actions from Marketplace

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Check out repo code |
| `actions/setup-node@v4` | Set up Node.js |
| `actions/setup-python@v5` | Set up Python |
| `actions/cache@v4` | Cache dependencies |
| `actions/upload-artifact@v4` | Upload build artifacts |
| `docker/build-push-action@v5` | Build and push Docker images |
| `softprops/action-gh-release@v2` | Create GitHub releases |

---

## Artifacts and Caching

```yaml
steps:
  # Cache dependencies for faster builds
  - name: Cache node modules
    uses: actions/cache@v4
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-

  # Upload build output as artifact
  - name: Upload build
    uses: actions/upload-artifact@v4
    with:
      name: build-output
      path: dist/
      retention-days: 7

  # Download artifact in another job
  - name: Download build
    uses: actions/download-artifact@v4
    with:
      name: build-output
```

---

## Real-World Workflow Examples

### Complete CI/CD Pipeline

```yaml
name: CI/CD
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: coverage/ }

  deploy:
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploying to production..."
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Workflow not triggering | Check branch name, event type, and file path (`.github/workflows/`) |
| "Permission denied" | Check repo permissions; use appropriate `GITHUB_TOKEN` permissions |
| Secrets not available | Secrets don't pass to workflows from forked repos |
| Workflow too slow | Use caching, matrix parallelism, and skip unnecessary steps |
| YAML syntax errors | Validate with `actionlint` or GitHub's workflow editor |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Workflow | YAML file in `.github/workflows/` |
| Event | Trigger: push, pull_request, schedule, etc. |
| Job | Group of steps on one runner |
| Step | Single task: `run` command or `uses` action |
| Runner | VM executing jobs (ubuntu, windows, macos) |
| Matrix | Run jobs across multiple configurations |
| Secrets | Encrypted environment variables |
| Artifacts | Files produced by workflow (uploaded/downloaded) |
| Cache | Speed up workflows by caching dependencies |

---

## Quick Revision Questions

1. **What is a GitHub Actions workflow?**
   <details><summary>Answer</summary>A workflow is an automated process defined in a YAML file under `.github/workflows/`. It consists of one or more jobs triggered by events (push, PR, schedule, etc.). Each job contains steps that run on a runner (virtual machine).</details>

2. **What is the difference between a Job and a Step?**
   <details><summary>Answer</summary>A Job is a set of steps that execute on the same runner. Jobs run in parallel by default (use `needs:` for sequential). A Step is an individual task within a job — either a shell command (`run:`) or a reusable action (`uses:`).</details>

3. **What is a matrix strategy?**
   <details><summary>Answer</summary>A matrix strategy runs the same job across multiple configurations in parallel. For example, testing across Node.js versions 18, 20, 22 and OS ubuntu and windows creates 6 parallel jobs (3 versions × 2 operating systems).</details>

4. **How are secrets handled in GitHub Actions?**
   <details><summary>Answer</summary>Secrets are encrypted at rest and masked in logs. They're set in repo/org settings and accessed via `${{ secrets.NAME }}`. They are not available to workflows triggered by forked repositories for security reasons.</details>

5. **How do you make one job depend on another?**
   <details><summary>Answer</summary>Use the `needs:` keyword. For example, `needs: [lint, test]` makes the job wait for both lint and test jobs to complete successfully before starting. Without `needs:`, jobs run in parallel.</details>

---

[← Previous: Issues and Project Boards](01-issues-and-project-boards.md) | [Back to README](../README.md) | [Next: GitLab CI/CD Basics →](03-gitlab-ci-cd-basics.md)
