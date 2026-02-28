# Chapter 2: DFS Fundamentals

## 📋 Chapter Overview
Depth-First Search explores as **deep** as possible before backtracking. Uses a **stack** (explicit or recursion call stack). Essential for **connectivity, cycle detection, topological sort, and path finding**.

---

## 🧠 Core Concept

```
  DFS on a tree/graph:
  Go as deep as possible along one branch,
  then backtrack and try the next branch.

        A
       / \
      B   C
     / \   \
    D   E   F
    
  DFS Order (pre-order): A → B → D → E → C → F
  
  Path: A ──→ B ──→ D (dead end)
              ↑ backtrack
              B ──→ E (dead end)
        ↑ backtrack
        A ──→ C ──→ F (dead end)
        DONE
```

---

## 📝 DFS Templates

### Recursive DFS (Most Common)

```
function dfs(graph, node, visited):
    visited.add(node)
    process(node)                       // pre-order processing
    
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
    
    // post-order processing can go here
```

### Iterative DFS (Using Stack)

```
function dfsIterative(graph, source):
    stack = new Stack()
    visited = new Set()
    
    stack.push(source)
    
    while stack is not empty:
        node = stack.pop()
        
        if node in visited: continue    // skip if already visited
        visited.add(node)
        process(node)
        
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.push(neighbor)
```

### Key Difference from BFS

```
  BFS:                          DFS:
  Queue (FIFO)                  Stack (LIFO) / Recursion
  Mark before enqueue           Mark on visit (pop/call)
  Level-by-level                Branch-by-branch
  Shortest path ✓               No shortest path guarantee
```

---

## 🔍 Three Traversal Orders (Trees)

```
        1
       / \
      2   3
     / \
    4   5
```

| Order | When Process | Sequence | Use Case |
|-------|-------------|----------|----------|
| Pre-order | BEFORE children | 1,2,4,5,3 | Copy tree, serialize |
| In-order | BETWEEN children | 4,2,5,1,3 | BST sorted output |
| Post-order | AFTER children | 4,5,2,3,1 | Delete tree, calc size |

### Templates

```
function preorder(node):
    if node is null: return
    process(node)              // ← before
    preorder(node.left)
    preorder(node.right)

function inorder(node):
    if node is null: return
    inorder(node.left)
    process(node)              // ← between
    inorder(node.right)

function postorder(node):
    if node is null: return
    postorder(node.left)
    postorder(node.right)
    process(node)              // ← after
```

---

## 🔍 DFS on Grid

### Number of Islands (LeetCode 200)

```
  Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

  Islands = 3
```

```
function numIslands(grid):
    count = 0
    for r = 0 to rows-1:
        for c = 0 to cols-1:
            if grid[r][c] == '1':
                count += 1
                dfs(grid, r, c)      // sink the island
    return count

function dfs(grid, r, c):
    if r < 0 OR r >= rows OR c < 0 OR c >= cols: return
    if grid[r][c] != '1': return
    
    grid[r][c] = '0'                 // mark visited (sink)
    dfs(grid, r-1, c)               // up
    dfs(grid, r+1, c)               // down
    dfs(grid, r, c-1)               // left
    dfs(grid, r, c+1)               // right
```

### Trace (first island)

```
  Start: (0,0)='1' → sink, recurse
  
  Step 1: (0,0)→'0', go up(-1,0)→out of bounds
                      go down(1,0)→'1'
  Step 2: (1,0)→'0', go down(2,0)→'0', go right(1,1)→'1'
  Step 3: (1,1)→'0', go up(0,1)→'1'
  Step 4: (0,1)→'0', all neighbors 0 or out → backtrack
  
  After DFS from (0,0):
  0 0 0 0 0
  0 0 0 0 0    ← first island fully sunk
  0 0 1 0 0
  0 0 0 1 1
```

---

## 🔍 Connected Components

```
function countComponents(n, edges):
    graph = buildAdjList(n, edges)
    visited = new Set()
    count = 0
    
    for node = 0 to n-1:
        if node not in visited:
            count += 1
            dfs(graph, node, visited)
    
    return count
```

**Pattern:** Call DFS from each unvisited node. Each call = one component.

---

## 🔍 Cycle Detection

### In Undirected Graph

```
function hasCycle(graph, node, visited, parent):
    visited.add(node)
    
    for neighbor in graph[node]:
        if neighbor not in visited:
            if hasCycle(graph, neighbor, visited, node):
                return true
        else if neighbor != parent:     // visited but not parent → cycle!
            return true
    
    return false
```

### In Directed Graph (3-color / state tracking)

```
  States: WHITE (unvisited), GRAY (in progress), BLACK (done)
  
  If we reach a GRAY node → back edge → CYCLE
```

```
function hasCycleDirected(graph, node, state):
    state[node] = GRAY                 // processing
    
    for neighbor in graph[node]:
        if state[neighbor] == GRAY:    // back edge!
            return true
        if state[neighbor] == WHITE:
            if hasCycleDirected(graph, neighbor, state):
                return true
    
    state[node] = BLACK                // done
    return false
```

---

## 📊 Complexity

| Aspect | Value |
|--------|-------|
| Time | O(V + E) for graph, O(m×n) for grid |
| Space | O(V) visited + O(V) recursion stack |
| Max recursion depth | O(V) worst case |

### Stack Overflow Risk

```
  Deep recursion (e.g., 10^5 nodes in a chain)
  → Use iterative DFS with explicit stack
  → Or increase stack size limit
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Data structure | Stack (explicit) or recursion (implicit) |
| Exploration | Go deep, then backtrack |
| Three orders | Pre, In, Post — choose based on when to process |
| Grid DFS | Sink visited cells (mark as '0' or visited) |
| Components | Count DFS calls from unvisited nodes |
| Cycle (undirected) | Visited neighbor ≠ parent → cycle |
| Cycle (directed) | GRAY → GRAY → cycle (3-color) |

---

## ❓ Revision Questions

1. **DFS uses which data structure?**
   → Stack (explicit) or recursion call stack (implicit).

2. **How does DFS differ from BFS for shortest path?**
   → DFS does NOT guarantee shortest path; BFS does (unweighted).

3. **How to detect a cycle in an undirected graph with DFS?**
   → If a visited neighbor is not the parent of the current node, there's a cycle.

4. **What's the 3-color method for directed graphs?**
   → WHITE=unvisited, GRAY=in-progress, BLACK=done. GRAY→GRAY edge = cycle.

5. **When to prefer iterative DFS over recursive?**
   → When graph is deep (risk of stack overflow), e.g., chain of 10⁵ nodes.

---

[← Previous: BFS Fundamentals](01-bfs-fundamentals.md) | [Next: Graph Traversal Applications →](03-graph-traversal-applications.md)
