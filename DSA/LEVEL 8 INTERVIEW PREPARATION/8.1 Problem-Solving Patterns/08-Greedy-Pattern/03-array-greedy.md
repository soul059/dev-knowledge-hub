# Chapter 3: Array Greedy Problems

## 📋 Chapter Overview
Greedy strategies on arrays: jump games, gas station, candy distribution, and task scheduling.

---

## 📝 Problem 1: Jump Game I (LeetCode 55)

**Problem:** Array of non-negative integers where each element is max jump length. Can you reach the last index?

```
function canJump(nums):
    maxReach = 0
    for i = 0 to n-1:
        if i > maxReach: return false  // can't reach this index
        maxReach = max(maxReach, i + nums[i])
        if maxReach >= n-1: return true
    return true
```

### Trace

```
  nums = [2, 3, 1, 1, 4]
  
  i=0: maxReach = max(0, 0+2) = 2
  i=1: maxReach = max(2, 1+3) = 4 ≥ 4 → return true
  
  nums = [3, 2, 1, 0, 4]
  
  i=0: maxReach = max(0, 0+3) = 3
  i=1: maxReach = max(3, 1+2) = 3
  i=2: maxReach = max(3, 2+1) = 3
  i=3: maxReach = max(3, 3+0) = 3
  i=4: 4 > 3 → return false (can't reach index 4)
```

---

## 📝 Problem 2: Jump Game II (LeetCode 45)

**Problem:** Minimum number of jumps to reach the last index (guaranteed reachable).

```
function minJumps(nums):
    jumps = 0
    currentEnd = 0     // farthest we can go with current jumps
    farthest = 0       // farthest we can go with one more jump
    
    for i = 0 to n-2:  // don't need to jump FROM last index
        farthest = max(farthest, i + nums[i])
        if i == currentEnd:     // must jump now
            jumps++
            currentEnd = farthest
    
    return jumps
```

### Trace

```
  nums = [2, 3, 1, 1, 4]
  
  i=0: farthest = max(0,2) = 2.  i==currentEnd(0) → jumps=1, currentEnd=2
  i=1: farthest = max(2,4) = 4.  i≠2 → skip
  i=2: farthest = max(4,3) = 4.  i==currentEnd(2) → jumps=2, currentEnd=4
  i=3: we stop (i=3 < n-2=3 is false)
  
  Answer: 2  (jump 0→1→4)
```

**Complexity:** O(n) time, O(1) space

---

## 📝 Problem 3: Gas Station (LeetCode 134)

**Problem:** `n` gas stations in a circle. `gas[i]` fuel gained, `cost[i]` fuel to next station. Find starting station to complete the circuit, or -1.

```
function canCompleteCircuit(gas, cost):
    totalGas = 0, totalCost = 0
    tank = 0, start = 0
    
    for i = 0 to n-1:
        totalGas += gas[i]
        totalCost += cost[i]
        tank += gas[i] - cost[i]
        if tank < 0:
            start = i + 1   // can't start at or before i
            tank = 0
    
    if totalGas < totalCost: return -1
    return start
```

### Trace

```
  gas  = [1, 2, 3, 4, 5]
  cost = [3, 4, 5, 1, 2]
  net  = [-2,-2,-2, 3, 3]  (gas - cost)
  
  i=0: tank = -2 < 0 → start=1, tank=0
  i=1: tank = -2 < 0 → start=2, tank=0
  i=2: tank = -2 < 0 → start=3, tank=0
  i=3: tank = 3 ≥ 0 → continue
  i=4: tank = 3+3 = 6 ≥ 0 → continue
  
  totalGas=15, totalCost=15 → feasible
  Answer: start = 3
```

**Key insight:** If total gas ≥ total cost, a solution exists. The greedy resets the starting point whenever the tank goes negative.

---

## 📝 Problem 4: Candy (LeetCode 135)

**Problem:** Each child has a rating. Each child gets at least 1 candy. Higher-rated child gets more candy than neighbors.

```
function candy(ratings):
    n = len(ratings)
    candies = array of 1s, size n
    
    // Left to right: if rating higher than left neighbor
    for i = 1 to n-1:
        if ratings[i] > ratings[i-1]:
            candies[i] = candies[i-1] + 1
    
    // Right to left: if rating higher than right neighbor
    for i = n-2 down to 0:
        if ratings[i] > ratings[i+1]:
            candies[i] = max(candies[i], candies[i+1] + 1)
    
    return sum(candies)
```

### Trace

```
  ratings = [1, 0, 2]
  
  Init:     [1, 1, 1]
  L→R pass: i=1: 0 > 1? no
            i=2: 2 > 0? yes → candies[2] = 1+1 = 2
            → [1, 1, 2]
  R→L pass: i=1: 0 > 2? no
            i=0: 1 > 0? yes → candies[0] = max(1, 1+1) = 2
            → [2, 1, 2]
  Total: 5
```

**Complexity:** O(n) time, O(n) space

---

## 📝 Problem 5: Task Scheduler (LeetCode 621)

**Problem:** Tasks with cooldown `n`. Same task must have at least `n` intervals between executions. Find minimum intervals.

```
function leastInterval(tasks, n):
    count = frequency map of tasks
    maxFreq = max value in count
    maxCount = number of tasks with frequency == maxFreq
    
    // Frame: (maxFreq - 1) groups of (n + 1) slots + last group
    result = (maxFreq - 1) * (n + 1) + maxCount
    
    // If tasks fill more than the frame
    return max(result, len(tasks))
```

### Trace

```
  tasks = [A,A,A,B,B,B], n = 2
  
  count: A=3, B=3
  maxFreq = 3, maxCount = 2 (both A and B)
  
  Frame: (3-1)*(2+1) + 2 = 2*3 + 2 = 8
  
  Layout:
  A B _ | A B _ | A B
  Slots: 8 ≥ len(tasks)=6 → Answer: 8
  
  With n=0: result = (3-1)*1 + 2 = 4, max(4, 6) = 6 → no idle needed
```

---

## 📊 Array Greedy Summary

| Problem | Greedy Strategy | Time | Space |
|---------|----------------|------|-------|
| Jump Game I | Track max reachable index | O(n) | O(1) |
| Jump Game II | BFS-like levels with farthest reach | O(n) | O(1) |
| Gas Station | Reset start when tank < 0 | O(n) | O(1) |
| Candy | Two passes (L→R, R→L) | O(n) | O(n) |
| Task Scheduler | Frame calculation from max frequency | O(n) | O(1) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Reachability | Track farthest reachable position greedily |
| Circular problems | Total feasibility check + greedy reset |
| Two-pass greedy | Handle left/right constraints separately |
| Frame technique | Calculate idle slots from most frequent element |

---

## ❓ Revision Questions

1. **Jump Game I: greedy idea?** → Track max reachable index; if current index exceeds it, return false.
2. **Jump Game II: why BFS-like?** → Each "level" is a jump; track the farthest position reachable within current jump's range.
3. **Gas Station: why reset works?** → If tank goes negative at i, starting from any station 0..i can't work; try i+1.
4. **Candy: why two passes?** → First pass handles left neighbor constraint; second handles right neighbor. `max` merges both.
5. **Task Scheduler: when is answer just len(tasks)?** → When there are enough distinct tasks to fill all cooldown gaps without idle time.

---

[← Previous: Interval Problems](02-interval-problems.md) | [Next: Template and Variations →](04-template-and-variations.md)
