# Chapter 1: Matrix Chain Multiplication (MCM)

## 📋 Overview
**Matrix Chain Multiplication** is the quintessential interval DP problem. Given a chain of matrices, find the optimal way to parenthesize their multiplication to **minimize the total number of scalar multiplications**. The order of multiplication doesn't change the result, but dramatically affects the cost.

---

## 🧠 Core Concept

```
Matrices: A₁(10×30), A₂(30×5), A₃(5×60)

Two ways to parenthesize:
  (A₁ × A₂) × A₃:
    A₁ × A₂ = 10×30 × 30×5 → cost 10×30×5 = 1500, result 10×5
    (result) × A₃ = 10×5 × 5×60 → cost 10×5×60 = 3000
    Total: 1500 + 3000 = 4500

  A₁ × (A₂ × A₃):
    A₂ × A₃ = 30×5 × 5×60 → cost 30×5×60 = 9000, result 30×60
    A₁ × (result) = 10×30 × 30×60 → cost 10×30×60 = 18000
    Total: 9000 + 18000 = 27000

Difference: 4500 vs 27000 — 6x difference!
Optimal: (A₁ × A₂) × A₃ with cost 4500
```

---

## 🔨 Interval DP Framework

```
Dimensions given as array p[0..n]:
  Matrix i has dimensions p[i-1] × p[i]

State: dp[i][j] = min cost to multiply matrices i through j

Recurrence:
  dp[i][j] = min over k from i to j-1 of:
      dp[i][k] + dp[k+1][j] + p[i-1] × p[k] × p[j]
      \_______/   \________/   \___________________/
      left part    right part   cost of combining

Base: dp[i][i] = 0 (single matrix, no multiplication needed)

Pattern:
  ┌──────────────────────────────────────────┐
  │ Try every possible split point k         │
  │ Left subchain:  matrices i..k            │
  │ Right subchain: matrices k+1..j          │
  │ Combining cost: p[i-1] × p[k] × p[j]    │
  │ Total = left_cost + right_cost + combine │
  └──────────────────────────────────────────┘
```

---

## 💻 Pseudocode

```
function MCM(p):
    n = len(p) - 1        // number of matrices
    dp = 2D array [n+1][n+1], all 0
    
    // Fill diagonally: increasing chain length
    for length = 2 to n:
        for i = 1 to n - length + 1:
            j = i + length - 1
            dp[i][j] = ∞
            for k = i to j - 1:
                cost = dp[i][k] + dp[k+1][j] + p[i-1] * p[k] * p[j]
                dp[i][j] = min(dp[i][j], cost)
    
    return dp[1][n]

Time: O(n³), Space: O(n²)
```

---

## 🔬 Trace: p = [10, 30, 5, 60]

```
Matrices: A₁(10×30), A₂(30×5), A₃(5×60)
n = 3 matrices

Base: dp[1][1]=0, dp[2][2]=0, dp[3][3]=0

Length 2:
  dp[1][2]: (A₁×A₂)
    k=1: dp[1][1] + dp[2][2] + p[0]×p[1]×p[2]
        = 0 + 0 + 10×30×5 = 1500
    dp[1][2] = 1500

  dp[2][3]: (A₂×A₃)
    k=2: dp[2][2] + dp[3][3] + p[1]×p[2]×p[3]
        = 0 + 0 + 30×5×60 = 9000
    dp[2][3] = 9000

Length 3:
  dp[1][3]: (A₁×A₂×A₃)
    k=1: dp[1][1] + dp[2][3] + p[0]×p[1]×p[3]
        = 0 + 9000 + 10×30×60 = 27000
    k=2: dp[1][2] + dp[3][3] + p[0]×p[2]×p[3]
        = 1500 + 0 + 10×5×60 = 4500
    dp[1][3] = min(27000, 4500) = 4500

Answer: dp[1][3] = 4500, split at k=2: (A₁A₂)A₃
```

---

## 🔄 Fill Order Visualization

```
The DP table is filled DIAGONALLY:

     j=1  j=2  j=3  j=4
i=1 [  0 | L2 | L3 | L4 ]   L = chain Length
i=2 [    |  0 | L2 | L3 ]
i=3 [    |    |  0 | L2 ]   Fill order:
i=4 [    |    |    |  0 ]   1. Main diagonal (base)
                             2. L2 diagonal
                             3. L3 diagonal
                             4. L4 diagonal

Each cell depends on cells to its LEFT and BELOW:

  dp[i][j] uses dp[i][k] (left in same row)
                 and dp[k+1][j] (below in same column)

  dp[i][j] ←── dp[i][k]
     ↑
  dp[k+1][j]
```

---

## 🔄 Reconstructing Optimal Parenthesization

```
function MCM_reconstruct(p):
    // Same DP but track split points
    split = 2D array [n+1][n+1]
    
    for length = 2 to n:
        for i = 1 to n-length+1:
            j = i + length - 1
            dp[i][j] = ∞
            for k = i to j-1:
                cost = dp[i][k] + dp[k+1][j] + p[i-1]*p[k]*p[j]
                if cost < dp[i][j]:
                    dp[i][j] = cost
                    split[i][j] = k
    
    return printOptimal(split, 1, n)

function printOptimal(split, i, j):
    if i == j:
        print "A" + i
    else:
        print "("
        printOptimal(split, i, split[i][j])
        printOptimal(split, split[i][j]+1, j)
        print ")"
```

---

## 🌐 Interval DP Pattern

```
MCM establishes the INTERVAL DP template:

  for length = 2 to n:              // interval size
      for i = start positions:       // left endpoint
          j = i + length - 1        // right endpoint
          for k = i to j-1:         // split point
              dp[i][j] = optimize(dp[i][k] ⊕ dp[k+1][j] ⊕ cost(i,k,j))

This pattern appears in:
  ┌────────────────────────────────┐
  │ • Matrix Chain Multiplication  │
  │ • Burst Balloons              │
  │ • Merge Stones                │
  │ • Optimal BST                 │
  │ • Palindrome Partitioning     │
  │ • Strange Printer             │
  └────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min cost for matrices i..j |
| **Split** | Try every k from i to j-1 |
| **Cost** | dp[i][k] + dp[k+1][j] + p[i-1]×p[k]×p[j] |
| **Fill order** | Diagonal (increasing interval length) |
| **Base** | dp[i][i] = 0 |
| **Complexity** | O(n³) time, O(n²) space |

---

## ❓ Quick Revision Questions

1. **What does each split point k represent?**
2. **Why is the fill order diagonal?**
3. **What is the combining cost p[i-1]×p[k]×p[j]?**
4. **How many split points does an interval of length L have?**
5. **How do you reconstruct the optimal parenthesization?**
6. **What is the general interval DP template?**

---

[← Previous Unit: Building Bridges](../09-LIS-Pattern/06-building-bridges.md) | [Next: Burst Balloons →](02-burst-balloons.md)

[← Back to README](../README.md)
