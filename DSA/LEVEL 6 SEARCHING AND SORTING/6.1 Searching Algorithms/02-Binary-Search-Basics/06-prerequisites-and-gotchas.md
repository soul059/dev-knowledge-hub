# Chapter 6: Prerequisites & Common Gotchas

[← Previous: Complexity Analysis](05-complexity-analysis.md) | [Next: Unit 3 — Find First Occurrence →](../03-Binary-Search-Variations/01-find-first-occurrence.md)

---

## Overview

Binary search is deceptively simple. Even experienced programmers get it wrong. This chapter covers the **prerequisites**, **common bugs**, and **subtle pitfalls** that you must be aware of.

> *"Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky."* — Donald Knuth

---

## Prerequisite 1: Sorted Data

Binary search **only works on sorted arrays**. Using it on unsorted data produces incorrect results.

```
   CORRECT (sorted ascending):
   ┌───┬───┬───┬───┬───┬───┐
   │ 2 │ 5 │ 8 │ 12│ 16│ 23│   Binary Search ✓
   └───┴───┴───┴───┴───┴───┘

   WRONG (unsorted):
   ┌───┬───┬───┬───┬───┬───┐
   │ 8 │ 2 │ 23│ 5 │ 16│ 12│   Binary Search ✗ — may miss target!
   └───┴───┴───┴───┴───┴───┘

   Why? If mid=2, arr[2]=23, target=5:
   23 > 5 → search left [8, 2]
   But 5 is actually at index 3 (right side)!
   INCORRECT elimination of search space!
```

### What About Descending Order?

Binary search works on **descending** arrays too — just reverse the comparison:

```java
// Ascending: arr[mid] < target → lo = mid + 1
// Descending: arr[mid] > target → lo = mid + 1

if (arr[mid] > target) {
    lo = mid + 1;   // In descending, larger values are on the LEFT
} else if (arr[mid] < target) {
    hi = mid - 1;
} else {
    return mid;
}
```

---

## Prerequisite 2: Random Access

Binary search needs to access `arr[mid]` in O(1) time. This is only possible with **random-access** data structures.

```
   Array (Random Access):
   ┌───┬───┬───┬───┬───┐
   │ 1 │ 3 │ 5 │ 7 │ 9 │   arr[2] → O(1) ✓
   └───┴───┴───┴───┴───┘

   Linked List (Sequential Access):
   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
   │ 1 │──►│ 3 │──►│ 5 │──►│ 7 │──►│ 9 │
   └───┘   └───┘   └───┘   └───┘   └───┘
   
   To reach "index 2": traverse 0 → 1 → 2 → O(n) ✗
   Binary search becomes O(n log n) — worse than linear!
```

---

## Prerequisite 3: Comparable Elements

Elements must support comparison operators (`<`, `>`, `==`).

```
   Works: integers, floats, strings (lexicographic), dates
   Doesn't work: complex numbers (no natural ordering),
                  unstructured data without a comparator
```

---

## Gotcha 1: Integer Overflow in Mid Calculation

**THE most famous bug in binary search.** It existed in Java's `Arrays.binarySearch()` for 9 years!

```
   WRONG:
   mid = (lo + hi) / 2
   
   If lo = 2,000,000,000 and hi = 2,100,000,000:
   lo + hi = 4,100,000,000 > Integer.MAX_VALUE (2,147,483,647)
   → OVERFLOW → negative number → ArrayIndexOutOfBounds!

   CORRECT:
   mid = lo + (hi - lo) / 2
   
   hi - lo = 100,000,000 (safe)
   100,000,000 / 2 = 50,000,000
   lo + 50,000,000 = 2,050,000,000 (safe)
```

```
   Alternative (bit shift — also correct):
   mid = (lo + hi) >>> 1     // unsigned right shift (Java)
   
   This treats the overflow result as unsigned, giving the correct answer.
   But lo + (hi - lo) / 2 is clearer and recommended.
```

---

## Gotcha 2: Off-by-One in Loop Condition

```
   WRONG: while (lo < hi)
   
   Array: [5], target = 5
   lo=0, hi=0 → lo < hi is FALSE → loop never runs → return -1 ✗
   
   CORRECT: while (lo <= hi)
   lo=0, hi=0 → lo <= hi is TRUE → check arr[0] → FOUND ✓
```

```
   When lo == hi, there's exactly ONE element left to check.
   Using < instead of <= skips this element!
   
   ┌─────────────────────────────────────────┐
   │  lo < hi  → can miss single elements    │
   │  lo <= hi → checks every element     ✓  │
   └─────────────────────────────────────────┘
```

---

## Gotcha 3: Infinite Loop from Wrong Update

```
   WRONG:
   if (arr[mid] < target):
       lo = mid       // Should be mid + 1!
   
   Example: arr = [1, 3], target = 3
   
   Iteration 1: lo=0, hi=1, mid=0
     arr[0]=1 < 3 → lo = mid = 0   (NOT mid+1!)
   
   Iteration 2: lo=0, hi=1, mid=0
     arr[0]=1 < 3 → lo = mid = 0   (SAME STATE!)
   
   INFINITE LOOP! The search space never shrinks.
```

