# Chapter 6.1: Generating Test Cases

```
╔══════════════════════════════════════════════════════════╗
║          GENERATING TEST CASES                           ║
║   Systematic test case creation for interviews           ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Testing your solution is the final — and often decisive — step of the interview. Generating good test cases demonstrates **thoroughness, attention to detail, and quality mindset**. This chapter gives you a systematic approach to creating test cases.

---

## The Test Case Generation Framework

```
┌──────────────────────────────────────────────────────────┐
│  THE 5-CATEGORY TEST SYSTEM:                             │
│                                                          │
│  1. HAPPY PATH — Normal, expected inputs                │
│     "The standard case everyone tests"                  │
│                                                          │
│  2. EDGE CASES — Boundary and limit inputs              │
│     "Empty, single, minimum, maximum"                   │
│                                                          │
│  3. LARGE INPUT — Performance/scale test                │
│     "Does it handle n = 10⁵?"                           │
│                                                          │
│  4. SPECIAL PATTERNS — Sorted, reversed, duplicates     │
│     "What if the input has a specific structure?"       │
│                                                          │
│  5. NEGATIVE/IMPOSSIBLE — Invalid or no-answer cases    │
│     "What if no valid answer exists?"                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Category Breakdown with Examples

```
PROBLEM: Two Sum — Find indices of two numbers that add to target

CATEGORY 1: HAPPY PATH
┌──────────────────────────────────────────────┐
│  Input:  nums = [2, 7, 11, 15], target = 9  │
│  Output: [0, 1]                              │
│  Why: Standard case, answer exists           │
└──────────────────────────────────────────────┘

CATEGORY 2: EDGE CASES
┌──────────────────────────────────────────────┐
│  Input:  nums = [3, 3], target = 6           │
│  Output: [0, 1]                              │
│  Why: Duplicate values form the pair         │
│                                              │
│  Input:  nums = [1], target = 2              │
│  Output: []      (or -1 / impossible)        │
│  Why: Single element, can't form pair        │
└──────────────────────────────────────────────┘

CATEGORY 3: LARGE INPUT
┌──────────────────────────────────────────────┐
│  Input:  nums = [1, 2, 3, ..., 100000],     │
│          target = 199999                     │
│  Output: [99998, 99999]                      │
│  Why: Answer is at the very end             │
└──────────────────────────────────────────────┘

CATEGORY 4: SPECIAL PATTERNS
┌──────────────────────────────────────────────┐
│  Input:  nums = [0, 0, 0, 0], target = 0    │
│  Output: [0, 1]                              │
│  Why: All zeros, all duplicates             │
│                                              │
│  Input:  nums = [-3, 4, 3, 90], target = 0  │
│  Output: [0, 2]                              │
│  Why: Negative numbers in input             │
└──────────────────────────────────────────────┘

CATEGORY 5: NEGATIVE/IMPOSSIBLE
┌──────────────────────────────────────────────┐
│  Input:  nums = [1, 2, 3], target = 100     │
│  Output: []                                  │
│  Why: No pair sums to target                │
└──────────────────────────────────────────────┘
```

---

## Quick Test Case Cheat Sheet by Problem Type

```
ARRAY PROBLEMS:
  ✓ Empty array []
  ✓ Single element [5]
  ✓ All same [3, 3, 3]
  ✓ Sorted [1, 2, 3, 4]
  ✓ Reverse sorted [4, 3, 2, 1]
  ✓ With negatives [-2, 0, 3]
  ✓ Answer at start, middle, end

STRING PROBLEMS:
  ✓ Empty string ""
  ✓ Single character "x"
  ✓ All same "aaaa"
  ✓ Palindrome "racecar"
  ✓ With spaces "hello world"
  ✓ Case mixed "aBcDe"

TREE PROBLEMS:
  ✓ Null tree
  ✓ Single node
  ✓ Left-skewed
  ✓ Right-skewed
  ✓ Perfect/balanced tree
  ✓ Negative values

GRAPH PROBLEMS:
  ✓ No nodes
  ✓ Single node
  ✓ Disconnected graph
  ✓ Complete graph
  ✓ Graph with cycle
  ✓ Graph as a tree (no cycles)
```

---

## How to Present Tests in Interview

```
TEMPLATE:
"Let me verify with a few test cases:

 Test 1 (happy path): nums = [2, 7, 11], target = 9
 Expected: [0, 1]
 *traces through code*
 Got: [0, 1] ✓

 Test 2 (edge case): nums = [3, 3], target = 6
 Expected: [0, 1]
 *traces through code*
 Got: [0, 1] ✓

 Test 3 (no answer): nums = [1, 2], target = 10
 Expected: []
 *traces through code*
 Got: [] ✓

 All tests pass. I'm confident in this solution."
```

---

## Summary Table

| Category | Purpose | Example Input |
|----------|---------|---------------|
| Happy path | Verify basic correctness | Standard input with answer |
| Edge cases | Catch boundary bugs | Empty, single, min/max |
| Large input | Verify performance | n = 10⁵ |
| Special patterns | Catch logic errors | All same, sorted, negatives |
| Impossible | Verify "not found" handling | No valid answer exists |

---

## Quick Revision Questions

1. **What are the 5 test case categories?**
   - Happy path, edge cases, large input, special patterns, negative/impossible cases

2. **What's the most important test category in interviews?**
   - Edge cases — they catch the most bugs and show the most thoroughness

3. **How many test cases should you run during an interview?**
   - 3-4 tests: one happy path, one edge case, and one or two special/impossible cases. Don't spend too long on testing

4. **Should you actually trace through test cases or just list them?**
   - Trace through at least 1-2 cases step by step to prove correctness. List others and verify the output matches expectations

5. **When should you generate test cases?**
   - At two points: when understanding the problem (to verify your understanding) and after coding (to verify correctness)

6. **What's the "answer at the end" test pattern?**
   - Test where the valid answer is at the last position — ensures your loop processes all elements and doesn't terminate early

---

[← Previous: Unit 5 — Code Readability](../05-Code-Quality/06-code-readability.md) | [Next: Edge Cases to Consider →](02-edge-cases-to-consider.md)

---
[← Back to README](../README.md)
