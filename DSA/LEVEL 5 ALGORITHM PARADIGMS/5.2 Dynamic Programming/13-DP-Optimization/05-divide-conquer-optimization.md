# Chapter 5: Divide and Conquer Optimization

## 📋 Overview
**Divide and Conquer Optimization** reduces certain $O(n \cdot m \cdot n)$ or $O(n^2 \cdot m)$ DP problems to $O(n \cdot m \cdot \log n)$. It applies when the **optimal split point** is **monotone**: $opt(i, j) \leq opt(i, j+1)$.

---

## 🧠 Core Idea

### The Setup
```
dp[k][i] = min over j<i { dp[k-1][j] + cost(j+1, i) }

Meaning: partition first i elements into k groups,
         minimizing total cost.

Naive: O(k · n²)  — try all j for each (k, i)
```

### The Monotonicity Condition
```
opt(k, i) = the j that minimizes dp[k-1][j] + cost(j+1, i)

If opt(k, i) ≤ opt(k, i+1) for all i:
  "As i increases, the optimal split point doesn't decrease"

This is the MONOTONE MINIMA property.
```

### Visualization
```
            j →
        0  1  2  3  4  5  6  7
  i=0   *               
  i=1      *            opt(i) moves right (or stays)
  i=2         *         
  i=3            *      
  i=4               *   
  i=5                  *
  i=6                     *
  i=7                        *

opt(k,0) ≤ opt(k,1) ≤ opt(k,2) ≤ ... ≤ opt(k,n)
```

### Divide and Conquer on i
```
Instead of computing dp[k][i] for all i left-to-right:

solve(lo, hi, opt_lo, opt_hi):
    mid = (lo + hi) / 2
    
    Find opt(k, mid) by scanning j from opt_lo to opt_hi
    Set dp[k][mid]
    
    Recurse:
      solve(lo, mid-1, opt_lo, opt(k,mid))     // left half
      solve(mid+1, hi, opt(k,mid), opt_hi)      // right half

Each level of recursion scans at most O(n) total
Depth: O(log n)
Total: O(n log n) per layer k → O(k · n · log n)
```

---

## 🔍 Step-by-Step Trace

```
n=8 elements, computing dp[k][0..7]
opt range: [0, 7]

solve(0, 7, 0, 7):
  mid = 3
  Scan j = 0..3 to find opt(k,3) → say opt=1
  dp[k][3] = dp[k-1][1] + cost(2,3)
  
  solve(0, 2, 0, 1):         solve(4, 7, 1, 7):
    mid = 1                     mid = 5
    Scan j = 0..1               Scan j = 1..5
    opt(k,1) → say 0            opt(k,5) → say 3
    
    solve(0,0,0,0):             solve(4,4,1,3):
      mid = 0                     mid = 4
      Scan j = 0..0               Scan j = 1..3
                                   
    solve(2,2,0,1):             solve(6,7,3,7):
      mid = 2                     mid = 6
      Scan j = 0..1               Scan j = 3..6

Total work per level: O(n)
Levels: O(log n)
Total: O(n log n)
```

---

## 💻 Pseudocode

```
function dpWithDCOptimization(n, K, cost):
    dp = 2D array [K+1][n] = INF
    
    // Base case: k=1
    for i = 0 to n-1:
        dp[1][i] = cost(0, i)
    
    // Fill each layer
    for k = 2 to K:
        solve(k, 0, n-1, 0, n-1)
    
    return dp[K][n-1]

function solve(k, lo, hi, opt_lo, opt_hi):
    if lo > hi: return
    
    mid = (lo + hi) / 2
    best_cost = INF
    best_opt = opt_lo
    
    // Scan j from opt_lo to min(opt_hi, mid-1)
    for j = opt_lo to min(opt_hi, mid - 1):
        val = dp[k-1][j] + cost(j+1, mid)
        if val < best_cost:
            best_cost = val
            best_opt = j
    
    dp[k][mid] = best_cost
    
    // Recurse with narrowed opt ranges
    solve(k, lo, mid-1, opt_lo, best_opt)
    solve(k, mid+1, hi, best_opt, opt_hi)
```

