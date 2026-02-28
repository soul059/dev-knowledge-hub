# Pattern Matching with Tries

## Overview
This chapter covers advanced pattern matching techniques using tries, including matching patterns where words must follow a structural template (e.g., same pattern of character repetition) and prefix/suffix combined queries.

---

## Problem: Word Pattern Match

Given a list of words and a pattern, find words that follow the same letter repetition pattern.

```
  Pattern: "abba"
  Structure: 1st=4th, 2nd=3rd, 1st≠2nd
  
  Matches: "noon" (n=n, o=o), "deed" (d=d, e=e)
  No match: "cats" (all different), "aabb" (wrong pattern)
```

### Approach: Normalize Pattern + Trie

```python
def normalize(word):
    """Convert word to canonical pattern form."""
    mapping = {}
    counter = 0
    pattern = []
    for ch in word:
        if ch not in mapping:
            mapping[ch] = counter
            counter += 1
        pattern.append(mapping[ch])
    return tuple(pattern)

# "noon"  → (0, 1, 1, 0)
# "deed"  → (0, 1, 1, 0)
# "abba"  → (0, 1, 1, 0)
# "cats"  → (0, 1, 2, 3)
# "aabb"  → (0, 0, 1, 1)
```

```python
class PatternTrie:
    """Trie indexed by normalized patterns."""
    
    def __init__(self):
        self.root = {}
    
    def insert(self, word):
        pattern = normalize(word)
        node = self.root
        for p in pattern:
            if p not in node:
                node[p] = {}
            node = node[p]
        node.setdefault('#', []).append(word)
    
    def find_matches(self, pattern_word):
        """Find all words with the same pattern structure."""
        pattern = normalize(pattern_word)
        node = self.root
        for p in pattern:
            if p not in node:
                return []
            node = node[p]
        return node.get('#', [])
```

---

## Problem: Prefix and Suffix Search (LeetCode 745)

Given words, find the word with the largest index that has a given prefix AND suffix.

```
  words = ["apple"]
  f("a", "e") → 0  (word "apple" has prefix "a" and suffix "e")
```

### Approach: Wrapped Trie

```python
class WordFilter:
    def __init__(self, words):
        self.trie = {}
        
        for weight, word in enumerate(words):
            # Insert all suffix#prefix combinations
            # For "apple": insert "e#apple", "le#apple", "ple#apple", 
            #              "pple#apple", "apple#apple", "#apple"
            for i in range(len(word) + 1):
                suffix = word[i:]  # All suffixes including empty
                key = suffix + '#' + word
                node = self.trie
                for ch in key:
                    if ch not in node:
                        node[ch] = {}
                    node = node[ch]
                    node['$'] = weight  # Store latest weight at every node
    
    def f(self, prefix, suffix):
        """Find word with largest index having given prefix and suffix."""
        search = suffix + '#' + prefix
        node = self.trie
        for ch in search:
            if ch not in node:
                return -1
            node = node[ch]
        return node.get('$', -1)
```

### Trace

```
  words = ["apple", "apply"]
  
  For "apple" (weight=0):
    Insert: "#apple", "e#apple", "le#apple", "ple#apple", "pple#apple", "apple#apple"
  
  For "apply" (weight=1):
    Insert: "#apply", "y#apply", "ly#apply", "ply#apply", "pply#apply", "apply#apply"
  
  Query: f("app", "le")
  Search: "le#app"
  
  Navigate: l → e → # → a → p → p
  At p: weight = 0 (from "apple")
  
  Return: 0
```

---

## Problem: Search in Rotated Words

Find if any rotation of a query word exists in the dictionary.

```python
class RotationTrie:
    """Insert all rotations of each word."""
    
    def __init__(self, words):
        self.root = TrieNode()
        for word in words:
            doubled = word + word  # "abc" → "abcabc"
            n = len(word)
            for i in range(n):
                rotation = doubled[i:i+n]  # All rotations
                self.insert(rotation, word)
    
    def search_rotation(self, query):
        """Check if query is a rotation of any stored word."""
        return self.search(query)
```

```
  Word "abc":
  Rotations: "abc", "bca", "cab"
  All inserted into trie.
  
  Query "bca" → found! It's a rotation of "abc"
```

---

## Problem: Matching with Fixed Positions

Given a pattern with some fixed characters and '_' for unknowns:

```python
def search_partial(self, node, pattern, idx, path, results):
    """Match pattern like "c_t" (c, any, t)."""
    if idx == len(pattern):
        if node.is_end:
            results.append(''.join(path))
        return
    
    ch = pattern[idx]
    
    if ch == '_':
        # Unknown: try all children
        for c, child in node.children.items():
            path.append(c)
            self.search_partial(child, pattern, idx + 1, path, results)
            path.pop()
    else:
        # Fixed: must match
        if ch in node.children:
            path.append(ch)
            self.search_partial(node.children[ch], pattern, idx + 1, path, results)
            path.pop()
```

```
  Pattern: "c_t"
  Matches: "cat", "cut", "cot"
  No match: "car", "cap"
```

---

## Problem: Anagram Search in Trie

Find all words that are anagrams of a query:

```python
class AnagramTrie:
    """Trie indexed by sorted characters."""
    
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        sorted_key = ''.join(sorted(word))
        node = self.root
        for ch in sorted_key:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        if not hasattr(node, 'words'):
            node.words = []
        node.words.append(word)
        node.is_end = True
    
    def find_anagrams(self, word):
        sorted_key = ''.join(sorted(word))
        node = self.root
        for ch in sorted_key:
            if ch not in node.children:
                return []
            node = node.children[ch]
        return node.words if node.is_end else []
```

```
  Words: "eat", "tea", "ate", "cat", "tac"
  
  Sorted keys:
  "eat" → "aet", "tea" → "aet", "ate" → "aet"   → all at same node
  "cat" → "act", "tac" → "act"                    → all at same node
  
  find_anagrams("eta") → sorted="aet" → ["eat","tea","ate"]
```

---

## Summary Table

| Pattern Problem | Technique | Key Idea |
|----------------|-----------|----------|
| Same structure (abba) | Normalize + trie | Map chars to indices |
| Prefix + suffix | Wrapped trie | Insert suffix#word |
| Rotations | Double string | Insert all rotations |
| Partial `c_t` | Recursive search | '_' = try all children |
| Anagrams | Sorted key trie | Sort chars before insert |

---

## Quick Revision Questions

1. **How do you find words with the same pattern as "abba"?**
   > Normalize both pattern and words by mapping characters to sequential integers: "abba" → (0,1,1,0). Words with the same normalized form match the pattern.

2. **What is the "wrapped trie" technique for prefix-suffix search?**
   > Insert `suffix + '#' + word` for all suffixes of each word. To query prefix P and suffix S, search for `S + '#' + P` in the trie.

3. **How do you find all rotations of a word efficiently?**
   > Concatenate the word with itself ("abc" → "abcabc"). All rotations are substrings of length N in this doubled string. Insert each rotation into the trie.

4. **How does anagram search using a trie work?**
   > Sort the characters of each word before insertion (key). All anagrams of a word produce the same sorted key, so they end up at the same trie node.

5. **What's the time complexity of partial pattern search "c_t"?**
   > O(A^D × L) where A = alphabet size, D = number of unknowns, L = pattern length. Each '_' requires trying all children (up to A), while fixed chars follow one path.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Wildcard Matching](04-wildcard-matching.md) | [Next: Replace Words ▶](06-replace-words.md) |
