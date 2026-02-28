# Compressed Trie (Radix Tree)

## Overview
A **compressed trie** (also called a **radix tree** or **Patricia trie variant**) merges chains of single-child nodes into one node with a multi-character label. This dramatically reduces node count and memory usage while maintaining the same time complexity.

---

## Problem with Standard Tries

```
  Standard Trie for ["apple", "application", "apply"]:
  
  root → a → p → p → l → e*          (apple)
                            → i → c → a → t → i → o → n*  (application)
                            → y*      (apply)
  
  13 internal nodes, many with only 1 child.
  
  Compressed Trie:
  
  root → "appl" → "e"*
                 → "ication"*
                 → "y"*
  
  Only 4 nodes! Chains merged into edge labels.
```

---

## Structure

```
  Standard Trie Node:
    - children: dict of single chars → nodes
    - is_end: bool
  
  Compressed Trie Node:
    - children: dict of first-char → (edge_label, child_node)
    - is_end: bool
  
  Each EDGE has a string label (not just 1 character).
  Branching only occurs where words actually diverge.
```

```
  Visual comparison for ["bear", "bell", "bid", "bull", "sell", "stock", "stop"]:
  
  Standard Trie:                    Compressed Trie:
  
  root                              root
  ├─ b                              ├─ "b" ── "e" ── "ar"*
  │  ├─ e                           │         └── "ll"*
  │  │  ├─ a                        │  └── "id"*
  │  │  │  └─ r*                    │  └── "ull"*
  │  │  └─ l                        ├─ "s" ── "ell"*
  │  │     └─ l*                    │         └── "to" ── "ck"*
  │  ├─ i                                            └── "p"*
  │  │  └─ d*
  │  └─ u
  │     └─ l
  │        └─ l*
  ├─ s
  │  ├─ e
  │  │  └─ l
  │  │     └─ l*
  │  └─ t
  │     └─ o
  │        ├─ c
  │        │  └─ k*
  │        └─ p*
  
  Standard: 18 nodes                Compressed: 10 nodes
```

---

## Implementation

```python
class CompressedTrieNode:
    def __init__(self):
        self.children = {}  # first_char → (edge_label, child_node)
        self.is_end = False

class CompressedTrie:
    def __init__(self):
        self.root = CompressedTrieNode()
    
    def insert(self, word):
        node = self.root
        i = 0
        
        while i < len(word):
            ch = word[i]
            
            if ch not in node.children:
                # No matching edge — create new edge with remaining word
                new_node = CompressedTrieNode()
                new_node.is_end = True
                node.children[ch] = (word[i:], new_node)
                return
            
            edge_label, child = node.children[ch]
            
            # Find common prefix between word[i:] and edge_label
            j = 0
            while j < len(edge_label) and i + j < len(word) and edge_label[j] == word[i + j]:
                j += 1
            
            if j == len(edge_label):
                # Entire edge consumed — continue into child
                i += j
                node = child
            else:
                # Partial match — need to split the edge
                # Create split node
                split_node = CompressedTrieNode()
                
                # Update parent: edge to split_node has common prefix
                node.children[ch] = (edge_label[:j], split_node)
                
                # Remaining of original edge goes from split to old child
                split_node.children[edge_label[j]] = (edge_label[j:], child)
                
                if i + j == len(word):
                    # Word ends exactly at split point
                    split_node.is_end = True
                else:
                    # Remaining of new word goes from split to new node
                    new_node = CompressedTrieNode()
                    new_node.is_end = True
                    split_node.children[word[i + j]] = (word[i + j:], new_node)
                
                return
        
        # Word consumed exactly at current node
        node.is_end = True
    
    def search(self, word):
        node = self.root
        i = 0
        
        while i < len(word):
            ch = word[i]
            if ch not in node.children:
                return False
            
            edge_label, child = node.children[ch]
            
            # Check if word matches edge label
            for j in range(len(edge_label)):
                if i + j >= len(word) or word[i + j] != edge_label[j]:
                    return False
            
            i += len(edge_label)
            node = child
        
        return node.is_end
    
    def starts_with(self, prefix):
        node = self.root
        i = 0
        
        while i < len(prefix):
            ch = prefix[i]
            if ch not in node.children:
                return False
            
            edge_label, child = node.children[ch]
            
            for j in range(len(edge_label)):
                if i + j >= len(prefix):
                    return True  # Prefix ends mid-edge → valid
                if prefix[i + j] != edge_label[j]:
                    return False
            
            i += len(edge_label)
            node = child
        
        return True
```

