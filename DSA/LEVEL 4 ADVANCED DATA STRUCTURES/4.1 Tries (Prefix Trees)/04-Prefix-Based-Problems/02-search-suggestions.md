# Search Suggestions System (LeetCode 1268)

## Overview
Given an array of products and a search word, suggest at most 3 product names from the products list that have a common prefix with the search word after each character is typed. This problem elegantly combines trie traversal with incremental prefix building.

---

## Problem Statement

```
  Input: products = ["mobile","mouse","moneypot","monitor","mousepad"]
         searchWord = "mouse"
  
  Output: [
    ["mobile","moneypot","monitor","mouse","mousepad"],  ← after 'm'
    ["mobile","moneypot","monitor","mouse","mousepad"],  ← after 'mo'
    ["mouse","mousepad"],                                 ← after 'mou'
    ["mouse","mousepad"],                                 ← after 'mous'
    ["mouse","mousepad"]                                  ← after 'mouse'
  ]
  
  → Return TOP 3 lexicographically smallest for each prefix
```

---

## Approach 1: Trie + DFS

### Algorithm

```
  1. Build trie from all products
  2. For each prefix (m, mo, mou, mous, mouse):
     a. Navigate trie to prefix node
     b. DFS to collect up to 3 words (lexicographic order)
     c. Add to results
```

### Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Solution:
    def suggestedProducts(self, products, searchWord):
        # Build trie
        root = TrieNode()
        for product in products:
            node = root
            for ch in product:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        
        results = []
        node = root
        prefix = []
        dead = False
        
        for ch in searchWord:
            if dead or ch not in node.children:
                dead = True
                results.append([])
                continue
            
            node = node.children[ch]
            prefix.append(ch)
            
            # Collect up to 3 lexicographically smallest words
            suggestions = []
            self._dfs(node, list(prefix), suggestions)
            results.append(suggestions)
        
        return results
    
    def _dfs(self, node, path, suggestions):
        if len(suggestions) >= 3:  # ← Early termination!
            return
        if node.is_end:
            suggestions.append(''.join(path))
        for ch in sorted(node.children):  # Sorted for lexicographic order
            if len(suggestions) >= 3:
                return
            path.append(ch)
            self._dfs(node.children[ch], path, suggestions)
            path.pop()
```

### Trace

```
  Products: ["mobile","mouse","moneypot","monitor","mousepad"]
  
  Trie structure (sorted children):
  root─m─o─b─i─l─e*
           ├─n─e─y─p─o─t*
           │ └─i─t─o─r*
           └─u─s─e*
                 └─p─a─d*
  
  Prefix "m":
    DFS from 'm': mobile*, moneypot*, monitor* → STOP (3 found)
    Result: ["mobile","moneypot","monitor"]
  
  Prefix "mo":
    DFS from 'o': mobile*, moneypot*, monitor* → STOP
    Result: ["mobile","moneypot","monitor"]
  
  Prefix "mou":
    DFS from 'u': mouse*, mousepad*
    Result: ["mouse","mousepad"]
  
  Prefix "mous":
    DFS from 's': mouse*, mousepad*
    Result: ["mouse","mousepad"]
  
  Prefix "mouse":
    DFS from 'e': mouse*, mousepad*
    Result: ["mouse","mousepad"]
```

---

## Approach 2: Trie with Pre-Stored Suggestions

Store top-3 words at every trie node during insertion:

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.suggestions = []  # ← Top 3 words passing through

class Solution:
    def suggestedProducts(self, products, searchWord):
        products.sort()  # Sort first for lexicographic order
        
        root = TrieNode()
        for product in products:
            node = root
            for ch in product:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
                if len(node.suggestions) < 3:
                    node.suggestions.append(product)
        
        results = []
        node = root
        dead = False
        
        for ch in searchWord:
            if dead or ch not in node.children:
                dead = True
                results.append([])
            else:
                node = node.children[ch]
                results.append(node.suggestions)
        
        return results
```

```
  ✓ O(P) per query — just read pre-stored list
  ✓ No DFS needed at query time
  ✓ Products sorted once → suggestions automatically ordered
```

---

## Approach 3: Binary Search (No Trie)

Alternative approach using sorted array + binary search:

```python
class Solution:
    def suggestedProducts(self, products, searchWord):
        products.sort()
        results = []
        prefix = ""
        left = 0
        
        for ch in searchWord:
            prefix += ch
            # Binary search for first product >= prefix
            left = bisect.bisect_left(products, prefix, left)
            
            suggestions = []
            for i in range(left, min(left + 3, len(products))):
                if products[i].startswith(prefix):
                    suggestions.append(products[i])
                else:
                    break
            results.append(suggestions)
        
        return results
```

---

## Complexity Comparison

| Approach | Build Time | Query Time | Space |
|----------|:----------:|:----------:|:-----:|
| Trie + DFS | O(N × L) | O(S × (P + 3)) | O(N × L) |
| Trie + pre-stored | O(N × L) + sort | O(S × P) | O(N × L) |
| Binary search | O(N log N) | O(S × L × log N) | O(1) extra |

Where: N = products, L = avg product length, S = searchWord length, P = trie depth

---

## Common Edge Cases

```
  1. No matches at all:
     products = ["abc"], searchWord = "xyz"
     → [[], [], []]
  
  2. Exact match only:
     products = ["mouse"], searchWord = "mouse"
     → [["mouse"],["mouse"],["mouse"],["mouse"],["mouse"]]
  
  3. Dead-end mid-search:
     products = ["mobile"], searchWord = "mobs"
     → [["mobile"],["mobile"],["mobile"],[]]
     After "mob" → follows to 'b', but 's' not found → dead
  
  4. More than 3 matches:
     → Only return first 3 lexicographically
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Suggest 3 products per keystroke |
| Trie approach | Build trie, DFS from prefix node |
| Optimization | Pre-store sorted suggestions at each node |
| Alternative | Binary search on sorted products array |
| Key insight | Sort products first; DFS in sorted child order = lexicographic results |
| Early termination | Stop DFS after finding 3 words |

---

## Quick Revision Questions

1. **Why do we sort children during DFS?**
   > To get lexicographically smallest results first. By visiting children in alphabetical order, the first 3 complete words found are the smallest.

2. **What's the advantage of pre-storing suggestions at each node?**
   > Query becomes O(P) — just navigate to the prefix node and return the pre-stored list. No DFS needed at query time.

3. **How does the pre-store approach ensure correct ordering?**
   > Sort the products array before building the trie. Since we insert in sorted order and only keep the first 3 at each node, they're automatically the 3 smallest.

4. **What happens when a prefix leads to a dead end?**
   > All subsequent results are empty lists. Once we can't follow a character, no products match any longer prefix.

5. **Could binary search be better than a trie for this problem?**
   > Yes, when memory is constrained. Binary search uses O(1) extra space vs O(N × L) for a trie. But for repeated queries or streaming input, trie is more efficient.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Autocomplete System](01-autocomplete-system.md) | [Next: Prefix Count ▶](03-prefix-count.md) |
