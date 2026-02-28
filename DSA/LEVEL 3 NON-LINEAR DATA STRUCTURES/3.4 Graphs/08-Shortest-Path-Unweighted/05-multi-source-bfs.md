# Multi-Source BFS

[← Previous: Minimum Moves Problems](04-minimum-moves-problems.md) | [Back to TOC](../README.md) | [Next: 0-1 BFS →](06-zero-one-bfs.md)

---

## Key Idea

```
    Start BFS from MULTIPLE sources simultaneously.
    Enqueue ALL source vertices at level 0.
    
    This finds the shortest distance from ANY source
    to every vertex, in a single BFS run.
    
    Trick: Imagine a "super source" S connected to
    all actual sources. BFS from S = multi-source BFS.
```

## Example: Rotten Oranges

```
    Grid of oranges: 0=empty, 1=fresh, 2=rotten.
    Every minute, rotten oranges rot adjacent fresh oranges.
    Find minimum time until all oranges are rotten (or -1 if impossible).

FUNCTION orangesRotting(grid):
    queue = new Queue()
    freshCount = 0
    
    // Step 1: Enqueue ALL rotten oranges
    FOR each cell (r, c):
        IF grid[r][c] == 2:
            queue.enqueue((r, c))          // multiple sources!
        ELSE IF grid[r][c] == 1:
            freshCount += 1
    
    IF freshCount == 0: RETURN 0           // nothing to rot
    
    // Step 2: BFS level by level
    minutes = 0
    dr = [-1, 1, 0, 0]
    dc = [0, 0, -1, 1]
    
    WHILE queue not empty:
        size = queue.size()
        FOR i = 1 to size:
            (r, c) = queue.dequeue()
            FOR d = 0 to 3:
                nr = r + dr[d]
                nc = c + dc[d]
                IF valid(nr, nc) AND grid[nr][nc] == 1:
                    grid[nr][nc] = 2       // rot it
                    freshCount -= 1
                    queue.enqueue((nr, nc))
        minutes += 1
    
    RETURN (freshCount == 0) ? minutes - 1 : -1
```

## Trace

```
    Initial:           t=1:             t=2:             t=3:
    ┌───┬───┬───┐     ┌───┬───┬───┐   ┌───┬───┬───┐   ┌───┬───┬───┐
    │ 2 │ 1 │ 1 │     │ 2 │ 2 │ 1 │   │ 2 │ 2 │ 2 │   │ 2 │ 2 │ 2 │
    ├───┼───┼───┤     ├───┼───┼───┤   ├───┼───┼───┤   ├───┼───┼───┤
    │ 1 │ 1 │ 0 │     │ 2 │ 1 │ 0 │   │ 2 │ 2 │ 0 │   │ 2 │ 2 │ 0 │
    ├───┼───┼───┤     ├───┼───┼───┤   ├───┼───┼───┤   ├───┼───┼───┤
    │ 0 │ 1 │ 1 │     │ 0 │ 1 │ 1 │   │ 0 │ 1 │ 1 │   │ 0 │ 2 │ 2 │
    └───┴───┴───┘     └───┴───┴───┘   └───┴───┴───┘   └───┴───┴───┘
                                                         t=4:
                                                        ┌───┬───┬───┐
                                                        │ 2 │ 2 │ 2 │
                                                        ├───┼───┼───┤
                                                        │ 2 │ 2 │ 0 │
                                                        ├───┼───┼───┤
                                                        │ 0 │ 2 │ 2 │
                                                        └───┴───┴───┘
    Answer: 4 minutes
```

---

[← Previous: Minimum Moves Problems](04-minimum-moves-problems.md) | [Back to TOC](../README.md) | [Next: 0-1 BFS →](06-zero-one-bfs.md)
