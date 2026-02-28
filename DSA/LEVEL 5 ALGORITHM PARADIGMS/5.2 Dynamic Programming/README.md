# 📘 Dynamic Programming — Complete Study Guide

## Course Overview

**Dynamic Programming (DP)** is one of the most powerful algorithmic paradigms for solving optimization problems. It works by breaking down complex problems into simpler overlapping subproblems, solving each subproblem only once, and storing the results for future use.

This guide covers DP from fundamentals to advanced optimization techniques, with a focus on **concept building**, **pattern recognition**, and **problem-solving strategies**.

```
┌─────────────────────────────────────────────────────────┐
│              DYNAMIC PROGRAMMING ROADMAP                │
│                                                         │
│   Fundamentals ──► Approach ──► 1D Problems             │
│        │                            │                   │
│        ▼                            ▼                   │
│   Fibonacci ──► Grid DP ──► String DP ──► Palindrome    │
│                                              │          │
│                                              ▼          │
│   Knapsack ──► LIS ──► Interval ──► State Machine       │
│                                         │               │
│                                         ▼               │
│                              Bitmask ──► Optimization   │
└─────────────────────────────────────────────────────────┘
```

---

## 📑 Complete Table of Contents

### [Unit 1: DP Fundamentals](01-DP-Fundamentals/)
| # | Topic | File |
|---|-------|------|
| 1 | What is Dynamic Programming? | [01-what-is-dp.md](01-DP-Fundamentals/01-what-is-dp.md) |
| 2 | Overlapping Subproblems | [02-overlapping-subproblems.md](01-DP-Fundamentals/02-overlapping-subproblems.md) |
| 3 | Optimal Substructure | [03-optimal-substructure.md](01-DP-Fundamentals/03-optimal-substructure.md) |
| 4 | Memoization (Top-Down) | [04-memoization.md](01-DP-Fundamentals/04-memoization.md) |
| 5 | Tabulation (Bottom-Up) | [05-tabulation.md](01-DP-Fundamentals/05-tabulation.md) |
| 6 | When to Use DP | [06-when-to-use-dp.md](01-DP-Fundamentals/06-when-to-use-dp.md) |

### [Unit 2: DP Approach](02-DP-Approach/)
| # | Topic | File |
|---|-------|------|
| 1 | Identify the Problem Type | [01-identify-problem-type.md](02-DP-Approach/01-identify-problem-type.md) |
| 2 | Define State | [02-define-state.md](02-DP-Approach/02-define-state.md) |
| 3 | Define Recurrence Relation | [03-define-recurrence-relation.md](02-DP-Approach/03-define-recurrence-relation.md) |
| 4 | Identify Base Cases | [04-identify-base-cases.md](02-DP-Approach/04-identify-base-cases.md) |
| 5 | Determine Order of Computation | [05-order-of-computation.md](02-DP-Approach/05-order-of-computation.md) |
| 6 | Optimize Space | [06-optimize-space.md](02-DP-Approach/06-optimize-space.md) |

