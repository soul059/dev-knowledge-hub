# Chapter 1: Merge Two Sorted Linked Lists

[← Previous: Palindrome and DLL Reversal](../06-Linked-List-Reversal/03-palindrome-and-dll-reversal.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Merge K Lists and Merge Sort →](02-merge-k-and-merge-sort.md)

---

## Chapter Overview

Merging two sorted linked lists is a foundational operation that appears everywhere: merge sort, combining datasets, and as a building block for merging K lists. This chapter covers iterative and recursive approaches with detailed traces.

---

## 1.1 Problem Statement

Given two sorted linked lists, merge them into one sorted list.

```
  List A:  1 → 3 → 5 → 7 → NULL
  List B:  2 → 4 → 6 → 8 → NULL
  
  Merged:  1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → NULL
```

The merge must be **stable** — equal elements maintain their original relative order.

---

## 1.2 Iterative Approach (Dummy Node)

### Algorithm

```
function mergeTwoSorted(A, B):
    dummy = createNode(0)
    tail = dummy
    
    while A ≠ NULL and B ≠ NULL:
        if A.data ≤ B.data:
            tail.next = A
            A = A.next
        else:
            tail.next = B
            B = B.next
        tail = tail.next
    
    // Attach remaining nodes
    if A ≠ NULL:
        tail.next = A
    else:
        tail.next = B
    
    return dummy.next
```

### Step-by-Step Trace

```
  A: 1 → 3 → 5 → NULL
  B: 2 → 4 → 6 → NULL
  
  dummy → ?          tail = dummy

  ─── Step 1: A(1) ≤ B(2) ───
  tail.next = A(1), A=3, tail=1
  dummy → [1]

  ─── Step 2: A(3) > B(2) ───
  tail.next = B(2), B=4, tail=2
  dummy → [1] → [2]

  ─── Step 3: A(3) ≤ B(4) ───
  tail.next = A(3), A=5, tail=3
  dummy → [1] → [2] → [3]

  ─── Step 4: A(5) > B(4) ───
  tail.next = B(4), B=6, tail=4
  dummy → [1] → [2] → [3] → [4]

  ─── Step 5: A(5) ≤ B(6) ───
  tail.next = A(5), A=NULL, tail=5
  dummy → [1] → [2] → [3] → [4] → [5]

  ─── A is NULL, exit loop ───
  tail.next = B = [6]
  
  Final: 1 → 2 → 3 → 4 → 5 → 6 → NULL ✓
```

### The Dummy Node Pattern

```
  Without dummy:
  ┌──────────────────────────────────────────┐
  │ Need special logic for the first node    │
  │ if A.data ≤ B.data:                      │
  │     result = A; A = A.next               │
  │ else:                                    │
  │     result = B; B = B.next               │
  │ tail = result                            │
  │ // Then enter the main loop...           │
  └──────────────────────────────────────────┘

  With dummy:
  ┌──────────────────────────────────────────┐
  │ No special case! Uniform logic.          │
  │ Just ignore dummy at the end:            │
  │ return dummy.next                        │
  └──────────────────────────────────────────┘
```

---

## 1.3 Recursive Approach

### Algorithm

```
function mergeTwoSortedRecursive(A, B):
    // Base cases
    if A == NULL: return B
    if B == NULL: return A
    
    if A.data ≤ B.data:
        A.next = mergeTwoSortedRecursive(A.next, B)
        return A
    else:
        B.next = mergeTwoSortedRecursive(A, B.next)
        return B
```

### Recursive Trace

```
  A: 1 → 3 → 5    B: 2 → 4

  merge(1→3→5, 2→4)
  │ 1 ≤ 2 → 1.next = merge(3→5, 2→4)
  │         │ 3 > 2 → 2.next = merge(3→5, 4)
  │         │         │ 3 ≤ 4 → 3.next = merge(5, 4)
  │         │         │         │ 5 > 4 → 4.next = merge(5, NULL)
  │         │         │         │         │ B=NULL → return 5
  │         │         │         │ 4.next = 5 → return 4
  │         │         │ 3.next = 4 → return 3
  │         │ 2.next = 3 → return 2
  │ 1.next = 2 → return 1
  
  Result: 1 → 2 → 3 → 4 → 5 ✓
```

---

## 1.4 Iterative vs Recursive

| Aspect | Iterative | Recursive |
|--------|-----------|-----------|
| Time | O(n + m) | O(n + m) |
| Space | O(1) | O(n + m) call stack |
| Code length | More lines | Very concise |
| Stack overflow | No | Yes, for long lists |
| In-place? | Yes | Yes (reuses nodes) |

---

## 1.5 In-Place vs Creating New Nodes

Both approaches above are **in-place** — they reuse existing nodes by re-linking pointers:

