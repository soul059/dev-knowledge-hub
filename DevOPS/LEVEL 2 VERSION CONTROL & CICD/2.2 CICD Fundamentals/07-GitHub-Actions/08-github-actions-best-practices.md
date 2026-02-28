# Chapter 7.8: GitHub Actions Best Practices

[← Previous: Environments & Deployments](07-environments-deployments.md) | [Back to README](../README.md) | [Next: GitLab CI/CD Configuration →](../08-GitLab-CICD/01-gitlab-ci-yml.md)

---

## Overview

This chapter consolidates best practices for writing maintainable, secure, and efficient GitHub Actions workflows.

---

## Workflow Organization

```
  RECOMMENDED STRUCTURE
  ═════════════════════

  .github/
  ├── workflows/
  │   ├── ci.yml                  ← Lint + test on PR/push
  │   ├── deploy.yml              ← Deploy on merge to main
  │   ├── release.yml             ← Create releases on tag
  │   ├── nightly.yml             ← Scheduled tests
  │   └── cleanup.yml             ← Stale branch/artifact cleanup
  ├── actions/
  │   ├── setup-project/          ← Local composite action
  │   │   └── action.yml
  │   └── notify/
  │       └── action.yml
  └── dependabot.yml              ← Keep actions updated
```

---

## Security Best Practices

```
  SECURITY CHECKLIST
  ══════════════════

  ✓ Pin actions to SHA (not mutable tags)
  ✓ Set minimum GITHUB_TOKEN permissions
  ✓ Use OIDC for cloud auth (no long-lived keys)
  ✓ Never echo secrets
  ✓ Review third-party actions before using
  ✓ Use Dependabot for action updates
  ✓ Restrict environment deployments to specific branches
  ✕ Don't use pull_request_target without careful review
  ✕ Don't run untrusted code with elevated permissions
```

```yaml
# Secure workflow template
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

# Minimum permissions at workflow level
permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      # Pin to SHA, not tag
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.7
      - uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8 # v4.0.2
        with:
          node-version: '20'
      - run: npm ci && npm test
```

---

## Performance Optimization

### Caching

```yaml
# Smart caching strategy
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Node modules cache
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'             # Built-in cache support

      # Custom cache for other tools
      - uses: actions/cache@v4
        with:
          path: |
            ~/.cache/playwright
            .next/cache
          key: ${{ runner.os }}-tools-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-tools-

      - run: npm ci
      - run: npm test
```

### Minimizing Runner Time

```yaml
jobs:
  # Skip CI for docs-only changes
  changes:
    runs-on: ubuntu-latest
    outputs:
      src: ${{ steps.filter.outputs.src }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            src:
              - 'src/**'
              - 'package.json'
              - 'package-lock.json'

  test:
    needs: changes
    if: needs.changes.outputs.src == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  # Cancel previous runs on same branch
  ci:
    runs-on: ubuntu-latest
    concurrency:
      group: ci-${{ github.ref }}
      cancel-in-progress: true
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

### Parallel Jobs

```
  SEQUENTIAL (SLOW)            PARALLEL (FAST)
  ═════════════════            ════════════════

  lint ──► test ──► build      lint  ─┐
  Total: 3 + 5 + 2 = 10 min   test  ─┼──► build
                               audit ─┘
                               Total: max(3,5,2) + 2 = 7 min
```

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run lint

  test:
    runs-on: ubuntu-latest    # Parallel with lint
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  audit:
    runs-on: ubuntu-latest    # Parallel with lint and test
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  build:
    needs: [lint, test, audit]  # Only after all pass
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
```

---

## Error Handling

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Retry flaky operations
      - uses: nick-fields/retry@v3
        with:
          timeout_minutes: 5
          max_attempts: 3
          command: npm run test:integration

      # Continue on non-critical failure
      - name: Optional quality check
        continue-on-error: true
        run: npm run lint:strict

      # Always run cleanup
      - name: Cleanup
        if: always()
        run: docker compose down

      # Run only on failure
      - name: Notify on failure
        if: failure()
        run: |
          curl -X POST "${{ secrets.SLACK_WEBHOOK }}" \
            -d '{"text":"CI failed: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}'
```

---

## Artifact Best Practices

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build

      # Upload with retention period
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
          retention-days: 7          # Don't keep forever
          if-no-files-found: error   # Fail if no files

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      - run: ./deploy.sh dist/
```

---

