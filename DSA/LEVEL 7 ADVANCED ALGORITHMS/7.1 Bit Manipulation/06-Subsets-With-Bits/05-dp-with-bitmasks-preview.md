# Chapter 6.5: DP with Bitmasks — Preview

> **Unit 6 — Subsets with Bits**
> A primer on using bitmasks as DP states — the bridge to Unit 8's full treatment.

---

## Overview

In bitmask DP, each **state** is a bitmask representing which elements have been "used" or "visited." This turns exponential subset problems into manageable DP tables of size 2^n × (extra dimensions).

---

## The Core Idea

```
  ┌──────────────────────────────────────────────────────┐
  │  Traditional DP state:  dp[i][j]                     │
  │  Bitmask DP state:      dp[mask]  or  dp[mask][j]    │
  │                                                      │
  │  mask encodes WHICH elements have been selected.     │
  │  j often encodes the "last" or "current" element.    │
  └──────────────────────────────────────────────────────┘
```

### Why it works

```
  Problem: "Select elements in some order, optimizing a value."
  
  Key insight: The optimal continuation depends only on
  WHICH elements are left, not the ORDER they were picked.
  
  → A bitmask captures "which" in a single integer!
```

---

## Example: Assignment Problem

```
  n workers, n jobs. cost[i][j] = cost to assign worker i to job j.
  Minimize total cost where each worker gets exactly one job.

  State: dp[mask] = minimum cost to assign workers 0..(popcount(mask)-1)
         to the jobs indicated by set bits in mask.

  Transition:
    Let k = popcount(mask)  // which worker we're assigning next
    dp[mask] = MIN over all set bits j in mask:
                 dp[mask ^ (1<<j)] + cost[k-1][j]
```

### Visual

```
  mask = 1011 (jobs 0,1,3 assigned)
  popcount = 3, so workers 0,1,2 are assigned
  
  This came from assigning worker 2 to some job j in {0,1,3}:
  
    If j=0: dp[1011] could be dp[1010] + cost[2][0]
    If j=1: dp[1011] could be dp[1001] + cost[2][1]
    If j=3: dp[1011] could be dp[0011] + cost[2][3]
    
    Take the minimum.
```

---

## State Space

```
  dp[mask] with n elements:
  
  Masks:     2^n states
  Per state: O(n) transitions (try each set bit)
  Total:     O(n × 2^n)
  
  n = 20:  20 × 2^20 ≈ 20M        ← feasible
  n = 25:  25 × 2^25 ≈ 800M       ← borderline
  n = 30:  30 × 2^30 ≈ 30B        ← too much
```

---

## Common Bitmask DP Patterns

### Pattern 1: dp[mask] — state is the subset

```
  Used for: subset sum variants, partition problems
  dp[mask] = optimal value using elements in mask
```

### Pattern 2: dp[mask][i] — subset + last element

```
  Used for: TSP, Hamiltonian path
  dp[mask][i] = optimal value with elements in mask, ending at i
  Extra dimension: O(n) → Total: O(n² × 2^n)
```

### Pattern 3: dp[mask][j] — subset + extra parameter

```
  Used for: scheduling with states, matching problems
  dp[mask][j] = value with mask used plus some position/state j
```

---

## Template Code

```
FUNCTION bitmaskDP(n, cost[][]):
    dp = array of size (1 << n), initialized to INFINITY
    dp[0] = 0    // empty set: no cost
    
    FOR mask = 1 TO (1 << n) - 1:
        k = popcount(mask) - 1    // which item we're assigning (0-indexed)
        FOR j = 0 TO n - 1:
            IF mask & (1 << j):   // j is in the mask
                prev_mask = mask ^ (1 << j)    // remove j
                dp[mask] = MIN(dp[mask], dp[prev_mask] + cost[k][j])
    
    RETURN dp[(1 << n) - 1]    // all items assigned
```

---

## Complexity

| Aspect | Value |
|--------|-------|
| States | 2^n (or 2^n × n) |
| Time per state | O(n) |
| Total time | O(n × 2^n) or O(n² × 2^n) |
| Space | O(2^n) or O(n × 2^n) |
| Practical limit | n ≤ 20–23 |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Bitmask as state | Each integer 0..2^n−1 represents a subset |
| Optimal substructure | Future depends on WHICH elements remain, not order |
| Common dimensions | dp[mask] or dp[mask][last] |
| Transitions | Add/remove one element: mask ^ (1<<j) |
| Complexity | O(n × 2^n) typical |

> Full treatment in Unit 8: TSP, Assignment Problem, and optimizations.

---

## Quick Revision Questions

1. **Why can't we use bitmask DP for n = 30?**
   <details><summary>Answer</summary>2^30 ≈ 10^9 states. With O(n) work per state, total ≈ 3×10^10 operations. Too slow and too much memory (~4 GB for integer array).</details>

2. **What does dp[mask ^ (1<<j)] represent?**
   <details><summary>Answer</summary>The state where element j is removed from mask (if it was present) or added (if absent). In DP transitions, we usually remove j to look at the "previous" state.</details>

3. **How do you know which "step" you're on given just the mask?**
   <details><summary>Answer</summary>popcount(mask) tells you how many elements have been selected. Step = popcount(mask).</details>

4. **What's the base case for most bitmask DPs?**
   <details><summary>Answer</summary>dp[0] = 0 (empty set, nothing selected, zero cost/value). Sometimes dp with single-element masks is the base.</details>

5. **Can bitmask DP handle constraints between elements?**
   <details><summary>Answer</summary>Yes! Check constraints during transitions. E.g., "element j can't be next if element i was last" → check in dp[mask][last] transitions.</details>

---

[← Previous: Bitmasking for Subsets](04-bitmasking-for-subsets.md) | [Next: Iterating Subsets of a Mask →](06-iterating-subsets-of-mask.md)
