# Chapter 3.4: Build Optimization

[← Previous: Dependency Management](03-dependency-management.md) | [Back to README](../README.md) | [Next: Multi-Stage Builds →](05-multi-stage-builds.md)

---

## Overview

Build optimization reduces pipeline execution time. Faster builds mean faster feedback, more frequent deployments, and happier developers. The target: **CI feedback in under 10 minutes**.

---

## Optimization Strategies

```
┌──────────────────────────────────────────────────────────────────┐
│            BUILD OPTIMIZATION STRATEGIES                          │
│                                                                   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │
│  │   CACHING   │ │ INCREMENTAL │ │  PARALLEL   │               │
│  │             │ │   BUILDS    │ │  EXECUTION  │               │
│  │ Deps, layers│ │ Only rebuild│ │ Independent │               │
│  │ build output│ │ what changed│ │ tasks at    │               │
│  │             │ │             │ │ same time   │               │
│  └─────────────┘ └─────────────┘ └─────────────┘               │
│                                                                   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │
│  │  MINIMIZE   │ │  OPTIMIZE   │ │   SMART     │               │
│  │  ARTIFACT   │ │   TESTS     │ │  TRIGGERS   │               │
│  │             │ │             │ │             │               │
│  │ Smaller =   │ │ Run only    │ │ Build only  │               │
│  │ faster push │ │ affected    │ │ what changed│               │
│  │ & pull      │ │ tests       │ │             │               │
│  └─────────────┘ └─────────────┘ └─────────────┘               │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Dependency Caching

```yaml
# GitHub Actions — Cache node_modules
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      node_modules
    key: deps-${{ hashFiles('package-lock.json') }}
    restore-keys: deps-

# Impact:
# Without cache:  npm ci takes ~90 seconds
# With cache:     npm ci takes ~10 seconds
```

## 2. Incremental Builds

```bash
# Gradle — incremental by default
gradle build    # First build: 45s
gradle build    # Second (no changes): 2s

# TypeScript — incremental compilation
tsc --incremental    # Only recompiles changed files
# tsconfig.json: { "compilerOptions": { "incremental": true } }

# Docker BuildKit — only rebuilds changed layers
DOCKER_BUILDKIT=1 docker build .
```

## 3. Minimize Build Context

```bash
# .dockerignore — exclude unnecessary files from Docker build
node_modules
.git
*.md
tests/
docs/
.env

# Impact: Build context from 500MB → 5MB = much faster
```

## 4. Use Smaller Base Images

```
Image Size Comparison
═════════════════════
  node:20         ████████████████████████████████  ~1.1 GB
  node:20-slim    ████████████████                  ~200 MB
  node:20-alpine  ██████                            ~130 MB
  distroless      ████                              ~80 MB
```

## 5. Optimize Test Execution

```yaml
# Run only affected tests (Jest)
- run: npx jest --changedSince=origin/main

# Parallel test execution
- run: npx jest --maxWorkers=4

# Test sharding across runners
strategy:
  matrix:
    shard: [1, 2, 3, 4]
steps:
  - run: npx jest --shard=${{ matrix.shard }}/4
```

---

## Build Time Reduction Example

```
BEFORE OPTIMIZATION: 28 minutes
════════════════════════════════

  Install deps ████████ 5 min
  Lint         ██       1 min
  Unit tests   ████████████████ 10 min
  Build        ████████ 5 min
  Docker build ██████   4 min
  Push image   ██████   3 min
  Total:       28 min

AFTER OPTIMIZATION: 8 minutes
══════════════════════════════

  Install (cached) █ 15s
                    ┌─ Lint       █ 1 min ─┐
  Parallel:         ├─ Unit tests ████ 4 min ├─ All parallel
                    └─ Security   ██ 2 min ─┘
  Build (incremental) ██ 1.5 min
  Docker (cached)     █ 1 min
  Push (smaller image) █ 30s
  Total:              8 min (-71%)
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| CI build 3x slower than local | Different hardware, no cache | Enable caching, use larger runners |
| Cache hit rate is low | Bad cache key | Use lockfile hash, add restore-keys |
| Incremental build produces wrong output | Stale cache | Clean build periodically (nightly) |
| Docker build slow despite cache | Wrong layer ordering | Put frequently changing layers last |

---

## Summary Table

| Technique | Typical Savings | Effort |
|---|---|---|
| Dependency caching | 1-5 min/build | Low |
| Incremental compilation | 30-80% build time | Low |
| Parallel test execution | 50-75% test time | Medium |
| Smaller Docker images | 50-80% push/pull time | Low |
| Build context optimization | 10-30% Docker build time | Low |
| Smart triggers (path filters) | Skip unnecessary builds | Low |

---

## Quick Revision Questions

1. **What is the target CI feedback time?** *Under 10 minutes.*
2. **How does dependency caching speed up builds?** *By reusing previously downloaded dependencies instead of re-downloading every time.*
3. **What is an incremental build?** *Only recompiling files that have changed since the last build, skipping unchanged code.*
4. **Why use alpine-based Docker images?** *They're much smaller (~130MB vs ~1.1GB), making builds and deployments faster.*
5. **How can test execution be optimized?** *Parallelization, test sharding, running only affected tests, and caching test dependencies.*
6. **What is the typical build time reduction achievable with optimization?** *50-80% reduction by combining caching, parallelization, and incremental builds.*

---

[← Previous: Dependency Management](03-dependency-management.md) | [Back to README](../README.md) | [Next: Multi-Stage Builds →](05-multi-stage-builds.md)
