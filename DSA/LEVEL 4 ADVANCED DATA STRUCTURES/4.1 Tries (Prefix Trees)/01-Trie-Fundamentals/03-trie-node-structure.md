# Trie Node Structure

## Overview
The building block of every trie is the **TrieNode**. Understanding its internal structure is essential before implementing any trie operation. This chapter details the anatomy of a trie node, its two primary implementation approaches, and how nodes connect to form the tree.

---

## Anatomy of a TrieNode

Every trie node has two essential components:

```
  ┌──────────────────────────────────┐
  │          TrieNode                │
  ├──────────────────────────────────┤
  │  children: map/array of nodes   │
  │     (one slot per character)     │
  │                                  │
  │  isEndOfWord: boolean            │
  │     (marks complete words)       │
  │                                  │
  │  [optional] count: int           │
  │  [optional] value: any           │
  │  [optional] prefixCount: int     │
  └──────────────────────────────────┘
```

### 1. Children — The Edges

The `children` field maps characters to child TrieNodes. It represents the edges going down from this node.

```
  Node 'a' has children 'p' and 'r':

       a       ← current node
      / \
     p   r     ← children['p'] and children['r']
```

### 2. isEndOfWord — The Word Marker

This boolean flag tells us whether a complete word ends at this node.

```
  Words: ["in", "inn"]

      (root)
        │
        i
        │
        n*     ← isEndOfWord = true ("in" ends here)
        │
        n*     ← isEndOfWord = true ("inn" ends here)
```

Without `isEndOfWord`, we couldn't distinguish "in" (a word) from the prefix "in" of "inn".

---

## Array-Based Node (Fixed Alphabet)

For lowercase English letters (a-z), each node holds an array of 26 pointers:

```
  TrieNode:
  ┌─────────────────────────────────────────────────────────────────┐
  │  children[26]                                                   │
  │  Index:  0    1    2    3    4    5   ...  25                   │
  │  Char:  'a'  'b'  'c'  'd'  'e'  'f' ... 'z'                  │
  │         ┌────┬────┬────┬────┬────┬────┬───┬────┐               │
  │  Value: │null│null│ ●  │null│null│null│...│null│               │
  │         └────┴────┴─┬──┴────┴────┴────┴───┴────┘               │
  │                     │                                           │
  │                     ▼  (pointer to child TrieNode for 'c')      │
  │                  TrieNode                                       │
  │                                                                 │
  │  isEndOfWord: false                                             │
  └─────────────────────────────────────────────────────────────────┘

  Character → Index mapping:  index = char - 'a'
  Example: 'a' → 0,  'c' → 2,  'z' → 25
```

**Pseudocode:**
```
class TrieNode:
    children = new Node[26]    // fixed-size array
    isEndOfWord = false
    
    // Access child for character 'c':
    getChild(c):
        return children[c - 'a']
    
    setChild(c, node):
        children[c - 'a'] = node
    
    hasChild(c):
        return children[c - 'a'] != null
```

---

## HashMap-Based Node (Flexible Alphabet)

Instead of a fixed array, store only the children that actually exist:

```
  TrieNode with HashMap children:

  Node for 'a' (has children 'p' and 'r' only):
  ┌───────────────────────────────────────────┐
  │  children: HashMap                        │
  │  ┌─────────────────────────────┐          │
  │  │  'p' ──► TrieNode          │          │
  │  │  'r' ──► TrieNode          │          │
  │  └─────────────────────────────┘          │
  │  (only 2 entries, not 26!)                │
  │                                           │
  │  isEndOfWord: false                       │
  └───────────────────────────────────────────┘
```

**Pseudocode:**
```
class TrieNode:
    children = {}              // empty hash map
    isEndOfWord = false
    
    // Access child for character 'c':
    getChild(c):
        return children.get(c, null)
    
    setChild(c, node):
        children[c] = node
    
    hasChild(c):
        return c in children
```

---

## Memory Layout Example

Inserting words `"cat"` and `"car"`:

```
  Level 0 (root):    (root)
                       │
  Level 1:             c
                       │
  Level 2:             a
                      / \
  Level 3:           t*   r*

  Node-by-node breakdown:
  ┌──────────────────────────────────────────────────────────┐
  │ Node     │ children            │ isEndOfWord │ Depth     │
  ├──────────┼─────────────────────┼─────────────┼───────────┤
  │ root     │ {'c': node_c}       │ false       │ 0         │
  │ node_c   │ {'a': node_a}       │ false       │ 1         │
  │ node_a   │ {'t': node_t,       │ false       │ 2         │
  │          │  'r': node_r}       │             │           │
  │ node_t   │ {}                  │ true ←"cat" │ 3         │
  │ node_r   │ {}                  │ true ←"car" │ 3         │
  └──────────┴─────────────────────┴─────────────┴───────────┘
```

