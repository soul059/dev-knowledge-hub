# Patricia Trie (Practical Algorithm to Retrieve Information Coded in Alphanumeric)

## Overview
A **Patricia trie** is a space-optimized variant of a radix tree (compressed trie) where every internal node has at least 2 children. This eliminates all one-way branching, achieving the most compact trie representation possible. It's widely used in **IP routing**, **networking**, and **database indexing**.

---

## Key Difference from Radix Tree

```
  Radix Tree: Compresses chains but can have internal nodes with 1 child
  Patricia Trie: EVERY internal node has ≥ 2 children
  
  In binary Patricia tries (PATRICIA was originally binary):
  - Each node stores a "skip" value (bit position to examine)
  - Edges are implicit — determined by the bit at skip position
  - Only leaves store actual keys
  - Internal nodes store bit-test positions
```

---

## Binary Patricia Trie

```
  Store: 1 (0001), 5 (0101), 7 (0111), 14 (1110), 15 (1111)
  
  Standard binary trie:
  
              root
             /    \
           [0]    [1]
           /        \
         [0]        [1]
         /   \      /   \
       [0]  [1]   [1]   [1]
       /    /  \    |     |
      1    5    7  14    15
  
  Patricia Trie (test bit positions):
  
          [bit 3]              ← test bit 3 (MSB)
         /       \
      [bit 1]   [bit 0]       ← test relevant bits, skip others
      /    \     /    \
    1(0001) [bit 0] 14(1110) 15(1111)
            /    \
         5(0101) 7(0111)
  
  Skips bit positions where all keys agree!
  Bit 3: separates {1,5,7} from {14,15}
  Bit 1: separates {1} from {5,7}
  Bit 0: separates {5} from {7}, and {14} from {15}
```

---

## String Patricia Trie Implementation

```python
class PatriciaNode:
    def __init__(self):
        self.children = {}   # first_char → (edge_label, child)
        self.is_end = False
        self.value = None    # Optional stored value

class PatriciaTrie:
    def __init__(self):
        self.root = PatriciaNode()
    
    def insert(self, key, value=None):
        if not key:
            self.root.is_end = True
            self.root.value = value
            return
        
        node = self.root
        i = 0
        
        while i < len(key):
            ch = key[i]
            
            if ch not in node.children:
                # Create new leaf
                leaf = PatriciaNode()
                leaf.is_end = True
                leaf.value = value
                node.children[ch] = (key[i:], leaf)
                return
            
            edge_label, child = node.children[ch]
            
            # Find longest common prefix
            j = 0
            while j < len(edge_label) and i + j < len(key) and edge_label[j] == key[i + j]:
                j += 1
            
            if j == len(edge_label):
                # Full edge matched
                i += j
                node = child
                continue
            
            # Split required
            split = PatriciaNode()
            node.children[ch] = (edge_label[:j], split)
            
            # Old child becomes child of split
            split.children[edge_label[j]] = (edge_label[j:], child)
            
            # New key remainder becomes another child
            if i + j < len(key):
                leaf = PatriciaNode()
                leaf.is_end = True
                leaf.value = value
                split.children[key[i + j]] = (key[i + j:], leaf)
            else:
                split.is_end = True
                split.value = value
            return
        
        # Exact match to existing node
        node.is_end = True
        node.value = value
    
    def search(self, key):
        node = self.root
        i = 0
        
        if not key:
            return node.is_end
        
        while i < len(key):
            ch = key[i]
            if ch not in node.children:
                return False
            
            edge_label, child = node.children[ch]
            
            # Verify edge matches
            if len(key) - i < len(edge_label):
                return False
            
            for j in range(len(edge_label)):
                if key[i + j] != edge_label[j]:
                    return False
            
            i += len(edge_label)
            node = child
        
        return node.is_end
    
    def delete(self, key):
        """Delete key and merge chains if possible."""
        self._delete(self.root, key, 0)
    
    def _delete(self, node, key, depth):
        if depth == len(key):
            if not node.is_end:
                return False
            node.is_end = False
            return len(node.children) == 0  # Can remove if leaf
        
        ch = key[depth]
        if ch not in node.children:
            return False
        
        edge_label, child = node.children[ch]
        
        if not key[depth:].startswith(edge_label):
            return False
        
        should_remove = self._delete(child, key, depth + len(edge_label))
        
        if should_remove:
            del node.children[ch]
            
            # Merge if node now has exactly 1 child and is not a word end
            if len(node.children) == 1 and not node.is_end and node != self.root:
                only_key = next(iter(node.children))
                only_label, only_child = node.children[only_key]
                # Merge would happen at the parent level
                pass
        elif len(child.children) == 1 and not child.is_end:
            # Child has 1 child and is not end → merge
            grandchild_key = next(iter(child.children))
            grandchild_label, grandchild = child.children[grandchild_key]
            merged_label = edge_label + grandchild_label
            node.children[ch] = (merged_label, grandchild)
        
        return False
```

