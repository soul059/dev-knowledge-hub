# Autocomplete System

## Overview
Autocomplete is the most iconic trie application. Given a prefix typed by a user, suggest matching words ranked by frequency, recency, or relevance. This chapter covers the complete design — from basic trie-based autocomplete to the full LeetCode 642 (Design Search Autocomplete System).

---

## Basic Autocomplete

### Algorithm

```
  1. Navigate trie to the prefix node
  2. DFS from that node to collect all complete words
  3. Return the collected words (optionally sorted/ranked)
  
  ┌─────────────────────────────────────────────────────────┐
  │  User types: "app"                                      │
  │                                                          │
  │  Trie contains: apple, application, app, ape, banana    │
  │                                                          │
  │  Step 1: Navigate root → a → p → p                      │
  │  Step 2: DFS from 'p' node:                             │
  │          p (isEnd → "app")                               │
  │          p → l → e (isEnd → "apple")                     │
  │          p → l → i → c → a → t → i → o → n             │
  │                              (isEnd → "application")     │
  │  Step 3: Return ["app", "apple", "application"]          │
  └─────────────────────────────────────────────────────────┘
```

### Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class AutocompleteTrie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
    
    def autocomplete(self, prefix):
        """Return all words starting with prefix."""
        # Phase 1: Navigate to prefix node
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return []  # No words with this prefix
            node = node.children[ch]
        
        # Phase 2: DFS to collect all words
        results = []
        self._dfs(node, list(prefix), results)
        return results
    
    def _dfs(self, node, path, results):
        if node.is_end:
            results.append(''.join(path))
        for ch in sorted(node.children):  # Alphabetical order
            path.append(ch)
            self._dfs(node.children[ch], path, results)
            path.pop()
```

---

## Ranked Autocomplete (By Frequency)

### Structure

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False
        self.frequency = 0  # ← Track how often this word is used

class RankedAutocomplete:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word, freq=1):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True
        node.frequency += freq
    
    def autocomplete(self, prefix, top_k=5):
        """Return top-k words by frequency."""
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return []
            node = node.children[ch]
        
        # Collect all matches with frequencies
        matches = []
        self._dfs(node, list(prefix), matches)
        
        # Sort by frequency (descending), then alphabetical
        matches.sort(key=lambda x: (-x[1], x[0]))
        return [word for word, freq in matches[:top_k]]
    
    def _dfs(self, node, path, matches):
        if node.is_end:
            matches.append((''.join(path), node.frequency))
        for ch in node.children:
            path.append(ch)
            self._dfs(node.children[ch], path, matches)
            path.pop()
```

### Trace: Top-3 Autocomplete

```
  Words with frequencies:
  "apple" (10), "application" (5), "app" (15), "appetite" (3), "apex" (7)
  
  User types: "ap"
  
  Navigate: root → a → p
  DFS from 'p':
    "app" (15), "apple" (10), "application" (5),
    "appetite" (3), "apex" (7)
  
  Sort by freq: app(15), apple(10), apex(7), application(5), appetite(3)
  Top 3: ["app", "apple", "apex"]
```

---

## LeetCode 642: Design Search Autocomplete System

### Problem

```
  Design a search autocomplete system:
  - Constructor: Initialize with sentences and their frequencies
  - input(char c): Process a character
    - If c == '#': save the current sentence (increment frequency)
    - Otherwise: return top 3 sentences with matching prefix
      sorted by frequency (desc), then lexicographic order
```

### Solution

```python
import heapq

class AutocompleteSystem:
    def __init__(self, sentences, times):
        self.root = {}
        self.current_input = []
        self.current_node = self.root
        self.dead = {}  # Dead-end sentinel
        
        # Build trie from initial data
        for sentence, count in zip(sentences, times):
            self._add(sentence, count)
    
    def _add(self, sentence, count):
        node = self.root
        for ch in sentence:
            if ch not in node:
                node[ch] = {}
            node = node[ch]
        # Store frequency at terminal
        node['#'] = node.get('#', 0) + count
    
    def input(self, c):
        if c == '#':
            # Save current sentence
            sentence = ''.join(self.current_input)
            self._add(sentence, 1)
            self.current_input = []
            self.current_node = self.root
            return []
        
        self.current_input.append(c)
        
        # Navigate to current prefix node
        if self.current_node is self.dead:
            return []
        
        if c not in self.current_node:
            self.current_node = self.dead
            return []
        
        self.current_node = self.current_node[c]
        
        # Collect all sentences under current prefix
        matches = []
        self._dfs(self.current_node, list(self.current_input), matches)
        
        # Sort: by frequency desc, then lexicographic
        matches.sort(key=lambda x: (-x[1], x[0]))
        return [s for s, f in matches[:3]]
    
    def _dfs(self, node, path, matches):
        if '#' in node:
            matches.append((''.join(path), node['#']))
        for ch in node:
            if ch != '#':
                path.append(ch)
                self._dfs(node[ch], path, matches)
                path.pop()
```

