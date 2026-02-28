# Chapter 3: Recursive Binary Search Implementation

[← Previous: Iterative Implementation](02-iterative-implementation.md) | [Next: Loop Invariant →](04-loop-invariant.md)

---

## Overview

The **recursive** implementation of binary search expresses the divide-and-conquer nature more naturally by calling itself on smaller subproblems. Each recursive call searches a smaller portion of the array.

---

## Algorithm (Pseudocode)

```
FUNCTION binarySearchRecursive(arr, lo, hi, target):
    IF lo > hi:
        RETURN -1                        // Base case: not found
    
    mid = lo + (hi - lo) / 2
    
    IF arr[mid] == target:
        RETURN mid                       // Base case: found
    ELSE IF arr[mid] < target:
        RETURN binarySearchRecursive(arr, mid + 1, hi, target)
    ELSE:
        RETURN binarySearchRecursive(arr, lo, mid - 1, target)
```

---

## Recursion Tree Visualization

### Array: `[2, 5, 8, 12, 16, 23, 38, 56]`, Target: `23`

```
   binarySearch(arr, 0, 7, 23)
   │  mid = 3, arr[3] = 12
   │  12 < 23 → search right half
   │
   └── binarySearch(arr, 4, 7, 23)
       │  mid = 5, arr[5] = 23
       │  23 == 23 → FOUND!
       │
       └── return 5
           └── return 5 (bubbles up)
```

### Target NOT found: `target = 10`

```
   binarySearch(arr, 0, 7, 10)
   │  mid=3, arr[3]=12, 12>10 → go left
   │
   └── binarySearch(arr, 0, 2, 10)
       │  mid=1, arr[1]=5, 5<10 → go right
       │
       └── binarySearch(arr, 2, 2, 10)
           │  mid=2, arr[2]=8, 8<10 → go right
           │
           └── binarySearch(arr, 3, 2, 10)
               │  lo=3 > hi=2 → NOT FOUND
               │
               └── return -1
                   └── return -1 (bubbles up)
                       └── return -1
                           └── return -1
```

---

## Call Stack Diagram

```
   Stack Growth for searching 23 in [2,5,8,12,16,23,38,56]:

   ┌─────────────────────────────────┐
   │ binarySearch(arr, 4, 7, 23)    │ ← Frame 2 (mid=5, FOUND!)
   │   lo=4, hi=7, mid=5           │    Returns 5
   ├─────────────────────────────────┤
   │ binarySearch(arr, 0, 7, 23)    │ ← Frame 1
   │   lo=0, hi=7, mid=3           │    Gets 5, Returns 5
   └─────────────────────────────────┘
   
   Maximum stack depth: O(log n)
```

---

## Implementation

### Java

```java
public static int binarySearch(int[] arr, int lo, int hi, int target) {
    if (lo > hi) return -1;  // Base case: not found
    
    int mid = lo + (hi - lo) / 2;
    
    if (arr[mid] == target) {
        return mid;
    } else if (arr[mid] < target) {
        return binarySearch(arr, mid + 1, hi, target);
    } else {
        return binarySearch(arr, lo, mid - 1, target);
    }
}

// Wrapper for clean API
public static int binarySearch(int[] arr, int target) {
    return binarySearch(arr, 0, arr.length - 1, target);
}
```

### Python

```python
def binary_search(arr, lo, hi, target):
    if lo > hi:
        return -1
    
    mid = lo + (hi - lo) // 2
    
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search(arr, mid + 1, hi, target)
    else:
        return binary_search(arr, lo, mid - 1, target)

# Wrapper
def search(arr, target):
    return binary_search(arr, 0, len(arr) - 1, target)
```

### C++

```cpp
int binarySearch(vector<int>& arr, int lo, int hi, int target) {
    if (lo > hi) return -1;
    
    int mid = lo + (hi - lo) / 2;
    
    if (arr[mid] == target)
        return mid;
    else if (arr[mid] < target)
        return binarySearch(arr, mid + 1, hi, target);
    else
        return binarySearch(arr, lo, mid - 1, target);
}
```

---

## Step-by-Step Trace

### Array: `[1, 4, 7, 10, 13, 16, 19]`, Target: `13`

| Call # | lo | hi | mid | arr[mid] | Decision | Next Call |
|--------|----|----|-----|----------|----------|-----------|
| 1 | 0 | 6 | 3 | 10 | 10 < 13 | (4, 6, 13) |
| 2 | 4 | 6 | 5 | 16 | 16 > 13 | (4, 4, 13) |
| 3 | 4 | 4 | 4 | 13 | 13 == 13 | **FOUND! Return 4** |

