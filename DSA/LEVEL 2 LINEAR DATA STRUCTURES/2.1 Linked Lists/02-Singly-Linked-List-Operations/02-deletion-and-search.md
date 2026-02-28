# Chapter 2: Deletion and Search Operations

[← Previous: Insertion Operations](01-insertion-operations.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Edge Cases and Complexity →](03-edge-cases-and-complexity.md)

---

## Chapter Overview

This chapter covers **deleting** nodes (by position and by value) and **searching** for elements in a singly linked list. You will learn the critical "previous pointer" technique and understand memory management during deletion.

---

## 2.1 Delete from the Beginning

The simplest deletion — remove the first node.

### Algorithm

```
function deleteFromBeginning(HEAD):
    if HEAD == NULL:
        error("List is empty")
    
    temp = HEAD                      ← Save reference to the first node
    HEAD = HEAD.next                 ← Move HEAD to the second node
    free(temp)                       ← Deallocate old first node
    return HEAD
```

### Step-by-Step Trace

**Before:**
```
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Step 1:** `temp = HEAD`
```
  temp   HEAD
   │      │
   ▼      ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Step 2:** `HEAD = HEAD.next`
```
  temp         HEAD
   │            │
   ▼            ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Step 3:** `free(temp)` — deallocate Node(10)
```
           HEAD
            │
            ▼
          [20] ──► [30] ──► NULL
```

### Complexity

| Metric | Value |
|--------|-------|
| Time | O(1) |
| Space | O(1) |

---

## 2.2 Delete from the End

Must traverse to the **second-to-last** node.

### Algorithm

```
function deleteFromEnd(HEAD):
    if HEAD == NULL:
        error("List is empty")
    
    if HEAD.next == NULL:            ← Only one node
        free(HEAD)
        HEAD = NULL
        return HEAD
    
    current = HEAD
    while current.next.next ≠ NULL:  ← Stop at second-to-last
        current = current.next
    
    free(current.next)               ← Deallocate last node
    current.next = NULL              ← Second-to-last becomes last
    return HEAD
```

### Step-by-Step Trace: Delete Last Node

**Before:**
```
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Traverse to second-to-last:**
```
  current = Node(10)
    current.next.next = Node(30).next = NULL? No (Node(30) exists)
    Wait — current.next.next = Node(20).next = Node(30) ≠ NULL → move

  current = Node(20)
    current.next.next = Node(30).next = NULL → STOP
```

**Delete:**
```
  free(current.next)     ← free Node(30)
  current.next = NULL    ← Node(20) becomes last

  HEAD
   │
   ▼
 [10] ──► [20] ──► NULL
```

### Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(n) | Must traverse to second-to-last node |
| Space | O(1) | Only pointer variables used |

---

## 2.3 Delete at a Given Position

### Algorithm

```
function deleteAtPosition(HEAD, position):
    if HEAD == NULL:
        error("List is empty")
    
    if position == 0:                    ← Delete first node
        temp = HEAD
        HEAD = HEAD.next
        free(temp)
        return HEAD
    
    current = HEAD
    for i = 0 to position - 2:          ← Traverse to node BEFORE target
        if current.next == NULL:
            error("Position out of bounds")
        current = current.next
    
    if current.next == NULL:
        error("Position out of bounds")
    
    temp = current.next                  ← Node to delete
    current.next = temp.next             ← Bypass the target node
    free(temp)                           ← Deallocate
    return HEAD
```

### Step-by-Step Trace: Delete at Position 2

**Before:**
```
  Position:  0      1      2      3
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► [40] ──► NULL
```

**Traverse to position 1 (node before target):**
```
  i = 0: current = Node(10), current = current.next = Node(20)
  
  current is now at Node(20) — the node BEFORE position 2
```

**Delete Node(30):**
```
  temp = current.next = Node(30)
  current.next = temp.next = Node(40)
  free(temp)

  HEAD
   │
   ▼
 [10] ──► [20] ──► [40] ──► NULL
```

### Diagram: The Bypass

```
  Before:
  ... ──► [20] ──► [30] ──► [40] ──► ...
           ↑        ↑
         current   temp (to delete)

  After:
  ... ──► [20] ──────────► [40] ──► ...
                   [30]  ← freed
                   (gone)
```

---

## 2.4 Delete by Value (First Occurrence)

### Algorithm

