# Chapter 3: Count Occurrences

[← Previous: Find Last Occurrence](02-find-last-occurrence.md) | [Next: Search Insert Position →](04-search-insert-position.md)

---

## Overview

Count how many times a target value appears in a sorted array. By combining **first** and **last** occurrence, we get an O(log n) solution — much better than O(n) linear scan.

---

## Problem Statement

```
   Input:  arr = [1, 2, 3, 3, 3, 3, 5, 8], target = 3
   Output: 4  (3 appears 4 times)

   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │
   └───┴───┴───┴───┴───┴───┴───┴───┘
            ↑               ↑
          first=2         last=5
          
   count = last - first + 1 = 5 - 2 + 1 = 4
```

---

## The Approach

```
   Step 1: Find FIRST occurrence → index 'f'
   Step 2: Find LAST  occurrence → index 'l'
   Step 3: Count = l - f + 1  (if found)

   ┌───────────────────────────────────────────┐
   │  [... first occurrence ... last occurrence ...]  │
   │        ↑                    ↑              │
   │      index f              index l         │
   │        ◄── count = l - f + 1 ──►          │
   └───────────────────────────────────────────┘
```

---

## Algorithm (Pseudocode)

```
FUNCTION countOccurrences(arr, n, target):
    first = findFirstOccurrence(arr, n, target)
    
    IF first == -1:
        RETURN 0                   // Target not found at all
    
    last = findLastOccurrence(arr, n, target)
    
    RETURN last - first + 1
```

---

## Step-by-Step Example

### Array: `[2, 4, 4, 4, 4, 6, 8, 8]`, Target: `4`

**Step 1: Find First Occurrence of 4**

| Step | lo | hi | mid | arr[mid] | Action | result |
|------|----|----|-----|----------|--------|--------|
| 1 | 0 | 7 | 3 | 4 | match → result=3, hi=2 | 3 |
| 2 | 0 | 2 | 1 | 4 | match → result=1, hi=0 | 1 |
| 3 | 0 | 0 | 0 | 2 | < target → lo=1 | 1 |
| 4 | 1 | 0 | — | — | STOP | first = **1** |

**Step 2: Find Last Occurrence of 4**

| Step | lo | hi | mid | arr[mid] | Action | result |
|------|----|----|-----|----------|--------|--------|
| 1 | 0 | 7 | 3 | 4 | match → result=3, lo=4 | 3 |
| 2 | 4 | 7 | 5 | 6 | > target → hi=4 | 3 |
| 3 | 4 | 4 | 4 | 4 | match → result=4, lo=5 | 4 |
| 4 | 5 | 4 | — | — | STOP | last = **4** |

**Step 3:** count = 4 - 1 + 1 = **4** ✓

```
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 2 │ 4 │ 4 │ 4 │ 4 │ 6 │ 8 │ 8 │
   └───┴───┴───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5   6   7
        ↑               ↑
      first=1         last=4
      
   count = 4 - 1 + 1 = 4 ✓
```

---

## Implementation

### Java

```java
public static int countOccurrences(int[] arr, int target) {
    int first = findFirst(arr, target);
    if (first == -1) return 0;
    
    int last = findLast(arr, target);
    return last - first + 1;
}
```

### Python

```python
def count_occurrences(arr, target):
    first = find_first(arr, target)
    if first == -1:
        return 0
    last = find_last(arr, target)
    return last - first + 1
```

---

## Complexity Comparison

```
   ┌──────────────────────────────────────────────┐
   │  Method          │ Time      │ Space          │
   ├──────────────────┼───────────┼────────────────┤
   │  Linear Scan     │ O(n)      │ O(1)           │
   │  Two Binary      │ O(log n)  │ O(1)           │
   │  Searches        │ (2×log n) │                │
   └──────────────────┴───────────┴────────────────┘
   
   For n = 1,000,000:
   Linear: ~1,000,000 operations
   Binary: ~40 operations (2 × 20)
```

---

## Edge Cases

| Case | first | last | Count |
|------|-------|------|-------|
| Target not found | -1 | — | 0 |
| Single occurrence | 3 | 3 | 1 |
| All elements same | 0 | n-1 | n |
| Target at start | 0 | 2 | 3 |
| Target at end | 5 | 7 | 3 |

---

## Related Problem: Check if Element Exists

```python
def exists(arr, target):
    return count_occurrences(arr, target) > 0

# Or simply use standard binary search (faster)
def exists(arr, target):
    return binary_search(arr, target) != -1
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Approach** | findFirst + findLast |
| **Formula** | `last - first + 1` |
| **Time** | O(log n) — two binary searches |
| **Space** | O(1) |
| **If not found** | Return 0 |

---

## Quick Revision Questions

1. **How do you count occurrences in O(log n) time?**
2. **Why is `last - first + 1` the count and not `last - first`?**
3. **What happens if you only call findFirst and the target isn't found?**
4. **For array `[5, 5, 5, 5, 5]` with target = 5, what are first, last, and count?**
5. **Can this approach work for unsorted arrays? Why or why not?**
6. **How many binary search calls does this approach make in total?**

---

[← Previous: Find Last Occurrence](02-find-last-occurrence.md) | [Next: Search Insert Position →](04-search-insert-position.md)
