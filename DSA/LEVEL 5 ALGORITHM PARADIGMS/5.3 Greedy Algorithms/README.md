# 5.3 Greedy Algorithms - Complete Study Guide

## Course Overview

Greedy algorithms are a powerful algorithmic paradigm that builds a solution piece by piece, always choosing the next piece that offers the **most immediate benefit** (the locally optimal choice). The key insight is that for certain problems, making the locally optimal choice at each step leads to a **globally optimal solution**.

This comprehensive guide covers greedy algorithms from fundamentals to advanced applications, including proofs of correctness, classic problems, and competitive programming patterns.

```
┌─────────────────────────────────────────────────────────┐
│                  GREEDY PARADIGM                        │
│                                                         │
│   Problem ──► Make Local ──► Build ──► Global           │
│               Best Choice    Solution   Optimum         │
│                                                         │
│   Key Properties:                                       │
│   ✓ Greedy Choice Property                              │
│   ✓ Optimal Substructure                                │
│                                                         │
│   "Think globally, act locally"                         │
└─────────────────────────────────────────────────────────┘
```

---

## Complete Table of Contents

### Unit 1: Greedy Fundamentals
| # | Topic | File |
|---|-------|------|
| 1 | What is Greedy Algorithm? | [01-what-is-greedy-algorithm.md](01-Greedy-Fundamentals/01-what-is-greedy-algorithm.md) |
| 2 | Greedy Choice Property | [02-greedy-choice-property.md](01-Greedy-Fundamentals/02-greedy-choice-property.md) |
| 3 | Optimal Substructure | [03-optimal-substructure.md](01-Greedy-Fundamentals/03-optimal-substructure.md) |
| 4 | When Greedy Works | [04-when-greedy-works.md](01-Greedy-Fundamentals/04-when-greedy-works.md) |
| 5 | When Greedy Fails | [05-when-greedy-fails.md](01-Greedy-Fundamentals/05-when-greedy-fails.md) |
| 6 | Greedy vs Dynamic Programming | [06-greedy-vs-dp.md](01-Greedy-Fundamentals/06-greedy-vs-dp.md) |

### Unit 2: Proving Greedy Correctness
| # | Topic | File |
|---|-------|------|
| 1 | Exchange Argument | [01-exchange-argument.md](02-Proving-Greedy-Correctness/01-exchange-argument.md) |
| 2 | Stays Ahead Argument | [02-stays-ahead-argument.md](02-Proving-Greedy-Correctness/02-stays-ahead-argument.md) |
| 3 | Contradiction Proof | [03-contradiction-proof.md](02-Proving-Greedy-Correctness/03-contradiction-proof.md) |
| 4 | Why Proofs Matter | [04-why-proofs-matter.md](02-Proving-Greedy-Correctness/04-why-proofs-matter.md) |
| 5 | Common Proof Patterns | [05-common-proof-patterns.md](02-Proving-Greedy-Correctness/05-common-proof-patterns.md) |

### Unit 3: Activity Selection
| # | Topic | File |
|---|-------|------|
| 1 | Problem Description | [01-problem-description.md](03-Activity-Selection/01-problem-description.md) |
| 2 | Greedy Approach | [02-greedy-approach.md](03-Activity-Selection/02-greedy-approach.md) |
| 3 | Proof of Correctness | [03-proof-of-correctness.md](03-Activity-Selection/03-proof-of-correctness.md) |
| 4 | Implementation | [04-implementation.md](03-Activity-Selection/04-implementation.md) |
| 5 | Weighted Version | [05-weighted-version.md](03-Activity-Selection/05-weighted-version.md) |
| 6 | Interval Scheduling | [06-interval-scheduling.md](03-Activity-Selection/06-interval-scheduling.md) |

