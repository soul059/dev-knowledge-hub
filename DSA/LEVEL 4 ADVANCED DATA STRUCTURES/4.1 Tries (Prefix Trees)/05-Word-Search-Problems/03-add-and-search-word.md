# Add and Search Word (LeetCode 211)

## Overview
Design a data structure that supports adding words and searching with wildcards. The dot '.' can match any letter. This is a classic trie problem that introduces recursive/backtracking search within the trie itself.

---

## Problem Statement

```
  WordDictionary dict = new WordDictionary();
  dict.addWord("bad");
  dict.addWord("dad");
  dict.addWord("mad");
  dict.search("pad");    → false
  dict.search("bad");    → true
  dict.search(".ad");    → true  (matches bad, dad, mad)
  dict.search("b..");    → true  (matches bad)
  dict.search("..d");    → true  (matches bad, dad, mad)
  dict.search("...");    → true
  dict.search("..");     → false (no 2-letter words)
```

---

## Key Insight

```
  Regular search: follow one path through the trie
  
  Wildcard search: '.' means TRY ALL children at this level
  
  For "b.d":
  b → follow 'b'
  . → try EVERY child of 'b'
  d → for each child, check if 'd' exists
  
  This is backtracking/DFS within the trie!
```

---

## Solution

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class WordDictionary:
    def __init__(self):
        self.root = TrieNode()
    
    def addWord(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
    
    def search(self, word: str) -> bool:
        return self._search_helper(self.root, word, 0)
    
    def _search_helper(self, node, word, idx):
        if idx == len(word):
            return node.is_end
        
        ch = word[idx]
        
        if ch == '.':
            # Wildcard: try ALL children
            for child in node.children.values():
                if self._search_helper(child, word, idx + 1):
                    return True
            return False
        else:
            # Regular character: follow specific path
            if ch not in node.children:
                return False
            return self._search_helper(node.children[ch], word, idx + 1)
```

---

## Step-by-Step Trace

```
  Words added: "bad", "dad", "mad"
  
  Trie:
  root ── b ── a ── d*
       ├── d ── a ── d*
       └── m ── a ── d*
  
  ── search(".ad") ──
  
  idx=0, ch='.', node=root
    Wildcard! Try all children:
    
    Try 'b': _search_helper(b_node, ".ad", 1)
      idx=1, ch='a', node=b_node
      'a' in children → follow
      idx=2, ch='d', node=a_node
      'd' in children → follow
      idx=3, idx==len → a_d_node.is_end = True → return True!
    
    Found! Return True immediately (short-circuit)
  
  ── search("b..") ──
  
  idx=0, ch='b', node=root
    'b' in children → follow b_node
  
  idx=1, ch='.', node=b_node
    Wildcard! Try all children of b_node:
    
    Try 'a': _search_helper(a_node, "b..", 2)
      idx=2, ch='.', node=a_node
      Wildcard! Try all children of a_node:
      
      Try 'd': _search_helper(d_node, "b..", 3)
        idx=3, idx==len → d_node.is_end = True → return True!
  
  ── search("b.") ──
  
  idx=0, ch='b' → follow
  idx=1, ch='.' → try all children of b_node:
    Try 'a': idx=2, idx==len → a_node.is_end = False → return False
  All children exhausted → return False
  
  (No 2-letter word starting with 'b')
```

---

## Complexity Analysis

```
  Without wildcards:
    Time: O(L) per search — same as standard trie
  
  With wildcards:
    Time: O(26^D × L) worst case
    Where D = number of dots in the pattern
    
    Example: "..." with 3 dots
    → Try 26 children at level 0
    → For each, try 26 at level 1
    → For each, try 26 at level 2
    → 26³ = 17,576 paths explored
    
  In practice: Much better because:
  - Most nodes don't have 26 children
  - Average branching factor is 3-5 for English
  - Early termination when path fails
```

| Operation | Time | Space |
|-----------|:----:|:-----:|
| addWord | O(L) | O(L) |
| search (no dots) | O(L) | O(L) stack |
| search (k dots) | O(A^k × L) | O(L) stack |
| A = avg children per node |

---

## Java Solution

```java
class WordDictionary {
    private TrieNode root;
    
    public WordDictionary() {
        root = new TrieNode();
    }
    
    public void addWord(String word) {
        TrieNode node = root;
        for (char ch : word.toCharArray()) {
            if (node.children[ch - 'a'] == null) {
                node.children[ch - 'a'] = new TrieNode();
            }
            node = node.children[ch - 'a'];
        }
        node.isEnd = true;
    }
    
    public boolean search(String word) {
        return dfs(root, word, 0);
    }
    
    private boolean dfs(TrieNode node, String word, int idx) {
        if (idx == word.length()) return node.isEnd;
        
        char ch = word.charAt(idx);
        
        if (ch == '.') {
            for (int i = 0; i < 26; i++) {
                if (node.children[i] != null && dfs(node.children[i], word, idx + 1)) {
                    return true;
                }
            }
            return false;
        } else {
            int ci = ch - 'a';
            if (node.children[ci] == null) return false;
            return dfs(node.children[ci], word, idx + 1);
        }
    }
}

class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd = false;
}
```

---

## Optimization: Group Words by Length

```python
class WordDictionary:
    def __init__(self):
        self.root = TrieNode()
        self.max_len = 0
    
    def addWord(self, word):
        self.max_len = max(self.max_len, len(word))
        # ... standard insert
    
    def search(self, word):
        if len(word) > self.max_len:
            return False  # Quick reject
        return self._search_helper(self.root, word, 0)
```

---

## Pattern: Multiple Wildcards

```
  search("..a..") — 2 fixed, 3 wildcards
  
  Level 0: '.' → try all children (branch out)
  Level 1: '.' → try all children (branch further)
  Level 2: 'a' → only follow 'a' (converge!)
  Level 3: '.' → branch again
  Level 4: '.' → branch again
  
  The fixed characters act as FILTERS that prune branches.
  More fixed characters → faster search.
  All wildcards "....." → slowest (try everything).
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Pattern | Trie + recursive backtracking for wildcards |
| '.' handling | Try all children at that level |
| Regular char | Follow specific child (standard trie) |
| Time without dots | O(L) — standard trie search |
| Time with dots | O(A^D × L) where D = dot count |
| LeetCode | 211 — Design Add and Search Words |

---

## Quick Revision Questions

1. **How does '.' (wildcard) change the search algorithm?**
   > Instead of following one specific child, '.' triggers trying ALL children at that level. If any child's subtree matches the remaining pattern, return True.

2. **What is the worst-case time complexity for search("...")?**
   > O(26^3 × 1) in theory if every node has 26 children. In practice, with English words, average branching is much lower (~3-5).

3. **Why do we need recursion for wildcard search?**
   > Because '.' creates multiple possible paths. We must explore each path independently and backtrack if it fails — this is naturally recursive.

4. **How does adding more fixed characters help performance?**
   > Fixed characters act as filters. At each fixed position, we follow only one child instead of all. More fixed chars = fewer branches explored.

5. **Can this approach handle multiple types of wildcards?**
   > Yes. You could extend it: '?' for single char (same as '.'), '*' for zero or more chars (requires more complex recursion with variable-length matching).

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Word Search II](02-word-search-ii.md) | [Next: Wildcard Matching ▶](04-wildcard-matching.md) |
