# Chapter 11.4: During the Interview

```
╔══════════════════════════════════════════════════════════╗
║          DURING THE INTERVIEW                            ║
║   Real-time strategy for the coding interview            ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

The interview itself is a structured performance. Success comes from managing your time, communicating clearly, and following a disciplined process — even under pressure. This chapter covers exactly what to do from the moment you say "hello" to the moment you submit your solution.

---

## The Interview Timeline

```
TYPICAL 45-MINUTE CODING INTERVIEW:

  0:00─────5:00──────15:00──────30:00──────40:00─────45:00
   │         │          │          │          │         │
   ▼         ▼          ▼          ▼          ▼         ▼
  Intro   Clarify    Design     Code      Test     Wrap-up
  & Chat  Problem   Approach  Solution   & Debug   & Q&A

  ┌────────────────────────────────────────────────────────┐
  │ Phase 1 (0-5 min):   Introduction & rapport           │
  │ Phase 2 (5-15 min):  Understand & plan approach       │
  │ Phase 3 (15-30 min): Write the code                   │
  │ Phase 4 (30-40 min): Test and fix bugs                │
  │ Phase 5 (40-45 min): Questions for interviewer        │
  └────────────────────────────────────────────────────────┘
```

---

## Phase 1: Introduction (0-5 min)

```
WHAT TO DO:
  ✓ Smile and greet warmly
  ✓ Brief self-introduction (30 seconds max)
  ✓ Show enthusiasm for the role and company
  ✓ Listen carefully to the interviewer's intro

SAMPLE INTRO:
  "Hi, I'm [Name]. I'm currently a [role/student]
   focusing on [area]. I'm excited to interview
   with [Company] because [specific reason].
   Looking forward to today's problem!"

WHAT NOT TO DO:
  ✗ Don't over-share your life story
  ✗ Don't be overly casual or too stiff
  ✗ Don't jump to the problem immediately
  ✗ Don't ask "Is this a hard problem?" 
```

---

## Phase 2: Understanding & Planning (5-15 min)

```
STEP-BY-STEP:

  1. LISTEN to the problem completely
     → Don't interrupt
     → Take notes on key details

  2. REPEAT the problem in your own words
     → "So I need to find the shortest path
        from node A to node B in a weighted graph?"

  3. ASK clarifying questions (at least 3-5)
     → Input size? Sorted? Duplicates?
     → Edge cases? Constraints?
     → Expected output format?

  4. WORK through examples
     → Use the given example first
     → Create 1-2 of your own examples
     → Include an edge case example

  5. STATE your approach
     → "I'm thinking of using a hash map because..."
     → Give time and space complexity
     → Wait for interviewer feedback before coding
```

```
CLARIFICATION CHECKLIST:
  ┌──────────────────────────────────────────┐
  │  ☐ Input:  Type? Size? Range?           │
  │  ☐ Output: Type? Format? Multiple?      │
  │  ☐ Constraints: Time? Space?             │
  │  ☐ Edge cases: Empty? Single? Negative?  │
  │  ☐ Assumptions: Sorted? Unique? Valid?   │
  └──────────────────────────────────────────┘
```

---

## Phase 3: Coding (15-30 min)

```
CODING STRATEGY:

  ┌─────────────────────────────────────────────────┐
  │  1. Write function signature first              │
  │  2. Add comments for major steps                │
  │  3. Fill in code section by section             │
  │  4. Talk through your logic as you write        │
  │  5. Use meaningful variable names               │
  │  6. Handle the core logic before edge cases     │
  └─────────────────────────────────────────────────┘

TALK-WHILE-CODING EXAMPLE:

  "I'll start by initializing a hash map to store
  the frequency of each character..."

  freq = {}
  for char in s:            ← "I iterate through the string"
      if char in freq:
          freq[char] += 1   ← "If seen before, increment"
      else:
          freq[char] = 1    ← "Otherwise, set to 1"

  "Now I need to find the first character with
  frequency 1..."
```

```
RED FLAGS TO AVOID WHILE CODING:

  ✗ Silence for more than 60 seconds
    → Always narrate what you're thinking

  ✗ Writing code without explaining approach first
    → Interviewer needs to understand your plan

  ✗ Getting stuck on syntax details
    → Say "I'll use pseudocode here and come back"

  ✗ Writing perfect code on first attempt
    → It's OK to write, then refine

  ✗ Changing approach mid-code without explaining
    → "I realize this won't work because... 
       let me switch to..."
