# Ternary Search Tree (TST)

## Overview
A **Ternary Search Tree** combines properties of tries and binary search trees. Each node has three children: **left** (less than), **middle** (equal), and **right** (greater than). TSTs use less memory than standard tries while maintaining efficient prefix-based operations.

---

## Structure

```
  Each node stores:
  - char: the character at this node
  - left: pointer to node with smaller character
  - mid: pointer to next character in the word (equal)
  - right: pointer to node with larger character
  - is_end: marks end of a word
  
  Comparison:
  Standard Trie node: 26 pointers (mostly null)
  TST node: 3 pointers + 1 char = much less wasted space
```

```
  Insert "cat", "car", "card", "cup", "bat":
  
              c                    ← root: first char
             / \
            b   u                  ← b < c, u > c
            |   |
            a   p*                 ← mid pointers
            |
            t*                     ← 'cat' ends here
            
  Actually, let's draw more carefully:
  
  root: 'c'
   ├── left:  'b' (b < c)
   │    └── mid: 'a'
   │          └── mid: 't'*
   ├── mid:   'a' (equal, next char of words starting with 'c')
   │    └── mid: 't'* 
   │          └── left: 'r'*        ← 'car' (r < t)
   │                └── mid: 'd'*   ← 'card'
   └── right: (none > c here)
   
  Wait — 'cup' starts with 'c' too.
  After 'c' mid→'a', but 'u' > 'a':
  
  root: 'c'
   ├── left: 'b'
   │    └── mid: 'a'
   │         └── mid: 't'*
   └── mid: 'a'               ← for words starting with 'c'
        └── right: 'u'        ← 'u' > 'a'
        │    └── mid: 'p'*    ← 'cup' ends
        └── mid: 't'*         ← 'cat' ends
             └── left: 'r'*   ← 'car' ends (r < t)
                  └── mid: 'd'*  ← 'card' ends
```

---

## Implementation

```python
class TSTNode:
    def __init__(self, char):
        self.char = char
        self.left = None    # Characters less than this
        self.mid = None     # Next character (match)
        self.right = None   # Characters greater than this
        self.is_end = False
        self.value = None   # Optional: for key-value storage

class TernarySearchTree:
    def __init__(self):
        self.root = None
    
    def insert(self, word, value=None):
        self.root = self._insert(self.root, word, 0, value)
    
    def _insert(self, node, word, index, value):
        ch = word[index]
        
        if node is None:
            node = TSTNode(ch)
        
        if ch < node.char:
            node.left = self._insert(node.left, word, index, value)
        elif ch > node.char:
            node.right = self._insert(node.right, word, index, value)
        else:  # ch == node.char
            if index + 1 < len(word):
                node.mid = self._insert(node.mid, word, index + 1, value)
            else:
                node.is_end = True
                node.value = value
        
        return node
    
    def search(self, word):
        node = self._search_node(word)
        return node is not None and node.is_end
    
    def _search_node(self, word):
        node = self.root
        index = 0
        
        while node is not None and index < len(word):
            ch = word[index]
            
            if ch < node.char:
                node = node.left
            elif ch > node.char:
                node = node.right
            else:
                if index + 1 == len(word):
                    return node
                node = node.mid
                index += 1
        
        return None
    
    def starts_with(self, prefix):
        """Check if any word starts with prefix."""
        node = self._search_node(prefix)
        return node is not None
    
    def words_with_prefix(self, prefix):
        """Return all words starting with prefix."""
        results = []
        node = self._search_node(prefix)
        
        if node is None:
            return results
        
        if node.is_end:
            results.append(prefix)
        
        self._collect(node.mid, prefix, results)
        return results
    
    def _collect(self, node, prefix, results):
        if node is None:
            return
        
        self._collect(node.left, prefix, results)
        
        current = prefix + node.char
        if node.is_end:
            results.append(current)
        self._collect(node.mid, current, results)
        
        self._collect(node.right, prefix, results)
```

### Java

```java
class TSTNode {
    char ch;
    TSTNode left, mid, right;
    boolean isEnd;
    
    TSTNode(char ch) { this.ch = ch; }
}

class TernarySearchTree {
    TSTNode root;
    
    void insert(String word) {
        root = insert(root, word, 0);
    }
    
    TSTNode insert(TSTNode node, String word, int idx) {
        char ch = word.charAt(idx);
        if (node == null) node = new TSTNode(ch);
        
        if (ch < node.ch) node.left = insert(node.left, word, idx);
        else if (ch > node.ch) node.right = insert(node.right, word, idx);
        else {
            if (idx + 1 < word.length())
                node.mid = insert(node.mid, word, idx + 1);
            else
                node.isEnd = true;
        }
        return node;
    }
    
    boolean search(String word) {
        TSTNode node = searchNode(word);
        return node != null && node.isEnd;
    }
    
    TSTNode searchNode(String word) {
        TSTNode node = root;
        int idx = 0;
        while (node != null && idx < word.length()) {
            char ch = word.charAt(idx);
            if (ch < node.ch) node = node.left;
            else if (ch > node.ch) node = node.right;
            else {
                if (idx + 1 == word.length()) return node;
                node = node.mid;
                idx++;
            }
        }
        return null;
    }
}
```

