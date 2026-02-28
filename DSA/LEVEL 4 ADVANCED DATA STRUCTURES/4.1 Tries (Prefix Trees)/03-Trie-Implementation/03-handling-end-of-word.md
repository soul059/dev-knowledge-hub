# Handling End of Word

## Overview
A trie node must distinguish between a node that's merely part of a path (intermediate) and one that marks the end of a stored word. This chapter explores different strategies for marking word endings and their implications for search, counting, and value storage.

---

## The Problem

```
  Words inserted: "app", "apple"
  
  root ── a ── p ── p ── l ── e
  
  Without end-of-word markers:
  
  search("app")   → reaches node 'p' → BUT is "app" stored?
  search("apple") → reaches node 'e' → BUT is "apple" stored?
  search("ap")    → reaches node 'p' → BUT is "ap" stored?
  
  ┌─────────────────────────────────────────────────────────┐
  │  ALL three queries reach valid nodes, but only "app"    │
  │  and "apple" were actually inserted.                    │
  │  Without markers, we can't tell the difference!         │
  └─────────────────────────────────────────────────────────┘
  
  With end-of-word markers:
  
  root ── a ── p ── p* ── l ── e*
                    ↑              ↑
                  isEnd=True    isEnd=True
  
  search("app")   → node 'p', isEnd=True  → Found!    ✓
  search("apple") → node 'e', isEnd=True  → Found!    ✓
  search("ap")    → node 'p', isEnd=False → Not found ✗
```

---

## Strategy 1: Boolean Flag (Most Common)

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False   # ← Simple boolean
```

### How It Works

```
  Insert "cat":
  root ── c ── a ── t
                    isEnd = True
  
  Insert "cats":
  root ── c ── a ── t ── s
                    ↑      ↑
               isEnd=T  isEnd=T
  
  search("cat")  → t.isEnd == True  → Found
  search("cats") → s.isEnd == True  → Found
  search("ca")   → a.isEnd == False → Not found (just a prefix)
```

### Pros and Cons

```
  ✓ Simplest approach — only 1 bit of storage
  ✓ Works for set-like operations (insert, search, delete)
  ✗ Cannot store values associated with words
  ✗ Cannot track duplicate insertions
```

---

## Strategy 2: Count Field

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.count = 0  # Number of times this word was inserted
```

### How It Works

```
  Insert "apple" twice:
  
  root ── a ── p ── p ── l ── e
                              count = 2
  
  Insert "app" once:
  
  root ── a ── p ── p ── l ── e
                    count = 1   count = 2
  
  search("apple") → count > 0 → Found (appeared 2 times)
  search("app")   → count > 0 → Found (appeared 1 time)
  search("ap")    → count == 0 → Not found
  
  delete("apple") → decrement count: e.count = 1
  delete("apple") → decrement count: e.count = 0 → word removed
```

### Implementation

```python
class Trie:
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.count += 1  # Increment instead of setting True
    
    def search(self, word):
        node = self._find_node(word)
        return node is not None and node.count > 0
    
    def get_count(self, word):
        """How many times was this word inserted?"""
        node = self._find_node(word)
        return node.count if node else 0
    
    def delete(self, word):
        node = self._find_node(word)
        if node and node.count > 0:
            node.count -= 1
            return True
        return False
```

### Use Cases

```
  ✓ Frequency counting (word frequency in text)
  ✓ Handling duplicate insertions gracefully
  ✓ Multiset semantics (same word stored multiple times)
  ✓ "Delete one occurrence" vs "delete all occurrences"
```

---

## Strategy 3: Value Storage

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.value = None  # None means no word ends here
```

### How It Works

```
  This turns the trie into a map/dictionary:
  
  insert("apple", 5)   →  key="apple", value=5
  insert("app", 10)    →  key="app",   value=10
  
  root ── a ── p ── p ── l ── e
                    ↑              ↑
               value=10        value=5
  
  get("apple") → 5
  get("app")   → 10
  get("ap")    → None (not a stored word)
  
  is_end equivalent: node.value is not None
```

### Implementation

```python
class TrieMap:
    """Trie as a key-value map."""
    
    def __init__(self):
        self.root = TrieNode()
    
    def put(self, key, value):
        node = self.root
        for ch in key:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.value = value
    
    def get(self, key):
        node = self._find_node(key)
        return node.value if node else None
    
    def contains(self, key):
        node = self._find_node(key)
        return node is not None and node.value is not None
    
    def _find_node(self, key):
        node = self.root
        for ch in key:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
