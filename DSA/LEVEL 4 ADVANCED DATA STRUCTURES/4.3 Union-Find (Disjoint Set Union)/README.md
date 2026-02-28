# 4.3 Union-Find (Disjoint Set Union) — Complete Study Guide

## Course Overview

**Union-Find** (also called **Disjoint Set Union** or **DSU**) is one of the most elegant and efficient data structures in computer science. It solves the **dynamic connectivity** problem — determining whether two elements belong to the same group and merging groups together — in nearly constant time per operation.

This guide covers everything from the basic concepts to advanced techniques, with detailed ASCII diagrams, pseudocode traces, complexity analysis, and problem-solving patterns.

```
┌─────────────────────────────────────────────────────┐
│              UNION-FIND AT A GLANCE                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Two Core Operations:                               │
│                                                     │
│  FIND(x)     → Which set does x belong to?          │
│  UNION(x, y) → Merge the sets containing x and y   │
│                                                     │
│  With optimizations:                                │
│    • Path Compression                               │
│    • Union by Rank / Size                           │
│                                                     │
│  Time Complexity: O(α(n)) ≈ O(1) amortized          │
│  Space Complexity: O(n)                             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Complete Table of Contents

### [Unit 1: DSU Fundamentals](01-DSU-Fundamentals/)
| # | Topic | File |
|---|-------|------|
| 1 | What is Union-Find? | [01-what-is-union-find.md](01-DSU-Fundamentals/01-what-is-union-find.md) |
| 2 | Disjoint Sets Concept | [02-disjoint-sets-concept.md](01-DSU-Fundamentals/02-disjoint-sets-concept.md) |
| 3 | Forest Representation | [03-forest-representation.md](01-DSU-Fundamentals/03-forest-representation.md) |
| 4 | Basic Operations | [04-basic-operations.md](01-DSU-Fundamentals/04-basic-operations.md) |
| 5 | Applications Overview | [05-applications-overview.md](01-DSU-Fundamentals/05-applications-overview.md) |
| 6 | When to Use DSU | [06-when-to-use-dsu.md](01-DSU-Fundamentals/06-when-to-use-dsu.md) |

### [Unit 2: Basic Implementation](02-Basic-Implementation/)
| # | Topic | File |
|---|-------|------|
| 1 | Find Operation | [01-find-operation.md](02-Basic-Implementation/01-find-operation.md) |
| 2 | Union Operation | [02-union-operation.md](02-Basic-Implementation/02-union-operation.md) |
| 3 | Naive Implementation | [03-naive-implementation.md](02-Basic-Implementation/03-naive-implementation.md) |
| 4 | Time Complexity (Naive) | [04-time-complexity-naive.md](02-Basic-Implementation/04-time-complexity-naive.md) |
| 5 | Example Walkthrough | [05-example-walkthrough.md](02-Basic-Implementation/05-example-walkthrough.md) |
| 6 | Limitations | [06-limitations.md](02-Basic-Implementation/06-limitations.md) |

### [Unit 3: Path Compression](03-Path-Compression/)
| # | Topic | File |
|---|-------|------|
| 1 | What is Path Compression? | [01-what-is-path-compression.md](03-Path-Compression/01-what-is-path-compression.md) |
| 2 | Implementation Techniques | [02-implementation-techniques.md](03-Path-Compression/02-implementation-techniques.md) |
| 3 | Amortized Complexity | [03-amortized-complexity.md](03-Path-Compression/03-amortized-complexity.md) |
| 4 | Visual Demonstration | [04-visual-demonstration.md](03-Path-Compression/04-visual-demonstration.md) |
| 5 | Path Halving | [05-path-halving.md](03-Path-Compression/05-path-halving.md) |
| 6 | Path Splitting | [06-path-splitting.md](03-Path-Compression/06-path-splitting.md) |

### [Unit 4: Union by Rank/Size](04-Union-By-Rank-Size/)
| # | Topic | File |
|---|-------|------|
| 1 | Union by Rank | [01-union-by-rank.md](04-Union-By-Rank-Size/01-union-by-rank.md) |
| 2 | Union by Size | [02-union-by-size.md](04-Union-By-Rank-Size/02-union-by-size.md) |
| 3 | Why It Helps | [03-why-it-helps.md](04-Union-By-Rank-Size/03-why-it-helps.md) |
| 4 | Implementation | [04-implementation.md](04-Union-By-Rank-Size/04-implementation.md) |
| 5 | Combined with Path Compression | [05-combined-with-path-compression.md](04-Union-By-Rank-Size/05-combined-with-path-compression.md) |
| 6 | Inverse Ackermann Function | [06-inverse-ackermann-function.md](04-Union-By-Rank-Size/06-inverse-ackermann-function.md) |

### [Unit 5: Connected Components](05-Connected-Components/)
| # | Topic | File |
|---|-------|------|
| 1 | Dynamic Connectivity | [01-dynamic-connectivity.md](05-Connected-Components/01-dynamic-connectivity.md) |
| 2 | Number of Components | [02-number-of-components.md](05-Connected-Components/02-number-of-components.md) |
| 3 | Is Connected Queries | [03-is-connected-queries.md](05-Connected-Components/03-is-connected-queries.md) |
| 4 | Size of Components | [04-size-of-components.md](05-Connected-Components/04-size-of-components.md) |
| 5 | Connected Components in Graph | [05-connected-components-in-graph.md](05-Connected-Components/05-connected-components-in-graph.md) |
| 6 | Grid Connectivity | [06-grid-connectivity.md](05-Connected-Components/06-grid-connectivity.md) |

### [Unit 6: Cycle Detection](06-Cycle-Detection/)
| # | Topic | File |
|---|-------|------|
| 1 | Cycle in Undirected Graph | [01-cycle-in-undirected-graph.md](06-Cycle-Detection/01-cycle-in-undirected-graph.md) |
| 2 | Edge Addition and Cycles | [02-edge-addition-and-cycles.md](06-Cycle-Detection/02-edge-addition-and-cycles.md) |
| 3 | Redundant Connection | [03-redundant-connection.md](06-Cycle-Detection/03-redundant-connection.md) |
| 4 | Graph Valid Tree | [04-graph-valid-tree.md](06-Cycle-Detection/04-graph-valid-tree.md) |
| 5 | Implementation Approach | [05-implementation-approach.md](06-Cycle-Detection/05-implementation-approach.md) |

### [Unit 7: MST with Union-Find](07-MST-With-Union-Find/)
| # | Topic | File |
|---|-------|------|
| 1 | Kruskal's Algorithm | [01-kruskals-algorithm.md](07-MST-With-Union-Find/01-kruskals-algorithm.md) |
| 2 | Edge Sorting | [02-edge-sorting.md](07-MST-With-Union-Find/02-edge-sorting.md) |
| 3 | Union-Find Integration | [03-union-find-integration.md](07-MST-With-Union-Find/03-union-find-integration.md) |
| 4 | Minimum Spanning Forest | [04-minimum-spanning-forest.md](07-MST-With-Union-Find/04-minimum-spanning-forest.md) |
| 5 | Implementation Details | [05-implementation-details.md](07-MST-With-Union-Find/05-implementation-details.md) |

### [Unit 8: Advanced DSU](08-Advanced-DSU/)
| # | Topic | File |
|---|-------|------|
| 1 | Weighted Union-Find | [01-weighted-union-find.md](08-Advanced-DSU/01-weighted-union-find.md) |
| 2 | Union-Find with Rollback | [02-union-find-with-rollback.md](08-Advanced-DSU/02-union-find-with-rollback.md) |
| 3 | Offline Queries | [03-offline-queries.md](08-Advanced-DSU/03-offline-queries.md) |
| 4 | Small to Large Merging | [04-small-to-large-merging.md](08-Advanced-DSU/04-small-to-large-merging.md) |
| 5 | Virtual Nodes | [05-virtual-nodes.md](08-Advanced-DSU/05-virtual-nodes.md) |
| 6 | 2-SAT Concept | [06-two-sat-concept.md](08-Advanced-DSU/06-two-sat-concept.md) |

### [Unit 9: Problem Patterns](09-Problem-Patterns/)
| # | Topic | File |
|---|-------|------|
| 1 | Accounts Merge | [01-accounts-merge.md](09-Problem-Patterns/01-accounts-merge.md) |
| 2 | Sentence Similarity | [02-sentence-similarity.md](09-Problem-Patterns/02-sentence-similarity.md) |
| 3 | Regions Cut by Slashes | [03-regions-cut-by-slashes.md](09-Problem-Patterns/03-regions-cut-by-slashes.md) |
| 4 | Most Stones Removed | [04-most-stones-removed.md](09-Problem-Patterns/04-most-stones-removed.md) |
| 5 | Swim in Rising Water | [05-swim-in-rising-water.md](09-Problem-Patterns/05-swim-in-rising-water.md) |
| 6 | Satisfiability of Equations | [06-satisfiability-of-equations.md](09-Problem-Patterns/06-satisfiability-of-equations.md) |

---

## Learning Path

```
START
  │
  ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Unit 1           │────▶│  Unit 2           │────▶│  Unit 3           │
│  DSU Fundamentals │     │  Basic Implement. │     │  Path Compression │
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                                          │
          ┌───────────────────────────────────────────────┘
          ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Unit 4           │────▶│  Unit 5           │────▶│  Unit 6           │
│  Union by Rank    │     │  Connected Comp.  │     │  Cycle Detection  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                                          │
          ┌───────────────────────────────────────────────┘
          ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Unit 7           │────▶│  Unit 8           │────▶│  Unit 9           │
│  MST + Union-Find │     │  Advanced DSU     │     │  Problem Patterns │
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                                          │
                                                          ▼
                                                        DONE ✓
```

---

## Prerequisites

- Arrays and basic data structures
- Recursion fundamentals
- Graph basics (vertices, edges, adjacency)
- Big-O notation understanding

## How to Use This Guide

1. **Follow the units in order** — each builds on the previous
2. **Draw the diagrams yourself** — helps solidify tree structures
3. **Implement from scratch** — don't just read, code it
4. **Answer revision questions** before moving on
5. **Practice the problem patterns** in Unit 9 on LeetCode/Codeforces

---

> **Estimated Study Time:** 15–20 hours for complete coverage  
> **Difficulty Level:** Intermediate to Advanced
