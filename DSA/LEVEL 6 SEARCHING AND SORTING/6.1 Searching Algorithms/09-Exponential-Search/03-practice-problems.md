# Chapter 3: Practice Problems — Exponential Search

[← Previous: Unbounded Search](02-unbounded-search.md) | [Next Unit: Interpolation Search →](../10-Interpolation-Search/01-concept.md)

---

## Problem Difficulty Guide

```
   ⭐       = Easy (warm-up)
   ⭐⭐     = Medium (apply concepts)
   ⭐⭐⭐   = Hard (combine techniques)
```

---

## Problem 1: Basic Exponential Search ⭐

```
   Given a sorted array and a target, implement
   exponential search and return the index (or -1).
   
   Input:  arr = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91], target = 23
   Output: 5
```

### Solution

```python
def exponential_search(arr, target):
    n = len(arr)
    if n == 0:
        return -1
    if arr[0] == target:
        return 0
    
    i = 1
    while i < n and arr[i] < target:
        i *= 2
    
    lo, hi = i // 2, min(i, n - 1)
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

```
   Trace for target = 23:
   i=1: arr[1]=5  < 23 → i=2
   i=2: arr[2]=8  < 23 → i=4
   i=4: arr[4]=16 < 23 → i=8
   i=8: arr[8]=72 ≥ 23 → STOP
   
   Binary search [4, 8]:
   mid=6: arr[6]=38 > 23 → hi=5
   mid=4: arr[4]=16 < 23 → lo=5
   mid=5: arr[5]=23 = target → FOUND at 5 ✓
```

---

## Problem 2: Search in Unknown Size Array (LC 702) ⭐⭐

```
   Given an ArrayReader (with .get(index)), search for target.
   get(index) returns 2^31 - 1 if out of bounds.
   
   Input:  secret array = [-1,0,3,5,9,12], target = 9
   Output: 4
```

### Solution (Java)

```java
class Solution {
    public int search(ArrayReader reader, int target) {
        // Phase 1: Exponential bound finding
        if (reader.get(0) == target) return 0;
        
        int lo = 0, hi = 1;
        while (reader.get(hi) < target) {
            lo = hi;
            hi *= 2;
        }
        
        // Phase 2: Binary search
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            int val = reader.get(mid);
            if (val == target) return mid;
            else if (val > target) hi = mid - 1;
            else lo = mid + 1;
        }
        return -1;
    }
}
```

**Key insight**: `reader.get(i)` returns `2^31 - 1` for out-of-bounds, which is > any valid target. This naturally pushes hi leftward in binary search.

---

## Problem 3: Find First Occurrence Using Exponential Search ⭐⭐

```
   Given a sorted array with duplicates, find the FIRST
   occurrence of target using exponential search to find
   the range, then modified binary search for first occurrence.
   
   Input:  arr = [1, 2, 3, 3, 3, 3, 5, 8], target = 3
   Output: 2
```

### Solution

```python
def first_occurrence_exp(arr, target):
    n = len(arr)
    if arr[0] == target:
        # Still might need to confirm it's first if target == arr[0]
        return 0
    
    # Find range
    i = 1
    while i < n and arr[i] < target:
        i *= 2
    
    lo, hi = i // 2, min(i, n - 1)
    result = -1
    
    # Modified binary search for first occurrence
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            result = mid
            hi = mid - 1  # Keep searching left
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    
    return result
```

```
   Trace:
   i=1: arr[1]=2 < 3 → i=2
   i=2: arr[2]=3 ≥ 3 → STOP
   
   Binary search [1, 2]:
   mid=1: arr[1]=2 < 3 → lo=2
   mid=2: arr[2]=3 = target → result=2, hi=1
   lo > hi → return 2 ✓
```

---

## Problem 4: Exponential Search on Linked List ⭐⭐⭐

```
   Given a sorted singly linked list, search for a target.
   Optimize using the exponential idea.
   
   Problem: We can't do random access on a linked list!
   Idea: Jump exponentially forward (1, 2, 4, 8, ... nodes),
         then search within the smaller range.
   
   Note: Best achievable is still O(√n) for linked lists
   due to sequential access, but this idea bridges to
   skip lists.