---

## Trace: Insert and Split

```
  Insert "apple":
  root has no 'a' → create edge "apple"
  
  root ──"apple"──→ (end)*
  
  Insert "application":
  root has 'a' → edge "apple"
  Compare "application" with "apple":
    a=a, p=p, p=p, l=l, e≠i → mismatch at j=4
  
  Split at position 4:
  - Common prefix: "appl"
  - Remaining old: "e" (from "apple")
  - Remaining new: "ication" (from "application")
  
  root ──"appl"──→ split_node
                    ├──"e"──→ (end)*        [apple]
                    └──"ication"──→ (end)*  [application]
  
  Insert "apply":
  root has 'a' → edge "appl" → split_node
  Compare "apply"[4:] = "y" with children of split_node:
    split has 'e' and 'i', not 'y' → create new edge
  
  root ──"appl"──→ split_node
                    ├──"e"──→ (end)*
                    ├──"ication"──→ (end)*
                    └──"y"──→ (end)*
```

---

## Comparison: Standard vs Compressed Trie

| Feature | Standard | Compressed |
|---------|:--------:|:----------:|
| Nodes per word | O(L) | O(1) amortized |
| Total nodes | O(total chars) | O(number of words) |
| Edge label | Single char | String |
| Single-child chains | Yes | Eliminated |
| Insert complexity | O(L) | O(L) |
| Search complexity | O(L) | O(L) |
| Memory | High | Much lower |
| Implementation | Simple | Complex (splitting) |

---

## When to Use

```
  USE Compressed Trie when:
  ✓ Words share long common prefixes
  ✓ Memory is a concern
  ✓ Sparse trie (few branches, long chains)
  ✓ IP routing tables (CIDR blocks)
  ✓ File system paths
  ✓ URL routing (web frameworks)
  
  PREFER Standard Trie when:
  ✓ Implementation simplicity matters
  ✓ Frequent insertions/deletions
  ✓ Words are short
  ✓ High branching factor (many children per node)
```

---

## Real-World Applications

```
  1. IP Routing: Longest prefix match on binary IP addresses
     Route table with CIDR blocks → compressed binary trie
  
  2. Web Frameworks: URL path routing
     "/api/users/:id", "/api/users/:id/posts"
     → compressed to "/api/users/:id" node with "/posts" child
  
  3. File Systems: Path indexing
     "/home/user/documents/file1.txt"
     → compress common path prefixes
  
  4. Databases: String index compression
     Column values sharing prefixes → radix tree index
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| AKA | Radix tree, Patricia tree (partial) |
| Core idea | Merge single-child chains into edge labels |
| Node count | O(N) where N = number of words (vs O(total chars)) |
| Trade-off | Less memory, same time, more complex code |
| Split operation | When new word diverges mid-edge |
| Key applications | IP routing, URL routing, file systems |

---

## Quick Revision Questions

1. **What triggers a node split in a compressed trie?**
   > When a new word shares a partial prefix with an existing edge label. The edge is split at the divergence point: common prefix becomes a new edge, and two children branch for the differing suffixes.

2. **How does the node count compare to a standard trie?**
   > A compressed trie has at most 2N-1 nodes for N words (each insertion adds at most 2 nodes: a split node + a leaf). A standard trie can have up to N×L nodes (L = word length).

3. **Does compression change the time complexity of operations?**
   > No. Insert, search, and prefix search all remain O(L) where L is the word length. The comparison just happens against edge labels instead of single characters.

4. **How does `starts_with` handle a prefix ending mid-edge?**
   > If the prefix is consumed before the edge label is fully traversed, we return `true` — the prefix matches a portion of the edge, confirming at least one word starts with it.

5. **Why are compressed tries used in IP routing?**
   > IP addresses are long bit strings with shared prefixes (network addresses). Compressed tries reduce the routing table to one node per unique prefix boundary, enabling fast longest prefix match in O(address bits).

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Map Sum Pairs](../07-Advanced-Trie-Problems/06-map-sum-pairs.md) | [Next: Ternary Search Tree ▶](02-ternary-search-tree.md) |
