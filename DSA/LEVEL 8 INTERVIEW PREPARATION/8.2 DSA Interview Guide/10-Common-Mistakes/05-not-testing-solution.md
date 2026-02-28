# Chapter 10.5: Not Testing Solution

```
╔══════════════════════════════════════════════════════════╗
║          NOT TESTING SOLUTION                            ║
║   "It compiles" is not "it works"                        ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

You finish coding, say "I think that's correct," and wait. The interviewer thinks: "This person ships untested code." Not testing your solution is a **major missed opportunity** to demonstrate quality engineering and catch bugs before the interviewer does.

---

## Why Candidates Skip Testing

```
EXCUSES AND REALITY:
  "I'm confident in my code"          → Overconfidence
  "There's no time left"              → Poor time management
  "The interviewer will test it"      → Not your job
  "I don't know how to test on paper" → Lack of practice

WHAT INTERVIEWERS THINK:
  ┌─────────────────────────────────────────────────────┐
  │ "No testing = this person doesn't check their work" │
  │ "In production, they'd push bugs to customers"      │
  │ "Testing is a BASIC engineering practice"            │
  └─────────────────────────────────────────────────────┘
```

---

## The 3-Tier Testing Protocol

```
TIER 1: GIVEN EXAMPLES (Mandatory, 2 min)
  → Trace through with the example from the problem
  → This catches the most obvious bugs
  → If this fails, you have a major logic error

TIER 2: EDGE CASES (Highly recommended, 2 min)
  → Empty input, single element, all same values
  → These catch off-by-one and initialization bugs

TIER 3: YOUR OWN EXAMPLE (If time permits, 1 min)
  → Create an input that tests a different code path
  → This catches missed branches

MINIMUM: Always do Tier 1.
TARGET: Do Tiers 1 + 2.
IDEAL:  Do all three tiers.
```

---

## How to Test on Paper/Whiteboard

```
STEP-BY-STEP TRACE:

  Code: Two Sum
  def two_sum(nums, target):
      seen = {}
      for i, num in enumerate(nums):
          comp = target - num
          if comp in seen:
              return [seen[comp], i]
          seen[num] = i
      return []

  Test: nums = [2, 7, 11, 15], target = 9

  | i | num | comp | seen       | Action          |
  |---|-----|------|-----------|-----------------|
  | 0 | 2   | 7    | {}        | 7 not in seen   |
  |   |     |      | {2:0}     | Add 2→0         |
  | 1 | 7   | 2    | {2:0}     | 2 IN seen!      |
  |   |     |      |           | Return [0, 1] ✓ |

  SAY: "Tracing through the example: i=0, num=2,
  complement is 7, not in seen, so I add 2 with
  index 0. For i=1, num=7, complement is 2,
  which IS in seen at index 0. I return [0, 1].
  This matches the expected output."
```

---

## What to Test and How to Communicate

```
TESTING SCRIPT:

"Let me verify with the given example first..."
  [trace through]
"That gives [result], which matches. ✓"

"Now let me check an edge case — empty input..."
  [trace or reason through]
"Empty array returns [], which is correct. ✓"

"Let me also try a single element where no pair
 exists..."
  [trace through]
"Returns [], which is the correct behavior. ✓"

"I'm satisfied this solution handles the general
 case and key edge cases correctly."
```

---

## Common Bugs Found During Testing

```
BUGS YOU'LL CATCH BY TESTING:

  1. Off-by-one: Loop misses first/last element
     Caught by: Tracing through a small example

  2. Wrong return value: Returning wrong variable
     Caught by: Checking output against expected

  3. Uninitialized variable: Using before assignment
     Caught by: Step-through reveals undefined state

  4. Missing early return: Function falls through
     Caught by: Testing the "not found" case

  5. Wrong condition: < instead of <=
     Caught by: Testing boundary values

FINDING A BUG AND FIXING IT YOURSELF:
  → Demonstrates engineering maturity
  → Scores higher than if interviewer finds it
```

---

## Summary Table

| Testing Level | Time | What to Test | Bugs Caught |
|---------------|------|-------------|-------------|
| Tier 1: Examples | 2 min | Given problem examples | Major logic errors |
| Tier 2: Edge cases | 2 min | Empty, single, boundaries | Off-by-one, init |
| Tier 3: Custom | 1 min | Different code paths | Branch coverage |

---

## Quick Revision Questions

1. **Why is not testing a major mistake?**
   - It signals to the interviewer that you don't check your work — a red flag for production engineering quality

2. **What's the minimum testing you should always do?**
   - Trace through the given example (Tier 1) — this takes only 2 minutes and catches major logic errors

3. **How do you test on paper or whiteboard?**
   - Create a variable trace table: step through each line tracking all variable values and verify the output matches expected

4. **Is finding a bug during testing bad for your score?**
   - No — finding and fixing your own bug demonstrates engineering maturity and scores higher than if the interviewer finds it

5. **What edge cases should you always test?**
   - Empty input, single element, and boundary values at minimum

6. **How should you communicate while testing?**
   - Narrate each step: "For i=0, num equals 2, complement is 7, not found in our map, so we add it. Moving to i=1..."

---

[← Previous: Poor Time Management](04-poor-time-management.md) | [Next: Over-Engineering →](06-over-engineering.md)

---
[← Back to README](../README.md)
