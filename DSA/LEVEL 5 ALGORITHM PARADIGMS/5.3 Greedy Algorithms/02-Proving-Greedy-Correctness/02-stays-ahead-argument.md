# Chapter 2: Stays Ahead Argument

## Overview
The **stays ahead argument** is a proof technique where you show that the greedy algorithm's solution is **at least as good as** any other solution **at every step** of the algorithm. If greedy "stays ahead" throughout, it must be optimal at the end.

---

## Core Idea

```
┌─────────────────────────────────────────────────────────┐
│              STAYS AHEAD ARGUMENT                       │
│                                                         │
│  Compare greedy solution G and optimal solution O       │
│  at EVERY step:                                         │
│                                                         │
│  Step:    1     2     3     4     ...    k              │
│  G:    |--g1--|--g2--|--g3--|--g4--|...|--gk--|         │
│  O:    |--o1--|--o2--|--o3--|--o4--|...|--om--|         │
│                                                         │
│  Show: At step i, greedy is "at least as good" as O    │
│  Formally: measure(G, i) ≥ measure(O, i) for all i    │
│                                                         │
│  Then: G's final solution ≥ O's final solution         │
│        → G is optimal!                                  │
└─────────────────────────────────────────────────────────┘
```

---

## How It Differs from Exchange Argument

```
┌──────────────────────────┬──────────────────────────┐
│    EXCHANGE ARGUMENT     │   STAYS AHEAD ARGUMENT   │
├──────────────────────────┼──────────────────────────┤
│ Transform OPT into       │ Compare G and OPT at     │
│ GREEDY by swaps          │ each step                 │
│                          │                          │
│ Focus on: individual     │ Focus on: cumulative     │
│ element differences      │ progress at each step    │
│                          │                          │
│ Show: each swap is safe  │ Show: G stays ≥ OPT     │
│                          │ at every step             │
│                          │                          │
│ Best for: orderings,     │ Best for: selection,     │
│ permutations             │ accumulation problems    │
└──────────────────────────┴──────────────────────────┘
```

---

## Formal Template

```
STAYS AHEAD PROOF:
═══════════════════

CLAIM: Greedy algorithm G produces an optimal solution.

PROOF:
1. Let G = (g1, g2, ..., gk) be the greedy solution
2. Let O = (o1, o2, ..., om) be any optimal solution
3. Define a MEASURE that compares G and O at each step

4. INDUCTION: Show that after step i, G stays ahead of O

   Base case (i=1):
     Show measure(G,1) ≥ measure(O,1)

   Inductive step (assume true for i, prove for i+1):
     Given measure(G,i) ≥ measure(O,i),
     show measure(G,i+1) ≥ measure(O,i+1)

5. CONCLUDE: After all steps, G ≥ O
   Since O was optimal and G ≥ O, G must be optimal. □
```

---

## Example 1: Activity Selection (Stays Ahead)

**Claim:** Greedy (sort by finish time) gives maximum activities.

```
PROOF:

Let G = {g1, g2, ..., gk} sorted by finish time
Let O = {o1, o2, ..., om} any optimal, sorted by finish time

STAYS AHEAD MEASURE: finish(gi) ≤ finish(oi) for all i ≤ k

Base case (i=1):
  g1 has the earliest finish time among ALL activities
  o1 is some activity in O
  → finish(g1) ≤ finish(o1) ✓

Inductive step:
  Assume finish(gi) ≤ finish(oi)
  
  Show: finish(gi+1) ≤ finish(oi+1)
  
  ┌──────────────────────────────────────────────────┐
  │  Timeline:                                       │
  │                                                  │
  │  G: ──[g1]──[g2]──...──[gi]──[gi+1]──          │
  │                          ▲     ▲                 │
  │                     finish(gi) finish(gi+1)      │
  │                                                  │
  │  O: ──[o1]──[o2]──...──[oi]──[oi+1]──          │
  │                          ▲     ▲                 │
  │                     finish(oi) finish(oi+1)      │
  │                                                  │
  │  Since finish(gi) ≤ finish(oi):                  │
  │  oi+1 starts after finish(oi) ≥ finish(gi)      │
  │  So oi+1 is a CANDIDATE for greedy at step i+1  │
  │  But greedy picks the earliest finish among      │
  │  all candidates → finish(gi+1) ≤ finish(oi+1) ✓│
  └──────────────────────────────────────────────────┘

CONCLUSION:
  Since finish(gi) ≤ finish(oi) for all i ≤ min(k,m),
  if m > k (O has more activities), then om+1 would be
  a candidate greedy could have picked but didn't.
  Contradiction! So k ≥ m, and G is optimal. □
```

---

## Example 2: Interval Scheduling (Earliest Deadline First)