### Unit 4: Interval Problems
| # | Topic | File |
|---|-------|------|
| 1 | Merge Intervals | [01-merge-intervals.md](04-Interval-Problems/01-merge-intervals.md) |
| 2 | Insert Interval | [02-insert-interval.md](04-Interval-Problems/02-insert-interval.md) |
| 3 | Non-Overlapping Intervals | [03-non-overlapping-intervals.md](04-Interval-Problems/03-non-overlapping-intervals.md) |
| 4 | Meeting Rooms Problems | [04-meeting-rooms.md](04-Interval-Problems/04-meeting-rooms.md) |
| 5 | Interval Partitioning | [05-interval-partitioning.md](04-Interval-Problems/05-interval-partitioning.md) |
| 6 | Minimum Arrows | [06-minimum-arrows.md](04-Interval-Problems/06-minimum-arrows.md) |

### Unit 5: Huffman Coding
| # | Topic | File |
|---|-------|------|
| 1 | Prefix Codes | [01-prefix-codes.md](05-Huffman-Coding/01-prefix-codes.md) |
| 2 | Huffman Algorithm | [02-huffman-algorithm.md](05-Huffman-Coding/02-huffman-algorithm.md) |
| 3 | Building Huffman Tree | [03-building-huffman-tree.md](05-Huffman-Coding/03-building-huffman-tree.md) |
| 4 | Encoding and Decoding | [04-encoding-and-decoding.md](05-Huffman-Coding/04-encoding-and-decoding.md) |
| 5 | Optimality Proof | [05-optimality-proof.md](05-Huffman-Coding/05-optimality-proof.md) |
| 6 | Applications | [06-applications.md](05-Huffman-Coding/06-applications.md) |

### Unit 6: Fractional Knapsack
| # | Topic | File |
|---|-------|------|
| 1 | Problem Description | [01-problem-description.md](06-Fractional-Knapsack/01-problem-description.md) |
| 2 | Greedy Approach | [02-greedy-approach.md](06-Fractional-Knapsack/02-greedy-approach.md) |
| 3 | Comparison with 0/1 Knapsack | [03-comparison-with-01-knapsack.md](06-Fractional-Knapsack/03-comparison-with-01-knapsack.md) |
| 4 | Implementation | [04-implementation.md](06-Fractional-Knapsack/04-implementation.md) |
| 5 | Proof of Optimality | [05-proof-of-optimality.md](06-Fractional-Knapsack/05-proof-of-optimality.md) |

### Unit 7: Job Scheduling
| # | Topic | File |
|---|-------|------|
| 1 | Job Sequencing with Deadlines | [01-job-sequencing-with-deadlines.md](07-Job-Scheduling/01-job-sequencing-with-deadlines.md) |
| 2 | Weighted Job Scheduling | [02-weighted-job-scheduling.md](07-Job-Scheduling/02-weighted-job-scheduling.md) |
| 3 | Minimum Platforms | [03-minimum-platforms.md](07-Job-Scheduling/03-minimum-platforms.md) |
| 4 | Task Scheduler | [04-task-scheduler.md](07-Job-Scheduling/04-task-scheduler.md) |
| 5 | Course Schedule III | [05-course-schedule-iii.md](07-Job-Scheduling/05-course-schedule-iii.md) |

### Unit 8: Array Greedy Problems
| # | Topic | File |
|---|-------|------|
| 1 | Jump Game | [01-jump-game.md](08-Array-Greedy-Problems/01-jump-game.md) |
| 2 | Jump Game II | [02-jump-game-ii.md](08-Array-Greedy-Problems/02-jump-game-ii.md) |
| 3 | Gas Station | [03-gas-station.md](08-Array-Greedy-Problems/03-gas-station.md) |
| 4 | Candy Distribution | [04-candy-distribution.md](08-Array-Greedy-Problems/04-candy-distribution.md) |
| 5 | Best Time to Buy/Sell Stock II | [05-best-time-buy-sell-stock-ii.md](08-Array-Greedy-Problems/05-best-time-buy-sell-stock-ii.md) |
| 6 | Maximum Units on Truck | [06-maximum-units-on-truck.md](08-Array-Greedy-Problems/06-maximum-units-on-truck.md) |

