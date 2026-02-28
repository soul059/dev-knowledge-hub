# Time Complexity Analysis

## Overview
Understanding the time complexity of trie operations is essential for choosing the right data structure. This chapter provides a comprehensive analysis of every operation, comparisons with alternative data structures, and explains why tries achieve O(L) independent of the number of stored words.

---

## Per-Operation Time Complexity

| Operation | Time | Variables |
|-----------|:----:|-----------|
| `insert(word)` | **O(L)** | L = length of word |
| `search(word)` | **O(L)** | L = length of word |
| `startsWith(prefix)` | **O(P)** | P = length of prefix |
| `delete(word)` | **O(L)** | L = length of word |
| `findAllWithPrefix(prefix)` | **O(P + K)** | K = total chars in results |
| `countWordsWithPrefix(prefix)` | **O(P)** | With prefixCount field |
| `getAllWords()` | **O(Total)** | Total chars across all words |

---

## Why O(L) — Key Insight

```
  ┌──────────────────────────────────────────────────────────┐
  │  BST (Balanced):                                         │
  │                                                          │
  │  Each node comparison: O(L) to compare two strings       │
  │  Tree height: O(log N) nodes to visit                    │
  │  Total: O(L × log N)                                     │
  │                                                          │
  │  Example: 1M words, avg length 10                        │
  │  → O(10 × 20) = O(200) per search                       │
  ├──────────────────────────────────────────────────────────┤
  │  HASH TABLE:                                             │
  │                                                          │
  │  Hash computation: O(L) to hash the string               │
  │  + O(L) to compare with stored key                       │
  │  Total: O(L) average, O(N × L) worst (collisions)       │
  │                                                          │
  │  Example: O(10) average                                  │
  ├──────────────────────────────────────────────────────────┤
  │  TRIE:                                                   │
  │                                                          │
  │  Each level: O(1) to check one character                 │
  │  Total levels: L (word length)                           │
  │  Total: O(L)  ← ALWAYS, no worst case degradation       │
  │                                                          │
  │  Example: O(10) always                                   │
  │  Independent of N (number of stored words)!              │
  └──────────────────────────────────────────────────────────┘
```

### Why O(1) Per Level?

```
  At each trie node, we do:
  
  Array-based:  children[ch - 'a']  → direct index → O(1)
  HashMap-based: children.get(ch)   → hash lookup  → O(1) amortized
  
  No string comparison, no scanning — just one character lookup!
```

---

## Batch Operation Complexity

| Operation | Time | Explanation |
|-----------|:----:|-------------|
| Insert N words (avg length L) | O(N × L) | Each insert is O(L) |
| Search N words | O(N × L) | Each search is O(L) |
| Build trie from dictionary | O(N × L) | Insert all words |
| Delete all words | O(N × L) | Each delete is O(L) |

---

## Comparison Across Data Structures

### Single-Word Operations

| Operation | Trie | Hash Table | BST | Sorted Array |
|-----------|:----:|:----------:|:---:|:------------:|
| Insert | O(L) | O(L) avg | O(L log N) | O(N × L) |
| Search | O(L) | O(L) avg | O(L log N) | O(L log N) |
| Delete | O(L) | O(L) avg | O(L log N) | O(N × L) |
| Prefix search | **O(P)** | O(N × L) | O(L log N) | O(L log N) |
| Find all prefix | **O(P+K)** | O(N × L) | O(L log N + K) | O(L log N + K) |

### Visual Comparison for N = 1,000,000 words, L = 10

```
  Operations to search for one word:
  
  Trie:         O(10)           = 10 steps         ██
  Hash Table:   O(10) avg       = 10 steps         ██
  BST:          O(10 × 20)      = 200 steps        ████████████████████
  Sorted Array: O(10 × 20)      = 200 steps        ████████████████████
  
  Operations to find all words with prefix of length 3:
  
  Trie:         O(3 + K)        = 3 + matches      ████
  Hash Table:   O(10,000,000)   = scan all × L     █████████████████████████████
  BST:          O(10 × 20 + K)  = 200 + matches    ████████████████████
  Sorted Array: O(10 × 20 + K)  = 200 + matches    ████████████████████
```

---

## Amortized Analysis

### Inserting N random words

```
  Best case:  All words share a long prefix
              → Many nodes reused, fewer creations
              → Still O(N × L) total but fewer actual node creations
  
  Worst case: No shared prefixes (e.g., all start with different chars)
              → N × L nodes created
              → O(N × L) total
  
  Average case for English words:
              → Significant prefix sharing
              → About 0.4 × N × L nodes typically
```

---

## The "Independent of N" Property

This is the trie's most important property:

```
  ┌────────────────────────────────────────────────────────────┐
  │                                                            │
  │   search("hello") takes O(5) whether the trie contains:   │
  │                                                            │
  │   • 10 words                                               │
  │   • 10,000 words                                           │
  │   • 10,000,000 words                                       │
  │                                                            │
  │   The number of stored words does NOT affect search time!  │
  │                                                            │
  │   Only the LENGTH of the searched word matters.            │
  │                                                            │
  └────────────────────────────────────────────────────────────┘
```

Compare with BST where searching "hello" takes O(5 × log N):
- 10 words: O(5 × 3) = 15 steps
- 10,000 words: O(5 × 13) = 65 steps
- 10,000,000 words: O(5 × 23) = 115 steps

---

## Edge Cases and Their Complexity

| Edge Case | Time |
|-----------|:----:|
| Empty string insert/search | O(1) — loop doesn't execute |
| Single character word | O(1) — one level |
| Very long word (L = 1000) | O(1000) — still linear in L |
| Word not in trie (early exit) | O(K) where K = mismatch position |

---

## Summary Table

| Key Insight | Detail |
|-------------|--------|
| Core complexity | O(L) per operation |
| Independent of | N (number of stored words) |
| Depends on | L (length of the word being operated on) |
| Per-level cost | O(1) — array index or hash lookup |
| Prefix advantage | O(P + K) vs O(N × L) for alternatives |
| Batch operations | O(N × L) to process N words |
| vs BST | Avoids O(L × log N) string comparisons |
| vs Hash Table | Same O(L) avg, but trie has no worst case |

---

## Quick Revision Questions

1. **Why is trie search O(L) while BST search is O(L × log N)?**
   > In a trie, each level processes one character in O(1). In a BST, each node requires O(L) to compare full strings, and there are O(log N) levels.

2. **Does the number of words in the trie affect search time?**
   > No! Search time depends only on L (the query word length). Whether the trie has 100 or 1 million words, searching "hello" always takes 5 steps.

3. **What is the time to find ALL words with a given prefix?**
   > O(P + K) — P steps to navigate to the prefix, then K total characters collected in matching words via DFS.

4. **What's the worst-case time for hash table search on strings?**
   > O(N × L) — if all keys hash to the same bucket, you must compare the query against all N stored strings, each comparison taking O(L).

5. **How does early exit affect trie search time?**
   > If the search fails at position K < L (character not found), it returns in O(K) time — faster than the full O(L).

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Deletion](04-deletion.md) | [Next: Space Complexity ▶](06-space-complexity.md) |