**Claim:** Scheduling jobs by earliest deadline minimizes maximum lateness.

```
Measure: After scheduling i jobs, greedy's completion time 
         for job i is ≤ any other schedule's completion time for job i.

Actually, for EDF, the stays-ahead measure is:
  "The maximum lateness after i jobs in G ≤ maximum lateness after
   i jobs in O"

But this is tricky. EDF is more naturally proved by exchange argument.

BETTER EXAMPLE for stays ahead:
```

## Example 3: Tape Storage Problem

**Problem:** Given n files with lengths L1, L2, ..., Ln, arrange them on tape to minimize **average access time**.

Access time for file i = sum of lengths of all files before it + Li.

**Claim:** Sorting by increasing length minimizes average access time.

```
Let G sort by length: L[1] ≤ L[2] ≤ ... ≤ L[n]
Let O be any optimal ordering

STAYS AHEAD MEASURE: 
  Cost_G(i) = total access time for first i files in G
  Cost_O(i) = total access time for first i files in O

Show Cost_G(i) ≤ Cost_O(i) for all i.

Intuition:
  G: [short][short][medium][long][long]
  O: [long][short][long][short][medium]
  
  After 2 files:
    G: access = L1 + (L1+L2) = 2L1 + L2
    O: access = O1 + (O1+O2) 
    Since G picked the 2 shortest, 2L1+L2 ≤ 2O1+O2 ✓

  This "stays ahead" at every prefix! □
```

---

## Visual Comparison: Exchange vs Stays Ahead

```
EXCHANGE:
  OPT:    [A] [B] [C] [D]
             ↕ swap
  OPT':   [A] [C] [B] [D]    ← One step closer to GREEDY
             ↕ swap  
  GREEDY: [A] [C] [D] [B]    ← Eventually reaches GREEDY

STAYS AHEAD:
  Step:     1      2      3      4
  GREEDY:   ●──────●──────●──────●     (always ≥)
  OPT:      ●──────●──────●──────●
  
  At EVERY step, GREEDY's cumulative measure ≥ OPT's
```

---

## When to Use Stays Ahead

```
┌────────────────────────────────────────────────────┐
│        STAYS AHEAD IS BEST WHEN:                  │
│                                                    │
│  ✓ Solutions are built INCREMENTALLY              │
│  ✓ There's a natural MEASURE at each step         │
│  ✓ Greedy's advantage accumulates over time       │
│  ✓ You can use INDUCTION on number of steps       │
│                                                    │
│  EXAMPLES:                                         │
│  • Activity selection (finish times stay ahead)    │
│  • Tape storage (costs stay ahead)                 │
│  • Scheduling with natural ordering                │
│  • Selection problems with monotonic measures      │
└────────────────────────────────────────────────────┘
```

---

## Common Mistakes

```
PITFALL 1: Wrong measure
  ✗ "Greedy has more items at each step"
  ✓ "Greedy's i-th finish time ≤ OPT's i-th finish time"

PITFALL 2: Forgetting the conclude step
  ✗ "G stays ahead, so it's optimal"
  ✓ "G stays ahead, so if |O| > |G|, O has an extra 
      element that G should have picked → contradiction"

PITFALL 3: Weak inductive hypothesis
  ✗ "Greedy is good so far"
  ✓ "Specifically, finish(gi) ≤ finish(oi) for all i ≤ k"
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **What** | Show greedy stays ≥ optimal at every step |
| **Method** | Mathematical induction on step number |
| **Measure** | Problem-specific comparison criterion |
| **Best For** | Incremental selection/accumulation problems |
| **Key Step** | Define the right "stays ahead" measure |
| **Result** | G ≥ O at every step → G is optimal |

---

## Quick Revision Questions

1. **What does "stays ahead" mean in this proof technique?**
   > At every step, the greedy solution's cumulative performance measure is at least as good as the optimal solution's.

2. **What mathematical technique is used in stays-ahead proofs?**
   > Mathematical induction on the step number.

3. **In activity selection, what is the "stays ahead" measure?**
   > The finish time of the greedy's i-th activity is ≤ the finish time of the optimal's i-th activity.

4. **How does the conclusion of a stays-ahead proof typically work?**
   > If the optimal has more elements than the greedy, the extra element would be a candidate the greedy should have picked — contradiction.

5. **When is stays ahead preferred over exchange argument?**
   > When the solution is built incrementally and there's a natural cumulative measure that greedy always leads on.

6. **What's the most common mistake in stays-ahead proofs?**
   > Choosing the wrong measure or forgetting to conclude that equal-or-better at every step implies overall optimality.

---

[← Previous: Exchange Argument](01-exchange-argument.md) | [Next: Contradiction Proof →](03-contradiction-proof.md)

[← Back to README](../README.md)
