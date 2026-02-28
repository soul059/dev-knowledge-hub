# Chapter 8.5: GitLab Artifacts & Caching

[← Previous: Variables & Secrets](04-variables-secrets.md) | [Back to README](../README.md) | [Next: Environments & Review Apps →](06-environments-review-apps.md)

---

## Overview

**Artifacts** pass files between jobs and stages. **Caching** speeds up pipelines by reusing files across pipeline runs. Understanding the difference is essential for efficient GitLab CI/CD.

---

## Artifacts vs Cache

```
  ARTIFACTS                          CACHE
  ═════════                          ═════

  Purpose:                           Purpose:
  Pass files BETWEEN JOBS            Speed up SAME JOB across runs

  Lifecycle:                         Lifecycle:
  Created per pipeline               Persistent across pipelines
  Uploaded to GitLab server           Stored on runner or distributed

  Example:                           Example:
  Build output → Deploy job          node_modules/ reused next run

  ┌─────────────────────────────────────────────────────┐
  │ Pipeline Run #1                                     │
  │   build ──(artifact: dist/)──► deploy               │
  │   test  uses cache: node_modules/ (miss → install)  │
  │                                                     │
  │ Pipeline Run #2                                     │
  │   build ──(artifact: dist/)──► deploy               │
  │   test  uses cache: node_modules/ (hit → fast!)     │
  └─────────────────────────────────────────────────────┘
```

---

## Artifacts

### Basic Artifacts

```yaml
build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
      - build/
    expire_in: 1 week            # Auto-cleanup
    name: "${CI_JOB_NAME}-${CI_COMMIT_SHORT_SHA}"
```

### Artifact Reports

```yaml
test:
  stage: test
  script:
    - npm test -- --ci --coverage
  artifacts:
    # Standard file artifacts
    paths:
      - coverage/

    # Report artifacts (parsed by GitLab UI)
    reports:
      junit: reports/junit.xml              # Test results in MR
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura.xml        # Coverage in MR diff
      dotenv: build.env                     # Pass vars between jobs
      sast: gl-sast-report.json              # Security dashboard
      dependency_scanning: gl-dependency-scanning-report.json
      container_scanning: gl-container-scanning-report.json
      license_scanning: gl-license-scanning-report.json

    # Conditional upload
    when: always                            # Upload even on failure

    expire_in: 30 days
```

### Artifact Dependencies

```yaml
build:
  stage: build
  script: npm run build
  artifacts:
    paths: [dist/]

# By default, ALL artifacts from previous stages are downloaded
deploy:
  stage: deploy
  dependencies:
    - build              # Download ONLY build's artifacts
  script: ./deploy.sh dist/

# Or with needs: (DAG)
deploy:
  stage: deploy
  needs:
    - job: build
      artifacts: true     # Explicitly download artifacts
  script: ./deploy.sh dist/

# Skip all artifact downloads
lint:
  stage: test
  dependencies: []         # Download nothing
  script: npm run lint
```

---

## Caching

### Basic Cache

```yaml
# Global cache
default:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .npm/

test:
  stage: test
  script:
    - npm ci --cache .npm
    - npm test
```

### Cache Key Strategies

```yaml
# Key by branch (separate cache per branch)
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

# Key by lockfile hash (invalidate on dependency change)
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/

# Key by prefix + files (branch-specific + dependency-aware)
cache:
  key:
    prefix: ${CI_COMMIT_REF_SLUG}
    files:
      - package-lock.json
  paths:
    - node_modules/
```

```
  CACHE KEY STRATEGIES
  ════════════════════

  Static key: "my-cache"
  └── Same cache for everything (risk: conflicts)

  Branch key: ${CI_COMMIT_REF_SLUG}
  └── Separate cache per branch (isolation)

  File key: package-lock.json hash
  └── New cache when deps change (precision)

  Combined: branch + file hash
  └── Best of both (recommended)
```

### Cache Policy

```yaml
# Populate cache in one job, use in others
install:
  stage: install
  cache:
    key:
      files: [package-lock.json]
    paths: [node_modules/]
    policy: pull-push           # Download and upload cache (default)
  script: npm ci

test:
  stage: test
  cache:
    key:
      files: [package-lock.json]
    paths: [node_modules/]
    policy: pull                # Only download, never upload
  script: npm test

lint:
  stage: test
  cache:
    key:
      files: [package-lock.json]
    paths: [node_modules/]
    policy: pull                # Only download
  script: npm run lint
```

### Fallback Keys

