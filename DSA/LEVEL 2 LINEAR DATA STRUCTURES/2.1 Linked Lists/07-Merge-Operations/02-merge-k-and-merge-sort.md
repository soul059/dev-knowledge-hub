# Chapter 2: Merge K Sorted Lists and Merge Sort

[← Previous: Merge Two Sorted Lists](01-merge-two-sorted-lists.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Intersection and Union →](03-intersection-and-union.md)

---

## Chapter Overview

Scaling from merging 2 lists to **K lists**, and applying merge to **sort** a linked list. These are critical interview topics combining divide-and-conquer with linked list manipulation.

---

## 2.1 Merge K Sorted Lists — Problem

Given K sorted linked lists, merge them into one sorted list.

```
  List 1:  1 → 4 → 7
  List 2:  2 → 5 → 8
  List 3:  3 → 6 → 9
  
  Merged:  1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9
```

If total number of elements across all lists is **N**, we want better than O(NK).

---

## 2.2 Approach 1: Sequential Merge — O(NK)

Merge one list at a time into a growing result.

```
function mergeKSequential(lists[], k):
    result = lists[0]
    for i = 1 to k-1:
        result = mergeTwoSorted(result, lists[i])
    return result
```

```
  Problem: Each merge grows the result.
  
  Merge 1+2: compare 2n elements → 2n
  Merge (1+2)+3: compare 3n → 3n
  Merge (1+2+3)+4: compare 4n → 4n
  ...
  Total: 2n + 3n + 4n + ... + kn = O(k²n) = O(NK) ← SLOW
```

---

## 2.3 Approach 2: Divide and Conquer — O(N log K) ⭐

Merge lists in **pairs**, like a tournament bracket.

```
function mergeKDivideConquer(lists[], k):
    if k == 0: return NULL
    if k == 1: return lists[0]
    
    interval = 1
    while interval < k:
        for i = 0 to k-1 step 2*interval:
            if i + interval < k:
                lists[i] = mergeTwoSorted(lists[i], lists[i + interval])
        interval *= 2
    
    return lists[0]
```

### Visual — Merging 4 Lists

```
  Round 1: Merge pairs
  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
  │ List 1  │  │ List 2  │  │ List 3  │  │ List 4  │
  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘
       └──────┬─────┘            └──────┬─────┘
         ┌────┴────┐              ┌────┴────┐
         │ Merge12 │              │ Merge34 │
         └────┬────┘              └────┬────┘

  Round 2: Merge results
              └──────────┬─────────┘
                   ┌─────┴─────┐
                   │  Final    │
                   └───────────┘

  log₂(4) = 2 rounds
  Each round processes all N elements once → O(N) per round
  Total: O(N log K)
```

### Why O(N log K)?

```
  K lists, N total elements.
  
  Number of merge rounds: log₂(K)
  Work per round: O(N)  (every element compared once)
  
  Total: O(N × log K)
  
  Compare:
  Sequential:       O(N × K)
  Divide & Conquer:  O(N × log K) ← Much better!
  
  K=1000, N=10⁶:
  Sequential:       10⁹ operations
  Divide & Conquer:  10⁷ operations ← 100x faster
```

---

## 2.4 Approach 3: Min-Heap (Priority Queue) — O(N log K) ⭐

Use a min-heap to always pick the smallest among K candidates.

```
function mergeKHeap(lists[], k):
    minHeap = new MinHeap()
    
    // Step 1: Insert first node of each list
    for i = 0 to k-1:
        if lists[i] ≠ NULL:
            minHeap.insert(lists[i])
    
    dummy = createNode(0)
    tail = dummy
    
    // Step 2: Extract min, add its next to heap
    while minHeap is not empty:
        smallest = minHeap.extractMin()
        tail.next = smallest
        tail = tail.next
        
        if smallest.next ≠ NULL:
            minHeap.insert(smallest.next)
    
    return dummy.next
```

### Step-by-Step Trace

```
  Lists:  A: 1→4→7    B: 2→5    C: 3→6

  Heap initially: {1, 2, 3}
  
  Extract 1 → result: [1]     Heap: {2, 3, 4} (insert 4 from A)
  Extract 2 → result: [1,2]   Heap: {3, 4, 5} (insert 5 from B)
  Extract 3 → result: [1,2,3] Heap: {4, 5, 6} (insert 6 from C)
  Extract 4 → result: [...,4] Heap: {5, 6, 7} (insert 7 from A)
  Extract 5 → result: [...,5] Heap: {6, 7}    (B exhausted)
  Extract 6 → result: [...,6] Heap: {7}       (C exhausted)
  Extract 7 → result: [...,7] Heap: {}        (A exhausted)
  
  Final: 1 → 2 → 3 → 4 → 5 → 6 → 7 ✓
```

