# Chapter 4: Coin Change (Minimum Coins)

## 📋 Overview
**Problem:** Given coins of different denominations and a total amount, find the **minimum number of coins** needed to make that amount. If it's not possible, return -1. You have infinite supply of each denomination.

---

## 🧠 Problem Visualization

```
Coins: [1, 5, 6]    Amount: 11

┌───────────────────────────────────────────┐
│  How to make 11 with fewest coins?        │
│                                           │
│  Option A: 11 × $1           = 11 coins   │
│  Option B: 2×$5 + 1×$1      = 3 coins    │
│  Option C: 1×$6 + 1×$5      = 2 coins ✓  │
│  Option D: 1×$6 + 5×$1      = 6 coins    │
│                                           │
│  Greedy (largest first):                  │
│    $6 + $5 = 11 → 2 coins ✓ (works here) │
│                                           │
│  Greedy fails for coins=[1,3,4], amount=6:│
│    Greedy: $4+$1+$1 = 3 coins             │
│    Optimal: $3+$3   = 2 coins ✓           │
└───────────────────────────────────────────┘
```

---

## 🔍 DP Development

### Step 1: Define State
```
dp[i] = minimum number of coins to make amount i
```

### Step 2: Recurrence
```
For each amount i, try EVERY coin c:
  If we use coin c, we need 1 + dp[i-c] more coins
  
dp[i] = min(dp[i - c] + 1) for all coins c where c ≤ i

dp[0] = 0 (base: 0 coins for amount 0)
dp[i] = ∞ initially (impossible until proven otherwise)
```

### Step 3: Pseudocode
```
function coinChange(coins, amount):
    dp = array[amount+1], filled with ∞
    dp[0] = 0
    
    for i = 1 to amount:
        for each coin c in coins:
            if c <= i and dp[i-c] + 1 < dp[i]:
                dp[i] = dp[i-c] + 1
    
    return dp[amount] if dp[amount] != ∞ else -1

Time: O(amount × |coins|) | Space: O(amount)
```

---

## 🧪 Step-by-Step Trace

```
Coins = [1, 5, 6], Amount = 11

i  │ Try c=1      │ Try c=5      │ Try c=6      │ dp[i]
───┼──────────────┼──────────────┼──────────────┼──────
0  │     -        │     -        │     -        │  0 (base)
1  │ dp[0]+1 = 1  │ i<5, skip    │ i<6, skip    │  1
2  │ dp[1]+1 = 2  │ skip         │ skip         │  2
3  │ dp[2]+1 = 3  │ skip         │ skip         │  3
4  │ dp[3]+1 = 4  │ skip         │ skip         │  4
5  │ dp[4]+1 = 5  │ dp[0]+1 = 1  │ skip         │  1
6  │ dp[5]+1 = 2  │ dp[1]+1 = 2  │ dp[0]+1 = 1  │  1
7  │ dp[6]+1 = 2  │ dp[2]+1 = 3  │ dp[1]+1 = 2  │  2
8  │ dp[7]+1 = 3  │ dp[3]+1 = 4  │ dp[2]+1 = 3  │  3
9  │ dp[8]+1 = 4  │ dp[4]+1 = 5  │ dp[3]+1 = 4  │  4
10 │ dp[9]+1 = 5  │ dp[5]+1 = 2  │ dp[4]+1 = 5  │  2
11 │ dp[10]+1 = 3 │ dp[6]+1 = 2  │ dp[5]+1 = 2  │  2

Answer: dp[11] = 2 (coins: 5 + 6)

DP Table:
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │ 1 │ 1 │ 2 │ 3 │ 4 │ 2 │ 2 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
  0   1   2   3   4   5   6   7   8   9  10  11
```

---

## 📐 Understanding the Recurrence

```
To make amount 11, the LAST coin used was either 1, 5, or 6:

                     dp[11] = 2
                    / |       \
            coin=1/  coin=5|   \coin=6
                 /    |      \
           dp[10]=2  dp[6]=1  dp[5]=1
           
  Using coin 1: 1 + dp[10] = 1 + 2 = 3
  Using coin 5: 1 + dp[6]  = 1 + 1 = 2 ← min
  Using coin 6: 1 + dp[5]  = 1 + 1 = 2 ← min

  dp[11] = min(3, 2, 2) = 2
```

---

## 💡 Reconstructing the Solution

```
To find WHICH coins were used, trace back:

function reconstructCoins(coins, amount, dp):
    result = []
    while amount > 0:
        for each coin c in coins:
            if amount >= c and dp[amount] == dp[amount-c] + 1:
                result.append(c)
                amount -= c
                break
    return result

Trace: amount=11 → dp[11]=2
  Try c=5: dp[11]==dp[6]+1? 2==1+1? YES → use 5, amount=6
  Try c=6: dp[6]==dp[0]+1?  1==0+1? YES → use 6, amount=0
  Result: [5, 6]
```

---

## 📊 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force (recursion) | O(amount^|coins|) | O(amount) | Exponential |
| Memoization | O(amount × |coins|) | O(amount) | Top-down |
| **Tabulation** | **O(amount × |coins|)** | **O(amount)** | **Best** |
| Greedy | O(amount) | O(1) | Wrong for some inputs! |

---

## ❓ Quick Revision Questions

1. **Why does the Greedy approach fail for some coin denominations?**
2. **What should dp[i] be initialized to and why?**
3. **What does dp[0] = 0 mean conceptually?**
4. **For coins=[2] and amount=3, what is the answer?**
5. **How would you reconstruct which coins were used?**
6. **What is the time complexity and what are the two factors?**

---

[← Previous: Maximum Subarray](03-maximum-subarray.md) | [Next: Coin Change 2 (Ways) →](05-coin-change-ways.md)

[← Back to README](../README.md)
