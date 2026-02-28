# Chapter 2: Balanced BST from Sorted Array

## Overview

Converting a sorted array (or sorted linked list) into a height-balanced BST is a classic D&C problem that appears frequently in interviews. The key insight: always pick the **middle element** as the root to ensure balance.

---

## Problem Statement

```
INPUT:  Sorted array nums = [-10, -3, 0, 5, 9]

OUTPUT: A height-balanced BST (any valid answer)

Height-balanced: For every node, the heights of its
left and right subtrees differ by at most 1.

Valid answers:
         0                    0
        / \                  / \
      -3    9             -10    5
      /    /                \    \
   -10   5                  -3    9
```

---

## The D&C Approach

```
┌────────────────────────────────────────────────────┐
│                                                    │
│  DIVIDE:   Pick middle element as root             │
│            Left half → future left subtree         │
│            Right half → future right subtree       │
│                                                    │
│  CONQUER:  Recursively build left subtree          │
│            Recursively build right subtree         │
│                                                    │
│  COMBINE:  Attach left and right to root           │
│                                                    │
│  WHY MIDDLE? → Equal elements on each side        │
│              → Heights differ by at most 1         │
│              → Guaranteed height-balanced!          │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace

```
nums = [-10, -3, 0, 5, 9]

Call: build(0, 4)
  mid = 2, root = 0
  │
  ├── build(0, 1): left subtree
  │   mid = 0, root = -10
  │   │
  │   ├── build(0, -1): return null
  │   └── build(1, 1): 
  │       mid = 1, root = -3
  │       ├── build(1, 0): return null
  │       └── build(2, 1): return null
  │       return Node(-3)
  │   return Node(-10, null, Node(-3))
  │
  └── build(3, 4): right subtree
      mid = 3, root = 5
      │
      ├── build(3, 2): return null
      └── build(4, 4):
          mid = 4, root = 9
          ├── build(4, 3): return null
          └── build(5, 4): return null
          return Node(9)
      return Node(5, null, Node(9))

Final tree:
           0
          / \
       -10    5
          \    \
          -3    9

Height = 3, balanced ✓
```

---

## Implementation

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def sortedArrayToBST(nums):
    """Convert sorted array to height-balanced BST."""
    
    def build(lo, hi):
        if lo > hi:
            return None
        
        mid = (lo + hi) // 2
        node = TreeNode(nums[mid])
        node.left = build(lo, mid - 1)
        node.right = build(mid + 1, hi)
        return node
    
    return build(0, len(nums) - 1)
```

---

## Why It Produces a Balanced Tree

```
PROOF: The tree produced has height ⌈log₂(n+1)⌉.

For array of size n:
  Root gets ~n/2 elements on each side.
  Height = 1 + max(height(left), height(right))
         = 1 + max(h(⌊n/2⌋), h(⌈n/2⌉ - 1))
         = 1 + h(⌊n/2⌋)     (within 1 of each other)

  h(n) = 1 + h(n/2) → h(n) = ⌈log₂(n+1)⌉

Balance factor at every node:
  |left_size - right_size| ≤ 1  (depending on parity)
  → Height difference ≤ 1 → balanced!

Even vs Odd n:
  n=7:  [1,2,3, 4 ,5,6,7]  → 3 left, 3 right, balanced
  n=6:  [1,2,3, 4 ,5,6]    → 3 left, 2 right, ok (diff=1)
  n=5:  [1,2, 3 ,4,5]      → 2 left, 2 right, balanced
```

---

## Sorted Linked List → BST

```
Problem: Same task, but input is a sorted LINKED LIST.

Challenge: Cannot access middle in O(1)!

Approach 1: Fast/Slow Pointer (Top-Down D&C)
  1. Find middle using slow/fast pointers: O(n)
  2. Split list at middle
  3. Recurse on each half
  Time: O(n log n) — finding middle at each level

Approach 2: Inorder Simulation (Bottom-Up D&C)
  1. Count nodes: O(n)
  2. Build tree in inorder, consuming list nodes
  Time: O(n) — each node visited exactly once
```

### Bottom-Up Approach (O(n))

