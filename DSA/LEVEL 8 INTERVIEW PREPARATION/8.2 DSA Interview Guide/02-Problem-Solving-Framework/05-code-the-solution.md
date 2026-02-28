# Chapter 2.5: Code the Solution

```
╔══════════════════════════════════════════════════════════╗
║            CODE THE SOLUTION                            ║
║   Translating your algorithm into clean, correct code   ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

By this point, you've understood the problem, worked through examples, discussed brute force, and planned your optimized approach. Now it's time to **write the actual code**. This step is where preparation meets execution — and where clean coding habits make the difference.

---

## The Coding Workflow

```
┌─────────────────────────────────────────────────────────────┐
│              CODING WORKFLOW (20-25 min)                    │
│                                                             │
│  Step 1 ──► Step 2 ──► Step 3 ──► Step 4 ──► Step 5       │
│  Write      Fill in    Handle     Narrate    Review         │
│  skeleton   logic      edges      aloud      once           │
│  (2 min)    (10-15min) (3 min)    (ongoing)  (2 min)       │
│                                                             │
│  KEY PRINCIPLE: Write code TOP-DOWN                         │
│  Start with the overall structure, then fill in details     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 1: Write the Skeleton First

```
BEFORE writing any logic, lay out the structure:

┌─────────────────────────────────────────────────────────┐
│                                                         │
│  def two_sum(nums, target):                             │
│      # Step 1: Initialize data structure                │
│                                                         │
│      # Step 2: Iterate through array                    │
│                                                         │
│          # Step 2a: Check if complement exists           │
│                                                         │
│          # Step 2b: Store current element                │
│                                                         │
│      # Step 3: Handle no-solution case                  │
│      pass                                               │
│                                                         │
│  WHY? This gives you:                                   │
│  • A roadmap to follow                                  │
│  • Shows interviewer your plan                          │
│  • Easy to fill in one step at a time                   │
│  • Natural narration points                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Step 2: Fill in the Logic

```
NOW fill in each section with actual code:

┌─────────────────────────────────────────────────────────┐
│                                                         │
│  def two_sum(nums, target):                             │
│      # Step 1: Initialize hash map for lookups          │
│      seen = {}                                          │
│                                                         │
│      # Step 2: Iterate through array                    │
│      for index, num in enumerate(nums):                 │
│                                                         │
│          # Step 2a: Check if complement exists           │
│          complement = target - num                      │
│          if complement in seen:                         │
│              return [seen[complement], index]            │
│                                                         │
│          # Step 2b: Store current element                │
│          seen[num] = index                              │
│                                                         │
│      # Step 3: No solution found                        │
│      return []                                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Step 3: Handle Edge Cases

```
ADD edge case handling at appropriate points:

┌─────────────────────────────────────────────────────────┐
│                                                         │
│  def two_sum(nums, target):                             │
│      if not nums or len(nums) < 2:    ← EDGE CASE     │
│          return []                                      │
│                                                         │
│      seen = {}                                          │
│                                                         │
│      for index, num in enumerate(nums):                 │
│          complement = target - num                      │
│          if complement in seen:                         │
│              return [seen[complement], index]            │
│          seen[num] = index                              │
│                                                         │
│      return []   ← HANDLES "no solution" edge case     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Coding Best Practices During Interview

```
┌──────────────────────────────────────────────────────────┐
│  DO's AND DON'Ts WHILE CODING                            │
│                                                          │
│  DO ✓                                                    │
│  ├── Use meaningful variable names (not i, j, k)        │
│  ├── Write modular code (helper functions)               │
│  ├── Narrate what you're writing as you write            │
│  ├── Leave blank lines between logical sections          │
│  ├── Use language idioms (enumerate, zip, etc.)          │
│  └── Keep functions short and focused                    │
│                                                          │
│  DON'T ✗                                                │
│  ├── Write everything in one giant function              │
│  ├── Use single-letter variables (except i for index)   │
│  ├── Go silent while coding                              │
│  ├── Write comments for every line (wastes time)         │
│  ├── Try to be clever with one-liners                    │
│  └── Second-guess your approach mid-coding               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Narrating While Coding — Scripts

```
WHAT TO SAY while writing each part:

┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Writing the function signature:                        │
│  "I'll create a function that takes nums and target     │
│   and returns the indices."                             │
│                                                         │
│  Initializing:                                          │
│  "I'll use a hash map to store values I've seen         │
│   and their indices for O(1) lookups."                  │
│                                                         │
│  Writing the loop:                                      │
│  "I'll iterate through each element. For each one,      │
│   I calculate its complement — which is target minus    │
│   the current number."                                  │
│                                                         │
│  Writing the condition:                                 │
│  "If the complement is already in our hash map,         │
│   we've found our pair. I return both indices."         │
│                                                         │
│  Writing the store:                                     │
│  "Otherwise I store the current number and its index    │
│   in the hash map for future lookups."                  │
│                                                         │
│  Writing the return:                                    │
│  "If we exhaust the array without finding a pair,       │
│   we return an empty array, though the problem          │
│   guarantees a solution exists."                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Common Code Patterns to Memorize

```
PATTERN 1: HASH MAP LOOKUP
┌──────────────────────────────────┐
│  seen = {}                       │
│  for item in collection:         │
│      if target_condition in seen:│
│          return result           │
│      seen[key] = value           │
└──────────────────────────────────┘

