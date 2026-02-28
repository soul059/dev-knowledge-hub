# 3.3 Hash Tables - Complete Study Guide

## Course Overview

Hash Tables are one of the most important and widely-used data structures in computer science. They provide **average-case O(1)** time complexity for insert, search, and delete operations, making them indispensable for efficient data retrieval. This guide covers everything from fundamental hashing concepts to advanced techniques and common interview patterns.

```
╔══════════════════════════════════════════════════════════════════╗
║                     HASH TABLE CONCEPT                         ║
║                                                                ║
║   Key ──► Hash Function ──► Index ──► Value                   ║
║                                                                ║
║   "apple"  ──► h("apple")  ──► 3  ──► { price: 1.50 }        ║
║   "banana" ──► h("banana") ──► 7  ──► { price: 0.75 }        ║
║   "cherry" ──► h("cherry") ──► 1  ──► { price: 2.00 }        ║
║                                                                ║
║   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐                  ║
║   │   │ C │   │ A │   │   │   │ B │   │   │  ◄── Array       ║
║   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘                  ║
║     0   1   2   3   4   5   6   7   8   9                     ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Prerequisites

- Arrays and basic data structures
- Linked Lists (for chaining)
- Basic modular arithmetic
- Understanding of time complexity (Big-O notation)

---

## Complete Table of Contents

### Unit 1: Hashing Fundamentals
> *Understanding the core concept of hashing and why it matters*

| # | Topic | File |
|---|-------|------|
| 1.1 | What is Hashing? | [01-what-is-hashing.md](01-Hashing-Fundamentals/01-what-is-hashing.md) |
| 1.2 | Hash Function Concept | [02-hash-function-concept.md](01-Hashing-Fundamentals/02-hash-function-concept.md) |
| 1.3 | Hash Table Structure | [03-hash-table-structure.md](01-Hashing-Fundamentals/03-hash-table-structure.md) |
| 1.4 | Direct Addressing | [04-direct-addressing.md](01-Hashing-Fundamentals/04-direct-addressing.md) |
| 1.5 | Why Hashing? | [05-why-hashing.md](01-Hashing-Fundamentals/05-why-hashing.md) |
| 1.6 | Hash Table Applications | [06-hash-table-applications.md](01-Hashing-Fundamentals/06-hash-table-applications.md) |

---

### Unit 2: Hash Functions
> *Designing effective hash functions for different data types*

| # | Topic | File |
|---|-------|------|
| 2.1 | Properties of Good Hash Function | [01-properties-of-good-hash-function.md](02-Hash-Functions/01-properties-of-good-hash-function.md) |
| 2.2 | Division Method | [02-division-method.md](02-Hash-Functions/02-division-method.md) |
| 2.3 | Multiplication Method | [03-multiplication-method.md](02-Hash-Functions/03-multiplication-method.md) |
| 2.4 | Universal Hashing | [04-universal-hashing.md](02-Hash-Functions/04-universal-hashing.md) |
| 2.5 | Cryptographic Hash Functions | [05-cryptographic-hash-functions.md](02-Hash-Functions/05-cryptographic-hash-functions.md) |
| 2.6 | String Hashing | [06-string-hashing.md](02-Hash-Functions/06-string-hashing.md) |

---

### Unit 3: Collision Handling - Chaining
> *Resolving collisions using linked structures*

| # | Topic | File |
|---|-------|------|
| 3.1 | Separate Chaining | [01-separate-chaining.md](03-Collision-Handling-Chaining/01-separate-chaining.md) |
| 3.2 | Linked List Implementation | [02-linked-list-implementation.md](03-Collision-Handling-Chaining/02-linked-list-implementation.md) |
| 3.3 | Load Factor | [03-load-factor.md](03-Collision-Handling-Chaining/03-load-factor.md) |
| 3.4 | Average Case Analysis | [04-average-case-analysis.md](03-Collision-Handling-Chaining/04-average-case-analysis.md) |
| 3.5 | Worst Case Scenario | [05-worst-case-scenario.md](03-Collision-Handling-Chaining/05-worst-case-scenario.md) |
| 3.6 | Implementation Details | [06-implementation-details.md](03-Collision-Handling-Chaining/06-implementation-details.md) |

---

### Unit 4: Collision Handling - Open Addressing
> *Resolving collisions within the table itself*

| # | Topic | File |
|---|-------|------|
| 4.1 | Open Addressing Concept | [01-open-addressing-concept.md](04-Collision-Handling-Open-Addressing/01-open-addressing-concept.md) |
| 4.2 | Linear Probing | [02-linear-probing.md](04-Collision-Handling-Open-Addressing/02-linear-probing.md) |
| 4.3 | Quadratic Probing | [03-quadratic-probing.md](04-Collision-Handling-Open-Addressing/03-quadratic-probing.md) |
| 4.4 | Double Hashing | [04-double-hashing.md](04-Collision-Handling-Open-Addressing/04-double-hashing.md) |
| 4.5 | Clustering Problems | [05-clustering-problems.md](04-Collision-Handling-Open-Addressing/05-clustering-problems.md) |
| 4.6 | Comparison of Methods | [06-comparison-of-methods.md](04-Collision-Handling-Open-Addressing/06-comparison-of-methods.md) |

---

### Unit 5: Hash Table Operations
> *Core operations and their complexity analysis*

| # | Topic | File |
|---|-------|------|
| 5.1 | Insert Operation | [01-insert-operation.md](05-Hash-Table-Operations/01-insert-operation.md) |
| 5.2 | Search Operation | [02-search-operation.md](05-Hash-Table-Operations/02-search-operation.md) |
| 5.3 | Delete Operation | [03-delete-operation.md](05-Hash-Table-Operations/03-delete-operation.md) |
| 5.4 | Resize and Rehash | [04-resize-and-rehash.md](05-Hash-Table-Operations/04-resize-and-rehash.md) |
| 5.5 | Time Complexity | [05-time-complexity-analysis.md](05-Hash-Table-Operations/05-time-complexity-analysis.md) |
| 5.6 | Space Considerations | [06-space-considerations.md](05-Hash-Table-Operations/06-space-considerations.md) |

---

### Unit 6: Hash Map Patterns
> *Common coding patterns using hash maps*

| # | Topic | File |
|---|-------|------|
| 6.1 | Two Sum Pattern | [01-two-sum-pattern.md](06-Hash-Map-Patterns/01-two-sum-pattern.md) |
| 6.2 | Frequency Counting | [02-frequency-counting.md](06-Hash-Map-Patterns/02-frequency-counting.md) |
| 6.3 | Grouping / Bucketing | [03-grouping-bucketing.md](06-Hash-Map-Patterns/03-grouping-bucketing.md) |
| 6.4 | Prefix Sums with Hash | [04-prefix-sums-with-hash.md](06-Hash-Map-Patterns/04-prefix-sums-with-hash.md) |
| 6.5 | Subarray Sum Equals K | [05-subarray-sum-equals-k.md](06-Hash-Map-Patterns/05-subarray-sum-equals-k.md) |
| 6.6 | Longest Substring Patterns | [06-longest-substring-patterns.md](06-Hash-Map-Patterns/06-longest-substring-patterns.md) |

---

### Unit 7: Hash Set Applications
> *Leveraging hash sets for efficient lookups*

| # | Topic | File |
|---|-------|------|
| 7.1 | Duplicate Detection | [01-duplicate-detection.md](07-Hash-Set-Applications/01-duplicate-detection.md) |
| 7.2 | Intersection Problems | [02-intersection-problems.md](07-Hash-Set-Applications/02-intersection-problems.md) |
| 7.3 | Union Problems | [03-union-problems.md](07-Hash-Set-Applications/03-union-problems.md) |
| 7.4 | First Unique Element | [04-first-unique-element.md](07-Hash-Set-Applications/04-first-unique-element.md) |
| 7.5 | Contains Duplicate Variations | [05-contains-duplicate-variations.md](07-Hash-Set-Applications/05-contains-duplicate-variations.md) |
| 7.6 | Cycle Detection | [06-cycle-detection.md](07-Hash-Set-Applications/06-cycle-detection.md) |

---

### Unit 8: String Hashing
> *Hashing techniques for string problems*

| # | Topic | File |
|---|-------|------|
| 8.1 | Rabin-Karp Algorithm | [01-rabin-karp-algorithm.md](08-String-Hashing/01-rabin-karp-algorithm.md) |
| 8.2 | Rolling Hash | [02-rolling-hash.md](08-String-Hashing/02-rolling-hash.md) |
| 8.3 | Polynomial Hashing | [03-polynomial-hashing.md](08-String-Hashing/03-polynomial-hashing.md) |
| 8.4 | Comparing Substrings | [04-comparing-substrings.md](08-String-Hashing/04-comparing-substrings.md) |
| 8.5 | String Matching | [05-string-matching.md](08-String-Hashing/05-string-matching.md) |
| 8.6 | Longest Duplicate Substring | [06-longest-duplicate-substring.md](08-String-Hashing/06-longest-duplicate-substring.md) |

---

### Unit 9: Advanced Hashing
> *Specialized hashing techniques and data structures*

| # | Topic | File |
|---|-------|------|
| 9.1 | Consistent Hashing | [01-consistent-hashing.md](09-Advanced-Hashing/01-consistent-hashing.md) |
| 9.2 | Bloom Filters | [02-bloom-filters.md](09-Advanced-Hashing/02-bloom-filters.md) |
| 9.3 | Count-Min Sketch | [03-count-min-sketch.md](09-Advanced-Hashing/03-count-min-sketch.md) |
| 9.4 | Cuckoo Hashing | [04-cuckoo-hashing.md](09-Advanced-Hashing/04-cuckoo-hashing.md) |
| 9.5 | Perfect Hashing | [05-perfect-hashing.md](09-Advanced-Hashing/05-perfect-hashing.md) |
| 9.6 | Locality-Sensitive Hashing | [06-locality-sensitive-hashing.md](09-Advanced-Hashing/06-locality-sensitive-hashing.md) |

---

### Unit 10: Custom Hash Functions
> *Building and optimizing hash functions for custom types*

| # | Topic | File |
|---|-------|------|
| 10.1 | Hashing Custom Objects | [01-hashing-custom-objects.md](10-Custom-Hash-Functions/01-hashing-custom-objects.md) |
| 10.2 | Hash Code and Equals | [02-hashcode-and-equals-contract.md](10-Custom-Hash-Functions/02-hashcode-and-equals-contract.md) |
| 10.3 | Immutability Importance | [03-immutability-importance.md](10-Custom-Hash-Functions/03-immutability-importance.md) |
| 10.4 | Collision Minimization | [04-collision-minimization.md](10-Custom-Hash-Functions/04-collision-minimization.md) |
| 10.5 | Performance Tuning | [05-performance-tuning.md](10-Custom-Hash-Functions/05-performance-tuning.md) |
| 10.6 | Common Mistakes | [06-common-mistakes.md](10-Custom-Hash-Functions/06-common-mistakes.md) |

---

## Complexity At-a-Glance

| Operation | Average Case | Worst Case | Space |
|-----------|:------------:|:----------:|:-----:|
| Insert    | O(1)         | O(n)       | O(n)  |
| Search    | O(1)         | O(n)       | O(n)  |
| Delete    | O(1)         | O(n)       | O(n)  |

> **Worst case** occurs when all keys hash to the same index (degenerate case).

---

## How to Use This Guide

1. **Sequential Study**: Follow units in order for building knowledge progressively
2. **Topic Reference**: Jump to specific topics using the table of contents
3. **Interview Prep**: Focus on Units 6, 7, and 8 for coding patterns
4. **Deep Dive**: Units 9 and 10 for advanced concepts
5. **Quick Revision**: Use summary tables and revision questions at the end of each chapter

---

## Legend for Diagrams

```
╔═══╗   Double-line box  = Major structure / container
║   ║
╚═══╝

┌───┐   Single-line box  = Individual element / node
│   │
└───┘

──►     Arrow             = Direction of flow / pointer
──▶     Bold arrow        = Primary data flow
- - -   Dashed line       = Optional / conditional path
███     Filled block       = Occupied slot
░░░     Shaded block       = Empty / available slot
[X]     Marked slot        = Deleted (tombstone)
```

---

> **Start Learning**: [Unit 1 - What is Hashing? →](01-Hashing-Fundamentals/01-what-is-hashing.md)
