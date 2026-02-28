# Chapter 6: Knuth's Optimization

## 📋 Overview
**Knuth's Optimization** reduces interval DP from $O(n^3)$ to $O(n^2)$ by exploiting the **monotonicity of optimal split points**. It applies to recurrences like $dp[i][j] = \min_{i \leq k < j}(dp[i][k] + dp[k+1][j] + w(i,j))$ when the cost $w$ satisfies the **quadrangle inequality**.

---

## 🧠 Core Idea

### The Interval DP Recurrence
```
dp[i][j] = min over k ∈ [i, j-1] of:
    dp[i][k] + dp[k+1][j] + w(i, j)

This is the structure of:
  - Optimal BST
  - Matrix chain multiplication
  - Merging stones
```

### Naive Complexity
```
For each (i, j) pair: try all k in [i, j-1]
  Pairs: O(n²)
  Splits per pair: O(n)
  Total: O(n³)
```

### Knuth's Key Observation
```
Let opt(i, j) = the k that minimizes dp[i][k] + dp[k+1][j] + w(i,j)

If w satisfies the quadrangle inequality:
  w(a, c) + w(b, d) ≤ w(a, d) + w(b, c)   for a ≤ b ≤ c ≤ d

AND w is monotone:
  w(a, c) ≤ w(b, d)   for a ≤ b ≤ c ≤ d

Then: opt(i, j-1) ≤ opt(i, j) ≤ opt(i+1, j)
```

### How This Helps
```
Instead of scanning k from i to j-1:
  Scan k from opt(i, j-1) to opt(i+1, j)

Total work across all (i, j) of fixed length L:
  Σ (opt(i+1, j) - opt(i, j-1))  telescopes!
  = O(n) per length L
  
n different lengths → O(n²) total!
```

---

## 🔍 Step-by-Step Trace: Optimal BST

```
Keys: 1, 2, 3 with frequencies [3, 1, 2]
w(i, j) = sum of frequencies from i to j

w(0,0)=3  w(0,1)=4  w(0,2)=6
           w(1,1)=1  w(1,2)=3
                      w(2,2)=2

Base: dp[i][i] = w[i][i], opt[i][i] = i

dp[0][0]=3, opt[0][0]=0
dp[1][1]=1, opt[1][1]=1
dp[2][2]=2, opt[2][2]=2

Length 2:
  dp[0][1]: k from opt[0][0]=0 to opt[1][1]=1
    k=0: dp[0][0] + dp[1][1] + w(0,1) = 3+1+4 = 8  ← min
    k=1: dp[0][1] is computing...wait — 
    Actually k=0: dp[0][-1] + dp[0][1]... 
    
  Let me use standard formulation:
  dp[i][j] = w(i,j) + min_k(dp[i][k-1] + dp[k+1][j])
  with dp[i][i-1] = 0 (empty tree)

  dp[0][1]: w(0,1)=4, try k ∈ {0,1}
    k=0: dp[0][-1] + dp[1][1] = 0+1 = 1
    k=1: dp[0][0] + dp[2][1] = 3+0 = 3
    min = 1 (k=0), dp[0][1] = 4+1 = 5, opt[0][1] = 0

  dp[1][2]: w(1,2)=3, try k ∈ {1,2}
    k=1: dp[1][0] + dp[2][2] = 0+2 = 2
    k=2: dp[1][1] + dp[3][2] = 1+0 = 1
    min = 1 (k=2), dp[1][2] = 3+1 = 4, opt[1][2] = 2

Length 3:
  dp[0][2]: w(0,2)=6, k from opt[0][1]=0 to opt[1][2]=2
    k=0: 0 + dp[1][2] = 0+4 = 4
    k=1: dp[0][0] + dp[2][2] = 3+2 = 5
    k=2: dp[0][1] + 0 = 5+0 = 5
    min = 4 (k=0), dp[0][2] = 6+4 = 10, opt[0][2] = 0

Without Knuth: tried 3 values of k (all)
With Knuth: tried 3 values (opt[0][1]..opt[1][2] = 0..2)
  Savings appear with larger n!
```

---

## 💻 Pseudocode

