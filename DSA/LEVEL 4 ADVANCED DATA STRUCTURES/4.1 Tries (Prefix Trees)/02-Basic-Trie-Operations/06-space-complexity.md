# Space Complexity Analysis

## Overview
Tries trade space for speed. This chapter analyzes the memory footprint of tries, compares array-based vs hashmap-based node storage, calculates real-world memory usage, and discusses when space becomes a concern.

---

## Core Space Formula

```
  Space = Number_of_Nodes × Size_per_Node
  
  ┌─────────────────────────────────────────────────┐
  │  Array-based (26 lowercase letters):            │
  │                                                  │
  │  Size per node = 26 pointers + 1 boolean        │
  │                = 26 × 8 bytes + 1 byte          │
  │                = 209 bytes per node (64-bit)     │
  │                                                  │
  │  HashMap-based:                                  │
  │                                                  │
  │  Size per node = HashMap overhead + entries      │
  │                ≈ 48 bytes base + 32 per entry    │
  │                (varies by language/load factor)  │
  └─────────────────────────────────────────────────┘
```

---

## Number of Nodes

### Best Case — Maximum Sharing

```
  Words: ["abc", "abd", "abef"]
  
  root ─ a ─ b ─ c*        4 shared nodes (root, a, b, e)
              ├─ d*         + 3 unique endpoints
              └─ e ─ f*     = 7 total nodes
  
  Characters in words: 3 + 3 + 4 = 10
  Nodes created: 7 (sharing saved 3 nodes = 30%)
```

### Worst Case — No Sharing

```
  Words: ["abc", "def", "ghi"]
  
  root ─ a ─ b ─ c*        No shared prefixes
       ├─ d ─ e ─ f*       Every character = new node
       └─ g ─ h ─ i*
  
  Characters in words: 3 + 3 + 3 = 9
  Nodes created: 10 (root + 9, no savings)
```

### General Formula

```
  Minimum nodes: 1 + L_longest    (all words share full prefix)
  Maximum nodes: 1 + Σ(L_i)      (no shared prefixes)
  
  Where L_i = length of word i
```

---

## Array-Based vs HashMap-Based Memory

### Array-Based (Alphabet Size = 26)

```
  Each node always allocates 26 pointers, even if unused.
  
  Example: Trie with "cat" only
  
  root: [_, _, c, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _]
            2 used, 24 wasted!
  c:    [a, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _]
            1 used, 25 wasted!
  a:    [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, t, _, _, _, _, _, _]
            1 used, 25 wasted!
  t:    [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _]
            0 used, all 26 wasted!
  
  Total: 4 nodes × 26 pointers = 104 pointers
  Used: only 3 pointers
  Utilization: 2.9%  ← Very wasteful for sparse tries!
```

### HashMap-Based

```
  Each node only stores existing children.
  
  Example: Trie with "cat" only
  
  root: {'c': →}                → 1 entry
  c:    {'a': →}                → 1 entry
  a:    {'t': →}                → 1 entry
  t:    {}                      → 0 entries
  
  Total entries: 3 (vs 104 allocated slots in array)
  But: HashMap has per-node overhead (object headers, buckets)
```

### Memory Comparison Table

| Metric | Array-Based | HashMap-Based |
|--------|:-----------:|:-------------:|
| Per-node base cost | 26 × 8 = 208 bytes | ~48 bytes (empty map) |
| Per-child cost | 0 (pre-allocated) | ~32 bytes per entry |
| Sparse node (1-2 children) | 208 bytes | 48 + 64 = 112 bytes |
| Dense node (10+ children) | 208 bytes | 48 + 320 = 368 bytes |
| Best for | Dense tries, fixed alphabet | Sparse tries, large alphabet |

### Break-Even Point

```
  Array cost:   208 bytes (constant)
  HashMap cost: 48 + 32 × K bytes (K = number of children)
  
  208 = 48 + 32K
  160 = 32K
  K = 5
  
  If average children per node > 5: Array wins
  If average children per node < 5: HashMap wins
  
  English dictionary: avg ~3-4 children per node → HashMap often wins
```

---

## Real-World Memory Calculations

### Example: 100,000 English Words (avg length 8)

```
  Estimated nodes: ~400,000 (with ~60% prefix sharing)
  
  Array-based:
  400,000 × 208 bytes = 83.2 MB
  
  HashMap-based (avg 3 children/node):
  400,000 × (48 + 3 × 32) = 400,000 × 144 = 57.6 MB
  
  Just storing the strings:
  100,000 × 8 bytes = 0.8 MB
  
  ┌────────────────────────────────────────────────────┐
  │  Trie uses 72-104x more memory than raw strings!  │
  │  This is the primary trade-off for O(L) lookups   │
  └────────────────────────────────────────────────────┘
```

