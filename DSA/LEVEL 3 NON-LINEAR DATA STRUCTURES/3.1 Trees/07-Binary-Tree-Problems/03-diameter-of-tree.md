# Unit 7 – Topic 3: Diameter of Tree

[← Previous: Minimum Depth](02-minimum-depth.md) | [Back to TOC](../README.md) | [Next: Check Balanced Tree →](04-check-balanced-tree.md)

---

## 7.3 Diameter of Tree

### 📖 Problem

The **diameter** (width) of a tree is the number of **edges** (or nodes) on the **longest path between any two nodes**. This path may or may not pass through the root.

```
    Example:
            1
           / \
          2    3
         / \
        4   5
       /     \
      8       6
               \
                7
    
    Longest path: 8 → 4 → 2 → 5 → 6 → 7
    Diameter = 5 edges (or 6 nodes)
    
    Note: This path does NOT go through root 1!
```

### 💡 Key Insight

> At every node, the diameter passing through it = height(left) + height(right). The overall diameter = maximum of this value across all nodes. We can compute this in **a single pass** by calculating height and updating diameter simultaneously.

### Algorithm

```
FUNCTION diameter(root):
    maxDia ← 0
    
    FUNCTION height(node):
        IF node == NULL:
            RETURN 0
        
        leftH  ← height(node.left)
        rightH ← height(node.right)
        
        // Update diameter: path through this node
        maxDia ← MAX(maxDia, leftH + rightH)
        
        RETURN 1 + MAX(leftH, rightH)
    
    height(root)
    RETURN maxDia
```

### 🔍 Trace

```
            1
           / \
          2    3
         / \
        4   5

    height(4) → leftH=0, rightH=0, dia=0+0=0, return 1
    height(5) → leftH=0, rightH=0, dia=0+0=0, return 1
    height(2) → leftH=1, rightH=1, dia=1+1=2 ✓ maxDia=2, return 2
    height(3) → leftH=0, rightH=0, dia=0+0=0, return 1
    height(1) → leftH=2, rightH=1, dia=2+1=3 ✓ maxDia=3, return 3
    
    Diameter = 3 (path: 4→2→1→3 or 5→2→1→3)
```

### ⏱️ Complexity: O(n) time, O(h) space

---

### ❓ Revision Question

**Why can the diameter of a tree NOT pass through the root?**
<details><summary>Answer</summary>The longest path between any two nodes might be entirely within one subtree. For example, in a tree where the left subtree is very deep and wide but the right subtree is shallow, the longest path could be entirely in the left subtree.</details>

---

[← Previous: Minimum Depth](02-minimum-depth.md) | [Back to TOC](../README.md) | [Next: Check Balanced Tree →](04-check-balanced-tree.md)