## Workflow Patterns

### Monorepo CI

```yaml
name: Monorepo CI

on:
  pull_request:

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.changes.outputs.frontend }}
      backend: ${{ steps.changes.outputs.backend }}
      infra: ${{ steps.changes.outputs.infra }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: changes
        with:
          filters: |
            frontend: 'apps/frontend/**'
            backend: 'apps/backend/**'
            infra: 'infra/**'

  frontend-ci:
    needs: detect-changes
    if: needs.detect-changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: apps/frontend
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test && npm run build

  backend-ci:
    needs: detect-changes
    if: needs.detect-changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: apps/backend
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

### Release Workflow

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - run: npm ci && npm run build

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true
          files: dist/*.tar.gz

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.ref_name }}
            ghcr.io/${{ github.repository }}:latest
```

---

## Anti-Patterns to Avoid

```
  ┌──────────────────────────────────┬──────────────────────────────────┐
  │ ANTI-PATTERN                     │ BEST PRACTICE                    │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │ Pinning to @main or @v4         │ Pin to SHA for security          │
  │ permissions: write-all          │ Set minimum required permissions │
  │ Running everything sequentially │ Parallelize independent jobs     │
  │ No caching                      │ Cache deps + build artifacts     │
  │ Building on every file change   │ Path filters for targeted builds │
  │ Huge monolithic workflow        │ Split into focused workflows     │
  │ No timeout                      │ Set timeout-minutes on jobs      │
  │ Secrets in environment          │ Use secrets context properly     │
  │ No concurrency control          │ Add concurrency groups           │
  │ Keeping all artifacts forever   │ Set retention-days               │
  └──────────────────────────────────┴──────────────────────────────────┘
```

---

## Real-World Scenario

> **Scenario**: A team's CI takes 25 minutes and runs on every push, costing excessive runner minutes.
>
> **Solution**: Parallelize lint, test, and audit (saves 5 min). Add path filters to skip for docs changes. Cache npm + Playwright. Add concurrency groups to cancel superseded runs. Pin to specific action SHAs. Set minimum `permissions:`. Result: CI drops to 8 min, 40% fewer runs, better security posture.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| CI runs for every file change | No path filters | Add `paths:` or use dorny/paths-filter |
| Slow CI | No caching, sequential jobs | Cache deps, parallelize independent jobs |
| High runner costs | No concurrency control | Add concurrency groups, cancel in-progress |
| "Token does not have permissions" | Default read-only permissions | Add `permissions:` block with needed scopes |
| Artifact too large | Uploading unnecessary files | Be specific with `path:`; add `.gitignore`-like patterns |
| Workflows hard to maintain | Duplicated logic across repos | Use reusable workflows + composite actions |

---

## Summary Table

| Category | Recommendation |
|---|---|
| **Security** | SHA-pinned actions, minimum permissions, OIDC |
| **Performance** | Caching, parallelization, path filters, concurrency |
| **Maintainability** | Reusable workflows, composite actions, small files |
| **Reliability** | Timeouts, retries, error notifications |
| **Cost** | Concurrency groups, path filters, Linux runners |
| **Organization** | Separate workflows per concern (CI, deploy, release) |

---

## Quick Revision Questions

1. **Why pin actions to a SHA instead of a tag?** *Tags are mutable — a compromised action could be re-tagged. SHAs are immutable, ensuring the exact code you reviewed is what runs.*
2. **How do path filters improve CI efficiency?** *They skip workflow runs when only irrelevant files change (e.g., docs), reducing unnecessary builds and runner costs.*
3. **What does `concurrency: cancel-in-progress: true` do?** *Cancels the currently running workflow for the same concurrency group when a new run starts — preventing wasted runner time on superseded commits.*
4. **When should you use `continue-on-error: true`?** *For non-critical steps that shouldn't block the pipeline — e.g., optional quality checks or experimental tests.*
5. **How should artifacts be managed?** *Set `retention-days` to avoid storing forever, use specific paths, and fail with `if-no-files-found: error` to catch build issues.*
6. **What is the recommended way to structure workflows for a monorepo?** *Use path filters to detect changed services, then conditionally run CI only for affected services using `if:` conditions on job outputs.*

---

[← Previous: Environments & Deployments](07-environments-deployments.md) | [Back to README](../README.md) | [Next: GitLab CI/CD Configuration →](../08-GitLab-CICD/01-gitlab-ci-yml.md)
