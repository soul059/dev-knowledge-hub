# Chapter 6: Backtracking vs Brute Force

## Overview
While both approaches explore the solution space, backtracking is a **smart exploration** that prunes invalid branches early, whereas brute force **blindly generates everything** and filters at the end. Understanding the difference is crucial for choosing the right approach.

---

## Side-by-Side Comparison

```
BRUTE FORCE:                       BACKTRACKING:

Generate ALL possible              Build solutions
combinations first                 incrementally
         ↓                                ↓
Filter valid ones                  Check validity at
at the end                         EACH step
         ↓                                ↓
Return valid solutions             Prune invalid branches
                                   IMMEDIATELY

Time: Always worst case            Time: Often much better
```

---

## Example: Finding Valid Passwords

```
Problem: Find all 4-digit PINs where no two adjacent 
         digits are the same (digits 0-9)

BRUTE FORCE:
  Generate ALL 10⁴ = 10,000 PINs
  Check each one: O(4) per check
  Total: 10,000 × 4 = 40,000 operations
  
  for d1 in 0..9:
    for d2 in 0..9:
      for d3 in 0..9:
        for d4 in 0..9:
          if d1≠d2 and d2≠d3 and d3≠d4:
            output [d1,d2,d3,d4]

BACKTRACKING:
  At each digit, only try digits ≠ previous
  Level 0: 10 choices
  Level 1: 9 choices (≠ previous)
  Level 2: 9 choices
  Level 3: 9 choices
  Total: 10 × 9 × 9 × 9 = 7,290 nodes
  
  function generate(pin, depth):
    if depth == 4: output pin; return
    for d in 0..9:
      if depth==0 or d ≠ pin[depth-1]:   ← PRUNE!
        pin[depth] = d
        generate(pin, depth+1)

Savings: 7,290 vs 10,000 = 27% pruned
For longer PINs or stricter rules: MUCH more savings
```

---

## When Each Approach Wins

```
┌────────────────────────────────────────────────────────┐
│  BRUTE FORCE is acceptable when:                       │
│  • Solution space is small                             │
│  • No good pruning criteria exist                      │
│  • Simplicity matters more than speed                  │
│  • You need to process ALL combinations anyway         │
│                                                        │
│  BACKTRACKING is better when:                          │
│  • Constraints can eliminate branches early             │
│  • Solution space is exponential                       │
│  • Only a fraction of possibilities are valid          │
│  • Constraints are detectable at partial solutions     │
└────────────────────────────────────────────────────────┘
```

---

## Complexity Comparison

```
Problem         Brute Force        Backtracking        Savings
─────────────────────────────────────────────────────────────────
N-Queens        n^n                 ~n! (better)       >99% for n≥8
(n=8)           16,777,216         ~15,720             99.9%

Sudoku          9^81               Much less           >99.99%
                ≈ 2×10^77          with constraints

Graph Color     k^n                varies              depends on
(k=3, n=10)    59,049             ~1000s              structure

Subsets         2^n                2^n                 0%
(no constraints)                  (no pruning possible!)

Permutations    n^n                n!                  large
(n=10)          10^10             3,628,800           99.96%
```

---

## The Spectrum of Exploration Strategies

```
LEAST SMART ─────────────────────────────── MOST SMART

Brute      Backtrack    Branch &     Dynamic      Greedy
Force      (prune       Bound        Programming  (local
(try all)  invalid)     (prune       (avoid       optimal)
                        suboptimal)  recomputing)

O(k^n)     O(k^n)       O(k^n)      O(n^k)       O(n log n)
worst      but better   but even     polynomial!  often
case       in practice  better                    optimal

Example:   Example:     Example:     Example:     Example:
PIN check  N-Queens     TSP          Fibonacci    Huffman
           Sudoku       Knapsack     Grid Paths   Activity Select
```

---

## Converting Brute Force to Backtracking

```
RECIPE:

Step 1: Identify nested loops (brute force structure)
Step 2: Find CONSTRAINTS that can be checked early
Step 3: Convert loops to recursive calls
Step 4: Add constraint checks BEFORE recursing
Step 5: Add UNDO after recursive call returns

BRUTE FORCE (nested loops):
  for i in range(n):
    for j in range(n):
      for k in range(n):
        if valid(i, j, k):
          process(i, j, k)

BACKTRACKING (recursive):
  function solve(level, choices):
    if level == 3:
      process(choices)
      return
    for c in range(n):
      if isValidSoFar(choices + [c]):    ← EARLY check!
        choices.add(c)
        solve(level + 1, choices)
        choices.removeLast()
```

---

## Key Insight: Pruning Multiplier Effect

```
If pruning eliminates 50% of branches at EACH level:

Level 0: 10 branches → 5 explored
Level 1: 5×10 = 50 branches → 25 explored
Level 2: 25×10 = 250 branches → 125 explored
Level 3: 125×10 = 1250 branches → 625 explored

Without pruning: 10^4 = 10,000 nodes
With 50% pruning: 5^4 = 625 nodes  ← 16× reduction!

With 90% pruning per level:
Without: 10^4 = 10,000
With:    1^4 = 1        ← 10,000× reduction!

PRUNING COMPOUNDS EXPONENTIALLY with depth!
```

---

## Summary Table

| Aspect | Brute Force | Backtracking |
|--------|------------|-------------|
| Strategy | Generate all, filter | Build + prune |
| When to validate | After complete | At each step |
| Pruning | None | Constraint-based |
| Worst case | Same | Same |
| Typical case | Always worst | Often much better |
| Implementation | Simpler (loops) | Slightly complex (recursion) |
| Space | May need to store all | One path at a time |
| Best for | Small spaces, no constraints | Large spaces with constraints |

---

## Quick Revision Questions

1. **What is the fundamental difference** between brute force and backtracking?
2. **Why does pruning** compound exponentially with tree depth?
3. **When is brute force** acceptable over backtracking?
4. **Convert this brute force** to backtracking: finding all 3-letter words from 'A'-'Z' with no repeated letters.
5. **For 8-Queens**, what percentage of the brute force space does backtracking explore?
6. **Where does backtracking** sit on the spectrum between brute force and DP?

---

[← Previous: Pruning Concept](05-pruning-concept.md) | [Next Unit: Subset and Combinations →](../07-Subset-and-Combinations/01-generate-all-subsets.md)

[← Back to README](../README.md)
