# 1.4 Tree vs Graph

[← Previous: Tree Properties](03-tree-properties.md) | [Back to TOC](../README.md) | [Next: Types of Trees →](05-types-of-trees.md)

---

## 📖 Key Differences

```
    TREE                          GRAPH
    
      A                        A --- B
     / \                       |   / |
    B   C                      |  /  |
   / \                         | /   |
  D   E                        C --- D
  
  - One root                   - No root concept
  - No cycles                  - Can have cycles
  - n-1 edges for n nodes      - Any number of edges
  - One path between nodes     - Multiple paths possible
  - Always connected           - Can be disconnected
```

| Feature | Tree | Graph |
|---------|------|-------|
| Structure | Hierarchical | Network |
| Root | Has exactly one root | No root concept |
| Cycles | **No cycles** (acyclic) | Can have cycles |
| Edges for `n` nodes | Exactly `n - 1` | 0 to `n(n-1)/2` |
| Path between two nodes | Exactly one | Zero or more |
| Connectivity | Always connected | May be disconnected |
| Direction | Implicitly directed (parent→child) | Directed or undirected |
| Parent-child | Every node (except root) has exactly one parent | No such constraint |

---

## 💡 Key Insight

> **Every tree is a graph, but not every graph is a tree.**
> A tree is a **connected, acyclic, undirected graph**.

---

## When to Use What?

| Use Case | Data Structure |
|----------|---------------|
| File systems, organizational charts | Tree |
| Social networks | Graph |
| XML/HTML parsing | Tree |
| Road networks, maps | Graph |
| Database indexing (B-Trees) | Tree |
| Web page links | Graph |

---

[← Previous: Tree Properties](03-tree-properties.md) | [Back to TOC](../README.md) | [Next: Types of Trees →](05-types-of-trees.md)
