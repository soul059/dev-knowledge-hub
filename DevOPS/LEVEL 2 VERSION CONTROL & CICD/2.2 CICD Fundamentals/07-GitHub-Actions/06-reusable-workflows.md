# Chapter 7.6: Reusable Workflows

[← Previous: Matrix Builds](05-matrix-builds.md) | [Back to README](../README.md) | [Next: Environments & Deployments →](07-environments-deployments.md)

---

## Overview

**Reusable workflows** let you define an entire workflow once and call it from other workflows. This is job-level reuse — DRY applied to CI/CD across an organization.

---

## Reusable vs Composite Actions

```
  COMPARISON
  ══════════

  COMPOSITE ACTIONS                 REUSABLE WORKFLOWS
  ├── Step-level reuse              ├── Job-level reuse
  ├── Called within a job's steps   ├── Called as an entire job
  ├── Defined in action.yml        ├── Defined in workflow YAML
  ├── uses: ./.github/actions/x    ├── uses: org/repo/.github/workflows/x.yml@v1
  └── Share steps                   └── Share complete jobs (with runners, env, etc.)

  WHEN TO USE WHAT:
  ┌────────────────────────────────────────────────────────┐
  │ Need to share a few steps?      → Composite Action    │
  │ Need to share a full job/flow?  → Reusable Workflow   │
  │ Need to share across repos?     → Either (in shared repo) │
  └────────────────────────────────────────────────────────┘
```

---

## Creating a Reusable Workflow

```yaml
# .github/workflows/reusable-ci.yml (in shared-workflows repo)
name: Reusable CI

on:
  workflow_call:               # Makes this workflow callable
    inputs:
      node-version:
        description: 'Node.js version'
        type: string
        default: '20'
      working-directory:
        description: 'Directory containing the project'
        type: string
        default: '.'
      run-e2e:
        description: 'Whether to run E2E tests'
        type: boolean
        default: false
    secrets:
      npm-token:
        description: 'NPM registry token'
        required: false
      sonar-token:
        description: 'SonarCloud token'
        required: false
    outputs:
      coverage:
        description: 'Test coverage percentage'
        value: ${{ jobs.test.outputs.coverage }}

jobs:
  lint:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    outputs:
      coverage: ${{ steps.coverage.outputs.value }}
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json
      - run: npm ci
      - run: npm test -- --ci --coverage
      - id: coverage
        run: |
          COV=$(grep -oP '"pct":\K[0-9.]+' coverage/coverage-summary.json | head -1)
          echo "value=${COV}" >> $GITHUB_OUTPUT

  e2e:
    if: inputs.run-e2e
    runs-on: ubuntu-latest
    needs: test
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
```

---

## Calling a Reusable Workflow

```yaml
# .github/workflows/ci.yml (in application repo)
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # Call reusable workflow from another repo
  ci:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
      working-directory: '.'
      run-e2e: true
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
      sonar-token: ${{ secrets.SONAR_TOKEN }}

  # Use output from reusable workflow
  report:
    needs: ci
    runs-on: ubuntu-latest
    steps:
      - run: echo "Coverage was ${{ needs.ci.outputs.coverage }}%"
```

### Call from Same Repository

```yaml
jobs:
  ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '20'
    secrets: inherit    # Pass all secrets automatically
```

---

## Inherit Secrets

```yaml
# Pass all secrets without listing each one
jobs:
  ci:
    uses: my-org/shared-workflows/.github/workflows/deploy.yml@v1
    with:
      environment: production
    secrets: inherit    # All org/repo secrets available
```

---

## Nesting Workflows

```
  WORKFLOW CHAIN
  ══════════════

  App CI (ci.yml)
  └── calls → Reusable CI (reusable-ci.yml)
              └── calls → Reusable Deploy (reusable-deploy.yml)

  LIMITS:
  • Max 4 levels of nesting
  • Max 20 reusable workflow calls in one file
  • Reusable workflows can call other reusable workflows
```

```yaml
# Reusable deploy workflow calls another reusable workflow
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        type: string
        required: true

jobs:
  deploy:
    uses: my-org/shared-workflows/.github/workflows/reusable-k8s-deploy.yml@v1
    with:
      environment: ${{ inputs.environment }}
    secrets: inherit
```

