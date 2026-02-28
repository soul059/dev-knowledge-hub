# Chapter 4: Selection Sort — Stability Consideration

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: Comparison with Bubble Sort →](05-comparison-with-bubble.md)

---

## Overview

Selection Sort is **unstable** in its standard implementation. This chapter explains why, demonstrates the instability with examples, and shows how to make it stable (at a cost).

---

## Why Selection Sort Is Unstable

The instability comes from the **long-range swap** operation. When we swap the minimum with the element at position `i`, we may jump over equal elements, disrupting their relative order.

### Concrete Example

```
  Array: [5a, 3, 5b, 2, 1]
  (5a and 5b are both value 5, originally 5a comes first)
  
  ═══ Pass 0 ═══
  Find minimum in entire array → min = 1 at index 4
  Swap arr[0] (5a) with arr[4] (1):
  
  Before: [5a, 3, 5b, 2, 1]
           ↑              ↑
           swap these
  
  After:  [1, 3, 5b, 2, 5a]
                        ↑   ↑
                    Now 5b is BEFORE 5a!
  
  Original order: 5a before 5b
  After pass 0:   5b before 5a   ← ORDER REVERSED! UNSTABLE!
  
  Even though 5a and 5b have the same value,
  their relative order has changed.
```

### Why Does This Happen?

```
  The SWAP operation moves 5a far to the right,
  jumping over 5b:
  
  [5a, 3, 5b, 2, 1]
   ╲                 ╱     5a and 1 swap — 5a jumps over 5b
    ╲               ╱
  [1,  3, 5b, 2, 5a]      5a is now AFTER 5b
  
  ┌──────────────────────────────────────────────────────────┐
  │  KEY INSIGHT: Any algorithm that performs LONG-RANGE     │
  │  swaps (swapping non-adjacent elements) risks breaking  │
  │  stability.                                             │
  └──────────────────────────────────────────────────────────┘
```

### Another Example

```
  Array: [4a, 4b, 1, 4c]
  
  Pass 0: min=1 at index 2, swap with index 0 (4a)
  [4a, 4b, 1, 4c] → [1, 4b, 4a, 4c]
  
  Original order of 4s: 4a, 4b, 4c
  New order of 4s:      4b, 4a, 4c  ← 4a and 4b swapped! UNSTABLE!
```

---

## Making Selection Sort Stable

Instead of swapping, we can **shift** elements (like insertion) to maintain stability:

### Stable Selection Sort

```
  Instead of SWAP, use INSERT:
  
  1. Find minimum in unsorted portion
  2. REMOVE it from its position
  3. SHIFT all elements between i and minIndex right by 1
  4. INSERT minimum at position i
  
  Array: [5a, 3, 5b, 2, 1]
  
  Pass 0: min=1 at index 4
  
  Standard (SWAP):
  [5a, 3, 5b, 2, 1] → [1, 3, 5b, 2, 5a]   ← 5a jumps past 5b → UNSTABLE
  
  Stable (SHIFT + INSERT):
  Save min: temp = 1
  Shift right: [_, 5a, 3, 5b, 2]
  Insert: [1, 5a, 3, 5b, 2]                  ← 5a still before 5b → STABLE!
```

### Implementation

```python
def stable_selection_sort(arr):
    n = len(arr)
    
    for i in range(n - 1):
        # Find minimum in unsorted portion
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        
        # Instead of swap: shift elements and insert minimum
        if min_idx != i:
            min_val = arr[min_idx]
            # Shift elements from i to min_idx-1 one position right
            for k in range(min_idx, i, -1):
                arr[k] = arr[k - 1]
            arr[i] = min_val
```

### Trace of Stable Version

```
  Array: [5a, 3, 5b, 2, 1]
  
  Pass 0: min=1 at index 4
  Save: temp = 1
  Shift: [5a,3,5b,2,_] → [_,5a,3,5b,2] (shift all right)
  Insert: [1,5a,3,5b,2]
  Order of 5s: 5a before 5b ✓ STABLE!
  
  Pass 1: min=2 at index 4
  Save: temp = 2
  Shift: [1,_,5a,3,5b]
  Insert: [1,2,5a,3,5b]
  
  Pass 2: min=3 at index 3
  Save: temp = 3
  Shift: [1,2,_,5a,5b]
  Insert: [1,2,3,5a,5b]
  
  Pass 3: 5a ≤ 5b → no change needed
  
  Result: [1,2,3,5a,5b] ✓ STABLE!
```

---

## Cost of Stability

```
  ┌───────────────────────────────────────────────────────────┐
  │              Standard (Unstable)  │  Stable Version      │
  ├───────────────────────────────────┼──────────────────────┤
  │  Swap operation: O(1)            │  Shift operation: O(n)│
  │  Total swaps: O(n)               │  Total shifts: O(n²) │
  │  Total writes: O(n)              │  Total writes: O(n²) │
  │  Comparisons: O(n²)             │  Comparisons: O(n²)  │
  └───────────────────────────────────┴──────────────────────┘
  
  Making Selection Sort stable REMOVES its main advantage
  (low number of writes). The stable version has O(n²) writes,
  making it similar to Insertion Sort — use Insertion Sort instead!
```

---

## Practical Advice

```
  Need stable O(n²) sort?
       │
       └── Use INSERTION SORT instead!
           It's naturally stable AND adaptive.
  
  Need minimal writes?
       │
       └── Use standard (unstable) SELECTION SORT.
           O(n) swaps can't be beaten.
  
  Need stable AND efficient?
       │
       └── Use MERGE SORT — O(n log n) and stable.
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Standard Selection Sort | Unstable |
| Cause of instability | Long-range swap jumps over equal elements |
| Making it stable | Replace swap with shift + insert |
| Cost of stability | O(n) swaps → O(n²) shifts |
| Better stable alternative | Insertion Sort (naturally stable + adaptive) |
| Key trade-off | Stability vs minimal writes |

---

## Quick Revision Questions

1. **Why is Selection Sort unstable? Show with an example.**
2. **What operation causes instability in Selection Sort?**
3. **How can Selection Sort be made stable?**
4. **What is the cost of making Selection Sort stable?**
5. **Why is Insertion Sort preferred over stable Selection Sort?**
6. **Is Bubble Sort stable? Why is its approach different from Selection Sort?**

---

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: Comparison with Bubble Sort →](05-comparison-with-bubble.md)
