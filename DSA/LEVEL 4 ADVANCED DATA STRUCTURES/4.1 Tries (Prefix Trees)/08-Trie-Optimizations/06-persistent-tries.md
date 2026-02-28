# Persistent Tries

## Overview
A **persistent trie** preserves all previous versions of the data structure after modifications. Each insert or delete creates a new version while sharing most of the structure with previous versions via **path copying**. This is essential for problems requiring **historical queries** or **rollback**.

---

## What Is Persistence?

```
  Ephemeral (normal): Modifications destroy the old version
  
  Persistent: Every modification creates a NEW version
              Old versions remain accessible
  
  Types:
  - Partially persistent: Can query any old version, modify only latest
  - Fully persistent: Can query AND modify any version
  
  Trie persistence is typically partial (most common use case).
```

---

## Path Copying Technique

```
  The key insight: To modify a node, copy only the nodes
  on the path from root to that node. All other nodes are SHARED.
  
  Version 0: Insert "cat"
  
  root₀ → c₀ → a₀ → t₀*
  
  Version 1: Insert "car" (creates new path, shares common prefix)
  
  root₁ → c₁ → a₁ → t₀*     (t₀ shared!)
                   └→ r₁*     (new node)
  
  root₀ still points to old structure:
  root₀ → c₀ → a₀ → t₀*
  
  root₁ shares 't₀' but has new root₁, c₁, a₁ (copies on the path).
```

```
  Visual of shared structure:
  
  root₀ ─→ c₀ ─→ a₀ ─→ t₀*
  
  root₁ ─→ c₁ ─→ a₁ ─→ t₀*  (shared!)
                      └──→ r₁*
  
  Memory: Only 3 new nodes (root₁, c₁, a₁, r₁) instead of copying all 4.
  For deep tries, most nodes are shared.
```

---

## Implementation

```python
class PersistentTrieNode:
    __slots__ = ['children', 'is_end', 'count']
    
    def __init__(self):
        self.children = {}
        self.is_end = False
        self.count = 0  # Numbers passing through
    
    def copy(self):
        """Create a shallow copy (children dict is new, but child nodes shared)."""
        new_node = PersistentTrieNode()
        new_node.children = dict(self.children)  # Shallow copy
        new_node.is_end = self.is_end
        new_node.count = self.count
        return new_node

class PersistentTrie:
    def __init__(self):
        self.roots = [PersistentTrieNode()]  # Version 0: empty
    
    def insert(self, word, version=-1):
        """Insert word, creating a new version based on given version."""
        if version == -1:
            version = len(self.roots) - 1
        
        old_root = self.roots[version]
        new_root = self._insert(old_root, word, 0)
        self.roots.append(new_root)
        return len(self.roots) - 1  # New version number
    
    def _insert(self, node, word, index):
        new_node = node.copy()  # Path copy
        new_node.count += 1
        
        if index == len(word):
            new_node.is_end = True
            return new_node
        
        ch = word[index]
        child = new_node.children.get(ch, PersistentTrieNode())
        new_node.children[ch] = self._insert(child, word, index + 1)
        
        return new_node
    
    def search(self, word, version=-1):
        """Search in a specific version."""
        if version == -1:
            version = len(self.roots) - 1
        
        node = self.roots[version]
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end
    
    def count_with_prefix(self, prefix, version=-1):
        """Count words with given prefix in a specific version."""
        if version == -1:
            version = len(self.roots) - 1
        
        node = self.roots[version]
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.count
```

---

## Trace

```
  pt = PersistentTrie()
  
  Version 0: empty trie
  roots = [root₀(empty)]
  
  pt.insert("cat")  → Version 1
  root₁ → c₁(cnt=1) → a₁(cnt=1) → t₁*(cnt=1)
  roots = [root₀, root₁]
  
  pt.insert("car")  → Version 2
  root₂ → c₂(cnt=2) → a₂(cnt=2) → t₁*(cnt=1)  ← SHARED
                                   → r₂*(cnt=1)
  roots = [root₀, root₁, root₂]
  
  pt.insert("dog")  → Version 3
  root₃ → c₂(cnt=2)  ← SHARED (entire 'c' subtree)
        → d₃(cnt=1) → o₃(cnt=1) → g₃*(cnt=1)
  roots = [root₀, root₁, root₂, root₃]
  
  Queries:
  pt.search("car", version=1)  → False (only "cat" in v1)
  pt.search("car", version=2)  → True  ("car" added in v2)
  pt.search("dog", version=2)  → False ("dog" added in v3)
  pt.search("dog", version=3)  → True
  
  pt.count_with_prefix("ca", version=1) → 1 (only "cat")
  pt.count_with_prefix("ca", version=2) → 2 ("cat" + "car")
```

---

## Application: Persistent Binary Trie for XOR Queries

