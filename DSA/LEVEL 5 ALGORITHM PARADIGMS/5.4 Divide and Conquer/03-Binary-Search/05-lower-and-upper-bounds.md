# Chapter 5: Lower and Upper Bounds

## Overview

Finding the **lower bound** and **upper bound** of a value in a sorted array are fundamental variations of binary search. These operations form the backbone of range queries, counting occurrences, and many advanced algorithms.

---

## Definitions

```
Sorted Array:  [1, 2, 2, 2, 3, 5, 5, 7, 9]
Indices:        0  1  2  3  4  5  6  7  8

Target = 2:
  Lower Bound → index 1  (first position where arr[i] >= 2)
  Upper Bound → index 4  (first position where arr[i] >  2)

Target = 4:
  Lower Bound → index 5  (first position where arr[i] >= 4)
  Upper Bound → index 5  (first position where arr[i] >  4)

Target = 10:
  Lower Bound → index 9  (past end, = n)
  Upper Bound → index 9  (past end, = n)
```

```
┌────────────────────────────────────────────────────────┐
│  LOWER BOUND of x:                                     │
│    Smallest index i such that arr[i] >= x              │
│    → First element NOT LESS than x                      │
│                                                        │
│  UPPER BOUND of x:                                     │
│    Smallest index i such that arr[i] > x               │
│    → First element GREATER than x                       │
│                                                        │
│  COUNT of x = upper_bound(x) - lower_bound(x)         │
└────────────────────────────────────────────────────────┘
```

---

## Visual Comparison

```
Array: [1, 3, 3, 3, 5, 7, 7, 9]
Index:  0  1  2  3  4  5  6  7

               Target = 3
               
Lower Bound ─────┐     ┌───── Upper Bound
                  ▼     ▼
        [1,  3,  3,  3,  5,  7,  7,  9]
              ▲           ▲
              │           │
         First 3      First > 3
         index=1       index=4

Count of 3 = upper_bound - lower_bound = 4 - 1 = 3 ✓
```

---

## Lower Bound Implementation

```
function lowerBound(arr, n, target):
    lo = 0, hi = n        // NOTE: hi = n (one past last)
    while lo < hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] < target:
            lo = mid + 1   // mid is too small
        else:
            hi = mid       // mid could be the answer
    return lo              // first index where arr[i] >= target
```

### Trace: arr = [1, 3, 3, 3, 5, 7], target = 3

```
lo=0, hi=6, mid=3: arr[3]=3 >= 3  → hi=3
lo=0, hi=3, mid=1: arr[1]=3 >= 3  → hi=1
lo=0, hi=1, mid=0: arr[0]=1 < 3   → lo=1
lo=1, hi=1: STOP → return 1

Answer: index 1 (first element ≥ 3) ✓
```

---

## Upper Bound Implementation

```
function upperBound(arr, n, target):
    lo = 0, hi = n
    while lo < hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] <= target:
            lo = mid + 1   // mid is ≤ target, go right
        else:
            hi = mid       // mid is > target, could be answer
    return lo              // first index where arr[i] > target
```

### Trace: arr = [1, 3, 3, 3, 5, 7], target = 3

```
lo=0, hi=6, mid=3: arr[3]=3 <= 3  → lo=4
lo=4, hi=6, mid=5: arr[5]=7 > 3   → hi=5
lo=4, hi=5, mid=4: arr[4]=5 > 3   → hi=4
lo=4, hi=4: STOP → return 4

Answer: index 4 (first element > 3) ✓
```

---

## Key Difference: One Character

```
Lower Bound:  if arr[mid] <  target  →  lo = mid + 1
                           ▲
                           │ strictly less
                           
Upper Bound:  if arr[mid] <= target  →  lo = mid + 1
                           ▲▲
                           │ less OR equal
```

**Lower bound**: stops at first `>=` target  
**Upper bound**: stops at first `>` target

---

## Derived Operations

Using lower and upper bounds, we can solve many problems:

