# Chapter 5: Symmetry Breaking

## Overview
**Symmetry breaking** eliminates redundant exploration of equivalent solutions. When a problem's solution space contains symmetries — rotations, reflections, permutations of identical elements — a naive backtracking algorithm wastes time rediscovering the same solution in different guises. Breaking these symmetries can halve, quarter, or even reduce the search by orders of magnitude.

---

## What Is Symmetry?

```
Two solutions are SYMMETRIC if one can be transformed
into the other by a symmetry operation (rotation, 
reflection, relabeling) and both are equally valid.

Example — N-Queens (4×4):
  Solution A:          Solution B (mirror):
  . Q . .              . . Q .
  . . . Q              Q . . .
  Q . . .              . . . Q
  . . Q .              . Q . .

  These are DIFFERENT solutions but related by reflection.
  If you only need the COUNT → divide by symmetry factor.
  If you need ALL solutions → generate one, derive others.

Types of symmetry in backtracking:
  1. Geometric   — rotation, reflection of grid/board
  2. Element     — identical items (duplicates in array)
  3. Positional  — which slot gets which value (assignments)
  4. Color       — relabeling colors in graph coloring
```

---

## Strategy 1: Fix the First Choice

```
IDEA: Restrict the first decision to break symmetry.

N-Queens:
  Full board has left-right mirror symmetry.
  → First queen: only try columns 0 to n/2-1
  → Multiply count by 2
  → For odd n, handle middle column separately

  Without fixing:  search explores n first-row positions
  With fixing:     search explores ⌊n/2⌋ + (1 if n odd)
  Speedup:         ~2×

Graph coloring:
  Colors are interchangeable. If 3 colors {R, G, B}:
  → Fix first vertex to color R (always)
  → Fix second vertex to R or G (skip B if same as trying G then B)
  → Reduces first-level branching by factor of k!/(k-used)!
```

---

## Strategy 2: Sort + Skip Duplicates

```
IDEA: When input has duplicate elements, enforce an ordering
      to prevent generating the same combination twice.

The canonical technique:
  1. SORT the input array
  2. At each level, SKIP element if it equals the previous
     AND the previous was NOT used (at this level)

FUNCTION subsetsWithDup(nums):
    sort(nums)
    backtrack(nums, 0, [], result)

FUNCTION backtrack(nums, start, current, result):
    result.add(copy(current))
    FOR i = start TO n-1:
        IF i > start AND nums[i] == nums[i-1]:
            CONTINUE                    ← Skip duplicate!
        current.add(nums[i])
        backtrack(nums, i+1, current, result)
        current.removeLast()

Example: nums = [1, 2, 2]  (sorted)
  Without skip: {}, {1}, {1,2a}, {1,2a,2b}, {1,2b}, {1,2b,2a}... 
              → duplicate subsets like {1,2} appearing twice
  With skip:   {}, {1}, {1,2}, {1,2,2}, {2}, {2,2} → all unique

Why it works:
  "If I didn't use the first copy of a value at this level,
   I shouldn't use the second copy either."
  → Enforces: always pick earlier copies first
```

---

## Strategy 3: Canonical Form

```
IDEA: Among all symmetric variants of a solution,
      define ONE as "canonical" and only generate that one.

Example — generating necklaces (rotational equivalence):
  "AABB" rotations: AABB, ABBA, BBAA, BAAB
  Canonical: lexicographically smallest = AABB
  
  Only generate if current string ≤ all its rotations.

Example — unlabeled graphs:
  Two graphs are equivalent if one can be obtained by
  relabeling vertices. 
  Canonical: adjacency matrix in lexicographic order.

Implementation:
  FUNCTION isCanonical(solution):
      FOR each symmetry transform T:
          IF T(solution) < solution:
              RETURN FALSE          ← Not canonical
      RETURN TRUE

  FUNCTION backtrack(state):
      IF isComplete(state):
          IF isCanonical(state):
              output(state)
          RETURN
      ...

Drawback: checking canonicality can be expensive.
Better: build canonical form incrementally during construction.
```

---

## Strategy 4: Symmetry-Aware Constraint Addition

```
IDEA: Add constraints that exclude symmetric solutions
      WITHOUT checking each one.

Example — Set partitioning:
  Partition {1,2,3,4} into 2 equal groups.
  Solutions: {1,2}|{3,4}, {1,3}|{2,4}, {1,4}|{2,3}
  
  Symmetric: {3,4}|{1,2} = {1,2}|{3,4} (just swapped)
  
  Constraint: "Element 1 must be in the first group"
  → Fixes element 1's partition → eliminates swap symmetry
  → Cuts search in half with zero overhead

Example — Bin packing:
  Constraint: "Use bins in order" (don't skip bin 2 to use bin 3)
  → Prevents permutations of identical bins

Example — Boolean satisfiability:
  Add "symmetry breaking predicates" — clauses that
  force one of the symmetric configurations.
```

