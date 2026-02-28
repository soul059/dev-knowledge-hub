# Chapter 4: Loop Invariant in Binary Search

[← Previous: Recursive Implementation](03-recursive-implementation.md) | [Next: Complexity Analysis →](05-complexity-analysis.md)

---

## Overview

A **loop invariant** is a condition that is **true before, during, and after** every iteration of a loop. Understanding the loop invariant of binary search helps prove its **correctness** and avoid subtle bugs.

---

## What is a Loop Invariant?

```
   ┌──────────────────────────────────────────┐
   │            LOOP INVARIANT                 │
   │                                           │
   │  A statement about the program state      │
   │  that is TRUE at these three points:      │
   │                                           │
   │  1. INITIALIZATION: True before loop      │
   │  2. MAINTENANCE: If true before an        │
   │     iteration, remains true after it      │
   │  3. TERMINATION: Gives useful result      │
   │     when loop ends                        │
   └──────────────────────────────────────────┘
```

Think of it like mathematical induction:
- **Base case** = Initialization
- **Inductive step** = Maintenance
- **Conclusion** = Termination

---

## Binary Search Loop Invariant

### The Invariant Statement

> **"If the target exists in the array, it must be in the range `arr[lo..hi]`."**

In other words: `target ∉ arr[0..lo-1]` and `target ∉ arr[hi+1..n-1]`.

```
   ┌─────────────────────────────────────────────────────────┐
   │ Already       │◄── Possible Range ──►│    Already       │
   │ Eliminated    │    arr[lo .. hi]      │    Eliminated    │
   │ (too small)   │    target MAY be here │    (too large)   │
   └─────────────────────────────────────────────────────────┘
     arr[0..lo-1]         ▲                  arr[hi+1..n-1]
                    INVARIANT: if target
                    exists, it's HERE
```

---

## Proving the Invariant

### 1. Initialization (Before Loop Starts)

```
   lo = 0, hi = n - 1

   ┌────┬────┬────┬────┬────┬────┬────┬────┐
   │    │    │    │    │    │    │    │    │
   └────┴────┴────┴────┴────┴────┴────┴────┘
    lo                                    hi

   Range [lo..hi] = [0..n-1] = ENTIRE ARRAY
   
   Invariant: "If target exists, it's in arr[0..n-1]"
   This is trivially TRUE — we haven't excluded anything!  ✓
```

---

### 2. Maintenance (During Each Iteration)

**Assume** the invariant holds at the start of an iteration: target is in `arr[lo..hi]`.

**Case A: `arr[mid] == target`**
```
   Found the target → return mid
   (Loop ends, invariant need not hold anymore)
```

**Case B: `arr[mid] < target`**
```
   arr[mid] is LESS than target
   Since array is sorted: arr[lo], arr[lo+1], ..., arr[mid] are ALL ≤ arr[mid] < target
   
   ┌──────────────────┬──────┬──────────────────┐
   │  ALL < target    │ mid  │  Target possibly  │
   │  ELIMINATE these │      │  here             │
   └──────────────────┴──────┴──────────────────┘
                              ▲
   Set lo = mid + 1          NEW lo
   
   Invariant maintained: target is in arr[mid+1..hi]  ✓
```

**Case C: `arr[mid] > target`**
```
   arr[mid] is GREATER than target
   Since array is sorted: arr[mid], arr[mid+1], ..., arr[hi] are ALL ≥ arr[mid] > target
   
   ┌──────────────────┬──────┬──────────────────┐
   │  Target possibly │ mid  │  ALL > target    │
   │  here            │      │  ELIMINATE these │
   └──────────────────┴──────┴──────────────────┘
                  ▲
               NEW hi
   Set hi = mid - 1
   
   Invariant maintained: target is in arr[lo..mid-1]  ✓
```

---

### 3. Termination (When Loop Ends)

The loop ends when `lo > hi`:

```
   lo > hi → the range [lo..hi] is EMPTY

   ┌──────────────────┬┬──────────────────┐
   │  Eliminated      ││   Eliminated     │
   │  (all < target)  ││   (all > target) │
   └──────────────────┴┴──────────────────┘
                       ^^
                    hi  lo
                    (hi < lo → empty range)

   By the invariant: "If target exists, it's in arr[lo..hi]"
   But arr[lo..hi] is empty!
   
   Therefore: target does NOT exist in the array.
   → Correctly return -1  ✓
```

