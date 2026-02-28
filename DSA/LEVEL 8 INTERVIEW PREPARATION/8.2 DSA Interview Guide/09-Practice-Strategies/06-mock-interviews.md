# Chapter 9.6: Mock Interviews

```
╔══════════════════════════════════════════════════════════╗
║          MOCK INTERVIEWS                                 ║
║   The closest simulation to the real thing               ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Solving problems alone is fundamentally different from solving them with someone watching. Mock interviews bridge that gap. This chapter covers how to set up, conduct, and learn from mock interviews effectively.

---

## Why Mock Interviews Are Essential

```
SKILLS THAT ONLY DEVELOP IN MOCK INTERVIEWS:

  1. Talking while thinking (hardest skill)
  2. Managing nerves under observation
  3. Handling hints gracefully
  4. Time management with real pressure
  5. Whiteboard/screen-share coding
  6. Recovering from mistakes publicly

STUDIES SHOW:
  Candidates who do 5+ mock interviews have
  2-3x higher success rates than those who don't.
```

---

## Types of Mock Interviews

```
┌─────────────────────────────────────────────────────────┐
│ TYPE 1: PEER MOCK (Free)                                │
│   Partner: Friend, classmate, study group member        │
│   Format: Take turns interviewing each other            │
│   Pros: Free, comfortable, mutual learning              │
│   Cons: May not simulate real pressure                  │
│                                                          │
│ TYPE 2: PLATFORM MOCK (Free/Paid)                       │
│   Platforms: Pramp, Interviewing.io, LeetCode           │
│   Format: Matched with random partner online            │
│   Pros: Anonymous → more realistic pressure             │
│   Cons: Partner quality varies                          │
│                                                          │
│ TYPE 3: PROFESSIONAL MOCK (Paid)                        │
│   Platforms: Interviewing.io (pros), expert sessions    │
│   Format: Interview with actual interviewer             │
│   Pros: Most realistic, expert feedback                 │
│   Cons: Expensive ($100-300 per session)                │
│                                                          │
│ TYPE 4: SELF MOCK (Free)                                │
│   Format: Record yourself solving a random problem      │
│   Pros: Always available, review your own performance   │
│   Cons: Missing human interaction and hints             │
└─────────────────────────────────────────────────────────┘
```

---

## Mock Interview Structure

```
STANDARD 45-MINUTE FORMAT:

PHASE 1: Problem Presentation (2 min)
  Interviewer: Read the problem statement
  Candidate: Listen carefully, take notes

