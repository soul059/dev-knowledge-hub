# Unit 10: Advanced Tree Concepts

[← Previous: Tree Views](../09-Tree-Views/01-tree-views.md) | [Back to TOC](../README.md)

---

## Chapter Overview

This final unit covers **advanced tree concepts** that go beyond basic binary tree operations. These include threaded binary trees for efficient traversal, expression trees for evaluating expressions, Huffman trees for data compression, and conversion problems like tree-to-DLL and tree flattening.

---

## 10.1 Threaded Binary Tree

### 📖 Concept

In a standard binary tree with `n` nodes, there are `n + 1` NULL pointers (wasted). A **threaded binary tree** replaces these NULL pointers with **threads** — pointers to the inorder predecessor or successor.

```
    Standard Binary Tree:                Threaded Binary Tree:
    
         4                                    4
        / \                                  / \
       2   6                                2   6
      / \ / \                              / \ / \
     1  3 5  7                            1  3 5  7
     ↓↓ ↓↓ ↓↓ ↓↓                          |  ↗ ↗  ↗
     All NULL pointers                    1→2 3→4 5→6 7→(end)
     (8 NULLs for 7 nodes)               Threads to inorder successor
```

### Types of Threaded Trees

| Type | Description |
|------|-------------|
| Single Threaded | Only right NULL pointers are replaced (→ inorder successor) |
| Double Threaded | Both NULL pointers replaced (left → predecessor, right → successor) |

### Node Structure

```
CLASS ThreadedNode:
    data
    left
    right
    leftThread   ← boolean (true if left points to predecessor)
    rightThread  ← boolean (true if right points to successor)
```

### Single-Threaded Tree (Right Threads)

```
    Inorder: 1, 2, 3, 4, 5, 6, 7
    
              4
             / \
            2   6
           / \ / \
          1  3 5  7
          
    Right threads (dotted arrows):
    
              4
             / \
            2   6
           / \ / \
          1  3 5  7
           ╰→2 ╰→4 ╰→6  ╰→NULL
          
    Node 1: right thread → 2 (inorder successor)
    Node 3: right thread → 4
    Node 5: right thread → 6
    Node 7: right thread → NULL (no successor)
```

### Threaded Inorder Traversal (No Stack/Recursion!)

```
FUNCTION threadedInorder(root):
    current ← leftmost(root)
    
    WHILE current ≠ NULL:
        PRINT current.data
        
        IF current.rightThread:
            current ← current.right     // Follow thread
        ELSE:
            current ← leftmost(current.right)  // Go to leftmost of right subtree

FUNCTION leftmost(node):
    WHILE node ≠ NULL AND NOT node.leftThread:
        node ← node.left
    RETURN node
```

### 🔍 Trace: Threaded Inorder

```
    Tree with right threads:
    1→(thread to 2), 3→(thread to 4), 5→(thread to 6)
    
    Step 1: current = leftmost(root) = 1
    Step 2: PRINT 1, rightThread=true → current = 2
    Step 3: PRINT 2, rightThread=false → current = leftmost(3) = 3
    Step 4: PRINT 3, rightThread=true → current = 4
    Step 5: PRINT 4, rightThread=false → current = leftmost(6) = 5
    Step 6: PRINT 5, rightThread=true → current = 6
    Step 7: PRINT 6, rightThread=false → current = leftmost(7) = 7
    Step 8: PRINT 7, rightThread=true → current = NULL → DONE
    
    Output: 1, 2, 3, 4, 5, 6, 7 ✓
```

### Advantages of Threaded Trees

| Feature | Standard BT | Threaded BT |
|---------|-------------|-------------|
| Inorder traversal | O(n) time, O(h) space | O(n) time, **O(1) space** |
| Find successor | O(h) | **O(1)** for threaded nodes |
| NULL pointer waste | n+1 NULLs wasted | 0 NULLs wasted |
| Insertion complexity | Simple | More complex (maintain threads) |

---

## 10.2 Expression Tree

### 📖 Concept

An **expression tree** is a binary tree used to represent arithmetic expressions.

- **Leaves**: Operands (numbers, variables)
- **Internal nodes**: Operators (+, -, ×, /)
- **Structure**: Encodes operator precedence naturally

```
    Expression: (3 + 5) × (4 - 2)
    
              ×
             / \
            +   -
           / \ / \
          3  5 4  2
    
    Inorder (infix):     3 + 5 × 4 - 2  (needs parentheses!)
    Preorder (prefix):   × + 3 5 - 4 2
    Postorder (postfix): 3 5 + 4 2 - ×
```

### Build Expression Tree from Postfix

