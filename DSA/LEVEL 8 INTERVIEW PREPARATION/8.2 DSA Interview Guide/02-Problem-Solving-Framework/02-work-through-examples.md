# Chapter 2.2: Work Through Examples

```
╔══════════════════════════════════════════════════════════╗
║          WORK THROUGH EXAMPLES                          ║
║   Examples are your thinking laboratory                 ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Working through examples before coding is like sketching before painting. It reveals **hidden patterns**, catches **edge cases** you'd miss, and often leads you directly to the algorithm. This chapter shows you how to create and trace examples systematically.

---

## Why Examples Matter

```
┌─────────────────────────────────────────────────────────────┐
│  WITHOUT EXAMPLES:                                          │
│                                                             │
│  Problem ──────────────────────────────────► Code           │
│            (Giant leap — high failure risk)                  │
│                                                             │
│  WITH EXAMPLES:                                             │
│                                                             │
│  Problem ──► Example 1 ──► Pattern ──► Algorithm ──► Code  │
│               Example 2       ↑                             │
│               Example 3 ──────┘                             │
│            (Small steps — each one validates the next)      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The Example Creation Strategy

### Step 1: Start with the Given Example

```
PROBLEM: "Find the first non-repeating character in a string"

GIVEN EXAMPLE:
┌─────────────────────────────────────────────────────────┐
│  Input:  "leetcode"                                     │
│  Output: 0  (character 'l')                             │
│                                                         │
│  Trace it:                                              │
│  Index:  0  1  2  3  4  5  6  7                        │
│  Char:   l  e  e  t  c  o  d  e                        │
│  Count:  1  3  3  1  1  1  1  3                        │
│          ↑                                              │
│          First character with count = 1                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Step 2: Create Your Own Normal Example

```
MY EXAMPLE:
┌─────────────────────────────────────────────────────────┐
│  Input:  "aabb"                                         │
│  Output: -1  (no non-repeating character)               │
│                                                         │
│  Index:  0  1  2  3                                    │
│  Char:   a  a  b  b                                    │
│  Count:  2  2  2  2                                    │
│          All counts > 1 → return -1                     │
│                                                         │
│  INSIGHT: Need to handle "no answer" case               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Step 3: Create Edge Case Examples

```
EDGE CASES:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Empty:     Input: ""        → Output: -1               │
│  Single:    Input: "z"       → Output: 0                │
│  All same:  Input: "aaaa"    → Output: -1               │
│  Last char: Input: "aabbc"   → Output: 4                │
│                                                         │
│  Each edge case reveals a behavior your code MUST handle│
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Step 4: Create a Complex Example

```
COMPLEX CASE:
┌─────────────────────────────────────────────────────────┐
│  Input:  "abaccdeff"                                    │
│                                                         │
│  Index:  0  1  2  3  4  5  6  7  8                     │
│  Char:   a  b  a  c  c  d  e  f  f                    │
│  Count:  2  1  2  2  2  1  1  2  2                    │
│             ↑                                           │
│             First char with count = 1 → return index 1  │
│                                                         │
│  Output: 1                                              │
│                                                         │
│  INSIGHT: I need two passes —                           │
│    Pass 1: count all characters                         │
│    Pass 2: find first with count 1                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Example Types You Should Create

```
┌──────────────────────────────────────────────────────────┐
│  THE 5 TYPES OF EXAMPLES                                 │
│                                                          │
│  1. NORMAL CASE                                          │
│     A straightforward input that should work             │
│     Purpose: Verify basic understanding                  │
│                                                          │
│  2. EDGE CASE                                            │
│     Empty input, single element, maximum size            │
│     Purpose: Find boundary conditions                    │
│                                                          │
│  3. SPECIAL CASE                                         │
│     All same values, sorted, reverse sorted              │
│     Purpose: Find degenerate cases                       │
│                                                          │
│  4. LARGE/COMPLEX CASE                                   │
│     Bigger input that stresses your approach             │
│     Purpose: Verify pattern holds at scale               │
│                                                          │
│  5. NEGATIVE/INVALID CASE                                │
│     Invalid input, no-answer scenario                    │
│     Purpose: Define error handling behavior              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Tracing Through Examples — The Art of Dry Running

### Example: Two Sum Problem

```
PROBLEM: Find two indices in nums where nums[i] + nums[j] = target

Input: nums = [2, 7, 11, 15], target = 9

APPROACH 1: Brute Force (trace)
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  i=0, j=1: nums[0]+nums[1] = 2+7  = 9  ✓  FOUND!     │
│  Return [0, 1]                                          │
│                                                          │
│  (Too easy — let me try a harder example)                │
│                                                          │
└──────────────────────────────────────────────────────────┘

Input: nums = [3, 2, 4], target = 6

BRUTE FORCE TRACE:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  i=0, j=1: nums[0]+nums[1] = 3+2 = 5  ✗               │
│  i=0, j=2: nums[0]+nums[2] = 3+4 = 7  ✗               │
│  i=1, j=2: nums[1]+nums[2] = 2+4 = 6  ✓  FOUND!      │
│  Return [1, 2]                                          │
│                                                          │
│  OBSERVATION: O(n²) — can we do better?                 │
│                                                          │
└──────────────────────────────────────────────────────────┘

