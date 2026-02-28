# 🔧 Bit Manipulation — Complete Study Guide

> **Level 7 — Advanced Algorithms**
> Master the art of thinking in bits: the fastest, most elegant layer of computation.

---

## Course Overview

Bit manipulation is one of the most powerful tools in a programmer's arsenal. Every piece of data — integers, characters, images, network packets — is ultimately a sequence of **0s and 1s**. Understanding how to manipulate individual bits lets you write code that is:

- **Blazingly fast** — bitwise operations execute in a single CPU cycle
- **Memory efficient** — pack multiple flags into a single integer
- **Elegant** — solve complex problems with one-liner expressions

This guide takes you from the fundamentals of binary representation through advanced bitmask dynamic programming, with detailed explanations, ASCII diagrams, step-by-step traces, and dozens of practice problems.

---

## How to Use These Notes

```
 ┌─────────────────────────────────────────────────────┐
 │  1. Read concepts & study ASCII diagrams            │
 │  2. Trace through pseudocode by hand                │
 │  3. Solve the quick revision questions              │
 │  4. Implement in your preferred language             │
 │  5. Move to the next chapter                        │
 └─────────────────────────────────────────────────────┘
```

Each chapter includes:
- **Concept explanation** with intuition
- **ASCII diagrams** & truth tables
- **Pseudocode** with step-by-step traces
- **Complexity analysis** (Time & Space)
- **Real-world applications**
- **Summary table**
- **5–6 Quick revision questions**
- **Navigation links** to previous/next chapters

---

## Complete Table of Contents

### Unit 1: Binary Number System
| # | Topic | File |
|---|-------|------|
| 1.1 | Binary Representation | [01-binary-representation.md](01-Binary-Number-System/01-binary-representation.md) |
| 1.2 | Converting Decimal to Binary | [02-decimal-to-binary.md](01-Binary-Number-System/02-decimal-to-binary.md) |
| 1.3 | Converting Binary to Decimal | [03-binary-to-decimal.md](01-Binary-Number-System/03-binary-to-decimal.md) |
| 1.4 | Signed vs Unsigned Integers | [04-signed-vs-unsigned.md](01-Binary-Number-System/04-signed-vs-unsigned.md) |
| 1.5 | Two's Complement | [05-twos-complement.md](01-Binary-Number-System/05-twos-complement.md) |
| 1.6 | Bit Positions | [06-bit-positions.md](01-Binary-Number-System/06-bit-positions.md) |

### Unit 2: Bitwise Operators
| # | Topic | File |
|---|-------|------|
| 2.1 | AND Operation | [01-and-operation.md](02-Bitwise-Operators/01-and-operation.md) |
| 2.2 | OR Operation | [02-or-operation.md](02-Bitwise-Operators/02-or-operation.md) |
| 2.3 | XOR Operation | [03-xor-operation.md](02-Bitwise-Operators/03-xor-operation.md) |
| 2.4 | NOT Operation | [04-not-operation.md](02-Bitwise-Operators/04-not-operation.md) |
| 2.5 | Left Shift | [05-left-shift.md](02-Bitwise-Operators/05-left-shift.md) |
| 2.6 | Right Shift (Arithmetic & Logical) | [06-right-shift.md](02-Bitwise-Operators/06-right-shift.md) |

### Unit 3: Basic Bit Techniques
| # | Topic | File |
|---|-------|------|
| 3.1 | Check if Bit is Set | [01-check-if-bit-is-set.md](03-Basic-Bit-Techniques/01-check-if-bit-is-set.md) |
| 3.2 | Set a Bit | [02-set-a-bit.md](03-Basic-Bit-Techniques/02-set-a-bit.md) |
| 3.3 | Clear a Bit | [03-clear-a-bit.md](03-Basic-Bit-Techniques/03-clear-a-bit.md) |
| 3.4 | Toggle a Bit | [04-toggle-a-bit.md](03-Basic-Bit-Techniques/04-toggle-a-bit.md) |
| 3.5 | Check Power of 2 | [05-check-power-of-2.md](03-Basic-Bit-Techniques/05-check-power-of-2.md) |
| 3.6 | Count Set Bits (Brian Kernighan) | [06-count-set-bits.md](03-Basic-Bit-Techniques/06-count-set-bits.md) |

