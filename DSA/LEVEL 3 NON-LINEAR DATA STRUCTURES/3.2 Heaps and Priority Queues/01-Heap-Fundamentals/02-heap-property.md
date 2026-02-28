# 1.2 Heap Property

[← Previous: What is a Heap?](01-what-is-a-heap.md) | [Next: Complete Binary Tree →](03-complete-binary-tree.md)

---

## Overview

The **heap property** is the ordering constraint that makes a heap useful. It guarantees that the most important element (min or max) is always at the root.

---

## Max-Heap Property

> Every parent node is **greater than or equal to** its children.

```
         90            ← Root is the MAXIMUM
        /  \
      80    70
     / \   / \
    50 60 65  30
```

**Formal**: For every node `i` (except root): `A[parent(i)] >= A[i]`

---

## Min-Heap Property

> Every parent node is **less than or equal to** its children.

```
         10            ← Root is the MINIMUM
        /  \
      20    15
     / \   / \
    30 25 50  40
```

**Formal**: For every node `i` (except root): `A[parent(i)] <= A[i]`

---

## Important Observations

| Observation | Detail |
|------------|--------|
| Root element | Always the min (min-heap) or max (max-heap) |
| Siblings | **No ordering** between siblings — left child can be larger than right |
| Levels | Lower levels are NOT necessarily sorted |
| Duplicates | Allowed in heaps |

---

## What a Heap is NOT

```
    NOT a valid Max-Heap:          Valid Max-Heap:
    
         90                            90
        /  \                          /  \
      80    85                      85    80
     / \                           / \
    95  60  ← 95 > 80, INVALID!  60  70
```

The heap property must hold for **every** parent-child pair, not just some.

---

## Quick Check

1. **In a max-heap, can the left child be larger than the right child of the same parent?**
   <details><summary>Answer</summary>
   Yes. The heap property only requires that both children are ≤ the parent. There is no ordering between siblings.
   </details>

2. **Is this array a valid max-heap? [100, 50, 80, 30, 60, 70, 20]**
   <details><summary>Answer</summary>
   Check: 100≥50✅, 100≥80✅, 50≥30✅, 50≥60? ❌ (50 < 60). NO, it is not a valid max-heap.
   </details>

---

[← Previous: What is a Heap?](01-what-is-a-heap.md) | [Next: Complete Binary Tree →](03-complete-binary-tree.md)
