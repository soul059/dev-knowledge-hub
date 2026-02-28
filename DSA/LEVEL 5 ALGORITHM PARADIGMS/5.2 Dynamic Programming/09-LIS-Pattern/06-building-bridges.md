# Chapter 6: Building Bridges

## 📋 Overview
Given pairs of cities on two parallel banks of a river, build the **maximum number of non-crossing bridges**. This classic problem reduces to **LIS** after sorting by one coordinate.

---

## 🧠 Core Concept

```
North bank: [1, 2, 3, 4, 5, 6]
South bank: [6, 4, 2, 1, 5, 3]

Connections (north → south):
  1→6, 2→4, 3→2, 4→1, 5→5, 6→3

Visualization:
  North: 1  2  3  4  5  6
         |  |  |  |  |  |
         |  |\ | /|  |  |
         | /|  X  |\ | /|
         |/ | / \ | \|/ |
  South: 6  4  2  1  5  3

Bridges cross if (n1 < n2 and s1 > s2) or (n1 > n2 and s1 < s2).
Non-crossing = both coordinates in same relative order.

Maximum non-crossing set: find pairs that form an 
increasing subsequence in BOTH dimensions.

Sort by north coordinate → find LIS on south coordinates!
```

---

## 🔨 Algorithm

```
function maxBridges(pairs):
    // pairs[i] = (north_i, south_i)
    
    // Step 1: Sort by north coordinate
    sort pairs by north ascending
    // (if ties in north, sort by south ascending)
    
    // Step 2: Extract south coordinates
    south = [pair[1] for pair in sorted_pairs]
    
    // Step 3: LIS on south coordinates
    return LIS(south)

Why this works:
  After sorting by north, north coordinates are increasing.
  Two bridges (n1,s1) and (n2,s2) with n1 < n2 don't cross
  IFF s1 < s2.
  → Finding non-crossing bridges = finding increasing 
    subsequence in south coordinates.
```

---

## 🔬 Trace

```
Pairs (north, south): (1,6), (2,4), (3,2), (4,1), (5,5), (6,3)

Step 1: Already sorted by north.

Step 2: South coordinates = [6, 4, 2, 1, 5, 3]

Step 3: LIS on [6, 4, 2, 1, 5, 3]:
  
  O(n log n) approach:
  tails = []
  
  num=6: append → tails=[6]
  num=4: 4<6, replace → tails=[4]
  num=2: 2<4, replace → tails=[2]
  num=1: 1<2, replace → tails=[1]
  num=5: 5>1, append → tails=[1,5]
  num=3: 3<5, replace → tails=[1,3]
  
  LIS length = 2

Answer: 2 non-crossing bridges

Possible sets: {(4,1),(5,5)} or {(4,1),(6,3)} or {(3,2),(5,5)} etc.
```

---

## 🌐 Variant: Weighted Bridges

```
Each bridge has a weight/profit. Maximize total weight of 
non-crossing bridges.

→ Reduces to Maximum Sum Increasing Subsequence!

function maxWeightBridges(pairs, weights):
    // Sort by north
    sort pairs (and weights) by north
    
    south = [pair[1] for pair in sorted_pairs]
    
    // Max sum increasing subsequence on south values
    // with corresponding weights
    n = len(south)
    dp = copy of weights
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if south[j] < south[i]:
                dp[i] = max(dp[i], dp[j] + weights[i])
    
    return max(dp)
```

---

## 🧩 Related Problem: Longest Chain of Pairs

```
Given pairs (a, b) where a < b.
Find longest chain where pair2.a > pair1.b.

pairs = [(5,24), (15,25), (27,40), (50,60)]
Chain: (5,24) → (27,40) → (50,60)  length 3

This is also an LIS variant:
  Sort by second element (b)
  Greedy / LIS on compatibility
  
  Activity Selection connection:
  Sort by end time → greedy pick works!
  
function longestChain(pairs):
    sort pairs by pair.b ascending
    end = -∞
    count = 0
    for pair in pairs:
        if pair.a > end:
            count++
            end = pair.b
    return count

Time: O(n log n) — actually GREEDY works!
```

---

## 📊 Why Sorting + LIS Works

```
The non-crossing condition:
  Bridges (n1,s1) and (n2,s2) don't cross iff:
    (n1 < n2 AND s1 < s2) OR (n1 > n2 AND s1 > s2)
  
  i.e., both in same order.

After sorting by north:
  n1 < n2 is automatically satisfied for i < j.
  Non-crossing ↔ s1 < s2 ↔ increasing south values.
  
  Maximum non-crossing ↔ LIS of south values.

This pattern appears whenever we need to find
a maximal set of non-crossing/compatible pairs.
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Max number of non-crossing bridges |
| **Reduction** | Sort by one dimension, LIS on other |
| **Non-crossing** | Both dimensions in same relative order |
| **Complexity** | O(n log n) with binary search LIS |
| **Weighted variant** | Max Sum Increasing Subsequence |
| **Key insight** | Sorting fixes one dimension; LIS handles the other |

---

## ❓ Quick Revision Questions

1. **When do two bridges cross?**
2. **Why does sorting by one coordinate reduce to LIS?**
3. **How does the weighted variant change the problem?**
4. **What is the connection to the Russian Doll Envelopes problem?**
5. **How is Longest Chain of Pairs different from Building Bridges?**
6. **What is the optimal time complexity?**

---

[← Previous: Bitonic Subsequence](05-bitonic-subsequence.md) | [Next Unit: Interval DP →](../10-Interval-DP/01-matrix-chain-multiplication.md)

[← Back to README](../README.md)
