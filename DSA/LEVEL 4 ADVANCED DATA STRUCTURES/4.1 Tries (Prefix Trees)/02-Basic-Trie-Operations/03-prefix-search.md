# Prefix Search (startsWith)

## Overview
Prefix search checks if **any** word in the trie starts with the given prefix. Unlike exact search, it does NOT require `isEndOfWord = true` — it only verifies that the path exists in the trie.

---

## Algorithm

```
FUNCTION startsWith(prefix):
    node = root
    FOR each character c in prefix:
        IF c NOT in node.children:
            RETURN false         // path doesn't exist
        node = node.children[c]
    RETURN true                  // path exists → some word has this prefix
```

---

## Step-by-Step Traces

### Trie: ["apple", "app", "application"]

```
              (root)
                │
                a
                │
                p
                │
                p*          ← "app" ends here
                │
                l
               / \
              e*   i        ← "apple" ends at 'e'
                   │
                   c
                   │
                   a
                   │
                   t
                   │
                   i
                   │
                   o
                   │
                   n*       ← "application" ends here
```

### startsWith("app") → `true` ✅

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'a' | ✅ | Move to 'a' |
| 2 | 'p' | ✅ | Move to 'p' |
| 3 | 'p' | ✅ | Move to 'p' |
| End | — | — | Path exists → **true** |

### startsWith("appl") → `true` ✅

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1-3 | 'a','p','p' | ✅ | Navigate through |
| 4 | 'l' | ✅ | Move to 'l' |
| End | — | — | Path exists → **true** |

> Even though "appl" is not a word itself, the prefix path exists because "apple" and "application" continue from here.

### startsWith("appx") → `false` ❌

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1-3 | 'a','p','p' | ✅ | Navigate through |
| 4 | 'x' | ❌ | No child 'x' at second 'p' → **false** |

### startsWith("b") → `false` ❌

| Step | Char | Found? | Action |
|------|------|:------:|--------|
| 1 | 'b' | ❌ | No child 'b' at root → **false** (immediate) |

---

## Extended: Find All Words With Prefix

The most powerful prefix operation collects all words that start with a given prefix:

```
FUNCTION findAllWithPrefix(prefix):
    node = root
    
    // Phase 1: Navigate to end of prefix
    FOR each character c in prefix:
        IF c NOT in node.children:
            RETURN []                    // no words with this prefix
        node = node.children[c]
    
    // Phase 2: DFS to collect all words in subtree
    results = []
    dfs(node, prefix, results)
    RETURN results

FUNCTION dfs(node, currentWord, results):
    IF node.isEndOfWord:
        results.append(currentWord)
    FOR each (char, child) in sorted(node.children):
        dfs(child, currentWord + char, results)
```

### Trace: findAllWithPrefix("app")

```
  Phase 1: Navigate root → a → p → p ✅

  Phase 2: DFS from second 'p' node:
  
  ┌──────────────────────────────────────────────────┐
  │ Node 'p' (isEnd=true) → add "app" ✓             │
  │   └── child 'l':                                  │
  │       ├── Node 'e' (isEnd=true) → add "apple" ✓  │
  │       └── child 'i':                              │
  │           └── 'c' → 'a' → 't' → 'i' → 'o'       │
  │               └── 'n' (isEnd=true)                │
  │                   → add "application" ✓           │
  └──────────────────────────────────────────────────┘

  Result: ["app", "apple", "application"]
  (returned in lexicographic order because we sort children)
```

---

## Implementation

### Python
```python
def starts_with(self, prefix):
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return False
        node = node.children[ch]
    return True

def find_all_with_prefix(self, prefix):
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return []
        node = node.children[ch]
    
    results = []
    self._dfs(node, list(prefix), results)
    return results

def _dfs(self, node, path, results):
    if node.is_end:
        results.append(''.join(path))
    for ch in sorted(node.children):
        path.append(ch)
        self._dfs(node.children[ch], path, results)
        path.pop()
```

### Java
```java
public boolean startsWith(String prefix) {
    TrieNode node = root;
    for (char ch : prefix.toCharArray()) {
        int idx = ch - 'a';
        if (node.children[idx] == null) return false;
        node = node.children[idx];
    }
    return true;
}

public List<String> findAllWithPrefix(String prefix) {
    TrieNode node = root;
    for (char ch : prefix.toCharArray()) {
        int idx = ch - 'a';
        if (node.children[idx] == null) return new ArrayList<>();
        node = node.children[idx];
    }
    List<String> results = new ArrayList<>();
    dfs(node, new StringBuilder(prefix), results);
    return results;
}

private void dfs(TrieNode node, StringBuilder sb, List<String> results) {
    if (node.isEndOfWord) results.add(sb.toString());
    for (int i = 0; i < 26; i++) {
        if (node.children[i] != null) {
            sb.append((char) ('a' + i));
            dfs(node.children[i], sb, results);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|:----:|:-----:|
| `startsWith(prefix)` | **O(P)** | O(1) |
| `findAllWithPrefix(prefix)` | **O(P + K)** | O(K) for results |

> P = prefix length, K = total characters in all matching words

---

## Common Patterns Using Prefix Search

### Pattern 1: Count Words With Prefix
```python
# Use prefixCount field for O(P) counting
def count_with_prefix(self, prefix):
    node = self.root
    for ch in prefix:
        if ch not in node.children:
            return 0
        node = node.children[ch]
    return node.prefix_count    # maintained during insert
```

### Pattern 2: Autocomplete Top-K
```python
# Collect all, then sort by frequency/priority
def autocomplete(self, prefix, k=5):
    words = self.find_all_with_prefix(prefix)
    # Sort by some priority metric
    words.sort(key=lambda w: -self.get_frequency(w))
    return words[:k]
```

### Pattern 3: Check If Any Word Has Prefix
```python
# Already covered by startsWith — O(P)
def has_words_with_prefix(self, prefix):
    return self.starts_with(prefix)
```

---

## Summary Table

| Query | What it Checks | Returns |
|-------|---------------|---------|
| `startsWith("app")` | Path exists | boolean |
| `search("app")` | Path exists AND isEnd | boolean |
| `findAllWithPrefix("app")` | All words in subtree | list of strings |
| `countWithPrefix("app")` | Words passing through | integer count |

---

## Quick Revision Questions

1. **Why does `startsWith("appl")` return true even though "appl" isn't a word?**
   > Because `startsWith` only checks if the path exists. The path a→p→p→l exists since "apple" goes through it.

2. **What's the difference between `startsWith` and `search`?**
   > `startsWith` checks for path existence only. `search` additionally requires isEndOfWord = true at the end.

3. **What is the time complexity of `findAllWithPrefix`?**
   > O(P + K) — P to navigate to prefix, K to collect all words in the subtree (K = total chars in results).

4. **How can you count prefix matches in O(P) time?**
   > Maintain a `prefixCount` field on each node, incremented during insertion. Navigate to prefix end, return that count.

5. **What does `findAllWithPrefix` return for a non-existent prefix?**
   > An empty list — the navigation phase fails and returns immediately.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Search (Exact Match)](02-search-exact-match.md) | [Next: Deletion ▶](04-deletion.md) |
