# Chapter 4.3: Space vs Time Trade-Offs

```
╔══════════════════════════════════════════════════════════╗
║         SPACE VS TIME TRADE-OFFS                         ║
║   The fundamental engineering balance                    ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

The space-time trade-off is the most fundamental concept in algorithm design. Almost every optimization involves **spending more memory to save time, or spending more time to save memory**. Mastering this trade-off is essential for interview success.

---

## The Trade-Off Spectrum Visualized

```
          MORE TIME ◄─────────────────────► LESS TIME
          LESS SPACE                        MORE SPACE

  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
  │ O(n²)   │  │ O(nlogn)│  │  O(n)   │  │  O(1)   │
  │  time   │  │  time   │  │  time   │  │  time   │
  │ O(1)    │  │ O(1)    │  │  O(n)   │  │  O(n²)  │
  │  space  │  │  space  │  │  space  │  │  space  │
  └─────────┘  └─────────┘  └─────────┘  └─────────┘
   Brute Force   Sort First   Hash Map    Precompute
                                           All Answers
```

---

## Classic Trade-Off Examples

### Example 1: Two Sum

```
PROBLEM: Find two numbers that add to target

APPROACH A — Time Heavy, Space Light:
┌──────────────────────────────────────────────┐
│  for i in range(n):                          │
│      for j in range(i+1, n):                 │
│          if nums[i] + nums[j] == target:     │
│              return [i, j]                   │
│                                              │
│  Time: O(n²)  ████████████████████          │
│  Space: O(1)  █                              │
└──────────────────────────────────────────────┘

APPROACH B — Balanced:
┌──────────────────────────────────────────────┐
│  seen = {}                                   │
│  for i, num in enumerate(nums):              │
│      complement = target - num               │
│      if complement in seen:                  │
│          return [seen[complement], i]        │
│      seen[num] = i                           │
│                                              │
│  Time: O(n)   ████                           │
│  Space: O(n)  ████                           │
└──────────────────────────────────────────────┘

VERDICT: Approach B wins in most cases.
We "spend" O(n) space to "save" O(n) time steps.
```

### Example 2: Fibonacci

```
APPROACH A — Exponential Time, Minimal Space:
┌──────────────────────────────────────────────┐
│  def fib(n):                                 │
│      if n <= 1: return n                     │
│      return fib(n-1) + fib(n-2)             │
│                                              │
│  Time: O(2ⁿ)  ████████████████████████████  │
│  Space: O(n)   ████  (call stack)            │
└──────────────────────────────────────────────┘

APPROACH B — Linear Time, Linear Space (Memoization):
┌──────────────────────────────────────────────┐
│  memo = {}                                   │
│  def fib(n):                                 │
│      if n in memo: return memo[n]            │
│      if n <= 1: return n                     │
│      memo[n] = fib(n-1) + fib(n-2)         │
│      return memo[n]                          │
│                                              │
│  Time: O(n)   ████                           │
│  Space: O(n)  ████                           │
└──────────────────────────────────────────────┘

APPROACH C — Linear Time, Constant Space:
┌──────────────────────────────────────────────┐
│  def fib(n):                                 │
│      a, b = 0, 1                             │
│      for _ in range(n):                      │
│          a, b = b, a + b                     │
│      return a                                │
│                                              │
│  Time: O(n)  ████                            │
│  Space: O(1) █  ← Best of both worlds!      │
└──────────────────────────────────────────────┘
```

### Example 3: Range Sum Queries

```
PROBLEM: Answer multiple "sum from index l to r" queries

APPROACH A — No preprocessing:
┌──────────────────────────────────────────────┐
│  def query(arr, l, r):                       │
│      return sum(arr[l:r+1])                  │
│                                              │
│  Build: O(1)                                 │
│  Query: O(n) per query                       │
│  Space: O(1)                                 │
│  Q queries total: O(Q × n)                   │
└──────────────────────────────────────────────┘

APPROACH B — Prefix sum preprocessing:
┌──────────────────────────────────────────────┐
│  # Build prefix sum array                    │
│  prefix = [0] * (n + 1)                      │
│  for i in range(n):                          │
│      prefix[i+1] = prefix[i] + arr[i]       │
│                                              │
│  def query(l, r):                            │
│      return prefix[r+1] - prefix[l]          │
│                                              │
│  Build: O(n)                                 │
│  Query: O(1) per query!                      │
│  Space: O(n)                                 │
│  Q queries total: O(n + Q)                   │
└──────────────────────────────────────────────┘

TRADE-OFF:
  If Q = 1   → Approach A better (no build cost)
  If Q = 100 → Approach B much better
  If Q > n   → Approach B dramatically better
