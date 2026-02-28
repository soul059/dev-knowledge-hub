# Map Sum Pairs (LeetCode 677)

## Overview
Design a data structure that supports two operations: **insert(key, val)** — insert or update a key-value pair, and **sum(prefix)** — return the sum of all values whose keys start with the given prefix. The trie stores cumulative prefix sums at each node.

---

## Problem Statement

```
  insert("apple", 3)
  sum("ap")           → 3
  insert("app", 2)
  sum("ap")           → 5 (apple=3 + app=2)
  insert("apple", 2)  → UPDATE apple from 3 to 2
  sum("ap")           → 4 (apple=2 + app=2)
```

---

## Approach: Trie with Prefix Sum

```
  Each trie node stores a running sum of all values passing through it.
  
  Insert "apple" with val=3:
    root(+3) → a(+3) → p(+3) → p(+3) → l(+3) → e(+3)*
  
  Insert "app" with val=2:
    root(+2) → a(+2) → p(+2) → p(+2)*
  
  Node sums:  root=5, a=5, p=5, p(second)=5, l=3, e=3
  
  sum("ap") = value at node 'p' (after a→p) = 5 ✓
  
  For updates: compute delta = new_val - old_val,
  then add delta along the path.
```

---

## Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.prefix_sum = 0

class MapSum:
    def __init__(self):
        self.root = TrieNode()
        self.map = {}  # key → current value (for updates)
    
    def insert(self, key, val):
        # Calculate delta for insert/update
        delta = val - self.map.get(key, 0)
        self.map[key] = val
        
        # Update prefix sums along the path
        node = self.root
        node.prefix_sum += delta
        for ch in key:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
            node.prefix_sum += delta
    
    def sum(self, prefix):
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return node.prefix_sum
```

### Java

```java
class MapSum {
    int[][] trie = new int[10001][26];
    int[] prefixSum = new int[10001];
    int idx = 0;
    Map<String, Integer> map = new HashMap<>();
    
    public MapSum() {
        Arrays.fill(trie[0], -1);
    }
    
    public void insert(String key, int val) {
        int delta = val - map.getOrDefault(key, 0);
        map.put(key, val);
        
        int node = 0;
        prefixSum[node] += delta;
        for (char ch : key.toCharArray()) {
            int c = ch - 'a';
            if (trie[node][c] == -1) {
                trie[node][c] = ++idx;
                Arrays.fill(trie[idx], -1);
            }
            node = trie[node][c];
            prefixSum[node] += delta;
        }
    }
    
    public int sum(String prefix) {
        int node = 0;
        for (char ch : prefix.toCharArray()) {
            int c = ch - 'a';
            if (trie[node][c] == -1) return 0;
            node = trie[node][c];
        }
        return prefixSum[node];
    }
}
```

### C++

```cpp
class MapSum {
    struct TrieNode {
        unordered_map<char, TrieNode*> children;
        int prefixSum = 0;
    };
    
    TrieNode* root = new TrieNode();
    unordered_map<string, int> map;
    
public:
    void insert(string key, int val) {
        int delta = val - map[key];
        map[key] = val;
        
        auto node = root;
        node->prefixSum += delta;
        for (char ch : key) {
            if (!node->children.count(ch))
                node->children[ch] = new TrieNode();
            node = node->children[ch];
            node->prefixSum += delta;
        }
    }
    
    int sum(string prefix) {
        auto node = root;
        for (char ch : prefix) {
            if (!node->children.count(ch)) return 0;
            node = node->children[ch];
        }
        return node->prefixSum;
    }
};
```

---

## Trace

```
  insert("apple", 3):
    delta = 3 - 0 = 3
    map: {"apple": 3}
    
    root(3) → a(3) → p(3) → p(3) → l(3) → e(3)*

  sum("ap"):
    root → a → p → return prefixSum = 3 ✓

  insert("app", 2):
    delta = 2 - 0 = 2
    map: {"apple": 3, "app": 2}
    
    root(5) → a(5) → p(5) → p(5)* → l(3) → e(3)*
    
    Nodes a, p (first), p (second) all get +2.
    Node l and e unchanged (not on "app" path).

  sum("ap"):
    root → a → p → return prefixSum = 5 ✓

  insert("apple", 2):  ← UPDATE
    delta = 2 - 3 = -1
    map: {"apple": 2, "app": 2}
    
    root(4) → a(4) → p(4) → p(4)* → l(2) → e(2)*
    
    All nodes on "apple" path get -1.

  sum("ap"):
    root → a → p → return prefixSum = 4 ✓ (apple=2 + app=2)
```

---

## Why Delta-Based Updates?

```
  Without delta tracking:
  
  Option A: Recalculate all prefix sums from scratch → O(total chars)
  Option B: Store individual values at leaves, DFS for sum → O(subtree)
  
  With delta tracking:
  
  Just update the path for the changed key → O(key length)
  
  The map stores current values to compute:
    delta = new_value - old_value
  
  This handles both inserts (old=0) and updates correctly.
```

---

## Alternative: Trie + DFS (No Prefix Sum)

```python
class MapSum:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, key, val):
        node = self.root
        for ch in key:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.val = val  # Store value at leaf
    
    def sum(self, prefix):
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return 0
            node = node.children[ch]
        return self._dfs_sum(node)
    
    def _dfs_sum(self, node):
        total = getattr(node, 'val', 0)
        for child in node.children.values():
            total += self._dfs_sum(child)
        return total
```

```
  Comparison:
  | Approach | insert() | sum() |
  |----------|----------|-------|
  | Prefix sum + delta | O(L) | O(P) |
  | DFS from prefix | O(L) | O(subtree size) |
  
  Prefix sum is better when sum() is called frequently.
  DFS is simpler but slower for large subtrees.
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|:----:|:-----:|
| insert | O(L) | O(L) new nodes |
| sum | O(P) | O(1) |
| Space total | — | O(N × L) trie + O(N) map |

Where L = key length, P = prefix length, N = number of keys.

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Insert key-val pairs; query sum for all keys with given prefix |
| Trie role | Store prefix_sum at each node |
| Delta update | new_val - old_val applied along path |
| HashMap | Tracks current value per key for delta computation |
| sum() | Walk prefix, return node.prefix_sum — O(P) |
| LeetCode | 677 |

---

## Quick Revision Questions

1. **Why do we need a separate HashMap alongside the trie?**
   > To track the current value associated with each key. When a key is re-inserted with a new value, we compute `delta = new_val - old_val` and only adjust prefix sums by delta, not the full new value.

2. **What happens if we insert the same key with a new value?**
   > We compute delta (new - old), update the map, and add delta to prefix_sum of every node along the key's path. This correctly adjusts the cumulative sums.

3. **Why is `sum()` O(P) instead of O(subtree)?**
   > Because prefix_sum at each node already aggregates the sum of all keys passing through it. We just walk to the prefix endpoint and read the stored sum — no DFS needed.

4. **What if we insert "apple" with value 0?**
   > Delta = 0 - old_val. The prefix sums are decremented by old_val (effectively removing its contribution). The key remains in the trie and map with value 0.

5. **Can this approach handle negative values?**
   > Yes. Delta computation works with any integer values. prefix_sum additions and subtractions handle negative values correctly.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Magic Dictionary](05-implement-magic-dictionary.md) | [Next: Compressed Trie (Radix Tree) ▶](../08-Trie-Optimizations/01-compressed-trie-radix-tree.md) |
