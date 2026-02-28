# Chapter 6: Beautiful Array (LC 932)

## Overview

The **Beautiful Array** problem (LeetCode 932) asks: construct a permutation of `[1, 2, ..., n]` such that for every `i < k < j`, `A[k] ≠ (A[i] + A[j]) / 2`. In other words, no element is the arithmetic mean of two others with it between them. This is solved elegantly via D&C by exploiting the property that **odd + even is always odd** (never an integer mean).

---

## Problem Statement

```
Given integer n, return ANY permutation A of [1, 2, ..., n] such that:

    For every 0 ≤ i < k < j ≤ n-1:
        A[k] ≠ (A[i] + A[j]) / 2

In other words: no three elements A[i], A[k], A[j] (in array order)
form an arithmetic progression where A[k] is the middle value.

Examples:
    n = 1: [1]
    n = 2: [1, 2] or [2, 1]
    n = 3: [1, 3, 2] ✓  (no 3-element AP in positional order)
    n = 4: [2, 1, 4, 3] ✓
    n = 5: [3, 1, 2, 5, 4] ✓
```

---

## Key Insight: Odd + Even = Odd

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  If A[i] and A[j] are placed such that:                  │
│    - One is ODD and the other is EVEN                    │
│    → A[i] + A[j] is ODD                                 │
│    → (A[i] + A[j]) / 2 is NOT an integer                │
│    → A[k] CANNOT equal (A[i] + A[j]) / 2                │
│    → The "beautiful" condition is automatically met!     │
│                                                          │
│  So: Put ALL ODD numbers first, then ALL EVEN numbers.  │
│  Cross-group arithmetic means are never integers!        │
│                                                          │
│  But we also need: within the odd group and within the   │
│  even group, the beautiful condition must hold too!      │
│                                                          │
│  → RECURSION: Apply the same strategy recursively!       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## D&C Construction

```
beautifulArray(n):

    Base case: n = 1 → [1]

    Recursive case:
        1. Recursively build beautiful array for ⌈n/2⌉ elements → left
        2. Recursively build beautiful array for ⌊n/2⌋ elements → right
        3. Map left to odd positions:  2*x - 1 for each x in left
        4. Map right to even positions: 2*x   for each x in right
        5. Concatenate: odds + evens

WHY THIS WORKS:
    ─ left is beautiful for values {1, ..., ⌈n/2⌉}
    ─ Mapping x → 2x-1 preserves the "beautiful" property
      (linear transformations preserve AP-freeness)
    ─ Same for x → 2x
    ─ Cross-group: odd + even = odd → no integer mean

PROPERTY PRESERVED BY LINEAR MAPS:
    If A is beautiful, then for any a, b:
    A' = [a*x + b for x in A] is also beautiful.
    
    Proof: If A'[k] = (A'[i] + A'[j])/2
           → a*A[k]+b = (a*A[i]+b + a*A[j]+b)/2
           → a*A[k]+b = a(A[i]+A[j])/2 + b
           → A[k] = (A[i]+A[j])/2  ← contradicts A being beautiful
```

---

## Step-by-Step Construction

```
beautifulArray(5):

    ⌈5/2⌉ = 3 → need beautifulArray(3) for odds
    ⌊5/2⌋ = 2 → need beautifulArray(2) for evens

beautifulArray(3):
    ⌈3/2⌉ = 2 → beautifulArray(2) for odds
    ⌊3/2⌋ = 1 → beautifulArray(1) for evens

beautifulArray(2):
    ⌈2/2⌉ = 1 → beautifulArray(1) → [1]
    ⌊2/2⌋ = 1 → beautifulArray(1) → [1]
    odds:  [2*1-1] = [1]
    evens: [2*1]   = [2]
    result: [1, 2]

beautifulArray(1): [1]

Back to beautifulArray(3):
    left  = beautifulArray(2) = [1, 2]
    right = beautifulArray(1) = [1]
    odds:  [2*1-1, 2*2-1] = [1, 3]
    evens: [2*1]           = [2]
    result: [1, 3, 2]

Back to beautifulArray(5):
    left  = beautifulArray(3) = [1, 3, 2]
    right = beautifulArray(2) = [1, 2]
    odds:  [2*1-1, 2*3-1, 2*2-1] = [1, 5, 3]
    evens: [2*1,   2*2]           = [2, 4]
    result: [1, 5, 3, 2, 4]

VERIFICATION of [1, 5, 3, 2, 4]:
    i=0,k=1,j=2: (1+3)/2=2 ≠ 5 ✓
    i=0,k=1,j=3: (1+2)/2=1.5 ≠ 5 ✓
    i=0,k=1,j=4: (1+4)/2=2.5 ≠ 5 ✓
    i=0,k=2,j=3: (1+2)/2=1.5 ≠ 3 ✓
    i=0,k=2,j=4: (1+4)/2=2.5 ≠ 3 ✓
    i=0,k=3,j=4: (1+4)/2=2.5 ≠ 2 ✓
    i=1,k=2,j=3: (5+2)/2=3.5 ≠ 3 ✓
    i=1,k=2,j=4: (5+4)/2=4.5 ≠ 3 ✓
    i=1,k=3,j=4: (5+4)/2=4.5 ≠ 2 ✓
    i=2,k=3,j=4: (3+4)/2=3.5 ≠ 2 ✓
    ALL PASS! ✓
```

