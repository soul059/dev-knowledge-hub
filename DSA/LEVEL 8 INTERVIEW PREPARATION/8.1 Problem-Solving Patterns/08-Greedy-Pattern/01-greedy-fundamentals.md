# Chapter 1: Greedy Fundamentals

## 📋 Chapter Overview
Understand the greedy paradigm: make the locally optimal choice at each step and prove it leads to a global optimum.

---

## 🧠 Core Concept

```
  Greedy Algorithm Structure:
  ┌──────────────────────────────────────┐
  │  1. Sort / arrange candidates        │
  │  2. Iterate through candidates       │
  │  3. At each step, pick the BEST      │
  │     available choice (greedy choice) │
  │  4. Never reconsider / backtrack     │
  └──────────────────────────────────────┘
  
  Key: once a choice is made, it's final.
```

---

## 📐 Two Properties Required

### 1. Greedy Choice Property
> A globally optimal solution can be arrived at by making a locally optimal choice.

### 2. Optimal Substructure
> An optimal solution contains optimal solutions to subproblems (shared with DP).

```
  Greedy = Greedy Choice Property + Optimal Substructure
  DP     = Overlapping Subproblems + Optimal Substructure
  
  If greedy choice property holds → use Greedy (simpler, faster)
  If not → fall back to DP
```

---

## 📝 Example 1: Activity Selection

**Problem:** Given `n` activities with start/finish times, select maximum non-overlapping activities.

```
  Activities (sorted by finish time):
  A: [1, 3]   B: [2, 5]   C: [4, 7]   D: [6, 8]   E: [5, 9]
  
  Greedy: always pick the activity that FINISHES EARLIEST
```

### Pseudocode

```
function activitySelection(activities):
    sort activities by finish time
    selected = [activities[0]]
    lastFinish = activities[0].finish
    
    for i = 1 to n-1:
        if activities[i].start >= lastFinish:
            selected.add(activities[i])
            lastFinish = activities[i].finish
    
    return selected
```

### Trace

```
  Sorted by finish: A[1,3]  B[2,5]  C[4,7]  D[6,8]  E[5,9]
  
  Step 1: Pick A[1,3].  lastFinish = 3
  Step 2: B[2,5] → start 2 < 3 → SKIP
  Step 3: C[4,7] → start 4 ≥ 3 → PICK. lastFinish = 7
  Step 4: D[6,8] → start 6 < 7 → SKIP
  Step 5: E[5,9] → start 5 < 7 → SKIP
  
  Result: {A, C} → 2 activities
  
  But wait, {A, D} also gives 2. Both optimal!
```

**Why greedy works:** Picking the earliest-finishing activity leaves maximum room for future activities. Proven by exchange argument.

---

## 📝 Example 2: Fractional Knapsack

**Problem:** Items have weight and value. Knapsack has capacity W. Can take fractions of items.

```
function fractionalKnapsack(items, W):
    sort items by value/weight ratio (descending)
    totalValue = 0
    
    for each item:
        if W >= item.weight:
            take entire item
            totalValue += item.value
            W -= item.weight
        else:
            take fraction W / item.weight
            totalValue += item.value * (W / item.weight)
            break
    
    return totalValue
```

### Trace

```
  Items: A(w=10,v=60), B(w=20,v=100), C(w=30,v=120)
  Capacity W = 50
  
  Ratios: A=6.0, B=5.0, C=4.0
  Sorted: A, B, C
  
  Take A: W=50-10=40, value=60
  Take B: W=40-20=20, value=60+100=160
  Take C: fraction 20/30 of C, value=160+120*(20/30)=160+80=240
  
  Total: 240
```

**Complexity:** O(n log n) for sort + O(n) scan = **O(n log n)**

---

## 🆚 Greedy vs DP vs Brute Force

```
  Problem: Coin Change (min coins for target)
  
  Coins = {1, 3, 4}, Target = 6
  
  Greedy: 4+1+1 = 3 coins ← WRONG (not optimal)
  DP:     3+3   = 2 coins ← CORRECT
  
  Greedy FAILS because greedy choice property doesn't hold!
```

| Approach | Time | Guarantees | Backtracks? |
|----------|------|-----------|-------------|
| Brute force | Exponential | Always correct | Explores all |
| Greedy | O(n log n) typically | Correct if property holds | Never |
| DP | Polynomial | Always correct | Explores all subproblems |

---

## 📐 Proving Greedy Correctness

### Exchange Argument (most common)

```
  1. Assume an optimal solution O that differs from greedy G
  2. Find the first place they differ
  3. Show you can SWAP O's choice for G's choice
     without making the solution worse
  4. Repeat until O = G → greedy is optimal
```

### Greedy Stays Ahead

```
  1. Show that after each step, greedy's partial solution
     is at least as good as any other algorithm's
  2. By induction, greedy's final solution is optimal
```

---

## 📋 Summary Table

| Concept | Key Point |
|---------|-----------|
| Greedy paradigm | Make locally optimal choice, never backtrack |
| Two properties | Greedy choice property + optimal substructure |
| Activity selection | Sort by finish time, greedily pick non-overlapping |
| Fractional knapsack | Sort by value/weight ratio, take greedily |
| Correctness proof | Exchange argument or greedy-stays-ahead |
| When fails | Coin change with arbitrary denominations |

---

## ❓ Revision Questions

1. **Two properties for greedy?** → Greedy choice property and optimal substructure.
2. **Why sort by finish time in activity selection?** → Earliest finish leaves maximum room for future activities.
3. **Fractional vs 0/1 knapsack?** → Fractional: greedy works (take by ratio). 0/1: need DP (can't split items).
4. **Exchange argument: idea?** → Show you can swap any optimal choice for the greedy choice without worsening the solution.
5. **When does greedy fail?** → When greedy choice property doesn't hold (e.g., coin change with {1,3,4}).

---

[← Previous: DP — When to Apply](../07-Dynamic-Programming-Patterns/06-when-to-apply.md) | [Next: Interval Problems →](02-interval-problems.md)
