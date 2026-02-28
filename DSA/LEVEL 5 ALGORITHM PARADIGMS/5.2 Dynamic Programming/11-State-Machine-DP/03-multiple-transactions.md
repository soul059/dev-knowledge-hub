# Chapter 3: Multiple Transactions (At Most K)

## 📋 Overview
Given stock prices and a limit of **at most K buy-sell transactions**, find the maximum profit. This generalizes all previous stock problems: K=1 is Variant I, K=∞ is Variant II, K=2 is a special case.

---

## 🧠 Core Concept

```
prices = [2, 4, 1], K = 2

Options:
  Buy day 0, sell day 1: profit 2 (1 transaction)
  Buy day 0, sell day 1, buy day 2: only 1 complete
  
Answer: 2

prices = [3, 2, 6, 5, 0, 3], K = 2

  Transaction 1: buy 2, sell 6 → profit 4
  Transaction 2: buy 0, sell 3 → profit 3
  Total: 7

Answer: 7
```

---

## 🔨 Full DP Solution

```
dp[k][0] = max profit with at most k transactions, NOT holding
dp[k][1] = max profit with at most k transactions, HOLDING

Transitions for each day (price):
  dp[k][0] = max(dp[k][0], dp[k][1] + price)     // sell
  dp[k][1] = max(dp[k][1], dp[k-1][0] - price)   // buy (uses k-1)

Note: buying starts a new transaction possibility.
      Selling completes the transaction (counted at sell).

function maxProfit(K, prices):
    n = len(prices)
    if n <= 1: return 0
    
    // Optimization: if K >= n/2, unlimited transactions
    if K >= n / 2:
        profit = 0
        for i = 1 to n-1:
            profit += max(0, prices[i] - prices[i-1])
        return profit
    
    dp = array [K+1][2]
    for k = 0 to K:
        dp[k][0] = 0
        dp[k][1] = -∞
    
    for price in prices:
        for k = 1 to K:
            dp[k][0] = max(dp[k][0], dp[k][1] + price)
            dp[k][1] = max(dp[k][1], dp[k-1][0] - price)
    
    return dp[K][0]
```

---

## 🔬 Trace: prices = [3,2,6,5,0,3], K=2

```
Init: dp[0]=[0,-∞], dp[1]=[0,-∞], dp[2]=[0,-∞]

price=3:
  k=1: dp[1][0]=max(0,-∞+3)=0, dp[1][1]=max(-∞,0-3)=-3
  k=2: dp[2][0]=max(0,-∞+3)=0, dp[2][1]=max(-∞,0-3)=-3
  dp = [[0,-∞], [0,-3], [0,-3]]

price=2:
  k=1: dp[1][0]=max(0,-3+2)=0, dp[1][1]=max(-3,0-2)=-2
  k=2: dp[2][0]=max(0,-3+2)=0, dp[2][1]=max(-3,0-2)=-2
  dp = [[0,-∞], [0,-2], [0,-2]]

price=6:
  k=1: dp[1][0]=max(0,-2+6)=4, dp[1][1]=max(-2,0-6)=-2
  k=2: dp[2][0]=max(0,-2+6)=4, dp[2][1]=max(-2,0-6)=-2
  dp = [[0,-∞], [4,-2], [4,-2]]

price=5:
  k=1: dp[1][0]=max(4,-2+5)=4, dp[1][1]=max(-2,0-5)=-2
  k=2: dp[2][0]=max(4,-2+5)=4, dp[2][1]=max(-2,4-5)=-1
  dp = [[0,-∞], [4,-2], [4,-1]]  ← k=2 buy uses k=1 sell profit!

price=0:
  k=1: dp[1][0]=max(4,-2+0)=4, dp[1][1]=max(-2,0-0)=0
  k=2: dp[2][0]=max(4,-1+0)=4, dp[2][1]=max(-1,4-0)=4
  dp = [[0,-∞], [4,0], [4,4]]

price=3:
  k=1: dp[1][0]=max(4,0+3)=4, dp[1][1]=max(0,0-3)=0
  k=2: dp[2][0]=max(4,4+3)=7, dp[2][1]=max(4,4-3)=4
  dp = [[0,-∞], [4,0], [7,4]]

Answer: dp[2][0] = 7 ✓
  Transaction 1: buy@2, sell@6 (profit 4)
  Transaction 2: buy@0, sell@3 (profit 3)
```

---

## 🔑 Why K ≥ n/2 = Unlimited

```
Each transaction needs at least 2 days (buy + sell).
Maximum possible transactions = n/2.

If K ≥ n/2, the constraint is never binding → unlimited.
Switch to O(n) greedy:
  Sum all positive daily price changes.

This optimization is CRITICAL because:
  Without it: O(n × K) could be O(n²) for large K.
  With it: O(n) when K is large.
```

---

## 🌐 Alternative: 2D DP with Days

```
Full table approach (more memory, clearer structure):

dp[i][k][hold] = max profit on day i, k transactions, holding state

function maxProfit_2D(K, prices):
    n = len(prices)
    dp = 3D array [n][K+1][2]
    
    // Base: day 0
    for k = 0 to K:
        dp[0][k][0] = 0
        dp[0][k][1] = -prices[0]
    
    for i = 1 to n-1:
        for k = 1 to K:
            dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
            dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
    
    return dp[n-1][K][0]

Time: O(n × K), Space: O(n × K) → optimizable to O(K)
```

---

## ⚡ Special Cases

```
K = 1:
  dp[1][0] = max(dp[1][0], dp[1][1] + price)   // sell
  dp[1][1] = max(dp[1][1], dp[0][0] - price)    // buy from 0 profit
  
  → Simplifies to tracking min price, same as Variant I

K = 2:
  4 variables: buy1, sell1, buy2, sell2
  → The classic "at most 2 transactions" solution

K ≥ n/2:
  → Unlimited, use greedy

These are all unified under the general K framework!
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[k][hold] for k transactions, hold ∈ {0,1} |
| **Buy** | dp[k][1] = max(dp[k][1], dp[k-1][0] - price) |
| **Sell** | dp[k][0] = max(dp[k][0], dp[k][1] + price) |
| **Optimization** | K ≥ n/2 → use greedy O(n) |
| **Time** | O(n × K) |
| **Space** | O(K) with rolling |

---

## ❓ Quick Revision Questions

1. **Why is buying linked to dp[k-1] while selling is at dp[k]?**
2. **When does the K constraint become non-binding?**
3. **How does K=1 simplify to the min-price-tracking approach?**
4. **What is the 3D DP formulation?**
5. **Why iterate k from 1 to K (not K down to 1)?**
6. **What is the maximum number of profitable transactions possible?**

---

[← Previous: State Transitions](02-state-transitions.md) | [Next: With Cooldown →](04-with-cooldown.md)

[← Back to README](../README.md)
