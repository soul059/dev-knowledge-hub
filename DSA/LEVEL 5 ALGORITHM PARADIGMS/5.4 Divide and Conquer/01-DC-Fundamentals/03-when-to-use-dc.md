# Chapter 3: When to Use Divide and Conquer

## Overview

Knowing **when** to apply D&C is just as important as knowing **how**. Not every problem benefits from a D&C approach. This chapter establishes clear criteria, patterns, and signals that tell you a problem is a good candidate for D&C.

---

## The D&C Checklist

Before applying D&C, ask these questions:

```
┌────────────────────────────────────────────────────────────┐
│           D&C APPLICABILITY CHECKLIST                     │
│                                                            │
│  ✓ Can the problem be broken into SMALLER instances       │
│    of the SAME problem?                                   │
│                                                            │
│  ✓ Are the subproblems INDEPENDENT of each other?         │
│    (No subproblem needs results from another)             │
│                                                            │
│  ✓ Can subproblem solutions be COMBINED efficiently       │
│    to solve the original?                                  │
│                                                            │
│  ✓ Is there a clear BASE CASE that can be solved          │
│    directly?                                               │
│                                                            │
│  ✓ Does the D&C approach give a BETTER time complexity    │
│    than brute force?                                       │
│                                                            │
│  If ALL answers are YES → D&C is likely a good fit!       │
└────────────────────────────────────────────────────────────┘
```

---

## Ideal Conditions for D&C

### 1. Self-Similar Structure

The problem at size `n` looks like the same problem at size `n/2`:

```
GOOD for D&C:                     BAD for D&C:
Sorting an array                   Traveling Salesman
├── Sorting left half              ├── Not decomposable into
├── Sorting right half                 similar subproblems
└── Merge sorted halves            └── Subproblems are entangled

Finding max in array               Longest Common Subsequence
├── Max of left half               ├── Subproblems OVERLAP
├── Max of right half              └── Better suited for DP
└── Max of the two maxes
```

### 2. Independent Subproblems

```
INDEPENDENT (D&C works):
┌─────────┐     ┌─────────┐
│ Sub A   │     │ Sub B   │   ← A doesn't need B's result
│ (works  │     │ (works  │      B doesn't need A's result
│ alone)  │     │ alone)  │
└─────────┘     └─────────┘

DEPENDENT (Use DP instead):
┌─────────┐ ──→ ┌─────────┐
│ Sub A   │     │ Sub B   │   ← B needs A's result
│         │ ←── │         │      A needs B's result
└─────────┘     └─────────┘
```

### 3. Efficient Combine Step

```
If combine cost is too high, D&C doesn't help:

Merge Sort:  Combine = O(n)  → Total = O(n log n)  ✓ Good!
Finding Max: Combine = O(1)  → Total = O(n)         ✓ Good!
Bad example: Combine = O(n²) → Total often = O(n²)  ✗ No gain!
```

---

## Decision Tree: Should You Use D&C?

```
                      START
                        │
           Can problem be split into
           similar smaller problems?
                  /           \
                YES            NO
                 │              │
        Are subproblems     Try other
        independent?        paradigms
              /      \         │
           YES        NO      DP, Greedy,
            │          │      Brute Force
   Can solutions    Overlapping
   be combined      subproblems?
   efficiently?         │
      /     \         Use DP
    YES      NO
     │        │
  Use D&C   Consider
     │      other
     │      approaches
     │
  Does it beat
  brute force?
    /      \
  YES      NO
   │        │
 D&C!    Brute force
         may be simpler
```

---

## Problems Well-Suited for D&C

| Problem Type | Why D&C Works |
|-------------|---------------|
| **Sorting** | Array splits into halves; sorted halves merge easily |
| **Searching** | Search space halves each step |
| **Computational Geometry** | Points split by a line; merge across boundary |
| **Algebraic Problems** | Large multiplications split into smaller ones |
| **Tree Problems** | Trees naturally split into subtrees |
| **Optimization on ranges** | Range splits; combine extremes across halves |

---

## Problems NOT Well-Suited for D&C

| Problem Type | Why D&C Fails | Better Approach |
|-------------|---------------|-----------------|
| **Overlapping subproblems** | Redundant computation | Dynamic Programming |
| **Greedy-optimal problems** | D&C is overkill | Greedy Algorithm |
| **Graph traversal** | Not naturally decomposable | BFS/DFS |
| **Problems with dependencies** | Subproblems not independent | DP or Topological Order |
| **Small input sizes** | Recursion overhead > benefit | Direct computation |

---

## Recognizing D&C Patterns in Problems

### Pattern 1: "Split and Merge"
```
Signal words: "sorted", "merge", "combine", "two halves"
Examples: Merge Sort, Count Inversions, Merge K Lists

Structure:
  Split array in half → Solve both → Merge results
```