### Why O(N log K)?

```
  Heap size ≤ K (one node per list)
  Insert/Extract: O(log K) each
  Total operations: N inserts + N extracts = 2N
  Total: O(N log K)
```

---

## 2.5 Comparison of All Approaches

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| Sequential merge | O(NK) | O(1) | Simple | Very slow for large K |
| Divide & Conquer | O(N log K) | O(log K) stack | Fast, low space | Slightly complex |
| Min-Heap | O(N log K) | O(K) heap | Intuitive, online-friendly | Needs heap structure |

---

## 2.6 Merge Sort for Linked Lists

Merge sort is the **ideal sorting algorithm** for linked lists because:
- No random access needed (unlike quicksort's partitioning)
- O(1) extra space (unlike array merge sort which needs O(n))
- Stable sort

### Algorithm

```
function mergeSort(HEAD):
    // Base case: 0 or 1 node
    if HEAD == NULL or HEAD.next == NULL:
        return HEAD
    
    // Step 1: Find the middle
    middle = getMiddle(HEAD)
    secondHalf = middle.next
    middle.next = NULL              ← Split into two halves
    
    // Step 2: Recursively sort both halves
    left = mergeSort(HEAD)
    right = mergeSort(secondHalf)
    
    // Step 3: Merge sorted halves
    return mergeTwoSorted(left, right)

function getMiddle(HEAD):
    slow = HEAD
    fast = HEAD.next                ← Start fast one step ahead
    while fast ≠ NULL and fast.next ≠ NULL:
        slow = slow.next
        fast = fast.next.next
    return slow                     ← Returns FIRST middle for even
```

### Why `fast = HEAD.next` Instead of `fast = HEAD`?

```
  For a 2-node list [1, 2]:
  
  fast = HEAD:
    slow=1, fast=1
    fast.next=2, fast.next.next=NULL → stop
    middle = 1
    Split: [1] and [2] → But middle.next=2, so secondHalf=2
    middle.next=NULL → [1] and [2] ✓
    
  fast = HEAD.next:
    slow=1, fast=2
    fast.next=NULL → stop immediately
    middle = 1 ✓
    
  Both work, but starting fast = HEAD.next consistently returns
  the first middle for even-length lists, giving balanced splits.
```

### Visual Trace: Sort [4, 2, 1, 3]

```
  mergeSort(4→2→1→3)
  │  middle(4→2→1→3) = Node(2)
  │  Split: [4→2] and [1→3]
  │
  ├─ mergeSort(4→2)
  │  │  middle(4→2) = Node(4)
  │  │  Split: [4] and [2]
  │  │
  │  ├─ mergeSort(4) → return 4 (base case)
  │  ├─ mergeSort(2) → return 2 (base case)
  │  │
  │  └─ merge(4, 2) → 2→4
  │
  ├─ mergeSort(1→3)
  │  │  middle(1→3) = Node(1)
  │  │  Split: [1] and [3]
  │  │
  │  ├─ mergeSort(1) → return 1 (base case)
  │  ├─ mergeSort(3) → return 3 (base case)
  │  │
  │  └─ merge(1, 3) → 1→3
  │
  └─ merge(2→4, 1→3) → 1→2→3→4 ✓
```

### Merge Sort Tree

```
            [4, 2, 1, 3]
           /              \
        [4, 2]          [1, 3]
        /    \          /    \
      [4]   [2]      [1]   [3]
        \    /          \    /
        [2, 4]        [1, 3]
           \              /
          [1, 2, 3, 4]
```

---

## 2.7 Merge Sort — Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(n log n) | log n levels, O(n) work per level |
| Space | O(log n) | Recursion stack depth |
| Stable? | Yes | Merge preserves relative order |
| In-place? | Yes | Only pointer manipulation |

### Why O(log n) Space, Not O(n)?

```
  Array merge sort: O(n) space for temporary array
  Linked list merge sort: O(log n) space for recursion stack only
  
  The merge step reuses existing nodes!
  No temporary array needed because we just re-link pointers.
```

---

## 2.8 Bottom-Up Merge Sort (Iterative)

Avoid recursion entirely — merge in passes of increasing size:

```
function mergeSortBottomUp(HEAD):
    if HEAD == NULL or HEAD.next == NULL: return HEAD
    
    // Count length
    n = countNodes(HEAD)
    
    dummy = createNode(0)
    dummy.next = HEAD
    
    size = 1
    while size < n:
        tail = dummy
        curr = dummy.next
        
        while curr ≠ NULL:
            left = curr
            right = split(left, size)
            curr = split(right, size)
            tail = merge(left, right, tail)
        
        size *= 2
    
    return dummy.next
```

```
  Pass 1 (size=1): Merge pairs of single nodes
  [4] [2] [1] [3] → [2,4] [1,3]
  
  Pass 2 (size=2): Merge pairs of 2-node lists
  [2,4] [1,3] → [1,2,3,4]
  
  Total passes: log₂(n)
  O(1) extra space — no recursion!
```

---

## 2.9 Sorting Algorithm Comparison for Linked Lists

| Algorithm | Time (avg) | Time (worst) | Space | Stable? | Good for LL? |
|-----------|-----------|-------------|-------|---------|--------------|
| Merge Sort | O(n log n) | O(n log n) | O(log n) | Yes | ⭐ Best |
| Insertion Sort | O(n²) | O(n²) | O(1) | Yes | Good for small/nearly sorted |
| Quick Sort | O(n log n) | O(n²) | O(log n) | No | Partition is tricky |
| Bubble Sort | O(n²) | O(n²) | O(1) | Yes | Possible but slow |
| Heap Sort | O(n log n) | O(n log n) | O(n) | No | Needs array conversion |

**Merge sort wins** for linked lists because:
1. Merge is O(1) space (just re-link pointers)
2. No random access needed (unlike quicksort partition)
3. Guaranteed O(n log n) (no worst case degradation)

---

## Summary Table

| Operation | Time | Space | Best For |
|-----------|------|-------|----------|
| Merge K – Sequential | O(NK) | O(1) | Small K |
| Merge K – Divide & Conquer | O(N log K) | O(log K) | Large K, space-constrained |
| Merge K – Min-Heap | O(N log K) | O(K) | Streaming / online merge |
| Merge Sort (recursive) | O(n log n) | O(log n) | General LL sorting |
| Merge Sort (iterative) | O(n log n) | O(1) | Avoid recursion |

---

## Quick Revision Questions

1. **Why is sequential merging O(NK) and not O(N log K)?**
   <details><summary>Answer</summary>Each merge step processes an ever-growing result list. The i-th merge processes ~i×(N/K) elements. Sum from 1 to K gives O(K²×N/K) = O(NK). Divide-and-conquer avoids this by merging equal-sized lists.</details>

2. **How many nodes are in the min-heap at any time during merge K?**
   <details><summary>Answer</summary>At most K nodes — one from each list. We never insert more than one node from the same list. When we extract a node, we insert its successor (if it exists) from the same list.</details>

3. **Why is merge sort preferred over quicksort for linked lists?**
   <details><summary>Answer</summary>Quicksort relies on efficient random access for partitioning (swapping elements by index), which is O(n) per swap in a linked list. Merge sort's divide (find middle) and conquer (merge) naturally use sequential access, which linked lists handle well.</details>

4. **Why does linked list merge sort use O(log n) space instead of O(n) like array merge sort?**
   <details><summary>Answer</summary>Array merge sort needs O(n) auxiliary space to merge subarrays. Linked list merge reuses existing nodes by re-linking pointers — no extra nodes are created. The only space used is the O(log n) recursion stack.</details>

5. **In `getMiddle`, why start `fast = HEAD.next` instead of `fast = HEAD`?**
   <details><summary>Answer</summary>For a 2-element list, `fast = HEAD` would return the 2nd middle (same node as split point), creating an infinite recursion since one half would still be the original list. `fast = HEAD.next` returns the 1st middle, ensuring both halves are strictly smaller.</details>

6. **How does bottom-up merge sort achieve O(1) space?**
   <details><summary>Answer</summary>It replaces recursion with iteration using a size variable that doubles each pass. It merges adjacent sublists of increasing size (1, 2, 4, 8, ...) in-place by re-linking pointers, requiring no recursion stack.</details>

---

[← Previous: Merge Two Sorted Lists](01-merge-two-sorted-lists.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Intersection and Union →](03-intersection-and-union.md)
