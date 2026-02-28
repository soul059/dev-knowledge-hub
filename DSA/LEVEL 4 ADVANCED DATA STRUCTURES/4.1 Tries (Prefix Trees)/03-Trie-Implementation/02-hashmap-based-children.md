# HashMap-Based Children Implementation

## Overview
The hashmap-based approach stores only the children that actually exist, using a dictionary/map keyed by character. This is more memory-efficient for sparse tries and supports arbitrary character sets without a fixed alphabet size.

---

## Core Concept

```
  Array-based (always 26 slots):
  ┌────────────────────────────────────────────────────────┐
  │ [_][_][c][_][_]...[_]  ← 26 slots, 25 wasted          │
  └────────────────────────────────────────────────────────┘
  
  HashMap-based (only existing children):
  ┌──────────────┐
  │ {'c': node}  │  ← 1 entry, 0 wasted
  └──────────────┘
  
  Trade-off: HashMap has per-entry overhead but no wasted slots
```

---

## Full Implementation

### Python

```python
class TrieNode:
    def __init__(self):
        self.children = {}   # dict: char → TrieNode
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
    
    def search(self, word: str) -> bool:
        node = self._find_node(word)
        return node is not None and node.is_end
    
    def starts_with(self, prefix: str) -> bool:
        return self._find_node(prefix) is not None
    
    def _find_node(self, prefix: str):
        """Navigate to the node at end of prefix."""
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
    
    def delete(self, word: str) -> bool:
        return self._delete_helper(self.root, word, 0)
    
    def _delete_helper(self, node, word, depth):
        if depth == len(word):
            if not node.is_end:
                return False
            node.is_end = False
            return len(node.children) == 0
        
        ch = word[depth]
        if ch not in node.children:
            return False
        
        should_delete = self._delete_helper(node.children[ch], word, depth + 1)
        
        if should_delete:
            del node.children[ch]
            return not node.is_end and len(node.children) == 0
        
        return False
    
    def get_all_words(self):
        """Collect all words in trie."""
        words = []
        self._dfs(self.root, [], words)
        return words
    
    def _dfs(self, node, path, words):
        if node.is_end:
            words.append(''.join(path))
        for ch, child in sorted(node.children.items()):
            path.append(ch)
            self._dfs(child, path, words)
            path.pop()
```

### Java

```java
import java.util.*;

class TrieNode {
    Map<Character, TrieNode> children;
    boolean isEnd;
    
    public TrieNode() {
        children = new HashMap<>();
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
            node.children.putIfAbsent(ch, new TrieNode());
            node = node.children.get(ch);
        }
        node.isEnd = true;
    }
    
    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isEnd;
    }
    
    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }
    
    private TrieNode findNode(String prefix) {
        TrieNode node = root;
        for (char ch : prefix.toCharArray()) {
            if (!node.children.containsKey(ch)) return null;
            node = node.children.get(ch);
        }
        return node;
    }
}
```

### C++

```cpp
#include <unordered_map>
#include <string>
using namespace std;

struct TrieNode {
    unordered_map<char, TrieNode*> children;
    bool isEnd;
    
    TrieNode() : isEnd(false) {}
    
    ~TrieNode() {
        for (auto& [ch, child] : children) {
            delete child;
        }
    }
};

class Trie {
    TrieNode* root;
    
public:
    Trie() : root(new TrieNode()) {}
    
    ~Trie() { delete root; }
    
    void insert(const string& word) {
        TrieNode* node = root;
        for (char ch : word) {
            if (node->children.find(ch) == node->children.end()) {
                node->children[ch] = new TrieNode();
            }
            node = node->children[ch];
        }
        node->isEnd = true;
    }
    
    bool search(const string& word) {
        TrieNode* node = findNode(word);
        return node && node->isEnd;
    }
    
    bool startsWith(const string& prefix) {
        return findNode(prefix) != nullptr;
    }
    
private:
    TrieNode* findNode(const string& s) {
        TrieNode* node = root;
        for (char ch : s) {
            auto it = node->children.find(ch);
            if (it == node->children.end()) return nullptr;
            node = it->second;
        }
        return node;
    }
};
```

---

## Step-by-Step Trace

### Inserting "app", "apple", "bat"

