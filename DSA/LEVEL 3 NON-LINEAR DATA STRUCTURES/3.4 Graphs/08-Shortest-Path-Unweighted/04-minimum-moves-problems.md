# Minimum Moves Problems

[← Previous: Word Ladder](03-word-ladder.md) | [Back to TOC](../README.md) | [Next: Multi-Source BFS →](05-multi-source-bfs.md)

---

## Pattern: State-Space BFS

```
    "Minimum number of moves" → BFS on a STATE GRAPH
    
    Vertex = a state (position, configuration, etc.)
    Edge = one valid move between states
    BFS level = number of moves
    
    Since each "move" has cost 1, BFS finds
    the minimum moves!
```

## Example: Knight Moves

```
    Chess knight on an infinite board.
    Minimum moves from (0,0) to (x,y)?
    
    Knight moves: 8 possible from any position
    dx = [-2,-2,-1,-1, 1, 1, 2, 2]
    dy = [-1, 1,-2, 2,-2, 2,-1, 1]
    
FUNCTION minKnightMoves(x, y):
    visited = HashSet()
    queue = [(0, 0)]
    visited.add((0, 0))
    moves = 0
    
    WHILE queue not empty:
        size = queue.size()
        FOR i = 1 to size:
            (cx, cy) = queue.dequeue()
            IF cx == x AND cy == y: RETURN moves
            
            FOR d = 0 to 7:
                nx = cx + dx[d]
                ny = cy + dy[d]
                IF (nx, ny) NOT in visited:
                    visited.add((nx, ny))
                    queue.enqueue((nx, ny))
        moves += 1
    
    RETURN -1
```

## "Open the Lock" Example

```
    Lock has 4 wheels, each 0-9. Start at "0000".
    Each move: turn one wheel by 1 (up or down, wrapping).
    Given deadends (forbidden states), find min moves to target.
    
    State = 4-digit string
    Neighbors = 8 possible states (4 wheels × 2 directions)
    
    "0000" → "1000", "9000", "0100", "0900", 
              "0010", "0090", "0001", "0009"
    
    BFS from "0000", skip deadends, find target!
```

---

[← Previous: Word Ladder](03-word-ladder.md) | [Back to TOC](../README.md) | [Next: Multi-Source BFS →](05-multi-source-bfs.md)
