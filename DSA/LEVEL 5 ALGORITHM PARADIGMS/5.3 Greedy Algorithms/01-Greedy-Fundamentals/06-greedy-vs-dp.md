# Chapter 6: Greedy vs Dynamic Programming

## Overview
Greedy algorithms and dynamic programming (DP) are both used to solve optimization problems, and both rely on **optimal substructure**. However, they differ fundamentally in how they make decisions. Understanding when to use which approach is a critical skill in algorithm design.

---

## Side-by-Side Comparison

```
┌──────────────────────────────┬──────────────────────────────┐
│         GREEDY               │     DYNAMIC PROGRAMMING      │
├──────────────────────────────┼──────────────────────────────┤
│                              │                              │
│  Makes ONE choice at each    │  Considers ALL choices at    │
│  step (the best-looking)     │  each step                   │
│                              │                              │
│  Never reconsiders choices   │  Combines results of all     │
│                              │  subproblems                 │
│                              │                              │
│  Choice BEFORE solving       │  Choice AFTER solving        │
│  subproblems                 │  subproblems                 │
│                              │                              │
│  Top-down approach only      │  Top-down or bottom-up       │
│                              │                              │
│  Usually O(n log n)          │  Usually O(n²) or O(n·W)     │
│                              │                              │
│  Requires: Greedy choice     │  Requires: Optimal           │
│  property + Optimal          │  substructure +              │
│  substructure                │  Overlapping subproblems     │
│                              │                              │
│  Simple implementation       │  More complex (table/memo)   │
│                              │                              │
│  Less memory                 │  More memory (DP table)      │
│                              │                              │
└──────────────────────────────┴──────────────────────────────┘
```

---

## Decision-Making Process

```
GREEDY:                          DP:
═══════                          ════

Step 1: Look at options          Step 1: Look at options
Step 2: Pick the BEST one        Step 2: Try ALL of them
Step 3: Move to subproblem       Step 3: Solve each subproblem
Step 4: Repeat                   Step 4: Pick the best OVERALL
                                 Step 5: Memoize results

     ┌─── Problem ───┐               ┌─── Problem ───┐
     │   /   |   \    │               │   /   |   \    │
     │  A    B    C   │               │  A    B    C   │
     │  ▲             │               │  │    │    │   │
     │  │ Pick best!  │               │  ▼    ▼    ▼   │
     │  └─ Done       │               │ sub  sub  sub  │
     └────────────────┘               │  │    │    │   │
                                      │  ▼    ▼    ▼   │
                                      │ Pick best of   │
                                      │ all results    │
                                      └────────────────┘
```

---

## Same Problem, Different Approaches

### Coin Change Problem

**Problem:** Make change for amount N using minimum coins.

#### With Standard Denominations {1, 5, 10, 25}

```
Target: 30 cents

GREEDY: Pick largest coin first
  30 → 25¢ → 5¢ → 5¢ → 0    Result: {25, 5} = 2 coins ✓ OPTIMAL

DP: Build table bottom-up
  dp[0]=0, dp[1]=1, dp[2]=2, ..., dp[5]=1, ..., dp[10]=1,
  ..., dp[25]=1, dp[26]=2, ..., dp[30]=2  Result: 2 coins ✓ OPTIMAL

Both give optimal! But greedy is O(1) per coin, DP is O(n·k).
```

#### With Arbitrary Denominations {1, 3, 4}

```
Target: 6

GREEDY: 6→4→1→1 = {4,1,1} = 3 coins  ✗ NOT OPTIMAL

DP Table:
  Amount:  0  1  2  3  4  5  6
  dp[i]:   0  1  2  1  1  2  2
  
  dp[6] = min(dp[6-1]+1, dp[6-3]+1, dp[6-4]+1)
        = min(dp[5]+1, dp[3]+1, dp[2]+1)
        = min(3, 2, 3) = 2
  
  Result: {3, 3} = 2 coins  ✓ OPTIMAL

DP wins! Greedy fails because it can't "look ahead."
```

