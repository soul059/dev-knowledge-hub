# Chapter 3: Merge Stones

## 📋 Overview
Given piles of stones in a row, merge **K consecutive piles** at a time. The cost of each merge is the total number of stones in those K piles. Find the **minimum cost** to merge all piles into one. When K=2, this is classic and always possible; for K>2, it may be impossible.

---

## 🧠 Core Concept (K=2)

```
piles = [3, 2, 4, 1]

Option 1:
  Merge [3,2] → cost 5, piles = [5, 4, 1]
  Merge [4,1] → cost 5, piles = [5, 5]
  Merge [5,5] → cost 10, piles = [10]
  Total: 5 + 5 + 10 = 20

Option 2:
  Merge [3,2] → cost 5, piles = [5, 4, 1]
  Merge [5,4] → cost 9, piles = [9, 1]
  Merge [9,1] → cost 10, piles = [10]
  Total: 5 + 9 + 10 = 24

Option 3:
  Merge [2,4] → cost 6, piles = [3, 6, 1]
  Merge [3,6] → cost 9, piles = [9, 1]
  Merge [9,1] → cost 10, piles = [10]
  Total: 6 + 9 + 10 = 25

Optimal: 20 (option 1)
```

---

## 🔨 K=2: Classic Interval DP

```
prefix[i] = sum of piles[0..i-1]
cost to merge interval [i,j] once = prefix[j+1] - prefix[i]

dp[i][j] = min cost to merge piles i..j into ONE pile

Recurrence (K=2):
  dp[i][j] = min over k from i to j-1 of:
      dp[i][k] + dp[k+1][j] + sum(piles[i..j])
      \_______/   \________/   \_____________/
      merge left   merge right  cost of final merge

Base: dp[i][i] = 0

function mergeStones2(piles):
    n = len(piles)
    prefix = prefix sum array
    dp = 2D array [n][n], all 0
    
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            dp[i][j] = ∞
            for k = i to j-1:
                dp[i][j] = min(dp[i][j], 
                    dp[i][k] + dp[k+1][j] + prefix[j+1] - prefix[i])
    
    return dp[0][n-1]
```

---

## 🔬 Trace (K=2): piles = [3, 2, 4, 1]

```
prefix = [0, 3, 5, 9, 10]

Length 2:
  dp[0][1] = dp[0][0]+dp[1][1]+sum(0,1) = 0+0+5 = 5
  dp[1][2] = 0+0+6 = 6
  dp[2][3] = 0+0+5 = 5

Length 3:
  dp[0][2]:
    k=0: dp[0][0]+dp[1][2]+sum(0,2) = 0+6+9 = 15
    k=1: dp[0][1]+dp[2][2]+sum(0,2) = 5+0+9 = 14
    dp[0][2] = 14
  
  dp[1][3]:
    k=1: dp[1][1]+dp[2][3]+sum(1,3) = 0+5+7 = 12
    k=2: dp[1][2]+dp[3][3]+sum(1,3) = 6+0+7 = 13
    dp[1][3] = 12

Length 4:
  dp[0][3]:
    k=0: dp[0][0]+dp[1][3]+sum(0,3) = 0+12+10 = 22
    k=1: dp[0][1]+dp[2][3]+sum(0,3) = 5+5+10 = 20
    k=2: dp[0][2]+dp[3][3]+sum(0,3) = 14+0+10 = 24
    dp[0][3] = 20

Answer: 20 ✓
```

---

## 🔨 General K: Merge K Consecutive

```
When can n piles be merged into 1 with K at a time?
  Each merge reduces pile count by K-1.
  Starting with n piles, after m merges: n - m×(K-1)
  For final 1 pile: n - m×(K-1) = 1 → m = (n-1)/(K-1)
  Required: (n-1) % (K-1) == 0

dp[i][j] = min cost to merge piles i..j into as few piles as possible

Recurrence:
  dp[i][j] = min over k (step K-1) of:
      dp[i][k] + dp[k+1][j]      // split into two groups
  
  // If interval can be fully merged (length-1 divisible by K-1):
  if (j - i) % (K-1) == 0:
      dp[i][j] += prefix[j+1] - prefix[i]   // add merge cost

function mergeStonesK(piles, K):
    n = len(piles)
    if (n - 1) % (K - 1) != 0: return -1
    
    prefix = prefix sum array
    dp = 2D array [n][n], all 0
    
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            dp[i][j] = ∞
            // Split points with step K-1
            for k = i to j-1 step (K-1):
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j])
            // If this interval can merge to 1 pile
            if (j - i) % (K - 1) == 0:
                dp[i][j] += prefix[j+1] - prefix[i]
    
    return dp[0][n-1]
```

---

## 🔑 Understanding the K>2 Step

```
Why step K-1 in split point?

K=3: merge 3 piles at a time.
  To merge [i..j], we need left part to form some piles
  and right part to form some piles, totaling = K or
  some configuration that allows K-merges.

  Left part should reduce to 1 pile: length-1 divisible by K-1
  So k increments by K-1: k = i, i+(K-1), i+2(K-1), ...

Example: K=3, piles=[1,2,3,4,5]
  n=5, (5-1)%(3-1) = 4%2 = 0 → possible

  Split at k=i, i+2, i+4... ensures left always 
  forms exactly 1 pile after full merge.
```

---

## 🌐 Variants

```
┌───────────────────────────────────┬──────────────────────────┐
│ Variant                          │ Key Change               │
├───────────────────────────────────┼──────────────────────────┤
│ K=2 (classic)                    │ Split at every k         │
│ General K                        │ Split with step K-1      │
│ Circular merge                   │ Double the array         │
│ Minimize max pile (instead of    │ Different optimization   │
│   total cost)                    │   target                 │
│ Huffman Coding (non-consecutive) │ Greedy with priority     │
│                                  │   queue (not interval DP)│
└───────────────────────────────────┴──────────────────────────┘

Note: If piles are NOT required to be consecutive,
      use Huffman coding (greedy) instead of DP!
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min cost to optimally reduce piles i..j |
| **K=2** | Classic MCM-like interval DP |
| **General K** | Split at steps of K-1; check (n-1)%(K-1)==0 |
| **Merge cost** | sum(piles[i..j]) = prefix[j+1] - prefix[i] |
| **Complexity** | O(n³) time (K=2), O(n³/K) for general K |
| **Impossible** | (n-1) % (K-1) ≠ 0 → return -1 |

---

## ❓ Quick Revision Questions

1. **How do you compute the merge cost of an interval efficiently?**
2. **When is merging K consecutive piles impossible?**
3. **Why does the split point step by K-1 for general K?**
4. **What is the relationship between this problem and Huffman coding?**
5. **How would you handle circular arrangement of piles?**
6. **What is the time complexity for K=2 vs general K?**

---

[← Previous: Burst Balloons](02-burst-balloons.md) | [Next: Strange Printer →](04-strange-printer.md)

[← Back to README](../README.md)
