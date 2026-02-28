# Chapter 2: 1D DP Patterns

## 📋 Chapter Overview
1D DP problems use a single array `dp[i]` where `i` represents a position, amount, or state. Most foundational DP problems fall here.

---

## 🔍 Pattern 1: Linear Sequence

**dp[i] depends on dp[i-1], dp[i-2], etc.**

### House Robber

```
  Can't rob adjacent houses. Maximize total.
  nums = [2, 7, 9, 3, 1]
  
  State: dp[i] = max money robbing houses 0..i
  Transition: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
              (skip house i) or (rob house i + best before i-1)
  Base: dp[0] = nums[0], dp[1] = max(nums[0], nums[1])
```

```
function rob(nums):
    if len(nums) == 1: return nums[0]
    prev2 = nums[0]
    prev1 = max(nums[0], nums[1])
    for i = 2 to len(nums)-1:
        curr = max(prev1, prev2 + nums[i])
        prev2 = prev1
        prev1 = curr
    return prev1
```

### Trace: [2, 7, 9, 3, 1]

| i | nums[i] | prev2 | prev1 | curr = max(prev1, prev2+nums[i]) |
|---|---------|-------|-------|----------------------------------|
| 0 | 2 | - | - | prev2=2 |
| 1 | 7 | 2 | 7 | prev1=7 |
| 2 | 9 | 2 | 7 | max(7, 2+9) = **11** |
| 3 | 3 | 7 | 11 | max(11, 7+3) = 11 |
| 4 | 1 | 11 | 11 | max(11, 11+1) = **12** |

Answer: 12 (rob houses 0, 2, 4 → 2+9+1=12) ✓

---

## 🔍 Pattern 2: Decision at Each Step

### Decode Ways

```
  "226" → how many ways to decode?
  2|2|6 → "BBF"
  22|6  → "VF"
  2|26  → "BZ"
  Answer: 3
```

```
function numDecodings(s):
    n = len(s)
    dp = array of size n+1
    dp[0] = 1                          // empty string
    dp[1] = 1 if s[0] != '0' else 0
    
    for i = 2 to n:
        oneDigit = int(s[i-1])
        twoDigit = int(s[i-2..i-1])
        
        if oneDigit >= 1:
            dp[i] += dp[i-1]          // single digit valid
        if twoDigit >= 10 AND twoDigit <= 26:
            dp[i] += dp[i-2]          // two digits valid
    
    return dp[n]
```

---

## 🔍 Pattern 3: Maximum Subarray (Kadane's)

```
  dp[i] = max subarray sum ending at index i
  dp[i] = max(nums[i], dp[i-1] + nums[i])
  
  "Start fresh or extend the previous subarray?"
```

```
function maxSubArray(nums):
    maxSum = nums[0]
    currentSum = nums[0]
    
    for i = 1 to len(nums)-1:
        currentSum = max(nums[i], currentSum + nums[i])
        maxSum = max(maxSum, currentSum)
    
    return maxSum
```

### Trace: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

| i | nums[i] | currentSum | maxSum |
|---|---------|------------|--------|
| 0 | -2 | -2 | -2 |
| 1 | 1 | max(1, -2+1)=1 | 1 |
| 2 | -3 | max(-3, 1-3)=-2 | 1 |
| 3 | 4 | max(4, -2+4)=4 | 4 |
| 4 | -1 | max(-1, 4-1)=3 | 4 |
| 5 | 2 | max(2, 3+2)=5 | 5 |
| 6 | 1 | max(1, 5+1)=6 | **6** |
| 7 | -5 | max(-5, 6-5)=1 | 6 |
| 8 | 4 | max(4, 1+4)=5 | 6 |

Answer: 6 (subarray [4, -1, 2, 1]) ✓

---

## 🔍 Pattern 4: Jump Game Variants

### Jump Game I (Can reach end?)

```
function canJump(nums):
    maxReach = 0
    for i = 0 to len(nums)-1:
        if i > maxReach: return false
        maxReach = max(maxReach, i + nums[i])
    return true
```

### Jump Game II (Minimum jumps)

```
function jump(nums):
    jumps = 0
    currentEnd = 0
    farthest = 0
    
    for i = 0 to len(nums)-2:
        farthest = max(farthest, i + nums[i])
        if i == currentEnd:
            jumps += 1
            currentEnd = farthest
    
    return jumps
```

---

## 🔍 Pattern 5: Longest Increasing Subsequence (LIS)

```
  nums = [10, 9, 2, 5, 3, 7, 101, 18]
  LIS:   [2, 3, 7, 18] → length 4
```

### O(n²) DP

```
function lengthOfLIS(nums):
    n = len(nums)
    dp = array of 1s, size n       // each element is LIS of length 1
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)
```

### Trace

| i | nums[i] | Check j | dp[i] |
|---|---------|---------|-------|
| 0 | 10 | - | 1 |
| 1 | 9 | 10>9 skip | 1 |
| 2 | 2 | 10>2, 9>2 skip | 1 |
| 3 | 5 | 2<5 → dp[3]=2 | 2 |
| 4 | 3 | 2<3 → dp[4]=2 | 2 |
| 5 | 7 | 2<7→2, 5<7→3, 3<7→3 | 3 |
| 6 | 101 | all < 101, best dp[5]+1 | 4 |
| 7 | 18 | 7<18→dp[5]+1=4 | 4 |

### O(n log n) with Patience Sort

```
function lengthOfLIS(nums):
    tails = []                     // tails[i] = smallest tail of LIS of length i+1
    
    for num in nums:
        pos = lowerBound(tails, num)
        if pos == len(tails):
            tails.append(num)      // extend
        else:
            tails[pos] = num       // replace
    
    return len(tails)
```

---

## 📊 1D DP Pattern Summary

| Pattern | State | Transition | Examples |
|---------|-------|------------|----------|
| Linear | dp[i] | dp[i-1], dp[i-2] | Climbing stairs, House robber |
| Decision | dp[i] | Choice at position i | Decode ways |
| Kadane's | dp[i] | max(start fresh, extend) | Max subarray |
| LIS | dp[i] | max(dp[j]+1) for j<i | LIS, Russian dolls |
| Jump | Greedy-like | Track farthest reach | Jump game |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| 1D DP | Single variable (position or amount) |
| Space optimization | Usually can reduce to O(1) with rolling vars |
| Kadane's | "Start fresh or extend?" — classic pattern |
| LIS O(n²) | For every i, check all j < i |
| LIS O(n log n) | Binary search on tails array |

---

## ❓ Revision Questions

1. **House Robber transition?** → `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — skip or rob.
2. **Kadane's key decision?** → At each element: start new subarray or extend current.
3. **LIS: O(n²) vs O(n log n)?** → O(n²) checks all j<i; O(n log n) uses binary search on tails.
4. **Decode Ways: when is two-digit valid?** → 10 ≤ two-digit ≤ 26.
5. **Jump Game II: why is it greedy?** → At each "level" (jump), choose the farthest reachable position.

---

[← Previous: DP Fundamentals](01-dp-fundamentals.md) | [Next: 2D DP Patterns →](03-2d-dp-patterns.md)
