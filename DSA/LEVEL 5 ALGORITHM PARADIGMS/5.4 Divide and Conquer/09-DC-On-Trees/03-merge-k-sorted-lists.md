# Chapter 3: Merge K Sorted Lists

## Overview

Merging K sorted lists into one sorted list is a classic D&C problem. While a min-heap approach is common, the **D&C approach** mirrors merge sort — recursively merge pairs of lists until one remains. Time: O(N log K) where N = total elements and K = number of lists.

---

## Problem Statement

```
INPUT:  K sorted linked lists
        [1→4→5, 1→3→4, 2→6]

OUTPUT: One sorted linked list
        1→1→2→3→4→4→5→6

Constraints:
  K = number of lists (can be large)
  N = total number of elements across all lists
```

---

## Approach Comparison

```
┌────────────────────────┬──────────┬──────────┬───────────────────┐
│ Approach               │ Time     │ Space    │ Notes             │
├────────────────────────┼──────────┼──────────┼───────────────────┤
│ Brute force (collect   │ O(N logN)│ O(N)     │ Ignore sorted     │
│ all, sort)             │          │          │ property          │
│                        │          │          │                   │
│ Merge one by one       │ O(N·K)   │ O(1)     │ K-1 merge passes  │
│                        │          │          │                   │
│ Min-Heap               │ O(N logK)│ O(K)     │ Heap of K heads   │
│                        │          │          │                   │
│ D&C (pair merge)       │ O(N logK)│ O(logK)  │ Merge sort style  │
└────────────────────────┴──────────┴──────────┴───────────────────┘
```

---

## Why "Merge One by One" Is Slow

```
Merge list₁ with list₂:                  cost ~ n₁ + n₂
Merge result with list₃:                 cost ~ (n₁+n₂) + n₃
Merge result with list₄:                 cost ~ (n₁+n₂+n₃) + n₄
...
Merge result with list_K:               cost ~ N

Total: n₁ appears in K-1 merges
       n₂ appears in K-2 merges, etc.
       ≈ O(N·K)   ← Too slow for large K!
```

---

## D&C Approach: Pair-wise Merge

```
Same idea as merge sort:
  Merge lists in PAIRS, halving the count each round.

Round 0: K lists
  [L₁] [L₂] [L₃] [L₄] [L₅] [L₆] [L₇] [L₈]

Round 1: K/2 lists (merge pairs)
  [L₁₂]    [L₃₄]    [L₅₆]    [L₇₈]

Round 2: K/4 lists
  [L₁₂₃₄]          [L₅₆₇₈]

Round 3: 1 list
  [L₁₂₃₄₅₆₇₈]

Rounds: log₂ K
Work per round: O(N) (each element merged once per round)
Total: O(N log K)
```

---

## Visual Trace

```
Input: L₁ = [1,4,5]  L₂ = [1,3,4]  L₃ = [2,6]  L₄ = [3,7]

Round 1: Merge pairs
  merge(L₁, L₂) = merge([1,4,5], [1,3,4])
    1 → compare 1,1 → take both
    = [1,1,3,4,4,5]

  merge(L₃, L₄) = merge([2,6], [3,7])
    = [2,3,6,7]

  After Round 1: [[1,1,3,4,4,5], [2,3,6,7]]

Round 2: Merge remaining pair
  merge([1,1,3,4,4,5], [2,3,6,7])
    1,1,2,3,3,4,4,5,6,7

  Final: [1,1,2,3,3,4,4,5,6,7]

Total work:
  Round 1: (3+3) + (2+2) = 10 comparisons
  Round 2: (6+4) = 10 comparisons
  Total: 20 = N × log₂K = 10 × 2
```

---

## Implementation: Recursive D&C

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def mergeKLists(lists):
    """Merge K sorted linked lists using D&C."""
    if not lists:
        return None
    return merge_range(lists, 0, len(lists) - 1)

def merge_range(lists, lo, hi):
    if lo == hi:
        return lists[lo]
    
    mid = (lo + hi) // 2
    left = merge_range(lists, lo, mid)
    right = merge_range(lists, mid + 1, hi)
    return merge_two(left, right)

