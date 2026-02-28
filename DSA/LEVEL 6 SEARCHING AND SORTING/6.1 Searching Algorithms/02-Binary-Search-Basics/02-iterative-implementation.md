# Chapter 2: Iterative Binary Search Implementation

[← Previous: Binary Search Concept](01-binary-search-concept.md) | [Next: Recursive Implementation →](03-recursive-implementation.md)

---

## Overview

The **iterative** implementation of binary search uses a `while` loop to repeatedly halve the search space. It is the **preferred** implementation in practice due to O(1) space complexity.

---

## Algorithm (Pseudocode)

```
FUNCTION binarySearchIterative(arr, n, target):
    lo = 0
    hi = n - 1

    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2

        IF arr[mid] == target:
            RETURN mid
        ELSE IF arr[mid] < target:
            lo = mid + 1
        ELSE:
            hi = mid - 1

    RETURN -1
```

---

## Visual Walkthrough

### Anatomy of Each Iteration

```
   Before iteration:
   ┌─────────────────────────────────────────────┐
   │  arr[0] ... arr[lo] ... arr[mid] ... arr[hi] ... arr[n-1]  │
   │              ▲            ▲            ▲                     │
   │             lo           mid          hi                    │
   │              ◄─── search space ────►                        │
   └─────────────────────────────────────────────┘

   Decision:
   ┌──────────────────┐
   │  arr[mid] == target  →  RETURN mid
   │  arr[mid] <  target  →  lo = mid + 1  (search right)
   │  arr[mid] >  target  →  hi = mid - 1  (search left)
   └──────────────────┘
```

---

## Complete Trace Example

### Array: `[2, 4, 7, 10, 14, 18, 23, 31, 45, 60]`, Target: `18`

```
   Iteration 1:
   lo=0, hi=9, mid=4
   ┌───┬───┬───┬───┬────┬────┬────┬────┬────┬────┐
   │ 2 │ 4 │ 7 │10 │ 14 │ 18 │ 23 │ 31 │ 45 │ 60 │
   └───┴───┴───┴───┴────┴────┴────┴────┴────┴────┘
    lo              mid                          hi
   arr[4]=14 < 18 → lo = 5

   Iteration 2:
   lo=5, hi=9, mid=7
   ┌───┬───┬───┬───┬────┬────┬────┬────┬────┬────┐
   │ 2 │ 4 │ 7 │10 │ 14 │ 18 │ 23 │ 31 │ 45 │ 60 │
   └───┴───┴───┴───┴────┴────┴────┴────┴────┴────┘
                         lo        mid          hi
   arr[7]=31 > 18 → hi = 6

   Iteration 3:
   lo=5, hi=6, mid=5
   ┌───┬───┬───┬───┬────┬────┬────┬────┬────┬────┐
   │ 2 │ 4 │ 7 │10 │ 14 │ 18 │ 23 │ 31 │ 45 │ 60 │
   └───┴───┴───┴───┴────┴────┴────┴────┴────┴────┘
                        lo=mid  hi
   arr[5]=18 == 18 → FOUND at index 5! ✓
```

**Result:** Found in **3 iterations** (vs 6 for linear search).

---

## Implementation

### Java

```java
public static int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
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
    int lo = 0, hi = arr.size() - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target)
            return mid;
        else if (arr[mid] < target)
            lo = mid + 1;
        else
            hi = mid - 1;
    }
    return -1;
}
```

---

## Flowchart

