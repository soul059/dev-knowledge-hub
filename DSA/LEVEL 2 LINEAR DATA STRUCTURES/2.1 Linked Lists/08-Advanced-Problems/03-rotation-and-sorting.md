# Chapter 3: Rotation, Sorting, and Miscellaneous Problems

[← Previous: Arithmetic and Reordering](02-arithmetic-and-reordering.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 9 — Memory Allocation and Leaks →](../09-Memory-Management/01-memory-allocation-and-leaks.md)

---

## Chapter Overview

A collection of important linked list problems: **rotating** a list by K positions, **sorting** with constraints, **partitioning** around a value, and other classic interview problems.

---

## 3.1 Rotate a Linked List by K Positions

### Problem (LeetCode 61)

Rotate the list to the right by K places.

```
  K=2:
  Before:  1 → 2 → 3 → 4 → 5
  After:   4 → 5 → 1 → 2 → 3
  
  Last K nodes move to the front.
```

### Algorithm

```
function rotateRight(HEAD, k):
    if HEAD == NULL or k == 0: return HEAD
    
    // Step 1: Find length and tail
    n = 1
    tail = HEAD
    while tail.next ≠ NULL:
        n++
        tail = tail.next
    
    // Step 2: Normalize K
    k = k % n
    if k == 0: return HEAD          ← No rotation needed
    
    // Step 3: Find the new tail (n-k-th node from head, 0-indexed)
    newTail = HEAD
    for i = 1 to n - k - 1:
        newTail = newTail.next
    
    // Step 4: Perform rotation
    newHead = newTail.next
    newTail.next = NULL
    tail.next = HEAD
    
    return newHead
```

### Visual Trace: K=2, List [1,2,3,4,5]

```
  Step 1: n=5, tail=Node(5)
  Step 2: k = 2 % 5 = 2
  Step 3: newTail position = 5-2-1 = 2 (0-indexed)
          newTail = Node(3)  (1→2→3, stop at 3)
  
  Step 4:
    newHead = 3.next = Node(4)
    3.next = NULL              ← Cut: [1→2→3] | [4→5]
    5.next = Node(1)           ← Connect tail to old head
  
  Before: [1] → [2] → [3] → [4] → [5] → NULL
  
  After:  [4] → [5] → [1] → [2] → [3] → NULL ✓
          newHead     tail.next → old HEAD

  Alternative view — it's just re-linking a circular chain:
  
  [1] → [2] → [3] → [4] → [5]
                       ↑ cut here  ↑ and here (tail→head)
  
  Make circular: 5→1
  Cut at position n-k: 3↛4
```

### Edge Cases

| Case | K | n | k%n | Result |
|------|---|---|-----|--------|
| K = 0 | 0 | 5 | 0 | No change |
| K = n | 5 | 5 | 0 | No change |
| K > n | 7 | 5 | 2 | Same as K=2 |
| K = 1 | 1 | 5 | 1 | Last node to front |
| Single node | any | 1 | 0 | No change |

---

## 3.2 Partition List

### Problem (LeetCode 86)

Partition a list around value X: all nodes < X come before nodes ≥ X. Preserve original order within each partition.

```
  x=3:
  Before:  1 → 4 → 3 → 2 → 5 → 2
  After:   1 → 2 → 2 → 4 → 3 → 5
           (< 3)       (≥ 3)
```

### Algorithm

```
function partition(HEAD, x):
    beforeDummy = createNode(0)
    afterDummy = createNode(0)
    before = beforeDummy
    after = afterDummy
    
    curr = HEAD
    while curr ≠ NULL:
        if curr.data < x:
            before.next = curr
            before = before.next
        else:
            after.next = curr
            after = after.next
        curr = curr.next
    
    after.next = NULL                 ← Terminate after-list
    before.next = afterDummy.next     ← Connect before to after
    
    return beforeDummy.next
```

### Trace

