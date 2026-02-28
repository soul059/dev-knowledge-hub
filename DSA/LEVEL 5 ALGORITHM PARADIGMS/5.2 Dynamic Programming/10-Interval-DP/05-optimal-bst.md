# Chapter 5: Optimal Binary Search Tree

## 📋 Overview
Given keys with known search frequencies, construct a BST that **minimizes the expected search cost**. This is a classic interval DP problem where we try every possible root for each interval of keys.

---

## 🧠 Core Concept

```
Keys:        [10, 20, 30]
Frequencies: [ 3,  2,  5]   (how often each key is searched)

BST Option 1 (root=20):          BST Option 2 (root=10):
        20                            10
       /  \                             \
     10    30                           20
                                          \
  Cost:                                   30
  20: depth 1 × freq 2 = 2
  10: depth 2 × freq 3 = 6       Cost:
  30: depth 2 × freq 5 = 10      10: depth 1 × freq 3 = 3
  Total: 18                       20: depth 2 × freq 2 = 4
                                  30: depth 3 × freq 5 = 15
BST Option 3 (root=30):          Total: 22
      30
     /
    20                            Best: Option 1, cost 18
   /
  10
  Cost: 30×1×5=5, 20×2×2=4, 10×3×3=9 = 18

Hmm, two have cost 18. Let's verify...
Actually root=30: 30: 1×5=5, 20: 2×2=4, 10: 3×3=9 → 18 too!
```

---

## 🔨 Recurrence

```
Keys sorted: key[1], key[2], ..., key[n]
Frequency: freq[i] for key[i]
Prefix sum: W(i,j) = sum of freq[i..j]

dp[i][j] = min expected cost for BST of keys i..j

If we choose key r as root of keys i..j:
  - Left subtree: keys i..r-1 → cost dp[i][r-1]
  - Right subtree: keys r+1..j → cost dp[r+1][j]
  - All nodes go 1 level deeper under root → add W(i,j)
  
dp[i][j] = min over r from i to j of:
    dp[i][r-1] + dp[r+1][j] + W(i,j)

(W(i,j) because when we root at r, every node in the subtree
 has its depth increased by 1, adding freq sum to total cost)

Base: dp[i][i] = freq[i]  (single node, depth 1)
      dp[i][i-1] = 0      (empty subtree)
```

---

## 💻 Pseudocode

```
function optimalBST(keys, freq):
    n = len(keys)
    
    // Prefix sum for range frequency sum
    W = 2D array where W[i][j] = sum(freq[i..j])
    
    dp = 2D array [n+2][n+1], all 0
    
    // Base: single keys
    for i = 1 to n:
        dp[i][i] = freq[i]
    
    // Fill by interval length
    for length = 2 to n:
        for i = 1 to n - length + 1:
            j = i + length - 1
            dp[i][j] = ∞
            for r = i to j:
                cost = dp[i][r-1] + dp[r+1][j] + W[i][j]
                dp[i][j] = min(dp[i][j], cost)
    
    return dp[1][n]
```

---

## 🔬 Trace: freq = [3, 2, 5] (keys 1,2,3)

```
W[1][1]=3, W[2][2]=2, W[3][3]=5
W[1][2]=5, W[2][3]=7
W[1][3]=10

Base:
  dp[1][1]=3, dp[2][2]=2, dp[3][3]=5

Length 2:
  dp[1][2] (keys 1,2):
    r=1: dp[1][0]+dp[2][2]+W[1][2] = 0+2+5 = 7
    r=2: dp[1][1]+dp[3][2]+W[1][2] = 3+0+5 = 8
    dp[1][2] = 7 (root=key1)

  dp[2][3] (keys 2,3):
    r=2: dp[2][1]+dp[3][3]+W[2][3] = 0+5+7 = 12
    r=3: dp[2][2]+dp[4][3]+W[2][3] = 2+0+7 = 9
    dp[2][3] = 9 (root=key3)

Length 3:
  dp[1][3] (keys 1,2,3):
    r=1: dp[1][0]+dp[2][3]+W[1][3] = 0+9+10 = 19
    r=2: dp[1][1]+dp[3][3]+W[1][3] = 3+5+10 = 18
    r=3: dp[1][2]+dp[4][3]+W[1][3] = 7+0+10 = 17
    dp[1][3] = 17 (root=key3)

Hmm wait — let me recheck with root=3:
  BST:    30(key3)
         /
        10(key1)
          \
          20(key2)
  
  Cost: 30: depth 1 × 5 = 5
        10: depth 2 × 3 = 6
        20: depth 3 × 2 = 6
        Total: 17 ✓

Answer: 17
```

---

## ⚡ Knuth's Optimization

```
Observation (Knuth 1971):
  The optimal root for [i..j] is between the optimal roots
  for [i..j-1] and [i+1..j]:
  
  root[i][j-1] ≤ root[i][j] ≤ root[i+1][j]

This limits the search range for r, reducing time to O(n²)!

function optimalBST_optimized(keys, freq):
    n = len(keys)
    dp = 2D array, root = 2D array
    
    for i = 1 to n:
        dp[i][i] = freq[i]
        root[i][i] = i
    
    for length = 2 to n:
        for i = 1 to n-length+1:
            j = i + length - 1
            dp[i][j] = ∞
            // Knuth's optimization: restrict r range
            for r = root[i][j-1] to root[i+1][j]:
                cost = dp[i][r-1] + dp[r+1][j] + W[i][j]
                if cost < dp[i][j]:
                    dp[i][j] = cost
                    root[i][j] = r
    
    return dp[1][n]

Time: O(n²) — significant improvement over O(n³)!
```

---

## 🌐 With Unsuccessful Search (Dummy Keys)

```
Extended problem includes "dummy keys" for unsuccessful searches:
  d[0], d[1], ..., d[n] where d[i] represents searches
  between key[i] and key[i+1].

  Weight: W(i,j) = sum(freq[i..j]) + sum(dummy[i-1..j])
  
This handles the full OBST problem as in CLRS textbook.

The recurrence remains the same, just W changes.
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min search cost for BST of keys i..j |
| **Root choice** | Try every key r in [i..j] as root |
| **Cost formula** | dp[i][r-1] + dp[r+1][j] + W(i,j) |
| **W(i,j)** | Sum of frequencies in range — depth penalty |
| **Complexity** | O(n³) naive, O(n²) with Knuth's optimization |
| **Knuth's insight** | root[i][j-1] ≤ root[i][j] ≤ root[i+1][j] |

---

## ❓ Quick Revision Questions

1. **Why does choosing root r add W(i,j) to the cost?**
2. **What is Knuth's optimization and why does it work?**
3. **How do dummy keys affect the weight calculation?**
4. **What is the time complexity before and after Knuth's optimization?**
5. **How does this relate to MCM structurally?**
6. **Can greedy choose the most frequent key as root?**

---

[← Previous: Strange Printer](04-strange-printer.md) | [Next: Palindrome Partitioning (Interval) →](06-palindrome-partitioning-interval.md)

[← Back to README](../README.md)
