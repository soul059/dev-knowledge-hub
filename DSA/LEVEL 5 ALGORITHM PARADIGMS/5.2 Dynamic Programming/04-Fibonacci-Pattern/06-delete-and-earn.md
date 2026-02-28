# Chapter 6: Delete and Earn

## 📋 Overview
**Delete and Earn** (LeetCode #740) is an elegant problem that **transforms into House Robber** after a preprocessing step. It beautifully demonstrates how to reduce an unfamiliar problem to the Fibonacci pattern.

---

## 🧠 Problem Statement

```
Given an array of integers nums:
- Pick any nums[i], earn nums[i] points
- Delete every occurrence of nums[i]-1 and nums[i]+1
- Repeat until array is empty
- Maximize total points earned

Example: nums = [3, 4, 2]
  Pick 4: earn 4, delete 3 and 5 → remaining [2]
  Pick 2: earn 2 → total = 6

Example: nums = [2, 2, 3, 3, 3, 4]
  Pick all 3s: earn 9, delete 2s and 4s → total = 9
```

---

## 🔍 Key Insight: Reduction to House Robber

```
Observation: If we pick value v, we earn ALL occurrences of v.
             Then we can't pick v-1 or v+1.

This is exactly "no two adjacent" — on SORTED VALUES!

Step 1: Compute total points for each value
  points[v] = v × count(v)

Step 2: Create an array indexed by value (0 to max_val)
  This becomes a House Robber problem:
  "Can't rob two adjacent houses" = "Can't pick two adjacent values"

┌──────────────────────────────────────────────┐
│  Delete and Earn                             │
│  ═══════════════                             │
│  nums = [2, 2, 3, 3, 3, 4]                  │
│                                              │
│  Step 1: points[2]=4, points[3]=9, points[4]=4 │
│                                              │
│  Step 2: House Robber on [0, 0, 4, 9, 4]    │
│  "Adjacent values can't both be picked"      │
│                                              │
│  dp[0]=0, dp[1]=0, dp[2]=4, dp[3]=9, dp[4]=9 │
│  Answer: 9 (pick all 3s)                    │
└──────────────────────────────────────────────┘
```

---

## 🔨 Complete Solution

```
function deleteAndEarn(nums):
    if nums is empty: return 0

    maxVal = max(nums)

    // Step 1: Build points array
    points = array of size maxVal+1, filled with 0
    for num in nums:
        points[num] += num

    // Step 2: Apply House Robber on points array
    prev2 = 0
    prev1 = 0
    for i = 0 to maxVal:
        current = max(prev1, prev2 + points[i])
        prev2 = prev1
        prev1 = current

    return prev1
```

---

## 🔬 Trace: nums = [2, 2, 3, 3, 3, 4]

```
Step 1: Build points array
  points[2] = 2+2 = 4
  points[3] = 3+3+3 = 9
  points[4] = 4

  points = [0, 0, 4, 9, 4]

Step 2: House Robber on [0, 0, 4, 9, 4]
  Initial: prev2=0, prev1=0

  i=0, points=0:  curr = max(0, 0+0) = 0   → prev2=0, prev1=0
  i=1, points=0:  curr = max(0, 0+0) = 0   → prev2=0, prev1=0
  i=2, points=4:  curr = max(0, 0+4) = 4   → prev2=0, prev1=4
  i=3, points=9:  curr = max(4, 0+9) = 9   → prev2=4, prev1=9
  i=4, points=4:  curr = max(9, 4+4) = 9   → prev2=9, prev1=9

  Answer: 9 (pick all 3s, earning 9 points)
```

---

## 🔬 Trace: nums = [3, 1]

```
Step 1: Build points array
  points[1] = 1
  points[3] = 3

  points = [0, 1, 0, 3]

Step 2: House Robber on [0, 1, 0, 3]
  i=0, p=0: curr = max(0, 0+0)=0  → prev2=0, prev1=0
  i=1, p=1: curr = max(0, 0+1)=1  → prev2=0, prev1=1
  i=2, p=0: curr = max(1, 0+0)=1  → prev2=1, prev1=1
  i=3, p=3: curr = max(1, 1+3)=4  → prev2=1, prev1=4

  Answer: 4 (pick 1 and 3 since they're not adjacent values)
```

---

## 🔬 Trace: nums = [1, 1, 1, 2, 4, 5, 5, 5, 6]

```
Step 1: Build points array
  points[1] = 1+1+1 = 3
  points[2] = 2
  points[4] = 4
  points[5] = 5+5+5 = 15
  points[6] = 6

  points = [0, 3, 2, 0, 4, 15, 6]

Step 2: House Robber on [0, 3, 2, 0, 4, 15, 6]
  i=0, p=0:  curr=max(0,0+0)=0    → prev2=0, prev1=0
  i=1, p=3:  curr=max(0,0+3)=3    → prev2=0, prev1=3
  i=2, p=2:  curr=max(3,0+2)=3    → prev2=3, prev1=3
  i=3, p=0:  curr=max(3,3+0)=3    → prev2=3, prev1=3
  i=4, p=4:  curr=max(3,3+4)=7    → prev2=3, prev1=7
  i=5, p=15: curr=max(7,3+15)=18  → prev2=7, prev1=18
  i=6, p=6:  curr=max(18,7+6)=18  → prev2=18, prev1=18

  Answer: 18

  Verification:
  Pick 1s (earn 3) + pick 5s (earn 15) = 18
  1 and 5 don't conflict (not adjacent values) ✓
  Picking 5s deletes 4s and 6s (blocks earn 4+6=10)
  3 + 15 = 18 > 2 + 4 + 6 = 12 ✓
```

---

## 💡 Alternative: Sparse Approach

```
When max value is very large but few distinct values:

function deleteAndEarn(nums):
    counts = Counter(nums)        // {value: count}
    values = sorted(counts.keys()) // sorted unique values

    prev2 = 0
    prev1 = 0
    lastVal = -infinity

    for val in values:
        points = val * counts[val]
        if val == lastVal + 1:
            // Adjacent value — House Robber choice
            current = max(prev1, prev2 + points)
        else:
            // Non-adjacent — always take
            current = prev1 + points
        prev2 = prev1
        prev1 = current
        lastVal = val

    return prev1

This handles nums = [1, 1000000] efficiently
without creating a million-element array.

Time: O(n log n) for sorting, Space: O(k) where k = distinct values
```

---

## 🌐 The Transformation Pattern

```
Many problems can be TRANSFORMED into House Robber:

┌────────────────────┬─────────────────────────────────┐
│ Original Problem   │ Transformation                  │
├────────────────────┼─────────────────────────────────┤
│ Delete and Earn    │ Group by value → points array   │
│ Pizza with 3n      │ Circular → two linear passes    │
│   slices           │                                 │
│ Reduce array to    │ Map operations → adjacency      │
│   half             │                                 │
│ Destroy sequential │ Group consecutive → points      │
│   pairs            │                                 │
└────────────────────┴─────────────────────────────────┘

Recognition checklist:
✓ Choosing element X prevents choosing X±1
✓ Can group/accumulate elements by value
✓ Adjacent exclusion in value space (not index space)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Core Idea** | Transform to House Robber on value-indexed array |
| **Step 1** | points[v] = v × count(v) |
| **Step 2** | dp[v] = max(dp[v-1], dp[v-2] + points[v]) |
| **Complexity** | O(n + maxVal) time, O(maxVal) space |
| **Sparse** | O(n log n) time, O(k) space |
| **Key Pattern** | Reduction to known Fibonacci pattern |

---

## ❓ Quick Revision Questions

1. **How does Delete and Earn reduce to House Robber?**
2. **What is the preprocessing step (building the points array)?**
3. **Why can we earn ALL occurrences of a value when we pick it?**
4. **Walk through Delete and Earn for nums = [1, 1, 2, 2, 3].**
5. **When would you use the sparse approach over the dense array?**
6. **What other problems can be transformed into House Robber?**

---

[← Previous: Max Sum Non-Adjacent](05-max-sum-non-adjacent.md) | [Next Unit: Grid DP →](../05-Grid-DP/01-unique-paths.md)

[← Back to README](../README.md)
