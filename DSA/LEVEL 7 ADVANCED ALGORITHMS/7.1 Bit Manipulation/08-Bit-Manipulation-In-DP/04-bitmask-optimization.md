# Chapter 8.4: Bitmask Optimization Techniques

> **Unit 8 — Bit Manipulation in DP**
> Speed up bitmask DP through subset enumeration, profile optimization, and broken profile techniques.

---

## Overview

When n pushes the limits (n = 20–25), raw bitmask DP may be too slow. These optimization techniques can provide constant factor or even asymptotic improvements.

---

## Technique 1: Subset Sum Optimization (SOS DP)

```
  Problem: For each mask, compute:
    F[mask] = Σ f[sub]  for all sub ⊆ mask

  Naive: O(3^n) using subset enumeration
  SOS:   O(n × 2^n) — much faster!
```

### The SOS DP Algorithm

```
FUNCTION sosDP(f[], n):
    // F[mask] = sum of f[sub] for all submasks sub of mask
    F = copy of f
    FOR k = 0 TO n-1:
        FOR mask = 0 TO (1<<n)-1:
            IF mask & (1<<k):
                F[mask] += F[mask ^ (1<<k)]
    RETURN F
```

### How It Works

```
  Process one bit dimension at a time:
  
  After processing bit 0:
    F[mask] includes contributions from masks differing in bit 0
  After processing bit 1:
    F[mask] includes contributions from bits 0 and 1
  ...
  After processing bit n-1:
    F[mask] = Σ f[sub] for all sub ⊆ mask  ✓

  Like multi-dimensional prefix sums, but in binary!
```

### Trace: n=3, f = [1,2,3,4,5,6,7,8]

```
  Initial: F = [1, 2, 3, 4, 5, 6, 7, 8]
                000 001 010 011 100 101 110 111

  bit 0 (k=0): for masks with bit 0 set:
    F[001] += F[000] → 2+1 = 3
    F[011] += F[010] → 4+3 = 7
    F[101] += F[100] → 6+5 = 11
    F[111] += F[110] → 8+7 = 15
    F = [1, 3, 3, 7, 5, 11, 7, 15]

  bit 1 (k=1): for masks with bit 1 set:
    F[010] += F[000] → 3+1 = 4
    F[011] += F[001] → 7+3 = 10
    F[110] += F[100] → 7+5 = 12
    F[111] += F[101] → 15+11 = 26
    F = [1, 3, 4, 10, 5, 11, 12, 26]

  bit 2 (k=2): for masks with bit 2 set:
    F[100] += F[000] → 5+1 = 6
    F[101] += F[001] → 11+3 = 14
    F[110] += F[010] → 12+4 = 16
    F[111] += F[011] → 26+10 = 36
    F = [1, 3, 4, 10, 6, 14, 16, 36]

  Verify F[111] = f[000]+f[001]+f[010]+f[011]+f[100]+f[101]+f[110]+f[111]
                = 1+2+3+4+5+6+7+8 = 36  ✓
```

---

## Technique 2: Broken Profile DP

```
  For grid problems (like tiling), instead of masking all cells,
  only mask the "boundary" between processed and unprocessed regions.

  ┌───────────────────────────────────────────────────────┐
  │  Full grid bitmask: 2^(rows×cols) states  ← too many │
  │  Broken profile:    2^(one row/col) states ← feasible│
  └───────────────────────────────────────────────────────┘

  Process column by column. The mask represents the boundary
  profile of the current column's state.
```

### Example: Domino Tiling

```
  Fill an m×n grid with dominoes.
  
  State: dp[col][mask] = ways to fill columns 0..col
         with boundary mask between col and col+1
  
  mask bit i = 1: cell (i, col) protrudes into column col+1
  mask bit i = 0: cell (i, col) is complete
  
  Transition: try all valid placements for column col+1
  
  Time: O(n × 2^m × 2^m) = O(n × 4^m)
  With optimization: O(n × 2^m × valid_transitions)
```

---

## Technique 3: Meet in the Middle for DP

```
  Split the elements into two halves.
  Compute DP for each half independently.
  Combine results.

  Total: O(n × 2^(n/2)) instead of O(n × 2^n)
  
  Already seen in subset sum (Chapter 6.3).
  Generalizes to other bitmask DP problems.
```

---

## Technique 4: Bit-Parallel Tricks

```
  // Instead of looping over set bits:
  FOR i = 0 TO n-1:           // O(n)
      IF mask & (1<<i): ...

  // Iterate only set bits:            
  temp = mask                  // O(popcount(mask))
  WHILE temp:
      i = ctz(temp)
      ...
      temp &= temp - 1
```

---

## Complexity Comparison

| Technique | Before | After |
|-----------|--------|-------|
| SOS DP | O(3^n) | O(n × 2^n) |
| Broken profile | O(2^(m×n)) | O(n × 4^m) |
| Meet in middle | O(2^n) | O(2^(n/2)) |
| Bit iteration | O(n) per state | O(k) per state |

---

## Summary Table

| Technique | When to Use |
|-----------|-------------|
| SOS DP | Sum/count over all submasks efficiently |
| Broken profile | Grid problems where full mask is too large |
| Meet in middle | n ≤ 40, can split problem into independent halves |
| Bit-parallel iteration | Reduce constant factor in inner loops |

---

## Quick Revision Questions

1. **What does SOS DP compute?**
   <details><summary>Answer</summary>For each mask, the sum of f[sub] over all submasks sub of mask. It's like a multi-dimensional prefix sum in the bitmask domain.</details>

2. **Why is SOS O(n × 2^n) instead of O(3^n)?**
   <details><summary>Answer</summary>SOS processes one bit dimension at a time, doing O(2^n) work per bit × n bits = O(n × 2^n). This avoids the explicit subset enumeration which costs O(3^n).</details>

3. **What is a "broken profile" in grid DP?**
   <details><summary>Answer</summary>The boundary between the processed and unprocessed regions of the grid. Instead of tracking the state of all cells, we only track this boundary, reducing the state space dramatically.</details>

4. **Can SOS DP compute superset sums too?**
   <details><summary>Answer</summary>Yes! Just flip the condition: process bits where mask does NOT have bit k set. F[mask] then equals the sum of f[sup] for all supersets sup of mask.</details>

5. **When is meet-in-the-middle NOT applicable?**
   <details><summary>Answer</summary>When the two halves are not independent — i.e., decisions in one half affect the other. The problem must decompose cleanly.</details>

---

[← Previous: Counting Paths with States](03-counting-paths-with-states.md) | [Next: Memory Savings →](05-memory-savings.md)
