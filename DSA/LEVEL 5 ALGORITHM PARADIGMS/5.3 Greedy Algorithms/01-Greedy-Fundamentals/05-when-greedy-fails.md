# Chapter 5: When Greedy Fails

## Overview
Greedy algorithms are elegant and efficient, but they **don't always produce optimal solutions**. Understanding when greedy fails is just as important as knowing when it works. This chapter explores classic failure cases, the underlying reasons, and how to recognize problems that resist greedy approaches.

---

## Why Greedy Can Fail

```
┌────────────────────────────────────────────────────────────┐
│              WHY GREEDY FAILS                              │
│                                                            │
│  The locally optimal choice does NOT lead to the           │
│  globally optimal solution.                                │
│                                                            │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Local view:   "This looks like the best choice!"  │    │
│  │  Global view:  "But it blocks even better options!" │    │
│  └────────────────────────────────────────────────────┘    │
│                                                            │
│  Root Cause: Current choice AFFECTS future possibilities   │
│  in ways that can't be recovered from.                     │
└────────────────────────────────────────────────────────────┘
```

---

## Classic Failure: Coin Change with Arbitrary Denominations

```
Denominations: {1, 3, 4}
Target: 6

GREEDY approach (always pick largest):
  6 → pick 4 → remaining 2
  2 → pick 1 → remaining 1  
  1 → pick 1 → remaining 0
  
  Greedy answer: {4, 1, 1} = 3 coins ✗

OPTIMAL solution:
  6 → pick 3 → remaining 3
  3 → pick 3 → remaining 0
  
  Optimal answer: {3, 3} = 2 coins ✓

WHY IT FAILED:
┌─────────────────────────────────────────┐
│  Picking 4 (locally best) blocked the   │
│  superior combination of 3+3.           │
│                                         │
│  The greedy choice (4) committed us to  │
│  a path where we can't reach the        │
│  optimal solution.                      │
└─────────────────────────────────────────┘
```

---

## Classic Failure: 0/1 Knapsack

```
Capacity = 10

Items:
  Item A: weight=6, value=30  (ratio: 5.0)
  Item B: weight=5, value=28  (ratio: 5.6) ← best ratio
  Item C: weight=5, value=27  (ratio: 5.4)

GREEDY by value/weight ratio:
  Pick B (w=5, v=28) → remaining capacity: 5
  Pick C (w=5, v=27) → remaining capacity: 0
  Total value = 55

But wait! Check:
  Pick A (w=6, v=30)
  Pick remaining? Only C fits? No, w=6+5=11 > 10... 
  Try: A alone = 30
  Try: B + C = 28 + 27 = 55 (capacity exactly 10) ✓
  Try: A + B = 30 + 28 = 58? weight = 11 > 10 ✗
  Try: A + C = 30 + 27 = 57? weight = 11 > 10 ✗

Actually greedy gives 55 here. Let's try a clearer example:

Items:
  Item A: weight=3, value=20  (ratio: 6.67)
  Item B: weight=5, value=30  (ratio: 6.0)
  Item C: weight=4, value=26  (ratio: 6.5)
Capacity = 7

GREEDY by ratio: Pick A(3,20) + C(4,26) = value 46, weight 7 ✓
Alternatives:     Pick B(5,30) + A(3,20) = weight 8 > 7 ✗
                  Pick B(5,30) alone = 30
                  Pick C(4,26) + A(3,20) = 46

Hmm, let's use the classic textbook example:

Items:
  Item A: weight=10, value=60  (ratio: 6.0)
  Item B: weight=20, value=100 (ratio: 5.0)
  Item C: weight=30, value=120 (ratio: 4.0)
Capacity = 50

GREEDY by ratio: A(10,60) + B(20,100) = weight 30, value 160
                  Can add C? 30+30=60 > 50 ✗
                  Total = 160

OPTIMAL: B(20,100) + C(30,120) = weight 50, value 220 ✓

Greedy gave 160 vs optimal 220! FAILED ✗

WHY: Can't take fractions, so best-ratio item may 
     waste capacity that could hold more valuable combos.
```

---

## Classic Failure: Traveling Salesman (Nearest Neighbor)

```
Cities and distances:
       A
      /|\
     2  5  7
    /   |   \
   B    C    D
    \   |   /
     3  6  1
      \ | /
       E

Nearest Neighbor from A:
  A → B (dist 2) → E (dist 3) → D (dist 1) → C (dist 6) → A (dist 5)
  Total: 2 + 3 + 1 + 6 + 5 = 17

Optimal Tour:
  A → B → E → D → C → A  
  2 + 3 + 1 + 6 + 5 = 17  (same here, but in general...)

Better example:
    1       10
A ──── B ──── C
│             │
│      5      │
└──── D ──────┘
  2       3

Nearest Neighbor from A: A→B(1)→D(via ?)...
The nearest neighbor heuristic can produce tours 
that are O(log n) times the optimal for general instances.

KEY INSIGHT: TSP has no greedy choice property.
Each city visit constrains all remaining visits.
```

---

## Classic Failure: Longest Path in DAG

```
Graph:
    A ──3──► B ──4──► D
    │                 ▲
    └──2──► C ──1───┘

GREEDY (always pick heaviest edge):
  From A: pick A→B (weight 3)
  From B: pick B→D (weight 4)
  Path: A→B→D, length = 7

But: A→C→D = 2+1 = 3 (shorter, greedy still "wins" here)

Better failure example with choices:
    A ──10──► B ──1──► E
    │                   
    └──2──► C ──20──► D ──1──► E

GREEDY from A: pick A→B (10) → B→E (1) = 11
OPTIMAL: A→C (2) → C→D (20) → D→E (1) = 23

Greedy picked the locally best edge but missed 
the globally superior path.
```

