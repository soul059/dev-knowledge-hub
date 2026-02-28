# Chapter 3.1 — Continuous Integration (CI)

[← Previous: Fail Fast, Recover Fast](../02-DevOps-Principles/05-fail-fast-recover-fast.md) | [Next: Continuous Delivery →](02-continuous-delivery.md)

---

## Overview

**Continuous Integration (CI)** is the practice of frequently merging code changes into a shared repository, where each merge triggers an automated build and test process. CI ensures that the codebase is always in a working state.

---

## What is Continuous Integration?

```
┌──────────────────────────────────────────────────────────┐
│  CONTINUOUS INTEGRATION                                  │
│                                                          │
│  Developer A ──commit──┐                                 │
│  Developer B ──commit──┤──► Shared Repo ──► CI Server    │
│  Developer C ──commit──┘       (main)          │         │
│                                                │         │
│                                    ┌───────────┘         │
│                                    ▼                     │
│                          ┌─────────────────┐             │
│                          │   CI PIPELINE   │             │
│                          │                 │             │
│                          │  1. Checkout    │             │
│                          │  2. Build       │             │
│                          │  3. Unit Test   │             │
│                          │  4. Lint/Format │             │
│                          │  5. SAST Scan   │             │
│                          │  6. Artifact    │             │
│                          └────────┬────────┘             │
│                                   │                      │
│                        ┌──────────┴──────────┐           │
│                        ▼                     ▼           │
│                    ┌───────┐             ┌───────┐       │
│                    │  ✅   │             │  ❌   │       │
│                    │ PASS  │             │ FAIL  │       │
│                    └───────┘             └───┬───┘       │
│                    Merge allowed         Notify dev      │
│                                         Block merge     │
└──────────────────────────────────────────────────────────┘
```

---

## CI Best Practices

### 1. Commit Frequently (at least daily)

```
  BAD: Long-lived branches
  ═══
  main     ●────────────────────────────────●  (merge hell)
            \                              /
  feature    ●──●──●──●──●──●──●──●──●──●─●   (3 weeks!)
  
  GOOD: Short-lived branches
  ═════
  main     ●──●──●──●──●──●──●──●──●──●
            \/ \/ \/ \/ \/ \/ \/ \/ \/
  features  ●  ●  ●  ●  ●  ●  ●  ●  ●    (hours, not weeks)
```

### 2. Automated Build and Test on Every Commit

```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Unit tests
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

### 3. Keep the Build Fast (<10 minutes)

```
┌──────────────────────────────────────────────┐
│  BUILD TIME OPTIMIZATION                     │
│                                              │
│  Slow Build (30 min):                        │
│  ┌────┬──────┬──────────┬────────┬──────┐   │
│  │Deps│Build │Unit Tests│Int.Test│Report│   │
│  │5m  │5m    │8m        │10m     │2m    │   │
│  └────┴──────┴──────────┴────────┴──────┘   │
│  Total: 30 minutes (sequential)              │
│                                              │
│  Fast Build (8 min):                         │
│  ┌────┬──────┐                               │
│  │Deps│Build │ (cached deps, 3 min)          │
│  └────┴──┬───┘                               │
│          │ (parallel)                        │
│   ┌──────┼──────┐                            │
│   ▼      ▼      ▼                            │
│  ┌────┐┌────┐┌────┐                         │
│  │Unit││Lint││SAST│ (5 min, parallel)        │
│  └────┘└────┘└────┘                         │
│  Total: 8 minutes                            │
└──────────────────────────────────────────────┘
```

### 4. Fix Broken Builds Immediately

```
  Priority when CI is red:
  
  1. ████████████████████████  FIX THE BUILD (highest)
  2. ████████████████         Current sprint work
  3. ████████████             Code reviews
  4. ████████                 Meetings
  5. ████                     Tech debt (lowest)

  Rule: "The build is everyone's responsibility.
         A red build is a team emergency."
```

### 5. Trunk-Based Development

```
┌──────────────────────────────────────────────────────────┐
│  TRUNK-BASED DEVELOPMENT                                 │
│                                                          │
│  main (trunk) ●──●──●──●──●──●──●──●──●──●──●──●       │
│                \/    \/    \/    \/    \/    \/          │
│  short-lived   ●     ●     ●     ●     ●     ●          │
│  branches     (hrs) (hrs) (hrs) (hrs) (hrs) (hrs)        │
│                                                          │
│  Rules:                                                  │
│  ├── Branches live < 1 day                               │
│  ├── Small, focused PRs (< 200 lines)                    │
│  ├── Feature flags for incomplete features               │
│  ├── Main is always deployable                           │
│  └── All tests pass before merge                         │
└──────────────────────────────────────────────────────────┘
```

---

## CI Pipeline Architecture

```
┌──────────────────────────────────────────────────────────┐
│  TYPICAL CI PIPELINE                                     │
│                                                          │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐ │
│  │Commit│──►│Build │──►│Test  │──►│Scan  │──►│Store │ │
│  │      │   │      │   │      │   │      │   │      │ │
│  └──────┘   └──────┘   └──────┘   └──────┘   └──────┘ │
│     │          │          │          │          │        │
│     ▼          ▼          ▼          ▼          ▼        │
│  Pull code  Compile    Unit &     SAST      Push to    │
│  from Git   Resolve    Integr.   SCA       artifact   │
│             deps       tests     License   registry   │
│             Docker               check                 │
│             build                                      │
│                                                          │
│  Quality Gates (must pass to proceed):                   │
│  ├── ✅ All tests pass                                   │
│  ├── ✅ Code coverage ≥ 80%                              │
│  ├── ✅ No critical vulnerabilities                      │
│  ├── ✅ Lint rules pass                                  │
│  └── ✅ Build succeeds                                   │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Setting Up CI for a Python Project

