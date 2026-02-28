# Chapter 2: State Transitions in DP

## 📋 Overview
State Machine DP formalizes the idea that a problem has **discrete states** with **defined transitions**. Each day/step, the system moves between states according to allowed transitions, and we optimize the cumulative value.

---

## 🧠 Core Framework

```
General State Machine DP:
  1. Identify all possible STATES
  2. Define allowed TRANSITIONS between states
  3. Assign COSTS/VALUES to transitions
  4. Find optimal path from start to end state

┌─────────┐        ┌─────────┐        ┌─────────┐
│ State A  │───────→│ State B │───────→│ State C │
│         │←───────│         │        │         │
└─────────┘        └─────────┘        └─────────┘
     ↑ self            ↑ self             ↑ self
     └──┘              └──┘               └──┘

dp_A[i] = optimal value at step i in state A
dp_B[i] = optimal value at step i in state B
dp_C[i] = optimal value at step i in state C

Transitions define how dp values flow between states.
```

---

## 🔨 Pattern: Stock Trading as State Machine

```
States for Buy/Sell Stock with K transactions:

State: (day, transactions_completed, holding_or_not)

  dp[i][k][0] = max profit on day i, k transactions done, NOT holding
  dp[i][k][1] = max profit on day i, k transactions done, HOLDING

Transitions (each buy-sell pair = 1 transaction):
  
  dp[i][k][0] = max(
      dp[i-1][k][0],              // rest (was not holding)
      dp[i-1][k-1][1] + price[i]  // sell (complete k-th transaction)
  )
  
  dp[i][k][1] = max(
      dp[i-1][k][1],              // rest (was holding)
      dp[i-1][k][0] - price[i]    // buy (start holding)
  )

Note: "transaction completed" counted on SELL.
```

---

## 🔬 Trace: K=2, prices=[3,3,5,0,0,3,1,4]

```
Optimize for at most 2 transactions.

Space-optimized: track 4 variables
  buy1:  best profit after first buy
  sell1: best profit after first sell
  buy2:  best profit after second buy
  sell2: best profit after second sell

function maxProfit_K2(prices):
    buy1 = -∞, sell1 = 0
    buy2 = -∞, sell2 = 0
    
    for price in prices:
        buy1  = max(buy1,  -price)
        sell1 = max(sell1, buy1 + price)
        buy2  = max(buy2,  sell1 - price)
        sell2 = max(sell2, buy2 + price)
    
    return sell2

Trace:
  price=3: buy1=-3, sell1=0, buy2=-3, sell2=0
  price=3: buy1=-3, sell1=0, buy2=-3, sell2=0
  price=5: buy1=-3, sell1=2, buy2=-3, sell2=2
  price=0: buy1=0,  sell1=2, buy2=2,  sell2=2
  price=0: buy1=0,  sell1=2, buy2=2,  sell2=2
  price=3: buy1=0,  sell1=3, buy2=2,  sell2=5
  price=1: buy1=0,  sell1=3, buy2=2,  sell2=5
  price=4: buy1=0,  sell1=4, buy2=2,  sell2=6

Answer: sell2 = 6 ✓
  Transaction 1: buy at 0, sell at 3 (+3)
  Transaction 2: buy at 1, sell at 4 (+3)
  Total: 6
```

---

## 🔨 General K Transactions

```
function maxProfit_K(prices, K):
    n = len(prices)
    
    // Edge case: if K >= n/2, unlimited transactions
    if K >= n / 2:
        return maxProfit_unlimited(prices)
    
    // dp[k][0] = not holding after k transactions
    // dp[k][1] = holding after k transactions
    dp = 2D array [K+1][2]
    
    for k = 0 to K:
        dp[k][0] = 0
        dp[k][1] = -∞
    
    for price in prices:
        for k = K down to 1:
            dp[k][0] = max(dp[k][0], dp[k][1] + price)     // sell
            dp[k][1] = max(dp[k][1], dp[k-1][0] - price)   // buy
    
    return dp[K][0]

Time: O(n × K), Space: O(K)
```

---

## 🌐 State Machine Beyond Stocks

### Example: String Matching States
```
Matching pattern with wildcards:

States: position in pattern
Transitions: match character, skip wildcard

dp[i][j] = can pattern[0..i] match string[0..j]?
```

### Example: Painting Houses
```
3 colors, no two adjacent houses same color.

States: last color used (Red, Blue, Green)
Transitions: any color except previous

dp[i][R] = min cost painting houses 0..i, house i is Red
dp[i][R] = cost[i][R] + min(dp[i-1][B], dp[i-1][G])
```

### Example: Traffic Light System
```
States: Green, Yellow, Red
Transitions: G→Y, Y→R, R→G (cyclic)

dp[state][time] = optimize behavior at each state/time
```

---

## 🔑 Identifying State Machine DP

```
Characteristics:
  ┌─────────────────────────────────────────────────┐
  │ 1. Problem has distinct "modes" or "phases"     │
  │ 2. Transitions between modes follow rules       │
  │ 3. At each step, decision depends on mode       │
  │ 4. Optimize cumulative value over all steps     │
  └─────────────────────────────────────────────────┘

Red flags in problem statements:
  - "can buy or sell"
  - "must wait/cool down"
  - "at most K times"
  - "no two adjacent same"
  - "alternate between"
  - "state changes with cost"
```

---

## 📊 State Count vs Complexity

```
┌──────────────────────────┬────────┬─────────────┐
│ Problem                  │ States │ Transitions  │
├──────────────────────────┼────────┼─────────────┤
│ Stock (1 txn)            │ 2      │ buy/sell/rest│
│ Stock (K txns)           │ 2K+1   │ sequential   │
│ Stock (cooldown)         │ 3      │ cyclic       │
│ Paint Houses (3 colors)  │ 3      │ 3→2         │
│ Paint Houses (K colors)  │ K      │ K→K-1       │
│ Keyboard (3 ops)         │ 3      │ varies       │
└──────────────────────────┴────────┴─────────────┘

Total complexity = O(n × |States| × |Transitions|)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **States** | Discrete modes the system can be in |
| **Transitions** | Allowed moves between states |
| **Framework** | dp[step][state] = optimal value |
| **Stock K txns** | dp[k][hold] with k=0..K, hold=0/1 |
| **Optimization** | K≥n/2 → unlimited (greedy) |
| **Identification** | Look for modes, constraints on switching |

---

## ❓ Quick Revision Questions

1. **What defines a state in State Machine DP?**
2. **How do you handle K transactions in the stock problem?**
3. **Why check K ≥ n/2 for the unlimited case?**
4. **Give an example of State Machine DP outside stock trading.**
5. **How does the number of states affect time complexity?**
6. **What is the space optimization for K-transaction stock problem?**

---

[← Previous: Buy & Sell Stock Intro](01-buy-sell-stock-intro.md) | [Next: Multiple Transactions →](03-multiple-transactions.md)

[← Back to README](../README.md)
