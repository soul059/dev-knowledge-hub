# Chapter 6: Coin Change — Knapsack Perspective

## 📋 Overview
Coin Change problems are classic Unbounded Knapsack applications. This chapter unifies **minimum coins**, **counting combinations**, and **counting permutations** through the knapsack lens, revealing how iteration order determines the variant solved.

---

## 🧠 The Three Coin Change Variants

```
coins = [1, 2, 5],  amount = 5

┌──────────────────────────────────────────────────┐
│ Variant 1: Minimum Coins to Make Amount          │
│   Answer: 1 (just use coin 5)                    │
│   Unbounded Knapsack: MINIMIZE                   │
├──────────────────────────────────────────────────┤
│ Variant 2: Count Combinations (order irrelevant) │
│   {5}, {2+2+1}, {2+1+1+1}, {1+1+1+1+1}          │
│   Answer: 4                                      │
│   Unbounded Knapsack: COUNT, item-first loop     │
├──────────────────────────────────────────────────┤
│ Variant 3: Count Permutations (order matters)    │
│   {5}, {2+2+1}, {2+1+2}, {1+2+2}, etc.          │
│   Answer: 9                                      │
│   NOT knapsack: amount-first loop                │
└──────────────────────────────────────────────────┘
```

---

## 🔨 Variant 1: Minimum Coins

```
function coinChangeMin(coins, amount):
    dp = array of size amount+1, filled with ∞
    dp[0] = 0
    
    for coin in coins:
        for a = coin to amount:         // left to right (unbounded)
            dp[a] = min(dp[a], dp[a - coin] + 1)
    
    return dp[amount] if dp[amount] ≠ ∞ else -1

Trace: coins=[1,2,5], amount=5
  dp = [0, ∞, ∞, ∞, ∞, ∞]
  
  coin=1: dp = [0, 1, 2, 3, 4, 5]
  coin=2: dp = [0, 1, 1, 2, 2, 3]
  coin=5: dp = [0, 1, 1, 2, 2, 1]
  
  Answer: dp[5] = 1
```

---

## 🔨 Variant 2: Count Combinations

```
function coinChangeCombinations(coins, amount):
    dp = array of size amount+1, all 0
    dp[0] = 1
    
    for coin in coins:                   // OUTER: coins
        for a = coin to amount:          // INNER: amount
            dp[a] += dp[a - coin]
    
    return dp[amount]

WHY item-first gives combinations:
  Each coin is processed fully before next coin.
  When processing coin=2, coin=1 is already "past."
  So {1,2} is counted but {2,1} is NOT a separate way.
  
  Coin ordering is fixed → no duplicate permutations.
```

### Trace: coins=[1,2,5], amount=5

```
dp = [1, 0, 0, 0, 0, 0]

coin=1 (a: 1→5):
  dp = [1, 1, 1, 1, 1, 1]
  // {1}, {1,1}, {1,1,1}, {1,1,1,1}, {1,1,1,1,1}

coin=2 (a: 2→5):
  a=2: dp[2]+=dp[0] → 2   // {1,1} and {2}
  a=3: dp[3]+=dp[1] → 2   // {1,1,1} and {2,1}
  a=4: dp[4]+=dp[2] → 3   // {1111},{21},{22}
  a=5: dp[5]+=dp[3] → 3   // {11111},{211},{221}
  dp = [1, 1, 2, 2, 3, 3]

coin=5 (a: 5→5):
  a=5: dp[5]+=dp[0] → 4   // + {5}
  dp = [1, 1, 2, 2, 3, 4]

Answer: dp[5] = 4  (combinations: {11111},{2111},{221},{5})
```

---

## 🔨 Variant 3: Count Permutations

