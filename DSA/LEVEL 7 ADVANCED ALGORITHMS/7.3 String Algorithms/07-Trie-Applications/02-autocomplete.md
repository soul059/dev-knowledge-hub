# Chapter 7.2 — Autocomplete with Tries

> **Unit 7: Trie Applications** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Autocomplete — suggesting completions as a user types — is one of the most
practical trie applications. The trie's prefix-sharing structure makes it
ideal for finding all words matching a given prefix.

---

## 1. How Autocomplete Works

```
  User types: "app"
  
  System finds all words starting with "app":
    → "app", "apple", "application", "append", "approve"

  ┌──────────────────────────────────────────────────┐
  │  Step 1: Navigate to the prefix node (O(p))     │
  │  Step 2: DFS/BFS to collect all words below     │
  │  Step 3: Return top-k results (optionally ranked)│
  └──────────────────────────────────────────────────┘
```

---

## 2. Visual Example

```
  Trie with: "app", "apple", "apply", "ape", "apex", "bat"

              root
             /    \
            a      b
            |      |
            p      a
           / \     |
          p   e    t*
         /|    \
        l* *    x*
       / \
      e*  y*

  Query: prefix = "ap"
  Navigate: root → a → p  (found prefix node)
  DFS from p: → p (end: "app")
                 → l → e (end: "apple")
                 → l → y (end: "apply")
               → e (end: "ape")
                 → x (end: "apex")

  Results: ["app", "apple", "apply", "ape", "apex"]
```

---

## 3. Implementation

```python
class AutocompleteTrie:
    def __init__(self):
        self.root = {}
    
    def insert(self, word, weight=1):
        """Insert word with optional weight for ranking."""
        node = self.root
        for c in word:
            if c not in node:
                node[c] = {}
            node = node[c]
        node['#'] = weight  # '#' marks end, stores weight
    
    def _find_prefix_node(self, prefix):
        """Navigate to the node at end of prefix."""
        node = self.root
        for c in prefix:
            if c not in node:
                return None
            node = node[c]
        return node
    
    def autocomplete(self, prefix, limit=10):
        """Return up to `limit` words with given prefix."""
        node = self._find_prefix_node(prefix)
        if node is None:
            return []
        
        results = []
        self._dfs(node, prefix, results, limit)
        return results
    
    def _dfs(self, node, current, results, limit):
        """DFS to collect all words from this node."""
        if len(results) >= limit:
            return
        if '#' in node:
            results.append((current, node['#']))  # (word, weight)
        for c in sorted(node):  # alphabetical order
            if c != '#':
                self._dfs(node[c], current + c, results, limit)


# Example
trie = AutocompleteTrie()
for word in ["app", "apple", "apply", "ape", "apex", "banana", "band"]:
    trie.insert(word)

print(trie.autocomplete("ap"))
# [('ape', 1), ('apex', 1), ('app', 1), ('apple', 1), ('apply', 1)]

print(trie.autocomplete("ban"))
# [('banana', 1), ('band', 1)]
```

---

## 4. Ranked Autocomplete

```
  Real autocomplete ranks results by:
  1. Frequency (how often the word is searched)
  2. Recency (most recent queries first)
  3. Relevance (context-dependent scoring)

  Approach: Store frequency/weight at each end node.
  Return results sorted by weight (descending).

  To get top-k efficiently:
  - Use a MIN-HEAP of size k during DFS
  - Or store top-k words at each node (precomputed)
```

```python
import heapq

def top_k_autocomplete(trie, prefix, k=5):
    """Return top-k most frequent completions."""
    node = trie._find_prefix_node(prefix)
    if not node:
        return []
    
    # Collect all completions with weights
    all_words = []
    _collect(node, prefix, all_words)
    
    # Return top-k by weight (descending)
    return heapq.nlargest(k, all_words, key=lambda x: x[1])

def _collect(node, current, results):
    if '#' in node:
        results.append((current, node['#']))
    for c in node:
        if c != '#':
            _collect(node[c], current + c, results)
```

---

## 5. Optimizations

```
  ┌─────────────────────────────────────────────────────┐
  │  1. COMPRESSED TRIE (Patricia Trie)                │
  │     Merge single-child chains into one node.        │
  │     "apple" → single node instead of a-p-p-l-e     │
  │     Reduces node count and memory.                  │
  │                                                     │
  │  2. PRECOMPUTED TOP-K                              │
  │     At each node, store the top-k words reachable.  │
  │     Query: O(prefix_length) to get results!         │
  │     Trade-off: Uses more memory per node.           │
  │                                                     │
  │  3. TERNARY SEARCH TREE                            │
  │     3 pointers per node: left, equal, right         │
  │     Less memory than array-based trie               │
  │     Still O(m) for lookup                           │
  └─────────────────────────────────────────────────────┘
```

---

## 6. Complexity

```
  ┌────────────────────────┬───────────────────────────────┐
  │ Operation              │ Time                          │
  ├────────────────────────┼───────────────────────────────┤
  │ Insert word            │ O(m)                          │
  │ Navigate to prefix     │ O(p)                          │
  │ Collect all completions│ O(number of completions × m)  │
  │ Top-k (with heap)     │ O(results × log k)            │
  │ Top-k (precomputed)   │ O(p) ← optimal!              │
  └────────────────────────┴───────────────────────────────┘
```

---

## 📝 Summary Table

| Aspect | Details |
|--------|---------|
| Core idea | Navigate to prefix node, DFS to find completions |
| Navigate cost | O(prefix length) |
| Collection cost | O(total characters in results) |
| Ranking | By frequency, recency, or relevance |
| Optimization | Precompute top-k at each node |
| Real-world use | Search engines, IDEs, phone keyboards |

---

## ❓ Quick Revision Questions

1. **What are the two phases of trie-based autocomplete?**
   <details><summary>Answer</summary>Phase 1: Navigate from root to the prefix node in O(p). Phase 2: DFS/BFS from that node to collect all words that start with the prefix.</details>

2. **How can you return results in alphabetical order?**
   <details><summary>Answer</summary>During DFS, iterate over children in sorted (alphabetical) order. This naturally produces results in lexicographic order.</details>

3. **How do you implement ranked autocomplete?**
   <details><summary>Answer</summary>Store frequency/weight at end-of-word nodes. After collecting all completions, sort by weight descending, or use a heap to extract the top-k.</details>

4. **What is the advantage of precomputing top-k at each node?**
   <details><summary>Answer</summary>Query time drops to O(prefix_length) since the top-k results are already stored at the prefix node. No DFS needed. Trade-off is higher memory usage.</details>

5. **What is a compressed trie and why does it help?**
   <details><summary>Answer</summary>A compressed (Patricia) trie merges chains of single-child nodes into one. This reduces the total number of nodes and memory, especially for sparse dictionaries.</details>

---

| [⬅️ Previous: Trie for Pattern Matching](01-trie-for-pattern-matching.md) | [Next: Spell Checker Concept ➡️](03-spell-checker-concept.md) |
|:---|---:|
