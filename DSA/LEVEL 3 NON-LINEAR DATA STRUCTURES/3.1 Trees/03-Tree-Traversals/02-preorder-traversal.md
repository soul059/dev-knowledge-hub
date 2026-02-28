# 3.2 Preorder Traversal (Root → Left → Right)

[← Previous: Inorder Traversal](01-inorder-traversal.md) | [Back to TOC](../README.md) | [Next: Postorder Traversal →](03-postorder-traversal.md)

---

## 📖 Concept

Visit the **root** first, then the **left subtree**, then the **right subtree**. Preorder is useful for **copying/cloning** trees and generating **prefix expressions**.

```
    Pattern: Root → Left → Right   (NLR)
    
         N
        / \
       L   R       Visit order: N, then L, then R
```

---

## Recursive Approach

```
FUNCTION preorder(node):
    IF node == NULL:
        RETURN
    
    PRINT node.data          // Visit root FIRST
    preorder(node.left)      // Visit left subtree
    preorder(node.right)     // Visit right subtree
```

---

## 🔍 Trace: Recursive Preorder

```
            1
           / \
          2    3
         / \    \
        4   5    6
           / \
          7   8

    preorder(1)
    ├── PRINT 1  ←────────────────── ①
    ├── preorder(2)
    │   ├── PRINT 2  ←────────────── ②
    │   ├── preorder(4)
    │   │   ├── PRINT 4  ←────────── ③
    │   │   ├── preorder(NULL)
    │   │   └── preorder(NULL)
    │   └── preorder(5)
    │       ├── PRINT 5  ←────────── ④
    │       ├── preorder(7)
    │       │   ├── PRINT 7  ←────── ⑤
    │       │   ├── preorder(NULL)
    │       │   └── preorder(NULL)
    │       └── preorder(8)
    │           ├── PRINT 8  ←────── ⑥
    │           ├── preorder(NULL)
    │           └── preorder(NULL)
    └── preorder(3)
        ├── PRINT 3  ←────────────── ⑦
        ├── preorder(NULL)
        └── preorder(6)
            ├── PRINT 6  ←────────── ⑧
            ├── preorder(NULL)
            └── preorder(NULL)

    Result: 1, 2, 4, 5, 7, 8, 3, 6
```

---

## Iterative Approach (Using Stack)

```
FUNCTION preorderIterative(root):
    IF root == NULL:
        RETURN
    
    stack ← empty stack
    stack.push(root)
    
    WHILE stack is not empty:
        node ← stack.pop()
        PRINT node.data
        
        // Push RIGHT first (so LEFT is processed first)
        IF node.right ≠ NULL:
            stack.push(node.right)
        IF node.left ≠ NULL:
            stack.push(node.left)
```

### 💡 Key Insight

> Push **right child before left** child onto the stack, because a stack is LIFO — so left will be popped (and processed) first.

---

## 🔍 Trace: Iterative Preorder

```
            1
           / \
          2    3
         / \
        4   5

    Step | Stack (top→)  | Action         | Output
    -----+---------------+----------------+---------
      1  | [1]           | pop 1, print   | 1
      2  | [3, 2]        | push R=3, L=2  |
      3  | [3]           | pop 2, print   | 1,2
      4  | [3, 5, 4]     | push R=5, L=4  |
      5  | [3, 5]        | pop 4, print   | 1,2,4
      6  | [3, 5]        | 4 has no child |
      7  | [3]           | pop 5, print   | 1,2,4,5
      8  | [3]           | 5 has no child |
      9  | []            | pop 3, print   | 1,2,4,5,3
    
    Result: 1, 2, 4, 5, 3
```

---

## ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Recursive | O(n) | O(h) — call stack |
| Iterative | O(n) | O(h) — explicit stack |

---

[← Previous: Inorder Traversal](01-inorder-traversal.md) | [Back to TOC](../README.md) | [Next: Postorder Traversal →](03-postorder-traversal.md)
