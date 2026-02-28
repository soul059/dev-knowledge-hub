# Unit 8 – Topic 1: Root to Leaf Paths

[← Previous: Symmetric Tree](../07-Binary-Tree-Problems/06-symmetric-tree.md) | [Back to TOC](../README.md) | [Next: Path Sum Problems →](02-path-sum-problems.md)

---

## Chapter Overview

Path problems are among the **most frequently asked** tree interview questions. They involve finding, computing, or validating paths within a tree. This unit covers root-to-leaf paths, path sums, maximum path sums, LCA, distance calculations, and nodes at a given distance.

---

## Reference Tree

```
            1
           / \
          2    3
         / \    \
        4   5    6
           / \
          7   8
```

---

## 8.1 Root to Leaf Paths

### 📖 Problem

Print all paths from the root to every leaf node.

### Algorithm

```
FUNCTION rootToLeafPaths(node, path, result):
    IF node == NULL:
        RETURN
    
    path.append(node.data)
    
    // If it's a leaf, we have a complete path
    IF node.left == NULL AND node.right == NULL:
        result.append(copy of path)
    ELSE:
        rootToLeafPaths(node.left, path, result)
        rootToLeafPaths(node.right, path, result)
    
    path.removeLast()     // BACKTRACK
```

### 🔍 Trace

```
            1
           / \
          2    3
         / \
        4   5

    Call rootToLeafPaths(1, [], [])
    path = [1]
    ├── rootToLeafPaths(2, [1], [])
    │   path = [1, 2]
    │   ├── rootToLeafPaths(4, [1, 2], [])
    │   │   path = [1, 2, 4] → LEAF! → result = [[1,2,4]]
    │   │   backtrack → path = [1, 2]
    │   └── rootToLeafPaths(5, [1, 2], [])
    │       path = [1, 2, 5] → LEAF! → result = [[1,2,4], [1,2,5]]
    │       backtrack → path = [1, 2]
    │   backtrack → path = [1]
    └── rootToLeafPaths(3, [1], [])
        path = [1, 3] → LEAF! → result = [[1,2,4], [1,2,5], [1,3]]
        backtrack → path = [1]
    
    All paths:
    1 → 2 → 4
    1 → 2 → 5
    1 → 3
```

### 💡 Key Technique: Backtracking

> After exploring a subtree, **remove the current node** from the path before returning. This ensures the path is correct for the next branch.

---

### ❓ Revision Question

**Why do we need backtracking in root-to-leaf path problems?**
<details><summary>Answer</summary>After exploring one branch (e.g., left subtree), we need to remove the current node from the path before exploring the sibling branch (right subtree). Otherwise, the path would incorrectly contain nodes from the previously explored branch.</details>

---

[← Previous: Symmetric Tree](../07-Binary-Tree-Problems/06-symmetric-tree.md) | [Back to TOC](../README.md) | [Next: Path Sum Problems →](02-path-sum-problems.md)
