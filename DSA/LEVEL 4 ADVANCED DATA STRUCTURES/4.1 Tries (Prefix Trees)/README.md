# 🌳 Tries (Prefix Trees) — Complete Study Guide

## Course Overview

A **Trie** (pronounced "try") is a tree-like data structure used for efficient storage and retrieval of strings. Also known as a **Prefix Tree**, it excels at prefix-based operations, making it indispensable for autocomplete systems, spell checkers, IP routing, and competitive programming.

This guide covers everything from fundamental concepts to advanced optimizations, with ASCII diagrams, pseudocode traces, complexity analysis, and problem-solving strategies.

```
                        ┌─────────┐
                        │  root   │
                        └────┬────┘
               ┌─────────┬──┴───┬─────────┐
               ▼         ▼      ▼         ▼
            ┌──┴──┐  ┌──┴──┐ ┌─┴──┐  ┌──┴──┐
            │  a  │  │  b  │ │ c  │  │  d  │
            └──┬──┘  └──┬──┘ └─┬──┘  └──┬──┘
            ┌──┴──┐  ┌──┴──┐ ┌─┴──┐  ┌──┴──┐
            │  p  │  │  a  │ │ a  │  │  o  │
            └──┬──┘  └──┬──┘ └─┬──┘  └──┬──┘
            ┌──┴──┐  ┌──┴──┐ ┌─┴──┐  ┌──┴──┐
            │  p  │  │  t  │ │ t  │  │  g  │
            └──┬──┘  └─────┘ └────┘  └─────┘
            ┌──┴──┐
            │  l  │
            └──┬──┘
            ┌──┴──┐
            │  e* │   (* = end of word)
            └─────┘
```

---

## 📋 Complete Table of Contents

### Unit 1: Trie Fundamentals
| # | Topic | Link |
|---|-------|------|
| 1 | What is a Trie? | [01-what-is-a-trie.md](01-Trie-Fundamentals/01-what-is-a-trie.md) |
| 2 | Trie vs Hash Table | [02-trie-vs-hash-table.md](01-Trie-Fundamentals/02-trie-vs-hash-table.md) |
| 3 | Trie Node Structure | [03-trie-node-structure.md](01-Trie-Fundamentals/03-trie-node-structure.md) |
| 4 | Prefix Matching Concept | [04-prefix-matching-concept.md](01-Trie-Fundamentals/04-prefix-matching-concept.md) |
| 5 | Trie Applications | [05-trie-applications.md](01-Trie-Fundamentals/05-trie-applications.md) |
| 6 | When to Use Tries | [06-when-to-use-tries.md](01-Trie-Fundamentals/06-when-to-use-tries.md) |

### Unit 2: Basic Trie Operations
| # | Topic | Link |
|---|-------|------|
| 1 | Insertion | [01-insertion.md](02-Basic-Trie-Operations/01-insertion.md) |
| 2 | Search (Exact Match) | [02-search-exact-match.md](02-Basic-Trie-Operations/02-search-exact-match.md) |
| 3 | Prefix Search | [03-prefix-search.md](02-Basic-Trie-Operations/03-prefix-search.md) |
| 4 | Deletion | [04-deletion.md](02-Basic-Trie-Operations/04-deletion.md) |
| 5 | Time Complexity | [05-time-complexity.md](02-Basic-Trie-Operations/05-time-complexity.md) |
| 6 | Space Complexity | [06-space-complexity.md](02-Basic-Trie-Operations/06-space-complexity.md) |

### Unit 3: Trie Implementation
| # | Topic | Link |
|---|-------|------|
| 1 | Array-Based Children | [01-array-based-children.md](03-Trie-Implementation/01-array-based-children.md) |
| 2 | HashMap-Based Children | [02-hashmap-based-children.md](03-Trie-Implementation/02-hashmap-based-children.md) |
| 3 | Handling End of Word | [03-handling-end-of-word.md](03-Trie-Implementation/03-handling-end-of-word.md) |
| 4 | Case Sensitivity | [04-case-sensitivity.md](03-Trie-Implementation/04-case-sensitivity.md) |
| 5 | Boolean vs Count | [05-boolean-vs-count.md](03-Trie-Implementation/05-boolean-vs-count.md) |
| 6 | Memory Considerations | [06-memory-considerations.md](03-Trie-Implementation/06-memory-considerations.md) |

### Unit 4: Prefix-Based Problems
| # | Topic | Link |
|---|-------|------|
| 1 | Autocomplete System | [01-autocomplete-system.md](04-Prefix-Based-Problems/01-autocomplete-system.md) |
| 2 | Search Suggestions | [02-search-suggestions.md](04-Prefix-Based-Problems/02-search-suggestions.md) |
| 3 | Prefix Count | [03-prefix-count.md](04-Prefix-Based-Problems/03-prefix-count.md) |
| 4 | Longest Common Prefix | [04-longest-common-prefix.md](04-Prefix-Based-Problems/04-longest-common-prefix.md) |
| 5 | Words with Prefix | [05-words-with-prefix.md](04-Prefix-Based-Problems/05-words-with-prefix.md) |
| 6 | Prefix Matching | [06-prefix-matching.md](04-Prefix-Based-Problems/06-prefix-matching.md) |

