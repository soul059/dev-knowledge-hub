# Word Squares (LeetCode 425)

## Overview
A **word square** is a sequence of words where the k-th row and k-th column read the same string. Given a set of words (all same length), find all word squares. The trie enables efficient **prefix lookup** to prune the backtracking search.

---

## Problem Statement

```
  Input: words = ["area", "lead", "wall", "lady", "ball"]
  Output: [["wall","area","lead","lady"],
           ["ball","area","lead","lady"]]
  
  Word Square verification for ["wall","area","lead","lady"]:
  
    w a l l       Column 0: w a l l = "wall" = Row 0 ✓
    a r e a       Column 1: a r e a = "area" = Row 1 ✓
    l e a d       Column 2: l e a d = "lead" = Row 2 ✓
    l a d y       Column 3: l a d y = "lady" = Row 3 ✓
```

---

## Key Insight

```
  When building a word square row by row:
  
  If we've placed rows 0..k-1, then row k must start with
  a specific PREFIX determined by column k of the existing rows.
  
  Example: After placing "wall" and "area":
    w a l l
    a r e a
    ? ? ? ?    ← Row 2 must start with "le" (column 2: l, e, ...)
    ? ? ? ?
  
  So we need: all words starting with "le" → ["lead"]
  
  This is exactly what a trie does efficiently!
```

---

## Algorithm

```
  1. Build trie from all words
     - At each node, store list of word indices passing through
  2. Backtrack: try each word as first row
  3. For row k:
     a. Compute required prefix from columns 0..k-1 of placed rows
     b. Search trie for all words with that prefix
     c. Try each candidate → recurse for row k+1
  4. If k == word_length, record complete word square
```

---

## Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word_indices = []  # All words passing through this node

class Solution:
    def wordSquares(self, words):
        n = len(words[0])  # All words same length
        
        # Build trie
        root = TrieNode()
        for idx, word in enumerate(words):
            node = root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
                node.word_indices.append(idx)
        
        def get_words_with_prefix(prefix):
            node = root
            for ch in prefix:
                if ch not in node.children:
                    return []
                node = node.children[ch]
            return node.word_indices
        
        results = []
        
        def backtrack(square):
            row = len(square)
            if row == n:
                results.append(list(square))
                return
            
            # Build required prefix for next row
            prefix = ''.join(square[i][row] for i in range(row))
            
            # Find all words with this prefix
            candidates = get_words_with_prefix(prefix)
            
            for idx in candidates:
                square.append(words[idx])
                backtrack(square)
                square.pop()
        
        # Try each word as first row
        for word in words:
            backtrack([word])
        
        return results
```

### Java

```java
class Solution {
    int[][] trie;
    List<Integer>[] wordIndices;
    int idx = 0;
    String[] words;
    int n;
    List<List<String>> results = new ArrayList<>();
    
    public List<List<String>> wordSquares(String[] words) {
        this.words = words;
        this.n = words[0].length();
        int maxNodes = words.length * n + 1;
        trie = new int[maxNodes][26];
        wordIndices = new ArrayList[maxNodes];
        for (int i = 0; i < maxNodes; i++) {
            Arrays.fill(trie[i], -1);
            wordIndices[i] = new ArrayList<>();
        }
        
        // Build trie
        for (int i = 0; i < words.length; i++) {
            int node = 0;
            for (char ch : words[i].toCharArray()) {
                int c = ch - 'a';
                if (trie[node][c] == -1) trie[node][c] = ++idx;
                node = trie[node][c];
                wordIndices[node].add(i);
            }
        }
        
        List<String> square = new ArrayList<>();
        for (String word : words) {
            square.add(word);
            backtrack(square);
            square.remove(square.size() - 1);
        }
        return results;
    }
    
    void backtrack(List<String> square) {
        int row = square.size();
        if (row == n) {
            results.add(new ArrayList<>(square));
            return;
        }
        
        StringBuilder prefix = new StringBuilder();
        for (int i = 0; i < row; i++) {
            prefix.append(square.get(i).charAt(row));
        }
        
        List<Integer> candidates = getWordsWithPrefix(prefix.toString());
        for (int idx : candidates) {
            square.add(words[idx]);
            backtrack(square);
            square.remove(square.size() - 1);
        }
    }
    
    List<Integer> getWordsWithPrefix(String prefix) {
        int node = 0;
        for (char ch : prefix.toCharArray()) {
            int c = ch - 'a';
            if (trie[node][c] == -1) return Collections.emptyList();
            node = trie[node][c];
        }
        return wordIndices[node];
    }
}
```

---

## Trace

```
  words = ["area", "lead", "wall", "lady", "ball"]
  n = 4
  
  Backtrack starting with "wall":
  
  Row 0: "wall"
    w a l l
    
  Row 1: prefix = column 1 of [wall] = "a"
    Words starting with "a": ["area"]
    Try "area":
      w a l l
      a r e a
  
  Row 2: prefix = column 2 of [wall, area] = "l" + "e" = "le"
    Words starting with "le": ["lead"]
    Try "lead":
      w a l l
      a r e a
      l e a d
  
  Row 3: prefix = column 3 of [wall, area, lead] = "l"+"a"+"d" = "lad"
    Words starting with "lad": ["lady"]
    Try "lady":
      w a l l
      a r e a
      l e a d
      l a d y
    
    Row 4: row == n → FOUND! Record ["wall","area","lead","lady"]
  
  Backtrack starting with "ball":
    Similar process → finds ["ball","area","lead","lady"]
```

---

## Why HashMap Alone Is Insufficient

```
  HashMap approach: store {prefix → [words]}
  
  Problem: Need to store ALL prefixes of ALL words.
  For words of length L with N words: O(N × L) entries.
  
  This works but trie is more natural:
  - Shared prefixes stored once
  - Prefix lookup is just a walk: O(P)
  - word_indices at each node is exactly the needed list
```

---

## Complexity Analysis

| Aspect | Complexity |
|--------|:----------:|
| Build trie | O(N × L) |
| Prefix query | O(P) where P = prefix length |
| Backtracking | O(N^L) worst case |
| With trie pruning | Significantly fewer branches |
| Space | O(N × L) for trie + word indices |

The dominant cost is the backtracking. Trie's role is to prune branches early by quickly finding valid candidates.

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Find all word squares from word list |
| Key property | Row k = Column k in word square |
| Trie role | Fast prefix lookup to find candidate rows |
| Algorithm | Backtracking with trie-pruned candidates |
| word_indices | Store word indices at each trie node |
| LeetCode | 425 |

---

## Quick Revision Questions

1. **How do you determine the required prefix for row k?**
   > Concatenate position k from each row placed so far: `prefix = square[0][k] + square[1][k] + ... + square[k-1][k]`.

2. **Why store word_indices at every trie node, not just leaves?**
   > We need all words that START with a given prefix, not just words that exactly match. Storing indices at every node along the path gives us this for any prefix length.

3. **What is the worst-case time complexity?**
   > O(N^L) where L is word length — each of L positions could have up to N candidates. In practice, trie pruning dramatically reduces this.

4. **Can duplicate words appear in a word square?**
   > Yes, if the same word appears multiple times in the input (the problem says "unique words" for LC 425, but the algorithm handles duplicates naturally through indices).

5. **How does the word square property constrain the search?**
   > Row k = Column k means placing each row constrains all subsequent rows. After placing row 0, column 0 is fixed, constraining row 1's first character, etc. This cascading constraint is what makes trie pruning effective.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Palindrome Pairs](01-palindrome-pairs.md) | [Next: Concatenated Words ▶](03-concatenated-words.md) |
