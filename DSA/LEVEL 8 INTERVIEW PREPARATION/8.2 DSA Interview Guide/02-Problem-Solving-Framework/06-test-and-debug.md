# Chapter 2.6: Test and Debug

```
╔══════════════════════════════════════════════════════════╗
║              TEST AND DEBUG                             ║
║   The final step that separates good from great         ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Many candidates write their code and say "I think that's it." Top candidates **proactively test** their solution. Testing and debugging during the interview demonstrates **rigor, confidence, and production-level thinking**. This chapter teaches you how to verify your solution systematically.

---

## The Testing Process

```
┌─────────────────────────────────────────────────────────────┐
│              TESTING WORKFLOW (5-7 min)                     │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ 1. Quick │─►│ 2. Trace │─►│ 3. Edge  │─►│ 4. State │  │
│  │ Visual   │  │ Normal   │  │ Case     │  │ Final    │  │
│  │ Review   │  │ Example  │  │ Check    │  │ Answer   │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│    (1 min)       (2-3 min)     (1-2 min)     (30 sec)     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 1: Quick Visual Code Review

```
SCAN YOUR CODE FOR COMMON BUGS:

┌─────────────────────────────────────────────────────────┐
│  CHECK                          EXAMPLE BUG             │
│  ─────                          ───────────             │
│  ☐ Return statement present?    Missing return          │
│  ☐ Loop bounds correct?         range(n) vs range(n+1) │
│  ☐ Off-by-one errors?           i < n vs i <= n        │
│  ☐ Variable initialization?     Forgetting seen = {}   │
│  ☐ Correct comparison?          = vs ==                 │
│  ☐ Edge case guards?            Empty array check       │
│  ☐ Right return type?           Returning [i,j] vs i,j │
│                                                         │
│  Time: 60 seconds — just scan, don't rewrite           │
└─────────────────────────────────────────────────────────┘
```

---

## Step 2: Trace Through a Normal Example (Dry Run)

```
EXAMPLE: Testing Two Sum implementation

Code:
def two_sum(nums, target):
    seen = {}
    for index, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], index]
        seen[num] = index
    return []

DRY RUN with nums = [2, 7, 11, 15], target = 9:

┌──────┬───────┬───────┬────────────┬──────────────────┬──────────┐
│ Step │ index │  num  │ complement │ complement       │ seen     │
│      │       │       │            │ in seen?         │ (after)  │
├──────┼───────┼───────┼────────────┼──────────────────┼──────────┤
│  1   │   0   │   2   │   9-2=7    │ 7 in {}? NO     │ {2: 0}   │
│  2   │   1   │   7   │   9-7=2    │ 2 in {2:0}? YES │ —        │
│      │       │       │            │ return [0, 1] ✓  │          │
└──────┴───────┴───────┴────────────┴──────────────────┴──────────┘

Expected: [0, 1]  ←  Matches! ✓
```

### How to Present the Dry Run

```
SAY THIS while tracing:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  "Let me trace through my code with the example input." │
│                                                         │
│  "Starting with nums = [2, 7, 11, 15], target = 9."   │
│  "Seen map is empty."                                   │
│                                                         │
│  "First iteration: index 0, num is 2."                  │
│  "Complement is 9 minus 2 equals 7."                    │
│  "Is 7 in our map? No. So we add 2 with index 0."     │
│                                                         │
│  "Second iteration: index 1, num is 7."                 │
│  "Complement is 9 minus 7 equals 2."                    │
│  "Is 2 in our map? Yes! At index 0."                   │
│  "So we return [0, 1]."                                 │
│                                                         │
│  "That matches the expected output. Let me check an     │
│   edge case..."                                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Step 3: Edge Case Testing

```
ESSENTIAL EDGE CASES TO ALWAYS CHECK:

┌─────────────────────────────────────────────────────────┐
│  DATA TYPE     EDGE CASES TO TEST                       │
│  ─────────     ──────────────────                       │
│                                                         │
│  Arrays:       • Empty array: []                        │
│                • Single element: [5]                     │
│                • Two elements: [3, 3]                    │
│                • All same: [1, 1, 1, 1]                 │
│                • Already sorted: [1, 2, 3, 4]           │
│                • Reverse sorted: [4, 3, 2, 1]           │
│                • Negative numbers: [-1, -2, 3]          │
│                • Contains zeros: [0, 0, 0]              │
│                                                         │
│  Strings:      • Empty: ""                              │
│                • Single char: "a"                        │
│                • All same: "aaaa"                        │
│                • Spaces: "  "                            │
│                • Special chars: "!@#"                    │
│                                                         │
│  Trees:        • Null/None root                         │
│                • Single node (root only)                 │
│                • Left-skewed (linked list shape)         │
│                • Right-skewed                            │
│                • Perfect balanced tree                   │
│                                                         │
│  Numbers:      • Zero                                   │
│                • Negative                                │
│                • MAX_INT / MIN_INT                       │
│                • One / -One                              │
│                                                         │
│  Graphs:       • Empty graph                            │
│                • Single node                             │
│                • Disconnected components                 │
│                • Cycles                                  │
│                • Self-loops                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Step 4: Debugging When Something Goes Wrong

