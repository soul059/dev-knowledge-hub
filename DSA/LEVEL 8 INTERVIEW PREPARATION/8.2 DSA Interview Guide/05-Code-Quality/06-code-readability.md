# Chapter 5.6: Code Readability

```
╔══════════════════════════════════════════════════════════╗
║            CODE READABILITY                              ║
║   Making your code a pleasure to review                  ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Readable code is code that **another developer can understand quickly**. In interviews, the "other developer" is your interviewer — they need to evaluate your solution in real time. Readable code gets better scores because it's easier to verify as correct.

---

## Readability Techniques

```
TECHNIQUE 1: VERTICAL SPACING
┌──────────────────────────────────────────────────────────┐
│  BAD (wall of code):                                     │
│  def solve(nums, target):                                │
│      seen = {}                                           │
│      for i, num in enumerate(nums):                     │
│          comp = target - num                             │
│          if comp in seen:                                │
│              return [seen[comp], i]                      │
│          seen[num] = i                                   │
│      return []                                           │
│                                                          │
│  GOOD (logical grouping with blank lines):              │
│  def solve(nums, target):                                │
│      seen = {}                                           │
│                                                          │
│      for i, num in enumerate(nums):                     │
│          comp = target - num                             │
│                                                          │
│          if comp in seen:                                │
│              return [seen[comp], i]                      │
│                                                          │
│          seen[num] = i                                   │
│                                                          │
│      return []                                           │
└──────────────────────────────────────────────────────────┘

TECHNIQUE 2: MEANINGFUL COMMENTS
┌──────────────────────────────────────────────────────────┐
│  BAD COMMENTS (state the obvious):                      │
│  i += 1  # increment i                                  │
│  return result  # return the result                     │
│                                                          │
│  GOOD COMMENTS (explain WHY):                           │
│  # Skip duplicates to avoid repeated triplets           │
│  while left < right and nums[left] == nums[left+1]:    │
│      left += 1                                           │
│                                                          │
│  # Use negative values to mark visited in-place         │
│  nums[abs(num) - 1] *= -1                              │
└──────────────────────────────────────────────────────────┘

TECHNIQUE 3: DESCRIPTIVE CONDITIONS
┌──────────────────────────────────────────────────────────┐
│  BAD:                                                    │
│  if x > 0 and x < n and y > 0 and y < m and g[x][y]:  │
│                                                          │
│  GOOD:                                                   │
│  is_in_bounds = 0 <= row < rows and 0 <= col < cols    │
│  is_land = grid[row][col] == '1'                       │
│  if is_in_bounds and is_land:                           │
│                                                          │
│  OR (with helper):                                       │
│  if is_valid(row, col) and is_land(row, col):          │
└──────────────────────────────────────────────────────────┘

TECHNIQUE 4: AVOID CLEVER CODE
┌──────────────────────────────────────────────────────────┐
│  CLEVER (impressive but confusing):                     │
│  return [x for x in (i for i in range(n)               │
│          if f(i)) if g(x)]                              │
│                                                          │
│  CLEAR (same result, easy to follow):                   │
│  result = []                                             │
│  for i in range(n):                                     │
│      if f(i) and g(i):                                  │
│          result.append(i)                                │
│  return result                                           │
│                                                          │
│  "Clear is better than clever"                          │
└──────────────────────────────────────────────────────────┘
```

---

## Readability Comparison: Full Example

```
HARD TO READ:
┌──────────────────────────────────────────────────────────┐
│  def f(g):                                               │
│      if not g:return 0                                   │
│      r,c=len(g),len(g[0])                               │
│      ct=0                                                │
│      for i in range(r):                                  │
│          for j in range(c):                              │
│              if g[i][j]=='1':                            │
│                  ct+=1;q=[(i,j)];g[i][j]='0'            │
│                  while q:                                │
│                      x,y=q.pop(0)                       │
│                      for dx,dy in[(0,1),(0,-1),(1,0),   │
│                      (-1,0)]:                            │
│                          nx,ny=x+dx,y+dy                │
│                          if 0<=nx<r and 0<=ny<c and     │
│                          g[nx][ny]=='1':                 │
│                              g[nx][ny]='0';q.append(    │
│                              (nx,ny))                    │
│      return ct                                           │
└──────────────────────────────────────────────────────────┘

