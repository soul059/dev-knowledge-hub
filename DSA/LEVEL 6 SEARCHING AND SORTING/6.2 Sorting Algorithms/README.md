# 📊 Sorting Algorithms — Complete Study Notes

> **Premium Quality DSA Notes** | Concepts · Logic Building · Problem Solving

---

## Course Overview

Sorting is one of the most fundamental operations in computer science. Nearly every real-world application—databases, search engines, operating systems, graphics—relies on efficient sorting. Mastering sorting algorithms builds critical thinking about **time-space tradeoffs**, **divide and conquer**, **greedy strategies**, and **algorithmic design paradigms**.

These notes cover **12 comprehensive units**, progressing from foundational concepts through elementary sorts, advanced divide-and-conquer sorts, non-comparison-based sorts, and finally practical problem-solving patterns.

```
  ┌─────────────────────────────────────────────────────────┐
  │                SORTING ALGORITHMS ROADMAP                │
  ├─────────────────────────────────────────────────────────┤
  │                                                         │
  │   Fundamentals ──► Elementary Sorts ──► Advanced Sorts  │
  │       │              (Bubble, Selection,   (Merge,      │
  │       │               Insertion)            Quick,      │
  │       │                                     Heap)       │
  │       │                                       │         │
  │       ▼                                       ▼         │
  │   Complexity        Non-Comparison Sorts   Comparison   │
  │   Analysis          (Counting, Radix,      & Special    │
  │                      Bucket)               Problems     │
  └─────────────────────────────────────────────────────────┘
```

---

## 📑 Complete Table of Contents

### [Unit 1: Sorting Fundamentals](01-Sorting-Fundamentals/)
| # | Topic | File |
|---|-------|------|
| 1 | Why Sorting Matters | [01-why-sorting-matters.md](01-Sorting-Fundamentals/01-why-sorting-matters.md) |
| 2 | In-place vs Out-of-place | [02-inplace-vs-outofplace.md](01-Sorting-Fundamentals/02-inplace-vs-outofplace.md) |
| 3 | Stable vs Unstable | [03-stable-vs-unstable.md](01-Sorting-Fundamentals/03-stable-vs-unstable.md) |
| 4 | Comparison vs Non-comparison | [04-comparison-vs-noncomparison.md](01-Sorting-Fundamentals/04-comparison-vs-noncomparison.md) |
| 5 | Internal vs External Sorting | [05-internal-vs-external.md](01-Sorting-Fundamentals/05-internal-vs-external.md) |
| 6 | Sorting Complexity Lower Bound | [06-sorting-lower-bound.md](01-Sorting-Fundamentals/06-sorting-lower-bound.md) |

### [Unit 2: Bubble Sort](02-Bubble-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](02-Bubble-Sort/01-algorithm-description.md) |
| 2 | Implementation | [02-implementation.md](02-Bubble-Sort/02-implementation.md) |
| 3 | Optimization (Early Termination) | [03-optimization.md](02-Bubble-Sort/03-optimization.md) |
| 4 | Complexity Analysis | [04-complexity-analysis.md](02-Bubble-Sort/04-complexity-analysis.md) |
| 5 | When to Use | [05-when-to-use.md](02-Bubble-Sort/05-when-to-use.md) |
| 6 | Variations | [06-variations.md](02-Bubble-Sort/06-variations.md) |

### [Unit 3: Selection Sort](03-Selection-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](03-Selection-Sort/01-algorithm-description.md) |
| 2 | Implementation | [02-implementation.md](03-Selection-Sort/02-implementation.md) |
| 3 | Complexity Analysis | [03-complexity-analysis.md](03-Selection-Sort/03-complexity-analysis.md) |
| 4 | Stability Consideration | [04-stability-consideration.md](03-Selection-Sort/04-stability-consideration.md) |
| 5 | Comparison with Bubble Sort | [05-comparison-with-bubble.md](03-Selection-Sort/05-comparison-with-bubble.md) |

### [Unit 4: Insertion Sort](04-Insertion-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](04-Insertion-Sort/01-algorithm-description.md) |
| 2 | Implementation | [02-implementation.md](04-Insertion-Sort/02-implementation.md) |
| 3 | Complexity Analysis | [03-complexity-analysis.md](04-Insertion-Sort/03-complexity-analysis.md) |
| 4 | Best Case Behavior | [04-best-case-behavior.md](04-Insertion-Sort/04-best-case-behavior.md) |
| 5 | When Insertion Sort Shines | [05-when-insertion-sort-shines.md](04-Insertion-Sort/05-when-insertion-sort-shines.md) |
| 6 | Binary Insertion Sort | [06-binary-insertion-sort.md](04-Insertion-Sort/06-binary-insertion-sort.md) |

