# Chapter 7.1: When Stuck on a Problem

```
╔══════════════════════════════════════════════════════════╗
║          WHEN STUCK ON A PROBLEM                         ║
║   Structured techniques to break through mental blocks   ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Getting stuck is **normal** — even strong candidates encounter problems they can't immediately solve. What matters is how you handle it. This chapter provides a systematic playbook to unblock yourself and demonstrate problem-solving maturity.

---

## The STUCK Framework

```
S — Simplify the problem
T — Try a different representation
U — Use a simpler example
C — Consider similar problems
K — Know when to ask for a hint

Each step takes ~1-2 minutes. If one doesn't help, move to the next.
```

---

## Step 1: Simplify the Problem

```
Ask: Can I solve an easier version?

SIMPLIFICATION STRATEGIES:
┌─────────────────────────────────────────────┐
│ Original Problem     → Simplified Version   │
├─────────────────────────────────────────────┤
│ Any element          → Sorted array         │
│ 2D grid              → 1D array             │
│ Optimal solution     → Any valid solution   │
│ General case         → Fixed-size input     │
│ k operations         → 1 operation          │
│ Weighted graph       → Unweighted graph     │
│ All substrings       → Fixed-length window  │
└─────────────────────────────────────────────┘

EXAMPLE:
  Problem: Find median of two sorted arrays in O(log(m+n))
  Simplified: Find median of one sorted array → O(1)
  Less simplified: Find median when one array has 1 element
  Build up: What if both arrays have 2 elements?
```

---

## Step 2: Try Different Representations

```
VISUALIZATION TECHNIQUES:
  ┌──────────────────┬────────────────────────────────┐
  │ Current View      │ Try Instead                    │
  ├──────────────────┼────────────────────────────────┤
  │ Array             │ Draw as number line            │
  │ Graph             │ Draw as adjacency matrix       │
  │ Tree              │ Write as array (BFS order)     │
  │ String            │ Write as character array       │
  │ Numbers           │ Write in binary                │
  │ Matrix            │ Flatten to 1D                  │
  │ Forward iteration │ Think backwards                │
  └──────────────────┴────────────────────────────────┘

EXAMPLE:
  Problem: Find if path exists in graph
  Stuck with: Adjacency list traversal
  Try: Draw the graph on paper → Pattern becomes visible
```

---

## Step 3: Use a Simpler Example

```
PROGRESSION:
  1. Start with trivially small input (1-2 elements)
  2. Manually solve it
  3. Increase to 3-4 elements
  4. Notice the pattern in your manual solution
  5. The pattern IS the algorithm

EXAMPLE: Longest Increasing Subsequence
  Input: [3]         → LIS = [3], length = 1
  Input: [3, 1]      → LIS = [3] or [1], length = 1
  Input: [3, 1, 4]   → LIS = [3, 4] or [1, 4], length = 2
  Input: [3, 1, 4, 1] → LIS = [3, 4] or [1, 4], length = 2
  
  Pattern: For each element, find longest subsequence ending there
  → DP approach revealed!
```

---

## Step 4: Consider Similar Problems

```
PATTERN MATCHING MENTAL DATABASE:
┌────────────────────────────┬──────────────────────────────┐
│ If problem involves...      │ Think about...               │
├────────────────────────────┼──────────────────────────────┤
│ Finding pairs               │ Two Sum (hash map)           │
│ Sorted array search         │ Binary Search                │
│ Subarray/substring          │ Sliding Window               │
│ Connected components        │ BFS/DFS/Union-Find           │
│ Max/min of something        │ Greedy or DP                 │
│ "Can I achieve X?"          │ Binary Search on answer      │
│ "In how many ways?"         │ DP (counting)                │
│ Generate all combinations   │ Backtracking                 │
│ k-th largest/smallest       │ Heap / Quickselect           │
│ Overlapping intervals       │ Sort + sweep                 │
└────────────────────────────┴──────────────────────────────┘

