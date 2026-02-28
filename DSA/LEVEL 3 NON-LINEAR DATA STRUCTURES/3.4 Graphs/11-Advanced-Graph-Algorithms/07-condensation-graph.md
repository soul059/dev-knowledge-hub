# Condensation Graph and Summary

[← Previous: Eulerian Path and Circuit](06-eulerian-path-and-circuit.md) | [Back to TOC](../README.md) | [Next Unit: Graph Problem Patterns →](../12-Graph-Problem-Patterns/01-grid-problems.md)

---

## Condensation Graph (SCC DAG)

```
    After finding all SCCs, COLLAPSE each SCC into a single
    super-node. The resulting graph is a DAG!
    
    Why a DAG?
    If there were a cycle among super-nodes, those SCCs 
    could be merged into one larger SCC — contradiction.
    
    Original:                    Condensation:
    ┌─────────┐                  
    │ 0→1→2→0 │──→ 3 ──→ 4     [SCC₁] ──→ [3] ──→ [4]
    └─────────┘                  
    
    Uses:
    • Apply DAG algorithms (topo-sort) on condensation
    • Simplify complex dependency analysis
    • Count minimum edges to make graph strongly connected
```

## Building the Condensation Graph

```
FUNCTION buildCondensation(V, graph, sccs):
    // Assign each vertex to its SCC ID
    sccId = array of size V
    FOR i = 0 to sccs.length - 1:
        FOR each vertex v in sccs[i]:
            sccId[v] = i
    
    // Build condensation graph (no duplicate edges)
    condEdges = set()
    FOR each edge (u, v) in graph:
        IF sccId[u] != sccId[v]:
            condEdges.add((sccId[u], sccId[v]))
    
    RETURN condensation graph from condEdges
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SCC** | Maximal set where every vertex reaches every other (directed graphs) |
| **Kosaraju's** | Two DFS passes + reversed graph, O(V+E) |
| **Tarjan's** | Single DFS pass, disc/low arrays, stack, O(V+E) |
| **Articulation Point** | Vertex removal disconnects graph; low[v] >= disc[u] |
| **Bridge** | Edge removal disconnects graph; low[v] > disc[u] (strictly) |
| **AP vs Bridge** | AP: >=, root needs 2+ children. Bridge: >, no root special case |
| **Eulerian Path** | Visits every EDGE once. Polynomial time |
| **Eulerian Circuit** | Eulerian path starting/ending at same vertex |
| **Euler existence (undirected)** | Circuit: all even degree. Path: 0 or 2 odd-degree vertices |
| **Hierholzer's** | Build Eulerian circuit/path in O(E) |
| **Condensation** | Collapse SCCs → DAG. Apply DAG algorithms |

## Quick Revision Questions

1. **What is a Strongly Connected Component?**
   <details><summary>Answer</summary> A maximal set of vertices in a directed graph such that there is a directed path from every vertex in the set to every other vertex in the set. Every vertex belongs to exactly one SCC.</details>

2. **How does Kosaraju's algorithm work?**
   <details><summary>Answer</summary> Pass 1: Run DFS on the original graph and push vertices to a stack in finish order. Pass 2: Reverse all edges, then process vertices from the stack (reverse finish order) with DFS. Each DFS in pass 2 discovers one SCC. Time: O(V+E).</details>

3. **What is the difference between the articulation point and bridge conditions?**
   <details><summary>Answer</summary> Articulation Point: low[v] >= disc[u] (greater or equal). Bridge: low[v] > disc[u] (strictly greater). When low[v] == disc[u], v can reach u via a back edge, so the edge isn't a bridge, but u is still an AP since v's subtree can't go above u. Also, the AP root case (2+ children) has no equivalent for bridges.</details>

4. **When does an Eulerian circuit exist in an undirected graph?**
   <details><summary>Answer</summary> When the graph is connected (ignoring isolated vertices) and ALL vertices have even degree. An Eulerian path (not circuit) exists when exactly 0 or 2 vertices have odd degree.</details>

5. **Why is the condensation graph always a DAG?**
   <details><summary>Answer</summary> If there were a cycle among super-nodes (SCCs) in the condensation graph, all vertices in those SCCs would be mutually reachable, making them one single larger SCC. This contradicts the maximality of SCCs. Therefore, no cycle can exist.</details>

6. **What algorithm finds an Eulerian circuit in O(E) time?**
   <details><summary>Answer</summary> Hierholzer's algorithm. It uses a stack to traverse edges, and when stuck (no more unused edges), it backtracks and appends vertices to the circuit. The result is reversed at the end to get the correct order.</details>

---

[← Previous: Eulerian Path and Circuit](06-eulerian-path-and-circuit.md) | [Back to TOC](../README.md) | [Next Unit: Graph Problem Patterns →](../12-Graph-Problem-Patterns/01-grid-problems.md)