EASY TO READ:
┌──────────────────────────────────────────────────────────┐
│  def num_islands(grid):                                  │
│      if not grid:                                        │
│          return 0                                        │
│                                                          │
│      rows, cols = len(grid), len(grid[0])               │
│      island_count = 0                                    │
│      directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]   │
│                                                          │
│      for row in range(rows):                            │
│          for col in range(cols):                        │
│              if grid[row][col] == '1':                  │
│                  island_count += 1                       │
│                  # BFS to sink entire island            │
│                  queue = [(row, col)]                   │
│                  grid[row][col] = '0'                   │
│                                                          │
│                  while queue:                            │
│                      r, c = queue.pop(0)                │
│                      for dr, dc in directions:          │
│                          nr, nc = r + dr, c + dc       │
│                          if (0 <= nr < rows and         │
│                              0 <= nc < cols and         │
│                              grid[nr][nc] == '1'):      │
│                              grid[nr][nc] = '0'         │
│                              queue.append((nr, nc))     │
│                                                          │
│      return island_count                                 │
└──────────────────────────────────────────────────────────┘
```

---

## The Readability Checklist

```
Before submitting your solution, check:

  ☐ Can the interviewer trace through it in 60 seconds?
  ☐ Are variable names descriptive?
  ☐ Is there logical grouping with blank lines?
  ☐ Are complex conditions given names or extracted?
  ☐ Are there brief comments for non-obvious logic?
  ☐ Is indentation consistent?
  ☐ Are there no unnecessarily clever one-liners?
  ☐ Could a teammate understand this without explanation?
```

---

## Language-Specific Idioms (Be Idiomatic)

```
PYTHON — Use These:
  enumerate() instead of range(len())
  zip() for parallel iteration
  list comprehensions for simple transforms
  defaultdict for counting
  f-strings for formatting
  in operator for membership

JAVA — Use These:
  Enhanced for-each loops
  StringBuilder for string concat
  Map.getOrDefault()
  Arrays.sort() with comparators
  var for local type inference

C++ — Use These:
  Range-based for loops
  auto for type inference
  STL algorithms (sort, find, count)
  emplace_back over push_back
  unordered_map over map when order doesn't matter
```

---

## Summary Table

| Readability Factor | Bad | Good |
|-------------------|-----|------|
| Spacing | Wall of code | Logical groups with blank lines |
| Comments | `i += 1 # increment i` | `# Skip duplicates` |
| Conditions | `if x>0 and x<n and...` | `if is_valid(x, y):` |
| Cleverness | Nested comprehensions | Simple loops |
| Naming | `f`, `g`, `ct` | `num_islands`, `grid`, `count` |
| Consistency | Mixed styles | One style throughout |

---

## Quick Revision Questions

1. **What's more important in interviews — clever code or clear code?**
   - Clear code. Clever one-liners are harder to debug and harder for the interviewer to verify. "Clear is better than clever"

2. **How should you use comments in interview code?**
   - Explain WHY, not WHAT. Comment non-obvious logic like "skip duplicates to avoid repeated results." Don't comment `i += 1`

3. **What's the purpose of blank lines in code?**
   - Group logically related lines together — initialization, main logic, result construction. Makes code scannable

4. **How should complex conditions be handled?**
   - Extract into named boolean variables (`is_valid`, `is_in_bounds`) or helper functions. Makes the if-statement read like English

5. **What makes code "idiomatic"?**
   - Using language-specific features appropriately: `enumerate()` in Python, enhanced for-loops in Java, range-based for in C++

6. **How can you quickly check if your code is readable?**
   - Ask: "Could someone trace through this in 60 seconds?" If not, simplify variable names, add spacing, and extract complex logic

---

[← Previous: Error Handling](05-error-handling.md) | [Next: Unit 6 — Generating Test Cases →](../06-Testing-Your-Solution/01-generating-test-cases.md)

---
[← Back to README](../README.md)
