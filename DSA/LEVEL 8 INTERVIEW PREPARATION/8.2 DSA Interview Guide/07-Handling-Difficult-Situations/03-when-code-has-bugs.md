# Chapter 7.3: When Code Has Bugs

```
╔══════════════════════════════════════════════════════════╗
║          WHEN CODE HAS BUGS                              ║
║   Stay calm, debug systematically, communicate clearly   ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Your code doesn't produce the right output. The interviewer is watching. This is a critical moment — how you debug reveals more about your engineering skill than the initial code. This chapter provides a systematic debugging approach for interview settings.

---

## The CALM Debugging Framework

```
C — Check the error type (wrong output? crash? infinite loop?)
A — Analyze where the problem occurs (narrow the scope)
L — Locate the exact line with a trace
M — Make the minimal fix

Time budget: 3-5 minutes maximum for debugging.
```

---

## Step 1: Check the Error Type

```
┌──────────────────┬──────────────────────────────────────┐
│ Symptom           │ Probable Cause                       │
├──────────────────┼──────────────────────────────────────┤
│ Wrong answer      │ Logic error, off-by-one, wrong init │
│ Index out of      │ Loop bounds, empty input not handled│
│ bounds            │                                      │
│ Null/None error   │ Missing base case, empty check      │
│ Infinite loop     │ Loop variable not progressing        │
│ Stack overflow    │ Missing/wrong recursion base case    │
│ Partial correct   │ Edge case not covered                │
│ TLE (too slow)    │ Wrong complexity (see Chapter 7.2)   │
└──────────────────┴──────────────────────────────────────┘

SAY: "The output is [X] but expected [Y]. Let me trace
through to find where it diverges."
```

---

## Step 2: Narrow the Scope

```
STRATEGY: Binary search for bugs

Don't read every line. Instead:
  1. Check output of first half of code → correct?
  2. If yes → bug is in second half
  3. If no  → bug is in first half
  4. Repeat until bug is located

EXAMPLE:
  def merge_sort(arr):
      if len(arr) <= 1: return arr
      mid = len(arr) // 2
      left = merge_sort(arr[:mid])     ← Check: is left correct?
      right = merge_sort(arr[mid:])    ← Check: is right correct?
      return merge(left, right)        ← If both correct, bug in merge

  "Left half sorts correctly. Right half sorts correctly.
   So the bug must be in my merge function."
```

---

## Step 3: Trace the Bug

```
TRACE TABLE APPROACH:
  Pick the SMALLEST failing input.
  Create a variable trace table.
  Walk through line by line.

EXAMPLE: Binary search returning wrong index
  Input: nums = [1, 3, 5, 7], target = 5

  | Step | left | right | mid | nums[mid] | Action      |
  |------|------|-------|-----|-----------|-------------|
  | 1    | 0    | 3     | 1   | 3         | 3 < 5: L=2 |
  | 2    | 2    | 3     | 2   | 5         | Found! ✓    |

  If it returns wrong index, trace reveals which
  comparison or update is wrong.
```

---

## Step 4: Make the Minimal Fix

```
IMPORTANT RULES:
┌─────────────────────────────────────────────────────────┐
│ ✗ Don't rewrite the whole solution                      │
│ ✗ Don't add complex special-case logic                  │
│ ✓ Fix the exact line that's wrong                       │
│ ✓ Verify fix with trace                                 │
│ ✓ Check that fix doesn't break other cases              │
└─────────────────────────────────────────────────────────┘

COMMON ONE-LINE FIXES:
  Off-by-one:     < → <=  or  range(n) → range(n+1)
  Wrong init:     max_val = 0 → max_val = float('-inf')
  Missing check:  Add "if not arr: return []"
  Wrong operator: += → -=  or  and → or
  Wrong variable: left → right (typo)
  Missing return: Add return statement
```

---

## Common Bug Scenarios and Fixes

```
SCENARIO 1: Off-by-one in loop
  ✗ for i in range(1, n):      ← Misses index 0
  ✓ for i in range(n):

SCENARIO 2: Wrong comparison direction
  ✗ if nums[mid] > target:
      left = mid + 1            ← Should decrease right
  ✓ if nums[mid] > target:
      right = mid - 1

SCENARIO 3: Forgetting to update result
  ✗ if current > max_val:
      pass                      ← Forgot to update!
  ✓ if current > max_val:
      max_val = current

SCENARIO 4: Wrong variable in return
  ✗ return left                 ← Should return result
  ✓ return result

SCENARIO 5: Mutating input unintentionally
  ✗ arr.sort()                  ← Modifies original array
  ✓ sorted_arr = sorted(arr)    ← Creates new sorted copy
```

---

## What to Say While Debugging

```
GOOD COMMUNICATION DURING DEBUGGING:

Phase 1 - Identify:
  "I see the output is wrong. Let me trace through
   with this test case..."

Phase 2 - Narrow:
  "The first half of the algorithm seems correct up
   to this point. The issue appears to be in..."

Phase 3 - Fix:
  "Ah, I see the problem — I have an off-by-one error
   on this line. It should be <= instead of <."

Phase 4 - Verify:
  "Let me quickly verify the fix with the same test
   case... yes, now it produces the correct output."

AVOID:
  ✗ Long silence while staring at code
  ✗ "I have no idea what's wrong"
  ✗ Randomly changing things hoping something works
  ✗ Starting completely over
```

---

## Debugging Decision Flowchart

```
Code doesn't work
      │
      ▼
Does it compile/run?
├── No → Syntax error
│       → Check brackets, colons, typos
│
▼ Yes
Does it crash?
├── Yes → What error?
│   ├── IndexError → Check bounds, empty input
│   ├── NullError  → Add null checks
│   └── StackOverflow → Check base case
│
▼ No crash
Is output wrong?
├── For all inputs → Core logic bug
│   → Trace smallest input line by line
│
├── For edge cases only → Missing edge handling
│   → Add specific checks for empty, single, etc.
│
└── Off by small amount → Off-by-one
    → Check loop bounds, <=  vs <, +1 vs -1
```

---

## Summary Table

| Bug Type | Detection | Typical Fix Time |
|----------|-----------|-----------------|
| Syntax error | Immediate | 10 seconds |
| Off-by-one | Wrong output by ±1 | 30 seconds |
| Wrong initialization | Fails on negative/empty | 30 seconds |
| Missing base case | Stack overflow | 1 minute |
| Logic error | Wrong output | 2-3 minutes |
| Wrong complexity | TLE / timeout | Need redesign |

---

## Quick Revision Questions

1. **What's the CALM framework for debugging?**
   - Check error type, Analyze where problem occurs, Locate exact line with trace, Make minimal fix

2. **How do you narrow down where a bug is?**
   - Binary search approach: check if output is correct at the midpoint of your code, then focus on the half that's wrong

3. **What should you do instead of rewriting your entire solution?**
   - Make the minimal fix on the exact line that's wrong, then verify with a trace

4. **What does an IndexError typically indicate?**
   - Loop bounds are wrong, or empty input isn't handled — check array access patterns and add boundary guards

5. **How should you communicate while debugging?**
   - Narrate your process: identify the symptom, state what you'll check, trace through the code aloud, explain the fix, verify it

6. **What's the time budget for debugging in an interview?**
   - 3-5 minutes maximum. If you can't fix it in that time, explain what you think is wrong and ask the interviewer for guidance

---

[← Previous: When Solution Is Too Slow](02-when-solution-is-too-slow.md) | [Next: When Question Is Unclear →](04-when-question-is-unclear.md)

---
[← Back to README](../README.md)
