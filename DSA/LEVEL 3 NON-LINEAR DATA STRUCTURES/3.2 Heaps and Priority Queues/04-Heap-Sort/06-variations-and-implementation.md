# 4.6 Variations & Implementation

[← Previous: Comparison & Stability](05-comparison-and-stability.md) | [Next Unit: Priority Queue →](../05-Priority-Queue/01-priority-queue-adt.md)

---

## Overview

Partial heap sort for K largest, optimization variations, and complete implementations in Python and Java.

---

## Partial Heap Sort — Finding K Largest Elements

A powerful variation: don't sort everything, just find the **K largest**.

```
PARTIAL_HEAP_SORT(array, K):
    BUILD_MAX_HEAP(array)           // O(n)
    result = []
    FOR i = 1 TO K:
        result.append(EXTRACT_MAX(array))  // O(log n) each
    RETURN result
```

**Total Time**: O(n + K log n)

| K Value | Time | Better Than Full Sort? |
|---------|------|----------------------|
| K = 1 | O(n) | Yes — same as finding max |
| K = O(log n) | O(n) | Yes! |
| K = O(√n) | O(n) | Yes! |
| K = O(n) | O(n log n) | Same — full sort |

---

## Heap Sort Variations

### Bottom-Up Heap Sort

An optimization that reduces comparisons:

1. Instead of comparing parent with both children going down...
2. Find the path of maximum children (only compare children with each other)
3. Then sift the element up along that path

This reduces comparisons from ~2n log n to ~n log n, making it competitive with Quick Sort.

### Ternary Heap Sort

Use a 3-ary heap instead of binary:
- Fewer levels (log₃ n vs log₂ n)
- But more comparisons per level (3 children instead of 2)
- Sometimes better cache performance

---

## Python Implementation

```python
def heap_sort(arr):
    n = len(arr)
    
    # Phase 1: Build max-heap
    for i in range(n // 2 - 1, -1, -1):
        sift_down(arr, i, n)
    
    # Phase 2: Extract elements
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]  # Swap max to end
        sift_down(arr, 0, i)             # Fix heap of size i

def sift_down(arr, i, size):
    while True:
        largest = i
        left = 2 * i + 1
        right = 2 * i + 2
        
        if left < size and arr[left] > arr[largest]:
            largest = left
        if right < size and arr[right] > arr[largest]:
            largest = right
        
        if largest != i:
            arr[i], arr[largest] = arr[largest], arr[i]
            i = largest
        else:
            break
```

---

## Java Implementation

```java
void heapSort(int[] arr) {
    int n = arr.length;
    
    // Build max-heap
    for (int i = n / 2 - 1; i >= 0; i--)
        siftDown(arr, i, n);
    
    // Extract
    for (int i = n - 1; i > 0; i--) {
        swap(arr, 0, i);
        siftDown(arr, 0, i);
    }
}

void siftDown(int[] arr, int i, int size) {
    while (true) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        
        if (left < size && arr[left] > arr[largest]) largest = left;
        if (right < size && arr[right] > arr[largest]) largest = right;
        
        if (largest != i) {
            int temp = arr[i]; arr[i] = arr[largest]; arr[largest] = temp;
            i = largest;
        } else break;
    }
}
```

---

## Real-World Applications

| Application | Why Heap Sort? |
|-------------|---------------|
| **Embedded systems** | O(1) space — cannot afford O(n) extra memory |
| **Real-time systems** | O(n log n) worst-case guarantee — no surprises |
| **Partial sorting** | K largest/smallest in O(n + K log n) |
| **Linux kernel** | Used in some kernel sort implementations for predictability |
| **External sorting** | Heap-based merge phase in external merge sort |

---

## Unit 4 Summary Table

| Concept | Key Point |
|---------|-----------|
| Algorithm | Build max-heap → repeatedly extract max |
| Build Phase | O(n) using Floyd's bottom-up method |
| Extract Phase | n-1 swap + sift-down operations |
| Time (all cases) | O(n log n) — guaranteed |
| Space | O(1) — fully in-place |
| Stable? | No — equal elements may reorder |
| vs Quick Sort | Worse in practice (cache), better worst case |
| vs Merge Sort | Less space, but not stable |
| Partial sort | O(n + K log n) for K largest elements |

---

## Quick Check

1. **If you only need the 5 largest elements from an array of 1 million, what's the time complexity using a heap?**
   <details><summary>Answer</summary>
   O(n + K log n) = O(1,000,000 + 5 × 20) ≈ O(n). Build the max-heap in O(n), then extract 5 times in O(5 log n). Much faster than fully sorting.
   </details>

---

[← Previous: Comparison & Stability](05-comparison-and-stability.md) | [Next Unit: Priority Queue →](../05-Priority-Queue/01-priority-queue-adt.md)