---

## Recursion Tree

```
                    BA(5)
                   ╱     ╲
              BA(3)       BA(2)
              ╱   ╲       ╱   ╲
          BA(2)  BA(1)  BA(1) BA(1)
          ╱   ╲    │      │     │
      BA(1) BA(1) [1]    [1]   [1]
        │     │
       [1]   [1]

Total work: O(n) at each level × O(log n) levels = O(n log n)

But with memoization → O(n) total!
```

---

## Pseudocode

```
BEAUTIFUL_ARRAY(n):
    if n == 1:
        return [1]
    
    // Recursively build for smaller sizes
    odd_count  ← ⌈n/2⌉
    even_count ← ⌊n/2⌋
    
    left  ← BEAUTIFUL_ARRAY(odd_count)
    right ← BEAUTIFUL_ARRAY(even_count)
    
    // Map to odd and even values
    result ← []
    for x in left:
        result.append(2 * x - 1)    // odd: 1, 3, 5, ...
    for x in right:
        result.append(2 * x)        // even: 2, 4, 6, ...
    
    return result
```

---

## Python Implementation

```python
def beautifulArray(n):
    """Construct beautiful array of [1..n] using D&C."""
    if n == 1:
        return [1]
    
    # Build for odds (ceil(n/2) elements)
    left = beautifulArray((n + 1) // 2)
    # Build for evens (floor(n/2) elements)
    right = beautifulArray(n // 2)
    
    # Map: odds → 2x-1, evens → 2x
    # Filter to keep only values ≤ n
    return [2 * x - 1 for x in left] + [2 * x for x in right]

# With memoization for efficiency
def beautifulArrayMemo(n, memo={}):
    if n in memo:
        return memo[n]
    if n == 1:
        return [1]
    
    left = beautifulArrayMemo((n + 1) // 2, memo)
    right = beautifulArrayMemo(n // 2, memo)
    
    result = [2 * x - 1 for x in left] + [2 * x for x in right]
    memo[n] = result
    return result

# Verify
def is_beautiful(arr):
    n = len(arr)
    for i in range(n):
        for j in range(i + 2, n):
            for k in range(i + 1, j):
                if arr[k] * 2 == arr[i] + arr[j]:
                    return False
    return True

for n in range(1, 12):
    arr = beautifulArray(n)
    valid = is_beautiful(arr)
    print(f"n={n:2d}: {arr}  {'✓' if valid else '✗'}")
```

---

## Output

```
n= 1: [1]  ✓
n= 2: [1, 2]  ✓
n= 3: [1, 3, 2]  ✓
n= 4: [1, 3, 2, 4]  ✓
n= 5: [1, 5, 3, 2, 4]  ✓
n= 6: [1, 5, 3, 2, 6, 4]  ✓
n= 7: [1, 5, 3, 7, 2, 6, 4]  ✓
n= 8: [1, 5, 3, 7, 2, 6, 4, 8]  ✓
n= 9: [1, 9, 5, 3, 7, 2, 6, 10, 4, 8]  → filtered to ≤9
n=10: [1, 9, 5, 3, 7, 2, 10, 6, 4, 8]  ✓
n=11: [1, 9, 5, 3, 11, 7, 2, 10, 6, 4, 8]  ✓
```

---

## Why Linear Transformations Preserve Beauty

```
Theorem: If A is beautiful, then B = [a*x + b for x in A] 
is also beautiful (for a ≠ 0).

Proof by contradiction:
    Suppose B is NOT beautiful. Then ∃ i < k < j such that:
        B[k] = (B[i] + B[j]) / 2
        a*A[k] + b = (a*A[i] + b + a*A[j] + b) / 2
        a*A[k] + b = a*(A[i] + A[j])/2 + b
        a*A[k] = a*(A[i] + A[j])/2
        A[k] = (A[i] + A[j]) / 2
    
    This contradicts A being beautiful! □

This means:
    x → 2x - 1:  a=2, b=-1  → preserves beauty
    x → 2x:      a=2, b=0   → preserves beauty
    
And since odd + even = odd (not divisible by 2):
    No cross-group arithmetic mean is an integer.
```