```

### Use Cases

```
  ✓ Autocomplete with scores/priorities
  ✓ IP routing tables (prefix → next hop)
  ✓ File systems (path → file metadata)
  ✓ MapSum problems (word → integer value)
```

---

## Strategy 4: Prefix Count

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False
        self.prefix_count = 0  # Words passing through this node
```

### How It Works

```
  Insert "app", "apple", "ape":
  
  root ── a(3) ── p(3) ── p(2) ── l(1) ── e(1)*
                           ↑
                        isEnd=T
                           └── e(1)*
  
  Numbers = prefix_count (how many words pass through)
  
  count_words_with_prefix("ap")  → p.prefix_count = 3
  count_words_with_prefix("app") → p2.prefix_count = 2
  count_words_with_prefix("ape") → e.prefix_count = 1
```

### Implementation

```python
class Trie:
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
            node.prefix_count += 1  # Increment on every node along path
        node.is_end = True
    
    def count_prefix(self, prefix):
        """How many words have this prefix?"""
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.prefix_count
    
    def delete(self, word):
        """Must decrement prefix_count along the path."""
        node = self.root
        # First verify word exists
        if not self.search(word):
            return False
        for ch in word:
            node = node.children[ch]
            node.prefix_count -= 1
        node.is_end = False
        return True
```

---

## Comparison Table

| Strategy | Storage | is_end Check | Extra Capability |
|----------|:-------:|:------------:|:-----------------|
| Boolean `is_end` | 1 bit | `node.is_end` | None |
| Count | integer | `node.count > 0` | Frequency tracking |
| Value | any type | `node.value is not None` | Key-value mapping |
| Prefix Count | integer | `node.is_end` | O(1) prefix counting |

---

## Combining Strategies

In practice, you often need multiple fields:

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False          # Does a word end here?
        self.count = 0               # How many times inserted?
        self.prefix_count = 0        # Words with this prefix?
        self.value = None            # Associated value?
        self.word = None             # Store the complete word?
```

### Decision Flowchart

```
  Need to just check word existence?
  ├── Yes → Boolean is_end
  └── No  → Need to store values?
            ├── Yes → Value field
            └── No  → Need duplicate tracking?
                      ├── Yes → Count field  
                      └── No  → Need prefix counting?
                                ├── Yes → Prefix count
                                └── No  → Boolean is_end
```

---

## Common Interview Pattern: Storing the Word

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None  # Store complete word at terminal node

class Trie:
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.word = word  # Store the entire word
```

Why store the word? Avoids reconstructing it during DFS:

```python
# Without stored word — must build during traversal
def dfs(node, path, results):
    if node.is_end:
        results.append(''.join(path))  # Build from path
    for ch, child in node.children.items():
        path.append(ch)
        dfs(child, path, results)
        path.pop()

# With stored word — direct access
def dfs(node, results):
    if node.word:
        results.append(node.word)  # Already stored!
    for child in node.children.values():
        dfs(child, results)
```

This is especially useful for **Word Search II (LeetCode 212)**.

---

## Summary Table

| Key Concept | Detail |
|-------------|--------|
| Why needed | Distinguish intermediate nodes from word endpoints |
| Simplest approach | Boolean `is_end` flag |
| For counts | Integer `count` field |
| For key-value | Store `value` at terminal node |
| For prefix queries | `prefix_count` incremented on all path nodes |
| Interview tip | Storing `word` at terminal avoids path reconstruction |
| Combining | Use multiple fields as needed for the problem |

---

## Quick Revision Questions

1. **Why can't we just check if a node exists to determine if a word was inserted?**
   > Because intermediate nodes are created for all prefixes. "app" creates nodes for "a", "ap", "app" — but only "app" was inserted as a word. Without `isEnd`, we'd falsely match "a" and "ap".

2. **When should you use a count field instead of a boolean?**
   > When the same word can be inserted multiple times and you need to track frequency, or when delete should remove one occurrence rather than the entire word.

3. **What's the advantage of storing `prefix_count` on every node?**
   > It allows O(P) counting of how many words share a prefix — just navigate to the prefix node and read the count instead of doing a DFS to collect and count all matching words.

4. **Why might you store the complete word at a terminal node?**
   > To avoid reconstructing the word character-by-character during DFS. Particularly useful in problems like Word Search II where you collect found words during traversal.

5. **How does deletion differ between boolean and count strategies?**
   > Boolean: set `is_end = False`. Count: decrement `count -= 1`. Count-based deletion only removes the word completely when count reaches 0.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: HashMap-Based Children](02-hashmap-based-children.md) | [Next: Case Sensitivity ▶](04-case-sensitivity.md) |