```
  List: 1→4→3→2→5→2,  x=3

  | curr | < 3? | before list | after list |
  |------|-------|-------------|------------|
  | 1    | Yes   | 1           | -          |
  | 4    | No    | 1           | 4          |
  | 3    | No    | 1           | 4→3        |
  | 2    | Yes   | 1→2         | 4→3        |
  | 5    | No    | 1→2         | 4→3→5      |
  | 2    | Yes   | 1→2→2       | 4→3→5      |

  Connect: 1→2→2→4→3→5→NULL ✓
  
  IMPORTANT: Must set after.next = NULL!
  Otherwise 5.next still points to 2, creating a cycle.
```

---

## 3.3 Sort a Linked List of 0s, 1s, and 2s

### Approach 1: Count and Overwrite

```
function sort012(HEAD):
    count = [0, 0, 0]                ← Count of 0s, 1s, 2s
    
    curr = HEAD
    while curr ≠ NULL:
        count[curr.data]++
        curr = curr.next
    
    curr = HEAD
    for val = 0 to 2:
        for i = 1 to count[val]:
            curr.data = val
            curr = curr.next
    
    return HEAD
```

### Approach 2: Three-Way Split (No Data Modification)

```
function sort012Pointers(HEAD):
    zeroD = createNode(0)           ← Dummy for 0s
    oneD = createNode(0)            ← Dummy for 1s
    twoD = createNode(0)            ← Dummy for 2s
    
    zero = zeroD, one = oneD, two = twoD
    
    curr = HEAD
    while curr ≠ NULL:
        if curr.data == 0:
            zero.next = curr; zero = zero.next
        else if curr.data == 1:
            one.next = curr; one = one.next
        else:
            two.next = curr; two = two.next
        curr = curr.next
    
    // Connect: 0s → 1s → 2s
    zero.next = (oneD.next ≠ NULL) ? oneD.next : twoD.next
    one.next = twoD.next
    two.next = NULL
    
    return zeroD.next
```

```
  Input: 1→2→0→1→0→2

  After splitting:
  0-list: 0→0
  1-list: 1→1
  2-list: 2→2

  Connected: 0→0→1→1→2→2 ✓
```

---

## 3.4 Sort List Using Insertion Sort

When the list is **nearly sorted** or **small**, insertion sort is practical.

```
function insertionSort(HEAD):
    dummy = createNode(-∞)           ← Sorted list starts empty
    
    curr = HEAD
    while curr ≠ NULL:
        next = curr.next             ← Save next
        
        // Find insertion position in sorted list
        prev = dummy
        while prev.next ≠ NULL and prev.next.data < curr.data:
            prev = prev.next
        
        // Insert curr after prev
        curr.next = prev.next
        prev.next = curr
        
        curr = next
    
    return dummy.next
```

### Trace: Sort [4, 2, 1, 3]

```
  Sorted: dummy → (empty)

  Process 4: insert after dummy
  Sorted: dummy → 4

  Process 2: 2 < 4, insert before 4
  Sorted: dummy → 2 → 4

  Process 1: 1 < 2, insert before 2
  Sorted: dummy → 1 → 2 → 4

  Process 3: 3 > 2, 3 < 4, insert between 2 and 4
  Sorted: dummy → 1 → 2 → 3 → 4 ✓
```

| Metric | Value |
|--------|-------|
| Time (worst) | O(n²) |
| Time (best — sorted) | O(n) |
| Space | O(1) |
| Stable? | Yes |

---

## 3.5 Remove Nth Node from End (Revisited)

A complete implementation combining the technique from Unit 5:

```
function removeNthFromEnd(HEAD, n):
    dummy = createNode(0)
    dummy.next = HEAD
    
    fast = dummy
    slow = dummy
    
    // Move fast n+1 steps ahead
    for i = 0 to n:
        fast = fast.next
    
    // Move both until fast hits NULL
    while fast ≠ NULL:
        fast = fast.next
        slow = slow.next
    
    // Delete slow.next
    slow.next = slow.next.next
    
    return dummy.next
```

---

## 3.6 Swap Nodes in Pairs

### Problem (LeetCode 24)

Swap every two adjacent nodes (not just values — swap the nodes).

```
  Before:  1 → 2 → 3 → 4
  After:   2 → 1 → 4 → 3
```

### Algorithm

```
function swapPairs(HEAD):
    dummy = createNode(0)
    dummy.next = HEAD
    prev = dummy
    
    while prev.next ≠ NULL and prev.next.next ≠ NULL:
        first = prev.next
        second = prev.next.next
        
        // Swap
        first.next = second.next
        second.next = first
        prev.next = second
        
        prev = first              ← first is now second in pair
    
    return dummy.next
```

