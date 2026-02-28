# Memory Considerations

## Overview
Memory is the primary cost of using tries. This chapter covers practical memory calculation, optimization techniques, language-specific overhead, and strategies to decide when the memory trade-off is worthwhile.

---

## Memory Breakdown Per Node

### Python

```python
# Python TrieNode with dict children
class TrieNode:
    def __init__(self):
        self.children = {}      # ~240 bytes (empty dict)
        self.is_end = False     # ~28 bytes (Python bool is an object!)

# Total per node: ~280-350 bytes (due to object overhead)
# Python has VERY high per-object overhead!
```

### Java

```java
// Java TrieNode with array children
class TrieNode {
    TrieNode[] children = new TrieNode[26];  // 26 × 4 = 104 bytes (32-bit refs)
    boolean isEnd;                           // 1 byte + padding
    // Object header: 12-16 bytes
    // Total per node: ~130-140 bytes
}

// Java TrieNode with HashMap
class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();  // ~128 bytes empty
    boolean isEnd;
    // Total per node: ~160-180 bytes
}
```

### C++

```cpp
// C++ TrieNode with array — most memory-efficient
struct TrieNode {
    TrieNode* children[26];  // 26 × 8 = 208 bytes (64-bit pointers)
    bool isEnd;              // 1 byte + 7 padding
    // Total per node: 216 bytes
};

// C++ TrieNode with unordered_map
struct TrieNode {
    unordered_map<char, TrieNode*> children;  // ~56 bytes empty
    bool isEnd;
    // Total per node: ~64 bytes empty + 40 per entry
};
```

---

## Total Memory Estimation

```
  ┌──────────────────────────────────────────────────────────┐
  │  Formula:                                                 │
  │                                                           │
  │  Total Memory = Num_Nodes × Bytes_Per_Node                │
  │                                                           │
  │  Num_Nodes ≤ 1 + Σ(word_length_i) for all words          │
  │  Num_Nodes ≥ 1 + max(word_length_i)                      │
  │                                                           │
  │  Actual depends on prefix sharing                         │
  └──────────────────────────────────────────────────────────┘
```

### Real-World Examples

| Dataset | Words | Avg Length | Est. Nodes | Array (C++) | HashMap (Python) |
|---------|:-----:|:----------:|:----------:|:-----------:|:----------------:|
| Small word list | 100 | 6 | ~500 | 108 KB | 175 KB |
| LeetCode typical | 10,000 | 8 | ~50,000 | 10.8 MB | 17.5 MB |
| English dictionary | 100,000 | 8 | ~400,000 | 86.4 MB | 140 MB |
| Large corpus | 1,000,000 | 10 | ~3,000,000 | 648 MB | 1.05 GB |

---

## Memory Optimization Techniques

### 1. Object Pool / Array-Based Allocation

Instead of `new TrieNode()` for each node, pre-allocate a flat array:

```python
class FlatTrie:
    """Pool-based allocation — avoids per-object overhead."""
    
    def __init__(self, max_nodes=100000):
        self.children = [[0] * 26 for _ in range(max_nodes)]
        self.is_end = [False] * max_nodes
        self.next_id = 1  # 0 = root
    
    def _new_node(self):
        node_id = self.next_id
        self.next_id += 1
        return node_id
    
    def insert(self, word):
        node = 0  # Start at root (index 0)
        for ch in word:
            idx = ord(ch) - ord('a')
            if self.children[node][idx] == 0:
                self.children[node][idx] = self._new_node()
            node = self.children[node][idx]
        self.is_end[node] = True
    
    def search(self, word):
        node = 0
        for ch in word:
            idx = ord(ch) - ord('a')
            if self.children[node][idx] == 0:
                return False
            node = self.children[node][idx]
        return self.is_end[node]
```

```
  Benefits:
  ✓ No object headers (saves ~16-28 bytes per node)
  ✓ Better cache locality (contiguous memory)
  ✓ Faster allocation (increment counter vs malloc)
  ✓ Common in competitive programming
  
  Drawbacks:
  ✗ Must estimate max nodes upfront
  ✗ Wastes memory if overestimated
  ✗ No automatic cleanup
```

### 2. Bit Packing

Pack the `is_end` and children info into fewer bits:

```cpp
// Instead of bool isEnd per node, use a bitset
bitset<MAX_NODES> isEnd;

// Instead of TrieNode* children[26], use short indices
short children[MAX_NODES][26];  // 2 bytes vs 8 bytes per pointer
// Supports up to 32,767 nodes — enough for most problems
```

### 3. Lazy Node Creation

Only create child arrays when accessed:

```python
class LazyTrieNode:
    __slots__ = ['children', 'is_end']  # ← Saves ~100 bytes/node in Python
    
    def __init__(self):
        self.children = None  # Created on first child insertion
        self.is_end = False
    
    def get_child(self, ch):
        if self.children is None:
            return None
        return self.children.get(ch)
    
    def set_child(self, ch, node):
        if self.children is None:
            self.children = {}
        self.children[ch] = node
```

### 4. Python `__slots__` Optimization

