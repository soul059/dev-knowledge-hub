# Course Schedule Problems

[← Previous: DFS vs Kahn's Comparison](04-dfs-vs-kahns-comparison.md) | [Back to TOC](../README.md) | [Next: Build Order and Advanced →](06-build-order-and-advanced.md)

---

## Problem 1: Can Finish? (Course Schedule I)

```
    Given numCourses and prerequisites list where
    prerequisites[i] = [a, b] means "to take course a,
    you must first take course b" (b → a),
    determine if you can finish all courses.
    
    Answer: YES ⟺ NO CYCLE in the prerequisite graph!

FUNCTION canFinish(numCourses, prerequisites):
    // Build graph
    graph = adjacency list from prerequisites
    
    // Run Kahn's algorithm
    result = topologicalSort_Kahn(graph, numCourses)
    
    RETURN result.size() == numCourses
    // If Kahn's processes all courses → no cycle → can finish!
```

## Problem 2: Find Order (Course Schedule II)

```
    Same setup, but now RETURN the order to take courses.
    
    Answer: Return the topological order itself!
    (If cycle exists, return empty array.)

FUNCTION findOrder(numCourses, prerequisites):
    // Build graph and run Kahn's
    result = topologicalSort_Kahn(graph, numCourses)
    
    IF result.size() != numCourses:
        RETURN []                      // cycle → impossible
    
    RETURN result                      // valid course order
```

## Example Trace

```
    numCourses = 4
    prerequisites = [[1,0], [2,0], [3,1], [3,2]]
    
    Graph (b → a):
    0 → [1, 2]
    1 → [3]
    2 → [3]
    3 → []
    
    In-degrees: 0:0, 1:1, 2:1, 3:2
    
    Kahn's: queue = [0]
    Process 0 → result = [0], decrease 1→0, 2→0
    queue = [1, 2]
    Process 1 → result = [0,1], decrease 3→1
    Process 2 → result = [0,1,2], decrease 3→0
    queue = [3]
    Process 3 → result = [0,1,2,3]
    
    Can finish? YES (4 courses processed)
    Order: [0, 1, 2, 3]
```

---

[← Previous: DFS vs Kahn's Comparison](04-dfs-vs-kahns-comparison.md) | [Back to TOC](../README.md) | [Next: Build Order and Advanced →](06-build-order-and-advanced.md)
