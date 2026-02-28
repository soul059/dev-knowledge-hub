# Concatenated Words (LeetCode 472)

## Overview
Given a list of words, find all words that can be formed by concatenating at least two shorter words from the same list. The trie enables efficient prefix matching combined with dynamic programming or recursive search.

---

## Problem Statement

```
  Input: words = ["cat","cats","catsdogcats","dog",
                   "dogcatsdog","hippopotamuses","rat",
                   "ratcatdogcat"]
  
  Output: ["catsdogcats","dogcatsdog","ratcatdogcat"]
  
  "catsdogcats" = "cats" + "dog" + "cats"
  "dogcatsdog"  = "dog" + "cats" + "dog"
  "ratcatdogcat"= "rat" + "cat" + "dog" + "cat"
```

---

## Approach 1: Trie + DFS

```
  1. Sort words by length (shorter first)
  2. For each word, check if it can be formed by concatenating
     words already in trie
  3. If not concatenatable, insert into trie
  4. If concatenatable, add to result (and optionally insert)
  
  Key: checking first means we only look at SHORTER words.
```

### Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Solution:
    def findAllConcatenatedWordsInADict(self, words):
        root = TrieNode()
        
        def insert(word):
            node = root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        
        def is_concatenated(word, start, count):
            """Can word[start:] be formed by concatenating trie words?"""
            if start == len(word):
                return count >= 2  # Need at least 2 words
            
            node = root
            for i in range(start, len(word)):
                ch = word[i]
                if ch not in node.children:
                    return False
                node = node.children[ch]
                
                if node.is_end:
                    # Found a complete word ending at position i
                    # Try to concatenate from position i+1
                    if is_concatenated(word, i + 1, count + 1):
                        return True
            
            return False
        
        # Sort by length — process shorter words first
        words.sort(key=len)
        result = []
        
        for word in words:
            if not word:
                continue
            if is_concatenated(word, 0, 0):
                result.append(word)
            insert(word)  # Insert whether concatenated or not
        
        return result
```

---

## Trace

```
  Sorted: ["cat","dog","rat","cats","catsdogcats",
           "dogcatsdog","ratcatdogcat","hippopotamuses"]
  
  Process "cat": trie empty → not concatenated → insert
  Process "dog": trie has {cat} → no match → insert
  Process "rat": trie has {cat,dog} → no match → insert
  Process "cats": 
    start=0: walk c→a→t→s
      at 't', is_end=True (cat)! Try from start=3:
        start=3: walk 's' → not in trie → False
      continue walking: 's' → not is_end → False
    Not concatenated → insert
  
  Process "catsdogcats":
    start=0: walk c→a→t
      't' is_end! Try start=3:
        start=3: walk s→d→o→g
          at 'd', trie has no 'd' from root → hmm
        Actually walk from root: s → not in trie → False
      continue: →s at position 3
        's' is_end(cats)! Try start=4:
          start=4: walk d→o→g
            'g' is_end! Try start=7:
              start=7: walk c→a→t
                't' is_end! Try start=10:
                  start=10: walk 's' → not in root → False
                continue: →s
                  's' is_end! Try start=11:
                    start=11 == len → count=4 ≥ 2 → TRUE!
    
    Result: "catsdogcats" IS concatenated ✓
    → add to result
```

---

## Approach 2: Trie + Memoized DFS

```python
class Solution:
    def findAllConcatenatedWordsInADict(self, words):
        root = TrieNode()
        
        # Insert ALL words first
        for word in words:
            if word:
                node = root
                for ch in word:
                    if ch not in node.children:
                        node.children[ch] = TrieNode()
                    node = node.children[ch]
                node.is_end = True
        
        def can_form(word, start, memo):
            if start == len(word):
                return True
            if start in memo:
                return memo[start]
            
            node = root
            for i in range(start, len(word)):
                ch = word[i]
                if ch not in node.children:
                    break
                node = node.children[ch]
                
                if node.is_end and i + 1 != len(word):  # Don't match whole word
                    if can_form(word, i + 1, memo):
                        memo[start] = True
                        return True
            
            memo[start] = False
            return False
        
        result = []
        for word in words:
            if word and can_form(word, 0, {}):
                result.append(word)
        
        return result
```

Note: The condition `i + 1 != len(word)` when `start == 0` prevents the word from matching itself. A cleaner approach is to temporarily remove the word from the trie before checking.

---

## Approach 3: DP (Non-Trie Alternative)

```python
def findAllConcatenatedWordsInADict(self, words):
    word_set = set(words)
    result = []
    
    for word in words:
        if not word:
            continue
        n = len(word)
        dp = [False] * (n + 1)
        dp[0] = True
        
        for i in range(1, n + 1):
            for j in range(0 if i < n else 1, i):
                # j > 0 or i < n ensures at least 2 parts
                if dp[j] and word[j:i] in word_set:
                    dp[i] = True
                    break
        
        if dp[n]:
            result.append(word)
    
    return result
```

---

## Comparison of Approaches

| Approach | Time | Space | Best When |
|----------|:----:|:-----:|-----------|
| Trie + DFS (sorted) | O(N × L²) | O(Total chars) | Many shared prefixes |
| Trie + Memo DFS | O(N × L²) | O(Total chars + L) | Repeated sub-problems |
| DP + HashSet | O(N × L²) | O(N × L) | Simple implementation |

---

## Edge Cases

```
  1. Empty string in words: skip it (can't be concatenated)
  2. Word = itself: "cat" alone is not concatenated (need ≥ 2 parts)
  3. Same word used twice: "aa" from ["a", "aa"] → "a" + "a" ✓
  4. All words are single chars: "abc" = "a"+"b"+"c" ✓
  5. No concatenated words exist: return empty list
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Find words that are concatenation of ≥2 other words |
| Trie role | Efficient multi-word prefix matching |
| Sort trick | Process short words first — only check against shorter |
| Recursive check | At each is_end, branch: continue matching or try new word |
| count ≥ 2 | Ensures at least 2 component words |
| LeetCode | 472 |

---

## Quick Revision Questions

1. **Why sort words by length before processing?**
   > So when checking if a word is concatenated, only shorter words are in the trie. This ensures we don't match a word against itself and that components are strictly shorter.

2. **What does `count >= 2` ensure?**
   > That the word is made of at least 2 other words. Without this check, every word in the dictionary would match itself (count = 1).

3. **How does memoization help in the DFS approach?**
   > When checking a long word, the same suffix may be reachable via different split points. Memoizing `can_form(start)` avoids redundant work, reducing time from exponential to O(L²).

4. **Can a word use the same component word multiple times?**
   > Yes. "aaa" can be formed from ["a", "aaa"] as "a" + "a" + "a". The trie search naturally allows revisiting the same word.

5. **What's the advantage of the trie approach over DP + HashSet?**
   > Trie shares prefix storage across all words and can walk character by character, identifying all matching prefixes in a single traversal. HashSet requires creating substrings for each potential split.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Word Squares](02-word-squares.md) | [Next: Stream of Characters ▶](04-stream-of-characters.md) |
