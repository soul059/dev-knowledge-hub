# Chapter 3.3: Explaining Your Approach

```
╔══════════════════════════════════════════════════════════╗
║          EXPLAINING YOUR APPROACH                        ║
║   Turning ideas into structured explanations             ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

After understanding the problem and asking clarifying questions, you need to **explain your planned approach before writing any code**. This is one of the most important moments in the interview — it shows your problem-solving ability, communication skills, and whether your solution will actually work.

---

## The Approach Explanation Framework

```
┌─────────────────────────────────────────────────────────────┐
│            THE 4-STEP EXPLANATION                           │
│                                                             │
│  STEP 1: STATE THE APPROACH (1 sentence)                   │
│  "I'm going to use a two-pointer technique on a sorted     │
│   array."                                                   │
│                                                             │
│  STEP 2: EXPLAIN THE INTUITION (2-3 sentences)             │
│  "The key insight is that if we sort the array, we can      │
│   use two pointers moving inward to find pairs. If the      │
│   sum is too large, we move the right pointer left."        │
│                                                             │
│  STEP 3: WALK THROUGH AN EXAMPLE (30-60 seconds)           │
│  "For [2,7,11,15] with target 9: left=2, right=15,        │
│   sum=17 > 9, so right moves. Now left=2, right=11,        │
│   sum=13 > 9, right moves. left=2, right=7, sum=9. ✓"     │
│                                                             │
│  STEP 4: STATE COMPLEXITY (1 sentence)                     │
│  "This gives us O(n log n) for sorting plus O(n) for       │
│   the scan, so O(n log n) time and O(1) extra space."      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## How to Describe Common Algorithms

```
HASH MAP APPROACH:
"I'll iterate through the array once. For each element, I'll
 check if its complement (target - current) exists in a hash
 map. If yes, we found our pair. If no, I'll store the current
 element. This gives O(n) time with O(n) space."

TWO POINTERS:
"I'll sort the array and use two pointers — one at the start,
 one at the end. If their sum is too large, I'll move the right
 pointer left. If too small, I'll move left pointer right. This
 narrows the search space each step. O(n log n) time, O(1) space."

SLIDING WINDOW:
"I'll maintain a window that expands to include elements and
 contracts when the window condition is violated. I'll track the
 best answer seen so far. This processes each element at most
 twice, giving O(n) time."

BFS/DFS:
"I'll model this as a graph traversal problem. Starting from
 [source], I'll explore neighbors level by level using BFS. I'll
 use a visited set to avoid cycles. Each node and edge is
 processed once, giving O(V + E) time."

BINARY SEARCH:
"Since the search space is sorted/monotonic, I can binary search
 for the answer. Each step eliminates half the remaining space,
 giving O(log n) time."

DYNAMIC PROGRAMMING:
"I notice overlapping subproblems: the answer for position i
 depends on previous positions. I'll build a DP table bottom-up
 where dp[i] represents [meaning]. The recurrence is
 dp[i] = [formula]. This gives O(n) time and space."
```

---

## Approach Explanation Flow

```
┌─────────────────┐
│ "Let me think    │
│  about this..."  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌──────────────────────────┐
│ Identify Pattern │────→│ "This reminds me of the  │
│ or Technique     │     │  two-pointer pattern..."  │
└────────┬────────┘     └──────────────────────────┘
         │
         ▼
┌─────────────────┐     ┌──────────────────────────┐
│ Explain Key      │────→│ "The key insight is that  │
│ Insight          │     │  sorting lets us..."      │
└────────┬────────┘     └──────────────────────────┘
         │
         ▼
┌─────────────────┐     ┌──────────────────────────┐
│ Walk Through     │────→│ "For example, with input  │
│ Example          │     │  [1,3,5], we first..."    │
└────────┬────────┘     └──────────────────────────┘
         │
         ▼
┌─────────────────┐     ┌──────────────────────────┐
│ State Complexity │────→│ "So overall, O(n) time    │
│ & Ask to Code    │     │  and O(n) space. Shall I  │
└─────────────────┘     │  code this up?"           │
                         └──────────────────────────┘
```

---

## Bad vs Good Approach Explanations

