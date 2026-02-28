# Words with Prefix

## Overview
Collecting all words that share a given prefix is a core trie strength. This chapter covers the algorithm, optimizations for large result sets, and variations including returning words sorted, limited (top-K), or filtered by additional criteria.

---

## Core Algorithm

```
  Phase 1: Navigate to the prefix node   → O(P)
  Phase 2: DFS to collect all words      → O(K)
  
  ┌─────────────────────────────────────────────────────────┐
  │  prefix = "ca"                                           │
  │                                                          │
  │  root ── c ── a ── t*                                    │
  │                ├── r*                                     │
  │                │   └── d*                                 │
  │                └── n*                                     │
  │                                                          │
  │  Navigate: root → c → a (prefix node)                    │
  │  DFS from 'a': "cat", "car", "card", "can"              │
  └─────────────────────────────────────────────────────────┘
```

### Implementation

```python
class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def find_all_with_prefix(self, prefix):
        """Return all words starting with prefix."""
        # Phase 1: Navigate to prefix node
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return []
            node = node.children[ch]
        
        # Phase 2: DFS to collect
        results = []
        self._collect(node, list(prefix), results)
        return results
    
    def _collect(self, node, path, results):
        if node.is_end:
            results.append(''.join(path))
        for ch in sorted(node.children):  # Sorted → lexicographic order
            path.append(ch)
            self._collect(node.children[ch], path, results)
            path.pop()
```

---

## Trace: Finding All Words with Prefix "ap"

```
  Trie contains: apple, application, ape, apex, app, banana
  
  ── Phase 1: Navigate to "ap" ──
  
  root → a → p   (prefix node found ✓)
  
  ── Phase 2: DFS from 'p' ──
  
  path = ['a', 'p']
  
  p: isEnd=False, children={e, p}
  
    → e: isEnd=True → result: "ape"
       children={x}
       → x: isEnd=True → result: "apex"
         (backtrack)
       (backtrack)
    
    → p: isEnd=True → result: "app"
       children={l}
       → l: isEnd=False
          children={e, i}
          → e: isEnd=True → result: "apple"
             (backtrack)
          → i: isEnd=False
             children={c}
             → c: isEnd=False
                ...→ → → n: isEnd=True → result: "application"
  
  Results (sorted): ["ape", "apex", "app", "apple", "application"]
```

---

## Variations

### 1. Iterative BFS Approach

```python
from collections import deque

def find_all_with_prefix_bfs(self, prefix):
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return []
        node = node.children[ch]
    
    results = []
    queue = deque([(node, list(prefix))])
    
    while queue:
        curr, path = queue.popleft()
        if curr.is_end:
            results.append(''.join(path))
        for ch in sorted(curr.children):
            queue.append((curr.children[ch], path + [ch]))
    
    return results
```

### 2. Top-K by Frequency

```python
import heapq

def find_top_k_with_prefix(self, prefix, k):
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return []
        node = node.children[ch]
    
    # Use min-heap of size k
    heap = []  # (frequency, word)
    self._collect_with_freq(node, list(prefix), heap, k)
    
    # Extract sorted results
    result = []
    while heap:
        freq, word = heapq.heappop(heap)
        result.append(word)
    result.reverse()
    return result

def _collect_with_freq(self, node, path, heap, k):
    if node.is_end:
        word = ''.join(path)
        if len(heap) < k:
            heapq.heappush(heap, (node.frequency, word))
        elif node.frequency > heap[0][0]:
            heapq.heapreplace(heap, (node.frequency, word))
    for ch in node.children:
        path.append(ch)
        self._collect_with_freq(node.children[ch], path, heap, k)
        path.pop()
```

### 3. With Maximum Results Limit

```python
def find_with_prefix_limited(self, prefix, max_results=10):
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return []
        node = node.children[ch]
    
    results = []
    self._collect_limited(node, list(prefix), results, max_results)
    return results

def _collect_limited(self, node, path, results, limit):
    if len(results) >= limit:
        return  # Early termination!
    if node.is_end:
        results.append(''.join(path))
    for ch in sorted(node.children):
        if len(results) >= limit:
            return
        path.append(ch)
        self._collect_limited(node.children[ch], path, results, limit)
        path.pop()
```

### 4. Using Stored Word Field

```python
def find_all_with_prefix_stored(self, prefix):
    """Uses node.word field for direct word retrieval."""
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return []
        node = node.children[ch]
    
    results = []
    self._collect_stored(node, results)
    return results

def _collect_stored(self, node, results):
    if node.word:
        results.append(node.word)  # No path building needed!
    for child in node.children.values():
        self._collect_stored(child, results)
```

---

## Complexity Analysis

| Variant | Time | Space |
|---------|:----:|:-----:|
| All words | O(P + K) | O(K) result + O(L) stack |
| Top-K by frequency | O(P + K × log K) | O(K) heap |
| Limited results | O(P + limit × L) | O(limit) |
| With stored word | O(P + matches) | O(matches) |

Where: P = prefix length, K = total chars in results, L = max word length

---

## Practical Applications

```
  1. Autocomplete dropdown: prefix → top suggestions
  2. Dictionary lookup: find all words matching prefix
  3. File system: find files under a path prefix
  4. IP routing: find all routes with IP prefix
  5. Contact search: find contacts by name prefix
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Algorithm | Navigate to prefix + DFS/BFS collect |
| Default order | Sorted children → lexicographic results |
| Top-K | Min-heap of size K during DFS |
| Early termination | Stop DFS once limit reached |
| Stored word | Avoids path reconstruction during DFS |
| Time | O(P + K) where K = result size |

---

## Quick Revision Questions

1. **What happens if the prefix doesn't exist in the trie?**
   > Phase 1 fails (a character not found in children), and we return an empty list immediately without any DFS.

2. **How do you get results in lexicographic order?**
   > Visit children in sorted order: `for ch in sorted(node.children)`. With array-based tries, children are naturally in alphabetical order.

3. **Why is early termination important for limited results?**
   > Without it, DFS visits ALL nodes in the subtree even if we only need 3 results. With it, we skip entire subtrees once the limit is reached.

4. **What's the advantage of storing the complete word at terminal nodes?**
   > Eliminates the need to maintain and build a path list during DFS. Just reference `node.word` directly when `is_end` is true.

5. **How would you find words with prefix "app" that are between 3 and 5 characters long?**
   > During DFS, only add to results when `len(path) >= 3 and len(path) <= 5` at `is_end` nodes. Skip branches where `len(path) > 5` for pruning.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Longest Common Prefix](04-longest-common-prefix.md) | [Next: Prefix Matching ▶](06-prefix-matching.md) |
