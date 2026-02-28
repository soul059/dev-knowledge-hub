# Chapter 5: Target Sum

## 📋 Overview
Given an array of non-negative integers and a target, assign **+ or -** to each number so their sum equals the target. Count the number of ways. This problem beautifully transforms into a **Subset Sum counting** problem through algebraic manipulation.

---

## 🧠 Core Concept

```
nums = [1, 1, 1, 1, 1],  target = 3

Assign + or - to each:
  +1 +1 +1 +1 -1 = 3  ✓
  +1 +1 +1 -1 +1 = 3  ✓
  +1 +1 -1 +1 +1 = 3  ✓
  +1 -1 +1 +1 +1 = 3  ✓
  -1 +1 +1 +1 +1 = 3  ✓

Answer: 5 ways
```

---

## 🔨 Transformation to Subset Sum

```
Let P = set of numbers assigned +
Let N = set of numbers assigned -

sum(P) - sum(N) = target            ... (1)
sum(P) + sum(N) = totalSum           ... (2)

Adding (1) and (2):
  2 × sum(P) = target + totalSum
  sum(P) = (target + totalSum) / 2

So: count subsets with sum = (target + totalSum) / 2

Validity checks:
  (target + totalSum) must be even
  |target| ≤ totalSum
  (target + totalSum) / 2 ≥ 0

Example: nums=[1,1,1,1,1], target=3
  totalSum = 5
  sum(P) = (3 + 5) / 2 = 4
  Count subsets summing to 4:
    {1,1,1,1} → 5 ways (choose any 4 of the five 1s)
    C(5,4) = 5 ✓
```

---

## 💻 Pseudocode

### Transformed Subset Sum Count
```
function findTargetSumWays(nums, target):
    totalSum = sum(nums)
    
    // Validity checks
    if (target + totalSum) % 2 != 0: return 0
    if abs(target) > totalSum: return 0
    
    subsetSum = (target + totalSum) / 2
    if subsetSum < 0: return 0
    
    // Count subsets with given sum (0/1 knapsack counting)
    dp = array of size subsetSum+1, all 0
    dp[0] = 1    // one way to make sum 0: empty subset
    
    for num in nums:
        for s = subsetSum down to num:
            dp[s] += dp[s - num]
    
    return dp[subsetSum]
```

### Alternative: Direct DP with Offset
```
Without transformation — track all possible sums:

function findTargetSumWays(nums, target):
    // possible sums range from -totalSum to +totalSum
    // use offset = totalSum to avoid negative indices
    
    totalSum = sum(nums)
    dp = map/dict {0: 1}   // sum → count of ways
    
    for num in nums:
        next_dp = {}
        for s, count in dp:
            next_dp[s + num] += count
            next_dp[s - num] += count
        dp = next_dp
    
    return dp.get(target, 0)

Time: O(n × totalSum), Space: O(totalSum)
```

---

## 🔬 Trace: nums=[1,1,1,1,1], target=3

```
Transformed approach:
  totalSum = 5, subsetSum = (3+5)/2 = 4
  
  dp = [1, 0, 0, 0, 0]
        0  1  2  3  4

  num=1 (s: 4→1):
    dp[4]+=dp[3]=0, dp[3]+=dp[2]=0, dp[2]+=dp[1]=0, dp[1]+=dp[0]=1
    dp = [1, 1, 0, 0, 0]

  num=1 (s: 4→1):
    dp[4]+=dp[3]=0, dp[3]+=dp[2]=0, dp[2]+=dp[1]=1, dp[1]+=dp[0]=1
    dp = [1, 2, 1, 0, 0]

  num=1 (s: 4→1):
    dp[4]+=dp[3]=0, dp[3]+=dp[2]=1, dp[2]+=dp[1]=2, dp[1]+=dp[0]=1
    dp = [1, 3, 3, 1, 0]

  num=1 (s: 4→1):
    dp[4]+=dp[3]=1, dp[3]+=dp[2]=3, dp[2]+=dp[1]=3, dp[1]+=dp[0]=1
    dp = [1, 4, 6, 4, 1]

  num=1 (s: 4→1):
    dp[4]+=dp[3]=4, dp[3]+=dp[2]=6, dp[2]+=dp[1]=4, dp[1]+=dp[0]=1
    dp = [1, 5, 10, 10, 5]

  Answer: dp[4] = 5 ✓

  (Notice: dp looks like Pascal's triangle row → C(5,k) pattern!)
```

---

## ⚠️ Edge Case: Zeros

```
nums = [0, 1], target = 1

totalSum = 1, subsetSum = (1+1)/2 = 1

dp = [1, 0]

num=0 (s: 1→0):
  dp[0] += dp[0] = 2   // 0 can go to P or N
  dp = [2, 0]

num=1 (s: 1→1):
  dp[1] += dp[0] = 2
  dp = [2, 2]

Answer: 2 ("+0+1" and "-0+1")

Each zero DOUBLES the count because +0 = -0.
If there are k zeros: multiply answer by 2^k.
```

---

## 🌐 Comparison of Approaches

```
               ┌──────────────────┬──────────────┬──────────┐
               │ Approach         │ Time         │ Space    │
               ├──────────────────┼──────────────┼──────────┤
               │ Brute Force      │ O(2^n)       │ O(n)     │
               │ Memoized DFS     │ O(n×totalSum)│ O(n×sum) │
               │ Direct DP (map)  │ O(n×totalSum)│ O(sum)   │
               │ Subset Sum Count │ O(n×sum/2)   │ O(sum/2) │
               └──────────────────┴──────────────┴──────────┘

Subset Sum transform is best:
  - Half the state space (sum/2 vs totalSum)
  - Simple 1D array instead of map
  - Clean implementation
```

---

## 🧩 Related Problems

```
1. Number of ways to add/subtract to reach target
   → Direct application

2. Count subsets with given difference
   S1 - S2 = diff, S1 + S2 = total
   → S1 = (diff + total) / 2
   → Same transformation!

3. Expression evaluation with + and -
   → Target sum variant with expression result
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Transformation** | sum(P) = (target + totalSum) / 2 |
| **Reduction** | Count subsets with sum = (target + totalSum) / 2 |
| **Validity** | (target + totalSum) must be even, \|target\| ≤ totalSum |
| **State** | dp[s] = number of subsets summing to s |
| **Complexity** | O(n × target') time, O(target') space |
| **Zero handling** | Each 0 doubles the count |

---

## ❓ Quick Revision Questions

1. **How do you transform Target Sum into Subset Sum?**
2. **What are the validity checks before running DP?**
3. **Why does each zero in the array double the answer?**
4. **What is sum(P) in terms of target and totalSum?**
5. **Why is the subset sum approach faster than the direct DP approach?**
6. **How would you solve "count subsets with given difference" using the same idea?**

---

[← Previous: Partition Equal Subset Sum](04-partition-equal-subset.md) | [Next: Coin Change (Knapsack View) →](06-coin-change-knapsack.md)

[← Back to README](../README.md)
