# Chapter 4: Classic DP Problems

## 📋 Chapter Overview
High-frequency DP problems that combine multiple patterns: interval DP, state machine DP, and partition DP.

---

## 🔍 Problem 1: Longest Palindromic Subsequence

```
  s = "bbbab"
  LPS = "bbbb" → length 4
  
  dp[i][j] = LPS length of s[i..j]
```

```
function longestPalindromeSubseq(s):
    n = len(s)
    dp = n × n array of 0s
    
    for i = 0 to n-1: dp[i][i] = 1    // single char
    
    for len = 2 to n:
        for i = 0 to n-len:
            j = i + len - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1] + 2
            else:
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
    
    return dp[0][n-1]
```

### Trace: "bbbab"

```
      b  b  b  a  b
  b  [1, 2, 3, 3, 4]
  b  [0, 1, 2, 2, 3]
  b  [0, 0, 1, 1, 3]
  a  [0, 0, 0, 1, 1]
  b  [0, 0, 0, 0, 1]
  
  Answer: dp[0][4] = 4
```

---

## 🔍 Problem 2: Word Break

```
  s = "leetcode", wordDict = ["leet", "code"]
  Can s be segmented? → true ("leet" + "code")
```

```
function wordBreak(s, wordDict):
    n = len(s)
    wordSet = Set(wordDict)
    dp = boolean array of size n+1
    dp[0] = true
    
    for i = 1 to n:
        for j = 0 to i-1:
            if dp[j] AND s[j..i-1] in wordSet:
                dp[i] = true
                break
    
    return dp[n]
```

### Trace: "leetcode"

| i | Check substrings | dp[i] |
|---|-------------------|-------|
| 1 | "l" ✗ | false |
| 2 | "le" ✗ | false |
| 3 | "lee" ✗ | false |
| 4 | dp[0]=T, "leet" ✓ | **true** |
| 5 | "leetc" ✗ | false |
| 6 | "leetco" ✗ | false |
| 7 | "leetcod" ✗ | false |
| 8 | dp[4]=T, "code" ✓ | **true** ✓ |

---

## 🔍 Problem 3: Buy and Sell Stock (State Machine)

### At Most One Transaction

```
function maxProfit(prices):
    minPrice = INF
    maxProfit = 0
    for price in prices:
        minPrice = min(minPrice, price)
        maxProfit = max(maxProfit, price - minPrice)
    return maxProfit
```

### With Cooldown (State Machine DP)

```
  States:
  HOLD:    holding a stock
  SOLD:    just sold (cooldown next)
  REST:    not holding, free to buy
  
  Transitions:
  hold[i] = max(hold[i-1], rest[i-1] - prices[i])   // keep or buy
  sold[i] = hold[i-1] + prices[i]                    // sell
  rest[i] = max(rest[i-1], sold[i-1])                // wait
```

```
           ┌──────┐
     buy   │      │  keep
  REST ───→│ HOLD │←──┘
  ↑    ←───│      │
  │  cool  └──────┘
  │    down    │ sell
  │            ↓
  └──── SOLD ──┘
```

```
function maxProfit(prices):
    hold = -INF, sold = 0, rest = 0
    for price in prices:
        prevHold = hold
        hold = max(hold, rest - price)
        rest = max(rest, sold)
        sold = prevHold + price
    return max(sold, rest)
```

---

## 🔍 Problem 4: Partition Equal Subset Sum

```
  nums = [1, 5, 11, 5]
  Total = 22, target = 11
  Can partition into two subsets with equal sum?
  [1, 5, 5] and [11] → YES
```

```
function canPartition(nums):
    total = sum(nums)
    if total % 2 != 0: return false
    target = total / 2
    
    dp = boolean array of size target+1
    dp[0] = true
    
    for num in nums:
        for j = target down to num:    // backwards (0/1 knapsack)
            dp[j] = dp[j] OR dp[j - num]
    
    return dp[target]
```

---

## 🔍 Problem 5: Matrix Chain Multiplication / Burst Balloons

### Burst Balloons (LeetCode 312)

```
  Burst balloons to maximize coins.
  When you burst balloon i, you get nums[i-1] × nums[i] × nums[i+1]
```

```
function maxCoins(nums):
    // Add boundary 1s
    arr = [1] + nums + [1]
    n = len(arr)
    dp = n × n array of 0s
    
    // dp[i][j] = max coins from bursting all balloons between i and j
    for length = 2 to n-1:
        for i = 0 to n-length-1:
            j = i + length
            for k = i+1 to j-1:        // k = last balloon to burst
                coins = arr[i] * arr[k] * arr[j]
                dp[i][j] = max(dp[i][j], dp[i][k] + coins + dp[k][j])
    
    return dp[0][n-1]
```

### Key Insight: Think About the LAST Action

```
  Instead of "which balloon to burst first?"
  Think: "which balloon to burst LAST in range [i,j]?"
  
  If k is burst last between i and j:
  - Left side dp[i][k] is already resolved
  - Right side dp[k][j] is already resolved
  - k's neighbors are now i and j (boundaries)
  - Coins = arr[i] × arr[k] × arr[j]
```

---

## 🔍 Problem 6: Coin Change 2 (Count Ways)

```
  coins = [1, 2, 5], amount = 5
  How many combinations? → 4
  {5}, {2+2+1}, {2+1+1+1}, {1+1+1+1+1}
```

```
function change(amount, coins):
    dp = array of 0s, size amount+1
    dp[0] = 1
    
    for coin in coins:                    // iterate coins in outer loop
        for a = coin to amount:           // FORWARDS (unbounded)
            dp[a] += dp[a - coin]
    
    return dp[amount]
```

**Why outer loop over coins?** → Avoids counting permutations (e.g., {1,2} and {2,1} as different).

---

## 📊 DP Pattern Categories

| Category | State | Examples |
|----------|-------|----------|
| Linear | dp[i] | House robber, climbing stairs |
| Grid | dp[i][j] | Unique paths, min path sum |
| String | dp[i][j] | LCS, edit distance |
| Interval | dp[i][j] range | Palindrome, burst balloons |
| Knapsack | dp[i] or dp[i][w] | Subset sum, coin change |
| State machine | Multiple dp arrays | Stock problems, regex |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Palindrome subseq | Interval DP: expand from single chars |
| Word break | dp[i] = can segment s[0..i-1] |
| Stock state machine | Model states (hold/sold/rest) and transitions |
| Partition subset | Reduce to subset sum = total/2 |
| Burst balloons | Think about LAST action, not first |
| Coin change count | Outer loop over coins → combinations (not permutations) |

---

## ❓ Revision Questions

1. **Burst Balloons: why think about LAST balloon?** → If we think about first, the subproblems change (neighbors shift). Last-to-burst keeps boundaries fixed.
2. **Word Break transition?** → `dp[i] = true` if any `dp[j]` is true and `s[j..i-1]` is a word.
3. **Stock with cooldown: what are the states?** → HOLD (have stock), SOLD (just sold, must cooldown), REST (free to buy).
4. **Coin Change 2: why outer loop over coins?** → To count combinations, not permutations (each coin is considered once).
5. **Partition equal subset: why is it a knapsack problem?** → It's equivalent to: "does a subset exist that sums to total/2?" → subset sum → 0/1 knapsack.

---

[← Previous: 2D DP Patterns](03-2d-dp-patterns.md) | [Next: DP Optimization →](05-dp-optimization.md)