PHASE 2: Clarification (3-5 min)
  Candidate: Ask 5-8 clarifying questions
  Interviewer: Answer clearly (or say "good question,
               you can assume [X]")

PHASE 3: Approach Discussion (5-7 min)
  Candidate: Explain brute force, then optimized approach
  Interviewer: Give feedback / hints if needed

PHASE 4: Coding (15-20 min)
  Candidate: Implement solution while explaining
  Interviewer: Observe, stay quiet mostly,
               hint if completely stuck

PHASE 5: Testing (5-7 min)
  Candidate: Walk through test cases
  Interviewer: Suggest edge cases if missed

PHASE 6: Discussion (5 min)
  Candidate: Analyze complexity
  Interviewer: Ask follow-up questions

PHASE 7: Feedback (5-10 min)
  Both: Discuss performance, strengths, areas to improve
```

---

## How to Be a Good Mock Interviewer

```
IF YOU'RE INTERVIEWING YOUR PARTNER:

DO:
  ✓ Choose a problem you've already solved
  ✓ Let them struggle (don't give answers too quickly)
  ✓ Give progressive hints (not full solutions)
  ✓ Track time and give warnings (15 min left, 5 min left)
  ✓ Take notes on their performance for feedback
  ✓ Be encouraging but honest in feedback

DON'T:
  ✗ Choose a problem you can't explain well
  ✗ Interrupt their thinking
  ✗ Give the answer when they're close
  ✗ Be harsh or discouraging in feedback
  ✗ Skip the feedback phase

HINT PROGRESSION:
  Hint 1: "What data structure might help here?"
  Hint 2: "Think about [specific technique]"
  Hint 3: "The key insight is [concept]"
  Hint 4: "Here's how to approach it: [algorithm]"
```

---

## Feedback Framework

```
GIVE FEEDBACK IN THIS ORDER:

1. STRENGTHS (What went well):
   "Your clarifying questions were thorough."
   "You explained the approach clearly before coding."
   "Good edge case coverage in testing."

2. AREAS TO IMPROVE (Specific & actionable):
   "You went silent for 3 minutes while coding.
    Try to narrate what you're writing."
   "You jumped to code without discussing the approach.
    Spend 5 minutes talking through the plan first."

3. SCORE (Optional):
   Problem Solving:    ★★★☆☆  (had right idea, slow)
   Coding:             ★★★★☆  (clean code, minor bugs)
   Communication:      ★★☆☆☆  (needs more talking)
   Testing:            ★★★☆☆  (missed empty input case)

4. ACTION ITEMS:
   "Focus on talking while coding this week."
   "Practice binary search — hesitation suggests weakness."
```

---

## Self-Mock Protocol

```
WHEN YOU DON'T HAVE A PARTNER:

SETUP:
  1. Pick a random medium problem (unseen)
  2. Open screen recorder (or phone camera)
  3. Set 45-minute timer
  4. Use plain text editor (no autocomplete)

DURING:
  1. Read problem aloud
  2. Ask clarifying questions aloud (answer yourself)
  3. Discuss approach aloud
  4. Code while narrating
  5. Test aloud with specific examples
  6. State complexity analysis

AFTER:
  1. Watch the recording
  2. Note where you went silent
  3. Note where you were unclear
  4. Note time spent on each phase
  5. Score yourself using the feedback framework
```

---

## Mock Interview Schedule

```
RECOMMENDED FREQUENCY:
┌──────────────────┬──────────────────────────────────────┐
│ Weeks Before      │ Mock Interview Schedule              │
│ Interview         │                                      │
├──────────────────┼──────────────────────────────────────┤
│ 4+ weeks out      │ 1 per week (self-mock is fine)      │
│ 2-3 weeks out     │ 2 per week (1 self + 1 peer/online) │
│ 1 week out        │ 3 per week (mix all types)          │
│ Day before        │ None — rest and review notes         │
└──────────────────┴──────────────────────────────────────┘

MINIMUM TOTAL BEFORE REAL INTERVIEW: 5 mocks
IDEAL TOTAL: 10+
```

---

## Summary Table

| Mock Type | Cost | Realism | Feedback Quality |
|-----------|------|---------|-----------------|
| Self-mock | Free | Low | Self-assessed |
| Peer mock | Free | Medium | Mutual feedback |
| Platform (Pramp) | Free | Medium-High | Peer + structured |
| Professional | $100-300 | Very High | Expert feedback |

---

## Quick Revision Questions

1. **Why are mock interviews essential for DSA interview prep?**
   - They build skills that can't be developed alone: talking while thinking, managing nerves, handling hints, and recovering from mistakes publicly

2. **What's the minimum number of mock interviews before a real one?**
   - At least 5, ideally 10+ — candidates who do 5+ mocks have 2-3x higher success rates

3. **How should you give feedback to a mock interview partner?**
   - Strengths first, then specific actionable improvements, then optional scoring, then concrete action items

4. **How do you do a self-mock interview?**
   - Record yourself solving a random unseen problem in 45 minutes while talking aloud, then review the recording for improvement areas

5. **What makes a good mock interviewer?**
   - Choosing problems you can explain, giving progressive hints (not answers), tracking time, and providing honest constructive feedback

6. **When should you stop doing mock interviews before the real thing?**
   - The day before — use that day for rest and light review, not active practice

---

[← Previous: Learning from Solutions](05-learning-from-solutions.md) | [Next: Unit 10 — Not Clarifying Requirements →](../10-Common-Mistakes/01-not-clarifying-requirements.md)

---
[← Back to README](../README.md)
