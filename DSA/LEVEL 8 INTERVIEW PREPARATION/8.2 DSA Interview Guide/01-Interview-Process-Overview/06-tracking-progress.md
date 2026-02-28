# Chapter 1.6: Tracking Progress

```
╔══════════════════════════════════════════════════════════╗
║             TRACKING PROGRESS                           ║
║   What gets measured gets improved                      ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Tracking progress is **not optional** — it's the feedback loop that turns random practice into deliberate improvement. Without tracking, you can't identify weak areas, measure growth, or know when you're ready. This chapter provides systems and tools to track every aspect of your preparation.

---

## The Progress Tracking System

```
┌─────────────────────────────────────────────────────────────┐
│              TRACKING ECOSYSTEM                             │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   PROBLEM    │  │   TOPIC      │  │   MOCK       │     │
│  │   TRACKER    │  │   TRACKER    │  │   INTERVIEW  │     │
│  │              │  │              │  │   TRACKER    │     │
│  │  Every prob  │  │  Mastery per │  │  Score +     │     │
│  │  you solve   │  │  topic area  │  │  feedback    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                  │              │
│         └─────────────────┼──────────────────┘              │
│                           │                                 │
│                    ┌──────▼───────┐                         │
│                    │   WEEKLY     │                         │
│                    │   REVIEW     │                         │
│                    │   DASHBOARD  │                         │
│                    └──────────────┘                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. Problem Tracking Log

Every problem you attempt should be logged with these details:

```
┌──────────────────────────────────────────────────────────────────┐
│  PROBLEM LOG TEMPLATE                                            │
│                                                                  │
│  Date: ________  Problem: _________________  Platform: ______   │
│                                                                  │
│  Difficulty:  ☐ Easy   ☐ Medium   ☐ Hard                       │
│  Topic:       ☐ Array  ☐ Tree    ☐ DP    ☐ Graph  ☐ Other     │
│  Pattern:     ________________________________                   │
│                                                                  │
│  Result:                                                         │
│  ☐ Solved independently (no hints)                              │
│  ☐ Solved with minor hints                                      │
│  ☐ Solved with significant help                                 │
│  ☐ Could not solve — studied solution                           │
│                                                                  │
│  Time Taken:  _____ minutes   (Target was: _____ min)           │
│  Optimal?     ☐ Yes   ☐ No (My: O(___) Optimal: O(___))       │
│                                                                  │
│  Key Learning: _____________________________________________    │
│  Need to Revisit? ☐ Yes (Date: ________)  ☐ No                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Problem Tracking Spreadsheet Layout

| # | Date | Problem Name | Difficulty | Topic | Pattern | Time | Result | Optimal? | Revisit |
|---|------|-------------|:---:|-------|---------|:---:|--------|:---:|:---:|
| 1 | Jan 5 | Two Sum | Easy | Array | Hash Map | 12m | Solo | Yes | No |
| 2 | Jan 5 | Valid Parens | Easy | Stack | Stack | 8m | Solo | Yes | No |
| 3 | Jan 5 | 3Sum | Med | Array | Two Ptr | 35m | Hints | Yes | Jan 12 |
| 4 | Jan 6 | LRU Cache | Med | Design | Hash+LL | 45m | Failed | No | Jan 10 |
| 5 | Jan 6 | Binary Search | Easy | Array | Bin Srch | 10m | Solo | Yes | No |

---

## 2. Topic Mastery Dashboard

```
┌──────────────────────────────────────────────────────────────┐
│  TOPIC MASTERY TRACKER (Update Weekly)                       │
│                                                              │
│  Topic              Problems  Success   Mastery   Status     │
│                     Solved    Rate      Level                │
│  ─────────────────  ────────  ────────  ────────  ────────  │
│  Arrays/Strings       25      88%       ████████  Ready     │
│  Hash Maps            18      83%       ████████  Ready     │
│  Two Pointers         12      75%       ██████░░  Good      │
│  Sliding Window        8      62%       █████░░░  Practice  │
│  Linked Lists         10      80%       ███████░  Good      │
│  Stacks/Queues        10      90%       █████████ Ready     │
│  Trees/BST            15      60%       █████░░░  Practice  │
│  Graphs                8      50%       ████░░░░  Weak      │
│  Dynamic Prog.        12      42%       ███░░░░░  Weak      │
│  Binary Search         8      75%       ██████░░  Good      │
│  Heaps                 5      60%       █████░░░  Practice  │
│  Backtracking          6      50%       ████░░░░  Weak      │
│  Sorting/Search       10      85%       ████████  Ready     │
│  Greedy                6      67%       █████░░░  Practice  │
│  Tries                 3      33%       ██░░░░░░  Weak      │
│                                                              │
│  Legend: Ready (>80%) | Good (65-80%) | Practice (50-65%)   │
│          Weak (<50%)                                         │
└──────────────────────────────────────────────────────────────┘
```

### How to Use the Mastery Dashboard

```
DECISION TREE:

          Success Rate?
         ╱             ╲
    < 50%               ≥ 50%
      │                    │
      ▼                    ▼
  ┌────────┐        Success Rate?
  │ STUDY  │       ╱             ╲
  │ THEORY │   50-65%            > 65%
  │ FIRST  │     │                  │
  │+ Easy  │     ▼                  ▼
  │problems│  ┌────────┐     Success Rate?
  └────────┘  │ MORE   │    ╱            ╲
              │PRACTICE│  65-80%         > 80%
              │(Medium)│    │               │
              └────────┘    ▼               ▼
                        ┌────────┐    ┌──────────┐
                        │ FOCUS  │    │ MAINTAIN  │
                        │ ON HARD│    │ (Review   │
                        │PROBLEMS│    │  weekly)  │
                        └────────┘    └──────────┘
