# Chapter 10.3: Ignoring Edge Cases

```
╔══════════════════════════════════════════════════════════╗
║          IGNORING EDGE CASES                             ║
║   The small inputs that break big solutions              ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Your solution works perfectly on the example input. Then the interviewer asks "What if the array is empty?" and everything breaks. Ignoring edge cases is one of the most costly mistakes because it shows a **lack of thoroughness** — a trait interviewers value highly.

---

## Most Commonly Missed Edge Cases

```
THE BIG 5 (miss any of these = red flag):

  1. EMPTY INPUT
     [] for arrays, "" for strings, None for trees
     
  2. SINGLE ELEMENT
     [1] for arrays, "a" for strings, root-only tree
     
  3. ALL SAME VALUES
     [5, 5, 5, 5] — tests uniqueness assumptions
     
  4. NEGATIVE VALUES
     [-1, -2, -3] — breaks max=0 initialization
     
  5. MAXIMUM SIZE
     n = 10^5 or 10^6 — tests time complexity
```

---

## Edge Cases by Data Type

```
ARRAYS:
  ☐ Empty array []
  ☐ Single element [x]
  ☐ Two elements [a, b]
  ☐ All same values
  ☐ Already sorted (ascending)
  ☐ Reverse sorted (descending)
  ☐ Contains zero
  ☐ All negative values

STRINGS:
  ☐ Empty string ""
  ☐ Single character "a"
  ☐ All same characters "aaaa"
  ☐ Palindrome "aba"
  ☐ Spaces " hello world "
  ☐ Special characters

TREES:
  ☐ Null tree (None)
  ☐ Single node (root only)
  ☐ Left-skewed (like a linked list)
  ☐ Right-skewed
  ☐ Perfect binary tree
  ☐ Negative values in nodes

GRAPHS:
  ☐ Disconnected components
  ☐ Single node
  ☐ Self-loops
  ☐ Cycles
  ☐ No edges
```

---

## How Edge Cases Break Code

```
EXAMPLE: Find maximum subarray sum

  def max_subarray(nums):
      max_sum = 0            ← BUG: wrong init
      current = 0
      for num in nums:
          current += num
          max_sum = max(max_sum, current)
          if current < 0:
              current = 0
      return max_sum

  Input: [-3, -1, -5]
  Expected: -1 (the maximum among all negatives)
  Your output: 0 ← WRONG (max_sum never updated)

  FIX: max_sum = float('-inf')
```

---

## When to Handle Edge Cases

```
TWO APPROACHES:

APPROACH 1: Handle upfront (RECOMMENDED for interviews)
  def solve(nums):
      if not nums:           ← Handle immediately
          return 0
      if len(nums) == 1:     ← Handle immediately
          return nums[0]
      # Main logic follows...

APPROACH 2: Handle in main logic
  Design the algorithm so edge cases are handled
  naturally without special checks.
  
  (Harder but more elegant)

INTERVIEW TIP:
  Even if your main logic handles edge cases,
  MENTION them explicitly:
  "My solution handles empty arrays because the
   loop simply won't execute, returning the default."
```

---

## The Edge Case Conversation

```
WHEN TO BRING UP EDGE CASES:

  PHASE 1 — Clarification:
    "What should I return for empty input?"
    "Can the array contain negative numbers?"
    → Shows proactive thinking

  PHASE 2 — Before coding:
    "I'll handle empty and single-element cases
     with early returns."
    → Shows planning

  PHASE 3 — During testing:
    "Let me test with an empty array... []
     returns 0, which is correct."
    → Shows thoroughness

  NEVER:
    Wait for the interviewer to ask about edge cases.
    It should come from YOU.
```

---

## Summary Table

| Edge Case | Why It's Dangerous | How to Prevent |
|-----------|-------------------|----------------|
| Empty input | Index errors, wrong defaults | Guard clause at start |
| Single element | Off-by-one, wrong loops | Test with size 1 |
| All same values | Duplicate handling bugs | Test with [5,5,5] |
| Negative values | Wrong max/min initialization | Use inf/-inf |
| Maximum size | TLE, stack overflow | Verify complexity |

---

## Quick Revision Questions

1. **What are the Big 5 edge cases to always consider?**
   - Empty input, single element, all same values, negative values, and maximum input size

2. **When should you mention edge cases in an interview?**
   - Three times: during clarification (ask about them), before coding (state how you'll handle them), and during testing (verify them)

3. **What's the most common bug caused by ignoring edge cases?**
   - Wrong initialization (using 0 instead of -infinity for maximum tracking) causing wrong results on all-negative inputs

4. **Should you handle edge cases before or during main logic?**
   - In interviews, handle upfront with guard clauses (early returns) — it's clearer and easier to verify

5. **Why do interviewers care about edge cases?**
   - Edge case handling demonstrates thoroughness and attention to detail — key traits for production engineering

6. **What tree edge case is most commonly missed?**
   - Null/None tree input and skewed trees (behaving like linked lists) that affect performance assumptions

---

[← Previous: Jumping to Code](02-jumping-to-code.md) | [Next: Poor Time Management →](04-poor-time-management.md)

---
[← Back to README](../README.md)
