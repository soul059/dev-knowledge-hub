# Chapter 4: Maximum Sum Increasing Subsequence

## 📋 Overview
Find the increasing subsequence with the **maximum sum** (not maximum length). This modifies the LIS DP by tracking sums instead of lengths — `dp[i]` stores the maximum sum of an increasing subsequence ending at index `i`.

---

## 🧠 Core Concept

```
nums = [1, 101, 2, 3, 100, 4, 5]

LIS by length:  [1, 2, 3, 100] length 4
                [1, 2, 3, 4, 5] length 5 ← longest

But max SUM:    [1, 2, 3, 100] sum = 106 ← maximum sum
                [1, 2, 3, 4, 5] sum = 15

Answer: 106

Different objective → different optimal subsequence!
```

---

## 🔨 Algorithm

```
function maxSumIS(nums):
    n = len(nums)
    dp = copy of nums      // dp[i] initialized to nums[i] itself
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + nums[i])
    
    return max(dp)

Key difference from LIS:
  LIS:    dp[i] = max(dp[i], dp[j] + 1)        // count
  MaxSum: dp[i] = max(dp[i], dp[j] + nums[i])  // sum
  
  Base case:
  LIS:    dp[i] = 1       (element alone has length 1)
  MaxSum: dp[i] = nums[i] (element alone has sum = its value)
```

---

## 🔬 Trace: nums = [1, 101, 2, 3, 100, 4, 5]

```
dp = [1, 101, 2, 3, 100, 4, 5]    (initialized to nums)

i=1 (101):
  j=0: 1<101, dp[1]=max(101, 1+101)=102
  dp = [1, 102, 2, 3, 100, 4, 5]

i=2 (2):
  j=0: 1<2, dp[2]=max(2, 1+2)=3
  j=1: 101>2, skip
  dp = [1, 102, 3, 3, 100, 4, 5]

i=3 (3):
  j=0: 1<3, dp[3]=max(3, 1+3)=4
  j=1: 101>3, skip
  j=2: 2<3, dp[3]=max(4, 3+3)=6
  dp = [1, 102, 3, 6, 100, 4, 5]

i=4 (100):
  j=0: 1<100, dp[4]=max(100, 1+100)=101
  j=1: 101>100, skip
  j=2: 2<100, dp[4]=max(101, 3+100)=103
  j=3: 3<100, dp[4]=max(103, 6+100)=106
  dp = [1, 102, 3, 6, 106, 4, 5]

i=5 (4):
  j=0: 1<4, dp[5]=max(4, 1+4)=5
  j=2: 2<4, dp[5]=max(5, 3+4)=7
  j=3: 3<4, dp[5]=max(7, 6+4)=10
  dp = [1, 102, 3, 6, 106, 10, 5]

i=6 (5):
  j=0: 1<5, dp[6]=max(5, 1+5)=6
  j=2: 2<5, dp[6]=max(6, 3+5)=8
  j=3: 3<5, dp[6]=max(8, 6+5)=11
  j=5: 4<5, dp[6]=max(11, 10+5)=15
  dp = [1, 102, 3, 6, 106, 10, 15]

Answer: max(dp) = 106 (subsequence [1, 2, 3, 100])
```

---

## 🔄 Reconstruction

```
function maxSumIS_reconstruct(nums):
    n = len(nums)
    dp = copy of nums
    parent = array of size n, all -1
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i] and dp[j] + nums[i] > dp[i]:
                dp[i] = dp[j] + nums[i]
                parent[i] = j
    
    // Find max index
    maxIdx = argmax(dp)
    
    // Trace back
    result = []
    idx = maxIdx
    while idx != -1:
        result.prepend(nums[idx])
        idx = parent[idx]
    return result

For our example:
  maxIdx = 4 (dp[4] = 106)
  parent[4] = 3, parent[3] = 2, parent[2] = 0, parent[0] = -1
  Result: [1, 2, 3, 100]
```

---

## 🌐 Variants

### With Negative Numbers
```
nums = [4, -1, 2, 3, -2, 5]

dp = [4, -1, 2, 3, -2, 5]

Still works correctly:
  i=2 (2): j=1: -1<2, dp[2]=max(2, -1+2)=2 (skip: -1 hurts)
  Best just start fresh at 2.
  
Negative numbers are naturally excluded when they 
decrease the sum — max() handles this.
```

### Weighted LIS
```
Each element has a separate weight to maximize:
  nums = [1, 2, 3, 100]
  weights = [10, 5, 8, 2]
  
  dp[i] = max sum of weights along increasing subsequence
  dp[i] = max(weights[i], dp[j] + weights[i]) 
          where nums[j] < nums[i]
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i] = max sum of increasing subseq ending at i |
| **Base** | dp[i] = nums[i] (element alone) |
| **Recurrence** | dp[i] = max(dp[j] + nums[i]) for j < i, nums[j] < nums[i] |
| **Answer** | max(dp[0..n-1]) |
| **Complexity** | O(n²) time, O(n) space |
| **Key difference** | Sum instead of count in transition |

---

## ❓ Quick Revision Questions

1. **How does the initialization differ from standard LIS?**
2. **Why might the maximum sum subsequence be different from the longest?**
3. **How do negative numbers behave in this formulation?**
4. **What is the recurrence relation?**
5. **Can this be solved in O(n log n)?**
6. **How do you reconstruct the actual subsequence?**

---

[← Previous: Russian Doll Envelopes](03-russian-doll-envelopes.md) | [Next: Bitonic Subsequence →](05-bitonic-subsequence.md)

[← Back to README](../README.md)