---

## Patricia vs Radix vs Standard Trie

| Feature | Standard Trie | Radix Tree | Patricia Trie |
|---------|:---:|:---:|:---:|
| Edge labels | 1 char | Multi-char | Multi-char |
| Single-child nodes | Many | Possible | None |
| Internal node children | ≥ 1 | ≥ 1 | ≥ 2 |
| Node count for N words | O(N×L) | ≤ 2N-1 | ≤ 2N-1 |
| Merging on delete | No | Optional | Required |
| Implementation | Simple | Moderate | Complex |

---

## Use in IP Routing (Longest Prefix Match)

```
  IP routing tables store CIDR blocks:
  192.168.1.0/24, 192.168.0.0/16, 10.0.0.0/8
  
  Binary Patricia Trie for IP lookup:
  - Each IP = 32-bit string
  - CIDR prefix length determines depth
  - Lookup: walk bits, track last matching prefix
  
  Example routes:
  10.0.0.0/8     → Gateway A (bit prefix: 00001010)
  192.168.0.0/16 → Gateway B (prefix: 11000000 10101000)
  192.168.1.0/24 → Gateway C (adds: 00000001)
  
  Lookup 192.168.1.5:
  → Walk bits → Match /16 at depth 16, /24 at depth 24
  → Longest match: /24 → Gateway C
  
  Patricia trie skips bits where all routes agree,
  making lookups very fast.
```

---

## Advantages of Patricia Trie

```
  1. Minimal nodes: exactly N leaves + at most N-1 internal nodes
  2. No wasted single-child chains
  3. Fast traversal: skip irrelevant positions
  4. Worst case bounded: operations are O(L) regardless of N
  5. Supports longest-prefix-match natively
  6. Cache-friendly compared to standard deep tries
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|:----:|:-----:|
| Insert | O(L) | O(L) for edge label |
| Search | O(L) | O(1) |
| Delete | O(L) | O(1) |
| Total space | — | O(N) nodes + O(total unique prefix chars) |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Origin | 1968, Donald R. Morrison |
| Key property | Every internal node has ≥ 2 children |
| Binary variant | Tests specific bit positions, skips common bits |
| String variant | Same as radix tree with mandatory chain merging |
| Use cases | IP routing, database indexing, network tables |
| Delete | Must merge chains after deletion |

---

## Quick Revision Questions

1. **What distinguishes a Patricia trie from a radix tree?**
   > In a Patricia trie, every internal node must have at least 2 children. After deletions, single-child internal nodes are merged with their child. A radix tree may allow single-child internal nodes.

2. **How does the binary Patricia trie skip bits?**
   > Each internal node stores a "skip" value indicating which bit position to test. This allows the trie to skip bit positions where all keys in that subtree agree, jumping directly to the discriminating bit.

3. **Why is Patricia trie ideal for IP routing?**
   > IP addresses are fixed-length bit strings where routes share long common prefixes. The Patricia trie compresses these shared prefixes, and longest-prefix-match is just a walk through the trie tracking the last matching prefix.

4. **What happens during deletion in a Patricia trie?**
   > After removing a word, if an internal node is left with only one child, it must be merged with that child (concatenating edge labels) to maintain the invariant that all internal nodes have ≥ 2 children.

5. **What is the maximum number of nodes in a Patricia trie with N keys?**
   > At most 2N - 1 nodes: N leaves and at most N - 1 internal nodes. This follows from the binary tree property where every internal node has ≥ 2 children.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Ternary Search Tree](02-ternary-search-tree.md) | [Next: Memory Optimization ▶](04-memory-optimization.md) |
