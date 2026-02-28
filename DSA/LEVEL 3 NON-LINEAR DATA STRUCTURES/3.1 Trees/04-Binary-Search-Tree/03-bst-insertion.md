# 4.3 BST Insertion Operation

[← Previous: BST Search](02-bst-search.md) | [Back to TOC](../README.md) | [Next: BST Deletion →](04-bst-deletion.md)

---

## 📖 Concept

Insertion always happens at a **leaf position**. We search for the correct position and create a new node there.

---

## Algorithm

```
FUNCTION insert(node, key):
    IF node == NULL:
        RETURN new TreeNode(key)     // Create new leaf
    
    IF key < node.data:
        node.left ← insert(node.left, key)
    ELSE IF key > node.data:
        node.right ← insert(node.right, key)
    // If key == node.data, duplicate — ignore or handle
    
    RETURN node
```

---

## 🔍 Trace: Insert 5

```
         8                    8
        / \                  / \
       3   10               3   10
      / \    \             / \    \
     1   6   14           1   6   14
        /                    /
       4                    4
                             \
                              5  ← NEW NODE
    
    Step 1: 5 < 8  → go left to 3
    Step 2: 5 > 3  → go right to 6
    Step 3: 5 < 6  → go left to 4
    Step 4: 5 > 4  → go right → NULL → create node(5)
```

---

## Building a BST from Scratch

```
    Insert sequence: 8, 3, 10, 1, 6, 14, 4, 7

    Insert 8:    8          Insert 3:    8       Insert 10:    8
                                        /                     / \
                                       3                     3   10
    
    Insert 1:    8          Insert 6:    8       Insert 14:    8
                / \                    / \                    / \
               3   10                 3   10                 3   10
              /                      / \                    / \    \
             1                      1   6                  1   6   14
    
    Insert 4:    8          Insert 7:    8
                / \                    / \
               3   10                 3   10
              / \    \               / \    \
             1   6   14             1   6   14
                /                      / \
               4                      4   7
```

---

## ⚠️ Order Matters!

```
    Same elements, different insertion orders:
    
    [1, 2, 3, 4, 5]:          [3, 1, 5, 2, 4]:
    
    1                              3
     \                            / \
      2                          1   5
       \                          \ /
        3                         2 4
         \
          4            Balanced! h = 2
           \
            5
            
    Skewed! h = 4        (Much better!)
```

---

## ⏱️ Complexity

| Case | Time |
|------|------|
| Average | O(log n) |
| Worst (skewed) | O(n) |
| Space | O(h) recursive |

---

[← Previous: BST Search](02-bst-search.md) | [Back to TOC](../README.md) | [Next: BST Deletion →](04-bst-deletion.md)