---

## Organization-Wide Standard Pipeline

```yaml
# my-org/shared-workflows/.github/workflows/standard-node-pipeline.yml
name: Standard Node.js Pipeline

on:
  workflow_call:
    inputs:
      app-name:
        type: string
        required: true
      node-version:
        type: string
        default: '20'
      deploy-environments:
        type: string
        default: '["staging"]'
        description: 'JSON array of environments'
    secrets:
      registry-token:
        required: true

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --ci
      - run: npm run build

  docker:
    needs: ci
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.registry-token }}
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}/${{ inputs.app-name }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}

  deploy:
    needs: docker
    strategy:
      matrix:
        environment: ${{ fromJson(inputs.deploy-environments) }}
    runs-on: ubuntu-latest
    environment: ${{ matrix.environment }}
    steps:
      - run: |
          echo "Deploying ${{ inputs.app-name }} to ${{ matrix.environment }}"
```

### Calling the Standard Pipeline

```yaml
# Any app repo — Jenkinsfile-equivalent simplicity
name: CI/CD
on:
  push:
    branches: [main]

jobs:
  pipeline:
    uses: my-org/shared-workflows/.github/workflows/standard-node-pipeline.yml@v1
    with:
      app-name: user-service
      node-version: '20'
      deploy-environments: '["staging", "production"]'
    secrets:
      registry-token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Real-World Scenario

> **Scenario**: An organization with 30 microservices wants consistent CI/CD without duplicating workflow code.
>
> **Solution**: Create a `shared-workflows` repository with reusable workflows: `reusable-ci.yml`, `reusable-docker.yml`, `reusable-deploy.yml`, and a combined `standard-node-pipeline.yml`. Each app repo's CI workflow is 10-15 lines calling the shared workflow with app-specific inputs. Updates to CI logic happen in one place and apply across all 30 services.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| "workflow was not found" | Wrong path or ref | Verify `org/repo/.github/workflows/file.yml@ref` exists |
| Secrets not passed | Missing `secrets:` block | Add `secrets: inherit` or list each secret explicitly |
| Cannot access outputs | Wrong output reference | Use `${{ needs.job-name.outputs.key }}` |
| "maximum depth exceeded" | >4 levels of nesting | Flatten workflow chain; reduce nesting |
| Permissions error | Caller lacks permissions | Set `permissions:` in calling workflow |
| Input type mismatch | String vs boolean mismatch | Ensure input types match between caller and callee |

---

## Summary Table

| Feature | Detail |
|---|---|
| **Trigger** | `on: workflow_call` |
| **Inputs** | `string`, `boolean`, `number` with defaults |
| **Secrets** | Explicit list or `secrets: inherit` |
| **Outputs** | Job outputs bubbled up to caller |
| **Nesting** | Up to 4 levels deep, 20 calls per file |
| **Versioning** | `@v1`, `@main`, or `@sha` |
| **Scope** | Same repo (`./.github/...`) or cross-repo (`org/repo/...@ref`) |

---

## Quick Revision Questions

1. **What makes a workflow reusable?** *Adding `on: workflow_call` as a trigger — this allows other workflows to call it using `uses:`.*
2. **How do you pass secrets to a reusable workflow?** *Either list each secret explicitly or use `secrets: inherit` to pass all available secrets.*
3. **What is the difference between reusable workflows and composite actions?** *Reusable workflows are job-level reuse (entire jobs with runners), while composite actions are step-level reuse (within a single job).*
4. **How do you version reusable workflows?** *Reference them with a Git ref: `@v1` (tag), `@main` (branch), or `@sha` (commit hash).*
5. **How deep can reusable workflows be nested?** *Up to 4 levels deep, with a maximum of 20 reusable workflow calls per workflow file.*
6. **How do you access outputs from a reusable workflow?** *Define outputs in the reusable workflow's `on.workflow_call.outputs` section, then access via `${{ needs.job-name.outputs.key }}`.*

---

[← Previous: Matrix Builds](05-matrix-builds.md) | [Back to README](../README.md) | [Next: Environments & Deployments →](07-environments-deployments.md)