```
BAD (vague, unstructured):
┌──────────────────────────────────────────────────────────┐
│ "I think maybe I'll use a loop... and then check each   │
│  element... and somehow find the pair... let me just     │
│  start coding and figure it out."                        │
│                                                          │
│  ✗ No clear technique identified                        │
│  ✗ No intuition for why it works                        │
│  ✗ No example walkthrough                               │
│  ✗ No complexity analysis                               │
│  ✗ Jumped straight to coding                            │
└──────────────────────────────────────────────────────────┘

GOOD (structured, clear):
┌──────────────────────────────────────────────────────────┐
│ "I'll use a hash map approach. The idea is: for each    │
│  number, its complement (target - number) either exists  │
│  in our map or it doesn't. We build the map as we go.   │
│                                                          │
│  For example: nums=[2,7,11], target=9                   │
│  - See 2, complement is 7, not in map. Store {2:0}     │
│  - See 7, complement is 2, IS in map! Return [0,1]     │
│                                                          │
│  This is O(n) time — one pass — and O(n) space for     │
│  the hash map. Shall I code this up?"                   │
│                                                          │
│  ✓ Clear technique                                      │
│  ✓ Intuition explained                                  │
│  ✓ Example walkthrough                                  │
│  ✓ Complexity stated                                    │
│  ✓ Asked before coding                                  │
└──────────────────────────────────────────────────────────┘
```

---

## When to Present Multiple Approaches

```
RECOMMENDED FLOW:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "I see two approaches here:"                           │
│                                                          │
│  APPROACH 1 (Brute Force):                              │
│  "Check all pairs with nested loops."                   │
│  "O(n²) time, O(1) space."                             │
│                                                          │
│  APPROACH 2 (Optimized):                                │
│  "Use a hash map for single-pass lookup."               │
│  "O(n) time, O(n) space."                              │
│                                                          │
│  "I'd go with Approach 2 since it's more efficient.    │
│   Does that sound good, or would you like me to         │
│   implement the brute force first?"                     │
│                                                          │
│  → This shows breadth of knowledge                      │
│  → Lets interviewer guide you                           │
│  → Demonstrates you can compare solutions               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Handling Uncertainty

```
WHEN YOU'RE NOT 100% SURE:

"I think a sliding window might work here. Let me verify
 with an example before committing to this approach..."
→ Walk through example → Confirm or pivot

"My initial thought is to use BFS, but I want to consider
 whether DFS might be better for this problem because..."
→ Compare briefly → Choose one → Proceed

DON'T DO:
✗ "I have no idea what to do."
✗ Long silence with no communication
✗ Starting to code without explaining anything
✗ Explaining 5 different approaches (wastes time)
```

---

## Summary Table

| What to Explain | Time to Spend | Example |
|-----------------|---------------|---------|
| Technique name | 5 seconds | "I'll use a hash map" |
| Key insight | 15-20 seconds | "Each element's complement either exists or doesn't" |
| Example walkthrough | 30-60 seconds | Trace through a small input |
| Complexity | 10 seconds | "O(n) time, O(n) space" |
| Ask to proceed | 5 seconds | "Shall I code this?" |
| **Total** | **~2 minutes** | |

---

## Quick Revision Questions

1. **What are the 4 steps of explaining your approach?**
   - State the technique → Explain the intuition → Walk through an example → State complexity

2. **Should you explain your approach before or after coding?**
   - BEFORE coding. Get confirmation from the interviewer before investing time in code

3. **How long should your approach explanation take?**
   - About 2 minutes total — concise enough to leave time for coding, detailed enough to demonstrate understanding

4. **Should you present multiple approaches?**
   - Yes, briefly mention brute force and optimized, then recommend one. Shows breadth of knowledge and comparison ability

5. **What's the most common mistake when explaining approach?**
   - Being too vague — saying "I'll use a loop" without explaining WHY or what the key insight is

6. **What should you do when you're unsure about your approach?**
   - Say "I think X might work, let me verify with an example" — then trace through. This is honest and shows methodical thinking

---

[← Previous: Asking Clarifying Questions](02-asking-clarifying-questions.md) | [Next: Discussing Trade-Offs →](04-discussing-trade-offs.md)

---
[← Back to README](../README.md)