```
function deleteByValue(HEAD, target):
    if HEAD == NULL:
        return HEAD                      ← Empty list
    
    // Special case: target is in the HEAD node
    if HEAD.data == target:
        temp = HEAD
        HEAD = HEAD.next
        free(temp)
        return HEAD
    
    // General case: find node before the target
    current = HEAD
    while current.next ≠ NULL and current.next.data ≠ target:
        current = current.next
    
    if current.next == NULL:
        print("Value not found")
        return HEAD
    
    // Delete current.next
    temp = current.next
    current.next = temp.next
    free(temp)
    return HEAD
```

### Step-by-Step Trace: Delete Value 30

**List:** `10 → 20 → 30 → 40 → NULL`

```
  HEAD.data = 10 ≠ 30 → not HEAD

  current = Node(10)
    current.next.data = 20 ≠ 30 → move
  
  current = Node(20)
    current.next.data = 30 == 30 → FOUND!

  temp = current.next = Node(30)
  current.next = Node(40)
  free(temp)

  Result: 10 → 20 → 40 → NULL  ✓
```

---

## 2.5 Delete All Occurrences of a Value

### Algorithm

```
function deleteAllOccurrences(HEAD, target):
    // Remove all leading nodes with target value
    while HEAD ≠ NULL and HEAD.data == target:
        temp = HEAD
        HEAD = HEAD.next
        free(temp)
    
    if HEAD == NULL:
        return HEAD
    
    // Remove remaining occurrences
    current = HEAD
    while current.next ≠ NULL:
        if current.next.data == target:
            temp = current.next
            current.next = temp.next
            free(temp)
            // DON'T move current — check new current.next
        else:
            current = current.next
    
    return HEAD
```

### Trace: Delete All 20s from `20 → 10 → 20 → 30 → 20 → NULL`

```
  Phase 1: Remove leading 20s
    HEAD = Node(20) → data = 20 → delete, HEAD = Node(10)
    HEAD = Node(10) → data ≠ 20 → stop

  Phase 2: Remove inner 20s
    current = Node(10)
      current.next = Node(20) → data = 20 → delete
      current.next = Node(30) now
      (don't move current — recheck)
    
    current = Node(10)
      current.next = Node(30) → data ≠ 20 → move
    
    current = Node(30)
      current.next = Node(20) → data = 20 → delete
      current.next = NULL now
    
    current.next = NULL → exit loop

  Result: 10 → 30 → NULL  ✓
```

---

## 2.6 Search Operation

### Linear Search

```
function search(HEAD, target):
    current = HEAD
    position = 0
    
    while current ≠ NULL:
        if current.data == target:
            return position              ← Found at this position
        current = current.next
        position = position + 1
    
    return -1                            ← Not found
```

### Recursive Search

```
function recursiveSearch(node, target, position = 0):
    if node == NULL:
        return -1                        ← Base case: not found
    
    if node.data == target:
        return position                  ← Found!
    
    return recursiveSearch(node.next, target, position + 1)
```

### Search Complexity

| Case | Time |
|------|------|
| Best | O(1) — target is at HEAD |
| Worst | O(n) — target at end or absent |
| Average | O(n/2) = O(n) |

---

## 2.7 The "Previous Pointer" Pattern

Most deletion operations need access to the **node before** the target. This is the fundamental challenge in singly linked lists.

### Pattern: Maintaining `prev` and `current`

```
function deleteWithPrev(HEAD, target):
    if HEAD == NULL: return HEAD
    
    prev = NULL
    current = HEAD
    
    // Find the target node
    while current ≠ NULL and current.data ≠ target:
        prev = current              ← prev trails behind current
        current = current.next
    
    if current == NULL: return HEAD  ← Not found
    
    if prev == NULL:                 ← Target is HEAD
        HEAD = HEAD.next
    else:
        prev.next = current.next    ← Bypass current
    
    free(current)
    return HEAD
```

### Visual: Prev-Current Dance

```
  Iteration 1:
    prev = NULL    current = Node(10)
                     │
                     ▼
                   [10] ──► [20] ──► [30] ──► NULL

  Iteration 2:
    prev = Node(10)    current = Node(20)
      │                  │
      ▼                  ▼
    [10] ──────────► [20] ──► [30] ──► NULL

  Iteration 3:
    prev = Node(20)    current = Node(30)
                │        │
                ▼        ▼
    [10] ──► [20] ──► [30] ──► NULL
```

---

## 2.8 The Dummy Node Technique

Using a **dummy (sentinel) node** simplifies deletion by eliminating the special case for HEAD.

### Algorithm with Dummy Node

