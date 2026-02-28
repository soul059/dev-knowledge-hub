# Chapter 3: House Robber (Fibonacci Perspective)

## 📋 Overview
The **House Robber** problem is revisited here through the lens of the Fibonacci pattern. While Unit 3 covered the problem in detail, this chapter focuses on **why** it follows the Fibonacci structure and how to recognize this pattern in similar problems.

---

## 🧠 The Fibonacci Connection

```
Fibonacci:    F(n) = F(n-1) + F(n-2)
House Robber: dp[i] = max(dp[i-1], dp[i-2] + nums[i])

Key Insight:
┌─────────────────────────────────────────────────┐
│  Both depend on exactly TWO previous states.    │
│  Both can be optimized to O(1) space with       │
│  two variables.                                 │
│  The recurrence has the same "shape."           │
└─────────────────────────────────────────────────┘

Fibonacci Pattern Generalization:
  dp[i] = f(dp[i-1], dp[i-2], ...)
  where we combine a fixed number of prior results
```

---

## 🔍 Pattern Anatomy

```
At each house i, we make a BINARY choice:
  ┌─ SKIP house i  → take best of previous: dp[i-1]
  │
  └─ ROB house i   → take house i + best from two back: dp[i-2] + nums[i]

dp[i] = max(SKIP, ROB)
       = max(dp[i-1], dp[i-2] + nums[i])

This is a "max-Fibonacci" — instead of ADD, we use MAX with a value.

Standard Fibonacci:
  dp[i] = dp[i-1]  +  dp[i-2]
           ↑ prev      ↑ two back

House Robber:
  dp[i] = max(dp[i-1], dp[i-2] + nums[i])
              ↑ prev    ↑ two back + value
```

---

## 🔬 Trace: Side-by-Side Comparison

```
nums = [2, 7, 9, 3, 1]

Plain Fibonacci (for reference):
  dp: 0, 1, 1, 2, 3, 5

House Robber dp:
  i=0: dp[0] = 2                    (base: first house)
  i=1: dp[1] = max(2, 7) = 7       (base: max of first two)
  i=2: dp[2] = max(7, 2+9) = 11    (skip=7, rob=11)
  i=3: dp[3] = max(11, 7+3) = 11   (skip=11, rob=10)
  i=4: dp[4] = max(11, 11+1) = 12  (skip=11, rob=12)

Answer: 12 (rob houses 0, 2, 4 → 2+9+1=12)
```

---

## 🔄 Space Optimization — Same as Fibonacci

```
function rob(nums):
    if len(nums) == 0: return 0
    if len(nums) == 1: return nums[0]

    prev2 = 0     // dp[i-2]
    prev1 = 0     // dp[i-1]

    for num in nums:
        current = max(prev1, prev2 + num)
        prev2 = prev1
        prev1 = current

    return prev1

Compare with Fibonacci:
    a, b = 0, 1
    for i = 2 to n:
        a, b = b, a + b     ← same structure!
    return b

House Robber:
    prev2, prev1 = 0, 0
    for num in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + num)
    return prev1
```

---

## 🌐 Variants Through the Fibonacci Lens

### Variant 1: House Robber II (Circular)
```
Houses are in a circle: house 0 and house n-1 are adjacent.

Solution: Two separate linear passes
  Case A: rob houses [0 .. n-2]  (exclude last)
  Case B: rob houses [1 .. n-1]  (exclude first)
  Answer: max(Case A, Case B)

Each case is the standard Fibonacci pattern.
```

### Variant 2: House Robber III (Tree)
```
Houses form a binary tree (not linear).

Fibonacci pattern extends to tree DP:
  rob(node) = max(
      SKIP: rob(left) + rob(right),
      ROB:  node.val + rob(left.left) + rob(left.right)
                     + rob(right.left) + rob(right.right)
  )

Better formulation — each node returns (rob_it, skip_it):
  function dfs(node):
      if node is null: return (0, 0)
      left = dfs(node.left)
      right = dfs(node.right)
      rob_it  = node.val + left[1] + right[1]
      skip_it = max(left) + max(right)
      return (rob_it, skip_it)
```

---

## 💡 The "No-Two-Adjacent" Family

```
Any problem with the constraint "cannot select two adjacent elements"
follows the House Robber / Fibonacci pattern:

┌──────────────────────────┬─────────────────────────────────────┐
│ Problem                  │ Recurrence                          │
├──────────────────────────┼─────────────────────────────────────┤
│ House Robber             │ max(skip, rob) = max(dp[i-1],       │
│                          │   dp[i-2]+val)                      │
│ Max Sum Non-Adjacent     │ Same as House Robber                │
│ Delete and Earn          │ Transform → House Robber            │
│ Count ways no 2 adj      │ dp[i] = dp[i-1] + dp[i-2]          │
│ Binary strings no 2 adj  │ dp[i] = dp[i-1] + dp[i-2]          │
│   consecutive 1s         │                                     │
│ Tiling 2×n board         │ dp[i] = dp[i-1] + dp[i-2]          │
└──────────────────────────┴─────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Connection** | House Robber is a "max-Fibonacci" recurrence |
| **Recurrence** | dp[i] = max(dp[i-1], dp[i-2] + nums[i]) |
| **Variables** | prev1 (dp[i-1]) and prev2 (dp[i-2]) |
| **Complexity** | O(n) time, O(1) space |
| **Pattern Family** | "No-two-adjacent" problems |
| **Key Insight** | Binary choice (skip/take) with 2-state dependency |

---

## ❓ Quick Revision Questions

1. **How does House Robber relate to the Fibonacci pattern structurally?**
2. **What is the binary choice at each step in House Robber?**
3. **How does the space optimization mirror Fibonacci's?**
4. **How does "House Robber II" (circular) break into two Fibonacci sub-problems?**
5. **What constraint makes a problem follow the "no-two-adjacent" pattern?**
6. **How does House Robber III extend this pattern to trees?**

---

[← Previous: Tribonacci](02-tribonacci.md) | [Next: Climbing Stairs Variations →](04-climbing-stairs-variations.md)

[← Back to README](../README.md)