```
FUNCTION buildExpressionTree(postfix):
    stack ← empty stack
    
    FOR each token in postfix:
        IF token is operand:
            stack.push(new TreeNode(token))
        ELSE:                                  // token is operator
            node ← new TreeNode(token)
            node.right ← stack.pop()           // Right first!
            node.left  ← stack.pop()
            stack.push(node)
    
    RETURN stack.pop()    // Root of expression tree
```

### 🔍 Trace: Build from Postfix "3 5 + 4 2 - ×"

```
    Token | Action           | Stack (top→)
    ------+------------------+----------------------
    3     | push leaf(3)     | [3]
    5     | push leaf(5)     | [3, 5]
    +     | pop 5,3; build + | [+]
          |      +           |     / \
          |     3 5          |    3   5
    4     | push leaf(4)     | [+, 4]
    2     | push leaf(2)     | [+, 4, 2]
    -     | pop 2,4; build - | [+, -]
          |      -           |     / \
          |     4 2          |    4   2
    ×     | pop -,+; build × | [×]
          |      ×           |     / \
          |     + -          |    +   -
          |    /\ /\         |   /\ / \
          |   3 5 4 2        |  3 5 4  2
    
    Result:    ×
              / \
             +   -
            / \ / \
           3  5 4  2
```

### Evaluate Expression Tree

```
FUNCTION evaluate(node):
    // Leaf = operand
    IF node.left == NULL AND node.right == NULL:
        RETURN node.data
    
    leftVal  ← evaluate(node.left)
    rightVal ← evaluate(node.right)
    
    SWITCH node.data:
        CASE '+': RETURN leftVal + rightVal
        CASE '-': RETURN leftVal - rightVal
        CASE '×': RETURN leftVal × rightVal
        CASE '/': RETURN leftVal / rightVal
```

### 🔍 Trace: Evaluate

```
              ×
             / \
            +   -
           / \ / \
          3  5 4  2
    
    evaluate(3) = 3
    evaluate(5) = 5
    evaluate(+) = 3 + 5 = 8
    
    evaluate(4) = 4
    evaluate(2) = 2
    evaluate(-) = 4 - 2 = 2
    
    evaluate(×) = 8 × 2 = 16  ✓
```

---

## 10.3 Huffman Tree (Introduction)

### 📖 Concept

A **Huffman tree** is a binary tree used for **lossless data compression**. Characters with **higher frequency** get **shorter codes**, and characters with **lower frequency** get **longer codes**.

### Algorithm: Build Huffman Tree

```
FUNCTION buildHuffmanTree(characters, frequencies):
    // Step 1: Create a min-heap (priority queue) of leaf nodes
    minHeap ← empty priority queue
    FOR each (char, freq) pair:
        minHeap.insert(new LeafNode(char, freq))
    
    // Step 2: Build tree bottom-up
    WHILE minHeap.size() > 1:
        left  ← minHeap.extractMin()
        right ← minHeap.extractMin()
        
        // Create internal node with combined frequency
        internal ← new InternalNode(freq = left.freq + right.freq)
        internal.left  ← left
        internal.right ← right
        
        minHeap.insert(internal)
    
    RETURN minHeap.extractMin()    // Root of Huffman tree
```

### 🔍 Trace: Building a Huffman Tree

```
    Characters:  a=5, b=9, c=12, d=13, e=16, f=45
    
    Step 1: Min-heap: [a:5, b:9, c:12, d:13, e:16, f:45]
    
    Step 2: Extract a(5) + b(9) = 14
            [c:12, d:13, 14, e:16, f:45]
            
                14
               / \
              a:5 b:9
    
    Step 3: Extract c(12) + d(13) = 25
            [14, e:16, 25, f:45]
            
                25
               / \
             c:12 d:13
    
    Step 4: Extract 14 + e(16) = 30
            [25, 30, f:45]
            
                30
               / \
              14  e:16
             / \
            a:5 b:9
    
    Step 5: Extract 25 + 30 = 55
            [f:45, 55]
            
                 55
                / \
              25   30
             / \  / \
           c:12 d:13 14 e:16
                    / \
                   a:5 b:9
    
    Step 6: Extract f(45) + 55 = 100
            
                    100
                   /    \
                f:45     55
                        / \
                      25   30
                     / \  / \
                   c:12 d:13 14 e:16
                            / \
                           a:5 b:9
```

### Generate Codes

