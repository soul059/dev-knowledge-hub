# Chapter 3: Maximum Subarray (Kadane's Algorithm)

## 📋 Overview
**Problem:** Given an integer array `nums`, find the contiguous subarray with the largest sum and return its sum.

Kadane's algorithm is a classic DP approach that solves this in O(n) time — one of the most elegant DP solutions.

---

## 🧠 Problem Visualization

```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

All contiguous subarrays and their sums:
  [-2] = -2
  [-2, 1] = -1
  [1] = 1
  [1, -3] = -2
  [4, -1, 2, 1] = 6  ◄── MAXIMUM!
  ...

  ┌────┬───┬────┬───┬────┬───┬───┬────┬───┐
  │ -2 │ 1 │ -3 │ 4 │ -1 │ 2 │ 1 │ -5 │ 4 │
  └────┴───┴────┴───┴────┴───┴───┴────┴───┘
                 └──────────────┘
                 Maximum subarray = 6
```

---

## 🔍 Kadane's Algorithm: DP Approach

### Key Insight
```
At each position i, decide:
  1. EXTEND the previous subarray (include nums[i])
  2. START a new subarray at nums[i]

If the previous subarray sum is negative, 
starting fresh is always better!

dp[i] = max sum of subarray ENDING at index i
dp[i] = max(dp[i-1] + nums[i],  nums[i])
              extend           start new
```

### Pseudocode
```
function maxSubArray(nums):
    max_ending_here = nums[0]   // dp[i]: best sum ending at i
    max_so_far = nums[0]        // global maximum

    for i = 1 to n-1:
        max_ending_here = max(nums[i], max_ending_here + nums[i])
        max_so_far = max(max_so_far, max_ending_here)
    
    return max_so_far

Time: O(n) | Space: O(1)
```

---

## 🧪 Step-by-Step Trace

```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

i │ nums[i] │ max_ending_here              │ max_so_far │ Action
──┼─────────┼──────────────────────────────┼────────────┼──────────
0 │   -2    │ -2                           │    -2      │ Start
1 │    1    │ max(1, -2+1) = max(1,-1) = 1 │     1      │ Start new
2 │   -3    │ max(-3, 1-3) = max(-3,-2) =-2│     1      │ Extend
3 │    4    │ max(4, -2+4) = max(4, 2) = 4 │     4      │ Start new
4 │   -1    │ max(-1, 4-1) = max(-1,3) = 3 │     4      │ Extend
5 │    2    │ max(2, 3+2) = max(2, 5) = 5  │     5      │ Extend
6 │    1    │ max(1, 5+1) = max(1, 6) = 6  │     6      │ Extend
7 │   -5    │ max(-5, 6-5) = max(-5,1) = 1 │     6      │ Extend
8 │    4    │ max(4, 1+4) = max(4, 5) = 5  │     6      │ Extend

Answer: 6 (subarray [4, -1, 2, 1])

Visual of max_ending_here:
  [-2]  [1]  [1,-3]  [4]  [4,-1]  [4,-1,2]  [4,-1,2,1]  [4,-1,2,1,-5]  [4,-1,2,1,-5,4]
   -2    1     -2      4     3        5          6            1              5
                    start new ↑                          still extending ──────►
```

---

## 📐 Why This Works

```
┌─────────────────────────────────────────────────────────┐
│  KEY INSIGHT: A negative prefix can NEVER help          │
│                                                         │
│  If max_ending_here < 0:                                │
│    Any future subarray including this prefix            │
│    would be SMALLER than starting fresh.                │
│                                                         │
│  Example:                                               │
│    prefix_sum = -5, next element = 3                    │
│    Extend: -5 + 3 = -2                                  │
│    New:    3                                             │
│    Starting new is better! (3 > -2)                     │
│                                                         │
│  dp[i] = max(nums[i], dp[i-1] + nums[i])               │
│                ↑              ↑                          │
│          start fresh    extend previous                 │
│                                                         │
│  This is equivalent to:                                 │
│    dp[i] = nums[i] + max(0, dp[i-1])                   │
│    "Add previous sum only if it's positive"             │
└─────────────────────────────────────────────────────────┘
```

---

## 💡 Variants

### Finding the Subarray (not just the sum)
```
function maxSubArrayWithIndices(nums):
    max_ending_here = nums[0]
    max_so_far = nums[0]
    start = 0, end = 0, temp_start = 0
    
    for i = 1 to n-1:
        if nums[i] > max_ending_here + nums[i]:
            max_ending_here = nums[i]
            temp_start = i          // new subarray starts here
        else:
            max_ending_here += nums[i]
        
        if max_ending_here > max_so_far:
            max_so_far = max_ending_here
            start = temp_start
            end = i
    
    return max_so_far, start, end
```

### Maximum Circular Subarray
```
Answer = max(
    standard_kadane(nums),         // max subarray (non-wrapping)
    total_sum - min_subarray(nums)  // wrapping case
)

Why? If max subarray wraps around, the NON-selected elements
form the MINIMUM subarray in the middle.
total_sum - min_subarray = max wrapping subarray
```

---

## 📊 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force (all subarrays) | O(n³) | O(1) | Check all pairs (i,j) |
| Prefix Sum optimization | O(n²) | O(n) | Precompute prefix sums |
| Divide and Conquer | O(n log n) | O(log n) | Split array in half |
| **Kadane's Algorithm** | **O(n)** | **O(1)** | **Optimal** |

---

## ❓ Quick Revision Questions

1. **What is the key decision at each index in Kadane's algorithm?**
2. **Why is starting a new subarray sometimes better than extending?**
3. **What does dp[i] represent in Kadane's algorithm?**
4. **How would you modify Kadane's to find the actual subarray boundaries?**
5. **What is the maximum subarray sum of [-3, -2, -1, -4]?**
6. **How does the circular variant work using Kadane's?**

---

[← Previous: House Robber](02-house-robber.md) | [Next: Coin Change (Min) →](04-coin-change-min.md)

[← Back to README](../README.md)