```
SYSTEMATIC DEBUGGING APPROACH:

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. DON'T PANIC                                             │
│     "I see the output is wrong. Let me trace through        │
│      carefully to find where it diverges."                  │
│                                                             │
│  2. TRACE WITH THE FAILING INPUT                            │
│     Track every variable at every step                      │
│     ┌──────────────────────────────────────────────┐        │
│     │ Step │ Variable A │ Variable B │ Expected    │        │
│     │  1   │    ...     │    ...     │   ...       │        │
│     │  2   │    ...     │    ...     │   ← HERE!  │        │
│     └──────────────────────────────────────────────┘        │
│     Find the FIRST step where actual ≠ expected            │
│                                                             │
│  3. IDENTIFY THE BUG TYPE                                   │
│     • Off-by-one error?                                     │
│     • Wrong comparison operator?                            │
│     • Missing edge case?                                    │
│     • Wrong variable used?                                  │
│     • Incorrect update logic?                               │
│                                                             │
│  4. FIX AND RE-VERIFY                                       │
│     Fix the specific line, then trace again                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Common Bug Patterns

```
BUG 1: OFF-BY-ONE
┌──────────────────────────────────────────────────────────┐
│  WRONG: for i in range(len(arr) - 1)   ← misses last   │
│  RIGHT: for i in range(len(arr))                        │
│                                                          │
│  WRONG: while left < right             ← misses equal   │
│  RIGHT: while left <= right  (for binary search)        │
└──────────────────────────────────────────────────────────┘

BUG 2: WRONG INITIALIZATION
┌──────────────────────────────────────────────────────────┐
│  WRONG: max_val = 0           ← fails for negative nums │
│  RIGHT: max_val = float('-inf')                          │
│                                                          │
│  WRONG: min_val = 0           ← fails if all positive   │
│  RIGHT: min_val = float('inf')                           │
└──────────────────────────────────────────────────────────┘

BUG 3: INFINITE LOOP
┌──────────────────────────────────────────────────────────┐
│  WRONG: while left < right:                              │
│           mid = (left + right) // 2                      │
│           if arr[mid] < target:                         │
│               left = mid       ← doesn't progress!      │
│  RIGHT:       left = mid + 1   ← moves forward          │
└──────────────────────────────────────────────────────────┘

BUG 4: MODIFYING COLLECTION DURING ITERATION
┌──────────────────────────────────────────────────────────┐
│  WRONG: for item in list:                                │
│           if condition:                                  │
│               list.remove(item)  ← causes skips!        │
│  RIGHT: list = [x for x in list if not condition]       │
└──────────────────────────────────────────────────────────┘
```

---

## The Testing Checklist

```
┌──────────────────────────────────────────────────────────┐
│  BEFORE SAYING "I'M DONE" — CHECK THESE:                │
│                                                          │
│  ☐ Code compiles / no syntax errors                     │
│  ☐ Traced through 1 normal example — correct output     │
│  ☐ Checked 1-2 edge cases mentally                      │
│  ☐ Return type matches what's expected                  │
│  ☐ All code paths return a value                        │
│  ☐ Loop termination is guaranteed                       │
│  ☐ Variable names are clear                             │
│  ☐ Stated the time and space complexity                 │
│                                                          │
│  THEN SAY:                                               │
│  "I've tested with [example], verified edge case         │
│   [case], and the complexity is O(n) time, O(n) space.  │
│   I'm confident this solution is correct."               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Pro Tips for Testing

```
┌──────────────────────────────────────────────────────────┐
│  PRO TIPS                                                │
│                                                          │
│  1. Test PROACTIVELY — don't wait for interviewer       │
│     "Let me verify this with an example..."              │
│                                                          │
│  2. Choose SMALL but REVEALING test cases               │
│     [3, 2, 4] target=6 is better than                   │
│     [1,2,3,4,5,6,7,8,9,10] target=15                   │
│                                                          │
│  3. If you find a bug, CELEBRATE it                      │
│     "Good catch! Let me fix this." ← shows rigor        │
│                                                          │
│  4. Don't trace EVERY line for large examples            │
│     Focus on the tricky parts — loop boundaries,         │
│     condition checks, variable updates                   │
│                                                          │
│  5. Use "spot checking" for complex code                │
│     Check key intermediate values instead of             │
│     tracing every single step                            │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Step | What to Do | Time | Purpose |
|------|-----------|:---:|---------|
| 1 | Visual code scan | 1 min | Catch syntax and obvious bugs |
| 2 | Trace normal example | 2-3 min | Verify logic is correct |
| 3 | Check edge cases | 1-2 min | Verify boundary conditions |
| 4 | Debug if needed | 2-3 min | Fix any issues found |
| 5 | State complexity + confidence | 30 sec | Signal completion professionally |
| — | **Total** | **5-7 min** | **Demonstrate production-level rigor** |

---

## Quick Revision Questions

1. **What's the first thing to do after writing your code?**
   - Quick visual scan for common bugs: missing returns, off-by-one errors, wrong operators, uninitialized variables

2. **How should you trace through an example?**
   - Track every variable at every step in a table format, comparing actual vs expected values at each step

3. **What are the four essential edge cases for array problems?**
   - Empty array, single element, all same values, and negative numbers

4. **How should you react when you find a bug during testing?**
   - Stay calm, verbalize the issue, fix the specific line, and re-verify — finding and fixing bugs is actually a positive signal

5. **What should you say when you're done testing?**
   - "I've tested with [example], checked [edge case], the complexity is O(X) time and O(Y) space. I'm confident this is correct."

6. **What's the difference between good and great testing?**
   - Good: testing when asked. Great: proactively testing, choosing revealing test cases, and considering edge cases without prompting

---

[← Previous: Code the Solution](05-code-the-solution.md) | [Next: Unit 3 — Thinking Aloud →](../03-Communication-During-Interview/01-thinking-aloud.md)

---
[← Back to README](../README.md)
