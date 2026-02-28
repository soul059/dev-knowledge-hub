# When to Use Tries

## Overview
Knowing when to reach for a trie (and when NOT to) is a critical skill. This chapter gives you a clear decision framework to choose the right data structure for any string-related problem.

---

## Decision Framework

```
  ┌─────────────────────────────────────────┐
  │     Do you need PREFIX operations?       │
  └──────────────────┬──────────────────────┘
                     │
              ┌──────┴──────┐
              ▼             ▼
             YES            NO
              │             │
     ┌────────┴────────┐   │
     │   USE A TRIE    │   │
     └─────────────────┘   │
                           ▼
              ┌────────────────────────┐
              │ Need sorted iteration? │
              └────────────┬───────────┘
                     ┌─────┴──────┐
                     ▼            ▼
                    YES           NO
                     │            │
            ┌────────┴────────┐  │
            │  Consider Trie  │  │
            │  or Balanced BST│  │
            └─────────────────┘  │
                                 ▼
                    ┌────────────────────────┐
                    │ Only exact lookups?    │
                    └────────────┬───────────┘
                           ┌────┴─────┐
                           ▼          ▼
                          YES         NO
                           │          │
                  ┌────────┴────────┐ │
                  │  Use Hash Table │ ▼
                  └─────────────────┘ 
                               Consider other
                               data structures
```

---

## ✅ USE a Trie When...

### 1. Prefix Search is Required
```
  Problem: "Find all words starting with 'pre'"
  
  Hash Table: O(N × L) — scan everything      ❌
  Trie:       O(P + K) — direct navigation     ✅ Winner
```

### 2. Autocomplete / Suggestions
```
  Problem: "User types 'hel', suggest completions"
  
  Trie: navigate to 'h'→'e'→'l', collect subtree
  Instant results: ["help", "hello", "helmet", ...]
```

### 3. Many Strings Share Prefixes
```
  Words: ["prefix", "preview", "prevent", "preorder", "prepare"]
  
  All share "pre" — trie stores it ONCE
  5 words, but "pre" path = 3 nodes (not 5 × 3 = 15 chars)
```

### 4. Pattern Matching with Wildcards
```
  Problem: "Find words matching 'h.llo'" (. = any char)
  
  Trie: DFS with branching at wildcard positions
  At '.': try ALL children, continue matching
```

### 5. Lexicographic Order Needed
```
  Problem: "Return all words in sorted order"
  
  Trie: DFS naturally visits in alphabetical order
  No sorting needed — it's inherent in the structure
```

### 6. XOR Optimization (Binary Trie)
```
  Problem: "Find max XOR of two numbers from array"
  
  Binary trie: greedily choose opposite bits
  O(N × B) instead of O(N²)
```

### 7. Word Validation with Early Termination
```
  Problem: "Boggle board — find all valid words"
  
  Trie: check if current path is a valid PREFIX
  If no word starts with current path → stop exploring (PRUNE)
  Massive speedup vs checking every possible path
```

---

## ❌ DON'T Use a Trie When...

### 1. Only Exact Lookups Needed
```
  Problem: "Check if word exists in dictionary"
  (no prefix queries, no autocomplete, no sorting)
  
  Hash Set: O(L) lookup, much less memory         ✅ Winner
  Trie:     O(L) lookup, but 3-5x more memory     ❌ Overkill
```

### 2. Keys Are Not Strings/Sequences
```
  Problem: "Store integer keys with fast lookup"
  
  Trie: designed for sequential character data    ❌
  Hash Map / BST: work with any comparable type    ✅
  
  Exception: Binary tries work on integer BITS
```

### 3. Memory is Severely Constrained
```
  Hash Table: ~50 bytes per entry
  Trie:       ~200+ bytes per NODE (many nodes per word)
  
  For 100K words: HashTable ≈ 5MB vs Trie ≈ 50MB
  If memory matters → Hash Table                   ✅ Winner
```

### 4. Small Dataset (< 100 words)
```
  With < 100 words, even brute force is fast
  Trie overhead not justified for tiny datasets
  
  Simple array + linear scan is simpler and sufficient
```

### 5. Range Queries on Numbers
```
  Problem: "Find all numbers between 100 and 500"
  
  Segment Tree / BIT: designed for this             ✅
  Trie: not designed for numeric ranges              ❌
```

---

## Comparison Matrix

