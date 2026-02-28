# Chapter 6.4: Dry Run Technique

```
╔══════════════════════════════════════════════════════════╗
║            DRY RUN TECHNIQUE                             ║
║   Executing your code mentally with precision            ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

A dry run is **manually executing your code step-by-step**, tracking variables and state changes. It's the single most effective technique for verifying correctness in interviews. Master this skill and you'll catch bugs before the interviewer does.

---

## The Dry Run Process

```
┌──────────────────────────────────────────────────────────┐
│  HOW TO DRY RUN:                                         │
│                                                          │
│  1. Choose a SMALL test case (3-5 elements)             │
│  2. Create a variable tracking table                    │
│  3. Execute each line in your head/paper                │
│  4. Update variables after EACH step                    │
│  5. Verify the final output matches expected            │
│                                                          │
│  RULES:                                                  │
│  ✗ Don't skip steps ("this part obviously works")      │
│  ✗ Don't run it on the SAME example you designed for   │
│  ✓ Execute exactly what the CODE says, not what you    │
│    THINK it should do                                    │
│  ✓ Track ALL changing variables                         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Dry Run Example 1: Two Pointers

```
CODE:
  def two_sum_sorted(nums, target):
      left, right = 0, len(nums) - 1
      while left < right:
          current_sum = nums[left] + nums[right]
          if current_sum == target:
              return [left, right]
          elif current_sum < target:
              left += 1
          else:
              right -= 1
      return [-1, -1]

INPUT: nums = [1, 3, 5, 7, 11], target = 10

TRACE TABLE:
┌──────┬──────┬───────┬─────────────┬────────────────────┐
│ Step │ left │ right │ current_sum │ Action             │
├──────┼──────┼───────┼─────────────┼────────────────────┤
│ Init │  0   │   4   │     —       │ Start              │
│  1   │  0   │   4   │ 1+11 = 12  │ 12>10 → right -= 1│
│  2   │  0   │   3   │ 1+7  = 8   │ 8<10  → left += 1 │
│  3   │  1   │   3   │ 3+7  = 10  │ 10==10 → FOUND!   │
└──────┴──────┴───────┴─────────────┴────────────────────┘

OUTPUT: [1, 3] ✓ (indices of 3 and 7)
```

---

## Dry Run Example 2: Sliding Window

```
CODE:
  def max_sum_subarray(nums, k):
      window_sum = sum(nums[:k])
      max_sum = window_sum
      for i in range(k, len(nums)):
          window_sum += nums[i] - nums[i - k]
          max_sum = max(max_sum, window_sum)
      return max_sum

INPUT: nums = [2, 1, 5, 1, 3, 2], k = 3

TRACE TABLE:
┌──────┬───┬────────────┬─────────────────┬─────────┐
│ Step │ i │ window_sum │ Calculation     │ max_sum │
├──────┼───┼────────────┼─────────────────┼─────────┤
│ Init │ — │ 2+1+5 = 8  │ Initial window  │    8    │
│  1   │ 3 │ 8+1-2 = 7  │ Add 1, remove 2 │    8    │
│  2   │ 4 │ 7+3-1 = 9  │ Add 3, remove 1 │    9    │
│  3   │ 5 │ 9+2-5 = 6  │ Add 2, remove 5 │    9    │
└──────┴───┴────────────┴─────────────────┴─────────┘

Windows:  [2,1,5]  [1,5,1]  [5,1,3]  [1,3,2]
Sums:        8        7        9        6

OUTPUT: 9 ✓ (subarray [5, 1, 3])
```

---

## Dry Run Example 3: BFS

```
CODE:
  def bfs(graph, start):
      visited = {start}
      queue = [start]
      order = []
      while queue:
          node = queue.pop(0)
          order.append(node)
          for neighbor in graph[node]:
              if neighbor not in visited:
                  visited.add(neighbor)
                  queue.append(neighbor)
      return order

INPUT: graph = {A:[B,C], B:[A,D], C:[A,D], D:[B,C]}
       start = A

TRACE TABLE:
┌──────┬──────┬─────────────┬─────────────────┬─────────┐
│ Step │ node │ queue       │ visited         │ order   │
├──────┼──────┼─────────────┼─────────────────┼─────────┤
│ Init │  —   │ [A]         │ {A}             │ []      │
│  1   │  A   │ [B, C]      │ {A, B, C}       │ [A]     │
│  2   │  B   │ [C, D]      │ {A, B, C, D}    │ [A,B]   │
│  3   │  C   │ [D]         │ {A, B, C, D}    │ [A,B,C] │
│  4   │  D   │ []          │ {A, B, C, D}    │ [A,B,C,D│
└──────┴──────┴─────────────┴─────────────────┴─────────┘

OUTPUT: [A, B, C, D] ✓ (BFS order)
```

---

## How to Present a Dry Run in Interview

```
SPEAKING TEMPLATE:

"Let me trace through with nums = [1, 3, 5, 7], target = 10.

 Starting with left = 0, right = 3.
 
 First iteration: sum = nums[0] + nums[3] = 1 + 7 = 8.
 8 is less than 10, so I move left to 1.
 
 Second iteration: sum = nums[1] + nums[3] = 3 + 7 = 10.
 10 equals our target — return [1, 3]. ✓
 
 Great, that matches the expected output."

TIPS:
├── Point to your code as you trace (if whiteboard)
├── Say variable VALUES not just names
├── State the COMPARISON and OUTCOME clearly
├── Keep it under 2 minutes for simple problems
└── Skip obvious steps for complex code
```

---

## Common Dry Run Mistakes

```
MISTAKE 1: Tracing what you THINK the code does
  Instead: Trace what the code ACTUALLY does
  Execute each line literally

MISTAKE 2: Skipping loop iterations
  "This loop obviously processes everything..."
  Instead: Show at least 2-3 iterations

MISTAKE 3: Not tracking all variables
  If a variable changes, track it
  Missing one variable = missing the bug

MISTAKE 4: Using too large an example
  Don't trace through n = 20
  Use n = 3-5 to keep it manageable

MISTAKE 5: Not verifying the result
  After tracing, compare output to expected
  "My code returns [1, 3], which matches ✓"
```

---

## Summary Table

| Aspect | Recommendation |
|--------|---------------|
| Test case size | 3-5 elements |
| Variables to track | All that change |
| Iterations to show | 2-3 minimum |
| Time budget | 1-2 minutes |
| Presentation | Verbalize each step |
| Verification | Compare output to expected |

---

## Quick Revision Questions

1. **What size test case should you use for a dry run?**
   - 3-5 elements — small enough to trace quickly, large enough to exercise the logic

2. **What's the most important rule of dry running?**
   - Execute what the code ACTUALLY says, not what you THINK it does. Read each line literally

3. **What variables should you track?**
   - All variables that change: loop counters, pointers, sums, collections (hash maps, arrays, queues)

4. **How long should a dry run take in an interview?**
   - 1-2 minutes for simple problems, up to 3 minutes for complex ones. If it takes longer, your test case is too large

5. **Should you dry run the same example you used to design the algorithm?**
   - Ideally no — use a different example to avoid confirmation bias and truly test your code

6. **What do you do after the dry run completes?**
   - Compare the output to the expected answer and say "This matches the expected output ✓" or identify the discrepancy if it doesn't match

---

[← Previous: Debugging Strategies](03-debugging-strategies.md) | [Next: Common Bugs to Avoid →](05-common-bugs-to-avoid.md)

---
[← Back to README](../README.md)