```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}-${CI_COMMIT_SHORT_SHA}
  fallback_keys:
    - ${CI_COMMIT_REF_SLUG}     # Try branch cache
    - main                       # Fall back to main branch cache
  paths:
    - node_modules/
```

---

## Distributed Caching

```
  LOCAL vs DISTRIBUTED CACHE
  ══════════════════════════

  LOCAL CACHE (single runner):
  Runner A: has cache ✓
  Runner B: no cache ✕  ← Cache miss if job runs here

  DISTRIBUTED CACHE (S3/GCS):
  Runner A ──► S3 bucket ◄── Runner B
                  │
  Any runner gets the cache ✓
```

```toml
# config.toml — S3 distributed cache
[[runners]]
  [runners.cache]
    Type = "s3"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      BucketName = "gitlab-ci-cache"
      BucketLocation = "us-east-1"
      AccessKey = "ACCESS_KEY"
      SecretKey = "SECRET_KEY"
```

---

## Multi-Language Caching

```yaml
variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.pip-cache"
  GOPATH: "$CI_PROJECT_DIR/.go"

python-test:
  image: python:3.12
  cache:
    key:
      files: [requirements.txt]
    paths:
      - .pip-cache/
  script:
    - pip install -r requirements.txt
    - pytest

go-build:
  image: golang:1.22
  cache:
    key:
      files: [go.sum]
    paths:
      - .go/pkg/mod/
  script:
    - go build ./...
    - go test ./...

java-build:
  image: maven:3.9-eclipse-temurin-21
  cache:
    key:
      files: [pom.xml]
    paths:
      - .m2/repository
  variables:
    MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  script:
    - mvn package
```

---

## Real-World Scenario

> **Scenario**: A team's pipeline takes 8 minutes, with 4 minutes spent on `npm install` every run.
>
> **Solution**: Add caching with `key: files: [package-lock.json]`. First job uses `policy: pull-push` to populate cache. Test and lint jobs use `policy: pull`. Configure S3 distributed cache so any runner gets the cache. Set `fallback_keys` to main branch cache for new branches. Result: `npm install` drops from 4 minutes to ~15 seconds on cache hit.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Cache miss every run | Key changes each time | Use stable key (file hash or branch slug) |
| Cache miss on different runner | Local cache only | Configure distributed cache (S3/GCS) |
| Artifacts not found in next stage | Job didn't produce artifacts | Check `artifacts: paths:` matches actual files |
| "artifact too large" | Exceeds size limit | Reduce paths; use `.gitignore`-style `exclude:` |
| Old cache causing issues | Stale dependencies | Clear cache (CI/CD → Pipelines → Clear runner caches) |
| JUnit report not showing in MR | Wrong file path or format | Verify path matches; ensure valid JUnit XML |

---

## Summary Table

| Feature | Artifacts | Cache |
|---|---|---|
| **Purpose** | Pass files between jobs | Speed up repeated builds |
| **Lifetime** | Per pipeline (with expiry) | Across pipelines |
| **Storage** | GitLab server | Runner local or S3/GCS |
| **Download** | Automatic by stage order or `needs:` | Automatic by matching key |
| **Best for** | Build outputs, test reports | Dependencies (node_modules, .m2) |
| **Reports** | JUnit, coverage, SAST, DAST | N/A |
| **Policy** | N/A | pull, push, pull-push |

---

## Quick Revision Questions

1. **What is the difference between artifacts and cache?** *Artifacts pass files between jobs within a pipeline (short-lived). Cache reuses files across pipeline runs to speed up repeated operations (persistent).*
2. **How do you prevent stale caches?** *Use file-based cache keys (e.g., `key: files: [package-lock.json]`) so the cache invalidates when dependencies change.*
3. **What is the `policy: pull` cache option?** *The job only downloads the cache but never uploads changes — useful for jobs that shouldn't modify the cached dependencies.*
4. **How do artifact reports work?** *Special artifact types (JUnit, Cobertura, SAST) are parsed by GitLab and displayed in the merge request UI — test results, coverage diffs, and security findings.*
5. **What is distributed caching?** *Storing cache in external storage (S3, GCS) instead of locally on the runner, so any runner can access the same cache.*
6. **How do you control which artifacts a job downloads?** *Use `dependencies:` to list specific jobs, or `dependencies: []` to download none. With DAG, use `needs: [{job: name, artifacts: true}]`.*

---

[← Previous: Variables & Secrets](04-variables-secrets.md) | [Back to README](../README.md) | [Next: Environments & Review Apps →](06-environments-review-apps.md)