---

## Optional Fields

Beyond the two essential fields, nodes can carry additional metadata:

### Word Count (for duplicates)
```
class TrieNode:
    children = {}
    isEndOfWord = false
    wordCount = 0        // how many times this word was inserted
```

### Prefix Count (for prefix queries)
```
class TrieNode:
    children = {}
    isEndOfWord = false
    prefixCount = 0      // how many words pass through this node
```

### Value Storage (for maps)
```
class TrieNode:
    children = {}
    isEndOfWord = false
    value = null         // associated value (trie as key-value map)
```

### Word Reference (for word search problems)
```
class TrieNode:
    children = {}
    word = null          // store the complete word at its end node
```

---

## Node Memory Footprint

```
  ╔════════════════════════════════════════════════════════════════╗
  ║              MEMORY PER NODE (64-bit system)                  ║
  ╠════════════════════════════╦═══════════════════════════════════╣
  ║  Array[26] pointers        ║  26 × 8 = 208 bytes              ║
  ║  isEndOfWord (boolean)     ║  1 byte (+ alignment ≈ 8)        ║
  ║  Object/Struct overhead    ║  ~16 bytes (Java/Python)          ║
  ║  ────────────────────────  ║  ───────────────────              ║
  ║  TOTAL (Array-based)       ║  ~232 bytes per node              ║
  ╠════════════════════════════╬═══════════════════════════════════╣
  ║  HashMap (2 children)      ║  ~48 base + 2×40 = ~128 bytes    ║
  ║  isEndOfWord (boolean)     ║  ~8 bytes                         ║
  ║  Object/Struct overhead    ║  ~16 bytes                        ║
  ║  ────────────────────────  ║  ───────────────────              ║
  ║  TOTAL (HashMap, sparse)   ║  ~152 bytes per node              ║
  ╚════════════════════════════╩═══════════════════════════════════╝
```

---

## Visualization: Full Node Graph

```
  Trie storing: ["abc", "abd", "bcd"]

  ┌──────────┐
  │   ROOT   │ children: {a: ●, b: ●}
  │  end: F  │ isEndOfWord: false
  └──┬───┬───┘
     │   │
     ▼   ▼
  ┌──┴──┐ ┌──┴──┐
  │  a  │ │  b  │
  │ E:F │ │ E:F │
  │ C:{b}│ │C:{c}│
  └──┬──┘ └──┬──┘
     ▼       ▼
  ┌──┴──┐ ┌──┴──┐
  │  b  │ │  c  │
  │ E:F │ │ E:F │
  │C:{c,d}│ │C:{d}│
  └─┬──┬┘ └──┬──┘
    ▼  ▼      ▼
  ┌─┴─┐┌─┴─┐┌─┴──┐
  │ c ││ d ││ d  │
  │E:T││E:T││E:T │
  │C:{}││C:{}││C:{}│
  └───┘└───┘└────┘

  E = isEndOfWord, C = children, F = false, T = true
```

---

## Summary Table

| Component | Purpose | Type |
|-----------|---------|------|
| `children` | Maps characters to child nodes | Array[Σ] or HashMap |
| `isEndOfWord` | Marks complete words | boolean |
| `wordCount` | Tracks insertion count | int (optional) |
| `prefixCount` | Counts words through node | int (optional) |
| `value` | Stores associated data | any (optional) |
| `word` | Stores complete word string | string (optional) |

---

## Quick Revision Questions

1. **What are the two essential fields in a TrieNode?**
   > `children` (map/array of child nodes) and `isEndOfWord` (boolean indicating if a complete word ends here).

2. **How does array indexing work for character 'g' in a 26-letter trie?**
   > `index = 'g' - 'a' = 6`. The child for 'g' is stored at `children[6]`.

3. **How much memory does an array-based node use on a 64-bit system?**
   > ~232 bytes: 26 pointers × 8 bytes + boolean + object overhead.

4. **Why might you store a `word` reference instead of just `isEndOfWord`?**
   > In problems like Word Search II, storing the complete word avoids reconstructing it during DFS traversal. Much simpler code.

5. **When is `prefixCount` useful?**
   > When you need to quickly answer "how many words start with this prefix?" — just navigate to the prefix end and read `prefixCount`.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Trie vs Hash Table](02-trie-vs-hash-table.md) | [Next: Prefix Matching Concept ▶](04-prefix-matching-concept.md) |
