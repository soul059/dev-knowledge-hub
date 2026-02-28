# Chapter 2.1: Understand the Problem

```
╔══════════════════════════════════════════════════════════╗
║          UNDERSTAND THE PROBLEM                         ║
║   The most skipped — and most important — step          ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

**80% of interview failures** happen not because candidates lack coding skills, but because they solve the **wrong problem**. Understanding the problem completely before touching code is the single most impactful habit you can build. This chapter teaches a rigorous system for problem comprehension.

---

## The UMPIRE Framework — Step 1: Understand

```
┌─────────────────────────────────────────────────────────────┐
│                   THE UMPIRE FRAMEWORK                      │
│                                                             │
│   U ─ Understand the problem    ◄── YOU ARE HERE           │
│   M ─ Match to known patterns                               │
│   P ─ Plan your approach                                    │
│   I ─ Implement the solution                                │
│   R ─ Review and test                                       │
│   E ─ Evaluate complexity                                   │
│                                                             │
│   The first step U is the FOUNDATION for everything else   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The 5 Questions to Ask Immediately

```
┌─────────────────────────────────────────────────────────────┐
│  THE 5 ESSENTIAL QUESTIONS                                  │
│                                                             │
│  1. WHAT are the inputs?                                    │
│     ├── Data types (int, string, array, tree?)              │
│     ├── Size constraints (1 ≤ n ≤ 10^5?)                   │
│     └── Value ranges (negative numbers? duplicates?)        │
│                                                             │
│  2. WHAT is the expected output?                            │
│     ├── Return type (int, array, boolean?)                  │
│     ├── Format (sorted? specific order?)                    │
│     └── What if no valid answer exists?                      │
│                                                             │
│  3. WHAT are the constraints?                               │
│     ├── Time limit hints at expected complexity              │
│     ├── Space constraints?                                   │
│     └── Must it be in-place?                                 │
│                                                             │
│  4. WHAT are the edge cases?                                │
│     ├── Empty input?                                         │
│     ├── Single element?                                      │
│     ├── All same values?                                     │
│     └── Maximum/minimum values?                              │
│                                                             │
│  5. CAN I restate the problem in my own words?             │
│     └── Verify with interviewer: "So what you're asking    │
│         is... is that correct?"                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Process

### Step 1: Read the Problem Twice

```
First Read:  Get the gist — what is being asked?
             ┌─────────────────────────────────────┐
             │ "Find two numbers that sum to target"│
             └─────────────────────────────────────┘

Second Read: Focus on CONSTRAINTS and DETAILS
             ┌─────────────────────────────────────────┐
             │ "Each input has EXACTLY one solution"    │
             │ "May NOT use the same element twice"     │
             │ "Return INDICES, not values"             │
             └─────────────────────────────────────────┘

             Notice what changed? The details MATTER.
```

### Step 2: Identify Input/Output Types

```
EXAMPLE: Two Sum Problem

Input Analysis:
┌─────────────────────────────────────────────────────────┐
│  nums: int[]     → Array of integers                    │
│  target: int     → Single integer target                │
│                                                         │
│  Questions to ask:                                      │
│  • Can nums contain negative numbers?          → Yes    │
│  • Can nums contain duplicates?                → Yes    │
│  • What's the size range?           → 2 ≤ n ≤ 10^4    │
│  • Is the array sorted?                        → No     │
│  • Can target be negative?                     → Yes    │
└─────────────────────────────────────────────────────────┘

Output Analysis:
┌─────────────────────────────────────────────────────────┐
│  Return: int[2]  → Two indices                          │
│                                                         │
│  Questions to ask:                                      │
│  • 0-indexed or 1-indexed?                    → 0-indexed│
│  • Order matters? [0,1] vs [1,0]?            → No       │
│  • What if no answer? (Problem says exactly 1 exists)   │
└─────────────────────────────────────────────────────────┘
```

### Step 3: Restate the Problem

```
YOUR RESTATEMENT (say this to the interviewer):
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  "So if I understand correctly, I'm given an array of   │
│   integers and a target sum. I need to find the indices │
│   of exactly two different elements whose values add up │
│   to the target. The problem guarantees exactly one     │
│   solution exists. Is that right?"                      │
│                                                         │
│   Interviewer: "Yes, that's correct."                   │
│                                                         │
│   → Now you KNOW you understand the problem             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Clarifying Questions Cheat Sheet

### For Array Problems
```
┌─────────────────────────────────────────────────────────┐
│  • Is the array sorted?                                  │
│  • Can it contain duplicates?                            │
│  • Can it contain negative numbers or zeros?             │
│  • What's the size range? (affects complexity)           │
│  • Should I modify the array in-place?                   │
│  • Is the array guaranteed non-empty?                    │
└─────────────────────────────────────────────────────────┘
```

### For String Problems
```
┌─────────────────────────────────────────────────────────┐
│  • Character set? (lowercase only? ASCII? Unicode?)      │
│  • Can it be empty?                                      │
│  • Are spaces/special characters included?               │
│  • Is comparison case-sensitive?                         │
│  • Should I handle null/undefined strings?               │
└─────────────────────────────────────────────────────────┘
```

### For Tree/Graph Problems
```
┌─────────────────────────────────────────────────────────┐
│  • Is it a binary tree or n-ary tree?                    │
│  • Is it a BST (binary search tree)?                     │
│  • Can it have duplicate values?                         │
│  • Is the graph directed or undirected?                  │
│  • Can it have cycles?                                   │
│  • Is it connected or could it have disconnected parts?  │
│  • Are edge weights positive only?                       │
└─────────────────────────────────────────────────────────┘
```

