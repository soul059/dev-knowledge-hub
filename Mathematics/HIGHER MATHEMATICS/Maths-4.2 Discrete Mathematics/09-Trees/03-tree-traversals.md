# Chapter 9.3: Tree Traversals

[← Previous: Binary Trees](02-binary-trees.md) | [Back to README](../README.md) | [Next: Spanning Trees →](04-spanning-trees.md)

---

## 📋 Chapter Overview

**Tree traversal** is the process of visiting every node in a tree exactly once in a systematic order. The three classic traversals — **preorder**, **inorder**, and **postorder** — plus **level-order** (BFS) are essential tools for processing tree-structured data.

---

## 1. The Three Depth-First Traversals

For a binary tree with root $R$, left subtree $L$, and right subtree $R'$:

| Traversal | Order | Mnemonic |
|-----------|:-----:|----------|
| **Preorder** | Root → Left → Right | **N**LR |
| **Inorder** | Left → Root → Right | L**N**R |
| **Postorder** | Left → Right → Root | LR**N** |

(N = Node/Root)

---

## 2. Example Tree

```
          A
         / \
        B   C
       / \   \
      D   E   F
     /   / \
    G   H   I
```

### Preorder (NLR): Visit root first, then left, then right

$$A \to B \to D \to G \to E \to H \to I \to C \to F$$

```
  Trace:  A → B → D → G (leaf, back)
                        → (D done, back to B)
               → E → H (leaf, back)
                    → I (leaf, back)
                        → (E done, back to B, done, back to A)
          → C → F (leaf, done)
```

### Inorder (LNR): Visit left, then root, then right

$$G \to D \to B \to H \to E \to I \to A \to C \to F$$

### Postorder (LRN): Visit left, then right, then root

$$G \to D \to H \to I \to E \to B \to F \to C \to A$$

---

## 3. Level-Order (Breadth-First) Traversal

Visit nodes **level by level**, left to right.

$$A \to B \to C \to D \to E \to F \to G \to H \to I$$

```
  Level 0:    A
  Level 1:    B, C
  Level 2:    D, E, F
  Level 3:    G, H, I
  
  Uses a QUEUE (not recursion):
  ┌───────────┐
  │ Enqueue A │
  │ Dequeue A → visit, enqueue B, C │
  │ Dequeue B → visit, enqueue D, E │
  │ Dequeue C → visit, enqueue F    │
  │ Dequeue D → visit, enqueue G    │
  │ Dequeue E → visit, enqueue H, I │
  │ Dequeue F → visit               │
  │ Dequeue G → visit               │
  │ Dequeue H → visit               │
  │ Dequeue I → visit               │
  └───────────┘
```

---

## 4. Traversal Algorithms (Pseudocode)

### Preorder (recursive)

```
  PREORDER(node):
      if node is NULL: return
      VISIT(node)
      PREORDER(node.left)
      PREORDER(node.right)
```

### Inorder (recursive)

```
  INORDER(node):
      if node is NULL: return
      INORDER(node.left)
      VISIT(node)
      INORDER(node.right)
```

### Postorder (recursive)

```
  POSTORDER(node):
      if node is NULL: return
      POSTORDER(node.left)
      POSTORDER(node.right)
      VISIT(node)
```

### Level-Order (iterative)

```
  LEVELORDER(root):
      queue ← empty queue
      enqueue(queue, root)
      while queue is not empty:
          node ← dequeue(queue)
          VISIT(node)
          if node.left ≠ NULL: enqueue(queue, node.left)
          if node.right ≠ NULL: enqueue(queue, node.right)
```

---

## 5. Expression Tree Traversals

```
  Expression: (3 + 4) × (5 - 2)
  
           ×
          / \
         +   -
        / \ / \
       3  4 5  2
```

| Traversal | Output | Notation |
|-----------|--------|----------|
| Preorder | $\times \; + \; 3 \; 4 \; - \; 5 \; 2$ | **Prefix** (Polish) |
| Inorder | $3 + 4 \times 5 - 2$ | **Infix** (needs parentheses) |
| Postorder | $3 \; 4 \; + \; 5 \; 2 \; - \; \times$ | **Postfix** (Reverse Polish) |

**Fully parenthesized inorder:** $((3 + 4) \times (5 - 2))$

### Evaluating Postfix with a Stack

