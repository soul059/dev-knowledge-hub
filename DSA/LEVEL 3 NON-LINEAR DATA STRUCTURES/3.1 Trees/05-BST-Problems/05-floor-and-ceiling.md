# Unit 5 – Topic 5: Floor and Ceiling

[← Previous: LCA in BST](04-lca-in-bst.md) | [Back to TOC](../README.md) | [Next: BST Iterator →](06-bst-iterator.md)

---

## 5.5 Floor and Ceiling

### 📖 Definitions

- **Floor(x)**: Largest element in the BST that is **≤ x**
- **Ceiling(x)**: Smallest element in the BST that is **≥ x**

```
    BST contains: [2, 5, 8, 10, 15]
    
    Floor(7)   = 5     (largest value ≤ 7)
    Ceiling(7) = 8     (smallest value ≥ 7)
    Floor(10)  = 10    (10 exists in the tree)
    Ceiling(3) = 5     (smallest value ≥ 3)
```

### Algorithm: Floor

```
FUNCTION floor(root, key):
    result ← -1
    current ← root
    
    WHILE current ≠ NULL:
        IF current.data == key:
            RETURN current.data         // Exact match
        ELSE IF current.data < key:
            result ← current.data       // Candidate for floor
            current ← current.right     // Look for larger candidate
        ELSE:
            current ← current.left      // Current too large
    
    RETURN result
```

### Algorithm: Ceiling

```
FUNCTION ceiling(root, key):
    result ← -1
    current ← root
    
    WHILE current ≠ NULL:
        IF current.data == key:
            RETURN current.data         // Exact match
        ELSE IF current.data > key:
            result ← current.data       // Candidate for ceiling
            current ← current.left      // Look for smaller candidate
        ELSE:
            current ← current.right     // Current too small
    
    RETURN result
```

### 🔍 Trace: Floor(7)

```
            10
           / \
          5   15
         / \
        2   8

    Step 1: current=10, 10 > 7 → go left
    Step 2: current=5,  5 < 7 → result=5, go right
    Step 3: current=8,  8 > 7 → go left
    Step 4: current=NULL → return result=5 ✓
    
    Floor(7) = 5
```

### 🔍 Trace: Ceiling(7)

```
    Step 1: current=10, 10 > 7 → result=10, go left
    Step 2: current=5,  5 < 7 → go right
    Step 3: current=8,  8 > 7 → result=8, go left
    Step 4: current=NULL → return result=8 ✓
    
    Ceiling(7) = 8
```

### ⏱️ Complexity: O(h) time, O(1) space

---

### ❓ Revision Question

**What is the difference between floor and inorder predecessor?**
<details><summary>Answer</summary>Floor(x) finds the largest value ≤ x in the BST (x may or may not be in the tree). Inorder predecessor of a node already in the tree finds the next smaller value. If x exists in the tree, floor(x) = x itself, while inorder predecessor gives the value before x.</details>

---

[← Previous: LCA in BST](04-lca-in-bst.md) | [Back to TOC](../README.md) | [Next: BST Iterator →](06-bst-iterator.md)
