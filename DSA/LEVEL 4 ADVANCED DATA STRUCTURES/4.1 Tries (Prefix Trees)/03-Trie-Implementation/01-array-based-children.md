# Array-Based Children Implementation

## Overview
The array-based approach stores children in a fixed-size array indexed by character. Each position maps directly to a character (e.g., index 0 = 'a', index 25 = 'z'). This is the most common trie implementation in competitive programming and interviews due to its simplicity and O(1) child access.

---

## Core Concept

```
  Character → Index mapping:
  
  'a' → 0,  'b' → 1,  'c' → 2, ..., 'z' → 25
  
  Formula:  index = char - 'a'    (for lowercase)
            index = char - 'A'    (for uppercase)
            index = char - '0'    (for digits)
  
  ┌─────────────────────────────────────────────────────┐
  │  Node Structure:                                     │
  │                                                      │
  │  ┌──────────────────────────────────┐                │
  │  │ children[26]  (array of pointers)│                │
  │  │ [0][1][2]...[25]                 │                │
  │  │  a   b   c     z                │                │
  │  │  ↓   ↓                           │                │
  │  │ node null                        │                │
  │  ├──────────────────────────────────┤                │
  │  │ isEnd: bool                      │                │
  │  └──────────────────────────────────┘                │
  └─────────────────────────────────────────────────────┘
```

---

## Full Implementation

### Python

```python
class TrieNode:
    def __init__(self):
        self.children = [None] * 26  # Fixed-size array
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def _char_to_index(self, ch):
        """Convert character to array index."""
        return ord(ch) - ord('a')
    
    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            idx = self._char_to_index(ch)
            if node.children[idx] is None:
                node.children[idx] = TrieNode()
            node = node.children[idx]
        node.is_end = True
    
    def search(self, word: str) -> bool:
        node = self.root
        for ch in word:
            idx = self._char_to_index(ch)
            if node.children[idx] is None:
                return False
            node = node.children[idx]
        return node.is_end
    
    def starts_with(self, prefix: str) -> bool:
        node = self.root
        for ch in prefix:
            idx = self._char_to_index(ch)
            if node.children[idx] is None:
                return False
            node = node.children[idx]
        return True
    
    def delete(self, word: str) -> bool:
        """Delete word from trie. Returns True if word existed."""
        return self._delete_helper(self.root, word, 0)
    
    def _delete_helper(self, node, word, depth):
        if node is None:
            return False
        
        if depth == len(word):
            if not node.is_end:
                return False
            node.is_end = False
            return self._is_empty(node)
        
        idx = self._char_to_index(word[depth])
        should_delete = self._delete_helper(node.children[idx], word, depth + 1)
        
        if should_delete:
            node.children[idx] = None
            return not node.is_end and self._is_empty(node)
        
        return False
    
    def _is_empty(self, node):
        """Check if node has no children."""
        return all(child is None for child in node.children)
```

### Java

```java
class TrieNode {
    TrieNode[] children;
    boolean isEnd;
    
    public TrieNode() {
        children = new TrieNode[26];
        isEnd = false;
    }
}

class Trie {
    private TrieNode root;
    
    public Trie() {
        root = new TrieNode();
    }
    
    public void insert(String word) {
        TrieNode node = root;
        for (char ch : word.toCharArray()) {
            int idx = ch - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }
    
    public boolean search(String word) {
        TrieNode node = root;
        for (char ch : word.toCharArray()) {
            int idx = ch - 'a';
            if (node.children[idx] == null) return false;
            node = node.children[idx];
        }
        return node.isEnd;
    }
    
    public boolean startsWith(String prefix) {
        TrieNode node = root;
        for (char ch : prefix.toCharArray()) {
            int idx = ch - 'a';
            if (node.children[idx] == null) return false;
            node = node.children[idx];
        }
        return true;
    }
}
```

### C++

```cpp
struct TrieNode {
    TrieNode* children[26];
    bool isEnd;
    
    TrieNode() : isEnd(false) {
        memset(children, 0, sizeof(children));  // All nullptrs
    }
};

class Trie {
private:
    TrieNode* root;
    
public:
    Trie() {
        root = new TrieNode();
    }
    
    void insert(string word) {
        TrieNode* node = root;
        for (char ch : word) {
            int idx = ch - 'a';
            if (!node->children[idx]) {
                node->children[idx] = new TrieNode();
            }
            node = node->children[idx];
        }
        node->isEnd = true;
    }
    
    bool search(string word) {
        TrieNode* node = root;
        for (char ch : word) {
            int idx = ch - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return node->isEnd;
    }
    
    bool startsWith(string prefix) {
        TrieNode* node = root;
        for (char ch : prefix) {
            int idx = ch - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return true;
    }
    
    // IMPORTANT: Destructor to prevent memory leaks
    ~Trie() {
        deleteTrie(root);
    }
    
    void deleteTrie(TrieNode* node) {
        if (!node) return;
        for (int i = 0; i < 26; i++) {
            deleteTrie(node->children[i]);
        }
        delete node;
    }
};
```