Range XOR maximum: "Find max XOR of x with any `nums[L..R]`."

```python
class PersistentBitTrie:
    def __init__(self, max_bits=30):
        self.max_bits = max_bits
        # Flat array approach for performance
        self.ch = [[0, 0]]  # ch[node][bit]
        self.cnt = [0]       # Count at each node
        self.roots = [0]     # Version roots
        self.size = 1
    
    def _new_node(self, src=0):
        """Create new node, optionally copying from src."""
        self.ch.append(list(self.ch[src]))
        self.cnt.append(self.cnt[src])
        idx = self.size
        self.size += 1
        return idx
    
    def insert(self, num, prev_version=-1):
        if prev_version == -1:
            prev_version = self.roots[-1]
        
        # Path copy from root
        new_root = self._new_node(prev_version)
        node = new_root
        old = prev_version
        
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            
            # Copy both children from old version
            new_child = self._new_node(self.ch[old][bit] if self.ch[old][bit] else 0)
            
            # The other child remains shared
            other = 1 - bit
            self.ch[node][other] = self.ch[old][other]
            self.ch[node][bit] = new_child
            
            self.cnt[new_child] = self.cnt[self.ch[old][bit]] + 1 if self.ch[old][bit] else 1
            
            old = self.ch[old][bit] if self.ch[old][bit] else 0
            node = new_child
        
        self.roots.append(new_root)
    
    def max_xor(self, num, l_version, r_version):
        """Max XOR of num with any number in versions (l_version, r_version]."""
        l_node = self.roots[l_version]
        r_node = self.roots[r_version]
        result = 0
        
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            toggled = 1 - bit
            
            # Count of numbers in range going through toggled branch
            r_cnt = self.cnt[self.ch[r_node][toggled]] if self.ch[r_node][toggled] else 0
            l_cnt = self.cnt[self.ch[l_node][toggled]] if self.ch[l_node][toggled] else 0
            
            if r_cnt - l_cnt > 0:
                result |= (1 << i)
                r_node = self.ch[r_node][toggled]
                l_node = self.ch[l_node][toggled]
            else:
                r_node = self.ch[r_node][bit]
                l_node = self.ch[l_node][bit]
        
        return result
```

```
  How it works:
  
  1. Insert nums[0], nums[1], ..., nums[N-1] one by one
  2. roots[i] = trie state after inserting first i elements
  3. Query(x, L, R): 
     cnt_in_range = cnt[version_R] - cnt[version_L]
     Use count difference to check if a path exists in range [L,R]
  
  This gives O(B) per query for range XOR maximum!
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|:----:|:-----:|
| Insert (path copy) | O(L) | O(L) new nodes per version |
| Search | O(L) | O(1) |
| Total for N inserts | O(N × L) | O(N × L) total nodes |
| Binary trie insert | O(B) | O(B) new nodes per version |
| All N versions | O(N × B) | O(N × B) total |

Path copying creates at most L (or B) new nodes per version — all other nodes are shared.

---

## When to Use Persistent Tries

```
  ✓ Need to query historical states of the trie
  ✓ Range XOR queries (nums[L..R])
  ✓ Undo/redo functionality
  ✓ Functional programming environments
  ✓ Multi-version databases
  ✓ Git-like version control for data
  
  ✗ Simple insert-and-query (no history needed)
  ✗ Memory is extremely tight (path copying has overhead)
  ✗ Need mutable operations on old versions frequently
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Persistence | Preserve all versions after modification |
| Path copying | Copy only root-to-modified-node path |
| Shared structure | Unmodified subtrees shared across versions |
| Space per version | O(L) or O(B) new nodes |
| Key application | Range XOR queries, historical lookups |
| Version access | Store root pointer per version |

---

## Quick Revision Questions

1. **What is path copying and why is it efficient?**
   > Path copying duplicates only the nodes from root to the modified leaf — O(L) nodes. All other nodes are shared between versions. This is efficient because most of the trie is unchanged per operation.

2. **How do you query a specific version?**
   > Store the root node of each version in an array. To query version k, start from `roots[k]` and traverse normally. The shared structure doesn't affect correctness because old nodes are never modified.

3. **How does 'count difference' enable range queries?**
   > Each node stores the count of elements passing through it. For range [L, R], the count in that range is `cnt[version_R] - cnt[version_L]`. If this is > 0, that branch contains elements in the range.

4. **What's the total space for N versions of a binary trie?**
   > O(N × B) where B = bits per number. Each insert creates B+1 new nodes (path copy). With N inserts, total nodes ≤ initial + N×(B+1).

5. **Can persistent tries support deletion?**
   > Yes — create a new version where the word is marked as deleted (decrement counts, unset is_end). The old version remains unchanged with the word still present.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Concurrent Tries](05-concurrent-tries.md) | [Back to README ▶](../README.md) |