```
   Fix:
   ┌────────────────────────────────────────────────┐
   │  ALWAYS exclude mid from the next search:      │
   │                                                  │
   │  arr[mid] < target  →  lo = mid + 1  (not mid) │
   │  arr[mid] > target  →  hi = mid - 1  (not mid) │
   │                                                  │
   │  Why? We already checked mid — don't check again│
   └────────────────────────────────────────────────┘
```

---

## Gotcha 4: Returning Wrong Value

```
   WRONG:
   if (arr[mid] == target)
       return true;     // Loses position information!
   return false;

   BETTER:
   if (arr[mid] == target)
       return mid;      // Returns the INDEX
   return -1;           // -1 means "not found"
```

---

## Gotcha 5: Not Handling Duplicates

Standard binary search finds **any** occurrence of the target, not necessarily the first or last.

```
   Array: [1, 3, 3, 3, 3, 5, 7]
   Target: 3
   
   Standard binary search might return index 2, 3, or 4!
   The exact index depends on when mid lands on a 3.
   
   If you need the FIRST occurrence → use modified binary search (Unit 3)
   If you need the LAST occurrence  → use modified binary search (Unit 3)
```

---

## Gotcha 6: Searching in Empty Array

```java
   int[] arr = {};
   // n = 0, lo = 0, hi = -1
   // lo <= hi → 0 <= -1 → FALSE → loop never runs
   // return -1  ✓ (correctly handles empty array)
```

Always ensure your code handles `n = 0` gracefully.

---

## Gotcha 7: Floating-Point Binary Search

For continuous domains, use an **epsilon-based** termination condition:

```
   // Find sqrt(x) using binary search
   lo = 0, hi = x
   
   WRONG:  while (lo <= hi)          // Never terminates for floats!
   
   CORRECT: while (hi - lo > 1e-9)   // Stop when precision is sufficient
       mid = (lo + hi) / 2
       if (mid * mid < x)
           lo = mid                   // Note: NOT mid + 1 for continuous!
       else
           hi = mid
```

---

## Common Bugs Checklist

```
   ╔══════════════════════════════════════════════════════╗
   ║  BINARY SEARCH BUG CHECKLIST                        ║
   ║                                                      ║
   ║  □ Is the array sorted?                              ║
   ║  □ Using lo + (hi - lo) / 2  instead of (lo+hi)/2?  ║
   ║  □ Loop condition is lo <= hi  (not lo < hi)?        ║
   ║  □ Updating lo = mid + 1  (not lo = mid)?            ║
   ║  □ Updating hi = mid - 1  (not hi = mid)?            ║
   ║  □ Returning -1 for not found (not 0)?               ║
   ║  □ Handling empty array?                              ║
   ║  □ Handling single element?                           ║
   ║  □ Handling duplicates correctly?                     ║
   ╚══════════════════════════════════════════════════════╝
```

---

## Testing Strategy

| Test Case | Input | Expected |
|-----------|-------|----------|
| Empty array | `[], 5` | -1 |
| Single element (found) | `[5], 5` | 0 |
| Single element (not found) | `[5], 3` | -1 |
| Target at start | `[1,2,3], 1` | 0 |
| Target at end | `[1,2,3], 3` | 2 |
| Target in middle | `[1,2,3], 2` | 1 |
| Target not present (too small) | `[2,4,6], 1` | -1 |
| Target not present (too large) | `[2,4,6], 7` | -1 |
| Target not present (between) | `[2,4,6], 3` | -1 |
| Even-sized array | `[1,2,3,4], 3` | 2 |
| Odd-sized array | `[1,2,3,4,5], 3` | 2 |
| Large array with duplicates | `[1,1,1,1,1], 1` | any valid index |

---

## Summary Table

| Prerequisite / Gotcha | Impact |
|----------------------|--------|
| **Sorted data** | Incorrect results if not sorted |
| **Random access** | O(n log n) on linked lists |
| **Overflow in mid** | ArrayIndexOutOfBounds |
| **lo < hi vs lo <= hi** | Misses single-element checks |
| **lo = mid (no +1)** | Infinite loop |
| **Duplicates** | Returns arbitrary occurrence |
| **Empty array** | Must return -1 gracefully |
| **Floating point** | Use epsilon termination |

---

## Quick Revision Questions

1. **What was the famous bug in Java's `Arrays.binarySearch()` and how is it fixed?**
2. **Why does `while (lo < hi)` fail for a single-element array?**
3. **What causes an infinite loop in binary search? Give an example.**
4. **Can binary search be applied to a sorted linked list? What would be the complexity?**
5. **How should you modify binary search to work on a descending-sorted array?**
6. **Name five edge cases you should always test when implementing binary search.**

---

[← Previous: Complexity Analysis](05-complexity-analysis.md) | [Next: Unit 3 — Find First Occurrence →](../03-Binary-Search-Variations/01-find-first-occurrence.md)
