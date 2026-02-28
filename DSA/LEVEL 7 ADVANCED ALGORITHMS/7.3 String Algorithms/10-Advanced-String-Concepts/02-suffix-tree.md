# Chapter 10.2 — Suffix Tree

> **Unit 10: Advanced String Concepts** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **suffix tree** is a compressed trie of all suffixes of a string.
It enables O(m) pattern matching and solves numerous string problems
in linear time.

---

## 1. From Suffix Trie to Suffix Tree

```
  Suffix Trie for "aab$":
  Suffixes: "aab$", "ab$", "b$", "$"

  Suffix Trie (uncompressed):
       (root)
      / | \  \
     a  b  $*
     |  |
     a  $*
    / \
   b   $*
   |
   $*

  Problem: O(n²) nodes — too much space!

  Suffix Tree (compressed):
  Merge single-child chains into one edge with a substring label.

       (root)
      /   |    \
   "a"   "b$"*  "$"*
    |
   / \
  "ab$"* "$"*

  Real representation with edge labels:
       (root)
      /    |     \
   a[0,0] b[2,3]* $[3,3]*
     |
    / \
  ab$[1,3]* $[3,3]*

  ┌────────────────────────────────────────────────────┐
  │  Suffix Tree:                                      │
  │  - At most 2n-1 nodes (n leaves + n-1 internal)   │
  │  - O(n) space using edge labels as (start, end)    │
  │  - Each edge labeled with a substring              │
  └────────────────────────────────────────────────────┘
```

---

## 2. Suffix Tree Properties

```
  For string S of length n (with sentinel $):

  1. Exactly n leaves (one per suffix)
  2. Each internal node has ≥ 2 children
  3. Edge labels are non-empty substrings
  4. No two edges from same node start with same character
  5. Path from root to leaf i spells out suffix S[i..n]
  6. At most 2n - 1 nodes total

  Space trick: Store edge labels as (start, end) indices
  into the original string — O(1) per edge!

  ┌──────────────────────────────────────────────────┐
  │  Total space: O(n) despite representing O(n²)    │
  │  total characters across all suffixes.            │
  └──────────────────────────────────────────────────┘
```

---

## 3. Construction: Ukkonen's Algorithm

```
  Ukkonen's algorithm builds the suffix tree in O(n) time
  using three key tricks:

  Trick 1: Implicit extensions
  ─────────────────────────────
  Leaf edges automatically extend as new characters arrive.
  Use a global "end" pointer.

  Trick 2: Skip/count
  ────────────────────
  When traversing edges, skip by length rather than
  comparing character by character.

  Trick 3: Early termination (Rule 3)
  ────────────────────────────────────
  If the current character already exists on an edge,
  do nothing (it's implicitly represented).
  Stop processing remaining suffixes for this phase.

  Suffix links: Pointers between internal nodes for
  fast traversal from one suffix to the next.

  ┌─────────────────────────────────────────┐
  │  Ukkonen's phases: n phases             │
  │  Each phase: amortized O(1) extensions  │
  │  Total: O(n)                            │
  └─────────────────────────────────────────┘
```

---

## 4. Pattern Matching with Suffix Tree

```
  To find if pattern P exists in text T:
  1. Build suffix tree for T
  2. Follow P from the root, character by character
  3. If we can follow all |P| characters → P exists
  4. Count leaves in the subtree → number of occurrences

  Example: T = "banana$", P = "ana"

       (root)
      /   |    \     \
    a     banana$*   na     $*
    |                |
   / \              / \
  na   $*          na   $*
  |                |
  na$* a$*         $*

  Follow "ana":
    root → 'a' edge → 'n' → 'a' → found!
    Subtree has 2 leaves → "ana" occurs 2 times
    At positions 1 and 3.

  Time: O(|P|) for search, O(|P| + k) to report k occurrences.
```

---

## 5. Key Applications

