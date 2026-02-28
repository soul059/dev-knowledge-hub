# Chapter 6.3: Subset Sum with Bitmasks

> **Unit 6 — Subsets with Bits**
> Solve the classic subset sum problem by brute-force bitmask enumeration and the meet-in-the-middle optimization.

---

## Overview

**Subset Sum Problem:** Given a set of n integers and a target T, determine if any subset sums to T. While the DP approach runs in O(n × T), the bitmask approach works regardless of T's value — useful when T is huge but n is small.

---

## Brute-Force Bitmask: O(n × 2^n)

```
FUNCTION subsetSum(arr[], n, target):
    FOR mask = 0 TO (1 << n) - 1:
        sum = 0
        FOR i = 0 TO n - 1:
            IF mask & (1 << i):
                sum += arr[i]
        IF sum == target:
            RETURN TRUE (and the subset)
    RETURN FALSE
```

### Trace: arr = [3, 7, 1, 8], target = 11

```
  mask = 0  (0000): sum = 0
  mask = 1  (0001): sum = 3
  mask = 2  (0010): sum = 7
  mask = 3  (0011): sum = 3+7 = 10
  mask = 4  (0100): sum = 1
  mask = 5  (0101): sum = 3+1 = 4
  mask = 6  (0110): sum = 7+1 = 8
  mask = 7  (0111): sum = 3+7+1 = 11  ✓ FOUND!
  
  Subset: {3, 7, 1}
```

---

## Meet in the Middle: O(n × 2^(n/2))

For n up to ~40, split the array in half and use a two-pointer or hash set approach.

```
  ┌────────────────────────────────────────────────────────┐
  │  1. Split array into two halves A and B (each ~n/2)    │
  │  2. Enumerate all subset sums of A → store in set SA  │
  │  3. Enumerate all subset sums of B                     │
  │  4. For each sum_b in SB, check if (target - sum_b)   │
  │     exists in SA                                       │
  └────────────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION meetInMiddle(arr[], n, target):
    mid = n / 2
    
    // Generate all subset sums for left half
    SA = set()
    FOR mask = 0 TO (1 << mid) - 1:
        sum = 0
        FOR i = 0 TO mid - 1:
            IF mask & (1 << i):
                sum += arr[i]
        SA.ADD(sum)
    
    // Check right half against left
    FOR mask = 0 TO (1 << (n - mid)) - 1:
        sum = 0
        FOR i = 0 TO (n - mid) - 1:
            IF mask & (1 << i):
                sum += arr[mid + i]
        IF (target - sum) IN SA:
            RETURN TRUE
    
    RETURN FALSE
```

### Complexity Comparison

```
  n = 40:
  
  Brute force:    2^40 ≈ 10^12     ← WAY too slow
  Meet in middle: 2^20 + 2^20 ≈ 2×10^6 ← very fast!
```

---

## Trace: Meet in the Middle

```
  arr = [1, 3, 5, 7, 9, 11], target = 20, n = 6

  Left half:  [1, 3, 5]    (mid = 3)
  Right half: [7, 9, 11]

  Left subset sums SA:
    000→0, 001→1, 010→3, 011→4, 100→5, 101→6, 110→8, 111→9
    SA = {0, 1, 3, 4, 5, 6, 8, 9}

  Right subset sums, check (20 - sum_b) in SA:
    000→0:  need 20 in SA? No
    001→7:  need 13 in SA? No
    010→9:  need 11 in SA? No
    011→16: need 4 in SA?  YES! ✓
    
  Found: left subset with sum=4 is {1,3} → mask=011
         right subset with sum=16 is {7,9} → mask=011
         
  Answer: {1, 3, 7, 9} sums to 20  ✓
```

---

## Summary Table

| Approach | Time | Space | n limit |
|----------|------|-------|---------|
| Brute force bitmask | O(n × 2^n) | O(1) | n ≤ 20 |
| Meet in the middle | O(n × 2^(n/2)) | O(2^(n/2)) | n ≤ 40 |
| DP approach | O(n × T) | O(T) | T ≤ 10^6 |

---

## When to Use Each

```
  ┌────────────────────────────────────────────────────┐
  │  n ≤ 20, any T        → Bitmask brute force       │
  │  n ≤ 40, any T        → Meet in the middle        │
  │  n large, T ≤ 10^6    → DP                        │
  │  n large, T large     → NP-hard, approximation    │
  └────────────────────────────────────────────────────┘
```

---

## Quick Revision Questions

1. **How many subsets does the brute-force check for n = 15?**
   <details><summary>Answer</summary>2^15 = 32,768 subsets. Very manageable.</details>

2. **In meet-in-the-middle with n=36, how many subsets per half?**
   <details><summary>Answer</summary>Each half has n/2 = 18 elements → 2^18 = 262,144 subsets per half. Total work ≈ 2 × 262,144 ≈ 500K — very fast.</details>

3. **Can meet-in-the-middle find ALL subsets that sum to target?**
   <details><summary>Answer</summary>Yes. Instead of a set, use a map from sum to list of masks. For each right sum, look up all matching left sums.</details>

4. **Why is DP sometimes better even for small n?**
   <details><summary>Answer</summary>When T is small, DP runs in O(n×T) which can be faster than O(n×2^n). E.g., n=20, T=100: DP=2000 vs Bitmask=20M.</details>

5. **What data structure is best for SA in meet-in-the-middle?**
   <details><summary>Answer</summary>A hash set gives O(1) average lookup per query. Alternatively, sort SA and use binary search for O(log 2^(n/2)) = O(n/2) per query.</details>

---

[← Previous: Count Subsets](02-count-subsets.md) | [Next: Bitmasking for Subsets →](04-bitmasking-for-subsets.md)
