# Chapter 6: When to Use Dynamic Programming

## 📋 Overview
Not every problem can or should be solved with DP. This chapter provides a systematic framework for identifying when DP is the right approach and recognizing common signals in problem statements.

---

## 🧠 Decision Framework

```
┌─────────────────────────────────────────────────────────┐
│           SHOULD I USE DP? — Decision Tree              │
│                                                         │
│   Is it an optimization problem                         │
│   (min/max/count)?                                      │
│        │                                                │
│        ├── YES ──► Does it have optimal substructure?    │
│        │                │                               │
│        │                ├── YES ──► Are there overlapping│
│        │                │          subproblems?          │
│        │                │              │                 │
│        │                │              ├── YES ──► USE DP│
│        │                │              │                 │
│        │                │              └── NO ──► Use    │
│        │                │                   Divide &    │
│        │                │                   Conquer     │
│        │                │                               │
│        │                └── NO ──► DP won't give        │
│        │                          optimal answer        │
│        │                                                │
│        └── NO ──► Can you define states & transitions?  │
│                        │                                │
│                        ├── YES ──► Still might use DP   │
│                        └── NO ──► Try other approaches  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Key Signals in Problem Statements

### Signal Words That Suggest DP

```
┌─────────────────────┬────────────────────────────────────┐
│  Signal Word/Phrase │  Likely DP Pattern                 │
├─────────────────────┼────────────────────────────────────┤
│  "minimum cost"     │  Optimization DP                   │
│  "maximum profit"   │  Optimization DP                   │
│  "count the ways"   │  Counting DP                       │
│  "number of paths"  │  Grid DP / Counting                │
│  "is it possible?"  │  Boolean DP (subset sum type)      │
│  "longest/shortest" │  Sequence DP                       │
│  "partition into"   │  Knapsack variant                  │
│  "divide into"      │  Interval DP or Knapsack           │
│  "subsequence"      │  Sequence DP (LCS/LIS)             │
│  "choices at each"  │  State Machine DP                  │
│  "with constraint"  │  Add constraint to state           │
│  "all subsets"      │  Bitmask DP (if n ≤ 20)           │
└─────────────────────┴────────────────────────────────────┘
```

### Signal Constraints

```
┌──────────────────────┬───────────────────────────────────┐
│  Constraint Range    │  Likely Approach                  │
├──────────────────────┼───────────────────────────────────┤
│  n ≤ 20             │  Bitmask DP or Backtracking       │
│  n ≤ 100            │  O(n³) DP (Interval DP)           │
│  n ≤ 1,000          │  O(n²) DP                         │
│  n ≤ 10,000         │  O(n²) or O(n log n) DP           │
│  n ≤ 100,000        │  O(n log n) or O(n) DP            │
│  n ≤ 1,000,000      │  O(n) DP or Greedy                │
│  n ≤ 10⁹           │  Math / Matrix Expo / Greedy       │
└──────────────────────┴───────────────────────────────────┘
```

---

## 📐 DP vs Other Approaches

### DP vs Greedy

```
┌─────────────────────────────────────────────────────────┐
│  GREEDY: Make locally optimal choice at each step       │
│  DP:     Consider ALL choices and pick globally optimal │
│                                                         │
│  Example: Coin Change                                   │
│  Coins = [1, 3, 4], Amount = 6                          │
│                                                         │
│  GREEDY: Pick largest first                             │
│    6 - 4 = 2, 2 - 1 = 1, 1 - 1 = 0                    │
│    Coins used: 4, 1, 1 → 3 coins                       │
│                                                         │
│  DP: Consider all options                               │
│    dp[6] = min(dp[6-1]+1, dp[6-3]+1, dp[6-4]+1)       │
│    3, 3 → 2 coins ✓ (BETTER!)                          │
│                                                         │
│  Greedy FAILS when local optimal ≠ global optimal      │
└─────────────────────────────────────────────────────────┘
```

### DP vs Brute Force (Backtracking)

```
┌─────────────────────────────────────────────────────────┐
│  BACKTRACKING: Try all possibilities, prune invalid     │
│  DP: Store results of subproblems to avoid recomputing  │
│                                                         │
│  When to choose what:                                   │
│                                                         │
│  Backtracking when:                 DP when:            │
│  • Need ALL solutions (not count)   • Need optimal/count│
│  • State hard to define compactly   • Clear state def   │
│  • Few valid paths in search tree   • Many overlapping  │
│  • Generation problems              • Optimization      │
└─────────────────────────────────────────────────────────┘
```

### DP vs Divide and Conquer

```
┌─────────────────────────────────────────────────────────┐
│  D&C: Subproblems are INDEPENDENT (no overlap)          │
│  DP:  Subproblems OVERLAP (shared computation)          │
│                                                         │
│  Merge Sort (D&C):                                      │
│    Left half and right half are independent              │
│    No benefit from caching                               │
│                                                         │
│  Matrix Chain (DP):                                     │
│    M(1,3) and M(2,4) both need M(2,3)                  │
│    Caching M(2,3) saves computation                     │
└─────────────────────────────────────────────────────────┘
```

---

## 🧪 Checklist: Before Applying DP

```
□ Can I define the problem recursively?
    └── What are the parameters that change?
    
