# Chapter 4: D&C vs Other Paradigms

## Overview

Algorithm design paradigms are strategies for structuring solutions. Understanding how **Divide and Conquer** compares with **Dynamic Programming**, **Greedy**, **Brute Force**, and **Backtracking** helps you choose the right tool for each problem.

---

## The Five Major Paradigms at a Glance

```
┌──────────────────────────────────────────────────────────────┐
│               ALGORITHM DESIGN PARADIGMS                     │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │  BRUTE   │  │  DIVIDE  │  │  GREEDY  │                  │
│  │  FORCE   │  │    &     │  │          │                  │
│  │          │  │ CONQUER  │  │          │                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
│       │              │              │                        │
│  Try all         Split into      Make best                  │
│  possibilities   parts and       local choice               │
│                  combine         each step                   │
│                                                              │
│  ┌──────────┐  ┌──────────┐                                 │
│  │ DYNAMIC  │  │  BACK-   │                                 │
│  │PROGRAMM- │  │ TRACKING │                                 │
│  │  ING     │  │          │                                 │
│  └──────────┘  └──────────┘                                 │
│       │              │                                       │
│  Solve & cache    Try and                                    │
│  overlapping      undo if                                    │
│  subproblems      fails                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Detailed Comparison Table

| Feature | Divide & Conquer | Dynamic Programming | Greedy | Backtracking | Brute Force |
|---------|:----------------:|:-------------------:|:------:|:------------:|:-----------:|
| **Approach** | Top-down split | Bottom-up / Top-down with memo | Local optimal | Trial & error | Try everything |
| **Subproblems** | Independent | Overlapping | None (single path) | Branching tree | All combinations |
| **Optimal guarantee** | Problem-specific | Yes (if correct formulation) | Only for certain problems | Yes (exhaustive) | Yes (exhaustive) |
| **Combine step** | Yes | Yes (table lookup) | No | No (undo) | No |
| **Memoization** | Not needed | Essential | Not needed | Pruning instead | N/A |
| **Time complexity** | Often O(n log n) | Often O(n²) or O(n·W) | Often O(n log n) | Often exponential | Exponential |
| **Space** | O(log n) stack | O(n) or O(n²) table | O(1) | O(n) stack | Varies |

---

## D&C vs Dynamic Programming

This is the most commonly confused pair. Here's the key distinction:

```
DIVIDE AND CONQUER:
┌──────────────────────────────────────────────┐
│                                              │
│  Problem(n)                                  │
│  ├── Sub(n/2)  ← INDEPENDENT                │
│  │   ├── Sub(n/4)                            │
│  │   └── Sub(n/4)                            │
│  └── Sub(n/2)  ← INDEPENDENT                │
│      ├── Sub(n/4)                            │
│      └── Sub(n/4)                            │
│                                              │
│  Each subproblem is UNIQUE — no repetition!  │
└──────────────────────────────────────────────┘

DYNAMIC PROGRAMMING:
┌──────────────────────────────────────────────┐
│                                              │
│  Problem(n)                                  │
│  ├── Sub(n-1) ──────────┐                   │
│  │   ├── Sub(n-2) ──┐   │                   │
│  │   └── Sub(n-3)   │   │                   │
│  └── Sub(n-2) ◄─────┘   │  OVERLAPPING!     │
│      ├── Sub(n-3) ◄──from above             │
│      └── Sub(n-4)        │                   │
│                          │                   │
│  Subproblems REPEAT — cache them!            │
└──────────────────────────────────────────────┘
```

### Concrete Example: Computing Power x^n

```
D&C Approach (no overlap):
  x^8 = x^4 × x^4
  x^4 = x^2 × x^2
  x^2 = x × x
  
  Each subproblem computed ONCE → O(log n)

DP Approach (Fibonacci — overlapping):
  fib(5) = fib(4) + fib(3)
  fib(4) = fib(3) + fib(2)    ← fib(3) appears again!
  fib(3) = fib(2) + fib(1)    ← fib(2) appears again!
  
  Without memo: O(2^n), With memo: O(n)
```

### Decision Guide: D&C or DP?

```
┌──────────────────────────────────────────┐
│ Does the problem have subproblems?       │
│            │                             │
│     ┌──────┴──────┐                      │
│   YES              NO → Greedy/Other     │
│     │                                    │
│ Do subproblems OVERLAP?                  │
│     │                                    │
│  ┌──┴───┐                               │
│ YES     NO                               │
│  │       │                               │
│  DP     D&C                              │
└──────────────────────────────────────────┘
```

---

## D&C vs Greedy

```
DIVIDE AND CONQUER:
┌────────────────────────────────────────────┐
│                                            │
│  "I'll divide the WHOLE problem,           │
│   solve ALL parts, then combine."          │
│                                            │
│  → Considers all subproblems               │
│  → Makes decision AFTER seeing all parts   │
│  → Guarantees optimal (for suitable probs) │
└────────────────────────────────────────────┘

GREEDY:
┌────────────────────────────────────────────┐
│                                            │
│  "I'll make the BEST choice right now      │
│   and never look back."                    │
│                                            │
│  → Only considers current state            │
│  → Makes decision IMMEDIATELY              │
│  → May not be optimal for all problems     │
└────────────────────────────────────────────┘
```

### Example: Activity Selection

```
GREEDY (works perfectly):
  Sort by end time → Pick first → Skip overlapping → Repeat
  Time: O(n log n)

D&C (overkill):
  Split activities → Solve each half → Combine
  More complex, same result for this problem

