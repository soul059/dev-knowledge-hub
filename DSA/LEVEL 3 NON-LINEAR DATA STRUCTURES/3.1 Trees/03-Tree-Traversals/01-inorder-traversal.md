# 3.1 Inorder Traversal (Left → Root → Right)

[← Previous Unit: Height Calculation](../02-Binary-Tree-Basics/06-height-calculation.md) | [Back to TOC](../README.md) | [Next: Preorder Traversal →](02-preorder-traversal.md)

---

## Reference Tree (Used Throughout Unit 3)

```
            1
           / \
          2    3
         / \    \
        4   5    6
           / \
          7   8
```

**Node count**: 8 | **Height**: 3

---

## 📖 Concept

Visit the **left subtree** first, then the **root**, then the **right subtree**. For BSTs, inorder traversal gives nodes in **sorted order**.

```
    Pattern: Left → Root → Right   (LNR)
    
         N
        / \
       L   R       Visit order: L, then N, then R
```

---

## Recursive Approach

```
FUNCTION inorder(node):
    IF node == NULL:
        RETURN
    
    inorder(node.left)       // Visit left subtree
    PRINT node.data          // Visit root
    inorder(node.right)      // Visit right subtree
```

---

## 🔍 Trace: Recursive Inorder

```
            1
           / \
          2    3
         / \    \
        4   5    6
           / \
          7   8

    inorder(1)
    ├── inorder(2)
    │   ├── inorder(4)
    │   │   ├── inorder(NULL) → return
    │   │   ├── PRINT 4  ←────────── ①
    │   │   └── inorder(NULL) → return
    │   ├── PRINT 2  ←────────────── ②
    │   └── inorder(5)
    │       ├── inorder(7)
    │       │   ├── inorder(NULL) → return
    │       │   ├── PRINT 7  ←────── ③
    │       │   └── inorder(NULL) → return
    │       ├── PRINT 5  ←────────── ④
    │       └── inorder(8)
    │           ├── inorder(NULL) → return
    │           ├── PRINT 8  ←────── ⑤
    │           └── inorder(NULL) → return
    ├── PRINT 1  ←────────────────── ⑥
    └── inorder(3)
        ├── inorder(NULL) → return
        ├── PRINT 3  ←────────────── ⑦
        └── inorder(6)
            ├── inorder(NULL) → return
            ├── PRINT 6  ←────────── ⑧
            └── inorder(NULL) → return

    Result: 4, 2, 7, 5, 8, 1, 3, 6
```

---

## Iterative Approach (Using Stack)

```
FUNCTION inorderIterative(root):
    stack ← empty stack
    current ← root
    
    WHILE current ≠ NULL OR stack is not empty:
        // Go as far left as possible
        WHILE current ≠ NULL:
            stack.push(current)
            current ← current.left
        
        // Process the node
        current ← stack.pop()
        PRINT current.data
        
        // Move to right subtree
        current ← current.right
```

---

## 🔍 Trace: Iterative Inorder

```
            1
           / \
          2    3
         / \
        4   5

    Step | Stack (top→)  | current | Action         | Output
    -----+---------------+---------+----------------+---------
      1  | []            | 1       | push 1, go left|
      2  | [1]           | 2       | push 2, go left|
      3  | [1,2]         | 4       | push 4, go left|
      4  | [1,2,4]       | NULL    | pop 4, print   | 4
      5  | [1,2]         | NULL    | (4.right=NULL) |
      6  | [1,2]         | NULL    | pop 2, print   | 4,2
      7  | [1]           | 5       | push 5, go left|
      8  | [1,5]         | NULL    | pop 5, print   | 4,2,5
      9  | [1]           | NULL    | pop 1, print   | 4,2,5,1
     10  | []            | 3       | push 3, go left|
     11  | [3]           | NULL    | pop 3, print   | 4,2,5,1,3
     12  | []            | NULL    | stack empty,done| 
    
    Result: 4, 2, 5, 1, 3
```

---

## ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Recursive | O(n) | O(h) — call stack |
| Iterative | O(n) | O(h) — explicit stack |

---

[← Previous Unit: Height Calculation](../02-Binary-Tree-Basics/06-height-calculation.md) | [Back to TOC](../README.md) | [Next: Preorder Traversal →](02-preorder-traversal.md)
