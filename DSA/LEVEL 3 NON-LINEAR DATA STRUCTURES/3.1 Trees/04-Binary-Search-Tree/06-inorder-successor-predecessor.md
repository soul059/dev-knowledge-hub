# 4.6 Inorder Successor & Predecessor

[← Previous: BST Validation](05-bst-validation.md) | [Back to TOC](../README.md) | [Next Unit: BST Problems →](../05-BST-Problems/01-sorted-array-to-bst.md)

---

## 📖 Definition

- **Inorder Successor**: The node with the **smallest value greater** than the given node
- **Inorder Predecessor**: The node with the **largest value smaller** than the given node

```
    Inorder: 1, 3, 4, 6, 7, 8, 10, 13, 14
    
    Successor of 6   = 7
    Predecessor of 6 = 4
    Successor of 8   = 10
    Predecessor of 3 = 1
```

---

## Algorithm: Inorder Successor

```
FUNCTION inorderSuccessor(root, target):
    successor ← NULL
    
    // Case 1: Target has a right subtree
    // Successor = leftmost node in right subtree
    IF target.right ≠ NULL:
        RETURN findMin(target.right)
    
    // Case 2: No right subtree
    // Successor = nearest ancestor for which target is in LEFT subtree
    current ← root
    WHILE current ≠ NULL:
        IF target.data < current.data:
            successor ← current        // Potential successor
            current ← current.left
        ELSE IF target.data > current.data:
            current ← current.right
        ELSE:
            BREAK
    
    RETURN successor
```

---

## 🔍 Trace: Find Successor of 6

```
         8
        / \
       3   10
      / \    \
     1   6   14
        / \  /
       4  7 13

    Case 1: Node 6 has right subtree (node 7)
    → Successor = leftmost in right subtree = 7 ✓
```

## 🔍 Trace: Find Successor of 7

```
    Case 2: Node 7 has NO right subtree
    → Search from root:
    
    current = 8:  7 < 8  → successor = 8, go left
    current = 3:  7 > 3  → go right
    current = 6:  7 > 6  → go right
    current = 7:  found target, break
    
    → Successor = 8 ✓
```

---

## Algorithm: Inorder Predecessor

```
FUNCTION inorderPredecessor(root, target):
    predecessor ← NULL
    
    // Case 1: Target has a left subtree
    IF target.left ≠ NULL:
        RETURN findMax(target.left)     // Rightmost in left subtree
    
    // Case 2: No left subtree
    current ← root
    WHILE current ≠ NULL:
        IF target.data > current.data:
            predecessor ← current       // Potential predecessor
            current ← current.right
        ELSE IF target.data < current.data:
            current ← current.left
        ELSE:
            BREAK
    
    RETURN predecessor
```

---

## Visual Summary: Successor Cases

```
    Case 1: Right subtree EXISTS           Case 2: No right subtree
    
         ...                                    ...
          |                                      |
        target                               ancestor ← SUCCESSOR
        /    \                                  /
      ...   right                            ...
              /                                \
           leftmost ← SUCCESSOR              target
                                               /
                                             ...
    
    Go right, then all the way left         Go up until you find an
                                            ancestor where target is
                                            in the LEFT subtree
```

---

## ⏱️ Complexity

| Operation | Time |
|-----------|------|
| Successor / Predecessor | O(h) |
| Space | O(1) iterative |

---

[← Previous: BST Validation](05-bst-validation.md) | [Back to TOC](../README.md) | [Next Unit: BST Problems →](../05-BST-Problems/01-sorted-array-to-bst.md)
