# Chapter 4.5: Common Complexity Patterns

```
╔══════════════════════════════════════════════════════════╗
║        COMMON COMPLEXITY PATTERNS                        ║
║   Recognizing and explaining recurring patterns          ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Certain complexity patterns appear again and again in DSA problems. Recognizing them instantly lets you **predict the expected complexity from the problem constraints**, choose the right algorithm family, and verify your solution is in the right ballpark.

---

## The Complexity Hierarchy

```
                    OPERATIONS FOR n = 1,000,000
  O(1)        │  1                         
  O(log n)    │  20                        
  O(√n)       │  1,000                     
  O(n)        │  1,000,000                 
  O(n log n)  │  20,000,000                
  O(n√n)      │  1,000,000,000    ← borderline
  O(n²)       │  1,000,000,000,000 ← TOO SLOW
  O(2ⁿ)      │  Astronomical      ← IMPOSSIBLE
              └─────────────────────────────────
```

---

## Pattern 1: O(n) — Single Pass

```
RECOGNIZERS:
├── "Find the maximum/minimum"
├── "Count the occurrences"
├── "Check if something exists"
├── "Merge two sorted arrays"
└── n can be up to 10⁷ or 10⁸

TYPICAL CODE SHAPES:
  for element in arr:        ← visit each once
      # constant work

  left, right = 0, n-1      ← two pointers
  while left < right:        ← each moves at most n times
      # process

HOW IT WORKS:
  Each element is processed exactly once (or a constant
  number of times). No element is revisited.