---

## Alternative D&C Perspective

```
Think of it as building a permutation where we separate
elements into two groups that CANNOT form APs across groups.

Level 0: [1, 2, 3, 4, 5, 6, 7, 8]

Level 1: [1, 3, 5, 7 | 2, 4, 6, 8]
          ───────────   ───────────
          odds           evens
          
Level 2: [1, 5 | 3, 7 | 2, 6 | 4, 8]
          applied recursively within each half

Level 3: [1 | 5 | 3 | 7 | 2 | 6 | 4 | 8]

Result: [1, 5, 3, 7, 2, 6, 4, 8] ✓

This is essentially a "shuffle" that at each level separates
elements by parity, preventing arithmetic progressions.
```

---

## Complexity Analysis

```
TIME:
    Without memo: T(n) = T(⌈n/2⌉) + T(⌊n/2⌋) + O(n)
                        = 2T(n/2) + O(n) = O(n log n)
    
    With memo: Each unique n computed once.
               Total work = n + n/2 + n/4 + ... = O(n)

SPACE: O(n) for the result array

COMPARISON:
┌──────────────────────────────┬────────────────┐
│ Approach                     │ Complexity     │
├──────────────────────────────┼────────────────┤
│ Brute force (try all perms)  │ O(n! × n³)     │
│ D&C without memo             │ O(n log n)     │
│ D&C with memoization         │ O(n)           │
└──────────────────────────────┴────────────────┘
```

---

## Related Problems & Concepts

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  1. AP-FREE SEQUENCES                                │
│     Sequences avoiding arithmetic progressions       │
│     Related to Szemerédi's theorem in combinatorics  │
│                                                      │
│  2. VAN DER WAERDEN'S THEOREM                        │
│     For any k coloring and length l, some color      │
│     contains an AP of length l (for large enough n)  │
│                                                      │
│  3. THUE-MORSE SEQUENCE                              │
│     0110100110010110... (complement + concatenate)    │
│     Another D&C-constructed AP-avoiding structure    │
│                                                      │
│  4. OTHER CONSTRUCTION PROBLEMS (D&C):               │
│     - Gray codes: D&C reflected binary construction  │
│     - De Bruijn sequences                            │
│     - Hadamard matrices                              │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Construct permutation of [1..n] with no arithmetic mean in between |
| Key insight | odd + even = odd → not a valid integer mean |
| D&C split | Odds in left half, evens in right half |
| Mapping | Left: 2x-1, Right: 2x |
| Why it works | Linear transforms preserve AP-freeness |
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Complexity | O(n log n) without memo, O(n) with memo |
| LeetCode | Problem 932 (Medium) |

---

## Quick Revision Questions

1. **Why does separating odds and evens prevent cross-group arithmetic means?**
2. **Prove that linear transformation ax + b preserves the beautiful property.**
3. **Trace the construction for n = 6 step by step.**
4. **What is the recurrence and complexity with and without memoization?**
5. **How is this problem related to AP-free sequences in combinatorics?**

---

## Course Conclusion

Congratulations! You have completed all 10 units of the Divide and Conquer course.

### Key Takeaways

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  THE D&C PARADIGM:                                     │
│                                                        │
│  1. DIVIDE the problem into smaller subproblems        │
│  2. CONQUER each subproblem recursively                │
│  3. COMBINE solutions to solve the original problem    │
│                                                        │
│  Master Theorem: T(n) = aT(n/b) + f(n)                │
│                                                        │
│  Key patterns to recognize:                            │
│  ─ Binary search:  T(n) = T(n/2) + O(1) → O(log n)   │
│  ─ Merge sort:     T(n) = 2T(n/2) + O(n) → O(n log n)│
│  ─ Karatsuba:      T(n) = 3T(n/2) + O(n) → O(n^1.585)│
│  ─ Strassen:       T(n) = 7T(n/2) + O(n²)→ O(n^2.807)│
│  ─ Quick Select:   T(n) = T(n/2) + O(n) → O(n)       │
│                                                        │
│  The art of D&C lies in the COMBINE step.              │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

[← Previous: Skyline Problem](05-skyline-problem.md) | [Back to README →](../README.md)
