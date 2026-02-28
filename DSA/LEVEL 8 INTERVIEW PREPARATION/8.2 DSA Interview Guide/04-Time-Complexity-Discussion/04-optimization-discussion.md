# Chapter 4.4: Optimization Discussion

```
╔══════════════════════════════════════════════════════════╗
║         OPTIMIZATION DISCUSSION                          ║
║   Systematic techniques to improve your solution         ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

After presenting a brute force solution, the interviewer often asks "Can you do better?" This chapter gives you a **systematic toolkit** for optimizing solutions — not random guesses, but structured techniques that reliably improve complexity.

---

## The Optimization Decision Tree

```
           ┌─────────────────────────┐
           │  Current solution is    │
           │  too slow. What next?   │
           └────────────┬────────────┘
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
  ┌─────────────┐ ┌──────────┐ ┌───────────────┐
  │ Redundant   │ │ Can we   │ │ Can we avoid  │
  │ computation?│ │ sort     │ │ nested loops? │
  │ → Hash Map  │ │ first?   │ │ → Two Pointers│
  │ → Memoize   │ │ → Binary │ │ → Sliding Win │
  └─────────────┘ │   Search │ └───────────────┘
                   └──────────┘
```

---

## Top 10 Optimization Techniques

### 1. Hash Map — Eliminate Repeated Lookups

```
BEFORE: Searching for complement in array → O(n) per search
AFTER:  Store in hash map → O(1) per lookup

PATTERN:
  seen = {}
  for element in arr:
      if complement in seen:    ← O(1)
          # found it
      seen[element] = True

REDUCES: O(n²) → O(n)
USE WHEN: Need to check existence, find pairs, count frequency
```

### 2. Sorting — Enable Binary Search & Two Pointers

```
BEFORE: Unsorted array, must check all pairs → O(n²)
AFTER:  Sort first → use two pointers → O(n log n)

  [3, 1, 4, 1, 5, 9] → Sort → [1, 1, 3, 4, 5, 9]
                                 ↑              ↑
                                left          right

REDUCES: O(n²) → O(n log n)
USE WHEN: Finding pairs, removing duplicates, range queries
```

### 3. Two Pointers — Replace Nested Loops

```
BEFORE:
  for i in range(n):
      for j in range(i+1, n):     ← O(n²)
          check pair(i, j)

AFTER (on sorted data):
  left, right = 0, n-1
  while left < right:              ← O(n)
      if condition_met: process
      elif too_small: left += 1
      else: right -= 1

REDUCES: O(n²) → O(n)
USE WHEN: Sorted arrays, pair finding, container problems
```

### 4. Sliding Window — Subarray/Substring Problems

```
BEFORE: Check all subarrays → O(n²) or O(n³)
AFTER:  Maintain a window that slides → O(n)

  [a b c d e f g h i]
   └───────┘           Window of size k
     └───────┘         Slide right
       └───────┘       Each element enters/exits once

REDUCES: O(n²) → O(n)
USE WHEN: Contiguous subarray/substring problems with a condition
```

### 5. Binary Search — Exploit Monotonicity

```
BEFORE: Linear scan → O(n)
AFTER:  Binary search → O(log n)

  Requirements:
  ├── Sorted array, OR
  ├── Monotonic function, OR
  └── "Search space" that can be halved

  Binary search on ANSWER:
  "What's the minimum speed to eat all bananas in h hours?"
  Low=1, High=max(piles)
  Mid = (Low+High)//2
  Can eat all at speed mid? → Move Low or High

REDUCES: O(n) → O(log n)
USE WHEN: Searching sorted data, optimization problems
```

### 6. Prefix Sum — Subarray Sum Queries

```
BEFORE: Sum subarray each time → O(n) per query
AFTER:  Build prefix, answer in O(1)

  arr:    [1, 2, 3, 4, 5]
  prefix: [0, 1, 3, 6, 10, 15]

  sum(l, r) = prefix[r+1] - prefix[l]

REDUCES: O(n) per query → O(1) per query
USE WHEN: Multiple range sum queries, subarray sum problems
```

### 7. Stack/Queue/Deque — Maintain Important Elements

```
MONOTONIC STACK:
  Find next greater element for each position
  
  BEFORE: For each element, scan right → O(n²)
  AFTER:  Maintain decreasing stack → O(n)

  Process: [2, 1, 4, 3]
  Stack tracks elements waiting for their "next greater"
  When 4 comes: pop 1 (next greater = 4), pop 2 (next = 4)

