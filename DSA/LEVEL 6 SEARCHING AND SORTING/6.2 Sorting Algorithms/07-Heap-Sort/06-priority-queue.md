# Unit 7: Heap Sort — Priority Queue Connection

[← Previous: Applications](05-heap-sort-applications.md) | [Next: Unit 8 - Counting Sort →](../08-Counting-Sort/01-algorithm-description.md)

---

## Overview

A **Priority Queue** is the abstract data structure that a binary heap implements. Understanding this connection completes your understanding of heaps. Priority queues power everything from CPU scheduling to Dijkstra's algorithm.

---

## What Is a Priority Queue?

```
  A Priority Queue supports:
  
  ┌────────────────────────────────────────────────────────┐
  │  INSERT(element, priority)  — Add element              │
  │  EXTRACT-MAX()              — Remove highest priority  │
  │  PEEK-MAX()                 — View highest priority    │
  │  INCREASE-KEY(element, p)   — Raise priority           │
  └────────────────────────────────────────────────────────┘
  
  It's NOT a queue (FIFO). Elements leave by PRIORITY order.
  
  Regular Queue:     A → B → C → D     (FIFO)
  Priority Queue:    D(p=10) leaves first, regardless of
                     insertion order, because it has
                     highest priority
```

---

## Heap as Priority Queue Implementation

```
  ┌─────────────────────────────────────────────────────────┐
  │  Operation        │ Array │ Sorted │ BST      │ Heap   │
  │                   │       │ Array  │          │        │
  ├───────────────────┼───────┼────────┼──────────┼────────┤
  │  Insert           │ O(1)  │ O(n)   │ O(log n) │ O(log n)│
  │  Extract-Max      │ O(n)  │ O(1)   │ O(log n) │ O(log n)│
  │  Peek-Max         │ O(n)  │ O(1)   │ O(log n) │ O(1)   │
  │  Build from array │ O(1)  │ O(n lg)│ O(n lg)  │ O(n)   │
  └───────────────────┴───────┴────────┴──────────┴────────┘
  
  Binary heap gives the BEST overall balance!
```

---

## Insert (Sift-Up)

```
  Insert 15 into max-heap:
  
  Step 1: Add to end (next available position)
  
         10                    10
        /  \                  /  \
       8    9     →          8    9
      / \                   / \  /
     4   7                 4  7 15 ← new
  
  Step 2: Sift-Up — compare with parent, swap if larger
  
  15 > parent(9)? YES → swap
  
         10                   
        /  \                 
       8    15   ← swapped          
      / \  /                 
     4  7 9                  

  15 > parent(10)? YES → swap
  
         15      ← swapped              
        /  \                 
       8    10              
      / \  /                 
     4  7 9                  
  
  Done! 15 is at root. Heap property restored.
```

### Code

```java
// Insert into max-heap
public void insert(int val) {
    heap[++size] = val;  // Add at end
    siftUp(size);        // Restore heap property
}

private void siftUp(int i) {
    while (i > 0) {
        int parent = (i - 1) / 2;
        if (heap[i] > heap[parent]) {
            swap(i, parent);
            i = parent;
        } else {
            break;  // Heap property satisfied
        }
    }
}
```

---

## Extract-Max (Sift-Down)

```
  Remove maximum (root) from max-heap:
  
  Step 1: Replace root with last element
  
         15                    9
        /  \                  / \
       8    10    →          8  10
      / \  /                / \
     4  7 9                4   7
  
  Step 2: Sift-Down root
  
  9 < max-child(10)? YES → swap with 10
  
         10
        /  \
       8    9
      / \
     4   7
  
  9 has no children violating → Done!
  
  Returned: 15 (the maximum)
```

### Code

```java
public int extractMax() {
    int max = heap[0];        // Store root
    heap[0] = heap[size--];   // Move last to root
    siftDown(0);              // Restore heap property
    return max;
}

private void siftDown(int i) {
    while (2 * i + 1 <= size) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        
        if (left <= size && heap[left] > heap[largest])
            largest = left;
        if (right <= size && heap[right] > heap[largest])
            largest = right;
        
        if (largest == i) break;
        
        swap(i, largest);
        i = largest;
    }
}
```

---

## Complete Priority Queue Implementation

