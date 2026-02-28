# Chapter 10.1: Not Clarifying Requirements

```
╔══════════════════════════════════════════════════════════╗
║          NOT CLARIFYING REQUIREMENTS                     ║
║   The #1 mistake that leads to solving the wrong problem ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

The single most common interview mistake is **jumping into solving before fully understanding the problem**. Candidates who skip clarification often solve the wrong problem entirely, wasting precious time and demonstrating poor engineering habits.

---

## Why This Mistake Is So Dangerous

```
IMPACT CHAIN:
  Skip clarification
    → Make wrong assumption
      → Solve wrong problem
        → Realize at minute 30
          → Not enough time to restart
            → REJECTION

INTERVIEWER'S VIEW:
  "This candidate didn't ask any questions.
   In a real job, they would build the wrong feature."
   → Red flag for engineering maturity
```

---

## What Goes Wrong Without Clarification

```
SCENARIO 1: Wrong output format
  Problem: "Find duplicates in an array"
  You assumed: Return list of duplicates
  Expected: Return true/false if any duplicate exists
  → WASTED 20 minutes building wrong solution

SCENARIO 2: Wrong constraint assumption
  Problem: "Sort the array"
  You assumed: General sorting (O(n log n))
  Actual: Values are 0-9 only → counting sort O(n)
  → Missed the optimal approach

SCENARIO 3: Wrong input type
  Problem: "Reverse the string"
  You assumed: ASCII characters only
  Actual: Unicode with multi-byte characters
  → Solution breaks on test cases

SCENARIO 4: Ambiguous definition
  Problem: "Find the longest palindromic substring"
  You assumed: Return the string itself
  Expected: Return the LENGTH of the string
  → Output format is completely wrong
```

---

## The 5-Question Minimum

```
BEFORE CODING, ALWAYS ASK AT LEAST:

  1. INPUT:  "What are the constraints on input size?"
  2. VALUES: "Can values be negative? Zero? Duplicates?"
  3. OUTPUT: "What exactly should I return?"
  4. EDGE:   "What should happen with empty input?"
  5. UNIQUE: "Is there always exactly one valid answer?"

THESE 5 QUESTIONS TAKE 2 MINUTES
AND PREVENT 20 MINUTES OF WASTED WORK.
```

---

## How to Build the Clarification Habit

```
PRACTICE TECHNIQUE:
  Before every LeetCode problem, HIDE the constraints
  and examples. Write down:
  
  1. What questions would I ask?
  2. What assumptions am I making?
  3. What could go wrong if I'm wrong?
  
  THEN reveal the constraints and check.
  
  Were you right? What did you miss?
  This builds the habit of questioning assumptions.

IN MOCK INTERVIEWS:
  Ask your mock interviewer to give VAGUE problems.
  Practice the full clarification phase.
  Get feedback on your questions.
```

---

## Summary Table

| Mistake | Consequence | Prevention |
|---------|------------|------------|
| Skip all questions | Solve wrong problem | 5-question minimum |
| Assume output format | Wrong return type | "What should I return?" |
| Assume input range | Miss optimization | "What's the input size?" |
| Assume no edge cases | Crashes on edge input | "What about empty input?" |
| Assume single answer | Miss multiple results | "Multiple answers possible?" |

---

## Quick Revision Questions

1. **Why is not clarifying the #1 interview mistake?**
   - It leads to solving the wrong problem entirely, wasting 20+ minutes with no way to recover

2. **What are the 5 essential questions to always ask?**
   - Input constraints, value ranges, exact output format, edge case behavior, and uniqueness of answer

3. **How long should clarification take?**
   - About 2-3 minutes — a small investment that prevents catastrophic misunderstandings

4. **How does skipping clarification look to the interviewer?**
   - It signals poor engineering habits — in a real job, this person would build the wrong feature

5. **How do you practice the clarification habit?**
   - Hide problem constraints before reading, write your questions first, then reveal and check what you missed

6. **What's the worst case scenario of not clarifying?**
   - Realizing you solved the wrong problem at minute 30 with not enough time to restart

---

[← Previous: Mock Interviews](../09-Practice-Strategies/06-mock-interviews.md) | [Next: Jumping to Code →](02-jumping-to-code.md)

---
[← Back to README](../README.md)
