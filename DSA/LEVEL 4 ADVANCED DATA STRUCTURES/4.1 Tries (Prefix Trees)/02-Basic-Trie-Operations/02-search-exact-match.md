# Search (Exact Match)

## Overview
Search in a trie checks whether a **complete word** exists. The key distinction is that the word must be fully traversable AND the final node must have `isEndOfWord = true`. A word that exists merely as a prefix of another word is NOT found by search.

---

## Algorithm

```
FUNCTION search(word):
    node = root
    FOR each character c in word:
        IF c NOT in node.children:
            RETURN false              // path doesn't exist
        node = node.children[c]       // move down
    RETURN node.isEndOfWord           // must be a complete word
```

---

## Step-by-Step Traces

### Trie Contents: ["cat", "car", "card"]

```
                (root)
                  │
                  c
                  │
                  a
                 / \
                t*   r*
                     │
                     d*
```

### Search "car" → `true` ✅

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'c' | ✅ | Move to node_c |
| 2 | 'a' | ✅ | Move to node_a |
| 3 | 'r' | ✅ | Move to node_r |
| End | — | — | node_r.isEndOfWord = **true** → **FOUND** |

### Search "ca" → `false` ❌

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'c' | ✅ | Move to node_c |
| 2 | 'a' | ✅ | Move to node_a |
| End | — | — | node_a.isEndOfWord = **false** → **NOT FOUND** |

> "ca" is a valid PREFIX but NOT a stored word!

### Search "card" → `true` ✅

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'c' | ✅ | Move to node_c |
| 2 | 'a' | ✅ | Move to node_a |
| 3 | 'r' | ✅ | Move to node_r |
| 4 | 'd' | ✅ | Move to node_d |
| End | — | — | node_d.isEndOfWord = **true** → **FOUND** |

### Search "can" → `false` ❌

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'c' | ✅ | Move to node_c |
| 2 | 'a' | ✅ | Move to node_a |
| 3 | 'n' | ❌ | 'n' not in children → **NOT FOUND** (early exit) |

### Search "dog" → `false` ❌

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'd' | ❌ | 'd' not in root.children → **NOT FOUND** (immediate exit) |

---

## Two Failure Modes

```
  ╔════════════════════════════════════════════════════════════╗
  ║  FAILURE MODE 1: Path breaks (character not found)        ║
  ║                                                           ║
  ║  search("can") in trie ["cat", "car"]                     ║
  ║  c ✓ → a ✓ → n ✗  (no child 'n' at node 'a')            ║
  ║  Return: false                                            ║
  ╠════════════════════════════════════════════════════════════╣
  ║  FAILURE MODE 2: Path exists but not end-of-word          ║
  ║                                                           ║
  ║  search("ca") in trie ["cat", "car"]                      ║
  ║  c ✓ → a ✓ → isEndOfWord = false                         ║
  ║  Return: false                                            ║
  ║                                                           ║
  ║  "ca" is a PREFIX, not a WORD!                            ║
  ╚════════════════════════════════════════════════════════════╝
```

---

## Search vs StartsWith — Critical Distinction

```
  Trie: ["apple", "app"]

       (root)
         │
         a
         │
         p
         │
         p*        ← "app" is a word (isEnd = true)
         │
         l
         │
         e*        ← "apple" is a word (isEnd = true)

  ┌─────────────────────────────────────────────────────────┐
  │  Query               │ search() │ startsWith() │ Why   │
  ├───────────────────────┼──────────┼──────────────┼───────┤
  │  "apple"              │  true    │  true        │ word  │
  │  "app"                │  true    │  true        │ word  │
  │  "appl"               │  false   │  true        │ prefix│
  │  "ap"                 │  false   │  true        │ prefix│
  │  "applex"             │  false   │  false       │ no 'x'│
  └─────────────────────────────────────────────────────────┘
  
  search:     path must exist AND isEndOfWord = true
  startsWith: path must exist (that's all)
```

---

## Implementation

### Python
```python
def search(self, word):
    node = self.root
    for ch in word:
        if ch not in node.children:
            return False
        node = node.children[ch]
    return node.is_end
```

### Java
```java
public boolean search(String word) {
    TrieNode node = root;
    for (char ch : word.toCharArray()) {
        int idx = ch - 'a';
        if (node.children[idx] == null)
            return false;
        node = node.children[idx];
    }
    return node.isEndOfWord;
}
```

### C++
```cpp
bool search(string word) {
    TrieNode* node = root;
    for (char ch : word) {
        int idx = ch - 'a';
        if (!node->children[idx]) return false;
        node = node->children[idx];
    }
    return node->isEndOfWord;
}
```

---

## Helper: Find Node (Reusable Pattern)

Both `search` and `startsWith` navigate the same path. Extract a helper:

```python
def _find_node(self, s):
    """Navigate to the node at end of string s. Return None if path breaks."""
    node = self.root
    for ch in s:
        if ch not in node.children:
            return None
        node = node.children[ch]
    return node

def search(self, word):
    node = self._find_node(word)
    return node is not None and node.is_end

def starts_with(self, prefix):
    return self._find_node(prefix) is not None
```

This avoids code duplication — a clean interview pattern.

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|:----------:|-------------|
| Time (word found) | **O(L)** | Visit L nodes |
| Time (early exit) | **O(K)** | K = position where path breaks (K ≤ L) |
| Space | **O(1)** | No new memory allocated |

---

## Summary Table

| Scenario | Result | Reason |
|----------|:------:|--------|
| Word exists and marked end | `true` | Complete word found |
| Path exists but not end | `false` | It's just a prefix |
| Path breaks midway | `false` | Character not in trie |
| Empty string in trie | Depends | true only if root.isEnd = true |

---

## Quick Revision Questions

1. **Why does `search("ca")` return false when "cat" is in the trie?**
   > Because the node for 'a' has `isEndOfWord = false`. "ca" is a prefix, not a stored word.

2. **What are the two failure modes of trie search?**
   > Path breaks (character not found in children) or path completes but isEndOfWord is false.

3. **How does the `_find_node` helper simplify implementation?**
   > It extracts the common path-navigation logic. `search` calls it and checks `isEnd`; `startsWith` calls it and just checks non-null.

4. **What is the space complexity of a search operation?**
   > O(1) — search doesn't allocate any new nodes; it only reads existing ones.

5. **If we search for a word longer than any word in the trie, what happens?**
   > The search fails early when it reaches a node that doesn't have the needed child character. Returns false.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Insertion](01-insertion.md) | [Next: Prefix Search ▶](03-prefix-search.md) |
