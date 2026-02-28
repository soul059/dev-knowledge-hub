# Chapter 1: Binary Search on Answer — Concept

[← Previous Unit: Rotated Array Practice](../05-Search-Rotated-Array/05-practice-problems.md) | [Next: Square Root Problems →](02-square-root.md)

---

## Overview

**Binary Search on Answer** (also called **Binary Search on Result** or **Parametric Search**) is a powerful technique where instead of searching for a target in an array, you search for the **optimal value of the answer** over a continuous or discrete range.

---

## Core Idea

```
   Traditional BS:       Search for target IN an array
   BS on Answer:         Search for the best ANSWER in a range [lo, hi]
   
   Key difference:
   - Traditional: arr[mid] == target?
   - On Answer:   canAchieve(mid)?   ← use a predicate/feasibility function
```

---

## The Pattern

```
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                 │
   │  1. Define the answer space: [lo, hi]                          │
   │     lo = minimum possible answer                                │
   │     hi = maximum possible answer                                │
   │                                                                 │
   │  2. Write a feasibility function: canAchieve(mid)              │
   │     Returns TRUE if "mid" is a feasible answer                 │
   │                                                                 │
   │  3. Binary search on [lo, hi] to find the optimal answer       │
   │     - If looking for MINIMUM feasible: use Template 3 (hi=mid) │
   │     - If looking for MAXIMUM feasible: use Template 2 (lo=mid) │
   │                                                                 │
   └─────────────────────────────────────────────────────────────────┘
```

---

## The Monotonicity Requirement

```
   The feasibility function must be MONOTONIC:
   
   For "find minimum where feasible":
   Answer:    1   2   3   4   5   6   7   8   9   10
   Feasible:  F   F   F   F   T   T   T   T   T   T
                              ↑
                        First True = optimal answer
   
   For "find maximum where feasible":
   Answer:    1   2   3   4   5   6   7   8   9   10
   Feasible:  T   T   T   T   T   T   F   F   F   F
                                  ↑
                           Last True = optimal answer
   
   Once the predicate switches, it STAYS switched.
   This is what makes binary search work!
```

---

## General Template: Find Minimum Feasible

```python
def solve():
    lo = MIN_POSSIBLE_ANSWER
    hi = MAX_POSSIBLE_ANSWER
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if canAchieve(mid):    # Is mid feasible?
            hi = mid           # mid works, try smaller
        else:
            lo = mid + 1       # mid doesn't work, need bigger
    
    return lo                  # smallest feasible answer
```

## General Template: Find Maximum Feasible

```python
def solve():
    lo = MIN_POSSIBLE_ANSWER
    hi = MAX_POSSIBLE_ANSWER
    
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2   # right-biased!
        if canAchieve(mid):    # Is mid feasible?
            lo = mid           # mid works, try bigger
        else:
            hi = mid - 1       # mid doesn't work, need smaller
    
    return lo                  # largest feasible answer
```

---

## How to Identify BS-on-Answer Problems

```
   Look for these keywords in the problem:
   
   ┌──────────────────────────────────────────────────┐
   │  "Find the minimum/maximum value such that..."   │
   │  "What is the least/most ... that satisfies..."  │
   │  "Minimize the maximum ..."                      │
   │  "Maximize the minimum ..."                      │
   │  "Can you achieve X with Y resources?"           │
   │  "Find the smallest capacity/speed/time to..."   │
   └──────────────────────────────────────────────────┘
   
   The telltale sign: a GREEDY CHECK function that 
   verifies if a given answer is feasible in O(n) or O(n log n).
```

---

## Example: Minimum Speed to Eat All Bananas

**Problem (Koko Eating Bananas — LeetCode 875):**  
Koko has `n` piles of bananas. She can eat `k` bananas per hour. She has `h` hours. Find the minimum `k`.

```
   piles = [3, 6, 7, 11], h = 8
   
   If k = 4: ceil(3/4) + ceil(6/4) + ceil(7/4) + ceil(11/4)
           = 1 + 2 + 2 + 3 = 8 hours → exactly fits! ✓
   
   If k = 3: 1 + 2 + 3 + 4 = 10 hours → too slow ✗
   
   Answer: k = 4 (minimum speed that works)
```

### The Search Space

```
   k can range from 1 to max(piles):
   
   k:        1    2    3    4    5    6    ...   11
   hours:   27   14   10    8    6    5    ...    4
   feasible: F    F    F    T    T    T    ...    T
                            ↑
                      minimum k = 4
```

### Solution

```python
def minEatingSpeed(piles, h):
    def canFinish(speed):
        hours = 0
        for pile in piles:
            hours += (pile + speed - 1) // speed   # ceil division
        return hours <= h
    
    lo, hi = 1, max(piles)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if canFinish(mid):
            hi = mid        # can do it, try slower
        else:
            lo = mid + 1    # too slow, must go faster
    return lo
```

---

## Framework for Solving

```
   Step 1: Identify the answer variable
           What quantity are we optimizing?
           → In Koko: the eating speed k
   
   Step 2: Define the search range [lo, hi]
           What's the min/max possible answer?
           → lo = 1, hi = max(piles)
   
   Step 3: Write the feasibility function
           Given a candidate answer, can we achieve the goal?
           → canFinish(speed): returns True if hours ≤ h
   
   Step 4: Apply binary search
           Is it monotonically increasing or decreasing?
           → Larger speed → fewer hours → more feasible
           → Find MINIMUM feasible → hi = mid template
   
   Step 5: Return lo
```

---

## Classic BS-on-Answer Problems

| Problem | Answer Variable | lo | hi | Check |
|---------|----------------|----|----|-------|
| Koko Bananas | speed | 1 | max(pile) | hours ≤ h |
| Ship Packages | capacity | max(weight) | sum(weights) | days ≤ d |
| Split Array Largest Sum | max sum | max(element) | sum(array) | splits ≤ m |
| Aggressive Cows | min distance | 1 | max_pos - min_pos | can place all cows |
| Painter's Partition | max time | max(board) | sum(boards) | painters ≤ k |
| Allocate Pages | max pages | max(pages) | sum(pages) | students ≤ m |
| Magnetic Balls | min distance | 1 | range / (m-1) | can place all |

---

## Complexity

| Component | Time |
|-----------|------|
| Binary search | O(log(hi - lo)) |
| Feasibility check | O(n) typically |
| **Total** | **O(n × log(hi - lo))** |

---

## Quick Revision Questions

1. **What is the key difference between BS-on-array and BS-on-answer?**
2. **What property must the feasibility function have?**
3. **How do you choose between hi=mid and lo=mid templates?**
4. **For Koko Bananas, why is lo=1 and hi=max(piles)?**
5. **What keywords in a problem hint at BS-on-answer?**
6. **What is the typical total time complexity?**

---

[← Previous Unit: Rotated Array Practice](../05-Search-Rotated-Array/05-practice-problems.md) | [Next: Square Root Problems →](02-square-root.md)