### Unit 4: Bit Manipulation Tricks
| # | Topic | File |
|---|-------|------|
| 4.1 | Swap Without Temp Variable | [01-swap-without-temp.md](04-Bit-Manipulation-Tricks/01-swap-without-temp.md) |
| 4.2 | Find Odd Occurring Element | [02-find-odd-occurring-element.md](04-Bit-Manipulation-Tricks/02-find-odd-occurring-element.md) |
| 4.3 | Check if Power of 4 | [03-check-power-of-4.md](04-Bit-Manipulation-Tricks/03-check-power-of-4.md) |
| 4.4 | Reverse Bits | [04-reverse-bits.md](04-Bit-Manipulation-Tricks/04-reverse-bits.md) |
| 4.5 | Next Power of 2 | [05-next-power-of-2.md](04-Bit-Manipulation-Tricks/05-next-power-of-2.md) |
| 4.6 | Turn Off Rightmost Set Bit | [06-turn-off-rightmost-set-bit.md](04-Bit-Manipulation-Tricks/06-turn-off-rightmost-set-bit.md) |

### Unit 5: XOR Applications
| # | Topic | File |
|---|-------|------|
| 5.1 | XOR Properties | [01-xor-properties.md](05-XOR-Applications/01-xor-properties.md) |
| 5.2 | Single Number (Appears Once) | [02-single-number.md](05-XOR-Applications/02-single-number.md) |
| 5.3 | Two Numbers Appearing Once | [03-two-numbers-appearing-once.md](05-XOR-Applications/03-two-numbers-appearing-once.md) |
| 5.4 | Find Missing Number | [04-find-missing-number.md](05-XOR-Applications/04-find-missing-number.md) |
| 5.5 | Find Duplicate Number | [05-find-duplicate-number.md](05-XOR-Applications/05-find-duplicate-number.md) |
| 5.6 | XOR of Range | [06-xor-of-range.md](05-XOR-Applications/06-xor-of-range.md) |

### Unit 6: Subsets with Bits
| # | Topic | File |
|---|-------|------|
| 6.1 | Generate All Subsets | [01-generate-all-subsets.md](06-Subsets-With-Bits/01-generate-all-subsets.md) |
| 6.2 | Count Subsets | [02-count-subsets.md](06-Subsets-With-Bits/02-count-subsets.md) |
| 6.3 | Subset Sum | [03-subset-sum.md](06-Subsets-With-Bits/03-subset-sum.md) |
| 6.4 | Bitmasking for Subsets | [04-bitmasking-for-subsets.md](06-Subsets-With-Bits/04-bitmasking-for-subsets.md) |
| 6.5 | DP with Bitmasks Preview | [05-dp-with-bitmasks-preview.md](06-Subsets-With-Bits/05-dp-with-bitmasks-preview.md) |
| 6.6 | Iterating Subsets of a Mask | [06-iterating-subsets-of-mask.md](06-Subsets-With-Bits/06-iterating-subsets-of-mask.md) |

### Unit 7: Advanced Bit Operations
| # | Topic | File |
|---|-------|------|
| 7.1 | Rightmost Set Bit Position | [01-rightmost-set-bit-position.md](07-Advanced-Bit-Operations/01-rightmost-set-bit-position.md) |
| 7.2 | Leftmost Set Bit Position | [02-leftmost-set-bit-position.md](07-Advanced-Bit-Operations/02-leftmost-set-bit-position.md) |
| 7.3 | Count Bits in Range | [03-count-bits-in-range.md](07-Advanced-Bit-Operations/03-count-bits-in-range.md) |
| 7.4 | Sum of XOR of All Subsets | [04-sum-of-xor-of-all-subsets.md](07-Advanced-Bit-Operations/04-sum-of-xor-of-all-subsets.md) |
| 7.5 | Maximum AND Pair | [05-maximum-and-pair.md](07-Advanced-Bit-Operations/05-maximum-and-pair.md) |
| 7.6 | Maximum XOR Pair | [06-maximum-xor-pair.md](07-Advanced-Bit-Operations/06-maximum-xor-pair.md) |

### Unit 8: Bit Manipulation in DP
| # | Topic | File |
|---|-------|------|
| 8.1 | Traveling Salesman Problem | [01-traveling-salesman.md](08-Bit-Manipulation-In-DP/01-traveling-salesman.md) |
| 8.2 | Assignment Problem | [02-assignment-problem.md](08-Bit-Manipulation-In-DP/02-assignment-problem.md) |
| 8.3 | Counting Paths with States | [03-counting-paths-with-states.md](08-Bit-Manipulation-In-DP/03-counting-paths-with-states.md) |
| 8.4 | Bitmask Optimization | [04-bitmask-optimization.md](08-Bit-Manipulation-In-DP/04-bitmask-optimization.md) |
| 8.5 | Memory Savings | [05-memory-savings.md](08-Bit-Manipulation-In-DP/05-memory-savings.md) |

