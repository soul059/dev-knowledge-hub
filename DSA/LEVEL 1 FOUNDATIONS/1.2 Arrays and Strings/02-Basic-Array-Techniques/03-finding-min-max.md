# Chapter 3: Finding Min/Max

[← Previous: Linear Search](02-linear-search.md) | [Back to README](../README.md) | [Next: Reversing an Array →](04-reversing-an-array.md)

---

## Overview

Finding the minimum and maximum elements in an array is one of the most basic and frequently needed operations. This chapter covers multiple approaches with different efficiency trade-offs.

---

## Approach 1: Simple Traversal

### Find Minimum
```
FUNCTION findMin(arr, n):
    min_val = arr[0]
    FOR i = 1 TO n-1:
        IF arr[i] < min_val:
            min_val = arr[i]
    RETURN min_val
```

### Find Maximum
```
FUNCTION findMax(arr, n):
    max_val = arr[0]
    FOR i = 1 TO n-1:
        IF arr[i] > max_val:
            max_val = arr[i]
    RETURN max_val
```

### Trace: Find Min in [38, 12, 45, 7, 23]

```
  i=0: min_val = 38 (initialize)

  i=1: arr[1]=12, 12 < 38 → min_val = 12
  ┌──────┬──────┬──────┬──────┬──────┐
  │  38  │>>12  │  45  │   7  │  23  │   min=12 ✓
  └──────┴──────┴──────┴──────┴──────┘

  i=2: arr[2]=45, 45 < 12? No → min_val = 12
  i=3: arr[3]=7,  7 < 12  → min_val = 7
  ┌──────┬──────┬──────┬──────┬──────┐
  │  38  │  12  │  45  │>> 7  │  23  │   min=7 ✓
  └──────┴──────┴──────┴──────┴──────┘

  i=4: arr[4]=23, 23 < 7? No → min_val = 7

  Result: min = 7
  Comparisons: n - 1 = 4
```

**Time: O(n)** | **Space: O(1)** | **Comparisons: n - 1**

---

## Approach 2: Find Both Min and Max (Naive)

```
FUNCTION findMinMax_naive(arr, n):
    min_val = arr[0]
    max_val = arr[0]
    FOR i = 1 TO n-1:
        IF arr[i] < min_val:
            min_val = arr[i]
        IF arr[i] > max_val:
            max_val = arr[i]
    RETURN (min_val, max_val)

Comparisons: 2(n - 1) — two comparisons per element
```

---

## Approach 3: Tournament Method (Optimized Min and Max)

Compare elements **in pairs** to reduce total comparisons:

```
FUNCTION findMinMax_optimized(arr, n):
    // Initialize based on odd or even n
    IF n is odd:
        min_val = arr[0]
        max_val = arr[0]
        start = 1
    ELSE:
        IF arr[0] < arr[1]:
            min_val = arr[0]
            max_val = arr[1]
        ELSE:
            min_val = arr[1]
            max_val = arr[0]
        start = 2

    // Process pairs
    FOR i = start TO n-2 STEP 2:
        IF arr[i] < arr[i+1]:
            smaller = arr[i]
            larger = arr[i+1]
        ELSE:
            smaller = arr[i+1]
            larger = arr[i]

        IF smaller < min_val:
            min_val = smaller
        IF larger > max_val:
            max_val = larger

    RETURN (min_val, max_val)
```

### Trace: [3, 5, 1, 8, 2, 9]

```
  n = 6 (even)
  
  Step 1: Compare arr[0]=3 and arr[1]=5
          min=3, max=5

  Step 2: Pair (arr[2], arr[3]) = (1, 8)
          Compare pair: 1 < 8 → smaller=1, larger=8
          Compare 1 < min(3)? Yes → min=1
          Compare 8 > max(5)? Yes → max=8

  Step 3: Pair (arr[4], arr[5]) = (2, 9)
          Compare pair: 2 < 9 → smaller=2, larger=9
          Compare 2 < min(1)? No
          Compare 9 > max(8)? Yes → max=9

  Result: min=1, max=9
  Comparisons: 1 + 3 + 3 = 7
```

### Comparison Count Analysis

