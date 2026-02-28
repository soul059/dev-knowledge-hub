# 2.6 Height Calculation

[← Previous: Properties and Theorems](05-properties-and-theorems.md) | [Back to TOC](../README.md) | [Next Unit: Tree Traversals →](../03-Tree-Traversals/01-inorder-traversal.md)

---

## 📖 Recursive Approach

```
FUNCTION height(node):
    IF node == NULL:
        RETURN -1     // -1 if counting edges, 0 if counting nodes
    
    RETURN 1 + MAX(height(node.left), height(node.right))
```

---

## 🔍 Detailed Trace

```
    Tree:        1
                / \
               2   3
              / \
             4   5
            /
           6

    Call Stack Trace:
    
    height(1)
    ├── height(2)
    │   ├── height(4)
    │   │   ├── height(6)
    │   │   │   ├── height(NULL) → -1
    │   │   │   └── height(NULL) → -1
    │   │   │   → 1 + max(-1,-1) = 0
    │   │   └── height(NULL) → -1
    │   │   → 1 + max(0,-1) = 1
    │   └── height(5)
    │       ├── height(NULL) → -1
    │       └── height(NULL) → -1
    │       → 1 + max(-1,-1) = 0
    │   → 1 + max(1,0) = 2
    └── height(3)
        ├── height(NULL) → -1
        └── height(NULL) → -1
        → 1 + max(-1,-1) = 0
    → 1 + max(2,0) = 3
    
    Final Answer: Height = 3
```

---

## Iterative Approach (Level Order)

```
FUNCTION heightIterative(root):
    IF root == NULL:
        RETURN -1
    
    queue ← empty queue
    queue.enqueue(root)
    height ← -1
    
    WHILE queue is not empty:
        levelSize ← queue.size()
        height ← height + 1
        
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            IF node.left ≠ NULL:
                queue.enqueue(node.left)
            IF node.right ≠ NULL:
                queue.enqueue(node.right)
    
    RETURN height
```

---

## 🔍 Trace: Iterative Height

```
    Tree:     1
             / \
            2   3
           /
          4

    Step 1: queue = [1],       height = -1
    Step 2: Process level 0    height = 0, queue = [2, 3]
    Step 3: Process level 1    height = 1, queue = [4]
    Step 4: Process level 2    height = 2, queue = []
    Step 5: Queue empty → return 2
```

---

## ⏱️ Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Recursive | O(n) — visit every node | O(h) — call stack depth |
| Iterative (BFS) | O(n) — visit every node | O(w) — max width of tree |

Where `h` = height and `w` = maximum width. For a balanced tree, `w` can be up to `n/2`.

---

[← Previous: Properties and Theorems](05-properties-and-theorems.md) | [Back to TOC](../README.md) | [Next Unit: Tree Traversals →](../03-Tree-Traversals/01-inorder-traversal.md)
