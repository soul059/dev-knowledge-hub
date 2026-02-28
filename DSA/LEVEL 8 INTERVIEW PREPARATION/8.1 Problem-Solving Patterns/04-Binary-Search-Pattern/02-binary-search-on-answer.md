# Chapter 2: Binary Search on Answer

## 📋 Chapter Overview
Instead of searching **for a value in an array**, we binary search **the answer space** — a range of possible answers — and check feasibility with a helper function. This is one of the most powerful and frequently tested patterns.

---

## 🧠 Core Concept

### Traditional vs Answer-Space Binary Search

```
  Traditional:
  ┌─────────────────────────┐
  │ Search IN the array     │  "Is arr[mid] == target?"
  └─────────────────────────┘

  Answer-Space:
  ┌─────────────────────────┐
  │ Search OVER a range     │  "Is answer = mid feasible?"
  │ of possible answers     │
  └─────────────────────────┘
```

### The Key Insight

If the answer space has a **monotonic property** — all values ≤ some threshold are feasible and all values above are not (or vice versa) — then binary search applies.

```
  Answer space: [lo ................ hi]

  Feasibility:  ✓ ✓ ✓ ✓ ✓ ✗ ✗ ✗ ✗ ✗
                          ↑
                    ANSWER (boundary)
```

---

## 📝 Template

```
function binarySearchOnAnswer(lo, hi):
    while lo < hi:                   // ← note: strict <
        mid = lo + (hi - lo) / 2
        
        if isFeasible(mid):
            hi = mid                 // mid might be answer
        else:
            lo = mid + 1             // mid too small
    
    return lo                        // lo == hi == answer

function isFeasible(candidate):
    // Problem-specific: can we achieve the goal
    // with candidate as the answer?
    return true/false
```

### Template Variants

| Goal | Condition | Update |
|------|-----------|--------|
| Minimize answer | isFeasible(mid) → hi = mid | lo = mid + 1 |
| Maximize answer | isFeasible(mid) → lo = mid | hi = mid - 1 (or mid) |

---

## 🔍 Classic Problem 1: Koko Eating Bananas

**Problem:** Koko has `n` piles of bananas. She eats at speed `k` (bananas/hour). Given `h` hours total, find the minimum `k` so she finishes all piles.

### Visualization

```
  Piles: [3, 6, 7, 11]   h = 8 hours

  Speed k=1: 3+6+7+11 = 27 hours  ✗ (too slow)
  Speed k=4: 1+2+2+3  = 8 hours   ✓ (just fits!)
  Speed k=11: 1+1+1+1  = 4 hours  ✓ (works but not minimum)

  Answer space: [1, max(piles)] = [1, 11]
  
  ✗ ✗ ✗ ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓
  1 2 3 4 5 6 7 8 9 10 11
        ↑
    minimum k = 4
```

### Solution

```
function minEatingSpeed(piles, h):
    lo = 1
    hi = max(piles)

    while lo < hi:
        mid = lo + (hi - lo) / 2
        if canFinish(piles, mid, h):
            hi = mid
        else:
            lo = mid + 1

    return lo

function canFinish(piles, speed, h):
    hours = 0
    for pile in piles:
        hours += ceil(pile / speed)
    return hours <= h
```

### Trace: piles = [3, 6, 7, 11], h = 8

| Step | lo | hi | mid | hours needed | Feasible? | Action |
|------|----|----|-----|-------------|-----------|--------|
| 1 | 1 | 11 | 6 | 1+1+2+2=6 | ✓ | hi = 6 |
| 2 | 1 | 6 | 3 | 1+2+3+4=10 | ✗ | lo = 4 |
| 3 | 4 | 6 | 5 | 1+2+2+3=8 | ✓ | hi = 5 |
| 4 | 4 | 5 | 4 | 1+2+2+3=8 | ✓ | hi = 4 |
| 5 | 4 | 4 | - | - | - | **Return 4** |

---

## 🔍 Classic Problem 2: Capacity to Ship Packages

**Problem:** Conveyor belt with packages of weights `w[]`. Ship must carry packages in order. Find minimum capacity such that all packages ship within `days` days.

### Visualization

```
  Weights: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]    days = 5

  Answer space: [max(w), sum(w)] = [10, 55]
  
  Capacity 10: [1,2,3,4] [5] [6] [7] [8] [9] [10] → 7 days ✗
  Capacity 15: [1,2,3,4,5] [6,7] [8] [9] [10]     → 5 days ✓
  
  Find minimum capacity → binary search!
```

### Solution

```
function shipWithinDays(weights, days):
    lo = max(weights)        // must fit largest single package
    hi = sum(weights)        // ship everything in 1 day

    while lo < hi:
        mid = lo + (hi - lo) / 2
        if canShip(weights, mid, days):
            hi = mid
        else:
            lo = mid + 1
    return lo

function canShip(weights, capacity, days):
    daysNeeded = 1
    currentLoad = 0
    for w in weights:
        if currentLoad + w > capacity:
            daysNeeded += 1
            currentLoad = 0
        currentLoad += w
    return daysNeeded <= days
```

### Trace: weights = [1,2,3,4,5,6,7,8,9,10], days = 5