### Example: Storing only 50 words (avg length 6)

```
  Estimated nodes: ~250 (minimal sharing)
  
  Array-based: 250 × 208 = 52 KB
  HashMap-based: 250 × 144 = 36 KB
  Raw strings: 300 bytes
  
  Overhead is huge percentage-wise, but absolute values are tiny.
  For small datasets, space doesn't matter — use tries freely.
```

---

## Space Complexity by Operation

| Operation | Additional Space | Notes |
|-----------|:----------------:|-------|
| insert(word) | O(L) worst | Creates up to L new nodes |
| search(word) | O(1) | No extra space (iterative) |
| search(word) | O(L) | If recursive implementation |
| delete(word) | O(L) | Recursive call stack |
| findAllWithPrefix | O(L + K) | Stack + result collection |
| Build N words | O(N × L × A) | A = alphabet-dependent constant |

---

## Reducing Space — Key Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  Strategy             │ Space Reduction │ Trade-off      │
  │───────────────────────│─────────────────│────────────────│
  │  HashMap children     │  ~30-50%        │ Slower access  │
  │  Compressed trie      │  ~60-80%        │ Complex code   │
  │  Array of pairs       │  ~40-60%        │ Linear search  │
  │  Bitmap + compact arr │  ~50-70%        │ Bit operations │
  │  External storage     │  Disk-based     │ I/O overhead   │
  └──────────────────────────────────────────────────────────┘
```

### Quick Comparison

```
  Standard trie:     root ─ a ─ p ─ p ─ l ─ e*    (6 nodes)
  
  Compressed trie:   root ─ "apple"*                (2 nodes)
  
  Savings: 4 nodes eliminated by path compression
```

---

## Space vs Other Data Structures

| Structure | Space for N strings (avg L) | Prefix Search |
|-----------|:---------------------------:|:-------------:|
| Array/List | O(N × L) | O(N × L) |
| Hash Set | O(N × L) + overhead | N/A natively |
| BST | O(N × L) + pointers | O(L × log N) |
| Trie (Array) | O(Nodes × Alphabet) | **O(P)** |
| Trie (HashMap) | O(Nodes × avg children) | **O(P)** |
| Compressed Trie | O(N × L) ≈ raw storage | **O(P)** |

---

## Interview Space Analysis Template

When analyzing trie space in interviews, use this framework:

```
  1. Count maximum nodes:
     - Upper bound: 1 + sum of all word lengths
     - Practical: depends on prefix sharing
  
  2. Multiply by per-node cost:
     - Array: ALPHABET_SIZE × pointer_size
     - HashMap: base + avg_children × entry_size
  
  3. Add auxiliary data:
     - isEnd boolean(s)
     - count fields
     - stored values
  
  4. State the complexity:
     - O(TOTAL_CHARS × ALPHABET_SIZE) for array-based
     - O(TOTAL_CHARS × AVG_BRANCHING) for hashmap-based
     
  5. Compare (if asked):
     - vs hash set: trie uses more but enables prefix ops
     - vs sorted array: similar total but different access patterns
```

---

## Summary Table

| Key Concept | Detail |
|-------------|--------|
| Space driver | Number of nodes × size per node |
| Array node size | ALPHABET × pointer_size (~208 bytes for a-z) |
| HashMap node size | Base (~48 bytes) + children × ~32 bytes |
| Prefix sharing | Reduces node count; more sharing = less space |
| Break-even point | Array wins when avg children > 5 per node |
| Real-world overhead | 72-104x more than raw string storage |
| Biggest optimization | Compressed tries (radix tree) |
| Interview answer | O(N × L × A) where A = alphabet-dependent constant |

---

## Quick Revision Questions

1. **What is the space per node for an array-based trie with 26 lowercase letters?**
   > 26 pointers × 8 bytes = 208 bytes (on 64-bit system), plus a boolean for isEnd.

2. **When does a HashMap-based trie use less memory than an array-based trie?**
   > When the average number of children per node is less than ~5. For sparse tries (typical English), HashMap wins.

3. **How does prefix sharing affect space?**
   > Words with common prefixes share trie nodes, reducing total node count. "cat", "car", "card" share the "ca" path (2 nodes instead of 6).

4. **What is the space complexity for inserting N words of average length L?**
   > O(N × L × A) in worst case, where A is the alphabet-dependent per-node cost. With prefix sharing, actual nodes created can be much less.

5. **How does a compressed trie reduce space?**
   > It merges chains of single-child nodes into one node storing multiple characters, dramatically reducing node count. "apple" → 2 nodes instead of 6.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Time Complexity](05-time-complexity.md) | [Next: Array-Based Children ▶](../03-Trie-Implementation/01-array-based-children.md) |