```
    ┌──────────────────────┐
    │  lo = 0, hi = n - 1  │
    └──────────┬───────────┘
               │
    ┌──────────▼───────────┐     No
    │    lo <= hi ?         ├──────────────┐
    └──────────┬───────────┘              │
               │ Yes                       │
    ┌──────────▼───────────┐              │
    │  mid = lo+(hi-lo)/2  │              │
    └──────────┬───────────┘              │
               │                           │
    ┌──────────▼───────────┐    Yes       │
    │  arr[mid] == target? ├────────┐     │
    └──────────┬───────────┘        │     │
               │ No                  │     │
    ┌──────────▼───────────┐   ┌────▼───┐ │
    │  arr[mid] < target?  │   │Return  │ │
    └────┬────────────┬────┘   │ mid    │ │
      Yes│            │No      └────────┘ │
    ┌────▼────┐  ┌────▼────┐              │
    │lo=mid+1 │  │hi=mid-1 │              │
    └────┬────┘  └────┬────┘              │
         │            │                    │
         └─────┬──────┘                    │
               │ (loop back)               │
                                     ┌─────▼─────┐
                                     │ Return -1  │
                                     │ (Not Found)│
                                     └───────────┘
```

---

## Edge Cases to Handle

### 1. Empty Array
```java
int[] arr = {};
// lo = 0, hi = -1 → lo > hi → loop never runs → return -1  ✓
```

### 2. Single Element — Match
```java
int[] arr = {5};  // target = 5
// lo=0, hi=0, mid=0 → arr[0]==5 → return 0  ✓
```

### 3. Single Element — No Match
```java
int[] arr = {5};  // target = 3
// lo=0, hi=0, mid=0 → arr[0]=5 ≠ 3, 5 > 3 → hi = -1
// lo=0 > hi=-1 → return -1  ✓
```

### 4. Target at First Position
```
   [5, 10, 15, 20, 25]  target = 5
   mid=2 → 15>5 → hi=1
   mid=0 → 5==5 → FOUND ✓
```

### 5. Target at Last Position
```
   [5, 10, 15, 20, 25]  target = 25
   mid=2 → 15<25 → lo=3
   mid=4 → 25==25 → FOUND ✓
```

---

## Dry Run Template

Use this template to trace binary search on any array:

```
   Array: [                              ]
   Target: ___

   | Iter | lo | hi | mid | arr[mid] | Compare | Action    |
   |------|----|----|-----|----------|---------|-----------|
   |  1   |    |    |     |          |         |           |
   |  2   |    |    |     |          |         |           |
   |  3   |    |    |     |          |         |           |
   |  4   |    |    |     |          |         |           |

   Result: ___
```

---

## Common Pitfalls in Iterative Implementation

| Pitfall | Wrong | Correct |
|---------|-------|---------|
| Loop condition | `while (lo < hi)` | `while (lo <= hi)` |
| Mid calculation | `(lo + hi) / 2` | `lo + (hi - lo) / 2` |
| Update lo | `lo = mid` | `lo = mid + 1` |
| Update hi | `hi = mid` | `hi = mid - 1` |
| Return value | `return 0` for not found | `return -1` for not found |

> **Critical:** Using `lo = mid` or `hi = mid` (without +1/-1) causes an **infinite loop** when `lo + 1 == hi`.

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Loop Condition** | `while (lo <= hi)` |
| **Mid Calculation** | `lo + (hi - lo) / 2` |
| **Found** | `arr[mid] == target → return mid` |
| **Go Right** | `arr[mid] < target → lo = mid + 1` |
| **Go Left** | `arr[mid] > target → hi = mid - 1` |
| **Not Found** | Loop ends → `return -1` |
| **Time** | O(log n) |
| **Space** | O(1) |

---

## Quick Revision Questions

1. **What is the loop condition for standard iterative binary search? Why `<=` and not `<`?**
2. **What happens if we use `lo = mid` instead of `lo = mid + 1`?**
3. **Walk through binary search on `[1, 3, 5, 7, 9, 11]` with target = 7.**
4. **What does binary search return when the target is not found?**
5. **How does the iterative version achieve O(1) space complexity?**
6. **What is the maximum number of iterations for an array of size 16?**

---

[← Previous: Binary Search Concept](01-binary-search-concept.md) | [Next: Recursive Implementation →](03-recursive-implementation.md)
