# Chapter 2: Operations on Circular Lists

[← Previous: Circular List Types](01-circular-list-types.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Use Cases and Classic Problems →](03-use-cases-and-problems.md)

---

## Chapter Overview

This chapter provides complete algorithms for **insertion, deletion, traversal, and search** on circular linked lists, with careful attention to maintaining the circular structure.

---

## 2.1 Insertion at the Beginning (SCLL)

### Using TAIL Pointer (Recommended)

```
function insertAtBeginning(TAIL, value):
    newNode = createNode(value)
    
    if TAIL == NULL:                 ← Empty list
        TAIL = newNode
        newNode.next = newNode       ← Points to itself
    else:
        newNode.next = TAIL.next     ← New points to old HEAD
        TAIL.next = newNode          ← TAIL now points to new HEAD
    
    return TAIL                      ← TAIL doesn't change!
```

### Step-by-Step Trace: Insert 5

**Before:**
```
       TAIL
        │
  ┌──► [10] ──► [20] ──► [30] ──┐
  │              ↑                │
  │         TAIL.next = HEAD      │
  └───────────────────────────────┘
```

**Step 1:** `newNode.next = TAIL.next` (new node points to old HEAD = Node 10)
```
  [5] ──► [10]
```

**Step 2:** `TAIL.next = newNode` (TAIL now points to new node as HEAD)
```
       TAIL
        │
  ┌──► [ 5] ──► [10] ──► [20] ──► [30] ──┐
  │                                        │
  └────────────────────────────────────────┘
  
  HEAD = TAIL.next = Node(5)  ✓
```

**Complexity:** O(1)

---

## 2.2 Insertion at the End (SCLL)

### Using TAIL Pointer

```
function insertAtEnd(TAIL, value):
    newNode = createNode(value)
    
    if TAIL == NULL:                 ← Empty list
        TAIL = newNode
        newNode.next = newNode
    else:
        newNode.next = TAIL.next     ← New points to HEAD
        TAIL.next = newNode          ← Old TAIL points to new
        TAIL = newNode               ← Update TAIL to new node
    
    return TAIL
```

### Trace: Insert 40 at End

```
  Before:
       TAIL
        │
  ┌──► [10] ──► [20] ──► [30] ──┐
  │                               │
  └───────────────────────────────┘

  After:
                         TAIL
                          │
  ┌──► [10] ──► [20] ──► [30] ──► [40] ──┐
  │                                        │
  └────────────────────────────────────────┘
```

**Key Insight:** Insert at beginning and end differ by just ONE line — whether we update TAIL or not.

```
  Insert at beginning: DON'T update TAIL (same steps otherwise)
  Insert at end:       DO update TAIL = newNode
```

---

## 2.3 Insertion at a Position (SCLL)

```
function insertAtPosition(TAIL, value, position):
    if position == 0:
        return insertAtBeginning(TAIL, value)
    
    newNode = createNode(value)
    
    if TAIL == NULL:
        error("Position out of bounds")
    
    current = TAIL.next              ← Start at HEAD
    for i = 0 to position - 2:
        current = current.next
        if current == TAIL.next:     ← Wrapped around — out of bounds
            error("Position out of bounds")
    
    newNode.next = current.next
    current.next = newNode
    
    if current == TAIL:              ← Inserted after TAIL → update TAIL
        TAIL = newNode
    
    return TAIL
```

---

## 2.4 Deletion from the Beginning (SCLL)

```
function deleteFromBeginning(TAIL):
    if TAIL == NULL:
        error("List is empty")
    
    if TAIL.next == TAIL:            ← Only one node
        free(TAIL)
        TAIL = NULL
    else:
        HEAD = TAIL.next             ← First node
        TAIL.next = HEAD.next        ← TAIL points to second node (new HEAD)
        free(HEAD)
    
    return TAIL
```

### Trace

```
  Before:
       TAIL
        │
  ┌──► [10] ──► [20] ──► [30] ──┐
  │                               │
  └───────────────────────────────┘

  HEAD = TAIL.next = Node(10)
  TAIL.next = HEAD.next = Node(20)
  free(Node(10))

  After:
       TAIL
        │
  ┌──► [20] ──► [30] ──┐
  │                      │
  └──────────────────────┘
```

---

## 2.5 Deletion from the End (SCLL)

```
function deleteFromEnd(TAIL):
    if TAIL == NULL:
        error("List is empty")
    
    if TAIL.next == TAIL:            ← Only one node
        free(TAIL)
        TAIL = NULL
    else:
        current = TAIL.next          ← Start at HEAD
        while current.next ≠ TAIL:   ← Find second-to-last
            current = current.next
        
        current.next = TAIL.next     ← Second-to-last points to HEAD
        free(TAIL)
        TAIL = current               ← Update TAIL
    
    return TAIL
```

### Trace

```
  Before:
                         TAIL
                          │
  ┌──► [10] ──► [20] ──► [30] ──┐
  │                               │
  └───────────────────────────────┘

  Traverse: current = [10] → [20] (next is TAIL, stop)
  current.next = TAIL.next = [10]  (point to HEAD)
  free([30])
  TAIL = current = [20]

  After:
              TAIL
               │
  ┌──► [10] ──► [20] ──┐
  │                      │
  └──────────────────────┘
```

**Complexity:** O(n) — same limitation as SLL

---

