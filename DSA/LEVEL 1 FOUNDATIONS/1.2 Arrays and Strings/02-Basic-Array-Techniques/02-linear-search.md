# Chapter 2: Linear Search

[← Previous: Traversal Patterns](01-traversal-patterns.md) | [Back to README](../README.md) | [Next: Finding Min/Max →](03-finding-min-max.md)

---

## Overview

Linear search (sequential search) is the simplest search algorithm. It checks every element one by one until the target is found or the array ends. While not the fastest, it works on **unsorted** data and is the basis for understanding more advanced searches.

---

## Algorithm

```
FUNCTION linearSearch(arr, n, target):
    FOR i = 0 TO n-1:
        IF arr[i] == target:
            RETURN i          // found at index i
    RETURN -1                 // not found
```

---

## Step-by-Step Trace

### Example: Search for 30 in [10, 25, 30, 45, 50]

```
  Target = 30

  Step 1: i=0, arr[0]=10, 10≠30 → continue
  ┌──────┬──────┬──────┬──────┬──────┐
  │>>10  │  25  │  30  │  45  │  50  │   ✗
  └──────┴──────┴──────┴──────┴──────┘

  Step 2: i=1, arr[1]=25, 25≠30 → continue
  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │>>25  │  30  │  45  │  50  │   ✗
  └──────┴──────┴──────┴──────┴──────┘

  Step 3: i=2, arr[2]=30, 30==30 → FOUND! Return 2
  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  25  │>>30  │  45  │  50  │   ✓ Found at index 2
  └──────┴──────┴──────┴──────┴──────┘
```

### Example: Search for 99 (not present)

```
  Target = 99

  i=0: 10≠99 ✗   i=1: 25≠99 ✗   i=2: 30≠99 ✗
  i=3: 45≠99 ✗   i=4: 50≠99 ✗

  All elements checked → Return -1 (not found)
  Comparisons made: 5 (= n)
```

---

## Complexity Analysis

| Case | Comparisons | Time |
|------|-------------|------|
| Best case | 1 (target at index 0) | O(1) |
| Worst case | n (target last or absent) | O(n) |
| Average case | n/2 | O(n) |

**Space complexity:** O(1) — only uses a loop variable.

---

## Variations

### 1. Search for First Occurrence
Standard linear search already returns the first occurrence.

### 2. Search for Last Occurrence
```
FUNCTION lastOccurrence(arr, n, target):
    result = -1
    FOR i = 0 TO n-1:
        IF arr[i] == target:
            result = i         // update each time found
    RETURN result
```

### 3. Count All Occurrences
```
FUNCTION countOccurrences(arr, n, target):
    count = 0
    FOR i = 0 TO n-1:
        IF arr[i] == target:
            count = count + 1
    RETURN count
```

### 4. Search with Sentinel (Optimization)

Eliminates the bounds check in the loop by placing the target at the end:

```
FUNCTION sentinelSearch(arr, n, target):
    last = arr[n-1]
    arr[n-1] = target     // place sentinel

    i = 0
    WHILE arr[i] != target:
        i = i + 1         // no need for i < n check!

    arr[n-1] = last       // restore original

    IF i < n-1 OR last == target:
        RETURN i
    RETURN -1
```

```
  Original:  [10, 25, 30, 45, 50]   target = 30
  Sentinel:  [10, 25, 30, 45, 30]   ← replaced last with target
                                      guarantees loop will stop

  Benefit: Removes one comparison (i < n) from each iteration
  Practical speedup: ~20-30% for large arrays
```

---

## Linear Search vs Binary Search

```
  Array: [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
  Target: 23

  LINEAR SEARCH: Check 2, 5, 8, 12, 16, 23 → 6 comparisons
  
  BINARY SEARCH: 
    mid=16 (too small) → right half
    mid=38 (too big)   → left half  
    mid=23 (found!)    → 3 comparisons

  ┌──────────────────────────────────────┐
  │  n = 1,000,000                        │
  │  Linear search: up to 1,000,000 comps │
  │  Binary search: up to 20 comps        │
  │                                        │
  │  BUT binary search needs SORTED data!  │
  └──────────────────────────────────────┘
```

---

## When to Use Linear Search

| Scenario | Use Linear Search? |
|----------|-------------------|
| Unsorted array | Yes — only option |
| Small array (n < 50) | Yes — fast enough |
| Search once | Yes — sorting first is wasteful |
| Linked list | Yes — no random access for binary search |
| Need all occurrences | Yes — must scan everything |
| Sorted array, many searches | No — use binary search |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Algorithm | Check each element sequentially |
| Time (best) | O(1) |
| Time (worst/avg) | O(n) |
| Space | O(1) |
| Requires sorted? | No |
| Returns | First index of target, or -1 |
| Optimization | Sentinel technique |

---

## Quick Revision Questions

1. **What is the average number of comparisons** in a linear search of an array with n elements?

2. **How does the sentinel technique improve performance?** What comparison is eliminated?

3. **If you need to search 1000 times in a sorted array**, is linear search still the best choice? What would you do instead?

4. **Write pseudocode to find all indices** where a target appears in an array.

5. **Can linear search work on a linked list?** Can binary search? Why or why not?

6. **What's the worst-case scenario for linear search?** How many comparisons?

---

[← Previous: Traversal Patterns](01-traversal-patterns.md) | [Back to README](../README.md) | [Next: Finding Min/Max →](03-finding-min-max.md)
