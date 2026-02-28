# Implement Magic Dictionary (LeetCode 676)

## Overview
Design a data structure that supports two operations: **buildDict** (initialize with a word list) and **search** (check if a word can be formed by changing exactly one character of any word in the dictionary). The trie enables efficient character-by-character exploration with a "mismatch budget" of exactly 1.

---

## Problem Statement

```
  buildDict(["hello", "leetcode"])
  
  search("hello")  → false  (can't change 0 chars — must change exactly 1)
  search("hhllo")  → true   (change 'h' → 'e' gives "hello")
  search("hell")   → false  (different length)
  search("leetcoded") → false
```

---

## Approach: Trie + DFS with Mismatch Counter

```
  For search(word):
  Walk the trie character by character.
  At each position, try ALL 26 children:
    - If child matches word[i]: continue with mismatches unchanged
    - If child differs: continue with mismatches + 1
  At the end: return true if mismatches == 1 AND node.is_end
  
  Prune: if mismatches > 1, stop exploring.
```

---

## Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class MagicDictionary:
    def __init__(self):
        self.root = TrieNode()
    
    def buildDict(self, dictionary):
        for word in dictionary:
            node = self.root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
    
    def search(self, searchWord):
        return self._dfs(self.root, searchWord, 0, 0)
    
    def _dfs(self, node, word, index, mismatches):
        if mismatches > 1:
            return False
        
        if index == len(word):
            return mismatches == 1 and node.is_end
        
        for ch, child in node.children.items():
            new_mismatches = mismatches + (0 if ch == word[index] else 1)
            if self._dfs(child, word, index + 1, new_mismatches):
                return True
        
        return False
```

### Java

```java
class MagicDictionary {
    int[][] trie = new int[10001][26];
    boolean[] isEnd = new boolean[10001];
    int idx = 0;
    
    public MagicDictionary() {
        Arrays.fill(trie[0], -1);
    }
    
    public void buildDict(String[] dictionary) {
        for (String word : dictionary) {
            int node = 0;
            for (char ch : word.toCharArray()) {
                int c = ch - 'a';
                if (trie[node][c] == -1) {
                    trie[node][c] = ++idx;
                    Arrays.fill(trie[idx], -1);
                }
                node = trie[node][c];
            }
            isEnd[node] = true;
        }
    }
    
    public boolean search(String searchWord) {
        return dfs(0, searchWord, 0, 0);
    }
    
    private boolean dfs(int node, String word, int index, int mismatches) {
        if (mismatches > 1) return false;
        if (index == word.length()) return mismatches == 1 && isEnd[node];
        
        for (int c = 0; c < 26; c++) {
            if (trie[node][c] == -1) continue;
            int newMis = mismatches + (c == word.charAt(index) - 'a' ? 0 : 1);
            if (dfs(trie[node][c], word, index + 1, newMis))
                return true;
        }
        return false;
    }
}
```

### C++

```cpp
class MagicDictionary {
    struct TrieNode {
        TrieNode* children[26] = {};
        bool isEnd = false;
    };
    TrieNode* root = new TrieNode();
    
public:
    void buildDict(vector<string> dictionary) {
        for (auto& word : dictionary) {
            auto node = root;
            for (char ch : word) {
                int c = ch - 'a';
                if (!node->children[c])
                    node->children[c] = new TrieNode();
                node = node->children[c];
            }
            node->isEnd = true;
        }
    }
    
    bool search(string searchWord) {
        return dfs(root, searchWord, 0, 0);
    }
    
