# Trie vs Hash Table

## Overview
Both tries and hash tables support fast string operations, but they differ fundamentally in structure, capabilities, and trade-offs. This chapter provides a deep dive into when and why to choose one over the other.

---

## Structural Difference

```
  ╔══════════════════════════════════════════════════════════════════╗
  ║                    HASH TABLE                                   ║
  ║                                                                 ║
  ║   hash("apple") ──► [bucket 3] ──► "apple"                     ║
  ║   hash("app")   ──► [bucket 7] ──► "app"                       ║
  ║   hash("ape")   ──► [bucket 3] ──► "ape" (collision!)          ║
  ║                                                                 ║
  ║   • Flat array of buckets                                       ║
  ║   • No relationship between "apple" and "app"                   ║
  ║   • Hash function decides placement                             ║
  ╚══════════════════════════════════════════════════════════════════╝

  ╔══════════════════════════════════════════════════════════════════╗
  ║                       TRIE                                      ║
  ║                                                                 ║
  ║              (root)                                             ║
  ║               │                                                 ║
  ║               a                                                 ║
  ║               │                                                 ║
  ║               p ─────────┐                                      ║
  ║              / \         │                                      ║
  ║             p*  e*       │  Prefix "ap" shared!                 ║
  ║            /             │                                      ║
  ║           l              │                                      ║
  ║          /                                                      ║
  ║         e*                                                      ║
  ║                                                                 ║
  ║   • Tree structure                                              ║
  ║   • "apple" and "app" naturally share prefix                    ║
  ║   • Characters determine placement                              ║
  ╚══════════════════════════════════════════════════════════════════╝
```

---

## Detailed Comparison

| Feature                  | Trie                   | Hash Table            |
|--------------------------|:----------------------:|:---------------------:|
| **Insert**               | O(L)                   | O(L) average          |
| **Search**               | O(L)                   | O(L) average          |
| **Delete**               | O(L)                   | O(L) average          |
| **Prefix search**        | O(P) ✅                | O(N × L) ❌           |
| **Find all with prefix** | O(P + K) ✅            | O(N × L) ❌           |
| **Sorted order**         | Yes (DFS) ✅           | No ❌                 |
| **Worst-case search**    | O(L) always            | O(N) collisions       |
| **Space per key**        | High (pointers)        | Lower (flat)          |
| **Collision handling**   | Not needed ✅          | Required ❌           |
| **Lexicographic order**  | Natural ✅             | Not possible ❌       |

> **L** = key length, **P** = prefix length, **K** = matching words, **N** = total keys

---

## Prefix Search: The Key Difference

### Problem: Find all words starting with "app"

**Hash Table Approach:**
```
words = {"apple", "app", "application", "banana", "bat", "append"}

FUNCTION findWithPrefix_HashTable(prefix):
    results = []
    FOR each word in hashTable:          // must scan ALL keys!
        IF word.startsWith(prefix):
            results.add(word)
    RETURN results
    
    // Time: O(N × L) — checks every single word
```

**Trie Approach:**
```
FUNCTION findWithPrefix_Trie(prefix):
    node = root
    FOR each char in prefix:             // navigate to prefix
        node = node.children[char]
    
    // DFS from here — only visits matching words
    RETURN collectAllWords(node, prefix)
    
    // Time: O(P + K) — only visits prefix path + matching subtree
```

```
  Hash Table: scans EVERYTHING
  ┌────────────────────────────────────────────────────┐
  │  "apple"      ← check startsWith("app") ✓         │
  │  "app"        ← check startsWith("app") ✓         │
  │  "application"← check startsWith("app") ✓         │
  │  "banana"     ← check startsWith("app") ✗ WASTED  │
  │  "bat"        ← check startsWith("app") ✗ WASTED  │
  │  "append"     ← check startsWith("app") ✓         │
  └────────────────────────────────────────────────────┘
  Scanned: 6 words (all of them)

  Trie: navigates directly
  ┌────────────────────────────────────────────────────┐
  │  root → a → p → p  (3 steps to reach prefix)      │
  │                  │                                  │
  │                  ├── *("app")                       │
  │                  ├── l → e *("apple")               │
  │                  ├── l → i → c → ... *("application")│
  │                  └── e → n → d *("append")          │
  └────────────────────────────────────────────────────┘
  Visited: only 4 matching words (skipped "banana", "bat")
```

---

## Collision Handling

```
  HASH TABLE:
  ┌──────────────────────────────────────────────────┐
  │  hash("apple") = 42                              │
  │  hash("elppa") = 42   ← COLLISION!               │
  │                                                   │
  │  Bucket 42: "apple" → "elppa" (chain/probe)      │
  │  Worst case: all N keys hash to same bucket       │
  │  → O(N) search time!                              │
  └──────────────────────────────────────────────────┘

  TRIE:
  ┌──────────────────────────────────────────────────┐
  │  "apple": root → a → p → p → l → e              │
  │  "elppa": root → e → l → p → p → a              │
  │                                                   │
  │  DIFFERENT PATHS — no collision possible!          │
  │  Always O(L) — guaranteed, no worst case           │
  └──────────────────────────────────────────────────┘
```

