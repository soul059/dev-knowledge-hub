# 8.6 Variations & Applications

[← Previous: Heap vs Divide & Conquer](05-heap-vs-divide-conquer.md) | [Next: Unit 9 — Reorganize String →](../09-Advanced-Heap-Problems/01-reorganize-string.md)

---

## Merge K Sorted Iterators (Streaming)

```
class KWayMergeIterator:
    heap = []
    
    INIT(iterators):
        FOR i, iter in enumerate(iterators):
            IF iter.hasNext():
                heap.insert((iter.next(), i, iter))
    
    HAS_NEXT():
        RETURN heap is not empty
    
    NEXT():
        (value, idx, iter) = heap.extract_min()
        IF iter.hasNext():
            heap.insert((iter.next(), idx, iter))
        RETURN value
```

---

## Merge with Duplicate Removal

```
MERGE_K_UNIQUE(lists):
    // Same as regular merge, but skip duplicates
    prev = null
    WHILE heap not empty:
        val = heap.extract_min()
        IF val != prev:
            output(val)
            prev = val
        // Still advance the list
```

---

## Merge K Sorted in Descending Order

Use a **max-heap** instead of min-heap, and each list should be sorted in descending order. Extract the maximum each time. Alternatively, merge ascending and reverse the result.

---

## Real-World Applications

| Application | Description |
|-------------|-------------|
| **External sorting** | Merge K sorted runs from disk |
| **Database query** | Merge results from K sorted indexes |
| **Log aggregation** | Merge K timestamped log streams |
| **Distributed systems** | Merge sorted results from K nodes |
| **Git merge** | Merge sorted commit histories |
| **Search engines** | Merge K ranked result lists |

---

## Unit 8 Summary Table

| Concept | Key Point |
|---------|-----------|
| Core idea | Min-heap holds one element per list; always extract global min |
| Merge K lists | O(N log K) time, O(K) space |
| Merge K arrays | Same approach, track (array_idx, elem_idx) |
| Smallest range | Track min (heap root) and max; minimize range width |
| Heap vs D&C | Same time complexity; heap is simpler, D&C more parallelizable |
| Tie-breaking | Use list index as secondary key in heap |
| Streaming | Heap approach works naturally with streams |
| Key invariant | Heap always holds exactly one element from each active list |

---

## Revision Questions

1. **If all K lists have the same length n, what is the total time complexity?**
   <details><summary>Answer</summary>
   N = K × n total elements. Time = O(Kn × log K). Each of the Kn elements is inserted and extracted from a heap of size K.
   </details>

2. **In the "smallest range" problem, why do we always advance the list that contributed the minimum?**
   <details><summary>Answer</summary>
   The minimum defines the lower bound of the range. To potentially shrink the range, we need to increase the lower bound. The only way to do that is to advance past the current minimum by getting the next element from its list.
   </details>

3. **Why do we include a list index as a tie-breaker in the heap?**
   <details><summary>Answer</summary>
   When two elements have equal values, Python's heapq tries to compare the next tuple element. If that element is a ListNode (which isn't comparable), it raises an error. The list index (an integer) serves as a safe tie-breaker.
   </details>

4. **How would you merge K sorted lists in DESCENDING order?**
   <details><summary>Answer</summary>
   Use a MAX-heap instead of min-heap, and each list should be sorted in descending order. Extract the maximum each time and add the next element from that list. Alternatively, merge ascending and reverse the result.
   </details>

---

[← Previous: Heap vs Divide & Conquer](05-heap-vs-divide-conquer.md) | [Next: Unit 9 — Reorganize String →](../09-Advanced-Heap-Problems/01-reorganize-string.md)
