# Chapter 2: Unbounded Knapsack

## 📋 Overview
In the **Unbounded Knapsack**, each item can be used **unlimited times**. This single change transforms the iteration direction and opens up a new family of problems including rod cutting and integer partitioning.

---

## 🧠 Core Concept

```
0/1 Knapsack:      Each item used AT MOST once
Unbounded Knapsack: Each item used ANY number of times

Example: items = [(weight=2, value=5), (weight=3, value=8)]
         Capacity W = 7

  Option 1: item1 × 3 + item2 × 0 = weight 6, value 15
  Option 2: item1 × 2 + item2 × 1 = weight 7, value 18 ← best
  Option 3: item1 × 0 + item2 × 2 = weight 6, value 16
  
  Answer: 18
```

---

## 🔨 Recurrence

```
2D formulation:
  dp[i][w] = max value using items 0..i-1 with capacity w

  if weight[i-1] > w:
      dp[i][w] = dp[i-1][w]
  else:
      dp[i][w] = max(
          dp[i-1][w],                        // SKIP this item entirely
          dp[i][w - weight[i-1]] + value[i-1] // TAKE item i-1 (can take again!)
               ↑ SAME row i (not i-1)
      )

Notice: dp[i][w - weight[i-1]] uses row i, not row i-1.
This is because after taking item i-1, we can still take it again.
```

---

## 🔑 Key Difference from 0/1 Knapsack

```
0/1 Knapsack (each item once):
  dp[w] = max(dp[w], dp[w - wt[i]] + val[i])
  Iterate w: W → wt[i]  (RIGHT to LEFT)
  ─────────────────>
  Uses previous row values ✓

Unbounded Knapsack (items reusable):
  dp[w] = max(dp[w], dp[w - wt[i]] + val[i])
  Iterate w: wt[i] → W  (LEFT to RIGHT)
  <─────────────────
  Uses current row values → allows repeating items ✓

  Same formula, OPPOSITE direction!
```

---

## 💻 Pseudocode

```
function unboundedKnapsack(weights, values, W):
    n = len(weights)
    dp = array of size W+1, filled with 0
    
    for i = 0 to n-1:
        for w = weights[i] to W:        // LEFT TO RIGHT
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[W]

Alternative (iterate by capacity first):
function unboundedKnapsack2(weights, values, W):
    dp = array of size W+1, filled with 0
    
    for w = 1 to W:
        for i = 0 to n-1:
            if weights[i] <= w:
                dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[W]
```

---

## 🔬 Trace: weights=[2,3], values=[5,8], W=7

```
dp = [0, 0, 0, 0, 0, 0, 0, 0]

Item 0 (wt=2, val=5), w: 2→7:
  w=2: max(0, dp[0]+5) = 5    → [0,0,5,0,0,0,0,0]
  w=3: max(0, dp[1]+5) = 0    → no change
  w=4: max(0, dp[2]+5) = 10   → [0,0,5,0,10,0,0,0]
  w=5: max(0, dp[3]+5) = 5    → [0,0,5,0,10,5,0,0]
  w=6: max(0, dp[4]+5) = 15   → [0,0,5,0,10,5,15,0]
  w=7: max(0, dp[5]+5) = 10   → [0,0,5,0,10,5,15,10]

Item 1 (wt=3, val=8), w: 3→7:
  w=3: max(0, dp[0]+8) = 8    → [0,0,5,8,10,0,0,0]
  w=4: max(10, dp[1]+8) = 10  → no change
  w=5: max(5, dp[2]+8) = 13   → [0,0,5,8,10,13,0,0]
  w=6: max(15, dp[3]+8) = 16  → [0,0,5,8,10,13,16,0]
  w=7: max(10, dp[4]+8) = 18  → [0,0,5,8,10,13,16,18]

Answer: dp[7] = 18 (item0×2 + item1×1 = weight 7)
```

---

## 🌐 Classic Unbounded Knapsack Problems

### Rod Cutting
```
Given rod of length n and prices for each length 1..n.
Cut rod to maximize total price.

This IS unbounded knapsack where:
  - items = lengths 1 to n
  - weights = lengths (1, 2, ..., n)
  - values = prices
  - Capacity = n

function rodCutting(prices, n):
    dp = array of size n+1, filled with 0
    for length = 1 to n:
        for cut = 1 to length:
            dp[length] = max(dp[length], dp[length - cut] + prices[cut])
    return dp[n]
```

### Coin Change (Minimum Coins)
```
Unbounded knapsack where:
  - Minimize instead of maximize
  - All items have value=1 (count of coins)
  - "weight" = coin denomination

dp[0] = 0
dp[w] = min(dp[w], dp[w - coin] + 1) for each coin

Already covered in Unit 3 — knapsack perspective here.
```

### Integer Break
```
Break integer n into sum of integers to maximize product.
Similar to rod cutting where price = multiplicative.

dp[i] = max over j in [1..i-1] of max(j*(i-j), j*dp[i-j])
```

---

## 📊 Comparison: 0/1 vs Unbounded

| Aspect | 0/1 Knapsack | Unbounded Knapsack |
|--------|-------------|-------------------|
| **Item usage** | At most once | Unlimited times |
| **2D recurrence** | dp[i-1][w-wt] | dp[i][w-wt] |
| **1D direction** | Right to Left | Left to Right |
| **Time** | O(n·W) | O(n·W) |
| **Space** | O(W) | O(W) |
| **Examples** | Subset sum | Rod cutting, coin change |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[w] = max value achievable with capacity w |
| **Recurrence** | dp[w] = max(dp[w], dp[w-wt[i]] + val[i]) |
| **Direction** | Left to right (allows item reuse) |
| **Base** | dp[0] = 0 |
| **Complexity** | O(n·W) time, O(W) space |
| **Key Insight** | Same formula as 0/1, opposite iteration direction |

---

## ❓ Quick Revision Questions

1. **What is the single change from 0/1 Knapsack to Unbounded Knapsack?**
2. **Why does left-to-right iteration allow item reuse?**
3. **How is Rod Cutting modeled as Unbounded Knapsack?**
4. **In the 2D table, what row index does the "take" case reference?**
5. **Name three classic problems that reduce to Unbounded Knapsack.**
6. **Can you solve Unbounded Knapsack by iterating capacity-first instead of item-first?**

---

[← Previous: 0/1 Knapsack](01-01-knapsack.md) | [Next: Subset Sum →](03-subset-sum.md)

[← Back to README](../README.md)
