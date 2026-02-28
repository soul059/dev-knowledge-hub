# Chapter 2: Variable Size Window

[← Previous: Fixed Size Window](01-fixed-size-window.md) | [Back to README](../README.md) | [Next: Max Sum Subarray (Fixed) →](03-max-sum-subarray-fixed.md)

---

## Overview

Unlike fixed-size windows, **variable-size (dynamic) windows** expand and contract based on a condition. The window grows to include more elements and shrinks when a constraint is violated. This technique solves problems where the subarray/substring size is NOT given—you must find the optimal size.

---

## Fixed vs Variable Window

```
  FIXED WINDOW (k=3):
  ┌───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 2 │ 8 │ 7 │ 4 │
  └───┴───┴───┴───┴───┴───┴───┘
  [─────────]                    always 3 elements
      [─────────]                always 3 elements
          [─────────]            always 3 elements

  VARIABLE WINDOW:
  ┌───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 2 │ 8 │ 7 │ 4 │
  └───┴───┴───┴───┴───┴───┴───┘
  [───]                          2 elements (satisfies condition)
  [───────────────]              5 elements (still satisfies)
      [─────────]                3 elements (after shrinking)
```

---

## Two Pointers: left and right

```
  left                right
    │                   │
    ▼                   ▼
  ┌───┬───┬───┬───┬───┬───┬───┐
  │ a │ b │ c │ d │ e │ f │ g │
  └───┴───┴───┴───┴───┴───┴───┘
    └──── window ────┘

  • right EXPANDS the window (include more)
  • left CONTRACTS the window (exclude old)
  • Window = arr[left..right]
  • Window size = right - left + 1
```

---

## Template: Variable-Size Window (Find Longest)

```
FUNCTION longestWindow(arr, n, condition):
    left = 0
    windowState = initial     // sum, count, freq map, etc.
    bestLength = 0

    FOR right = 0 TO n-1:
        // EXPAND: add arr[right] to window
        UPDATE windowState WITH arr[right]

        // CONTRACT: shrink until valid
        WHILE NOT condition(windowState):
            REMOVE arr[left] FROM windowState
            left = left + 1

        // UPDATE ANSWER (window is valid here)
        bestLength = MAX(bestLength, right - left + 1)

    RETURN bestLength
```

---

## Template: Variable-Size Window (Find Shortest)

```
FUNCTION shortestWindow(arr, n, condition):
    left = 0
    windowState = initial
    bestLength = INFINITY

    FOR right = 0 TO n-1:
        // EXPAND: add arr[right] to window
        UPDATE windowState WITH arr[right]

        // CONTRACT: shrink while still valid
        WHILE condition(windowState):
            bestLength = MIN(bestLength, right - left + 1)
            REMOVE arr[left] FROM windowState
            left = left + 1

    RETURN bestLength (or -1 if INFINITY)
```

### Key Difference

```
  ┌────────────────────────────────────────────────────┐
  │  FIND LONGEST: Shrink while INVALID → answer after │
  │  FIND SHORTEST: Shrink while VALID → answer inside │
  └────────────────────────────────────────────────────┘
```

---

## Detailed Trace: Longest Subarray with Sum ≤ Target

Find the longest subarray whose sum ≤ 7.

```
  arr = [3, 1, 2, 7, 4, 2, 1, 1, 5]   target = 7

  right=0: add 3, sum=3, valid         → best = max(0, 0-0+1) = 1
           window: [3]

  right=1: add 1, sum=4, valid         → best = max(1, 1-0+1) = 2
           window: [3, 1]

  right=2: add 2, sum=6, valid         → best = max(2, 2-0+1) = 3
           window: [3, 1, 2]

  right=3: add 7, sum=13, INVALID
           shrink: remove 3, sum=10, left=1, INVALID
           shrink: remove 1, sum=9, left=2, INVALID
           shrink: remove 2, sum=7, left=3, valid → best = max(3, 3-3+1) = 3
           window: [7]

  right=4: add 4, sum=11, INVALID
           shrink: remove 7, sum=4, left=4, valid → best = max(3, 4-4+1) = 3
           window: [4]

  right=5: add 2, sum=6, valid         → best = max(3, 5-4+1) = 3
           window: [4, 2]

  right=6: add 1, sum=7, valid         → best = max(3, 6-4+1) = 3
           window: [4, 2, 1]

  right=7: add 1, sum=8, INVALID
           shrink: remove 4, sum=4, left=5, valid → best = max(3, 7-5+1) = 3
           window: [2, 1, 1]

  right=8: add 5, sum=9, INVALID
           shrink: remove 2, sum=7, left=6, valid → best = max(3, 8-6+1) = 3
           window: [1, 1, 5]

  Answer: 3  ✓
```

---

## Why Is It O(n)?