### [Unit 5: Merge Sort](05-Merge-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](05-Merge-Sort/01-algorithm-description.md) |
| 2 | Divide and Conquer Approach | [02-divide-and-conquer.md](05-Merge-Sort/02-divide-and-conquer.md) |
| 3 | Merge Process | [03-merge-process.md](05-Merge-Sort/03-merge-process.md) |
| 4 | Implementation | [04-implementation.md](05-Merge-Sort/04-implementation.md) |
| 5 | Complexity Analysis | [05-complexity-analysis.md](05-Merge-Sort/05-complexity-analysis.md) |
| 6 | Stability | [06-stability.md](05-Merge-Sort/06-stability.md) |

### [Unit 6: Quick Sort](06-Quick-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](06-Quick-Sort/01-algorithm-description.md) |
| 2 | Partition Process | [02-partition-process.md](06-Quick-Sort/02-partition-process.md) |
| 3 | Pivot Selection Strategies | [03-pivot-selection.md](06-Quick-Sort/03-pivot-selection.md) |
| 4 | Implementation | [04-implementation.md](06-Quick-Sort/04-implementation.md) |
| 5 | Complexity Analysis | [05-complexity-analysis.md](06-Quick-Sort/05-complexity-analysis.md) |
| 6 | Randomization | [06-randomization.md](06-Quick-Sort/06-randomization.md) |

### [Unit 7: Heap Sort](07-Heap-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](07-Heap-Sort/01-algorithm-description.md) |
| 2 | Heapify Process | [02-heapify-process.md](07-Heap-Sort/02-heapify-process.md) |
| 3 | Implementation | [03-implementation.md](07-Heap-Sort/03-implementation.md) |
| 4 | Complexity Analysis | [04-complexity-analysis.md](07-Heap-Sort/04-complexity-analysis.md) |
| 5 | In-place Property | [05-inplace-property.md](07-Heap-Sort/05-inplace-property.md) |
| 6 | Comparison with Others | [06-comparison-with-others.md](07-Heap-Sort/06-comparison-with-others.md) |

### [Unit 8: Counting Sort](08-Counting-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](08-Counting-Sort/01-algorithm-description.md) |
| 2 | When to Use | [02-when-to-use.md](08-Counting-Sort/02-when-to-use.md) |
| 3 | Implementation | [03-implementation.md](08-Counting-Sort/03-implementation.md) |
| 4 | Complexity Analysis | [04-complexity-analysis.md](08-Counting-Sort/04-complexity-analysis.md) |
| 5 | Stability | [05-stability.md](08-Counting-Sort/05-stability.md) |
| 6 | Limitations | [06-limitations.md](08-Counting-Sort/06-limitations.md) |

### [Unit 9: Radix Sort](09-Radix-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](09-Radix-Sort/01-algorithm-description.md) |
| 2 | LSD vs MSD | [02-lsd-vs-msd.md](09-Radix-Sort/02-lsd-vs-msd.md) |
| 3 | Implementation | [03-implementation.md](09-Radix-Sort/03-implementation.md) |
| 4 | Complexity Analysis | [04-complexity-analysis.md](09-Radix-Sort/04-complexity-analysis.md) |
| 5 | Use Cases | [05-use-cases.md](09-Radix-Sort/05-use-cases.md) |
| 6 | Radix for Strings | [06-radix-for-strings.md](09-Radix-Sort/06-radix-for-strings.md) |

### [Unit 10: Bucket Sort](10-Bucket-Sort/)
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Description | [01-algorithm-description.md](10-Bucket-Sort/01-algorithm-description.md) |
| 2 | When to Use | [02-when-to-use.md](10-Bucket-Sort/02-when-to-use.md) |
| 3 | Implementation | [03-implementation.md](10-Bucket-Sort/03-implementation.md) |
| 4 | Complexity Analysis | [04-complexity-analysis.md](10-Bucket-Sort/04-complexity-analysis.md) |
| 5 | Choosing Bucket Count | [05-choosing-bucket-count.md](10-Bucket-Sort/05-choosing-bucket-count.md) |
| 6 | Hybrid Approaches | [06-hybrid-approaches.md](10-Bucket-Sort/06-hybrid-approaches.md) |

