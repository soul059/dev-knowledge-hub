# Chapter 2: Shortest Path Visiting All Nodes (Bitmask BFS/DP)

## 📋 Overview
Given an **undirected, unweighted graph**, find the shortest path that visits **every node at least once**. Unlike TSP, nodes can be **revisited**. This combines **BFS** with **bitmask DP** for an $O(n^2 \cdot 2^n)$ solution.

---

## 🧠 Core Idea

### Difference from TSP
```
TSP:                          This problem:
  Visit each exactly once       Visit each at least once
  Weighted edges                Unweighted (or weighted)
  Hamiltonian cycle             Any walk covering all nodes
  DP only                       BFS + bitmask (shortest path)
```

### State
```
State: (current_node, visited_mask)

dp[mask][i] = minimum steps to reach node i
              having visited exactly the nodes in mask

BFS explores states in order of increasing steps.
```

### Why BFS works
```
Edges are unweighted → BFS finds shortest path.
State space: n · 2ⁿ states.
Each state visited at most once → guaranteed termination.
```

---

## 🔍 Step-by-Step Trace

```
Graph (n=4):
  0 — 1
  |   |
  3 — 2

Adjacency: 0:[1,3]  1:[0,2]  2:[1,3]  3:[0,2]

BFS starts from EVERY node simultaneously:
  Queue: (0, 0001, 0), (1, 0010, 0), (2, 0100, 0), (3, 1000, 0)
         (node, mask, steps)

Step 0: Initial states above

Step 1: Expand each:
  From (0, 0001): → (1, 0011, 1), (3, 1001, 1)
  From (1, 0010): → (0, 0011, 1), (2, 0110, 1)
  From (2, 0100): → (1, 0110, 1), (3, 1100, 1)
  From (3, 1000): → (0, 1001, 1), (2, 1100, 1)
  
  (0, 0011) already seen? No → add
  ...

Step 2: Expand step-1 states:
  From (1, 0011, 1): → (0, 0011, 2)✗seen, (2, 0111, 2)
  From (3, 1001, 1): → (0, 1001, 2)✗seen, (2, 1101, 2)
  From (2, 0110, 1): → (1, 0110, 2)✗seen, (3, 1110, 2)
  ...

Step 3: Some 3-bit masks expand to full mask
  From (2, 0111, 2): → (3, 1111, 3) ✓ FOUND!

Answer: 3 (could also be found via other paths at step 3)
```

---

## 💻 Pseudocode

```
function shortestPathAllNodes(graph, n):
    FULL = (1 << n) - 1
    
    // visited[mask][node] = true if this state was seen
    visited = 2D array [FULL+1][n] = false
    
    queue = empty Queue
    
    // Start BFS from every node
    for i = 0 to n-1:
        mask = 1 << i
        queue.enqueue((i, mask, 0))
        visited[mask][i] = true
    
    while queue not empty:
        (node, mask, steps) = queue.dequeue()
        
        // Check if all nodes visited
        if mask == FULL:
            return steps
        
        // Explore neighbors
        for neighbor in graph[node]:
            new_mask = mask | (1 << neighbor)
            
            if not visited[new_mask][neighbor]:
                visited[new_mask][neighbor] = true
                queue.enqueue((neighbor, new_mask, steps + 1))
    
    return -1  // not reachable (disconnected graph)
```

---

## 🔄 Why Start from Every Node?

```
We don't know the optimal starting point.

BFS from all nodes simultaneously:
  ┌───┐          ┌───┐          ┌───┐
  │ 0 │          │ 1 │          │ 2 │  ...
  │0001│         │0010│         │0100│
  └───┘          └───┘          └───┘
    ↓              ↓              ↓
  Expand        Expand        Expand
    ↓              ↓              ↓
  ──────── compete for states ────────
    ↓
  First to reach FULL mask wins

This is equivalent to adding a virtual start node
connected to all real nodes with edge weight 0.
```

---

## 🆚 Comparison with Floyd + TSP Approach

```
Alternative approach:
  1. Floyd-Warshall: all-pairs shortest paths O(n³)
  2. TSP on the distance matrix O(n²·2ⁿ)
  Total: O(n²·2ⁿ)

BFS + Bitmask:
  Direct O(n²·2ⁿ) without precomputation
  
Both are valid; BFS approach is more natural for
unweighted graphs.
```

---

## ⚡ Complexity Analysis

```
┌──────────────────────────────────────┐
│ States: n · 2ⁿ                      │
│ Each state: O(degree) transitions    │
│   sum of degrees = 2·E               │
│                                      │
│ Time:  O(n · 2ⁿ · avg_degree)       │
│        ≤ O(n² · 2ⁿ)                 │
│                                      │
│ Space: O(n · 2ⁿ)                    │
│   visited array + queue              │
│                                      │
│ n ≤ 12-15 typically feasible         │
└──────────────────────────────────────┘
```

---

## 🧩 Variants

| Variant | Change |
|---------|--------|
| Weighted edges | Use Dijkstra instead of BFS |
| Directed graph | Same algorithm, directed adjacency |
| Specific start/end | Start BFS from one node, check end node at full mask |
| Minimum edges | Same as unweighted shortest path (this problem) |
| Return to start | Check dist[node][start] at full mask |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Shortest walk visiting all nodes |
| **Key insight** | BFS on (node, visited_mask) state space |
| **State** | (current_node, bitmask of visited) |
| **Start** | All nodes simultaneously |
| **Termination** | First state with full mask |
| **Complexity** | O(n² · 2ⁿ) time, O(n · 2ⁿ) space |

---

## ❓ Quick Revision Questions

1. **How does this problem differ from TSP?**
2. **Why do we start BFS from every node simultaneously?**
3. **What is the state in the BFS?**
4. **How do we know BFS terminates?**
5. **What change would you make for weighted edges?**
6. **Why is the state space n · 2ⁿ and not 2ⁿ?**

---

[← Previous: TSP Bitmask](01-tsp-bitmask.md) | [Next: Hamiltonian Path →](03-hamiltonian-path.md)

[← Back to README](../README.md)