```python
def sortedListToBST(head):
    # Count nodes
    n, curr = 0, head
    while curr:
        n += 1
        curr = curr.next
    
    # Use a mutable reference to track current list position
    current = [head]
    
    def build(lo, hi):
        if lo > hi:
            return None
        
        mid = (lo + hi) // 2
        
        # Build left subtree first (inorder!)
        left = build(lo, mid - 1)
        
        # Current node becomes root
        node = TreeNode(current[0].val)
        node.left = left
        current[0] = current[0].next
        
        # Build right subtree
        node.right = build(mid + 1, hi)
        
        return node
    
    return build(0, n - 1)
```

### How Bottom-Up Works

```
List: 1 → 2 → 3 → 4 → 5
Build(0, 4), mid=2

  Build(0, 1), mid=0         ← go left first
    Build(0, -1) → null
    consume 1 → Node(1)      ← 1st node consumed
    Build(1, 1), mid=1
      Build(1, 0) → null
      consume 2 → Node(2)    ← 2nd node consumed
      Build(2, 1) → null
    Node(1).right = Node(2)
    return Node(1)
  
  consume 3 → root = Node(3)   ← 3rd node consumed
  root.left = Node(1, null, Node(2))
  
  Build(3, 4), mid=3          ← go right
    Build(3, 2) → null
    consume 4 → Node(4)       ← 4th node consumed
    Build(4, 4), mid=4
      Build(4, 3) → null
      consume 5 → Node(5)     ← 5th node consumed
      Build(5, 4) → null
    Node(4).right = Node(5)
    return Node(4)
  
  root.right = Node(4, null, Node(5))
  
Result:     3
           / \
          1    4
           \    \
            2    5
```

---

## Complexity Comparison

```
┌─────────────────────────┬──────────┬──────────┐
│ Approach                │ Time     │ Space    │
├─────────────────────────┼──────────┼──────────┤
│ Array (top-down)        │ O(n)     │ O(log n) │
│ List (fast/slow)        │ O(n log n)│ O(log n) │
│ List (bottom-up)        │ O(n)     │ O(log n) │
└─────────────────────────┴──────────┴──────────┘

Space is O(log n) for recursion stack in all cases.
The tree itself uses O(n) additional space.
```

---

## Variations

### Multiple Valid Trees

```
When n is even, there are two valid choices for root:

  nums = [1, 2, 3, 4]

  Choice 1: mid = 1 → root = 2    Choice 2: mid = 2 → root = 3
       2                                3
      / \                              / \
     1   3                            2   4
          \                          /
           4                        1

Both are height-balanced. Either is accepted.
```

### Build AVL Tree (Self-Balancing)

```
If input is NOT sorted:
  1. Sort it first: O(n log n)
  2. Build balanced BST: O(n)
  Total: O(n log n)

Alternatively: Insert one by one into AVL tree
  → O(n log n) but with rotations
  → Sorted array approach is simpler!
```

---

## LeetCode Connections

```
┌──────┬──────────────────────────────────────┬──────────┐
│ LC # │ Problem                              │ Approach │
├──────┼──────────────────────────────────────┼──────────┤
│ 108  │ Convert Sorted Array to BST          │ Top-down │
│ 109  │ Convert Sorted List to BST           │ Bottom-up│
│ 1382 │ Balance a BST                        │ Flatten  │
│      │                                      │ + rebuild│
│ 105  │ Construct BT from Preorder & Inorder│ D&C      │
│ 106  │ Construct BT from Inorder & Postorder│ D&C      │
└──────┴──────────────────────────────────────┴──────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Core idea | Middle element = root → balance guaranteed |
| Array input | Direct index access → O(n) |
| List input (naive) | Find middle each time → O(n log n) |
| List input (optimal) | Inorder simulation → O(n) |
| Height guarantee | ⌈log₂(n+1)⌉ |
| Space | O(log n) stack + O(n) tree |

---

## Quick Revision Questions

1. **Why does picking the middle element guarantee balance?**
2. **What is the height of the resulting BST for n elements?**
3. **How does the bottom-up approach for linked lists work?**
4. **Why is the bottom-up approach O(n) instead of O(n log n)?**
5. **How many valid height-balanced BSTs exist for n=4?**

---

[← Previous: Tree Construction](01-tree-construction.md) | [Next: Merge K Sorted Lists →](03-merge-k-sorted-lists.md)
