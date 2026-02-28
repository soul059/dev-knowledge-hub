# Chapter 1: Heap Fundamentals

## 📋 Chapter Overview
Heap (priority queue) structure, operations, heapify, and core properties for interview problems.

---

## 🧠 What is a Heap?

```
  Complete binary tree with heap property:
  
  MAX HEAP: parent ≥ children     MIN HEAP: parent ≤ children
  
       90                              10
      /  \                            /  \
    80    70                        20    30
   / \   /                         / \   /
  50 60 65                       40  50 35
  
  Root = maximum                  Root = minimum
```

### Array Representation

```
  For node at index i (0-based):
    Parent:      (i - 1) / 2
    Left child:  2*i + 1
    Right child: 2*i + 2
  
  Max heap array: [90, 80, 70, 50, 60, 65]
  Index:           0   1   2   3   4   5
  
       90(0)
      /     \
   80(1)   70(2)
   /  \    /
 50(3) 60(4) 65(5)
```

---

## 📐 Core Operations

### 1. Insert (Push) — O(log n)

```
function heapPush(heap, val):
    heap.append(val)        // add at end
    siftUp(heap, len(heap) - 1)

function siftUp(heap, i):
    while i > 0:
        parent = (i - 1) / 2
        if heap[i] > heap[parent]:    // max heap
            swap(heap[i], heap[parent])
            i = parent
        else:
            break
```

### 2. Extract Max/Min (Pop) — O(log n)

```
function heapPop(heap):
    result = heap[0]             // root (max or min)
    heap[0] = heap[last]         // move last to root
    heap.removeLast()
    siftDown(heap, 0)
    return result

function siftDown(heap, i):
    while 2*i + 1 < len(heap):
        child = 2*i + 1         // left child
        if child+1 < len(heap) and heap[child+1] > heap[child]:
            child = child + 1   // right child is larger
        if heap[i] < heap[child]:
            swap(heap[i], heap[child])
            i = child
        else:
            break
```

### 3. Heapify — Build Heap in O(n)

```
function buildHeap(arr):
    // Start from last non-leaf node
    for i = len(arr)/2 - 1 down to 0:
        siftDown(arr, i)
    
    // O(n) NOT O(n log n) — mathematical proof:
    // Most nodes near bottom need few swaps
    // Sum: n/4*1 + n/8*2 + n/16*3 + ... = O(n)
```

---

## 📝 Heap Sort

```
function heapSort(arr):
    buildHeap(arr)           // O(n)
    
    for i = n-1 down to 1:
        swap(arr[0], arr[i])  // move max to end
        siftDown(arr, 0, size=i)  // restore heap on remaining
    
    // Array is now sorted ascending
```

### Trace

```
  arr = [4, 10, 3, 5, 1]
  
  buildHeap: [10, 5, 3, 4, 1]
  
  Swap 10↔1: [1, 5, 3, 4, | 10]  siftDown → [5, 4, 3, 1, | 10]
  Swap 5↔1:  [1, 4, 3, | 5, 10]  siftDown → [4, 1, 3, | 5, 10]
  Swap 4↔3:  [3, 1, | 4, 5, 10]  siftDown → [3, 1, | 4, 5, 10]
  Swap 3↔1:  [1, | 3, 4, 5, 10]
  
  Sorted: [1, 3, 4, 5, 10]
```

**Complexity:** O(n log n) time, O(1) space (in-place)

---

## 📊 Operations Complexity

| Operation | Time | Notes |
|-----------|------|-------|
| Insert (push) | O(log n) | Sift up |
| Extract (pop) | O(log n) | Sift down |
| Peek (top) | O(1) | Return root |
| Build heap | O(n) | Bottom-up heapify |
| Heap sort | O(n log n) | n extractions |
| Search | O(n) | Not designed for search |

---

## 🆚 Min Heap vs Max Heap

| Property | Min Heap | Max Heap |
|----------|----------|----------|
| Root | Smallest element | Largest element |
| Use case | Kth largest, merge sorted | Kth smallest, max priority |
| Trick | Negate values to simulate one with other | |

```
  In languages with only min-heap (Python heapq):
  Max heap → push -value, pop and negate result
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Heap | Complete binary tree with parent-child ordering |
| Array storage | parent = (i-1)/2, children = 2i+1, 2i+2 |
| Insert/Extract | O(log n) via sift up/down |
| Build heap | O(n) — bottom-up heapify |
| Heap sort | O(n log n) in-place, not stable |

---

## ❓ Revision Questions

1. **Heap vs BST?** → Heap: O(1) min/max access, O(log n) insert/extract. BST: O(log n) search/insert/delete, ordered traversal.
2. **Why is buildHeap O(n) not O(n log n)?** → Most nodes are near the leaves and need O(1) swaps; the sum converges to O(n).
3. **Min heap to max heap trick?** → Insert negated values; negate when extracting.
4. **Is heap sort stable?** → No — equal elements may be reordered during swaps.
5. **Parent/child index formulas?** → Parent: (i-1)/2. Left: 2i+1. Right: 2i+2.

---

[← Previous: Stack/Queue — When to Apply](../09-Stack-Queue-Pattern/05-when-to-apply.md) | [Next: Top-K Problems →](02-top-k-problems.md)
