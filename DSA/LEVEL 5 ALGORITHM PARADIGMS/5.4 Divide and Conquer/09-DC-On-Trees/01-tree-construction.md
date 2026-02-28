# Chapter 1: Tree Construction with Divide and Conquer

## Overview

Many tree construction problems use D&C naturally: split input into parts, build subtrees recursively, and connect them to a root. This chapter explores how D&C builds trees from various input formats — sorted arrays, preorder/inorder traversals, and more.

---

## Why Trees and D&C Are Natural Partners

```
Trees ARE recursive structures:

    A tree is either:
      - Empty (base case)
      - A root node with left and right subtrees

        ┌──────────┐
        │   Root   │
        └────┬─────┘
        ┌────┴─────┐
    ┌───┴───┐  ┌───┴───┐
    │ Left  │  │ Right │
    │ Tree  │  │ Tree  │
    └───────┘  └───────┘

This directly maps to D&C:
  DIVIDE:  Split input for left subtree and right subtree
  CONQUER: Recursively build left and right subtrees
  COMBINE: Attach subtrees to root node
```

---

## Problem 1: Build BST from Sorted Array

```
INPUT:  Sorted array [1, 2, 3, 4, 5, 6, 7]
OUTPUT: Height-balanced BST

       ┌───4───┐
       │       │
    ┌──2──┐ ┌──6──┐
    │     │ │     │
    1     3 5     7

D&C Approach:
  1. DIVIDE:  Pick middle element as root
  2. CONQUER: Left half → left subtree
             Right half → right subtree
  3. COMBINE: Attach subtrees to root
```

### Algorithm

```
BUILD_BST(arr, lo, hi):
    if lo > hi:
        return null
    
    mid ← (lo + hi) / 2
    node ← new Node(arr[mid])
    node.left  ← BUILD_BST(arr, lo, mid-1)
    node.right ← BUILD_BST(arr, mid+1, hi)
    
    return node

// Call: BUILD_BST(arr, 0, n-1)
```

### Trace

```
arr = [1, 2, 3, 4, 5, 6, 7]

BUILD_BST(0, 6):  mid=3 → root=4
  ├── BUILD_BST(0, 2):  mid=1 → root=2
  │     ├── BUILD_BST(0, 0):  mid=0 → root=1
  │     │     ├── BUILD_BST(0, -1): null
  │     │     └── BUILD_BST(1, 0): null
  │     └── BUILD_BST(2, 2):  mid=2 → root=3
  │           ├── BUILD_BST(2, 1): null
  │           └── BUILD_BST(3, 2): null
  └── BUILD_BST(4, 6):  mid=5 → root=6
        ├── BUILD_BST(4, 4):  mid=4 → root=5
        │     ├── null
        │     └── null
        └── BUILD_BST(6, 6):  mid=6 → root=7
              ├── null
              └── null

Result:
         4
       /   \
      2     6
     / \   / \
    1   3 5   7

Height: ⌈log₂(7+1)⌉ = 3 — optimal!
```

### Complexity

```
T(n) = 2T(n/2) + O(1)

Master Theorem: a=2, b=2, f(n)=O(1)
  log_b a = 1, n^(log_b a) = n
  f(n) = O(1) = O(n^(1-1)) → Case 1

  T(n) = Θ(n)

Space: O(log n) recursion stack + O(n) for tree = O(n)
```

---

## Problem 2: Construct Binary Tree from Preorder and Inorder

```
INPUT:
  Preorder: [3, 9, 20, 15, 7]
  Inorder:  [9, 3, 15, 20, 7]

OUTPUT:
        3
       / \
      9   20
         /  \
        15   7

KEY INSIGHT:
  - First element of preorder = ROOT
  - Find root in inorder → splits into left and right subtrees
  - Sizes of left/right determine splits in preorder
```

### Algorithm

```
BUILD_TREE(preorder, inorder):
    if preorder is empty:
        return null
    
    root_val ← preorder[0]
    root ← new Node(root_val)
    
    idx ← index of root_val in inorder
    
    root.left  ← BUILD_TREE(preorder[1..idx], inorder[0..idx-1])
    root.right ← BUILD_TREE(preorder[idx+1..], inorder[idx+1..])
    
    return root
```