### Unit 9: String Greedy
| # | Topic | File |
|---|-------|------|
| 1 | Remove K Digits | [01-remove-k-digits.md](09-String-Greedy/01-remove-k-digits.md) |
| 2 | Remove Duplicate Letters | [02-remove-duplicate-letters.md](09-String-Greedy/02-remove-duplicate-letters.md) |
| 3 | Reorganize String | [03-reorganize-string.md](09-String-Greedy/03-reorganize-string.md) |
| 4 | Smallest String with Swaps | [04-smallest-string-with-swaps.md](09-String-Greedy/04-smallest-string-with-swaps.md) |
| 5 | Partition Labels | [05-partition-labels.md](09-String-Greedy/05-partition-labels.md) |
| 6 | Optimal Partition | [06-optimal-partition.md](09-String-Greedy/06-optimal-partition.md) |

### Unit 10: Graph Greedy
| # | Topic | File |
|---|-------|------|
| 1 | MST Algorithms (Revisited) | [01-mst-algorithms.md](10-Graph-Greedy/01-mst-algorithms.md) |
| 2 | Dijkstra's Algorithm (Revisited) | [02-dijkstras-algorithm.md](10-Graph-Greedy/02-dijkstras-algorithm.md) |
| 3 | Graph Coloring Heuristics | [03-graph-coloring-heuristics.md](10-Graph-Greedy/03-graph-coloring-heuristics.md) |
| 4 | Vertex Cover Approximation | [04-vertex-cover-approximation.md](10-Graph-Greedy/04-vertex-cover-approximation.md) |
| 5 | Set Cover Approximation | [05-set-cover-approximation.md](10-Graph-Greedy/05-set-cover-approximation.md) |

### Unit 11: Advanced Greedy
| # | Topic | File |
|---|-------|------|
| 1 | Huffman with Different Costs | [01-huffman-with-different-costs.md](11-Advanced-Greedy/01-huffman-with-different-costs.md) |
| 2 | Minimum Cost to Connect Sticks | [02-minimum-cost-connect-sticks.md](11-Advanced-Greedy/02-minimum-cost-connect-sticks.md) |
| 3 | Construct Target Array | [03-construct-target-array.md](11-Advanced-Greedy/03-construct-target-array.md) |
| 4 | IPO Problem | [04-ipo-problem.md](11-Advanced-Greedy/04-ipo-problem.md) |
| 5 | Two City Scheduling | [05-two-city-scheduling.md](11-Advanced-Greedy/05-two-city-scheduling.md) |
| 6 | Min Deletions for Unique Frequencies | [06-min-deletions-unique-frequencies.md](11-Advanced-Greedy/06-min-deletions-unique-frequencies.md) |

---

## How to Use This Guide

```
┌──────────────────────────────────────────────────┐
│              RECOMMENDED STUDY PATH              │
│                                                  │
│  Fundamentals ──► Proofs ──► Activity Selection  │
│       │                           │              │
│       ▼                           ▼              │
│  Intervals ──► Huffman ──► Fractional Knapsack   │
│       │                           │              │
│       ▼                           ▼              │
│  Job Scheduling ──► Arrays ──► Strings           │
│       │                           │              │
│       ▼                           ▼              │
│  Graph Greedy ──────────► Advanced Greedy        │
└──────────────────────────────────────────────────┘
```

1. **Start with Unit 1-2** to build strong foundations
2. **Units 3-6** cover classic greedy problems
3. **Units 7-9** focus on coding interview patterns
4. **Units 10-11** cover advanced applications

---

## Quick Reference: Greedy Algorithm Checklist

| Step | Action |
|------|--------|
| 1 | Identify if the problem has **optimal substructure** |
| 2 | Identify the **greedy choice** (locally optimal decision) |
| 3 | Verify the **greedy choice property** holds |
| 4 | Prove correctness (exchange/stays-ahead argument) |
| 5 | Implement: Sort → Iterate → Choose → Build solution |

---

*Happy Learning! Master the art of making the right local choices.* 🚀
