# 10.1 D-ary Heap

[← Previous: Problem-Solving Framework](../09-Advanced-Heap-Problems/06-problem-solving-framework.md) | [Next: Fibonacci Heap →](02-fibonacci-heap.md)

---

## What Is It?

A **D-ary heap** generalizes the binary heap by allowing each node to have **D children** instead of 2.

```
Binary Heap (D=2):           Ternary Heap (D=3):        4-ary Heap (D=4):

       10                          10                        10
      /  \                      /  |  \                   / | | \
    20    30                  20   30  40               20 30 40 50
   / \   / \                / | \
  40 50 60  70             50 60 70
```

---

## Array Representation (0-indexed)

For node at index `i` in a D-ary heap:

| Relationship | Formula |
|-------------|---------|
| **Parent** | `(i - 1) / D` (integer division) |
| **j-th child** (j = 0..D-1) | `D * i + j + 1` |
| **First child** | `D * i + 1` |
| **Last child** | `D * i + D` |

### Example: 3-ary Heap Array

```
Tree:
           5
        /  |  \
       8   12  10
      /|\  |
    15 20 25 18

Array: [5, 8, 12, 10, 15, 20, 25, 18]
Index:  0  1   2   3   4   5   6   7

Verify for i=1 (value 8), D=3:
  Parent: (1-1)/3 = 0 ✅ (value 5)
  Children: 3*1+1=4, 3*1+2=5, 3*1+3=6 → indices 4,5,6 (values 15,20,25) ✅
```

---

## Sift Down in D-ary Heap

```
SIFT_DOWN_DARY(heap, i, D):
    n = heap.size
    WHILE TRUE:
        smallest = i
        // Check all D children
        FOR j = 1 TO D:
            child = D * i + j
            IF child < n AND heap[child] < heap[smallest]:
                smallest = child
        
        IF smallest != i:
            SWAP(heap[i], heap[smallest])
            i = smallest
        ELSE:
            BREAK
```

---

## Complexity Comparison

| Operation | Binary (D=2) | D-ary |
|-----------|-------------|-------|
| **Sift Up** | O(log₂ n) | O(log_D n) = **faster** |
| **Sift Down** | O(2 · log₂ n) comparisons | O(D · log_D n) comparisons |
| **Insert** | O(log₂ n) | O(log_D n) — **better** |
| **Extract Min** | O(log₂ n) | O(D · log_D n) — more comparisons per level |
| **Decrease Key** | O(log₂ n) | O(log_D n) — **better** |

---

## When to Use D-ary?

| Scenario | Best D |
|----------|--------|
| Insert-heavy (Dijkstra) | D = 4 (good balance) |
| Extract-heavy | D = 2 (binary) |
| Cache optimization | D = 4 or 8 (fits cache line) |
| Equal insert/extract | D = 3 or 4 |

---

## Key Insight

> **Increasing D reduces tree height** (fewer levels to traverse during sift-up) but **increases work per level** (more children to compare during sift-down). D=4 is often optimal in practice due to cache effects.

---

[← Previous: Problem-Solving Framework](../09-Advanced-Heap-Problems/06-problem-solving-framework.md) | [Next: Fibonacci Heap →](02-fibonacci-heap.md)