□ Does the problem have overlapping subproblems?
    └── Does the recursion tree have repeated nodes?
    
□ Does the problem have optimal substructure?
    └── Does the best answer use best sub-answers?
    
□ Can I define states clearly?
    └── What information do I need at each step?
    
□ Can I write a recurrence relation?
    └── How does dp[current] relate to dp[previous]?
    
□ Can I identify base cases?
    └── What are the smallest problems I can solve directly?
    
□ Is the state space manageable?
    └── Will the DP table fit in memory?
```

---

## 🏗️ Problem Classification

```
┌──────────────────────────────────────────────────────────┐
│                   DP PROBLEM TYPES                       │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │ OPTIMIZATION │  │   COUNTING   │  │   EXISTENCE    │  │
│  │ min/max cost │  │ # of ways    │  │ is it possible?│  │
│  │ min/max path │  │ # of paths   │  │ can we reach?  │  │
│  │ min/max value│  │ # of subsets │  │ true/false     │  │
│  └──────┬──────┘  └──────┬───────┘  └───────┬────────┘  │
│         │                │                   │           │
│         └────────────────┼───────────────────┘           │
│                          │                               │
│                   All solved with DP!                    │
│                                                          │
│  Recurrence uses:                                        │
│    Optimization → min() or max()                         │
│    Counting     → sum (+)                                │
│    Existence    → OR (||)                                │
└──────────────────────────────────────────────────────────┘
```

---

## 💡 Common Mistakes

```
┌─────────────────────────────────────────────────────────┐
│  MISTAKE 1: Using DP when Greedy works                  │
│  Example: Activity Selection → Greedy is O(n log n)     │
│           DP would be O(n²) unnecessarily               │
│                                                         │
│  MISTAKE 2: Missing a state variable                    │
│  Example: Stock with cooldown                           │
│    Wrong: dp[i] = max profit at day i                   │
│    Right: dp[i][holding] = max profit at day i          │
│           being in state holding/not-holding             │
│                                                         │
│  MISTAKE 3: Wrong base cases                            │
│  Example: Coin change                                   │
│    Wrong: dp[0] = 1                                     │
│    Right: dp[0] = 0 (min coins for amount 0)            │
│                                                         │
│  MISTAKE 4: Wrong fill order                            │
│  Computing dp[i] before its dependencies are ready      │
│                                                         │
│  MISTAKE 5: Forgetting that DP needs BOTH properties    │
│  Overlapping subproblems alone isn't enough              │
└─────────────────────────────────────────────────────────┘
```

---

## 🔄 Quick Pattern Recognition

```
Problem asks for...          →  Try this DP pattern
───────────────────────────────────────────────────
"Nth number in sequence"     →  1D DP (Fibonacci-like)
"Min/max in linear array"   →  1D DP (Kadane's/House Robber)
"Paths in a grid"            →  2D Grid DP
"Comparing two strings"      →  2D String DP (LCS/Edit Dist.)
"Choose items with capacity" →  Knapsack DP
"Longest increasing..."      →  LIS pattern
"Split array optimally"      →  Interval DP
"Buy/sell with rules"        →  State Machine DP
"Visit all nodes/cities"     →  Bitmask DP
"Palindrome related"         →  Palindrome DP
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Use DP When** | Overlapping subproblems + Optimal substructure |
| **Signal Words** | min, max, count, ways, possible, longest, shortest |
| **DP vs Greedy** | DP explores all; Greedy picks locally optimal |
| **DP vs D&C** | DP has overlapping subproblems; D&C doesn't |
| **DP vs Backtracking** | DP for optimal/count; Backtracking for all solutions |
| **Three DP Types** | Optimization (min/max), Counting (sum), Existence (or) |

---

## ❓ Quick Revision Questions

1. **List three signal words in a problem statement that suggest DP.**
2. **When would Greedy be preferred over DP?**
3. **A problem has optimal substructure but NO overlapping subproblems. What approach should you use?**
4. **How does the constraint `n ≤ 20` hint at a specific DP technique?**
5. **What are the three types of DP problems (by what they ask for)?**
6. **Give an example where a Greedy approach gives the wrong answer but DP gives the correct one.**

---

[← Previous: Tabulation](05-tabulation.md) | [Next Unit: DP Approach →](../02-DP-Approach/01-identify-problem-type.md)

[← Back to README](../README.md)
