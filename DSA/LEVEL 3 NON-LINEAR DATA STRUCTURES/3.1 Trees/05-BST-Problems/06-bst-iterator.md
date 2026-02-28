# Unit 5 – Topic 6: BST Iterator

[← Previous: Floor and Ceiling](05-floor-and-ceiling.md) | [Back to TOC](../README.md) | [Next: Tree Construction →](../06-Tree-Construction/01-build-from-inorder-preorder.md)

---

## 5.6 BST Iterator

### 📖 Problem

Implement an iterator over a BST that returns the **next smallest** element each time `next()` is called. Also implement `hasNext()`.

### 💡 Key Insight

> This is essentially a **controlled inorder traversal**. Instead of traversing all at once, we maintain a **stack** and process nodes on-demand.

### Algorithm

```
CLASS BSTIterator:
    stack ← empty stack
    
    CONSTRUCTOR(root):
        pushAllLeft(root)
    
    FUNCTION pushAllLeft(node):
        WHILE node ≠ NULL:
            stack.push(node)
            node ← node.left
    
    FUNCTION next():
        node ← stack.pop()
        IF node.right ≠ NULL:
            pushAllLeft(node.right)
        RETURN node.data
    
    FUNCTION hasNext():
        RETURN stack is not empty
```

### 🔍 Trace

```
            7
           / \
          3   15
             / \
            9   20

    Constructor: pushAllLeft(7) → stack = [7, 3]
    
    next(): pop 3, 3.right=NULL → return 3       stack = [7]
    next(): pop 7, pushAllLeft(15) → push 15,9   stack = [15, 9]
            return 7
    next(): pop 9, 9.right=NULL → return 9       stack = [15]
    next(): pop 15, pushAllLeft(20) → push 20    stack = [20]
            return 15
    next(): pop 20, 20.right=NULL → return 20    stack = []
    hasNext(): stack empty → false
    
    Sequence: 3, 7, 9, 15, 20  ← Sorted order ✓
```

### ⏱️ Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `next()` | O(1) amortized (each node pushed/popped once) | O(h) — stack stores at most h nodes |
| `hasNext()` | O(1) | O(1) |

---

## 🧩 Problem-Solving Patterns

### Pattern 1: Exploit BST Ordering
Many BST problems reduce to controlled traversal:
- Kth smallest → inorder count
- Validate BST → inorder must be sorted
- Convert to sorted array → inorder traversal

### Pattern 2: Binary Search on Tree
- Floor/Ceiling → narrow down like binary search
- LCA in BST → move based on value comparisons
- Search → left or right based on comparison

### Pattern 3: Divide and Conquer
- Sorted array to BST → middle element is root, recurse on halves
- Think of BST as recursively defined: root splits elements into left < root < right

---

## 📝 Unit 5 Summary Table

| Problem | Approach | Time | Space |
|---------|----------|------|-------|
| Sorted Array → BST | Pick middle as root, recurse | O(n) | O(log n) |
| BST → Sorted Array | Inorder traversal | O(n) | O(n) |
| Kth Smallest | Inorder with counter | O(h + k) | O(h) |
| LCA in BST | Compare both values, find split | O(h) | O(1) |
| Floor | Track candidate, go right if smaller | O(h) | O(1) |
| Ceiling | Track candidate, go left if larger | O(h) | O(1) |
| BST Iterator | Stack-based controlled inorder | O(1) amortized | O(h) |

---

### ❓ Revision Questions

1. **How does the BST iterator achieve O(1) amortized time for next()?**
   <details><summary>Answer</summary>Each node is pushed onto the stack exactly once and popped exactly once across all calls to next(). For n nodes, there are n pushes and n pops total across n calls, giving O(2n/n) = O(1) amortized per call.</details>

2. **If a BST contains duplicate values, how would floor and ceiling behave?**
   <details><summary>Answer</summary>Floor(x) returns the largest value ≤ x (could be x itself if present). Ceiling(x) returns the smallest value ≥ x (could be x itself). With duplicates, both could return x if it exists. The standard BST approach handles this naturally since we use ≤ and ≥ comparisons.</details>

---

[← Previous: Floor and Ceiling](05-floor-and-ceiling.md) | [Back to TOC](../README.md) | [Next: Tree Construction →](../06-Tree-Construction/01-build-from-inorder-preorder.md)
