# Chapter 2: House Robber

## 📋 Overview
**Problem:** You are a robber planning to rob houses along a street. Each house has a certain amount of money. You **cannot rob two adjacent houses** (security systems are linked). Find the maximum amount of money you can rob.

---

## 🧠 Problem Visualization

```
Houses:  [2, 7, 9, 3, 1]
          │  │  │  │  │
          ▼  ▼  ▼  ▼  ▼
         $2 $7 $9 $3 $1

Rule: Cannot rob adjacent houses
         ┌──✗──┐
         │     │
        [2]   [7]   Can't rob both house 0 and house 1

Best strategy: Rob houses 0, 2, 4 → $2 + $9 + $1 = $12
    OR:        Rob houses 1, 3    → $7 + $3     = $10
    OR:        Rob house 2 alone  → $9          = $9
    Best: $12 (houses at index 0, 2, 4)
    
Actually let's check: houses 1, 2 → adjacent, invalid
houses 0, 2 → $2+$9 = $11
houses 0, 2, 4 → $2+$9+$1 = $12 ✓
houses 1, 3 → $7+$3 = $10
Answer: $12
```

---

## 🔍 DP Development

### Step 1: Define State
```
dp[i] = maximum money we can rob from houses 0..i
```

### Step 2: Recurrence (Think about the LAST decision)
```
At house i, we have TWO choices:
  1. ROB house i:   gain nums[i], but can't use house i-1
                     → nums[i] + dp[i-2]
  2. SKIP house i:  best we could do with houses 0..i-1
                     → dp[i-1]

dp[i] = max(dp[i-1], dp[i-2] + nums[i])
              skip        rob
```

### Step 3: Base Cases
```
dp[0] = nums[0]                    // only one house → rob it
dp[1] = max(nums[0], nums[1])      // two houses → rob the richer one
```

### Step 4: Tabulation
```
function rob(nums):
    n = len(nums)
    if n == 1: return nums[0]
    dp = array[n]
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    for i = 2 to n-1:
        dp[i] = max(dp[i-1], dp[i-2] + nums[i])
    return dp[n-1]
```

### Step 5: Space Optimized
```
function rob(nums):
    prev2 = 0, prev1 = 0
    for num in nums:
        curr = max(prev1, prev2 + num)
        prev2 = prev1
        prev1 = curr
    return prev1
```

---

## 🧪 Step-by-Step Trace

```
nums = [2, 7, 9, 3, 1]

i │ nums[i] │ prev2 │ prev1 │ max(prev1, prev2+nums[i]) │ Decision
──┼─────────┼───────┼───────┼──────────────────────────┼──────────
0 │    2    │   0   │   0   │ max(0, 0+2) = 2          │ Rob house 0
1 │    7    │   0   │   2   │ max(2, 0+7) = 7          │ Rob house 1
2 │    9    │   2   │   7   │ max(7, 2+9) = 11         │ Rob house 2
3 │    3    │   7   │  11   │ max(11, 7+3) = 11        │ Skip house 3
4 │    1    │  11   │  11   │ max(11, 11+1) = 12       │ Rob house 4

Answer: 12 (rob houses 0, 2, 4: $2+$9+$1)

Decision visualization:
  [2]  7  [9]  3  [1]     ← rob these = 2+9+1 = 12
   ✓   ✗   ✓   ✗   ✓
```

---

## 💡 Variant: House Robber II (Circular)

```
Houses are arranged in a CIRCLE (first and last are adjacent).

Strategy: Can't rob both house 0 and house n-1
  
  Solution: max(
      rob(houses[0..n-2]),    // exclude last house
      rob(houses[1..n-1])     // exclude first house
  )

  ┌──────────────────┐
  │  H0 ── H1 ── H2 │
  │  │              │ │
  │  H4 ── H3 ─────┘ │
  │  ↑              ↑ │
  │  └── adjacent ──┘ │  ← H0 and H4 can't both be robbed
  └──────────────────┘
```

---

## 📊 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | O(2ⁿ) | O(n) | Try all subsets |
| Memoization | O(n) | O(n) | Cache recursive calls |
| Tabulation | O(n) | O(n) | Fill array left-to-right |
| Space Optimized | O(n) | O(1) | Two variables |

---

## ❓ Quick Revision Questions

1. **Why can't we just pick all odd-indexed or all even-indexed houses?**
2. **Write the recurrence relation and explain each term.**
3. **How does the circular variant (House Robber II) break down into subproblems?**
4. **What happens if all house values are equal?**
5. **How would you reconstruct WHICH houses to rob (not just the max amount)?**
6. **What's the relationship between this problem and "Maximum sum of non-adjacent elements"?**

---

[← Previous: Climbing Stairs](01-climbing-stairs.md) | [Next: Maximum Subarray →](03-maximum-subarray.md)

[← Back to README](../README.md)
