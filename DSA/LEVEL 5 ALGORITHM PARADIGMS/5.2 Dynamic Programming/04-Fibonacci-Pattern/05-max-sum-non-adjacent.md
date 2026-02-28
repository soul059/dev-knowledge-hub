# Chapter 5: Maximum Sum of Non-Adjacent Elements

## 📋 Overview
Given an array of positive integers, find the **maximum sum** such that no two selected elements are adjacent. This is equivalent to **House Robber** but framed as a pure algorithmic problem — making the Fibonacci pattern crystal clear.

---

## 🧠 Core Concept

```
Input:  [3, 2, 5, 10, 7]
Goal:   Select elements such that no two are neighbors,
        maximize their sum.

At each position i, we choose:
  INCLUDE i: take nums[i], must skip i-1 → dp[i-2] + nums[i]
  EXCLUDE i: skip nums[i], take best so far → dp[i-1]

  dp[i] = max(dp[i-1], dp[i-2] + nums[i])

Decision tree visualization:
                     i=0 (value 3)
                   /            \
            TAKE 3              SKIP 3
            (sum=3)             (sum=0)
           /      \            /      \
      SKIP 2    can't      TAKE 2   SKIP 2
      (sum=3)   take 2     (sum=2)  (sum=0)
      /    \               /    \
   TAKE 5  SKIP 5      SKIP 5  TAKE 5
   ...      ...          ...    (sum=7)
```

---

## 🔨 Step-by-Step Solution

### Step 1: Define State
```
dp[i] = maximum sum considering elements from index 0 to i
```

### Step 2: Recurrence
```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
  ↑             ↑              ↑
  best up to i  skip i         include i (+ best skipping i-1)
```

### Step 3: Base Cases
```
dp[0] = nums[0]                    // only one element
dp[1] = max(nums[0], nums[1])      // best of first two
```

### Step 4: Space-Optimized Implementation
```
function maxSumNonAdjacent(nums):
    if len(nums) == 0: return 0
    if len(nums) == 1: return nums[0]

    prev2 = 0      // dp[i-2]
    prev1 = 0      // dp[i-1]

    for num in nums:
        current = max(prev1, prev2 + num)
        prev2 = prev1
        prev1 = current

    return prev1
```

---

## 🔬 Trace: nums = [3, 2, 5, 10, 7]

```
Initial: prev2=0, prev1=0

i=0, num=3:  current = max(0, 0+3)  = 3   → prev2=0, prev1=3
i=1, num=2:  current = max(3, 0+2)  = 3   → prev2=3, prev1=3
i=2, num=5:  current = max(3, 3+5)  = 8   → prev2=3, prev1=8
i=3, num=10: current = max(8, 3+10) = 13  → prev2=8, prev1=13
i=4, num=7:  current = max(13, 8+7) = 15  → prev2=13, prev1=15

Answer: 15 (select indices 0, 2, 4 → 3+5+7=15)

Wait — let's verify: 3+5+7=15 vs 2+10=12 vs 3+10=13
Best is 3+5+7 = 15 ✓
```

---

## 🔬 Trace: nums = [5, 1, 1, 5]

```
Initial: prev2=0, prev1=0

i=0, num=5:  current = max(0, 0+5) = 5  → prev2=0, prev1=5
i=1, num=1:  current = max(5, 0+1) = 5  → prev2=5, prev1=5
i=2, num=1:  current = max(5, 5+1) = 6  → prev2=5, prev1=6
i=3, num=5:  current = max(6, 5+5) = 10 → prev2=6, prev1=10

Answer: 10 (select indices 0, 3 → 5+5=10)

Other options: 5+1=6, 1+5=6, 5+1=6
Best is 5+5=10 ✓
```

---

## 🌐 Important Variants

### Variant 1: Circular Array
```
First and last elements are also adjacent.

Solution: Two passes (same as House Robber II)
  Pass 1: maxSum(nums[0..n-2])
  Pass 2: maxSum(nums[1..n-1])
  Answer: max(Pass 1, Pass 2)
```

### Variant 2: Count Ways to Achieve Max Sum
```
Not just the max sum, but how many subsets achieve it.

Track both value and count:
  (val1, cnt1) = (dp[i-1], ways[i-1])   // skip
  (val2, cnt2) = (dp[i-2]+nums[i], ways[i-2])  // include

  if val1 > val2:  dp[i]=val1, ways[i]=cnt1
  if val2 > val1:  dp[i]=val2, ways[i]=cnt2
  if val1 == val2: dp[i]=val1, ways[i]=cnt1+cnt2
```

### Variant 3: Return the Selected Indices
```
Track decisions alongside DP:

function maxSumWithIndices(nums):
    n = len(nums)
    dp = array of size n
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])

    for i = 2 to n-1:
        dp[i] = max(dp[i-1], dp[i-2] + nums[i])

    // Backtrack to find selected indices
    result = []
    i = n - 1
    while i >= 0:
        if i == 0 or dp[i] != dp[i-1]:
            result.append(i)
            i -= 2    // skip adjacent
        else:
            i -= 1    // was skipped
    return reverse(result)
```

### Variant 4: Allow Negative Numbers
```
When array contains negatives, we may want to select nothing.

Adjustment: prev2=0, prev1=0 (selecting nothing = sum 0)

dp[i] = max(dp[i-1], dp[i-2] + nums[i], 0)
         ↑ skip      ↑ include            ↑ select nothing

Or simply: the regular algorithm already handles this
since we start with prev2=prev1=0 and take max.
```

---

## 💡 Why Fibonacci?

```
The Fibonacci structure appears whenever:

1. Linear sequence of choices           ✓
2. Each decision is binary (take/skip)  ✓
3. Taking creates a gap (skip neighbor) ✓
4. Optimal substructure holds           ✓

This creates the recurrence:
  dp[i] = combine(dp[i-1], dp[i-2] + local_value)

Where combine = max for optimization problems
      combine = +   for counting problems
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Max sum with no two adjacent elements selected |
| **Recurrence** | dp[i] = max(dp[i-1], dp[i-2] + nums[i]) |
| **Base Cases** | prev2 = 0, prev1 = 0 |
| **Complexity** | O(n) time, O(1) space |
| **Fibonacci Link** | Same 2-state dependency structure |
| **Variants** | Circular, count ways, reconstruct indices |

---

## ❓ Quick Revision Questions

1. **What is the recurrence for maximum sum of non-adjacent elements?**
2. **How do you handle the circular variant?**
3. **How do you reconstruct which elements were selected?**
4. **Does the algorithm work with negative numbers?**
5. **How would you count the number of subsets that achieve the maximum sum?**
6. **What makes this problem follow the Fibonacci pattern?**

---

[← Previous: Climbing Stairs Variations](04-climbing-stairs-variations.md) | [Next: Delete and Earn →](06-delete-and-earn.md)

[← Back to README](../README.md)
