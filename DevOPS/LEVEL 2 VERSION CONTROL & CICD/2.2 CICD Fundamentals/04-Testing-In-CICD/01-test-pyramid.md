# Chapter 4.1: Test Pyramid

[← Previous: Build Caching Strategies](../03-Build-Automation/06-build-caching-strategies.md) | [Back to README](../README.md) | [Next: Unit Testing →](02-unit-testing.md)

---

## Overview

The **Test Pyramid** is a model for structuring your automated test suite. It guides how many tests of each type you should have, balancing speed, cost, and confidence.

---

## The Test Pyramid

```
                         /\
                        /  \
                       / E2E\         Few, slow, expensive
                      / Tests\        Test full user journeys
                     /────────\       (10-20% of tests)
                    /          \
                   / Integration\     Moderate number
                  /    Tests     \    Test component interactions
                 /────────────────\   (20-30% of tests)
                /                  \
               /    Unit Tests      \ Many, fast, cheap
              /                      \ Test individual functions
             /________________________\ (60-70% of tests)

  Speed:     ◄── SLOW ──────────────── FAST ──►
  Cost:      ◄── EXPENSIVE ────────── CHEAP ──►
  Quantity:  ◄── FEW ───────────────── MANY ──►
  Confidence:◄── HIGH (real) ─── HIGH (focused) ──►
```

---

## Test Types Comparison

| Type | Scope | Speed | Count | Dependencies | Example |
|---|---|---|---|---|---|
| **Unit** | Single function/class | ms | 1000s | None (mocked) | "Does `add(2,3)` return `5`?" |
| **Integration** | Multiple components | seconds | 100s | DB, API, cache | "Does user signup write to DB?" |
| **E2E** | Full application | minutes | 10s | Everything | "Can a user complete checkout?" |

---

## Anti-Pattern: Ice Cream Cone

```
          THE ICE CREAM CONE (Anti-Pattern)
          ══════════════════════════════════

              ┌──────────────────┐
              │   Manual Tests   │    Many manual tests
              │   (lots!)        │    Slow, unreliable
              ├──────────────────┤
              │  E2E Tests       │    Too many E2E
              │  (too many)      │    Slow, flaky
              ├────────────┤
              │ Integration│         Some integration
              ├────────┤
              │  Unit  │              Few unit tests
              └────────┘

          EVERYTHING IS BACKWARDS!
          Slow, expensive, fragile test suite
```

---

## Applying the Pyramid in CI/CD

```yaml
# GitHub Actions — Test pyramid in pipeline
jobs:
  # Base of pyramid: Unit tests (fast, run first)
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --coverage
    # Duration: ~2 min, runs on every commit

  # Middle: Integration tests
  integration-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    services:
      postgres: { image: 'postgres:16', env: { POSTGRES_PASSWORD: test } }
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:integration
    # Duration: ~5 min

  # Top: E2E tests (slow, run selectively)
  e2e-tests:
    needs: integration-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
    # Duration: ~10 min
```

---

## Summary Table

| Level | Tests | Duration | Runs When | Catches |
|---|---|---|---|---|
| **Unit** | 60-70% | Seconds | Every commit | Logic bugs |
| **Integration** | 20-30% | Minutes | Every PR | Interface bugs |
| **E2E** | 10-20% | 10+ min | Before deploy | User flow bugs |

---

## Quick Revision Questions

1. **What is the test pyramid?** *A model recommending many fast unit tests at the base, fewer integration tests in the middle, and few slow E2E tests at the top.*
2. **What is the "ice cream cone" anti-pattern?** *Having too many manual/E2E tests and too few unit tests — making the test suite slow and fragile.*
3. **What percentage of tests should be unit tests?** *60-70% of the total test suite.*
4. **Why are E2E tests at the top of the pyramid?** *They're the slowest, most expensive, and most fragile — you should have the fewest of them.*
5. **How does the test pyramid map to CI/CD pipeline stages?** *Unit tests run first (fast feedback), then integration, then E2E (slower, more comprehensive).*
6. **What distinguishes a unit test from an integration test?** *Unit tests verify a single function in isolation (mocked deps); integration tests verify multiple components working together with real dependencies.*

---

[← Previous: Build Caching Strategies](../03-Build-Automation/06-build-caching-strategies.md) | [Back to README](../README.md) | [Next: Unit Testing →](02-unit-testing.md)
