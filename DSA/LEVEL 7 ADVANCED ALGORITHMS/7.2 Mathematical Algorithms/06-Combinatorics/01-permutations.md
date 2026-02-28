# Chapter 1: Permutations

[← Previous Unit: Applications](../05-Fast-Exponentiation/06-applications.md) | [Back to README](../README.md) | [Next: Combinations →](02-combinations.md)

---

## Overview

A **permutation** is an ordered arrangement of elements. This chapter covers counting permutations, generating them, and their applications in competitive programming.

---

## 1.1 Definition

```
  A permutation of n elements taken r at a time:
  
  P(n, r) = n! / (n - r)!

  "Choose r items from n and ARRANGE them in order"

  P(5, 3) = 5! / 2! = 120 / 2 = 60

  ┌──────────────────────────────────────────────────────────┐
  │  KEY DISTINCTION:                                        │
  │  Permutation: ORDER MATTERS       {A,B} ≠ {B,A}        │
  │  Combination: Order doesn't matter {A,B} = {B,A}        │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Counting Permutations

```
  Full permutations (r = n):
  P(n, n) = n!

  ┌────┬────────┬──────────────────────────────────┐
  │ n  │  n!    │ Example                           │
  ├────┼────────┼──────────────────────────────────┤
  │ 1  │  1     │ {A}                               │
  │ 2  │  2     │ AB, BA                            │
  │ 3  │  6     │ ABC, ACB, BAC, BCA, CAB, CBA     │
  │ 4  │  24    │ 24 arrangements                   │
  │ 5  │  120   │ 120 arrangements                  │
  │ 10 │3628800 │ ~3.6 million                      │
  │ 20 │~2.4×10¹⁸│ exceeds 64-bit!                  │
  └────┴────────┴──────────────────────────────────┘

  Partial permutations:
  P(5, 2) = 5 × 4 = 20
  "Pick 2 from 5, order matters"
  
  First slot: 5 choices
  Second slot: 4 choices
  Total: 5 × 4 = 20
```

---

## 1.3 Permutations with Repetition

```
  n items with repetitions of counts r₁, r₂, ..., rₖ:

  Total = n! / (r₁! × r₂! × ... × rₖ!)

  Example: arrangements of "MISSISSIPPI"
  M:1, I:4, S:4, P:2
  
  Total = 11! / (1! × 4! × 4! × 2!)
        = 39916800 / (1 × 24 × 24 × 2)
        = 39916800 / 1152
        = 34650
```

---

## 1.4 Generating All Permutations

### Method 1: Recursive (Backtracking)

```
FUNCTION permute(arr, start, end):
    IF start == end:
        PRINT arr
        RETURN
    FOR i = start TO end:
        SWAP(arr[start], arr[i])
        permute(arr, start + 1, end)
        SWAP(arr[start], arr[i])    // backtrack

  Call: permute([1,2,3], 0, 2)

  Tree for [1, 2, 3]:
  
                    [1,2,3]
                 /     |     \
           [1,2,3]  [2,1,3]  [3,2,1]
            / \      / \      / \
       [1,2,3][1,3,2][2,1,3][2,3,1][3,2,1][3,1,2]

  Output: 123, 132, 213, 231, 321, 312
```

### Method 2: Next Permutation

```
FUNCTION nextPermutation(arr):
    // Step 1: Find largest i such that arr[i] < arr[i+1]
    i = len(arr) - 2
    WHILE i >= 0 AND arr[i] >= arr[i+1]:
        i--
    IF i < 0:
        REVERSE(arr)    // last permutation → wrap to first
        RETURN
    // Step 2: Find largest j > i such that arr[j] > arr[i]
    j = len(arr) - 1
    WHILE arr[j] <= arr[i]:
        j--
    // Step 3: Swap and reverse
    SWAP(arr[i], arr[j])
    REVERSE(arr[i+1 .. end])

