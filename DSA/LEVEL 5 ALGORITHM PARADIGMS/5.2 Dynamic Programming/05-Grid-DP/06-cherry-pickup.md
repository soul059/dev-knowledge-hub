# Chapter 6: Cherry Pickup

## 📋 Overview
**Cherry Pickup** (LeetCode #741) is an advanced grid DP problem where you travel from (0,0) to (n-1,n-1) and back, collecting cherries. The key insight is that the round trip is equivalent to **two people walking simultaneously** from start to end, which requires a **multi-dimensional DP state**.

---

## 🧠 Problem Statement

```
n×n grid:
  0 = empty, 1 = cherry, -1 = thorn (blocked)

Go from (0,0) to (n-1,n-1) collecting cherries (set to 0 once picked).
Then go from (n-1,n-1) back to (0,0) collecting more.
Maximize total cherries.

Why greedy (find best path, then find next best) fails:
  ┌───┬───┬───┐
  │ 1 │ 1 │ 1 │   Greedy path 1: top row → right col = 4 cherries
  ├───┼───┼───┤   Greedy path 2: nothing left on return!
  │ 0 │ 0 │ 1 │   
  ├───┼───┼───┤   But optimal: path1 = top row (3), path2 = middle→right (1)
  │ 0 │ 0 │ 1 │   Actually we need to think simultaneously.
  └───┴───┴───┘
```

---

## 🔍 Key Insight: Two Simultaneous Walkers

```
Instead of one person going and returning,
model as TWO PEOPLE walking from (0,0) to (n-1,n-1) simultaneously.

Person 1: at position (r1, c1)
Person 2: at position (r2, c2)

Both move right or down at each step.
If both are on the same cherry cell, count it only once.

Since both take the same number of steps t:
  r1 + c1 = t = r2 + c2
  So: c1 = t - r1, c2 = t - r2

We can reduce 4D state to 3D:
  dp[t][r1][r2] = max cherries when at step t,
                   person 1 at row r1, person 2 at row r2
                   (columns derived: c1 = t-r1, c2 = t-r2)
```

---

## 🔨 Solution

```
function cherryPickup(grid):
    n = len(grid)
    // dp[r1][r2] = max cherries at step t with persons at rows r1, r2
    // Initialize with -infinity (invalid state)
    dp = 2D array of size n × n, filled with -infinity
    dp[0][0] = grid[0][0]
    
    // Total steps from (0,0) to (n-1,n-1) = 2*(n-1)
    for t = 1 to 2*(n-1):
        // New dp for this step
        newDp = 2D array of size n × n, filled with -infinity
        
        for r1 = max(0, t-(n-1)) to min(n-1, t):
            for r2 = max(0, t-(n-1)) to min(n-1, t):
                c1 = t - r1
                c2 = t - r2
                
                // Bounds check
                if c1 < 0 or c1 >= n or c2 < 0 or c2 >= n: continue
                
                // Thorn check
                if grid[r1][c1] == -1 or grid[r2][c2] == -1: continue
                
                // Cherries collected at this step
                cherries = grid[r1][c1]
                if r1 != r2:  // different cells
                    cherries += grid[r2][c2]
                // else: same cell, count once
                
                // Try all 4 previous states (each person came from up or left)
                for pr1 in [r1, r1-1]:      // person 1: same row (came from left) or row above
                    for pr2 in [r2, r2-1]:  // person 2: same row or row above
                        if pr1 >= 0 and pr2 >= 0 and dp[pr1][pr2] > -infinity:
                            newDp[r1][r2] = max(newDp[r1][r2], dp[pr1][pr2] + cherries)
        
        dp = newDp
    
    return max(0, dp[n-1][n-1])
```

---

## 🔬 Trace: Small Example

```
Grid:
  0  1  1
  1  0  1
  1  1  1

n=3, total steps = 4

t=0: dp[0][0] = grid[0][0] = 0

t=1: Person can be at (0,1) or (1,0)
  (r1=0,r2=0): c1=1,c2=1 → grid[0][1]=1, same cell → cherry=1
    from dp[0][0]=0 → newDp[0][0] = 0+1 = 1
  (r1=0,r2=1): c1=1,c2=0 → grid[0][1]+grid[1][0] = 1+1 = 2
    from dp[0][0]=0 → newDp[0][1] = 0+2 = 2
  (r1=1,r2=0): symmetric → newDp[1][0] = 2
  (r1=1,r2=1): c1=0,c2=0 → grid[1][0]=1, same cell → cherry=1
    from dp[0][0]=0 → newDp[1][1] = 0+1 = 1

...continuing through t=2,3,4...

Final: dp[2][2] = 5 (collect 5 out of 6 cherries)
```

---

## 🌐 Cherry Pickup II (LeetCode #1463)

```
Two robots at top of grid, start at (0,0) and (0,n-1).
Both move DOWN one row per step.
Each can go left, stay, or right (3 choices).
Maximize total cherries collected.

State: dp[row][c1][c2]
  row = current row (both robots on same row)
  c1  = column of robot 1
  c2  = column of robot 2

Recurrence:
  dp[row][c1][c2] = cherries(row,c1,c2) +
    max over all (dc1, dc2) in {-1,0,1}×{-1,0,1}:
      dp[row+1][c1+dc1][c2+dc2]

Cherries:
  if c1 == c2: grid[row][c1]          // same cell, count once
  else:        grid[row][c1] + grid[row][c2]

Time: O(m × n²× 9), Space: O(n²)
```

---

## 💡 Multi-Agent Grid DP Pattern

```
┌────────────────────────────────────────────────────┐
│ When a problem involves:                           │
│ • Multiple paths through same grid                 │
│ • Collecting items (counted once)                  │
│ • Maximizing total collected across paths          │
│                                                    │
│ Solution pattern:                                  │
│ 1. Model as simultaneous walkers                   │
│ 2. Synchronize by step count (t = r + c)           │
│ 3. State: (step, positions of all walkers)         │
│ 4. Handle overlapping cells (count once)           │
│ 5. Reduce dimensions using step constraint         │
└────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Key Insight** | Round trip = two simultaneous forward paths |
| **State** | dp[t][r1][r2], columns derived from t |
| **Dimension Reduction** | 4D→3D using c = t - r |
| **Overlap Handling** | If r1==r2 (same cell), count cherry once |
| **Complexity** | O(n³) time, O(n²) space |
| **Cherry Pickup II** | Two top-down robots, dp[row][c1][c2] |

---

## ❓ Quick Revision Questions

1. **Why is the round-trip modeled as two simultaneous forward paths?**
2. **How do you reduce the state from 4D to 3D?**
3. **How do you handle both persons landing on the same cherry?**
4. **Why does a greedy approach (best path first) fail?**
5. **What are the 4 possible transitions for two walkers at each step?**
6. **How does Cherry Pickup II differ from Cherry Pickup I?**

---

[← Previous: Dungeon Game](05-dungeon-game.md) | [Next Unit: String DP →](../06-String-DP/01-longest-common-subsequence.md)

[← Back to README](../README.md)
