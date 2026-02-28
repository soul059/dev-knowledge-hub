# 📚 Introduction to DSA and Complexity Analysis

> **Complete Study Notes** — Concepts, Logic Building & Problem-Solving Approaches

---

## Course Overview

This course lays the **foundation** for understanding Data Structures and Algorithms (DSA). Before diving into specific data structures like arrays, linked lists, or trees, it is essential to understand:

- **What** data structures and algorithms are and **why** they matter
- **How** to measure and compare algorithm efficiency
- **Mathematical tools** needed for rigorous analysis
- **Problem-solving frameworks** used by top engineers

These notes are designed for **deep conceptual understanding** — not just memorization. Every topic includes ASCII diagrams, step-by-step traces, pseudocode, and practice problems.

---

## How to Use These Notes

```
1. Read each unit sequentially — concepts build on each other
2. Trace through every example by hand on paper
3. Attempt the revision questions BEFORE looking at answers
4. Revisit the summary tables for quick revision
5. Practice the suggested problem types
```

---

## Complete Table of Contents

### [Unit 1: Introduction to DSA](01-Introduction-to-DSA/01-introduction-to-dsa.md)
| # | Topic |
|---|-------|
| 1.1 | What is a Data Structure? |
| 1.2 | What is an Algorithm? |
| 1.3 | Why Study DSA? |
| 1.4 | Abstract Data Types (ADT) |
| 1.5 | DSA in Real-World Applications |
| 1.6 | DSA Roadmap for Learning |

### [Unit 2: Algorithm Analysis Basics](02-Algorithm-Analysis-Basics/02-algorithm-analysis-basics.md)
| # | Topic |
|---|-------|
| 2.1 | Why Analyze Algorithms? |
| 2.2 | Time Complexity |
| 2.3 | Space Complexity |
| 2.4 | Best, Average, Worst Case |
| 2.5 | Asymptotic Analysis |
| 2.6 | Comparing Algorithms |

### [Unit 3: Big O Notation](03-Big-O-Notation/03-big-o-notation.md)
| # | Topic |
|---|-------|
| 3.1 | What is Big O? |
| 3.2 | Common Complexities — O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ) |
| 3.3 | Rules for Calculating Big O |
| 3.4 | Drop Constants and Non-Dominant Terms |
| 3.5 | Big O of Nested Loops |
| 3.6 | Big O of Recursive Functions |

### [Unit 4: Big Omega and Big Theta](04-Big-Omega-and-Big-Theta/04-big-omega-and-big-theta.md)
| # | Topic |
|---|-------|
| 4.1 | Big Omega (Ω) — Lower Bound |
| 4.2 | Big Theta (Θ) — Tight Bound |
| 4.3 | Little o and Little omega |
| 4.4 | When to Use Which Notation |
| 4.5 | Practical Implications |

### [Unit 5: Space Complexity](05-Space-Complexity/05-space-complexity.md)
| # | Topic |
|---|-------|
| 5.1 | Auxiliary Space |
| 5.2 | Input Space |
| 5.3 | Stack Space (Recursion) |
| 5.4 | In-Place Algorithms |
| 5.5 | Space-Time Trade-offs |
| 5.6 | Memory Optimization |

### [Unit 6: Recurrence Relations](06-Recurrence-Relations/06-recurrence-relations.md)
| # | Topic |
|---|-------|
| 6.1 | What is a Recurrence Relation? |
| 6.2 | Substitution Method |
| 6.3 | Recursion Tree Method |
| 6.4 | Master Theorem |
| 6.5 | Examples with Common Algorithms |
| 6.6 | Solving Complex Recurrences |

### [Unit 7: Mathematical Foundations](07-Mathematical-Foundations/07-mathematical-foundations.md)
| # | Topic |
|---|-------|
| 7.1 | Logarithms Review |
| 7.2 | Arithmetic and Geometric Series |
| 7.3 | Permutations and Combinations |
| 7.4 | Probability Basics |
| 7.5 | Proof Techniques |
| 7.6 | Modular Arithmetic |

### [Unit 8: Problem-Solving Approach](08-Problem-Solving-Approach/08-problem-solving-approach.md)
| # | Topic |
|---|-------|
| 8.1 | Understand the Problem |
| 8.2 | Example Walkthrough |
| 8.3 | Brute Force First |
| 8.4 | Optimize Step by Step |
| 8.5 | Code and Test |
| 8.6 | Analyze Complexity |

---

## Quick Reference — Complexity Cheat Sheet

```
┌──────────────┬───────────────┬─────────────────────────────────┐
│  Complexity  │     Name      │         Example                 │
├──────────────┼───────────────┼─────────────────────────────────┤
│   O(1)       │  Constant     │  Array access by index          │
│   O(log n)   │  Logarithmic  │  Binary search                  │
│   O(n)       │  Linear       │  Linear search                  │
│   O(n log n) │  Linearithmic │  Merge sort                     │
│   O(n²)      │  Quadratic    │  Bubble sort                    │
│   O(2ⁿ)      │  Exponential  │  Recursive Fibonacci            │
│   O(n!)      │  Factorial    │  Generating all permutations    │
└──────────────┴───────────────┴─────────────────────────────────┘
```

---

## Growth Rate Visualization

```
Time ▲
     │                                              * 2ⁿ
     │                                         *
     │                                    *
     │                               *         * n²
     │                          *         *
     │                     *         *
     │                *         *
     │           *    ·    *              · n log n
     │      *    ·    *              ·
     │ *    ·    *              ·
     │ ·    *              ·                  - n
     │ · *            ·
     │ *          ·                            . log n
     │·      ·
     │·  ·  . . . . . . . . . . . . . . . .   __ 1
     │· . . . . . . . . . . . . . . . . . .
     └──────────────────────────────────────► Input Size (n)
```

---

## Prerequisites

- Basic programming knowledge (any language)
- Understanding of variables, loops, conditionals, functions
- Willingness to think mathematically

---

## Legend / Conventions Used

| Symbol | Meaning |
|--------|---------|
| `n` | Input size |
| `T(n)` | Time as a function of input size |
| `S(n)` | Space as a function of input size |
| `→` | "leads to" or "implies" |
| `≈` | "approximately equal to" |
| `⌊x⌋` | Floor of x |
| `⌈x⌉` | Ceiling of x |
| `log` | Logarithm base 2 (unless stated) |

---

> **Start here →** [Unit 1: Introduction to DSA](01-Introduction-to-DSA/01-introduction-to-dsa.md)
