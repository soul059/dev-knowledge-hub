# Chapter 3: Identifying Overlapping Subproblems

## Overview
Overlapping subproblems is one of the **two essential properties** for dynamic programming. Learning to spot them in a recursion tree is the key skill that bridges recursion and DP.

---

## What Are Overlapping Subproblems?

```
Definition: A problem has overlapping subproblems when
the SAME smaller problem is solved MULTIPLE times during
the recursive computation.

┌───────────────────────────────────────────────────┐
│  NOT every recursive problem has this property!    │
│                                                   │
│  Has Overlapping:     Does NOT Have:              │
│  ✓ Fibonacci          ✗ Merge Sort                │
│  ✓ Grid Paths         ✗ Binary Search             │
│  ✓ Coin Change        ✗ Quick Sort (partitions)   │
│  ✓ LCS               ✗ Tower of Hanoi            │
│  ✓ Knapsack           ✗ Factorial                 │
└───────────────────────────────────────────────────┘
```

---

## How to Spot Them: The Tree Method

### Step 1: Write the Recurrence
```
Example: Climbing Stairs (1 or 2 steps at a time)

ways(n) = ways(n-1) + ways(n-2)
ways(0) = 1  (reached the top)
ways(1) = 1  (one way: single step)
```

### Step 2: Draw the Tree
```
                    ways(5)
                   /        \
              ways(4)       ways(3)
             /      \       /     \
         ways(3)  ways(2) ways(2) ways(1)
         /    \    /   \   /   \    |
     ways(2) w(1) w(1) w(0) w(1) w(0)  1
      /  \    |    |    |    |    |
   w(1) w(0)  1    1    1    1    1
    |    |
    1    1
```

### Step 3: Circle Identical Subtrees
```
Look for SAME function calls with SAME arguments:

ways(3) appears: 2 times  ← OVERLAP!
ways(2) appears: 3 times  ← OVERLAP!
ways(1) appears: 5 times  ← OVERLAP!
ways(0) appears: 3 times  ← OVERLAP!

Verdict: HAS overlapping subproblems → DP applicable!
```

---

## Counter-Example: Merge Sort (No Overlap)

```
                mergeSort([5,2,8,1])
                /                   \
    mergeSort([5,2])          mergeSort([8,1])
       /        \                /        \
  mSort([5])  mSort([2])   mSort([8])  mSort([1])

Each subproblem operates on a DIFFERENT portion of the array.
No subproblem is ever repeated.
→ Divide and Conquer, NOT Dynamic Programming!
```

---

## Detection Techniques

### Technique 1: Parameter Analysis
```
If recursive calls have arguments that can REPEAT
across different branches, overlapping exists.

fib(n) → fib(n-1) and fib(n-2)
  Both branches will eventually call fib(n-2), fib(n-3), etc.
  → Parameters converge → OVERLAP!

mergeSort(arr, lo, hi)
  calls mergeSort(arr, lo, mid) and mergeSort(arr, mid+1, hi)
  → Disjoint ranges → NO OVERLAP!
```

### Technique 2: Count Parameters
```
If the number of DISTINCT parameter combinations
is much smaller than the number of recursive calls:

Example: Grid Paths(m, n)
  Unique parameter pairs: m × n
  Recursive calls: C(m+n-2, m-1) (can be exponential)
  SMALL ≪ LARGE → OVERLAP!

The subproblem space is POLYNOMIAL,
but naive recursion is EXPONENTIAL.
This gap = evidence of overlap!
```

### Technique 3: DAG Test
```
If the recursion tree can be compressed into a
Directed Acyclic Graph (DAG), overlapping exists.

     TREE:                       DAG:
      f(4)                       f(4)
     /    \                     /    \
   f(3)  f(2)    compress→   f(3)  f(2)
   / \   / \                  / \ ↗  |
 f(2) f(1)f(1)f(0)          f(1) f(0)

Tree: 9 nodes → DAG: 5 nodes
Compression possible → OVERLAP!
```

---

## Classic Problems with Overlapping Subproblems

```
Problem               Recurrence                    Overlap Source
─────────────────────────────────────────────────────────────────
Fibonacci             f(n)=f(n-1)+f(n-2)           Both branches share
Climbing Stairs       w(n)=w(n-1)+w(n-2)           Same as Fibonacci  
Grid Paths            p(i,j)=p(i-1,j)+p(i,j-1)    Multiple paths merge
Coin Change           c(a)=min(c(a-coin)+1)         Same amounts repeat
Longest Common Subseq L(i,j)=L(i-1,j-1)+1 or...   Indices converge
0/1 Knapsack          K(i,w)=max(take,skip)         Same (item,weight)
Edit Distance         E(i,j)=min(3 options)         Indices converge
```

---

## Testing Framework

```
DECISION FLOWCHART:

Start: Do you have a recursive solution?
  │
  ├──→ YES: Can the same (parameters) appear in different branches?
  │    │
  │    ├──→ YES: Does the subproblem space ≪ total calls?
  │    │    │
  │    │    ├──→ YES → OVERLAPPING SUBPROBLEMS! → Use DP
  │    │    └──→ NO  → Probably no overlap → Use Divide & Conquer
  │    │
  │    └──→ NO: Each call has unique parameters
  │         → No overlap → Divide & Conquer or plain recursion
  │
  └──→ NO: Write recursive solution first!
```

---

## Summary Table

| Property | Has Overlap | No Overlap |
|----------|------------|------------|
| Same subproblem | Solved multiple times | Solved exactly once |
| Tree vs DAG | Tree compresses to DAG | Tree IS the DAG |
| Subproblem space | ≪ Total recursive calls | ≈ Total calls |
| Optimization | Memoization / DP | Divide & Conquer |
| Examples | Fibonacci, Grid Paths | Merge Sort, Binary Search |

---

## Quick Revision Questions

1. **What are the TWO properties** required for DP to apply?
2. **Why doesn't Merge Sort** have overlapping subproblems?
3. **Given a recurrence** f(n, k) = f(n-1, k) + f(n-1, k-1), does it have overlap?
4. **What is the DAG test** for overlapping subproblems?
5. **If subproblem space is O(n²)** but total calls is O(2^n), what does this suggest?
6. **How do you convert** overlapping-subproblem recursion into DP?

---

[← Previous: Fibonacci Recursion Tree](02-fibonacci-recursion-tree.md) | [Next: Tree Structure Analysis →](04-tree-structure-analysis.md)

[← Back to README](../README.md)
