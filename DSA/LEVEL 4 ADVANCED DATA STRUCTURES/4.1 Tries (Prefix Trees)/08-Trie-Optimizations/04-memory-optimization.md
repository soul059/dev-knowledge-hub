# Memory Optimization Techniques for Tries

## Overview
Standard tries consume significant memory due to large child arrays and per-node overhead. This chapter covers practical techniques to reduce memory by **2-10× or more**, critical for production systems and competitive programming with tight memory limits.

---

## Technique 1: Array-Based Flat Trie

Replace pointer-based nodes with flat arrays. Eliminates per-node object overhead.

```python
class FlatTrie:
    """All nodes stored in contiguous arrays."""
    
    def __init__(self, max_nodes=1000000):
        self.children = [[0] * 26 for _ in range(max_nodes)]
        self.is_end = [False] * max_nodes
        self.size = 1  # Node 0 = root
    
    def _new_node(self):
        idx = self.size
        self.size += 1
        return idx
    
    def insert(self, word):
        node = 0
        for ch in word:
            c = ord(ch) - ord('a')
            if not self.children[node][c]:
                self.children[node][c] = self._new_node()
            node = self.children[node][c]
        self.is_end[node] = True
    
    def search(self, word):
        node = 0
        for ch in word:
            c = ord(ch) - ord('a')
            if not self.children[node][c]:
                return False
            node = self.children[node][c]
        return self.is_end[node]
```

```cpp
// C++ flat trie — fastest possible
const int MAXN = 1000005;
int ch[MAXN][26];
bool is_end[MAXN];
int sz = 1;

void insert(const string& s) {
    int u = 0;
    for (char c : s) {
        int x = c - 'a';
        if (!ch[u][x]) ch[u][x] = sz++;
        u = ch[u][x];
    }
    is_end[u] = true;
}
```

```
  Memory comparison per node:
  
  Python class-based:  ~280-350 bytes (dict + object overhead)
  Python flat array:   ~26 ints + 1 bool ≈ 232 bytes
  Java array-based:    ~130 bytes (int[26] + boolean)
  C++ flat array:      ~105 bytes (26 × 4 byte ints + 1 byte)
  C++ flat with short: ~53 bytes (26 × 2 byte shorts + 1 byte)
```

---

## Technique 2: HashMap Children (Sparse Nodes)

When most nodes have few children, use HashMap instead of array.

```python
class SparseTrie:
    def __init__(self):
        self.nodes = [{'children': {}, 'is_end': False}]
    
    def insert(self, word):
        node = 0
        for ch in word:
            if ch not in self.nodes[node]['children']:
                self.nodes.append({'children': {}, 'is_end': False})
                self.nodes[node]['children'][ch] = len(self.nodes) - 1
            node = self.nodes[node]['children'][ch]
        self.nodes[node]['is_end'] = True
```

```
  Average children per node in English dictionary: ~2-3
  
  Array: 26 slots × 4 bytes = 104 bytes (wasted: 88-92 bytes)
  HashMap: ~3 entries × 16 bytes = 48 bytes (plus overhead ≈ 80 bytes)
  
  Break-even: ~5 children per node
  Below 5: HashMap wins
  Above 5: Array wins
```

---

## Technique 3: Python `__slots__`

```python
class TrieNode:
    __slots__ = ['children', 'is_end']
    
    def __init__(self):
        self.children = {}
        self.is_end = False
```

```
  Without __slots__: each object has a __dict__ (~232 bytes)
  With __slots__: no __dict__, attributes stored directly (~72 bytes)
  
  Savings: ~160 bytes per node!
  For 100K nodes: 16 MB saved
```

---

## Technique 4: Bit Packing (Compact Children)

Store children as a bitmask + dense array.

```python
class CompactNode:
    __slots__ = ['bitmap', 'children', 'is_end']
    
    def __init__(self):
        self.bitmap = 0       # 26-bit mask: which children exist
        self.children = []    # Dense array of only existing children
        self.is_end = False
    
    def get_child(self, ch):
        bit = 1 << (ord(ch) - ord('a'))
        if not (self.bitmap & bit):
            return None
        # Count set bits below this position = index in dense array
        index = bin(self.bitmap & (bit - 1)).count('1')
        return self.children[index]
    
    def add_child(self, ch, child_node):
        bit = 1 << (ord(ch) - ord('a'))
        index = bin(self.bitmap & (bit - 1)).count('1')
        self.bitmap |= bit
        self.children.insert(index, child_node)
```

```
  Standard: 26 pointers × 8 bytes = 208 bytes per node
  Compact:  4 bytes (bitmap) + N × 8 bytes (actual children)
  
  If node has 3 children: 4 + 24 = 28 bytes (vs 208)
  Savings: 86% for sparse nodes!
  
  Trade-off: get_child is O(popcount) instead of O(1)
  But popcount is very fast (single CPU instruction).
```

---