Winner: GREEDY (simpler and equally correct)
```

### Example: Sorting

```
GREEDY (doesn't work well):
  Pick the smallest → Put it first → Repeat → Selection Sort O(n²)

D&C (Merge Sort):
  Split → Sort halves → Merge → O(n log n)

Winner: D&C (much faster!)
```

---

## D&C vs Backtracking

```
DIVIDE AND CONQUER:
┌────────────────────────────────────┐
│  Split → Solve ALL → Combine      │
│  Every branch leads to a result   │
│  No "undo" needed                 │
│  Polynomial time (usually)        │
└────────────────────────────────────┘

BACKTRACKING:
┌────────────────────────────────────┐
│  Try → Check → Undo if wrong      │
│  Explore tree, prune bad branches  │
│  "Undo" (backtrack) is essential   │
│  Exponential time (usually)        │
└────────────────────────────────────┘
```

### Comparison on N-Queens

```
BACKTRACKING (natural fit):
  Place queen → Check conflicts → 
  If valid: continue to next row
  If invalid: remove queen, try next column

  ┌───┬───┬───┬───┐
  │ Q │   │   │   │  Place in row 0
  ├───┼───┼───┼───┤
  │   │   │ Q │   │  Try row 1 (columns 0,1 fail → col 2 works)
  ├───┼───┼───┼───┤
  │   │   │   │   │  Try row 2... all fail → BACKTRACK to row 1
  ├───┼───┼───┼───┤
  │   │   │   │   │  Try col 3 for row 1... continue
  └───┴───┴───┴───┘

D&C: Not naturally applicable to N-Queens
     (How would you split the board and combine solutions?)
```

---

## D&C vs Brute Force

```
BRUTE FORCE:                        DIVIDE AND CONQUER:
┌─────────────────────┐             ┌─────────────────────┐
│ Check EVERY possible│             │ ELIMINATE portions   │
│ solution            │             │ of the problem       │
│                     │             │                      │
│ Finding element:    │             │ Finding element:     │
│ Scan all n elements │             │ Binary Search        │
│ → O(n)              │             │ → O(log n)           │
│                     │             │                      │
│ Sorting:            │             │ Sorting:             │
│ Check all n! perms  │             │ Merge/Quick Sort     │
│ → O(n! × n)         │             │ → O(n log n)         │
│                     │             │                      │
│ Closest pair:       │             │ Closest pair:        │
│ Check all n² pairs  │             │ D&C approach         │
│ → O(n²)             │             │ → O(n log n)         │
└─────────────────────┘             └─────────────────────┘
```

---

## Side-by-Side Problem Mapping

| Problem | Best Paradigm | Why? |
|---------|--------------|------|
| Sorting | **D&C** (Merge/Quick Sort) | Independent halves, efficient merge |
| Fibonacci | **DP** | Overlapping subproblems |
| Shortest Path | **DP** (Dijkstra/BFS + DP) | Overlapping subpaths |
| Huffman Coding | **Greedy** | Local choice is globally optimal |
| N-Queens | **Backtracking** | Constraint satisfaction, trial & error |
| Closest Pair | **D&C** | Geometric split works perfectly |
| Knapsack | **DP** | Overlapping sub-decisions |
| Activity Selection | **Greedy** | Greedy choice property holds |
| Matrix Chain Mult | **DP** | Overlapping subproblems |
| Binary Search | **D&C** | Search space halves |
| Sudoku Solver | **Backtracking** | Constraint propagation + undo |

---

## The Paradigm Spectrum

```
Simple ◄──────────────────────────────────────────────► Complex

Brute     Greedy     D&C         DP          Backtracking
Force
 │          │         │           │              │
 │        Best for  Best for   Best for       Best for
 │        local     independent overlapping   constraint
 │        optimal   subproblems subproblems   satisfaction
 │        choices
 │
 │   ◄─ INCREASING SOPHISTICATION ─►
 │
 O(n!) ← O(n log n) ← O(n log n) ← O(n²) ← O(2^n)
 (worst)                                      (worst)
```

---

## Hybrid Approaches

Some algorithms combine paradigms:

```
INTROSORT = Quick Sort (D&C) + Heap Sort (when depth too deep)
  → D&C with fallback

TIMSORT = Merge Sort (D&C) + Insertion Sort (for small runs)
  → D&C with greedy optimization for small cases

MEMOIZED D&C = D&C + DP
  → When D&C produces overlapping subproblems
  → Example: Matrix Chain Multiplication via recursion + memo

BRANCH AND BOUND = D&C + Backtracking + Greedy (bounds)
  → Used in optimization problems
```

---

## Summary Table

| Paradigm | Subproblems | Overlap | Best For |
|----------|-------------|---------|----------|
| **D&C** | Independent, similar | No | Sorting, searching, geometric |
| **DP** | Overlapping, similar | Yes | Optimization, counting |
| **Greedy** | None (single path) | N/A | Scheduling, compression |
| **Backtracking** | Branching tree | No | Constraint satisfaction |
| **Brute Force** | All possibilities | N/A | Small inputs, verification |

---

## Quick Revision Questions

1. **What is the key difference between D&C and DP in terms of subproblem structure?**
2. **Why is Greedy preferred over D&C for the Activity Selection problem?**
3. **Give an example where D&C is clearly better than brute force and explain the complexity improvement.**
4. **Why can't D&C be directly applied to the N-Queens problem?**
5. **What is a hybrid algorithm? Give one example that combines D&C with another paradigm.**
6. **When might you use memoization with D&C, and what paradigm does it then resemble?**

---

[← Previous: When to Use D&C](03-when-to-use-dc.md) | [Next: Advantages and Disadvantages →](05-advantages-and-disadvantages.md)
