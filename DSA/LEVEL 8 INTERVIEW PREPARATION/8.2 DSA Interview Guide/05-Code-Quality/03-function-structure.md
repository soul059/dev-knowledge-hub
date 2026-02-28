# Chapter 5.3: Function Structure

```
╔══════════════════════════════════════════════════════════╗
║            FUNCTION STRUCTURE                            ║
║   Organizing code into logical, testable units           ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Well-structured functions make your code easier to write, debug, and explain. In interviews, breaking your solution into smaller functions demonstrates **software engineering skills** beyond just knowing algorithms.

---

## The Interview Function Template

```
┌──────────────────────────────────────────────────────────┐
│  def solve(params):                                      │
│      """Main entry point."""                             │
│                                                          │
│      # SECTION 1: Validate & handle edge cases          │
│      if not params:                                      │
│          return default_result                            │
│                                                          │
│      # SECTION 2: Initialize data structures            │
│      data_structure = initialize()                       │
│                                                          │
│      # SECTION 3: Core algorithm                        │
│      for element in params:                              │
│          process(element, data_structure)                │
│                                                          │
│      # SECTION 4: Build and return result               │
│      return build_result(data_structure)                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## When to Extract Helper Functions

```
EXTRACT A HELPER WHEN:

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  1. A block has a CLEAR, SINGLE PURPOSE                 │
│     "This part calculates the distance between nodes"   │
│     → def calculate_distance(node1, node2)              │
│                                                          │
│  2. Code is REPEATED                                    │
│     Same check appears in multiple places               │
│     → def is_valid(row, col, grid)                      │
│                                                          │
│  3. Logic is COMPLEX enough to name                     │
│     "This converts the string into a frequency map"     │
│     → def build_frequency_map(s)                        │
│                                                          │
│  4. It improves READABILITY of the main function        │
│     Main function reads like a story when helpers       │
│     handle the details                                   │
│                                                          │
└──────────────────────────────────────────────────────────┘

DON'T EXTRACT WHEN:
┌──────────────────────────────────────────────────────────┐
│  ✗ The helper would be 1-2 lines                       │
│  ✗ It's only used once and is already clear            │
│  ✗ Extracting would break the flow/readability          │
│  ✗ Time pressure — helper can be mentioned verbally    │
└──────────────────────────────────────────────────────────┘
```

---

## Real Examples: With and Without Helpers

```
WITHOUT HELPERS (everything in one function):
┌──────────────────────────────────────────────────────────┐
│  def num_islands(grid):                                  │
│      if not grid: return 0                               │
│      rows, cols = len(grid), len(grid[0])               │
│      count = 0                                           │
│      for r in range(rows):                              │
│          for c in range(cols):                           │
│              if grid[r][c] == '1':                       │
│                  count += 1                              │
│                  # DFS inline                            │
│                  stack = [(r, c)]                        │
│                  while stack:                            │
│                      cr, cc = stack.pop()               │
│                      for dr, dc in directions:          │
│                          nr, nc = cr+dr, cc+dc          │
│                          if 0<=nr<rows and 0<=nc<cols   │
│                             and grid[nr][nc]=='1':      │
│                              grid[nr][nc] = '0'         │
│                              stack.append((nr,nc))      │
│      return count                                        │
└──────────────────────────────────────────────────────────┘

WITH HELPERS (cleaner structure):
┌──────────────────────────────────────────────────────────┐
│  def num_islands(grid):                                  │
│      if not grid:                                        │
│          return 0                                        │
│                                                          │
│      count = 0                                           │
│      for r in range(len(grid)):                         │
│          for c in range(len(grid[0])):                  │
│              if grid[r][c] == '1':                       │
│                  count += 1                              │
│                  sink_island(grid, r, c)                 │
│      return count                                        │
│                                                          │
│  def sink_island(grid, row, col):                       │
│      """DFS to mark all connected land as visited."""   │
│      if (row < 0 or row >= len(grid) or                │
│          col < 0 or col >= len(grid[0]) or             │
│          grid[row][col] != '1'):                        │
│          return                                          │
│      grid[row][col] = '0'                               │
│      for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:        │
│          sink_island(grid, row+dr, col+dc)              │
│                                                          │
│  ✓ Main function reads like English                     │
│  ✓ DFS logic is isolated and testable                  │
│  ✓ Each function has one clear purpose                 │
└──────────────────────────────────────────────────────────┘
```