```
function deleteByValueDummy(HEAD, target):
    dummy = createNode(0)            ← Dummy node before HEAD
    dummy.next = HEAD
    
    current = dummy
    while current.next ≠ NULL:
        if current.next.data == target:
            temp = current.next
            current.next = temp.next
            free(temp)
            break                    ← Remove first occurrence only
        current = current.next
    
    HEAD = dummy.next                ← New HEAD (in case old HEAD was deleted)
    free(dummy)                      ← Clean up dummy
    return HEAD
```

### Why Dummy Nodes Help

```
  Without dummy — must handle HEAD deletion separately:
  ┌──────────────────────────────┐
  │ if target is at HEAD:        │
  │   HEAD = HEAD.next           │  ← Special case!
  │ else:                        │
  │   find prev, prev.next = ... │
  └──────────────────────────────┘

  With dummy — uniform logic for ALL positions:
  ┌──────────────────────────────┐
  │ dummy ──► [HEAD] ──► [...]   │
  │                              │
  │ Always: prev.next = target   │  ← No special case!
  │ Even for HEAD, prev = dummy  │
  └──────────────────────────────┘
```

---

## 2.9 Delete the Node Itself (Without HEAD Access)

**Problem:** Given only a pointer to the node to be deleted (not HEAD), delete it.

**Trick:** Copy the next node's data into the current node, then delete the next node.

```
function deleteNodeDirect(nodeToDelete):
    if nodeToDelete.next == NULL:
        error("Cannot delete last node this way")
    
    nodeToDelete.data = nodeToDelete.next.data   ← Copy next's data
    temp = nodeToDelete.next
    nodeToDelete.next = temp.next                ← Skip next node
    free(temp)
```

### Diagram

```
  Before: ... ──► [A] ──► [B] ──► [C] ──► ...
                   ↑
                 delete this

  Step 1: Copy B's data into A's position
          ... ──► [B] ──► [B] ──► [C] ──► ...
                   ↑
                 (was A, now has B's data)

  Step 2: Delete the original B
          ... ──► [B] ──► [C] ──► ...
                   ↑
                 (effectively, A is deleted)
```

> **Limitation:** This trick cannot delete the **last node** — there's no next node to copy from.

---

## Summary Table

| Operation | Time | Space | Key Technique |
|-----------|------|-------|---------------|
| Delete from beginning | O(1) | O(1) | Move HEAD forward |
| Delete from end | O(n) | O(1) | Traverse to second-to-last |
| Delete at position k | O(k) | O(1) | Traverse to k-1, bypass target |
| Delete by value | O(n) | O(1) | Previous pointer pattern |
| Delete all occurrences | O(n) | O(1) | Don't advance after deletion |
| Search | O(n) | O(1) | Linear scan |
| Delete without HEAD | O(1) | O(1) | Copy-and-skip trick |
| Dummy node technique | varies | O(1) | Eliminates HEAD special case |

---

## Quick Revision Questions

1. **Why is deleting from the end of a singly linked list O(n)?**
   <details><summary>Answer</summary>You need to find the second-to-last node to update its `next` pointer to NULL. In a singly linked list, you can't go backward from the last node — you must traverse from HEAD.</details>

2. **What is the "previous pointer" pattern and why is it needed?**
   <details><summary>Answer</summary>It's maintaining a `prev` pointer that trails one behind `current`. Since singly linked list nodes don't have a `prev` pointer, you need to track the previous node yourself to update its `next` during deletion.</details>

3. **How does a dummy node simplify deletion code?**
   <details><summary>Answer</summary>A dummy node placed before HEAD ensures every real node (including HEAD) has a predecessor. This eliminates the special case for deleting the HEAD node, making the code uniform.</details>

4. **Can you delete a node if you only have a pointer to it (not HEAD)?**
   <details><summary>Answer</summary>Yes, using the copy-and-skip trick: copy the next node's data into the current node, then delete the next node. Exception: this doesn't work for the last node.</details>

5. **When deleting all occurrences of a value, why shouldn't you advance `current` after a deletion?**
   <details><summary>Answer</summary>After deletion, `current.next` now points to a new node that hasn't been checked yet. If you advance, you might skip a consecutive occurrence of the target value.</details>

6. **What's the time complexity of searching for a value in a sorted linked list?**
   <details><summary>Answer</summary>Still O(n). Unlike sorted arrays, you can't do binary search on linked lists because accessing the middle element requires O(n) traversal. However, you can stop early if `current.data > target`.</details>

---

[← Previous: Insertion Operations](01-insertion-operations.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Edge Cases and Complexity →](03-edge-cases-and-complexity.md)
