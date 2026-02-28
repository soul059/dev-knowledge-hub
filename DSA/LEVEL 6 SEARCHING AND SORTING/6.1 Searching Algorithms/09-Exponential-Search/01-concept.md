# Chapter 1: Exponential Search — Concept

[← Previous Unit: Ternary Comparison](../08-Ternary-Search/03-comparison.md) | [Next: Unbounded Search →](02-unbounded-search.md)

---

## Overview

**Exponential search** (also called **doubling search** or **galloping search**) finds the range where a target exists by doubling the search bound, then applies binary search within that range. It's especially useful when the target is **near the beginning** of a sorted array or when the array size is **unknown/infinite**.

---

## How It Works

```
   Phase 1: EXPONENTIAL DOUBLING — Find the range
   ─────────────────────────────────────────────
   Start with bound = 1, double it each time:
   
   Check index:  1, 2, 4, 8, 16, 32, 64, ...
   
   Stop when arr[bound] ≥ target
   
   Phase 2: BINARY SEARCH — Search within the range
   ─────────────────────────────────────────────────
   Binary search in [bound/2, min(bound, n-1)]
```

---

## Visual Walkthrough

```
   arr = [2, 3, 4, 10, 40, 50, 55, 60, 70, 80, 90, 100]
   target = 55
   
   Phase 1: Exponential doubling
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬────┐
   │ 2 │ 3 │ 4 │10 │40 │50 │55 │60 │70 │80 │90 │100 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴────┘
     0   1   2   3   4   5   6   7   8   9  10   11
         ↑       ↑               ↑
       i=1     i=2             i=4
       3<55   4<55            40<55
   
   i=8: arr[8]=70 ≥ 55 → STOP!
   
   Range found: [i/2, i] = [4, 8]
   
   Phase 2: Binary search in [4, 8]
   ┌───┬───┬───┬───┬───┐
   │40 │50 │55 │60 │70 │
   └───┴───┴───┴───┴───┘
     4   5   6   7   8
   
   mid=6: arr[6]=55 = target → FOUND at index 6 ✓
```

---

## Pseudocode

```
   EXPONENTIAL-SEARCH(arr, n, target):
       // Edge case
       if arr[0] == target: return 0
       
       // Phase 1: Find range by doubling
       i = 1
       while i < n AND arr[i] < target:
           i = i * 2
       
       // Phase 2: Binary search in [i/2, min(i, n-1)]
       return binarySearch(arr, i/2, min(i, n-1), target)
```

---

## Implementation (Java)

```java
public int exponentialSearch(int[] arr, int target) {
    int n = arr.length;
    if (arr[0] == target) return 0;
    
    // Phase 1: Find range
    int i = 1;
    while (i < n && arr[i] < target) {
        i *= 2;
    }
    
    // Phase 2: Binary search
    int lo = i / 2;
    int hi = Math.min(i, n - 1);
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

---

## Implementation (Python)

```python
def exponential_search(arr, target):
    n = len(arr)
    if arr[0] == target:
        return 0
    
    # Phase 1: Find range
    i = 1
    while i < n and arr[i] < target:
        i *= 2
    
    # Phase 2: Binary search in [i//2, min(i, n-1)]
    lo, hi = i // 2, min(i, n - 1)
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

---

## Implementation (C++)

```cpp
int exponentialSearch(vector<int>& arr, int target) {
    int n = arr.size();
    if (arr[0] == target) return 0;
    
    int i = 1;
    while (i < n && arr[i] < target)
        i *= 2;
    
    int lo = i / 2, hi = min(i, n - 1);
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

---

## Time Complexity

```
   Phase 1: Doubling takes O(log i) steps, where i is the position of target
   Phase 2: Binary search on range of size i, takes O(log i)
   
   Total: O(log i) where i is the INDEX of the target
   
   Key insight: When the target is at position i:
   - Doubling: 1, 2, 4, ..., 2^k where 2^k ≥ i → k = log i steps
   - Binary search range: [i/2, i] → size i/2 → log(i/2) = O(log i)
   
   Best case:  O(1) — target at index 0
   Worst case: O(log n) — target at end
   Average:    O(log i) — adapts to target position!
```

---

## When Is Exponential Search Better Than Binary Search?

```
   ┌────────────────────────────────────────────────────────────┐
   │                                                            │
   │  Binary search: ALWAYS O(log n) regardless of target pos  │
   │                                                            │
   │  Exponential search: O(log i) where i = target position!  │
   │                                                            │
   │  If target is near the beginning:                         │
   │    Exponential: O(log i) << O(log n)  ← FASTER            │
   │                                                            │
   │  If target is near the end:                               │
   │    Exponential: O(log n) ≈ O(log n)  ← SAME              │
   │                                                            │
   └────────────────────────────────────────────────────────────┘
   
   Example: arr has 1 billion elements, target at index 100
   Binary:      log₂(10⁹) ≈ 30 comparisons
   Exponential: log₂(100)  ≈ 7 comparisons (+ 7 doubling)  ≈ 14 total
```

---

## Advantages of Exponential Search

```
   1. Adapts to target position → O(log i) not O(log n)
   2. Works on UNBOUNDED/INFINITE arrays (no n needed!)
   3. Cache-friendly in phase 1 (accesses beginning of array)
   4. Better for arrays where target is likely near the start
   5. Used in Java's Arrays.sort for TimSort merge step
```

---

## Summary

| Aspect | Value |
|--------|-------|
| Phase 1 | Doubling: O(log i) |
| Phase 2 | Binary search: O(log i) |
| Total | O(log i) |
| Space | O(1) |
| Best for | Target near beginning, unbounded arrays |

---

## Quick Revision Questions

1. **What are the two phases of exponential search?**
2. **Why is it called "exponential" search?**
3. **What is the time complexity in terms of target position?**
4. **When is exponential search faster than binary search?**
5. **Trace exponential search on `[1, 3, 5, 7, 9, 11]` for target = 3.**

---

[← Previous Unit: Ternary Comparison](../08-Ternary-Search/03-comparison.md) | [Next: Unbounded Search →](02-unbounded-search.md)
