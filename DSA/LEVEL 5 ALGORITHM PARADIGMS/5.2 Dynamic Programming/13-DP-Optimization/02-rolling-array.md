# Chapter 2: Rolling Array Technique

## 📋 Overview
The **rolling array** generalizes the two-row trick to keep only the **last K rows** (or layers) of a DP table using **modular indexing**. It's essential when DP depends on a fixed window of previous states.

---

## 🧠 Core Idea

```
If dp[i] depends only on dp[i-1], dp[i-2], ..., dp[i-K]:
  Keep only K+1 rows using modular index: i % (K+1)

Example (K=2):
  dp[i] depends on dp[i-1] and dp[i-2]
  Use: dp[i % 3]

  i=0: slot 0    ┌───┐
  i=1: slot 1    │ 0 │ ← i%3=0
  i=2: slot 2    │ 1 │ ← i%3=1
  i=3: slot 0 ←  │ 2 │ ← i%3=2
  i=4: slot 1 ←  └───┘
  i=5: slot 2 ←  Overwrite oldest
```

---

## 📊 Example 1: Tribonacci Number

### Before (Full Array)
```
dp[0] = 0, dp[1] = 0, dp[2] = 1
dp[i] = dp[i-1] + dp[i-2] + dp[i-3]

Space: O(n)
```

### After (Rolling Array, K=3)
```
dp = array[4]  // indices 0,1,2,3
dp[0%4]=0, dp[1%4]=0, dp[2%4]=1

for i = 3 to n:
    dp[i % 4] = dp[(i-1) % 4] + dp[(i-2) % 4] + dp[(i-3) % 4]

return dp[n % 4]

Space: O(1)  (fixed 4 slots)
```

---

## 📊 Example 2: Edit Distance

### Dependency Pattern
```
dp[i][j] depends on:
  dp[i-1][j]     ← above
  dp[i][j-1]     ← left (same row)
  dp[i-1][j-1]   ← diagonal

Only row i-1 needed → K=1 → 2 rows
```

### Rolling Array Implementation
```
function editDistance(A, B):
    n, m = len(A), len(B)
    dp = 2D array [2][m+1]
    
    // Base case: row 0
    for j = 0 to m:
        dp[0][j] = j
    
    for i = 1 to n:
        cur = i % 2
        prv = (i-1) % 2
        dp[cur][0] = i    // base: first column
        
        for j = 1 to m:
            if A[i-1] == B[j-1]:
                dp[cur][j] = dp[prv][j-1]
            else:
                dp[cur][j] = 1 + min(
                    dp[prv][j],      // delete
                    dp[cur][j-1],    // insert
                    dp[prv][j-1]     // replace
                )
    
    return dp[n % 2][m]

Space: O(m) instead of O(n·m)
```

---

## 📊 Example 3: 3D DP with Rolling

### Problem: Cherry Pickup
```
dp[step][r1][r2] depends only on dp[step-1][...][...]

Keep 2 layers of the 3D table:

dp = 3D array [2][n][n]

for step in range:
    cur = step % 2
    prv = (step - 1) % 2
    
    for r1 = ...:
        for r2 = ...:
            dp[cur][r1][r2] = compute from dp[prv][...][...]

Space: O(n²) instead of O(n³)
```

---

## 🔧 Implementation Pattern

```
General rolling array template:

// K = number of previous rows needed
// Use (K+1) rows with modular indexing

ROWS = K + 1
dp = array [ROWS][...other dims...]

// Initialize base cases in dp[0..K-1]

for i = K to n:
    cur = i % ROWS
    
    // Clear current row if needed
    clear(dp[cur])
    
    // Compute from previous rows
    for j = ...:
        dp[cur][j] = f(
            dp[(i-1) % ROWS][j],
            dp[(i-2) % ROWS][j],
            ...,
            dp[(i-K) % ROWS][j]
        )

return dp[n % ROWS][answer_index]
```

---

## 🆚 Rolling Array vs Two-Row Swap

```
Two-row swap:                Rolling array:
  prev, curr arrays            dp[i % K] indexing
  Manual swap after row        No swap needed
  
  + Clear intent              + Generalizes to K rows
  + No modular arithmetic     + No swap overhead
  - Only for K=1              + Works for any K
  
Recommendation:
  K=1 → either works, two-row may be clearer
  K≥2 → rolling array with modular index
```

---

## ⚠️ Common Pitfalls

```
1. Forgetting to clear/reinitialize rolled-over rows
   dp[cur] may have stale values from (K+1) iterations ago
   
   Fix: explicitly clear or overwrite all cells in dp[cur]

2. Off-by-one in modular indexing
   dp[(i-K) % ROWS] when i < K → negative index!
   
   Fix: handle base cases before the loop
   
3. Accessing dp[(i-K-1) % ROWS] — already overwritten
   
   Fix: ensure you only access rows within the window

4. Not updating base cases for each new row
   e.g., dp[cur][0] = i for edit distance row header
```

---

## 📊 When to Use Rolling Array

| Scenario | Rows Needed | Modular Index |
|----------|-------------|---------------|
| dp[i] = f(dp[i-1]) | 2 | i % 2 |
| dp[i] = f(dp[i-1], dp[i-2]) | 3 | i % 3 |
| dp[i] = f(dp[i-1], ..., dp[i-K]) | K+1 | i % (K+1) |
| dp[i][j] = f(dp[i-1][j]) | 2 rows | i % 2 |
| 3D: dp[k][i][j] layer k-1 | 2 layers | k % 2 |

---

## ⚡ Complexity Impact

```
┌───────────────────────────────────────────────┐
│ Time:  UNCHANGED — same operations            │
│                                               │
│ Space savings:                                │
│   2D → 1D:  O(n·m) → O(m)                    │
│   3D → 2D:  O(n·m·k) → O(m·k)               │
│                                               │
│ Rolling K rows:                               │
│   O(n · other_dims) → O(K · other_dims)       │
│                                               │
│ Tradeoff: Cannot reconstruct path             │
│   (old rows overwritten)                      │
└───────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Technique** | Keep last K+1 rows using `i % (K+1)` |
| **K=1** | Two rows, equivalent to prev/curr swap |
| **K≥2** | General rolling array with modular index |
| **Pitfall** | Clear stale values, handle base cases |
| **Space** | Reduces one dimension by factor n/K |
| **Tradeoff** | Path reconstruction lost |

---

## ❓ Quick Revision Questions

1. **How many rows do you need to keep for a dependency on the last K rows?**
2. **What modular index do you use for accessing the current row?**
3. **Why must you clear/reinitialize the rolled-over row?**
4. **How does rolling array differ from two-row swap?**
5. **Give an example where K=2 (three rows needed).**
6. **Can you reconstruct the solution path with a rolling array?**

---

[← Previous: Space Optimization](01-space-optimization.md) | [Next: Matrix Exponentiation →](03-matrix-exponentiation.md)

[← Back to README](../README.md)
