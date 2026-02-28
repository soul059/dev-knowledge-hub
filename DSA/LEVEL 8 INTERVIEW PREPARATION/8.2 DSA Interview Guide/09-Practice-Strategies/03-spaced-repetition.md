# Chapter 9.3: Spaced Repetition

```
╔══════════════════════════════════════════════════════════╗
║          SPACED REPETITION                               ║
║   The science of remembering what you've learned         ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

You solve a problem today and forget the approach in a week. This is the **forgetting curve** — and spaced repetition is the proven technique to combat it. This chapter shows how to schedule reviews so patterns stick permanently.

---

## The Forgetting Curve

```
MEMORY RETENTION WITHOUT REVIEW:
  
  100% ┤██████████
       │ ██
   80% ┤   ██
       │     ██
   60% ┤       ███
       │          ████
   40% ┤              ████
       │                  ██████
   20% ┤                        █████████
       │                                  ████████████
    0% ┤─────────────────────────────────────────────
       Day 1   Day 2   Day 4   Day 7   Day 14   Day 30

  After 1 day:  You retain ~50%
  After 1 week: You retain ~25%
  After 1 month: You retain ~10%

  WITHOUT review, you WILL forget most of what you learn.
```

---

## How Spaced Repetition Works

```
MEMORY RETENTION WITH SPACED REVIEWS:

  100% ┤██  ██  ████  ████████  ████████████████████
       │ ██ █ ██  █ ██   █  ██████    █ █████████
   80% ┤   █      █       █
       │
   60% ┤
       │
   40% ┤
       │
   20% ┤
       │
    0% ┤─────────────────────────────────────────────
        R1  R2   R3     R4           R5
        (R = Review)

  Each review resets the curve AND makes it decay slower.
  After 5 reviews, the knowledge is near-permanent.
```

---

## The Schedule

```
SPACED REPETITION INTERVALS:

Review 1:  1 day after solving
Review 2:  3 days after Review 1
Review 3:  7 days after Review 2
Review 4:  14 days after Review 3
Review 5:  30 days after Review 4

EXAMPLE:
  Solve problem on Jan 1
  Review 1: Jan 2  (re-solve from memory)
  Review 2: Jan 5  (re-solve from memory)
  Review 3: Jan 12 (re-solve from memory)
  Review 4: Jan 26 (re-solve, should be easy)
  Review 5: Feb 25 (final check, should be instant)

IF YOU FAIL A REVIEW:
  Reset the interval back to 1 day.
  The problem needs more reinforcement.
```

---

## What to Review (Not Re-Read)

```
IMPORTANT: Review ≠ reading your notes

EFFECTIVE REVIEW:
  1. Look at problem statement only
  2. Try to recall the approach (without looking)
  3. Code the solution from memory
  4. Compare with original solution
  5. Note any differences

GRADING EACH REVIEW:
  Easy:    Solved in < 5 min → increase interval
  Good:    Solved in 5-15 min → keep interval
  Hard:    Solved with struggle → decrease interval
  Forgot:  Couldn't solve → reset to Day 1

THIS IS ACTIVE RECALL, NOT PASSIVE REVIEW.
Active recall strengthens neural pathways.
Passive review (re-reading) does NOT.
```

---

## Implementing Spaced Repetition

```
METHOD 1: SPREADSHEET TRACKER
┌──────────┬─────────┬──────┬──────┬──────┬──────┬──────┐
│ Problem   │ Solved  │ R1   │ R2   │ R3   │ R4   │ R5   │
├──────────┼─────────┼──────┼──────┼──────┼──────┼──────┤
│ Two Sum   │ Jan 1   │ Jan 2│ Jan 5│Jan 12│Jan 26│ Feb 25│
│           │         │  ✓   │  ✓   │  ✓   │  ✓   │  ✓   │
│ 3Sum      │ Jan 3   │ Jan 4│ Jan 7│Jan 14│Jan 28│      │
│           │         │  ✓   │  ✗→R │  ✓   │      │      │
│ Coin Chg  │ Jan 5   │ Jan 6│ Jan 9│      │      │      │
│           │         │  ✓   │  ✓   │      │      │      │
└──────────┴─────────┴──────┴──────┴──────┴──────┴──────┘
  ✗→R = Failed, reset interval

METHOD 2: ANKI-STYLE FLASHCARDS
  Front: Problem statement
  Back:  Key insight + approach + complexity
  
  Use any flashcard app with spaced repetition:
  Anki, Quizlet, or even physical index cards

METHOD 3: LEETCODE LISTS
  Create lists by review date:
  "Review Jan 5" → problems due for review that day
  Check off as you re-solve them
```

---

## What to Put on Flashcards

```
FRONT (Question):
  "Array of integers, find two numbers that sum to target.
   Return indices. One solution guaranteed. O(n) required."

BACK (Key Points):
  Pattern: Hash Map lookup
  Approach: Store {value: index} as you iterate
  Key insight: complement = target - current
  Time: O(n)  Space: O(n)
  Edge cases: Same element used twice → check index
  
DO NOT put full code on flashcards.
Put the KEY INSIGHT and APPROACH only.
The goal is to recall the strategy, not memorize code.
```

---

## Daily Review Routine

```
OPTIMAL DAILY SCHEDULE WITH SPACED REPETITION:

  ┌─────────────────────────────────────────────────────┐
  │  10 min: Review 2-3 problems due today              │
  │          (quickly re-solve or recall approach)       │
  │                                                      │
  │  90 min: Learn 1-2 new problems                     │
  │          (deliberate practice with timer)            │
  │                                                      │
  │  10 min: Add new problems to review schedule         │
  │          (update spreadsheet/flashcards)             │
  └─────────────────────────────────────────────────────┘

  RATIO: 80% new learning, 20% review
  As interview approaches: 50% new, 50% review
```

---

## Summary Table

| Review # | Interval | What to Do | On Failure |
|----------|----------|------------|------------|
| 1 | 1 day | Re-solve from memory | Study solution again |
| 2 | 3 days | Re-solve, should be faster | Reset to R1 |
| 3 | 7 days | Quick recall + solve | Reset to R1 |
| 4 | 14 days | Should feel natural | Reset to R2 |
| 5 | 30 days | Nearly instant recall | Reset to R3 |

---

## Quick Revision Questions

1. **What is the forgetting curve?**
   - Without review, you forget ~50% after 1 day, ~75% after 1 week, and ~90% after 1 month

2. **How does spaced repetition combat forgetting?**
   - By reviewing at increasing intervals (1, 3, 7, 14, 30 days), each review resets the forgetting curve and makes it decay slower

3. **What's the difference between active recall and passive review?**
   - Active recall: trying to solve the problem from memory without hints. Passive review: re-reading notes. Active recall is far more effective

4. **What should you do if you fail a review?**
   - Reset the interval back to Day 1 and review the solution — the concept needs more reinforcement

5. **What should flashcards contain?**
   - The key insight and approach, NOT full code — the goal is to recall the strategy, not memorize implementations

6. **How should you balance new problems vs reviews?**
   - Early in prep: 80% new, 20% review. As interview approaches: shift to 50-50 or even 40% new, 60% review

---

[← Previous: Timed Practice](02-timed-practice.md) | [Next: Problem Variety →](04-problem-variety.md)

---
[← Back to README](../README.md)
