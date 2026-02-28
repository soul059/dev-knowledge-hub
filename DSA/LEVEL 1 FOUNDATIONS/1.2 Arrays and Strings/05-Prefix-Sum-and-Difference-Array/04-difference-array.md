# Chapter 4: Difference Array

[← Previous: Range Sum Queries](03-range-sum-queries.md) | [Back to README](../README.md) | [Next: Subarray Sum Equals K →](05-subarray-sum-equals-k.md)

---

## Overview

A **difference array** is the inverse of a prefix sum. While prefix sums help answer range **queries**, difference arrays help perform range **updates** efficiently. If you need to add a value to every element in a range [l, r], the difference array lets you do it in O(1) instead of O(n).

---

## Core Idea

```
  PROBLEM: Add 5 to every element from index 2 to index 5.

  BRUTE FORCE: O(r - l + 1) per update
  arr[2] += 5
  arr[3] += 5
  arr[4] += 5
  arr[5] += 5

  DIFFERENCE ARRAY: O(1) per update
  diff[2] += 5     ← start of range
  diff[6] -= 5     ← one past end of range

  Then reconstruct the array with prefix sum of diff.
```

---

## What Is a Difference Array?

```
  Given:  arr = [a₀, a₁, a₂, a₃, a₄]

  Difference array:
  diff[0] = arr[0]
  diff[i] = arr[i] - arr[i-1]   for i ≥ 1

  Example:
  arr  = [3, 5, 7, 7, 2]
  diff = [3, 2, 2, 0, -5]
          │  │  │  │   │
          │  │  │  │   └─ 2-7 = -5
          │  │  │  └─── 7-7 = 0
          │  │  └──── 7-5 = 2
          │  └───── 5-3 = 2
          └────── 3 (first element)

  PROPERTY: prefix_sum(diff) = arr
  prefix[0] = 3
  prefix[1] = 3+2 = 5
  prefix[2] = 5+2 = 7
  prefix[3] = 7+0 = 7
  prefix[4] = 7+(-5) = 2
  → [3, 5, 7, 7, 2] = original arr  ✓
```

---

## Range Update with Difference Array

To add value `val` to all elements in range [l, r]:

```
  diff[l]   += val     ← "start adding val from index l"
  diff[r+1] -= val     ← "stop adding val after index r"

  (If r+1 >= n, skip the second operation)
```

### Why This Works

```
  diff[l] += val means: when we take prefix sums,
  the effect of +val persists from index l onward.

  diff[r+1] -= val means: the +val effect is cancelled
  starting at index r+1.

  Net result: only indices l through r get +val.

  ┌─────────────────────────────────────────────┐
  │  0   1   2   3   4   5   6   7   8   9     │
  │  ─   ─   ─   ─   ─   ─   ─   ─   ─   ─   │
  │              +5              -5             │
  │              │←── +5 zone ──→│              │
  │                                             │
  │  diff:   0   0  +5   0   0   0  -5   0     │
  │  prefix: 0   0   5   5   5   5   0   0     │
  │              [───────────────]              │
  │              adds 5 to indices 2-5          │
  └─────────────────────────────────────────────┘
```

---

## Algorithm: Multiple Range Updates

```
FUNCTION rangeUpdates(n, updates):
    // updates = list of (l, r, val)
    diff = array of size n+1, all zeros

    // Step 1: Apply all updates to diff array
    FOR EACH (l, r, val) IN updates:
        diff[l] += val
        IF r+1 <= n:
            diff[r+1] -= val

    // Step 2: Convert diff to actual array (prefix sum)
    arr = array of size n
    arr[0] = diff[0]
    FOR i = 1 TO n-1:
        arr[i] = arr[i-1] + diff[i]

    RETURN arr
```

---

## Detailed Trace

```
  n = 6, initial arr = [0, 0, 0, 0, 0, 0]
  
  Updates:
    1. Add 3 to [1, 4]
    2. Add 2 to [2, 5]
    3. Add -1 to [0, 3]

  diff = [0, 0, 0, 0, 0, 0, 0]   (size n+1 = 7)
                                    extra slot for r+1

  Update 1: Add 3 to [1, 4]
  diff[1] += 3, diff[5] -= 3
  diff = [0, 3, 0, 0, 0, -3, 0]

  Update 2: Add 2 to [2, 5]
  diff[2] += 2, diff[6] -= 2
  diff = [0, 3, 2, 0, 0, -3, -2]

  Update 3: Add -1 to [0, 3]
  diff[0] += -1, diff[4] -= -1
  diff = [-1, 3, 2, 0, 1, -3, -2]

  Prefix sum of diff (first 6 elements):
  arr[0] = -1
  arr[1] = -1 + 3 = 2
  arr[2] = 2 + 2 = 4
  arr[3] = 4 + 0 = 4
  arr[4] = 4 + 1 = 5
  arr[5] = 5 + (-3) = 2

  Result: [-1, 2, 4, 4, 5, 2]

  Verify manually:
  Index 0: -1 (only update 3: -1)         ✓
  Index 1: -1+3 = 2 (update 1: +3, update 3: -1) ✓
  Index 2: -1+3+2 = 4 (updates 1,2,3)     ✓
  Index 3: -1+3+2 = 4 (updates 1,2,3)     ✓
  Index 4: 3+2 = 5 (updates 1,2)           ✓
  Index 5: 2 (only update 2: +2)           ✓
```

