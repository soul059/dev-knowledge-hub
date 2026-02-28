# Chapter 6: When to Apply Greedy

## 📋 Chapter Overview
Recognize when greedy works, when it fails, and how to verify your greedy intuition.

---

## 🔍 Greedy Signal Checklist

```
  ✓ Problem asks for OPTIMAL (min/max) or FEASIBILITY
  ✓ At each step, a local best choice is clear
  ✓ Never need to reconsider past decisions
  ✓ Can prove greedy choice property (exchange argument)
  ✓ Sorting the input reveals a natural processing order
  ✓ Constraint is simple (one dimension to optimize)
```

---

## 🧭 Decision Flowchart

```
  Is there a clear "locally best" choice?
  │
  ├─ YES ──▶ Does taking it ever hurt future choices?
  │           │
  │           ├─ NO  ──▶ Greedy ✅ (verify with examples)
  │           │
  │           └─ YES ──▶ Need DP (subproblem overlap) or
  │                      Backtracking (explore all)
  │
  └─ NO ───▶ Not greedy. Try DP / BFS / other approach.
```

---

## ✅ Greedy Works — Examples

| Problem | Why Greedy Works |
|---------|-----------------|
| Activity selection | Earliest finish leaves max room — exchange-provable |
| Fractional knapsack | Best ratio first — can take fractions |
| Huffman coding | Merge cheapest — optimal substructure proved |
| Jump Game I/II | Farthest reach only grows — monotonic |
| Gas Station | Deficit forces reset — total feasibility guarantees answer |
| Merge intervals | Sorted scan covers all overlaps |

---

## ❌ Greedy Fails — Examples

| Problem | Why Greedy Fails | Use Instead |
|---------|-----------------|-------------|
| 0/1 Knapsack | Can't take fractions; ratio order fails | DP |
| Coin change (general) | Largest first may overshoot | DP |
| Longest path (general graph) | Local longest edge ≠ global longest path | DP / DFS |
| TSP | Nearest neighbor heuristic is suboptimal | DP + bitmask |
| Edit distance | No clear local choice | DP |

---

## 🔑 Verification Techniques

### 1. Check Small Counter-examples

```
  Before coding, test greedy on:
  • Smallest non-trivial input (n=3-5)
  • Edge cases: all same, sorted, reverse sorted
  • Known tricky inputs from the problem
  
  If greedy gives wrong answer on ANY → it doesn't work.
```

### 2. Exchange Argument Sketch

```
  1. Take any optimal solution O
  2. If O ≠ greedy G, find first difference
  3. Argue: swapping O's choice for G's choice doesn't worsen O
  4. If argument holds → greedy is correct
```

### 3. Common Pitfalls

```
  • "Sort by X" doesn't mean X is right — try different sort keys
  • Greedy on one metric may violate another constraint
  • Works for special cases ≠ works always
  • Greedy ≠ brute force + pruning
```

---

## 🆚 Greedy vs Alternatives

| Criterion | Greedy | DP | Backtracking |
|-----------|--------|-----|-------------|
| Explores | Single path | All subproblems | All solutions |
| Reconsiders | Never | Via recurrence | Via undo |
| Correctness | Must prove | Always (if correct recurrence) | Always |
| Speed | O(n log n) typical | O(n²) or O(n×W) | Exponential |
| When to use | Provable local = global | Count ways / optimize | Find all solutions |

---

## 📊 Interview Decision Guide

```
  1. Read problem → identify: optimize / count / enumerate?
  2. If OPTIMIZE:
     a. Is there a natural sort order?
     b. Does greedy pass 3+ test cases manually?
     c. Can you sketch an exchange argument?
     → YES to all → implement greedy
     → NO to any → try DP
  3. If COUNT: almost always DP
  4. If ENUMERATE ALL: backtracking
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Greedy signals | Clear local best, no reconsideration, sortable |
| Verification | Small counter-examples + exchange argument |
| Greedy fails | When local choice can hurt future (knapsack, coin change) |
| Interview flow | Check greedy first (simpler), fall back to DP |

---

## ❓ Revision Questions

1. **How to quickly test if greedy works?** → Try 3+ small inputs manually; if any gives wrong answer, greedy fails.
2. **Exchange argument: purpose?** → Formally proves that greedy choice is at least as good as any alternative.
3. **Greedy vs DP: main difference?** → Greedy makes one irrevocable choice per step; DP considers all subproblems.
4. **When is greedy faster than DP?** → Almost always — typically O(n log n) vs O(n²) or O(n×W).
5. **Coin change: why greedy fails for {1,3,4}?** → Greedy picks 4, then 1+1 = 3 coins. DP finds 3+3 = 2 coins. Local best ≠ global best.
6. **Should you always try greedy first in interviews?** → Yes, if applicable — it's simpler and faster. Mention why it works (or doesn't) to show understanding.

---

[← Previous: Classic Problems](05-classic-problems.md) | [Next: Stack/Queue Fundamentals →](../09-Stack-Queue-Pattern/01-stack-fundamentals.md)