### Trace

```
  dummy → [1] → [2] → [3] → [4] → NULL
  prev = dummy

  Step 1: first=1, second=2
    1.next = 2.next = 3
    2.next = 1
    dummy.next = 2
    State: dummy → [2] → [1] → [3] → [4]
    prev = 1

  Step 2: first=3, second=4
    3.next = 4.next = NULL
    4.next = 3
    1.next = 4
    State: dummy → [2] → [1] → [4] → [3] → NULL
    prev = 3

  prev.next = NULL → STOP
  Result: 2 → 1 → 4 → 3 ✓
```

---

## 3.7 Delete Nodes with Greater Value on Right

Delete every node that has a node with a **greater value** to its right.

```
  Before:  12 → 15 → 10 → 11 → 5 → 6 → 2 → 3
  After:   15 → 11 → 6 → 3
  
  Keep a node only if no node to its right is larger.
```

### Algorithm

```
function deleteGreaterRight(HEAD):
    // Reverse the list
    HEAD = reverse(HEAD)
    
    // Traverse and keep track of max
    maxSoFar = HEAD.data
    curr = HEAD
    
    while curr.next ≠ NULL:
        if curr.next.data < maxSoFar:
            curr.next = curr.next.next    ← Delete
        else:
            curr = curr.next
            maxSoFar = curr.data
    
    // Reverse back
    return reverse(HEAD)
```

```
  Why reverse?
  After reversing, "greater on the right" becomes "greater on the left"
  which is just "keep running maximum" — easy to track!
```

---

## Summary Table

| Problem | Key Technique | Time | Space |
|---------|--------------|------|-------|
| Rotate by K | Make circular, cut at n-k | O(n) | O(1) |
| Partition around X | Two separate lists + connect | O(n) | O(1) |
| Sort 0,1,2 | Three-way split | O(n) | O(1) |
| Insertion sort | Build sorted list one by one | O(n²) | O(1) |
| Swap pairs | Three-pointer manipulation | O(n) | O(1) |
| Delete greater right | Reverse + running max + reverse | O(n) | O(1) |

---

## Quick Revision Questions

1. **Why do we compute `k % n` when rotating a linked list?**
   <details><summary>Answer</summary>Rotating by n positions (the length) gives the same list. So k=7 with n=5 is the same as k=2. Modular arithmetic eliminates unnecessary full rotations. Also handles k > n gracefully.</details>

2. **In the partition problem, why must we set `after.next = NULL`?**
   <details><summary>Answer</summary>The last node in the "after" partition might originally have pointed to a node in the "before" partition. Without setting `after.next = NULL`, we'd create a cycle (the list would loop back to an earlier node).</details>

3. **Why is three-way split better than count-and-overwrite for sorting 0,1,2?**
   <details><summary>Answer</summary>Count-and-overwrite modifies node data (values), which isn't possible if nodes carry additional data (like satellite data). Three-way split only moves nodes by re-linking pointers, preserving all node data intact.</details>

4. **How does rotating by K relate to making the list circular?**
   <details><summary>Answer</summary>Rotation is equivalent to: making the list circular (connect tail to head), then cutting at position n-k. This gives the last K nodes as the new beginning, which is exactly a right rotation by K.</details>

5. **What's the time complexity of sorting 0s, 1s, 2s? Can we do better than O(n)?**
   <details><summary>Answer</summary>O(n) — we visit each node exactly once. We cannot do better because we must examine every node at least once to determine its value. O(n) is optimal.</details>

6. **In swap pairs, why does `prev` move to `first` (not `second`) after a swap?**
   <details><summary>Answer</summary>After swapping, `first` is now the second node in the pair (it moved behind `second`). So `first` is the last processed node, making it the `prev` for the next pair. `second` is now the first in the pair and isn't adjacent to the next pair.</details>

---

[← Previous: Arithmetic and Reordering](02-arithmetic-and-reordering.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 9 — Memory Allocation and Leaks →](../09-Memory-Management/01-memory-allocation-and-leaks.md)
