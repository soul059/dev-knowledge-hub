# Chapter 5: DP Optimization

## 📋 Chapter Overview
Techniques to reduce DP time or space: rolling arrays, state compression, and monotonic optimization.

---

## 📝 Optimization 1: Space — Rolling Array

### Concept

```
  If dp[i] only depends on dp[i-1] (previous row),
  keep only 2 rows instead of n.
  
  2D → two 1D arrays (or even one with careful ordering)
```

### Example: LCS Space Optimization

```
  Standard: O(m×n) space
  Optimized: O(min(m,n)) space

function lcs(text1, text2):
    if len(text1) < len(text2): swap(text1, text2)
    m = len(text1), n = len(text2)
    
    prev = array of 0s, size n+1
    curr = array of 0s, size n+1
    
    for i = 1 to m:
        for j = 1 to n:
            if text1[i-1] == text2[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        swap(prev, curr)
        fill curr with 0s
    
    return prev[n]
```

### Single Row Optimization (Knapsack)

```
  0/1 Knapsack: iterate BACKWARDS on single array
  
  dp = array of 0s, size capacity+1
  for each item:
      for w = capacity down to weight[item]:
          dp[w] = max(dp[w], dp[w - weight] + value)
  
  Backwards ensures we don't use updated values from same item
```

---

## 📝 Optimization 2: State Compression (Bitmask DP)

### Concept

```
  When state involves a SET of items (which are visited/selected),
  represent the set as a bitmask.
  
  Set {0, 2, 3} with n=4: binary 1101 = 13
  
  dp[mask] = best value given the items in mask are used
```

### Example: Travelling Salesman (TSP)

```
function tsp(dist, n):
    full = (1 << n) - 1            // all bits set
    dp = 2D array [1<<n][n], filled with INF
    dp[1][0] = 0                   // start at city 0
    
    for mask = 1 to full:
        for u = 0 to n-1:
            if bit u not set in mask: continue
            for v = 0 to n-1:
                if bit v set in mask: continue  // already visited
                newMask = mask | (1 << v)
                dp[newMask][v] = min(dp[newMask][v],
                                    dp[mask][u] + dist[u][v])
    
    // Return to start
    result = INF
    for u = 1 to n-1:
        result = min(result, dp[full][u] + dist[u][0])
    return result
```

**Complexity:** O(2^n × n²) time, O(2^n × n) space. Works for n ≤ ~20.

---

## 📝 Optimization 3: Prefix Sum DP

### Concept

```
  When transition involves sum over a range:
  dp[i] = min(dp[j] + cost(j+1, i)) for all j
  
  If cost is a sum → precompute prefix sums → O(1) range sum
```

### Example: Minimum Cost to Split Array

```
  cost(j+1, i) = prefix[i] - prefix[j]
  No need to recompute sum each time
```

---

## 📝 Optimization 4: Monotonic Stack/Deque DP

### Concept

```
  dp[i] = min/max(dp[j] + f(j, i)) for j in some range
  
  If we can maintain candidates in a monotonic structure,
  reduce from O(n²) to O(n).
```

### Example: Sliding Window Maximum DP

```
  dp[i] = max(dp[j]) + nums[i] for j in [i-k, i-1]
  
  Use deque to track max dp value in window of size k → O(n)
```

---

## 📝 Optimization 5: Matrix Exponentiation

### Concept

```
  Linear recurrences like dp[i] = a*dp[i-1] + b*dp[i-2]
  can be expressed as matrix multiplication.
  
  Fibonacci: [F(n)]   = [1 1]^n   [F(1)]
             [F(n-1)]   [1 0]   × [F(0)]
  
  Matrix power via repeated squaring → O(log n).
```

### When to Use

```
  n is very large (10^18) but state is small (2-3 variables).
  Standard DP would be O(n) — too slow.
  Matrix exponentiation: O(k³ × log n) where k = state size.
```

---

## 📊 Optimization Summary

| Technique | Reduces | From → To | When |
|-----------|---------|-----------|------|
| Rolling array | Space | O(m×n) → O(n) | dp[i] depends only on dp[i-1] |
| Single row | Space | O(n×W) → O(W) | Knapsack-type |
| Bitmask | Representation | Set → integer | n ≤ 20, track subset |
| Prefix sum | Time per transition | O(n) → O(1) | Range sum in transition |
| Monotonic deque | Time | O(n²) → O(n) | Min/max in sliding range |
| Matrix exponent | Time | O(n) → O(log n) | Linear recurrence, huge n |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Space optimization | Keep only needed rows/states |
| Bitmask DP | Represent subsets as integers, n ≤ 20 |
| Backwards iteration | Prevents double-counting in 0/1 knapsack |
| Prefix sums | O(1) range queries in transitions |
| Matrix power | O(log n) for linear recurrences with tiny state |

---

## ❓ Revision Questions

1. **Rolling array: when applicable?** → When dp[i] depends only on dp[i-1] (or fixed number of previous rows).
2. **0/1 Knapsack: why backward iteration?** → Ensures each item is used at most once by not reusing updated cells.
3. **Bitmask DP: max n?** → ~20 (2^20 ≈ 1M states, manageable).
4. **Matrix exponentiation: when useful?** → Linear recurrence with small state size (k ≤ 5) but huge n (up to 10^18).
5. **Monotonic deque DP: what does it replace?** → Inner loop scanning for min/max in a range, reducing O(n²) to O(n).

---

[← Previous: Classic DP Problems](04-classic-dp-problems.md) | [Next: When to Apply →](06-when-to-apply.md)
