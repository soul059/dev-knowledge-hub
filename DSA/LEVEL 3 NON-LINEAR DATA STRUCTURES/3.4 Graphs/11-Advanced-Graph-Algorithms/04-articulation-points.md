# Articulation Points (Cut Vertices)

[← Previous: Tarjan's Algorithm](03-tarjans-algorithm.md) | [Back to TOC](../README.md) | [Next: Bridges →](05-bridges.md)

---

## Definition

```
    An ARTICULATION POINT (cut vertex) is a vertex whose
    removal DISCONNECTS the graph (increases the number 
    of connected components).
    
    ┌─────────────────────────────────────────┐
    │  Only applies to UNDIRECTED graphs.     │
    │  Critical for network reliability.      │
    │                                         │
    │  Example: If a router is an AP, losing  │
    │  it splits the network!                 │
    └─────────────────────────────────────────┘
```

## Visual Example

```
    (0)───(1)───(3)───(4)
           │     │
          (2)   (5)
    
    Articulation Points: 1, 3
    
    Remove 1 → {0} separated from {2,3,4,5}
    Remove 3 → {0,1,2} separated from {4}, {5}
```

## Algorithm (Modified Tarjan's)

```
    Uses disc[] and low[] arrays (like Tarjan for SCC).
    
    A vertex u is an Articulation Point if:
    
    Case 1: u is the ROOT of DFS tree
            AND u has 2+ children in DFS tree
    
    Case 2: u is NOT the root
            AND has a child v such that:
            low[v] >= disc[u]
            (no back edge from v's subtree reaches above u)
```

## Pseudocode

```
FUNCTION findAPs(V, graph):
    disc = array of size V, all -1
    low = array of size V, all -1
    parent = array of size V, all -1
    isAP = array of size V, all false
    timer = 0
    
    FOR i = 0 to V-1:
        IF disc[i] == -1:
            dfs(i, graph, disc, low, parent, isAP, timer)
    
    RETURN all i where isAP[i] = true

FUNCTION dfs(u, graph, disc, low, parent, isAP, timer):
    disc[u] = low[u] = timer++
    children = 0
    
    FOR each v in graph[u]:
        IF disc[v] == -1:                    // unvisited
            children++
            parent[v] = u
            dfs(v, ...)
            low[u] = min(low[u], low[v])
            
            // Case 1: root with 2+ children
            IF parent[u] == -1 AND children > 1:
                isAP[u] = true
            
            // Case 2: non-root, subtree can't bypass u
            IF parent[u] != -1 AND low[v] >= disc[u]:
                isAP[u] = true
        
        ELSE IF v != parent[u]:              // back edge
            low[u] = min(low[u], disc[v])
```

## Trace

```
    Graph: 0─1, 1─2, 1─3, 3─4, 3─5
    
    DFS from 0:
    
    Visit 0: disc=0, low=0
      Visit 1: disc=1, low=1, parent=0
        Visit 2: disc=2, low=2, parent=1
          No unvisited neighbors
          disc[2]==low[2]? → but checking AP:
          Back to 1: low[1]=min(1,low[2])=min(1,2)=1
                     low[2]=2 >= disc[1]=1 → 1 is AP? YES
        Visit 3: disc=3, low=3, parent=1
          Visit 4: disc=4, low=4, parent=3
            Back to 3: low[3]=min(3,4)=3
                       low[4]=4 >= disc[3]=3 → 3 is AP? YES
          Visit 5: disc=5, low=5, parent=3
            Back to 3: low[3]=min(3,5)=3
                       low[5]=5 >= disc[3]=3 → 3 is AP (already)
          Back to 1: low[1]=min(1,3)=1
                     low[3]=3 >= disc[1]=1 → 1 is AP (already)
      Back to 0: low[0]=min(0,1)=0
        0 is root, children=1 → NOT an AP
    
    Articulation Points: {1, 3} ✓
```

## Complexity

```
    Time:  O(V + E)  — single DFS
    Space: O(V)      — arrays + recursion stack
```

---

[← Previous: Tarjan's Algorithm](03-tarjans-algorithm.md) | [Back to TOC](../README.md) | [Next: Bridges →](05-bridges.md)