```

---

## 3. Time Tracking Metrics

```
┌──────────────────────────────────────────────────────────┐
│  TIME PERFORMANCE TRACKING                               │
│                                                          │
│  Average Solve Times (Your Goal):                        │
│                                                          │
│  Easy Problems:                                          │
│  Week 1:  ████████████████████████  24 min               │
│  Week 3:  ████████████████         16 min               │
│  Week 6:  ██████████               10 min  ← TARGET    │
│  Week 9:  ████████                  8 min               │
│                                                          │
│  Medium Problems:                                        │
│  Week 1:  ████████████████████████████████████  45 min  │
│  Week 3:  ████████████████████████████████     40 min   │
│  Week 6:  ████████████████████████████         30 min   │
│  Week 9:  ████████████████████                 25 min   │
│  Week 12: ████████████████                     20 min ← │
│                                                          │
│  Hard Problems:                                          │
│  Week 6:  ████████████████████████████████████  60 min+ │
│  Week 9:  ████████████████████████████████     45 min   │
│  Week 12: ████████████████████████████         35 min ← │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 4. Mock Interview Score Tracking

```
┌──────────────────────────────────────────────────────────┐
│  MOCK INTERVIEW LOG                                      │
│                                                          │
│  Date: ________  Interviewer: ___________               │
│  Problem: ________________  Difficulty: ___             │
│                                                          │
│  SCORING (1-5 each):                                     │
│  ┌────────────────────────────┬───────┐                  │
│  │ Problem Solving            │  _/5  │                  │
│  │ Communication              │  _/5  │                  │
│  │ Code Quality               │  _/5  │                  │
│  │ Testing                    │  _/5  │                  │
│  │ Overall                    │  _/5  │                  │
│  └────────────────────────────┴───────┘                  │
│                                                          │
│  Feedback Received:                                      │
│  + Strengths: _____________________________________     │
│  - Weaknesses: ____________________________________     │
│                                                          │
│  Action Items:                                           │
│  1. _______________________________________________     │
│  2. _______________________________________________     │
│  3. _______________________________________________     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Mock Interview Progress Chart

```
Score
5.0 │                                          ★
    │                                     ★
4.5 │                                ★
    │                           ★
4.0 │                      ★
    │                 ★
3.5 │            ★
    │       ★
3.0 │  ★
    │
2.5 │
    └──────────────────────────────────────────
     Mock  Mock  Mock  Mock  Mock  Mock  Mock
      1     2     3     4     5     6     7

