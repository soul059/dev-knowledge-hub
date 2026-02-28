# Chapter 10.2: Jumping to Code

```
╔══════════════════════════════════════════════════════════╗
║          JUMPING TO CODE                                 ║
║   Why coding first and thinking later costs interviews   ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

The second most common mistake: starting to code the moment you hear the problem. Without thinking through the approach first, you'll write code you'll have to delete, confuse yourself mid-implementation, and show the interviewer you lack planning discipline.

---

## Why People Jump to Code

```
REASONS:
  1. Anxiety:  "I need to show I can code!"
  2. Habit:    "I always code first, think later on LC"
  3. Pressure: "The clock is ticking!"
  4. Confidence: "I've seen this before" (but maybe not)

WHAT ACTUALLY HAPPENS:
  ┌─────────────────────────────────────────────────────┐
  │ Start coding immediately                            │
  │   → Get halfway through                             │
  │     → Realize approach won't work                   │
  │       → Delete and restart                          │
  │         → Now 15 min behind with more anxiety       │
  └─────────────────────────────────────────────────────┘
```

---

## The Right Sequence

```
CORRECT ORDER:
  ┌─────────────────────────────────────────────────────┐
  │ 1. UNDERSTAND (3-5 min)                             │
  │    Ask questions, restate problem                   │
  │                                                      │
  │ 2. PLAN (5-7 min)                                   │
  │    Discuss approach, consider alternatives           │
  │    Get interviewer buy-in: "Does this sound right?" │
  │                                                      │
  │ 3. CODE (15-20 min)                                  │
  │    Now code with confidence                          │
  │    You know exactly what to write                   │
  │                                                      │
  │ 4. TEST (5-7 min)                                    │
  │    Verify correctness                                │
  └─────────────────────────────────────────────────────┘

  Planning time is NOT wasted time.
  5 minutes of planning saves 15 minutes of rewriting.
```

---

## What "Planning" Looks Like

```
PLANNING CHECKLIST (before touching keyboard):

  ☐ What approach am I using? (name the technique)
  ☐ What data structures do I need?
  ☐ What's the overall flow?
      (e.g., "Sort → two pointers → return result")
  ☐ What are the edge cases?
  ☐ What's the expected complexity?
  ☐ Have I got interviewer buy-in?

EXAMPLE PLANNING STATEMENT:
  "I'll use a sliding window approach. I'll maintain a
   hash map to track character frequencies. I'll expand
   the right pointer and shrink the left when the window
   violates the constraint. This should be O(n) time
   and O(k) space where k is the character set size.
   Does this approach sound reasonable?"
```

---

## Recognizing the "Jump to Code" Impulse

```
WARNING SIGNS:
  ✗ You open your editor within 30 seconds of hearing
    the problem
  ✗ You start typing before explaining anything
  ✗ You say "let me just start coding and figure it out"
  ✗ You haven't asked a single clarifying question
  ✗ You can't state your approach in 2 sentences

SELF-CHECK:
  Before typing the first line of code, ask yourself:
  "Can I explain my full approach in 30 seconds?"
  
  If no → you're not ready to code.
```

---

## How to Break the Habit

```
TRAINING EXERCISES:

1. WHITEBOARD-ONLY SESSIONS:
   Solve problems on paper/whiteboard.
   No computer → forces planning before coding.

2. APPROACH-FIRST RULE:
   Write 3-5 comment lines describing your approach
   BEFORE writing any code:
   
   # 1. Build hash map of values to indices
   # 2. For each element, check if complement exists
   # 3. Return indices if found
   # 4. Edge case: same element can't be used twice
   
   Then fill in the code under each comment.

3. SILENT PLANNING TIME:
   In every practice session, set a 5-minute timer
   where you can ONLY think and write pseudocode.
   No real code allowed until timer ends.
```

---

## Summary Table

| Behavior | Risk | Better Alternative |
|----------|------|-------------------|
| Code immediately | Rewrite from scratch | Plan 5-7 min first |
| No approach discussion | Wrong algorithm chosen | State approach, get buy-in |
| Skip pseudocode | Confused mid-implementation | Write outline as comments |
| Code without edge cases | Fails on tests | List edge cases before coding |

---

## Quick Revision Questions

1. **Why is jumping to code so dangerous in interviews?**
   - You often realize the approach is wrong mid-implementation, forcing you to delete and restart with less time and more anxiety

2. **How long should you plan before coding?**
   - 5-7 minutes, including approach discussion and getting interviewer buy-in

3. **What should you be able to do before writing code?**
   - Explain your full approach in 2 sentences, name the technique, and identify the data structures needed

4. **How can you break the habit of jumping to code?**
   - Practice whiteboard-only sessions, write approach as comments first, and enforce a 5-minute planning-only timer

5. **What should you say before starting to code?**
   - "My approach is [technique]. I'll use [data structure]. The flow is [steps]. Expected complexity is [O(?)]. Does this sound reasonable?"

6. **Is planning time wasted time?**
   - No — 5 minutes of planning saves 15 minutes of rewriting and debugging incorrect code

---

[← Previous: Not Clarifying Requirements](01-not-clarifying-requirements.md) | [Next: Ignoring Edge Cases →](03-ignoring-edge-cases.md)

---
[← Back to README](../README.md)
