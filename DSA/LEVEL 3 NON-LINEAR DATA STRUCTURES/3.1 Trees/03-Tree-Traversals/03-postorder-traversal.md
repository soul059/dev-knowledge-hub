# 3.3 Postorder Traversal (Left → Right → Root)

[← Previous: Preorder Traversal](02-preorder-traversal.md) | [Back to TOC](../README.md) | [Next: Level Order Traversal →](04-level-order-traversal.md)

---

## 📖 Concept

Visit the **left subtree**, then the **right subtree**, then the **root** last. Useful for **deleting trees** and evaluating **postfix expressions**.

```
    Pattern: Left → Right → Root   (LRN)
    
         N
        / \
       L   R       Visit order: L, then R, then N
```

---

## Recursive Approach

```
FUNCTION postorder(node):
    IF node == NULL:
        RETURN
    
    postorder(node.left)     // Visit left subtree
    postorder(node.right)    // Visit right subtree
    PRINT node.data          // Visit root LAST
```

---

## 🔍 Trace: Recursive Postorder

```
            1
           / \
          2    3
         / \    \
        4   5    6
           / \
          7   8

    Result: 4, 7, 8, 5, 2, 6, 3, 1
    
    (Leaves are visited first, root is visited last)
```

---

## Iterative Approach (Using Two Stacks)

```
FUNCTION postorderIterative(root):
    IF root == NULL:
        RETURN
    
    stack1 ← empty stack
    stack2 ← empty stack
    
    stack1.push(root)
    
    WHILE stack1 is not empty:
        node ← stack1.pop()
        stack2.push(node)
        
        IF node.left ≠ NULL:
            stack1.push(node.left)
        IF node.right ≠ NULL:
            stack1.push(node.right)
    
    WHILE stack2 is not empty:
        PRINT stack2.pop().data
```

---

## Iterative Approach (Using One Stack)

```
FUNCTION postorderOneStack(root):
    stack ← empty stack
    current ← root
    lastVisited ← NULL
    
    WHILE current ≠ NULL OR stack is not empty:
        WHILE current ≠ NULL:
            stack.push(current)
            current ← current.left
        
        peekNode ← stack.top()
        
        // If right child exists and not yet processed
        IF peekNode.right ≠ NULL AND lastVisited ≠ peekNode.right:
            current ← peekNode.right
        ELSE:
            PRINT peekNode.data
            lastVisited ← stack.pop()
```

---

## 🔍 Trace: Two-Stack Postorder

```
            1
           / \
          2    3
         / \
        4   5

    Phase 1: Fill stack2 (modified preorder: Root, Right, Left)
    
    Step | Stack1     | Stack2      | Action
    -----+------------+-------------+--------
      1  | [1]        | []          | 
      2  | [2,3]      | [1]        | pop 1, push to s2, push L=2,R=3
      3  | [2]        | [1,3]      | pop 3, push to s2 (no children)
      4  | [4,5]      | [1,3,2]    | pop 2, push to s2, push L=4,R=5
      5  | [4]        | [1,3,2,5]  | pop 5, push to s2
      6  | []         | [1,3,2,5,4]| pop 4, push to s2
    
    Phase 2: Pop all from stack2
    Result: 4, 5, 2, 3, 1  ✓
```

---

## 💡 Traversal Comparison (Memory Trick)

```
            1
           / \
          2    3
         / \
        4   5

    Inorder   (L,N,R):  4, 2, 5, 1, 3
    Preorder  (N,L,R):  1, 2, 4, 5, 3
    Postorder (L,R,N):  4, 5, 2, 3, 1
    
    Think of where the "N" (Node/Root) goes:
    
    Pre-order:  N L R  → Node is FIRST  (pre = before)
    In-order:   L N R  → Node is in the MIDDLE
    Post-order: L R N  → Node is LAST   (post = after)
```

---

## ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Recursive | O(n) | O(h) — call stack |
| Two stacks | O(n) | O(n) — second stack holds all nodes |
| One stack | O(n) | O(h) — explicit stack |

---

[← Previous: Preorder Traversal](02-preorder-traversal.md) | [Back to TOC](../README.md) | [Next: Level Order Traversal →](04-level-order-traversal.md)
