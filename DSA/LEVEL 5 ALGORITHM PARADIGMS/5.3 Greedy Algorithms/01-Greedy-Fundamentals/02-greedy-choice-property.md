# Chapter 2: Greedy Choice Property

## Overview
The **Greedy Choice Property** is one of the two essential ingredients for a correct greedy algorithm. It states that a globally optimal solution can be reached by making a **locally optimal (greedy) choice** at each step. In other words, we can assemble an optimal solution by choosing what looks best right now, without worrying about future consequences.

---

## Formal Definition

> **Greedy Choice Property:** A problem exhibits the greedy choice property if an optimal solution can be constructed by making the locally optimal choice at each step, without reconsidering previous choices.

```
┌────────────────────────────────────────────────────────────┐
│                GREEDY CHOICE PROPERTY                      │
│                                                            │
│   "There exists an optimal solution that includes the      │
│    greedy choice made at the first step."                   │
│                                                            │
│   Optimal Solution = Greedy Choice + Optimal Subproblem    │
│                                                            │
│   ┌──────┐   ┌──────────┐   ┌──────────────────────┐      │
│   │Greedy│ + │Remaining │ = │ Globally Optimal     │      │
│   │Choice│   │Subproblem│   │ Solution             │      │
│   └──────┘   └──────────┘   └──────────────────────┘      │
└────────────────────────────────────────────────────────────┘
```

---

## Intuition: Why Does It Work?

The key insight is:

> If making the **best local choice** at step 1 doesn't prevent us from finding the optimal solution for the remaining problem, then we can safely commit to that choice.

```
Decision Tree View:

                    Problem
                   /   |   \
                 c1    c2    c3      ← All possible first choices
                / \   / \   / \
              ...  ... ... ... ...   ← Sub-decisions
              
Greedy picks ONE path (e.g., c1) and never explores others.

For greedy to be correct:
  The path through c1 MUST lead to an optimal leaf.
```

---

## Example 1: Coin Change (Standard Denominations)

**Denominations:** {25, 10, 5, 1}  
**Target:** 30 cents

**Claim:** The greedy choice (pick largest coin ≤ remaining) is always part of an optimal solution.

```
Greedy trace:
  30¢ → pick 25¢ → 5¢ remaining
   5¢ → pick  5¢ → 0¢ remaining
   
Result: {25, 5} = 2 coins

Why greedy choice property holds:
┌────────────────────────────────────────────────────────┐
│ Suppose optimal solution does NOT use 25¢ for 30¢.    │
│                                                        │
│ Then it must use: 3×10¢ = 3 coins, or                 │
│                   2×10¢ + 2×5¢ = 4 coins, or          │
│                   ... (all worse)                       │
│                                                        │
│ So any optimal solution CAN include 25¢.               │
│ → Greedy choice property holds ✓                       │
└────────────────────────────────────────────────────────┘
```

---

## Example 2: Activity Selection

**Greedy Choice:** Pick the activity with the **earliest finish time**.

```
Why this choice is safe:

Suppose OPT is any optimal solution.
Let a* = activity with earliest finish time.
Let a_k = first activity chosen in OPT.

Case 1: a* = a_k  → OPT already includes our greedy choice ✓

Case 2: a* ≠ a_k  → 
  Since finish(a*) ≤ finish(a_k),
  we can REPLACE a_k with a* in OPT.
  
  ┌──── OPT: ──[a_k]──[a_2]──[a_3]──...──[a_n] ────┐
  │                                                    │
  │  Replace a_k with a*:                              │
  │                                                    │
  │  OPT': ──[a*]──[a_2]──[a_3]──...──[a_n] ────     │
  │                                                    │
  │  Since finish(a*) ≤ finish(a_k), a* doesn't       │
  │  conflict with a_2, so OPT' is also optimal! ✓    │
  └────────────────────────────────────────────────────┘
```

---

## The Proof Pattern

To prove the greedy choice property, follow this template:

```
PROOF TEMPLATE:
═══════════════

1. Let OPT be any optimal solution
2. Let g be the greedy choice
3. Show that OPT can be MODIFIED to include g
   without decreasing quality
4. Conclude: there exists an optimal solution
   containing the greedy choice

     ┌─────────────────┐
     │  Any Optimal     │
     │  Solution (OPT)  │
     └────────┬─────────┘
              │ Modify
              ▼
     ┌─────────────────┐
     │  Optimal Solution│
     │  containing      │
     │  greedy choice   │
     └─────────────────┘
```

