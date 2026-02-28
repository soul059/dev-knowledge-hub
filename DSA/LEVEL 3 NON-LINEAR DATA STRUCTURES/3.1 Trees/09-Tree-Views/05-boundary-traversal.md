# Unit 9 – Topic 5: Boundary Traversal

[← Previous: Bottom View](04-bottom-view.md) | [Back to TOC](../README.md) | [Next: Zigzag Traversal →](06-zigzag-traversal.md)

---

## 9.5 Boundary Traversal

### 📖 Problem

Print the **boundary** of the tree in anti-clockwise order: left boundary (top-down) → leaf nodes (left-right) → right boundary (bottom-up).

```
              1
            /   \
           2     3
          / \   / \
         4   5 6   7
            / \     \
           8   9    10

    Boundary: [1, 2, 4, 8, 9, 6, 10, 7, 3]
    
    Left boundary (excl leaves): 1, 2
    Leaf nodes (L to R):         4, 8, 9, 6, 10
    Right boundary (excl leaves, bottom→up): 7, 3
```

### Algorithm

```
FUNCTION boundaryTraversal(root):
    IF root == NULL: RETURN []
    
    result ← [root.data]
    
    // Step 1: Left boundary (top-down, exclude root and leaves)
    leftBoundary(root.left, result)
    
    // Step 2: All leaf nodes (left to right)
    leaves(root.left, result)
    leaves(root.right, result)
    
    // Step 3: Right boundary (bottom-up, exclude root and leaves)
    rightBoundary(root.right, result)
    
    RETURN result

FUNCTION leftBoundary(node, result):
    IF node == NULL OR isLeaf(node):
        RETURN
    result.append(node.data)
    IF node.left:
        leftBoundary(node.left, result)
    ELSE:
        leftBoundary(node.right, result)  // If no left, go right

FUNCTION rightBoundary(node, result):
    IF node == NULL OR isLeaf(node):
        RETURN
    IF node.right:
        rightBoundary(node.right, result)
    ELSE:
        rightBoundary(node.left, result)
    result.append(node.data)              // Add AFTER recursion (bottom-up)

FUNCTION leaves(node, result):
    IF node == NULL: RETURN
    IF isLeaf(node):
        result.append(node.data)
        RETURN
    leaves(node.left, result)
    leaves(node.right, result)

FUNCTION isLeaf(node):
    RETURN node.left == NULL AND node.right == NULL
```

### 🔍 Trace

```
              20
            /    \
           8      22
          / \       \
         4  12      25
           /  \
          10  14

    Step 1: Left boundary (excl root, excl leaves)
            Start at 8: not leaf → add 8
            8.left = 4: IS leaf → stop
            Left boundary: [8]
    
    Step 2: Leaf nodes (L to R)
            4 (leaf), 10 (leaf), 14 (leaf), 25 (leaf)
            Leaves: [4, 10, 14, 25]
    
    Step 3: Right boundary (excl root, excl leaves, bottom-up)
            Start at 22: not leaf → recurse
            22.right = 25: IS leaf → stop
            Going back up: add 22
            Right boundary: [22]
    
    Full boundary: [20] + [8] + [4, 10, 14, 25] + [22]
                 = [20, 8, 4, 10, 14, 25, 22]
```

### ⏱️ Complexity: O(n) time, O(h) space

---

### ❓ Revision Question

**Why does Boundary Traversal add right boundary nodes in reverse (bottom-up)?**
<details><summary>Answer</summary>To maintain anti-clockwise order. The boundary goes: root → down the left side → across leaves left-to-right → up the right side. Going up the right side means we need the rightmost nodes from bottom to top, which we achieve by recursing first and adding after (post-order for right boundary).</details>

---

[← Previous: Bottom View](04-bottom-view.md) | [Back to TOC](../README.md) | [Next: Zigzag Traversal →](06-zigzag-traversal.md)
