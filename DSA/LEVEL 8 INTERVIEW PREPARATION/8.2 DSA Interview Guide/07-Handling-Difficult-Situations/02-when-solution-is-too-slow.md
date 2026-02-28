# Chapter 7.2: When Solution Is Too Slow

```
╔══════════════════════════════════════════════════════════╗
║          WHEN SOLUTION IS TOO SLOW                       ║
║   Systematic optimization when brute force isn't enough  ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

You've coded a working solution, but it's O(n²) or worse, and the interviewer wants better. This chapter provides a **step-by-step optimization playbook** — how to identify bottlenecks, apply specific speedup techniques, and communicate optimizations clearly.

---

## Step 1: Identify the Bottleneck

```
ASK: Where does my solution spend the most time?

COMMON BOTTLENECK TYPES:
┌───────────────────────────────────────────────────────┐
│ Nested loops         → O(n²) or worse                │
│ Repeated computation → Same subproblem solved again   │
│ Unnecessary sorting  → Sorting when not needed        │
│ Linear search inside loop → O(n) lookup each time     │
│ Full traversal each query → No preprocessing          │
└───────────────────────────────────────────────────────┘

EXAMPLE:
  for i in range(n):           ← Outer: O(n)
      for j in range(n):       ← Inner: O(n)  ← BOTTLENECK
          if arr[i] + arr[j] == target:
              return [i, j]

  Bottleneck: Inner loop does linear search for complement
```

---

## Step 2: Apply the Right Optimization

```
OPTIMIZATION DECISION TREE:

Is bottleneck a nested loop?
├── Looking for a pair?        → Hash Map (O(1) lookup)
├── Need sorted order?         → Sort + Two Pointers
├── Contiguous subarray?       → Sliding Window
└── All pairs comparison?      → Sort and skip duplicates

Is bottleneck repeated work?
├── Overlapping subproblems?   → Dynamic Programming
├── Repeated range queries?    → Prefix Sum
└── Same elements reprocessed? → Memoization

Is bottleneck a search?
├── Sorted data?               → Binary Search
├── Finding k-th element?      → Heap / Quickselect
└── Existence check?           → Hash Set
```

---

## Optimization Technique #1: Hash Map for O(1) Lookup

```
BEFORE: O(n²)                    AFTER: O(n)
for i in range(n):               seen = {}
  for j in range(i+1, n):        for i, num in enumerate(nums):
    if nums[i]+nums[j]==target:     comp = target - num
      return [i, j]                 if comp in seen:
                                      return [seen[comp], i]
                                    seen[num] = i

WHEN TO USE:
  - Need to find complement/pair
  - Need to check "have I seen X?"
  - Need frequency counting
```

---

## Optimization Technique #2: Sorting + Two Pointers

```
BEFORE: O(n²)                    AFTER: O(n log n)
for i in range(n):               nums.sort()
  for j in range(i+1, n):        left, right = 0, n-1
    if nums[i]+nums[j]==target:   while left < right:
      return True                   s = nums[left] + nums[right]
                                    if s == target: return True
                                    elif s < target: left += 1
                                    else: right -= 1

WHEN TO USE:
  - Need pair/triplet that hits a target
  - Array can be sorted (indices not needed)
  - Need to remove duplicates
```

---

## Optimization Technique #3: Sliding Window

```
BEFORE: O(n × k)                  AFTER: O(n)
for i in range(n - k + 1):        window_sum = sum(nums[:k])
  window = sum(nums[i:i+k])       max_sum = window_sum
  max_sum = max(max_sum, window)   for i in range(k, n):
                                     window_sum += nums[i] - nums[i-k]
                                     max_sum = max(max_sum, window_sum)

WHEN TO USE:
  - Contiguous subarray/substring
  - Fixed or variable window size
  - Need max/min of subarray property
```

---

## Optimization Technique #4: Prefix Sum

```
BEFORE: O(n × q) for q queries    AFTER: O(n + q)
# Each query sums a range         prefix = [0] * (n + 1)
for l, r in queries:               for i in range(n):
  total = sum(nums[l:r+1])           prefix[i+1] = prefix[i] + nums[i]
                                   for l, r in queries:
                                     total = prefix[r+1] - prefix[l]