APPROACH 2: Hash Map (trace)
┌──────────────────────────────────────────────────────────┐
│  nums = [3, 2, 4], target = 6                           │
│  seen = {}                                               │
│                                                          │
│  i=0: num=3, complement=6-3=3                           │
│       3 in seen? NO → seen = {3: 0}                     │
│                                                          │
│  i=1: num=2, complement=6-2=4                           │
│       4 in seen? NO → seen = {3: 0, 2: 1}              │
│                                                          │
│  i=2: num=4, complement=6-4=2                           │
│       2 in seen? YES! seen[2]=1                         │
│       Return [1, 2]  ✓                                  │
│                                                          │
│  RESULT: O(n) time, O(n) space — much better!           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Pattern Discovery Through Examples

```
PROBLEM: "Maximum subarray sum"

Example 1: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

TRACE (manually finding max subarray):
┌─────────────────────────────────────────────────────────┐
│  Index:    0   1   2   3   4   5   6   7   8           │
│  Value:   -2   1  -3   4  -1   2   1  -5   4          │
│                                                         │
│  Subarrays ending at each position:                     │
│  i=0: [-2]                          → max = -2         │
│  i=1: [-2,1], [1]                   → max = 1          │
│  i=2: [-2,1,-3], [1,-3], [-3]      → max = -2         │
│  i=3: [...,4], [1,-3,4], [-3,4],[4] → max = 4         │
│  i=4: [...,-1], [4,-1]              → max = 3          │
│  i=5: [...,2], [4,-1,2]             → max = 5          │
│  i=6: [...,1], [4,-1,2,1]           → max = 6  ★      │
│  i=7: [...,-5]                       → max = 1         │
│  i=8: [...,4]                        → max = 5         │
│                                                         │
│  PATTERN DISCOVERED:                                    │
│  At each position, max subarray either:                 │
│    1. Extends the previous best, OR                     │
│    2. Starts fresh from current element                 │
│                                                         │
│  current_max = max(nums[i], current_max + nums[i])     │
│                                                         │
│  This IS Kadane's Algorithm!                            │
│  (Discovered just from tracing examples)                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## How to Present Examples to the Interviewer

```
SCRIPT FOR TALKING THROUGH EXAMPLES:

┌─────────────────────────────────────────────────────────┐
│                                                         │
│  1. "Let me work through an example to make sure I     │
│      understand the problem."                           │
│                                                         │
│  2. "For input [2, 7, 11, 15] with target 9..."        │
│      (Write on board/editor)                            │
│                                                         │
│  3. "I need to find two numbers that add to 9.          │
│      2 + 7 = 9, so the answer is indices [0, 1]."     │
│                                                         │
│  4. "Let me try an edge case: what if the array         │
│      has only two elements? [3, 3] with target 6 →     │
│      return [0, 1]. Seems fine."                        │
│                                                         │
│  5. "I notice that for each element, I'm looking for   │
│      its complement (target - element). This suggests   │
│      a hash map approach."                              │
│                                                         │
│  → You've understood the problem AND found the approach │
│    just from examples!                                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Example Table Template

Use this format in interviews:

| Input | Expected Output | Notes |
|-------|----------------|-------|
| `[2,7,11,15], t=9` | `[0,1]` | Normal case |
| `[3,2,4], t=6` | `[1,2]` | Answer not at start |
| `[3,3], t=6` | `[0,1]` | Duplicate values |
| `[1], t=2` | N/A | Invalid (too short) |
| `[-1,-2,-3], t=-4` | `[0,2]` | Negative numbers |

---

## Common Mistakes When Working Examples

```
┌──────────────────────────────────────────────────────────┐
│  MISTAKES TO AVOID                                       │
│                                                          │
│  ✗ Only tracing the given example                        │
│    → Create your own examples to truly understand        │
│                                                          │
│  ✗ Choosing trivial examples (size 1-2)                  │
│    → Use examples with 4-6 elements for pattern finding │
│                                                          │
│  ✗ Skipping the trace when "you see the answer"          │
│    → Trace step by step — it reveals the algorithm      │
│                                                          │
│  ✗ Not doing edge cases                                  │
│    → Empty, single element, all same, no valid answer   │
│                                                          │
│  ✗ Spending too long on examples (>5 min)                │
│    → 3-5 min total for example work is ideal            │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Step | What to Do | Example Count | Purpose |
|------|-----------|:---:|---------|
| 1 | Trace given examples | 1-2 | Verify understanding |
| 2 | Create normal examples | 1-2 | Build intuition |
| 3 | Create edge cases | 2-3 | Find boundary conditions |
| 4 | Discover patterns from traces | — | Guide approach |
| 5 | Verify approach on examples | 1-2 | Validate before coding |
| — | **Total time** | **3-5 min** | **Foundation for correct solution** |

---

## Quick Revision Questions

1. **How many examples should you work through before coding?**
   - 3-5 examples: 1-2 given, 1-2 normal, 2-3 edge cases — taking about 3-5 minutes

2. **What are the 5 types of examples you should create?**
   - Normal, Edge, Special (all same, sorted), Complex (larger), and Negative/Invalid

3. **How can tracing examples reveal the algorithm?**
   - By manually solving step-by-step, you see the pattern your code needs to follow; e.g., the Two Sum trace reveals the hash map approach

4. **What's a common mistake candidates make with examples?**
   - Only tracing the given example without creating their own, missing patterns and edge cases

5. **How should you present examples to the interviewer?**
   - Write them down, trace step by step while narrating, and explicitly state insights or patterns you discover

6. **When should you stop working through examples and start coding?**
   - When you have a clear pattern, can predict the output for new inputs, and have identified key edge cases (typically 3-5 min)

---

[← Previous: Understand the Problem](01-understand-the-problem.md) | [Next: Brute Force First →](03-brute-force-first.md)

---
[← Back to README](../README.md)