```
function optimalBSTKnuth(freq, n):
    // w[i][j] = sum of frequencies from i to j
    // Precompute using prefix sums
    w = 2D array [n][n]
    for i = 0 to n-1:
        w[i][i] = freq[i]
        for j = i+1 to n-1:
            w[i][j] = w[i][j-1] + freq[j]
    
    dp = 2D array [n][n] = 0
    opt = 2D array [n][n]
    
    // Base case: single elements
    for i = 0 to n-1:
        dp[i][i] = freq[i]
        opt[i][i] = i
    
    // Fill by increasing length
    for length = 2 to n:
        for i = 0 to n - length:
            j = i + length - 1
            dp[i][j] = INF
            
            // KEY: only scan k in [opt[i][j-1], opt[i+1][j]]
            lo = opt[i][j-1]
            hi = opt[i+1][j]     // handle boundary if i+1 > j
            
            for k = lo to hi:
                // dp[i][k-1] and dp[k+1][j] with boundary checks
                left = (k > i) ? dp[i][k-1] : 0
                right = (k < j) ? dp[k+1][j] : 0
                val = left + right + w[i][j]
                
                if val < dp[i][j]:
                    dp[i][j] = val
                    opt[i][j] = k
    
    return dp[0][n-1]
```

---

## 🔧 Verifying Quadrangle Inequality

### For Optimal BST
```
w(i, j) = Σ freq[i..j]  (sum of subarray)

Check: w(a,c) + w(b,d) ≤ w(a,d) + w(b,c)  for a ≤ b ≤ c ≤ d

LHS = Σ[a..c] + Σ[b..d]
RHS = Σ[a..d] + Σ[b..c]

RHS - LHS = Σ[c+1..d] - Σ[c+1..d] + Σ[a..b-1] - Σ[a..b-1]... 

Actually, for prefix sums:
  Both sides equal Σ[a..c] + Σ[b..d]
  Since Σ[a..d] = Σ[a..c] + Σ[c+1..d]
  and   Σ[b..c] ⊂ Σ[a..c]

The sum function satisfies QI with equality. ✓
```

### For Merge Stones
```
cost(i, j) = sum of stones from i to j
Same as above — satisfies QI.

Matrix Chain: does NOT satisfy QI in general
  (Knuth's optim doesn't apply to standard MCM)
```

---

## 🆚 Comparison with Other Optimizations

```
┌──────────────────────────────────────────────────────────┐
│ Optimization        │ From      │ To        │ Applies to │
├──────────────────────┼───────────┼───────────┼────────────┤
│ Knuth's             │ O(n³)     │ O(n²)     │ Interval DP│
│                     │           │           │ w has QI    │
├──────────────────────┼───────────┼───────────┼────────────┤
│ D&C Optimization    │ O(Kn²)   │ O(Kn㏒n)  │ 1D split   │
│                     │           │           │ Monotone opt│
├──────────────────────┼───────────┼───────────┼────────────┤
│ CHT                 │ O(n²)     │ O(n㏒n)/n │ Linear cost│
│                     │           │           │ in DP form  │
└──────────────────────────────────────────────────────────┘

Knuth's: specifically for INTERVAL DP
D&C Opt: for PARTITIONING into K groups
CHT:     for LINEAR transition costs
```

---

## 📊 Problems Where Knuth's Applies

| Problem | w(i,j) | QI holds? |
|---------|--------|-----------|
| Optimal BST | Σ freq[i..j] | ✓ (equality) |
| Merge stones (K=2) | Σ stones[i..j] | ✓ |
| Paragraph formatting | Penalty for line lengths | ✓ for certain penalties |
| Alphabetic tree | Σ weights[i..j] | ✓ |

---

## ⚡ Complexity Proof Sketch

```
Total inner loop iterations:
  For fixed length L:
    Σ_{i} (opt[i+1][j] - opt[i][j-1] + 1)
    
  Where j = i + L - 1:
    = Σ_{i} (opt[i+1][i+L-1] - opt[i][i+L-2] + 1)
    
  Telescoping:
    ≤ opt[n-L+1][n-1] - opt[0][L-2] + n - L + 1
    ≤ n + n = O(n)
    
  Summed over all lengths L=2..n:
    O(n) × n = O(n²)  ✓
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Form** | dp[i][j] = min_k(dp[i][k] + dp[k+1][j]) + w(i,j) |
| **Condition** | w satisfies quadrangle inequality + monotonicity |
| **Key insight** | opt(i,j-1) ≤ opt(i,j) ≤ opt(i+1,j) |
| **Complexity** | O(n²) instead of O(n³) |
| **Classic use** | Optimal BST, merging stones |
| **Store** | opt[i][j] table alongside dp[i][j] |

---

## ❓ Quick Revision Questions

1. **What is the quadrangle inequality for cost function w?**
2. **How does opt(i,j-1) ≤ opt(i,j) ≤ opt(i+1,j) reduce complexity?**
3. **Why does sum-of-subarray satisfy the quadrangle inequality?**
4. **Does Knuth's optimization apply to matrix chain multiplication?**
5. **What must you store in addition to dp[i][j]?**
6. **How does the telescoping argument prove O(n²) total work?**

---

[← Previous: D&C Optimization](05-divide-conquer-optimization.md)

[← Back to README](../README.md)
