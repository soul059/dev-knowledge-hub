# Chapter 3.2: Asking Clarifying Questions

```
╔══════════════════════════════════════════════════════════╗
║        ASKING CLARIFYING QUESTIONS                      ║
║   The mark of a professional engineer                   ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Asking clarifying questions isn't a sign of weakness — it's a **sign of experience**. In real engineering, requirements are always ambiguous. Interviewers deliberately leave problems vague to see if you'll ask the right questions. This chapter gives you a complete system for identifying what to clarify.

---

## Why Clarifying Questions Matter

```
┌─────────────────────────────────────────────────────────────┐
│  WITHOUT CLARIFYING:                                        │
│  Problem → Assumption → Code → WRONG ANSWER                │
│                          ↑                                  │
│                    "I assumed it was sorted"                 │
│                                                             │
│  WITH CLARIFYING:                                           │
│  Problem → Questions → Confirmed Understanding → Code → ✓  │
│                                                             │
│  Interviewers EXPECT and REWARD clarifying questions        │
│  It shows you code like a professional, not a student       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The Clarifying Questions Checklist

```
┌──────────────────────────────────────────────────────────┐
│  UNIVERSAL QUESTIONS (Ask for EVERY problem):            │
│                                                          │
│  INPUT:                                                  │
│  ☐ What are the data types?                             │
│  ☐ What are the size constraints?                       │
│  ☐ Can inputs be null/empty?                            │
│  ☐ Are there duplicates?                                │
│  ☐ What's the value range?                              │
│                                                          │
│  OUTPUT:                                                 │
│  ☐ What should I return? (type, format)                 │
│  ☐ What if there's no valid answer?                     │
│  ☐ Multiple valid answers — return any or all?          │
│                                                          │
│  CONSTRAINTS:                                            │
│  ☐ Time/space requirements?                             │
│  ☐ Can I modify the input?                              │
│  ☐ Can I use extra space?                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Data Structure-Specific Questions

```
ARRAYS:
├── Is it sorted? → Enables binary search, two pointers
├── Duplicates allowed? → Affects uniqueness logic
├── Negative numbers? → Affects sum/product calculations
├── Can I modify in-place? → Affects space complexity
└── 0-indexed or 1-indexed? → Affects return values

STRINGS:
├── Character set? (a-z, A-Z, ASCII, Unicode?)
├── Case-sensitive comparison?
├── Can contain spaces/special chars?
├── Mutable or immutable in this language?
└── Empty string valid?

TREES:
├── Binary tree or BST?
├── Balanced or could be skewed?
├── Duplicate values allowed?
├── Null nodes possible?
└── What to return for empty tree?

GRAPHS:
├── Directed or undirected?
├── Weighted or unweighted?
├── Can have cycles?
├── Connected or disconnected?
├── Self-loops possible?
├── Representation? (adj list, matrix, edge list)
└── Negative weights? (affects algorithm choice)

LINKED LISTS:
├── Singly or doubly linked?
├── Circular or linear?
├── Has a tail pointer?
└── Can be empty?
```

---

## How to Ask (The Right Tone)

```
┌──────────────────────────────────────────────────────────┐
│  GOOD WAY TO ASK:                                        │
│                                                          │
│  "Before I start, I have a few clarifying questions."    │
│  "Can the array contain negative numbers?"               │
│  "What should I return if no valid pair exists?"         │
│  "Is the input guaranteed to be sorted?"                │
│                                                          │
│  ─────────────────────────────────────────────           │
│                                                          │
│  BAD WAY TO ASK:                                         │
│                                                          │
│  "I don't understand the problem."                       │
│  "What do you mean?" (too vague)                        │
│  "Can you explain it again?" (shows you didn't listen)  │
│  Asking 20 questions (wastes too much time)              │
│                                                          │
│  RULES:                                                  │
│  • Ask 3-5 focused questions (not 20)                   │
│  • Ask BEFORE starting your approach                     │
│  • Stay under 2-3 minutes for all questions             │
│  • Don't ask things clearly stated in the problem       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Real Interview Example

```
PROBLEM: "Given an array of integers, return all unique triplets
          that sum to zero."

GOOD CLARIFYING SEQUENCE:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Q1: "Can the array have duplicates?"                   │
│  A1: "Yes, it can."                                     │
│  → This means I need to handle duplicate triplets       │
│                                                          │
│  Q2: "When you say 'unique triplets,' do you mean the   │
│       values should be unique, or the combinations?"    │
│  A2: "Unique combinations — no duplicate triplets."     │
│  → I need to skip duplicates in my iteration            │
│                                                          │
│  Q3: "Should the triplets be returned in any specific   │
│       order?"                                           │
│  A3: "Each triplet should be sorted, and the overall    │
│       result order doesn't matter."                     │
│  → I'll sort the array first                            │
│                                                          │
│  Q4: "Can the array be empty or have fewer than 3       │
│       elements?"                                        │
│  A4: "Yes, return empty list in those cases."           │
│  → I need an early return for small arrays              │
│                                                          │
│  "Great, I think I have a clear picture now. Let me     │
│   discuss my approach."                                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Questions That Reveal Hidden Complexity

```
SMART QUESTIONS THAT IMPRESS INTERVIEWERS:

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "Is there a constraint on space complexity?"            │
│  → Shows you think about space-time trade-offs          │
│                                                          │
│  "Should the solution work for very large inputs?"      │
│  → Shows you think about scalability                    │
│                                                          │
│  "Will this function be called many times with          │
│   different inputs, or just once?"                      │
│  → Could justify preprocessing / caching                │
│                                                          │
│  "Is the input coming from a stream or is it all        │
│   available at once?"                                   │
│  → Stream processing requires different algorithms      │
│                                                          │
│  "Does this need to be thread-safe?"                    │
│  → Shows real-world engineering awareness               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Question Category | Example | What It Reveals |
|-------------------|---------|-----------------|
| Input type/range | "Can values be negative?" | Affects algorithm choice |
| Size constraints | "How large can n be?" | Determines complexity target |
| Output format | "Return indices or values?" | Prevents wrong answer type |
| Edge cases | "What if array is empty?" | Shows thoroughness |
| Modification | "Can I modify the input?" | Determines space needs |
| Multiple answers | "Return one or all solutions?" | Changes approach entirely |
| Order | "Does output order matter?" | Affects sorting decisions |

---

## Quick Revision Questions

1. **How many clarifying questions should you ask?**
   - 3-5 focused questions in 2-3 minutes — enough to eliminate key ambiguities without wasting time

2. **What's the most important question to ask for array problems?**
   - "Is the array sorted?" — this single answer determines whether you can use binary search, two pointers, or need a hash map

3. **How should you phrase your questions?**
   - Specific and hypothesis-based: "Can the array contain negative numbers?" not "What about the array?"

4. **What's a "smart" question that impresses interviewers?**
   - Questions about scalability, space constraints, or whether the function is called once vs many times — shows real-world engineering thinking

5. **When should you stop asking questions and start solving?**
   - After 3-5 questions when you can confidently restate the problem and know the input/output/constraints

6. **What questions should you NOT ask?**
   - Things clearly stated in the problem, overly basic questions ("What is an array?"), or too many micro-questions that waste time

---

[← Previous: Thinking Aloud](01-thinking-aloud.md) | [Next: Explaining Approach →](03-explaining-approach.md)

---
[← Back to README](../README.md)
