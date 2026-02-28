# Insertion in a Trie

## Overview
Insertion is the most fundamental trie operation. It creates a path of nodes for each character in the word, then marks the final node as a complete word. This chapter covers the algorithm, step-by-step traces, and edge cases.

---

## Algorithm

```
FUNCTION insert(word):
    node = root
    FOR each character c in word:
        IF c NOT in node.children:
            node.children[c] = new TrieNode()    // create if missing
        node = node.children[c]                  // move down
    node.isEndOfWord = true                      // mark complete word
```

### Key Points
- Start at the root (always)
- For each character: create node if absent, then descend
- After all characters: mark the final node as end-of-word
- Never modifies existing nodes (only adds new ones or marks flags)

---

## Step-by-Step Trace

### Insert "cat", "car", "card", "dog" into an empty trie

**Step 1: Insert "cat"**

```
  Before:  (root)          After:   (root)
                                      │
                                      c       ← created
                                      │
                                      a       ← created
                                      │
                                      t*      ← created + marked end
```

| Char | Exists? | Action |
|------|:-------:|--------|
| 'c' | No | Create TrieNode, attach to root.children['c'] |
| 'a' | No | Create TrieNode, attach to c.children['a'] |
| 't' | No | Create TrieNode, attach to a.children['t'], mark isEnd=true |

**Step 2: Insert "car"**

```
                (root)               
                  │                  
                  c          ← exists, reuse
                  │                  
                  a          ← exists, reuse
                 / \                 
                t*   r*      ← 'r' created + marked end
```

| Char | Exists? | Action |
|------|:-------:|--------|
| 'c' | **Yes** | Move to existing node |
| 'a' | **Yes** | Move to existing node |
| 'r' | No | Create TrieNode, mark isEnd=true |

**Step 3: Insert "card"**

```
                (root)
                  │
                  c
                  │
                  a
                 / \
                t*   r*
                     │
                     d*      ← created + marked end
```

| Char | Exists? | Action |
|------|:-------:|--------|
| 'c' | Yes | Reuse |
| 'a' | Yes | Reuse |
| 'r' | **Yes** | Reuse (already exists from "car") |
| 'd' | No | Create TrieNode, mark isEnd=true |

**Step 4: Insert "dog"**

```
                (root)
               /     \
              c       d       ← 'd' created at root level
              │       │
              a       o       ← created
             / \      │
            t*   r*   g*      ← created + marked end
                 │
                 d*
```

---

## Detailed Internal State After All Insertions

```
  ┌──────────┬──────────────────────────┬──────────┐
  │   Node   │     children             │  isEnd   │
  ├──────────┼──────────────────────────┼──────────┤
  │  root    │ {'c': nodeC, 'd': nodeD} │  false   │
  │  nodeC   │ {'a': nodeA}             │  false   │
  │  nodeA   │ {'t': nodeT, 'r': nodeR} │  false   │
  │  nodeT   │ {}                       │  true    │ ← "cat"
  │  nodeR   │ {'d': nodeD2}            │  true    │ ← "car"
  │  nodeD2  │ {}                       │  true    │ ← "card"
  │  nodeD   │ {'o': nodeO}             │  false   │
  │  nodeO   │ {'g': nodeG}             │  false   │
  │  nodeG   │ {}                       │  true    │ ← "dog"
  └──────────┴──────────────────────────┴──────────┘
  
  Total nodes created: 9 (root + 8)
  Total characters stored: 3+3+4+3 = 13
  Savings from sharing: "ca" shared by cat/car/card = saved 4 chars
```

---

## Edge Cases

### 1. Inserting Duplicate Word
```
  Trie has: ["apple"]
  Insert "apple" again:
  
  → All nodes exist, just re-set isEndOfWord = true (no-op)
  → No new nodes created
  → SAFE: inserting duplicates is harmless
```

### 2. Inserting Prefix of Existing Word
```
  Trie has: ["apple"]
  Insert "app":
  
  a → p → p → l → e*    (existing)
              ↑
              mark isEnd = true here
  
  Before: a → p → p → l → e*
  After:  a → p → p* → l → e*
  
  No new nodes! Just flip a boolean.
```

### 3. Inserting Word That Extends Existing
```
  Trie has: ["app"]
  Insert "apple":
  
  a → p → p* (existing)
              │
              l ← NEW
              │
              e* ← NEW
  
  2 new nodes created to extend "app" to "apple"
```

### 4. Inserting Empty String
```
  Insert "":
  → Loop body never executes (word is empty)
  → root.isEndOfWord = true
  → The empty string is now a "word" in the trie
  
  This is valid but unusual. Handle if needed.
```

---

## Implementation

### Python
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
```

### Java
```java
class Trie {
    private TrieNode root = new TrieNode();
    
    public void insert(String word) {
        TrieNode node = root;
        for (char ch : word.toCharArray()) {
            int idx = ch - 'a';
            if (node.children[idx] == null)
                node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }
}
```

### C++
```cpp
void insert(string word) {
    TrieNode* node = root;
    for (char ch : word) {
        int idx = ch - 'a';
        if (!node->children[idx])
            node->children[idx] = new TrieNode();
        node = node->children[idx];
    }
    node->isEndOfWord = true;
}
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|:----------:|-------------|
| Time | **O(L)** | Visit each of L characters once |
| Space (new word) | **O(L)** | Create up to L new nodes |
| Space (shared prefix) | **O(L - P)** | Only create nodes after shared prefix of length P |
| Space (duplicate) | **O(1)** | No new nodes, just flip boolean |

---

## Summary Table

| Scenario | Nodes Created | Effect |
|----------|:------------:|--------|
| New unique word | L nodes | Full path created |
| Word sharing prefix P | L - P nodes | Only new suffix nodes |
| Duplicate word | 0 nodes | Just re-marks isEnd |
| Prefix of existing | 0 nodes | Just marks isEnd |
| Extension of existing | Extension length nodes | Extends the path |

---

## Quick Revision Questions

1. **What happens when inserting "app" into a trie already containing "apple"?**
   > No new nodes are created. The node for the second 'p' just gets isEndOfWord set to true.

2. **How many new nodes are created when inserting "card" into a trie containing "car"?**
   > 1 new node — only 'd' is new. "c", "a", "r" already exist.

3. **What is the time complexity of inserting a word of length L?**
   > O(L) — we visit exactly L nodes, doing O(1) work at each.

4. **Is inserting the same word twice harmful?**
   > No — it simply re-sets isEndOfWord to true on the existing end node. No duplication.

5. **What is the maximum space used by inserting N words each of length L?**
   > O(N × L) in the worst case (no shared prefixes). In practice, much less due to prefix sharing.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: When to Use Tries](../01-Trie-Fundamentals/06-when-to-use-tries.md) | [Next: Search (Exact Match) ▶](02-search-exact-match.md) |
