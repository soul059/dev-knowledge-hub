# Chapter 2: Burst Balloons

## 📋 Overview
Given balloons with numbers, bursting balloon `i` earns `nums[i-1] × nums[i] × nums[i+1]` coins. Find the **maximum coins** by bursting all balloons. The trick: think about which balloon to burst **last** rather than first.

---

## 🧠 Core Concept

```
nums = [3, 1, 5, 8]

After padding with 1s: [1, 3, 1, 5, 8, 1]
                        ^                ^
                     boundary         boundary

If we burst in order 1, 5, 3, 8:
  Burst 1: 3×1×5 = 15   remaining: [3, 5, 8]
  Burst 5: 3×5×8 = 120  remaining: [3, 8]
  Burst 3: 1×3×8 = 24   remaining: [8]
  Burst 8: 1×8×1 = 8    remaining: []
  Total: 15 + 120 + 24 + 8 = 167

Optimal order: 3, 1, 5, 8
  Burst 3: 1×3×1 = 3    → no wait...

Actually optimal is 167 (order varies).

KEY INSIGHT: When we burst a balloon, neighbors change.
This makes "first to burst" thinking complicated.
Instead: think about which balloon is LAST to burst!
```

---

## 🔑 Why "Last to Burst" Works

```
Problem with "first to burst":
  When balloon i is burst first, its neighbors become
  adjacent. The subproblems are NOT independent!
  
  [3, 1, 5, 8]
  Burst 1: [3, 5, 8] — but 3 and 5 are now adjacent
  Future bursting of 3 uses 5 as neighbor, not 1.
  Subproblems overlap in complex ways.

"Last to burst" advantage:
  If balloon k is the LAST burst in interval [i..j]:
  - All balloons except k are already burst
  - k's neighbors are bounds[i-1] and bounds[j+1] (fixed!)
  - Left subproblem [i..k-1] and right [k+1..j] are INDEPENDENT
  
  This gives clean interval DP structure!
```

---

## 🔨 Recurrence

```
Pad array: add 1 at both ends. New array has indices 0..n+1.
Balloons are at indices 1..n.

dp[i][j] = max coins from bursting ALL balloons in (i, j)
           exclusive boundaries: i and j are NOT burst

For each k in (i+1, i+2, ..., j-1):
  If k is the LAST balloon burst in range (i,j):
  
  dp[i][j] = max over k of:
      dp[i][k] + dp[k][j] + nums[i] × nums[k] × nums[j]
      \_______/   \_______/   \________________________/
      left side   right side   cost of bursting k last
      (all burst) (all burst)  (only i,k,j remain)

Base: dp[i][i+1] = 0 (no balloons between adjacent indices)
```

---

## 💻 Pseudocode

```
function maxCoins(nums):
    // Pad with 1s
    arr = [1] + nums + [1]
    n = len(arr)
    dp = 2D array [n][n], all 0
    
    // Fill by interval length
    for length = 2 to n-1:
        for i = 0 to n-length-1:
            j = i + length
            for k = i+1 to j-1:
                coins = dp[i][k] + dp[k][j] + arr[i]*arr[k]*arr[j]
                dp[i][j] = max(dp[i][j], coins)
    
    return dp[0][n-1]
```

---

## 🔬 Trace: nums = [3, 1, 5, 8]

```
arr = [1, 3, 1, 5, 8, 1]
Index: 0  1  2  3  4  5

Base: all dp[i][i+1] = 0

Length 2 (gaps of 2):
  dp[0][2]: k=1 → arr[0]*arr[1]*arr[2] = 1×3×1 = 3
  dp[1][3]: k=2 → arr[1]*arr[2]*arr[3] = 3×1×5 = 15
  dp[2][4]: k=3 → arr[2]*arr[3]*arr[4] = 1×5×8 = 40
  dp[3][5]: k=4 → arr[3]*arr[4]*arr[5] = 5×8×1 = 40

Length 3:
  dp[0][3]: 
    k=1: dp[0][1]+dp[1][3]+1×3×5 = 0+15+15 = 30
    k=2: dp[0][2]+dp[2][3]+1×1×5 = 3+0+5 = 8
    dp[0][3] = 30

  dp[1][4]:
    k=2: dp[1][2]+dp[2][4]+3×1×8 = 0+40+24 = 64
    k=3: dp[1][3]+dp[3][4]+3×5×8 = 15+0+120 = 135
    dp[1][4] = 135

  dp[2][5]:
    k=3: dp[2][3]+dp[3][5]+1×5×1 = 0+40+5 = 45
    k=4: dp[2][4]+dp[4][5]+1×8×1 = 40+0+8 = 48
    dp[2][5] = 48

Length 4:
  dp[0][4]:
    k=1: dp[0][1]+dp[1][4]+1×3×8 = 0+135+24 = 159
    k=2: dp[0][2]+dp[2][4]+1×1×8 = 3+40+8 = 51
    k=3: dp[0][3]+dp[3][4]+1×5×8 = 30+0+40 = 70
    dp[0][4] = 159

  dp[1][5]:
    k=2: dp[1][2]+dp[2][5]+3×1×1 = 0+48+3 = 51
    k=3: dp[1][3]+dp[3][5]+3×5×1 = 15+40+15 = 70
    k=4: dp[1][4]+dp[4][5]+3×8×1 = 135+0+24 = 159
    dp[1][5] = 159

Length 5:
  dp[0][5]:
    k=1: dp[0][1]+dp[1][5]+1×3×1 = 0+159+3 = 162
    k=2: dp[0][2]+dp[2][5]+1×1×1 = 3+48+1 = 52
    k=3: dp[0][3]+dp[3][5]+1×5×1 = 30+40+5 = 75
    k=4: dp[0][4]+dp[4][5]+1×8×1 = 159+0+8 = 167
    dp[0][5] = 167

Answer: 167
```

---

## 🧩 Memoized Top-Down (Alternative)

```
function maxCoins(nums):
    arr = [1] + nums + [1]
    n = len(arr)
    memo = {}
    
    function dp(i, j):
        if j - i < 2: return 0
        if (i,j) in memo: return memo[(i,j)]
        
        result = 0
        for k = i+1 to j-1:
            coins = dp(i,k) + dp(k,j) + arr[i]*arr[k]*arr[j]
            result = max(result, coins)
        
        memo[(i,j)] = result
        return result
    
    return dp(0, n-1)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Key insight** | Think about which balloon to burst LAST |
| **Padding** | Add 1 at both ends of array |
| **State** | dp[i][j] = max coins within exclusive bounds (i,j) |
| **Recurrence** | max over k: dp[i][k] + dp[k][j] + arr[i]×arr[k]×arr[j] |
| **Base** | dp[i][i+1] = 0 (empty interval) |
| **Complexity** | O(n³) time, O(n²) space |

---

## ❓ Quick Revision Questions

1. **Why is thinking about "first to burst" difficult?**
2. **What does "last to burst" give us in terms of subproblem independence?**
3. **Why do we pad the array with 1s?**
4. **What does dp[i][j] represent? Are i and j inclusive?**
5. **What are the boundaries when balloon k is burst last in interval (i,j)?**
6. **How does this relate to Matrix Chain Multiplication?**

---

[← Previous: Matrix Chain Multiplication](01-matrix-chain-multiplication.md) | [Next: Merge Stones →](03-merge-stones.md)

[← Back to README](../README.md)
