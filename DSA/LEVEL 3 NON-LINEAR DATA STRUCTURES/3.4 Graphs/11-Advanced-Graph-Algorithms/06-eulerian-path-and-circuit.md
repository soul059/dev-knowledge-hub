# Eulerian Path and Circuit

[← Previous: Bridges](05-bridges.md) | [Back to TOC](../README.md) | [Next: Condensation Graph →](07-condensation-graph.md)

---

## Definitions

```
    EULERIAN PATH:    visits every EDGE exactly once
    EULERIAN CIRCUIT: Eulerian path that starts & ends at same vertex
    
    Compare with:
    HAMILTONIAN PATH: visits every VERTEX exactly once (NP-complete!)
    
    ┌──────────────────────────────────────────────┐
    │  Euler = EDGES, Hamilton = VERTICES           │
    │  Euler is polynomial, Hamilton is NP-complete │
    └──────────────────────────────────────────────┘
```

## Existence Conditions

### Undirected Graph

```
    Eulerian CIRCUIT exists iff:
    1. Graph is connected (ignoring degree-0 vertices)
    2. ALL vertices have EVEN degree
    
    Eulerian PATH (not circuit) exists iff:
    1. Graph is connected
    2. Exactly 0 or 2 vertices have ODD degree
       (If 2: they are the start and end vertices)
```

### Directed Graph

```
    Eulerian CIRCUIT exists iff:
    1. Graph is strongly connected
    2. in-degree == out-degree for ALL vertices
    
    Eulerian PATH exists iff:
    1. Graph is connected (weakly)
    2. At most 1 vertex: out-degree = in-degree + 1 (start)
       At most 1 vertex: in-degree = out-degree + 1 (end)
       All others: in-degree == out-degree
```

## Hierholzer's Algorithm

```
    Finds an Eulerian circuit/path in O(E) time.
    
    FUNCTION hierholzer(graph, start):
        stack = [start]
        circuit = []
        
        WHILE stack not empty:
            u = stack.top()
            IF graph[u] has remaining edges:
                v = next unused neighbor of u
                Remove edge (u, v) from graph
                stack.push(v)
            ELSE:
                stack.pop()
                circuit.append(u)
        
        RETURN circuit reversed
    
    Key Insight: When stuck (no more edges from u),
    add u to result. Then backtrack and explore 
    remaining edges from previous vertices.
```

## Trace

```
    Graph (directed): 0→1, 1→2, 2→0, 0→3, 3→0
    All vertices: in-degree == out-degree ✓
    
    Start at 0:
    Stack: [0]
      Edge 0→1: Stack: [0,1]
      Edge 1→2: Stack: [0,1,2]
      Edge 2→0: Stack: [0,1,2,0]
      Edge 0→3: Stack: [0,1,2,0,3]
      Edge 3→0: Stack: [0,1,2,0,3,0]
    
    No more edges from 0: pop → circuit: [0]
    No more edges from 3: pop → circuit: [0,3]
    No more edges from 0: pop → circuit: [0,3,0]
    No more edges from 2: pop → circuit: [0,3,0,2]
    No more edges from 1: pop → circuit: [0,3,0,2,1]
    No more edges from 0: pop → circuit: [0,3,0,2,1,0]
    
    Reverse: 0→1→2→0→3→0 ✓ (visits all 5 edges exactly once)
```

---

[← Previous: Bridges](05-bridges.md) | [Back to TOC](../README.md) | [Next: Condensation Graph →](07-condensation-graph.md)
