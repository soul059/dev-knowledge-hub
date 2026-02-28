# Chapter 5: Coin Change 2 (Number of Ways)

## 📋 Overview
**Problem:** Given coins of different denominations and a total amount, find the **number of combinations** that make up that amount. You have infinite supply of each denomination. Order does NOT matter (combinations, not permutations).

---

## 🧠 Problem Visualization

```
Coins: [1, 2, 5]    Amount: 5

All combinations:
  5 = 5               (one $5 coin)
  5 = 2+2+1           (two $2 + one $1)
  5 = 2+1+1+1         (one $2 + three $1)
  5 = 1+1+1+1+1       (five $1 coins)

Answer: 4 combinations

Note: [2,2,1] and [1,2,2] are the SAME combination
      (order doesn't matter)
```

---

## 🔍 Key Difference: Combinations vs Permutations

```
┌─────────────────────────────────┬────────────────────────────────┐
│    COMBINATIONS (this problem)  │    PERMUTATIONS                │
│    Order does NOT matter        │    Order DOES matter           │
│                                 │                                │
│    [1,2,2] = [2,1,2] = [2,2,1] │    [1,2,2] ≠ [2,1,2] ≠ [2,2,1]│
│    Count as ONE way             │    Count as THREE ways         │
│                                 │                                │
│    Loop: coins in OUTER loop    │    Loop: amount in OUTER loop  │
│          amount in INNER loop   │          coins in INNER loop   │
└─────────────────────────────────┴────────────────────────────────┘
```

---

## 📐 DP Development

### State
```
dp[i] = number of ways to make amount i
```

### Recurrence (Combinations — coins outer, amount inner)
```
for each coin c:
    for i = c to amount:
        dp[i] += dp[i - c]

Base: dp[0] = 1 (one way to make 0: use no coins)
```

### Why Outer Loop on Coins Gives Combinations
```
By processing ONE COIN AT A TIME, we ensure:
  - When processing coin=2, we only consider ways using coins ≤ 2
  - This prevents counting [1,2] and [2,1] as separate ways

After processing coin=1:  dp counts ways using {1} only
After processing coin=2:  dp counts ways using {1, 2}
After processing coin=5:  dp counts ways using {1, 2, 5}
```

---

## 🧪 Step-by-Step Trace

```
Coins = [1, 2, 5], Amount = 5

Initial: dp = [1, 0, 0, 0, 0, 0]
               ↑ base case

═══ Processing coin = 1 ═══
i=1: dp[1] += dp[0] = 0+1 = 1     {1}
i=2: dp[2] += dp[1] = 0+1 = 1     {1,1}
i=3: dp[3] += dp[2] = 0+1 = 1     {1,1,1}
i=4: dp[4] += dp[3] = 0+1 = 1     {1,1,1,1}
i=5: dp[5] += dp[4] = 0+1 = 1     {1,1,1,1,1}

dp = [1, 1, 1, 1, 1, 1]

═══ Processing coin = 2 ═══
i=2: dp[2] += dp[0] = 1+1 = 2     {1,1}, {2}
i=3: dp[3] += dp[1] = 1+1 = 2     {1,1,1}, {2,1}
i=4: dp[4] += dp[2] = 1+2 = 3     {1,1,1,1}, {2,1,1}, {2,2}
i=5: dp[5] += dp[3] = 1+2 = 3     {1,1,1,1,1}, {2,1,1,1}, {2,2,1}

dp = [1, 1, 2, 2, 3, 3]

═══ Processing coin = 5 ═══
i=5: dp[5] += dp[0] = 3+1 = 4     adds {5}

dp = [1, 1, 2, 2, 3, 4]

Answer: dp[5] = 4
```

---

## 📐 Comparison: Combinations vs Permutations Approach

```
COMBINATIONS (this problem):         PERMUTATIONS (different problem):
for coin in coins:   ← OUTER        for i = 1 to amount:   ← OUTER
    for i = coin to amount: ← INNER     for coin in coins:  ← INNER
        dp[i] += dp[i-coin]                 dp[i] += dp[i-coin]

Combinations for amount=3, coins=[1,2]:    Permutations:
  {1,1,1}, {1,2}                           [1,1,1], [1,2], [2,1]
  = 2 ways                                 = 3 ways
```

---

## 💡 2D Perspective (for Understanding)

```
The 1D approach is a space-optimized version of 2D:

dp[i][j] = ways to make amount j using first i coin types

         amount →  0   1   2   3   4   5
                ┌───┬───┬───┬───┬───┬───┐
  no coins  (0) │ 1 │ 0 │ 0 │ 0 │ 0 │ 0 │
                ├───┼───┼───┼───┼───┼───┤
  +coin 1   (1) │ 1 │ 1 │ 1 │ 1 │ 1 │ 1 │
                ├───┼───┼───┼───┼───┼───┤
  +coin 2   (2) │ 1 │ 1 │ 2 │ 2 │ 3 │ 3 │
                ├───┼───┼───┼───┼───┼───┤
  +coin 5   (3) │ 1 │ 1 │ 2 │ 2 │ 3 │ 4 │← answer
                └───┴───┴───┴───┴───┴───┘

dp[i][j] = dp[i-1][j]        (don't use coin i)
          + dp[i][j-coin[i]]  (use coin i at least once)

Space optimizable to 1D because each row only depends on
current row (left) and previous row (above).
```

---

## 📊 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force recursion | Exponential | O(amount) | Too slow |
| Memoization (2D) | O(n × amount) | O(n × amount) | n = # coins |
| Tabulation (2D) | O(n × amount) | O(n × amount) | Clearer structure |
| **Tabulation (1D)** | **O(n × amount)** | **O(amount)** | **Optimal** |

---

## ❓ Quick Revision Questions

1. **What's the difference between Coin Change 1 and Coin Change 2?**
2. **Why does putting coins in the outer loop give combinations (not permutations)?**
3. **What is dp[0] and why is it 1?**
4. **How would you modify this to count permutations instead?**
5. **For coins=[2,5] and amount=3, what is the answer?**
6. **Why can we iterate left-to-right for the inner loop (unbounded knapsack style)?**

---

[← Previous: Coin Change (Min)](04-coin-change-min.md) | [Next: Decode Ways →](06-decode-ways.md)

[← Back to README](../README.md)