```
┌─────────────────────────────┬────────────────────────────────────┐
│ Operation                   │ Formula                            │
├─────────────────────────────┼────────────────────────────────────┤
│ First occurrence of x      │ lb = lower_bound(x)                │
│                             │ if lb < n && arr[lb] == x: lb     │
│                             │ else: not found                    │
├─────────────────────────────┼────────────────────────────────────┤
│ Last occurrence of x       │ ub = upper_bound(x) - 1            │
│                             │ if ub >= 0 && arr[ub] == x: ub   │
│                             │ else: not found                    │
├─────────────────────────────┼────────────────────────────────────┤
│ Count of x                 │ upper_bound(x) - lower_bound(x)    │
├─────────────────────────────┼────────────────────────────────────┤
│ Elements less than x       │ lower_bound(x)                     │
├─────────────────────────────┼────────────────────────────────────┤
│ Elements less than or      │ upper_bound(x)                     │
│   equal to x               │                                    │
├─────────────────────────────┼────────────────────────────────────┤
│ Elements greater than x    │ n - upper_bound(x)                 │
├─────────────────────────────┼────────────────────────────────────┤
│ Elements greater than or   │ n - lower_bound(x)                 │
│   equal to x               │                                    │
├─────────────────────────────┼────────────────────────────────────┤
│ Elements in range [a, b]   │ lower_bound(b+1) - lower_bound(a)  │
│                             │ OR upper_bound(b) - lower_bound(a) │
├─────────────────────────────┼────────────────────────────────────┤
│ Insertion point for x      │ lower_bound(x)                     │
│ (maintaining sorted order) │                                    │
├─────────────────────────────┼────────────────────────────────────┤
│ Closest element to x       │ Compare arr[lb-1] and arr[lb]      │
│ (nearest neighbor)         │                                    │
└─────────────────────────────┴────────────────────────────────────┘
```

---

## Floor and Ceiling

```
Floor(x): Largest element ≤ x
  → upper_bound(x) - 1
  → If result < 0: no floor exists

Ceiling(x): Smallest element ≥ x
  → lower_bound(x)
  → If result == n: no ceiling exists
```

### Example

```
Array: [2, 5, 8, 12, 16]

Floor(10)   = upper_bound(10) - 1 = 3 - 1 = 2 → arr[2] = 8  ✓
Ceiling(10) = lower_bound(10) = 3             → arr[3] = 12 ✓

Floor(5)    = upper_bound(5) - 1 = 2 - 1 = 1 → arr[1] = 5  ✓
Ceiling(5)  = lower_bound(5) = 1              → arr[1] = 5  ✓

Floor(1)    = upper_bound(1) - 1 = 0 - 1 = -1 → NO FLOOR
Ceiling(20) = lower_bound(20) = 5 = n         → NO CEILING
```

---

## Range Count Example

**Find the number of elements in [3, 7] in a sorted array.**

```
Array: [1, 2, 3, 4, 5, 6, 7, 8, 9]

lower_bound(3) = 2     (index of first element >= 3)
upper_bound(7) = 7     (index of first element > 7)

Count = 7 - 2 = 5      (elements: 3, 4, 5, 6, 7) ✓
```

---

## Language Standard Library Functions

| Language | Lower Bound | Upper Bound |
|----------|-------------|-------------|
| C++ | `std::lower_bound(begin, end, val)` | `std::upper_bound(begin, end, val)` |
| Java | Manual (or `Collections.binarySearch` + adjustment) | Manual |
| Python | `bisect.bisect_left(arr, val)` | `bisect.bisect_right(arr, val)` |
| C# | Manual | Manual |

### Python Example

```python
import bisect

arr = [1, 3, 3, 3, 5, 7]

lower = bisect.bisect_left(arr, 3)    # 1
upper = bisect.bisect_right(arr, 3)   # 4
count = upper - lower                  # 3
```

### C++ Example

```cpp
vector<int> arr = {1, 3, 3, 3, 5, 7};

auto lb = lower_bound(arr.begin(), arr.end(), 3);  // points to index 1
auto ub = upper_bound(arr.begin(), arr.end(), 3);  // points to index 4
int count = ub - lb;                                // 3
```

---

## Common Pitfalls

```
┌──────────────────────────────────────────────────────────────┐
│  PITFALL 1: Using hi = n-1 instead of hi = n                │
│  → Lower/upper bound can return n (past end)                │
│  → Must use hi = n to allow this result                     │
│                                                              │
│  PITFALL 2: Forgetting to check if element exists            │
│  → lower_bound returns position, not existence               │
│  → Must check: lb < n && arr[lb] == target                  │
│                                                              │
│  PITFALL 3: Confusing < with <= in the condition             │
│  → < gives lower_bound; <= gives upper_bound                │
│  → Off-by-one in the operator = off-by-many in result       │
│                                                              │
│  PITFALL 4: Using lo <= hi loop with hi = n                  │
│  → Use lo < hi loop when hi = n                              │
│  → Otherwise infinite loop risk                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Lower Bound | First index where arr[i] >= target |
| Upper Bound | First index where arr[i] > target |
| Count of x | upper_bound - lower_bound |
| Search range | [0, n] — note hi = n, not n-1 |
| Loop condition | while lo < hi (not lo <= hi) |
| Key difference | < vs <= in comparison |

---

## Quick Revision Questions

1. **What is the difference between lower bound and upper bound?**
2. **How do you count occurrences of element x using lower and upper bounds?**
3. **Why is hi initialized to n instead of n-1?**
4. **How do you find the floor of a value using these operations?**
5. **What Python module provides lower and upper bound functions, and what are they called?**

---

[← Previous: Binary Search on Answer](04-binary-search-on-answer.md) | [Next: Rotated Array Search →](06-rotated-array-search.md)
