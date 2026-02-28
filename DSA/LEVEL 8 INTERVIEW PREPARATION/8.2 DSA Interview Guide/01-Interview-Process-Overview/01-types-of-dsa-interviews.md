# Chapter 1.1: Types of DSA Interviews

```
╔══════════════════════════════════════════════════════════╗
║          TYPES OF DSA INTERVIEWS                        ║
║   Know the battlefield before you enter it              ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Not all DSA interviews are the same. Understanding the **format, expectations, and environment** of each type helps you prepare strategically. A phone screen demands different skills than a whiteboard interview. This chapter breaks down every type you might encounter.

---

## 1. Online Assessment (OA)

```
┌─────────────────────────────────────────────────────────┐
│                  ONLINE ASSESSMENT                      │
│                                                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐             │
│  │ Problem │    │ Problem │    │ Problem │             │
│  │   1     │    │   2     │    │   3     │             │
│  │ (Easy)  │    │ (Med)   │    │ (Hard)  │             │
│  └────┬────┘    └────┬────┘    └────┬────┘             │
│       │              │              │                   │
│       ▼              ▼              ▼                   │
│  ┌──────────────────────────────────────┐               │
│  │     Auto-Graded Test Cases           │               │
│  │     (Hidden + Visible)               │               │
│  └──────────────────────────────────────┘               │
│                                                         │
│  Duration: 60–120 minutes                               │
│  No interviewer present                                 │
│  Platform: HackerRank, LeetCode, CodeSignal             │
└─────────────────────────────────────────────────────────┘
```

### Key Characteristics
- **No human interaction** — purely algorithm + code correctness
- **Auto-graded** with hidden test cases
- **Time-limited** (usually 60–120 min for 2–4 problems)
- Often includes **MCQs** on DSA concepts

### What Matters Most
| Factor | Importance |
|--------|-----------|
| Correct output | ★★★★★ |
| Time efficiency | ★★★★☆ |
| Code style | ★☆☆☆☆ |
| Communication | N/A |

### Strategy
1. Read ALL problems first — prioritize by difficulty
2. Solve easy problems first for guaranteed points
3. Handle edge cases for partial credit on hard problems
4. Don't waste time on perfect code style — correctness wins

---

## 2. Phone/Video Screen (Technical)

```
┌─────────────────────────────────────────────────────────┐
│              PHONE / VIDEO SCREEN                       │
│                                                         │
│  ┌─────────────┐          ┌─────────────┐              │
│  │ Interviewer  │◄────────►│  Candidate  │              │
│  │ (Watching    │  Shared  │ (Coding in  │              │
│  │  your code)  │  Editor  │  real-time) │              │
│  └─────────────┘          └─────────────┘              │
│                                                         │
│  Duration: 45–60 minutes                                │
│  Usually 1–2 problems                                   │
│  Shared editor: CoderPad, CodePair, Google Docs         │
│  Communication is CRITICAL                              │
└─────────────────────────────────────────────────────────┘
```

### Key Characteristics
- **Live coding** in a shared editor (no IDE features)
- Interviewer can see your **thought process in real-time**
- Usually **1 medium + 1 easy** or **1 medium-hard** problem
- **30% of evaluation** is communication

### What Matters Most
| Factor | Importance |
|--------|-----------|
| Problem solving | ★★★★★ |
| Communication | ★★★★☆ |
| Code quality | ★★★☆☆ |
| Edge cases | ★★★★☆ |

### Strategy
1. Think aloud — narrate your thought process
2. Clarify the problem before coding
3. Discuss your approach before writing code
4. Test your solution verbally at the end

---

## 3. Whiteboard Interview (On-site)

```
┌─────────────────────────────────────────────────────────┐
│              WHITEBOARD INTERVIEW                       │
│                                                         │
│  ┌─────────────────────────────────────────────┐        │
│  │                                             │        │
│  │     function twoSum(nums, target) {         │        │
│  │       const map = {};                       │        │
│  │       for (let i = 0; i < nums.length; i++) │        │
│  │         if (map[target - nums[i]] !== undef) │       │
│  │           return [map[target-nums[i]], i];   │        │
│  │         map[nums[i]] = i;                   │        │
│  │       }                                     │        │
│  │     }                                       │        │
│  │             ← WHITEBOARD →                  │        │
│  └─────────────────────────────────────────────┘        │
│                                                         │
│  ┌───────┐                        ┌───────┐            │
│  │ Cand. │  ◄── Face to Face ──►  │ Inter │            │
│  └───────┘                        └───────┘            │
│                                                         │
│  Duration: 45–60 minutes                                │
│  No autocomplete, no syntax checking                    │
│  Physical presence — body language matters               │
└─────────────────────────────────────────────────────────┘
```

### Key Characteristics
- **No computer** — write code on a whiteboard or paper
- **Syntax doesn't need to be perfect** (pseudocode-ish is OK)
- Focus is on **logic, approach, and communication**
- Body language and confidence matter

### What Matters Most
| Factor | Importance |
|--------|-----------|
| Logical thinking | ★★★★★ |
| Communication | ★★★★★ |
| Code correctness | ★★★★☆ |
| Syntax precision | ★★☆☆☆ |

### Strategy
1. Use the whiteboard strategically — draw diagrams first
2. Write pseudocode before actual code
3. Leave space between lines for modifications
4. Face the interviewer when explaining, board when writing

---

## 4. System Design + DSA Hybrid

```
┌─────────────────────────────────────────────────────────┐
│           SYSTEM DESIGN + DSA HYBRID                    │
│                                                         │
│  Phase 1 (20 min)          Phase 2 (25 min)            │
│  ┌─────────────────┐      ┌─────────────────┐          │
│  │ Design a URL     │      │ Implement the   │          │
│  │ shortener system │ ───► │ hashing function│          │
│  │                  │      │ + collision      │          │
│  │ • Architecture   │      │   resolution    │          │
│  │ • Data model     │      │                 │          │
│  │ • API design     │      │ [Actual Code]   │          │
│  └─────────────────┘      └─────────────────┘          │
│                                                         │
│  Common at: Senior roles, FAANG L5+                     │
└─────────────────────────────────────────────────────────┘
```

### Key Characteristics
- Combines **high-level design** with **low-level implementation**
- Tests both **architectural thinking** and **coding ability**
- More common at **senior engineer levels** (L4/L5+)
- May involve implementing a **key component** of the system

---

## 5. Take-Home Assignment

```
┌─────────────────────────────────────────────────────────┐
│              TAKE-HOME ASSIGNMENT                       │
│                                                         │
│  ┌──────┐    ┌──────────┐    ┌──────────┐    ┌──────┐  │
│  │ Get  │───►│ Complete │───►│ Submit   │───►│Follow│  │
│  │ Prob │    │ at home  │    │ solution │    │ up   │  │
│  └──────┘    └──────────┘    └──────────┘    └──────┘  │
│                                                         │
│  Duration: 2–7 days                                     │
│  Full IDE, internet access allowed                      │
│  Code quality is paramount                              │
│  Often followed by a code review discussion             │
└─────────────────────────────────────────────────────────┘
```

### What Matters Most
| Factor | Importance |
|--------|-----------|
| Code quality | ★★★★★ |
| Tests | ★★★★★ |
| Documentation | ★★★★☆ |
| Correct output | ★★★★★ |
| Architecture | ★★★★☆ |

---

## 6. Live Coding (Pair Programming)

```
┌─────────────────────────────────────────────────────────┐
│              PAIR PROGRAMMING                           │
│                                                         │
│        Candidate                Interviewer             │
│      ┌──────────┐            ┌──────────┐              │
│      │ DRIVER   │◄──────────►│NAVIGATOR │              │
│      │ (Writes  │   Discuss  │ (Guides, │              │
│      │  code)   │   & Plan   │  reviews) │             │
│      └──────────┘            └──────────┘              │
│                                                         │
│  Real IDE environment (VS Code, IntelliJ)               │
│  Collaboration is the key metric                        │
│  Tests teamwork as much as technical skill              │
└─────────────────────────────────────────────────────────┘
```

### Key Characteristics
- Work **together** with the interviewer on a problem
- Uses a **real IDE** with autocomplete and debugging
- Tests **collaboration, communication, and adaptability**
- Interviewer may give hints and guidance intentionally

---

## Comparison Matrix

```
┌──────────────┬───────┬───────┬───────┬───────┬───────┬───────┐
│              │Online │Phone  │White- │Hybrid │Take-  │Pair   │
│ Factor       │Assess.│Screen │board  │       │Home   │Prog.  │
├──────────────┼───────┼───────┼───────┼───────┼───────┼───────┤
│ Duration     │60-120m│45-60m │45-60m │60-90m │2-7day │60-90m │
│ # Problems   │2-4    │1-2    │1-2    │1      │1      │1      │
│ IDE Access   │  Yes  │Shared │  No   │Shared │ Full  │ Full  │
│ Comm. Weight │  Low  │ High  │V.High │ High  │  Low  │V.High │
│ Code Quality │  Low  │ Med   │  Low  │  Med  │V.High │ High  │
│ Auto-graded  │  Yes  │  No   │  No   │  No   │  No   │  No   │
│ Stress Level │  Med  │ High  │V.High │ High  │  Low  │  Med  │
└──────────────┴───────┴───────┴───────┴───────┴───────┴───────┘
```

---

## Interview Type by Company Tier

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  FAANG/MAANG:   OA ──► Phone ──► On-site (4-6 rounds)    │
│                                   ├── Whiteboard x2       │
│                                   ├── System Design       │
│                                   └── Behavioral          │
│                                                            │
│  Mid-Tier:      OA ──► Phone ──► On-site (2-3 rounds)    │
│                                   ├── Live Coding          │
│                                   └── Behavioral          │
│                                                            │
│  Startups:      Take-Home ──► Code Review ──► Cultural   │
│                     OR                                     │
│                 Phone ──► Pair Programming                 │
│                                                            │
│  Service Co:    OA ──► Technical Interview (1-2 rounds)   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Online Assessment | Focus on correctness and speed; no communication needed |
| Phone Screen | Balance coding with clear communication |
| Whiteboard | Logic and communication trump syntax |
| Hybrid | Show both design thinking and coding ability |
| Take-Home | Code quality, testing, and documentation matter most |
| Pair Programming | Collaboration and adaptability are key metrics |
| Preparation | Tailor your practice to the interview format you'll face |

---

## Quick Revision Questions

1. **What is the primary evaluation criterion in an Online Assessment?**
   - Correctness of output against hidden test cases

2. **Why is communication more important in whiteboard interviews than phone screens?**
   - Because there's no code execution; the interviewer judges your logic entirely through your verbal explanation and written pseudocode

3. **Which interview type emphasizes code quality the most?**
   - Take-Home Assignments, where you have full IDE access and time to write production-quality code

4. **How does a pair programming interview differ from a standard phone screen?**
   - In pair programming, you collaborate with the interviewer using a real IDE; it tests teamwork alongside technical skill

5. **What strategy should you use for Online Assessments with multiple problems?**
   - Read all problems first, solve easy ones for guaranteed points, then tackle harder ones

6. **For a FAANG on-site, how many interview rounds should you typically expect?**
   - 4–6 rounds including whiteboard coding, system design, and behavioral

---

[Next: What Interviewers Look For →](02-what-interviewers-look-for.md)

---
[← Back to README](../README.md)
