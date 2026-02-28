# Chapter 3: Memoization in Backtracking

## Overview
**Memoization** caches results of subproblems so they're computed only once. When backtracking encounters overlapping subproblems — the same state reached via different paths — memoization transforms exponential recursion into polynomial dynamic programming. This chapter clarifies when memoization applies to backtracking and when it doesn't.

---

## When Does Memoization Apply?

```
Memoization works when:
  ✓ Same STATE can be reached via different PATHS
  ✓ The RESULT depends only on the state, not the path
  ✓ State is small enough to use as a cache key

Memoization does NOT work when:
  ✗ You need ALL paths (not just count/existence)
  ✗ Result depends on HOW you got to the state
  ✗ State space is too large to cache
  ✗ Each state is visited only once (no overlap)

┌──────────────────────┬─────────────┬──────────────┐
│ Problem              │ Overlapping?│ Memo helpful? │
├──────────────────────┼─────────────┼──────────────┤
│ Fibonacci            │ Yes         │ Yes (O(n))   │
│ Grid paths (count)   │ Yes         │ Yes (O(mn))  │
│ Subset sum (exists?) │ Yes         │ Yes          │
│ N-Queens (list all)  │ No          │ No           │
│ Permutations         │ No          │ No           │
│ Sudoku               │ No          │ No           │
│ Word Break (exists)  │ Yes         │ Yes          │
│ Word Break II (list) │ Partial     │ Partial      │
└──────────────────────┴─────────────┴──────────────┘
```

---

## Pattern: Backtracking → Memoization

```
Step 1: Write pure backtracking
────────────────────────────────
FUNCTION solve(state):
    IF isBase(state): RETURN baseValue
    result = initial
    FOR each choice:
        result = combine(result, solve(nextState))
    RETURN result

Step 2: Add memoization
────────────────────────────────
memo = {}

FUNCTION solve(state):
    IF state IN memo: RETURN memo[state]    ← Cache hit!
    IF isBase(state): RETURN baseValue
    result = initial
    FOR each choice:
        result = combine(result, solve(nextState))
    memo[state] = result                     ← Cache store!
    RETURN result

That's it. Two lines added. Exponential → polynomial.
```

---

## Example 1: Subset Sum (Can Sum?)

```
"Can you pick elements from nums to sum to target?"

WITHOUT memo — O(2^n):
  FUNCTION canSum(nums, idx, remaining):
      IF remaining == 0: RETURN TRUE
      IF remaining < 0 OR idx == n: RETURN FALSE
      // Include or exclude nums[idx]
      RETURN canSum(nums, idx+1, remaining - nums[idx])
          OR canSum(nums, idx+1, remaining)

WITH memo — O(n × target):
  memo = {}
  FUNCTION canSum(nums, idx, remaining):
      IF (idx, remaining) IN memo: 
          RETURN memo[(idx, remaining)]
      IF remaining == 0: RETURN TRUE
      IF remaining < 0 OR idx == n: RETURN FALSE
      
      result = canSum(nums, idx+1, remaining - nums[idx])
            OR canSum(nums, idx+1, remaining)
      
      memo[(idx, remaining)] = result
      RETURN result

State: (idx, remaining) — only 2 parameters
Cache: at most n × target entries
```

---

## Example 2: Word Break

```
"Can string s be segmented into dictionary words?"

WITHOUT memo — O(2^n):
  FUNCTION wordBreak(s, start, wordSet):
      IF start == length(s): RETURN TRUE
      FOR end = start+1 TO length(s):
          IF s[start..end-1] IN wordSet:
              IF wordBreak(s, end, wordSet):
                  RETURN TRUE
      RETURN FALSE

WITH memo — O(n²):
  memo = {}
  FUNCTION wordBreak(s, start, wordSet):
      IF start IN memo: RETURN memo[start]
      IF start == length(s): RETURN TRUE
      result = FALSE
      FOR end = start+1 TO length(s):
          IF s[start..end-1] IN wordSet:
              IF wordBreak(s, end, wordSet):
                  result = TRUE
                  BREAK
      memo[start] = result
      RETURN result

State: just the start index!
  → Same suffix reached from different dict word splits
  → n possible start indices → O(n) cache entries
  → O(n) work per entry → O(n²) total
```

---

## Example 3: Grid Paths with Obstacles (Count)

```
WITHOUT memo — O(2^(m+n)):
  FUNCTION paths(grid, r, c):
      IF r == m-1 AND c == n-1: RETURN 1
      IF outOfBounds OR grid[r][c] == 1: RETURN 0
      RETURN paths(grid, r+1, c) + paths(grid, r, c+1)

WITH memo — O(m × n):
  memo = {}
  FUNCTION paths(grid, r, c):
      IF (r,c) IN memo: RETURN memo[(r,c)]
      IF r == m-1 AND c == n-1: RETURN 1
      IF outOfBounds OR grid[r][c] == 1: RETURN 0
      memo[(r,c)] = paths(grid, r+1, c) + paths(grid, r, c+1)
      RETURN memo[(r,c)]

Overlap visualization:
  Cell (2,3) reached from (1,3)↓ and (2,2)→
  Without memo: solve (2,3) twice
  With memo: solve once, retrieve once
```

