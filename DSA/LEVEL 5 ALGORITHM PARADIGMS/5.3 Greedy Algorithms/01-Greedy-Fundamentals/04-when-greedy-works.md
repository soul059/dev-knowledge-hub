# Chapter 4: When Greedy Works

## Overview
Greedy algorithms don't work for every optimization problem. Understanding **when** greedy is the right approach is crucial. This chapter covers the conditions, problem types, and structural patterns that signal greedy will produce optimal results.

---

## The Two Required Properties

```
┌─────────────────────────────────────────────────────────┐
│          GREEDY WORKS WHEN BOTH HOLD:                   │
│                                                         │
│  ┌─────────────────────┐  ┌─────────────────────────┐  │
│  │  GREEDY CHOICE      │  │  OPTIMAL                │  │
│  │  PROPERTY            │  │  SUBSTRUCTURE           │  │
│  │                      │  │                          │  │
│  │  Local optimal      │  │  Optimal solution       │  │
│  │  choice is part     │  │  contains optimal       │  │
│  │  of global optimal  │  │  sub-solutions          │  │
│  └─────────────────────┘  └─────────────────────────┘  │
│                                                         │
│              BOTH REQUIRED ──► GREEDY IS CORRECT        │
└─────────────────────────────────────────────────────────┘
```

---

## Categories Where Greedy Works

### 1. Scheduling and Interval Problems

```
Pattern: Sort elements, process in order, make greedy pick

┌────────────────────────────────────────────────┐
│  SCHEDULING PROBLEMS                           │
│                                                │
│  • Activity Selection → sort by finish time    │
│  • Job Scheduling → sort by deadline/profit    │
│  • Interval Partitioning → sort by start time  │
│  • Meeting Rooms → sort by start/end times     │
│                                                │
│  WHY GREEDY WORKS:                             │
│  Choosing the "least greedy" option (earliest  │
│  finish, tightest deadline) leaves maximum     │
│  room for future choices.                      │
└────────────────────────────────────────────────┘
```

### 2. Matroid Problems

```
A matroid is a structure (S, I) where:
  S = ground set of elements
  I = collection of "independent" subsets

Properties:
  1. Empty set is independent
  2. Subset of independent set is independent (hereditary)
  3. Exchange property: if A, B ∈ I and |A| < |B|,
     then ∃ b ∈ B\A such that A ∪ {b} ∈ I

EXAMPLES OF MATROIDS:
┌──────────────────────────────────────────────┐
│  • Linear independence of vectors            │
│  • Forests in a graph (→ MST)               │
│  • Matchings in bipartite graphs             │
│  • Subsets within capacity limits            │
│                                              │
│  KEY THEOREM:                                │
│  For ANY weighted matroid problem,            │
│  GREEDY gives the optimal solution!          │
└──────────────────────────────────────────────┘
```

### 3. Fractional/Continuous Problems

```
When you can take FRACTIONS of items:

  ┌──────────────┐    ┌──────────────┐
  │  Fractional  │    │    0/1       │
  │  Knapsack    │    │  Knapsack    │
  │              │    │              │
  │  Can take    │    │  Must take   │
  │  0.5 of item │    │  whole item  │
  │              │    │  or nothing  │
  │  GREEDY ✓    │    │  GREEDY ✗    │
  └──────────────┘    └──────────────┘
```

### 4. Problems with Natural Ordering

```
When the problem has a natural way to sort and process:

Problem Type          Sort By              Greedy Pick
─────────────────────────────────────────────────────────
Activity Selection    Finish time          Earliest finish
Huffman Coding        Frequency            Two lowest freq
Fractional Knapsack   Value/weight ratio   Highest ratio
Coin Change (std)     Denomination         Largest fitting
Kruskal's MST         Edge weight          Smallest weight
Dijkstra              Distance             Shortest dist
```

---

## Structural Indicators That Greedy Works

```
┌─────────────────────────────────────────────────────────┐
│                GREEDY-FRIENDLY SIGNALS                  │
│                                                         │
│  ✓ Problem asks for MAX/MIN of something               │
│  ✓ Elements can be sorted by a meaningful criterion    │
│  ✓ Choosing "greedily" never blocks future choices     │
│  ✓ Problem decomposes cleanly after each choice        │
│  ✓ No complex dependencies between choices              │
│  ✓ Solution is built incrementally                     │
│  ✓ Partial solutions can be extended                   │
│  ✓ Exchange argument is possible                       │
│                                                         │
│  RED FLAGS (Greedy might NOT work):                    │
│  ✗ Choices affect each other in complex ways           │
│  ✗ Must consider ALL combinations (NP-hard feel)       │
│  ✗ Small changes in input drastically change output    │
│  ✗ Problem requires "looking ahead"                    │
└─────────────────────────────────────────────────────────┘
```

---

## Decision Framework

