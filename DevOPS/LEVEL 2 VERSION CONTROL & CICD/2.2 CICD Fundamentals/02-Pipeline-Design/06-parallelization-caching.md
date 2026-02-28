# Chapter 2.6: Parallelization and Caching

[← Previous: Pipeline Triggers](05-pipeline-triggers.md) | [Back to README](../README.md) | [Next: Build Tools Overview →](../03-Build-Automation/01-build-tools-overview.md)

---

## Overview

Pipeline speed is critical — slow pipelines discourage frequent commits. **Parallelization** runs independent jobs simultaneously, while **caching** avoids redundant work. Together, they can reduce pipeline duration by 50-80%.

---

## Parallelization

### Sequential vs Parallel

```
SEQUENTIAL (Slow: 25 min)
═════════════════════════
  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
  │Build │▶│ Unit │▶│ Lint │▶│Integ │▶│Deploy│
  │5 min │ │8 min │ │2 min │ │5 min │ │5 min │
  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘
  ├──────────────── 25 min ───────────────────┤

PARALLEL (Fast: 18 min)
═══════════════════════
                 ┌──────┐
                 │ Unit │ 8 min
                 │ Test │
  ┌──────┐      ├──────┤        ┌──────┐
  │Build │──────│ Lint │ 2 min──│Deploy│
  │5 min │      ├──────┤        │5 min │
  └──────┘      │Integ │ 5 min  └──────┘
                │ Test │
                └──────┘
  ├──── 5 ────┤├── 8 (max) ──┤├── 5 ──┤
  ├──────────────── 18 min ──────────────┤
  Saved: 7 min (28% faster)
```

### GitHub Actions — Parallel Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/ }

  # These three jobs run IN PARALLEL
  unit-test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  lint:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run lint

  security:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  # This job waits for ALL parallel jobs
  deploy:
    needs: [unit-test, lint, security]
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

### Test Splitting / Sharding

```
BEFORE: One test runner (20 min)
════════════════════════════════
  Runner 1: [Test1, Test2, Test3, ... Test100]  → 20 min

AFTER: Sharded across 4 runners (5 min)
════════════════════════════════════════
  Runner 1: [Test1  - Test25]  → 5 min  ┐
  Runner 2: [Test26 - Test50]  → 5 min  ├─ All run in parallel
  Runner 3: [Test51 - Test75]  → 5 min  │
  Runner 4: [Test76 - Test100] → 5 min  ┘
```

```yaml
# GitHub Actions — Matrix for test sharding
jobs:
  test:
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --shard=${{ matrix.shard }}/4

# GitLab CI — Parallel keyword
test:
  parallel: 4
  script:
    - npm test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
```

---

## Caching

### What to Cache

```
┌──────────────────────────────────────────────────────────────┐
│                   WHAT TO CACHE                               │
│                                                               │
│  ┌────────────────────┐  ┌────────────────────┐             │
│  │   DEPENDENCIES     │  │   BUILD OUTPUTS    │             │
│  │                    │  │                    │             │
│  │  node_modules/     │  │  .next/cache/      │             │
│  │  ~/.m2/repository/ │  │  target/           │             │
│  │  ~/.cache/pip/     │  │  dist/             │             │
│  │  vendor/           │  │  .gradle/          │             │
│  │  ~/.cargo/         │  │                    │             │
│  └────────────────────┘  └────────────────────┘             │
│                                                               │
│  ┌────────────────────┐  ┌────────────────────┐             │
│  │   DOCKER LAYERS    │  │   TEST DATA        │             │
│  │                    │  │                    │             │
│  │  Docker build      │  │  Fixtures          │             │
│  │  layer cache       │  │  Generated data    │             │
│  │                    │  │  Browser binaries   │             │
│  └────────────────────┘  └────────────────────┘             │
└──────────────────────────────────────────────────────────────┘
```

### Cache Flow

```
FIRST RUN (cache miss):
════════════════════════
  ┌──────────┐   miss   ┌──────────┐   install   ┌──────────┐
  │ Check    │─────────▶│ Install  │────────────▶│  Save    │
  │ Cache    │          │ Deps     │             │  Cache   │
  └──────────┘          │ (slow)   │             └──────────┘
                        │ 2-5 min  │
                        └──────────┘

SUBSEQUENT RUNS (cache hit):
═══════════════════════════
  ┌──────────┐   hit    ┌──────────┐
  │ Check    │─────────▶│ Restore  │  Done! (5-15 seconds)
  │ Cache    │          │ Cache    │
  └──────────┘          └──────────┘
```

### GitHub Actions Caching

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Cache npm dependencies
      - uses: actions/cache@v4
        with:
          path: ~/.npm
          key: npm-${{ hashFiles('package-lock.json') }}
          restore-keys: npm-

      - run: npm ci
      - run: npm run build

      # Alternative: setup-node has built-in caching
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'        # Auto-caches ~/.npm
```

### GitLab CI Caching

```yaml
test:
  image: node:20-alpine
  cache:
    key:
      files:
        - package-lock.json        # Cache key based on lockfile
    paths:
      - node_modules/              # What to cache
    policy: pull-push              # Pull on start, push on success
  script:
    - npm ci
    - npm test