### Pattern 2: "Eliminate Half"
```
Signal words: "sorted array", "find", "search", "log n"  
Examples: Binary Search, Rotated Array Search

Structure:
  Compare with middle → Eliminate one half → Recurse on other
```

### Pattern 3: "Partition Around Pivot"
```
Signal words: "kth element", "partition", "rearrange"
Examples: Quick Sort, Quick Select

Structure:
  Pick pivot → Partition → Recurse on relevant part
```

### Pattern 4: "Geometric Division"
```
Signal words: "points", "closest", "convex", "2D/3D"
Examples: Closest Pair, Convex Hull

Structure:
  Split by coordinate → Solve halves → Check boundary
```

### Pattern 5: "Algebraic Decomposition"
```
Signal words: "multiply", "polynomial", "matrix", "large numbers"
Examples: Strassen's, Karatsuba, FFT

Structure:
  Split operands → Fewer recursive multiplications → Combine
```

---

## D&C vs Other Strategies: When to Choose What

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Problem has OPTIMAL SUBSTRUCTURE?                            │
│           │                                                    │
│     ┌─────┴──────┐                                            │
│    YES           NO → Brute Force / Backtracking              │
│     │                                                          │
│  Subproblems OVERLAP?                                         │
│     │                                                          │
│  ┌──┴───┐                                                     │
│ YES     NO                                                     │
│  │       │                                                     │
│  DP    Subproblems INDEPENDENT?                               │
│         │                                                      │
│      ┌──┴───┐                                                 │
│     YES     NO → Other methods                                │
│      │                                                         │
│   Local choice sufficient?                                     │
│      │                                                         │
│   ┌──┴───┐                                                    │
│  YES     NO                                                    │
│   │       │                                                    │
│ Greedy   D&C  ← You are here!                                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Anti-Patterns: When D&C Hurts

### 1. Overlapping Subproblems (Use DP Instead)

```
Fibonacci with D&C (BAD):
                    fib(5)
                   /      \
              fib(4)      fib(3)      ← fib(3) computed TWICE!
             /     \      /    \
         fib(3)  fib(2) fib(2) fib(1) ← fib(2) computed THREE TIMES!
         /   \
     fib(2) fib(1)

Exponential time O(2^n) — Use DP: O(n)
```

### 2. Unbalanced Splits (Degrades Performance)

```
Quick Sort worst case (already sorted, first element as pivot):

[1, 2, 3, 4, 5, 6, 7, 8]
pivot = 1
Left: []   Right: [2,3,4,5,6,7,8]    ← Extremely unbalanced!
           pivot = 2
           Left: []   Right: [3,4,5,6,7,8]
                       ...

Becomes O(n²) instead of O(n log n)!
```

### 3. Expensive Combine Step

```
If combining takes O(n²), your recurrence becomes:
T(n) = 2T(n/2) + O(n²)

By Master Theorem: T(n) = O(n²)
No improvement over brute force!
```

---

## Real-World Applications

| Domain | Application | D&C Algorithm |
|--------|-------------|---------------|
| Databases | Sorting large datasets | Merge Sort |
| Search Engines | Searching sorted indices | Binary Search |
| Signal Processing | Audio/Image processing | FFT |
| Computer Graphics | Rendering, collision detection | Closest Pair, Convex Hull |
| Cryptography | Large number arithmetic | Karatsuba |
| Machine Learning | k-d trees, matrix operations | Strassen's |
| Operating Systems | Memory allocation | Buddy System |

---

## Summary Table

| Criterion | Use D&C | Don't Use D&C |
|-----------|---------|---------------|
| **Subproblem structure** | Self-similar | Different structure |
| **Subproblem independence** | Independent | Overlapping/Dependent |
| **Combine efficiency** | O(n) or less | O(n²) or more |
| **Size reduction** | Significant (n → n/2) | Minimal (n → n-1) |
| **Recursion depth** | O(log n) | O(n) |
| **Alternative** | Worse or same complexity | DP/Greedy is better |

---

## Quick Revision Questions

1. **What are the four key conditions for a problem to be a good candidate for D&C?**
2. **Why is the Fibonacci problem a bad fit for D&C? What paradigm should be used instead?**
3. **What happens to Quick Sort's complexity when splits are consistently unbalanced?**
4. **Name three problem domains where D&C is particularly effective.**
5. **How do you distinguish between a problem that needs D&C vs Dynamic Programming?**
6. **What is the maximum useful combine cost for D&C to outperform brute force?**

---

[← Previous: Three Steps](02-three-steps.md) | [Next: D&C vs Other Paradigms →](04-dc-vs-other-paradigms.md)