```
  Naive method:     2(n-1) comparisons
  Tournament method: 3⌊n/2⌋ comparisons

  For n = 100:
    Naive:      198 comparisons
    Tournament: 150 comparisons (25% fewer!)

  For n = 1,000,000:
    Naive:      1,999,998 comparisons
    Tournament: 1,500,000 comparisons (saves ~500K!)
```

---

## Approach 4: Divide and Conquer

```
FUNCTION findMinMax_DC(arr, low, high):
    IF low == high:
        RETURN (arr[low], arr[low])    // single element
    
    IF high == low + 1:                // two elements
        IF arr[low] < arr[high]:
            RETURN (arr[low], arr[high])
        ELSE:
            RETURN (arr[high], arr[low])
    
    mid = (low + high) / 2
    (min1, max1) = findMinMax_DC(arr, low, mid)
    (min2, max2) = findMinMax_DC(arr, mid+1, high)
    
    RETURN (MIN(min1, min2), MAX(max1, max2))
```

```
  Array: [3, 5, 1, 8, 2, 9]

                    [3, 5, 1, 8, 2, 9]
                   ╱                   ╲
          [3, 5, 1]                   [8, 2, 9]
          ╱      ╲                    ╱      ╲
       [3, 5]   [1]               [8, 2]   [9]
       min=3     min=1            min=2     min=9
       max=5     max=1            max=8     max=9
          ╲      ╱                    ╲      ╱
       min=1, max=5              min=2, max=9
                   ╲                   ╱
                  min=1, max=9

  Same number of comparisons as tournament method: 3n/2 - 2
```

---

## Finding Kth Smallest/Largest (Preview)

For finding elements other than min/max:

| Problem | Simple Approach | Optimal |
|---------|----------------|---------|
| 2nd smallest | Track two variables | O(n) |
| Kth smallest | Sort then access | O(n log n) |
| Kth smallest | QuickSelect | O(n) average |
| Kth smallest | Min-Heap | O(n + k log n) |

### Finding 2nd Minimum

```
FUNCTION secondMin(arr, n):
    first = INFINITY
    second = INFINITY
    FOR i = 0 TO n-1:
        IF arr[i] < first:
            second = first
            first = arr[i]
        ELSE IF arr[i] < second AND arr[i] != first:
            second = arr[i]
    RETURN second
```

---

## Common Pitfalls

```
  ┌───────────────────────────────────────────────┐
  │  MISTAKES TO AVOID:                            │
  │                                                │
  │  1. Initializing min/max to 0                  │
  │     ✗ min = 0        (fails if all positive)   │
  │     ✓ min = arr[0]   (use first element)       │
  │     ✓ min = INFINITY (guaranteed safe)         │
  │                                                │
  │  2. Not handling empty arrays                   │
  │     Always check: IF n == 0: RETURN ERROR      │
  │                                                │
  │  3. Off-by-one in loop                          │
  │     If min = arr[0], start loop from i = 1     │
  │     If min = INFINITY, start from i = 0        │
  └───────────────────────────────────────────────┘
```

---

## Summary Table

| Method | Comparisons for Min | Comparisons for Both | Space |
|--------|-------------------|---------------------|-------|
| Simple traversal | n - 1 | 2(n - 1) | O(1) |
| Tournament/Pair | — | 3⌊n/2⌋ | O(1) |
| Divide & Conquer | — | 3n/2 - 2 | O(log n) stack |
| Sorting first | O(n log n) | O(n log n) | O(1)–O(n) |

---

## Quick Revision Questions

1. **What is the minimum number of comparisons needed** to find the maximum in an array of n elements? Prove it.

2. **Why is the tournament method better** for finding both min and max simultaneously?

3. **If you initialize `min_val = 0`**, what bug can occur? Give an example array.

4. **How would you find the second largest element** in an array in a single pass?

5. **What is the space complexity of the divide-and-conquer approach?** Why?

6. **Can you find both min and max in fewer than 3n/2 comparisons?** (This is actually the theoretical lower bound.)

---

[← Previous: Linear Search](02-linear-search.md) | [Back to README](../README.md) | [Next: Reversing an Array →](04-reversing-an-array.md)
