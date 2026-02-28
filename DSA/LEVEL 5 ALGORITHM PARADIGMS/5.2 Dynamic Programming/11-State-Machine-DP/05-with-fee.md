# Chapter 5: Buy & Sell Stock with Transaction Fee

## 📋 Overview
Each transaction (buy-sell pair) incurs a **fee**. This discourages frequent trading — you only trade when the profit exceeds the fee. The state machine remains 2-state but transitions include the fee.

---

## 🧠 Core Concept

```
prices = [1, 3, 2, 8, 4, 9],  fee = 2

Without fee (unlimited):
  All upward moves: (3-1)+(8-2)+(9-4) = 2+6+5 = 13

With fee:
  Each transaction costs 2 extra.
  
  Option 1: buy@1, sell@8, buy@4, sell@9
    Profit = (8-1-2) + (9-4-2) = 5+3 = 8 ✓
    
  Option 2: buy@1, sell@3, buy@2, sell@8, buy@4, sell@9
    Profit = (3-1-2) + (8-2-2) + (9-4-2) = 0+4+3 = 7
    
  Option 3: buy@1, sell@9
    Profit = 9-1-2 = 6

Best: 8 (option 1)
Fee makes us prefer fewer, larger transactions.
```

---

## 🔨 State Machine (Same as Unlimited + Fee)

```
  ┌──────────┐   buy (-price)    ┌──────────┐
  │   NOT    │──────────────────→│          │
  │ HOLDING  │                   │ HOLDING  │
  │          │←──────────────────│          │
  └──────────┘  sell (+price-fee)└──────────┘
       ↑ rest                        ↑ rest
       └──┘                          └──┘

notHold[i] = max(notHold[i-1],             // rest
                 hold[i-1] + price - fee)   // sell (pay fee)

hold[i]    = max(hold[i-1],                // rest
                 notHold[i-1] - price)      // buy

Base: notHold = 0, hold = -∞
Answer: notHold at end
```

---

## 💻 Pseudocode

```
function maxProfit(prices, fee):
    notHold = 0
    hold = -∞
    
    for price in prices:
        prevNotHold = notHold
        notHold = max(notHold, hold + price - fee)
        hold = max(hold, prevNotHold - price)
    
    return notHold

// Alternative: pay fee on buy instead of sell
function maxProfit_feeOnBuy(prices, fee):
    notHold = 0
    hold = -∞
    
    for price in prices:
        prevNotHold = notHold
        notHold = max(notHold, hold + price)
        hold = max(hold, prevNotHold - price - fee)  // fee on buy
    
    return notHold

Both are equivalent — fee is paid once per transaction.
```

---

## 🔬 Trace: prices = [1, 3, 2, 8, 4, 9], fee = 2

```
notHold = 0, hold = -∞

Day 0 (price=1):
  notHold = max(0, -∞+1-2) = 0
  hold    = max(-∞, 0-1) = -1
  State: notHold=0, hold=-1

Day 1 (price=3):
  notHold = max(0, -1+3-2) = 0
  hold    = max(-1, 0-3) = -1
  State: notHold=0, hold=-1
  (selling gives 0 profit after fee — not worth it!)

Day 2 (price=2):
  notHold = max(0, -1+2-2) = 0
  hold    = max(-1, 0-2) = -1
  State: notHold=0, hold=-1

Day 3 (price=8):
  notHold = max(0, -1+8-2) = 5
  hold    = max(-1, 0-8) = -1
  State: notHold=5, hold=-1
  (Sell! profit = 8-1-2 = 5)

Day 4 (price=4):
  notHold = max(5, -1+4-2) = 5
  hold    = max(-1, 5-4) = 1
  State: notHold=5, hold=1
  (Buy at 4 with accumulated profit 5)

Day 5 (price=9):
  notHold = max(5, 1+9-2) = 8
  hold    = max(1, 5-9) = 1
  State: notHold=8, hold=1
  (Sell! profit = 5+(9-4-2) = 8)

Answer: 8 ✓
```

---

## 🔑 Effect of Fee on Strategy

```
Fee = 0: Trade at every uptick (sum all positive diffs)
Fee = small: Group close upticks into single trades
Fee = large: Only trade for significant moves

Visualization:
  Price: 1  3  2  8  4  9
  
  fee=0 trades:   buy1  sell3  buy2  sell8  buy4  sell9
  fee=2 trades:   buy1  ────────→ sell8  buy4 → sell9
  fee=5 trades:   buy1  ──────────────────────→ sell9
  
  Higher fee = fewer, longer transactions
```

---

## 🌐 Connection to Other Variants

```
┌────────────────────────────────────────────────────────────┐
│ All stock problems with unlimited transactions:            │
│                                                            │
│ Base:     notHold = max(notHold, hold + price)             │
│           hold    = max(hold, notHold - price)             │
│                                                            │
│ + Fee:    notHold = max(notHold, hold + price - fee)       │
│           (or hold = max(hold, notHold - price - fee))     │
│                                                            │
│ + Cooldown: hold = max(hold, prevPrevNotHold - price)      │
│             (look back 2 steps for buy)                    │
│                                                            │
│ + Fee + Cooldown: combine both modifications               │
│   notHold = max(notHold, hold + price - fee)               │
│   hold    = max(hold, prevPrevNotHold - price)             │
└────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Modification** | Subtract fee on sell (or add to buy cost) |
| **States** | hold, notHold (same as unlimited) |
| **Sell transition** | notHold = max(notHold, hold + price - fee) |
| **Effect** | Fewer, larger transactions |
| **Complexity** | O(n) time, O(1) space |
| **Combined** | Can add cooldown + fee together |

---

## ❓ Quick Revision Questions

1. **Where do you subtract the fee — on buy or sell?**
2. **Why does the fee discourage frequent trading?**
3. **What happens when fee = 0?**
4. **How would you combine fee AND cooldown?**
5. **Why is the state machine identical to unlimited except for fee?**
6. **What greedy approach does fee=0 reduce to?**

---

[← Previous: With Cooldown](04-with-cooldown.md) | [Next: Pattern Recognition →](06-pattern-recognition.md)

[← Back to README](../README.md)