### [Unit 11: Sorting Algorithm Comparison](11-Sorting-Algorithm-Comparison/)
| # | Topic | File |
|---|-------|------|
| 1 | Time Complexity Comparison | [01-time-complexity-comparison.md](11-Sorting-Algorithm-Comparison/01-time-complexity-comparison.md) |
| 2 | Space Complexity Comparison | [02-space-complexity-comparison.md](11-Sorting-Algorithm-Comparison/02-space-complexity-comparison.md) |
| 3 | Stability Comparison | [03-stability-comparison.md](11-Sorting-Algorithm-Comparison/03-stability-comparison.md) |
| 4 | When to Use Which | [04-when-to-use-which.md](11-Sorting-Algorithm-Comparison/04-when-to-use-which.md) |
| 5 | Practical Considerations | [05-practical-considerations.md](11-Sorting-Algorithm-Comparison/05-practical-considerations.md) |
| 6 | Language Library Implementations | [06-language-library-implementations.md](11-Sorting-Algorithm-Comparison/06-language-library-implementations.md) |

### [Unit 12: Special Sorting Problems](12-Special-Sorting-Problems/)
| # | Topic | File |
|---|-------|------|
| 1 | Sort Colors (Dutch National Flag) | [01-sort-colors.md](12-Special-Sorting-Problems/01-sort-colors.md) |
| 2 | Sort by Parity | [02-sort-by-parity.md](12-Special-Sorting-Problems/02-sort-by-parity.md) |
| 3 | Sort Array by Frequency | [03-sort-by-frequency.md](12-Special-Sorting-Problems/03-sort-by-frequency.md) |
| 4 | Relative Sort Array | [04-relative-sort-array.md](12-Special-Sorting-Problems/04-relative-sort-array.md) |
| 5 | Custom Comparators | [05-custom-comparators.md](12-Special-Sorting-Problems/05-custom-comparators.md) |
| 6 | Sorting Linked Lists | [06-sorting-linked-lists.md](12-Special-Sorting-Problems/06-sorting-linked-lists.md) |

---

## 🎯 How to Use These Notes

1. **Sequential Study**: Follow units 1–12 in order for a structured learning path
2. **Quick Reference**: Jump to any topic using the table of contents
3. **Revision Mode**: Use the summary tables and revision questions at the end of each chapter
4. **Problem Solving**: Unit 12 covers practical interview-style sorting problems

## 📊 Master Complexity Cheat Sheet

```
┌──────────────────┬──────────────┬──────────────┬──────────────┬───────┬────────┐
│    Algorithm     │  Best Case   │  Avg Case    │  Worst Case  │ Space │ Stable │
├──────────────────┼──────────────┼──────────────┼──────────────┼───────┼────────┤
│ Bubble Sort      │    O(n)      │    O(n²)     │    O(n²)     │ O(1)  │  Yes   │
│ Selection Sort   │    O(n²)     │    O(n²)     │    O(n²)     │ O(1)  │  No    │
│ Insertion Sort   │    O(n)      │    O(n²)     │    O(n²)     │ O(1)  │  Yes   │
│ Merge Sort       │  O(n log n)  │  O(n log n)  │  O(n log n)  │ O(n)  │  Yes   │
│ Quick Sort       │  O(n log n)  │  O(n log n)  │    O(n²)     │O(logn)│  No    │
│ Heap Sort        │  O(n log n)  │  O(n log n)  │  O(n log n)  │ O(1)  │  No    │
│ Counting Sort    │   O(n+k)     │   O(n+k)     │   O(n+k)     │O(n+k) │  Yes   │
│ Radix Sort       │   O(d·n)     │   O(d·n)     │   O(d·n)     │O(n+k) │  Yes   │
│ Bucket Sort      │   O(n+k)     │   O(n+k)     │    O(n²)     │O(n+k) │  Yes   │
└──────────────────┴──────────────┴──────────────┴──────────────┴───────┴────────┘
```

---

## Prerequisites

- Arrays and basic data structures
- Recursion fundamentals
- Big-O notation basics
- Basic programming in any language (examples use pseudocode + Java/Python)

---

> **Start your journey** → [Unit 1: Sorting Fundamentals](01-Sorting-Fundamentals/01-why-sorting-matters.md)
