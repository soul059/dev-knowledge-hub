# Stack Operations Complexity Analysis

## Overview

Understanding the time and space complexity of stack operations is crucial for making informed decisions about which implementation to use and predicting application performance. This chapter provides comprehensive complexity analysis for all stack operations across different implementations.

---

## Summary Table

| Operation | Array | Linked List | Dynamic Array (Worst) | Dynamic Array (Amortized) |
|-----------|-------|-------------|----------------------|---------------------------|
| **Push** | O(1) | O(1) | O(n) | O(1) |
| **Pop** | O(1) | O(1) | O(n) | O(1) |
| **Peek** | O(1) | O(1) | O(1) | O(1) |
| **isEmpty** | O(1) | O(1) | O(1) | O(1) |
| **isFull** | O(1) | N/A | O(1) | O(1) |
| **Size** | O(1) | O(1) or O(n)* | O(1) | O(1) |
| **Space** | O(n) | O(n) | O(n) | O(n) |

*O(1) if size counter maintained, O(n) if must count nodes

---

## Time Complexity Details

### Push Operation

```
Array Stack:
PUSH(item):
    top = top + 1        // O(1)
    array[top] = item    // O(1)
    Total: O(1) ✓

Linked List Stack:
PUSH(item):
    newNode = new Node(item)  // O(1)
    newNode.next = top        // O(1)
    top = newNode             // O(1)
    Total: O(1) ✓

Dynamic Array Stack:
PUSH(item):
    IF full THEN
        resize()         // O(n) - copy all elements
    ENDIF
    array[++top] = item  // O(1)
    
    Worst Case: O(n)
    Amortized: O(1) ✓
```

### Space Complexity

```
Array Stack:
• Array: n elements × element_size
• Metadata: top, capacity (constant)
• Total: O(n) + O(1) = O(n)
• Overhead: Minimal

Linked List Stack:
• Nodes: n × (element_size + pointer_size)
• Metadata: top, size (constant)
• Total: O(n) + O(1) = O(n)
• Overhead: ~2-3× array due to pointers

Dynamic Array Stack:
• Array: capacity × element_size
• capacity ≥ size (may have unused space)
• Metadata: top, size, capacity
• Total: O(n) + O(1) = O(n)
• Overhead: Up to 50% unused capacity
```

---

## Amortized Analysis Deep Dive

### Push in Dynamic Array

```
Sequence of n pushes starting from empty:

Pushes 1,2:     Cost 2 (no resize)
Push 3:         Cost 3 (resize from 2 to 4: copy 2 + insert 1)
Push 4:         Cost 1
Push 5:         Cost 5 (resize from 4 to 8: copy 4 + insert 1)
Pushes 6,7,8:   Cost 3
Push 9:         Cost 9 (resize from 8 to 16: copy 8 + insert 1)
...

Total cost = n + (2 + 4 + 8 + 16 + ... + n/2)
           = n + (n - 2)
           ≈ 2n
           = O(n)

Average per push = O(n) / n = O(1) ✓
```

---

## Quick Revision Questions

1. **Q: Why is dynamic array push O(1) amortized but O(n) worst case?**
   - A: Occasional resize operations take O(n), but they're infrequent enough that the average over many operations is O(1).

2. **Q: Which implementation has the best space efficiency?**
   - A: Array stack (when size known) - no pointer overhead or unused capacity.

3. **Q: In linked list stack, why might Size() be O(n)?**
   - A: If no size counter is maintained, must traverse entire list to count nodes.

4. **Q: What operation is O(1) in all implementations?**
   - A: Peek/Top - always simple pointer/index access to top element.

5. **Q: Which implementation guarantees O(1) worst-case for all operations?**
   - A: Array stack (if pre-sized) and Linked list stack - no resizing needed.

---

## Navigation

- **[← Previous: Comparison of Approaches](04-comparison-approaches.md)**
- **[Next: Implementation Trade-offs →](06-implementation-tradeoffs.md)**
- **[↑ Back to Unit 2](README.md)**
- **[⌂ Home](../README.md)**

---

*Understanding complexity helps you choose the right implementation for your performance requirements!*
