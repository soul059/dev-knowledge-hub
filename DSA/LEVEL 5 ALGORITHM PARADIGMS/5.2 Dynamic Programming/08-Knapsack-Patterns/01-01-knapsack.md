# Chapter 1: 0/1 Knapsack

## 📋 Overview
The **0/1 Knapsack** is arguably the most important DP pattern. Given items with weights and values, and a knapsack with limited capacity, maximize the total value you can carry. Each item can be either taken (1) or left (0) — hence "0/1."

---

## 🧠 Core Concept

```
Items:
  ┌──────┬────────┬───────┐
  │ Item │ Weight │ Value │
  ├──────┼────────┼───────┤
  │  A   │   1    │   1   │
  │  B   │   3    │   4   │
  │  C   │   4    │   5   │
  │  D   │   5    │   7   │
  └──────┴────────┴───────┘
  
  Knapsack capacity W = 7

  Optimal: B + C = weight 7, value 9
  Or:      A + B + ? = weight 4, value 5 (worse)
  Or:      A + D = weight 6, value 8
  Or:      B + C = weight 7, value 9 ← best

  Decision at each item: TAKE or SKIP
```

---

## 🔨 Recurrence

```
State: dp[i][w] = max value using items 0..i-1 with capacity w

For item i-1 (0-indexed):
  if weight[i-1] > w:
      dp[i][w] = dp[i-1][w]        // can't take → skip
  else:
      dp[i][w] = max(
          dp[i-1][w],                          // SKIP item i-1
          dp[i-1][w - weight[i-1]] + value[i-1] // TAKE item i-1
      )

Base: dp[0][w] = 0 for all w (no items → no value)
      dp[i][0] = 0 for all i (no capacity → no value)
```

---

## 🔬 Trace: weights=[1,3,4,5], values=[1,4,5,7], W=7

```
       w=0  w=1  w=2  w=3  w=4  w=5  w=6  w=7
  0  [  0    0    0    0    0    0    0    0  ]
  A  [  0    1    1    1    1    1    1    1  ]
  B  [  0    1    1    4    5    5    5    5  ]
  C  [  0    1    1    4    5    6    6    9  ]
  D  [  0    1    1    4    5    7    8    9  ]

Key cells:
  dp[2][3]: B fits (w=3≥3) → max(dp[1][3]=1, dp[1][0]+4=4) = 4
  dp[2][4]: B fits → max(dp[1][4]=1, dp[1][1]+4=5) = 5
  dp[3][7]: C fits (w=7≥4) → max(dp[2][7]=5, dp[2][3]+5=9) = 9
  dp[4][7]: D fits (w=7≥5) → max(dp[3][7]=9, dp[3][2]+7=8) = 9

Answer: dp[4][7] = 9 (items B+C)
```

---

## 💻 Space Optimization: 1D Array

```
Key insight: dp[i][w] only depends on dp[i-1][w] and dp[i-1][w-weight]
             Both from the PREVIOUS row.

Iterate w from RIGHT to LEFT to avoid using updated values:

function knapsack(weights, values, W):
    n = len(weights)
    dp = array of size W+1, filled with 0
    
    for i = 0 to n-1:
        for w = W down to weights[i]:    // RIGHT TO LEFT!
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[W]

Why right-to-left?
  w=7: dp[7] = max(dp[7], dp[7-4]+5) = max(5, dp[3]+5)
       dp[3] hasn't been updated yet → uses PREVIOUS row's value ✓
  
  If left-to-right: dp[3] would already be updated → 
  dp[7] would use THIS row's dp[3] → effectively allowing item twice ✗

Time: O(n·W), Space: O(W)
```

---

## 🔄 Reconstructing Selected Items

```
function reconstruct(dp_table, weights, values, W):
    items = []
    w = W
    for i = n down to 1:
        if dp_table[i][w] != dp_table[i-1][w]:
            items.append(i-1)       // item i-1 was taken
            w -= weights[i-1]
    return reverse(items)

Trace: dp[4][7]=9 ≠ dp[3][7]=9 → D not taken
       dp[3][7]=9 ≠ dp[2][7]=5 → C taken! w=7-4=3
       dp[2][3]=4 ≠ dp[1][3]=1 → B taken! w=3-3=0
       dp[1][0]=0 == dp[0][0]=0 → A not taken

Items: B, C ✓ (value 4+5=9, weight 3+4=7)
```

---

## 🌐 Variants

```
┌─────────────────────┬──────────────────────────────────┐
│ Variant             │ Modification                     │
├─────────────────────┼──────────────────────────────────┤
│ Unbounded Knapsack  │ Items can be used multiple times │
│                     │ → Left-to-right iteration        │
│ Bounded Knapsack    │ Each item has limited quantity   │
│                     │ → Binary splitting technique     │
│ Fractional Knapsack │ Can take fractions of items      │
│                     │ → Greedy (not DP!)               │
│ Multi-dimensional   │ Multiple constraints (weight +   │
│                     │ volume) → 3D dp array            │
└─────────────────────┴──────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][w] = max value with items 0..i-1, capacity w |
| **Recurrence** | max(skip, take) = max(dp[i-1][w], dp[i-1][w-wt]+val) |
| **Base** | dp[0][w] = 0, dp[i][0] = 0 |
| **Space Opt** | 1D array, iterate w right-to-left |
| **Complexity** | O(n·W) time, O(W) space |
| **Constraint** | Each item used at most once |

---

## ❓ Quick Revision Questions

1. **What is the binary choice at each item?**
2. **Why must we iterate right-to-left in the space-optimized version?**
3. **What happens if we iterate left-to-right instead?**
4. **How do you reconstruct which items were selected?**
5. **What is the time complexity and why is it pseudo-polynomial?**
6. **When would Fractional Knapsack use Greedy instead of DP?**

---

[← Previous Unit: Palindrome Removal](../07-Palindrome-DP/05-palindrome-removal.md) | [Next: Unbounded Knapsack →](02-unbounded-knapsack.md)

[← Back to README](../README.md)
