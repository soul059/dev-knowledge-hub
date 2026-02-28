# Chapter 4.1: Analyzing Your Solution

```
╔══════════════════════════════════════════════════════════╗
║          ANALYZING YOUR SOLUTION                         ║
║   Rigorous complexity analysis step by step              ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Every interview requires you to analyze the time and space complexity of your solution. This isn't just stating "O(n)" — you need to **derive, justify, and explain** why your solution has that complexity. This chapter teaches systematic analysis.

---

## The Analysis Framework

```
┌──────────────────────────────────────────────────────────┐
│  STEP-BY-STEP COMPLEXITY ANALYSIS:                       │
│                                                          │
│  1. Identify all loops and recursive calls              │
│  2. Count iterations for each                           │
│  3. Identify nested vs sequential operations            │
│  4. Multiply nested, add sequential                     │
│  5. Drop constants and lower-order terms                │
│  6. State the final Big-O                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Analyzing Loop Structures

```
SINGLE LOOP:
┌──────────────────────────────────────────────────────────┐
│  for i in range(n):         ← Runs n times             │
│      # O(1) work            ← Constant per iteration   │
│                                                          │
│  TIME: O(n) × O(1) = O(n)                              │
└──────────────────────────────────────────────────────────┘

NESTED LOOPS:
┌──────────────────────────────────────────────────────────┐
│  for i in range(n):         ← Outer: n times           │
│      for j in range(n):     ← Inner: n times each      │
│          # O(1) work                                    │
│                                                          │
│  TIME: O(n) × O(n) = O(n²)                             │
└──────────────────────────────────────────────────────────┘

DEPENDENT NESTED LOOPS:
┌──────────────────────────────────────────────────────────┐
│  for i in range(n):         ← Outer: n times           │
│      for j in range(i):     ← Inner: 0,1,2,...,n-1     │
│          # O(1) work                                    │
│                                                          │
│  Total iterations: 0+1+2+...+(n-1) = n(n-1)/2          │
│  TIME: O(n²)                                            │
│                                                          │
│  Even though inner loop varies, the SUM is O(n²)        │
└──────────────────────────────────────────────────────────┘

SEQUENTIAL LOOPS:
┌──────────────────────────────────────────────────────────┐
│  for i in range(n):         ← O(n)                     │
│      # work                                              │
│                                                          │
│  for j in range(m):         ← O(m)                     │
│      # work                                              │
│                                                          │
│  TIME: O(n) + O(m) = O(n + m)                           │
│  (NOT O(n × m) — they're NOT nested)                    │
└──────────────────────────────────────────────────────────┘

LOOP WITH MULTIPLICATIVE STEP:
┌──────────────────────────────────────────────────────────┐
│  i = 1                                                   │
│  while i < n:               ← How many times?          │
│      # O(1) work                                         │
│      i *= 2                 ← Doubles each time         │
│                                                          │
│  i goes: 1, 2, 4, 8, ..., n                            │
│  Number of steps: log₂(n)                               │
│  TIME: O(log n)                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Analyzing Recursive Solutions

```
TECHNIQUE: RECURRENCE RELATIONS

EXAMPLE 1: Simple Recursion (Factorial)
┌──────────────────────────────────────────────┐
│  def factorial(n):                           │
│      if n <= 1: return 1      ← Base case   │
│      return n * factorial(n-1) ← 1 call     │
│                                              │
│  Recurrence: T(n) = T(n-1) + O(1)          │
│  Expansion:  T(n) = T(n-1) + c             │
│              T(n) = T(n-2) + 2c            │
│              ...                             │
│              T(n) = T(1) + (n-1)c           │
│  TIME: O(n)                                  │
└──────────────────────────────────────────────┘

EXAMPLE 2: Binary Recursion (Merge Sort)
┌──────────────────────────────────────────────┐
│  def mergeSort(arr):                         │
│      if len(arr) <= 1: return arr            │
│      mid = len(arr) // 2                     │
│      left = mergeSort(arr[:mid])    ← T(n/2)│
│      right = mergeSort(arr[mid:])   ← T(n/2)│
│      return merge(left, right)      ← O(n)  │
│                                              │
│  Recurrence: T(n) = 2T(n/2) + O(n)         │
│  Master theorem: a=2, b=2, f(n)=n          │
│  n^(log_b(a)) = n^1 = n = f(n)             │
│  Case 2: T(n) = O(n log n)                 │
└──────────────────────────────────────────────┘

EXAMPLE 3: Exponential Recursion (Fibonacci)
┌──────────────────────────────────────────────┐
│  def fib(n):                                 │
│      if n <= 1: return n                     │
│      return fib(n-1) + fib(n-2)             │
│                                              │
│  Recurrence: T(n) = T(n-1) + T(n-2) + O(1) │
│                                              │
│         fib(5)                               │
│        /      \                              │
│     fib(4)   fib(3)                          │
│     /   \    /   \                           │
│  fib(3) f(2) f(2) f(1)                      │
│  /  \                                        │
│ f(2) f(1)   ← Tree doubles each level       │
│                                              │
│  Levels: n, Branching: 2                    │
│  TIME: O(2ⁿ) approximately                  │
└──────────────────────────────────────────────┘
```

---

## Analyzing Space Complexity

```
WHAT COUNTS AS SPACE:

