# Case Sensitivity in Tries

## Overview
Handling case sensitivity determines whether "Apple" and "apple" are treated as the same or different words. This chapter covers strategies for case-insensitive search, mixed-case storage, and their impact on index calculations and memory.

---

## The Challenge

```
  Case-sensitive trie:
  
  root ── A ── p ── p ── l ── e*      ("Apple")
       └── a ── p ── p ── l ── e*      ("apple")
  
  Two separate branches! "Apple" ≠ "apple"
  
  Case-insensitive trie:
  
  root ── a ── p ── p ── l ── e*      stores both "Apple" and "apple"
  
  Single branch — both map to the same path
```

---

## Strategy 1: Normalize at Input (Recommended)

Convert all input to lowercase (or uppercase) before any operation:

```python
class CaseInsensitiveTrie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word: str) -> None:
        word = word.lower()  # ← Normalize
        node = self.root
        for ch in word:
            idx = ord(ch) - ord('a')
            if node.children[idx] is None:
                node.children[idx] = TrieNode()
            node = node.children[idx]
        node.is_end = True
    
    def search(self, word: str) -> bool:
        word = word.lower()  # ← Same normalization
        node = self.root
        for ch in word:
            idx = ord(ch) - ord('a')
            if node.children[idx] is None:
                return False
            node = node.children[idx]
        return node.is_end
```

```
  trie.insert("Apple")   → stored as "apple"
  trie.insert("BANANA")  → stored as "banana"
  
  trie.search("apple")   → True  (matches "apple")
  trie.search("APPLE")   → True  (normalized to "apple")
  trie.search("ApPlE")   → True  (normalized to "apple")
  
  ✓ Uses standard 26-size array
  ✓ Simple and memory-efficient
  ✗ Loses original case information
```

---

## Strategy 2: Store Both Cases (52-size Array)

Support A-Z and a-z as separate characters:

```python
class MixedCaseTrie:
    """Array-based with 52 slots: A-Z (0-25), a-z (26-51)"""
    
    def __init__(self):
        self.root = [None] * 52 + [False]  # 52 children + isEnd
    
    def _char_to_index(self, ch):
        if 'A' <= ch <= 'Z':
            return ord(ch) - ord('A')       # 0-25
        else:
            return ord(ch) - ord('a') + 26  # 26-51
    
    def insert(self, word):
        node = self.root
        for ch in word:
            idx = self._char_to_index(ch)
            if node[idx] is None:
                node[idx] = [None] * 52 + [False]
            node = node[idx]
        node[52] = True  # isEnd
```

```
  Memory per node: 52 × 8 = 416 bytes (vs 208 for lowercase only)
  
  trie.insert("Apple")   → A(0) → p(41) → p(41) → l(37) → e(30)*
  trie.insert("apple")   → a(26) → p(41) → p(41) → l(37) → e(30)*
  
  ✓ Preserves original case
  ✓ Case-sensitive search
  ✗ Double the memory per node
  ✗ Usually unnecessary
```

---

## Strategy 3: HashMap for Flexible Case Handling

```python
class FlexibleCaseTrie:
    def __init__(self, case_sensitive=True):
        self.root = TrieNode()  # HashMap-based
        self.case_sensitive = case_sensitive
    
    def _normalize(self, word):
        if not self.case_sensitive:
            return word.lower()
        return word
    
    def insert(self, word):
        word = self._normalize(word)
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
    
    def search(self, word):
        word = self._normalize(word)
        node = self.root
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end
```

```
  ✓ Configurable case sensitivity
  ✓ No wasted memory on unused characters
  ✓ Supports any Unicode
  ✗ HashMap per-entry overhead
```

---

## Strategy 4: Case-Insensitive with Original Preservation

Store normalized keys but remember the original:

```python
class PreservingTrie:
    def insert(self, word):
        normalized = word.lower()
        node = self.root
        for ch in normalized:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
        node.original = word  # ← Preserve original
    
    def search(self, word):
        """Case-insensitive search returning original form."""
        normalized = word.lower()
        node = self.root
        for ch in normalized:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node.original if node.is_end else None

# Usage:
trie.insert("JavaScript")
trie.search("javascript")   # Returns "JavaScript" (original case)
trie.search("JAVASCRIPT")   # Returns "JavaScript" (original case)
```

