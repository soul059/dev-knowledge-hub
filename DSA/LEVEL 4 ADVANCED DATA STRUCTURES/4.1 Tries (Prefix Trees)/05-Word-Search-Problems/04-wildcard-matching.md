# Wildcard and Pattern Matching in Tries

## Overview
Beyond the single-character wildcard '.', tries can support more complex pattern matching including multi-character wildcards, character classes, and regex-like patterns. This chapter covers trie-based approaches to pattern matching problems.

---

## Single Character Wildcard ('.')

Already covered in [Add and Search Word](03-add-and-search-word.md). Quick recap:

```python
def search_with_dot(self, node, pattern, idx):
    if idx == len(pattern):
        return node.is_end
    
    ch = pattern[idx]
    if ch == '.':
        return any(self.search_with_dot(child, pattern, idx + 1)
                    for child in node.children.values())
    else:
        if ch not in node.children:
            return False
        return self.search_with_dot(node.children[ch], pattern, idx + 1)
```

---

## Multi-Character Wildcard ('*')

'*' matches zero or more characters:

```
  Pattern: "ca*"  → matches "ca", "cat", "card", "castle", etc.
  Pattern: "*at"  → matches "at", "cat", "bat", "flat", etc.
  Pattern: "c*t"  → matches "ct", "cat", "cart", "coconut", etc.
```

### Trie-Based Implementation

```python
def search_with_star(self, node, pattern, idx):
    """'*' matches zero or more characters."""
    if idx == len(pattern):
        return node.is_end
    
    ch = pattern[idx]
    
    if ch == '*':
        # Option 1: '*' matches zero characters → skip it
        if self.search_with_star(node, pattern, idx + 1):
            return True
        
        # Option 2: '*' matches one character → try each child
        for child in node.children.values():
            # Keep '*' in pattern (can match more characters)
            if self.search_with_star(child, pattern, idx):
                return True
        
        return False
    else:
        if ch not in node.children:
            return False
        return self.search_with_star(node.children[ch], pattern, idx + 1)
```

### Trace: Pattern "c*t" on trie with "cat", "cart", "ct", "cut"

```
  search_with_star(root, "c*t", 0)
    ch='c', follow 'c'
    
    search_with_star(c_node, "c*t", 1)
      ch='*'
      
      Option 1: skip '*' → search(c_node, "c*t", 2)
        ch='t', 't' in c.children? 
        If "ct" exists: → t.is_end? Yes → True!
      
      Option 2: match one char with '*'
        Try child 'a': search(a_node, "c*t", 1) ← '*' stays!
          ch='*'
          Option 1: skip → search(a_node, "c*t", 2)
            ch='t', 't' in a.children? Yes → t.is_end? Yes → "cat" ✓
          (found, return True)
        
        Try child 'u': similar path → "cut" ✓
        Try child 'a' → 'r' → 't': "cart" ✓
```

---

## Character Class Matching ('[abc]')

Match one character from a set:

```python
def search_with_class(self, node, pattern, idx):
    if idx == len(pattern):
        return node.is_end
    
    ch = pattern[idx]
    
    if ch == '[':
        # Find closing bracket
        end = pattern.index(']', idx)
        char_class = pattern[idx+1:end]
        
        # Try each character in the class
        for c in char_class:
            if c in node.children:
                if self.search_with_class(node.children[c], pattern, end + 1):
                    return True
        return False
    elif ch == '.':
        return any(self.search_with_class(child, pattern, idx + 1)
                    for child in node.children.values())
    else:
        if ch not in node.children:
            return False
        return self.search_with_class(node.children[ch], pattern, idx + 1)
```

```
  Pattern: "[bc]at"
  Matches: "bat", "cat"
  No match: "hat", "mat"
  
  At idx 0: try 'b' and 'c' from the class
  At idx 1: follow 'a'
  At idx 2: follow 't', check is_end
```

---

## Negated Character Class ('[^abc]')

Match any character NOT in the set:

```python
if ch == '[' and pattern[idx+1] == '^':
    end = pattern.index(']', idx)
    excluded = set(pattern[idx+2:end])
    
    for c, child in node.children.items():
        if c not in excluded:
            if self.search_with_class(child, pattern, end + 1):
                return True
    return False
```

---

## Practical Pattern: Spell Check with Edit Distance

Find words within edit distance 1 (one substitution):

```python
def search_with_one_substitution(self, word):
    """Find all words that differ by exactly one character."""
    results = []
    
    def dfs(node, idx, substitutions, path):
        if idx == len(word):
            if node.is_end and substitutions == 1:
                results.append(''.join(path))
            return
        
        for ch, child in node.children.items():
            if ch == word[idx]:
                # Match — no substitution
                path.append(ch)
                dfs(child, idx + 1, substitutions, path)
                path.pop()
            elif substitutions == 0:
                # Substitution — use if we haven't already
                path.append(ch)
                dfs(child, idx + 1, 1, path)
                path.pop()
    
    dfs(self.root, 0, 0, [])
    return results
```

```
  word = "cat"
  Trie: bat, car, cat, cut, hat
  
  DFS with substitutions:
  "bat" — substitution at position 0 (c→b) — ✓ (distance 1)
  "car" — substitution at position 2 (t→r) — ✓ (distance 1)
  "cat" — no substitutions — ✗ (distance 0, need exactly 1)
  "cut" — substitution at position 1 (a→u) — ✓ (distance 1)
  "hat" — substitution at position 0 (c→h) — ✓ (distance 1)
  
  Results: ["bat", "car", "cut", "hat"]
```

---

## Complexity Summary

| Pattern Type | Time per Search |
|-------------|:---------------:|
| Exact | O(L) |
| Single '.' | O(A^D × L) |
| '*' wildcard | O(A^L × L) worst |
| Character class `[abc]` | O(K × ...) where K = class size |
| Edit distance 1 | O(A × L) |

Where A = alphabet size, D = number of wildcards, L = pattern length

---

## Summary Table

| Pattern | Meaning | Trie Approach |
|---------|---------|---------------|
| `.` | Any single char | Try all children |
| `*` | Zero or more chars | Skip or consume children |
| `[abc]` | One of a/b/c | Try 'a', 'b', 'c' only |
| `[^abc]` | Not a/b/c | Try all children except a/b/c |
| Edit distance | Similar words | DFS with substitution counter |

---

## Quick Revision Questions

1. **How does '*' wildcard differ from '.' in trie search?**
   > '.' matches exactly one character (try all children, advance pattern). '*' matches zero or more characters (skip or consume children without advancing pattern).

2. **Why is '*' search potentially exponential?**
   > '*' can match any number of characters, creating a branching factor at every level. Each child can be explored with '*' still active, leading to exploring the entire trie subtree.

3. **How do you implement character class `[abc]` matching?**
   > Parse the characters between brackets, then only try those specific children instead of all children (like '.' does).

4. **How does edit-distance-1 search work in a trie?**
   > DFS with a substitution counter. At each level, follow matching children normally. For non-matching children, follow them too but increment the counter. Only accept words where exactly 1 substitution was made.

5. **Can these patterns be combined?**
   > Yes. A unified `search` function checks each character type (literal, '.', '*', '[...]') and dispatches accordingly. Each type uses a different branching strategy during trie traversal.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Add and Search Word](03-add-and-search-word.md) | [Next: Pattern Matching ▶](05-pattern-matching.md) |
