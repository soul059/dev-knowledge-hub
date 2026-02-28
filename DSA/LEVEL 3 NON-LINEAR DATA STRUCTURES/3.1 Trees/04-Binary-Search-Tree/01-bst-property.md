# 4.1 BST Property

[← Previous Unit: Vertical & Diagonal Traversal](../03-Tree-Traversals/06-vertical-diagonal-traversal.md) | [Back to TOC](../README.md) | [Next: BST Search →](02-bst-search.md)

---

## 📖 Definition

A **Binary Search Tree** is a binary tree where for **every** node:
- All values in the **left subtree** are **less than** the node's value
- All values in the **right subtree** are **greater than** the node's value
- Both left and right subtrees are also BSTs

```
    Valid BST:                  Invalid BST:
    
         8                          8
        / \                        / \
       3   10                     3   10
      / \    \                   / \    \
     1   6   14                 1   9   14    ← 9 > 8, but in left
        / \  /                     / \  /       subtree of 8!
       4  7 13                    4  7 13
```

---

## 💡 Key Insight

> The BST property must hold for the **entire subtree**, not just the immediate children. Node 9 in the invalid example above is a valid right child of 3, but it violates the BST property with respect to node 8 (the root).

---

## Inorder Traversal → Sorted Order

```
         8
        / \
       3   10
      / \    \
     1   6   14
        / \  /
       4  7 13

    Inorder: 1, 3, 4, 6, 7, 8, 10, 13, 14  ← SORTED!
```

> **This is the defining insight**: An inorder traversal of a valid BST always produces elements in **ascending sorted order**.

---

[← Previous Unit: Vertical & Diagonal Traversal](../03-Tree-Traversals/06-vertical-diagonal-traversal.md) | [Back to TOC](../README.md) | [Next: BST Search →](02-bst-search.md)
