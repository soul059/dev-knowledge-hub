# Chapter 2.3: Brute Force First

```
╔══════════════════════════════════════════════════════════╗
║              BRUTE FORCE FIRST                          ║
║   Start simple, then optimize — never the reverse       ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

**Always start with the brute force solution.** This is not a weakness — it's a strategic advantage. A working brute force gives you a baseline to optimize from, demonstrates your problem-solving process, and ensures you have **something working** even under time pressure.

---

## Why Brute Force First?

```
┌─────────────────────────────────────────────────────────────┐
│  THE STRATEGIC VALUE OF BRUTE FORCE                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 1. CORRECTNESS ANCHOR                               │   │
│  │    A correct O(n²) beats a buggy O(n) every time    │   │
│  │                                                     │   │
│  │ 2. SHOWS UNDERSTANDING                              │   │
│  │    Proves you know what the problem is asking        │   │
│  │                                                     │   │
│  │ 3. OPTIMIZATION STARTING POINT                      │   │
│  │    "Where's the bottleneck?" becomes answerable     │   │
│  │                                                     │   │
│  │ 4. SAFETY NET                                       │   │
│  │    If time runs out, you still have a solution      │   │
│  │                                                     │   │
│  │ 5. INTERVIEW SIGNAL                                 │   │
│  │    Interviewers LOVE seeing brute → optimize flow  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The Brute Force Mindset

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  ASK YOURSELF:                                           │
│                                                          │
│  "What's the simplest, most obvious way to solve this,  │
│   even if it's slow?"                                    │
│                                                          │
│  Common brute force patterns:                            │
│  ┌────────────────────────────────────────────────┐      │
│  │ • Try ALL possible combinations   → O(n², n³)  │      │
│  │ • Check EVERY element against all  → O(n²)     │      │
│  │ • Generate ALL subsets            → O(2ⁿ)      │      │
│  │ • Try ALL permutations            → O(n!)       │      │
│  │ • Sort and scan                   → O(n log n)  │      │
│  │ • Simple recursion without memo   → Exponential │      │
│  └────────────────────────────────────────────────┘      │
│                                                          │
│  Don't worry about efficiency yet — just CORRECTNESS     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Step-by-Step: Building Brute Force Solutions

### Example 1: Two Sum

```
PROBLEM: Find two numbers in array that sum to target

BRUTE FORCE THOUGHT PROCESS:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "The simplest approach? Check EVERY pair."              │
│                                                          │
│  for i in range(n):                                      │
│      for j in range(i+1, n):                            │
│          if nums[i] + nums[j] == target:                │
│              return [i, j]                               │
│                                                          │
│  Time: O(n²)   Space: O(1)                              │
│                                                          │
│  INTERVIEW SCRIPT:                                       │
│  "The brute force would be to check every pair of        │
│   elements. That's O(n²). Let me code this first,       │
│   then we can optimize."                                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Example 2: Maximum Subarray

```
PROBLEM: Find contiguous subarray with max sum

BRUTE FORCE THOUGHT PROCESS:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "Try EVERY possible subarray and find the maximum."    │
│                                                          │
│  PSEUDOCODE:                                             │
│  max_sum = -infinity                                     │
│  for i in range(n):          // start of subarray       │
│      for j in range(i, n):   // end of subarray         │
│          current_sum = sum(nums[i..j])                  │
│          max_sum = max(max_sum, current_sum)            │
│                                                          │
│  Time: O(n³)   Space: O(1)                              │
│                                                          │
│  Can optimize inner loop to O(n²) by keeping running sum:│
│                                                          │
│  for i in range(n):                                      │
│      current_sum = 0                                     │
│      for j in range(i, n):                              │
│          current_sum += nums[j]                          │
│          max_sum = max(max_sum, current_sum)             │
│                                                          │
│  Time: O(n²)   Space: O(1)                              │
│  (Already an optimization! From O(n³) to O(n²))        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Example 3: Longest Common Subsequence

```
PROBLEM: Find LCS of two strings

BRUTE FORCE THOUGHT PROCESS:
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "Generate ALL subsequences of string 1, check each     │
│   against string 2."                                     │
│                                                          │
│  String "abc" has subsequences:                          │
│  "", "a", "b", "c", "ab", "ac", "bc", "abc"            │
│  = 2³ = 8 subsequences                                  │
│                                                          │
│  For each subsequence, check if it's also a              │
│  subsequence of string 2.                                │
│                                                          │
│  Time: O(2ⁿ × m)  where n = len(s1), m = len(s2)      │
│                                                          │
│  "This is exponential, but it gives us the correct      │
│   answer. Now I can see this screams Dynamic             │
│   Programming because of overlapping subproblems."       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## How to Present Brute Force to the Interviewer

```
THE MAGIC SCRIPT:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Step 1: State the approach                                 │
│  "My initial approach would be [brute force description].   │
│   This would be O(___) time and O(___) space."             │
│                                                             │
│  Step 2: Acknowledge the inefficiency                       │
│  "I know this isn't optimal — it's O(n²) when we           │
│   probably want O(n), but let me start with this to         │
│   make sure I have the logic right."                        │
│                                                             │
│  Step 3: Ask for direction                                  │
│  "Would you like me to code this brute force first,         │
│   or should I go ahead and optimize?"                       │
│                                                             │
│  Common interviewer responses:                              │
│  • "Go ahead with brute force" → Code it cleanly           │
│  • "Can you optimize?" → Discuss the optimization           │
│  • "What's the bottleneck?" → Identify optimization point  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Common Brute Force Patterns

