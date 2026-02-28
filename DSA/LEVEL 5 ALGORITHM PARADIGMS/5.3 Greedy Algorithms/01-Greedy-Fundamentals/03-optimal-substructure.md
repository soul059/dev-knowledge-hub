# Chapter 3: Optimal Substructure

## Overview
**Optimal substructure** is the second essential property for greedy algorithms (and also for dynamic programming). A problem has optimal substructure if an **optimal solution to the problem contains optimal solutions to its subproblems**. In the greedy context, after making a greedy choice, the remaining subproblem must also have an optimal solution that combines with the greedy choice to give the overall optimal solution.

---

## Formal Definition

> **Optimal Substructure:** A problem exhibits optimal substructure if an optimal solution to the problem can be constructed efficiently from optimal solutions of its subproblems.

```
┌─────────────────────────────────────────────────────────────┐
│               OPTIMAL SUBSTRUCTURE                          │
│                                                             │
│   Optimal(Problem) = Greedy Choice + Optimal(Subproblem)    │
│                                                             │
│   ┌──────────────────────────────────────────┐              │
│   │        Original Problem                   │              │
│   │  ┌──────────┬───────────────────────┐    │              │
│   │  │  Greedy  │    Remaining          │    │              │
│   │  │  Choice  │    Subproblem         │    │              │
│   │  │          │  (also optimal)       │    │              │
│   │  └──────────┴───────────────────────┘    │              │
│   └──────────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

---

## Intuition with a Visual Example

### Shortest Path (Optimal Substructure ✓)

```
Shortest path from A to D:

    A ──3──► B ──2──► C ──4──► D
    │                          ▲
    └──────────12──────────────┘

Shortest path A→D = A→B→C→D (cost 9)

Subpath A→C = A→B→C (cost 5) — this IS the shortest A→C path
Subpath B→D = B→C→D (cost 6) — this IS the shortest B→D path

Every subpath of a shortest path is also a shortest path!
→ Optimal substructure holds ✓
```

### Longest Simple Path (NO Optimal Substructure ✗)

```
Graph:
    A ── B ── C
    │         │
    └─── D ───┘

Longest simple path A→C: A→D→C (length 2)
But subpath A→D = A→D (length 1)
And there exists A→B→C→D (length 3) for A→D

But we can't use A→B→C→D→... for A→C because
we'd revisit C! The "simple" constraint breaks 
optimal substructure.

→ Optimal substructure does NOT hold ✗
```

---

## How Optimal Substructure Works in Greedy

```
STEP-BY-STEP DECOMPOSITION:

Original Problem: Maximize/Minimize something from set S

Step 1: Make greedy choice g
         ┌─────────────────────────────────────┐
         │  S = {s1, s2, s3, ..., sn}          │
         │       ▲                              │
         │       │ greedy choice = s1           │
         └─────────────────────────────────────┘

Step 2: Reduce to subproblem S' = S minus elements 
        incompatible with g
         ┌─────────────────────────────────────┐
         │  S' = {s4, s5, ..., sn}             │
         │  (remaining feasible elements)       │
         └─────────────────────────────────────┘