---

## Patterns That Break Greedy

```
┌────────────────────────────────────────────────────────────┐
│           PATTERNS WHERE GREEDY FAILS                      │
│                                                            │
│  1. DEPENDENCY BETWEEN CHOICES                             │
│     Choosing item A affects whether B is optimal           │
│     Example: 0/1 Knapsack                                  │
│                                                            │
│  2. NON-STANDARD STRUCTURES                                │
│     Denominations/graph structures without nice properties  │
│     Example: Coin change with {1, 3, 4}                    │
│                                                            │
│  3. COMBINATORIAL EXPLOSIONS                               │
│     Must consider exponentially many combinations          │
│     Example: TSP, Subset Sum                               │
│                                                            │
│  4. ALL-OR-NOTHING CONSTRAINTS                             │
│     Can't take partial items/solutions                     │
│     Example: 0/1 Knapsack vs Fractional Knapsack           │
│                                                            │
│  5. FUTURE INFORMATION MATTERS                             │
│     Current choice quality depends on future input         │
│     Example: Online algorithms without lookahead           │
└────────────────────────────────────────────────────────────┘
```

---

## How to Detect Greedy Will Fail

### The Counterexample Method

```
STEPS TO FIND A COUNTEREXAMPLE:
════════════════════════════════

1. Identify what "greedy" would choose
2. Construct an input where:
   - The greedy choice looks good initially
   - But a different choice leads to a better overall solution
3. Common construction: 
   - One "attractive" option that blocks better combos
   - Two "modest" options that together beat the greedy

Template:
  Greedy picks: [BIG] → remaining is [small, small]
  Optimal:      skip BIG, pick [medium, medium] = MORE
```

### Red Flags Checklist

```
□ Can a "mediocre" first choice lead to better overall?
□ Does taking item X prevent taking Y AND Z (both valuable)?
□ Are there complex interactions between choices?
□ Does the problem feel like it needs "looking ahead"?
□ Is the problem NP-hard? (Greedy usually only approximates)
□ Can you construct a small counterexample?

If YES to any → Greedy likely fails for this problem
```

---

## What to Do When Greedy Fails

```
┌────────────────────────────────────────────┐
│  GREEDY FAILS ──► WHAT NOW?               │
│                                            │
│  Option 1: DYNAMIC PROGRAMMING             │
│    - 0/1 Knapsack                          │
│    - Coin change (arbitrary)               │
│    - Edit distance                         │
│    - Optimal time: polynomial usually      │
│                                            │
│  Option 2: BACKTRACKING                    │
│    - N-Queens                              │
│    - Sudoku                                │
│    - Subset sum                            │
│    - Time: exponential worst case          │
│                                            │
│  Option 3: GREEDY APPROXIMATION            │
│    - When exact solution is too expensive   │
│    - TSP: nearest neighbor ≤ O(log n)·OPT  │
│    - Set cover: greedy ≤ ln(n)·OPT        │
│    - Vertex cover: 2-approximation         │
│                                            │
│  Option 4: BRANCH AND BOUND               │
│    - Systematic search with pruning        │
│    - Use greedy for bounds                 │
└────────────────────────────────────────────┘
```

---

## Greedy as Approximation

Even when greedy doesn't give the optimal solution, it often provides a **good approximation**:

| Problem | Greedy Approximation Ratio | Optimal Method |
|---------|---------------------------|----------------|
| TSP (nearest neighbor) | O(log n) | Exact: O(n! ) |
| Set Cover | O(ln n) | Exact: NP-hard |
| Vertex Cover | 2x optimal | Exact: NP-hard |
| MAX-SAT | 3/4 of optimal | Exact: NP-hard |
| Bin Packing (First Fit) | 1.7x optimal | Exact: NP-hard |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Core Issue** | Local optimal ≠ Global optimal |
| **Root Cause** | Choices have hidden dependencies on future decisions |
| **Detection** | Find counterexamples with small inputs |
| **Common Failures** | 0/1 Knapsack, arbitrary coin change, TSP |
| **Alternatives** | Dynamic programming, backtracking, B&B |
| **Silver Lining** | Greedy often gives good approximations even when not optimal |

---

## Quick Revision Questions

1. **Why does greedy fail for 0/1 Knapsack?**
   > Taking the item with the best value/weight ratio may consume capacity that prevents a better combination of items.

2. **Give a counterexample where greedy coin change fails.**
   > Denominations {1, 3, 4}, target 6: Greedy gives {4,1,1} (3 coins), optimal is {3,3} (2 coins).

3. **What is the fundamental reason greedy algorithms fail?**
   > The locally optimal choice prevents reaching the globally optimal solution because choices have hidden interdependencies.

4. **How can greedy still be useful for problems where it doesn't give optimal solutions?**
   > It can serve as an approximation algorithm, providing solutions within a guaranteed factor of optimal, often very efficiently.

5. **What approach should you try first when greedy fails for an optimization problem?**
   > Dynamic programming, which explores all sub-choices systematically while avoiding redundant computation.

6. **How do you systematically test whether greedy works for a given problem?**
   > Try the greedy strategy on small inputs and compare with brute-force optimal. Look for counterexamples where greedy makes an attractive initial choice that leads to a suboptimal overall solution.

---

[← Previous: When Greedy Works](04-when-greedy-works.md) | [Next: Greedy vs DP →](06-greedy-vs-dp.md)

[← Back to README](../README.md)
