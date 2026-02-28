# Chapter 8.1: Must-Know Topics

```
╔══════════════════════════════════════════════════════════╗
║          MUST-KNOW TOPICS                                ║
║   The non-negotiable DSA topics for every interview      ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Not all DSA topics are equally important. Some appear in **80%+ of interviews** — these are your must-know topics. Master these first before anything else. This chapter ranks them by frequency and provides a study priority map.

---

## Tier 1: Absolute Must-Know (Appear in 80%+ of Interviews)

```
┌─────────────────────────────────────────────────────────┐
│  1. ARRAYS & STRINGS                                     │
│     - Two Pointers                                       │
│     - Sliding Window                                     │
│     - Prefix Sum                                         │
│     - In-place modifications                             │
│     Key problems: Two Sum, Max Subarray, Valid Anagram    │
│                                                          │
│  2. HASH MAPS / HASH SETS                                │
│     - Frequency counting                                 │
│     - Lookup acceleration                                │
│     - Grouping / anagram detection                       │
│     Key problems: Group Anagrams, Subarray Sum Equals K  │
│                                                          │
│  3. BINARY SEARCH                                        │
│     - Standard search                                    │
│     - Search on answer space                             │
│     - Rotated array variants                             │
│     Key problems: Search Rotated Array, Find Peak        │
│                                                          │
│  4. LINKED LISTS                                         │
│     - Fast/slow pointers                                 │
│     - Reversal                                           │
│     - Merge operations                                   │
│     Key problems: Reverse LL, Detect Cycle, Merge Lists  │
│                                                          │
│  5. TREES (BINARY TREES & BST)                           │
│     - DFS (inorder, preorder, postorder)                 │
│     - BFS (level order)                                  │
│     - BST properties                                     │
│     Key problems: Max Depth, Validate BST, LCA           │
│                                                          │
│  6. RECURSION & BACKTRACKING                             │
│     - Subsets, permutations, combinations                │
│     - Constraint satisfaction                            │
│     Key problems: Subsets, Permutations, N-Queens         │
│                                                          │
│  7. SORTING                                              │
│     - Merge sort, quicksort concepts                     │
│     - Custom comparators                                 │
│     - When to sort as preprocessing                      │
│     Key problems: Merge Intervals, Sort Colors            │
└─────────────────────────────────────────────────────────┘
```

---

## Tier 2: Frequently Asked (Appear in 50-80% of Interviews)

```
┌─────────────────────────────────────────────────────────┐
│  8. STACKS & QUEUES                                      │
│     - Monotonic stack                                    │
│     - Balanced parentheses                               │
│     - Next greater element                               │
│     Key problems: Valid Parentheses, Daily Temperatures  │
│                                                          │
│  9. GRAPHS                                               │
│     - BFS / DFS traversal                                │
│     - Connected components                               │
│     - Topological sort                                   │
│     Key problems: Number of Islands, Course Schedule     │
│                                                          │
│  10. DYNAMIC PROGRAMMING                                 │
│      - 1D DP (climbing stairs, house robber)             │
│      - 2D DP (grid paths, edit distance)                 │
│      - Common patterns (knapsack, LCS, LIS)             │
│      Key problems: Coin Change, Longest Common Subseq   │
│                                                          │
│  11. HEAPS / PRIORITY QUEUES                             │
│      - Top-K problems                                    │
│      - Merge K sorted lists                              │
│      - Stream processing                                 │
│      Key problems: Kth Largest, Merge K Sorted Lists     │
└─────────────────────────────────────────────────────────┘
```

---

## Mastery Levels for Each Topic

```
LEVEL 1 — Recognition (Week 1-2):
  Can identify which technique applies to a problem

LEVEL 2 — Implementation (Week 3-4):
  Can code the standard version from memory

LEVEL 3 — Variation (Week 5-8):
  Can adapt the technique to non-standard problems

LEVEL 4 — Optimization (Week 9-12):
  Can discuss trade-offs and choose optimal approach

TARGET PER TOPIC:
┌─────────────┬────────────────┐
│ Must-Know    │ Level 3-4      │
│ Frequently   │ Level 2-3      │
│ Good-to-Know │ Level 1-2      │
│ Rare         │ Level 1        │
└─────────────┴────────────────┘
```

---

## Problems Per Topic (Minimum Study List)

```
MINIMUM 50-PROBLEM CORE STUDY LIST:

