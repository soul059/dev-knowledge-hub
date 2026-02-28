# Chapter 1: Buy & Sell Stock — Introduction

## 📋 Overview
The "Best Time to Buy and Sell Stock" family is the perfect introduction to **State Machine DP**. Each variant adds a constraint (transaction limit, cooldown, fee), modeled as states with transitions. This chapter covers the simplest versions.

---

## 🧠 The Stock Problem Family

```
Given prices[0..n-1], buy and sell stock to maximize profit.

┌────────────────────────────┬──────────────────────────────┐
│ Variant                    │ Constraint                   │
├────────────────────────────┼──────────────────────────────┤
│ I.   Single Transaction    │ At most 1 buy/sell pair      │
│ II.  Unlimited Transactions│ Any number of transactions   │
│ III. At Most K Transactions│ At most k buy/sell pairs     │
│ IV.  With Cooldown         │ Must wait 1 day after sell   │
│ V.   With Transaction Fee  │ Pay fee on each transaction  │
└────────────────────────────┴──────────────────────────────┘
```

---

## 🔨 Variant I: Single Transaction

```
prices = [7, 1, 5, 3, 6, 4]

Buy on day 1 (price=1), sell on day 4 (price=6).
Profit = 6 - 1 = 5.

Algorithm: Track minimum price seen so far.
  
function maxProfit_I(prices):
    minPrice = ∞
    maxProfit = 0
    for price in prices:
        minPrice = min(minPrice, price)
        maxProfit = max(maxProfit, price - minPrice)
    return maxProfit

Trace:
  Day 0: min=7, profit=0
  Day 1: min=1, profit=0
  Day 2: min=1, profit=4
  Day 3: min=1, profit=2
  Day 4: min=1, profit=5 ✓
  Day 5: min=1, profit=3

Time: O(n), Space: O(1)
```

---

## 🔨 Variant II: Unlimited Transactions

```
prices = [7, 1, 5, 3, 6, 4]

Buy on day 1 (1), sell on day 2 (5): profit 4
Buy on day 3 (3), sell on day 4 (6): profit 3
Total: 7

Greedy insight: capture every upward movement!

function maxProfit_II(prices):
    profit = 0
    for i = 1 to n-1:
        if prices[i] > prices[i-1]:
            profit += prices[i] - prices[i-1]
    return profit

Why this works:
  Any transaction buy on day a, sell on day b equals:
  (prices[b] - prices[a]) = sum of daily gains from a to b-1
  
  So picking all positive daily changes = optimal.

Time: O(n), Space: O(1)
```

---

## 🔨 State Machine View (Foundation)

```
Two states: HOLDING stock vs NOT HOLDING stock

  ┌──────────┐    buy     ┌──────────┐
  │          │──────────→│          │
  │   NOT    │           │ HOLDING  │
  │ HOLDING  │←──────────│          │
  │          │    sell    │          │
  └──────────┘           └──────────┘
       ↑ rest                ↑ rest
       └──┘                  └──┘

State variables at day i:
  notHold[i] = max profit on day i when NOT holding stock
  hold[i]    = max profit on day i when HOLDING stock

Transitions:
  notHold[i] = max(notHold[i-1],     // rest (stay not holding)
                   hold[i-1] + price) // sell (was holding → sell)
  
  hold[i]    = max(hold[i-1],        // rest (keep holding)
                   notHold[i-1] - price) // buy (was not holding → buy)

Base: notHold[0] = 0, hold[0] = -prices[0]
Answer: notHold[n-1] (always better to end not holding)
```

---

## 🔬 State Machine Trace: prices = [7, 1, 5, 3, 6, 4]

```
Day 0: notHold=0, hold=-7
Day 1: notHold=max(0, -7+1)=0,    hold=max(-7, 0-1)=-1
Day 2: notHold=max(0, -1+5)=4,    hold=max(-1, 0-5)=-1
Day 3: notHold=max(4, -1+3)=4,    hold=max(-1, 4-3)=1
Day 4: notHold=max(4, 1+6)=7,     hold=max(1, 4-6)=1
Day 5: notHold=max(7, 1+4)=7,     hold=max(1, 7-4)=3

Answer: notHold[5] = 7 ✓ (matches greedy for unlimited)
```

---

## 🔑 Variant I as State Machine

```
For single transaction, modify the buy transition:
  hold[i] = max(hold[i-1], -price)    // NOT notHold[i-1] - price
                                       // Start from 0 profit always

This ensures at most ONE buy (and thus one sell).

function maxProfit_I_stateMachine(prices):
    notHold = 0
    hold = -∞
    for price in prices:
        notHold = max(notHold, hold + price)
        hold = max(hold, -price)    // buy from 0, not from notHold
    return notHold

Note: hold = max(hold, -price) = -min(price so far)
      Same as tracking minimum price!
```

---

## 📊 Summary: Variant I vs II

| Aspect | Variant I (Single) | Variant II (Unlimited) |
|--------|-------------------|----------------------|
| **Buy transition** | hold = max(hold, **-price**) | hold = max(hold, **notHold-price**) |
| **Constraint** | 1 transaction | Any number |
| **Simple approach** | Track min price | Sum positive diffs |
| **State machine** | Same framework, different buy rule | Full state machine |
| **Time/Space** | O(n) / O(1) | O(n) / O(1) |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **States** | hold (have stock), notHold (no stock) |
| **Transitions** | buy, sell, rest |
| **Variant I** | hold = max(hold, -price) — single buy |
| **Variant II** | hold = max(hold, notHold - price) — rebuy |
| **Base** | notHold = 0, hold = -∞ |
| **Answer** | notHold at end |

---

## ❓ Quick Revision Questions

1. **What are the two states in the stock state machine?**
2. **How does the buy transition differ between Variant I and II?**
3. **Why is the answer always notHold (not hold)?**
4. **Why can Variant II be solved greedily?**
5. **What does hold = -price mean in Variant I?**
6. **Draw the state machine diagram from memory.**

---

[← Previous Unit: Palindrome Partitioning Interval](../10-Interval-DP/06-palindrome-partitioning-interval.md) | [Next: State Transitions →](02-state-transitions.md)

[← Back to README](../README.md)
