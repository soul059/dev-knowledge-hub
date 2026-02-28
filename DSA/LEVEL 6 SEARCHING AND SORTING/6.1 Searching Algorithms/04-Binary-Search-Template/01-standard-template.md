# Chapter 1: Standard Binary Search Template

[← Previous: Unit 3 — Peak Element](../03-Binary-Search-Variations/06-peak-element.md) | [Next: Boundary Finding Template →](02-boundary-finding-template.md)

---

## Overview

Having a **reliable template** makes binary search problems much easier. The standard template is the most common and works for basic "find exact match" problems.

---

## Template 1: Standard (Exact Match)

```
FUNCTION binarySearch(arr, target):
    lo = 0
    hi = n - 1

    WHILE lo <= hi:                        // ① Inclusive range
        mid = lo + (hi - lo) / 2           // ② Safe mid calculation

        IF arr[mid] == target:             // ③ Found
            RETURN mid
        ELSE IF arr[mid] < target:         // ④ Go right
            lo = mid + 1
        ELSE:                              // ⑤ Go left
            hi = mid - 1

    RETURN -1                               // ⑥ Not found
```

### Template Characteristics

```
   ┌────────────────────────────────────────────────┐
   │  Template 1: Standard                          │
   │                                                │
   │  Loop:       while (lo <= hi)                  │
   │  Range:      [lo, hi] — both inclusive          │
   │  Search:     Exact match                       │
   │  Update lo:  mid + 1                           │
   │  Update hi:  mid - 1                           │
   │  Termination: lo > hi (range empty)            │
   │  Post-loop:  return -1                         │
   └────────────────────────────────────────────────┘
```

### When to Use

- Finding exact value in sorted array
- Simple "is target present?" queries
- When you need the index of the target

### Visual

```
   Search Space: [lo .............. hi]
   
   Iteration 1:  [lo ...... mid ...... hi]
                           │
                  ┌────────┼─────────┐
                  │ Found  │         │
               go left    done    go right
                  │                  │
   Iteration 2: [lo .. mid .. hi]  [lo .. mid .. hi]
                        │                  │
                  (repeat)           (repeat)
   
   Terminates: [lo > hi] → empty range → not found
```

---

## Template Code (All Languages)

### Java

```java
public static int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

### Python

```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
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

### C++

```cpp
int binarySearch(vector<int>& arr, int target) {
    int lo = 0, hi = (int)arr.size() - 1;
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

## The Five Components to Remember

```
   ╔═══════════════════════════════════════════════╗
   ║             BINARY SEARCH TEMPLATE            ║
   ║                                               ║
   ║  1. INITIALIZE:  lo = 0, hi = n - 1           ║
   ║  2. CONDITION:   while (lo <= hi)              ║
   ║  3. MIDPOINT:    mid = lo + (hi - lo) / 2     ║
   ║  4. COMPARE:     3-way if/else                ║
   ║  5. RESULT:      return mid or return -1       ║
   ╚═══════════════════════════════════════════════╝
```

---

## Correctness Checklist

Before moving on, verify your implementation:

| Check | What to Verify |
|-------|---------------|
| ✅ Initialization | `lo = 0`, `hi = n - 1` |
| ✅ Loop condition | `lo <= hi` (not `<`) |
| ✅ Mid formula | `lo + (hi - lo) / 2` (no overflow) |
| ✅ Update rules | `lo = mid + 1`, `hi = mid - 1` |
| ✅ Termination | Loop always terminates (search space shrinks) |
| ✅ Return value | Return index on match, -1 on miss |

---

## Summary Table

| Aspect | Template 1 (Standard) |
|--------|----------------------|
| **Purpose** | Find exact match |
| **Loop** | `while (lo <= hi)` |
| **Range** | `[lo, hi]` inclusive |
| **On match** | Return immediately |
| **lo update** | `mid + 1` |
| **hi update** | `mid - 1` |
| **Post-loop** | `-1` (not found) |
| **Time** | O(log n) |

---

## Quick Revision Questions

1. **What are the five components of the standard binary search template?**
2. **Why is the loop condition `<=` and not `<`?**
3. **What guarantees that the loop terminates?**
4. **Can this template handle duplicates? What does it return?**
5. **Adapt this template for a descending-sorted array.**
6. **What is the state of `lo` and `hi` after the loop ends with "not found"?**

---

[← Previous: Unit 3 — Peak Element](../03-Binary-Search-Variations/06-peak-element.md) | [Next: Boundary Finding Template →](02-boundary-finding-template.md)