Time: O(n) per call, O(n! × n) total for all permutations
```

### Trace: Next Permutation of [1, 3, 2]

```
  Step 1: Find i where arr[i] < arr[i+1]
          i=1: arr[1]=3 ≥ arr[2]=2 → skip
          i=0: arr[0]=1 < arr[1]=3 → i=0
  
  Step 2: Find j > 0 where arr[j] > arr[0]=1
          j=2: arr[2]=2 > 1 → j=2
  
  Step 3: Swap arr[0] and arr[2]: [2, 3, 1]
          Reverse arr[1..2]: [2, 1, 3]
  
  Result: [2, 1, 3] ✓ (next after [1,3,2] in lex order)
```

---

## 1.5 Derangements

```
  A derangement is a permutation where NO element appears
  in its original position.

  D(n) = n! × Σ (-1)^k / k!  for k = 0 to n

  ┌────┬────┬──────┬─────────────────────────────┐
  │ n  │ n! │ D(n) │ Examples                     │
  ├────┼────┼──────┼─────────────────────────────┤
  │ 1  │  1 │  0   │ none                         │
  │ 2  │  2 │  1   │ {2,1}                        │
  │ 3  │  6 │  2   │ {2,3,1}, {3,1,2}             │
  │ 4  │ 24 │  9   │ 9 derangements               │
  │ 5  │120 │ 44   │ 44 derangements              │
  └────┴────┴──────┴─────────────────────────────┘

  Recurrence: D(n) = (n-1)(D(n-1) + D(n-2))
  Base: D(0) = 1, D(1) = 0

  Probability: D(n)/n! → 1/e ≈ 0.3679 as n → ∞
```

---

## 1.6 Circular Permutations

```
  Arrange n items in a CIRCLE:
  
  Circular permutations = (n-1)!
  
  Why? Fix one element's position to remove rotational symmetry.
  
  Example: Seat 4 people around a circular table.
  Linear: 4! = 24
  Circular: 3! = 6

  ┌──────────────────────────────────┐
  │        A                         │
  │      /   \                       │
  │     D     B     ← one arrangement│
  │      \   /                       │
  │        C                         │
  │                                  │
  │  Rotations ABCD, BCDA, CDAB,    │
  │  DABC are all the SAME circular │
  │  arrangement. So divide by n.    │
  └──────────────────────────────────┘

  With reflection (necklace): (n-1)!/2
```

---

## 1.7 Permutation as a Function

```
  A permutation π of {1,...,n} maps each element to a unique element.
  
  π = (2, 3, 1, 5, 4)
  
  π(1)=2, π(2)=3, π(3)=1, π(4)=5, π(5)=4
  
  Cycle notation: (1 2 3)(4 5)
  Means: 1→2→3→1 and 4→5→4
  
  ┌──────────────────────────────────────────────────────────┐
  │  ORDER of a permutation = LCM of cycle lengths          │
  │                                                          │
  │  (1 2 3)(4 5): cycles of length 3 and 2                 │
  │  Order = LCM(3, 2) = 6                                  │
  │  π⁶ = identity (applying π six times returns to start) │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Formula | Key Note |
|---------|---------|----------|
| P(n, r) | n!/(n-r)! | Order matters |
| Full permutation | n! | All arrangements |
| With repetition | n!/(r₁!×...×rₖ!) | Repeated elements |
| Circular | (n-1)! | Fix one position |
| Derangements | (n-1)(D(n-1)+D(n-2)) | No fixed points |
| Next permutation | O(n) per step | Lexicographic order |
| Permutation order | LCM of cycle lengths | π^order = identity |

---

## Quick Revision Questions

1. **What is P(8, 3)?** Compute it.
2. **How many distinct arrangements** of the letters in "BANANA"?
3. **Generate the next permutation** of [2, 3, 1].
4. **What is D(4)?** List all derangements of {1,2,3,4}.
5. **How many ways** to seat 6 people around a circular table?
6. **What is the order** of the permutation (1 3 5 2)(4 6)?

---

[← Previous Unit: Applications](../05-Fast-Exponentiation/06-applications.md) | [Back to README](../README.md) | [Next: Combinations →](02-combinations.md)
