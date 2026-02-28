# Chapter 4: Buy & Sell Stock with Cooldown

## 📋 Overview
After selling stock, you must wait **one day** before buying again (cooldown period). This adds a third state to the state machine, creating a more interesting transition graph.

---

## 🧠 Core Concept

```
prices = [1, 2, 3, 0, 2]

Without cooldown: buy 1, sell 3, buy 0, sell 2 → profit 4
With cooldown:    buy 1, sell 3, [cooldown], buy 0, sell 2 → profit 4
                  Day: 0    1      2         3      4

Cooldown forces a gap day between sell and next buy.
                  
State Machine (3 states):
  ┌──────────┐   sell    ┌──────────┐   rest    ┌──────────┐
  │ HOLDING  │─────────→│ JUST SOLD│─────────→│   REST   │
  │ (hold)   │          │(cooldown)│          │(can buy) │
  └──────────┘          └──────────┘          └──────────┘
       ↑ rest                                       │
       └──┘                          buy            │
       ↑                                            │
       └────────────────────────────────────────────┘
```

---

## 🔨 Three-State Formulation

```
hold[i]    = max profit on day i, HOLDING stock
sold[i]    = max profit on day i, JUST SOLD (cooldown tomorrow)
rest[i]    = max profit on day i, RESTING (can buy tomorrow)

Transitions:
  hold[i] = max(hold[i-1],          // keep holding
                rest[i-1] - price)   // buy (from rest, NOT from sold!)
  
  sold[i] = hold[i-1] + price       // sell what we hold
  
  rest[i] = max(rest[i-1],          // keep resting
                sold[i-1])           // transition from sold to rest

Base: hold[0] = -prices[0], sold[0] = -∞, rest[0] = 0
Answer: max(sold[n-1], rest[n-1])   // not hold (would lose money)
```

---

## 💻 Pseudocode

```
function maxProfit(prices):
    if len(prices) <= 1: return 0
    
    hold = -prices[0]
    sold = -∞     // can't have sold on day 0
    rest = 0
    
    for i = 1 to n-1:
        prevHold = hold
        prevSold = sold
        prevRest = rest
        
        hold = max(prevHold, prevRest - prices[i])
        sold = prevHold + prices[i]
        rest = max(prevRest, prevSold)
    
    return max(sold, rest)
```

---

## 🔬 Trace: prices = [1, 2, 3, 0, 2]

```
Day 0: hold=-1, sold=-∞, rest=0

Day 1 (price=2):
  hold = max(-1, 0-2) = -1          // keep holding (bought at 1)
  sold = -1+2 = 1                    // sell: held at -1, get +2
  rest = max(0, -∞) = 0             // nothing sold yesterday
  State: hold=-1, sold=1, rest=0

Day 2 (price=3):
  hold = max(-1, 0-3) = -1          // keep holding
  sold = -1+3 = 2                    // sell for 3
  rest = max(0, 1) = 1              // yesterday's sold becomes rest
  State: hold=-1, sold=2, rest=1

Day 3 (price=0):
  hold = max(-1, 1-0) = 1           // buy at 0 from rest=1!
  sold = -1+0 = -1                   // sell at 0 (bad, won't use)
  rest = max(1, 2) = 2              // yesterday's sold=2 → rest
  State: hold=1, sold=-1, rest=2

Day 4 (price=2):
  hold = max(1, 2-2) = 1            // keep holding
  sold = 1+2 = 3                     // sell for 2 with hold profit 1
  rest = max(2, -1) = 2             // keep resting
  State: hold=1, sold=3, rest=2

Answer: max(sold=3, rest=2) = 3 ✓

Transaction 1: buy@1 (day 0), sell@2 (day 1) → +1
Cooldown day 2
Transaction 2: buy@0 (day 3), sell@2 (day 4) → +2
Total: 3
```

---

## 🔑 Simplification to Two States

```
We can reduce to 2 states by combining sold and rest:

notHold[i] = max profit, not holding stock
hold[i]    = max profit, holding stock

BUT the cooldown means we can't buy the day after selling.
Track what happened TWO days ago:

hold[i] = max(hold[i-1], notHold[i-2] - prices[i])
                          ↑ TWO days back!

notHold[i] = max(notHold[i-1], hold[i-1] + prices[i])

function maxProfit_2state(prices):
    n = len(prices)
    if n <= 1: return 0
    
    prevNotHold = 0       // notHold[i-2]
    notHold = 0           // notHold[i-1]
    hold = -prices[0]
    
    for i = 1 to n-1:
        newNotHold = max(notHold, hold + prices[i])
        newHold = max(hold, prevNotHold - prices[i])
        prevNotHold = notHold
        notHold = newNotHold
        hold = newHold
    
    return notHold
```

---

## 🌐 Generalized Cooldown

```
What if cooldown = C days?

hold[i] = max(hold[i-1], notHold[i-C-1] - prices[i])

Need to remember notHold from C+1 days ago.
Maintain a sliding window or buffer of size C+1.

function maxProfit_cooldownC(prices, C):
    n = len(prices)
    hold = -prices[0]
    notHold = array of last C+1 values of notHold, all 0
    
    for i = 1 to n-1:
        newNotHold = max(notHold[current], hold + prices[i])
        newHold = max(hold, notHold[(current-C) mod (C+1)] - prices[i])
        // update circular buffer
    
    return notHold[current]
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **States** | hold, sold (cooldown), rest (can buy) |
| **Buy rule** | Only from rest (not from sold) |
| **Sell rule** | From hold → sold |
| **Rest rule** | From sold → rest, or rest → rest |
| **Complexity** | O(n) time, O(1) space |
| **2-state alt** | notHold[i-2] for buy (look back 2 days) |

---

## ❓ Quick Revision Questions

1. **What are the three states and their meanings?**
2. **Why can't we buy from the "sold" state?**
3. **How does the 2-state formulation handle cooldown?**
4. **What is the base case for the "sold" state?**
5. **How would you generalize to a C-day cooldown?**
6. **Draw the state machine transition diagram from memory.**

---

[← Previous: Multiple Transactions](03-multiple-transactions.md) | [Next: With Transaction Fee →](05-with-fee.md)

[← Back to README](../README.md)