---

## Why N-Queens Can't Be Memoized

```
In N-Queens, the state includes:
  - Current row
  - SET of columns used
  - SET of diagonals used
  - SET of anti-diagonals used

State key: (row, frozenset(cols), frozenset(diags), frozenset(antidiags))

Problem 1: State space is HUGE
  Columns: 2^n possible subsets
  Diags: 2^(2n-1) subsets
  → Cache would be larger than solutions!

Problem 2: States rarely repeat
  Each path uses different column combinations
  → Almost no cache hits
  → Memoization overhead without benefit

Problem 3: We want ALL solutions
  → Can't short-circuit with cached counts
  → Must enumerate each one
```

---

## Choosing the Cache Key

```
GOOD keys (small, hashable):
  ✓ (index, remaining_sum)     — subset sum
  ✓ (start_index)              — word break
  ✓ (row, col)                 — grid paths
  ✓ (mask)                     — bitmask DP (items as bits)
  ✓ (i, j, k)                 — 3D DP states

BAD keys (too large, can't hash):
  ✗ Entire board configuration  — too large
  ✗ List of choices made        — path-dependent
  ✗ Current partial solution    — varies per path
  ✗ String being built          — exponential variants

Rule of thumb:
  If the key has O(polynomial) possible values → memo works
  If the key has O(exponential) possible values → memo doesn't help
```

---

## Bitmask Memoization

```
For problems with n ≤ 20 items, represent "used" set as bitmask.

Example — Traveling Salesman (n cities):
  State: (current_city, visited_mask)
  visited_mask: n-bit integer where bit i = city i visited
  
  memo[current][mask] = min cost to visit remaining cities

  Total states: n × 2^n
  Much better than n! (brute force)

  n=15: 15 × 2^15 = 491,520 states (fast)
  vs    15! = 1.3 trillion (impossible)

FUNCTION tsp(city, mask, memo):
    IF mask == (1 << n) - 1:             ← All visited
        RETURN dist[city][0]
    IF memo[city][mask] != -1:
        RETURN memo[city][mask]
    result = INFINITY
    FOR next = 0 TO n-1:
        IF mask & (1 << next) == 0:      ← Not visited
            result = min(result, 
                dist[city][next] + tsp(next, mask | (1<<next), memo))
    memo[city][mask] = result
    RETURN result
```

---

## Memoization vs Tabulation

```
┌───────────────────┬──────────────────┬──────────────────┐
│                   │ Memoization      │ Tabulation       │
│                   │ (Top-Down)       │ (Bottom-Up)      │
├───────────────────┼──────────────────┼──────────────────┤
│ Direction         │ Start → recurse  │ Base → build up  │
│ Code structure    │ Recursive + cache│ Iterative + table│
│ Subproblems solved│ Only needed ones │ All of them      │
│ Stack overflow    │ Possible (deep)  │ No               │
│ Ease of writing   │ Add 2 lines      │ Think about order│
│ When to prefer    │ Sparse subproblem│ Dense subproblems│
│                   │ space            │                  │
└───────────────────┴──────────────────┴──────────────────┘

Start with memoization (easier).
Convert to tabulation if:
  - Recursion depth too large
  - Need space optimization (rolling array)
  - All subproblems will be visited anyway
```

---

## Complexity Comparison

```
┌──────────────────────┬────────────────┬───────────────┐
│ Problem              │ Without Memo   │ With Memo     │
├──────────────────────┼────────────────┼───────────────┤
│ Fibonacci            │ O(2^n)         │ O(n)          │
│ Grid Paths (m×n)     │ O(2^(m+n))     │ O(m×n)        │
│ Subset Sum           │ O(2^n)         │ O(n×target)   │
│ Word Break           │ O(2^n)         │ O(n²)         │
│ Coin Change          │ O(coins^amount)│ O(n×amount)   │
│ Edit Distance        │ O(3^(m+n))     │ O(m×n)        │
│ TSP (bitmask)        │ O(n!)          │ O(n²×2^n)     │
│ Longest Common Subseq│ O(2^(m+n))     │ O(m×n)        │
└──────────────────────┴────────────────┴───────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| When to use | Overlapping subproblems + state-dependent results |
| When NOT to use | Need all paths, state too large, no overlap |
| How to add | 2 lines: check cache before work, store after |
| Key design | Cache key = smallest set of parameters defining state |
| Bitmask DP | n ≤ 20 items, represent set as integer |
| Effect | Exponential → polynomial (typically) |
| Trade-off | Extra space for time savings |

---

## Quick Revision Questions

1. **What two conditions** must hold for memoization to help?
2. **Why can't N-Queens** be effectively memoized?
3. **What makes a good** cache key?
4. **How does bitmask DP** reduce the state space?
5. **When should you prefer** tabulation over memoization?
6. **What's the typical complexity** reduction from memoization?

---

[← Previous: Early Termination](02-early-termination.md) | [Next: Bit Manipulation Optimization →](04-bit-manipulation-optimization.md)

[← Back to README](../README.md)