### Unit 5: Word Search Problems
| # | Topic | Link |
|---|-------|------|
| 1 | Word Search in Grid | [01-word-search-in-grid.md](05-Word-Search-Problems/01-word-search-in-grid.md) |
| 2 | Word Search II | [02-word-search-ii.md](05-Word-Search-Problems/02-word-search-ii.md) |
| 3 | Add and Search Word | [03-add-and-search-word.md](05-Word-Search-Problems/03-add-and-search-word.md) |
| 4 | Wildcard Matching | [04-wildcard-matching.md](05-Word-Search-Problems/04-wildcard-matching.md) |
| 5 | Pattern Matching | [05-pattern-matching.md](05-Word-Search-Problems/05-pattern-matching.md) |
| 6 | Replace Words | [06-replace-words.md](05-Word-Search-Problems/06-replace-words.md) |

### Unit 6: Trie with Bitwise
| # | Topic | Link |
|---|-------|------|
| 1 | XOR Problems with Trie | [01-xor-problems-with-trie.md](06-Trie-With-Bitwise/01-xor-problems-with-trie.md) |
| 2 | Maximum XOR of Two Numbers | [02-maximum-xor-of-two-numbers.md](06-Trie-With-Bitwise/02-maximum-xor-of-two-numbers.md) |
| 3 | Maximum XOR with Element | [03-maximum-xor-with-element.md](06-Trie-With-Bitwise/03-maximum-xor-with-element.md) |
| 4 | Bit Manipulation in Trie | [04-bit-manipulation-in-trie.md](06-Trie-With-Bitwise/04-bit-manipulation-in-trie.md) |
| 5 | Binary Trie Concept | [05-binary-trie-concept.md](06-Trie-With-Bitwise/05-binary-trie-concept.md) |

### Unit 7: Advanced Trie Problems
| # | Topic | Link |
|---|-------|------|
| 1 | Palindrome Pairs | [01-palindrome-pairs.md](07-Advanced-Trie-Problems/01-palindrome-pairs.md) |
| 2 | Word Squares | [02-word-squares.md](07-Advanced-Trie-Problems/02-word-squares.md) |
| 3 | Concatenated Words | [03-concatenated-words.md](07-Advanced-Trie-Problems/03-concatenated-words.md) |
| 4 | Stream of Characters | [04-stream-of-characters.md](07-Advanced-Trie-Problems/04-stream-of-characters.md) |
| 5 | Implement Magic Dictionary | [05-implement-magic-dictionary.md](07-Advanced-Trie-Problems/05-implement-magic-dictionary.md) |
| 6 | Map Sum Pairs | [06-map-sum-pairs.md](07-Advanced-Trie-Problems/06-map-sum-pairs.md) |

### Unit 8: Trie Optimizations
| # | Topic | Link |
|---|-------|------|
| 1 | Compressed Trie (Radix Tree) | [01-compressed-trie-radix-tree.md](08-Trie-Optimizations/01-compressed-trie-radix-tree.md) |
| 2 | Ternary Search Tree | [02-ternary-search-tree.md](08-Trie-Optimizations/02-ternary-search-tree.md) |
| 3 | PATRICIA Trie | [03-patricia-trie.md](08-Trie-Optimizations/03-patricia-trie.md) |
| 4 | Memory Optimization | [04-memory-optimization.md](08-Trie-Optimizations/04-memory-optimization.md) |
| 5 | Concurrent Tries | [05-concurrent-tries.md](08-Trie-Optimizations/05-concurrent-tries.md) |
| 6 | Persistent Tries | [06-persistent-tries.md](08-Trie-Optimizations/06-persistent-tries.md) |

---

## 🗺️ Learning Path

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Unit 1         │     │   Unit 2         │     │   Unit 3         │
│   Fundamentals   │────▶│   Operations     │────▶│   Implementation │
│   (6 topics)     │     │   (6 topics)     │     │   (6 topics)     │
└──────────────────┘     └──────────────────┘     └────────┬─────────┘
                                                           │
                    ┌──────────────────────────────────────┘
                    ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Unit 4         │     │   Unit 5         │     │   Unit 6         │
│   Prefix-Based   │────▶│   Word Search    │────▶│   Bitwise Trie   │
│   (6 topics)     │     │   (6 topics)     │     │   (5 topics)     │
└──────────────────┘     └──────────────────┘     └────────┬─────────┘
                                                           │
                    ┌──────────────────────────────────────┘
                    ▼
┌──────────────────┐     ┌──────────────────┐
│   Unit 7         │     │   Unit 8         │
│   Advanced       │────▶│   Optimizations  │
│   (6 topics)     │     │   (6 topics)     │
└──────────────────┘     └──────────────────┘
```

---

## 🎯 Key Complexity Summary

| Operation        | Time Complexity | Space Complexity |
|------------------|:--------------:|:----------------:|
| Insert word      | O(L)           | O(L × ALPHABET)  |
| Search word      | O(L)           | O(1)             |
| Prefix search    | O(L)           | O(1)             |
| Delete word      | O(L)           | O(1)             |
| Autocomplete     | O(L + K)       | O(N × L)         |
| XOR maximum      | O(N × B)       | O(N × B)         |

> L = length of word, K = number of matches, N = number of words, B = number of bits

---

*Total: 8 Units, 47 Topic Files — Happy Learning!* 🚀
