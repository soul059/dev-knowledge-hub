# Chapter 7.1 — Trie for Pattern Matching

> **Unit 7: Trie Applications** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **Trie** (prefix tree) is a tree-based data structure for storing strings
that enables O(m) search, insert, and prefix queries. This chapter covers
trie construction, operations, and its role in pattern matching.

---

## 1. Trie Structure

```
  A trie stores strings by sharing common prefixes.

  Words: "apple", "app", "ape", "bat", "bar"

              (root)
             /      \
            a        b
           /          \
          p            a
         / \          / \
        p   e*       t*  r*
       /
      l
     /
    e*

  * = end-of-word marker

  Each path from root to a marked node represents a stored word.
  Common prefix "ap" is stored ONCE, shared by "apple", "app", "ape".
```

---

## 2. Trie Node Structure

```
  ┌──────────────────────────────────────────────┐
  │  TrieNode:                                   │
  │    children: map/array of child nodes        │
  │    is_end: boolean (marks end of word)       │
  │    count: int (number of words through here) │
  │    end_count: int (words ending here)        │
  └──────────────────────────────────────────────┘

  Two implementations:
  
  Array-based (fixed alphabet):
    children = array of 26 (for lowercase letters)
    Memory: O(26 × total_nodes)
    Lookup: O(1)

  HashMap-based (variable alphabet):
    children = dictionary/hashmap
    Memory: O(total_edges)
    Lookup: O(1) average
```

---

## 3. Core Operations

### Insert — O(m)

```
  function Insert(root, word):
      node = root
      for char in word:
          if char not in node.children:
              node.children[char] = new TrieNode()
          node = node.children[char]
          node.count += 1      // words passing through
      node.is_end = True
      node.end_count += 1

  Trace: Insert "app" then "ape"
  
  Insert "app":
    root → a(new) → p(new) → p(new,end*)
  
  Insert "ape":
    root → a(exists) → p(exists) → e(new,end*)

              root
               |
               a  (count=2)
               |
               p  (count=2)
              / \
         p(end) e(end)
```

### Search — O(m)

```
  function Search(root, word):
      node = root
      for char in word:
          if char not in node.children:
              return False
          node = node.children[char]
      return node.is_end
```

### Prefix Search — O(m)

```
  function StartsWith(root, prefix):
      node = root
      for char in prefix:
          if char not in node.children:
              return False
          node = node.children[char]
      return True    // don't check is_end!
```

---

## 4. Implementation — Python

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False
        self.count = 0      # words passing through this node

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
            node.count += 1
        node.is_end = True
    
    def search(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                return False
            node = node.children[c]
        return node.is_end
    
    def starts_with(self, prefix):
        node = self.root
        for c in prefix:
            if c not in node.children:
                return False
            node = node.children[c]
        return True
    
    def count_prefix(self, prefix):
        node = self.root
        for c in prefix:
            if c not in node.children:
                return 0
            node = node.children[c]
        return node.count
```

---

## 5. Trie for Multi-Pattern Matching

```
  Problem: Given text T and a set of patterns P₁...Pₖ,
           find all patterns that appear in T.

  Approach 1: Insert all patterns into trie.
  For each position i in T, traverse from root matching T[i], T[i+1], ...
  Any node marked as end → pattern found!

  Time: O(n × L) where L = longest pattern length
  (Not great — Aho-Corasick improves this to O(n + total_pattern_length))
```

```python
def find_patterns_in_text(trie, text):
    """Find all trie patterns occurring in text."""
    results = []
    n = len(text)
    
    for i in range(n):
        node = trie.root
        j = i
        while j < n and text[j] in node.children:
            node = node.children[text[j]]
            if node.is_end:
                results.append((i, text[i:j+1]))
            j += 1
    
    return results
```

---

## 6. Complexity Analysis

```
  ┌───────────────────┬──────────────┬───────────────────┐
  │ Operation         │ Time         │ Space             │
  ├───────────────────┼──────────────┼───────────────────┤
  │ Insert word       │ O(m)         │ O(m) worst case   │
  │ Search word       │ O(m)         │ O(1)              │
  │ Prefix search     │ O(m)         │ O(1)              │
  │ Count with prefix │ O(m)         │ O(1)              │
  │ Build trie (k     │ O(Σ|Pᵢ|)    │ O(Σ|Pᵢ| × |Σ|)  │
  │ patterns)         │              │                   │
  └───────────────────┴──────────────┴───────────────────┘

  Space: O(total characters × alphabet size) worst case
  In practice, shared prefixes save significant space.
```

---

## 📝 Summary Table

| Feature | Trie | Hash Set | Sorted Array |
|---------|------|----------|--------------|
| Insert | O(m) | O(m) avg | O(m log n) |
| Search | O(m) | O(m) avg | O(m log n) |
| Prefix query | O(p) ✓ | O(n×m) ✗ | O(m log n) |
| Space | High | Medium | Low |
| Ordered iteration | ✓ | ✗ | ✓ |

---

## ❓ Quick Revision Questions

1. **What is the time complexity of inserting a word of length m into a trie?**
   <details><summary>Answer</summary>O(m). We traverse or create at most m nodes, one per character.</details>

2. **What is the difference between search and starts_with?**
   <details><summary>Answer</summary>Search checks that the word exists AND is marked as a complete word (is_end = True). starts_with only checks that the prefix path exists, regardless of end markers.</details>

3. **Why are tries better than hash sets for prefix operations?**
   <details><summary>Answer</summary>Tries share common prefixes in the tree structure, enabling O(p) prefix searches. Hash sets would need to check every stored string, taking O(n×m).</details>

4. **What is the space overhead of an array-based trie for lowercase letters?**
   <details><summary>Answer</summary>Each node stores an array of 26 pointers, most of which may be null. Space = O(26 × number_of_nodes), which can be wasteful for sparse tries.</details>

5. **How does a trie provide multi-pattern matching?**
   <details><summary>Answer</summary>Insert all patterns into the trie. For each position in the text, walk down the trie matching characters. Each end-of-word node encountered indicates a pattern match at that position.</details>

---

| [⬅️ Previous Unit: Double Hashing](../06-String-Hashing/06-double-hashing.md) | [Next: Autocomplete ➡️](02-autocomplete.md) |
|:---|---:|
