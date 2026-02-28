# Palindrome Pairs (LeetCode 336)

## Overview
Given a list of unique words, find all pairs `(i, j)` such that the concatenation `words[i] + words[j]` is a palindrome. The trie approach achieves **O(N × L²)** time, where L is the average word length.

---

## Problem Statement

```
  Input: words = ["abcd", "dcba", "lls", "s", "sssll"]
  Output: [[0,1], [1,0], [3,2], [2,4]]
  
  Explanations:
  "abcd" + "dcba" = "abcddcba" → palindrome
  "dcba" + "abcd" = "dcbaabcd" → palindrome
  "s" + "lls"     = "slls"     → palindrome
  "lls" + "sssll" = "llssssll" → palindrome
```

---

## Key Insight

```
  For words[i] + words[j] to be a palindrome, one of these must hold:
  
  Case 1: words[j] is the reverse of words[i]
          "abcd" + "dcba" → palindrome
  
  Case 2: words[i] is longer. Its prefix matches reverse of words[j],
          and the remaining suffix of words[i] is itself a palindrome.
          words[i] = "lls", words[j] = "s"
          "s" reversed = "s", which matches prefix "s" of "lls"
          remaining "ll" is palindrome → "s" + "lls" = "slls" ✓
  
  Case 3: words[j] is longer (symmetric to Case 2).
```

---

## Approach: Reverse Trie

```
  1. Insert all words IN REVERSE into a trie
  2. At each trie node, store:
     - index: if a reversed word ends here
     - palindrome_below: indices of words whose remaining 
       suffix (after this node) forms a palindrome
  3. For each word, walk forward through the trie
     and check for matches
```

---

## Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word_idx = -1         # Index if a reversed word ends here
        self.palindrome_below = [] # Words below with palindromic remainder

class Solution:
    def palindromePairs(self, words):
        root = TrieNode()
        
        # Insert all words in REVERSE
        for idx, word in enumerate(words):
            node = root
            reversed_word = word[::-1]
            
            for i, ch in enumerate(reversed_word):
                # Check if remaining part of reversed_word is palindrome
                if self._is_palindrome(reversed_word, i, len(reversed_word) - 1):
                    node.palindrome_below.append(idx)
                
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            
            node.word_idx = idx
        
        # Search for palindrome pairs
        result = []
        for idx, word in enumerate(words):
            node = root
            
            for i, ch in enumerate(word):
                # Case 2: word is longer than a reversed word in trie
                # Reversed word ended at this node, remaining word is palindrome
                if node.word_idx >= 0 and node.word_idx != idx:
                    if self._is_palindrome(word, i, len(word) - 1):
                        result.append([idx, node.word_idx])
                
                if ch not in node.children:
                    node = None
                    break
                node = node.children[ch]
            
            if node is None:
                continue
            
            # Case 1: Exact reverse match (same length)
            if node.word_idx >= 0 and node.word_idx != idx:
                result.append([idx, node.word_idx])
            
            # Case 3: Some reversed words below have palindromic remainder
            for j in node.palindrome_below:
                if j != idx:
                    result.append([idx, j])
        
        return result
    
    def _is_palindrome(self, s, left, right):
        while left < right:
            if s[left] != s[right]:
                return False
            left += 1
            right -= 1
        return True