```

### Jenkins Caching

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Using stash/unstash for artifacts between stages
                sh 'npm ci'
                sh 'npm run build'
                stash includes: 'dist/**', name: 'build-output'
            }
        }
        stage('Test') {
            steps {
                unstash 'build-output'
                sh 'npm test'
            }
        }
    }
}
```

---

## Cache Key Strategies

```
┌──────────────────────────────────────────────────────────────────┐
│               CACHE KEY STRATEGIES                                │
│                                                                   │
│  Strategy          │ Key Pattern               │ Invalidation    │
│  ══════════════════╪═══════════════════════════╪═════════════════│
│  Lock file hash    │ npm-${{ hash(lock) }}     │ On dep change   │
│  Branch + lockfile │ $branch-${{ hash(lock) }} │ Per branch      │
│  Weekly rotation   │ npm-week-${{ week }}      │ Every Monday    │
│  Exact match only  │ exact-${{ hash(all) }}    │ Any file change │
│                                                                   │
│  Best Practice: Use lockfile hash as primary key with            │
│  fallback to partial match (restore-keys)                        │
└──────────────────────────────────────────────────────────────────┘
```

```yaml
# Multi-level cache keys (GitHub Actions)
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      npm-${{ runner.os }}-     # Fallback: same OS, any lockfile
      npm-                       # Last resort: any npm cache
```

---

## Docker Layer Caching

```dockerfile
# Dockerfile optimized for layer caching
# ─── Layer 1: Base image (rarely changes) ───
FROM node:20-alpine

WORKDIR /app

# ─── Layer 2: Dependencies (changes occasionally) ───
COPY package.json package-lock.json ./
RUN npm ci --production

# ─── Layer 3: Source code (changes frequently) ───
COPY src/ ./src/

# ─── Layer 4: Build ───
RUN npm run build

CMD ["node", "dist/server.js"]
```

```yaml
# GitHub Actions — Docker layer caching
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: myapp:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

## Performance Comparison

```
Pipeline Duration Comparison
═══════════════════════════════════════════════

  No optimization:     ████████████████████████████████ 30 min
  + Caching:           ██████████████████████           21 min (-30%)
  + Parallelization:   ████████████████                 15 min (-50%)
  + Caching + Parallel:████████                          8 min (-73%)
  + Test sharding:     █████                             5 min (-83%)
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Cache never hits | Incorrect cache key | Check hash function, verify path |
| Cache restores stale data | Key too broad | Use lockfile hash for precision |
| Parallel jobs fail randomly | Shared resource conflicts | Isolate test databases, use unique ports |
| Test sharding uneven | Unbalanced distribution | Use timing-based sharding (e.g., CircleCI) |
| Docker build slow | No layer caching | Optimize Dockerfile order, enable BuildKit cache |

---

## Summary Table

| Technique | Benefit | Typical Savings |
|---|---|---|
| **Job Parallelization** | Run independent jobs concurrently | 20-50% time reduction |
| **Test Sharding** | Split tests across multiple runners | 60-80% test time reduction |
| **Dependency Caching** | Skip re-downloading deps | 1-5 min saved per run |
| **Build Caching** | Skip recompilation of unchanged code | 30-60% build time |
| **Docker Layer Cache** | Reuse unchanged image layers | 50-80% image build time |
| **Matrix Builds** | Test across versions in parallel | Multiplied coverage, same time |

---

## Quick Revision Questions

1. **What is the difference between parallelization and caching in CI/CD?**  
   *Parallelization runs independent tasks simultaneously; caching avoids repeating work by storing and reusing previous results.*

2. **What is test sharding and when should you use it?**  
   *Splitting a test suite across multiple runners to run in parallel. Use when the test suite is the pipeline bottleneck (>10-15 min).*

3. **What makes a good cache key?**  
   *A hash of the lockfile (package-lock.json, Gemfile.lock) — it changes exactly when dependencies change, with fallback keys for partial matches.*

4. **Why is Dockerfile layer order important for caching?**  
   *Docker caches layers sequentially; changing an early layer invalidates all subsequent layers. Put rarely-changing layers first (base image, deps).*

5. **How much total pipeline time reduction can you typically achieve with caching + parallelization?**  
   *50-80% reduction is typical when combining dependency caching, job parallelization, and test sharding.*

6. **What is a cache restore-key (fallback key)?**  
   *A partial key prefix that matches older cache entries when the exact key misses, providing a stale-but-useful cache as a starting point.*

---

[← Previous: Pipeline Triggers](05-pipeline-triggers.md) | [Back to README](../README.md) | [Next: Build Tools Overview →](../03-Build-Automation/01-build-tools-overview.md)
