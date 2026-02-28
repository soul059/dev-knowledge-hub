# Chapter 1: What is a Greedy Algorithm?

## Overview
A **greedy algorithm** is an algorithmic paradigm that constructs a solution incrementally, always making the choice that looks **best at the current moment** — the locally optimal choice — with the hope that this leads to a **globally optimal solution**.

Unlike dynamic programming which considers all possibilities, a greedy algorithm never reconsiders its choices. Once a decision is made, it's final.

---

## Core Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                    GREEDY ALGORITHM FLOW                        │
│                                                                 │
│   Problem ──► Sort/Order ──► Pick Best ──► Reduce ──► Repeat   │
│                               Choice      Problem              │
│                                                                 │
│   ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌──────────┐    │
│   │Start│───►│Sort │───►│Pick │───►│Build│───►│ Solution │    │
│   │     │    │Data │    │Best │    │Part │    │ Complete?│    │
│   └─────┘    └─────┘    └─────┘    └──┬──┘    └────┬─────┘    │
│                                       │        YES │ NO        │
│                                       │         ▼  │           │
│                                       │      Done  │           │
│                                       └────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Greedy Strategy in Simple Terms

Think of it like a **coin change problem** (with standard denominations):

> **Problem:** Make change for 93 cents using fewest coins.  
> **Denominations:** 25¢, 10¢, 5¢, 1¢

**Greedy Approach — always pick the largest coin that fits:**

```
Step 1: 93¢ remaining → Pick 25¢ → 68¢ remaining
Step 2: 68¢ remaining → Pick 25¢ → 43¢ remaining
Step 3: 43¢ remaining → Pick 25¢ → 18¢ remaining
Step 4: 18¢ remaining → Pick 10¢ →  8¢ remaining
Step 5:  8¢ remaining → Pick  5¢ →  3¢ remaining
Step 6:  3¢ remaining → Pick  1¢ →  2¢ remaining
Step 7:  2¢ remaining → Pick  1¢ →  1¢ remaining
Step 8:  1¢ remaining → Pick  1¢ →  0¢ remaining

Total: 3×25¢ + 1×10¢ + 1×5¢ + 3×1¢ = 8 coins ✓ (Optimal!)
```

---

## General Template for Greedy Algorithms

### Pseudocode

```
GREEDY-ALGORITHM(problem):
    solution = empty
    candidates = all possible choices
    
    SORT candidates by some greedy criterion
    
    FOR each candidate c in candidates:
        IF c is feasible (doesn't violate constraints):
            ADD c to solution
    
    RETURN solution
```

### Key Components

```
┌────────────────────────────────────────────────────────┐
│              GREEDY ALGORITHM COMPONENTS               │
│                                                        │
│  1. CANDIDATE SET                                      │
│     └── All possible elements to choose from           │
│                                                        │
│  2. SELECTION FUNCTION                                 │
│     └── Chooses the "best" candidate                   │
│                                                        │
│  3. FEASIBILITY FUNCTION                               │
│     └── Checks if a candidate can be added             │
│                                                        │
│  4. OBJECTIVE FUNCTION                                 │
│     └── Assigns value to a (partial) solution          │
│                                                        │
│  5. SOLUTION FUNCTION                                  │
│     └── Checks if we've reached a complete solution    │
└────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace: Activity Selection

**Problem:** Given activities with start and finish times, select the maximum number of non-overlapping activities.

```
Activities: A1(1,4)  A2(3,5)  A3(0,6)  A4(5,7)  A5(3,9)  A6(5,9)
                     A7(6,10) A8(8,11) A9(8,12) A10(2,14) A11(12,16)

Timeline:
0    2    4    6    8    10   12   14   16
|    |    |    |    |    |    |    |    |
├────────┤                                  A1(1,4)
     ├────────┤                             A2(3,5)
├──────────────┤                            A3(0,6)
               ├────┤                       A4(5,7)
     ├──────────────────┤                   A5(3,9)
               ├────────────┤               A6(5,9)
                    ├────────────┤           A7(6,10)
                         ├──────────┤       A8(8,11)
                         ├──────────────┤   A9(8,12)
     ├──────────────────────────────┤       A10(2,14)
                                   ├────────┤ A11(12,16)