### Trace

```
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]

Step 1: root = 3
  Find 3 in inorder at index 1
  Left inorder:  [9]        Left preorder:  [9]
  Right inorder: [15,20,7]  Right preorder: [20,15,7]

Step 2 (Left): root = 9
  Find 9 in inorder [9] at index 0
  Left: [],  Right: []  → leaf node

Step 3 (Right): root = 20
  Find 20 in inorder [15,20,7] at index 1
  Left inorder:  [15]  Left preorder:  [15]
  Right inorder: [7]   Right preorder: [7]

Step 4: root = 15 (leaf)
Step 5: root = 7  (leaf)

Result:    3
          / \
         9   20
            /  \
           15   7
```

### Optimization with HashMap

```
Naive: finding root in inorder = O(n) each time
  → Total: O(n²)

With HashMap (inorder value → index):
  Preprocessing: O(n)
  Each lookup: O(1)
  → Total: O(n)
```

---

## Problem 3: Construct from Postorder and Inorder

```
KEY INSIGHT: Last element of postorder = ROOT

Postorder: [9, 15, 7, 20, 3]  → root = 3 (last)
Inorder:   [9, 3, 15, 20, 7]

Same approach, but take root from END of postorder.

BUILD_TREE_POST(postorder, inorder):
    if empty: return null
    root_val ← postorder[last]
    idx ← find root_val in inorder
    root.left  ← left portion
    root.right ← right portion
    return root
```

---

## Problem 4: Construct from Preorder Only (BST)

```
For a BST, preorder alone is sufficient!

Preorder: [8, 5, 1, 7, 10, 12]

BST property constrains structure:
  Root = 8
  Values < 8 go left:  [5, 1, 7]
  Values > 8 go right: [10, 12]

          8
         / \
        5   10
       / \    \
      1   7   12

Algorithm (using bounds):
BUILD_BST_PREORDER(preorder, min, max, idx):
    if idx ≥ n or preorder[idx] < min or preorder[idx] > max:
        return null
    
    val ← preorder[idx]
    idx ← idx + 1
    node ← new Node(val)
    node.left  ← BUILD_BST_PREORDER(preorder, min, val, idx)
    node.right ← BUILD_BST_PREORDER(preorder, val, max, idx)
    return node

Time: O(n)
```

---

## D&C Pattern for Tree Construction

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  All tree construction follows the same D&C pattern:   │
│                                                        │
│  1. IDENTIFY ROOT                                      │
│     - Sorted array: middle element                     │
│     - Preorder: first element                          │
│     - Postorder: last element                          │
│     - Level order: first element                       │
│                                                        │
│  2. DIVIDE into left and right portions                │
│     - Using size (sorted array)                        │
│     - Using inorder position                           │
│     - Using BST property (value bounds)                │
│                                                        │
│  3. RECURSE on each portion                            │
│     - Left subtree from left portion                   │
│     - Right subtree from right portion                 │
│                                                        │
│  4. CONNECT subtrees to root                           │
│     - root.left = left result                          │
│     - root.right = right result                        │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Problem | Root Source | Split Method | Time |
|---------|-----------|-------------|------|
| Sorted array → BST | Middle element | Index split | O(n) |
| Pre + Inorder → BT | First of preorder | Inorder index | O(n) |
| Post + Inorder → BT | Last of postorder | Inorder index | O(n) |
| Preorder → BST | First element | Value bounds | O(n) |
| Sorted list → BST | Middle node | Fast/slow pointer | O(n log n) |

---

## Quick Revision Questions

1. **Why is the middle element chosen as root for sorted array → BST?**
2. **How does preorder identify the root of each subtree?**
3. **Why do we need inorder traversal along with preorder?**
4. **How does a HashMap optimize the inorder lookup?**
5. **Why is preorder alone sufficient for BST construction?**

---

[Next: Balanced BST from Sorted Array →](02-balanced-bst-from-sorted-array.md)
