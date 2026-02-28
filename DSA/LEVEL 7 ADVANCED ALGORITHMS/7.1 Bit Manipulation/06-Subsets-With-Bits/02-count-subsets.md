# Chapter 6.2: Count Subsets with a Given Property

> **Unit 6 — Subsets with Bits**
> Enumerate all 2^n subsets and count those satisfying a condition — the brute-force bitmask approach.

---

## Overview

Many problems ask: "How many subsets of a set satisfy property P?" When n is small (≤ 20), we can check every subset using bitmasks.

---

## General Template

```
FUNCTION countSubsets(set[], n, property):
    count = 0
    FOR mask = 0 TO (1 << n) - 1:
        IF property(set, mask) == TRUE:
            count += 1
    RETURN count
```

---

## Example 1: Count Subsets with Sum = Target

```
FUNCTION countSubsetsWithSum(arr[], n, target):
    count = 0
    FOR mask = 0 TO (1 << n) - 1:
        sum = 0
        FOR i = 0 TO n - 1:
            IF mask & (1 << i) != 0:
                sum += arr[i]
        IF sum == target:
            count += 1
    RETURN count
```

### Trace: arr = [1, 2, 3], target = 3

```
  mask=0 (000): sum=0          ✗
  mask=1 (001): sum=1          ✗
  mask=2 (010): sum=2          ✗
  mask=3 (011): sum=1+2=3      ✓  count=1
  mask=4 (100): sum=3          ✓  count=2
  mask=5 (101): sum=1+3=4      ✗
  mask=6 (110): sum=2+3=5      ✗
  mask=7 (111): sum=1+2+3=6    ✗

  Answer: 2 subsets ({1,2} and {3})
```

---

## Example 2: Count Subsets of Size Exactly K

```
FUNCTION countSubsetsOfSizeK(n, k):
    count = 0
    FOR mask = 0 TO (1 << n) - 1:
        IF popcount(mask) == k:
            count += 1
    RETURN count
```

> This is just C(n, k) = n! / (k! × (n-k)!), but bitmask enumeration lets you also **process** each such subset.

---

## Example 3: Count Subsets Where Max - Min ≤ D

```
FUNCTION countBoundedSubsets(arr[], n, D):
    count = 0
    FOR mask = 1 TO (1 << n) - 1:    // skip empty set
        min_val = ∞
        max_val = -∞
        FOR i = 0 TO n - 1:
            IF mask & (1 << i) != 0:
                min_val = MIN(min_val, arr[i])
                max_val = MAX(max_val, arr[i])
        IF max_val - min_val <= D:
            count += 1
    RETURN count
```

---

## Optimization: Gosper's Hack

When you only want subsets of a **specific size k**, iterate only those using Gosper's hack:

```
  ┌──────────────────────────────────────────────────────┐
  │  Gosper's Hack: Next mask with same number of bits   │
  │                                                      │
  │  c = mask & (-mask)          // lowest set bit       │
  │  r = mask + c                // carry into next      │
  │  next = (((r ^ mask) >> 2) / c) | r                 │
  └──────────────────────────────────────────────────────┘
```

```
FUNCTION iterateSubsetsOfSizeK(n, k):
    mask = (1 << k) - 1         // smallest mask with k bits
    limit = 1 << n
    WHILE mask < limit:
        process(mask)
        c = mask & (-mask)
        r = mask + c
        mask = (((r ^ mask) >> 2) / c) | r
```

### Trace: n=4, k=2 (iterate all 4C2 = 6 masks)

```
  mask = 0011 (3)   → {0,1}
  mask = 0101 (5)   → {0,2}
  mask = 0110 (6)   → {1,2}
  mask = 1001 (9)   → {0,3}
  mask = 1010 (10)  → {1,3}
  mask = 1100 (12)  → {2,3}

  Only 6 iterations instead of 16!
```

---

## Complexity

| Approach | Time |
|----------|------|
| All subsets, check property | O(n × 2^n) |
| Gosper's hack (size k only) | O(n × C(n,k)) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| General pattern | Loop 0 to 2^n−1, check condition |
| Subset sum | Accumulate included elements, compare to target |
| Size-k subsets | Use popcount or Gosper's hack |
| Gosper's hack | Next permutation of bits — skips irrelevant masks |
| Practical limit | n ≤ 20 for full enumeration |

---

## Quick Revision Questions

1. **How many subsets of [1,2,3,4] have sum ≥ 7?**
   <details><summary>Answer</summary>Check all 16: {3,4}=7 ✓, {1,3,4}=8 ✓, {2,3,4}=9 ✓, {1,2,4}=7 ✓, {1,2,3}=6 ✗, {1,2,3,4}=10 ✓. Count = 5.</details>

2. **What is the first mask produced by Gosper's hack for n=5, k=3?**
   <details><summary>Answer</summary>(1<<3) - 1 = 0b00111 = 7. First subset: {0, 1, 2}.</details>

3. **Why do we skip mask=0 when counting non-empty subsets?**
   <details><summary>Answer</summary>mask=0 represents the empty set {}. Start the loop from mask=1 to exclude it.</details>

4. **Can bitmask subset enumeration find the OPTIMAL subset?**
   <details><summary>Answer</summary>Yes, by tracking the best value across all masks. It's brute force but guaranteed correct for small n.</details>

5. **How would you count subsets containing element 0?**
   <details><summary>Answer</summary>Only count masks where bit 0 is set: `if mask & 1 != 0`. This halves the valid masks to 2^(n-1).</details>

---

[← Previous: Generate All Subsets](01-generate-all-subsets.md) | [Next: Subset Sum with Bitmasks →](03-subset-sum.md)
