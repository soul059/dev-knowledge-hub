# Chapter 1: What is Union-Find?

## Overview

Union-Find (also called **Disjoint Set Union** or **DSU**) is a data structure that tracks a collection of elements partitioned into **non-overlapping (disjoint) sets**. It provides near-constant-time operations for two key tasks:

1. **Find**: Determine which set a particular element belongs to
2. **Union**: Merge two sets into one

---

## The Core Idea

Imagine you have a group of people at a party. Initially, nobody knows each other — everyone is in their own "group." As people start talking, groups form. Union-Find answers two questions efficiently:

- **"Are person A and person B in the same group?"** → `Find`
- **"Person A's group and person B's group should merge"** → `Union`

```
INITIAL STATE: Everyone is their own group
┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
│ 0 │ │ 1 │ │ 2 │ │ 3 │ │ 4 │
└───┘ └───┘ └───┘ └───┘ └───┘
  ↑     ↑     ↑     ↑     ↑
 Set0  Set1  Set2  Set3  Set4

AFTER Union(0,1) and Union(2,3):
┌───────┐   ┌───────┐   ┌───┐
│  0  1 │   │  2  3 │   │ 4 │
└───────┘   └───────┘   └───┘
   Set A       Set B     Set C

AFTER Union(1,3):       (merges Set A and Set B)
┌───────────────┐   ┌───┐
│  0  1  2  3   │   │ 4 │
└───────────────┘   └───┘
      Set D          Set C
```

---

## Why "Union-Find"?

The name comes directly from its two operations:

| Operation | Purpose | Question Answered |
|-----------|---------|-------------------|
| **Find(x)** | Find the representative (root) of x's set | "Which group does x belong to?" |
| **Union(x, y)** | Merge the sets containing x and y | "Combine these two groups" |

---

## Key Terminology

```
┌─────────────────────────────────────────────────┐
│                 TERMINOLOGY                      │
├─────────────┬───────────────────────────────────┤
│ Element     │ An individual item in a set        │
│ Set/Group   │ A collection of connected elements │
│ Representative │ The "leader" or root of a set   │
│ Parent      │ The node above in the tree         │
│ Root        │ Node whose parent is itself        │
│ Rank        │ Upper bound on tree height         │
│ Size        │ Number of elements in a set        │
└─────────────┴───────────────────────────────────┘
```

---

## Union-Find as a Forest

Union-Find represents each set as a **rooted tree**. The collection of all sets forms a **forest** (a collection of trees).

```
Forest with 3 sets:

    Tree 1      Tree 2      Tree 3
      0           2           4
     / \          |
    1   5         3
    |
    6

Set 1: {0, 1, 5, 6}    root = 0
Set 2: {2, 3}           root = 2
Set 3: {4}              root = 4
```

- Each node has a **parent pointer**
- The **root** points to itself
- Two elements are in the same set ⟺ they have the same root

---

## How It Differs from Other Data Structures

| Feature | Union-Find | Hash Map | Graph (Adj List) |
|---------|-----------|----------|-------------------|
| "Same set?" query | O(α(n)) ≈ O(1) | O(1) per lookup | O(V + E) BFS/DFS |
| Merge two sets | O(α(n)) ≈ O(1) | O(n) rebuild | O(1) add edge |
| Dynamic groups | ✅ Excellent | ❌ Expensive | ❌ Re-traverse |
| Space | O(n) | O(n) | O(V + E) |
| Split/Delete | ❌ Not supported | ✅ Easy | ✅ Easy |

**Key insight**: Union-Find excels when you need to **repeatedly merge groups** and **check connectivity**, but never need to split them apart.

---

## Real-World Analogy

Think of **social networks**:
- Each person starts as their own "network"
- When two people become friends, their networks merge
- You can quickly check: "Are Alice and Bob in the same network?"
- Networks only grow — friendships aren't broken

```
Step 1: Alice ─ friends ─ Bob       (Union Alice, Bob)
Step 2: Carol ─ friends ─ Dave      (Union Carol, Dave)
Step 3: Bob ─ friends ─ Carol       (Union Bob, Carol)

Now: Alice, Bob, Carol, Dave are ALL in the same network!
      Find(Alice) == Find(Dave)  → TRUE
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Data Structure | Tracks elements in non-overlapping sets |
| Core Operations | Find (which set?) and Union (merge sets) |
| Representation | Forest of rooted trees |
| Best For | Dynamic connectivity, grouping problems |
| Not For | Splitting groups, ordered data |
| Time Complexity | Nearly O(1) with optimizations |

---

## Quick Revision Questions

1. **What two operations does Union-Find support?**
2. **What does the "Find" operation return?**
3. **How does Union-Find represent sets internally?**
4. **Can Union-Find split a group into two smaller groups?**
5. **When two elements have the same root, what does that mean?**

---

[Next: Disjoint Sets Concept →](02-disjoint-sets-concept.md)

[← Back to Unit 1 README](../README.md)