---

### Knapsack Problem

```
Items: A(w=2,v=12), B(w=1,v=10), C(w=3,v=20), D(w=2,v=15)
Capacity: 5

╔═══════════════════════════════════════════════════════════╗
║  FRACTIONAL KNAPSACK (Greedy Works ✓)                    ║
╠═══════════════════════════════════════════════════════════╣
║  Ratios: A=6, B=10, C=6.67, D=7.5                       ║
║  Sort by ratio: B(10) > D(7.5) > C(6.67) > A(6)         ║
║  Pick B(w=1) → cap=4 → Pick D(w=2) → cap=2              ║
║  → Pick 2/3 of C(w=2) → cap=0                           ║
║  Value = 10 + 15 + (2/3)×20 = 38.33 ✓                   ║
╠═══════════════════════════════════════════════════════════╣
║  0/1 KNAPSACK (Need DP ✗ for Greedy)                     ║
╠═══════════════════════════════════════════════════════════╣
║  Greedy by ratio: B + D + ? (can't fit C or A) = 25     ║
║  Greedy by ratio: B(1) + D(2) = cap 3, A(2) fits!       ║
║  B + D + A = 10 + 15 + 12 = 37, weight = 5 ✓            ║
║  But is it optimal?                                       ║
║  Check: B + C = 10+20 = 30 (weight 4)                    ║
║  Check: D + C = 15+20 = 35 (weight 5)                    ║
║  Check: B + D + A = 37 (weight 5) ← Optimal!             ║
║                                                           ║
║  DP approach would systematically find this.              ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Required Properties Comparison

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│           Greedy              DP                        │
│         ┌────────┐         ┌──────────┐                │
│         │Greedy  │         │Overlapping│                │
│         │Choice  │         │Subproblems│                │
│         │Property│         │           │                │
│         └───┬────┘         └─────┬─────┘                │
│             │                    │                       │
│         ┌───┴────────────────────┴───┐                  │
│         │    Optimal Substructure    │                  │
│         │      (SHARED)              │                  │
│         └────────────────────────────┘                  │
│                                                         │
│  Greedy needs:                DP needs:                 │
│  ✓ Greedy choice property    ✓ Overlapping subproblems │
│  ✓ Optimal substructure      ✓ Optimal substructure    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## When to Choose Which

```
┌────────────────────────────────────────────────────────┐
│              DECISION GUIDE                            │
│                                                        │
│  START: Can you identify a greedy choice?              │
│         │                                              │
│    ┌────┴────┐                                         │
│    │YES      │NO                                       │
│    ▼         ▼                                         │
│  Can you    Use DP                                     │
│  PROVE it   directly                                   │
│  works?                                                │
│    │                                                   │
│  ┌─┴──┐                                               │
│  │YES │NO                                              │
│  ▼    ▼                                                │
│ USE   Can you find                                     │
│GREEDY counterexample?                                  │
│       │                                                │
│   ┌───┴───┐                                            │
│   │YES    │NO                                          │
│   ▼       ▼                                            │
│  Use DP   Greedy is                                    │
│           probably OK                                  │
│           (test more)                                  │
└────────────────────────────────────────────────────────┘
```

---

## Problem Classification Table

| Problem | Greedy? | DP? | Best Approach | Why |
|---------|---------|-----|---------------|-----|
| Activity Selection | ✓ | ✓ | Greedy | Greedy is simpler and faster |
| 0/1 Knapsack | ✗ | ✓ | DP | No greedy choice property |
| Fractional Knapsack | ✓ | ✓ | Greedy | Greedy is O(n log n) vs DP O(nW) |
| Coin Change (std) | ✓ | ✓ | Greedy | Simpler, O(1) per coin |
| Coin Change (arb) | ✗ | ✓ | DP | Greedy fails on arbitrary denominations |
| Huffman Coding | ✓ | ✗ | Greedy | Natural greedy structure |
| LCS | ✗ | ✓ | DP | No greedy strategy exists |
| LIS | ✗ | ✓ | DP + Binary Search | Greedy gives wrong order |
| MST | ✓ | ✗ | Greedy | Matroid structure |
| Shortest Path (+ve) | ✓ | ✓ | Greedy (Dijkstra) | Faster than DP Bellman-Ford |
| Shortest Path (-ve) | ✗ | ✓ | DP (Bellman-Ford) | Negative edges break greedy |
| Edit Distance | ✗ | ✓ | DP | Complex character-level dependencies |
| Rod Cutting | ✗ | ✓ | DP | Must consider all cut positions |
| Matrix Chain | ✗ | ✓ | DP | Parenthesization order matters globally |

---

## Complexity Comparison

```
┌──────────────────────────────────────────────────┐
│  TYPICAL COMPLEXITIES                            │
│                                                  │
│  GREEDY:                                         │
│  Time:  O(n log n) [sort] + O(n) [scan] = O(n log n)│
│  Space: O(1) or O(n)                             │
│                                                  │
│  DP:                                             │
│  Time:  O(n²), O(n·W), O(n·m) etc.             │
│  Space: O(n), O(n·W), O(n·m) etc.              │
│                                                  │
│  KEY RULE:                                       │
│  If greedy works → ALWAYS prefer it over DP      │
│  It's faster, simpler, and uses less memory      │
└──────────────────────────────────────────────────┘
```

---

## Hybrid Approaches

Sometimes the best solution uses both:

```
GREEDY + DP HYBRIDS:
════════════════════