```

**Greedy Strategy:** Sort by finish time, always pick the activity that finishes earliest.

```
Sorted: A1(1,4) → A2(3,5) → A3(0,6) → A4(5,7) → A5(3,9) → 
        A6(5,9) → A7(6,10) → A8(8,11) → A9(8,12) → A10(2,14) → A11(12,16)

Step 1: Pick A1(1,4)    ✓  [last_finish = 4]
Step 2: A2(3,5)  start=3 < 4  ✗ overlaps
Step 3: A3(0,6)  start=0 < 4  ✗ overlaps
Step 4: A4(5,7)  start=5 ≥ 4  ✓ Pick!  [last_finish = 7]
Step 5: A5(3,9)  start=3 < 7  ✗ overlaps
Step 6: A6(5,9)  start=5 < 7  ✗ overlaps
Step 7: A7(6,10) start=6 < 7  ✗ overlaps
Step 8: A8(8,11) start=8 ≥ 7  ✓ Pick!  [last_finish = 11]
Step 9: A9(8,12) start=8 < 11 ✗ overlaps
Step 10: A10(2,14) start=2 < 11 ✗ overlaps
Step 11: A11(12,16) start=12 ≥ 11 ✓ Pick! [last_finish = 16]

Result: {A1, A4, A8, A11} = 4 activities (OPTIMAL!)
```

---

## Characteristics of Greedy Algorithms

| Characteristic | Description |
|---------------|-------------|
| **Irrevocable** | Once a choice is made, it's never undone |
| **Myopic** | Only considers current state, not future consequences |
| **Top-down** | Builds solution from the top, making choices one at a time |
| **Efficient** | Usually O(n log n) due to sorting, O(n) for the greedy pass |
| **Simple** | Easy to code and understand |

---

## When to Suspect Greedy Might Work

```
┌─────────────────────────────────────────────┐
│         GREEDY ALGORITHM INDICATORS         │
│                                             │
│  ✓ Problem asks for maximum/minimum         │
│  ✓ Problem involves scheduling/ordering     │
│  ✓ Problem has "at each step" language      │
│  ✓ Sorting input helps solve the problem    │
│  ✓ Making locally best choice seems right   │
│  ✓ Problem has optimal substructure         │
│  ✓ Greedy choice property can be proved     │
└─────────────────────────────────────────────┘
```

---

## Real-World Applications

1. **Navigation (GPS):** Dijkstra's algorithm uses greedy to find shortest paths
2. **File Compression:** Huffman coding builds optimal prefix codes greedily
3. **Network Design:** Minimum spanning tree algorithms (Prim's, Kruskal's)
4. **Task Scheduling:** Operating systems use greedy for job scheduling
5. **Cash Registers:** Making change with minimum coins
6. **Resource Allocation:** Assigning resources to maximize utilization

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **What** | Algorithm that makes locally optimal choices |
| **How** | Sort → Pick best → Check feasibility → Repeat |
| **When** | Problems with greedy choice property + optimal substructure |
| **Complexity** | Usually O(n log n) due to sorting |
| **Advantage** | Simple, efficient, easy to implement |
| **Disadvantage** | Doesn't always give optimal solution |
| **Key Insight** | Local optimum → Global optimum (when it works) |

---

## Quick Revision Questions

1. **What makes a greedy algorithm different from brute force?**
   > Greedy makes one choice per step and never backtracks, while brute force explores all possibilities.

2. **What are the two properties required for a greedy algorithm to work optimally?**
   > Greedy choice property and optimal substructure.

3. **Why is sorting often the first step in greedy algorithms?**
   > Sorting organizes candidates so that the "best" choice can be identified efficiently at each step.

4. **Can a greedy algorithm ever reconsider a previous choice?**
   > No. Once a choice is made, it is irrevocable — this is a defining characteristic.

5. **What is the typical time complexity of a greedy algorithm?**
   > O(n log n) due to the initial sorting step, followed by an O(n) greedy selection pass.

6. **Give an example where greedy gives the optimal solution.**
   > Activity selection: sorting by finish time and greedily picking non-overlapping activities gives the maximum number of activities.

---

[Next: Greedy Choice Property →](02-greedy-choice-property.md)

[← Back to README](../README.md)