```
  In-place (re-link):
  ┌───┐    ┌───┐    ┌───┐
  │ 1 │    │ 3 │    │ 5 │    ← Same nodes from list A
  └───┘    └───┘    └───┘
  ┌───┐    ┌───┐
  │ 2 │    │ 4 │              ← Same nodes from list B
  └───┘    └───┘
  
  After merge: just the .next pointers are rearranged
  to weave them into sorted order.
  
  No new nodes created!
  
  
  Copy (create new nodes):
  You'd create a new node for each element → O(n+m) space
  Original lists remain unchanged
```

---

## 1.6 Edge Cases

| Case | List A | List B | Result |
|------|--------|--------|--------|
| Both empty | NULL | NULL | NULL |
| One empty | 1→3→5 | NULL | 1→3→5 |
| One element each | 1 | 2 | 1→2 |
| All from A first | 1→2→3 | 4→5→6 | 1→2→3→4→5→6 |
| Duplicates | 1→2→2 | 2→3 | 1→2→2→2→3 |
| Different lengths | 1 | 2→3→4 | 1→2→3→4 |

### Handling Duplicates

```
  The condition A.data ≤ B.data (using ≤, not <) ensures stability:
  
  A: [2a] → [4]
  B: [2b] → [5]
  
  A(2a) ≤ B(2b) → pick A first
  Result: 2a → 2b → 4 → 5
  
  Elements from A always come before equal elements from B.
```

---

## 1.7 Merge Without Dummy Node

If you want to avoid the dummy node:

```
function mergeNoDummy(A, B):
    if A == NULL: return B
    if B == NULL: return A
    
    // Determine head
    if A.data ≤ B.data:
        head = A
        A = A.next
    else:
        head = B
        B = B.next
    
    tail = head
    
    // Standard merge loop
    while A ≠ NULL and B ≠ NULL:
        if A.data ≤ B.data:
            tail.next = A
            A = A.next
        else:
            tail.next = B
            B = B.next
        tail = tail.next
    
    tail.next = (A ≠ NULL) ? A : B
    
    return head
```

---

## 1.8 Merge Two Sorted DLLs

For doubly linked lists, also maintain the `prev` pointer:

```
function mergeSortedDLL(A, B):
    dummy = createDLLNode(0)
    tail = dummy
    
    while A ≠ NULL and B ≠ NULL:
        if A.data ≤ B.data:
            tail.next = A
            A.prev = tail            ← Extra step for DLL
            A = A.next
        else:
            tail.next = B
            B.prev = tail            ← Extra step for DLL
            B = B.next
        tail = tail.next
    
    remaining = (A ≠ NULL) ? A : B
    if remaining ≠ NULL:
        tail.next = remaining
        remaining.prev = tail        ← Fix prev pointer
    
    result = dummy.next
    result.prev = NULL               ← First node has no prev
    return result
```

---

## Summary Table

| Variant | Time | Space | Notes |
|---------|------|-------|-------|
| Iterative (dummy node) | O(n+m) | O(1) | Cleanest approach |
| Iterative (no dummy) | O(n+m) | O(1) | Extra code to handle first node |
| Recursive | O(n+m) | O(n+m) | Elegant but stack risk |
| DLL merge | O(n+m) | O(1) | Must maintain prev pointers |
| Copy merge (new nodes) | O(n+m) | O(n+m) | Preserves original lists |

---

## Quick Revision Questions

1. **Why does the dummy node simplify the merge algorithm?**
   <details><summary>Answer</summary>Without a dummy node, you need special logic to determine which node becomes the head of the merged list. The dummy node provides a uniform `tail.next = ...` operation for every node, including the first. Just return `dummy.next` at the end.</details>

2. **Is the merge stable? How do you ensure stability?**
   <details><summary>Answer</summary>Yes, it's stable. Use `≤` (not `<`) when comparing: `if A.data ≤ B.data`. This picks from list A first when values are equal, preserving the relative order of equal elements.</details>

3. **What happens if you don't attach the remaining nodes after the loop?**
   <details><summary>Answer</summary>You lose the remaining elements. When one list is exhausted, all remaining nodes in the other list must be appended. Since the remaining list is already sorted, just set `tail.next = remaining_list`.</details>

4. **Can merge be done without modifying the original lists?**
   <details><summary>Answer</summary>Yes — create new nodes for each element instead of re-linking. This uses O(n+m) extra space but preserves both original lists. Useful when you need the originals afterward.</details>

5. **What's the minimum number of comparisons to merge two sorted lists of sizes n and m?**
   <details><summary>Answer</summary>Minimum: min(n, m) comparisons (when all elements of one list are smaller). Maximum: n + m - 1 comparisons (when elements interleave perfectly).</details>

6. **How does merging sorted DLLs differ from merging sorted SLLs?**
   <details><summary>Answer</summary>For DLLs, you must also update the `prev` pointer for each node as you link it. When `tail.next = node`, you must also set `node.prev = tail`. And the first node of the result must have `prev = NULL`.</details>

---

[← Previous: Palindrome and DLL Reversal](../06-Linked-List-Reversal/03-palindrome-and-dll-reversal.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Merge K Lists and Merge Sort →](02-merge-k-and-merge-sort.md)
