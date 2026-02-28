# Chapter 1: Exchange Argument

## Overview
The **exchange argument** is the most widely used technique for proving greedy algorithm correctness. The idea is simple but powerful: take **any** optimal solution, and show that it can be **transformed** into the greedy solution without losing quality — by "exchanging" elements one at a time.

---

## Core Idea

```
┌─────────────────────────────────────────────────────────┐
│                EXCHANGE ARGUMENT                        │
│                                                         │
│  1. Start with ANY optimal solution OPT                 │
│  2. Find a difference between OPT and GREEDY            │
│  3. EXCHANGE one element to make OPT more like GREEDY   │
│  4. Show the exchange doesn't worsen the solution       │
│  5. Repeat until OPT = GREEDY                           │
│                                                         │
│  OPT ──swap──► OPT' ──swap──► OPT'' ──...──► GREEDY   │
│  (optimal)    (still       (still          (also        │
│                optimal)     optimal)        optimal!)    │
└─────────────────────────────────────────────────────────┘
```

---

## Formal Template

```
EXCHANGE ARGUMENT PROOF:
════════════════════════

CLAIM: Greedy algorithm produces an optimal solution G.

PROOF:
1. Let O be any optimal solution.
2. If O = G, we're done.
3. If O ≠ G, find the FIRST point where they differ:
   - Let g_i be the greedy choice at step i
   - Let o_i be the choice in O at step i
   - Suppose g_i ≠ o_i for some i

4. EXCHANGE o_i with g_i in O to get O':
   - Show O' is still feasible (valid solution)
   - Show cost(O') ≤ cost(O) (not worse)

5. O' agrees with G on one more step than O did.

6. By repeating, we can transform O into G without 
   increasing cost.

7. Since O was optimal and G has the same cost, 
   G is also optimal. □
```

---

## Example 1: Activity Selection

**Claim:** Sorting by finish time and greedily picking non-overlapping activities gives the maximum number.

```
PROOF BY EXCHANGE:

Let G = {g1, g2, ..., gk} be greedy solution (sorted by finish)
Let O = {o1, o2, ..., om} be any optimal solution (sorted by finish)

We want to show k ≥ m (greedy is as good as optimal).

Find first difference: Let i be smallest index where gi ≠ oi.

Key observation: finish(gi) ≤ finish(oi)
  (Because greedy picks earliest finish time among compatible)

Exchange oi with gi in O:
┌────────────────────────────────────────────┐
│  O: ...oi-1....[oi]....oi+1...            │
│                  ↓ replace with gi         │
│  O': ...oi-1...[gi]....oi+1...            │
│                                            │
│  Since finish(gi) ≤ finish(oi):            │
│  • gi doesn't conflict with oi-1 (gi is    │
│    compatible with everything oi was)       │
│  • gi finishes earlier, so oi+1 is still   │
│    compatible                               │
│  • |O'| = |O| = m (same size)              │
│  • O' is still valid and optimal ✓         │
└────────────────────────────────────────────┘

Repeat: Each exchange makes O agree with G on one more position.
After at most m exchanges, O becomes G.
Since |O| never decreased, |G| = m, so G is optimal. □
```

---

## Example 2: Fractional Knapsack

**Claim:** Taking items by decreasing value/weight ratio is optimal.

```
PROOF BY EXCHANGE:

Sort items by ratio: r1 ≥ r2 ≥ ... ≥ rn
Greedy G takes as much of item 1 as possible, then item 2, etc.

Let O be any optimal solution.
Suppose O differs from G: there exist items i < j where
  O takes less of item i and more of item j than G does.

Exchange: Move ε amount from item j to item i in O.

  ┌──────────────────────────────────────────────────┐
  │  Before exchange:                                 │
  │  Item i: O takes xi, G takes yi, where xi < yi   │
  │  Item j: O takes xj, G takes yj, where xj > yj  │
  │                                                   │
  │  After exchange (move ε from j to i):             │
  │  Value change = ε × (ri - rj)                     │
  │                                                   │
  │  Since i < j, ri ≥ rj, so ri - rj ≥ 0           │
  │  Value change ≥ 0 (solution doesn't get worse!)   │
  │                                                   │
  │  Weight stays same (moved ε from j to i)          │
  └──────────────────────────────────────────────────┘

Repeating transforms O into G without decreasing value.
Therefore G is optimal. □
```