| Step | lo | hi | mid | Days needed | Feasible? | Action |
|------|----|----|-----|------------|-----------|--------|
| 1 | 10 | 55 | 32 | 2 | ✓ | hi = 32 |
| 2 | 10 | 32 | 21 | 3 | ✓ | hi = 21 |
| 3 | 10 | 21 | 15 | 5 | ✓ | hi = 15 |
| 4 | 10 | 15 | 12 | 6 | ✗ | lo = 13 |
| 5 | 13 | 15 | 14 | 5 | ✓ | hi = 14 |
| 6 | 13 | 14 | 13 | 6 | ✗ | lo = 14 |
| 7 | 14 | 14 | - | - | - | **Return 15** |

Wait — let me re-check step 3. With capacity 15:
- Day1: 1+2+3+4+5=15, Day2: 6+7=13, Day3: 8, Day4: 9, Day5: 10 → 5 days ✓

Actually re-tracing step 7: lo=14, hi=14 → return 14.
With capacity 14: Day1: 1+2+3+4=10, Day2: 5+6=11, Day3: 7, Day4: 8, Day5: 9, Day6: 10 → 6 days. Not feasible.

Let me re-trace properly:

| Step | lo | hi | mid | Feasible? | Action |
|------|----|----|-----|-----------|--------|
| 1 | 10 | 55 | 32 | ✓ | hi = 32 |
| 2 | 10 | 32 | 21 | ✓ | hi = 21 |
| 3 | 10 | 21 | 15 | ✓ | hi = 15 |
| 4 | 10 | 15 | 12 | ✗ | lo = 13 |
| 5 | 13 | 15 | 14 | ✗ | lo = 15 |
| 6 | 15 | 15 | - | - | **Return 15** |

Answer: **15** ✓

---

## 🔍 Classic Problem 3: Split Array Largest Sum

**Problem:** Split array into `m` subarrays to minimize the largest subarray sum.

```
  [7, 2, 5, 10, 8]  m = 2

  Answer space: [max(arr), sum(arr)] = [10, 32]
  
  Split: [7,2,5] [10,8] → max(14, 18) = 18 ✓
  Split: [7,2,5,10] [8] → max(24, 8) = 24 ✗ (worse)
  
  The feasibility check: can we split into ≤ m groups
  where no group sum exceeds 'mid'?
```

### Solution

```
function splitArray(nums, m):
    lo = max(nums)
    hi = sum(nums)
    
    while lo < hi:
        mid = lo + (hi - lo) / 2
        if canSplit(nums, mid, m):
            hi = mid
        else:
            lo = mid + 1
    return lo

function canSplit(nums, maxSum, m):
    groups = 1
    currentSum = 0
    for num in nums:
        if currentSum + num > maxSum:
            groups += 1
            currentSum = 0
        currentSum += num
    return groups <= m
```

---

## 🎯 Pattern Recognition Framework

### How to Identify "Binary Search on Answer"

```
  ┌──────────────────────────────────────┐
  │ 1. Problem asks: "minimum maximum"   │
  │    or "maximum minimum"              │
  │                                      │
  │ 2. Answer lies in a bounded range    │
  │    [lo, hi] that you can define      │
  │                                      │
  │ 3. A feasibility function exists     │
  │    that is monotonic over the range  │
  │                                      │
  │ → Binary Search on Answer!           │
  └──────────────────────────────────────┘
```

### Bounds Selection Guide

| Problem Type | lo | hi |
|-------------|----|----|
| Eating speed | 1 | max(piles) |
| Ship capacity | max(weights) | sum(weights) |
| Split array | max(arr) | sum(arr) |
| Allocate pages | max(pages) | sum(pages) |
| Min distance | 0 | max_position - min_position |

---

## 📊 Complexity

| Aspect | Value |
|--------|-------|
| Binary search iterations | O(log(hi - lo)) |
| Feasibility check per iteration | O(n) typically |
| Total time | O(n × log(hi - lo)) |
| Space | O(1) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Core idea | Search answer space, not array |
| Prerequisite | Monotonic feasibility function |
| Template | `while lo < hi`, check `isFeasible(mid)` |
| Minimize | hi = mid when feasible |
| Maximize | lo = mid when feasible |
| Common bounds | [max(arr), sum(arr)] |
| Signal phrase | "Minimize the maximum" or "Maximum the minimum" |

---

## ❓ Revision Questions

1. **What is the key difference from standard binary search?**
   → We search over a range of possible answers, not indices in an array.

2. **What makes this approach valid?**
   → The feasibility function is monotonic: once feasible at some value, it stays feasible for all larger (or smaller) values.

3. **Why `while lo < hi` instead of `while lo <= hi`?**
   → We converge lo and hi to the same point; when lo == hi, that's our answer. No exact-match check needed.

4. **How do you determine lo and hi bounds?**
   → lo = smallest possible answer (often max element), hi = largest possible answer (often total sum).

5. **Time complexity of Binary Search on Answer?**
   → O(n × log(range)) where n is the cost of the feasibility check and range = hi - lo.

---

[← Previous: Standard Binary Search](01-standard-binary-search.md) | [Next: Finding Boundaries →](03-finding-boundaries.md)
