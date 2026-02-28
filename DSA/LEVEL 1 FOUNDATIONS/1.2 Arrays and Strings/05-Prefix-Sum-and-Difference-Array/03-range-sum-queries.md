# Chapter 3: Range Sum Queries

[← Previous: Building Prefix Sum Array](02-building-prefix-sum.md) | [Back to README](../README.md) | [Next: Difference Array →](04-difference-array.md)

---

## Overview

Range sum queries ask: "What is the sum of elements between index `l` and index `r`?" This chapter covers multiple approaches, from brute force to prefix sums, and explores classic problems that use range queries as a subroutine.

---

## Problem Types

```
  ┌────────────────────────────────────────────────────┐
  │  TYPE 1: Static array, multiple queries            │
  │  → Use prefix sum (this chapter)                   │
  │                                                    │
  │  TYPE 2: Dynamic array (updates + queries)         │
  │  → Use Segment Tree or BIT (advanced topic)        │
  │                                                    │
  │  TYPE 3: Single query                              │
  │  → Just iterate (no preprocessing needed)          │
  └────────────────────────────────────────────────────┘
```

---

## Approach 1: Brute Force — O(n) per query

```
FUNCTION rangeSum(arr, l, r):
    total = 0
    FOR i = l TO r:
        total = total + arr[i]
    RETURN total
```

```
  For Q queries: O(n × Q) total
```

---

## Approach 2: Prefix Sum — O(1) per query

```
// Preprocessing
FUNCTION buildPrefix(arr, n):
    prefix = new array of size n+1
    prefix[0] = 0
    FOR i = 1 TO n:
        prefix[i] = prefix[i-1] + arr[i-1]
    RETURN prefix

// Query
FUNCTION rangeSum(prefix, l, r):
    RETURN prefix[r+1] - prefix[l]
```

### Full Example with Multiple Queries

```
  arr = [1, 3, 5, 7, 9, 11]
  prefix = [0, 1, 4, 9, 16, 25, 36]

  Query 1: sum(0, 2) = prefix[3] - prefix[0] = 9 - 0 = 9
  Verify:  1+3+5 = 9  ✓

  Query 2: sum(1, 4) = prefix[5] - prefix[1] = 25 - 1 = 24
  Verify:  3+5+7+9 = 24  ✓

  Query 3: sum(3, 5) = prefix[6] - prefix[3] = 36 - 9 = 27
  Verify:  7+9+11 = 27  ✓

  Query 4: sum(2, 2) = prefix[3] - prefix[2] = 9 - 4 = 5
  Verify:  5 = 5  ✓ (single element)

  Query 5: sum(0, 5) = prefix[6] - prefix[0] = 36 - 0 = 36
  Verify:  1+3+5+7+9+11 = 36  ✓ (entire array)
```

---

## Comparison

| Metric | Brute Force | Prefix Sum |
|--------|-------------|------------|
| Preprocessing | O(1) | O(n) |
| Per query | O(n) | O(1) |
| Q queries total | O(n × Q) | O(n + Q) |
| Space | O(1) | O(n) |
| Best when | Single query | Multiple queries |

```
  Break-even: When Q > 1, prefix sum starts winning.
  For Q = 10⁵, n = 10⁵:
    Brute: 10¹⁰ operations → TLE
    Prefix: 2 × 10⁵ operations → instant
```

---

## Problem: Subarray Sum Equals K

**Count the number of subarrays whose sum equals k.**

```
  arr = [1, 1, 1], k = 2
  Answer: 2  (subarrays [1,1] at positions 0-1 and 1-2)
```

### Approach: Prefix Sum + HashMap

```
  Key insight:
  sum(l, r) = k
  prefix[r] - prefix[l-1] = k
  prefix[l-1] = prefix[r] - k

  So at each position r, count how many previous prefix
  values equal (current prefix - k).
```

```
FUNCTION subarraySum(arr, n, k):
    prefixMap = {0: 1}    // prefix sum → count of occurrences
    currentPrefix = 0
    count = 0

    FOR i = 0 TO n-1:
        currentPrefix = currentPrefix + arr[i]
        
        target = currentPrefix - k
        IF target IN prefixMap:
            count = count + prefixMap[target]

        prefixMap[currentPrefix] = prefixMap.getOrDefault(currentPrefix, 0) + 1

    RETURN count
```

### Trace: arr = [1, 2, 3, -1, 2], k = 4

```
  ┌─────┬────────┬────────────┬──────┬──────────────────────────┐
  │  i  │ prefix │ target=p-k │count │ map                      │
  ├─────┼────────┼────────────┼──────┼──────────────────────────┤
  │init │   0    │     -      │  0   │ {0:1}                    │
  │  0  │   1    │  1-4=-3    │  0   │ {0:1, 1:1}               │
  │  1  │   3    │  3-4=-1    │  0   │ {0:1, 1:1, 3:1}          │
  │  2  │   6    │  6-4=2     │  0   │ {0:1, 1:1, 3:1, 6:1}    │
  │  3  │   5    │  5-4=1     │  1   │ {0:1, 1:1, 3:1, 6:1, 5:1}│
  │  4  │   7    │  7-4=3     │  2   │ {..., 7:1}               │
  └─────┴────────┴────────────┴──────┴──────────────────────────┘

  Answer: 2
  Subarrays: [1,2,3,-1] sum=5? No...
  
  Let me re-verify:
  At i=3: prefix=5, target=1, map has {1:1} → count += 1
    meaning prefix[3]-prefix[0]=5-1=4=k ✓ → subarray [2,3,-1] (indices 1-3)
  At i=4: prefix=7, target=3, map has {3:1} → count += 1
    meaning prefix[4]-prefix[1]=7-3=4=k ✓ → subarray [3,-1,2] (indices 2-4)

  Answer: 2  ✓
```

