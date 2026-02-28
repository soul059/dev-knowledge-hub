# Chapter 5.2: Variable Naming

```
╔══════════════════════════════════════════════════════════╗
║             VARIABLE NAMING                              ║
║   Names that make code self-documenting                  ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Variable naming is one of the simplest ways to improve code quality. Good names make code **self-documenting** — the reader understands what's happening without comments. In interviews, clear names show engineering discipline and make your code easier for the interviewer to follow.

---

## Naming Rules

```
┌──────────────────────────────────────────────────────────┐
│  THE NAMING RULES:                                       │
│                                                          │
│  1. DESCRIPTIVE: Name describes the data it holds       │
│     BAD:  x, d, temp                                    │
│     GOOD: max_sum, char_count, prev_node                │
│                                                          │
│  2. APPROPRIATE LENGTH:                                  │
│     - Loop counters: i, j, k (OK — universally known)  │
│     - Local variables: 2-4 words                        │
│     - Functions: verb + noun                            │
│                                                          │
│  3. CONSISTENT CONVENTION:                              │
│     Python: snake_case  (max_profit, is_valid)          │
│     Java:   camelCase   (maxProfit, isValid)            │
│     C++:    snake_case or camelCase (pick one)          │
│                                                          │
│  4. NO ABBREVIATIONS (usually):                         │
│     BAD:  cnt, idx, arr, len, str, val                  │
│     GOOD: count, index, nums, length, text, value       │
│     OK:   i, j, n, m (conventional in DSA)              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Context-Specific Naming Guide

```
ARRAYS AND LISTS:
  BAD:  a, arr, l, lst
  GOOD: nums, prices, heights, intervals, matrix

INDICES AND POINTERS:
  BAD:  x, y, p
  GOOD: left, right, slow, fast, start, end
  OK:   i, j, k (for simple loop counters)

HASH MAPS:
  BAD:  d, m, h
  GOOD: char_count, index_map, seen, frequency
  TIP:  Name by what it maps: value_to_index

RESULTS:
  BAD:  r, res, ans
  OK:   result, answer (acceptable in interviews)
  GOOD: max_profit, longest_length, valid_pairs

BOOLEANS:
  BAD:  flag, check, status
  GOOD: is_valid, has_cycle, can_reach, found

COUNTERS:
  BAD:  c, cnt, x
  GOOD: count, total, frequency, depth, level

TREE NODES:
  BAD:  n, t, x
  GOOD: node, root, current, parent, left_child

GRAPH:
  BAD:  g, v, e
  GOOD: graph, vertex, neighbor, visited, distance
```

---

## Real Interview Code Comparison

```
BAD NAMING:
┌──────────────────────────────────────────────────────────┐
│  def f(s):                                               │
│      d = {}                                              │
│      l = 0                                               │
│      a = 0                                               │
│      for r in range(len(s)):                            │
│          if s[r] in d:                                   │
│              l = max(l, d[s[r]] + 1)                    │
│          d[s[r]] = r                                     │
│          a = max(a, r - l + 1)                          │
│      return a                                            │
│                                                          │
│  ✗ What does this do? Hard to tell.                     │
└──────────────────────────────────────────────────────────┘

GOOD NAMING:
┌──────────────────────────────────────────────────────────┐
│  def longest_substring_without_repeating(s):             │
│      last_seen = {}     # char -> last index            │
│      window_start = 0                                    │
│      max_length = 0                                      │
│                                                          │
│      for window_end in range(len(s)):                   │
│          char = s[window_end]                            │
│          if char in last_seen:                           │
│              window_start = max(window_start,            │
│                                last_seen[char] + 1)     │
│          last_seen[char] = window_end                    │
│          max_length = max(max_length,                    │
│                          window_end - window_start + 1) │
│      return max_length                                   │
│                                                          │
│  ✓ Crystal clear — reads like English                   │
└──────────────────────────────────────────────────────────┘
```

---

## Naming Patterns by Algorithm Type

```
TWO POINTERS:
  left, right          (array ends)
  slow, fast           (linked list cycle)
  read, write          (in-place modification)

SLIDING WINDOW:
  window_start, window_end
  char_count (frequency map)
  max_length, min_length

BFS/DFS:
  queue, stack
  visited
  current, neighbor
  level, depth

BINARY SEARCH:
  lo, hi, mid
  target
  first_pos, last_pos

DYNAMIC PROGRAMMING:
  dp (the table)
  prev, curr (rolling array)
  include, exclude
  profit, cost, count

TREE TRAVERSAL:
  root, node, parent
  left_child, right_child
  result (accumulator list)
```

---

## Common Acceptable Short Names

```
┌────────────┬──────────────────────────────────┐
│ Short Name │ Acceptable When                  │
├────────────┼──────────────────────────────────┤
│ i, j, k    │ Loop indices                     │
│ n, m       │ Array/input sizes                │
│ p, q       │ Two tree node pointers           │
│ s          │ A single string input            │
│ c          │ A single character               │
│ k          │ A constant/parameter             │
│ lo, hi     │ Binary search bounds             │
│ dx, dy     │ Direction offsets in grid         │
└────────────┴──────────────────────────────────┘

Everything else: USE DESCRIPTIVE NAMES
```

---

## Summary Table

| Naming Aspect | Bad | Good |
|--------------|-----|------|
| Hash map | `d`, `m` | `char_count`, `seen` |
| Result | `a`, `r` | `max_length`, `result` |
| Boolean | `flag` | `is_valid`, `has_cycle` |
| Pointer | `p` | `left`, `slow`, `current` |
| Counter | `c`, `cnt` | `count`, `frequency` |
| Function | `f`, `solve` | `find_longest_path` |

---

## Quick Revision Questions

1. **When is it OK to use single-letter variable names?**
   - For conventional loop counters (i, j, k), input sizes (n, m), binary search bounds (lo, hi), and single character/string inputs (c, s)

2. **What naming convention should you follow?**
   - Match the language: Python uses snake_case, Java uses camelCase. Be consistent throughout your solution

3. **How should you name hash maps?**
   - By what they map: `value_to_index`, `char_count`, `last_seen`. Not `d`, `m`, or `map`

4. **How should you name boolean variables?**
   - Use `is_`, `has_`, `can_`, `found` prefixes: `is_valid`, `has_cycle`, `can_reach`

5. **Is `result` an acceptable variable name?**
   - Yes, in interviews it's acceptable. Even better is a descriptive name like `max_profit` or `valid_pairs`

6. **What's the main benefit of good naming in interviews?**
   - Self-documenting code that the interviewer can follow without explanation, showing engineering maturity and making evaluation easier

---

[← Previous: Clean Code Principles](01-clean-code-principles.md) | [Next: Function Structure →](03-function-structure.md)

---
[← Back to README](../README.md)
