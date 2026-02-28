# Unit 7 – Topic 6: Symmetric Tree

[← Previous: Same Tree & Subtree](05-same-tree-subtree.md) | [Back to TOC](../README.md) | [Next: Path Problems →](../08-Path-Problems/01-root-to-leaf-paths.md)

---

## 7.6 Symmetric Tree

### 📖 Problem

Check if a binary tree is a **mirror of itself** (symmetric around its center).

```
    Symmetric:              Not Symmetric:
    
        1                       1
       / \                     / \
      2   2                   2   2
     / \ / \                   \   \
    3  4 4  3                   3   3
```

### 💡 Key Insight

> A tree is symmetric if the **left subtree is a mirror image** of the right subtree. Two trees are mirrors if:
> 1. Their roots have the same value
> 2. The left subtree of one is a mirror of the right subtree of the other

### Algorithm

```
FUNCTION isSymmetric(root):
    IF root == NULL:
        RETURN true
    RETURN isMirror(root.left, root.right)

FUNCTION isMirror(left, right):
    IF left == NULL AND right == NULL:
        RETURN true
    IF left == NULL OR right == NULL:
        RETURN false
    
    RETURN left.data == right.data
       AND isMirror(left.left, right.right)     // outer pair
       AND isMirror(left.right, right.left)     // inner pair
```

### 🔍 Trace

```
        1
       / \
      2   2
     / \ / \
    3  4 4  3

    isMirror(2, 2):
    ├── data: 2 == 2 ✓
    ├── isMirror(3, 3):  ← outer pair (left.left, right.right)
    │   ├── 3 == 3 ✓, both children NULL → true
    └── isMirror(4, 4):  ← inner pair (left.right, right.left)
        ├── 4 == 4 ✓, both children NULL → true
    → true ✓  Symmetric!
```

### Iterative Approach (Using Queue)

```
FUNCTION isSymmetricIterative(root):
    IF root == NULL: RETURN true
    
    queue ← empty queue
    queue.enqueue(root.left)
    queue.enqueue(root.right)
    
    WHILE queue is not empty:
        left  ← queue.dequeue()
        right ← queue.dequeue()
        
        IF left == NULL AND right == NULL:
            CONTINUE
        IF left == NULL OR right == NULL:
            RETURN false
        IF left.data ≠ right.data:
            RETURN false
        
        // Enqueue in mirror order
        queue.enqueue(left.left)
        queue.enqueue(right.right)
        queue.enqueue(left.right)
        queue.enqueue(right.left)
    
    RETURN true
```

### ⏱️ Complexity: O(n) time, O(h) space recursive / O(n) space iterative

---

## 🧩 Problem-Solving Patterns

### Pattern: Bottom-Up Recursion

Most tree problems follow this pattern:
```
FUNCTION solve(node):
    IF node == NULL:
        RETURN base_case
    
    leftResult  ← solve(node.left)
    rightResult ← solve(node.right)
    
    // Combine results + possibly update global variable
    RETURN combine(leftResult, rightResult, node)
```

**Used in**: max depth, min depth, diameter, balanced check, same tree.

### Pattern: Global Variable Update

For problems like diameter, we compute a **local value** at each node and update a **global maximum**:
```
    At each node:
        local_value = f(leftResult, rightResult)
        global_answer = max(global_answer, local_value)
        return value_for_parent
```

### Pattern: Two-Pointer on Trees

For symmetry and same-tree checks, we compare **two trees simultaneously**:
```
    compare(tree1, tree2):
        Both NULL → true
        One NULL → false
        Values differ → false
        Recurse on children (possibly crossed)
```

---

## 📝 Unit 7 Summary Table

| Problem | Approach | Time | Space |
|---------|----------|------|-------|
| Max Depth | Bottom-up: 1 + max(left, right) | O(n) | O(h) |
| Min Depth | Handle one-child case; BFS optimal | O(n) | O(h) or O(w) |
| Diameter | Height + global variable for max(L+R) | O(n) | O(h) |
| Balanced Check | Return -1 on imbalance, height otherwise | O(n) | O(h) |
| Same Tree | Compare both trees recursively | O(n) | O(h) |
| Subtree Check | For each node in s, check isSameTree(s, t) | O(m×n) | O(h) |
| Symmetric Tree | Mirror check: cross-compare left/right | O(n) | O(h) |

---

### ❓ Revision Questions

1. **What is the key difference between checking "same tree" and "symmetric tree"?**
   <details><summary>Answer</summary>Same tree: compare left↔left and right↔right (same positions). Symmetric tree: compare left↔right and right↔left (mirrored positions). Symmetric checks if a tree is its own mirror image.</details>

2. **Can a tree be balanced but have a large diameter?**
   <details><summary>Answer</summary>Yes! A perfect binary tree of height h is balanced, and its diameter is 2h (path from leftmost leaf to rightmost leaf through the root). A balanced tree with 1 million nodes has diameter ≈ 40 (2 × log₂(10⁶)).</details>

---

[← Previous: Same Tree & Subtree](05-same-tree-subtree.md) | [Back to TOC](../README.md) | [Next: Path Problems →](../08-Path-Problems/01-root-to-leaf-paths.md)