```

---

## Phase 4: Testing (30-40 min)

```
TESTING PROTOCOL:

  Step 1: TRACE through code with the given example
          → Line by line, track variable values
          → Write values on paper or in comments

  Step 2: TRY a simple edge case
          → Empty input, single element, etc.

  Step 3: TRY a boundary case
          → Maximum/minimum values
          → Exactly the constraint limit

  Step 4: CHECK for common bugs
          → Off-by-one errors
          → Null/None checks
          → Wrong variable references

EXAMPLE DRY RUN NARRATION:

  "Let me trace through with input [2, 7, 11, 15],
   target = 9...
   
   i=0: num=2, complement=7, not in map
        map = {2: 0}
   i=1: num=7, complement=2, found in map!
        return [0, 1]
   
   Output: [0, 1] ✓ Matches expected answer."
```

---

## Phase 5: Questions & Wrap-up (40-45 min)

```
GOOD QUESTIONS TO ASK:

  About the team:
  → "What does the team's day-to-day look like?"
  → "How does the team handle code reviews?"
  → "What's the tech stack for this team?"

  About growth:
  → "What are the learning opportunities here?"
  → "How does mentorship work?"

  About the product:
  → "What's the biggest technical challenge
     the team is working on?"

BAD QUESTIONS:
  ✗ "Did I pass?" (they can't tell you)
  ✗ "What's the salary?" (save for recruiter)
  ✗ "Was my solution optimal?" (seems insecure)
  ✗ Nothing at all (shows lack of interest)
```

---

## Universal Interview Tips

```
                    THE INTERVIEW MINDSET
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  1. COLLABORATE, don't compete                  │
  │     → Treat the interviewer as a teammate       │
  │                                                 │
  │  2. EXPLAIN before you code                     │
  │     → Always get a thumbs-up on approach first  │
  │                                                 │
  │  3. SIMPLIFY first, optimize later              │
  │     → Working brute force > broken optimal      │
  │                                                 │
  │  4. LISTEN to hints carefully                   │
  │     → Hints are gifts, not failures             │
  │                                                 │
  │  5. MANAGE your time actively                   │
  │     → Glance at clock at 15-min and 30-min      │
  │                                                 │
  │  6. STAY POSITIVE throughout                    │
  │     → Even if struggling, show resilience       │
  │                                                 │
  └─────────────────────────────────────────────────┘
```

---

## Summary Table

| Phase | Time | Key Action | Goal |
|-------|------|-----------|------|
| Intro | 0-5m | Greet, brief intro | Build rapport |
| Clarify | 5-15m | Ask questions, plan approach | Understand fully |
| Code | 15-30m | Write solution, talk through logic | Working code |
| Test | 30-40m | Trace examples, fix bugs | Verified solution |
| Q&A | 40-45m | Ask thoughtful questions | Show interest |

---

## Quick Revision Questions

1. **What should you do before writing any code?**
   - Repeat the problem, ask 3-5 clarifying questions, work through examples, and state your approach with complexity analysis — get interviewer buy-in first

2. **How should you handle silence during coding?**
   - Never be silent for more than 60 seconds — narrate your thought process even if it's "I'm considering whether to use a hash map or sorting here"

3. **What's the testing protocol after writing code?**
   - Trace through the given example line by line, try a simple edge case, try a boundary case, and check for common bugs like off-by-one errors

4. **What should you do if you realize your approach is wrong mid-code?**
   - Explain to the interviewer: "I realize this won't work because [reason], let me switch to [new approach]" — don't silently change or panic

5. **What questions should you NOT ask at the end?**
   - "Did I pass?", salary questions, "Was my solution optimal?" — instead ask about team culture, tech challenges, or growth opportunities

6. **How should you manage time during a 45-minute interview?**
   - Check the clock at 15 and 30 minutes — if you haven't started coding by 15 min, move to code; if not testing by 35 min, wrap up the coding

---

[← Previous: Technical Setup](03-technical-setup.md) | [Next: After the Interview →](05-after-the-interview.md)

---
[← Back to README](../README.md)