---

## Greedy Choice Property vs No Greedy Choice Property

### HAS Greedy Choice Property ✓

| Problem | Greedy Choice | Why It Works |
|---------|--------------|--------------|
| Activity Selection | Earliest finish time | Frees up most time for remaining activities |
| Fractional Knapsack | Highest value/weight | Taking best ratio maximizes value |
| Huffman Coding | Merge two lowest frequencies | Pushes rare symbols deeper in tree |
| Coin Change (std) | Largest denomination | Minimizes number of coins needed |

### LACKS Greedy Choice Property ✗

| Problem | Tempting Greedy Choice | Why It Fails |
|---------|----------------------|--------------|
| 0/1 Knapsack | Highest value/weight | Can't take fractions, may miss combos |
| Coin Change (arbitrary) | Largest denomination | May not lead to minimum coins |
| Longest Path | Heaviest edge | May lead to dead ends |
| Traveling Salesman | Nearest neighbor | May create poor global tour |

---

## Key Differences from Dynamic Programming

```
┌──────────────────────────┬──────────────────────────┐
│       GREEDY             │    DYNAMIC PROGRAMMING   │
├──────────────────────────┼──────────────────────────┤
│ Makes ONE choice per     │ Considers ALL choices    │
│ subproblem               │ per subproblem           │
│                          │                          │
│ Choice BEFORE solving    │ Choice AFTER solving     │
│ subproblems              │ subproblems              │
│                          │                          │
│ Top-down only            │ Top-down or bottom-up    │
│                          │                          │
│ Greedy choice property   │ Optimal substructure     │
│ REQUIRED                 │ REQUIRED                 │
│                          │                          │
│ Faster (usually)         │ Slower (usually)         │
└──────────────────────────┴──────────────────────────┘
```

---

## How to Identify Greedy Choice Property

```
┌───────────────────────────────────────────────────────┐
│          IDENTIFICATION CHECKLIST                     │
│                                                       │
│  Ask yourself:                                        │
│                                                       │
│  □ Can I identify a "best" first choice?              │
│  □ Does making this choice leave a smaller            │
│    subproblem of the same type?                       │
│  □ Can I show no optimal solution is "hurt"           │
│    by including this choice?                          │
│  □ Can I swap any other first choice with mine        │
│    without making things worse?                       │
│                                                       │
│  If YES to all → Greedy Choice Property likely holds  │
└───────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Definition** | Locally optimal choice leads to globally optimal solution |
| **Key Idea** | Safe to commit to the "best" choice right now |
| **Proof Method** | Show optimal solution can be modified to include greedy choice |
| **Required For** | Correct greedy algorithms |
| **Combined With** | Optimal substructure (both needed for greedy correctness) |
| **Contrast With** | DP where all subproblem solutions are considered |

---

## Quick Revision Questions

1. **What does the greedy choice property guarantee?**
   > That an optimal solution can be constructed starting with the locally optimal choice at each step.

2. **How do you prove the greedy choice property for a problem?**
   > Show that any optimal solution can be modified to include the greedy choice without decreasing the quality of the solution.

3. **What is the "exchange argument" in the context of greedy choice property?**
   > Replacing a non-greedy choice in an optimal solution with the greedy choice and showing the solution remains optimal (or improves).

4. **Does coin change always have the greedy choice property?**
   > No. It works for standard denominations (1, 5, 10, 25) but fails for arbitrary denominations (e.g., {1, 3, 4} for target 6).

5. **In activity selection, why is "earliest finish time" the greedy choice?**
   > Because finishing earliest leaves maximum room for other activities, and any optimal solution can be modified to include this choice.

6. **What happens if a problem lacks the greedy choice property?**
   > The greedy algorithm may produce a suboptimal solution, and a different approach (like dynamic programming) is needed.

---

[← Previous: What is Greedy Algorithm?](01-what-is-greedy-algorithm.md) | [Next: Optimal Substructure →](03-optimal-substructure.md)

[← Back to README](../README.md)