Step 3: Optimal(S) = g + Optimal(S')
         ┌─────────────────────────────────────┐
         │  Best solution for S is:             │
         │  {g} ∪ {best solution for S'}        │
         └─────────────────────────────────────┘
```

---

## Example: Activity Selection

```
Activities sorted by finish time:
A1(1,4)  A4(5,7)  A8(8,11)  A11(12,16)

After choosing A1 (greedy choice), subproblem becomes:
"Select maximum activities from those starting at or after time 4"

Subproblem: {A4(5,7), A8(8,11), A11(12,16), ...}

Optimal for subproblem: {A4, A8, A11} = 3 activities

Combined: {A1} + {A4, A8, A11} = 4 activities (OPTIMAL for original)

┌─────────────────────────────────────────────────────┐
│ KEY POINT: The optimal solution for the subproblem  │
│ is INDEPENDENT of the greedy choice already made.   │
│ We just need to solve a smaller version of the same │
│ problem.                                             │
└─────────────────────────────────────────────────────┘
```

---

## Proving Optimal Substructure

### Proof by Cut-and-Paste

This is the standard technique:

```
PROOF TEMPLATE (Cut-and-Paste):
═════════════════════════════════

1. Assume we have optimal solution OPT for the whole problem
2. OPT = greedy_choice + solution_for_subproblem
3. Suppose solution_for_subproblem is NOT optimal for subproblem
4. Then there exists a BETTER sub-solution SUB*
5. But then greedy_choice + SUB* would be BETTER than OPT
6. This contradicts OPT being optimal!
7. Therefore, solution_for_subproblem MUST be optimal
   → Optimal substructure holds ✓

┌──────────────┐     ┌──────────────┐
│  OPT         │     │  Better?     │
│  ┌────┬─────┐│     │  ┌────┬─────┐│
│  │ g  │ sub ││     │  │ g  │SUB* ││
│  └────┴─────┘│     │  └────┴─────┘│
└──────────────┘     └──────────────┘
                     If SUB* > sub, then
                     g + SUB* > g + sub = OPT
                     Contradiction! ✗
```

---

## Optimal Substructure in Different Contexts

| Problem | Subproblem After Greedy Choice | Substructure? |
|---------|-------------------------------|---------------|
| Activity Selection | Activities starting after current finish | ✓ |
| Fractional Knapsack | Fill remaining capacity | ✓ |
| Huffman Coding | Build tree for n-1 nodes after merge | ✓ |
| Shortest Path | Shortest path on remaining graph | ✓ |
| MST | MST on contracted graph | ✓ |
| 0/1 Knapsack | Knapsack with remaining items/capacity | ✓ (but no greedy choice property) |
| Longest Simple Path | Longest simple path in subgraph | ✗ |

---

## Optimal Substructure + Greedy Choice Property

```
┌─────────────────────────────────────────────────────────┐
│       BOTH PROPERTIES ARE NEEDED                        │
│                                                         │
│  Greedy Choice     Optimal           Greedy Algorithm   │
│  Property     +    Substructure  =   Correctness        │
│                                                         │
│  "Safe to pick     "After picking,   "Greedy gives      │
│   the local         remaining         the globally       │
│   best"            problem is         optimal            │
│                    self-similar"       solution"          │
│                                                         │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐      │
│  │ Can commit │ + │ Subproblem │ = │ Correct    │      │
│  │ to choice  │   │ is smaller │   │ Algorithm  │      │
│  │            │   │ same type  │   │            │      │
│  └────────────┘   └────────────┘   └────────────┘      │
└─────────────────────────────────────────────────────────┘
```

---

## Common Pitfalls

### Pitfall 1: Assuming Optimal Substructure Without Proof

```
WRONG thinking:
"After I make my choice, the rest is a subproblem, 
 so optimal substructure holds."

RIGHT thinking:
"After I make my choice, I need to PROVE that the 
 optimal overall solution uses an optimal sub-solution."
```

### Pitfall 2: Confusing with DP Substructure

```
DP: Considers ALL choices, picks best among subproblems
    optimal(i) = max/min over all choices { choice + optimal(subproblem) }

Greedy: Makes ONE choice, shows it's safe
    optimal(i) = greedy_choice + optimal(reduced subproblem)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Definition** | Optimal solution contains optimal solutions to subproblems |
| **Role in Greedy** | After greedy choice, remaining problem has optimal sub-solution |
| **Proof Method** | Cut-and-paste argument |
| **Shared With** | Dynamic programming (also requires optimal substructure) |
| **Not Sufficient Alone** | Also need greedy choice property for greedy to work |
| **Counter Example** | Longest simple path lacks optimal substructure |

---

## Quick Revision Questions

1. **What is optimal substructure in simple terms?**
   > The best solution to a problem is built from the best solutions to its smaller parts.

2. **How does optimal substructure differ between greedy and DP?**
   > Both use it, but greedy makes one choice without exploring alternatives, while DP explores all choices and picks the best.

3. **What is the cut-and-paste proof technique?**
   > Assume the sub-solution is not optimal, replace it with a better one, and show this contradicts the overall solution being optimal.

4. **Why doesn't the longest simple path problem have optimal substructure?**
   > Because subpaths of a longest simple path may not be longest simple paths themselves (the "simple" constraint interferes).

5. **Can a problem have optimal substructure but not greedy choice property?**
   > Yes! The 0/1 Knapsack problem has optimal substructure but lacks greedy choice property. It requires DP instead.

6. **After making a greedy choice, what must be true about the subproblem?**
   > The subproblem must be a smaller instance of the same type of problem, and its optimal solution combined with the greedy choice must give the optimal solution for the original problem.

---

[← Previous: Greedy Choice Property](02-greedy-choice-property.md) | [Next: When Greedy Works →](04-when-greedy-works.md)

[← Back to README](../README.md)