```

---

## Trace

```
  words = ["abcd", "dcba", "lls", "s", "sssll"]
  
  Insert reversed words into trie:
  "abcd"[::-1] = "dcba" → insert "dcba", idx=0
  "dcba"[::-1] = "abcd" → insert "abcd", idx=1
  "lls"[::-1]  = "sll"  → insert "sll", idx=2
  "s"[::-1]    = "s"    → insert "s", idx=3
  "sssll"[::-1]= "llsss"→ insert "llsss", idx=4
  
  During insert of "sll" (idx=2):
    At position 0 ('s'): remaining "ll" → palindrome? "ll" YES
    → root.palindrome_below = [2]
  
  During insert of "llsss" (idx=4):
    At position 0 ('l'): remaining "lsss" → palindrome? NO
    At position 1 ('l'): remaining "sss" → palindrome? YES
    → node_l.palindrome_below = [4]
  
  Query word "lls" (idx=2):
    Walk: l → l → s
    At 'l' (first), node.word_idx = -1
    At 'l' (second), node.word_idx = -1
    At 's', node has child? trie has "sll", "s", "llsss"...
    → After walking "lls" in reverse trie:
      At final node: check palindrome_below → [4]
      result: [2, 4] → "lls" + "sssll" = "llssssll" ✓
  
  Query word "s" (idx=3):
    Walk: s
    → remaining path check: root.palindrome_below has [2]
    At 's': node.word_idx? "sll" hasn't ended yet at 's'... 
    But "s" reversed = "s" which ends at first 's' node
    → node.word_idx = 3 (self, skip)
    → node.palindrome_below: check for reversed words below
    → Finds "sll" (idx=2) below: remaining "ll" is palindrome
    → result: [3, 2] → "s" + "lls" = "slls" ✓
```

---

## Alternative: Hash Map Approach

```python
def palindromePairs(self, words):
    word_map = {w: i for i, w in enumerate(words)}
    result = []
    
    for i, word in enumerate(words):
        for j in range(len(word) + 1):
            prefix = word[:j]
            suffix = word[j:]
            
            # Case: prefix is palindrome, reverse of suffix exists
            if self._is_palindrome(prefix, 0, len(prefix)-1):
                rev_suffix = suffix[::-1]
                if rev_suffix in word_map and word_map[rev_suffix] != i:
                    result.append([word_map[rev_suffix], i])
            
            # Case: suffix is palindrome, reverse of prefix exists
            if j < len(word) and self._is_palindrome(suffix, 0, len(suffix)-1):
                rev_prefix = prefix[::-1]
                if rev_prefix in word_map and word_map[rev_prefix] != i:
                    result.append([i, word_map[rev_prefix]])
    
    return result
```

```
  Hash approach: O(N × L²) — same asymptotic but simpler
  Trie approach: Better constant factors when many words share prefixes
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute Force | O(N² × L) | O(L) |
| Trie | O(N × L²) | O(N × L) |
| Hash Map | O(N × L²) | O(N × L) |

The L² factor comes from palindrome checks at each split point.

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Find all (i,j) where words[i]+words[j] is palindrome |
| Trie approach | Insert reversed words; walk forward words |
| Three cases | Exact reverse, longer word, shorter word |
| palindrome_below | Track which words have palindromic remainders |
| Time | O(N × L²) |
| LeetCode | 336 |

---

## Quick Revision Questions

1. **Why do we insert words in reverse into the trie?**
   > When we walk a word forward through the reverse-trie, matching characters verify that corresponding ends of the two words mirror each other — which is exactly the palindrome condition.

2. **What are the three cases for palindrome pairs?**
   > (1) words[j] is the exact reverse of words[i]. (2) words[i] has a palindromic suffix and its prefix matches reverse of words[j]. (3) words[j] has a palindromic suffix (handled by palindrome_below in trie).

3. **What does the `palindrome_below` list store?**
   > Indices of reversed words where the remaining characters (below this node in the trie) form a palindrome. This handles the case where one word is longer than the other.

4. **How do you avoid duplicate pairs?**
   > Case 1 (exact reverse) naturally produces (i,j) and (j,i) separately. The palindrome check at split boundaries avoids double-counting in Cases 2/3. Check `word_idx != idx` to avoid pairing with self.

5. **When is the trie approach better than hash map?**
   > When many words share common prefixes/suffixes and the trie can short-circuit walks early. For random strings, hash map is simpler and equally fast.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Binary Trie Concept](../06-Trie-With-Bitwise/05-binary-trie-concept.md) | [Next: Word Squares ▶](02-word-squares.md) |