### Unit 9: Practical Applications
| # | Topic | File |
|---|-------|------|
| 9.1 | IP Address Manipulation | [01-ip-address-manipulation.md](09-Practical-Applications/01-ip-address-manipulation.md) |
| 9.2 | Permission Systems | [02-permission-systems.md](09-Practical-Applications/02-permission-systems.md) |
| 9.3 | Compression Basics | [03-compression-basics.md](09-Practical-Applications/03-compression-basics.md) |
| 9.4 | Error Detection | [04-error-detection.md](09-Practical-Applications/04-error-detection.md) |
| 9.5 | Bloom Filter Bits | [05-bloom-filter-bits.md](09-Practical-Applications/05-bloom-filter-bits.md) |
| 9.6 | Graphics Basics | [06-graphics-basics.md](09-Practical-Applications/06-graphics-basics.md) |

---

## Prerequisite Knowledge

```
 ┌────────────────────────────────────────────┐
 │         PREREQUISITE MAP                   │
 ├────────────────────────────────────────────┤
 │                                            │
 │  Basic Math ──────► Number Systems         │
 │                         │                  │
 │  Any Programming ──►  Operators            │
 │  Language               │                  │
 │                    Bit Techniques           │
 │                         │                  │
 │  Arrays & ────────► XOR & Subsets           │
 │  Hashing                │                  │
 │                    Advanced Ops             │
 │  Dynamic ─────────►    │                   │
 │  Programming       Bitmask DP              │
 │                         │                  │
 │  Systems ─────────► Applications           │
 │  Knowledge                                 │
 └────────────────────────────────────────────┘
```

---

## Complexity Cheat Sheet

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| Single bitwise op (AND, OR, XOR, NOT) | O(1) | O(1) | Single CPU cycle |
| Shift operation | O(1) | O(1) | Hardware-level |
| Count set bits (naive) | O(log n) | O(1) | Check each bit |
| Count set bits (Kernighan) | O(k) | O(1) | k = number of set bits |
| Generate all subsets | O(2ⁿ) | O(2ⁿ) | Exponential |
| Bitmask DP (TSP) | O(2ⁿ · n²) | O(2ⁿ · n) | Exponential but optimal |
| Maximum XOR (Trie) | O(n · 32) | O(n · 32) | Linear after trie build |

---

## Quick Reference: Common Bit Formulas

```
 ╔══════════════════════════════════════════════════════════╗
 ║  ESSENTIAL BIT MANIPULATION FORMULAS                    ║
 ╠══════════════════════════════════════════════════════════╣
 ║                                                         ║
 ║  Check bit i       :  (n >> i) & 1                      ║
 ║  Set bit i         :  n | (1 << i)                      ║
 ║  Clear bit i       :  n & ~(1 << i)                     ║
 ║  Toggle bit i      :  n ^ (1 << i)                      ║
 ║  Power of 2 check  :  n & (n - 1) == 0                  ║
 ║  Lowest set bit    :  n & (-n)                          ║
 ║  Turn off lowest   :  n & (n - 1)                       ║
 ║  Count set bits    :  while(n) { count++; n &= n-1; }   ║
 ║  All subsets of n  :  for i in 0..2^n-1                  ║
 ║  Check odd/even    :  n & 1                              ║
 ║  Multiply by 2^k   :  n << k                             ║
 ║  Divide by 2^k     :  n >> k                             ║
 ║  Swap a,b          :  a^=b; b^=a; a^=b;                  ║
 ║                                                         ║
 ╚══════════════════════════════════════════════════════════╝
```

---

## Study Plan

| Week | Units | Focus |
|------|-------|-------|
| Week 1 | Unit 1–2 | Build strong foundation in binary and operators |
| Week 2 | Unit 3–4 | Master essential bit tricks |
| Week 3 | Unit 5–6 | XOR mastery and subset generation |
| Week 4 | Unit 7–8 | Advanced operations and bitmask DP |
| Week 5 | Unit 9 | Real-world applications and revision |

---

> **Start your journey →** [Unit 1: Binary Representation](01-Binary-Number-System/01-binary-representation.md)
