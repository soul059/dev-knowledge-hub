# Chapter 7.5: When Time Runs Out

```
╔══════════════════════════════════════════════════════════╗
║          WHEN TIME RUNS OUT                              ║
║   Maximize your score in the final minutes               ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

You have 5 minutes left and your solution isn't complete. This happens more often than you'd think. What you do in these final minutes can make the difference between a hire and a rejection. This chapter covers **damage control strategies** that maximize your evaluation score.

---

## Time Awareness During Interview

```
TYPICAL 45-MINUTE INTERVIEW TIMELINE:
┌─────────────────────────────────────────────────────────┐
│ 0-5 min:   Understand problem, ask questions            │
│ 5-10 min:  Discuss approach, get buy-in                 │
│ 10-30 min: Code the solution                            │
│ 30-40 min: Test and debug                               │
│ 40-45 min: Discuss complexity, follow-ups               │
│                                                          │
│ WARNING SIGNS YOU'RE RUNNING LOW:                       │
│  ⚠ 15 min in and no code written                       │
│  ⚠ 25 min in and solution not half done                │
│  ⚠ 35 min in and code not working yet                  │
└─────────────────────────────────────────────────────────┘
```

---

## Strategy 1: Incomplete Code — Describe the Rest

```
IF YOUR CODE IS 70% DONE:

  def solve(nums, target):
      # Completed: preprocessing, main loop
      sorted_nums = sorted(nums)
      left, right = 0, len(sorted_nums) - 1
      
      while left < right:
          current = sorted_nums[left] + sorted_nums[right]
          if current == target:
              return [left, right]
          elif current < target:
              left += 1
          else:
              right -= 1
      
      # TODO: Handle the case where no pair found
      # I would add: return [-1, -1]
      
      # TODO: Map sorted indices back to original
      # I would store original indices alongside values
      # using a list of (value, original_index) tuples

SAY:
  "I'm running low on time. Let me describe what the
   remaining code would do: [explain logic clearly].
   The approach is correct and would run in O(n log n)."
```

---

## Strategy 2: Skip Edge Cases — Cover Them Verbally

```
WHAT TO DO:
  1. Get the main logic working for the happy path
  2. Verbally note edge cases you'd handle

SAY:
  "This works for the general case. Given more time,
   I would add these checks:
   - Empty array → return []
   - Single element → return element
   - All duplicates → [specific handling]
   I'll add a comment noting these."

ADD COMMENTS:
  def solve(nums):
      # Edge cases: empty, single element, all same
      # (would handle with early returns)
      
      # Main logic (implemented):
      ...
```

---

## Strategy 3: Brute Force Done > Optimal Incomplete

```
DECISION MATRIX:
┌────────────────────────┬────────────────────────────────┐
│ Situation               │ Best Action                    │
├────────────────────────┼────────────────────────────────┤
│ No code at all          │ Write brute force FAST         │
│ Brute force done,       │ Keep brute force, describe     │
│ optimal half-done       │ the optimal approach verbally  │
│ Optimal almost done     │ Finish optimal solution        │
│ Code done but buggy     │ Fix the bug, skip optimization │
└────────────────────────┴────────────────────────────────┘

RULE: A working brute force ALWAYS beats an incomplete
optimal solution.

INTERVIEWER'S SCORING:
  Working brute force: 60-70% credit
  Working optimal: 90-100% credit
  Incomplete code (no output): 20-30% credit
```

---

## Strategy 4: Pseudo-Code the Remainder

```
IF YOU CAN'T FINISH REAL CODE:

  def optimize_route(graph, start, end):
      # Step 1: Build adjacency list (DONE)
      adj = build_adjacency_list(graph)
      
      # Step 2: BFS to find shortest path (DONE)
      queue = deque([(start, [start])])
      visited = {start}
      
      while queue:
          node, path = queue.popleft()
          if node == end:
              return path
          for neighbor in adj[node]:
              if neighbor not in visited:
                  visited.add(neighbor)
                  queue.append((neighbor, path + [neighbor]))
      
      # Step 3: Optimize path with weights (PSEUDOCODE)
      # - Use Dijkstra's instead of BFS
      # - Replace queue with min-heap (heapq)
      # - Track distances dict: {node: min_distance}
      # - Pop minimum distance node each iteration
      # - Update distances for neighbors
      # - Return path when end node is popped
      
      return None  # No path found