## Technique 5: Double-Array Trie

Used extensively in CJK text processing (Chinese/Japanese/Korean).

```
  Two arrays: BASE[] and CHECK[]
  
  BASE[s] + c = t  (transition from state s with char c to state t)
  CHECK[t] = s     (verify that t was indeed reached from s)
  
  If CHECK[BASE[s] + c] == s, then transition exists.
  Otherwise, no child c from state s.
  
  Memory: 2 integers per state (8 bytes)
  vs. 26 pointers per node in standard trie (208 bytes)
  
  Extremely space-efficient but complex to build.
```

```python
class DoubleArrayTrie:
    def __init__(self, max_size=1000000):
        self.base = [0] * max_size
        self.check = [-1] * max_size
        self.is_end = [False] * max_size
    
    def search(self, word):
        state = 0
        for ch in word:
            c = ord(ch) - ord('a') + 1  # 1-indexed
            next_state = self.base[state] + c
            
            if next_state < 0 or next_state >= len(self.check):
                return False
            if self.check[next_state] != state:
                return False
            
            state = next_state
        
        return self.is_end[state]
    
    # Build is complex — requires conflict resolution
    # when BASE[s] + c collides with another state
```

---

## Technique 6: Memory Pool Allocation

Pre-allocate node memory to avoid per-allocation overhead.

```cpp
class TriePool {
    struct Node {
        int children[26] = {};
        bool is_end = false;
    };
    
    Node pool[500000];  // Pre-allocated pool
    int pool_size = 1;  // 0 = root
    
public:
    int newNode() {
        return pool_size++;
    }
    
    void insert(const string& word) {
        int node = 0;
        for (char ch : word) {
            int c = ch - 'a';
            if (!pool[node].children[c])
                pool[node].children[c] = newNode();
            node = pool[node].children[c];
        }
        pool[node].is_end = true;
    }
};
```

```
  Benefits:
  - No malloc/new overhead per node
  - Better cache locality (contiguous memory)
  - Faster allocation: just increment counter
  - Easy reset: pool_size = 1 (reuse all memory)
  
  Used extensively in competitive programming.
```

---

## Memory Comparison Summary

| Technique | Bytes/Node (26 children) | Relative |
|-----------|:---:|:---:|
| Python dict-based | ~350 | 100% |
| Python `__slots__` + dict | ~170 | 49% |
| Python flat array | ~232 | 66% |
| Java class-based | ~130 | 37% |
| Java flat array | ~108 | 31% |
| C++ class-based | ~112 | 32% |
| C++ flat/pool | ~105 | 30% |
| Bit-packed (3 children avg) | ~28 | 8% |
| Double-array | ~8 | 2% |

---

## Decision Guide

```
  ┌─ Is this competitive programming?
  │   ├── YES → Flat C++ arrays / memory pool
  │   └── NO ↓
  ├─ Is alphabet large (Unicode)?
  │   ├── YES → HashMap children
  │   └── NO ↓
  ├─ Are nodes mostly sparse (<5 children)?
  │   ├── YES → Bit-packed or HashMap
  │   └── NO ↓
  ├─ Is the trie built once, queried many times?
  │   ├── YES → Double-array trie
  │   └── NO → Standard array with __slots__ (Python) / arrays (Java/C++)
```

---

## Summary Table

| Technique | Space Savings | Speed Impact | Complexity |
|-----------|:---:|:---:|:---:|
| Flat arrays | 20-50% | Faster | Low |
| `__slots__` | 45-50% | Same | Trivial |
| HashMap children | 50-80%* | Slower | Low |
| Bit packing | 80-90% | Slightly slower | Medium |
| Memory pool | 20-30% | Faster | Low |
| Double-array | 95%+ | Fast lookups | High |

*For sparse nodes only.

---

## Quick Revision Questions

1. **Why are flat arrays faster than pointer-based nodes?**
   > Flat arrays store data contiguously in memory, improving cache locality. Pointer-based nodes scatter data across the heap, causing cache misses on every node access.

2. **When does HashMap outperform array for children storage?**
   > When the average number of children per node is below ~5. Above that, array's O(1) access and lower per-entry overhead make it more efficient.

3. **What does `__slots__` do in Python?**
   > It replaces the per-instance `__dict__` (a hash table) with a fixed-size tuple of attribute slots, eliminating ~160 bytes of overhead per object.

4. **How does bit packing find a child's index?**
   > Use a bitmask where bit i is set if child i exists. To find the dense array index for child c, count the number of set bits below position c using popcount: `index = popcount(bitmap & ((1 << c) - 1))`.

5. **Why is the double-array trie so space-efficient?**
   > It represents the entire trie with just two integer arrays. Each node uses 8 bytes total (2 ints) regardless of alphabet size, compared to 26 × pointer_size for standard arrays.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Patricia Trie](03-patricia-trie.md) | [Next: Concurrent Tries ▶](05-concurrent-tries.md) |