### For General Problems
```
┌─────────────────────────────────────────────────────────┐
│  • What should I return if the answer doesn't exist?     │
│  • Are there any constraints on time/space complexity?   │
│  • Can I use extra space?                                │
│  • Is there a specific output format required?           │
│  • Any constraints I should know about?                  │
└─────────────────────────────────────────────────────────┘
```

---

## How Constraints Guide Your Approach

```
CONSTRAINT → EXPECTED COMPLEXITY → APPROACH

┌──────────────────┬──────────────────┬──────────────────────┐
│  Constraint      │  Complexity      │  Likely Approach      │
├──────────────────┼──────────────────┼──────────────────────┤
│  n ≤ 10          │  O(n!) or O(2^n) │  Brute force / perm  │
│  n ≤ 20          │  O(2^n)          │  Backtracking / bitmask│
│  n ≤ 100         │  O(n^3)          │  3 nested loops / DP  │
│  n ≤ 1,000       │  O(n^2)          │  2 nested loops / DP  │
│  n ≤ 10,000      │  O(n√n) or O(n²) │  Two pointers / DP   │
│  n ≤ 100,000     │  O(n log n)      │  Sort + binary search │
│  n ≤ 1,000,000   │  O(n)            │  Hash map / 1 pass   │
│  n ≤ 10^9        │  O(log n)        │  Binary search / math │
└──────────────────┴──────────────────┴──────────────────────┘

USE THIS TABLE to determine what approach the interviewer expects!
```

---

## Common Misunderstandings

```
┌──────────────────────────────────────────────────────────┐
│  WHAT YOU MIGHT ASSUME  →  WHAT'S ACTUALLY TRUE          │
│                                                          │
│  "Return the values"    →  "Return the indices"          │
│  "Array is sorted"      →  "Array is unsorted"           │
│  "Unique solution"      →  "Multiple solutions possible" │
│  "Positive numbers"     →  "Can be negative"             │
│  "0-indexed"            →  "1-indexed"                   │
│  "Connected graph"      →  "May be disconnected"         │
│  "Binary tree"          →  "Could be n-ary"              │
│                                                          │
│  NEVER ASSUME — ALWAYS ASK                               │
└──────────────────────────────────────────────────────────┘
```

---

## Worked Example: Understanding a Problem

```
PROBLEM: "Given a string s, find the length of the longest 
          substring without repeating characters."

UNDERSTANDING PROCESS:

Step 1: First Read
┌──────────────────────────────────────────────────────┐
│ Find longest substring with no repeating characters  │
└──────────────────────────────────────────────────────┘

Step 2: Clarifying Questions
┌──────────────────────────────────────────────────────┐
│ Q: What characters? → All ASCII characters           │
│ Q: Case sensitive?  → Yes ("A" ≠ "a")              │
│ Q: Empty string?    → Return 0                       │
│ Q: Spaces count?    → Yes, spaces are characters     │
│ Q: Return what?     → The LENGTH, not the substring  │
└──────────────────────────────────────────────────────┘

Step 3: Examples
┌──────────────────────────────────────────────────────┐
│ "abcabcbb" → 3 ("abc")                              │
│ "bbbbb"    → 1 ("b")                                │
│ "pwwkew"   → 3 ("wke")                              │
│ ""         → 0                                       │
│ "a"        → 1                                       │
│ " "        → 1                                       │
└──────────────────────────────────────────────────────┘

Step 4: Restate
┌──────────────────────────────────────────────────────┐
│ "I need to find the longest contiguous sequence of   │
│  characters where no character appears more than     │
│  once, and return its length."                       │
└──────────────────────────────────────────────────────┘

Step 5: Constraint Analysis
┌──────────────────────────────────────────────────────┐
│ 0 ≤ s.length ≤ 5 * 10^4                            │
│ → Need O(n) or O(n log n) solution                  │
│ → Sliding Window pattern likely works                │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Step | Action | Time Spent | Purpose |
|------|--------|:---:|---------|
| 1 | Read problem twice | 1-2 min | Get full picture before jumping in |
| 2 | Ask clarifying questions | 2-3 min | Eliminate assumptions |
| 3 | Identify input/output | 1 min | Know what you're working with |
| 4 | Analyze constraints | 1 min | Determine expected complexity |
| 5 | Restate the problem | 30 sec | Confirm understanding |
| — | **Total** | **5-7 min** | **Invest this time — it saves you 15+ min later** |

---

## Quick Revision Questions

1. **What is the most skipped step in problem-solving, and why does it matter?**
   - Understanding the problem; 80% of failures are from solving the wrong problem, not from inability to code

2. **What five questions should you always ask when given a problem?**
   - What are the inputs? What is the output? What are the constraints? What are the edge cases? Can I restate the problem?

3. **How do constraints like n ≤ 10^6 guide your approach?**
   - They indicate the expected time complexity; n ≤ 10^6 means you need O(n) or O(n log n), not O(n^2)

4. **Why should you restate the problem to the interviewer?**
   - To verify you understood correctly and to avoid wasting 30+ minutes solving the wrong problem

5. **What's one dangerous assumption candidates make about array problems?**
   - Assuming the array is sorted, contains only positive numbers, or has no duplicates — always ask

6. **How much time should you spend on understanding before coding?**
   - 5-7 minutes (25% of a 45-minute interview), which saves 15+ minutes of misdirected coding later

---

[Next: Work Through Examples →](02-work-through-examples.md)

---
[← Back to README](../README.md)