```
  ── Insert "app" ──
  
  root.children = {}
  
  ch='a': 'a' not in {} → create node
          root.children = {'a': Node}
  
  ch='p': 'p' not in {} → create node
          a.children = {'p': Node}
  
  ch='p': 'p' not in {} → create node
          p1.children = {'p': Node}
          p2.isEnd = True ✓
  
  ── Insert "apple" ──
  
  ch='a': 'a' IN root.children → follow
  ch='p': 'p' IN a.children → follow
  ch='p': 'p' IN p1.children → follow (p2, isEnd=True)
  ch='l': 'l' not in p2.children → create node
          p2.children = {'l': Node}
  ch='e': 'e' not in l.children → create node
          l.children = {'e': Node}
          e.isEnd = True ✓
  
  ── Insert "bat" ──
  
  ch='b': 'b' not in root.children → create node
          root.children = {'a': Node, 'b': Node}
  ch='a': 'a' not in b.children → create
  ch='t': 't' not in a2.children → create
          t.isEnd = True ✓
  
  Final trie:
  
  root ─── 'a' → a ─── 'p' → p1 ─── 'p' → p2* ─── 'l' → l ─── 'e' → e*
       └── 'b' → b ─── 'a' → a2 ─── 't' → t*
  
  * = isEnd = True
```

---

## Python `defaultdict` Approach

A more elegant Python version using `defaultdict`:

```python
from collections import defaultdict

class TrieNode:
    def __init__(self):
        self.children = defaultdict(TrieNode)  # Auto-creates on access
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            node = node.children[ch]  # Auto-creates if missing!
        node.is_end = True
    
    def search(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end
```

> **Warning**: `defaultdict(TrieNode)` creates nodes on mere access! Use `ch in node.children` to check without creating.

---

## Array vs HashMap — Side-by-Side

```
  ┌────────────────────┬────────────────────────┐
  │   ARRAY-BASED      │   HASHMAP-BASED        │
  ├────────────────────┼────────────────────────┤
  │ children[26]       │ children = {}          │
  │ idx = ch - 'a'     │ key = ch               │
  │ children[idx]      │ children[ch]           │
  │ == null check      │ 'in' / contains check  │
  │ O(1) guaranteed    │ O(1) amortized         │
  │ O(26) to iterate   │ O(K) to iterate        │
  │ 208 bytes/node     │ ~48 + 32K bytes/node   │
  │ Fixed alphabet     │ Any character set      │
  └────────────────────┴────────────────────────┘
```

### Iteration Comparison

```python
# Array: Must check all 26 slots
for i in range(26):
    if node.children[i] is not None:
        child = node.children[i]
        char = chr(i + ord('a'))
        # process child

# HashMap: Only iterate existing children
for char, child in node.children.items():
    # process child  ← cleaner and faster for sparse nodes
```

---

## Handling Multiple Character Sets

HashMap naturally supports any character:

```python
class FlexibleTrie:
    """Supports mixed case, digits, special characters."""
    
    def __init__(self):
        self.root = TrieNode()  # HashMap-based
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

# Works with any string:
trie = FlexibleTrie()
trie.insert("Hello World!")      # Mixed case + space + punctuation
trie.insert("user@email.com")    # Special characters
trie.insert("日本語")              # Unicode
trie.insert("key123")            # Alphanumeric
```

With array-based, you'd need:
```python
# Awkward and error-prone for mixed character sets:
children = [None] * 128   # ASCII
# or
children = [None] * 65536  # Unicode BMP — enormous waste!
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Child storage | `dict` / `HashMap` / `unordered_map` |
| Access time | O(1) amortized (hash lookup) |
| Space per node | Base overhead + actual children only |
| Best for | Sparse tries, large/dynamic alphabets |
| Iteration | O(K) where K = actual children count |
| Flexibility | Any character including Unicode |
| Python shortcut | `defaultdict(TrieNode)` for auto-creation |
| Trade-off | Higher per-entry cost but no wasted slots |

---

## Quick Revision Questions

1. **What is the main advantage of HashMap-based tries over array-based?**
   > Memory efficiency for sparse tries — only stores children that exist. Also supports arbitrary character sets without pre-defining alphabet size.

2. **What's the risk of using `defaultdict(TrieNode)` in Python?**
   > Accessing `node.children[ch]` creates a new node even during search! Always use `ch in node.children` to check existence without creating.

3. **How do you iterate children in sorted order with a HashMap?**
   > Use `sorted(node.children.items())` in Python or `TreeMap` instead of `HashMap` in Java. Array-based gives sorted order naturally.

4. **When would HashMap-based be worse than array-based?**
   > For dense tries where most nodes have many children (>5). The per-entry overhead of HashMap exceeds the fixed array cost.

5. **How does HashMap handle the delete operation differently?**
   > Use `del node.children[ch]` / `node.children.remove(ch)` to remove the entry. Check `len(node.children) == 0` to see if node is empty (vs scanning all 26 slots).

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Array-Based Children](01-array-based-children.md) | [Next: Handling End of Word ▶](03-handling-end-of-word.md) |
