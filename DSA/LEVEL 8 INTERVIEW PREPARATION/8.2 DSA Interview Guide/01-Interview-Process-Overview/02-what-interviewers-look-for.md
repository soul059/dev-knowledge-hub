# Chapter 1.2: What Interviewers Look For

```
╔══════════════════════════════════════════════════════════╗
║         WHAT INTERVIEWERS LOOK FOR                      ║
║   Understand the scorecard to maximize your score       ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Most candidates think interviews are only about solving the problem. In reality, interviewers evaluate you across **multiple dimensions simultaneously**. Understanding each dimension lets you optimize your performance even when you can't solve the problem perfectly.

---

## The Evaluation Framework

```
┌─────────────────────────────────────────────────────────────┐
│              INTERVIEWER'S MENTAL SCORECARD                 │
│                                                             │
│  ┌─────────────────┐  Weight                                │
│  │ Problem Solving  │  ████████████████████  35%            │
│  ├─────────────────┤                                        │
│  │ Coding Ability   │  █████████████████     30%            │
│  ├─────────────────┤                                        │
│  │ Communication    │  ██████████████        20%            │
│  ├─────────────────┤                                        │
│  │ Testing/Rigor    │  ██████████            10%            │
│  ├─────────────────┤                                        │
│  │ Culture Fit      │  ██████                 5%            │
│  └─────────────────┘                                        │
│                                                             │
│  Note: Weights vary by company and role level               │
└─────────────────────────────────────────────────────────────┘
```

---

## Dimension 1: Problem-Solving Ability (35%)

This is the **most heavily weighted** dimension. It's not about whether you've seen the problem before — it's about **how you approach** an unfamiliar problem.

```
WHAT THEY OBSERVE:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Can you...                                             │
│                                                         │
│  ✓ Break down a complex problem into smaller parts?     │
│  ✓ Identify the right data structure for the job?       │
│  ✓ Recognize patterns from similar problems?            │
│  ✓ Move from brute force to optimized solution?         │
│  ✓ Handle constraints and edge cases logically?         │
│  ✓ Reason about correctness before coding?              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### What Gets HIGH Marks

```
STRONG Candidate:
┌──────────────────────────────────────────────────────────┐
│ "Let me think about what data structure fits here...     │
│  We need O(1) lookups, so a hash map makes sense.       │
│  But wait — we also need ordering, so maybe a           │
│  LinkedHashMap or a combination of hash map + list."    │
│                                                          │
│  → Shows structured thinking                             │
│  → Considers alternatives                                │
│  → Justifies decisions                                   │
└──────────────────────────────────────────────────────────┘

WEAK Candidate:
┌──────────────────────────────────────────────────────────┐
│ "I'll use a hash map."                                   │
│  *starts coding immediately*                             │
│                                                          │
│  → No reasoning shown                                    │
│  → No alternatives considered                            │
│  → Interviewer can't assess thinking process             │
└──────────────────────────────────────────────────────────┘
```

### The Problem-Solving Score Rubric

| Score | Description |
|-------|-------------|
| 5/5 | Identifies optimal approach independently, handles all edge cases |
| 4/5 | Gets to optimal with minor hints, handles most edge cases |
| 3/5 | Gets a working solution (may not be optimal), misses some edges |
| 2/5 | Needs significant guidance, partially working solution |
| 1/5 | Cannot make meaningful progress even with hints |

---

## Dimension 2: Coding Ability (30%)

```
CODING ABILITY BREAKDOWN:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Correctness │  │   Fluency    │  │   Quality    │  │
│  │              │  │              │  │              │  │
│  │• Logic works │  │• Types code  │  │• Clean vars  │  │
│  │• Handles     │  │  quickly     │  │• Good funcs  │  │
│  │  all inputs  │  │• Doesn't     │  │• Readable    │  │
│  │• No bugs     │  │  struggle    │  │• Modular     │  │
│  │              │  │  with syntax │  │              │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│       40%               30%               30%           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### What They Look For in Code

```python
# BAD — What weak candidates write
def f(a, t):
    for i in range(len(a)):
        for j in range(i+1, len(a)):
            if a[i] + a[j] == t:
                return [i, j]

# GOOD — What strong candidates write  
def two_sum(nums, target):
    """Find two indices whose values sum to target."""
    seen = {}  # value -> index mapping
    
    for index, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], index]
        seen[num] = index
    
    return []  # No valid pair found
```

### Code Quality Signals

| Signal | Strong | Weak |
|--------|--------|------|
| Variable names | `complement`, `seen` | `x`, `d` |
| Logic structure | Clear, linear flow | Nested, confusing |
| Edge handling | Explicitly addressed | Ignored |
| Comments | Where needed | None or too many |
| Language mastery | Uses built-in features well | Reinvents the wheel |

---

## Dimension 3: Communication (20%)

```
COMMUNICATION TIMELINE IN AN INTERVIEW:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  0 min ──── 5 min ──── 15 min ──── 35 min ──── 45 min │
│    │          │           │           │           │     │
│    ▼          ▼           ▼           ▼           ▼     │
│ Clarify   Discuss     Explain     Code with    Explain │
│ problem   approach    trade-offs  narration    tests   │
│                                                         │
│  ◄───── All of these are communication events ──────►  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### What Good Communication Looks Like

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  1. CLARIFYING:                                         │
│     "Can the array contain negative numbers?"           │
│     "Is the input always sorted?"                       │
│     "What should I return if no solution exists?"       │
│                                                         │
│  2. PLANNING:                                           │
│     "My approach is to use a hash map because..."       │
│     "The time complexity would be O(n) because..."      │
│     "An alternative approach would be..."               │
│                                                         │
│  3. CODING:                                             │
│     "I'm initializing a hash map to store seen values"  │
│     "This loop iterates through each element once"      │
│     "Here I'm checking if the complement exists"        │
│                                                         │
│  4. TESTING:                                            │
│     "Let me trace through with input [2,7,11], t=9"    │
│     "Edge case: empty array should return []"           │
│     "Edge case: duplicate values..."                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Dimension 4: Testing & Rigor (10%)

