# Unit 6 – Topic 3: Build from Level Order

[← Previous: Build from Inorder + Postorder](02-build-from-inorder-postorder.md) | [Back to TOC](../README.md) | [Next: Serialize & Deserialize →](04-serialize-deserialize.md)

---

## 6.3 Build from Level Order

### 📖 Problem

Given a level-order traversal (with NULLs indicating missing children), reconstruct the tree.

### Algorithm

```
FUNCTION buildFromLevelOrder(arr):
    IF arr is empty OR arr[0] == NULL:
        RETURN NULL
    
    root ← new TreeNode(arr[0])
    queue ← [root]
    i ← 1
    
    WHILE queue is not empty AND i < arr.length:
        node ← queue.dequeue()
        
        // Left child
        IF i < arr.length AND arr[i] ≠ NULL:
            node.left ← new TreeNode(arr[i])
            queue.enqueue(node.left)
        i ← i + 1
        
        // Right child
        IF i < arr.length AND arr[i] ≠ NULL:
            node.right ← new TreeNode(arr[i])
            queue.enqueue(node.right)
        i ← i + 1
    
    RETURN root
```

### 🔍 Trace

```
    Input: [1, 2, 3, 4, 5, NULL, 6]
    
    Step 1: root = 1, queue = [1], i = 1
    
    Step 2: node = 1
            arr[1]=2 → 1.left = 2, queue = [2]
            arr[2]=3 → 1.right = 3, queue = [2, 3]
            i = 3
    
    Step 3: node = 2
            arr[3]=4 → 2.left = 4, queue = [3, 4]
            arr[4]=5 → 2.right = 5, queue = [3, 4, 5]
            i = 5
    
    Step 4: node = 3
            arr[5]=NULL → skip
            arr[6]=6 → 3.right = 6, queue = [4, 5, 6]
            i = 7
    
    Result:
            1
           / \
          2   3
         / \   \
        4   5   6
```

---

[← Previous: Build from Inorder + Postorder](02-build-from-inorder-postorder.md) | [Back to TOC](../README.md) | [Next: Serialize & Deserialize →](04-serialize-deserialize.md)