┌──────────────────────────────────────────────────────────┐
│  COUNTS:                                                 │
│  ├── Extra arrays/lists you create                      │
│  ├── Hash maps/sets you create                          │
│  ├── Recursion call stack depth                         │
│  └── Queue/stack for BFS/DFS                            │
│                                                          │
│  DOES NOT COUNT (usually):                               │
│  ├── The input itself                                   │
│  ├── The output (if required)                           │
│  └── Constant extra variables (i, j, temp)              │
│                                                          │
└──────────────────────────────────────────────────────────┘

EXAMPLES:

  In-place sort (like quicksort):
  Space: O(log n) — recursion stack only

  Hash map storing all elements:
  Space: O(n) — map grows with input

  BFS on a graph:
  Space: O(V) — queue + visited set

  DP table (n × m):
  Space: O(n × m) — 2D array

  Matrix rotation in-place:
  Space: O(1) — only temp variables
```

---

## Common Patterns Quick Reference

```
┌─────────────────────────┬──────────┬──────────┐
│ Pattern                 │ Time     │ Space    │
├─────────────────────────┼──────────┼──────────┤
│ Single loop             │ O(n)     │ O(1)     │
│ Two nested loops        │ O(n²)    │ O(1)     │
│ Binary search           │ O(log n) │ O(1)     │
│ Sort + scan             │ O(nlogn) │ O(1)*    │
│ Hash map lookup         │ O(n)     │ O(n)     │
│ Two pointers (sorted)   │ O(n)     │ O(1)     │
│ Sliding window          │ O(n)     │ O(k)     │
│ BFS/DFS on graph        │ O(V+E)   │ O(V)     │
│ DP (1D)                 │ O(n)     │ O(n)     │
│ DP (2D)                 │ O(n×m)   │ O(n×m)   │
│ Backtracking            │ O(k^n)   │ O(n)     │
│ Merge sort              │ O(nlogn) │ O(n)     │
│ Heap operations         │ O(nlogk) │ O(k)     │
└─────────────────────────┴──────────┴──────────┘
* Depends on sort implementation
```

---

## How to Present Your Analysis

```
TEMPLATE:
"Let me walk through the complexity:
 - The outer loop runs n times.
 - Inside, the hash map lookup is O(1) amortized.
 - So overall: O(n) time.
 - I'm using a hash map that stores at most n elements,
   so O(n) space."

BAD: "It's O(n)." (no justification)
GOOD: "It's O(n) because we visit each element exactly
       once, and each hash map operation is O(1) amortized."
```

---

## Summary Table

| Code Pattern | How to Analyze | Result |
|-------------|----------------|--------|
| `for i in range(n)` | Count iterations | O(n) |
| Nested `for i..n, for j..n` | Multiply | O(n²) |
| `for i..n, for j..i` | Sum series | O(n²) |
| `while i < n: i *= 2` | Count doublings | O(log n) |
| `for i..n` then `for j..m` | Add | O(n + m) |
| Recursion splitting in half | Master theorem | O(n log n) |
| Two recursive calls | Recursion tree | O(2ⁿ) |

---

## Quick Revision Questions

1. **How do you analyze nested loops with dependent bounds?**
   - Sum the iterations: `for i in range(n): for j in range(i)` gives 0+1+2+...+(n-1) = n(n-1)/2 = O(n²)

2. **What's the Master Theorem for T(n) = 2T(n/2) + O(n)?**
   - Case 2: a=2, b=2, f(n)=n, n^(log₂2)=n=f(n), so T(n) = O(n log n)

3. **Does the input count toward space complexity?**
   - Generally no. Space complexity measures EXTRA space beyond the input. Exception: if you modify the input array in-place, note that

4. **How do you analyze recursive solutions?**
   - Write the recurrence relation → expand it or use Master Theorem → draw recursion tree if needed → count total work

5. **What's the space complexity of a recursive function?**
   - At minimum O(depth) for the call stack, plus any extra data structures. For balanced binary tree recursion: O(log n). For linear recursion: O(n)

6. **How should you present complexity analysis in an interview?**
   - Walk through each component: "The loop runs n times, each iteration does O(1) work, total is O(n). The hash map uses O(n) space." Always justify, don't just state

---

[← Previous: Unit 3 — Admitting When Stuck](../03-Communication-During-Interview/06-admitting-when-stuck.md) | [Next: Explaining Complexity →](02-explaining-complexity.md)

---
[← Back to README](../README.md)
