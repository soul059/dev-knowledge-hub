# Prefix Matching and Replace Words (LeetCode 648)

## Overview
Prefix matching involves finding the shortest prefix in a dictionary that matches the beginning of a given word. The classic problem is **Replace Words** (LeetCode 648): replace every word in a sentence with its shortest dictionary root (prefix).

---

## Problem Statement

```
  Input: dictionary = ["cat","bat","rat"]
         sentence = "the cattle was rattled by the battery"
  
  Output: "the cat was rat by the bat"
  
  "cattle" → root "cat"  (shortest prefix match)
  "rattled" → root "rat"
  "battery" → root "bat"
  "the", "was", "by" → no root found, keep as-is
```

---

## Trie-Based Solution

### Key Insight

```
  Build trie from dictionary roots.
  For each word in sentence:
    Walk the trie character by character.
    If we encounter is_end=True → STOP, use this root.
    If we hit a dead end (char not in trie) → no match, keep original.
  
  This naturally finds the SHORTEST matching prefix!
  
  ┌──────────────────────────────────────────────────────┐
  │  Word: "cattle"                                       │
  │  Trie roots: cat, category                            │
  │                                                       │
  │  Walk: c → a → t (is_end!) → STOP                    │
  │  Root: "cat" (not "category" — shortest wins!)        │
  │                                                       │
  │  This is O(P) where P = shortest matching root length │
  └──────────────────────────────────────────────────────┘
```

### Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Solution:
    def replaceWords(self, dictionary, sentence):
        # Build trie from dictionary
        root = TrieNode()
        for word in dictionary:
            node = root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        
        # Replace each word in sentence
        words = sentence.split()
        result = []
        
        for word in words:
            replacement = self._find_root(root, word)
            result.append(replacement if replacement else word)
        
        return ' '.join(result)
    
    def _find_root(self, trie_root, word):
        """Find shortest prefix from trie matching this word."""
        node = trie_root
        for i, ch in enumerate(word):
            if node.is_end:
                return word[:i]  # Found shortest root!
            if ch not in node.children:
                return None  # No matching root
            node = node.children[ch]
        
        # Check if entire word is a root
        if node.is_end:
            return word
        return None
```

### Trace

```
  dictionary = ["cat", "bat", "rat"]
  sentence = "the cattle was rattled by the battery"
  
  Trie:
  root ── c ── a ── t*
       ├── b ── a ── t*
       └── r ── a ── t*
  
  Word: "the"
    t → found in trie
    h → NOT in t's children → no root → keep "the"
  
  Word: "cattle"
    c → found
    a → found
    t → found, is_end=True! → return "cat"
  
  Word: "was"
    w → NOT in root's children → no root → keep "was"
  
  Word: "rattled"
    r → found
    a → found
    t → found, is_end=True! → return "rat"
  
  Word: "battery"
    b → found
    a → found
    t → found, is_end=True! → return "bat"
  
  Result: "the cat was rat by the bat"
```

---

## Without Trie: HashSet Approach

```python
def replaceWords(self, dictionary, sentence):
    root_set = set(dictionary)
    words = sentence.split()
    result = []
    
    for word in words:
        replacement = word
        for i in range(1, len(word) + 1):
            prefix = word[:i]
            if prefix in root_set:
                replacement = prefix
                break  # Shortest first
        result.append(replacement)
    
    return ' '.join(result)
```

### Complexity Comparison

| Approach | Build | Per-Word | Total |
|----------|:-----:|:--------:|:-----:|
| Trie | O(D × L) | O(min(L, R)) | O(D×L + W×R) |
| HashSet | O(D × L) | O(L²) | O(D×L + W×L²) |

Where: D = dictionary size, L = word length, R = root length, W = words in sentence

```
  Trie is faster per word: O(R) vs O(L²)
  Because trie stops at first is_end, while HashSet
  checks all prefixes word[:1], word[:2], ... creating new strings!
```

---

## Variation: Multiple Matching Roots

```
  What if dictionary = ["ca", "cat", "category"]?
  Word: "cattle"
  
  Trie walk: c → a (is_end! for "ca") → STOP
  
  Root: "ca" (the SHORTEST one)
  
  The trie naturally gives the shortest because we check is_end
  at every step and return on first match.
```

---

## Variation: Longest Prefix Match

Sometimes you need the LONGEST matching prefix (e.g., IP routing):

```python
def find_longest_root(self, trie_root, word):
    """Find longest prefix from trie matching this word."""
    node = trie_root
    longest = None
    
    for i, ch in enumerate(word):
        if node.is_end:
            longest = word[:i]  # Record but DON'T stop
        if ch not in node.children:
            break
        node = node.children[ch]
    
    if node.is_end:
        longest = word[:len(word)]  # Check final position
    
    return longest
```

```
  dictionary = ["ca", "cat", "cata"]
  word = "catalogue"
  
  Walk: c → a (is_end, record "ca") → t (is_end, record "cat")
       → a (is_end, record "cata") → l (not found, break)
  
  Longest: "cata"
  Shortest: "ca"
```

---

## Edge Cases

```
  1. Empty dictionary → all words unchanged
  2. Root equals word → replace with itself (no change)
  3. Word shorter than any root → no match
  4. Multiple roots, different lengths → shortest wins
  5. Root is empty string → everything matches (unusual edge case)
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Replace words with shortest matching dictionary prefix |
| Algorithm | Build trie from roots; walk trie per word; stop at first is_end |
| Time | O(D×L + W×R) |
| Key insight | Trie naturally finds shortest prefix (early is_end check) |
| Longest variant | Don't stop at first is_end; remember last match |
| LeetCode | 648 — Replace Words |

---

## Quick Revision Questions

1. **Why does the trie naturally find the shortest prefix?**
   > We check `is_end` at every step while walking the trie. The first `is_end=True` encountered is the shortest dictionary root that matches the word's prefix.

2. **What's the time complexity advantage over HashSet?**
   > Trie: O(R) per word where R = shortest root length. HashSet: O(L²) per word because it generates all prefixes as substrings. For long words with short roots, trie is much faster.

3. **How would you find the longest matching prefix instead?**
   > Don't return on the first `is_end`. Instead, record each `is_end` position as a candidate and continue walking. Return the last recorded match after the walk ends.

4. **What if no dictionary root matches a word?**
   > The walk either hits a dead end (character not in trie) or completes without encountering `is_end`. In both cases, return the original word unchanged.

5. **Can there be overlapping roots like "ca" and "cat"?**
   > Yes. The trie handles this correctly — "ca" has `is_end=True` at depth 2, "cat" at depth 3. For shortest match, we return "ca" immediately when we reach it.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Words with Prefix](05-words-with-prefix.md) | [Next: Word Search in Grid ▶](../05-Word-Search-Problems/01-word-search-in-grid.md) |