```

---

## Strategy 5: The 2-Minute Summary

```
WITH 2 MINUTES LEFT, DELIVER THIS:

1. WHAT YOU HAVE (15 sec):
   "My solution handles [X] correctly using [approach]."

2. WHAT'S MISSING (15 sec):
   "I still need to handle [edge case] and [optimize Y]."

3. COMPLEXITY (30 sec):
   "Current time complexity is O(n²). With [optimization],
   it would be O(n log n)."

4. TRADE-OFFS (30 sec):
   "I chose [approach] because [reason]. An alternative
   would be [other approach] which trades [X] for [Y]."

5. TESTING (30 sec):
   "I would test with [cases]. The edge cases I'd check
   are [list]."
```

---

## Preventing Time Crunch

```
TIME MANAGEMENT TIPS:
┌─────────────────────────────────────────────────────────┐
│ 1. Set mental checkpoints:                              │
│    "If I don't have an approach by 10 min, I'll         │
│     go with brute force."                               │
│                                                          │
│ 2. Code the approach you're CONFIDENT in:               │
│    Don't chase the optimal if you can't code it fast    │
│                                                          │
│ 3. Say: "I'll start with brute force, then optimize":   │
│    This sets expectations and guarantees working code   │
│                                                          │
│ 4. Skip helper functions initially:                     │
│    Inline logic first, extract later if time permits    │
│                                                          │
│ 5. Write the skeleton first:                            │
│    Function signature → main structure → fill details   │
└─────────────────────────────────────────────────────────┘
```

---

## What NOT to Do When Time Runs Out

```
MISTAKES THAT HURT YOUR SCORE:
  ✗ Panicking and going silent
  ✗ Frantically writing code with errors
  ✗ Saying "I would have solved it with more time"
  ✗ Blaming the problem for being hard
  ✗ Apologizing excessively
  ✗ Stopping work before officially told time's up

INSTEAD:
  ✓ Stay calm and professional
  ✓ Summarize what you have and what's left
  ✓ Show awareness of the complete solution
  ✓ Demonstrate you know the path even if incomplete
```

---

## Summary Table

| Time Left | Best Strategy |
|-----------|---------------|
| 10 min | Prioritize: get working code, skip optimization |
| 5 min | Finish current function, add comments for rest |
| 2 min | Stop coding, deliver 2-minute summary |
| 0 min | Summarize approach, state complexity, thank interviewer |

---

## Quick Revision Questions

1. **What's better: incomplete optimal or complete brute force?**
   - Complete brute force — always. A working solution scores 60-70%, while incomplete code scores only 20-30%

2. **What should you do in the last 2 minutes?**
   - Stop coding and deliver a summary: what you have, what's missing, complexity, trade-offs, and what tests you'd run

3. **How do you handle code that's 70% complete when time is up?**
   - Add comments describing the remaining logic, verbally explain the approach, and state the overall complexity

4. **What's the biggest time management mistake in interviews?**
   - Spending too long on approach discussion or chasing optimal solution without writing code first

5. **How should you handle edge cases when pressed for time?**
   - Get the main logic working first, then verbally list the edge cases you'd handle and add brief comments

6. **What should you NEVER say when time runs out?**
   - "I would have solved it with more time" or blaming the problem — instead, calmly summarize what you built and what you'd do next

---

[← Previous: When Question Is Unclear](04-when-question-is-unclear.md) | [Next: Unit 8 — Must-Know Topics →](../08-Topic-Prioritization/01-must-know-topics.md)

---
[← Back to README](../README.md)