```python
# Without __slots__ — each node has a __dict__
class TrieNode:
    def __init__(self):               # ~340 bytes per node
        self.children = {}
        self.is_end = False

# With __slots__ — no __dict__, fixed attributes
class TrieNode:
    __slots__ = ['children', 'is_end']  # ~200 bytes per node
    def __init__(self):
        self.children = {}
        self.is_end = False

# Savings: ~140 bytes per node = 40% reduction!
```

---

## Memory vs Speed Trade-offs

```
  ┌──────────────────────────────────────────────────────────┐
  │                    Speed                                  │
  │        ▲                                                  │
  │        │  Array (26)                                      │
  │        │    ●                                             │
  │        │                                                  │
  │        │        HashMap                                   │
  │        │          ●                                       │
  │        │                                                  │
  │        │              Sorted List                         │
  │        │                ●                                 │
  │        │                                                  │
  │        │                    Compressed Trie               │
  │        │                      ●                           │
  │        │                                                  │
  │        └────────────────────────────────────── Memory     │
  │          Less ◄────────────────────────► More             │
  └──────────────────────────────────────────────────────────┘
  
  Array:      Most memory,  fastest access      O(1) lookup
  HashMap:    Medium memory, amortized O(1)      O(1) avg
  Sorted List: Less memory,  O(log A) lookup    Binary search
  Compressed:  Least memory, complex operations  O(1) per edge
```

---

## When Memory Matters

### Competitive Programming Constraints

```
  Typical constraint: Memory Limit = 256 MB
  
  Max nodes with array-based (C++):
  256 MB / 216 bytes = ~1.18 million nodes
  
  Max nodes with flat array (int indices):
  256 MB / (26 × 4) bytes = ~2.46 million nodes
  
  Rule of thumb:
  N words × avg_length L = N × L max nodes
  
  If N = 100,000 and L = 10:
  → Up to 1,000,000 nodes → fits in 256 MB ✓
  
  If N = 500,000 and L = 20:
  → Up to 10,000,000 nodes → DOES NOT FIT ✗
  → Need compression or alternative approach
```

### Interview Context

```
  Memory is rarely the bottleneck in interviews.
  
  Interviewer asks about space complexity:
  → State O(N × L × A) for array-based
  → Mention HashMap reduces to O(N × L × avg_branching)
  → Compressed trie: O(N × L)
  
  If they ask to optimize:
  → Mention compressed tries (radix tree)
  → Mention __slots__ in Python
  → Mention pool allocation
```

---

## Garbage Collection Considerations

```
  Python/Java: Deleted nodes are garbage collected
  → Memory freed eventually, but GC has overhead
  → Many small objects = GC pressure
  → Pool allocation reduces GC pressure
  
  C++: Manual memory management
  → Must implement destructor to avoid leaks
  → unique_ptr/shared_ptr add refcount overhead
  → Pool allocation avoids individual deletes
```

Example C++ destructor:

```cpp
~Trie() {
    // Recursive deletion of all nodes
    function<void(TrieNode*)> cleanup = [&](TrieNode* node) {
        if (!node) return;
        for (auto& [ch, child] : node->children) {
            cleanup(child);
        }
        delete node;
    };
    cleanup(root);
}
```

---

## Summary Table

| Technique | Memory Saved | Complexity Added | Best For |
|-----------|:------------:|:----------------:|----------|
| HashMap children | ~30-50% | Minimal | Sparse tries |
| `__slots__` (Python) | ~40% | None | All Python tries |
| Pool/flat allocation | ~50-60% | Moderate | Competitive prog |
| Bit packing | ~60-70% | Significant | Memory-critical |
| Compressed trie | ~60-80% | Significant | Large dictionaries |
| Short indices (2B) | ~75% of pointers | Minor | < 32K nodes |

---

## Quick Revision Questions

1. **How much memory does a single trie node use in C++ with an array-based approach?**
   > 26 pointers × 8 bytes = 208 bytes for children, plus 1 byte for `isEnd` (+ padding) = ~216 bytes total.

2. **How does Python's `__slots__` reduce trie memory?**
   > It eliminates the per-instance `__dict__` dictionary (which stores attribute names and values), saving ~140 bytes per node — a ~40% reduction.

3. **What is pool/flat allocation and why is it used in competitive programming?**
   > Pre-allocating a large array of nodes and using integer indices instead of pointers. This avoids per-object overhead, improves cache locality, and makes allocation O(1) via counter increment.

4. **When would trie memory usage exceed a 256 MB limit?**
   > With array-based nodes (~216 bytes each), exceeding ~1.18 million nodes. For N=100K words of avg length 10, worst case is 1M nodes — borderline. Large alphabets or many long words push past the limit.

5. **What is the most effective single optimization for trie memory?**
   > Compressed tries (radix trees) — they merge chains of single-child nodes, often reducing node count by 60-80% while preserving O(L) operations.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Boolean vs Count](05-boolean-vs-count.md) | [Next: Autocomplete System ▶](../04-Prefix-Based-Problems/01-autocomplete-system.md) |
