# 3.5 Morris Traversal

[← Previous: Level Order Traversal](04-level-order-traversal.md) | [Back to TOC](../README.md) | [Next: Vertical & Diagonal Traversal →](06-vertical-diagonal-traversal.md)

---

## 📖 Concept

Morris traversal achieves **O(1) space** (no stack, no recursion) by using **threaded links** — temporarily modifying the tree structure to create links from a node's inorder predecessor back to the node.

### 💡 Key Idea

> Instead of using a stack to "remember" where to go back, we create a temporary **thread** (pointer) from a node's inorder predecessor to the node itself. After visiting, we remove the thread to restore the tree.

---

## Morris Inorder Algorithm

```
FUNCTION morrisInorder(root):
    current ← root
    
    WHILE current ≠ NULL:
        IF current.left == NULL:
            PRINT current.data           // No left subtree, visit node
            current ← current.right      // Move to right (or thread)
        ELSE:
            // Find inorder predecessor
            predecessor ← current.left
            WHILE predecessor.right ≠ NULL AND predecessor.right ≠ current:
                predecessor ← predecessor.right
            
            IF predecessor.right == NULL:
                // Create thread
                predecessor.right ← current
                current ← current.left
            ELSE:
                // Thread already exists → left subtree visited
                predecessor.right ← NULL       // Remove thread
                PRINT current.data             // Visit node
                current ← current.right
```

---

## 🔍 Trace: Morris Inorder

```
    Tree:     4
             / \
            2   5
           / \
          1   3

    Step 1: current = 4
            Left exists → find predecessor of 4 in left subtree
            predecessor = 2 → 3 (rightmost in left subtree)
            3.right == NULL → create thread: 3.right = 4
            current = 2
            
                4           (3 now points back to 4)
               / \
              2   5
             / \
            1   3 ─→ (thread to 4)

    Step 2: current = 2
            Left exists → find predecessor of 2
            predecessor = 1
            1.right == NULL → create thread: 1.right = 2
            current = 1
            
    Step 3: current = 1
            Left is NULL → PRINT 1, current = 1.right = 2 (thread!)
            
    Step 4: current = 2
            Left exists → find predecessor
            predecessor = 1, 1.right == current (thread found!)
            Remove thread: 1.right = NULL
            PRINT 2, current = 2.right = 3
            
    Step 5: current = 3
            Left is NULL → PRINT 3, current = 3.right = 4 (thread!)
            
    Step 6: current = 4
            Left exists → find predecessor
            predecessor = 2 → 3, 3.right == current (thread found!)
            Remove thread: 3.right = NULL
            PRINT 4, current = 4.right = 5
            
    Step 7: current = 5
            Left is NULL → PRINT 5, current = 5.right = NULL
            
    Step 8: current = NULL → DONE
    
    Result: 1, 2, 3, 4, 5  ✓
```

---

## ⏱️ Complexity

| Aspect | Value |
|--------|-------|
| Time | O(n) — each edge traversed at most 3 times |
| Space | **O(1)** — no stack, no recursion |

---

## ⚠️ Important Notes

- Morris traversal **temporarily modifies** the tree
- Tree is **restored** to its original state after traversal
- Not suitable when tree cannot be modified (read-only scenarios)
- **Morris preorder variant**: print **when creating** the thread instead of when removing it

---

[← Previous: Level Order Traversal](04-level-order-traversal.md) | [Back to TOC](../README.md) | [Next: Vertical & Diagonal Traversal →](06-vertical-diagonal-traversal.md)