```
TESTING RIGOR SPECTRUM:
                                                          
  Weak ◄──────────────────────────────────────► Strong   
                                                          
  "I think        "Let me test      "Let me trace       
   it works"       with [1,2,3]"     through normal,    
                                     edge, and corner    
                                     cases step by step" 
```

### What Interviewers Want to See

1. **Proactive testing** — don't wait to be asked
2. **Edge case awareness** — empty input, single element, duplicates
3. **Systematic debugging** — trace through code methodically
4. **Confidence calibration** — know when your code is correct

---

## Dimension 5: Culture Fit & Soft Skills (5%)

```
┌─────────────────────────────────────────────────────────┐
│  CULTURE FIT SIGNALS                                    │
│                                                         │
│  Positive ✓                    Negative ✗               │
│  ─────────                     ─────────                │
│  • Handles hints gracefully    • Gets defensive         │
│  • Admits what they don't know • Pretends to know       │
│  • Shows enthusiasm            • Shows frustration      │
│  • Asks thoughtful questions   • No curiosity           │
│  • Collaborative attitude      • Dismissive of input    │
│  • Takes feedback well         • Argues with feedback   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## The Hidden Evaluation: "Would I Want to Work with This Person?"

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  After every interview, the interviewer asks:           │
│                                                         │
│   ┌───────────────────────────────────────┐             │
│   │ "Would I enjoy working with this      │             │
│   │  person on a daily basis?"            │             │
│   │                                       │             │
│   │  • Can they explain their thinking?   │             │
│   │  • Do they listen to feedback?        │             │
│   │  • Are they pleasant under pressure?  │             │
│   │  • Can they learn and adapt quickly?  │             │
│   └───────────────────────────────────────┘             │
│                                                         │
│  This question influences the final decision            │
│  MORE than most candidates realize.                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## What Matters at Each Level

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  JUNIOR (0-2 years):                                        │
│  ├── Problem solving: Can you solve medium problems?        │
│  ├── Coding: Is your code correct and readable?             │
│  ├── Communication: Can you explain your approach?          │
│  └── Growth potential: Can you learn quickly?               │
│                                                             │
│  MID-LEVEL (2-5 years):                                     │
│  ├── Problem solving: Can you optimize effectively?         │
│  ├── Coding: Production-quality code with error handling    │
│  ├── Communication: Can you discuss trade-offs?             │
│  └── Design: Can you think about system implications?       │
│                                                             │
│  SENIOR (5+ years):                                         │
│  ├── Problem solving: Can you tackle hard, ambiguous probs? │
│  ├── Coding: Elegant, extensible solutions                  │
│  ├── Communication: Can you mentor through explanation?     │
│  └── Leadership: Do you drive the interview constructively? │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Red Flags That Get You Rejected

```
┌──────────────────────────────────────────────────────────┐
│  INSTANT RED FLAGS                                       │
│                                                          │
│  ✗ Starting to code without understanding the problem    │
│  ✗ Unable to explain your own code                       │
│  ✗ Getting frustrated or arguing with the interviewer    │
│  ✗ Claiming to know something you clearly don't          │
│  ✗ Refusing to consider alternative approaches           │
│  ✗ Not handling ANY edge cases                           │
│  ✗ Writing code that doesn't compile / has syntax errors │
│    (in a typed language interview)                        │
│  ✗ Complete silence for extended periods                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Dimension | Weight | What They Evaluate | How to Excel |
|-----------|--------|-------------------|--------------|
| Problem Solving | 35% | Approach, pattern recognition, optimization | Practice the UMPIRE framework |
| Coding Ability | 30% | Correctness, fluency, code quality | Write clean, production-quality code |
| Communication | 20% | Clarity, collaboration, explaining trade-offs | Think aloud, narrate your process |
| Testing & Rigor | 10% | Edge cases, debugging, verification | Always test before saying "done" |
| Culture Fit | 5% | Attitude, coachability, enthusiasm | Be genuine, collaborative, and humble |

---

## Quick Revision Questions

1. **What is typically the most heavily weighted evaluation dimension?**
   - Problem-solving ability (~35%), followed by coding ability (~30%)

2. **What does "coding fluency" mean to an interviewer?**
   - The ability to translate your algorithm into code quickly without struggling with syntax or language features

3. **Why can a candidate who doesn't solve the problem still get hired?**
   - If they demonstrate strong problem-solving approach, great communication, handled edge cases, and showed they were close to the solution with the right direction

4. **What is the "hidden evaluation" that interviewers make?**
   - "Would I want to work with this person daily?" — evaluating collaboration, attitude, and coachability

5. **How do expectations differ between junior and senior candidates?**
   - Juniors: correctness + growth potential; Seniors: optimization + elegance + ability to drive the interview + design thinking

6. **Name three instant red flags that lead to rejection.**
   - Coding without understanding the problem, inability to explain your own code, getting frustrated or argumentative

---

[← Previous: Types of DSA Interviews](01-types-of-dsa-interviews.md) | [Next: Interview Stages →](03-interview-stages.md)

---
[← Back to README](../README.md)
