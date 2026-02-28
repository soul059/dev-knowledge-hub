# Chapter 5: Aggressive Cows & Distance Problems

[← Previous: Splitting Problems](04-splitting-problems.md) | [Next: Practice Problems →](06-practice-problems.md)

---

## Overview

Distance problems ask: **"Place objects to maximize the minimum distance between them."** These are classic "maximize the minimum" BS-on-answer problems.

---

## Problem: Aggressive Cows (SPOJ)

> `n` stalls at positions `[x1, x2, ..., xn]`. Place `c` cows such that the **minimum distance** between any two cows is **maximized**.

```
   stalls = [1, 2, 4, 8, 9],  cows = 3
   
   Option A: place at [1, 4, 9]  → distances: 3, 5   → min = 3
   Option B: place at [1, 8, 9]  → distances: 7, 1   → min = 1
   Option C: place at [1, 4, 8]  → distances: 3, 4   → min = 3
   Option D: place at [2, 4, 9]  → distances: 2, 5   → min = 2
   
   Best: min distance = 3 (options A or C)
```

---

## Search Space

```
   lo = 1                               (minimum possible distance)
   hi = max(stalls) - min(stalls)        (place at extremes)
   
   For [1, 2, 4, 8, 9], c = 3:
   lo = 1, hi = 9 - 1 = 8
   
   minDist:  1   2   3   4   5   6   7   8
   canPlace: T   T   T   T   F   F   F   F
   3 cows?                 ↑
                     last True = 3 = answer
```

---

## Feasibility Function

```
   canPlace(minDist):
       1. Sort stalls (if not already sorted)
       2. Place first cow at stalls[0]
       3. For each subsequent stall:
          If distance from last placed cow ≥ minDist:
              Place cow here
       4. Return: placed cows ≥ c
```

```
   Trace: canPlace(3) with stalls [1, 2, 4, 8, 9], c = 3
   
   Cow 1 at stall 1 (always place first cow at first stall)
   Stall 2: dist = 2-1 = 1 < 3 → skip
   Stall 4: dist = 4-1 = 3 ≥ 3 → Cow 2 at stall 4 ✓
   Stall 8: dist = 8-4 = 4 ≥ 3 → Cow 3 at stall 8 ✓
   
   Placed 3 ≥ 3 → TRUE ✓
   
   Trace: canPlace(4) with same input
   
   Cow 1 at stall 1
   Stall 2: 1 < 4 → skip
   Stall 4: 3 < 4 → skip
   Stall 8: 7 ≥ 4 → Cow 2 at stall 8 ✓
   Stall 9: 1 < 4 → skip
   
   Placed 2 < 3 → FALSE ✗
```

---

## Solution (Java)

```java
public int aggressiveCows(int[] stalls, int c) {
    Arrays.sort(stalls);
    int lo = 1;
    int hi = stalls[stalls.length - 1] - stalls[0];
    
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;    // right-biased (lo = mid)
        if (canPlace(stalls, c, mid)) {
            lo = mid;           // mid works, try larger distance
        } else {
            hi = mid - 1;       // mid too large
        }
    }
    return lo;
}

boolean canPlace(int[] stalls, int c, int minDist) {
    int count = 1;
    int lastPos = stalls[0];
    for (int i = 1; i < stalls.length; i++) {
        if (stalls[i] - lastPos >= minDist) {
            count++;
            lastPos = stalls[i];
        }
    }
    return count >= c;
}
```

---

## Solution (Python)

```python
def aggressiveCows(stalls, c):
    stalls.sort()
    
    def canPlace(minDist):
        count = 1
        lastPos = stalls[0]
        for i in range(1, len(stalls)):
            if stalls[i] - lastPos >= minDist:
                count += 1
                lastPos = stalls[i]
        return count >= c
    
    lo, hi = 1, stalls[-1] - stalls[0]
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2
        if canPlace(mid):
            lo = mid
        else:
            hi = mid - 1
    return lo
```

---

## Solution (C++)

```cpp
int aggressiveCows(vector<int>& stalls, int c) {
    sort(stalls.begin(), stalls.end());
    int lo = 1, hi = stalls.back() - stalls[0];
    
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;
        if (canPlace(stalls, c, mid))
            lo = mid;
        else
            hi = mid - 1;
    }
    return lo;
}

bool canPlace(vector<int>& stalls, int c, int minDist) {
    int count = 1, lastPos = stalls[0];
    for (int i = 1; i < stalls.size(); i++) {
        if (stalls[i] - lastPos >= minDist) {
            count++;
            lastPos = stalls[i];
        }
    }
    return count >= c;
}
```

---

## Related: Magnetic Balls (LeetCode 1552)

> Place `m` balls in `n` positions to **maximize the minimum** magnetic force (distance) between any two balls.

This is essentially the **same problem** as Aggressive Cows with different wording.

```python
def maxDistance(position, m):
    position.sort()
    
    def canPlace(minDist):
        count, last = 1, position[0]
        for p in position[1:]:
            if p - last >= minDist:
                count += 1
                last = p
        return count >= m
    
    lo, hi = 1, position[-1] - position[0]
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2
        if canPlace(mid):
            lo = mid
        else:
            hi = mid - 1
    return lo
```

---

## Why Greedy Placement Works

```
   Why place greedily (first available position ≥ minDist)?
   
   Proof sketch:
   - If we skip a valid position and place later, the remaining
     positions only get fewer options (more constrained).
   - Placing as early as possible gives the most room for remaining
     cows/balls.
   - Therefore, greedy (place at first valid) maximizes the number
     of placements for a given minDist.
   
   If greedy can't place c cows → no placement can → canPlace = false
   If greedy CAN place c cows → answer is at least minDist → canPlace = true
```

---

## Template: Maximize the Minimum

```
   ╔══════════════════════════════════════════════════════════╗
   ║  MAXIMIZE THE MINIMUM PATTERN                           ║
   ║                                                          ║
   ║  1. Sort positions                                       ║
   ║  2. lo = 1 (or minimum possible)                        ║
   ║     hi = max_pos - min_pos                               ║
   ║  3. canPlace(minDist):                                   ║
   ║       Greedily place from left                           ║
   ║       Count placements                                   ║
   ║       Return count ≥ required                            ║
   ║  4. BS with lo = mid (right-biased, find last TRUE)     ║
   ║  5. Return lo                                            ║
   ╚══════════════════════════════════════════════════════════╝
```

---

## Complexity

| Component | Time |
|-----------|------|
| Sort | O(n log n) |
| Binary search | O(log(max - min)) |
| Feasibility | O(n) per check |
| **Total** | **O(n log n + n log(max - min))** |

---

## Quick Revision Questions

1. **Why must the stalls be sorted before applying the algorithm?**
2. **Why is this a "maximize the minimum" problem?**
3. **Why do we use right-biased mid for this problem?**
4. **Why does placing cows greedily give the optimal count?**
5. **What is the relationship between Aggressive Cows and LeetCode 1552?**
6. **Trace the algorithm on stalls = `[1, 3, 5, 7]`, cows = 2.**

---

[← Previous: Splitting Problems](04-splitting-problems.md) | [Next: Practice Problems →](06-practice-problems.md)
