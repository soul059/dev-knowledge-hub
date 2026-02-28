# Prefix Count

## Overview
Counting how many words share a given prefix is a fundamental trie operation. This chapter covers naive and optimized approaches, including the `prefix_count` field technique that answers prefix count queries in O(P) time — the same as a simple prefix search.

---

## Problem Variants

```
  Given a trie with words, answer queries like:
  
  1. countWordsWithPrefix("app")  → How many words start with "app"?
  2. countWordsEqualTo("app")     → How many times was "app" inserted?
  3. countWordsStartingWith(char) → How many words start with 'a'?
```

---

## Approach 1: DFS Count (Naive)

Navigate to prefix node, then DFS to count all `is_end` nodes:

```python
class Trie:
    def count_words_with_prefix(self, prefix):
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return self._count_dfs(node)
    
    def _count_dfs(self, node):
        count = 1 if node.is_end else 0
        for child in node.children.values():
            count += self._count_dfs(child)
        return count
```

```
  Time: O(P + K) where K = total nodes in subtree
  
  For prefix "ap" in a trie with 100,000 words starting with "ap":
  → Must visit ALL 100,000 word paths!
  → Very slow for common prefixes
```

---

## Approach 2: Prefix Count Field (Optimal)

Maintain a counter at each node during insertion:

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.end_count = 0     # Words ending at this node
        self.prefix_count = 0  # Words passing through this node

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
            node.prefix_count += 1  # ← Increment on every node
        node.end_count += 1
    
    def count_words_equal_to(self, word):
        """How many times was this exact word inserted?"""
        node = self.root
        for ch in word:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.end_count
    
    def count_words_starting_with(self, prefix):
        """How many words have this prefix?"""
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.prefix_count  # ← O(P) answer!
    
    def erase(self, word):
        """Remove one occurrence (assumes word exists)."""
        node = self.root
        for ch in word:
            node = node.children[ch]
            node.prefix_count -= 1  # ← Decrement along path
        node.end_count -= 1
```

```
  Time: O(P) for count_words_starting_with — no DFS needed!
  Space: +4 bytes per node (one integer)
```

---

## Step-by-Step Trace

```
  Operations:
  1. insert("apple")
  2. insert("app")
  3. insert("apple")
  4. insert("apex")
  5. count_words_starting_with("app")
  6. count_words_equal_to("apple")
  7. erase("apple")
  8. count_words_starting_with("app")
  
  ── After insert("apple") ──
  
  root ── a(pc=1) ── p(pc=1) ── p(pc=1) ── l(pc=1) ── e(pc=1, ec=1)
  
  ── After insert("app") ──
  
  root ── a(pc=2) ── p(pc=2) ── p(pc=2, ec=1) ── l(pc=1) ── e(pc=1, ec=1)
  
  ── After insert("apple") [again] ──
  
  root ── a(pc=3) ── p(pc=3) ── p(pc=3, ec=1) ── l(pc=2) ── e(pc=2, ec=2)
  
  ── After insert("apex") ──
  
  root ── a(pc=4) ── p(pc=4) ── p(pc=3, ec=1) ── l(pc=2) ── e(pc=2, ec=2)
                                  └── e(pc=1) ── x(pc=1, ec=1)
  
  pc = prefix_count, ec = end_count
  
  ── Queries ──
  
  count_words_starting_with("app"):
    Navigate: a→p→p
    p.prefix_count = 3
    Answer: 3  (apple×2 + app×1)
  
  count_words_equal_to("apple"):
    Navigate: a→p→p→l→e
    e.end_count = 2
    Answer: 2
  
  ── erase("apple") ──
  
  Decrement path: a(3)→p(3)→p(2,ec=1)→l(1)→e(1,ec=1)
  
  count_words_starting_with("app"):
    Navigate: a→p→p
    p.prefix_count = 2
    Answer: 2  (apple×1 + app×1)
```

---

## LeetCode 1804: Implement Trie II

This problem requires exactly these three operations:

```python
class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
            node.prefix_count += 1
        node.end_count += 1
    
    def countWordsEqualTo(self, word: str) -> int:
        node = self.root
        for ch in word:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.end_count
    
    def countWordsStartingWith(self, prefix: str) -> int:
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.prefix_count
    
    def erase(self, word: str) -> None:
        node = self.root
        for ch in word:
            node = node.children[ch]
            node.prefix_count -= 1
        node.end_count -= 1
```

---

## Prefix Count with Node Cleanup

The basic `erase` doesn't remove empty nodes. Here's a version that does:

```python
def erase_with_cleanup(self, word):
    """Remove word and clean up empty nodes."""
    # First verify word exists
    if self.count_words_equal_to(word) == 0:
        return
    
    node = self.root
    for i, ch in enumerate(word):
        child = node.children[ch]
        child.prefix_count -= 1
        
        if child.prefix_count == 0:
            # No more words pass through — remove entire subtree
            del node.children[ch]
            return
        
        node = child
    
    node.end_count -= 1
```

---

## Comparison: DFS vs Prefix Count Field

| Aspect | DFS Count | Prefix Count Field |
|--------|:---------:|:------------------:|
| Query time | O(P + K) | O(P) |
| Insert time | O(L) | O(L) |
| Delete time | O(L) | O(L) |
| Extra space | None | +1 int per node |
| Handles duplicates | Need count + DFS | Native support |
| Implementation | Simple | Slightly more complex |
| Best for | One-time queries | Repeated queries |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Naive approach | Navigate + DFS count: O(P + K) |
| Optimal approach | `prefix_count` field: O(P) |
| Insert cost | Increment prefix_count at every node: O(L) |
| Delete cost | Decrement prefix_count along path: O(L) |
| LeetCode | 1804 — Implement Trie II |
| Key insight | Count words as they pass through each node |

---

## Quick Revision Questions

1. **What is `prefix_count` and when is it incremented?**
   > `prefix_count` at a node tracks how many inserted words pass through it. It's incremented for every node along the word's path during insertion (not just the terminal node).

2. **How is `prefix_count` different from `end_count`?**
   > `end_count` only records completions at that exact node. `prefix_count` records all words that use this node as part of their path. For node 'p' in "app" with words "app" and "apple", `end_count = 1` (just "app") but `prefix_count = 2` (both words pass through).

3. **What happens if you forget to decrement `prefix_count` during deletion?**
   > The count becomes stale. `count_words_starting_with()` will over-count, reporting deleted words as still existing.

4. **Can `prefix_count` ever be less than `end_count` at a node?**
   > No. `prefix_count` >= `end_count` always, because every word ending at this node also passes through it. `prefix_count` also includes words that continue beyond this node.

5. **Why is O(P) better than O(P + K) for prefix count queries?**
   > K can be enormous for common prefixes. Counting words starting with "a" might require visiting millions of nodes with DFS, while `prefix_count` answers instantly at depth 1.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Search Suggestions](02-search-suggestions.md) | [Next: Longest Common Prefix ▶](04-longest-common-prefix.md) |