WHEN TO USE:
  - Multiple range sum queries
  - Cumulative calculations
  - Subarray sum equals target
```

---

## Optimization Technique #5: Binary Search

```
BEFORE: O(n)                      AFTER: O(log n)
for i in range(n):                left, right = 0, n - 1
  if nums[i] == target:           while left <= right:
    return i                        mid = left + (right - left) // 2
                                    if nums[mid] == target:
                                      return mid
                                    elif nums[mid] < target:
                                      left = mid + 1
                                    else:
                                      right = mid - 1

WHEN TO USE:
  - Sorted array
  - Monotonic function
  - "Find minimum X such that condition holds"
```

---

## How to Communicate Optimization

```
SCRIPT:

STEP 1: Acknowledge current solution
  "My current solution is O(n²) because of the nested loop."

STEP 2: Identify bottleneck
  "The inner loop is searching for the complement linearly."

STEP 3: Propose optimization
  "If I use a hash map to store seen values, I can look up
   the complement in O(1), bringing total time to O(n)."

STEP 4: Discuss trade-off
  "This trades O(n) extra space for O(n) time improvement."

STEP 5: Get confirmation
  "Shall I implement this optimized version?"
```

---

## Common Optimization Paths

```
O(n³) → O(n²):
  Technique: Precompute pairwise results, prefix sums
  Example: 3Sum → sort + two pointers

O(n²) → O(n log n):
  Technique: Sort + binary search, divide and conquer
  Example: Closest pair of points

O(n²) → O(n):
  Technique: Hash map, sliding window, two pointers
  Example: Two Sum, max subarray, longest substring

O(n log n) → O(n):
  Technique: Counting sort (limited range), bucket sort
  Example: Finding duplicates in bounded range

O(n) → O(log n):
  Technique: Binary search, binary lifting
  Example: Search in sorted array

O(2ⁿ) → O(n × 2^(n/2)):
  Technique: Meet in the middle
  Example: Subset sum with large n

O(2ⁿ) → O(n²) or O(n³):
  Technique: Dynamic programming
  Example: Fibonacci, edit distance, knapsack
```

---

## Summary Table

| Bottleneck | Optimization | New Complexity |
|------------|-------------|----------------|
| Nested search loop | Hash Map | O(n²) → O(n) |
| Pair finding | Sort + Two Pointers | O(n²) → O(n log n) |
| Repeated subarray sum | Sliding Window | O(n×k) → O(n) |
| Range queries | Prefix Sum | O(n×q) → O(n+q) |
| Linear search on sorted | Binary Search | O(n) → O(log n) |
| Overlapping subproblems | DP / Memoization | Exp → Polynomial |

---

## Quick Revision Questions

1. **What's the first step when your solution is too slow?**
   - Identify the bottleneck: which part of the code consumes the most time (usually a nested loop or repeated computation)

2. **How does a hash map optimize nested loops?**
   - It replaces the inner O(n) linear search with O(1) lookup, converting O(n²) to O(n) at the cost of O(n) space

3. **When should you use sliding window vs two pointers?**
   - Sliding window: contiguous subarrays/substrings with size constraints. Two pointers: converging search on sorted data or pair-based problems

4. **How do you communicate an optimization to the interviewer?**
   - State current complexity, identify the bottleneck, propose the technique, explain the trade-off, then ask to proceed

5. **What optimization converts O(2^n) to polynomial time?**
   - Dynamic Programming — by storing solutions to overlapping subproblems and building solutions bottom-up

6. **What's the trade-off in most optimizations?**
   - Almost always space vs time: you use more memory (hash maps, DP tables, prefix arrays) to gain faster execution

---

[← Previous: When Stuck on a Problem](01-when-stuck-on-a-problem.md) | [Next: When Code Has Bugs →](03-when-code-has-bugs.md)

---
[← Back to README](../README.md)
