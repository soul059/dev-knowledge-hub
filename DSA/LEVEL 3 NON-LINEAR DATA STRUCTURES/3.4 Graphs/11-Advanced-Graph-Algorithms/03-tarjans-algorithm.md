# Tarjan's Algorithm

[← Previous: Kosaraju's Algorithm](02-kosarajus-algorithm.md) | [Back to TOC](../README.md) | [Next: Articulation Points →](04-articulation-points.md)

---

## Key Idea

```
    Single-pass DFS algorithm for finding SCCs.
    
    Uses two arrays:
    • disc[u] — discovery time of vertex u
    • low[u]  — lowest discovery time reachable from 
                subtree rooted at u (through back edges)
    
    Uses a STACK to track vertices in current DFS path.
    
    SCC root condition: disc[u] == low[u]
    → When true, pop all vertices up to u from stack = one SCC
```

## Algorithm

```
FUNCTION tarjanSCC(V, graph):
    disc = array of size V, all -1
    low = array of size V, all -1
    onStack = array of size V, all false
    stack = []
    timer = 0
    sccs = []
    
    FOR i = 0 to V-1:
        IF disc[i] == -1:
            dfs(i, graph, disc, low, onStack, stack, timer, sccs)
    
    RETURN sccs

FUNCTION dfs(u, graph, disc, low, onStack, stack, timer, sccs):
    disc[u] = low[u] = timer++
    stack.push(u)
    onStack[u] = true
    
    FOR each v in graph[u]:
        IF disc[v] == -1:          // unvisited
            dfs(v, ...)
            low[u] = min(low[u], low[v])
        ELSE IF onStack[v]:        // back edge to ancestor in current SCC
            low[u] = min(low[u], disc[v])
    
    // If u is root of an SCC
    IF disc[u] == low[u]:
        component = []
        DO:
            w = stack.pop()
            onStack[w] = false
            component.append(w)
        WHILE w != u
        sccs.append(component)
```

## Step-by-Step Trace

```
    Graph: 0→1, 1→2, 2→0, 1→3, 3→4
    
    DFS from 0:
    
    Visit 0: disc[0]=0, low[0]=0, stack=[0]
      Visit 1: disc[1]=1, low[1]=1, stack=[0,1]
        Visit 2: disc[2]=2, low[2]=2, stack=[0,1,2]
          Edge 2→0: onStack[0]=T → low[2]=min(2,disc[0])=0
        Back to 1: low[1]=min(1,low[2])=0
        Visit 3: disc[3]=3, low[3]=3, stack=[0,1,2,3]
          Visit 4: disc[4]=4, low[4]=4, stack=[0,1,2,3,4]
            No neighbors.
            disc[4]==low[4]? 4==4 YES → SCC: pop until 4 → {4}
            stack=[0,1,2,3]
          Back to 3: low[3]=min(3,low[4])=min(3,4)=3
          disc[3]==low[3]? 3==3 YES → SCC: pop until 3 → {3}
          stack=[0,1,2]
        Back to 1: low[1]=min(0,low[3])=min(0,3)=0
      Back to 0: low[0]=min(0,low[1])=0
      disc[0]==low[0]? 0==0 YES → SCC: pop until 0 → {2,1,0}
      stack=[]
    
    Result: SCCs = {4}, {3}, {0,1,2} ✓
```

## Tarjan vs Kosaraju

| Feature | Kosaraju's | Tarjan's |
|---------|-----------|----------|
| **DFS passes** | 2 | 1 |
| **Extra graph** | Reversed graph needed | No |
| **Space** | O(V + E) extra for reversal | O(V) for stack + arrays |
| **Complexity** | O(V + E) | O(V + E) |
| **Conceptual** | Easier to understand | More elegant but tricky |
| **Implementation** | Simpler code | More complex |

---

[← Previous: Kosaraju's Algorithm](02-kosarajus-algorithm.md) | [Back to TOC](../README.md) | [Next: Articulation Points →](04-articulation-points.md)