```
    Traverse: left = 0, right = 1
    
    Character | Code    | Frequency | Bits Used
    ----------+---------+-----------+-----------
    f         | 0       | 45        | 45 × 1 = 45
    c         | 100     | 12        | 12 × 3 = 36
    d         | 101     | 13        | 13 × 3 = 39
    a         | 1100    | 5         | 5 × 4 = 20
    b         | 1101    | 9         | 9 × 4 = 36
    e         | 111     | 16        | 16 × 3 = 48
    
    Total bits: 224
    
    Without Huffman (3 bits each for 6 chars):
    Total: (5+9+12+13+16+45) × 3 = 300 bits
    
    Savings: 300 - 224 = 76 bits (25% smaller!)
```

### Key Properties

- **Prefix-free code**: No code is a prefix of another → unambiguous decoding
- **Optimal**: Huffman coding produces the most efficient prefix-free code
- Frequently used characters → shorter codes (closer to root)
- **Time complexity**: O(n log n) to build

---

## 10.4 Binary Tree to Doubly Linked List (DLL)

### 📖 Problem

Convert a binary tree to a **doubly linked list** where the DLL follows **inorder** order. The `left` pointer acts as `prev` and the `right` pointer acts as `next`.

```
    Tree:           DLL (inorder sorted):
            4
           / \      1 ⟷ 2 ⟷ 3 ⟷ 4 ⟷ 5 ⟷ 6 ⟷ 7
          2   6
         / \ / \
        1  3 5  7
```

### Algorithm

```
FUNCTION treeToDLL(root):
    prev ← NULL
    head ← NULL
    
    FUNCTION inorder(node):
        IF node == NULL: RETURN
        
        inorder(node.left)
        
        // Process current node
        IF prev == NULL:
            head ← node            // First node = head of DLL
        ELSE:
            prev.right ← node      // prev.next = current
            node.left ← prev       // current.prev = prev
        
        prev ← node
        
        inorder(node.right)
    
    inorder(root)
    RETURN head
```

### 🔍 Trace

```
            4
           / \
          2   6
         / \
        1   3

    Inorder visits: 1, 2, 3, 4, 6
    
    Visit 1: prev=NULL → head=1, prev=1
    Visit 2: prev=1 → 1.right=2, 2.left=1, prev=2
    Visit 3: prev=2 → 2.right=3, 3.left=2, prev=3
    Visit 4: prev=3 → 3.right=4, 4.left=3, prev=4
    Visit 6: prev=4 → 4.right=6, 6.left=4, prev=6
    
    DLL: 1 ⟷ 2 ⟷ 3 ⟷ 4 ⟷ 6
         ↑ head
    
    Node pointers:
    1: left=NULL, right=2
    2: left=1, right=3
    3: left=2, right=4
    4: left=3, right=6
    6: left=4, right=NULL
```

### ⏱️ Complexity: O(n) time, O(h) space (recursion stack)

---

## 10.5 Flatten Binary Tree to Linked List

### 📖 Problem

Flatten a binary tree into a "linked list" **in-place** using preorder order. The linked list uses the **right** pointer, and the **left** pointer is always NULL.

```
    Tree:              Flattened:
            1
           / \          1
          2   5          \
         / \   \          2
        3   4   6          \
                            3
                             \
                              4
                               \
                                5
                                 \
                                  6
    
    Preorder: 1, 2, 3, 4, 5, 6
```

### Approach 1: Reverse Postorder (Right → Left → Root)

```
FUNCTION flatten(root):
    prev ← NULL
    
    FUNCTION dfs(node):
        IF node == NULL: RETURN
        
        dfs(node.right)      // Process right first
        dfs(node.left)       // Then left
        
        // Link current node to previously processed node
        node.right ← prev
        node.left ← NULL
        prev ← node
    
    dfs(root)
```

### 🔍 Trace: Reverse Postorder Flatten

```
            1
           / \
          2   5
         / \   \
        3   4   6

    dfs(6): prev=NULL → 6.right=NULL, 6.left=NULL, prev=6
    dfs(5): prev=6 → 5.right=6, 5.left=NULL, prev=5
    dfs(4): prev=5 → 4.right=5, 4.left=NULL, prev=4
    dfs(3): prev=4 → 3.right=4, 3.left=NULL, prev=3
    dfs(2): prev=3 → 2.right=3, 2.left=NULL, prev=2
    dfs(1): prev=2 → 1.right=2, 1.left=NULL, prev=1
    
    Result: 1 → 2 → 3 → 4 → 5 → 6 ✓ (preorder)
```

### Approach 2: Morris-like (O(1) Space, Iterative)

```
FUNCTION flattenIterative(root):
    current ← root
    
    WHILE current ≠ NULL:
        IF current.left ≠ NULL:
            // Find rightmost node in left subtree
            rightmost ← current.left
            WHILE rightmost.right ≠ NULL:
                rightmost ← rightmost.right
            
            // Connect rightmost to current's right subtree
            rightmost.right ← current.right
            
            // Move left subtree to right
            current.right ← current.left
            current.left ← NULL
        
        current ← current.right
```

