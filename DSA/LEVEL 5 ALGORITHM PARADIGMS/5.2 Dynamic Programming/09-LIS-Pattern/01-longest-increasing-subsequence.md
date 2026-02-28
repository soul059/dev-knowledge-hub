# Chapter 1: Longest Increasing Subsequence (LIS)

## 📋 Overview
The **Longest Increasing Subsequence** finds the longest subsequence where each element is strictly greater than the previous. LIS is a foundational DP pattern with an elegant O(n log n) solution using patience sorting / binary search.

---

## 🧠 Core Concept

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

Subsequences (not contiguous):
  [2, 5, 7, 101]  length 4  ✓  (LIS)
  [2, 5, 7, 18]   length 4  ✓  (another LIS)
  [2, 3, 7, 101]  length 4  ✓  (another LIS)
  [10, 101]        length 2
  [9, 101]         length 2

  Answer: 4

Important: SUBSEQUENCE (not substring)
  ┌───────────────────────────────────┐
  │ Substring:    contiguous          │
  │ Subsequence:  NOT necessarily     │
  │               contiguous          │
  └───────────────────────────────────┘
```

---

## 🔨 Approach 1: O(n²) DP

```
State: dp[i] = length of LIS ending at index i

Recurrence:
  dp[i] = 1 + max(dp[j]) for all j < i where nums[j] < nums[i]
  dp[i] = 1 if no such j exists (element alone)

Answer: max(dp[0], dp[1], ..., dp[n-1])

function LIS(nums):
    n = len(nums)
    dp = array of size n, all 1
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)
```

### Trace

```
nums = [10,  9,  2,  5,  3,  7, 101, 18]
index    0   1   2   3   4   5    6    7

dp[0] = 1  (10 alone)
dp[1] = 1  (9: no j where nums[j]<9, actually 2 comes later)
            Wait: j=0, nums[0]=10 > 9, skip. dp[1]=1
dp[2] = 1  (2: 10>2, 9>2. dp[2]=1)
dp[3] = 2  (5: j=2, nums[2]=2<5, dp[3]=dp[2]+1=2)
dp[4] = 2  (3: j=2, nums[2]=2<3, dp[4]=dp[2]+1=2)
dp[5] = 3  (7: j=2→dp[2]+1=2, j=3→dp[3]+1=3, j=4→dp[4]+1=3)
dp[6] = 4  (101: j=5→dp[5]+1=4)
dp[7] = 4  (18: j=5→dp[5]+1=4)

dp = [1, 1, 1, 2, 2, 3, 4, 4]

Answer: max(dp) = 4
```

---

## 🔨 Approach 2: O(n log n) with Binary Search

```
Maintain an array 'tails' where:
  tails[i] = smallest tail element of all increasing 
             subsequences of length i+1

Key property: tails is always sorted!

Algorithm:
  for each num in nums:
      if num > tails.last():
          tails.append(num)        // extends longest subsequence
      else:
          find smallest tails[i] >= num (binary search)
          tails[i] = num           // replace to keep potential open

  LIS length = len(tails)
```

### Why This Works

```
tails = [2, 3, 7, 18]   represents:
  tails[0]=2:  LIS of length 1 ending with smallest value 2
  tails[1]=3:  LIS of length 2 ending with smallest value 3
  tails[2]=7:  LIS of length 3 ending with smallest value 7
  tails[3]=18: LIS of length 4 ending with smallest value 18

By keeping the smallest possible tail for each length,
we maximize the chance that future elements can extend subsequences.
```

### Trace

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]
tails = []

num=10:  tails=[] → append.        tails=[10]
num=9:   9 < 10 → replace tails[0] tails=[9]
num=2:   2 < 9  → replace tails[0] tails=[2]
num=5:   5 > 2  → append.          tails=[2, 5]
num=3:   3 < 5  → replace tails[1] tails=[2, 3]
num=7:   7 > 3  → append.          tails=[2, 3, 7]
num=101: 101>7  → append.          tails=[2, 3, 7, 101]
num=18:  18<101 → replace tails[3] tails=[2, 3, 7, 18]

LIS length = len(tails) = 4 ✓

NOTE: tails = [2,3,7,18] is NOT the actual LIS!
      It represents the optimal tail values.
      The actual LIS could be [2,5,7,101] or [2,3,7,18].
```

---

## 💻 Pseudocode: O(n log n)

```
function LIS(nums):
    tails = []
    
    for num in nums:
        // Binary search: find leftmost position where tails[pos] >= num
        lo = 0, hi = len(tails)
        while lo < hi:
            mid = (lo + hi) / 2
            if tails[mid] < num:
                lo = mid + 1
            else:
                hi = mid
        
        if lo == len(tails):
            tails.append(num)
        else:
            tails[lo] = num
    
    return len(tails)
```

---

## 🔄 Reconstructing the Actual LIS

```
Track parent pointers during the O(n²) approach:

function LIS_with_reconstruction(nums):
    n = len(nums)
    dp = [1] * n
    parent = [-1] * n
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i] and dp[j] + 1 > dp[i]:
                dp[i] = dp[j] + 1
                parent[i] = j
    
    // Find index of maximum dp value
    maxIdx = argmax(dp)
    
    // Trace back
    result = []
    idx = maxIdx
    while idx != -1:
        result.prepend(nums[idx])
        idx = parent[idx]
    
    return result
```

---

## 🌐 Variants

```
┌────────────────────────────────┬─────────────────────────────────┐
│ Variant                       │ Change                          │
├────────────────────────────────┼─────────────────────────────────┤
│ Non-decreasing (≤)            │ Use upper_bound in binary search│
│ Strictly decreasing           │ Reverse array, find LIS         │
│ Longest non-increasing        │ Reverse + non-decreasing LIS    │
│ Number of LIS                 │ Track count[] alongside dp[]    │
│ LIS in 2D (envelope/dolls)    │ Sort by one dim, LIS on other   │
│ Maximum sum increasing subseq │ dp[i] = nums[i]+max(dp[j])     │
└────────────────────────────────┴─────────────────────────────────┘
```

---

## 📊 Complexity Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force (all subsequences) | O(2^n) | O(n) | Exponential |
| O(n²) DP | O(n²) | O(n) | Simple, allows reconstruction |
| Binary search | O(n log n) | O(n) | Optimal |
| Segment tree | O(n log n) | O(n) | For advanced variants |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State (O(n²))** | dp[i] = length of LIS ending at index i |
| **Recurrence** | dp[i] = 1 + max(dp[j]) where j < i, nums[j] < nums[i] |
| **O(n log n) idea** | Maintain tails array; binary search to insert |
| **tails property** | tails[k] = smallest tail of all LIS of length k+1 |
| **Answer** | max(dp) or len(tails) |
| **Reconstruction** | Parent pointers (O(n²)) or index tracking |

---

## ❓ Quick Revision Questions

1. **What does dp[i] represent in the O(n²) solution?**
2. **What does tails[k] represent in the O(n log n) solution?**
3. **Why is the tails array always sorted?**
4. **Is the tails array the actual LIS? Why or why not?**
5. **How would you modify LIS for non-decreasing (≤) instead of strict (<)?**
6. **What is the time complexity of the binary search approach and why?**

---

[← Previous Unit: Coin Change Knapsack](../08-Knapsack-Patterns/06-coin-change-knapsack.md) | [Next: Number of LIS →](02-number-of-lis.md)

[← Back to README](../README.md)
