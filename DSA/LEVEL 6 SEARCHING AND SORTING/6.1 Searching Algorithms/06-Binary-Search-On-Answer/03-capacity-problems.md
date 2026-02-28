# Chapter 3: Capacity & Shipping Problems

[← Previous: Square Root Problems](02-square-root.md) | [Next: Splitting Problems →](04-splitting-problems.md)

---

## Overview

Capacity problems ask: **"What is the minimum capacity/speed needed to complete a task within a given time limit?"** These are classic BS-on-answer problems.

---

## Problem 1: Capacity to Ship Packages (LeetCode 1011)

> Packages with weights `[w1, w2, ..., wn]` must be shipped in order within `D` days. Find the minimum ship capacity.

```
   weights = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], D = 5
   
   Minimum capacity = 15:
   Day 1: [1, 2, 3, 4, 5]  sum = 15
   Day 2: [6, 7]            sum = 13
   Day 3: [8]               sum = 8
   Day 4: [9]               sum = 9
   Day 5: [10]              sum = 10
```

### Search Space

```
   lo = max(weights)     // Must carry the heaviest single package
   hi = sum(weights)     // Carry everything in one day
   
   For example: lo = 10, hi = 55
   
   Capacity:  10  11  12  13  14  15  16  ...  55
   Days need: 10   9   8   7   6   5   5  ...   1
   ≤ D days?   F   F   F   F   F   T   T  ...   T
                                   ↑
                             minimum = 15
```

### Feasibility Function

```python
def canShip(weights, capacity, D):
    days = 1
    current_load = 0
    for w in weights:
        if current_load + w > capacity:
            days += 1
            current_load = 0
        current_load += w
    return days <= D
```

### Full Solution (Java)

```java
public int shipWithinDays(int[] weights, int D) {
    int lo = 0, hi = 0;
    for (int w : weights) {
        lo = Math.max(lo, w);
        hi += w;
    }
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canShip(weights, mid, D)) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}

boolean canShip(int[] weights, int capacity, int D) {
    int days = 1, load = 0;
    for (int w : weights) {
        if (load + w > capacity) {
            days++;
            load = 0;
        }
        load += w;
    }
    return days <= D;
}
```

### Trace

```
   weights = [1,2,3,4,5,6,7,8,9,10], D = 5
   lo = 10, hi = 55
   
   mid=32: canShip → 2 days ≤ 5 → hi=32
   mid=21: canShip → 3 days ≤ 5 → hi=21
   mid=15: canShip → 5 days ≤ 5 → hi=15
   mid=12: canShip → 7 days > 5 → lo=13
   mid=14: canShip → 6 days > 5 → lo=15
   lo=15 hi=15 → return 15 ✓
```

---

## Problem 2: Koko Eating Bananas (LeetCode 875)

> `n` piles, `h` hours. Find minimum eating speed `k`.

```
   piles = [30, 11, 23, 4, 20], h = 5
   
   lo = 1, hi = max(piles) = 30
```

### Solution (Python)

```python
def minEatingSpeed(piles, h):
    def canFinish(speed):
        return sum((p + speed - 1) // speed for p in piles) <= h
    
    lo, hi = 1, max(piles)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if canFinish(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Trace

```
   piles = [30, 11, 23, 4, 20], h = 6
   lo=1, hi=30
   
   mid=15: ceil(30/15)+ceil(11/15)+ceil(23/15)+ceil(4/15)+ceil(20/15)
         = 2 + 1 + 2 + 1 + 2 = 8 > 6 → lo=16
   
   mid=23: 2+1+1+1+1 = 6 ≤ 6 → hi=23
   mid=19: 2+1+2+1+2 = 8 > 6 → lo=20
   mid=21: 2+1+2+1+1 = 7 > 6 → lo=22
   mid=22: 2+1+2+1+1 = 7 > 6 → lo=23
   lo=23 hi=23 → return 23 ✓
```

---

## Problem 3: Minimum Number of Days to Make Bouquets (LeetCode 1482)

> `bloomDay[i]` = day flower `i` blooms. Need `m` bouquets of `k` adjacent flowers. Find minimum days.

```
   bloomDay = [1, 10, 3, 10, 2], m = 3, k = 1
   
   Day 1: flowers [✓, _, _, _, _] → 1 bouquet
   Day 2: flowers [✓, _, _, _, ✓] → 2 bouquets
   Day 3: flowers [✓, _, ✓, _, ✓] → 3 bouquets ✓
   
   Answer: 3
```

### Solution (C++)

```cpp
int minDays(vector<int>& bloomDay, int m, int k) {
    long long total = (long long)m * k;
    if (total > bloomDay.size()) return -1;
    
    int lo = *min_element(bloomDay.begin(), bloomDay.end());
    int hi = *max_element(bloomDay.begin(), bloomDay.end());
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canMake(bloomDay, m, k, mid))
            hi = mid;
        else
            lo = mid + 1;
    }
    return lo;
}

bool canMake(vector<int>& bloomDay, int m, int k, int day) {
    int bouquets = 0, flowers = 0;
    for (int bd : bloomDay) {
        if (bd <= day) {
            flowers++;
            if (flowers == k) {
                bouquets++;
                flowers = 0;
            }
        } else {
            flowers = 0;   // Reset — must be adjacent
        }
    }
    return bouquets >= m;
}
```

---

## Common Structure Across Capacity Problems

```
   ╔═══════════════════════════════════════════════════════════╗
   ║                                                           ║
   ║   1. IDENTIFY what you're minimizing                     ║
   ║      (capacity, speed, days, etc.)                       ║
   ║                                                           ║
   ║   2. SET lo = minimum possible (often max single item)   ║
   ║      SET hi = maximum possible (often sum of all)        ║
   ║                                                           ║
   ║   3. WRITE canAchieve(mid):                              ║
   ║      Greedily simulate with candidate answer = mid       ║
   ║      Count resources used (days, groups, etc.)           ║
   ║      Return count ≤ limit                                ║
   ║                                                           ║
   ║   4. BINARY SEARCH with hi = mid template                ║
   ║      (finding minimum feasible)                          ║
   ║                                                           ║
   ╚═══════════════════════════════════════════════════════════╝
```

---

## Side-by-Side Comparison

| Aspect | Ship Packages | Koko Bananas | Bouquets |
|--------|--------------|--------------|----------|
| **Answer** | capacity | speed | day |
| **lo** | max(weight) | 1 | min(bloom) |
| **hi** | sum(weights) | max(pile) | max(bloom) |
| **Check** | days ≤ D | hours ≤ h | bouquets ≥ m |
| **Greedy** | Ship until full | Eat ceil(p/k) | Count adjacent |
| **Total** | O(n log S) | O(n log M) | O(n log M) |

---

## Quick Revision Questions

1. **Why is `lo = max(weights)` for ship packages?**
2. **Why is `hi = sum(weights)` the upper bound?**
3. **What does the feasibility function check in each problem?**
4. **Why are these "minimize" problems (not "maximize")?**
5. **What's the difference between Koko and Ship in terms of adjacency?**
6. **How would you handle the case where it's impossible to make bouquets?**

---

[← Previous: Square Root Problems](02-square-root.md) | [Next: Splitting Problems →](04-splitting-problems.md)