---

## Trace: Search

```
  Tree contains: "cat", "car", "cup", "bat"
  
  Search "car":
  
  Step 1: node=root('c'), word[0]='c' → EQUAL → mid, index=1
  Step 2: node='a', word[1]='a' → EQUAL → mid, index=2
  Step 3: node='t', word[2]='r' → r < t → go LEFT
  Step 4: node='r', word[2]='r' → EQUAL, index=3==len → return node
          node.is_end = true → FOUND ✓
  
  Search "cab":
  
  Step 1: node='c', 'c'='c' → mid, index=1
  Step 2: node='a', 'a'='a' → mid, index=2
  Step 3: node='t', 'b' < 't' → go LEFT
  Step 4: node='r', 'b' < 'r' → go LEFT
  Step 5: node=None → NOT FOUND ✗
```

---

## Comparison: Trie vs TST vs HashMap

| Feature | Standard Trie | TST | HashMap |
|---------|:---:|:---:|:---:|
| Search | O(L) | O(L + log A) | O(L) avg |
| Insert | O(L) | O(L + log A) | O(L) avg |
| Space per node | 26 pointers | 3 pointers + char | — |
| Prefix search | O(P + K) | O(P + log A + K) | O(N × L) |
| Sorted iteration | Yes | Yes | No |
| Memory efficiency | Low | High | Medium |
| Cache performance | Good | Poor (pointer chasing) | Best |

A = alphabet size. The `log A` factor comes from the BST-like left/right traversal at each level.

---

## Balancing a TST

```
  Issue: If words are inserted in sorted order,
  the left/right trees degenerate into linked lists.
  
  Solutions:
  1. Insert words in random order
  2. Use a balanced BST (AVL/Red-Black) for left/right
  3. Pre-sort and insert median first (like balanced BST construction)
  
  Balanced TST search: O(L × log A) → O(L × log 26) ≈ O(5L)
  Unbalanced worst case: O(L × A) → O(26L)
```

---

## Wildcard Search with TST

```python
def wildcard_search(self, pattern):
    """Search for pattern with '.' as wildcard."""
    results = []
    self._wildcard(self.root, pattern, 0, "", results)
    return results

def _wildcard(self, node, pattern, index, current, results):
    if node is None or index > len(pattern):
        return
    
    ch = pattern[index] if index < len(pattern) else None
    
    if ch == '.' or (ch and ch < node.char):
        self._wildcard(node.left, pattern, index, current, results)
    
    if ch == '.' or ch == node.char:
        if index + 1 == len(pattern) and node.is_end:
            results.append(current + node.char)
        if index + 1 < len(pattern):
            self._wildcard(node.mid, pattern, index + 1, 
                          current + node.char, results)
    
    if ch == '.' or (ch and ch > node.char):
        self._wildcard(node.right, pattern, index, current, results)
```

---

## When to Use TST

```
  ✓ Memory is tight and alphabet is large
  ✓ Need sorted iteration of keys
  ✓ Moderate-size dictionaries (spell checkers)
  ✓ Need prefix search but can't afford 26-wide arrays
  ✓ Unicode text (massive potential alphabet)
  
  ✗ Maximum speed required (standard trie is faster)
  ✗ Very small alphabet (2-4 chars → binary/small trie)
  ✗ Only exact lookups needed (HashMap is simpler)
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Structure | BST at each level; 3 children: left, mid, right |
| Space | 3 pointers per node (vs 26 in standard trie) |
| Search | O(L + log A) per operation |
| Memory | ~10× less than standard trie |
| Trade-off | Less memory, slightly slower, more complex |
| Best for | Large alphabets, memory-constrained scenarios |

---

## Quick Revision Questions

1. **How does a TST combine trie and BST properties?**
   > The middle pointer acts like a trie link (advancing to the next character). The left and right pointers act like a BST, organizing characters at the same position alphabetically.

2. **What determines the height of a TST?**
   > The height depends on word lengths (mid depth) and alphabet distribution at each level (left/right depth). Total height is bounded by max_word_length × tree_balance_factor.

3. **Why is TST more memory-efficient than a standard trie?**
   > A standard trie allocates 26 child pointers per node (most null). A TST uses only 3 pointers per node plus a character field, avoiding wasted null pointers.

4. **How does prefix search work in a TST?**
   > Navigate to the node matching the last character of the prefix (using the BST search at each level). Then collect all words in the subtree rooted at that node's mid child.

5. **What happens if words are inserted in sorted order?**
   > The left/right BST at each level degenerates into a linked list, making search O(L × A) instead of O(L × log A). Randomizing insertion order or using balanced BST techniques fixes this.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Compressed Trie](01-compressed-trie-radix-tree.md) | [Next: Patricia Trie ▶](03-patricia-trie.md) |
