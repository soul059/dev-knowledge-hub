# Unit 7 – Topic 4: Check Balanced Tree

[← Previous: Diameter of Tree](03-diameter-of-tree.md) | [Back to TOC](../README.md) | [Next: Same Tree & Subtree →](05-same-tree-subtree.md)

---

## 7.4 Check Balanced Tree

### 📖 Problem

Determine if a binary tree is **height-balanced** — for every node, the height difference of left and right subtrees is at most 1.

### Naive Approach: O(n²)

```
FUNCTION isBalancedNaive(node):
    IF node == NULL: RETURN true
    
    leftH  ← height(node.left)     // O(n)
    rightH ← height(node.right)    // O(n)
    
    IF |leftH - rightH| > 1:
        RETURN false
    
    RETURN isBalancedNaive(node.left) AND isBalancedNaive(node.right)
    
    // Problem: height() is called repeatedly → O(n²)
```

### Optimized Approach: O(n)

Compute height and check balance in a **single pass**. Return `-1` if unbalanced.

```
FUNCTION isBalanced(root):
    
    FUNCTION checkHeight(node):
        IF node == NULL:
            RETURN 0
        
        leftH ← checkHeight(node.left)
        IF leftH == -1: RETURN -1       // Left subtree unbalanced
        
        rightH ← checkHeight(node.right)
        IF rightH == -1: RETURN -1      // Right subtree unbalanced
        
        IF |leftH - rightH| > 1:
            RETURN -1                    // Current node unbalanced
        
        RETURN 1 + MAX(leftH, rightH)
    
    RETURN checkHeight(root) ≠ -1
```

### 🔍 Trace

```
    Balanced:               Unbalanced:
    
        1                       1
       / \                     / \
      2   3                   2   3
     / \                     /
    4   5                   4
                           /
                          5
    
    checkHeight on balanced tree:
    h(4)=1, h(5)=1
    h(2): |1-1|=0 ≤ 1 ✓, return 2
    h(3): return 1
    h(1): |2-1|=1 ≤ 1 ✓ → BALANCED
    
    checkHeight on unbalanced tree:
    h(5)=1, h(4): |1-0|=1 ✓, return 2
    h(2): |2-0|=2 > 1 → return -1 → UNBALANCED
```

### ⏱️ Complexity: O(n) time, O(h) space

---

### ❓ Revision Question

**How does the optimized balanced tree check achieve O(n) instead of O(n²)?**
<details><summary>Answer</summary>Instead of computing height separately (which re-traverses nodes), it computes height and checks balance in a single bottom-up pass. The sentinel value -1 indicates "unbalanced" and short-circuits further computation.</details>

---

[← Previous: Diameter of Tree](03-diameter-of-tree.md) | [Back to TOC](../README.md) | [Next: Same Tree & Subtree →](05-same-tree-subtree.md)
