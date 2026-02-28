# Chapter 5.4: Edge Case Handling

```
╔══════════════════════════════════════════════════════════╗
║           EDGE CASE HANDLING                             ║
║   The difference between "almost works" and "correct"    ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Edge cases are the **special inputs** that break naive implementations. Handling them correctly shows thoroughness and attention to detail — qualities interviewers highly value. Missing even one edge case can turn a "Hire" into a "No Hire."

---

## Universal Edge Cases (Check for EVERY Problem)

```
┌──────────────────────────────────────────────────────────┐
│  THE BIG 5 — Always Check These:                        │
│                                                          │
│  1. EMPTY INPUT                                         │
│     arr = [], s = "", tree = None, graph = {}           │
│                                                          │
│  2. SINGLE ELEMENT                                      │
│     arr = [1], s = "a", tree with only root             │
│                                                          │
│  3. ALL SAME ELEMENTS                                   │
│     arr = [5, 5, 5, 5], s = "aaaa"                     │
│                                                          │
│  4. BOUNDARY VALUES                                     │
│     0, -1, INT_MAX, INT_MIN, n=1, n=max                │
│                                                          │
│  5. ALREADY SOLVED / TRIVIAL INPUT                      │
│     Already sorted, no duplicates needed, target = 0    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Data Structure-Specific Edge Cases

```
ARRAYS:
┌─────────────────────────────────────────────────┐
│ ☐ Empty array []                               │
│ ☐ Single element [5]                           │
│ ☐ Two elements [1, 2]                          │
│ ☐ All duplicates [3, 3, 3]                     │
│ ☐ Already sorted [1, 2, 3, 4]                 │
│ ☐ Reverse sorted [4, 3, 2, 1]                 │
│ ☐ Contains negatives [-3, -1, 0, 2]           │
│ ☐ Contains zeros [0, 0, 1, 2]                 │
│ ☐ Very large values [INT_MAX, INT_MIN]         │
│ ☐ Single pair that works                       │
└─────────────────────────────────────────────────┘

STRINGS:
┌─────────────────────────────────────────────────┐
│ ☐ Empty string ""                              │
│ ☐ Single character "a"                         │
│ ☐ All same characters "aaaa"                   │
│ ☐ Palindrome "abcba"                           │
│ ☐ With spaces "a b c"                          │
│ ☐ With special characters "a@b#c"              │
│ ☐ Upper and lower case "aBcD"                  │
│ ☐ Very long string (performance)               │
│ ☐ Unicode characters (if applicable)           │
└─────────────────────────────────────────────────┘

TREES:
┌─────────────────────────────────────────────────┐
│ ☐ Null / empty tree                            │
│ ☐ Single node (root only)                      │
│ ☐ Only left children (left-skewed)             │
│ ☐ Only right children (right-skewed)           │
│ ☐ Perfect binary tree                          │
│ ☐ All same values                              │
│ ☐ Negative values in tree                      │
│ ☐ Very deep tree (stack overflow?)             │
└─────────────────────────────────────────────────┘

   Balanced:       Left-Skewed:    Single:
      1               1             1
     / \             /
    2   3           2
   / \             /
  4   5           3

LINKED LISTS:
┌─────────────────────────────────────────────────┐
│ ☐ Null / empty list                            │
│ ☐ Single node                                  │
│ ☐ Two nodes                                    │
│ ☐ Cycle present (for cycle detection)          │
│ ☐ All same values                              │
│ ☐ Already sorted vs unsorted                   │
└─────────────────────────────────────────────────┘

GRAPHS:
┌─────────────────────────────────────────────────┐
│ ☐ Empty graph (no nodes)                       │
│ ☐ Single node, no edges                        │
│ ☐ Disconnected components                      │
│ ☐ Cycles                                       │
│ ☐ Self-loops                                   │
│ ☐ Parallel edges (multigraph)                  │
│ ☐ Complete graph (all connected)               │
│ ☐ Tree (graph with no cycles)                  │
└─────────────────────────────────────────────────┘
```

---

## How to Handle Edge Cases in Code

