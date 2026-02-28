# Replace Words (LeetCode 648)

## Overview
The Replace Words problem asks you to replace every derivative (longer word) in a sentence with its root (shortest prefix from a dictionary). This chapter provides the full solution, optimization strategies, and related variations.

---

## Problem Statement

```
  dictionary = ["cat", "bat", "rat"]
  sentence = "the cattle was rattled by the battery"
  Output: "the cat was rat by the bat"
  
  dictionary = ["a", "b", "c"]
  sentence = "aadsfasf absbd bbab cadsfabd"
  Output: "a]a b c"
  
  dictionary = ["a", "aa", "aaa", "aaaa"]
  sentence = "a aa a aaaa aaa aaa aaa aaaaaa bbb baba ababa"
  Output: "a a a a a a a a bbb baba a"
```

---

## Solution: Trie-Based

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Solution:
    def replaceWords(self, dictionary, sentence):
        # Build trie from dictionary roots
        root = TrieNode()
        for word in dictionary:
            node = root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        
        # Process each word in sentence
        result = []
        for word in sentence.split():
            result.append(self._find_shortest_root(root, word))
        
        return ' '.join(result)
    
    def _find_shortest_root(self, root, word):
        """Find shortest dictionary root that is a prefix of word."""
        node = root
        for i, ch in enumerate(word):
            if node.is_end:
                return word[:i]  # Found shortest root!
            if ch not in node.children:
                return word  # No matching root
            node = node.children[ch]
        
        # Word itself might be a root
        return word
```

### Trace

```
  dictionary = ["cat", "bat", "rat"]
  
  Trie:
  root ── b ── a ── t*
       ├── c ── a ── t*
       └── r ── a ── t*
  
  Processing "cattle":
  c → found, isEnd? No
  a → found, isEnd? No
  t → found, isEnd? Yes! → return "cat"  (word[:3])
  
  Processing "rattled":
  r → found, isEnd? No
  a → found, isEnd? No
  t → found, isEnd? Yes! → return "rat"
  
  Processing "the":
  t → not in root.children (root has b, c, r) → return "the"
  
  Processing "battery":
  b → found, isEnd? No
  a → found, isEnd? No
  t → found, isEnd? Yes! → return "bat"
  
  Result: "the cat was rat by the bat"
```

---

## Optimization: Trie Pruning During Build

If we have both "a" and "ab" in dictionary, "ab" is never needed:

```python
def replaceWords(self, dictionary, sentence):
    # Sort by length — insert shorter roots first
    dictionary.sort(key=len)
    
    root = TrieNode()
    for word in dictionary:
        node = root
        skip = False
        for ch in word:
            if node.is_end:
                skip = True  # Shorter root already exists
                break
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        
        if not skip:
            node.is_end = True
            node.children.clear()  # Remove descendants!
```

```
  dictionary = ["a", "ab", "abc"]
  
  Insert "a": root → a*
  Insert "ab": at 'a' node, isEnd=True → skip! "a" covers "ab"
  Insert "abc": at 'a' node, isEnd=True → skip! "a" covers "abc"
  
  Trie only has: root → a*
  Much smaller and faster for lookups!
```

---

## Alternative: HashSet Approach

```python
def replaceWords(self, dictionary, sentence):
    root_set = set(dictionary)
    
    result = []
    for word in sentence.split():
        replacement = word
        for i in range(1, len(word) + 1):
            if word[:i] in root_set:
                replacement = word[:i]
                break
        result.append(replacement)
    
    return ' '.join(result)
```

### Comparison

| Aspect | Trie | HashSet |
|--------|:----:|:-------:|
| Build time | O(D × L) | O(D × L) |
| Per-word search | O(min(L, R)) | O(L²) substring creation |
| Space | O(D × L) | O(D × L) |
| Better when | Many words, long roots | Few short roots |

```
  HashSet creates word[:1], word[:2], ... each as a new string
  → O(L²) per word due to substring creation
  
  Trie walks character by character
  → O(R) per word where R = root length (usually << L)
```

---

## Related Variations

### Variation 1: Replace with Longest Root

```python
def _find_longest_root(self, root, word):
    node = root
    longest = None
    
    for i, ch in enumerate(word):
        if node.is_end:
            longest = word[:i]  # Record but continue
        if ch not in node.children:
            break
        node = node.children[ch]
    
    if node.is_end:
        longest = word  # Check final
    
    return longest if longest else word
```

### Variation 2: Replace Only If Root Is Different

```python
def _find_root_if_different(self, root, word):
    result = self._find_shortest_root(root, word)
    return result if result != word else word  # Only replace if shortened
```

### Variation 3: Count Replacements

```python
def replaceWords(self, dictionary, sentence):
    # ... build trie ...
    
    replacement_count = 0
    result = []
    
    for word in sentence.split():
        replacement = self._find_shortest_root(root, word)
        if replacement != word:
            replacement_count += 1
        result.append(replacement)
    
    return ' '.join(result), replacement_count
```

---

## Edge Cases

```
  1. Root equals word exactly:
     dictionary = ["cat"], word = "cat"
     → Return "cat" (unchanged, but technically replaced)
  
  2. Multiple roots for same prefix:
     dictionary = ["c", "ca", "cat"], word = "cattle"  
     → Return "c" (shortest root wins)
  
  3. Word has no matching root:
     dictionary = ["bat"], word = "cat"
     → Return "cat" unchanged
  
  4. Empty word or single char:
     dictionary = ["a"], word = "a"
     → Return "a"
  
  5. Root longer than word:
     dictionary = ["cattle"], word = "cat"
     → Return "cat" unchanged (root isn't a prefix of word)
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Algorithm | Build trie from roots; walk per word; stop at first is_end |
| Time | O(D×L + W×R) where R = avg root length |
| Space | O(D × L) for trie |
| Optimization | Sort roots by length; prune longer when shorter exists |
| Key insight | First is_end in walk = shortest root |
| LeetCode | 648 — Replace Words |

---

## Quick Revision Questions

1. **Why does the trie naturally find the shortest matching root?**
   > Because we check `is_end` at every step while walking down the trie. The first `is_end=True` hit is necessarily the shortest root prefix of the word.

2. **How does trie pruning during build optimize the solution?**
   > If a shorter root already covers a path, we skip inserting longer roots and clear their descendants. This reduces trie size and search time.

3. **What's the per-word complexity advantage of trie over HashSet?**
   > Trie: O(R) where R = shortest root length. HashSet: O(L²) because it creates new substring objects for `word[:1]`, `word[:2]`, etc. until a match is found.

4. **How do you handle words with no matching root?**
   > If the walk through the trie hits a dead end (character not found in children) without encountering `is_end=True`, return the original word unchanged.

5. **Can the same word have multiple matching roots?**
   > Yes. Dictionary ["c", "ca", "cat"] all match "cattle". The trie returns "c" — the shortest one — because it's the first `is_end` encountered during the walk.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Pattern Matching](05-pattern-matching.md) | [Next: XOR Problems with Trie ▶](../06-Trie-With-Bitwise/01-xor-problems-with-trie.md) |
