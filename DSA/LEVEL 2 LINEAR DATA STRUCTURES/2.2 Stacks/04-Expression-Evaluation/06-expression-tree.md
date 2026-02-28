# Chapter 6: Expression Tree

[← Previous: Prefix Evaluation](05-prefix-evaluation.md) | [Next: What is Monotonic Stack? →](../05-Monotonic-Stack/01-monotonic-stack-intro.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)

---

## Overview

An **expression tree** is a binary tree that represents a mathematical expression. Internal nodes are **operators** and leaf nodes are **operands**. The beauty of expression trees is that different tree traversals produce different notations (infix, prefix, postfix), unifying all three concepts.

---

## Structure

```
Expression: (A + B) * (C - D)

              *
             / \
            +   -
           / \ / \
          A  B C  D

Rules:
  - Leaf nodes     = Operands (A, B, C, D)
  - Internal nodes = Operators (+, -, *, /)
  - Each operator has exactly 2 children
  - Deeper nodes = higher precedence
```

---

## Traversals and Notation

```
              *
             / \
            +   -
           / \ / \
          A  B C  D

┌────────────────────┬──────────────┬───────────────────┐
│ Traversal          │ Order        │ Result            │
├────────────────────┼──────────────┼───────────────────┤
│ Inorder (L,N,R)    │ A + B * C - D│ (A+B) * (C-D)   │
│ Preorder (N,L,R)   │ * + A B - C D│ Prefix notation  │
│ Postorder (L,R,N)  │ A B + C D - *│ Postfix notation │
└────────────────────┴──────────────┴───────────────────┘

Inorder traversal (with parentheses):
  Visit left → print node → visit right
  
Preorder traversal:
  Print node → visit left → visit right
  
Postorder traversal:
  Visit left → visit right → print node
```

---

## Building Expression Tree from Postfix

### Algorithm

```
┌──────────────────────────────────────────────────────┐
│        BUILD TREE FROM POSTFIX                       │
│                                                      │
│  Scan LEFT to RIGHT:                                 │
│                                                      │
│  1. OPERAND → Create leaf node, push to stack        │
│                                                      │
│  2. OPERATOR → Pop 2 nodes from stack                │
│     Create new node with operator                    │
│     Right child = first pop                          │
│     Left child = second pop                          │
│     Push new node to stack                           │
│                                                      │
│  3. END → Stack top = root of expression tree        │
└──────────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION buildTreeFromPostfix(postfix):
    stack ← empty stack
    
    FOR each token in postfix:
        IF token is OPERAND:
            node ← createNode(token)
            stack.push(node)
        ELSE:
            node ← createNode(token)
            node.right ← stack.pop()    // First pop = right
            node.left ← stack.pop()     // Second pop = left
            stack.push(node)
    
    RETURN stack.pop()    // Root of tree
```

### Trace: Build tree from `A B + C D - *`

```
Step 1: Token 'A' → Create leaf A, push
  Stack: [A]

Step 2: Token 'B' → Create leaf B, push
  Stack: [A, B]

Step 3: Token '+' → Pop B (right), Pop A (left)
  Create + node:    +
                   / \
                  A   B
  Push + node
  Stack: [+]

Step 4: Token 'C' → Create leaf C, push
  Stack: [+, C]

Step 5: Token 'D' → Create leaf D, push
  Stack: [+, C, D]

Step 6: Token '-' → Pop D (right), Pop C (left)
  Create - node:    -
                   / \
                  C   D
  Push - node
  Stack: [+, -]

Step 7: Token '*' → Pop - (right), Pop + (left)
  Create * node:      *
                     / \
                    +   -
                   / \ / \
                  A  B C  D
  Push * node
  Stack: [*]

Result: Root = *

Final Tree:
          *
         / \
        +   -
       / \ / \
      A  B C  D
```

---

## Building Expression Tree from Prefix

### Algorithm

```
Scan RIGHT to LEFT (same as prefix evaluation):

  1. OPERAND → Create leaf node, push to stack
  2. OPERATOR → Pop 2 nodes
     Left child  = first pop
     Right child = second pop
     Push new node

  3. END → Stack top = root
```

### Trace: Build from `* + A B - C D`