---

## Problem: Equilibrium Index

Find an index where the sum of elements to its left equals the sum to its right.

```
FUNCTION equilibriumIndex(arr, n):
    totalSum = SUM(arr)
    leftSum = 0

    FOR i = 0 TO n-1:
        rightSum = totalSum - leftSum - arr[i]
        IF leftSum == rightSum:
            RETURN i
        leftSum = leftSum + arr[i]

    RETURN -1
```

### Trace: arr = [1, 7, 3, 6, 5, 6]

```
  totalSum = 28

  i=0: rightSum = 28-0-1 = 27,  leftSum=0  ≠ 27
       leftSum = 0+1 = 1
  i=1: rightSum = 28-1-7 = 20,  leftSum=1  ≠ 20
       leftSum = 1+7 = 8
  i=2: rightSum = 28-8-3 = 17,  leftSum=8  ≠ 17
       leftSum = 8+3 = 11
  i=3: rightSum = 28-11-6 = 11, leftSum=11 == 11  ✓
       RETURN 3

  Verify: left = 1+7+3 = 11, right = 5+6 = 11  ✓
```

---

## Problem: Subarray with Zero Sum

Check if any subarray has sum = 0.

```
  Key insight: If prefix[i] == prefix[j] for i ≠ j,
  then sum(i+1, j) = prefix[j] - prefix[i] = 0.

  Also if prefix[i] == 0, then sum(0, i) = 0.
```

```
FUNCTION hasZeroSumSubarray(arr, n):
    seen = empty set
    ADD 0 TO seen    // handles subarray starting at index 0
    currentPrefix = 0

    FOR i = 0 TO n-1:
        currentPrefix = currentPrefix + arr[i]
        IF currentPrefix IN seen:
            RETURN true
        ADD currentPrefix TO seen

    RETURN false
```

---

## Problem: Maximum Subarray Sum in Range Queries

Given Q queries, each asking for max element in a range — prefix sums alone won't help (need Sparse Table or Segment Tree). But for **sum** queries, prefix sum is perfect.

---

## Problem: Count Subarrays with Sum Divisible by K

```
  sum(l, r) mod k = 0
  ↔ prefix[r] mod k = prefix[l-1] mod k

  Count pairs of prefix values with same remainder.
```

```
FUNCTION subarraysDivByK(arr, n, k):
    remainderCount = array of size k, all zeros
    remainderCount[0] = 1    // empty prefix has remainder 0
    currentPrefix = 0
    count = 0

    FOR i = 0 TO n-1:
        currentPrefix = currentPrefix + arr[i]
        remainder = ((currentPrefix % k) + k) % k   // handle negatives
        count = count + remainderCount[remainder]
        remainderCount[remainder] = remainderCount[remainder] + 1

    RETURN count
```

### Trace: arr = [4, 5, 0, -2, -3, 1], k = 5

```
  ┌───┬────────┬───────────┬─────────────────────┬───────┐
  │ i │ prefix │ rem (% 5) │ remainderCount      │ count │
  ├───┼────────┼───────────┼─────────────────────┼───────┤
  │   │   0    │     0     │ [1,0,0,0,0]         │   0   │
  │ 0 │   4    │     4     │ [1,0,0,0,1]         │   0   │
  │ 1 │   9    │     4     │ [1,0,0,0,2]         │   1   │
  │ 2 │   9    │     4     │ [1,0,0,0,3]         │   3   │
  │ 3 │   7    │     2     │ [1,0,1,0,3]         │   3   │
  │ 4 │   4    │     4     │ [1,0,1,0,4]         │   6   │
  │ 5 │   5    │     0     │ [2,0,1,0,4]         │   7   │
  └───┴────────┴───────────┴─────────────────────┴───────┘

  Answer: 7  ✓
```

---

## Range Queries Decision Guide

```
  ┌──────────────────────────────────────────────────────┐
  │  Need range SUM on static array?                     │
  │  → Prefix Sum: O(n) build, O(1) query               │
  │                                                      │
  │  Need range SUM with updates?                        │
  │  → BIT (Fenwick Tree): O(n) build, O(log n) both    │
  │  → Segment Tree: O(n) build, O(log n) both          │
  │                                                      │
  │  Need range MIN/MAX on static array?                 │
  │  → Sparse Table: O(n log n) build, O(1) query       │
  │                                                      │
  │  Need range MIN/MAX with updates?                    │
  │  → Segment Tree: O(n) build, O(log n) both          │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Range sum (static) | O(1) with prefix sum |
| Subarray sum = k | Prefix sum + HashMap, O(n) |
| Equilibrium index | Use totalSum − leftSum − arr[i] |
| Zero sum subarray | Repeated prefix value |
| Sum divisible by k | Group prefix remainders |
| Multiple queries | O(n + Q) with preprocessing |

---

## Quick Revision Questions

1. **How do you answer a range sum query** from index l to r using prefix sums?

2. **For the "subarray sum equals k" problem**, what equation connects prefix sums to subarrays of sum k?

3. **Why do we initialize the prefix map with {0: 1}** in the subarray sum problem?

4. **How do you find the equilibrium index** without building a full prefix and suffix array?

5. **What does it mean** when two prefix sum values are equal? What subarray does that imply?

6. **For subarrays divisible by k**, why do we need `((prefix % k) + k) % k` instead of just `prefix % k`?

---

[← Previous: Building Prefix Sum Array](02-building-prefix-sum.md) | [Back to README](../README.md) | [Next: Difference Array →](04-difference-array.md)