```

---

## Space Optimization Techniques

```
TECHNIQUE 1: ROLLING ARRAY (DP Space Reduction)
┌──────────────────────────────────────────────────────────┐
│  BEFORE: 2D DP table — O(n × m) space                  │
│                                                          │
│  dp[i][j] only depends on dp[i-1][...] row              │
│                                                          │
│  AFTER: Keep only 2 rows — O(m) space                   │
│                                                          │
│     [prev row] ──→ compute ──→ [curr row]               │
│         ↑                          │                     │
│         └──── swap rows ───────────┘                     │
│                                                          │
│  EXAMPLE: Longest Common Subsequence                     │
│  Full table: O(n × m) space                             │
│  Rolling: O(min(n, m)) space                            │
└──────────────────────────────────────────────────────────┘

TECHNIQUE 2: BIT MANIPULATION (Replace arrays with bits)
┌──────────────────────────────────────────────────────────┐
│  Track which elements we've seen:                       │
│                                                          │
│  BEFORE: seen = set()     → O(n) space                  │
│  AFTER:  seen = 0 (bitmask)→ O(1) space (if n ≤ 32)   │
│                                                          │
│  seen |= (1 << element)   # Add element                 │
│  if seen & (1 << element) # Check element               │
└──────────────────────────────────────────────────────────┘

TECHNIQUE 3: IN-PLACE MODIFICATION
┌──────────────────────────────────────────────────────────┐
│  Use the input array itself to store state               │
│                                                          │
│  BEFORE: visited = set()                                │
│  AFTER:  Mark visited by negating: nums[i] *= -1        │
│                                                          │
│  BEFORE: count = {}                                     │
│  AFTER:  Use array indices as hash keys                 │
│          nums[nums[i] % n] += n                         │
│                                                          │
│  ⚠ Only do this if problem allows input modification   │
└──────────────────────────────────────────────────────────┘
```

---

## Decision Framework

```
WHEN TO CHOOSE TIME OVER SPACE:
┌────────────────────────────────────────────┐
│ ✓ Large input size (n > 10⁴)             │
│ ✓ Real-time systems (latency matters)     │
│ ✓ Interview default (optimize time first) │
│ ✓ Server applications (RAM is cheap)      │
└────────────────────────────────────────────┘

WHEN TO CHOOSE SPACE OVER TIME:
┌────────────────────────────────────────────┐
│ ✓ Embedded systems (limited memory)       │
│ ✓ Very large data (doesn't fit in memory) │
│ ✓ Mobile applications (limited RAM)       │
│ ✓ When time difference is negligible      │
└────────────────────────────────────────────┘

IN INTERVIEWS:
"In most cases, I'd optimize for time since memory
 is relatively cheap. But if the interviewer mentions
 memory constraints, I'd switch to an in-place approach."
```

---

## Summary Table

| Scenario | Spend Time | Spend Space | Best Choice |
|----------|-----------|-------------|-------------|
| Two Sum | O(n²), O(1) | O(n), O(n) | Spend space (hash map) |
| Fibonacci | O(2ⁿ), O(n) | O(n), O(n) | Spend space (memo) or O(n),O(1) |
| Range queries | O(Qn), O(1) | O(n+Q), O(n) | Depends on Q |
| Sorting | O(n²), O(1) | O(n log n), O(n) | Usually spend space |
| DP problems | — | O(n²) table | Reduce with rolling array |

---

## Quick Revision Questions

1. **What is the space-time trade-off?**
   - Using extra memory (hash maps, precomputation) to reduce computation time, or accepting slower algorithms to reduce memory usage

2. **Which should you optimize first in interviews — time or space?**
   - Time, by default. Most interviewers prioritize time complexity unless they specifically ask about memory optimization

3. **What is the rolling array technique?**
   - Reducing DP space from O(n×m) to O(m) by only keeping the current and previous rows, since dp[i] only depends on dp[i-1]

4. **Give an example where preprocessing saves time.**
   - Prefix sum array: O(n) build time enables O(1) range sum queries instead of O(n) per query

5. **When should you optimize for space instead of time?**
   - Embedded systems, very large data that doesn't fit in memory, mobile apps, or when the interviewer explicitly mentions memory constraints

6. **How does the Two Sum problem illustrate the trade-off?**
   - Brute force uses O(1) space but O(n²) time. A hash map uses O(n) space but reduces time to O(n) — spending space for massive time savings

---

[← Previous: Explaining Complexity](02-explaining-complexity.md) | [Next: Optimization Discussion →](04-optimization-discussion.md)

---
[← Back to README](../README.md)
