# Stream of Characters (LeetCode 1032)

## Overview
Design a data structure that accepts a stream of characters one at a time and checks if any suffix of the characters received so far matches a word in a given list. The key insight is to use a **reverse trie** — insert words backward and match the stream from the most recent character.

---

## Problem Statement

```
  words = ["cd", "f", "kl"]
  
  stream: a → false
  stream: b → false
  stream: c → false
  stream: d → true   (suffix "cd" matches)
  stream: e → false
  stream: f → true   (suffix "f" matches)
  stream: g → false
  stream: h → false
  stream: i → false
  stream: j → false
  stream: k → false
  stream: l → true   (suffix "kl" matches)
```

---

## Key Insight: Reverse Trie

```
  Problem: Check if ANY suffix of stream matches a dictionary word.
  
  Naive: For each new char, check all suffixes → O(stream_len × max_word_len)
  
  Better: Insert words REVERSED into trie.
         Keep a buffer of recent characters.
         On each new char, walk the reverse trie from the newest char backward.
  
  Why reversed? 
    Word "cd" reversed = "dc"
    Stream so far: [a, b, c, d]
    Walk backward: d → c → found "dc" in reverse trie → match!
```

---

## Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class StreamChecker:
    def __init__(self, words):
        self.root = TrieNode()
        self.stream = []
        self.max_len = 0
        
        # Insert each word REVERSED
        for word in words:
            self.max_len = max(self.max_len, len(word))
            node = self.root
            for ch in reversed(word):
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
    
    def query(self, letter):
        self.stream.append(letter)
        
        # Walk reverse trie matching stream backward
        node = self.root
        # Only check last max_len characters
        start = max(0, len(self.stream) - self.max_len)
        
        for i in range(len(self.stream) - 1, start - 1, -1):
            ch = self.stream[i]
            if ch not in node.children:
                return False
            node = node.children[ch]
            if node.is_end:
                return True
        
        return False
```

### Java

```java
class StreamChecker {
    int[][] trie;
    boolean[] isEnd;
    int idx = 0;
    StringBuilder stream = new StringBuilder();
    int maxLen = 0;
    
    public StreamChecker(String[] words) {
        int totalChars = 0;
        for (String w : words) totalChars += w.length();
        trie = new int[totalChars + 1][26];
        isEnd = new boolean[totalChars + 1];
        for (int[] row : trie) Arrays.fill(row, -1);
        
        for (String word : words) {
            maxLen = Math.max(maxLen, word.length());
            int node = 0;
            for (int i = word.length() - 1; i >= 0; i--) {
                int c = word.charAt(i) - 'a';
                if (trie[node][c] == -1) trie[node][c] = ++idx;
                node = trie[node][c];
            }
            isEnd[node] = true;
        }
    }
    
    public boolean query(char letter) {
        stream.append(letter);
        int node = 0;
        int start = Math.max(0, stream.length() - maxLen);
        
        for (int i = stream.length() - 1; i >= start; i--) {
            int c = stream.charAt(i) - 'a';
            if (trie[node][c] == -1) return false;
            node = trie[node][c];
            if (isEnd[node]) return true;
        }
        return false;
    }
}
```

---

## Trace

```
  words = ["cd", "f", "kl"]
  Reversed inserts: "dc", "f", "lk"
  
  Reverse Trie:
  root ── d ── c*
       ├── f*
       └── l ── k*
  
  Stream: []
  
  query('a'): stream=[a]
    Walk backward: a → not in root.children → false
  
  query('b'): stream=[a,b]
    Walk backward: b → not in root → false
  
  query('c'): stream=[a,b,c]
    Walk backward: c → not in root → false
    (root has d, f, l — not c)
  
  query('d'): stream=[a,b,c,d]
    Walk backward: d → in root ✓, isEnd? No
                   c → in d's children ✓, isEnd? Yes! → true
    ← suffix "cd" matched!
  
  query('e'): stream=[a,b,c,d,e]
    Walk backward: e → not in root → false
  
  query('f'): stream=[a,b,c,d,e,f]
    Walk backward: f → in root ✓, isEnd? Yes! → true
    ← suffix "f" matched!
  
  query('k'): stream=[...,k]
    Walk backward: k → not in root → false
  
  query('l'): stream=[...,k,l]
    Walk backward: l → in root ✓, isEnd? No
                   k → in l's children ✓, isEnd? Yes! → true
    ← suffix "kl" matched!
```

---

## Optimization: Deque with Max Length

```python
from collections import deque

class StreamChecker:
    def __init__(self, words):
        self.root = TrieNode()
        self.max_len = 0
        
        for word in words:
            self.max_len = max(self.max_len, len(word))
            node = self.root
            for ch in reversed(word):
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        
        # Only keep last max_len characters
        self.buffer = deque(maxlen=self.max_len)
    
    def query(self, letter):
        self.buffer.append(letter)
        
        node = self.root
        for ch in reversed(self.buffer):
            if ch not in node.children:
                return False
            node = node.children[ch]
            if node.is_end:
                return True
        
        return False
```

```
  Advantage: O(1) memory for stream (only keep max_len chars)
  vs O(total queries) if we store entire stream.
  
  max_len is typically small (≤ 200), so buffer is fixed size.
```

---

## Alternative: Forward Trie with Multiple Pointers

```
  Instead of reverse trie, use forward trie with active pointers:
  
  1. Build forward trie from words
  2. Maintain a SET of active trie nodes
  3. On each new char:
     a. For each active node, try to advance with new char
     b. Also start a NEW walk from root
     c. If any active node is at is_end → return true
     d. Remove dead-end nodes
  
  This is more complex and typically slower than reverse trie.
  Reverse trie is the standard approach.
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|:----:|:-----:|
| Constructor | O(W × L) | O(W × L) for trie |
| query() | O(L) per call | O(L) for buffer |
| N queries total | O(N × L) | O(W × L + L) |

Where W = number of words, L = max word length, N = number of queries.

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Check if any suffix of stream matches a word |
| Key trick | Insert words reversed into trie |
| Query | Walk backward from newest char through reverse trie |
| Buffer | Keep only last max_len characters |
| Per-query time | O(max_word_length) |
| LeetCode | 1032 |

---

## Quick Revision Questions

1. **Why insert words in reverse order into the trie?**
   > We need to match suffixes of the stream. By reversing words and walking the stream backward, the most recent characters align with the start of reversed words. This turns suffix matching into prefix matching.

2. **Why is `max_len` important for the buffer?**
   > No word is longer than `max_len`, so we never need to check more than the last `max_len` characters. This bounds per-query time and allows using a fixed-size deque.

3. **What happens if multiple words could match?**
   > We return `true` as soon as the first match is found (any `is_end` reached). All matching words share the reverse trie, so the walk naturally finds the shortest match first.

4. **Could you use Aho-Corasick instead?**
   > Yes, Aho-Corasick with its failure links provides a multi-pattern matching solution. But for this problem, the reverse trie is simpler and equally efficient since we only check suffixes.

5. **What's the space trade-off between storing the full stream vs. deque?**
   > Full stream: O(Q) space for Q queries. Deque(maxlen=L): O(L) space, constant regardless of query count. Since L is typically ≤ 200, the deque is far better for long streams.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Concatenated Words](03-concatenated-words.md) | [Next: Implement Magic Dictionary ▶](05-implement-magic-dictionary.md) |