## 2.6 Deletion by Value (SCLL)

```
function deleteByValue(TAIL, target):
    if TAIL == NULL:
        return NULL
    
    // Case 1: Only one node
    if TAIL.next == TAIL:
        if TAIL.data == target:
            free(TAIL)
            return NULL
        return TAIL                  ← Value not found
    
    // Case 2: Target is HEAD
    if TAIL.next.data == target:
        HEAD = TAIL.next
        TAIL.next = HEAD.next
        free(HEAD)
        return TAIL
    
    // Case 3: Search for target
    current = TAIL.next
    while current.next ≠ TAIL.next:
        if current.next.data == target:
            temp = current.next
            current.next = temp.next
            if temp == TAIL:         ← Deleting TAIL
                TAIL = current
            free(temp)
            return TAIL
        current = current.next
    
    return TAIL                      ← Not found
```

---

## 2.7 Traversal (SCLL)

```
function traverse(TAIL):
    if TAIL == NULL:
        print("Empty list")
        return
    
    current = TAIL.next              ← Start at HEAD
    do:
        print(current.data)
        current = current.next
    while current ≠ TAIL.next        ← Until we loop back to HEAD
```

### Counting Nodes

```
function count(TAIL):
    if TAIL == NULL: return 0
    
    count = 0
    current = TAIL.next
    do:
        count = count + 1
        current = current.next
    while current ≠ TAIL.next
    
    return count
```

---

## 2.8 Search (SCLL)

```
function search(TAIL, target):
    if TAIL == NULL: return -1
    
    position = 0
    current = TAIL.next              ← Start at HEAD
    do:
        if current.data == target:
            return position
        current = current.next
        position = position + 1
    while current ≠ TAIL.next
    
    return -1                        ← Not found
```

---

## 2.9 DCLL Operations (Summary)

DCLL operations are similar to DLL but with circular connections.

### Insert at Beginning (DCLL)

```
function insertAtBeginning(HEAD, value):
    newNode = createDCLLNode(value)
    
    if HEAD == NULL:
        newNode.next = newNode
        newNode.prev = newNode
        HEAD = newNode
    else:
        TAIL = HEAD.prev             ← Last node
        
        newNode.next = HEAD
        newNode.prev = TAIL
        HEAD.prev = newNode
        TAIL.next = newNode
        HEAD = newNode
    
    return HEAD
```

### Delete from DCLL

```
function deleteNode(node, HEAD):
    if node.next == node:            ← Only one node
        HEAD = NULL
        free(node)
        return HEAD
    
    node.prev.next = node.next
    node.next.prev = node.prev
    
    if node == HEAD:
        HEAD = node.next
    
    free(node)
    return HEAD
```

**Complexity:** O(1) for DCLL deletion given a node pointer.

---

## 2.10 Complexity Summary for Circular Lists

| Operation | SCLL (with TAIL) | DCLL |
|-----------|------------------|------|
| Insert at beginning | O(1) | O(1) |
| Insert at end | O(1) | O(1) |
| Insert at position k | O(k) | O(k) |
| Delete from beginning | O(1) | O(1) |
| Delete from end | O(n) | O(1) |
| Delete given node | O(n)* | O(1) |
| Search | O(n) | O(n) |
| Traversal | O(n) | O(n) |

\* Need to find previous node in SCLL

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| SCLL insert at front/end | Nearly identical — only difference is whether TAIL updates |
| TAIL pointer advantage | Access both HEAD (TAIL.next) and end (TAIL) in O(1) |
| Traversal termination | Use do-while with start-point check |
| Single node detection | `TAIL.next == TAIL` or `node.next == node` |
| DCLL delete | O(1) with node pointer — same as DLL |
| Key gotcha | Must maintain circular link — never set any `next` to NULL |

---

## Quick Revision Questions

1. **Why is TAIL preferred over HEAD for managing a singly circular list?**
   <details><summary>Answer</summary>TAIL gives O(1) access to both ends: the last node (TAIL) and the first node (TAIL.next). With only HEAD, reaching the end requires O(n) traversal.</details>

2. **What is the only difference between insert-at-beginning and insert-at-end in a SCLL with TAIL?**
   <details><summary>Answer</summary>Insert at beginning: TAIL stays the same. Insert at end: TAIL is updated to the new node. All other pointer operations are identical.</details>

3. **How do you detect that a circular list has only one node?**
   <details><summary>Answer</summary>Check if `TAIL.next == TAIL` (or `node.next == node`). The single node points to itself.</details>

4. **Why can't you use a regular while loop for circular list traversal?**
   <details><summary>Answer</summary>The loop condition `current ≠ HEAD` is true at the start since current IS HEAD. The loop body never executes. A do-while loop processes the node first, then checks.</details>

5. **What happens if you accidentally set a node's `next` to NULL in a circular list?**
   <details><summary>Answer</summary>The circular structure breaks. Traversal will hit NULL and crash (or terminate prematurely). All nodes after the break become unreachable.</details>

6. **Is delete-from-end O(1) in a singly circular list with TAIL?**
   <details><summary>Answer</summary>No, it's still O(n). You need to find the second-to-last node to update its `next` to HEAD. TAIL doesn't help with finding the predecessor in a singly linked structure.</details>

---

[← Previous: Circular List Types](01-circular-list-types.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Use Cases and Classic Problems →](03-use-cases-and-problems.md)