```
Scan R→L: D, C, -, B, A, +, *

Step 1: D → leaf, push       Stack: [D]
Step 2: C → leaf, push       Stack: [D, C]
Step 3: - → pop C(L), D(R)   Stack: [-]
         -
        / \
       C   D

Step 4: B → leaf, push       Stack: [-, B]
Step 5: A → leaf, push       Stack: [-, B, A]
Step 6: + → pop A(L), B(R)   Stack: [-, +]
         +
        / \
       A   B

Step 7: * → pop +(L), -(R)   Stack: [*]
         *
        / \
       +   -
      / \ / \
     A  B C  D

Same tree! ✓
```

---

## Evaluating an Expression Tree

```
FUNCTION evaluate(node):
    IF node is a LEAF (operand):
        RETURN node.value
    
    leftVal ← evaluate(node.left)
    rightVal ← evaluate(node.right)
    
    RETURN applyOperator(leftVal, node.operator, rightVal)
```

### Trace: Evaluate with A=3, B=4, C=7, D=2

```
          *
         / \
        +   -
       / \ / \
      3  4 7  2

evaluate(*)
├── evaluate(+)
│   ├── evaluate(3) → 3
│   └── evaluate(4) → 4
│   └── 3 + 4 = 7
├── evaluate(-)
│   ├── evaluate(7) → 7
│   └── evaluate(2) → 2
│   └── 7 - 2 = 5
└── 7 * 5 = 35

Result: 35
```

---

## Generating Expressions from Tree

### Infix with Parentheses

```
FUNCTION inorder(node):
    IF node is LEAF:
        RETURN node.value
    
    result ← "("
    result ← result + inorder(node.left)
    result ← result + node.operator
    result ← result + inorder(node.right)
    result ← result + ")"
    RETURN result

Output: ((A+B)*(C-D))
```

### Postfix

```
FUNCTION postorder(node):
    IF node is LEAF:
        RETURN node.value
    
    result ← postorder(node.left)
    result ← result + " " + postorder(node.right)
    result ← result + " " + node.operator
    RETURN result

Output: A B + C D - *
```

### Prefix

```
FUNCTION preorder(node):
    IF node is LEAF:
        RETURN node.value
    
    result ← node.operator
    result ← result + " " + preorder(node.left)
    result ← result + " " + preorder(node.right)
    RETURN result

Output: * + A B - C D
```

---

## More Examples

```
Expression: A + B * C

Tree (respecting precedence):
       +
      / \
     A   *
        / \
       B   C

Inorder:   (A+(B*C))
Preorder:  + A * B C  
Postorder: A B C * +

─────────────────────────────────

Expression: (A - B) * (C / (D + E))

Tree:
            *
           / \
          -   /
         / \ / \
        A  B C  +
               / \
              D   E

Inorder:   ((A-B)*(C/(D+E)))
Preorder:  * - A B / C + D E
Postorder: A B - C D E + / *
```

---

## Complexity Analysis

| Operation | Time | Space | Explanation |
|-----------|------|-------|-------------|
| **Build from postfix** | O(n) | O(n) | Single pass, stack holds nodes |
| **Build from prefix** | O(n) | O(n) | Single pass (R→L) |
| **Evaluate** | O(n) | O(h) | Visit each node; recursion depth = height |
| **Generate notation** | O(n) | O(h) | Visit each node |

Where `h` = height of tree, `n` = number of nodes.

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Structure** | Binary tree: operators=internal, operands=leaves |
| **Build from Postfix** | Scan L→R, stack of tree nodes |
| **Build from Prefix** | Scan R→L, stack of tree nodes |
| **Inorder traversal** | Produces infix notation |
| **Preorder traversal** | Produces prefix notation |
| **Postorder traversal** | Produces postfix notation |
| **Evaluation** | Recursive postorder traversal with computation |

---

## Quick Revision Questions

1. **What do leaf nodes and internal nodes represent in an expression tree?**
   > Leaf nodes = operands (numbers/variables), internal nodes = operators.

2. **Which traversal gives postfix notation?**
   > Postorder traversal (left, right, node).

3. **How do you build an expression tree from postfix?**
   > Scan left to right. Push operand leaves. For operators, pop two nodes, make them children, push the new subtree.

4. **Why is the inorder traversal of an expression tree not sufficient without parentheses?**
   > Without parentheses, inorder traversal loses precedence information. `A + B * C` and `(A + B) * C` would look the same.

5. **What is the time complexity of evaluating an expression tree?**
   > O(n) — each node is visited exactly once.

---

[← Previous: Prefix Evaluation](05-prefix-evaluation.md) | [Next: What is Monotonic Stack? →](../05-Monotonic-Stack/01-monotonic-stack-intro.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)
