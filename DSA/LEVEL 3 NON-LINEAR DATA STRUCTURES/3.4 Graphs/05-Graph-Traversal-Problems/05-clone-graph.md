# Clone Graph

[← Previous: Surrounded Regions](04-surrounded-regions.md) | [Back to TOC](../README.md) | [Next: Bipartite Check →](06-bipartite-check.md)

---

## Problem

Given a reference to a node in a connected undirected graph, return a **deep copy** (clone) of the graph. Each node has a value and a list of neighbors.

## Approach: DFS + HashMap

```
    Key Challenge: Cycles!
    If A → B → C → A, naive DFS enters infinite loop.
    
    Solution: 
    - HashMap maps: original node → cloned node
    - Before cloning, check if node already cloned
    - If yes, return the existing clone

FUNCTION cloneGraph(node):
    IF node is null: RETURN null
    
    visited = HashMap()      // old node → new node
    RETURN DFS_Clone(node, visited)

FUNCTION DFS_Clone(node, visited):
    IF node in visited:                    // already cloned!
        RETURN visited[node]               // return existing clone
    
    clone = new Node(node.val)             // create clone
    visited[node] = clone                  // map original → clone
    
    FOR each neighbor in node.neighbors:
        clone.neighbors.add(DFS_Clone(neighbor, visited))
    
    RETURN clone
```

## Trace

```
    Original:                    Clone:
       (1)──(2)                    (1')──(2')
        │    │                      │      │
       (4)──(3)                    (4')──(3')

    DFS_Clone(1):
    ├─ Create clone 1'
    ├─ visited = {1→1'}
    ├─ Process neighbor 2:
    │   ├─ DFS_Clone(2): create 2', visited = {1→1', 2→2'}
    │   ├─ Process neighbor 1: already in visited → return 1'
    │   └─ Process neighbor 3:
    │       ├─ DFS_Clone(3): create 3', add to visited
    │       └─ Process neighbor 4:
    │           ├─ DFS_Clone(4): create 4', add to visited
    │           ├─ Process neighbor 1: return 1' (existing)
    │           └─ Process neighbor 3: return 3' (existing)
    └─ Process neighbor 4: return 4' (existing)
    
    Result: Complete deep copy with same structure!
```

---

[← Previous: Surrounded Regions](04-surrounded-regions.md) | [Back to TOC](../README.md) | [Next: Bipartite Check →](06-bipartite-check.md)
