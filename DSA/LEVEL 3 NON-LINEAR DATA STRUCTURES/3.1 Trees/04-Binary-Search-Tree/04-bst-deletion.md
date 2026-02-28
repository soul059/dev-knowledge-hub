# 4.4 BST Deletion Operation

[← Previous: BST Insertion](03-bst-insertion.md) | [Back to TOC](../README.md) | [Next: BST Validation →](05-bst-validation.md)

---

## 📖 Concept

Deletion is the most complex BST operation. There are **three cases**:

```
    Case 1: Node is a LEAF (0 children)
    → Simply remove it
    
    Case 2: Node has ONE child
    → Replace node with its child
    
    Case 3: Node has TWO children
    → Replace with inorder successor (or predecessor)
       then delete the successor
```

---

## Algorithm

```
FUNCTION delete(node, key):
    IF node == NULL:
        RETURN NULL
    
    // Step 1: Find the node
    IF key < node.data:
        node.left ← delete(node.left, key)
    ELSE IF key > node.data:
        node.right ← delete(node.right, key)
    ELSE:
        // Found the node to delete
        
        // Case 1: Leaf node
        IF node.left == NULL AND node.right == NULL:
            RETURN NULL
        
        // Case 2a: Only right child
        ELSE IF node.left == NULL:
            RETURN node.right
        
        // Case 2b: Only left child
        ELSE IF node.right == NULL:
            RETURN node.left
        
        // Case 3: Two children
        ELSE:
            successor ← findMin(node.right)    // Inorder successor
            node.data ← successor.data         // Copy data
            node.right ← delete(node.right, successor.data)  // Delete successor
    
    RETURN node

FUNCTION findMin(node):
    WHILE node.left ≠ NULL:
        node ← node.left
    RETURN node
```

---

## 🔍 Trace: Case 1 — Delete Leaf (Delete 4)

```
    Before:          After:
    
         8              8
        / \            / \
       3   10         3   10
      / \            / \
     1   6          1   6
        /
       4
    
    4 is a leaf → simply remove it (return NULL)
```

---

## 🔍 Trace: Case 2 — Delete Node with One Child (Delete 10)

```
    Before:          After:
    
         8              8
        / \            / \
       3   10         3   14
      / \    \       / \
     1   6   14     1   6
    
    10 has only right child (14)
    → Replace 10 with 14
```

---

## 🔍 Trace: Case 3 — Delete Node with Two Children (Delete 3)

```
    Before:                After:
    
         8                    8
        / \                  / \
       3   10               4   10
      / \    \             / \    \
     1   6   14           1   6   14
        / \                    \
       4   7                    7
    
    Step 1: Node 3 has two children
    Step 2: Find inorder successor = smallest in right subtree
            Right subtree of 3: {6, 4, 7}
            Smallest = 4 (leftmost in right subtree)
    Step 3: Copy 4's data to node 3's position
    Step 4: Delete 4 from right subtree (it's a leaf → Case 1)
```

---

## 💡 Why Inorder Successor?

```
    When deleting a node with 2 children, we need a replacement that:
    ✓ Is greater than everything in left subtree
    ✓ Is less than everything in right subtree
    
    Inorder successor (smallest in right subtree) satisfies both!
    Alternatively, inorder predecessor (largest in left subtree) also works.
```

---

## ⏱️ Complexity

| Case | Time |
|------|------|
| Average | O(log n) |
| Worst (skewed) | O(n) |
| Space | O(h) recursive |

---

[← Previous: BST Insertion](03-bst-insertion.md) | [Back to TOC](../README.md) | [Next: BST Validation →](05-bst-validation.md)