```
  "But there's a while loop inside the for loop—isn't it O(n²)?"

  NO! Each element is added at most once and removed at most once.

  right pointer: moves from 0 to n-1          → n moves
  left pointer:  moves from 0 to at most n-1  → at most n moves
                                                ─────────────────
  Total operations:                            ≤ 2n = O(n)

  Amortized Analysis:
  ┌──────────────────────────────────┐
  │                                  │
  │    right ─────────────────→      │
  │    left  ─────────────────→      │
  │                                  │
  │    Both pointers only move       │
  │    forward. Total movement       │
  │    is at most 2n.                │
  │                                  │
  └──────────────────────────────────┘
```

---

## Common Variable Window Patterns

```
  ┌─────────────────────┬──────────────────────────────────────┐
  │ Pattern             │ Examples                             │
  ├─────────────────────┼──────────────────────────────────────┤
  │ Longest subarray    │ Sum ≤ k, at most k distinct chars    │
  │ with property       │ at most k zeros (after flip)         │
  ├─────────────────────┼──────────────────────────────────────┤
  │ Shortest subarray   │ Sum ≥ target, containing all chars   │
  │ with property       │ of pattern                           │
  ├─────────────────────┼──────────────────────────────────────┤
  │ Count subarrays     │ Subarrays with sum = k               │
  │ with property       │ (use "at most k" trick)              │
  └─────────────────────┴──────────────────────────────────────┘
```

---

## Example: Smallest Subarray with Sum ≥ S

```
FUNCTION minSubArrayLen(arr, n, S):
    left = 0
    windowSum = 0
    bestLen = INFINITY

    FOR right = 0 TO n-1:
        windowSum = windowSum + arr[right]

        WHILE windowSum >= S:
            bestLen = MIN(bestLen, right - left + 1)
            windowSum = windowSum - arr[left]
            left = left + 1

    IF bestLen == INFINITY:
        RETURN 0
    RETURN bestLen
```

### Trace: arr = [2, 3, 1, 2, 4, 3], S = 7

```
  r=0: sum=2                                    best=∞
  r=1: sum=5                                    best=∞
  r=2: sum=6                                    best=∞
  r=3: sum=8 ≥ 7 → best=min(∞,4)=4, rm 2, sum=6, l=1   best=4
  r=4: sum=10 ≥ 7 → best=min(4,4)=4, rm 3, sum=7, l=2
                     ≥ 7 → best=min(4,3)=3, rm 1, sum=6, l=3   best=3
  r=5: sum=9 ≥ 7 → best=min(3,3)=3, rm 2, sum=7, l=4
                    ≥ 7 → best=min(3,2)=2, rm 4, sum=3, l=5    best=2

  Answer: 2  (subarray [4,3])  ✓
```

---

## Conditions That Work with Variable Window

The condition must be **monotonic**: if a window is invalid, making it larger won't make it valid (for "longest" problems), OR if a window is valid, making it smaller won't break it in a non-recoverable way.

```
  ✓ Sum ≤ k         (adding elements can only increase sum)
  ✓ At most k distinct (adding can only increase distinct count)
  ✓ Sum ≥ k         (removing elements can only decrease sum)
  ✓ Contains all chars of pattern (removing can break it)

  ✗ Sum == k exactly  (not monotonic — can't simply shrink/grow)
     → Use hashmap approach or "at most k" − "at most k-1" trick
```

---

## The "At Most K" Trick for "Exactly K"

```
  count(exactly k distinct) = count(at most k distinct) 
                             − count(at most k-1 distinct)

  Example: Subarrays with exactly 2 distinct characters
  = subarrays with at most 2 distinct
  − subarrays with at most 1 distinct
```

---

## Decision Flowchart

```
  Start with problem
        │
        ▼
  Is subarray size fixed?
       / \
     YES   NO
      │     │
      ▼     ▼
   Fixed   Variable
   Window  Window
      │     │
      ▼     ▼
   Use k   What to find?
             /        \
          Longest    Shortest
            │           │
            ▼           ▼
        Shrink       Shrink
        while        while
        INVALID      VALID
```

---

## Summary Table

| Aspect | Fixed Window | Variable Window |
|--------|-------------|-----------------|
| Window size | Constant k | Changes dynamically |
| Right pointer | Advances one at a time | Advances one at a time |
| Left pointer | Advances one at a time | May skip multiple positions |
| Condition | N/A | Defines valid/invalid window |
| Core operation | Add new, remove old | Expand right, contract left |
| Time complexity | O(n) | O(n) amortized |
| Space complexity | O(1) to O(k) | O(1) to O(n) |
| Use case | Given k | Find optimal k |

---

## Quick Revision Questions

1. **How does a variable-size window differ from a fixed-size window?** What governs the window size?

2. **What is the amortized time complexity** of the variable window technique, and why is it O(n) despite nested loops?

3. **What is the difference in the shrinking condition** between "find longest" and "find shortest" problems?

4. **Why does Sum == k (exact) not work** directly with sliding window? What trick can you use?

5. **What property must the condition have** for the sliding window technique to work correctly?

6. **In the template, which pointer only moves forward one step at a time?** Which pointer can jump multiple steps?

---

[← Previous: Fixed Size Window](01-fixed-size-window.md) | [Back to README](../README.md) | [Next: Max Sum Subarray (Fixed) →](03-max-sum-subarray-fixed.md)
