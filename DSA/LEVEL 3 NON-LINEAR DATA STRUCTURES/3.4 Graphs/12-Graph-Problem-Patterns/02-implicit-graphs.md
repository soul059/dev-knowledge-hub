# Implicit Graphs and State-Space BFS

[← Previous: Grid Problems](01-grid-problems.md) | [Back to TOC](../README.md) | [Next: Multi-State BFS →](03-multi-state-bfs.md)

---

## What is an Implicit Graph?

```
    An IMPLICIT GRAPH is one where:
    • Vertices and edges are NOT explicitly listed
    • Vertices = STATES
    • Edges = TRANSITIONS between states
    • Neighbors are generated on the fly
    
    ┌──────────────────────────────────────┐
    │  "If you can model it as a graph,   │
    │   you can BFS/DFS on it."           │
    │                                      │
    │  State = Vertex                      │
    │  Valid Move = Edge                   │
    │  Shortest # of moves = BFS          │
    └──────────────────────────────────────┘
```

## Example: Open the Lock (LeetCode 752)

```
    Lock has 4 wheels, each 0-9.
    Start: "0000"
    Target: given (e.g., "0202")
    Dead-ends: list of forbidden combinations
    Find: minimum turns to reach target.
    
    STATE:  4-digit string (e.g., "0000")
    EDGES:  each digit can go up or down (8 neighbors per state)
    BFS:    shortest path from "0000" to target
    
    "0000" → "1000", "9000", "0100", "0900",
              "0010", "0090", "0001", "0009"
```

## Pseudocode

```
FUNCTION openLock(deadends, target):
    dead = set(deadends)
    IF "0000" in dead: RETURN -1
    
    queue = [("0000", 0)]        // (state, moves)
    visited = set(["0000"])
    
    WHILE queue not empty:
        (state, moves) = queue.dequeue()
        
        IF state == target: RETURN moves
        
        FOR i = 0 to 3:                     // each wheel
            FOR delta in [-1, +1]:           // turn up or down
                newDigit = (state[i] + delta) % 10
                newState = state with position i = newDigit
                
                IF newState NOT in visited AND NOT in dead:
                    visited.add(newState)
                    queue.enqueue((newState, moves + 1))
    
    RETURN -1    // unreachable
```

## General Pattern

```
    1. Define your STATE (what information fully describes
       the current configuration?)
    
    2. Define TRANSITIONS (what valid moves can you make
       from any state?)
    
    3. BFS from initial state to target state
    
    4. Use a SET for visited states (states may not be 
       simple integers — could be strings, tuples, etc.)
    
    Common State-Space Problems:
    • Sliding puzzle (state = board configuration)
    • Water jug problem (state = (jug1_level, jug2_level))
    • Word ladder (state = current word)
    • Open the lock (state = 4-digit combo)
```

---

[← Previous: Grid Problems](01-grid-problems.md) | [Back to TOC](../README.md) | [Next: Multi-State BFS →](03-multi-state-bfs.md)