REDUCES: O(n²) → O(n)
USE WHEN: Next greater/smaller, histogram problems, span
```

### 8. Heap (Priority Queue) — Track Top K

```
BEFORE: Sort entire array → O(n log n)
AFTER:  Maintain heap of size k → O(n log k)

  To find kth largest:
  Use a min-heap of size k.
  For each element:
      If element > heap top: replace top, heapify

REDUCES: O(n log n) → O(n log k)
USE WHEN: Top K elements, merge K sorted lists, median stream
```

### 9. Dynamic Programming — Avoid Recomputation

```
BEFORE: Recursive exploration → O(2ⁿ)
AFTER:  DP table stores results → O(n) or O(n²)

  RECOGNIZE DP:
  ├── Optimal substructure (best answer uses best sub-answers)
  ├── Overlapping subproblems (same computation repeated)
  └── Can define state as dp[i] = answer for first i elements

REDUCES: O(2ⁿ) → O(n²) or O(n×m)
USE WHEN: Counting paths, optimization, sequence problems
```

### 10. Divide and Conquer — Split Problem in Half

```
BEFORE: Process entire array → O(n²)
AFTER:  Split, solve halves, combine → O(n log n)

  Merge Sort:
  Split array in half → sort each → merge
  
  Levels: log n
  Work per level: O(n)
  Total: O(n log n)

REDUCES: O(n²) → O(n log n)
USE WHEN: Sorting, closest pair, count inversions
```

---

## Optimization Conversation Template

```
STEP 1: Acknowledge the brute force
"My current solution is O(n²) because of the nested loops."

STEP 2: Identify the bottleneck
"The bottleneck is the inner loop searching for the
 complement — that's a repeated linear scan."

STEP 3: Apply optimization
"If I store seen elements in a hash map, the lookup
 becomes O(1) instead of O(n)."

STEP 4: State new complexity
"This brings us from O(n²) to O(n) time, using O(n)
 extra space for the hash map."

STEP 5: Verify with example
"Let me trace through the example to confirm..."
```

---

## Summary Table

| Technique | Typical Reduction | Key Requirement |
|-----------|-------------------|-----------------|
| Hash Map | O(n²) → O(n) | Need fast lookup |
| Sorting | O(n²) → O(n log n) | Order helps |
| Two Pointers | O(n²) → O(n) | Sorted data |
| Sliding Window | O(n²) → O(n) | Contiguous subarray |
| Binary Search | O(n) → O(log n) | Monotonic property |
| Prefix Sum | O(n) → O(1) per query | Range sum queries |
| Monotonic Stack | O(n²) → O(n) | Next greater/smaller |
| Heap | O(n log n) → O(n log k) | Top-K problems |
| DP | O(2ⁿ) → O(n²) | Overlapping subproblems |
| Divide & Conquer | O(n²) → O(n log n) | Splittable problem |

---

## Quick Revision Questions

1. **What's the first thing to do when asked to optimize?**
   - Identify the bottleneck — the specific part of your solution that dominates the time complexity

2. **How does a hash map optimize brute force pair-finding?**
   - Replaces O(n) linear scan for complement with O(1) hash lookup, reducing total from O(n²) to O(n)

3. **When should you use two pointers vs sliding window?**
   - Two pointers: finding pairs in sorted data (converging from both ends). Sliding window: finding optimal contiguous subarray with a property

4. **What are the three signs that DP applies?**
   - Optimal substructure, overlapping subproblems, and the ability to define a recurrence relation

5. **How do you present optimization in an interview?**
   - Acknowledge brute force → Identify bottleneck → Propose technique → State new complexity → Verify with example

6. **What optimization does sorting enable?**
   - Binary search (O(log n) lookup), two pointers (O(n) pair finding), and easy duplicate removal

---

[← Previous: Space vs Time Trade-Offs](03-space-vs-time-trade-offs.md) | [Next: Common Complexity Patterns →](05-common-complexity-patterns.md)

---
[← Back to README](../README.md)
