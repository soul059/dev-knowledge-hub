# Chapter 7.4 — Word Dictionary with Wildcards

> **Unit 7: Trie Applications** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **word dictionary** supporting wildcard search (`'.'` matches any
single character) is a classic trie problem (LeetCode 211). This chapter
covers design, DFS-based wildcard matching, and optimizations.

---

## 1. Problem Definition

```
  Design a data structure that supports:
    1. addWord(word)    — Add a word to the dictionary
    2. search(word)     — Search; '.' matches any single character

  Example:
    addWord("bad")
    addWord("dad")
    addWord("mad")
    search("pad")  → false
    search("bad")  → true
    search(".ad")  → true  (matches bad, dad, mad)
    search("b..")  → true  (matches bad)
    search("...")  → true  (matches bad, dad, mad)
    search(".a.")  → true
    search("b.d")  → true
```

---

## 2. Trie Structure

```
  After adding "bad", "dad", "mad":

        (root)
      /   |   \
    b     d     m
    |     |     |
    a     a     a
    |     |     |
   d*    d*    d*

  * = end of word

  Standard insert: traverse, create nodes as needed,
  mark last node as end.
```

---

## 3. Wildcard Search Strategy

```
  search(".ad"):

  At root, '.' encountered → try ALL children:
    → Branch 'b': match 'a' → match 'd' → is_end? YES ✓
    → Branch 'd': match 'a' → match 'd' → is_end? YES ✓
    → Branch 'm': match 'a' → match 'd' → is_end? YES ✓
  Result: true (found at least one match)

  search("b.d"):

  At root, 'b' → go to child 'b'
  At 'b', '.' → try ALL children:
    → Branch 'a': match 'd' → is_end? YES ✓
  Result: true

  Key idea:
  ┌────────────────────────────────────────────────┐
  │  '.' causes BRANCHING — DFS explores all       │
  │  children. Regular chars follow one path.       │
  └────────────────────────────────────────────────┘
```

---

## 4. Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}   # char → TrieNode
        self.is_end = False

class WordDictionary:
    def __init__(self):
        self.root = TrieNode()
    
    def addWord(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
    
    def search(self, word: str) -> bool:
        return self._dfs(self.root, word, 0)
    
    def _dfs(self, node, word, index):
        if index == len(word):
            return node.is_end
        
        ch = word[index]
        
        if ch == '.':
            # Wildcard: try every child
            for child in node.children.values():
                if self._dfs(child, word, index + 1):
                    return True
            return False
        else:
            # Regular character: follow specific path
            if ch not in node.children:
                return False
            return self._dfs(node.children[ch], word, index + 1)
```

---

## 5. Complexity Analysis

```
  addWord(word):
    Time:  O(m)          m = word length
    Space: O(m)          worst case new nodes

  search(word) without wildcards:
    Time:  O(m)          single path

  search(word) with wildcards:
    Time:  O(26^k × m)   k = number of '.' characters
    Worst: O(26^m)        all wildcards "..."

  ┌──────────────────────────────────────────────┐
  │  Practical Performance:                      │
  │  - Words share prefixes → fewer branches     │
  │  - '.' at end is cheaper than at start       │
  │  - Typically much better than worst case     │
  └──────────────────────────────────────────────┘

  Space: O(N × m)  total characters across all words
```

---

## 6. Trace Example

```
  Dictionary: {"apple", "apply", "ape", "bat"}

  Trie:       (root)
             /      \
            a        b
            |        |
            p        a
          /   \      |
         p     e*    t*
         |
         l
        / \
       e*  y*

  search("ap.le"):
    root → 'a' → child 'a'
    'a'  → 'p' → child 'p'
    'p'  → '.' → try all children:
      child 'p': → 'l' → child 'l' → 'e' → child 'e' → is_end? YES ✓
      child 'e': → 'l' → no child 'l' ✗
    Result: true (matched "apple")

  search("a.."):
    root → 'a' → child 'a'
    'a'  → '.' → try all children:
      child 'p': → '.' → try all children:
        child 'p': is_end? NO ✗
        child 'e': is_end? YES ✓ → return true!
    Result: true (matched "ape")

  search("b.."):
    root → 'b' → child 'b'
    'b'  → '.' → try all children:
      child 'a': → '.' → try all children:
        child 't': is_end? YES ✓ → return true!
    Result: true (matched "bat")
```

---

## 7. Optimization: Length Bucketing

```
  For queries like "...", we only need words of length 3.
  Store words by length in separate structures.

  length_map = {
      3: ["bat", "ape"],
      5: ["apple", "apply"]
  }

  search("...") → only check words in length_map[3]

  Hybrid approach:
  ┌──────────────────────────────────────────────┐
  │  If query has NO wildcards → use trie         │
  │  If query is ALL wildcards → use length map   │
  │  Otherwise → use trie with DFS               │
  └──────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Operation | Standard | With '.' Wildcard |
|-----------|----------|-------------------|
| addWord | O(m) | O(m) — same |
| search | O(m) | O(26^k × m), k = # of '.' |
| Space | O(total chars) | O(total chars) |
| Approach | Direct traversal | DFS branching at '.' |
| Data structure | Trie (HashMap children) | Same trie |

---

## ❓ Quick Revision Questions

1. **How does '.' work differently from a regular character during search?**
   <details><summary>Answer</summary>A regular character follows exactly one child edge. '.' triggers a DFS that tries ALL children of the current node, returning true if any branch leads to a match.</details>

2. **What is the worst-case time for searching "..." (all dots)?**
   <details><summary>Answer</summary>O(26^m) where m is the pattern length — every node could have up to 26 children at each level. In practice it's much less since tries share prefixes.</details>

3. **Why is DFS preferred over BFS for wildcard search?**
   <details><summary>Answer</summary>DFS can return immediately upon finding the first match (early termination), uses less memory than BFS (stack vs queue of many nodes), and naturally handles the recursive structure of trie + wildcards.</details>

4. **How does the length-bucketing optimization help?**
   <details><summary>Answer</summary>For queries with all wildcards (e.g., "..."), bucketing by word length lets us check only words of matching length, avoiding traversal of the entire trie.</details>

5. **Can this approach handle multiple wildcards like "a.p.e"?**
   <details><summary>Answer</summary>Yes. Each '.' causes branching at that position. With k wildcards, we branch at k positions but can still prune when paths don't exist in the trie. The complexity is O(26^k × m).</details>

---

| [⬅️ Previous: Spell Checker](03-spell-checker-concept.md) | [Next: Prefix Counting ➡️](05-prefix-counting.md) |
|:---|---:|
