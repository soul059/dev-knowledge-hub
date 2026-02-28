# Chapter 3.6: Build Caching Strategies

[← Previous: Multi-Stage Builds](05-multi-stage-builds.md) | [Back to README](../README.md) | [Next: Test Pyramid →](../04-Testing-In-CICD/01-test-pyramid.md)

---

## Overview

Build caching stores intermediate results so subsequent builds skip redundant work. Effective caching can reduce build times by **50-90%**. This chapter covers caching at every level: dependencies, compilation, Docker layers, and CI platform caches.

---

## Caching Layers

```
┌──────────────────────────────────────────────────────────────┐
│              CACHING LAYERS                                   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Level 1: CI Platform Cache                           │   │
│  │  actions/cache, GitLab cache:, Jenkins stash         │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │ Level 2: Docker Layer Cache                  │   │   │
│  │  │  BuildKit cache, registry cache              │   │   │
│  │  │  ┌──────────────────────────────────────┐   │   │   │
│  │  │  │ Level 3: Build Tool Cache            │   │   │   │
│  │  │  │  Gradle cache, tsc incremental,      │   │   │   │
│  │  │  │  webpack cache, .cargo/registry      │   │   │   │
│  │  │  │  ┌──────────────────────────────┐   │   │   │   │
│  │  │  │  │ Level 4: Dependency Cache    │   │   │   │   │
│  │  │  │  │  node_modules, ~/.m2,        │   │   │   │   │
│  │  │  │  │  ~/.cache/pip, vendor/       │   │   │   │   │
│  │  │  │  └──────────────────────────────┘   │   │   │   │
│  │  │  └──────────────────────────────────────┘   │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## CI Platform Caching

### GitHub Actions

```yaml
# Cache multiple directories
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ~/.cache/puppeteer
      node_modules
    key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-deps-
```

### GitLab CI

```yaml
build:
  cache:
    - key:
        files: [package-lock.json]
      paths: [node_modules/]
      policy: pull-push
    - key: build-cache
      paths: [.next/cache/]
      policy: pull-push
```

### Jenkins

```groovy
pipeline {
    agent any
    options {
        // Use workspace caching plugin
        cache(maxCacheSize: 500, defaultBranch: 'main',
              caches: [
                  arbitraryFileCache(path: 'node_modules',
                      cacheValidityDecidingFile: 'package-lock.json')
              ])
    }
}
```

---

## Docker Build Cache

### BuildKit Cache Mounts

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./

# Cache npm packages across builds
RUN --mount=type=cache,target=/root/.npm \
    npm ci

COPY . .
RUN npm run build
```

### Remote Cache (Registry-Based)

```yaml
# GitHub Actions — Docker cache to registry
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: myapp:latest
    cache-from: type=registry,ref=registry.example.com/myapp:cache
    cache-to: type=registry,ref=registry.example.com/myapp:cache,mode=max

# Or use GitHub Actions cache backend
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

## Cache Key Design

```
GOOD CACHE KEYS
════════════════

  Purpose                Key Pattern                         Invalidates When
  ──────────────────────────────────────────────────────────────────────────
  Dependencies           deps-{os}-{hash(lockfile)}          Lockfile changes
  Build output           build-{os}-{hash(src/**)}           Source changes  
  Docker layers          layer-{hash(Dockerfile)}            Dockerfile changes
  Test fixtures          fixtures-{hash(test/fixtures/**)}   Fixtures change

BAD CACHE KEYS
═══════════════

  ❌ cache-key (static — never invalidates)
  ❌ cache-{timestamp} (always misses — new key every time)
  ❌ cache-{branch} (shared across commits — stale data)
```

---

## Cache Warming

```yaml
# Pre-warm cache for new branches by falling back to main cache
- uses: actions/cache@v4
  with:
    path: node_modules
    key: deps-${{ github.ref }}-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      deps-${{ github.ref }}-                # Same branch, any lockfile
      deps-refs/heads/main-                   # Fall back to main branch
      deps-                                   # Last resort
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Cache never hits | Key too specific | Add restore-keys for partial matches |
| Cache restores stale data | Key too broad | Use content hash (lockfile, source) |
| Cache size exceeds limits | Caching too much | Cache only essential dirs, set limits |
| Corrupted cache causes failures | Bad cached state | Add periodic clean builds (nightly) |
| Docker cache not working in CI | BuildKit not enabled | Set `DOCKER_BUILDKIT=1` |

---

## Summary Table

| Cache Level | What's Cached | Key Strategy | Savings |
|---|---|---|---|
| **Dependencies** | node_modules, ~/.m2 | Hash of lockfile | 1-5 min |
| **Build output** | dist/, target/ | Hash of source | 30-80% |
| **Docker layers** | Image layers | Layer content hash | 50-90% |
| **Build tool** | Gradle/.tsc cache | Hash of config | 20-50% |
| **CI platform** | All above | Composite keys | Cumulative |

---

## Quick Revision Questions

1. **What are the four levels of build caching?** *CI platform cache, Docker layer cache, build tool cache, and dependency cache.*
2. **What makes a good cache key?** *A content-based hash (e.g., lockfile hash) that changes exactly when the cached content needs updating.*
3. **What is a restore-key / fallback key?** *A partial key prefix that matches older caches when the exact key misses, providing a stale but usable starting point.*
4. **How does `--mount=type=cache` work in Docker?** *It mounts a persistent cache directory that survives across builds, avoiding re-downloading packages.*
5. **Why should you run clean builds periodically?** *To catch issues hidden by stale caches and ensure builds work from scratch.*
6. **What is the typical total build time reduction from effective caching?** *50-90% when combining all caching layers.*

---

[← Previous: Multi-Stage Builds](05-multi-stage-builds.md) | [Back to README](../README.md) | [Next: Test Pyramid →](../04-Testing-In-CICD/01-test-pyramid.md)