### Trace

```
  Init: sentences=["i love you","island","iroman","i love leetcode"]
        times=   [5, 3, 2, 2]
  
  input('i'):
    prefix = "i"
    matches: "i love you"(5), "island"(3), "i love leetcode"(2), "iroman"(2)
    sorted:  "i love you"(5), "island"(3), "i love leetcode"(2)
    return:  ["i love you", "island", "i love leetcode"]
  
  input(' '):
    prefix = "i "
    matches: "i love you"(5), "i love leetcode"(2)
    return:  ["i love you", "i love leetcode"]
  
  input('a'):
    prefix = "i a"
    matches: [] (no sentences start with "i a")
    return:  []
  
  input('#'):
    save "i a" with frequency 1
    reset
    return: []
```

---

## Optimization: Pre-Store Top Results

For large datasets, DFS on every keystroke is expensive. Pre-compute suggestions:

```python
class OptimizedAutocomplete:
    class Node:
        def __init__(self):
            self.children = {}
            self.top_suggestions = []  # ← Pre-computed top-k
    
    def insert(self, word, freq):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = self.Node()
            node = node.children[ch]
            # Update top suggestions at EVERY node along the path
            self._update_top(node, word, freq)
        node.is_end = True
    
    def _update_top(self, node, word, freq, k=3):
        """Maintain top-k suggestions at this node."""
        # Check if word already in suggestions
        for i, (w, f) in enumerate(node.top_suggestions):
            if w == word:
                node.top_suggestions[i] = (word, freq)
                node.top_suggestions.sort(key=lambda x: (-x[1], x[0]))
                return
        
        node.top_suggestions.append((word, freq))
        node.top_suggestions.sort(key=lambda x: (-x[1], x[0]))
        node.top_suggestions = node.top_suggestions[:k]
    
    def autocomplete(self, prefix):
        """O(P) — just navigate to prefix node!"""
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return []
            node = node.children[ch]
        return [word for word, freq in node.top_suggestions]
```

```
  Time: O(P) per query instead of O(P + total_matching_chars)
  Space: O(K × Nodes) extra for storing top-K at each node
  
  Trade-off: More memory, but instant suggestions!
```

---

## Complexity Analysis

| Approach | Query Time | Insert Time | Extra Space |
|----------|:----------:|:-----------:|:-----------:|
| Basic DFS | O(P + K) | O(L) | None |
| Ranked (sort after DFS) | O(P + K log K) | O(L) | None |
| Pre-stored top results | O(P) | O(L × K) | O(K × Nodes) |

Where: P = prefix length, K = total chars in matching words, L = word length

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Algorithm | Navigate to prefix + DFS collect |
| Ranking | Sort by frequency (desc), then alphabetical |
| LeetCode 642 | Track `current_node` across calls; '#' saves |
| Optimization | Pre-store top-K at each node for O(P) queries |
| ASCII only | Map space to children too (not just a-z) |

---

## Quick Revision Questions

1. **What are the two phases of basic autocomplete?**
   > Phase 1: Navigate from root to the prefix node (O(P)). Phase 2: DFS from that node to collect all complete words (O(K) where K = total characters in matching words).

2. **How do you rank autocomplete results by frequency?**
   > Store a `frequency` counter at terminal nodes. After collecting all matches via DFS, sort by frequency descending, then alphabetically for ties.

3. **In LeetCode 642, what happens when '#' is input?**
   > The current sentence (all characters typed since last '#') is saved into the trie with its frequency incremented by 1. Then the current input and position are reset.

4. **What's the key optimization for real-time autocomplete?**
   > Pre-compute and store top-K suggestions at every trie node. This reduces query time from O(P + K) to O(P) — just navigate to the prefix node and return pre-stored results.

5. **How does the autocomplete handle spaces and special characters?**
   > Use a hashmap-based trie (not array-based with 26 slots). Spaces, digits, and special characters become valid keys in the children map.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Memory Considerations](../03-Trie-Implementation/06-memory-considerations.md) | [Next: Search Suggestions ▶](02-search-suggestions.md) |