```
  ┌──────────────────────────────────────────────────────────┐
  │  1. Exact pattern matching:     O(m)                     │
  │  2. All occurrences:            O(m + occ)               │
  │  3. Longest repeated substring: deepest internal node    │
  │  4. Longest common substring:   generalized suffix tree  │
  │  5. Longest palindromic substr: combined with tree       │
  │  6. Number of distinct substrings: count nodes           │
  │  7. Shortest unique substring:  shallowest unique path   │
  │  8. String compression (LZ):   factorize using tree     │
  └──────────────────────────────────────────────────────────┘

  Longest repeated substring:
    Find the deepest internal node (most characters on
    root-to-node path). It has ≥ 2 children → ≥ 2 occurrences.

  Distinct substrings:
    Each edge label contributes (end - start + 1) new substrings.
    Sum across all edges = number of distinct substrings.
```

---

## 6. Suffix Tree vs Suffix Array

```
  ┌──────────────────┬───────────────┬───────────────────┐
  │ Aspect           │ Suffix Tree   │ Suffix Array      │
  ├──────────────────┼───────────────┼───────────────────┤
  │ Build time       │ O(n)          │ O(n log n) or O(n)│
  │ Space            │ O(n) ×20 const│ O(n) ×4 const     │
  │ Pattern search   │ O(m)          │ O(m log n)        │
  │ Implementation   │ Very complex  │ Moderate           │
  │ Cache behavior   │ Poor          │ Good               │
  │ Practical speed  │ Slower (const)│ Faster (const)     │
  │ Flexibility      │ Higher        │ Needs LCP array    │
  └──────────────────┴───────────────┴───────────────────┘

  In competitive programming:
  Suffix arrays are preferred due to simpler implementation
  and lower memory. Suffix trees are preferred in theory
  for their O(m) search guarantee.
```

---

## 7. Generalized Suffix Tree

```
  Build a single suffix tree for MULTIPLE strings.

  S1 = "ab"   S2 = "ba"

  Concatenate: "ab$₁ba$₂" or build incrementally.

  Each leaf is labeled with (string_id, position).

  Applications:
  - Longest common substring of k strings
  - Shared/unique substrings
  - Multiple pattern matching

  ┌─────────────────────────────────────────────────────┐
  │  For k strings of total length N:                   │
  │  Build time: O(N)                                   │
  │  Space: O(N)                                        │
  │  LCS of k strings: O(N) using DFS                   │
  └─────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Feature | Details |
|---------|---------|
| Structure | Compressed trie of all suffixes |
| Nodes | At most 2n - 1 |
| Space | O(n) using (start, end) edge labels |
| Construction | Ukkonen's O(n) |
| Pattern search | O(m) |
| All occurrences | O(m + occ) |
| Longest repeated substring | Deepest internal node |
| Distinct substrings | Sum of edge label lengths |

---

## ❓ Quick Revision Questions

1. **How does a suffix tree differ from a suffix trie?**
   <details><summary>Answer</summary>A suffix trie has one character per edge and can have O(n²) nodes. A suffix tree compresses chains of single-child nodes into single edges with substring labels, giving at most 2n-1 nodes and O(n) space.</details>

2. **Why is there a sentinel character '$'?**
   <details><summary>Answer</summary>The sentinel ensures no suffix is a prefix of another, guaranteeing exactly n leaves. Without it, some suffixes might end at internal nodes, complicating the structure.</details>

3. **What are the three tricks in Ukkonen's algorithm?**
   <details><summary>Answer</summary>1) Implicit extensions: leaf edges extend automatically via global end pointer. 2) Skip/count: traverse edges by length, not character-by-character. 3) Rule 3 (early termination): if character exists on current edge, stop processing.</details>

4. **How to find the longest repeated substring?**
   <details><summary>Answer</summary>Find the deepest internal node (maximum string depth from root). Its path label is the longest repeated substring. It's repeated because internal nodes have ≥ 2 children (≥ 2 suffixes pass through).</details>

5. **When to prefer suffix tree over suffix array?**
   <details><summary>Answer</summary>When O(m) pattern search is critical (vs O(m log n)), when solving problems requiring tree traversal (LCA queries, DFS), or when building a generalized suffix tree for multiple strings. Otherwise, suffix arrays are simpler and more cache-friendly.</details>

---

| [⬅️ Previous: Suffix Array](01-suffix-array.md) | [Next: Burrows-Wheeler Transform ➡️](03-burrows-wheeler-transform.md) |
|:---|---:|
