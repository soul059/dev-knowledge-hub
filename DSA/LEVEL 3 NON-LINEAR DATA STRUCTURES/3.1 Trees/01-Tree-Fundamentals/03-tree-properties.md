# 1.3 Tree Properties (Height, Depth, Level)

[← Previous: Tree Terminology](02-tree-terminology.md) | [Back to TOC](../README.md) | [Next: Tree vs Graph →](04-tree-vs-graph.md)

---

## 📖 Definitions

| Property | Definition | Counting |
|----------|-----------|----------|
| **Depth of a node** | Number of edges from the **root** to that node | Top → Down |
| **Height of a node** | Number of edges from that node to the **deepest leaf** in its subtree | Bottom → Up |
| **Height of tree** | Height of the root node | = max depth of any leaf |
| **Level** | Set of all nodes at a given depth | Level `d` = all nodes with depth `d` |

---

## Step-by-Step Calculation

```
            A          Depth=0, Height=3
           / \
          B   C        Depth=1, Height(B)=2, Height(C)=1
         / \   \
        D   E   F      Depth=2, Height(D)=1, Height(E)=0, Height(F)=0
       /
      G                Depth=3, Height=0
```

| Node | Depth | Height | Level |
|------|-------|--------|-------|
| A    | 0     | 3      | 0     |
| B    | 1     | 2      | 1     |
| C    | 1     | 1      | 1     |
| D    | 2     | 1      | 2     |
| E    | 2     | 0      | 2     |
| F    | 2     | 0      | 2     |
| G    | 3     | 0      | 3     |

---

## Height Calculation Algorithm

```
FUNCTION height(node):
    IF node is NULL:
        RETURN -1              // Convention: height of empty tree = -1
    
    leftHeight  = height(node.left)
    rightHeight = height(node.right)
    
    RETURN 1 + MAX(leftHeight, rightHeight)
```

**Alternative convention**: Some textbooks count **nodes** instead of **edges**, so an empty tree has height `0` and a single node has height `1`. Be consistent in your approach.

---

## 🔍 Trace: Height Calculation

```
Tree:       1
           / \
          2   3
         /
        4

height(4) → left=height(NULL)=-1, right=height(NULL)=-1 → 1+max(-1,-1) = 0
height(2) → left=height(4)=0, right=height(NULL)=-1 → 1+max(0,-1) = 1
height(3) → left=height(NULL)=-1, right=height(NULL)=-1 → 1+max(-1,-1) = 0
height(1) → left=height(2)=1, right=height(3)=0 → 1+max(1,0) = 2

Answer: Height = 2
```

---

## ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Recursive height | O(n) | O(h) — call stack |

---

[← Previous: Tree Terminology](02-tree-terminology.md) | [Back to TOC](../README.md) | [Next: Tree vs Graph →](04-tree-vs-graph.md)
