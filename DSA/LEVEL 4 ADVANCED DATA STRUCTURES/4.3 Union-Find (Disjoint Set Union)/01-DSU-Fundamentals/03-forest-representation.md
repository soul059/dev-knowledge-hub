# Chapter 3: Forest Representation

## Overview

Union-Find represents its disjoint sets as a **forest** — a collection of rooted trees. Each tree represents one set, and the root of each tree serves as the **representative** of that set.

---

## From Abstract Sets to Trees

Each element becomes a **node**, and each node has a **parent pointer**. The root points to itself.

```
Abstract Set: {0, 1, 2, 3}  with representative 0

Tree Representation:

        0          ← root (parent[0] = 0)
       / \
      1   2        ← parent[1] = 0, parent[2] = 0
      |
      3            ← parent[3] = 1

parent[] array: [0, 0, 0, 1]
                 ↑  ↑  ↑  ↑
     index:      0  1  2  3
```

---

## The Parent Array

The entire Union-Find structure is stored in a single **parent array** (or `parent[]` / `p[]`).

```
Rule: parent[root] = root  (root is its own parent)

Example with 8 elements, 3 sets:

  Set 1: {0, 1, 3, 5}     Set 2: {2, 4}     Set 3: {6, 7}

  Trees:
      0              2            6
     / \             |            |
    1   5            4            7
    |
    3

  parent[]: 
  Index:    0  1  2  3  4  5  6  7
  Value:   [0, 0, 2, 1, 2, 0, 6, 6]
            ↑        ↑        ↑
          root     root     root
       (self)    (self)   (self)
```

---

## How to Read the Parent Array

```
parent[] = [0, 0, 2, 1, 2, 0, 6, 6]

Element 0: parent[0] = 0 → root! (representative of its set)
Element 1: parent[1] = 0 → parent is 0 → 0 is root → Find(1) = 0
Element 2: parent[2] = 2 → root! (representative of its set)
Element 3: parent[3] = 1 → parent is 1 → parent[1] = 0 → root!
                           → Find(3) = 0
Element 4: parent[4] = 2 → parent is 2 → 2 is root → Find(4) = 2
Element 5: parent[5] = 0 → parent is 0 → 0 is root → Find(5) = 0
Element 6: parent[6] = 6 → root!
Element 7: parent[7] = 6 → parent is 6 → 6 is root → Find(7) = 6
```

---

## Initialization: The Forest of Singletons

Initially, every element is its own root (each element is a singleton set):

```
MakeSet for elements 0..5:

  parent[] = [0, 1, 2, 3, 4, 5]

  Visualization:
  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐
  │0│  │1│  │2│  │3│  │4│  │5│
  └┬┘  └┬┘  └┬┘  └┬┘  └┬┘  └┬┘
   ↺    ↺    ↺    ↺    ↺    ↺
   │    │    │    │    │    │
  self  self self self self self

  Each node points to itself → 6 separate trees → 6 sets
```

**Pseudocode:**
```
function MakeSet(n):
    parent = new array of size n
    for i = 0 to n-1:
        parent[i] = i       // each element is its own root
```

---

## Building Trees via Union

When we call `Union(x, y)`, we make one root point to the other:

```
Step 1: Union(3, 4)
  Before:  parent = [0, 1, 2, 3, 4, 5]
  
     3      4
     ↺      ↺
  
  Make root of 4 point to root of 3:
     parent[4] = 3
  
  After:   parent = [0, 1, 2, 3, 3, 5]
  
     3
     |
     4

Step 2: Union(1, 3)
  Before:  parent = [0, 1, 2, 3, 3, 5]
  
     1      3
     ↺      |
            4
  
  Make root of 1 point to root of 3:
     parent[1] = 3
  
  After:   parent = [0, 3, 2, 3, 3, 5]
  
       3
      / \
     1   4

Step 3: Union(0, 1)
  Before:  parent = [0, 3, 2, 3, 3, 5]
  
     0        3
     ↺       / \
            1   4
  
  Find(0) = 0, Find(1) = 3
  Make root 0 point to root 3:
     parent[0] = 3
  
  After:   parent = [3, 3, 2, 3, 3, 5]
  
         3
       / | \
      0  1  4
```

---

## Why Trees? (Not Lists or Arrays)

| Representation | Find | Union | Memory |
|---------------|------|-------|--------|
| **Linked Lists** | O(n) scan | O(1) append | O(n) |
| **Arrays (set ID)** | O(1) lookup | O(n) update all | O(n) |
| **Trees (parent)** | O(tree height) | O(1) link roots | O(n) |

Trees give the best **balance** between Find and Union. With optimizations (path compression + union by rank), both become nearly O(1).

---

## Tree Shape Matters!

The shape of the tree directly impacts performance:

```
GOOD: Wide & Shallow (balanced)     BAD: Tall & Skinny (degenerate)

        0                                0
      / | \                              |
     1  2  3                             1
    /|    / \                            |
   4 5   6   7                           2
                                         |
  Height = 2                             3
  Find = O(2)                            |
                                         4
                                    Height = 4
                                    Find = O(4)
```

This is why optimizations like **path compression** and **union by rank** are critical — they keep trees shallow.

---

## Memory Layout

```
Typical implementation uses 1-3 arrays:

┌──────────────────────────────────────┐
│  parent[]:  Stores parent of each    │  Required
│             node. Root: parent[i]=i  │
├──────────────────────────────────────┤
│  rank[]:    Upper bound on subtree   │  Optional
│             height (for union by     │  (for optimization)
│             rank)                    │
├──────────────────────────────────────┤
│  size[]:    Number of elements in    │  Optional
│             subtree (for union by    │  (for optimization)
│             size)                    │
└──────────────────────────────────────┘

Total space: O(n) regardless of which arrays are used
```

---

## Step-by-Step Trace: Building a Forest

```
Elements: {A, B, C, D, E}  (using indices 0-4)

Initial:  parent = [0, 1, 2, 3, 4]
Trees:    A  B  C  D  E     (5 singleton trees)

Union(A, B):  parent = [0, 0, 2, 3, 4]
Trees:    A   C  D  E
          |
          B

Union(C, D):  parent = [0, 0, 2, 2, 4]
Trees:    A   C   E
          |   |
          B   D

Union(B, D):  Find(B)=A, Find(D)=C → parent[C] = A
              parent = [0, 0, 0, 2, 4]
Trees:      A        E
           / \
          B   C
              |
              D

Union(E, A):  Find(E)=E, Find(A)=A → parent[E] = A
              parent = [0, 0, 0, 2, 0]
Trees:        A
           / | \
          B  C  E
             |
             D

Final forest: ONE tree with root A containing all elements
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Forest | Collection of rooted trees, one per set |
| Parent Array | `parent[i]` stores i's parent; root has `parent[root]=root` |
| Root | Representative of the set; its own parent |
| Initialization | Every element starts as its own root |
| Union | Link one root to another |
| Tree Height | Determines Find performance |
| Space | O(n) — just the parent array |

---

## Quick Revision Questions

1. **What does `parent[i] = i` signify?**
2. **How many arrays are minimally needed for Union-Find?**
3. **How do you determine the root of an element in the parent array?**
4. **What happens to the number of trees after a Union operation?**
5. **Why is a tall, skinny tree bad for Union-Find performance?**
6. **What is the initial state of the parent array for n elements?**

---

[← Previous: Disjoint Sets Concept](02-disjoint-sets-concept.md) | [Next: Basic Operations →](04-basic-operations.md)