```java
public class MaxPriorityQueue {
    private int[] heap;
    private int size;
    private int capacity;
    
    public MaxPriorityQueue(int capacity) {
        this.capacity = capacity;
        this.heap = new int[capacity];
        this.size = -1;
    }
    
    // Insert: O(log n)
    public void insert(int val) {
        if (size >= capacity - 1)
            throw new RuntimeException("Heap overflow");
        heap[++size] = val;
        siftUp(size);
    }
    
    // Extract max: O(log n)
    public int extractMax() {
        if (size < 0)
            throw new RuntimeException("Heap underflow");
        int max = heap[0];
        heap[0] = heap[size--];
        if (size >= 0) siftDown(0);
        return max;
    }
    
    // Peek max: O(1)
    public int peekMax() {
        if (size < 0)
            throw new RuntimeException("Heap empty");
        return heap[0];
    }
    
    // Increase key: O(log n)
    public void increaseKey(int i, int newVal) {
        if (newVal < heap[i])
            throw new RuntimeException("New value is smaller");
        heap[i] = newVal;
        siftUp(i);
    }
    
    // Delete arbitrary element: O(log n)
    public void delete(int i) {
        increaseKey(i, Integer.MAX_VALUE);  // Move to root
        extractMax();                        // Remove root
    }
    
    public boolean isEmpty() { return size < 0; }
    public int size() { return size + 1; }
    
    // --- Helpers ---
    
    private void siftUp(int i) {
        while (i > 0 && heap[i] > heap[(i - 1) / 2]) {
            swap(i, (i - 1) / 2);
            i = (i - 1) / 2;
        }
    }
    
    private void siftDown(int i) {
        while (true) {
            int largest = i;
            int l = 2 * i + 1, r = 2 * i + 2;
            if (l <= size && heap[l] > heap[largest]) largest = l;
            if (r <= size && heap[r] > heap[largest]) largest = r;
            if (largest == i) break;
            swap(i, largest);
            i = largest;
        }
    }
    
    private void swap(int a, int b) {
        int temp = heap[a];
        heap[a] = heap[b];
        heap[b] = temp;
    }
}
```

---

## Java's Built-in PriorityQueue

```java
import java.util.PriorityQueue;
import java.util.Collections;

// Min-heap (default in Java)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);        // insert
minHeap.offer(2);
minHeap.offer(8);
minHeap.poll();          // extract-min → returns 2
minHeap.peek();          // peek-min → returns 5

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(2);
maxHeap.offer(8);
maxHeap.poll();          // returns 8 (max)

// Custom objects
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
pq.offer(new int[]{3, 100});
pq.offer(new int[]{1, 200});
pq.poll();  // returns [1, 200] (smallest first element)
```

---

## From Priority Queue to Heap Sort

```
  Heap Sort is literally:
  
  1. Build a priority queue from the array (build-heap)
  2. Extract-max n times into sorted order
  
  ┌──────────────────────────────────────────────────────────┐
  │  function heapSort(arr):                                │
  │      pq = new MaxPriorityQueue(arr)  // O(n) build     │
  │      for i = n-1 down to 0:                             │
  │          arr[i] = pq.extractMax()    // O(log n) each  │
  │                                                          │
  │  The clever part: we do this IN-PLACE by using the      │
  │  array itself as the heap, and growing the sorted       │
  │  section from the end.                                  │
  └──────────────────────────────────────────────────────────┘
```

---

## Real-World Applications of Priority Queues

```
  ┌────────────────────────────────────────────────────────────┐
  │  1. CPU Scheduling                                        │
  │     Higher-priority processes run first                   │
  │                                                            │
  │  2. Dijkstra's Shortest Path                              │
  │     Always process the closest unvisited vertex           │
  │                                                            │
  │  3. Prim's Minimum Spanning Tree                          │
  │     Always add the cheapest edge                          │
  │                                                            │
  │  4. Huffman Coding (Data Compression)                     │
  │     Build tree by merging two lowest-frequency nodes      │
  │                                                            │
  │  5. A* Search Algorithm                                   │
  │     Process nodes with lowest estimated total cost        │
  │                                                            │
  │  6. Event-Driven Simulation                               │
  │     Process events in chronological order                 │
  │                                                            │
  │  7. Load Balancing                                        │
  │     Assign tasks to least-loaded server                   │
  └────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Operation | Time | Description |
|---|---|---|
| Insert (sift-up) | O(log n) | Add element, bubble up |
| Extract-max (sift-down) | O(log n) | Remove root, sink replacement |
| Peek-max | O(1) | Return root |
| Increase-key (sift-up) | O(log n) | Update value, bubble up |
| Delete arbitrary | O(log n) | Increase to ∞, extract |
| Build from array | O(n) | Bottom-up heapify |

---

## Quick Revision Questions

1. **What's the difference between a priority queue and a regular queue?**
2. **Why is a binary heap the most common implementation of a priority queue?**
3. **How does insert (sift-up) differ from heapify (sift-down)?**
4. **How does Java's PriorityQueue create a max-heap?**
5. **Name 3 graph algorithms that rely on priority queues.**
6. **How is Heap Sort just "build priority queue + extract all"?**

---

[← Previous: Applications](05-heap-sort-applications.md) | [Next: Unit 8 - Counting Sort →](../08-Counting-Sort/01-algorithm-description.md)