---

## Sorted/Lexicographic Order

```
  Hash Table: NO natural ordering
  Keys: {"zebra", "apple", "mango"}
  Iteration order: unpredictable (depends on hash values)

  Trie: NATURAL lexicographic order via DFS
  
         (root)
        /  |   \
       a   m    z
       │   │    │
       p   a    e
       │   │    │
       p   n    b
       │   │    │
       l   g    r
       │   │    │
       e*  o*   a*

  DFS traversal: "apple" → "mango" → "zebra"  ← sorted!
```

---

## Memory Comparison

```
  Storing: ["apple", "app", "application"] (3 words)

  Hash Table:
  ┌─────────────────────────────────────────────┐
  │ Bucket array:     ~100 entries × 8B = 800B  │
  │ "apple" key:      5 chars + overhead ≈ 60B  │
  │ "app" key:        3 chars + overhead ≈ 60B  │
  │ "application":    11 chars + overhead ≈ 70B │
  │ ─────────────────────────────────────────    │
  │ Total: ~990 bytes                           │
  └─────────────────────────────────────────────┘

  Trie (Array[26]):
  ┌─────────────────────────────────────────────┐
  │ 12 nodes × ~210B each = ~2520 bytes         │
  │                                              │
  │ Nodes: root,a,p,p,l,e,i,c,a,t,i,o,n        │
  │ (13 nodes with prefix sharing)               │
  │ ─────────────────────────────────────────    │
  │ Total: ~2730 bytes  (2.7x more!)            │
  └─────────────────────────────────────────────┘
```

> Tries use more memory but provide prefix search capability that hash tables cannot match.

---

## When Trie Wins

| Scenario | Why Trie Wins |
|----------|--------------|
| Autocomplete | O(P + K) vs O(N × L) |
| Spell checking | Can explore similar words efficiently |
| Prefix counting | O(P) with prefix_count field |
| Lexicographic sorting | Natural DFS ordering |
| Longest common prefix | O(L) — follow shared path |
| No collision guarantee | Always O(L), never degrades |
| IP routing | Longest prefix match |

## When Hash Table Wins

| Scenario | Why Hash Table Wins |
|----------|-------------------|
| Exact-match only | Same O(L) but less memory |
| Memory constrained | 2-5x less memory typically |
| Non-string keys | Tries only work with sequences |
| Simple implementation | No tree management needed |
| Small datasets | Overhead of trie not justified |

---

## Decision Flowchart

```
  ┌─────────────────────────────────┐
  │ Do you need PREFIX operations?  │
  └──────────────┬──────────────────┘
           ┌─────┴─────┐
           ▼           ▼
          YES          NO
           │           │
     ┌─────┴──────┐   ▼
     │  USE TRIE  │   ┌──────────────────────────┐
     └────────────┘   │ Do you need sorted iter? │
                      └────────────┬─────────────┘
                            ┌──────┴──────┐
                            ▼             ▼
                           YES            NO
                            │             │
                   ┌────────┴───────┐   ┌─┴────────────┐
                   │ Trie or        │   │ Hash Table    │
                   │ Balanced BST   │   │ (best choice) │
                   └────────────────┘   └──────────────┘
```

---

## Summary Table

| Criterion | Trie | Hash Table |
|-----------|:----:|:----------:|
| Prefix search | O(P) ✅ | O(N×L) ❌ |
| Exact search | O(L) | O(L) avg |
| Worst case | O(L) guaranteed | O(N) possible |
| Memory | Higher | Lower |
| Ordering | Sorted naturally | Unordered |
| Collisions | Impossible | Must handle |
| Key types | Strings only | Any hashable |

---

## Quick Revision Questions

1. **Why can't hash tables do prefix search efficiently?**
   > Hash functions destroy prefix relationships. "apple" and "app" hash to completely different buckets, so finding all words with prefix "app" requires scanning every key.

2. **When would you choose a hash table over a trie for string storage?**
   > When you only need exact-match lookups, memory is constrained, or the dataset is small.

3. **What advantage does a trie have over a hash table regarding worst-case performance?**
   > Trie search is always O(L). Hash tables can degrade to O(N) if many keys hash to the same bucket (collisions).

4. **How does a trie naturally provide sorted output?**
   > DFS traversal of a trie visits children in alphabetical order, producing words in lexicographic order automatically.

5. **Is a trie always more memory-efficient than a hash table?**
   > No! Tries typically use 2-5x more memory due to pointer overhead. The trade-off is the prefix search capability.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: What is a Trie?](01-what-is-a-trie.md) | [Next: Trie Node Structure ▶](03-trie-node-structure.md) |