```
function coinChangePermutations(coins, amount):
    dp = array of size amount+1, all 0
    dp[0] = 1
    
    for a = 1 to amount:                 // OUTER: amount
        for coin in coins:               // INNER: coins
            if a >= coin:
                dp[a] += dp[a - coin]
    
    return dp[amount]

WHY amount-first gives permutations:
  At each amount, ALL coins are considered.
  dp[3] = dp[2]+dp[1] = ({1,2},{2,1},{1,1,1}) + ({1,2},{2,1})... 
  Different orderings ARE counted separately.
```

### Trace: coins=[1,2,5], amount=5

```
dp = [1, 0, 0, 0, 0, 0]

a=1: dp[1]+=dp[0] → 1             // {1}
a=2: dp[2]+=dp[1]+dp[0] → 2       // {1,1},{2}
a=3: dp[3]+=dp[2]+dp[1] → 3       // {1,1,1},{1,2},{2,1}
a=4: dp[4]+=dp[3]+dp[2] → 5       // {1111},{12},{21},{112},{211}... 
a=5: dp[5]+=dp[4]+dp[3]+dp[0] → 9

Answer: dp[5] = 9 permutations
```

---

## 🔑 The Critical Difference

```
┌──────────────────────────────────────────────────────┐
│                  LOOP ORDER MATTERS!                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  for coin in coins:        for a = 1 to amount:      │
│      for a = coin to amt:      for coin in coins:    │
│          dp[a] += ...              dp[a] += ...      │
│                                                      │
│  → COMBINATIONS              → PERMUTATIONS          │
│  (coin order fixed)          (all orders counted)    │
│                                                      │
│  This is the MOST IMPORTANT  Equivalent to the       │
│  distinction in knapsack     "Combination Sum IV"    │
│  problems!                   problem on LeetCode.    │
└──────────────────────────────────────────────────────┘
```

---

## 🌐 Full Knapsack Family Map

```
                          Knapsack
                         /        \
                      0/1        Unbounded
                     /              \
              right-to-left      left-to-right
             /       \           /        \
         Subset   Partition   CoinMin   CoinWays
          Sum     Equal       Coins     (Combos)
           |                              |
        Target                        Permutations
         Sum                        (amount-first)

Decision Guide:
  ┌────────────────────┬──────────────────┐
  │ Question           │ Determines       │
  ├────────────────────┼──────────────────┤
  │ Each item once?    │ 0/1 vs Unbounded │
  │ Max/Min or Count?  │ max/min vs +=    │
  │ Order matters?     │ item-first vs    │
  │                    │ capacity-first   │
  └────────────────────┴──────────────────┘
```

---

## 🧩 Reconstructing Coins Used

```
For minimum coins, track which coin was used:

function reconstructCoins(coins, amount):
    dp = [∞] * (amount + 1)
    parent = [-1] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for a = coin to amount:
            if dp[a - coin] + 1 < dp[a]:
                dp[a] = dp[a - coin] + 1
                parent[a] = coin
    
    // Reconstruct
    result = []
    a = amount
    while a > 0:
        result.append(parent[a])
        a -= parent[a]
    return result
```

---

## 📊 Summary Table

| Variant | Loop Order | Operation | Result |
|---------|-----------|-----------|--------|
| **Min Coins** | coins→amount (L→R) | min | Fewest coins |
| **Combinations** | coins→amount (L→R) | += | Unordered ways |
| **Permutations** | amount→coins | += | Ordered ways |
| **All variants** | O(n × amount) | — | Same complexity |

---

## ❓ Quick Revision Questions

1. **What determines whether you count combinations vs permutations?**
2. **Why does item-first looping avoid counting different orderings?**
3. **How is Coin Change an Unbounded Knapsack problem?**
4. **What is the iteration direction for unbounded knapsack and why?**
5. **Draw the knapsack family tree from memory.**
6. **How would you modify the min-coins solution to reconstruct the actual coins used?**

---

[← Previous: Target Sum](05-target-sum.md) | [Next Unit: LIS Pattern →](../09-LIS-Pattern/01-longest-increasing-subsequence.md)

[← Back to README](../README.md)
