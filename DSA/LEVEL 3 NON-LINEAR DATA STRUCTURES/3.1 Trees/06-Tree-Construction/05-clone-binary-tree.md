# Unit 6 – Topic 5: Clone a Binary Tree

[← Previous: Serialize & Deserialize](04-serialize-deserialize.md) | [Back to TOC](../README.md) | [Next: Convert Tree Types →](06-convert-tree-types.md)

---

## 6.5 Clone a Binary Tree

### 📖 Problem

Create a deep copy of a binary tree.

### Algorithm

```
FUNCTION cloneTree(node):
    IF node == NULL:
        RETURN NULL
    
    newNode ← new TreeNode(node.data)
    newNode.left  ← cloneTree(node.left)
    newNode.right ← cloneTree(node.right)
    
    RETURN newNode
```

### 🔍 Trace

```
    Original:       1            Clone:        1'
                   / \                        / \
                  2   3                      2'  3'
                 /                          /
                4                          4'
    
    Each node is a NEW object with the same data
    Modifying the clone does NOT affect the original
```

### Cloning with Random Pointers

If nodes have an additional **random pointer**, cloning becomes harder:

```
FUNCTION cloneWithRandom(root):
    IF root == NULL: RETURN NULL
    
    // Step 1: Clone nodes and store mapping
    map ← {}    // original node → cloned node
    
    FUNCTION cloneNodes(node):
        IF node == NULL: RETURN NULL
        clone ← new TreeNode(node.data)
        map[node] ← clone
        clone.left = cloneNodes(node.left)
        clone.right = cloneNodes(node.right)
        RETURN clone
    
    clonedRoot ← cloneNodes(root)
    
    // Step 2: Set random pointers using map
    FUNCTION setRandom(node):
        IF node == NULL: RETURN
        IF node.random ≠ NULL:
            map[node].random ← map[node.random]
        setRandom(node.left)
        setRandom(node.right)
    
    setRandom(root)
    RETURN clonedRoot
```

---

### ❓ Revision Question

**How does cloning a tree with random pointers differ from a normal clone?**
<details><summary>Answer</summary>Normal clone just copies left/right. With random pointers, we need a mapping (hash map) from original to cloned nodes. First clone all nodes, then set random pointers using the map, because the random target might not be cloned yet during the first pass.</details>

---

[← Previous: Serialize & Deserialize](04-serialize-deserialize.md) | [Back to TOC](../README.md) | [Next: Convert Tree Types →](06-convert-tree-types.md)
