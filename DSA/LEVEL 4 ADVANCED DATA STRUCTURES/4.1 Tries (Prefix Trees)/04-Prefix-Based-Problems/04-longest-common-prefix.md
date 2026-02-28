# Longest Common Prefix (LeetCode 14)

## Overview
Finding the longest common prefix (LCP) among a set of strings is a classic problem with an elegant trie-based solution. The LCP is the longest string that is a prefix of all strings in the set.

---

## Problem Statement

```
  Input: strs = ["flower", "flow", "flight"]
  Output: "fl"
  
  Input: strs = ["dog", "racecar", "car"]
  Output: ""  (no common prefix)
  
  Input: strs = ["apple", "apple", "apple"]
  Output: "apple"
```

---

## Trie-Based Approach

### Key Insight

```
  Insert all words into a trie. The LCP is the path from root
  where EVERY node has exactly ONE child and is NOT an end-of-word.
  
  Words: "flower", "flow", "flight"
  
  root ── f ── l ── o ── w ── e ── r*
                    ↑         └──*
                    └── i ── g ── h ── t*
  
  Walk from root:
  f → only child 'l'           continue ✓
  l → TWO children ('o', 'i')  STOP ✗
  
  LCP = "fl"
  
  Rules to continue:
  1. Node has EXACTLY one child
  2. Node is NOT marked as end-of-word
  
  If either fails, the path so far is the LCP.
```

### Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Solution:
    def longestCommonPrefix(self, strs):
        if not strs:
            return ""
        
        # Build trie
        root = TrieNode()
        for word in strs:
            if not word:
                return ""  # Empty string → LCP is ""
            node = root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        
        # Walk the trie
        lcp = []
        node = root
        while len(node.children) == 1 and not node.is_end:
            ch = next(iter(node.children))  # The single child
            lcp.append(ch)
            node = node.children[ch]
        
        return ''.join(lcp)
```

### Step-by-Step Trace

```
  strs = ["flower", "flow", "flight"]
  
  After building trie:
  
  root ─┬─ f ─┬─ l ─┬─ o ─┬─ w ─┬─ e ── r*
        │     │     │     │     └── *
        │     │     └── i ── g ── h ── t*
  
  Walk from root:
  ┌─────────┬──────────┬────────┬──────────┐
  │ Node    │ Children │ isEnd? │ Action   │
  ├─────────┼──────────┼────────┼──────────┤
  │ root    │ {'f': …} │ No     │ Add 'f'  │
  │ f       │ {'l': …} │ No     │ Add 'l'  │
  │ l       │ {'o','i'}│ No     │ STOP (2) │
  └─────────┴──────────┴────────┴──────────┘
  
  LCP = "fl"
```

---

## Why Check `is_end`?

```
  strs = ["app", "apple"]
  
  root ── a ── p ── p* ── l ── e*
  
  Walk from root:
  root → 1 child ('a'), not end → continue, add 'a'
  a    → 1 child ('p'), not end → continue, add 'p'
  p    → 1 child ('p'), not end → continue, add 'p'
  p    → 1 child ('l'), IS END  → STOP!
  
  LCP = "app"
  
  If we didn't check is_end:
  Would continue to "apple" — but "apple" is NOT a prefix of "app"!
  
  ┌───────────────────────────────────────────────────────────┐
  │  is_end marks that one of the strings ends here.          │
  │  Since LCP can't be longer than the shortest string,      │
  │  we must stop when we reach any word's end.               │
  └───────────────────────────────────────────────────────────┘
```

---

## Alternative Approaches (Non-Trie)

### Horizontal Scanning

```python
def longestCommonPrefix(strs):
    if not strs:
        return ""
    prefix = strs[0]
    for s in strs[1:]:
        while not s.startswith(prefix):
            prefix = prefix[:-1]
            if not prefix:
                return ""
    return prefix
```

### Vertical Scanning

```python
def longestCommonPrefix(strs):
    if not strs:
        return ""
    for i in range(len(strs[0])):
        ch = strs[0][i]
        for s in strs[1:]:
            if i >= len(s) or s[i] != ch:
                return strs[0][:i]
    return strs[0]
```

### Sorting Approach

```python
def longestCommonPrefix(strs):
    if not strs:
        return ""
    strs.sort()
    # LCP of entire array = LCP of first and last (after sorting)
    first, last = strs[0], strs[-1]
    i = 0
    while i < len(first) and i < len(last) and first[i] == last[i]:
        i += 1
    return first[:i]
```

---

## Complexity Comparison

| Approach | Time | Space |
|----------|:----:|:-----:|
| Trie | O(N × L) | O(N × L) |
| Horizontal scan | O(N × L) | O(1) |
| Vertical scan | O(N × L) | O(1) |
| Sort + compare | O(N × L × log N) | O(1) |

Where N = number of strings, L = average length

```
  ┌─────────────────────────────────────────────────────────┐
  │  For THIS problem alone, the trie approach uses more     │
  │  memory without a speed advantage.                       │
  │                                                          │
  │  But if you need to answer LCP queries for different     │
  │  subsets or the trie already exists, it becomes the      │
  │  better choice.                                          │
  └─────────────────────────────────────────────────────────┘
```

---

## LCP of Two Specific Words in a Trie

Given a trie, find LCP of any two words without scanning all words:

```python
def lcp_of_two(self, word1, word2):
    """Find LCP of two words using the trie structure."""
    node = self.root
    lcp = []
    for c1, c2 in zip(word1, word2):
        if c1 != c2:
            break
        if c1 not in node.children:
            break
        lcp.append(c1)
        node = node.children[c1]
    return ''.join(lcp)
```

---

## Related: LCP Array with Suffix Trie

For advanced use cases (suffix arrays, string matching):

```
  Building a suffix trie and finding LCP between suffixes
  is the basis for many advanced string algorithms.
  
  This is covered in more detail in suffix tree/array topics.
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Algorithm | Build trie, walk while single child & not end |
| Stop conditions | Multiple children OR is_end = True |
| Time | O(N × L) to build, O(L) to walk |
| Space | O(N × L) for trie |
| Better alternatives | Vertical scanning (O(1) space) for simple case |
| Trie advantage | When trie already exists or multiple LCP queries |

---

## Quick Revision Questions

1. **What are the two conditions to stop walking the trie for LCP?**
   > Stop when a node has more than one child (branching means strings diverge) or when a node is marked `is_end` (a string ends here, so LCP can't be longer).

2. **Why does sorting + comparing first & last work?**
   > After lexicographic sorting, the most different strings are at the ends. If the first and last share a prefix of length K, all strings between them must also share that prefix (by sorted order property).

3. **When is the trie approach preferred over vertical scanning?**
   > When the trie already exists for other operations (autocomplete, prefix search), or when you need to answer multiple LCP queries on different subsets efficiently.

4. **What happens if one of the strings is empty?**
   > LCP is "" — an empty string is a prefix of everything, and nothing longer can be a prefix of an empty string.

5. **Can you find LCP of a subset of words using a trie?**
   > Not directly with the standard trie walk. You'd need either a separate trie for the subset, or to mark which words include each node and check if all subset words pass through.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Prefix Count](03-prefix-count.md) | [Next: Words with Prefix ▶](05-words-with-prefix.md) |