---

## Index Mapping Reference

```
  ┌──────────────────────────────────────────────────────────┐
  │  Lowercase only (26 slots):                              │
  │  index = ord(ch) - ord('a')                              │
  │  'a'=0, 'b'=1, ..., 'z'=25                              │
  │                                                          │
  │  Uppercase only (26 slots):                              │
  │  index = ord(ch) - ord('A')                              │
  │  'A'=0, 'B'=1, ..., 'Z'=25                              │
  │                                                          │
  │  Both cases (52 slots):                                  │
  │  if uppercase: index = ord(ch) - ord('A')        (0-25)  │
  │  if lowercase: index = ord(ch) - ord('a') + 26  (26-51) │
  │                                                          │
  │  Alphanumeric (62 slots):                                │
  │  digit: ord(ch) - ord('0')           (0-9)               │
  │  upper: ord(ch) - ord('A') + 10      (10-35)             │
  │  lower: ord(ch) - ord('a') + 36      (36-61)             │
  │                                                          │
  │  Full ASCII (128 slots):                                 │
  │  index = ord(ch)                     (0-127)             │
  └──────────────────────────────────────────────────────────┘
```

---

## Interview Considerations

### LeetCode Problems and Case Sensitivity

```
  Most LeetCode trie problems specify:
  "word consists of lowercase English letters"
  
  → Use 26-size array, index = ch - 'a'
  → No case handling needed!
  
  If problem says "word contains lowercase and uppercase":
  → Ask: Should search be case-sensitive?
  → Usually: normalize to lowercase (Strategy 1)
  → Rarely: keep both cases (Strategy 2)
```

### Common Mistakes

```
  1. Forgetting to normalize in ALL operations
     ✗ Normalize in insert but not search
     → "Apple" is stored as "apple"
     → search("Apple") tries to find "A" (uppercase) → fails!
  
  2. Wrong index formula for mixed case
     ✗ index = ch - 'a' with uppercase input
     → 'A' - 'a' = -32 → NEGATIVE INDEX → crash!
  
  3. Mixing strategies
     ✗ Some words normalized, others not
     → Inconsistent state, unpredictable results
```

---

## Summary Table

| Strategy | Array Size | Memory/Node | Preserves Case | Best For |
|----------|:----------:|:-----------:|:--------------:|----------|
| Normalize to lower | 26 | 208 bytes | No | Most use cases |
| 52-slot array | 52 | 416 bytes | Yes | When case matters |
| HashMap | Dynamic | ~48+ bytes | Configurable | Unicode/flexible |
| Normalize + store | 26 | 208 + word | Original form | Autocomplete |

---

## Quick Revision Questions

1. **What's the simplest way to make a trie case-insensitive?**
   > Call `word.lower()` (or `word.upper()`) at the start of every operation — insert, search, delete, prefix queries. Use the standard 26-size array.

2. **What happens if you use `ch - 'a'` with an uppercase character?**
   > You get a negative index (e.g., `'A' - 'a' = -32`), causing an array out-of-bounds error or accessing invalid memory.

3. **When would you use a 52-slot array?**
   > When both uppercase and lowercase must be treated as distinct characters (e.g., "Apple" ≠ "apple") and you need O(1) array-based access.

4. **How do you map characters for an alphanumeric trie?**
   > Digits: `ch - '0'` (indices 0-9), uppercase: `ch - 'A' + 10` (10-35), lowercase: `ch - 'a' + 36` (36-61). Total array size: 62.

5. **Why is normalize-at-input preferred over storing both cases?**
   > It uses half the memory (26 vs 52 slots), is simpler to implement, and covers 95%+ of use cases. The original case can be stored separately if needed.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Handling End of Word](03-handling-end-of-word.md) | [Next: Boolean vs Count Marking ▶](05-boolean-vs-count.md) |