SAY TO INTERVIEWER:
"This reminds me of [similar problem]. Let me think about whether
that approach applies here..."
```

---

## Step 5: Know When to Ask for Help

```
TIMING:
  ┌──────────────────────────────────────────┐
  │  0-5 min: Understand and plan            │
  │  5-10 min: Code brute force              │
  │  10-15 min: If stuck → ask for hint      │
  │  Don't wait past 15 min of no progress   │
  └──────────────────────────────────────────┘

HOW TO ASK:
  ✗ "I don't know how to solve this."
  ✗ "Can you give me a hint?"
  
  ✓ "I'm considering [approach X] and [approach Y].
     I think X might work because [reason],
     but I'm unsure about [specific part].
     Could you help me think through that?"
  
  ✓ "I have a brute force solution in O(n²).
     I believe we can optimize using [technique],
     but I'm stuck on how to handle [specific case].
     Am I on the right track?"
```

---

## Recovery Techniques

```
TECHNIQUE 1: Work Backwards
  Instead of: How do I get from input → output?
  Try:        What does the last step look like?
  
  Example: Build BST from sorted array
  Last step: Root connects to left and right subtrees
  → Middle element is root → Recursion!

TECHNIQUE 2: Add/Remove Constraints
  Add: "What if the array were sorted?"
  Remove: "What if I had unlimited memory?"
  → Solve the modified problem
  → Adapt back to original

TECHNIQUE 3: Think About Data Structures
  Run through: Array → Hash Map → Stack → Queue
               → Heap → Tree → Graph
  Ask: "Would storing data in [X] make this easier?"

TECHNIQUE 4: Think About Algorithms
  Run through: Sorting → Binary Search → BFS/DFS
               → Two Pointers → Sliding Window → DP
  Ask: "Does [X] technique apply here?"
```

---

## What to Say While Stuck

```
DEMONSTRATING THOUGHT PROCESS:
┌─────────────────────────────────────────────────────────┐
│ "Let me think about what approach might work here..."   │
│ "I'm exploring whether [X] could help with this..."     │
│ "My brute force would be [Y], let me see if I can       │
│  optimize..."                                            │
│ "Let me trace through a small example to build           │
│  intuition..."                                           │
│ "I notice this has [property], which reminds me of..."   │
└─────────────────────────────────────────────────────────┘

NEVER GO SILENT. Thinking out loud:
  - Shows your problem-solving process
  - Gives interviewer opportunity to redirect you
  - Demonstrates persistence and systematic thinking
```

---

## Summary Table

| Technique | When to Use | Time Spent |
|-----------|-------------|------------|
| Simplify | Problem feels overwhelming | 1-2 min |
| Re-represent | Current view isn't helping | 1-2 min |
| Smaller example | Can't see the pattern | 2-3 min |
| Similar problems | Need algorithm inspiration | 1-2 min |
| Ask for help | 10-15 min with no progress | 1 min |

---

## Quick Revision Questions

1. **What's the first thing to try when stuck?**
   - Simplify the problem: remove constraints, reduce dimensions, solve a smaller version first

2. **How do you use smaller examples to get unstuck?**
   - Start with trivially small inputs (1-2 elements), manually solve them, gradually increase size, and look for the pattern in your manual solution

3. **When should you ask for a hint?**
   - After 10-15 minutes of no progress, and after showing your thought process and what you've tried

4. **How should you phrase a request for help?**
   - Share what approaches you've considered, explain your reasoning, and ask about a specific part you're unsure about

5. **What's the 'Work Backwards' technique?**
   - Instead of thinking input→output, consider what the last step looks like and build the algorithm from the end

6. **Why is it important to keep talking when stuck?**
   - It shows your problem-solving process, gives the interviewer chances to help, and demonstrates persistence — silence is the worst response

---

[← Previous: Common Bugs to Avoid](../06-Testing-Your-Solution/05-common-bugs-to-avoid.md) | [Next: When Solution Is Too Slow →](02-when-solution-is-too-slow.md)

---
[← Back to README](../README.md)