---

## Visual Proof — Full Example

### Array: `[3, 7, 11, 15, 19, 23]`, Target: `15`

```
   INITIALIZATION:
   ┌───┬───┬────┬────┬────┬────┐
   │ 3 │ 7 │ 11 │ 15 │ 19 │ 23 │    Invariant: target in [0..5] ✓
   └───┴───┴────┴────┴────┴────┘
    lo                          hi

   ITERATION 1: mid=2, arr[2]=11 < 15 → lo=3
   ┌───┬───┬────┬────┬────┬────┐
   │ 3 │ 7 │ 11 │ 15 │ 19 │ 23 │    Invariant: target in [3..5] ✓
   └───┴───┴────┴────┴────┴────┘
    eliminated    lo         hi       (3, 7, 11 are all < 15, correctly removed)

   ITERATION 2: mid=4, arr[4]=19 > 15 → hi=3
   ┌───┬───┬────┬────┬────┬────┐
   │ 3 │ 7 │ 11 │ 15 │ 19 │ 23 │    Invariant: target in [3..3] ✓
   └───┴───┴────┴────┴────┴────┘
    eliminated   lo=hi  eliminated   (19, 23 are all > 15, correctly removed)

   ITERATION 3: mid=3, arr[3]=15 == 15 → FOUND!
   Return 3  ✓
```

---

## Why Loop Invariants Matter

### Bug Detection

Wrong code: `hi = mid` instead of `hi = mid - 1`

```
   Array: [1, 3], target = 1

   Iteration 1: lo=0, hi=1, mid=0, arr[0]=1 → FOUND ✓ (lucky)

   BUT with target = 2:
   Iteration 1: lo=0, hi=1, mid=0, arr[0]=1 < 2 → lo=1
   Iteration 2: lo=1, hi=1, mid=1, arr[1]=3 > 2 → hi=mid=1  (BUG!)
   Iteration 3: lo=1, hi=1, mid=1, arr[1]=3 > 2 → hi=mid=1  (INFINITE LOOP!)
   
   With hi = mid - 1:
   Iteration 2: ... → hi = 0
   Iteration 3: lo=1 > hi=0 → return -1  ✓
```

### Correctness Argument

```
   ┌──────────────────────────────────────────────────┐
   │  To prove binary search is correct, we need:     │
   │                                                    │
   │  1. Invariant is true initially           ✓       │
   │  2. Each iteration preserves it           ✓       │
   │  3. When loop ends, we get correct result ✓       │
   │                                                    │
   │  Therefore: binary search is CORRECT.             │
   └──────────────────────────────────────────────────┘
```

---

## Loop Invariant for Variations

Different binary search variations have different invariants:

| Variation | Loop Invariant |
|-----------|---------------|
| Standard | Target (if exists) is in `arr[lo..hi]` |
| Find First | First occurrence is in `arr[lo..hi]` |
| Find Last | Last occurrence is in `arr[lo..hi]` |
| Lower Bound | `arr[lo-1] < target ≤ arr[hi+1]` |
| Upper Bound | `arr[lo-1] ≤ target < arr[hi+1]` |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Invariant** | If target exists, it's in `arr[lo..hi]` |
| **Initialization** | `lo=0, hi=n-1` covers entire array |
| **Maintenance** | Each case correctly shrinks the range |
| **Termination** | `lo > hi` means empty range → not found |
| **Purpose** | Proves correctness, helps debug |
| **Key Insight** | Sorted property guarantees safe elimination |

---

## Quick Revision Questions

1. **State the loop invariant for standard binary search in one sentence.**
2. **What are the three parts of proving a loop invariant?**
3. **Why can we safely set `lo = mid + 1` when `arr[mid] < target`?**
4. **What does the invariant tell us when the loop terminates with `lo > hi`?**
5. **How does understanding the loop invariant help you debug binary search code?**
6. **If we used `hi = mid` instead of `hi = mid - 1`, does the invariant still hold? What goes wrong?**

---

[← Previous: Recursive Implementation](03-recursive-implementation.md) | [Next: Complexity Analysis →](05-complexity-analysis.md)