def merge_two(l1, l2):
    """Merge two sorted linked lists."""
    dummy = ListNode(0)
    curr = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    
    curr.next = l1 or l2
    return dummy.next
```

---

## Implementation: Iterative Bottom-Up

```python
def mergeKLists_iterative(lists):
    """Merge K sorted lists using iterative pair-wise merge."""
    if not lists:
        return None
    
    while len(lists) > 1:
        merged = []
        for i in range(0, len(lists), 2):
            l1 = lists[i]
            l2 = lists[i + 1] if i + 1 < len(lists) else None
            merged.append(merge_two(l1, l2))
        lists = merged
    
    return lists[0]
```

---

## Recursion Tree

```
mergeKLists([L₁, L₂, L₃, L₄, L₅])

                merge(0,4)
               /          \
         merge(0,2)      merge(3,4)
        /        \        /      \
   merge(0,1)  L₃     L₄       L₅
   /      \
  L₁      L₂

Bottom-up merge:
  Level 1: merge(L₁,L₂), L₃ alone, merge(L₄,L₅)
  Level 2: merge(L₁₂, L₃), L₄₅ alone
  Level 3: merge(L₁₂₃, L₄₅) → final result

Depth: ⌈log₂ K⌉
Work per level: O(N)
Total: O(N log K)
```

---

## Complexity Analysis

```
Let N = total elements, K = number of lists.

TIME:
  Levels of recursion: log₂ K
  Work per level: Each element participates in one merge → O(N)
  Total: O(N log K)

SPACE:
  Recursion depth: O(log K)
  No extra arrays (reuse linked list nodes): O(1) extra
  Total: O(log K) stack space

COMPARISON:
  ┌─────────────────┬──────────┬─────────────────────┐
  │ Approach         │ Time     │ Advantage           │
  ├─────────────────┼──────────┼─────────────────────┤
  │ D&C pair merge   │ O(N logK)│ No extra structures │
  │ Min-Heap         │ O(N logK)│ Simpler logic       │
  │ Sequential merge │ O(N·K)   │ Easiest to code     │
  └─────────────────┴──────────┴─────────────────────┘
```

---

## Extension: Merge K Sorted Arrays

```
Same D&C approach works for arrays:

def mergeKArrays(arrays):
    if len(arrays) == 1:
        return arrays[0]
    
    mid = len(arrays) // 2
    left = mergeKArrays(arrays[:mid])
    right = mergeKArrays(arrays[mid:])
    return merge_sorted_arrays(left, right)

Time: O(N log K) — same as lists
Space: O(N) — need to allocate merged arrays

For arrays, min-heap might be preferred (O(N log K) time, O(K) space).
```

---

## Connection to External Sort

```
External merge sort uses the same K-way merge:

  Phase 1: Sort chunks that fit in RAM
  Phase 2: K-way merge the sorted chunks

       ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
       │Chunk1│ │Chunk2│ │Chunk3│ │Chunk4│  (on disk)
       └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
          │        │        │        │
     ┌────┴────────┴────┐ ┌─┴────────┴─────┐
     │   Merge 1,2      │ │   Merge 3,4    │  Round 1
     └────────┬─────────┘ └───────┬────────┘
              │                   │
         ┌────┴───────────────────┴────┐
         │      Final Merge            │       Round 2
         └─────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Merge K sorted lists into one |
| D&C idea | Pair-wise merge, halving K each round |
| Time | O(N log K) |
| Space | O(log K) stack |
| Rounds | log₂ K |
| Each round | O(N) total work |
| Alternative | Min-heap: same time, O(K) space |

---

## Quick Revision Questions

1. **Why is merging one-by-one O(N·K)?**
2. **How many rounds does pair-wise merge take?**
3. **What is the total work per round?**
4. **How does D&C compare to min-heap for this problem?**
5. **How does this relate to merge sort?**

---

[← Previous: Balanced BST](02-balanced-bst-from-sorted-array.md) | [Next: Maximum Subarray →](04-maximum-subarray.md)
