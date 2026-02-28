# Chapter 7.5 — Prefix Counting

> **Unit 7: Trie Applications** | [Course Home](../README.md)

---

## 📋 Chapter Overview

**Prefix counting** — determining how many stored words share a given
prefix — is one of the most natural and efficient trie operations.
This chapter covers the technique and common variations.

---

## 1. Core Idea

```
  Store a count at each trie node tracking how many words
  pass through (or end at) that node.

  Dictionary: {"apple", "apply", "ape", "bat", "bar"}

        (root) cnt=5
        /          \
      a cnt=3      b cnt=2
      |            |
      p cnt=3      a cnt=2
     / \          / \
    p   e*       t*  r*
    cnt=2 cnt=1  cnt=1 cnt=1
    |
    l cnt=2
   / \
  e*  y*
  cnt=1 cnt=1

  countPrefix("ap") → node 'p' has cnt=3 → 3 words
  countPrefix("app") → node 'p'(2nd) has cnt=2 → 2 words
  countPrefix("b") → node 'b' has cnt=2 → 2 words
  countPrefix("c") → no node → 0 words
```

---

## 2. Two Counting Strategies

```
  Strategy 1: prefix_count (count at each node = words through it)
  ─────────────────────────────────────────────────────────────────
  Increment counter at EVERY node along the insertion path.
  Answers: "How many words have this prefix?"

  Strategy 2: end_count (count only at terminal nodes)
  ─────────────────────────────────────────────────────
  Increment counter only at the final node.
  Answers: "How many times was this exact word inserted?"
  (Useful when duplicates are allowed)

  Best practice: maintain BOTH counts.

  ┌──────────────────────────────────────────┐
  │  Node:                                   │
  │    children: {char → Node}               │
  │    prefix_count: int   # words through   │
  │    end_count: int      # words ending    │
  └──────────────────────────────────────────┘
```

---

## 3. Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.prefix_count = 0   # words passing through this node
        self.end_count = 0      # words ending at this node

class PrefixTrie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
            node.prefix_count += 1    # every node on path
        node.end_count += 1           # only last node

    def count_prefix(self, prefix: str) -> int:
        """How many words start with this prefix?"""
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.prefix_count

    def count_exact(self, word: str) -> int:
        """How many times was this exact word inserted?"""
        node = self.root
        for ch in word:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.end_count

    def delete(self, word: str) -> bool:
        """Delete one occurrence of word. Returns False if not found."""
        # First verify word exists
        node = self.root
        path = [self.root]
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
            path.append(node)
        if node.end_count == 0:
            return False

        # Decrement counts along path
        node = self.root
        for ch in word:
            node = node.children[ch]
            node.prefix_count -= 1
        node.end_count -= 1

        # Optional: clean up empty nodes (prefix_count == 0)
        return True
```

---

## 4. Trace: Insert and Query

```
  Operations:
    insert("tea")
    insert("team")
    insert("tear")
    insert("ten")
    insert("the")

  Trie after all inserts:

         (root)
           |
          t pfx=5
          |
          e pfx=4 ──────── h pfx=1
         / \                |
        a   n               e pfx=1, end=1
 pfx=3  pfx=1,end=1
       / \
      m    r
 pfx=1   pfx=1
 end=1   end=1
   ↑
  (also: "tea" → 'a' has end=1)

  Queries:
    count_prefix("t")   → 5  (all words)
    count_prefix("te")  → 4  (tea, team, tear, ten)
    count_prefix("tea") → 3  (tea, team, tear)
    count_prefix("the") → 1  (the)
    count_prefix("ti")  → 0  (no words)
    count_exact("tea")  → 1
    count_exact("team") → 1
    count_exact("xyz")  → 0
```

---

## 5. Common Problems

### 5.1 Unique Prefix Count

```
  Given N words, for each word find the shortest unique prefix.

  Words: ["zebra", "zen", "zoo", "zone"]

  Trie with prefix counts:
    z pfx=4
   / \
  e   o
  pfx=2 pfx=2
  |    / \
  ...  o   n
       pfx=1 pfx=1

  Shortest unique prefix = first node where prefix_count == 1
    "zebra" → "zeb"  (at 'b', pfx=1)
    "zen"   → "zen"  (at 'n', pfx=1)
    "zoo"   → "zoo"  (at 'o' after 'zo', pfx=1)
    "zone"  → "zon"  (at 'n', pfx=1)
```

### 5.2 Sum of Prefix Scores

```
  Given words[], for each word compute sum of count_prefix
  for all its prefixes.

  Words: ["abc", "ab", "b"]
  
  Trie:   (root)
          /    \
        a(2)   b(1,end=1)
        |
       b(2,end=1)
        |
       c(1,end=1)

  score("abc") = pfx("a") + pfx("ab") + pfx("abc")
               = 2 + 2 + 1 = 5
  score("ab")  = pfx("a") + pfx("ab") = 2 + 2 = 4
  score("b")   = pfx("b") = 1

  Algorithm: DFS through trie, accumulate prefix_count sum.
```

---

## 6. Deletion with Prefix Count

```
  Before delete("team"):
    t pfx=5 → e pfx=4 → a pfx=3,end=1 → m pfx=1,end=1

  After delete("team"):
    t pfx=4 → e pfx=3 → a pfx=2,end=1 → m pfx=0,end=0

  Node 'm' has pfx=0 → can be pruned (optional cleanup)
  "tea" still exists (end=1 at 'a')

  ┌──────────────────────────────────────────┐
  │  Deletion decrements prefix_count along  │
  │  the entire path, preserving correctness │
  │  for all prefix queries.                 │
  └──────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Operation | Time | Description |
|-----------|------|-------------|
| insert | O(m) | Increment prefix_count at each node |
| count_prefix | O(p) | Traverse prefix, return prefix_count |
| count_exact | O(m) | Traverse word, return end_count |
| delete | O(m) | Decrement counts along path |
| unique prefix | O(m) | First node with prefix_count == 1 |
| sum of prefix scores | O(total chars) | DFS accumulating counts |

---

## ❓ Quick Revision Questions

1. **What is the difference between prefix_count and end_count?**
   <details><summary>Answer</summary>prefix_count at a node equals the number of words in the trie that pass through that node (i.e., words having that prefix). end_count counts how many words end exactly at that node.</details>

2. **How do you find the number of words starting with "app"?**
   <details><summary>Answer</summary>Traverse the trie following 'a' → 'p' → 'p'. Return the prefix_count at the final node. If any character is missing, return 0.</details>

3. **How does deletion maintain prefix_count correctness?**
   <details><summary>Answer</summary>When deleting a word, decrement prefix_count at every node along the path and decrement end_count at the terminal node. This ensures all prefix queries remain accurate.</details>

4. **What is a "shortest unique prefix" and how to find it?**
   <details><summary>Answer</summary>The shortest prefix of a word that uniquely identifies it among all dictionary words. Found by traversing the trie until reaching a node with prefix_count == 1.</details>

5. **How to compute prefix scores for all words efficiently?**
   <details><summary>Answer</summary>Insert all words, building prefix_count. For each word, traverse its path and sum the prefix_count values at each node. Total time is O(sum of all word lengths).</details>

6. **Can prefix_count handle duplicate word insertions?**
   <details><summary>Answer</summary>Yes. Each insertion increments prefix_count along the path, so "apple" inserted 3 times gives prefix_count=3 at the 'e' node. end_count=3 at the terminal confirms exact duplicates.</details>

---

| [⬅️ Previous: Word Dictionary](04-word-dictionary.md) | [Next: XOR with Trie ➡️](06-xor-with-trie.md) |
|:---|---:|