---

## Function Design Principles

```
PRINCIPLE 1: SINGLE RESPONSIBILITY
  Each function does ONE thing well.
  
  ✗ def process_and_sort_and_filter(data)
  ✓ def filter_valid(data)
  ✓ def sort_by_key(filtered)
  ✓ def process(sorted_data)

PRINCIPLE 2: CONSISTENT ABSTRACTION LEVEL
  Don't mix high-level and low-level in one function.
  
  ✗ def solve():
      data = parse(input)          ← high level
      for i in range(len(data)):   ← low level
          if data[i] > 0:          ← low level
      return format(result)        ← high level

  ✓ def solve():
      data = parse(input)         ← all high level
      filtered = filter_positive(data)
      return format(filtered)

PRINCIPLE 3: MINIMAL PARAMETERS
  Keep function signatures simple.
  
  ✗ def helper(arr, left, right, target, seen, result, depth)
  ✓ Use class variables or closures for shared state
  ✓ Group related params into a structure if needed

PRINCIPLE 4: RETURN EARLY
  Reduce nesting with guard clauses.
  
  ✗ def process(node):
      if node:
          if node.val > 0:
              if node.left:
                  # actual logic (deeply nested)

  ✓ def process(node):
      if not node: return
      if node.val <= 0: return
      if not node.left: return
      # actual logic (flat)
```

---

## Interview-Specific Function Tips

```
┌──────────────────────────────────────────────────────────┐
│  TIP 1: Write the main function first as a SKELETON     │
│                                                          │
│  def solve(nums, target):                                │
│      nums.sort()                                         │
│      result = []                                         │
│      for i in range(len(nums)):                         │
│          pairs = find_pairs(nums, i, target - nums[i])  │
│          result.extend(pairs)                            │
│      return result                                       │
│                                                          │
│  # "I'll implement find_pairs next"                     │
│                                                          │
│  TIP 2: Mention helpers you WOULD write                 │
│  "In production, I'd extract this validation into a     │
│   separate function, but for time I'll keep it inline." │
│                                                          │
│  TIP 3: Use built-in functions confidently              │
│  sorted(), min(), max(), sum(), enumerate(), zip()      │
│  collections.Counter(), collections.defaultdict()       │
│  heapq.heappush(), heapq.heappop()                     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Bad Practice | Good Practice |
|--------|-------------|---------------|
| Length | 50+ line monolith | 15-25 lines per function |
| Responsibility | Does 3 things | Does 1 thing well |
| Parameters | 7+ parameters | 2-4 parameters |
| Nesting | 4+ levels deep | 1-2 levels with early returns |
| Naming | `helper()`, `process()` | `sink_island()`, `find_pairs()` |
| Flow | Hard to follow | Reads top-to-bottom like a story |

---

## Quick Revision Questions

1. **When should you extract a helper function in an interview?**
   - When code has a clear single purpose (like DFS traversal), is repeated, or would significantly improve readability of the main function

2. **What's the ideal function structure?**
   - Edge cases → Initialize → Core logic → Return. Each section is small and focused

3. **How long should an interview function be?**
   - 15-25 lines ideally. If it's longer, consider extracting helpers. But under time pressure, inline code with good comments is acceptable

4. **What's the "consistent abstraction level" principle?**
   - Don't mix high-level calls (parse, format) with low-level details (manual loops, index arithmetic) in the same function

5. **Should you use built-in functions in interviews?**
   - Yes. Using `sorted()`, `collections.Counter()`, etc. shows language fluency. Only implement from scratch if asked

6. **How do you handle time pressure vs clean structure?**
   - Write the skeleton with helper function calls first, implement helpers next. If time is tight, mention "I'd extract this in production" and keep it inline

---

[← Previous: Variable Naming](02-variable-naming.md) | [Next: Edge Case Handling →](04-edge-case-handling.md)

---
[← Back to README](../README.md)
