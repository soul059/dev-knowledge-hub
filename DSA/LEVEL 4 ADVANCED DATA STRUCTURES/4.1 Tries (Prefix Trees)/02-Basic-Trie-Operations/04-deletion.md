# Deletion in a Trie

## Overview
Deletion is the most complex trie operation. We must remove the word from the trie while preserving other words that share common prefixes. The algorithm uses recursion to clean up unused nodes bottom-up.

---

## Algorithm

```
FUNCTION delete(word):
    _delete(root, word, depth=0)

FUNCTION _delete(node, word, depth):
    // Base case: reached end of word
    IF depth == length(word):
        IF NOT node.isEndOfWord:
            RETURN false              // word doesn't exist
        node.isEndOfWord = false      // unmark the word
        RETURN isEmpty(node.children) // can delete if no children

    // Recursive case
    char = word[depth]
    IF char NOT in node.children:
        RETURN false                  // word doesn't exist

    childShouldBeDeleted = _delete(node.children[char], word, depth + 1)

    IF childShouldBeDeleted:
        REMOVE node.children[char]            // remove the child
        RETURN isEmpty(node.children) AND NOT node.isEndOfWord
    
    RETURN false
```

---

## The Four Cases of Deletion

### Case 1: Word is a Prefix of Another Word

```
  Delete "app" from ["app", "apple"]

  Before:  a → p → p* → l → e*
  After:   a → p → p  → l → e*
                   ↑
                   ONLY unmark isEndOfWord
                   DO NOT delete any nodes (still needed for "apple")
```

Action: Just set `isEndOfWord = false`. No node deletion.

### Case 2: Another Word is a Prefix of This Word

```
  Delete "apple" from ["app", "apple"]

  Before:  a → p → p* → l → e*
  After:   a → p → p*
                      ↑
                      Delete 'l' and 'e' nodes (no longer needed)
```

Action: Unmark `isEndOfWord` at 'e', then delete 'e' (no children), then delete 'l' (no children, not end).

### Case 3: Word Shares No Nodes with Others

```
  Delete "dog" from ["cat", "dog"]

  Before:  root → c → a → t*
                → d → o → g*
  
  After:   root → c → a → t*
                ↑
                Entire "dog" branch removed
```

Action: Delete all nodes in the "dog" path bottom-up.

### Case 4: Word Branches from a Shared Prefix

```
  Delete "car" from ["car", "card"]

  Before:  c → a → r* → d*
  After:   c → a → r  → d*
                   ↑
                   Unmark only, 'd' child keeps 'r' alive
```

Action: Just unmark `isEndOfWord` at 'r'. Node 'r' stays because it has child 'd'.

---

## Detailed Trace: Delete "cars" from trie with ["car", "cars", "card"]

```
  Before:
      (root)
        │
        c
        │
        a
        │
        r*
       / \
      d*   s*

  Recursive calls:
  ┌───────┬──────────┬────────────────────────────────────────────┐
  │ Depth │   Char   │ Action                                     │
  ├───────┼──────────┼────────────────────────────────────────────┤
  │   0   │   'c'    │ Found 'c' in root → recurse deeper        │
  │   1   │   'a'    │ Found 'a' in 'c'  → recurse deeper        │
  │   2   │   'r'    │ Found 'r' in 'a'  → recurse deeper        │
  │   3   │   's'    │ Found 's' in 'r'  → recurse deeper        │
  │   4   │  (end)   │ depth == len("cars"), unmark isEnd         │
  │       │          │ 's' has no children → return true (DELETE) │
  └───────┴──────────┴────────────────────────────────────────────┘

  Unwinding:
  ┌───────┬────────────────────────────────────────────────────────┐
  │ Depth │ Decision                                               │
  ├───────┼────────────────────────────────────────────────────────┤
  │   3   │ Delete 's' from 'r'.children ✓                        │
  │       │ 'r' still has child 'd' and isEnd=true → KEEP 'r'     │
  │   2   │ childShouldBeDeleted = false → KEEP 'a'               │
  │   1   │ KEEP 'c'                                               │
  │   0   │ KEEP root                                              │
  └───────┴────────────────────────────────────────────────────────┘

  After:
      (root)
        │
        c
        │
        a
        │
        r*
        │
        d*
```

---

## Decision Logic Diagram

```
  After unmarking isEndOfWord:

  ┌──────────────────────────────┐
  │ Does node have children?     │
  └──────────────┬───────────────┘
           ┌─────┴─────┐
           ▼           ▼
          YES          NO
           │           │
     ┌─────┴─────┐   ┌┴───────────────────────┐
     │ KEEP node │   │ Is node isEndOfWord?    │
     │ (needed   │   └──────────┬──────────────┘
     │  by other │        ┌─────┴─────┐
     │  words)   │        ▼           ▼
     └───────────┘       YES          NO
                          │           │
                    ┌─────┴─────┐   ┌┴─────────────┐
                    │ KEEP node │   │ DELETE node   │
                    │ (marks a  │   │ (completely   │
                    │  word)    │   │  unused)      │
                    └───────────┘   └───────────────┘
```