| Scenario | Best Choice | Why |
|----------|:-----------:|-----|
| Prefix search | **Trie** | O(P + K) vs O(N × L) |
| Autocomplete | **Trie** | Natural prefix collection |
| Exact lookup only | **Hash Table** | Less memory, same O(L) |
| Sorted order | **Trie** or BST | DFS gives sorted order |
| Wildcard matching | **Trie** | Branch at wildcards |
| Max XOR pair | **Binary Trie** | Greedy bit selection |
| Frequency count | **Hash Map** | Simpler, less memory |
| Non-string keys | **Hash Map/BST** | Trie is string-specific |
| Memory limited | **Hash Table** | 3-5x less memory |
| Spell checking | **Trie** | Explore similar words |
| IP routing | **Binary Trie** | Longest prefix match |
| Small dataset | **Array/List** | Simplest approach |

---

## Interview Pattern Recognition

When you see these phrases in a problem, think **TRIE**:

```
  ┌──────────────────────────────────────────────────────────┐
  │  TRIGGER PHRASES → USE TRIE                              │
  ├──────────────────────────────────────────────────────────┤
  │  "Find all words starting with..."                       │
  │  "Implement autocomplete..."                             │
  │  "Search for words with prefix..."                       │
  │  "Longest common prefix..."                              │
  │  "Word dictionary with add and search..."                │
  │  "Replace words with their shortest prefix..."           │
  │  "Maximum XOR of two numbers..."                         │
  │  "Word search in a grid (multiple words)..."             │
  │  "Design a search suggestion system..."                  │
  │  "Stream of characters - check suffix..."                │
  │  "Implement a magic dictionary..."                       │
  └──────────────────────────────────────────────────────────┘
```

---

## Common LeetCode Problems by Data Structure

```
  TRIE problems:
  ├── #208  Implement Trie
  ├── #211  Add and Search Word
  ├── #212  Word Search II
  ├── #648  Replace Words
  ├── #421  Maximum XOR of Two Numbers
  ├── #1268 Search Suggestions System
  ├── #336  Palindrome Pairs
  ├── #472  Concatenated Words
  └── #677  Map Sum Pairs

  NOT trie (despite involving strings):
  ├── #3    Longest Substring Without Repeating → Sliding Window
  ├── #5    Longest Palindromic Substring → DP
  ├── #49   Group Anagrams → Hash Map
  ├── #76   Minimum Window Substring → Sliding Window
  └── #242  Valid Anagram → Hash Map / Sort
```

---

## Quick Decision Table

| Question to Ask | If YES | If NO |
|----------------|:------:|:-----:|
| Need prefix queries? | Trie | Hash Table |
| Multiple words share prefixes? | Trie | Hash Table |
| Need sorted output? | Trie/BST | Hash Table |
| Wildcard/pattern matching? | Trie | Regex/DP |
| XOR optimization? | Binary Trie | Brute Force |
| Memory constrained? | Hash Table | Trie OK |
| Keys are not strings? | Hash/BST | — |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Primary use case | Prefix-based string operations |
| Signal phrases | "prefix", "autocomplete", "starts with" |
| Main advantage | O(P + K) prefix search |
| Main disadvantage | High memory usage |
| Alternative | Hash Table (for exact-match only) |
| Best for interviews | Word search, autocomplete, XOR problems |

---

## Quick Revision Questions

1. **You need to check if a word exists in a dictionary of 1M words. No prefix queries needed. Trie or Hash Table?**
   > Hash Table — same O(L) lookup but 3-5x less memory. Trie is overkill without prefix needs.

2. **A problem says "find all words starting with a given prefix." What data structure?**
   > Trie — this is the textbook use case. O(P + K) time, far better than scanning all words.

3. **When solving "Maximum XOR of two numbers in an array," why use a trie?**
   > Build a binary trie of all numbers. For each number, greedily choose opposite bits to maximize XOR. Reduces O(N²) to O(N × B).

4. **Name three interview trigger phrases that indicate a trie solution.**
   > "Find words starting with prefix", "implement autocomplete", "word search with multiple words in a grid."

5. **Why is a trie better than a hash table for spell checking?**
   > Tries can explore nearby paths (1-2 edit distance) via DFS. Hash tables can't navigate to "similar" keys — they only do exact lookups.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Trie Applications](05-trie-applications.md) | [Next: Unit 2 — Insertion ▶](../02-Basic-Trie-Operations/01-insertion.md) |
