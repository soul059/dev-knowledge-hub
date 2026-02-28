# Chapter 2: Boundary Finding Template

[← Previous: Standard Template](01-standard-template.md) | [Next: Left / Right Bias →](03-left-right-bias.md)

---

## Overview

The **boundary finding template** is the most versatile binary search template. Instead of finding an exact match, it finds the **boundary** between elements that satisfy a condition and those that don't.

> **Key Idea:** Think of the array as divided into two zones — TRUE and FALSE — and find where the boundary lies.

---

## The Core Concept

```
   Given a condition (predicate), the array looks like:

   ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
   │FALSE│FALSE│FALSE│FALSE│TRUE │TRUE │TRUE │TRUE │
   └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
                              ↑
                           BOUNDARY
                    (First TRUE / Last FALSE)
```

This pattern appears EVERYWHERE in binary search problems:
- First occurrence: "Is `arr[i] >= target`?"
- Search insert: "Is the value at position `i` ≥ target?"
- Capacity problems: "Can we ship with capacity C?"

---

## Template 2: Find First TRUE (Left Boundary)

```
FUNCTION findFirstTrue(arr, n, condition):
    lo = 0
    hi = n                              // Note: n, not n-1 (to handle "all false")

    WHILE lo < hi:                       // Note: < not <=
        mid = lo + (hi - lo) / 2

        IF condition(arr[mid]):
            hi = mid                     // mid could be the answer
        ELSE:
            lo = mid + 1                 // mid is definitely NOT the answer

    RETURN lo                             // lo == hi == boundary
```

### Visual Walk-Through

```
   Condition: arr[i] >= 7
   arr = [1, 3, 5, 7, 9, 11]
   
   Mapped to:  F  F  F  T  T  T
   
   Iteration 1: lo=0, hi=6, mid=3
   ┌───┬───┬───┬───┬───┬────┐
   │ 1 │ 3 │ 5 │ 7 │ 9 │ 11 │    arr[3]=7 ≥ 7 → TRUE → hi=3
   └───┴───┴───┴───┴───┴────┘
    lo          mid           hi

   Iteration 2: lo=0, hi=3, mid=1
   ┌───┬───┬───┬───┐
   │ 1 │ 3 │ 5 │ 7 │                arr[1]=3 ≥ 7 → FALSE → lo=2
   └───┴───┴───┴───┘
    lo  mid      hi

   Iteration 3: lo=2, hi=3, mid=2
         ┌───┬───┐
         │ 5 │ 7 │                   arr[2]=5 ≥ 7 → FALSE → lo=3
         └───┴───┘
         lo   hi

   lo=3 == hi=3 → STOP. Answer: index 3 (arr[3]=7)  ✓
```

---

## Template 3: Find Last FALSE (Right Boundary)

```
FUNCTION findLastFalse(arr, n, condition):
    lo = -1                             // To handle "all true"
    hi = n - 1

    WHILE lo < hi:
        mid = lo + (hi - lo + 1) / 2    // Ceiling division (right-biased mid)

        IF condition(arr[mid]):
            hi = mid - 1                // mid is TRUE, answer is before it
        ELSE:
            lo = mid                     // mid is FALSE, could be the answer

    RETURN lo                             // Last FALSE position
```

---

## Why This Template is Powerful

Almost every binary search problem can be reframed as a boundary problem:

```
   ┌──────────────────────────────┬──────────────────────────────┐
   │         Problem              │     Condition (predicate)    │
   ├──────────────────────────────┼──────────────────────────────┤
   │ First occurrence of x       │ arr[i] >= x                  │
   │ Last occurrence of x        │ arr[i] > x (then go left)    │
   │ Search insert position      │ arr[i] >= target             │
   │ Ceiling of x                │ arr[i] >= x                  │
   │ Floor of x                  │ arr[i] > x (then go left)    │
   │ Sqrt of n                   │ mid*mid > n                  │
   │ Min capacity to ship        │ canShip(capacity)            │
   │ Koko eating bananas         │ canFinish(speed)             │
   └──────────────────────────────┴──────────────────────────────┘
```

---

## Implementation

### Java (Find First True)

```java
// Generic boundary finder — pass any condition
public static int findFirstTrue(int lo, int hi, IntPredicate condition) {
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (condition.test(mid)) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}

// Example: Find first element >= target
int idx = findFirstTrue(0, arr.length, mid -> arr[mid] >= target);
```

### Python (Find First True)

```python
def find_first_true(lo, hi, condition):
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if condition(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo

# Example: First element >= 7
idx = find_first_true(0, len(arr), lambda mid: arr[mid] >= 7)
```

---

## Key Differences from Standard Template

```
   ┌─────────────────────┬──────────────────┬────────────────────┐
   │     Aspect          │ Standard (T1)    │ Boundary (T2)      │
   ├─────────────────────┼──────────────────┼────────────────────┤
   │ Loop condition      │ lo <= hi         │ lo < hi            │
   │ Update hi           │ hi = mid - 1     │ hi = mid           │
   │ Update lo           │ lo = mid + 1     │ lo = mid + 1       │
   │ Found action        │ return mid       │ hi = mid           │
   │ Post-loop result    │ return -1        │ return lo (=hi)    │
   │ Loop purpose        │ Find exact match │ Find boundary      │
   └─────────────────────┴──────────────────┴────────────────────┘
```

---

## The "Moving Walls" Mental Model

```
   Think of lo and hi as walls closing in:

   Start:
   lo ◄────────── search space ──────────► hi

   After each step, one wall moves:
   lo ◄─── shrinks ─── mid ─── stays ───► hi   (if TRUE: hi=mid)
   lo ◄─── stays ──── mid ─── shrinks ──► hi   (if FALSE: lo=mid+1)

   Eventually:
   lo=hi ← walls meet! This is the boundary.
```

---

## Proof of Correctness

**Invariant:** The answer is always in `[lo, hi]`

1. **Initialization:** `lo=0, hi=n` covers all possibilities including "no TRUE exists"
2. **`condition(mid)` is TRUE:** Answer could be at `mid` or before → `hi = mid` keeps answer in range
3. **`condition(mid)` is FALSE:** Answer must be after `mid` → `lo = mid + 1` keeps answer in range
4. **Termination:** `lo < hi` and at least one of `lo`/`hi` changes → search space shrinks → terminates

---

## Summary Table

| Aspect | Template 2 (Boundary) |
|--------|-----------------------|
| **Purpose** | Find first element satisfying condition |
| **Loop** | `while (lo < hi)` |
| **Range** | `[lo, hi)` or `[lo, hi]` |
| **On TRUE** | `hi = mid` (keep candidate) |
| **On FALSE** | `lo = mid + 1` (exclude) |
| **Post-loop** | `return lo` (boundary point) |
| **Time** | O(log n) |
| **Key Advantage** | Generalizes to any monotonic predicate |

---

## Quick Revision Questions

1. **What is the "boundary" in the boundary finding template?**
2. **Why is the loop condition `lo < hi` instead of `lo <= hi`?**
3. **Why do we set `hi = mid` (not `mid - 1`) when the condition is TRUE?**
4. **How can you use this template to implement `searchInsertPosition`?**
5. **What does the boundary template return when ALL elements are FALSE?**
6. **Explain the "moving walls" mental model for this template.**

---

[← Previous: Standard Template](01-standard-template.md) | [Next: Left / Right Bias →](03-left-right-bias.md)