---

## Prefix Sum vs Difference Array: Duality

```
  ┌──────────────────────────────────────────────────┐
  │         PREFIX SUM ←──── inverse ────→ DIFF ARRAY │
  │                                                   │
  │  If B = prefix_sum(A), then A = diff(B)          │
  │  If A = diff(B), then B = prefix_sum(A)          │
  │                                                   │
  │  PREFIX SUM:                                      │
  │    • Answers: range SUM queries in O(1)           │
  │    • Preprocesses: O(n)                           │
  │                                                   │
  │  DIFFERENCE ARRAY:                                │
  │    • Performs: range UPDATES in O(1)               │
  │    • Reconstructs: O(n) prefix sum at the end     │
  └──────────────────────────────────────────────────┘
```

---

## Handling Non-Zero Initial Array

If the array starts with existing values:

```
FUNCTION buildDiff(arr, n):
    diff = array of size n
    diff[0] = arr[0]
    FOR i = 1 TO n-1:
        diff[i] = arr[i] - arr[i-1]
    RETURN diff

// Or simply: apply updates to a diff initialized
// from the original array, then prefix-sum back.
```

---

## Problem: Corporate Flight Bookings (LeetCode 1109)

```
  n flights (1-indexed), bookings = [[l, r, seats], ...]
  Find total seats reserved for each flight.

  n = 5, bookings = [[1,2,10], [2,3,20], [2,5,25]]

  diff = [0, 0, 0, 0, 0, 0]   (size n+1, 0-indexed)

  Booking [1,2,10]: diff[0]+=10, diff[2]-=10
  diff = [10, 0, -10, 0, 0, 0]

  Booking [2,3,20]: diff[1]+=20, diff[3]-=20
  diff = [10, 20, -10, -20, 0, 0]

  Booking [2,5,25]: diff[1]+=25, diff[5]-=25
  diff = [10, 45, -10, -20, 0, -25]

  Prefix sum:
  [10, 55, 45, 25, 25]

  Answer: [10, 55, 45, 25, 25]  ✓
```

---

## Problem: Range Addition (LeetCode 370)

```
  Initialize array of size n to all zeros.
  Apply updates of form [startIndex, endIndex, increment].
  Return final array.

  → This is EXACTLY the difference array use case!
```

---

## 2D Difference Array

For rectangle updates on a 2D matrix:

```
  To add val to rectangle (r1,c1) to (r2,c2):

  diff[r1][c1]     += val
  diff[r1][c2+1]   -= val
  diff[r2+1][c1]   -= val
  diff[r2+1][c2+1] += val

  Then 2D prefix sum to reconstruct.
```

```
  ┌─────────────────────────────┐
  │  +val ──────── -val         │
  │  │              │           │
  │  │   (r1,c1)    │  (r1,c2+1)│
  │  │              │           │
  │  │    RANGE     │           │
  │  │              │           │
  │  -val ──────── +val         │
  │  (r2+1,c1)    (r2+1,c2+1)  │
  └─────────────────────────────┘
```

---

## Complexity

| Operation | Brute Force | Difference Array |
|-----------|-------------|-----------------|
| Single range update | O(r - l + 1) | O(1) |
| Q range updates | O(n × Q) | O(Q) |
| Reconstruction | N/A | O(n) |
| Total (Q updates) | O(n × Q) | O(n + Q) |

---

## When to Use Difference Array

```
  ✓ Multiple range updates, query final state
  ✓ All updates applied first, then query once
  ✓ Flight/bus booking problems
  ✓ Adding constant to range of array

  ✗ Need to query between updates (use Segment Tree + lazy)
  ✗ Updates are point-wise, not range (just update directly)
  ✗ Need range max/min after updates (more complex)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| What is diff array | diff[i] = arr[i] - arr[i-1] |
| Range update [l,r] by val | diff[l] += val, diff[r+1] -= val |
| Reconstruct | Prefix sum of diff array |
| Build time | O(n) |
| Update time | O(1) per update |
| Total for Q updates | O(n + Q) |
| Inverse of | Prefix sum |

---

## Quick Revision Questions

1. **What is the relationship** between prefix sum and difference array?

2. **To add 10 to indices 3 through 7**, what two operations do you perform on the difference array?

3. **Why do we need diff[r+1] -= val** and not diff[r] -= val?

4. **Given arr = [2, 4, 6, 8], build the difference array.** Then add 3 to range [1, 3] and reconstruct.

5. **How many operations does it take** to apply 1000 range updates on an array of size 10⁶ using difference array vs brute force?

6. **Can you mix difference array with prefix sum** in the same problem? When would you do that?

---

[← Previous: Range Sum Queries](03-range-sum-queries.md) | [Back to README](../README.md) | [Next: Subarray Sum Equals K →](05-subarray-sum-equals-k.md)
