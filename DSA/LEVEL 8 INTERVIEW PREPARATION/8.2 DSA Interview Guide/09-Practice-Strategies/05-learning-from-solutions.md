# Chapter 9.5: Learning from Solutions

```
╔══════════════════════════════════════════════════════════╗
║          LEARNING FROM SOLUTIONS                         ║
║   How to study editorial solutions for maximum learning  ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Reading someone else's solution is only useful if you do it **the right way**. Most people glance at the code, say "oh that's clever," and move on — learning nothing. This chapter teaches you how to extract maximum value from every solution you study.

---

## When to Read Solutions

```
READ A SOLUTION AFTER:
  ✓ You spent 20-30 min genuinely attempting the problem
  ✓ You have a brute force but can't optimize
  ✓ You identified the approach but can't implement it
  ✓ Your solution works but is much slower than optimal

DON'T READ A SOLUTION:
  ✗ As the first thing you do (before trying)
  ✗ After only 5 minutes of thinking
  ✗ If you haven't at least written pseudocode
  
  THE STRUGGLE IS WHERE LEARNING HAPPENS.
  Looking at solutions too early robs you of learning.
```

---

## The Solution Study Method (5 Steps)

```
STEP 1: READ WITHOUT CODE (2 min)
  → Read only the text explanation
  → Understand the HIGH-LEVEL approach
  → Don't look at code yet
  → Ask: "What technique is this? Why does it work?"

STEP 2: PREDICT THE CODE (3 min)
  → Based on the explanation, try to write code yourself
  → This tests if you truly understood the approach
  → If you can code it → you understood it
  → If you can't → you only understood surface-level

STEP 3: READ THE CODE (5 min)
  → Now read line by line
  → For each line, ask: "Why this, not something else?"
  → Identify:
    • Data structure choices and WHY
    • Loop structure and WHY
    • Clever tricks or optimizations

STEP 4: TRACE WITH EXAMPLE (5 min)
  → Pick a small example
  → Trace through the code step by step
  → Verify it produces the correct output
  → This cements understanding

STEP 5: RE-IMPLEMENT FROM MEMORY (10 min)
  → Close the solution entirely
  → Re-code it from scratch
  → Compare with original
  → If different, understand why
```

---

## What to Extract from Each Solution

```
EXTRACTION TEMPLATE:

PROBLEM: [Name]
TECHNIQUE: [e.g., Sliding Window + Hash Map]

KEY INSIGHT:
  What's the ONE idea that makes this solution work?
  (Usually 1-2 sentences)

PATTERN:
  Where would I use this technique again?
  "When I see [X], I should think [Y]"

COMPLEXITY:
  Time: O(?) — Why?
  Space: O(?) — Why?

CLEVER TRICK (if any):
  Specific implementation detail that's worth remembering
  
EXAMPLE:
  Problem: Longest Substring Without Repeating Characters
  Technique: Sliding Window + Hash Map
  Key Insight: Use hash map to track last position of each
               character. When duplicate found, jump left
               pointer past the previous occurrence.
  Pattern: "When tracking unique elements in a window"
  Time: O(n), Space: O(min(n, alphabet_size))
  Trick: left = max(left, last_seen[char] + 1) — the max
         prevents left from moving backwards
```

---

## Comparing Multiple Solutions

```
MOST PROBLEMS HAVE 2-4 DIFFERENT APPROACHES.
Study at least 2 for each problem.

COMPARISON TABLE:
┌──────────────┬───────────────┬───────────────┬──────────┐
│ Approach      │ Time          │ Space         │ Tradeoff │
├──────────────┼───────────────┼───────────────┼──────────┤
│ Brute Force   │ O(n²)         │ O(1)          │ Simple   │
│ Hash Map      │ O(n)          │ O(n)          │ Fast     │
│ Two Pointers  │ O(n log n)    │ O(1) / O(n)   │ No extra │
│ (after sort)  │               │ (if sort in   │ space if │
│               │               │  place)       │ in-place │
└──────────────┴───────────────┴───────────────┴──────────┘

ASK FOR EACH PAIR:
  "When would I choose approach A over approach B?"
  "What constraint would make one better than the other?"
```

---

## Learning from Wrong Approaches

```
YOUR WRONG APPROACH IS VALUABLE TOO.

AFTER SEEING THE CORRECT SOLUTION:
  1. Why didn't my approach work?
     → Off by one?
     → Wrong data structure?
     → Missing edge case?
     → Fundamentally wrong complexity?

  2. Where exactly did I diverge?
     → Was my thinking correct up to point X?
     → At what point should I have changed direction?

  3. What signal did I miss?
     → Was there a hint in the constraints?
       (n ≤ 10^5 → needs O(n) or O(n log n))
     → Was there a hint in the examples?
     → Was there a pattern I should have recognized?

  LOGGING MISTAKES IS MORE VALUABLE THAN LOGGING SUCCESSES.
```

---

## Building a Pattern Notebook

```
AFTER EVERY ~5 PROBLEMS, UPDATE YOUR NOTEBOOK:

PATTERN: Sliding Window
  ┌─────────────────────────────────────────────────────┐
  │ Trigger: "contiguous subarray/substring"            │
  │          "window of size k" or "minimum length"     │
  │                                                      │
  │ Template:                                            │
  │   left = 0                                           │
  │   for right in range(len(arr)):                     │
  │       # expand: add arr[right] to window            │
  │       while window_invalid:                          │
  │           # shrink: remove arr[left] from window    │
  │           left += 1                                  │
  │       # update answer                               │
  │                                                      │
  │ Variants:                                            │
  │   Fixed size: window = right - left + 1 == k        │
  │   Variable: shrink when constraint violated         │
  │   With hash map: track frequency/uniqueness         │
  │                                                      │
  │ Problems solved:                                     │
  │   ✓ Max Sum Subarray of Size K                      │
  │   ✓ Smallest Subarray with Sum ≥ S                  │
  │   ✓ Longest Substring Without Repeating             │
  │   ✓ Minimum Window Substring                        │
  └─────────────────────────────────────────────────────┘

  This notebook becomes your most valuable study resource.
```

---

## Summary Table

| Step | Time | What to Do |
|------|------|-----------|
| Read explanation | 2 min | Understand approach, no code |
| Predict code | 3 min | Try to code based on explanation |
| Read code | 5 min | Line-by-line analysis |
| Trace example | 5 min | Step through with real input |
| Re-implement | 10 min | Code from memory, compare |

---

## Quick Revision Questions

1. **When should you read a solution to a problem?**
   - After at least 20-30 minutes of genuine attempt, having tried brute force and possibly identified but not implemented the approach

2. **What's the most important step in studying a solution?**
   - Re-implementing from memory without looking — this tests true understanding vs surface-level recognition

3. **What should you extract from every solution?**
   - The key insight (1-2 sentences), the pattern trigger, complexity analysis, and any clever tricks worth remembering

4. **Why study multiple approaches to the same problem?**
   - Different approaches have different trade-offs; understanding when to choose each helps you handle variations and constraints

5. **How is a wrong approach valuable?**
   - It reveals your thinking gaps — identifying exactly where and why you diverged helps prevent the same mistake on similar problems

6. **What is a pattern notebook?**
   - A personal reference with trigger words, code templates, variants, and solved problem lists for each technique you've learned

---

[← Previous: Problem Variety](04-problem-variety.md) | [Next: Mock Interviews →](06-mock-interviews.md)

---
[← Back to README](../README.md)