```

### Approach

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def exp_search_linked_list(head, target):
    if not head:
        return None
    if head.val == target:
        return head
    
    # Phase 1: Jump exponentially to find range
    prev = head
    curr = head
    jump = 1
    while curr and curr.val < target:
        prev = curr
        for _ in range(jump):
            if curr.next:
                curr = curr.next
            else:
                break
        jump *= 2
    
    # Phase 2: Linear search from prev to curr
    # (Can't do binary search on linked list)
    node = prev
    while node != curr and node:
        if node.val == target:
            return node
        node = node.next
    if curr and curr.val == target:
        return curr
    return None
```

**Analysis**: Phase 1 takes O(log n) jumps but each jump traverses nodes → O(n) total. This is why skip lists add extra layers for true O(log n).

---

## Problem 5: Two-Sided Exponential Search ⭐⭐

```
   You're given a sorted array and a HINT index where the
   target might be. Search outward from the hint.
   
   Use case: Autocomplete — previous query result gives
   a hint for next query's location.
   
   Input:  arr = [1,3,5,7,9,11,13,15,17,19], hint = 3, target = 11
   Output: 5
```

### Solution

```python
def hinted_search(arr, target, hint):
    n = len(arr)
    if arr[hint] == target:
        return hint
    
    # Determine direction
    if arr[hint] < target:
        # Search right
        lo = hint
        offset = 1
        while lo + offset < n and arr[lo + offset] < target:
            offset *= 2
        hi = min(lo + offset, n - 1)
        lo = lo + offset // 2
    else:
        # Search left
        hi = hint
        offset = 1
        while hi - offset >= 0 and arr[hi - offset] > target:
            offset *= 2
        lo = max(hi - offset, 0)
        hi = hi - offset // 2
    
    # Binary search in [lo, hi]
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

**Complexity**: O(log d) where d = |target_index - hint|. If the hint is good (close), this is very fast!

---

## Problem 6: Find Position in Infinite Binary Sorted Stream ⭐⭐⭐

```
   You receive an infinite stream of 0s followed by 1s:
   0, 0, 0, 0, ..., 0, 1, 1, 1, 1, ...
   
   Find the position of the FIRST 1.
   
   This combines:
   - Exponential search (to find bounds)
   - Boundary binary search (to find first true)
```

### Solution

```python
def first_one(stream):
    """stream.get(i) returns 0 or 1"""
    # Phase 1: Find a 1
    if stream.get(0) == 1:
        return 0
    
    hi = 1
    while stream.get(hi) == 0:
        hi *= 2
    # Now stream.get(hi) == 1 and stream.get(hi//2) == 0
    
    # Phase 2: Binary search for first 1 in [hi//2, hi]
    lo = hi // 2
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if stream.get(mid) == 0:
            lo = mid + 1
        else:
            hi = mid
    return lo
```

```
   Stream: 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 ...
   Index:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 ...
   
   Phase 1: hi=1→0, hi=2→0, hi=4→0, hi=8→0, hi=16→1 STOP
   Phase 2: BS in [8, 16]
     mid=12: 1 → hi=12
     mid=10: 0 → lo=11
     mid=11: 0 → lo=12
     lo=hi=12 → return 12 ✓
```

---

## Problem Summary Table

| # | Problem | Difficulty | Key Technique | Time |
|---|---------|-----------|---------------|------|
| 1 | Basic exponential search | ⭐ | Standard template | O(log i) |
| 2 | Unknown size array (LC 702) | ⭐⭐ | Sentinel for OOB | O(log n) |
| 3 | First occurrence | ⭐⭐ | Exp + boundary BS | O(log i) |
| 4 | Linked list search | ⭐⭐⭐ | Exp jumping + linear scan | O(n) |
| 5 | Hinted/two-sided search | ⭐⭐ | Bidirectional doubling | O(log d) |
| 6 | First 1 in binary stream | ⭐⭐⭐ | Exp + first-true BS | O(log i) |

---

## Practice Checklist

```
   □ Implement basic exponential search from scratch
   □ Solve LC 702 (Search in Unknown Size Array)
   □ Combine with first/last occurrence variants
   □ Understand when O(log i) beats O(log n)
   □ Explain galloping in TimSort context
   □ Identify problems hinting at exponential search
```

---

## Quick Revision Questions

1. **Name three real-world uses of exponential search.**
2. **What is the time complexity if the target is at position 1024?**
3. **How would you modify exponential search for descending arrays?**
4. **Why is hinted exponential search O(log d)?**
5. **What data structure extends exponential search to linked lists?**

---

[← Previous: Unbounded Search](02-unbounded-search.md) | [Next Unit: Interpolation Search →](../10-Interpolation-Search/01-concept.md)
