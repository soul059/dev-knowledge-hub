# 4.2 BST Search Operation

[← Previous: BST Property](01-bst-property.md) | [Back to TOC](../README.md) | [Next: BST Insertion →](03-bst-insertion.md)

---

## 📖 Algorithm

```
FUNCTION search(node, key):
    IF node == NULL:
        RETURN NULL           // Key not found
    
    IF key == node.data:
        RETURN node           // Found!
    ELSE IF key < node.data:
        RETURN search(node.left, key)     // Go left
    ELSE:
        RETURN search(node.right, key)    // Go right
```

---

## Iterative Version

```
FUNCTION searchIterative(root, key):
    current ← root
    
    WHILE current ≠ NULL:
        IF key == current.data:
            RETURN current
        ELSE IF key < current.data:
            current ← current.left
        ELSE:
            current ← current.right
    
    RETURN NULL    // Not found
```

---

## 🔍 Trace: Search for 7

```
         8
        / \
       3   10
      / \    \
     1   6   14
        / \
       4   7

    Step 1: current = 8,  7 < 8  → go LEFT
    Step 2: current = 3,  7 > 3  → go RIGHT
    Step 3: current = 6,  7 > 6  → go RIGHT
    Step 4: current = 7,  7 == 7 → FOUND! ✓
    
    Path: 8 → 3 → 6 → 7  (4 comparisons)
```

## 🔍 Trace: Search for 5 (Not Found)

```
    Step 1: current = 8,  5 < 8  → go LEFT
    Step 2: current = 3,  5 > 3  → go RIGHT
    Step 3: current = 6,  5 < 6  → go LEFT
    Step 4: current = 4,  5 > 4  → go RIGHT
    Step 5: current = NULL → NOT FOUND ✗
```

---

## ⏱️ Complexity

| Case | Time | When? |
|------|------|-------|
| Best | O(1) | Key is at root |
| Average | O(log n) | Balanced tree |
| Worst | O(n) | Skewed tree (like linked list) |
| Space | O(h) recursive, O(1) iterative | h = height |

---

[← Previous: BST Property](01-bst-property.md) | [Back to TOC](../README.md) | [Next: BST Insertion →](03-bst-insertion.md)