Arrays (8):     Two Sum, Best Time Buy/Sell Stock,
                Container With Most Water, 3Sum,
                Product of Array Except Self,
                Max Subarray, Merge Intervals,
                Rotate Array

Strings (5):    Valid Anagram, Longest Substring
                Without Repeating, Valid Palindrome,
                Longest Palindromic Substring,
                Group Anagrams

Binary Search (4): Binary Search, Search Rotated Array,
                    Find Minimum in Rotated Array,
                    Search 2D Matrix

Linked List (5):   Reverse LL, Merge Two Sorted Lists,
                    Linked List Cycle, Remove Nth Node,
                    Reorder List

Trees (7):     Max Depth, Same Tree, Invert Tree,
               Validate BST, LCA of BST, Level Order,
               Serialize/Deserialize

Recursion (4): Subsets, Permutations, Combination Sum,
               Word Search

Stack (4):     Valid Parentheses, Min Stack,
               Daily Temperatures, Largest Rectangle

Graph (5):     Number of Islands, Clone Graph,
               Course Schedule, Pacific Atlantic Water,
               Word Ladder

DP (5):        Climbing Stairs, Coin Change, House Robber,
               Longest Increasing Subsequence,
               Longest Common Subsequence

Heap (3):      Kth Largest in Stream, Top K Frequent,
               Merge K Sorted Lists
```

---

## Study Priority Matrix

```
HIGH FREQUENCY + EASY TO LEARN → Study First
  Arrays, Hash Maps, Binary Search, Strings

HIGH FREQUENCY + HARD TO LEARN → Study Second
  Trees, Recursion, Linked Lists

MEDIUM FREQUENCY + EASY → Study Third
  Stacks, Heaps, Sorting

MEDIUM FREQUENCY + HARD → Study Last
  Graphs, Dynamic Programming

      │ Easy to Learn  │ Hard to Learn
──────┼────────────────┼──────────────────
High  │ ★ FIRST        │ ★★ SECOND
Freq  │ Arrays, Hash   │ Trees, Recursion
      │ Binary Search  │ Linked Lists
──────┼────────────────┼──────────────────
Med   │ ★★★ THIRD      │ ★★★★ LAST
Freq  │ Stacks, Heaps  │ Graphs, DP
      │ Sorting        │
```

---

## Summary Table

| Topic | Interview Frequency | Study Priority | Min Problems |
|-------|-------------------|----------------|-------------|
| Arrays & Strings | 90%+ | 1st | 13 |
| Hash Maps | 80%+ | 1st | (included in arrays) |
| Binary Search | 70%+ | 1st | 4 |
| Linked Lists | 70%+ | 2nd | 5 |
| Trees (BT/BST) | 80%+ | 2nd | 7 |
| Recursion/Backtracking | 65%+ | 2nd | 4 |
| Stacks & Queues | 55%+ | 3rd | 4 |
| Graphs | 50%+ | 4th | 5 |
| Dynamic Programming | 55%+ | 4th | 5 |
| Heaps | 45%+ | 3rd | 3 |

---

## Quick Revision Questions

1. **What are the absolute must-know topics for DSA interviews?**
   - Arrays/Strings, Hash Maps, Binary Search, Linked Lists, Trees, Recursion/Backtracking, and Sorting — these appear in 80%+ of interviews

2. **Which topics should you study first?**
   - High frequency + easy to learn: Arrays, Hash Maps, Binary Search, Strings

3. **How many problems should you solve at minimum?**
   - About 50 core problems across all topics, with more focus on arrays/strings (13) and trees (7)

4. **What mastery level should you aim for on must-know topics?**
   - Level 3-4: able to adapt techniques to non-standard problems and discuss trade-offs

5. **Which topics are hardest and should be studied last?**
   - Dynamic Programming and Graphs — medium frequency but high difficulty, study them after building strong fundamentals

6. **What's the most frequently tested topic overall?**
   - Arrays and Strings — they form the foundation and appear in some form in nearly every interview

---

[← Previous: When Time Runs Out](../07-Handling-Difficult-Situations/05-when-time-runs-out.md) | [Next: Good-to-Know Topics →](02-good-to-know-topics.md)

---
[← Back to README](../README.md)