### 🔍 Trace: Iterative Flatten

```
    Step 1: current=1, left exists
            rightmost of left subtree(2→4) = 4
            4.right = 5 (current's right)
            1.right = 2, 1.left = NULL
            
            1 → 2 → 3
                 \
                  4 → 5 → 6
    
    Step 2: current=2, left exists (3)
            rightmost of left subtree = 3
            3.right = 4 (current's right)
            2.right = 3, 2.left = NULL
            
            1 → 2 → 3 → 4 → 5 → 6
    
    Step 3: current=3, no left → move right
    Step 4: current=4, no left → move right
    Step 5: current=5, no left → move right
    Step 6: current=6, no left → move right → NULL → DONE
    
    Result: 1 → 2 → 3 → 4 → 5 → 6 ✓
```

### ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Reverse postorder | O(n) | O(h) |
| Iterative (Morris-like) | O(n) | **O(1)** |

---

## 10.6 Vertical Sum

### 📖 Problem

Find the sum of nodes in each **vertical line** (same horizontal distance).

```
              1 (HD=0)
            /     \
       (HD=-1)2    3(HD=1)
          / \       \
    (HD=-2)4 5(0)    7(HD=2)
    
    HD = -2: sum = 4
    HD = -1: sum = 2
    HD =  0: sum = 1 + 5 = 6
    HD =  1: sum = 3
    HD =  2: sum = 7
    
    Vertical Sums: [4, 2, 6, 3, 7]
```

### Algorithm

```
FUNCTION verticalSum(root):
    IF root == NULL: RETURN []
    
    map ← TreeMap()    // HD → sum
    
    FUNCTION dfs(node, hd):
        IF node == NULL: RETURN
        
        map[hd] ← map.getOrDefault(hd, 0) + node.data
        
        dfs(node.left, hd - 1)
        dfs(node.right, hd + 1)
    
    dfs(root, 0)
    RETURN map values sorted by HD
```

### BFS Variant

```
FUNCTION verticalSumBFS(root):
    IF root == NULL: RETURN []
    
    map ← TreeMap()
    queue ← [(root, 0)]
    
    WHILE queue is not empty:
        (node, hd) ← queue.dequeue()
        map[hd] ← map.getOrDefault(hd, 0) + node.data
        
        IF node.left: queue.enqueue((node.left, hd - 1))
        IF node.right: queue.enqueue((node.right, hd + 1))
    
    RETURN map values sorted by HD
```

### 🔍 Trace

```
              1
            /   \
           2     3
          / \   / \
         4   5 6   7

    DFS traversal:
    dfs(1, 0):  map = {0: 1}
    dfs(2, -1): map = {-1: 2, 0: 1}
    dfs(4, -2): map = {-2: 4, -1: 2, 0: 1}
    dfs(5, 0):  map = {-2: 4, -1: 2, 0: 6}       ← 1+5
    dfs(3, 1):  map = {-2: 4, -1: 2, 0: 6, 1: 3}
    dfs(6, 0):  map = {-2: 4, -1: 2, 0: 12, 1: 3}  ← 6+6
    dfs(7, 2):  map = {-2: 4, -1: 2, 0: 12, 1: 3, 2: 7}
    
    Vertical Sums: [4, 2, 12, 3, 7]
```

### ⏱️ Complexity: O(n) time, O(n) space

---

## 🧩 Advanced Patterns Summary

### When to Use What

| Technique | Use Case |
|-----------|----------|
| Threaded Tree | O(1) space traversal, fast successor |
| Expression Tree | Expression evaluation, compiler design |
| Huffman Tree | Data compression, encoding |
| Tree ↔ DLL | In-place conversion problems |
| Flatten to List | Simplify tree to linear structure |
| Vertical Sum | Column-based aggregation |

### Tree Conversion Patterns

```
    Binary Tree ─────► DLL (inorder)
         │                  Uses: left=prev, right=next
         │
         ├──────────► Linked List (preorder)
         │                  Uses: right=next, left=NULL
         │
         ├──────────► Array (level order)
         │                  Uses: index formulas
         │
         └──────────► String (serialize)
                            Uses: preorder + NULL markers
```

---

## ⏱️ Complexity Summary

