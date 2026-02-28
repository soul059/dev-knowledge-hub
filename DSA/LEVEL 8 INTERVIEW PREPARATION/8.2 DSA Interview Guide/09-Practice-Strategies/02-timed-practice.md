# Chapter 9.2: Timed Practice

```
╔══════════════════════════════════════════════════════════╗
║          TIMED PRACTICE                                  ║
║   Building speed without sacrificing quality              ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

In real interviews, you have **30-45 minutes** per problem. Solving it in 90 minutes at home doesn't count. Timed practice trains you to think, code, and test under pressure.

---

## Why Timing Matters

```
INTERVIEW REALITY:
┌─────────────────────────────────────────────────────────┐
│ Total interview: 45 minutes                             │
│ - Introductions: 2-3 min                                │
│ - Clarifying questions: 3-5 min                         │
│ - Discuss approach: 5-7 min                             │
│ - Code solution: 15-20 min                              │
│ - Test & debug: 5-7 min                                 │
│ - Complexity discuss: 2-3 min                           │
│ - Questions for interviewer: 3-5 min                    │
│                                                          │
│ ACTUAL CODING TIME: ~20 minutes                         │
│                                                          │
│ If you need 40 min to code at home, you need 2x         │
│ speedup for the interview.                               │
└─────────────────────────────────────────────────────────┘
```

---

## Timed Practice Protocol

```
PHASE 1: LEARN (No Timer) — Weeks 1-2
  → Focus on understanding patterns
  → Take as long as needed
  → Study solutions carefully
  → Build pattern recognition

PHASE 2: SOFT TIMER — Weeks 3-4
  → Set timer but don't stress about it
  → Goal: Awareness of how long things take
  → Easy: 20 min, Medium: 35 min, Hard: 50 min

PHASE 3: STRICT TIMER — Weeks 5+
  → Stop when timer ends
  → If not done, note where you got stuck
  → Analyze: Was it understanding? Coding? Debugging?

  Target times:
  ┌────────────┬──────────┬───────────────────────┐
  │ Difficulty  │ Target   │ Includes              │
  ├────────────┼──────────┼───────────────────────┤
  │ Easy        │ 15 min   │ Understand + Code     │
  │ Medium      │ 25 min   │ Full solve cycle      │
  │ Hard        │ 40 min   │ Full solve cycle      │
  └────────────┴──────────┴───────────────────────┘
```

---

## Speed Optimization Techniques

```
TECHNIQUE 1: Template Memorization
  Have templates ready for common patterns:
  
  Binary Search:
    left, right = 0, n - 1
    while left <= right:
        mid = left + (right - left) // 2
        if condition: left = mid + 1
        else: right = mid - 1
  
  BFS:
    queue = deque([start])
    visited = {start}
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

  If you can type these without thinking,
  you save 5-10 min per problem.

TECHNIQUE 2: Think Before You Type
  Spend 5 min planning → Save 15 min debugging
  
  PLAN:
  1. What data structure?
  2. What's the main loop structure?
  3. What are the edge cases?
  → THEN start coding

TECHNIQUE 3: Keyboard Shortcuts
  Learn your IDE's shortcuts:
  - Auto-complete
  - Multi-cursor editing
  - Quick variable rename
  - Block selection
  
  These save seconds that add up.

TECHNIQUE 4: Code In Your Interview Language
  Always practice in the language you'll use.
  Don't practice in Python then interview in Java.
```

---

## Simulated Interview Sessions

```
FULL SIMULATION (do weekly):

┌─────────────────────────────────────────────────────────┐
│ SET UP:                                                  │
│   - Random medium problem (use LeetCode random)         │
│   - 45-minute timer                                     │
│   - No looking at hints or solutions                    │
│   - Talk out loud (simulate explaining to interviewer)  │
│   - Use plain text editor (not IDE with autocomplete)   │
│                                                          │
│ TIMELINE:                                                │
│   0:00-3:00  — Read and clarify (talk aloud)            │
│   3:00-8:00  — Discuss approach (talk aloud)            │
│   8:00-30:00 — Code the solution                        │
│   30:00-40:00 — Test with examples                      │
│   40:00-45:00 — Analyze complexity                      │
│                                                          │
│ AFTER:                                                   │
│   - Grade yourself on each dimension                    │
│   - Compare time vs target                               │
│   - Note what slowed you down                            │
└─────────────────────────────────────────────────────────┘
```

---

## Self-Scoring After Timed Practice

```
SCORING RUBRIC (out of 10):

Understanding (2 points):
  2: Identified approach in < 5 min
  1: Needed 5-10 min to find approach
  0: Couldn't identify approach

Coding (3 points):
  3: Clean, correct code in target time
  2: Working code, slightly over time
  1: Code with bugs, needed debugging
  0: Couldn't complete code

Testing (2 points):
  2: Tested edge cases + normal cases
  1: Tested normal cases only
  0: No testing

Communication (2 points):
  2: Explained clearly throughout
  1: Some explanation, some silence
  0: Silent coding

Complexity (1 point):
  1: Correctly stated time & space
  0: Incorrect or missing analysis

TARGET: 7+/10 consistently = interview ready
```

---

## Tracking Speed Over Time

```
SPEED LOG:
┌────────┬────────────┬───────┬────────┬─────────────────┐
│ Date   │ Problem     │ Diff  │ Time   │ Target Met?     │
├────────┼────────────┼───────┼────────┼─────────────────┤
│ Jan 1  │ Two Sum     │ Easy  │ 12min  │ ✓ (< 15 min)   │
│ Jan 2  │ 3Sum        │ Med   │ 38min  │ ✗ (> 25 min)   │
│ Jan 5  │ 3Sum (redo) │ Med   │ 22min  │ ✓ (< 25 min)   │
│ Jan 8  │ Coin Change │ Med   │ 30min  │ ✗ (> 25 min)   │
│ Jan 12 │ Coin Change │ Med   │ 18min  │ ✓               │
└────────┴────────────┴───────┴────────┴─────────────────┘

TREND: You should see times decrease for the same
difficulty level over weeks.
```

---

## Summary Table

| Phase | Timer Setting | Goal | Duration |
|-------|--------------|------|----------|
| Learn | None | Understanding patterns | Weeks 1-2 |
| Soft Timer | Loose | Speed awareness | Weeks 3-4 |
| Strict Timer | Firm cutoff | Real time pressure | Weeks 5+ |
| Simulation | Full 45 min | Complete interview practice | Weekly |

---

## Quick Revision Questions

1. **How much actual coding time do you get in a 45-minute interview?**
   - About 20 minutes — the rest is introductions, clarification, approach discussion, testing, and complexity analysis

2. **What are the target times for solving problems?**
   - Easy: 15 minutes, Medium: 25 minutes, Hard: 40 minutes (including full solve cycle)

3. **What's the most effective speed optimization technique?**
   - Template memorization — having common patterns (BFS, binary search, DFS) ready to type without thinking saves 5-10 minutes

4. **When should you start using strict timers?**
   - After weeks 1-4 of learning patterns; start strict timing in week 5+ once you have foundational understanding

5. **How should you simulate a real interview?**
   - Random problem, 45-minute timer, no hints, talk out loud, use a plain text editor, and self-score afterward

6. **What score indicates interview readiness?**
   - Consistently scoring 7+/10 on the self-scoring rubric across multiple simulated sessions

---

[← Previous: Deliberate Practice](01-deliberate-practice.md) | [Next: Spaced Repetition →](03-spaced-repetition.md)

---
[← Back to README](../README.md)
