# Chapter 8.5: Memory Savings in Bitmask DP

> **Unit 8 — Bit Manipulation in DP**
> Reduce memory footprint for bitmask DP from gigabytes to megabytes using rolling arrays, state compression, and careful encoding.

---

## Overview

Bitmask DP tables can be enormous — `dp[1 << 20][20]` already needs ~80 MB of integers. Memory can become the bottleneck before time does. This chapter explores strategies to reduce memory usage.

---

## Problem: Memory Explosion

```
  n = 20:  2^20 × 20 × 4 bytes = 80 MB
  n = 22:  2^22 × 22 × 4 bytes ≈ 370 MB
  n = 24:  2^24 × 24 × 4 bytes ≈ 1.5 GB  ← often exceeds limits!

  Even n = 20 can be tight with 256 MB memory limits.
```

---

## Technique 1: Eliminating Redundant Dimensions

```
  If the second dimension is determined by the mask,
  remove it entirely!

  Example: Assignment Problem
  ┌───────────────────────────────┐
  │ Worker number = popcount(mask)│
  │ dp[mask][worker] → dp[mask]  │
  │ Memory: 2^n × n → 2^n       │
  │ Savings: n× less memory      │
  └───────────────────────────────┘
```

### Pseudocode

```
  // BEFORE: dp[mask][worker]
  dp[mask][popcount(mask)] = value
  // worker is always popcount(mask), so it's redundant!

  // AFTER: dp[mask]
  dp[mask] = value
  // worker = popcount(mask) — computed on the fly
```

---

## Technique 2: Rolling Array (Layer Optimization)

```
  When transitions go from masks with k bits set
  to masks with k+1 bits set:

  ┌────────────────────────────────────────────────┐
  │  Only need two layers: current and previous    │
  │  dp_prev[] and dp_curr[] instead of dp[][]     │
  │  Memory: 2 × 2^n instead of n × 2^n           │
  └────────────────────────────────────────────────┘

  BUT: bitmask DP often has irregular transitions
       (not always layer-by-layer), so this is not
       always applicable.
```

### When It Works

```
  ✓ Transitions go from popcount=k to popcount=k+1
  ✓ Each layer only depends on the previous layer
  ✓ No need to backtrack through intermediate layers

  Example: Assignment problem processes workers 0..n-1
           Layer k has masks with exactly k bits set
           Layer k+1 depends only on layer k  ✓
```

---

## Technique 3: Hash Map for Sparse States

```
  If only a fraction of 2^n masks are reachable:
  
  ┌──────────────────────────────────────────────┐
  │  Use a hash map instead of an array!         │
  │  Map<int, value> instead of value[1 << n]    │
  │  Memory: O(reachable states) not O(2^n)      │
  └──────────────────────────────────────────────┘

  Example: If constraints make only ~100K out of 2^20
           masks reachable, hash map uses ~100K entries
           instead of ~1M array slots.
```

### Pseudocode

```
  // BEFORE: Fixed array
  int dp[1 << n];

  // AFTER: Hash map
  HashMap<int, int> dp;

  // Access remains the same
  dp[mask] = value;

  // Trade-off: ~2-3× slower per access, but much less memory
```

---

## Technique 4: Smaller Data Types

```
  ┌────────────────────────────────┐
  │  int (4 bytes) → short (2)    │
  │  int (4 bytes) → byte  (1)    │
  │  Use bool arrays when possible│
  └────────────────────────────────┘

  For counting problems: values may fit in short (up to 32767)
  For reachability:      bool array (1 byte each)
  For bitset approach:   1 bit per state with std::bitset
```

---

## Technique 5: In-Place Computation

```
  Process masks in the right order so that
  dp[mask] is computed before it's needed and
  never needed again after use.

  // If lower masks feed higher masks:
  FOR mask = 0 TO (1<<n)-1:
      FOR each bit i NOT in mask:
          new_mask = mask | (1<<i)
          dp[new_mask] = combine(dp[new_mask], dp[mask])

  // Single array, no extra space!
  // Works because mask < new_mask always
```

---

## Technique 6: State Encoding Compression

```
  Some problems have additional constraints that
  reduce valid states. Encode only valid masks.

  Example: n=20 items, but no two adjacent items together
  Valid masks: those with no consecutive 1-bits
  Count: Fibonacci(n+2) ≈ 17,711 for n=20
  vs. 2^20 = 1,048,576 total masks

  ┌──────────────────────────────────────────┐
  │  Map valid masks to indices 0, 1, 2, ...│
  │  Use compact array indexed by position   │
  │  60× less memory in this example!        │
  └──────────────────────────────────────────┘
```

### Implementation

```
  // Precompute valid masks
  valid = []
  FOR mask = 0 TO (1<<n)-1:
      IF mask & (mask << 1) == 0:   // no consecutive bits
          valid.append(mask)
  
  // index[mask] = position in valid[]
  // dp[index] instead of dp[mask]
```

---

## Summary of Memory Techniques

| Technique | Memory Reduction | When to Use |
|-----------|-----------------|-------------|
| Remove redundant dim | n× | Dimension derivable from mask |
| Rolling array | n× or more | Layer-by-layer transitions |
| Hash map | up to 2^n / reachable | Sparse reachable states |
| Smaller types | 2–4× | Values fit in smaller types |
| In-place | 2× | Monotonic mask ordering |
| State encoding | varies (up to 60×+) | Structural constraints on masks |

---

## Practical Decision Guide

```
  Memory over limit?
  ├── Is a dimension redundant?
  │   └── YES → Remove it (Technique 1)       ← try first
  ├── Do transitions go layer-by-layer?
  │   └── YES → Rolling array (Technique 2)
  ├── Are most states unreachable?
  │   └── YES → Hash map (Technique 3)
  ├── Do values fit in short/byte?
  │   └── YES → Use smaller types (Technique 4)
  ├── Can you order computation?
  │   └── YES → In-place (Technique 5)
  └── Are there structural constraints?
      └── YES → State encoding (Technique 6)
```

---

## Quick Revision Questions

1. **Why is dimension elimination the first technique to try?**
   <details><summary>Answer</summary>It provides a clean n× reduction with zero overhead. If a dimension is derivable from the mask (e.g., worker = popcount(mask)), removing it simplifies both memory and code.</details>

2. **When does a rolling array work for bitmask DP?**
   <details><summary>Answer</summary>When transitions go strictly from layer k (masks with k set bits) to layer k+1 (masks with k+1 set bits), and the final answer doesn't require backtracking through all layers.</details>

3. **What is the trade-off of using hash maps?**
   <details><summary>Answer</summary>Hash maps save memory when states are sparse, but incur 2-3× slower access times due to hashing overhead. They also have higher per-entry memory overhead (key + value + pointers).</details>

4. **How does state encoding compression work?**
   <details><summary>Answer</summary>When structural constraints (like "no two adjacent bits set") reduce valid masks, we enumerate only valid masks, assign them compact indices 0,1,2,..., and use these indices to address the DP array instead of the raw mask values.</details>

5. **Can multiple techniques be combined?**
   <details><summary>Answer</summary>Yes! For example, combine dimension elimination (Technique 1) with state encoding (Technique 6) and smaller data types (Technique 4) for maximum savings. These techniques are largely independent and composable.</details>

---

[← Previous: Bitmask Optimization](04-bitmask-optimization.md) | [Next: IP Address Manipulation →](../09-Practical-Applications/01-ip-address-manipulation.md)