---

## Implementation

### Python (Recursive)
```python
def delete(self, word):
    self._delete(self.root, word, 0)

def _delete(self, node, word, depth):
    if depth == len(word):
        if not node.is_end:
            return False
        node.is_end = False
        return len(node.children) == 0
    
    ch = word[depth]
    if ch not in node.children:
        return False
    
    should_delete = self._delete(node.children[ch], word, depth + 1)
    
    if should_delete:
        del node.children[ch]
        return len(node.children) == 0 and not node.is_end
    
    return False
```

### Python (Iterative — Alternative)
```python
def delete_iterative(self, word):
    # First, check if word exists
    node = self.root
    path = [(self.root, '')]  # track path for cleanup
    for ch in word:
        if ch not in node.children:
            return False
        path.append((node, ch))
        node = node.children[ch]
    
    if not node.is_end:
        return False
    node.is_end = False
    
    # Clean up unused nodes bottom-up
    if len(node.children) > 0:
        return True  # node has children, can't delete
    
    for i in range(len(path) - 1, 0, -1):
        parent, char = path[i]
        del parent.children[char]
        if len(parent.children) > 0 or parent.is_end:
            break  # parent is still needed
    
    return True
```

### Java (Recursive)
```java
public boolean delete(String word) {
    return delete(root, word, 0);
}

private boolean delete(TrieNode node, String word, int depth) {
    if (depth == word.length()) {
        if (!node.isEndOfWord) return false;
        node.isEndOfWord = false;
        return isEmpty(node);
    }
    
    int idx = word.charAt(depth) - 'a';
    if (node.children[idx] == null) return false;
    
    boolean shouldDelete = delete(node.children[idx], word, depth + 1);
    
    if (shouldDelete) {
        node.children[idx] = null;
        return isEmpty(node) && !node.isEndOfWord;
    }
    return false;
}

private boolean isEmpty(TrieNode node) {
    for (TrieNode child : node.children)
        if (child != null) return false;
    return true;
}
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|:----------:|-------------|
| Time | **O(L)** | Visit L nodes, O(1) work each |
| Space | **O(L)** | Recursion stack depth = L |
| Node deletions | 0 to L | Depends on sharing |

---

## More Deletion Examples

```
  ╔═══════════════════════════════════════════════════════════╗
  ║  Trie: ["abc", "abcd", "abce", "xyz"]                   ║
  ║                                                          ║
  ║       (root)                                             ║
  ║       /    \                                             ║
  ║      a      x                                            ║
  ║      │      │                                            ║
  ║      b      y                                            ║
  ║      │      │                                            ║
  ║      c*     z*                                           ║
  ║     / \                                                  ║
  ║    d*  e*                                                ║
  ╠═══════════════════════════════════════════════════════════╣
  ║  Delete "abcd":                                          ║
  ║  → Unmark 'd', delete 'd' (no children)                  ║
  ║  → 'c' still has child 'e' and isEnd=true → KEEP        ║
  ║  Result: ["abc", "abce", "xyz"]                          ║
  ╠═══════════════════════════════════════════════════════════╣
  ║  Delete "abc" (from previous result):                    ║
  ║  → Unmark 'c', but 'c' has child 'e' → KEEP            ║
  ║  Result: ["abce", "xyz"]                                 ║
  ╠═══════════════════════════════════════════════════════════╣
  ║  Delete "abce" (from previous result):                   ║
  ║  → Unmark 'e', delete 'e', 'c' has no children/end      ║
  ║  → Delete 'c', 'b' has no children → delete 'b'         ║
  ║  → Delete 'a' branch entirely                            ║
  ║  Result: ["xyz"]                                         ║
  ╚═══════════════════════════════════════════════════════════╝
```

---

## Summary Table

| Deletion Case | Nodes Removed | isEnd Changed |
|--------------|:------------:|:-------------:|
| Word is prefix of another | 0 | Yes (unmark) |
| Another word is prefix | 1 to L | Yes |
| No shared prefix | Up to L | Yes |
| Branching from shared | 0 | Yes (unmark) |
| Word doesn't exist | 0 | No |

---

## Quick Revision Questions

1. **When deleting "app" from a trie with both "app" and "apple", what happens?**
   > Only `isEndOfWord` is set to false at 'p'. No nodes are deleted because they're needed for "apple".

2. **Why is deletion done recursively (bottom-up)?**
   > Because we can only decide to delete a node AFTER checking if its descendant was deleted. The decision propagates upward.

3. **What determines whether a node can be safely deleted?**
   > A node can be deleted if it has no children AND `isEndOfWord` is false. If either condition is true, the node must be kept.

4. **What happens if we try to delete a word that doesn't exist?**
   > The function returns false without modifying the trie. It detects non-existence either by a missing child or by isEndOfWord being false.

5. **What is the space complexity of recursive deletion?**
   > O(L) for the recursion call stack, where L is the word length.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Prefix Search](03-prefix-search.md) | [Next: Time Complexity ▶](05-time-complexity.md) |
