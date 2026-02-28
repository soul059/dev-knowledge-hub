# Chapter 4: Partition Equal Subset Sum

## 📋 Overview
Given an array of positive integers, determine if it can be partitioned into **two subsets with equal sum**. This elegantly reduces to a Subset Sum problem — check if any subset sums to `totalSum / 2`.

---

## 🧠 Core Concept

```
nums = [1, 5, 11, 5]
Total sum = 22

Can we partition into two subsets with sum 11 each?
  Subset 1: {1, 5, 5} = 11  ✓
  Subset 2: {11}       = 11  ✓

Answer: TRUE

Reduction:
  If total sum is ODD → immediately FALSE (can't split evenly)
  If total sum is EVEN → find subset with sum = totalSum / 2
                          (the rest automatically sums to totalSum / 2)
```

---

## 🔨 Algorithm

```
function canPartition(nums):
    totalSum = sum(nums)
    
    // Step 1: Quick check
    if totalSum is odd:
        return false
    
    target = totalSum / 2
    
    // Step 2: Subset sum for target
    dp = boolean array of size target+1, all false
    dp[0] = true
    
    for num in nums:
        for s = target down to num:
            dp[s] = dp[s] OR dp[s - num]
    
    return dp[target]
```

---

## 🔬 Trace: nums = [1, 5, 11, 5]

```
totalSum = 22, target = 11

dp = [T, F, F, F, F, F, F, F, F, F, F, F]
      0  1  2  3  4  5  6  7  8  9  10 11

After num=1 (s: 11→1):
  dp[1] = dp[1] OR dp[0] = T
  dp = [T, T, F, F, F, F, F, F, F, F, F, F]

After num=5 (s: 11→5):
  dp[6] = dp[6] OR dp[1] = T
  dp[5] = dp[5] OR dp[0] = T
  dp = [T, T, F, F, F, T, T, F, F, F, F, F]

After num=11 (s: 11→11):
  dp[11] = dp[11] OR dp[0] = T  ← FOUND!
  dp = [T, T, F, F, F, T, T, F, F, F, F, T]
  
  (could stop early here since dp[target] = true)

After num=5 (s: 11→5):
  dp[11] already T, dp[10] = dp[10] OR dp[5] = T, etc.
  dp = [T, T, F, F, F, T, T, F, F, F, T, T]

Answer: dp[11] = true ✓
```

---

## ⚡ Optimizations

### Early Termination
```
function canPartition(nums):
    totalSum = sum(nums)
    if totalSum % 2 != 0: return false
    target = totalSum / 2
    
    // Optimization: if any element > target, impossible
    if max(nums) > target: return false
    
    // Optimization: if any element == target, done
    if max(nums) == target: return true
    
    dp = boolean array, dp[0] = true
    for num in nums:
        for s = target down to num:
            dp[s] = dp[s] OR dp[s - num]
        if dp[target]: return true   // early exit!
    
    return dp[target]
```

### Bitset Optimization
```
Use a bitset (bit array) to represent dp:
  dp = bitset with bit 0 set

  for num in nums:
      dp = dp | (dp << num)
      // shift left by num = try adding num to every existing sum

  return bit 'target' of dp is set

Time: O(n × target / 64)  — 64x speedup with bit packing
```

---

## 🌐 Variants

### Partition into Two with Minimum Difference
```
Instead of equal partition, minimize |sum1 - sum2|.

  Run subset sum DP for all sums 0..totalSum/2.
  Find largest achievable sum ≤ totalSum/2.

function minDiffPartition(nums):
    total = sum(nums)
    target = total / 2
    dp = boolean array, dp[0] = true
    
    for num in nums:
        for s = target down to num:
            dp[s] = dp[s] OR dp[s - num]
    
    for s = target down to 0:
        if dp[s]:
            return total - 2 * s    // |s - (total-s)| = |total - 2s|

Example: nums = [1, 6, 11, 5]
  total = 23, target = 11
  Best subset sum ≤ 11: {1,5,5}=11? → {1,5}=6, {6,5}=11 ✓
  Min diff = 23 - 2×11 = 1
  Partition: {6,5} and {1,11} → sums 11 and 12
```

### Partition into K Equal Subsets
```
Much harder! Can't reduce to simple subset sum.
Requires backtracking or bitmask DP (covered in Unit 12).

dp[mask] = which subsets of elements can be assigned
           to have k groups each summing to total/k.
```

### Three-Way Partition
```
Can we split into 3 subsets with equal sum?

If totalSum % 3 ≠ 0: false
Otherwise: 2D subset sum problem
  dp[s1][s2] = can we form subsets with sums s1 and s2?
  Third subset automatically has sum total - s1 - s2.
  
  Time: O(n × (total/3)²)
```

---

## 🧩 Decision Tree Visualization

```
nums = [1, 5, 11, 5], target = 11

                  capacity=11
                 /          \
            skip 1         take 1
           cap=11          cap=10
          /      \        /      \
       skip 5  take 5  skip 5  take 5
       c=11    c=6     c=10    c=5
        |       |       |       |
       ...     ...     ...    ...
                                |
                            take 5
                            c=0 ✓ (subset {1,5,5})
                            
Also: take 11 from cap=11 → cap=0 ✓ (subset {11})
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Reduction** | Subset Sum with target = totalSum / 2 |
| **Quick check** | Odd total → impossible |
| **State** | dp[s] = can subset sum to s? |
| **Complexity** | O(n × sum/2) time, O(sum/2) space |
| **Early exit** | max(nums) > target → false |
| **Variant** | Min-diff partition: find largest achievable sum ≤ target |

---

## ❓ Quick Revision Questions

1. **Why can we immediately return false if the total sum is odd?**
2. **What is the target for the Subset Sum reduction?**
3. **How does the bitset trick improve performance?**
4. **How do you find the minimum difference partition instead of equal?**
5. **What element-level check can immediately determine the answer?**
6. **How does the problem change for partitioning into 3 or K subsets?**

---

[← Previous: Subset Sum](03-subset-sum.md) | [Next: Target Sum →](05-target-sum.md)

[← Back to README](../README.md)