```
   Call 1: binarySearch(arr, 0, 6, 13)
   ┌───┬───┬───┬────┬────┬────┬────┐
   │ 1 │ 4 │ 7 │ 10 │ 13 │ 16 │ 19 │
   └───┴───┴───┴────┴────┴────┴────┘
    lo          mid                hi    arr[3]=10 < 13

   Call 2: binarySearch(arr, 4, 6, 13)
   ┌───┬───┬───┬────┬────┬────┬────┐
   │ 1 │ 4 │ 7 │ 10 │ 13 │ 16 │ 19 │
   └───┴───┴───┴────┴────┴────┴────┘
                      lo   mid   hi    arr[5]=16 > 13

   Call 3: binarySearch(arr, 4, 4, 13)
   ┌───┬───┬───┬────┬────┬────┬────┐
   │ 1 │ 4 │ 7 │ 10 │ 13 │ 16 │ 19 │
   └───┴───┴───┴────┴────┴────┴────┘
                     lo=mid=hi         arr[4]=13 == 13 ✓

   Return chain: 4 → 4 → 4  (answer bubbles up)
```

---

## Iterative vs Recursive Comparison

```
   ┌────────────────────┬──────────────┬───────────────┐
   │     Aspect         │  Iterative   │  Recursive    │
   ├────────────────────┼──────────────┼───────────────┤
   │  Time Complexity   │  O(log n)    │  O(log n)     │
   │  Space Complexity  │  O(1)        │  O(log n)     │
   │  Code Clarity      │  Moderate    │  More natural  │
   │  Stack Overflow?   │  No          │  Possible*    │
   │  Tail Recursive?   │  N/A        │  Yes          │
   │  Performance       │  Slightly    │  Slight       │
   │                    │  better      │  overhead     │
   └────────────────────┴──────────────┴───────────────┘
   
   * Stack overflow very unlikely: log₂(10⁹) ≈ 30 frames
```

### Space Comparison Visual

```
   Iterative (O(1) space):
   ┌──────────────────┐
   │ lo, hi, mid vars │  ← Only one set exists at a time
   └──────────────────┘

   Recursive (O(log n) space):
   ┌──────────────────┐
   │ Frame 4: lo,hi   │  ← Each frame has its own variables
   ├──────────────────┤
   │ Frame 3: lo,hi   │
   ├──────────────────┤
   │ Frame 2: lo,hi   │
   ├──────────────────┤
   │ Frame 1: lo,hi   │
   └──────────────────┘
   depth = O(log n)
```

---

## Tail Recursion Optimization

Binary search is **tail-recursive** — the recursive call is the last operation. Some compilers can optimize this to use O(1) space:

```
   // Tail recursive — recursive call is the LAST thing done
   FUNCTION bsRecursive(arr, lo, hi, target):
       IF lo > hi: RETURN -1
       mid = lo + (hi - lo) / 2
       IF arr[mid] == target: RETURN mid
       IF arr[mid] < target:
           RETURN bsRecursive(arr, mid+1, hi, target)  ← tail call
       RETURN bsRecursive(arr, lo, mid-1, target)      ← tail call
```

> **Note:** Java does NOT optimize tail recursion. Python has a recursion depth limit (~1000). C++ with `-O2` may optimize it.

---

## When to Use Recursive vs Iterative

| Use Recursive When | Use Iterative When |
|--------------------|--------------------|
| Code clarity matters more | Performance is critical |
| Problem is naturally recursive | Stack space is limited |
| Learning/educational context | Production code |
| Language supports tail-call optimization | Java or Python |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Base Case 1** | `lo > hi → return -1` (not found) |
| **Base Case 2** | `arr[mid] == target → return mid` (found) |
| **Recursive Case 1** | `arr[mid] < target → search(mid+1, hi)` |
| **Recursive Case 2** | `arr[mid] > target → search(lo, mid-1)` |
| **Time** | O(log n) |
| **Space** | O(log n) — call stack |
| **Tail Recursive** | Yes |
| **Preferred?** | Generally no — iterative is preferred |

---

## Quick Revision Questions

1. **What are the two base cases in recursive binary search?**
2. **What is the space complexity of recursive binary search? Why?**
3. **What does "tail recursive" mean and why is it relevant to binary search?**
4. **For an array of size 2³² (about 4 billion), what is the maximum recursion depth?**
5. **Why is the iterative version generally preferred in production code?**
6. **Trace recursive binary search on `[3, 6, 9, 12, 15]` with target = 6.**

---

[← Previous: Iterative Implementation](02-iterative-implementation.md) | [Next: Loop Invariant →](04-loop-invariant.md)