---

## Example 3: Job Scheduling (Minimize Lateness)

**Problem:** Schedule n jobs on one machine, each with processing time and deadline. Minimize maximum lateness.

**Claim:** Scheduling by earliest deadline first (EDF) minimizes maximum lateness.

```
PROOF BY EXCHANGE:

Let O be an optimal schedule with an INVERSION:
  jobs i and j where deadline(i) < deadline(j) but j is scheduled before i.

  ┌───────────────────────────────────────────────────┐
  │  O:  ... [  job j  ] [  job i  ] ...              │
  │          ▲───────────▲                             │
  │          Inversion! j before i but d(i) < d(j)    │
  │                                                    │
  │  O': ... [  job i  ] [  job j  ] ...              │
  │          ▲───────────▲                             │
  │          Swap! Now in deadline order               │
  └───────────────────────────────────────────────────┘

Analysis of the swap:
  - Job i now finishes earlier → its lateness decreases ✓
  - Job j now finishes where i used to → 
    lateness(j in O') = finish_time(i in O) - deadline(j)
                       ≤ finish_time(i in O) - deadline(i)  
                         [since deadline(j) > deadline(i)]
                       ≤ lateness(i in O)

  So max lateness of {i,j} doesn't increase after swap!

By repeatedly removing inversions, O becomes EDF schedule G
without increasing maximum lateness. □
```

---

## When to Use Exchange Argument

```
┌────────────────────────────────────────────────────────┐
│           EXCHANGE ARGUMENT IS BEST WHEN:             │
│                                                        │
│  ✓ Greedy and optimal produce SEQUENCES or ORDERINGS  │
│  ✓ You can identify "inversions" between solutions    │
│  ✓ Swapping adjacent elements is natural              │
│  ✓ The swap preserves feasibility                     │
│  ✓ The swap doesn't worsen the objective              │
│                                                        │
│  COMMON PROBLEM TYPES:                                │
│  • Scheduling problems (swap job order)               │
│  • Selection problems (swap selected items)           │
│  • Ordering problems (swap element positions)         │
│  • Knapsack variants (exchange quantities)            │
└────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Exchange Argument Recipe

```
RECIPE:
═══════

Step 1: DEFINE the greedy algorithm precisely
        "Greedy sorts by X and picks Y"

Step 2: ASSUME any optimal solution O exists

Step 3: IDENTIFY the first difference between O and G
        "At position i, O has element a, G has element b"

Step 4: EXCHANGE a with b (or swap their positions)

Step 5: PROVE feasibility of O' (new solution is valid)

Step 6: PROVE quality of O' (not worse than O)

Step 7: ARGUE convergence (O' is "closer" to G, 
        so repeating gives G)

Step 8: CONCLUDE that G must be optimal
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **What** | Proof technique: transform optimal → greedy by swaps |
| **When to Use** | Ordering, scheduling, selection problems |
| **Key Step** | Show each swap doesn't worsen solution quality |
| **Convergence** | Each swap makes OPT closer to GREEDY |
| **Result** | GREEDY has same quality as OPT → GREEDY is optimal |
| **Also Called** | "Exchange lemma" or "exchange proof" |

---

## Quick Revision Questions

1. **What is the exchange argument in one sentence?**
   > Transform any optimal solution into the greedy solution by swapping elements, showing no swap worsens quality.

2. **What are the two things you must show for each swap?**
   > (1) The new solution is still feasible, and (2) its quality is no worse than before.

3. **Why does the exchange argument guarantee convergence?**
   > Each swap makes the optimal solution agree with the greedy solution on one more position, and there are finitely many positions.

4. **In activity selection, what property of the greedy choice makes the exchange valid?**
   > The greedy choice (earliest finish) finishes no later than the optimal choice, so replacing doesn't create conflicts.

5. **What is an "inversion" in the context of exchange arguments?**
   > Two elements that appear in different relative orders in the optimal vs greedy solutions.

6. **Can exchange arguments be used for approximate greedy algorithms?**
   > Yes — you can bound how much each exchange worsens/improves the solution to get approximation ratios.

---

[Next: Stays Ahead Argument →](02-stays-ahead-argument.md)

[← Back to README](../README.md)
