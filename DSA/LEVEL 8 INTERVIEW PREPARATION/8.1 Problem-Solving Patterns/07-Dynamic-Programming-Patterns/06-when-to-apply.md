# Chapter 6: When to Apply DP

## 📋 Chapter Overview
How to recognize DP problems, choose the right pattern, and avoid common traps.

---

## 🔍 Signal Checklist

```
  ✓ Ask for OPTIMAL value (min, max, longest, shortest)
  ✓ Ask for COUNT of ways
  ✓ Problem has overlapping subproblems (same inputs recur)
  ✓ Problem has optimal substructure (optimal solution built from optimal sub-solutions)
  ✓ Choices at each step affect future options
  ✓ Brute force is exponential, but states are polynomial
```

---

## 🧭 Decision Flowchart

```
  Problem asks for optimal / count?
  │
  ├─ YES ──▶ Can you define state + recurrence?
  │           │
  │           ├─ YES ──▶ Overlapping subproblems?
  │           │           │
  │           │           ├─ YES ──▶ DP ✅
  │           │           │
  │           │           └─ NO ───▶ Divide & Conquer
  │           │
  │           └─ NO ───▶ Try Greedy first
  │                       (verify greedy choice property)
  │
  └─ NO ──▶ Ask for ALL solutions? → Backtracking
            Ask for existence? → Could be DP/Greedy/BFS
```

---

## 🎯 Pattern-to-Problem Mapping

| Pattern | Signal Keywords | Examples |
|---------|----------------|----------|
| 1D DP (linear) | "sequence", "steps", "days" | Climbing stairs, house robber, decode ways |
| 2D DP (grid) | "grid", "matrix", "paths" | Unique paths, min path sum, dungeon game |
| 2D DP (two strings) | "transform", "subsequence", "edit" | LCS, edit distance, interleaving |
| Knapsack | "weight/capacity", "subset", "limit" | 0/1 knapsack, partition equal subset |
| Interval DP | "merge", "burst", "parenthesise" | Burst balloons, matrix chain, palindrome partition |
| State machine | "cooldown", "fee", "at most k" | Stock buy/sell variants |
| Bitmask | "assign", "all subsets", small n | TSP, partition into K subsets |

---

## 🔄 DP vs Greedy

```
  Greedy: locally optimal → globally optimal (must PROVE)
  DP:     explore ALL subproblems, take global optimal
  
  If greedy choice property holds → Greedy (simpler)
  If not → need DP
  
  Example: Coin change
    Greedy works for {1, 5, 10, 25} but FAILS for {1, 3, 4} target 6
    DP always works: dp[6] = min(dp[3]+1, dp[2]+1) = 2
```

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong state definition | State must capture ALL information needed for future decisions |
| Missing base case | Verify dp[0] or dp[0][0] before filling |
| Wrong iteration order | Dependencies must be computed before used |
| Off-by-one | Check bounds carefully: does i start at 0 or 1? |
| Not considering empty | What if string is empty, array is size 0? |
| Space overflow | Use rolling array or bitmask size check |

---

## 📝 5-Step DP Problem-Solving Template

```
  1. DEFINE state: dp[i] = ...    (or dp[i][j] = ...)
  2. IDENTIFY recurrence: dp[i] = f(dp[earlier states])
  3. SET base cases: dp[0] = ..., dp[1] = ...
  4. DETERMINE iteration order: ensure dependencies ready
  5. EXTRACT answer: dp[n], dp[m][n], max(dp[...])
```

---

## 📊 Complexity Guide

| DP Type | Typical Time | Typical Space |
|---------|-------------|---------------|
| 1D | O(n) or O(n²) | O(n) |
| 2D grid | O(m×n) | O(m×n) → O(n) |
| 2D strings | O(m×n) | O(m×n) → O(n) |
| Knapsack | O(n×W) | O(n×W) → O(W) |
| Interval | O(n³) | O(n²) |
| Bitmask | O(2^n × n) | O(2^n) |
| Matrix exp | O(k³ log n) | O(k²) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Recognize DP | Optimal/count + overlapping subproblems |
| DP vs Greedy | DP when greedy property unproven |
| State design | Must capture all info for future decisions |
| 5-step template | State → Recurrence → Base → Order → Answer |
| Pattern matching | Use keywords to pick 1D/2D/knapsack/interval |

---

## ❓ Revision Questions

1. **Two properties of DP problems?** → Optimal substructure and overlapping subproblems.
2. **When to prefer Greedy over DP?** → When greedy choice property can be proven (local optimal = global optimal).
3. **How to define state for string problems?** → dp[i][j] = answer for text1[0..i-1] and text2[0..j-1].
4. **Interval DP iteration order?** → By increasing interval length (gap), so smaller intervals computed first.
5. **Why does greedy fail for coins {1,3,4} target 6?** → Greedy picks 4+1+1=3 coins; optimal is 3+3=2 coins.

---

[← Previous: DP Optimization](05-dp-optimization.md) | [Next: Greedy Fundamentals →](../08-Greedy-Pattern/01-greedy-fundamentals.md)