```
┌──────────────────────────────────────────────────────────────┐
│  BRUTE FORCE PATTERN LIBRARY                                 │
│                                                              │
│  1. NESTED LOOPS — Check ALL pairs/triples                  │
│     ┌──────────────────────────────────────┐                │
│     │ for i in range(n):                   │                │
│     │     for j in range(i+1, n):          │   O(n²)       │
│     │         check(i, j)                  │                │
│     └──────────────────────────────────────┘                │
│                                                              │
│  2. GENERATE ALL SUBSETS — Bit masking / recursion          │
│     ┌──────────────────────────────────────┐                │
│     │ for mask in range(1 << n):           │                │
│     │     subset = elements where bit is 1 │   O(2ⁿ)      │
│     │     check(subset)                    │                │
│     └──────────────────────────────────────┘                │
│                                                              │
│  3. GENERATE ALL PERMUTATIONS                                │
│     ┌──────────────────────────────────────┐                │
│     │ for perm in permutations(arr):       │                │
│     │     check(perm)                      │   O(n!)       │
│     └──────────────────────────────────────┘                │
│                                                              │
│  4. RECURSIVE EXPLORATION                                    │
│     ┌──────────────────────────────────────┐                │
│     │ def solve(state):                    │                │
│     │     if base_case: return result      │  O(branching^n)│
│     │     for choice in choices:           │                │
│     │         solve(next_state)            │                │
│     └──────────────────────────────────────┘                │
│                                                              │
│  5. SORT + LINEAR SCAN                                       │
│     ┌──────────────────────────────────────┐                │
│     │ arr.sort()                           │                │
│     │ for element in arr:                  │   O(n log n)  │
│     │     process(element)                 │                │
│     └──────────────────────────────────────┘                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Brute Force → Optimization Bridge

```
IDENTIFY THE BOTTLENECK:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Brute Force Code:                                          │
│  ┌──────────────────────────────────────────────────┐      │
│  │  for i in range(n):                   ← Loop 1   │      │
│  │      for j in range(i+1, n):          ← Loop 2   │      │
│  │          if nums[i] + nums[j] == t:   ← Check    │      │
│  │                                                   │      │
│  └──────────────────────────────────────────────────┘      │
│                                                             │
│  Ask: "What is the inner loop REALLY doing?"               │
│                                                             │
│  Answer: "For each nums[i], it's SEARCHING for             │
│           (target - nums[i]) in the rest of the array."    │
│                                                             │
│  Optimization: "Replace the search (O(n)) with a           │
│                 hash map lookup (O(1))."                    │
│                                                             │
│  Result: O(n²) → O(n) ✓                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Common Bottleneck → Optimization Map

```
┌──────────────────────┬─────────────────────────────────────┐
│  Bottleneck          │  Optimization                       │
├──────────────────────┼─────────────────────────────────────┤
│  Searching in loop   │  Hash Map → O(1) lookup             │
│  Nested loops        │  Two Pointers on sorted array       │
│  Repeated subprob.   │  Dynamic Programming / Memoization  │
│  Trying all combos   │  Greedy choice property             │
│  Linear search       │  Binary Search on sorted data       │
│  Recomputing sums    │  Prefix Sum array                   │
│  Full tree traversal │  Pruning conditions                 │
│  Processing all      │  Priority Queue / Heap              │
│  Repeated work       │  Caching / Memoization              │
└──────────────────────┴─────────────────────────────────────┘
```

---

## When NOT to Code Brute Force

```
┌──────────────────────────────────────────────────────────┐
│  SKIP CODING brute force when:                           │
│                                                          │
│  ✗ Brute force is O(n!) and n > 10                      │
│    → too slow to even be useful as a baseline           │
│                                                          │
│  ✗ Interviewer explicitly says "give me an optimal"     │
│    → still MENTION the brute force, just don't code it  │
│                                                          │
│  ✗ You can clearly see the optimal approach              │
│    → mention brute force for comparison, skip to optimal │
│                                                          │
│  BUT ALWAYS:                                             │
│  ✓ MENTION the brute force approach and its complexity   │
│  ✓ Use it as a stepping stone to explain your optimized  │
│    approach                                              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Why brute force first | Correctness anchor, shows process, safety net |
| Brute force mindset | "What's the simplest way, even if slow?" |
| Common patterns | Nested loops, all subsets, all permutations, recursion |
| Interview script | State approach → acknowledge inefficiency → ask direction |
| Bottleneck analysis | Ask "What is the inner loop really doing?" |
| Optimization bridge | Identify bottleneck → replace with efficient data structure |
| When to skip coding it | O(n!) brute force, interviewer says skip, or optimal is obvious |
| Golden rule | Always MENTION brute force even if you don't code it |

---

## Quick Revision Questions

1. **Why should you always start with brute force?**
   - It provides a correctness anchor, demonstrates understanding, creates an optimization baseline, and acts as a safety net if time runs out

2. **What's the "magic script" for presenting brute force?**
   - State the approach + complexity, acknowledge it's not optimal, ask if interviewer wants you to code it or optimize

3. **How do you identify the bottleneck in a brute force solution?**
   - Ask "What is the inner loop really doing?" — usually it's searching, recomputing, or checking something that could be done more efficiently

4. **What optimization replaces searching in an inner loop?**
   - Hash Map — converts O(n) search to O(1) lookup, dropping overall from O(n²) to O(n)

5. **When should you NOT code the brute force?**
   - When it's exponential and clearly impractical, interviewer explicitly asks for optimal, or you clearly see the optimal approach

6. **What's the most common brute force pattern?**
   - Nested loops checking all pairs — O(n²) for two loops, O(n³) for three loops

---

[← Previous: Work Through Examples](02-work-through-examples.md) | [Next: Optimize →](04-optimize.md)

---
[← Back to README](../README.md)