```
PATTERN 1: Guard Clauses (Early Returns)
┌──────────────────────────────────────────┐
│  def max_depth(root):                    │
│      if root is None:    ← Edge case    │
│          return 0                        │
│      # Main logic below                 │
│      left = max_depth(root.left)        │
│      right = max_depth(root.right)      │
│      return max(left, right) + 1        │
└──────────────────────────────────────────┘

PATTERN 2: Default Values
┌──────────────────────────────────────────┐
│  def find_max(nums):                     │
│      if not nums:                        │
│          return float('-inf')  ← or None│
│      max_val = nums[0]  ← Safe init    │
│      for num in nums[1:]:               │
│          max_val = max(max_val, num)    │
│      return max_val                      │
└──────────────────────────────────────────┘

PATTERN 3: Boundary Checks
┌──────────────────────────────────────────┐
│  def is_valid(row, col, grid):           │
│      return (0 <= row < len(grid) and   │
│              0 <= col < len(grid[0]))    │
│                                          │
│  # Use before accessing grid[row][col]  │
└──────────────────────────────────────────┘
```

---

## Edge Case Thinking Process

```
WHEN YOU SEE THE PROBLEM, ASK:

  "What if the input is... 
   ┌──────────────────────────────────────┐
   │  ...empty?"           → return []   │
   │  ...has one element?" → still works?│
   │  ...has all same?"    → still works?│
   │  ...has negatives?"   → still works?│
   │  ...is at maximum?"   → overflow?   │
   │  ...is already done?" → skip work?  │
   └──────────────────────────────────────┘

  THEN after coding, re-check:
   ┌──────────────────────────────────────┐
   │  Does my loop start correctly?       │
   │  Does my loop end correctly?         │
   │  Is my initial value correct?        │
   │  Do I access out-of-bounds indices?  │
   │  Do I handle the "not found" case?   │
   └──────────────────────────────────────┘
```

---

## Common Edge Case Bugs

```
BUG 1: Off-by-one in array bounds
  ✗ for i in range(len(arr) + 1)  ← out of bounds
  ✓ for i in range(len(arr))

BUG 2: Wrong initialization
  ✗ max_val = 0  ← fails for all-negative arrays
  ✓ max_val = float('-inf')
  ✓ max_val = nums[0]

BUG 3: Empty check missing
  ✗ return nums[0]  ← crashes on empty array
  ✓ if not nums: return None
     return nums[0]

BUG 4: Integer overflow
  ✗ mid = (lo + hi) / 2  ← overflow for large values
  ✓ mid = lo + (hi - lo) // 2

BUG 5: Division by zero
  ✗ average = total / count
  ✓ average = total / count if count > 0 else 0
```

---

## Summary Table

| Data Type | Must-Check Edge Cases | Common Bug |
|-----------|----------------------|------------|
| Array | Empty, single element, negatives | Off-by-one, wrong init |
| String | Empty, single char, all same | Missing empty check |
| Tree | Null root, single node, skewed | Not checking null |
| Graph | Disconnected, cycles, empty | Missing visited check |
| Number | 0, negative, MAX_INT | Overflow, division by zero |

---

## Quick Revision Questions

1. **What are the "Big 5" universal edge cases?**
   - Empty input, single element, all same elements, boundary values, already-solved input

2. **How should edge cases be handled in code?**
   - Guard clauses (early returns) at the top of the function, before the main logic

3. **What's the most common edge case bug in array problems?**
   - Off-by-one errors in loop bounds and wrong initialization (e.g., `max_val = 0` instead of `float('-inf')`)

4. **Why is `mid = lo + (hi - lo) // 2` better than `(lo + hi) // 2`?**
   - Prevents integer overflow when lo and hi are both large values

5. **When should you check for edge cases during an interview?**
   - Twice: once when understanding the problem (ask about edge cases), and again after coding (test your solution against them)

6. **What tree edge case is most commonly missed?**
   - Skewed trees (all left or all right children) — they turn O(log n) assumptions into O(n) reality

---

[← Previous: Function Structure](03-function-structure.md) | [Next: Error Handling →](05-error-handling.md)

---
[← Back to README](../README.md)