---

## Strategy 5: Orbit Counting (Burnside's Lemma)

```
For COUNTING (not listing) solutions up to symmetry:

         1
|X/G| = ─── Σ |Fix(g)|
        |G|  g∈G

Where:
  |X/G| = number of distinct solutions (up to symmetry)
  |G|   = number of symmetry operations
  |Fix(g)| = solutions unchanged by operation g

Example — Coloring a square with 2 colors:
  4 rotations: 0°, 90°, 180°, 270°
  
  Fix(0°)   = 2^4 = 16   (all colorings fixed by identity)
  Fix(90°)  = 2^1 = 2    (all corners same color)
  Fix(180°) = 2^2 = 4    (opposite corners same)
  Fix(270°) = 2^1 = 2    (same as 90°)
  
  Distinct = (16 + 2 + 4 + 2) / 4 = 6

This gives the COUNT without enumeration!
```

---

## Comparison of Strategies

```
┌─────────────────────┬───────────┬──────────┬──────────────┐
│ Strategy            │ Speedup   │ Overhead │ Applicability│
├─────────────────────┼───────────┼──────────┼──────────────┤
│ Fix first choice    │ ~k×       │ None     │ Easy, always │
│ Sort + skip dupes   │ ~|dupes|× │ O(nlogn) │ Duplicate el.│
│ Canonical form      │ ~|G|×     │ O(|G|)   │ General      │
│ Add constraints     │ ~|G|×     │ None     │ Structured   │
│ Burnside counting   │ Exact     │ O(|G|)   │ Count only   │
└─────────────────────┴───────────┴──────────┴──────────────┘

|G| = size of symmetry group
|dupes| = number of duplicate groups
k = symmetry factor (e.g., 2 for mirror, 8 for all rotations)
```

---

## Common Symmetry Groups in Problems

```
┌──────────────────┬──────────────┬────────────────────────┐
│ Problem          │ Symmetry     │ Group Size             │
├──────────────────┼──────────────┼────────────────────────┤
│ N-Queens         │ D4 (dihedral)│ 8 (4 rot + 4 reflect) │
│ Subsets w/ dupes │ Permutations │ Πi (count_i!)          │
│ Graph coloring   │ Color perm.  │ k! (k colors)          │
│ Necklaces        │ Cyclic       │ n (n rotations)        │
│ Bracelets        │ Dihedral     │ 2n (rot + reflect)     │
│ Set partition    │ Group swap   │ k! (k equal groups)    │
│ Sudoku           │ Various      │ ~10^7                  │
└──────────────────┴──────────────┴────────────────────────┘
```

---

## Practical Tips

```
1. START with the easiest symmetry break:
   Fix the first choice. Costs nothing, always helps.

2. Sort + skip is your go-to for duplicate elements.
   Works for subsets, combinations, permutations.

3. Don't over-engineer symmetry breaking.
   If simple pruning gives 2× speedup and that's enough, stop.
   Canonical form checking can be complex and bug-prone.

4. Verify correctness:
   Count solutions WITH and WITHOUT symmetry breaking.
   solutions_without = solutions_with × symmetry_factor
   (Adjust for partial symmetries)

5. Symmetry breaking + other optimizations compound:
   Fix first choice (2×) + constraint pruning (5×) = 10× total
```

---

## Summary Table

| Strategy | How It Works | Best For |
|----------|-------------|----------|
| Fix first choice | Restrict first decision | Grid, assignment problems |
| Sort + skip | Skip same-level duplicates | Arrays with duplicate values |
| Canonical form | Only generate smallest variant | Complex symmetries |
| Add constraints | Extra rules excluding mirrors | Structured problems |
| Burnside counting | Mathematical formula | Counting, not listing |

---

## Quick Revision Questions

1. **What does symmetry mean** in the context of backtracking?
2. **How does "fix first choice"** break mirror symmetry?
3. **Why does "sort + skip"** work for duplicate elements?
4. **What is a canonical form** and when is it useful?
5. **When should you use** Burnside's lemma instead of enumeration?
6. **How do you verify** symmetry breaking is correct?

---

[← Previous: Bit Manipulation Optimization](04-bit-manipulation-optimization.md) | [Next: Branch and Bound Introduction →](06-branch-and-bound-intro.md)

[← Back to README](../README.md)