---

## 🔧 When Does Monotonicity Hold?

### Sufficient Condition: Quadrangle Inequality
```
cost(a, c) + cost(b, d) ≤ cost(a, d) + cost(b, c)
for all a ≤ b ≤ c ≤ d

This guarantees opt(k, i) ≤ opt(k, i+1).

Common cost functions satisfying this:
  cost(i, j) = (prefix[j] - prefix[i])²
  cost(i, j) = w[i][j] · (j - i)
  Concave SMAWK-type costs
```

### How to Verify
```
1. Prove quadrangle inequality algebraically
2. OR compute opt(k, i) for small cases and check monotonicity
3. OR recognize known problem patterns
```

---

## 📊 Classic Problems

### Divide Array into K Parts (Min Sum of Squares)
```
dp[k][i] = min_j { dp[k-1][j] + (sum(j+1..i))² }

cost(l, r) = (prefix[r] - prefix[l])²

Quadrangle inequality holds:
  (sum of shorter)² + (sum of shorter)²
  ≤ (sum of longer)² + (sum of 0)²   ← by convexity

O(K · n · log n)
```

### CSES - Elevator Rides
```
n items into minimum groups with weight limit
→ bitmask DP (different technique, not D&C opt)
```

### Post Office Problem
```
Place k post offices on a line to minimize total distance.
dp[k][i] = min_j { dp[k-1][j] + w(j+1, i) }
w(l, r) = cost of assigning interval [l,r] to one post office.
Monotonicity holds → D&C optimization.
```

---

## ⚡ Complexity Analysis

```
┌───────────────────────────────────────────┐
│ Naive:      O(K · n²)                    │
│ D&C Opt:    O(K · n · log n)             │
│                                           │
│ Speedup:    n / log n                     │
│   n=10⁵: ~10⁵/17 ≈ 6000× faster         │
│                                           │
│ Space: O(K · n) for dp table              │
│   Can optimize to O(n) with rolling array │
│                                           │
│ Requirement: opt(k,i) ≤ opt(k,i+1)       │
│   (monotone optimal split point)          │
└───────────────────────────────────────────┘
```

---

## 🆚 D&C Optimization vs Other Techniques

```
┌──────────────────────────────────────────────────┐
│ Technique          │ Reduces        │ Condition   │
├────────────────────┼────────────────┼─────────────┤
│ D&C Optimization   │ O(Kn²)→O(Kn㏒n)│ Monotone opt│
│ Convex Hull Trick  │ O(n²) → O(n)  │ Linear cost │
│ Knuth's Opt        │ O(n³) → O(n²) │ Quad. ineq. │
│ SMAWK              │ O(nm) → O(n+m)│ Totally mono│
└──────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Form** | dp[k][i] = min_j { dp[k-1][j] + cost(j+1,i) } |
| **Condition** | opt(k,i) ≤ opt(k,i+1) (monotone) |
| **Technique** | D&C on the range of i, narrow opt range |
| **Complexity** | O(K · n · log n) |
| **Guarantee** | Quadrangle inequality → monotonicity |
| **Vs naive** | Factor n/log n speedup |

---

## ❓ Quick Revision Questions

1. **What condition must hold for D&C optimization to apply?**
2. **How does the divide and conquer work on the index range?**
3. **Why is the total work O(n log n) per layer?**
4. **What is the quadrangle inequality and why does it imply monotonicity?**
5. **Give an example problem where D&C optimization applies.**
6. **How does D&C optimization compare to the Convex Hull Trick?**

---

[← Previous: Convex Hull Trick](04-convex-hull-trick.md) | [Next: Knuth's Optimization →](06-knuth-optimization.md)

[← Back to README](../README.md)