---

## Step-by-Step Trace

### Inserting "cat" and "car"

```
  Initial: root = [null × 26], isEnd = false
  
  ── Insert "cat" ──
  
  Step 1: ch='c', idx=2
    root.children[2] is null → create node
    root: [_, _, ✓, _, ..., _]
                ↓
               (c)
  
  Step 2: ch='a', idx=0
    c.children[0] is null → create node
    c: [✓, _, _, _, ..., _]
        ↓
       (a)
  
  Step 3: ch='t', idx=19
    a.children[19] is null → create node
    a: [_, _, ..., ✓, ..., _]
                   ↓
                  (t) isEnd = true
  
  ── Insert "car" ──
  
  Step 1: ch='c', idx=2
    root.children[2] EXISTS → follow it
  
  Step 2: ch='a', idx=0
    c.children[0] EXISTS → follow it
  
  Step 3: ch='r', idx=17
    a.children[17] is null → create node
    a: [_, _, ..., ✓, _, ✓, ..., _]
                   ↓      ↓
                  (t)*   (r) isEnd = true
  
  Final array state for node 'a':
  Index: [0][1][2]...[17][18][19]...[25]
  Value:  _   _   _    ✓    _    ✓     _
                       r         t
```

---

## Advantages of Array-Based

```
  ┌───────────────────────────────────────────────────────┐
  │  1. O(1) child access:  children[ch - 'a']           │
  │     No hashing, no collision handling                  │
  │                                                       │
  │  2. Cache-friendly:  Contiguous memory                │
  │     Array elements are adjacent in memory              │
  │                                                       │
  │  3. Simple code:  Easy to implement correctly         │
  │     Less error-prone than hash maps                    │
  │                                                       │
  │  4. Predictable performance:  No hash collisions      │
  │     Always O(1) per level, never degrades              │
  │                                                       │
  │  5. Competitive programming:  Industry standard       │
  │     Most trie solutions use arrays                     │
  └───────────────────────────────────────────────────────┘
```

---

## Disadvantages of Array-Based

```
  ┌───────────────────────────────────────────────────────┐
  │  1. Wasted memory:  26 slots even if 1 child          │
  │     Sparse nodes waste ~96% of allocated space         │
  │                                                       │
  │  2. Fixed alphabet:  Can't handle arbitrary chars      │
  │     Must know alphabet size at compile time            │
  │                                                       │
  │  3. Scaling issues:  Unicode needs 65,536 slots!      │
  │     Impractical for large alphabets                    │
  │                                                       │
  │  4. Iteration overhead:  Must scan all 26 slots       │
  │     To find children: O(ALPHABET_SIZE) per node        │
  └───────────────────────────────────────────────────────┘
```

---

## When to Use Array-Based

```
  USE ARRAY when:
  ✓ Alphabet is small and fixed (a-z, A-Z, 0-9)
  ✓ Speed is priority over memory
  ✓ Competitive programming / interviews
  ✓ Dense tries (many words, lots of branching)
  ✓ You know the character set upfront
  
  AVOID ARRAY when:
  ✗ Large alphabet (Unicode, multilingual)
  ✗ Memory is severely constrained
  ✗ Very sparse tries (few words, long unique paths)
  ✗ Character set is unknown at compile time
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Child storage | `children[ALPHABET_SIZE]` fixed array |
| Access time | O(1) via `children[ch - 'a']` |
| Space per node | ALPHABET_SIZE × pointer_size |
| Best for | Small fixed alphabets (a-z) |
| Worst for | Large/dynamic alphabets |
| Iterator | Must scan all ALPHABET_SIZE slots |
| Code complexity | Simple — ideal for interviews |

---

## Quick Revision Questions

1. **How do you convert a character to an array index?**
   > `index = ch - 'a'` for lowercase letters. This gives 'a' → 0, 'b' → 1, ..., 'z' → 25.

2. **What is the main disadvantage of the array-based approach?**
   > Memory waste — each node allocates 26 pointer slots even if only 1-2 are used. For sparse tries, ~96% of allocated space is wasted.

3. **Why is array-based access considered O(1)?**
   > Direct array indexing (`children[idx]`) accesses memory at a computed address in constant time — no searching, no hashing, no comparisons.

4. **How would you adapt the array for uppercase + lowercase letters?**
   > Use `children[52]` with mapping: `idx = ch - 'A'` for uppercase (0-25), `idx = ch - 'a' + 26` for lowercase (26-51).

5. **What happens if you forget to initialize the array to null/None?**
   > Uninitialized pointers contain garbage values, causing undefined behavior (segfault in C++). Always initialize to null/None/0.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Space Complexity](../02-Basic-Trie-Operations/06-space-complexity.md) | [Next: HashMap-Based Children ▶](02-hashmap-based-children.md) |
