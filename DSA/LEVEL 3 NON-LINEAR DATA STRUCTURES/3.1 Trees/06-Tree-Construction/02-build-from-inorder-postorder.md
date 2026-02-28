# Unit 6 – Topic 2: Build from Inorder + Postorder

[← Previous: Build from Inorder + Preorder](01-build-from-inorder-preorder.md) | [Back to TOC](../README.md) | [Next: Build from Level Order →](03-build-from-level-order.md)

---

## 6.2 Build from Inorder + Postorder

### 📖 Problem

Given the inorder and postorder traversals, reconstruct the tree.

### 💡 Key Insight

```
    Postorder: [Left subtree] [Right subtree] [Root]
                                                ↑
                                    LAST element is the ROOT

    Inorder:   [Left subtree] [Root] [Right subtree]
```

> **Strategy**: Pick root from postorder (last element), find it in inorder, recurse. **Build right subtree before left** (since we traverse postorder from the end).

### Algorithm

```
FUNCTION buildTree(inorder, postorder):
    map ← {}
    FOR i ← 0 to inorder.length - 1:
        map[inorder[i]] ← i
    
    postIdx ← postorder.length - 1   // Start from end
    
    FUNCTION build(inLeft, inRight):
        IF inLeft > inRight:
            RETURN NULL
        
        rootVal ← postorder[postIdx]
        postIdx ← postIdx - 1
        root ← new TreeNode(rootVal)
        
        mid ← map[rootVal]
        
        root.right ← build(mid + 1, inRight)   // RIGHT first!
        root.left  ← build(inLeft, mid - 1)
        
        RETURN root
    
    RETURN build(0, inorder.length - 1)
```

### 🔍 Trace

```
    Inorder:   [9, 3, 15, 20, 7]
    Postorder: [9, 15, 7, 20, 3]
    
    Step 1: root = 3 (last in postorder), postIdx = 3
            Find 3 in inorder → index 1
    
    Step 2: Build RIGHT subtree first (inorder [15,20,7])
            root = 20 (postorder[3]), postIdx = 2
            Build right of 20: root = 7 (leaf), postIdx = 1
            Build left of 20:  root = 15 (leaf), postIdx = 0
    
    Step 3: Build LEFT subtree (inorder [9])
            root = 9 (postorder[0]), leaf
    
    Result:
            3
           / \
          9   20
             / \
            15   7
    
    Same tree as before! ✓
```

### ⚠️ Important

> When building from postorder, you **must build the right subtree before the left** because the postorder index decrements from the end, and right subtree elements come just before the root.

---

### ❓ Revision Question

**When building from inorder + postorder, why must we build the right subtree before the left?**
<details><summary>Answer</summary>Because we consume the postorder array from right to left. In postorder (L, R, Root), the root is last, followed by right subtree elements, then left. So when we decrement the postorder index, right subtree elements come first.</details>

---

[← Previous: Build from Inorder + Preorder](01-build-from-inorder-preorder.md) | [Back to TOC](../README.md) | [Next: Build from Level Order →](03-build-from-level-order.md)
