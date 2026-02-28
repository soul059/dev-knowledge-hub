# Chapter 6.3: Debugging Strategies

```
╔══════════════════════════════════════════════════════════╗
║          DEBUGGING STRATEGIES                            ║
║   Systematic approaches to finding and fixing bugs       ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Finding bugs under time pressure is a critical interview skill. Panicking and randomly changing code wastes time. This chapter teaches a **systematic debugging process** that efficiently identifies and fixes issues.

---

## The 4-Step Debugging Process

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  STEP 1: REPRODUCE → Find an input that fails           │
│  "Test case [1, 3, 5], target = 8 gives wrong output"  │
│                                                          │
│  STEP 2: LOCATE → Narrow down where the bug is          │
│  "The loop iterates correctly but the comparison is off"│
│                                                          │
│  STEP 3: UNDERSTAND → Figure out WHY it's wrong         │
│  "I'm comparing indices instead of values"              │
│                                                          │
│  STEP 4: FIX → Make the minimal change                  │
│  "Change nums[i] > j to nums[i] > nums[j]"            │
│                                                          │
│  → Verify fix with the same failing test case           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Debugging Technique 1: Dry Run (Manual Trace)

```
TRACE TABLE METHOD:

Code: Find if pair sums to target
  seen = {}
  for i, num in enumerate(nums):
      comp = target - num
      if comp in seen:
          return [seen[comp], i]
      seen[num] = i

Input: nums = [2, 7, 11, 15], target = 9

┌──────┬─────┬──────┬──────────────┬─────────────┐
│ Step │ num │ comp │ comp in seen?│ seen        │
├──────┼─────┼──────┼──────────────┼─────────────┤
│ i=0  │  2  │  7   │ No           │ {2: 0}      │
│ i=1  │  7  │  2   │ Yes! → [0,1]│             │
└──────┴─────┴──────┴──────────────┴─────────────┘
Result: [0, 1] ✓

WHEN TO USE: Always. This is the most reliable method.
Takes 1-2 minutes but catches almost every bug.
```

---

## Debugging Technique 2: Check Common Bug Patterns

```
BUG PATTERN CHECKLIST:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  ☐ OFF-BY-ONE ERROR                                    │
│    range(n) vs range(n-1) vs range(n+1)                │
│    < vs <= in loop condition                            │
│    Are you starting at 0 or 1?                          │
│                                                          │
│  ☐ WRONG VARIABLE                                      │
│    Using i instead of j                                 │
│    Using nums[i] instead of nums[j]                    │
│    Using left instead of right                          │
│                                                          │
│  ☐ WRONG OPERATOR                                      │
│    > instead of >=                                      │
│    and instead of or                                    │
│    + instead of -                                       │
│                                                          │
│  ☐ WRONG INITIALIZATION                                │
│    max_val = 0 (fails for negative arrays)             │
│    count = 1 (should be 0)                              │
│    result = None (should be [])                         │
│                                                          │
│  ☐ MISSING UPDATE                                      │
│    Forgot to update pointer                             │
│    Forgot to add to visited set                        │
│    Forgot to increment counter                          │
│                                                          │
│  ☐ INFINITE LOOP                                       │
│    Loop variable never changes                         │
│    Convergence condition never met                      │
│    Recursive call doesn't reduce problem size           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Debugging Technique 3: Divide and Conquer

```
When code is long, ISOLATE the bug:

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  def solve(data):                                        │
│      step1_result = process(data)    ← Check this      │
│      step2_result = transform(step1) ← Then this       │
│      return finalize(step2)          ← Then this       │
│                                                          │
│  "Let me check step1 first..."                          │
│  Is step1_result correct? Yes → bug is later            │
│  Is step1_result correct? No → bug is in process()     │
│                                                          │
│  Binary search for the bug location:                    │
│  Check the middle of your code.                         │
│  If correct at middle → bug is in second half           │
│  If wrong at middle → bug is in first half             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Debugging Technique 4: Print Debugging (Mental)

```
IN INTERVIEWS (no actual print statements needed):

"Let me trace through the key variables at each step."

MENTALLY TRACK:
  Iteration 1: left=0, right=4, sum=6      (too small)
  Iteration 2: left=1, right=4, sum=9      (too small?)
  
  WAIT — 9 equals the target! Why didn't my code return?
  
  *looks at code*
  
  "Ah, I wrote > instead of >= in my comparison.
   Let me fix that to >= ."

THIS PROCESS:
  1. Pick the failing test case
  2. Track 2-3 key variables per step
  3. Compare expected vs actual behavior
  4. The mismatch reveals the bug
```

---

## Interview Debugging Communication

```
HOW TO DEBUG OUT LOUD:

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "My test case gives [wrong output] instead of          │
│   [expected]. Let me trace through..."                  │
│                                                          │
│  "At iteration 3, left = 2 and right = 5.              │
│   The sum is 7, which is less than target 9.            │
│   So left should move right... yes, that's correct."   │
│                                                          │
│  "At iteration 4, left = 3 and right = 5.              │
│   The sum is 9, which equals the target.               │
│   But my condition says 'sum > target'...              │
│   I should be checking 'sum == target' first!"         │
│                                                          │
│  "Found it — I need to check equality before           │
│   the comparison. Let me fix that."                    │
│                                                          │
│  *makes fix*                                             │
│                                                          │
│  "Let me re-run the test case... yes, now it           │
│   correctly returns [3, 5]. ✓"                         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Technique | When to Use | Time Cost |
|-----------|------------|-----------|
| Dry run trace | First attempt — always | 1-2 min |
| Bug pattern checklist | When trace reveals discrepancy | 30 sec |
| Divide and conquer | Long/complex code | 1 min |
| Print debugging (mental) | When bug location is unclear | 1-2 min |

---

## Quick Revision Questions

1. **What's the first step when your code produces wrong output?**
   - Reproduce with a specific failing test case, then trace through your code step by step with that input

2. **What are the 4 debugging steps?**
   - Reproduce → Locate → Understand → Fix. Don't skip steps — random changes waste time

3. **What's the most common type of interview bug?**
   - Off-by-one errors: wrong loop bounds, `<` vs `<=`, starting at 0 vs 1

4. **How should you debug in front of the interviewer?**
   - Verbalize: "Let me trace through this test case. At step 3, I expect X but get Y..." — shows systematic thinking

5. **What should you check first when debugging array code?**
   - Loop bounds (off-by-one), initial values, and whether you're accessing the right variable (i vs j, nums[i] vs nums[j])

6. **How long should you spend debugging before asking for help?**
   - 2-3 minutes of systematic tracing. If you can't find the bug, describe what you've checked and ask the interviewer for a hint

---

[← Previous: Edge Cases to Consider](02-edge-cases-to-consider.md) | [Next: Dry Run Technique →](04-dry-run-technique.md)

---
[← Back to README](../README.md)
