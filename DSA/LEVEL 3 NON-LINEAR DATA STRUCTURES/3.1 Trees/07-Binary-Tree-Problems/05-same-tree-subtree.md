# Unit 7 – Topic 5: Same Tree / Subtree Check

[← Previous: Check Balanced Tree](04-check-balanced-tree.md) | [Back to TOC](../README.md) | [Next: Symmetric Tree →](06-symmetric-tree.md)

---

## 7.5 Same Tree / Subtree Check

### Same Tree

Two trees are the same if they have **identical structure and values**.

```
FUNCTION isSameTree(p, q):
    IF p == NULL AND q == NULL:
        RETURN true
    
    IF p == NULL OR q == NULL:
        RETURN false
    
    IF p.data ≠ q.data:
        RETURN false
    
    RETURN isSameTree(p.left, q.left) AND isSameTree(p.right, q.right)
```

### 🔍 Trace: Same Tree Check

```
    Tree p:     Tree q:
       1           1
      / \         / \
     2   3       2   3
    
    isSameTree(1, 1):
    ├── data: 1 == 1 ✓
    ├── isSameTree(2, 2):
    │   ├── data: 2 == 2 ✓
    │   ├── isSameTree(NULL, NULL) → true
    │   └── isSameTree(NULL, NULL) → true
    │   → true
    └── isSameTree(3, 3):
        ├── data: 3 == 3 ✓
        ├── isSameTree(NULL, NULL) → true
        └── isSameTree(NULL, NULL) → true
        → true
    → true ✓
```

### Subtree Check

Check if tree `t` is a subtree of tree `s`.

```
FUNCTION isSubtree(s, t):
    IF s == NULL:
        RETURN false
    
    IF isSameTree(s, t):
        RETURN true
    
    RETURN isSubtree(s.left, t) OR isSubtree(s.right, t)
```

### 🔍 Trace: Subtree Check

```
    Tree s:         Tree t:
        3               4
       / \              / \
      4   5            1   2
     / \
    1   2

    isSubtree(3, t): isSameTree(3, 4)? No
    ├── isSubtree(4, t): isSameTree(4, 4)?
    │   ├── 4 == 4 ✓
    │   ├── isSameTree(1, 1) → true
    │   └── isSameTree(2, 2) → true
    │   → true ✓  FOUND!
    
    Yes, t is a subtree of s ✓
```

### ⏱️ Complexity

| Problem | Time | Space |
|---------|------|-------|
| Same tree | O(min(m, n)) | O(min(h₁, h₂)) |
| Subtree | O(m × n) worst case | O(h₁) |

---

### ❓ Revision Question

**What is the time complexity of checking if tree t is a subtree of tree s?**
<details><summary>Answer</summary>O(m × n) in the worst case, where m and n are the sizes of s and t. For each of the m nodes in s, we might compare up to n nodes of t. An O(m + n) approach exists using tree serialization and string matching.</details>

---

[← Previous: Check Balanced Tree](04-check-balanced-tree.md) | [Back to TOC](../README.md) | [Next: Symmetric Tree →](06-symmetric-tree.md)
