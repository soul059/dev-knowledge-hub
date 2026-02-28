# Boolean vs Count Marking

## Overview
This chapter compares the two primary ways to mark word endings: a simple boolean flag (`is_end = True/False`) and an integer counter (`count = N`). Understanding when to use each is critical for solving different categories of problems correctly.

---

## Boolean Approach

### Structure

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False  # Binary: word is here or not
```

### Behavior

```
  Insert "apple":     is_end = True
  Insert "apple" again: is_end = True  (no change!)
  Delete "apple":     is_end = False
  
  ┌─────────────────────────────────────────────────┐
  │  Boolean treats the trie as a SET:              │
  │  • Each word is either present or absent        │
  │  • Duplicate inserts have no effect             │
  │  • Single delete removes the word entirely      │
  └─────────────────────────────────────────────────┘
```

### Implementation

```python
class BooleanTrie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
    
    def search(self, word):
        node = self._find(word)
        return node is not None and node.is_end
    
    def delete(self, word):
        node = self._find(word)
        if node and node.is_end:
            node.is_end = False
            return True
        return False
    
    def _find(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children: return None
            node = node.children[ch]
        return node
```

---

## Count Approach

### Structure

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.count = 0  # How many times this word was inserted
```

### Behavior

```
  Insert "apple":       count = 1
  Insert "apple" again: count = 2
  Insert "apple" again: count = 3
  Delete "apple":       count = 2  (still exists!)
  Delete "apple":       count = 1
  Delete "apple":       count = 0  (now removed)
  
  ┌─────────────────────────────────────────────────┐
  │  Count treats the trie as a MULTISET (BAG):     │
  │  • Same word can appear multiple times          │
  │  • Each delete removes one occurrence           │
  │  • word exists when count > 0                   │
  └─────────────────────────────────────────────────┘
```

### Implementation

```python
class CountTrie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.count += 1
    
    def search(self, word):
        node = self._find(word)
        return node is not None and node.count > 0
    
    def count_word(self, word):
        """Return frequency of word."""
        node = self._find(word)
        return node.count if node else 0
    
    def delete(self, word):
        """Remove one occurrence."""
        node = self._find(word)
        if node and node.count > 0:
            node.count -= 1
            return True
        return False
    
    def delete_all(self, word):
        """Remove all occurrences."""
        node = self._find(word)
        if node and node.count > 0:
            removed = node.count
            node.count = 0
            return removed
        return 0
    
    def _find(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children: return None
            node = node.children[ch]
        return node
```

---

## Side-by-Side Comparison

```
  Scenario: Insert "apple" 3 times, then delete once
  
  BOOLEAN:                        COUNT:
  insert("apple") → is_end=T     insert("apple") → count=1
  insert("apple") → is_end=T     insert("apple") → count=2
  insert("apple") → is_end=T     insert("apple") → count=3
  delete("apple") → is_end=F     delete("apple") → count=2
  search("apple") → False ✗      search("apple") → True  ✓
                   (word gone!)                    (2 copies remain)
```

---

## Visual Trace: Mixed Operations

```
  Operations:
  1. insert("cat")
  2. insert("cat")
  3. insert("car")
  4. delete("cat")
  5. search("cat")
  
  ┌───────────────────────┬──────────────────────┐
  │  Boolean              │  Count               │
  ├───────────────────────┼──────────────────────┤
  │  1. t.isEnd = True    │  1. t.count = 1      │
  │  2. t.isEnd = True    │  2. t.count = 2      │
  │     (no change)       │     (incremented)    │
  │  3. r.isEnd = True    │  3. r.count = 1      │
  │  4. t.isEnd = False   │  4. t.count = 1      │
  │  5. "cat" → False     │  5. "cat" → True     │
  │     (completely gone) │     (1 copy remains)  │
  └───────────────────────┴──────────────────────┘
```

---

## Decision Guide: When to Use Which

```
  ┌──────────────────────────────────────────────────────┐
  │  Use BOOLEAN when:                                    │
  │                                                       │
  │  ✓ Words are unique (no duplicates)                   │
  │  ✓ Problem says "set of words"                        │
  │  ✓ Standard trie problems (LeetCode 208)              │
  │  ✓ You want simpler code                              │
  │  ✓ Memory is tight (bool < int)                       │
  │                                                       │
  ├──────────────────────────────────────────────────────┤
  │  Use COUNT when:                                      │
  │                                                       │
  │  ✓ Duplicates are meaningful                          │
  │  ✓ Frequency counting (word count in text)            │
  │  ✓ Multiset operations needed                         │
  │  ✓ "Delete one occurrence" semantics                  │
  │  ✓ Stream processing (add/remove words over time)     │
  │  ✓ Anagram / word frequency problems                  │
  └──────────────────────────────────────────────────────┘
```

---

## Combined Approach: Count + Prefix Count

For maximum flexibility:

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.end_count = 0     # Words ending here
        self.prefix_count = 0  # Words passing through

class PowerTrie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
            node.prefix_count += 1
        node.end_count += 1
    
    def count_word(self, word):
        """Exact frequency of this word."""
        node = self._find(word)
        return node.end_count if node else 0
    
    def count_prefix(self, prefix):
        """Number of words with this prefix."""
        node = self._find(prefix)
        return node.prefix_count if node else 0
    
    def delete(self, word):
        if self.count_word(word) == 0:
            return False
        node = self.root
        for ch in word:
            node = node.children[ch]
            node.prefix_count -= 1
        node.end_count -= 1
        return True
    
    def _find(self, s):
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
```

### Example Trace

```
  insert("app"):   prefix_count path: a(1) p(1) p(1, end=1)
  insert("apple"): prefix_count path: a(2) p(2) p(2, end=1) l(1) e(1, end=1)
  insert("app"):   prefix_count path: a(3) p(3) p(3, end=2) — existing l/e unchanged
  
  count_word("app")    → 2
  count_prefix("app")  → 3  (includes "app" × 2 and "apple" × 1)
  
  delete("app"):  prefix_count path: a(2) p(2) p(2, end=1)
  count_word("app")    → 1
  count_prefix("app")  → 2
```

---

## Common LeetCode Problem Types

| Problem Type | Marking Strategy | Example |
|-------------|:----------------:|---------|
| Implement Trie | Boolean | LC 208 |
| Word frequency | Count | Custom |
| Prefix count | Prefix count | LC 1804 |
| Map Sum Pairs | Value storage | LC 677 |
| Add/Search Word | Boolean | LC 211 |

---

## Summary Table

| Feature | Boolean | Count |
|---------|:-------:|:-----:|
| Word exists? | `is_end == True` | `count > 0` |
| Insert duplicate | No-op | Increments |
| Delete behavior | Removes entirely | Removes one copy |
| Memory per node | 1 bit | 4-8 bytes |
| Use case | Set | Multiset / Bag |
| Code complexity | Simpler | Slightly more |

---

## Quick Revision Questions

1. **What's the key behavioral difference between boolean and count marking?**
   > Boolean treats the trie as a set (word is present or absent). Count treats it as a multiset (same word can exist multiple times). Deleting with boolean removes the word entirely; with count, it removes one occurrence.

2. **If you insert "hello" 5 times with boolean marking, then delete once, what happens?**
   > `is_end` becomes `False`. The word is completely gone because boolean doesn't track duplicates.

3. **How do you implement "delete all occurrences" with count marking?**
   > Set `node.count = 0` at the terminal node (instead of decrementing by 1).

4. **Why might you combine `end_count` and `prefix_count` in one node?**
   > `end_count` tells how many times a specific word was inserted. `prefix_count` tells how many words pass through this node. Together they support word queries AND prefix queries in O(L) and O(P).

5. **For LeetCode 208 (Implement Trie), which approach is expected?**
   > Boolean (`is_end`). The problem defines set semantics — no duplicate tracking is needed.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Case Sensitivity](04-case-sensitivity.md) | [Next: Memory Considerations ▶](06-memory-considerations.md) |