```
                    Is the problem an
                    optimization problem?
                         │
                    ┌────┴────┐
                    │ YES     │ NO
                    ▼         ▼
              Does it have    Not a greedy
              optimal         candidate
              substructure?
                    │
               ┌────┴────┐
               │ YES     │ NO
               ▼         ▼
         Can you identify  Can't use greedy
         a safe greedy     or DP easily
         choice?
               │
          ┌────┴────┐
          │ YES     │ NO
          ▼         ▼
     Can you prove  Try Dynamic
     greedy choice  Programming
     property?      instead
          │
     ┌────┴────┐
     │ YES     │ NO
     ▼         ▼
  USE GREEDY!  Use DP or
               other approach
```

---

## Classic Problems Where Greedy Works

### With Step-by-Step Reasoning

**1. Minimum Spanning Tree (Kruskal's)**
```
WHY GREEDY WORKS:
- Sort edges by weight
- Adding cheapest edge that doesn't form cycle
- Cut property: lightest edge crossing any cut must be in MST
- This IS a matroid problem (graphic matroid)

Trace for graph:
    A──1──B     Sorted edges: (A,B,1), (B,C,2), (A,C,3)
    │    /
    3  2        Step 1: Add (A,B,1) — no cycle ✓
    │ /         Step 2: Add (B,C,2) — no cycle ✓
    C           Step 3: Skip (A,C,3) — creates cycle ✗
                
                MST = {(A,B,1), (B,C,2)}, weight = 3 ✓
```

**2. Dijkstra's Shortest Path**
```
WHY GREEDY WORKS:
- Always process vertex with minimum known distance
- Once processed, a vertex's distance is finalized
- Works because all edge weights are non-negative
- Each greedy choice (process closest unvisited) is safe

Note: Fails with negative edges! (need Bellman-Ford)
```

**3. Huffman Coding**
```
WHY GREEDY WORKS:
- Merge two least frequent symbols
- Least frequent should be deepest in tree
- Each merge reduces problem size by 1
- Can prove no better prefix code exists
```

---

## Verifying Greedy with Small Examples

Before implementing, always test with small cases:

```
TEST STRATEGY:
═══════════════

1. Try greedy on a small example (n = 3-5)
2. Enumerate ALL possible solutions
3. Check if greedy gives the best one
4. Try "adversarial" examples designed to break greedy
5. If greedy survives, try to prove correctness

Example: Testing "sort by finish time" for activities

Activities: A(1,3), B(2,5), C(4,6)

All valid selections:
  {A}       → 1 activity
  {B}       → 1 activity  
  {C}       → 1 activity
  {A, C}    → 2 activities  ← maximum
  {B} only  → 1 (B conflicts with both A and C? No, C starts at 4)
  {B, C}    → B ends at 5, C starts at 4... OVERLAP! ✗

Greedy (sort by finish): Pick A(1,3), then C(4,6) → {A,C} = 2 ✓
```

---

## Summary Table

| Category | Examples | Why Greedy Works |
|----------|----------|-----------------|
| **Scheduling** | Activity selection, job sequencing | Natural ordering by time allows safe greedy picks |
| **Matroid** | MST, independent sets | Matroid theory guarantees greedy optimality |
| **Fractional** | Fractional knapsack | Continuous choices allow precise greedy allocation |
| **Shortest Path** | Dijkstra (non-negative weights) | Processed vertices have finalized distances |
| **Coding Theory** | Huffman coding | Optimal prefix code built bottom-up greedily |
| **Coin Change** | Standard denominations | Denomination structure ensures greedy works |

---

## Quick Revision Questions

1. **What two properties must hold for a greedy algorithm to be correct?**
   > Greedy choice property and optimal substructure.

2. **What is a matroid, and why is it relevant to greedy algorithms?**
   > A matroid is an algebraic structure with hereditary and exchange properties. Any weighted optimization over a matroid can be solved optimally with greedy.

3. **Why does greedy work for fractional knapsack but not 0/1 knapsack?**
   > Fractional knapsack allows partial items, so the greedy choice (best value/weight ratio) can always fill capacity optimally. 0/1 knapsack requires whole items, creating dependencies between choices.

4. **What is the simplest way to test if greedy might work for a problem?**
   > Try it on small examples and compare with the brute-force optimal solution. Also try adversarial inputs designed to break the greedy strategy.

5. **Why does Dijkstra's algorithm fail with negative edge weights?**
   > The greedy choice of processing the closest unvisited vertex is no longer safe, because a longer path through a negative edge could be shorter overall.

6. **Name three problem types where greedy consistently gives optimal results.**
   > Activity/interval scheduling, minimum spanning tree, and Huffman coding.

---

[← Previous: Optimal Substructure](03-optimal-substructure.md) | [Next: When Greedy Fails →](05-when-greedy-fails.md)

[← Back to README](../README.md)
