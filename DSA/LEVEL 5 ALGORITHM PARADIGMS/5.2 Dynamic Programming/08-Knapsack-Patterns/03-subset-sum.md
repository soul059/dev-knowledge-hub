# Chapter 3: Subset Sum

## 📋 Overview
The **Subset Sum** problem asks: given a set of positive integers and a target sum, does there exist a subset whose elements sum exactly to the target? This is the **boolean version** of 0/1 Knapsack — instead of maximizing value, we check feasibility.

---

## 🧠 Core Concept

```
nums = [3, 34, 4, 12, 5, 2],  target = 9

Subsets that sum to 9:
  {4, 5}    → 4 + 5 = 9  ✓
  {3, 4, 2} → 3 + 4 + 2 = 9  ✓

Answer: TRUE

Mapping to 0/1 Knapsack:
  ┌────────────────────────────────────┐
  │ weight[i] = nums[i]               │
  │ value[i]  = nums[i]  (or just 1)  │
  │ capacity  = target                 │
  │ Question: can we fill exactly?     │
  └────────────────────────────────────┘
```

---

## 🔨 Recurrence

```
State: dp[i][s] = can we form sum s using elements 0..i-1?
       (boolean: true/false)

dp[i][s] = dp[i-1][s]              // SKIP element i-1
            OR 
            dp[i-1][s - nums[i-1]]  // TAKE element i-1 (if s ≥ nums[i-1])

Base:  dp[i][0] = true  for all i  (empty subset sums to 0)
       dp[0][s] = false for s > 0  (no elements, can't form positive sum)
```

---

## 🔬 Trace: nums=[3,4,5,2], target=9

```
       s=0  s=1  s=2  s=3  s=4  s=5  s=6  s=7  s=8  s=9
  {}  [ T    F    F    F    F    F    F    F    F    F  ]
  {3} [ T    F    F    T    F    F    F    F    F    F  ]
  {4} [ T    F    F    T    T    F    F    T    F    F  ]
  {5} [ T    F    F    T    T    T    F    T    T    T  ]
  {2} [ T    F    T    T    T    T    T    T    T    T  ]

Building row {5}, s=9:
  dp[3][9] = dp[2][9] OR dp[2][9-5]
           = dp[2][9] OR dp[2][4]
           = false    OR true
           = true ✓

Answer: dp[4][9] = true (subset {4,5} or {3,4,2})
```

---

## 💻 Pseudocode

### 2D Boolean Table
```
function subsetSum(nums, target):
    n = len(nums)
    dp = boolean array [n+1][target+1]
    
    // base case: sum 0 is always achievable
    for i = 0 to n:
        dp[i][0] = true
    
    for i = 1 to n:
        for s = 1 to target:
            dp[i][s] = dp[i-1][s]      // skip
            if s >= nums[i-1]:
                dp[i][s] = dp[i][s] OR dp[i-1][s - nums[i-1]]  // take
    
    return dp[n][target]
```

### Space-Optimized 1D
```
function subsetSum(nums, target):
    dp = boolean array of size target+1, all false
    dp[0] = true
    
    for num in nums:
        for s = target down to num:    // RIGHT TO LEFT (0/1 knapsack!)
            dp[s] = dp[s] OR dp[s - num]
    
    return dp[target]

Time: O(n × target), Space: O(target)
```

---

## 🔑 Counting Subsets

```
Instead of boolean, COUNT how many subsets sum to target:

dp[i][s] = dp[i-1][s] + dp[i-1][s - nums[i-1]]

1D version:
  dp[0] = 1
  for num in nums:
      for s = target down to num:
          dp[s] += dp[s - num]

This counts ALL subsets, including those with different orderings
of the same elements.
```

---

## 🌐 Variants & Related Problems

### Subset Sum with Negative Numbers
```
If nums can be negative:
  - Offset all values: shift range so minimum is 0
  - Or use a hash map instead of array

map approach:
  dp = {0: true}
  for num in nums:
      new_dp = copy(dp)
      for s in dp.keys():
          new_dp[s + num] = true
      dp = new_dp
  return target in dp
```

### Finding the Subset (Reconstruction)
```
function findSubset(nums, target):
    // First, build the DP table (2D)
    // Then backtrack:
    result = []
    s = target
    for i = n down to 1:
        if dp[i][s] and not dp[i-1][s]:
            result.append(nums[i-1])
            s -= nums[i-1]
        // if dp[i-1][s] is true, item i-1 was optional
    return result
```

### Closest Subset Sum
```
Find subset with sum closest to target:

  Run subset sum DP for all sums 0..target.
  Find largest s where dp[s] = true.

  dp = boolean array, all false
  dp[0] = true
  for num in nums:
      for s = target down to num:
          dp[s] = dp[s] OR dp[s - num]
  
  for s = target down to 0:
      if dp[s]: return s
```

---

## 🧩 NP-Completeness Note

```
Subset Sum is NP-Complete!

But our DP solution is O(n × target) — polynomial in n and target.
Why isn't this P = NP?

The target value can be EXPONENTIALLY large in the input size.
A number with d digits has value up to 10^d.

So O(n × target) is pseudo-polynomial — polynomial in the 
VALUE of target, not in the number of BITS needed to represent it.

Input size = O(n × log(target))
Target value = O(2^(log(target)))

The DP solution IS efficient when target is reasonably bounded.
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[s] = can sum s be formed from a subset? |
| **Recurrence** | dp[s] = dp[s] OR dp[s - num] |
| **Base** | dp[0] = true |
| **Direction** | Right to left (0/1 knapsack pattern) |
| **Complexity** | O(n × target) time, O(target) space |
| **NP-hardness** | Pseudo-polynomial solution |

---

## ❓ Quick Revision Questions

1. **How does Subset Sum relate to 0/1 Knapsack?**
2. **Why is dp[0] = true?**
3. **How do you modify the recurrence to COUNT subsets instead of checking existence?**
4. **Why is the DP solution pseudo-polynomial rather than truly polynomial?**
5. **How would you handle negative numbers in the input?**
6. **How do you reconstruct which elements form the subset?**

---

[← Previous: Unbounded Knapsack](02-unbounded-knapsack.md) | [Next: Partition Equal Subset Sum →](04-partition-equal-subset.md)

[← Back to README](../README.md)