EXAMPLES:
  - Maximum subarray (Kadane's) → O(n)
  - Two sum with hash map → O(n)
  - Valid parentheses with stack → O(n)
  - Sliding window maximum → O(n) amortized
```

---

## Pattern 2: O(n log n) — Sort or Divide

```
RECOGNIZERS:
├── Problem involves sorting
├── "Find the kth element"
├── Divide and conquer structure
├── n up to 10⁵ or 10⁶
└── O(n) seems impossible, O(n²) is too slow

TYPICAL CODE SHAPES:
  arr.sort()                 ← O(n log n)
  # then O(n) scan

  def solve(arr):            ← divide and conquer
      mid = len(arr) // 2
      left = solve(arr[:mid])
      right = solve(arr[mid:])
      return merge(left, right)  ← O(n) merge

WHY n log n:
  log n levels of recursion × n work per level
  
  Level 0:  [████████████████]     n work
  Level 1:  [████████] [████████]  n work  
  Level 2:  [████][████][████][████] n work
  ...
  log n levels total → O(n log n)

EXAMPLES:
  - Merge sort, quicksort, heapsort
  - Count inversions
  - Closest pair of points
  - Meeting rooms (sort by start time)
```

---

## Pattern 3: O(n²) — All Pairs

```
RECOGNIZERS:
├── "Check all pairs"
├── "Compare each with every other"
├── DP with two dimensions on same array
├── n ≤ 5000 or 10,000
└── Brute force for most pair problems

TYPICAL CODE SHAPES:
  for i in range(n):
      for j in range(i+1, n):
          # process pair (i, j)

  dp[i][j] = some function of dp[i-1][j], dp[i][j-1]

EXAMPLES:
  - Brute force two sum → O(n²)
  - Longest Palindromic Substring (expand around center)
  - DP: Longest Common Subsequence → O(n × m)
  - Matrix chain multiplication
```

---

## Pattern 4: O(log n) — Binary Search

```
RECOGNIZERS:
├── Sorted array with search query
├── "Find the boundary" (first/last occurrence)
├── "Minimize the maximum" or "maximize the minimum"
├── Monotonic search space
└── n can be up to 10⁹ or more

TYPICAL CODE SHAPES:
  lo, hi = 0, n - 1
  while lo <= hi:
      mid = (lo + hi) // 2
      if condition(mid):
          # update answer
          hi = mid - 1    # or lo = mid + 1
      else:
          lo = mid + 1    # or hi = mid - 1

WHY log n:
  ┌─────────────────────────────────────────┐
  │  Start: 1,000,000 elements             │
  │  Step 1: 500,000 remaining             │
  │  Step 2: 250,000 remaining             │
  │  Step 3: 125,000 remaining             │
  │  ...                                    │
  │  Step 20: 1 remaining → FOUND          │
  │                                         │
  │  Only 20 steps for 1 million elements! │
  └─────────────────────────────────────────┘

EXAMPLES:
  - Search in sorted array
  - Find first bad version
  - Search in rotated array
  - Koko eating bananas (binary search on answer)
```

---

## Pattern 5: O(V + E) — Graph Traversal

```
RECOGNIZERS:
├── "Connected components"
├── "Shortest path (unweighted)"
├── "Is there a path from A to B?"
├── "Detect cycle"
└── Anything with nodes and edges

TYPICAL CODE SHAPES:
  # BFS
  queue = deque([start])
  visited = {start}
  while queue:
      node = queue.popleft()
      for neighbor in graph[node]:
          if neighbor not in visited:
              visited.add(neighbor)
              queue.append(neighbor)

WHY V + E:
  Visit each vertex once → O(V)
  Traverse each edge once → O(E)
  Total: O(V + E)

EXAMPLES:
  - BFS/DFS traversal
  - Topological sort → O(V + E)
  - Detect cycle in graph → O(V + E)
  - Connected components → O(V + E)
  - Bipartite check → O(V + E)
```

---

## Pattern 6: O(2ⁿ) — Subsets / Exponential

```
RECOGNIZERS:
├── "Generate all subsets/combinations"
├── "Try all possibilities"
├── "Knapsack without DP optimization"
├── n ≤ 20 or n ≤ 25
└── Decision at each step: include or exclude

TYPICAL CODE SHAPE:
  def backtrack(index, current):
      if index == n:
          process(current)
          return
      # Include element
      backtrack(index + 1, current + [arr[index]])
      # Exclude element
      backtrack(index + 1, current)

WHY 2ⁿ:
  Each element: 2 choices (include/exclude)
  n elements → 2 × 2 × ... × 2 = 2ⁿ subsets
  
  n=3: {}, {a}, {b}, {c}, {a,b}, {a,c}, {b,c}, {a,b,c}
        = 8 = 2³ subsets

EXAMPLES:
  - Generate all subsets
  - Sum of subsets
  - Bitmask DP
  - Traveling salesman (with memoization → O(2ⁿ × n))
```

---

## Pattern Recognition Cheat Sheet

```
┌──────────────┬──────────────┬────────────────────────┐
│ Constraint   │ Target       │ Likely Technique       │
├──────────────┼──────────────┼────────────────────────┤
│ n ≤ 10       │ O(n!)        │ Permutations           │
│ n ≤ 20       │ O(2ⁿ)       │ Bitmask/Backtracking   │
│ n ≤ 500      │ O(n³)        │ Floyd-Warshall, 3D DP  │
│ n ≤ 5000     │ O(n²)        │ DP, all pairs          │
│ n ≤ 10⁶      │ O(n log n)   │ Sort + scan, D&C      │
│ n ≤ 10⁸      │ O(n)         │ Single pass, hash map  │
│ n ≤ 10¹²     │ O(√n),O(logn)│ Math, binary search    │
│ n ≤ 10¹⁸     │ O(log n)     │ Binary search, math    │
└──────────────┴──────────────┴────────────────────────┘
```

---

## Summary Table

| Pattern | Complexity | Key Signal | Common Algorithms |
|---------|-----------|------------|-------------------|
| Single pass | O(n) | Large n, simple scan | Hash map, two pointers, Kadane's |
| Sort-based | O(n log n) | Need ordering | Merge sort, sort+scan |
| All pairs | O(n²) | Small n, pairs | Brute force, 2D DP |
| Halving | O(log n) | Sorted/monotonic | Binary search |
| Graph walk | O(V+E) | Nodes and edges | BFS, DFS, topological sort |
| All subsets | O(2ⁿ) | Very small n, choices | Backtracking, bitmask |

---

## Quick Revision Questions

1. **How do you predict the expected complexity from constraints?**
   - Use constraint mapping: n ≤ 10⁴ → O(n²) feasible, n ≤ 10⁶ → need O(n log n) or O(n), n ≤ 10⁸ → must be O(n) or better

2. **Why is O(n log n) so common?**
   - It's the complexity of sorting, and many problems reduce to "sort first, then scan." It's also the result of divide-and-conquer with linear merge

3. **How do you recognize an O(n) solution is possible?**
   - If n can be 10⁷ or 10⁸, you need O(n). Look for hash maps, two pointers, sliding window, or single-pass techniques

4. **What's the difference between O(V+E) and O(n²) for graphs?**
   - O(V+E) for sparse graphs (few edges); O(V²) when using adjacency matrix or dense graphs where E ≈ V²

5. **When does O(2ⁿ) appear?**
   - When you need to consider all subsets (include/exclude each element). Only feasible for n ≤ 20-25

6. **How do you explain why binary search is O(log n)?**
   - Each step eliminates half the search space. Starting from n elements: n → n/2 → n/4 → ... → 1, which takes log₂(n) steps

---

[← Previous: Optimization Discussion](04-optimization-discussion.md) | [Next: Unit 5 — Clean Code Principles →](../05-Code-Quality/01-clean-code-principles.md)

---
[← Back to README](../README.md)