| Problem | Time | Space |
|---------|------|-------|
| Threaded traversal | O(n) | **O(1)** |
| Build expression tree | O(n) | O(n) |
| Evaluate expression tree | O(n) | O(h) |
| Build Huffman tree | O(n log n) | O(n) |
| Tree to DLL | O(n) | O(h) |
| Flatten (reverse postorder) | O(n) | O(h) |
| Flatten (iterative) | O(n) | **O(1)** |
| Vertical sum | O(n) | O(n) |

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Threaded binary tree | NULL pointers replaced with inorder successor/predecessor threads |
| Right-threaded | Right NULL → inorder successor; enables O(1) space traversal |
| Expression tree | Leaves=operands, internal=operators; postfix → tree using stack |
| Evaluate expr tree | Recursive: evaluate left, right, apply operator |
| Huffman tree | Greedy: combine two smallest frequencies; prefix-free codes |
| Huffman property | Higher frequency → shorter code → closer to root |
| Tree to DLL | Inorder traversal linking prev.right→current, current.left→prev |
| Flatten to list | Reverse postorder (R,L,N) with prev pointer; or Morris-like iterative |
| Vertical sum | Map HD → cumulative sum using DFS or BFS |

---

## ❓ Quick Revision Questions

1. **How many NULL pointers does a binary tree with n nodes have, and how does threading use them?**
   <details><summary>Answer</summary>n+1 NULL pointers. Threading replaces these with pointers to inorder predecessor (left NULLs) and/or inorder successor (right NULLs), enabling O(1) space traversal without a stack.</details>

2. **In what order (postfix, prefix, or infix) is it easiest to build an expression tree, and why?**
   <details><summary>Answer</summary>Postfix (or prefix). In postfix, we process operands first and operators last. Using a stack: push operands, and when an operator is encountered, pop two operands, create a subtree, and push it back. This naturally builds the tree bottom-up without parentheses.</details>

3. **Why does Huffman coding produce optimal prefix-free codes?**
   <details><summary>Answer</summary>The greedy strategy always combines the two lowest-frequency nodes, ensuring rare characters get longer codes (deeper in tree) and frequent characters get shorter codes (closer to root). It's provably optimal among prefix-free codes by a proof using exchange argument.</details>

4. **When flattening a binary tree iteratively, why do we connect the rightmost node of the left subtree to the current right child?**
   <details><summary>Answer</summary>In preorder, the right subtree comes after all nodes in the left subtree. The rightmost node of the left subtree is the last node visited in the left subtree (in preorder). By connecting it to the right child, we ensure the preorder sequence: left subtree → right subtree, linked correctly.</details>

5. **What is the difference between "Tree to DLL" and "Flatten to Linked List"?**
   <details><summary>Answer</summary>Tree to DLL: Uses inorder order, creates a doubly linked list (both left/prev and right/next pointers). Flatten: Uses preorder order, creates a singly linked list (only right/next pointer, left is set to NULL).</details>

6. **Can vertical sum be computed using DFS? What's the advantage of BFS?**
   <details><summary>Answer</summary>Yes, both DFS and BFS work for vertical sum since we're summing all nodes at each HD regardless of level. BFS has the advantage when you also need level-based ordering (like in vertical order traversal where same-HD, same-level nodes should be ordered left to right). For pure sum, DFS is simpler.</details>

---

## 🎓 Course Completion: Master Reference

### All Tree Operations Complexity

| Operation | BST (avg) | BST (worst) | Balanced BST | General BT |
|-----------|-----------|-------------|--------------|------------|
| Search | O(log n) | O(n) | O(log n) | O(n) |
| Insert | O(log n) | O(n) | O(log n) | O(n) |
| Delete | O(log n) | O(n) | O(log n) | O(n) |
| Traversal | O(n) | O(n) | O(n) | O(n) |
| Min/Max | O(log n) | O(n) | O(log n) | O(n) |
| Height | O(n) | O(n) | O(n) | O(n) |
| LCA | O(h) | O(n) | O(log n) | O(n) |

### Problem Category Quick Reference

| Category | Key Problems | Core Technique |
|----------|-------------|----------------|
| Traversals | Inorder, Preorder, Postorder, Level Order | Recursion, Stack, Queue |
| BST Operations | Search, Insert, Delete | BST property exploitation |
| Construction | Build from traversals, Serialize/Deserialize | Divide & conquer |
| Depth/Height | Max depth, Min depth, Balanced check | Bottom-up recursion |
| Path Problems | Path sum, Max path sum, LCA | DFS + backtracking |
| View Problems | Left, Right, Top, Bottom, Boundary | BFS + HD tracking |
| Advanced | DLL conversion, Flatten, Huffman | Modified traversals |

---

[← Previous: Tree Views](../09-Tree-Views/01-tree-views.md) | [Back to TOC](../README.md)
