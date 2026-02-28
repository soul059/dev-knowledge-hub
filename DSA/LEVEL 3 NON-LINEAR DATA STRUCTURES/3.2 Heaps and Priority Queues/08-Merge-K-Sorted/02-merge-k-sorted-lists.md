# 8.2 Merge K Sorted Lists

[← Previous: Core Problem](01-core-problem.md) | [Next: Merge K Sorted Arrays →](03-merge-k-sorted-arrays.md)

---

## Problem

Given K sorted linked lists, merge them into one sorted linked list.

---

## Visual Walkthrough

```
Input:
  List 1: 1 → 4 → 7
  List 2: 2 → 5 → 8
  List 3: 3 → 6 → 9

═══ Initialize ═══
Heap: [1, 2, 3]   (one head from each list)
Output: []

  Min-heap:     1
               / \
              2   3

═══ Step 1: Extract 1 (from List 1) ═══
Output: [1]
Next from List 1: 4 → insert
Heap: [2, 4, 3]

═══ Step 2: Extract 2 (from List 2) ═══
Output: [1, 2]
Next from List 2: 5 → insert
Heap: [3, 4, 5]

═══ Step 3: Extract 3 (from List 3) ═══
Output: [1, 2, 3]
Next from List 3: 6 → insert
Heap: [4, 6, 5]

═══ Step 4: Extract 4 (from List 1) ═══
Output: [1, 2, 3, 4]
Next from List 1: 7 → insert
Heap: [5, 6, 7]

═══ Steps 5-9: Continue extracting... ═══
Output: [1, 2, 3, 4, 5, 6, 7, 8, 9] ✅
```

---

## Pseudocode

```
MERGE_K_SORTED_LISTS(lists):
    min_heap = new MinHeap()
    
    // Initialize — add head of each non-empty list
    FOR i = 0 TO K-1:
        IF lists[i] is not empty:
            min_heap.insert((lists[i].value, i, lists[i]))
    
    dummy = new ListNode(0)
    current = dummy
    
    WHILE min_heap is not empty:
        (value, list_idx, node) = min_heap.extract_min()
        
        current.next = node
        current = current.next
        
        IF node.next is not null:
            min_heap.insert((node.next.value, list_idx, node.next))
    
    RETURN dummy.next
```

---

## Python Implementation

```python
import heapq

def mergeKLists(lists):
    heap = []
    
    # Initialize with head of each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst.val, i, lst))
    
    dummy = ListNode(0)
    curr = dummy
    
    while heap:
        val, i, node = heapq.heappop(heap)
        curr.next = node
        curr = curr.next
        
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    
    return dummy.next
```

> **Note**: The `i` (list index) serves as a tie-breaker when values are equal, avoiding comparison of `ListNode` objects.

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(N log K) where N = total elements across all lists |
| Space | O(K) for the heap |

---

[← Previous: Core Problem](01-core-problem.md) | [Next: Merge K Sorted Arrays →](03-merge-k-sorted-arrays.md)
