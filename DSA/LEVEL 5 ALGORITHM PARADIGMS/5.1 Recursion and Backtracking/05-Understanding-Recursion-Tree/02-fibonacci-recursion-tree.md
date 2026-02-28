# Chapter 2: Fibonacci Recursion Tree

## Overview
The Fibonacci recursion tree is the most important example for understanding **why naive recursion can be exponentially slow** and how **overlapping subproblems** lead to wasted computation.

---

## The Full Tree for fib(6)

```
                                    fib(6)
                                /            \
                          fib(5)              fib(4)
                        /       \            /       \
                   fib(4)      fib(3)    fib(3)    fib(2)
                  /     \      /   \     /   \      /  \
              fib(3)  fib(2) f(2) f(1) f(2) f(1) f(1) f(0)
              /   \    / \   / \   |   / \   |    |    |
          fib(2) f(1) f(1)f(0)f(1)f(0) 1 f(1)f(0) 1   1   0
          / \    |    |   |   |   |       |   |
        f(1)f(0) 1   1   0   1   0       1   0
         |   |
         1   0
```

---

## Counting Duplicate Computations

```
How many times is each subproblem computed?

┌──────────┬────────────┐
│ Function │ Call Count  │
├──────────┼────────────┤
│ fib(6)   │     1       │
│ fib(5)   │     1       │
│ fib(4)   │     2  ←!   │
│ fib(3)   │     3  ←!   │
│ fib(2)   │     5  ←!   │
│ fib(1)   │     8  ←!   │
│ fib(0)   │     5  ←!   │
├──────────┼────────────┤
│ TOTAL    │    25       │
└──────────┴────────────┘

Only 7 UNIQUE subproblems, but 25 total calls!
That's 25/7 ≈ 3.6× redundant work

For fib(40): ~331 million calls but only 41 unique subproblems!
```

---

## Level-by-Level Analysis

```
Level 0:  1 node    → fib(6)                       Work: 1
Level 1:  2 nodes   → fib(5), fib(4)               Work: 2
Level 2:  4 nodes   → fib(4), fib(3), fib(3), f(2) Work: 4
Level 3:  8 nodes   → ...                          Work: ≤8
Level 4:  ≤16 nodes                                 Work: ≤16
Level 5:  ≤32 nodes                                 Work: ≤32
Level 6:  ≤64 nodes (leaves)                        Work: ≤64

Total ≤ 1+2+4+8+16+32+64 = 127 = 2^7 - 1 ≈ O(2^n)

(Actual is less due to uneven tree, ≈ O(1.618^n))
```

---

## Before and After Memoization

```
WITHOUT MEMOIZATION:              WITH MEMOIZATION:

         fib(5)                          fib(5)
        /      \                        /      \
     fib(4)   fib(3)                fib(4)    fib(3)←MEMO
     /    \    /   \                /    \     HIT!
  fib(3) f(2) f(2) f(1)         fib(3) f(2)←MEMO
  /   \  / \  / \   |           /    \  HIT!
f(2) f(1)... ...    1         fib(2) f(1)
/ \   |                       / \    |
...   1                     f(1) f(0) 1
                             |    |
25 calls                     1    0
O(2^n)                      
                             9 calls
                             O(n)

[!] Memoization eliminates ALL duplicate subtrees!
```

---

## The Key Lesson

```
┌──────────────────────────────────────────────────────┐
│  OVERLAPPING SUBPROBLEMS + OPTIMAL SUBSTRUCTURE       │
│  = Use Dynamic Programming!                           │
│                                                      │
│  Step 1: Write naive recursion                       │
│  Step 2: Draw the recursion tree                     │
│  Step 3: Identify repeated subtrees                  │
│  Step 4: Add memoization (top-down DP)               │
│  Step 5: Convert to tabulation (bottom-up DP)        │
│                                                      │
│  This is the BRIDGE from Recursion to DP!            │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Metric | Without Memo | With Memo |
|--------|-------------|-----------|
| Total calls (n=6) | 25 | 11 |
| Unique subproblems | 7 | 7 |
| Time complexity | O(2^n) | O(n) |
| Space complexity | O(n) | O(n) |
| Redundant calls | 18 (72%!) | 0 |

---

## Quick Revision Questions

1. **How many times** is fib(3) computed in naive fib(6)?
2. **What is the total number** of calls for naive fib(6)?
3. **Why is the tree** not perfectly balanced?
4. **How does memoization** reduce O(2^n) to O(n)?
5. **Draw the recursion tree** for fib(5) and circle repeated subtrees.
6. **What two properties** must a problem have for DP to apply?

---

[← Previous: Visualizing Recursion](01-visualizing-recursion.md) | [Next: Identifying Overlapping Subproblems →](03-identifying-overlapping-subproblems.md)

[← Back to README](../README.md)
