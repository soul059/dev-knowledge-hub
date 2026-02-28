# Chapter 4: Common Mistakes in Binary Search

[← Previous: Left / Right Bias](03-left-right-bias.md) | [Next: Template Comparison →](05-template-comparison.md)

---

## Overview

Binary search is notorious for **off-by-one errors** and subtle bugs. This chapter catalogs the most common mistakes with examples of what goes wrong and how to fix them.

---

## Mistake 1: Wrong Loop Condition

### Bug: `while (lo < hi)` in standard template

```java
// WRONG for exact match search
while (lo < hi) {             // Misses when target is the only remaining element
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```

```
   arr = [5], target = 5
   lo = 0, hi = 0
   lo < hi → 0 < 0 → FALSE → loop never runs → return -1 ✗

   FIX: while (lo <= hi) → 0 <= 0 → TRUE → checks arr[0] → FOUND ✓
```

### Rule

| Template | Use `<=` | Use `<` |
|----------|----------|---------|
| Standard (exact match) | ✓ | ✗ |
| Boundary finding | ✗ | ✓ |
| Peak element | ✗ | ✓ |

---

## Mistake 2: Integer Overflow

```java
// WRONG
int mid = (lo + hi) / 2;     // Overflow when lo + hi > Integer.MAX_VALUE

// CORRECT
int mid = lo + (hi - lo) / 2;
```

```
   lo = 2,000,000,000    hi = 2,100,000,000
   
   lo + hi = 4,100,000,000 > 2,147,483,647 → OVERFLOW → negative → crash!
   
   hi - lo = 100,000,000 → safe!
   lo + 50,000,000 = 2,050,000,000 → safe ✓
```

---

## Mistake 3: Infinite Loop

### Bug: `lo = mid` with left-biased mid

```java
// WRONG
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;    // Left-biased
    if (condition(mid))
        hi = mid;
    else
        lo = mid;                     // BUG! When lo + 1 == hi, mid = lo → no progress
}
```

```
   lo = 4, hi = 5:
   mid = 4 + (5-4)/2 = 4
   lo = mid = 4  ← SAME VALUE!
   
   Next: lo = 4, hi = 5, mid = 4 → lo = 4 → INFINITE LOOP!
   
   FIX: Either use lo = mid + 1, or use right-biased mid
```

---

## Mistake 4: Off-by-One in Updates

### Bug: Not excluding `mid`

```java
// WRONG
if (arr[mid] < target)
    lo = mid;         // Should be mid + 1
if (arr[mid] > target)
    hi = mid;         // Should be mid - 1
```

```
   We ALREADY checked arr[mid] — including it again is wasteful
   and can cause infinite loops.

   CORRECT:
   lo = mid + 1    // mid was checked and is too small
   hi = mid - 1    // mid was checked and is too large
```

**Exception:** In boundary template, `hi = mid` is correct because `mid` could BE the answer.

---

## Mistake 5: Wrong Return Value

### Bug: Returning 0 for not found

```java
// WRONG
if (arr[mid] == target) return mid;
return 0;                  // 0 is a VALID index! Ambiguous!

// CORRECT
return -1;                 // -1 is never a valid index → unambiguous
```

---

## Mistake 6: Forgetting Edge Cases

```
   ┌────────────────────────────────┬────────────────────────────────┐
   │     Edge Case                  │   What Goes Wrong              │
   ├────────────────────────────────┼────────────────────────────────┤
   │  Empty array (n = 0)          │  hi = -1, undefined behavior   │
   │  Single element               │  Must handle lo == hi          │
   │  Target before first element  │  lo ends at 0                  │
   │  Target after last element    │  hi ends at -1 or lo ends at n │
   │  All elements identical       │  First/last occurrence differ  │
   │  Two elements                 │  Tests update logic properly   │
   └────────────────────────────────┴────────────────────────────────┘
```

---

## Mistake 7: Applying BS to Unsorted Data

```
   arr = [3, 1, 4, 1, 5, 9, 2, 6]    target = 5
   
   Standard binary search:
   mid = 3, arr[3] = 1 < 5 → search right half [5, 9, 2, 6]
   mid = 5, arr[5] = 9 > 5 → search left half [5]
   mid = 4, arr[4] = 5 → FOUND ✓ (lucky!)
   
   But for target = 4:
   mid = 3, arr[3] = 1 < 4 → search right [5, 9, 2, 6]
   mid = 5, arr[5] = 9 > 4 → search right' [5]
   mid = 4, arr[4] = 5 > 4 → search left of right' → empty → NOT FOUND ✗
   
   4 IS in the array at index 2 — but BS missed it!
```

---

## Mistake 8: Not Thinking About Data Types

```java
// Searching on a function over large range
long lo = 0, hi = 1_000_000_000_000L;  // Use long, not int!

// For floating-point binary search
double lo = 0, hi = 1e18;
while (hi - lo > 1e-9) {              // Epsilon termination
    double mid = (lo + hi) / 2;        // OK for doubles (no overflow concern)
    // ...
}
```

---

## Debugging Checklist

```
   ╔══════════════════════════════════════════════════════════════╗
   ║  BINARY SEARCH DEBUGGING CHECKLIST                          ║
   ║                                                              ║
   ║  □ 1. Is the array sorted? (ascending or descending?)       ║
   ║  □ 2. Is mid calculation overflow-safe?                     ║
   ║  □ 3. Does the loop condition match the template?           ║
   ║  □ 4. Does mid change every iteration? (no infinite loop?)  ║
   ║  □ 5. Are lo/hi updates excluding checked elements?         ║
   ║  □ 6. Have I tested all edge cases?                         ║
   ║  □ 7. Is the return value correct for "not found"?          ║
   ║  □ 8. Am I using the right bias for the template?           ║
   ║  □ 9. Does the code work for array sizes 0, 1, and 2?      ║
   ╚══════════════════════════════════════════════════════════════╝
```

---

## Test These Arrays to Find Your Bugs

```java
int[][] tests = {
    {},           // Empty
    {5},          // Single element
    {1, 2},       // Two elements - target at start
    {1, 2},       // Two elements - target at end
    {1, 2},       // Two elements - target not present
    {1, 2, 3},    // Odd length
    {1, 2, 3, 4}, // Even length
    {3, 3, 3, 3}, // All duplicates
};
```

---

## Summary: Top Mistakes Ranked by Frequency

| Rank | Mistake | Consequence |
|------|---------|-------------|
| 1 | Infinite loop (`lo=mid` with left bias) | Program hangs |
| 2 | Wrong loop condition (`<` vs `<=`) | Misses elements |
| 3 | Integer overflow in mid | ArrayIndexOutOfBounds |
| 4 | Not handling empty/single array | NullPointer / wrong result |
| 5 | Searching unsorted data | Incorrect results |
| 6 | Wrong return for not found | Logic errors downstream |
| 7 | Off-by-one in updates | Wrong results or infinite loop |
| 8 | Wrong bias for the template | Infinite loop |

---

## Quick Revision Questions

1. **Why does `while (lo < hi)` fail for the standard template but work for boundary?**
2. **What's the fix for integer overflow in mid calculation?**
3. **Give an example where `lo = mid` causes an infinite loop.**
4. **Why should you return `-1` instead of `0` for "not found"?**
5. **List 4 edge case arrays you should always test binary search with.**
6. **What happens if you apply binary search to the unsorted array `[3, 1, 4]` looking for 4?**

---

[← Previous: Left / Right Bias](03-left-right-bias.md) | [Next: Template Comparison →](05-template-comparison.md)
