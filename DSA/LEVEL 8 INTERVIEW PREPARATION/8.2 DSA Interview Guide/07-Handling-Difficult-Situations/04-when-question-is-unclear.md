# Chapter 7.4: When Question Is Unclear

```
╔══════════════════════════════════════════════════════════╗
║          WHEN QUESTION IS UNCLEAR                        ║
║   Turning ambiguity into opportunity                     ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Sometimes interviewers give deliberately vague problems to test how you handle ambiguity. Other times, problems are genuinely unclear. Either way, asking good questions transforms confusion into clarity and demonstrates senior-level thinking.

---

## Why Questions Are Intentionally Vague

```
INTERVIEWER'S PERSPECTIVE:
┌─────────────────────────────────────────────────────────┐
│ Real-world problems are NEVER fully specified.          │
│                                                          │
│ They want to see if you:                                 │
│  1. Identify ambiguity (don't just assume)              │
│  2. Ask relevant questions (good engineering instinct)  │
│  3. Handle constraints (define scope)                    │
│  4. Communicate before coding (not jump in blindly)     │
│                                                          │
│ Clarifying IS PART OF THE EVALUATION.                   │
│ It's not a sign of weakness — it's a sign of strength.  │
└─────────────────────────────────────────────────────────┘
```

---

## The CLARIFY Framework

```
C — Constraints: What are the input limits?
L — Language: What do specific terms mean?
A — Assumptions: State and verify your assumptions
R — Range: What are valid input values?
I — Input/Output format: What exactly goes in and comes out?
F — Feasibility: Is there always a valid answer?
Y — You sure?: Confirm understanding with an example
```

---

## Essential Questions by Category

```
INPUT QUESTIONS:
  "What's the size range of the input?"
  "Can the input be empty?"
  "Are there duplicates?"
  "Is the input sorted?"
  "Can values be negative?"
  "Are there special characters (for strings)?"

OUTPUT QUESTIONS:
  "Should I return the value or the index?"
  "What should I return if no answer exists?"
  "Is there only one valid answer, or multiple?"
  "Does the order of the output matter?"
  "Should I modify in-place or return a new structure?"

CONSTRAINT QUESTIONS:
  "What's the expected time complexity?"
  "Are there memory constraints?"
  "Can I use built-in sorting/data structures?"
  "Can I modify the original input?"
```

---

## Example: Vague to Clear

```
VAGUE PROBLEM:
  "Find duplicates in an array."

QUESTIONS TO ASK:
  Q: "What type of elements? Integers? Strings?"
  A: "Integers."
  
  Q: "What's the size range? Could be millions?"
  A: "Up to 10^5 elements."
  
  Q: "What should I return? The duplicate values?
      Their indices? Just true/false?"
  A: "Return a list of duplicate values."
  
  Q: "If a number appears 3 times, include it once
      or three times in the output?"
  A: "Just once."
  
  Q: "Does the output order matter?"
  A: "No."
  
  Q: "Are the numbers in any particular range?"
  A: "Yes, all between 1 and n where n = array length."

CLEAR PROBLEM:
  "Given an array of n integers, each in range [1, n],
   return a list of values that appear more than once.
   Each duplicate appears only once in output.
   Order doesn't matter."

  → This changes the approach entirely!
     (Can use array indices as hash, O(1) space)
```

---

## Handling Ambiguous Terms

```
TERM         │ CLARIFICATION NEEDED
─────────────┼────────────────────────────────────
"Subarray"   │ Contiguous? Or subsequence (non-contiguous)?
"Duplicate"  │ Exact same value? Or within tolerance?
"Sorted"     │ Ascending? Non-decreasing (allows ties)?
"Connected"  │ Directly connected? Or through any path?
"Balanced"   │ Height-balanced? Weight-balanced? AVL?
"Shortest"   │ Fewest edges? Or minimum weight?
"Valid"      │ What constitutes valid? Define precisely
"Optimal"    │ Minimize what? Time? Space? Operations?
"Unique"     │ All elements distinct? Or deduplicate output?

ALWAYS say:
"Just to make sure we're on the same page, 
by [term] do you mean [interpretation A] or [interpretation B]?"
```

---

## What If You Still Don't Understand?

```
LEVEL 1: Rephrase
  "Let me make sure I understand: you want me to [restate]?"

LEVEL 2: Ask for example
  "Could you walk me through an example input and
   what the expected output would be?"

LEVEL 3: Propose and verify
  "I'm going to assume [X]. For this input [1,2,3],
   that would give output [Y]. Is that correct?"

LEVEL 4: State your interpretation
  "I'm going to interpret this as [specific definition]
   and solve that. If that's not what you meant,
   please let me know."

NEVER:
  ✗ Pretend you understand when you don't
  ✗ Start coding without understanding the problem
  ✗ Ask the same question repeatedly
  ✗ Ask too many questions (5-8 is the sweet spot)
```

---

## Timing Your Questions

```
QUESTION TIMING MAP:

0:00 ──── Read problem ────────────────── 1:00
1:00 ──── Ask clarifying questions ────── 3:00
3:00 ──── Confirm with example ─────────  4:00
4:00 ──── Start solving ───────────────── ...

BUDGET: 3-4 minutes for clarification
  ~1 min reading problem
  ~2 min asking key questions
  ~1 min verifying with example

AFTER clarification, summarize:
  "So to summarize: I need to [restate problem clearly].
   The input is [constraints]. I should return [output format].
   Let me start with an approach."
```

---

## Summary Table

| Situation | Strategy | Example Phrase |
|-----------|----------|----------------|
| Vague input spec | Ask constraints | "What's the input size range?" |
| Ambiguous term | Ask for definition | "By 'subarray', do you mean contiguous?" |
| Unclear output | Ask format | "Should I return indices or values?" |
| Multiple interpretations | State and verify | "I'll interpret it as X — is that right?" |
| Still confused | Ask for example | "Could you show me an example?" |

---

## Quick Revision Questions

1. **Why do interviewers give vague problems?**
   - To test if you can identify ambiguity, ask good questions, and handle requirements like a real-world engineer

2. **How many clarifying questions should you ask?**
   - 5-8 questions is the sweet spot — enough to clarify without wasting too much time

3. **What's the most effective way to verify your understanding?**
   - Propose a specific input-output example and ask: "For input [X], should the output be [Y]?"

4. **What should you do if you still don't understand after asking questions?**
   - State your interpretation clearly: "I'm going to solve for [specific interpretation]" — and proceed

5. **How much time should you spend on clarification?**
   - About 3-4 minutes: 1 minute reading, 2 minutes asking questions, 1 minute confirming with an example

6. **What should you NEVER do with an unclear question?**
   - Never pretend you understand and start coding — this leads to solving the wrong problem entirely

---

[← Previous: When Code Has Bugs](03-when-code-has-bugs.md) | [Next: When Time Runs Out →](05-when-time-runs-out.md)

---
[← Back to README](../README.md)