```
  Input: 3 4 + 5 2 - ×
  
  Read 3   → push 3           Stack: [3]
  Read 4   → push 4           Stack: [3, 4]
  Read +   → pop 4,3 → 3+4=7  Stack: [7]
  Read 5   → push 5           Stack: [7, 5]
  Read 2   → push 2           Stack: [7, 5, 2]
  Read -   → pop 2,5 → 5-2=3  Stack: [7, 3]
  Read ×   → pop 3,7 → 7×3=21 Stack: [21]
  
  Result: 21 ✓
```

---

## 6. Reconstructing Trees from Traversals

### Given Preorder + Inorder → Unique Tree

- Preorder first element = root
- Find root in inorder → splits into left and right subtree
- Recurse

### Example

Preorder: $A, B, D, E, C, F$  
Inorder: $D, B, E, A, C, F$

1. Root = $A$ (first in preorder)
2. In inorder: left of $A$ = $\{D, B, E\}$, right = $\{C, F\}$
3. Left subtree preorder: $B, D, E$; root = $B$
4. In inorder: left of $B$ = $\{D\}$, right = $\{E\}$
5. Right subtree preorder: $C, F$; root = $C$, right of $C$ = $\{F\}$

```
  Result:
        A
       / \
      B   C
     / \   \
    D   E   F
```

### What Pairs Determine a Unique Tree?

| Given | Unique Tree? |
|-------|:------------:|
| Preorder + Inorder | **Yes** |
| Postorder + Inorder | **Yes** |
| Preorder + Postorder | **No** (ambiguous for nodes with one child) |

---

## 7. General Tree Traversals

For non-binary (general) trees with any number of children:

```
  General tree:
  
         A
       / | \
      B  C  D
     /|     |
    E  F    G
```

| Traversal | Order |
|-----------|-------|
| Preorder | $A, B, E, F, C, D, G$ |
| Postorder | $E, F, B, C, G, D, A$ |
| Level-order | $A, B, C, D, E, F, G$ |

(Inorder is not naturally defined for general trees.)

---

## 8. Iterative Traversals Using a Stack

### Iterative Preorder

```
  PREORDER_ITERATIVE(root):
      stack ← empty stack
      push(stack, root)
      while stack is not empty:
          node ← pop(stack)
          VISIT(node)
          if node.right ≠ NULL: push(stack, node.right)
          if node.left ≠ NULL: push(stack, node.left)
```

(Push right first so left is processed first — LIFO order.)

---

## 9. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │        Tree Traversal Applications               │
  │                                                  │
  │  Preorder:                                       │
  │  • Copy a tree, serialize a tree                 │
  │  • Prefix expression evaluation                  │
  │  • Directory listing (folder before contents)    │
  │                                                  │
  │  Inorder:                                        │
  │  • BST → sorted output                          │
  │  • Expression tree → infix notation              │
  │                                                  │
  │  Postorder:                                      │
  │  • Delete a tree (children before parent)        │
  │  • Calculate directory sizes                     │
  │  • Postfix expression evaluation                 │
  │                                                  │
  │  Level-order:                                    │
  │  • Print tree level by level                     │
  │  • Shortest path in unweighted tree              │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Traversal | Order | Data Structure | Use Case |
|-----------|:-----:|:--------------:|----------|
| Preorder | Root, Left, Right | Stack/Recursion | Copying, prefix notation |
| Inorder | Left, Root, Right | Stack/Recursion | Sorted output (BST) |
| Postorder | Left, Right, Root | Stack/Recursion | Deletion, postfix eval |
| Level-order | Level by level | **Queue** | BFS, level printing |

---

## ❓ Quick Revision Questions

1. **Given the tree in Section 2, write the preorder, inorder, and postorder traversals.**

2. **Evaluate the postfix expression: 6 2 / 3 + 4 ×**

3. **Reconstruct the binary tree from: Preorder = {F, B, A, D, C, E, G, I, H} and Inorder = {A, B, C, D, E, F, G, H, I}.**

4. **Why can't preorder + postorder alone uniquely determine a binary tree?**

5. **Write the level-order traversal for a BST containing {1, 2, 3, 4, 5, 6, 7} inserted in order 4, 2, 6, 1, 3, 5, 7.**

6. **An inorder traversal of a BST produces a sorted sequence. Why?**

---

[← Previous: Binary Trees](02-binary-trees.md) | [Back to README](../README.md) | [Next: Spanning Trees →](04-spanning-trees.md)
