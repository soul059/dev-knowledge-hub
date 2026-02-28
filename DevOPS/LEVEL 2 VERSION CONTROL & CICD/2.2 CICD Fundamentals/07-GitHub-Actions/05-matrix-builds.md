# Chapter 7.5: Matrix Builds

[← Previous: Secrets & Variables](04-secrets-variables.md) | [Back to README](../README.md) | [Next: Reusable Workflows →](06-reusable-workflows.md)

---

## Overview

**Matrix builds** run the same job across multiple configurations in parallel — different OS versions, language versions, or feature combinations. This ensures compatibility without duplicating workflow code.

---

## How Matrix Builds Work

```
  MATRIX STRATEGY
  ═══════════════

  strategy:
    matrix:
      node: [18, 20, 22]
      os: [ubuntu-latest, windows-latest]

  GENERATES 6 PARALLEL JOBS:
  ┌──────────────────┬──────────────────┐
  │ ubuntu + Node 18 │ windows + Node 18│
  ├──────────────────┼──────────────────┤
  │ ubuntu + Node 20 │ windows + Node 20│
  ├──────────────────┼──────────────────┤
  │ ubuntu + Node 22 │ windows + Node 22│
  └──────────────────┴──────────────────┘

  Total jobs = node(3) × os(2) = 6
```

---

## Basic Matrix

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

---

## Include & Exclude

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]

        # Remove specific combinations
        exclude:
          - os: windows-latest
            node-version: 18    # Don't test Node 18 on Windows

        # Add extra combinations with custom variables
        include:
          - os: ubuntu-latest
            node-version: 22
            experimental: true   # Extra variable for this combo
          - os: macos-latest     # Add a combo not in base matrix
            node-version: 20
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
        continue-on-error: ${{ matrix.experimental == true }}
```

---

## Fail-Fast & Max Parallel

```yaml
jobs:
  test:
    strategy:
      fail-fast: false           # Don't cancel others if one fails
      max-parallel: 3            # Run at most 3 jobs at a time
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

```
  FAIL-FAST BEHAVIOR
  ══════════════════

  fail-fast: true (DEFAULT)         fail-fast: false
  ───────────────────               ────────────────
  Job 1: ✓ Pass                     Job 1: ✓ Pass
  Job 2: ✕ FAIL → Cancel Job 3,4   Job 2: ✕ FAIL
  Job 3: ⊘ Cancelled               Job 3: ✓ Pass (continues)
  Job 4: ⊘ Cancelled               Job 4: ✓ Pass (continues)

  Result: Fast failure              Result: Full picture
  Use: Quick feedback               Use: See all failures
```

---

## Dynamic Matrix

```yaml
jobs:
  # Job 1: Generate matrix dynamically
  generate-matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v4
      - id: set-matrix
        run: |
          # Generate matrix from directory structure
          SERVICES=$(ls -d services/*/  | sed 's|services/||;s|/||' | jq -R . | jq -s .)
          echo "matrix={\"service\":$SERVICES}" >> $GITHUB_OUTPUT

  # Job 2: Use dynamic matrix
  test:
    needs: generate-matrix
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJson(needs.generate-matrix.outputs.matrix) }}
    steps:
      - uses: actions/checkout@v4
      - run: |
          cd services/${{ matrix.service }}
          npm ci && npm test
```

---

## Multi-Language Matrix

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - language: node
            version: '20'
            install: 'npm ci'
            test: 'npm test'
          - language: python
            version: '3.12'
            install: 'pip install -r requirements.txt'
            test: 'pytest'
          - language: go
            version: '1.22'
            install: 'go mod download'
            test: 'go test ./...'
    steps:
      - uses: actions/checkout@v4

      - if: matrix.language == 'node'
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}

      - if: matrix.language == 'python'
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.version }}

      - if: matrix.language == 'go'
        uses: actions/setup-go@v5
        with:
          go-version: ${{ matrix.version }}

      - run: ${{ matrix.install }}
      - run: ${{ matrix.test }}
```

---

## Monorepo Service Matrix

```yaml
# Test all services in a monorepo
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      services: ${{ steps.filter.outputs.changes }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            user-service: 'services/user-service/**'
            order-service: 'services/order-service/**'
            payment-service: 'services/payment-service/**'

  test:
    needs: changes
    if: needs.changes.outputs.services != '[]'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: ${{ fromJson(needs.changes.outputs.services) }}
    steps:
      - uses: actions/checkout@v4
      - run: |
          cd services/${{ matrix.service }}
          npm ci && npm test
```

---

## Real-World Scenario

> **Scenario**: An open-source library must ensure compatibility with Node.js 18, 20, and 22 on Linux, Windows, and macOS.
>
> **Solution**: Use a 3×3 matrix (9 jobs). Set `fail-fast: false` to see all failures. Exclude `macos-latest + Node 18` (EOL). Include an `experimental: true` flag on Node 22 with `continue-on-error`. Cache npm dependencies per OS+Node combination. Total CI time equals the slowest single job (all run in parallel).

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Too many matrix jobs | Large cross-product | Use `exclude:` or reduce dimensions |
| Dynamic matrix empty | JSON parsing error | Validate JSON with `echo` and `jq` |
| One failure cancels all | `fail-fast: true` (default) | Set `fail-fast: false` |
| Can't reference matrix vars | Wrong syntax | Use `${{ matrix.key }}` |
| Jobs hit rate limit | >256 jobs per workflow | Split into multiple workflows or reduce matrix |
| Cache misses per matrix | Key doesn't include matrix values | Include `${{ matrix.os }}-${{ matrix.node }}` in cache key |

---

## Summary Table

| Feature | Purpose |
|---|---|
| `matrix:` | Define configuration dimensions |
| `include:` | Add specific combinations or extra variables |
| `exclude:` | Remove specific combinations |
| `fail-fast:` | Cancel all jobs on first failure (default: true) |
| `max-parallel:` | Limit concurrent matrix jobs |
| `fromJson()` | Parse dynamic matrix from job output |
| `${{ matrix.key }}` | Access matrix variable in steps |

---

## Quick Revision Questions

1. **What is a matrix build?** *Running the same job across multiple configurations (e.g., OS × language version) in parallel, generating one job per combination.*
2. **How do you calculate total matrix jobs?** *Multiply the count of values in each dimension: `os(3) × node(3) = 9 jobs`.*
3. **What does `fail-fast: false` do?** *Allows all matrix jobs to complete even if one fails, giving a complete picture of failures across configurations.*
4. **How do you add a variable only to specific matrix combinations?** *Use `include:` to add extra key-value pairs to specific combinations.*
5. **How do you create a dynamic matrix?** *Generate JSON in one job's output, then use `fromJson()` in the matrix strategy of a dependent job.*
6. **What is the maximum number of jobs in a matrix?** *256 jobs per workflow run — plan matrix dimensions accordingly.*

---

[← Previous: Secrets & Variables](04-secrets-variables.md) | [Back to README](../README.md) | [Next: Reusable Workflows →](06-reusable-workflows.md)