Target: Consistent 4.0+ before real interviews
```

---

## 5. Weekly Review Dashboard

```
┌──────────────────────────────────────────────────────────────┐
│  WEEKLY REVIEW — Week #___                                   │
│                                                              │
│  METRICS THIS WEEK:                                          │
│  ┌───────────────────────┬────────┬────────┐                │
│  │ Metric                │ Target │ Actual │                │
│  ├───────────────────────┼────────┼────────┤                │
│  │ Problems solved       │   15   │        │                │
│  │ Study hours           │   28   │        │                │
│  │ New topics covered    │    2   │        │                │
│  │ Problems revisited    │   10   │        │                │
│  │ Mock interviews       │    2   │        │                │
│  │ Average solve time    │  25m   │        │                │
│  │ Solo solve rate       │  70%   │        │                │
│  └───────────────────────┴────────┴────────┘                │
│                                                              │
│  PATTERN RECOGNITION:                                        │
│  Patterns I used well: _________________________________    │
│  Patterns I struggled with: ____________________________    │
│                                                              │
│  WHAT'S WORKING:                                             │
│  1. ________________________________________________        │
│  2. ________________________________________________        │
│                                                              │
│  WHAT NEEDS CHANGE:                                          │
│  1. ________________________________________________        │
│  2. ________________________________________________        │
│                                                              │
│  NEXT WEEK'S PRIORITY:                                       │
│  Focus Topic: ___________________                           │
│  Problem Count Target: __________                            │
│  Special Goals: _________________                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Spaced Repetition for Problem Review

```
┌─────────────────────────────────────────────────────────────┐
│  SPACED REPETITION SCHEDULE                                 │
│                                                             │
│  After solving a problem, review it on:                     │
│                                                             │
│  Day 1 ───► Day 3 ───► Day 7 ───► Day 14 ───► Day 30     │
│    │          │          │           │            │         │
│    ▼          ▼          ▼           ▼            ▼         │
│  Solve     Quick      Re-solve    Can you     Final        │
│  for the   recall     without     explain     check        │
│  first     of key     looking     the key     — mastered?  │
│  time      approach   at notes    insight?                 │
│                                                             │
│  If you FAIL a review → restart the schedule               │
│  If you PASS → move to the next interval                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Tools & Platforms for Tracking

| Tool | Purpose | Free? |
|------|---------|:---:|
| Google Sheets / Excel | Custom tracking spreadsheet | Yes |
| LeetCode Progress | Problem completion tracking | Yes (Limited) |
| Notion | Flexible workspace for all tracking | Yes |
| Anki | Spaced repetition for concepts | Yes |
| GitHub | Track code solutions over time | Yes |
| Toggl / Clockify | Time tracking | Yes |
| Pramp / Interviewing.io | Mock interview scheduling | Free/Paid |

---

## Red Flags in Your Progress

```
┌──────────────────────────────────────────────────────────┐
│  WARNING SIGNS TO WATCH FOR                              │
│                                                          │
│  ⚠️  Same solve rate for 2+ weeks → change approach     │
│  ⚠️  Can't solve any Medium independently → go back to  │
│       Easy + study patterns                              │
│  ⚠️  Solve time not improving → focus on pattern recog. │
│  ⚠️  Mock scores plateauing → get different feedback    │
│  ⚠️  Avoiding certain topics → those are your weak spots│
│  ⚠️  Studying 6+ hours but not improving → quality issue│
│  ⚠️  Feeling burned out → take mandatory rest           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Tracking Area | What to Track | Frequency | Key Metric |
|---------------|--------------|-----------|------------|
| Problem Log | Every problem attempted | Daily | Solo solve rate |
| Topic Mastery | Success rate per topic | Weekly | Mastery level per topic |
| Time Performance | Solve time by difficulty | Weekly | Average solve time trend |
| Mock Interviews | Scores and feedback | Per mock | Average score trend |
| Weekly Review | Overall progress metrics | Weekly | Target vs Actual comparison |
| Spaced Repetition | Review schedule adherence | Daily | Problems due today |

---

## Quick Revision Questions

1. **What are the five key data points to log for every problem you solve?**
   - Date, difficulty, topic/pattern, time taken, result (solo/hints/failed), and whether it was optimal

2. **What success rate indicates a topic is "ready" vs "weak"?**
   - Ready: >80% success rate; Weak: <50% success rate

3. **When should you use the spaced repetition review schedule?**
   - After solving a problem, review it on Day 1, 3, 7, 14, and 30 to ensure long-term retention

4. **What is a red flag in your progress tracking?**
   - Same solve rate for 2+ weeks, avoiding certain topics, or studying many hours without improvement

5. **What target mock interview score should you aim for before real interviews?**
   - Consistent 4.0+/5.0 across multiple mock interviews

6. **Why is tracking study hours alone insufficient?**
   - Hours don't measure quality; you could study 6 hours doing easy problems and not improve — track solve rate, time improvement, and topic mastery alongside hours

---

[← Previous: Setting Goals](05-setting-goals.md) | [Next: Unit 2 — Understand the Problem →](../02-Problem-Solving-Framework/01-understand-the-problem.md)

---
[← Back to README](../README.md)