    bool dfs(TrieNode* node, string& word, int idx, int mis) {
        if (mis > 1) return false;
        if (idx == word.size()) return mis == 1 && node->isEnd;
        
        for (int c = 0; c < 26; c++) {
            if (!node->children[c]) continue;
            int newMis = mis + (c == word[idx] - 'a' ? 0 : 1);
            if (dfs(node->children[c], word, idx + 1, newMis))
                return true;
        }
        return false;
    }
};
```

---

## Trace

```
  Dictionary: ["hello", "leetcode"]
  
  Trie:
  root → h → e → l → l → o*
       → l → e → e → t → c → o → d → e*
  
  search("hhllo"):
  
  DFS(root, "hhllo", 0, mismatches=0):
    Try 'h': matches word[0]='h' → DFS(h_node, 1, 0)
      Try 'e': doesn't match word[1]='h' → DFS(e_node, 2, 1)
        Try 'l': matches word[2]='l' → DFS(l_node, 3, 1)
          Try 'l': matches word[3]='l' → DFS(l_node, 4, 1)
            Try 'o': matches word[4]='o' → DFS(o_node, 5, 1)
              index==5==len("hhllo"), mismatches==1, is_end==true
              → return TRUE ✓
    
  "hhllo" matches "hello" with 1 change (h→e at position 1)
  
  search("hello"):
  
  DFS(root, "hello", 0, 0):
    Try 'h': matches → (h, 1, 0)
      Try 'e': matches → (e, 2, 0)
        Try 'l': matches → (l, 3, 0)
          Try 'l': matches → (l, 4, 0)
            Try 'o': matches → (o, 5, 0)
              mismatches==0 ≠ 1 → false
    Try 'l': doesn't match 'h' → (l, 1, 1)
      Try 'e': matches 'e' → (e, 2, 1)
        Try 'e': doesn't match 'l' → mismatches=2 → pruned
        Try 't': doesn't match 'l' → mismatches=2 → pruned
      → false
  → return FALSE ✓ (exact match not allowed)
```

---

## Alternative: HashMap with Wildcard Patterns

```python
class MagicDictionary:
    def __init__(self):
        self.patterns = {}  # pattern → list of original words
    
    def buildDict(self, dictionary):
        for word in dictionary:
            for i in range(len(word)):
                pattern = word[:i] + '*' + word[i+1:]
                if pattern not in self.patterns:
                    self.patterns[pattern] = []
                self.patterns[pattern].append(word)
    
    def search(self, searchWord):
        for i in range(len(searchWord)):
            pattern = searchWord[:i] + '*' + searchWord[i+1:]
            if pattern in self.patterns:
                for word in self.patterns[pattern]:
                    if word != searchWord:
                        return True
        return False
```

```
  HashMap approach: Generate all 1-wildcard patterns.
  
  "hello" → ["*ello", "h*llo", "he*lo", "hel*o", "hell*"]
  
  Build: O(N × L²) for all patterns
  Search: O(L²) per query
  Space: O(N × L²)
  
  Trie is more space-efficient when words share prefixes.
```

---

## Complexity Analysis

| Operation | Trie | HashMap |
|-----------|:----:|:------:|
| Build | O(N × L) | O(N × L²) |
| Search | O(26^1 × L) = O(26L) | O(L²) |
| Space | O(N × L) | O(N × L²) |

Trie search explores up to 26 branches at one position (the mismatch) and follows one path at all other positions. Total: O(26 × L).

---

## Edge Cases

```
  1. Search word matches dictionary word exactly → false
     (must change EXACTLY one character)
  
  2. Search word differs at multiple positions → false
     (only one change allowed)
  
  3. Search word length differs from all dict words → false
     (different length paths don't exist in trie)
  
  4. Dictionary has words of different lengths → trie handles naturally
  
  5. Empty dictionary → all searches return false
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Can word be formed by changing exactly 1 char of a dict word? |
| Trie + DFS | Walk trie, try all 26 children, track mismatches |
| Pruning | Stop when mismatches > 1 |
| End condition | mismatches == 1 AND is_end == true |
| Time per search | O(26 × L) |
| LeetCode | 676 |

---

## Quick Revision Questions

1. **Why must we check `mismatches == 1` (not ≥ 1) at the end?**
   > The problem requires EXACTLY one character change. If mismatches == 0, the word is the same as a dictionary word (not allowed). If mismatches > 1, too many changes.

2. **How does the trie handle words of different lengths?**
   > Words of different lengths terminate at different depths. If searchWord has length L and a dictionary word has length M ≠ L, the DFS will either run out of trie path (too long) or not reach is_end (too short).

3. **What's the worst-case branching in the DFS?**
   > At the mismatch position, we try up to 25 wrong branches (plus 1 correct = 26 total). But only one position has a mismatch, so total work is O(25 × remaining_length + L) ≈ O(26L).

4. **Why is the HashMap approach O(L²) per search?**
   > For each of L positions, it creates a pattern string of length L (O(L) each), totaling O(L²) work. Plus dictionary lookup is O(L) for string hashing.

5. **Can this approach be extended to allow k mismatches?**
   > Yes — change the pruning condition to `mismatches > k` and the end condition to `mismatches == k`. But branching increases exponentially: O(26^k × L), making it practical only for small k.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Stream of Characters](04-stream-of-characters.md) | [Next: Map Sum Pairs ▶](06-map-sum-pairs.md) |