### [Unit 3: 1D DP Problems](03-1D-DP-Problems/)
| # | Topic | File |
|---|-------|------|
| 1 | Climbing Stairs | [01-climbing-stairs.md](03-1D-DP-Problems/01-climbing-stairs.md) |
| 2 | House Robber | [02-house-robber.md](03-1D-DP-Problems/02-house-robber.md) |
| 3 | Maximum Subarray (Kadane's) | [03-maximum-subarray.md](03-1D-DP-Problems/03-maximum-subarray.md) |
| 4 | Coin Change (Minimum Coins) | [04-coin-change-min.md](03-1D-DP-Problems/04-coin-change-min.md) |
| 5 | Coin Change 2 (Number of Ways) | [05-coin-change-ways.md](03-1D-DP-Problems/05-coin-change-ways.md) |
| 6 | Decode Ways | [06-decode-ways.md](03-1D-DP-Problems/06-decode-ways.md) |

### [Unit 4: Fibonacci Pattern](04-Fibonacci-Pattern/)
| # | Topic | File |
|---|-------|------|
| 1 | Fibonacci Number | [01-fibonacci-number.md](04-Fibonacci-Pattern/01-fibonacci-number.md) |
| 2 | Tribonacci | [02-tribonacci.md](04-Fibonacci-Pattern/02-tribonacci.md) |
| 3 | House Robber | [03-house-robber.md](04-Fibonacci-Pattern/03-house-robber.md) |
| 4 | Climbing Stairs Variations | [04-climbing-stairs-variations.md](04-Fibonacci-Pattern/04-climbing-stairs-variations.md) |
| 5 | Max Sum Non-Adjacent | [05-max-sum-non-adjacent.md](04-Fibonacci-Pattern/05-max-sum-non-adjacent.md) |
| 6 | Delete and Earn | [06-delete-and-earn.md](04-Fibonacci-Pattern/06-delete-and-earn.md) |

### [Unit 5: Grid DP](05-Grid-DP/)
| # | Topic | File |
|---|-------|------|
| 1 | Unique Paths | [01-unique-paths.md](05-Grid-DP/01-unique-paths.md) |
| 2 | Unique Paths with Obstacles | [02-unique-paths-obstacles.md](05-Grid-DP/02-unique-paths-obstacles.md) |
| 3 | Minimum Path Sum | [03-minimum-path-sum.md](05-Grid-DP/03-minimum-path-sum.md) |
| 4 | Triangle | [04-triangle.md](05-Grid-DP/04-triangle.md) |
| 5 | Dungeon Game | [05-dungeon-game.md](05-Grid-DP/05-dungeon-game.md) |
| 6 | Cherry Pickup | [06-cherry-pickup.md](05-Grid-DP/06-cherry-pickup.md) |

### [Unit 6: String DP](06-String-DP/)
| # | Topic | File |
|---|-------|------|
| 1 | Longest Common Subsequence | [01-longest-common-subsequence.md](06-String-DP/01-longest-common-subsequence.md) |
| 2 | Longest Common Substring | [02-longest-common-substring.md](06-String-DP/02-longest-common-substring.md) |
| 3 | Edit Distance | [03-edit-distance.md](06-String-DP/03-edit-distance.md) |
| 4 | Distinct Subsequences | [04-distinct-subsequences.md](06-String-DP/04-distinct-subsequences.md) |
| 5 | Interleaving String | [05-interleaving-string.md](06-String-DP/05-interleaving-string.md) |
| 6 | Shortest Common Supersequence | [06-shortest-common-supersequence.md](06-String-DP/06-shortest-common-supersequence.md) |

### [Unit 7: Palindrome DP](07-Palindrome-DP/)
| # | Topic | File |
|---|-------|------|
| 1 | Longest Palindromic Subsequence | [01-longest-palindromic-subsequence.md](07-Palindrome-DP/01-longest-palindromic-subsequence.md) |
| 2 | Longest Palindromic Substring | [02-longest-palindromic-substring.md](07-Palindrome-DP/02-longest-palindromic-substring.md) |
| 3 | Palindrome Partitioning II | [03-palindrome-partitioning-ii.md](07-Palindrome-DP/03-palindrome-partitioning-ii.md) |
| 4 | Count Palindromic Substrings | [04-count-palindromic-substrings.md](07-Palindrome-DP/04-count-palindromic-substrings.md) |
| 5 | Palindrome Removal | [05-palindrome-removal.md](07-Palindrome-DP/05-palindrome-removal.md) |

### [Unit 8: Knapsack Patterns](08-Knapsack-Patterns/)
| # | Topic | File |
|---|-------|------|
| 1 | 0/1 Knapsack | [01-01-knapsack.md](08-Knapsack-Patterns/01-01-knapsack.md) |
| 2 | Unbounded Knapsack | [02-unbounded-knapsack.md](08-Knapsack-Patterns/02-unbounded-knapsack.md) |
| 3 | Subset Sum | [03-subset-sum.md](08-Knapsack-Patterns/03-subset-sum.md) |
| 4 | Partition Equal Subset Sum | [04-partition-equal-subset-sum.md](08-Knapsack-Patterns/04-partition-equal-subset-sum.md) |
| 5 | Target Sum | [05-target-sum.md](08-Knapsack-Patterns/05-target-sum.md) |
| 6 | Coin Change Problems | [06-coin-change-problems.md](08-Knapsack-Patterns/06-coin-change-problems.md) |

### [Unit 9: LIS Pattern](09-LIS-Pattern/)
| # | Topic | File |
|---|-------|------|
| 1 | Longest Increasing Subsequence | [01-longest-increasing-subsequence.md](09-LIS-Pattern/01-longest-increasing-subsequence.md) |
| 2 | Number of LIS | [02-number-of-lis.md](09-LIS-Pattern/02-number-of-lis.md) |
| 3 | Russian Doll Envelopes | [03-russian-doll-envelopes.md](09-LIS-Pattern/03-russian-doll-envelopes.md) |
| 4 | Maximum Sum Increasing Subsequence | [04-max-sum-increasing-subsequence.md](09-LIS-Pattern/04-max-sum-increasing-subsequence.md) |
| 5 | Longest Bitonic Subsequence | [05-longest-bitonic-subsequence.md](09-LIS-Pattern/05-longest-bitonic-subsequence.md) |
| 6 | Building Bridges | [06-building-bridges.md](09-LIS-Pattern/06-building-bridges.md) |

### [Unit 10: Interval DP](10-Interval-DP/)
| # | Topic | File |
|---|-------|------|
| 1 | Matrix Chain Multiplication | [01-matrix-chain-multiplication.md](10-Interval-DP/01-matrix-chain-multiplication.md) |
| 2 | Burst Balloons | [02-burst-balloons.md](10-Interval-DP/02-burst-balloons.md) |
| 3 | Minimum Cost to Merge Stones | [03-min-cost-merge-stones.md](10-Interval-DP/03-min-cost-merge-stones.md) |
| 4 | Strange Printer | [04-strange-printer.md](10-Interval-DP/04-strange-printer.md) |
| 5 | Optimal BST | [05-optimal-bst.md](10-Interval-DP/05-optimal-bst.md) |
| 6 | Palindrome Partitioning | [06-palindrome-partitioning.md](10-Interval-DP/06-palindrome-partitioning.md) |

### [Unit 11: State Machine DP](11-State-Machine-DP/)
| # | Topic | File |
|---|-------|------|
| 1 | Buy and Sell Stock Problems | [01-buy-sell-stock.md](11-State-Machine-DP/01-buy-sell-stock.md) |
| 2 | State Transitions | [02-state-transitions.md](11-State-Machine-DP/02-state-transitions.md) |
| 3 | Multiple Transactions | [03-multiple-transactions.md](11-State-Machine-DP/03-multiple-transactions.md) |
| 4 | With Cooldown | [04-with-cooldown.md](11-State-Machine-DP/04-with-cooldown.md) |
| 5 | With Transaction Fee | [05-with-transaction-fee.md](11-State-Machine-DP/05-with-transaction-fee.md) |
| 6 | Pattern Recognition | [06-pattern-recognition.md](11-State-Machine-DP/06-pattern-recognition.md) |

### [Unit 12: Bitmask DP](12-Bitmask-DP/)
| # | Topic | File |
|---|-------|------|
| 1 | Traveling Salesman | [01-traveling-salesman.md](12-Bitmask-DP/01-traveling-salesman.md) |
| 2 | Shortest Path Visiting All Nodes | [02-shortest-path-all-nodes.md](12-Bitmask-DP/02-shortest-path-all-nodes.md) |
| 3 | Hamiltonian Path | [03-hamiltonian-path.md](12-Bitmask-DP/03-hamiltonian-path.md) |
| 4 | Partition to K Equal Sum Subsets | [04-partition-k-equal-sum.md](12-Bitmask-DP/04-partition-k-equal-sum.md) |
| 5 | Parallel Courses II | [05-parallel-courses-ii.md](12-Bitmask-DP/05-parallel-courses-ii.md) |
| 6 | Special Perm | [06-special-perm.md](12-Bitmask-DP/06-special-perm.md) |

### [Unit 13: DP Optimization](13-DP-Optimization/)
| # | Topic | File |
|---|-------|------|
| 1 | Space Optimization Techniques | [01-space-optimization.md](13-DP-Optimization/01-space-optimization.md) |
| 2 | Rolling Array | [02-rolling-array.md](13-DP-Optimization/02-rolling-array.md) |
| 3 | Matrix Exponentiation | [03-matrix-exponentiation.md](13-DP-Optimization/03-matrix-exponentiation.md) |
| 4 | Convex Hull Trick (Concept) | [04-convex-hull-trick.md](13-DP-Optimization/04-convex-hull-trick.md) |
| 5 | Divide & Conquer Optimization (Concept) | [05-divide-conquer-optimization.md](13-DP-Optimization/05-divide-conquer-optimization.md) |
| 6 | Knuth Optimization (Concept) | [06-knuth-optimization.md](13-DP-Optimization/06-knuth-optimization.md) |

---

## 🎯 How to Use This Guide

1. **Start with Units 1–2** to build a strong foundation in DP thinking
2. **Work through Units 3–4** to master basic 1D DP and Fibonacci patterns
3. **Progress to Units 5–7** for 2D DP on grids, strings, and palindromes
4. **Tackle Units 8–10** for classic DP patterns (Knapsack, LIS, Intervals)
5. **Advance to Units 11–12** for state machine and bitmask techniques
6. **Finish with Unit 13** for optimization techniques used in competitive programming

---

## 🧠 Key Study Tips

| Tip | Description |
|-----|-------------|
| **Draw the recursion tree** | Always visualize the recursive calls first |
| **Identify overlapping subproblems** | Look for repeated computations |
| **Start with brute force** | Recursive solution → Memoization → Tabulation |
| **Practice state definition** | The hardest part of DP is defining what state represents |
| **Master transitions** | Once state is defined, focus on how states relate |
| **Optimize last** | Get correctness first, optimize space/time later |

---

## 📊 Complexity Quick Reference

| Pattern | Typical Time | Typical Space |
|---------|-------------|---------------|
| 1D DP | O(n) | O(n) → O(1) |
| Grid DP | O(m×n) | O(m×n) → O(n) |
| String DP | O(m×n) | O(m×n) → O(n) |
| Knapsack | O(n×W) | O(n×W) → O(W) |
| LIS | O(n²) / O(n log n) | O(n) |
| Interval DP | O(n³) | O(n²) |
| Bitmask DP | O(2ⁿ × n) | O(2ⁿ) |

---

> **"Those who cannot remember the past are condemned to repeat it."**
> — Dynamic Programming, literally.

[Begin your journey → Unit 1: DP Fundamentals](01-DP-Fundamentals/01-what-is-dp.md)
