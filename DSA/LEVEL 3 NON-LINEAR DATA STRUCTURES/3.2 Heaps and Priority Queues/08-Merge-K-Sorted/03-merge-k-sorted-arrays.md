# 8.3 Merge K Sorted Arrays

[← Previous: Merge K Sorted Lists](02-merge-k-sorted-lists.md) | [Next: Smallest Range →](04-smallest-range.md)

---

## Same Problem, Different Data Structure

Instead of linked list nodes, track `(value, array_index, element_index)`.

```
Input:
  arr1 = [1, 4, 7]
  arr2 = [2, 5, 8]
  arr3 = [3, 6, 9]
```

---

## Pseudocode

```
MERGE_K_SORTED_ARRAYS(arrays):
    min_heap = []
    result = []
    
    // Initialize with first element of each array
    FOR i = 0 TO K-1:
        IF arrays[i] is not empty:
            min_heap.insert((arrays[i][0], i, 0))
    
    WHILE min_heap is not empty:
        (value, arr_idx, elem_idx) = min_heap.extract_min()
        result.append(value)
        
        // If more elements in this array, add next one
        IF elem_idx + 1 < arrays[arr_idx].length:
            next_val = arrays[arr_idx][elem_idx + 1]
            min_heap.insert((next_val, arr_idx, elem_idx + 1))
    
    RETURN result
```

---

## Trace

```
arrays = [[1,5,9], [2,6,10], [3,7,11]]

Init: heap = [(1,0,0), (2,1,0), (3,2,0)]

Extract (1,0,0) → result=[1], insert (5,0,1)
  heap = [(2,1,0), (5,0,1), (3,2,0)]

Extract (2,1,0) → result=[1,2], insert (6,1,1)
  heap = [(3,2,0), (5,0,1), (6,1,1)]

Extract (3,2,0) → result=[1,2,3], insert (7,2,1)
  heap = [(5,0,1), (7,2,1), (6,1,1)]

... continue until all extracted ...

Result: [1, 2, 3, 5, 6, 7, 9, 10, 11] ✅
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(N log K) |
| Space | O(K) + O(N) for result array |

---

[← Previous: Merge K Sorted Lists](02-merge-k-sorted-lists.md) | [Next: Smallest Range →](04-smallest-range.md)