PATTERN 2: TWO POINTERS (sorted)
┌──────────────────────────────────┐
│  left, right = 0, len(arr) - 1  │
│  while left < right:            │
│      current = arr[left]+arr[r] │
│      if current == target:      │
│          return [left, right]   │
│      elif current < target:     │
│          left += 1              │
│      else:                      │
│          right -= 1             │
└──────────────────────────────────┘

PATTERN 3: SLIDING WINDOW
┌──────────────────────────────────┐
│  left = 0                        │
│  window_state = initial          │
│  for right in range(n):         │
│      update_window(arr[right])  │
│      while window_invalid():    │
│          remove_from_window(l)  │
│          left += 1              │
│      update_answer()            │
└──────────────────────────────────┘

PATTERN 4: BFS
┌──────────────────────────────────┐
│  queue = deque([start])         │
│  visited = {start}              │
│  while queue:                   │
│      node = queue.popleft()     │
│      for neighbor in adj[node]: │
│          if neighbor not visited:│
│              visited.add(nbr)   │
│              queue.append(nbr)  │
└──────────────────────────────────┘

PATTERN 5: DFS (Recursive)
┌──────────────────────────────────┐
│  def dfs(node, visited):        │
│      if node in visited:        │
│          return                  │
│      visited.add(node)          │
│      process(node)              │
│      for nbr in adj[node]:     │
│          dfs(nbr, visited)      │
└──────────────────────────────────┘

PATTERN 6: BINARY SEARCH
┌──────────────────────────────────┐
│  lo, hi = 0, len(arr) - 1      │
│  while lo <= hi:                │
│      mid = lo + (hi - lo) // 2 │
│      if arr[mid] == target:    │
│          return mid             │
│      elif arr[mid] < target:   │
│          lo = mid + 1           │
│      else:                      │
│          hi = mid - 1           │
└──────────────────────────────────┘
```

---

## When You Make a Mistake While Coding

```
┌──────────────────────────────────────────────────────────┐
│  HOW TO HANDLE MISTAKES                                  │
│                                                          │
│  1. SYNTAX ERROR                                         │
│     "Oh, let me fix that syntax real quick."             │
│     → Fix it calmly, don't panic                        │
│                                                          │
│  2. LOGIC ERROR                                          │
│     "Wait, I think there's an off-by-one here.           │
│      Let me trace through to check."                     │
│     → Trace with a small example to verify              │
│                                                          │
│  3. WRONG APPROACH MID-CODE                              │
│     "Actually, I realize this approach won't handle      │
│      [edge case]. Let me adjust the approach."           │
│     → Don't start over — modify what you have           │
│                                                          │
│  4. STUCK ON IMPLEMENTATION DETAIL                       │
│     "I know the concept but I'm blanking on the          │
│      exact syntax. Could I use pseudocode here?"         │
│     → Most interviewers allow this                      │
│                                                          │
│  KEY: Stay calm, verbalize the issue, fix methodically   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Time Management While Coding

```
┌──────────────────────────────────────────────────────────┐
│  TIME BUDGET (for a 45-min interview)                    │
│                                                          │
│  ┌──────────────────────────────────────────────┐       │
│  │  0:00 - 5:00  │ Understand + Clarify         │       │
│  │  5:00 - 8:00  │ Examples                      │       │
│  │  8:00 - 12:00 │ Discuss Approach              │       │
│  │  12:00 - 32:00│ ████ CODE THE SOLUTION ████  │       │
│  │  32:00 - 38:00│ Test + Debug                  │       │
│  │  38:00 - 45:00│ Your questions                │       │
│  └──────────────────────────────────────────────┘       │
│                                                          │
│  WARNING SIGNS:                                          │
│  ⚠️ 20 min coding and not halfway → simplify approach   │
│  ⚠️ 25 min coding and function not complete → rush to   │
│     a working version, skip optimization                 │
│  ⚠️ 30 min coding → wrap up with whatever you have      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Step | Action | Time | Key Principle |
|------|--------|:---:|---------|
| 1 | Write skeleton with comments | 2 min | Top-down structure first |
| 2 | Fill in logic section by section | 10-15 min | One step at a time |
| 3 | Add edge case handling | 3 min | Guards at function entry |
| 4 | Narrate throughout | Ongoing | Never go silent |
| 5 | Quick review before declaring done | 2 min | Catch obvious bugs |
| — | **Total coding time** | **20 min** | **Stay within budget** |

---

## Quick Revision Questions

1. **What's the first thing you should write when starting to code?**
   - A skeleton with comments outlining each logical step — it provides a roadmap and shows the interviewer your plan

2. **Why should you narrate while coding?**
   - It shows your thought process, helps the interviewer follow along, catches your own logical errors, and demonstrates communication skills

3. **What are the six code patterns every candidate should memorize?**
   - Hash Map lookup, Two Pointers, Sliding Window, BFS, DFS, and Binary Search

4. **How should you handle a mistake you notice while coding?**
   - Stay calm, verbalize the issue ("I think there's an off-by-one here"), trace with a small example, and fix methodically

5. **What should you do if you run out of coding time?**
   - Explain what the remaining code would do, state the overall complexity, and test what you have — partial correct solutions still earn points

6. **How much time should coding take in a 45-minute interview?**
   - About 20 minutes — start around minute 12(after understanding + approach), finish by minute 32 to leave time for testing

---

[← Previous: Optimize](04-optimize.md) | [Next: Test and Debug →](06-test-and-debug.md)

---
[← Back to README](../README.md)