1. Greedy for initial solution, DP for refinement
2. Greedy choice for one dimension, DP for another
3. Greedy to set bounds, DP to find exact optimum

Example: Weighted Job Scheduling
  - Sort by finish time (greedy-style preprocessing)
  - Use DP to find optimal subset: 
    dp[i] = max(profit[i] + dp[latest_non-conflict], dp[i-1])
```

---

## Summary Table

| Aspect | Greedy | Dynamic Programming |
|--------|--------|-------------------|
| **Decision** | One choice per step | All choices explored |
| **Timing** | Choice before subproblems | Choice after subproblems |
| **Properties** | Greedy choice + Opt. substructure | Overlapping subproblems + Opt. substructure |
| **Time** | O(n log n) typical | O(n²) or more typical |
| **Space** | O(1)-O(n) | O(n)-O(n²) |
| **Implementation** | Simple | Moderate-Complex |
| **Correctness** | Must prove carefully | Always correct if formulated right |
| **When to Use** | When greedy choice property holds | When choices have dependencies |

---

## Quick Revision Questions

1. **What is the key difference in how greedy and DP make decisions?**
   > Greedy makes one irreversible choice per step before solving subproblems. DP considers all possible choices and picks the best after solving all subproblems.

2. **Both greedy and DP require optimal substructure. What additional property does each need?**
   > Greedy needs the greedy choice property. DP needs overlapping subproblems.

3. **If both greedy and DP can solve a problem, which should you prefer and why?**
   > Greedy, because it's typically faster (O(n log n) vs O(n²)) and simpler to implement.

4. **Why can DP solve problems that greedy cannot?**
   > DP explores all possible choices and compares them, so it can find the optimal even when the locally best choice isn't globally best.

5. **Give an example where the same problem type can be greedy or DP depending on constraints.**
   > Coin change: greedy works with standard denominations {1, 5, 10, 25} but needs DP with arbitrary denominations {1, 3, 4}.

6. **Can a greedy algorithm be thought of as a special case of DP?**
   > Yes, in a sense. Greedy can be viewed as DP where only one subproblem needs to be solved at each step (the one after the greedy choice), making the DP table degenerate into a single path.

---

[← Previous: When Greedy Fails](05-when-greedy-fails.md)

[← Back to README](../README.md)
