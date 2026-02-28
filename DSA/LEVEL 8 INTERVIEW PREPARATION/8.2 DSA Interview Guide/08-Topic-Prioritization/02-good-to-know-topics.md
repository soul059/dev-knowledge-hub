# Chapter 8.2: Good-to-Know Topics

```
╔══════════════════════════════════════════════════════════╗
║          GOOD-TO-KNOW TOPICS                             ║
║   Topics that set you apart from average candidates      ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Once you've mastered the must-know topics, these good-to-know topics give you an **edge over other candidates**. They appear in 20-50% of interviews depending on the company and role level. Knowing them shows depth and breadth.

---

## Good-to-Know Topic List

```
┌─────────────────────────────────────────────────────────┐
│  1. TRIES (PREFIX TREES)                                 │
│     Frequency: ~25% of interviews                       │
│     Used for: Autocomplete, word search, prefix matching│
│     Key problems: Implement Trie, Word Search II,        │
│                   Design Search Autocomplete             │
│                                                          │
│  2. UNION-FIND (DISJOINT SET)                           │
│     Frequency: ~20% of interviews                       │
│     Used for: Connected components, cycle detection,     │
│               grouping, minimum spanning tree            │
│     Key problems: Number of Provinces, Redundant         │
│                   Connection, Accounts Merge             │
│                                                          │
│  3. BIT MANIPULATION                                     │
│     Frequency: ~20% of interviews                       │
│     Used for: XOR tricks, power of 2 checks,            │
│               counting bits, masks                       │
│     Key problems: Single Number, Number of 1 Bits,       │
│                   Counting Bits, Reverse Bits            │
│                                                          │
│  4. ADVANCED GRAPH ALGORITHMS                            │
│     Frequency: ~25% of interviews                       │
│     Includes: Dijkstra's, Bellman-Ford, Floyd-Warshall, │
│               Minimum Spanning Tree (Kruskal/Prim)       │
│     Key problems: Network Delay Time, Cheapest Flight,   │
│                   Min Cost to Connect All Points          │
│                                                          │
│  5. ADVANCED TREE STRUCTURES                             │
│     Frequency: ~20% of interviews                       │
│     Includes: AVL trees, Red-Black trees (concepts),    │
│               Segment trees, Binary Indexed Trees (BIT) │
│     Key problems: Range Sum Query, Count of Smaller      │
│                   Numbers After Self                     │
│                                                          │
│  6. GREEDY ALGORITHMS                                    │
│     Frequency: ~35% of interviews                       │
│     Used for: Interval scheduling, activity selection,   │
│               Huffman coding concepts                    │
│     Key problems: Jump Game, Gas Station, Task Scheduler,│
│                   Non-overlapping Intervals              │
│                                                          │
│  7. TWO HEAPS PATTERN                                    │
│     Frequency: ~15% of interviews                       │
│     Used for: Median finding, scheduling                │
│     Key problems: Find Median from Data Stream,          │
│                   Sliding Window Median                   │
│                                                          │
│  8. MONOTONIC STACK / DEQUE                              │
│     Frequency: ~20% of interviews                       │
│     Used for: Next greater/smaller element, sliding      │
│               window max/min                             │
│     Key problems: Largest Rectangle in Histogram,        │
│                   Sliding Window Maximum                  │
└─────────────────────────────────────────────────────────┘
```

---

## When These Topics Come Up

```
CONTEXT WHERE GOOD-TO-KNOW TOPICS APPEAR:

Senior Roles (3+ years):
  → Advanced graphs, system design components
  → Expected to know Union-Find, Tries

FAANG/Big Tech:
  → All good-to-know topics are fair game
  → Greedy and Bit Manipulation very common at Google
  → Tries common at Amazon (autocomplete systems)

Competitive Programming Background:
  → Segment trees, BIT, advanced DP
  → Interviewers may test deeper knowledge

Follow-up Questions:
  → "Can you optimize further?" → Advanced structures
  → "What if data is streaming?" → Two Heaps
  → "What if we need prefix matching?" → Trie
```

---

## Study Guide for Good-to-Know Topics

```
STUDY APPROACH (2-3 problems each):

WEEK 1 — Greedy (most common of the good-to-knows):
  Day 1-2: Jump Game, Jump Game II
  Day 3-4: Task Scheduler, Gas Station
  
WEEK 2 — Tries & Union-Find:
  Day 1-2: Implement Trie, Word Search II
  Day 3-4: Number of Provinces, Accounts Merge

WEEK 3 — Bit Manipulation & Monotonic Stack:
  Day 1-2: Single Number, Counting Bits
  Day 3-4: Daily Temperatures, Largest Rectangle

WEEK 4 — Advanced Graphs & Two Heaps:
  Day 1-2: Network Delay Time (Dijkstra's)
  Day 3-4: Find Median from Data Stream
```

---

## Key Patterns to Recognize

```
TRIGGER → TECHNIQUE:

"Find words with common prefix"     → Trie
"Group connected elements"          → Union-Find
"Single element among pairs"        → XOR (bit manipulation)
"Next greater/smaller element"      → Monotonic Stack
"Running median / top K stream"     → Two Heaps
"Shortest weighted path"            → Dijkstra's
"Choose locally optimal at each step" → Greedy
"Range queries with updates"        → Segment Tree / BIT
```

---

## Summary Table

| Topic | Frequency | Priority After Must-Knows | Problems to Solve |
|-------|-----------|--------------------------|-------------------|
| Greedy | 35% | High | 4 |
| Tries | 25% | Medium | 2-3 |
| Advanced Graphs | 25% | Medium | 2-3 |
| Bit Manipulation | 20% | Medium | 3 |
| Union-Find | 20% | Medium | 2-3 |
| Monotonic Stack/Deque | 20% | Medium | 2-3 |
| Advanced Trees | 20% | Low | 2 |
| Two Heaps | 15% | Low | 1-2 |

---

## Quick Revision Questions

1. **What separates good-to-know from must-know topics?**
   - Must-know topics appear in 80%+ of interviews; good-to-know appear in 20-50% and give you an edge over other candidates

2. **Which good-to-know topic should you study first?**
   - Greedy algorithms — they appear in ~35% of interviews and build on already-known sorting and array concepts

3. **When do Tries come up in interviews?**
   - Prefix matching, autocomplete systems, and word search problems — especially common at companies with search features like Amazon

4. **What's Union-Find used for?**
   - Efficiently grouping connected elements, detecting cycles in undirected graphs, and tracking connected components

5. **When should you learn these topics?**
   - After mastering all must-know topics to Level 3-4, typically weeks 9-12 of a 12-week preparation plan

6. **How many problems should you solve per good-to-know topic?**
   - 2-3 problems each is sufficient — focus on understanding the pattern rather than volume

---

[← Previous: Must-Know Topics](01-must-know-topics.md) | [Next: Rarely Asked Topics →](03-rarely-asked-topics.md)

---
[← Back to README](../README.md)