```bash
# Project structure
my-python-app/
├── src/
│   ├── __init__.py
│   ├── app.py
│   └── utils.py
├── tests/
│   ├── test_app.py
│   └── test_utils.py
├── Dockerfile
├── requirements.txt
├── pyproject.toml
└── .github/
    └── workflows/
        └── ci.yml
```

```yaml
# .github/workflows/ci.yml
name: Python CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.11', '3.12']

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov flake8 black mypy

      - name: Lint with flake8
        run: flake8 src/ --max-line-length=120

      - name: Check formatting with black
        run: black --check src/

      - name: Type check with mypy
        run: mypy src/

      - name: Run tests with coverage
        run: pytest tests/ --cov=src --cov-report=xml --cov-fail-under=80

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Build takes 30+ minutes | Too many sequential steps | Parallelize tests, cache dependencies |
| Flaky tests (pass/fail randomly) | Timing issues, shared state | Isolate tests, use test containers, add retries |
| "Works on my machine" | Environment differences | Use Docker in CI, pin dependency versions |
| Merge conflicts | Long-lived branches | Use trunk-based development, merge main daily |
| CI queue is slow | Too many builds, not enough runners | Add more runners, use matrix builds wisely |
| Coverage keeps dropping | New code without tests | Enforce coverage threshold in CI gate |

---

## Summary Table

| CI Concept | Description |
|-----------|-------------|
| **Core Idea** | Integrate code frequently, validate automatically |
| **Frequency** | Every commit / at least daily |
| **Pipeline** | Build → Test → Scan → Store artifact |
| **Branch Strategy** | Trunk-based development, short-lived branches |
| **Build Speed** | Target < 10 minutes |
| **Broken Build** | Fix immediately — highest priority |
| **Quality Gates** | Tests pass, coverage met, no vulnerabilities |

---

## Quick Revision Questions

1. **What is Continuous Integration, and what problem does it solve?**
   <details><summary>Answer</summary>CI is the practice of frequently merging code into a shared repository with automated builds and tests. It solves "integration hell" — the pain of merging long-lived branches that have diverged significantly, causing complex merge conflicts and late-stage bugs.</details>

2. **What is trunk-based development, and why is it recommended for CI?**
   <details><summary>Answer</summary>Trunk-based development keeps all developers working on a single main branch (trunk) with very short-lived feature branches (hours, not weeks). It's recommended because it minimizes merge conflicts, ensures continuous integration, and keeps the codebase always close to a deployable state.</details>

3. **What are the typical quality gates in a CI pipeline?**
   <details><summary>Answer</summary>1) All tests pass (unit + integration). 2) Code coverage meets threshold (e.g., ≥80%). 3) No critical security vulnerabilities (SAST/SCA). 4) Code style/linting rules pass. 5) Build compiles successfully. All gates must pass before merge is allowed.</details>

4. **Why should you keep the CI build under 10 minutes?**
   <details><summary>Answer</summary>Slow builds reduce developer productivity and discourage frequent commits. If builds take 30+ minutes, developers batch changes (defeating CI's purpose), lose context waiting for results, and may push broken code because they don't wait for feedback. Fast builds enable tight feedback loops.</details>

5. **What causes flaky tests, and how do you fix them?**
   <details><summary>Answer</summary>Flaky tests are caused by: timing/race conditions, shared state between tests, external dependency failures, or order-dependent tests. Fix by: isolating test state, using test containers for databases, mocking external services, avoiding sleep/timing-dependent assertions, and running tests in random order.</details>

6. **What should happen when the CI build breaks?**
   <details><summary>Answer</summary>Fixing the build should be the team's highest priority. The developer who broke it should fix it immediately. No new code should be merged until the build is green. If it can't be fixed quickly, the breaking change should be reverted. A red build blocks the entire team's ability to deliver.</details>

---

[← Previous: Fail Fast, Recover Fast](../02-DevOps-Principles/05-fail-fast-recover-fast.md) | [Next: Continuous Delivery →](02-continuous-delivery.md)
