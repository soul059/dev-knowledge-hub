# Chapter 2: DLL Insertion and Deletion

[← Previous: DLL Structure and Advantages](01-dll-structure-and-advantages.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: DLL Traversal and Trade-offs →](03-dll-traversal-and-tradeoffs.md)

---

## Chapter Overview

This chapter provides **complete algorithms with step-by-step traces** for all insertion and deletion operations on a Doubly Linked List. The key challenge is keeping the **bidirectional invariant** intact: if `A.next = B`, then `B.prev = A`.

---

## 2.1 Insert at the Beginning

### Algorithm

```
function insertAtBeginning(HEAD, TAIL, value):
    newNode = createDLLNode(value)
    
    if HEAD == NULL:                 ← Empty list
        HEAD = newNode
        TAIL = newNode
    else:
        newNode.next = HEAD          ← Point new to old HEAD
        HEAD.prev = newNode          ← Old HEAD points back to new
        HEAD = newNode               ← Update HEAD
    
    return HEAD, TAIL
```

### Step-by-Step Trace: Insert 5 at Beginning

**Before:**
```
  HEAD                          TAIL
   │                             │
   ▼                             ▼
 [NULL|10|──]──►[◄──|20|──]──►[◄──|30|NULL]
```

**Step 1:** Create newNode(5)
```
  newNode = [NULL|5|NULL]
```

**Step 2:** `newNode.next = HEAD`
```
  newNode
   │
   ▼
 [NULL|5|──]──► [NULL|10|──]──►[◄──|20|──]──►[◄──|30|NULL]
                  ↑
                 HEAD
```

**Step 3:** `HEAD.prev = newNode`
```
  newNode
   │
   ▼
 [NULL|5|──]⇄[◄──|10|──]──►[◄──|20|──]──►[◄──|30|NULL]
                  ↑
                 HEAD
```

**Step 4:** `HEAD = newNode`
```
  HEAD                                       TAIL
   │                                          │
   ▼                                          ▼
 [NULL|5|──]⇄[◄──|10|──]⇄[◄──|20|──]⇄[◄──|30|NULL]
```

**Complexity:** O(1) time, O(1) space

---

## 2.2 Insert at the End

### Algorithm

```
function insertAtEnd(HEAD, TAIL, value):
    newNode = createDLLNode(value)
    
    if TAIL == NULL:                 ← Empty list
        HEAD = newNode
        TAIL = newNode
    else:
        TAIL.next = newNode          ← Old TAIL points to new
        newNode.prev = TAIL          ← New points back to old TAIL
        TAIL = newNode               ← Update TAIL
    
    return HEAD, TAIL
```

### Step-by-Step Trace: Insert 40 at End

**Before:**
```
  NULL ◄── [10] ⇄ [20] ⇄ [30] ──► NULL
            ↑                 ↑
           HEAD              TAIL
```

**After:**
```
  NULL ◄── [10] ⇄ [20] ⇄ [30] ⇄ [40] ──► NULL
            ↑                        ↑
           HEAD                    TAIL
```

**Complexity:** O(1) time, O(1) space (with TAIL pointer)

---

## 2.3 Insert After a Given Node

### Algorithm

```
function insertAfter(givenNode, value, TAIL):
    if givenNode == NULL:
        error("Given node is NULL")
    
    newNode = createDLLNode(value)
    
    newNode.next = givenNode.next            ← Step 1
    newNode.prev = givenNode                 ← Step 2
    
    if givenNode.next ≠ NULL:
        givenNode.next.prev = newNode        ← Step 3: Update successor's prev
    else:
        TAIL = newNode                       ← givenNode was TAIL, update TAIL
    
    givenNode.next = newNode                 ← Step 4
    
    return TAIL
```

### Detailed Diagram: Insert 25 After Node(20)

```
  Before:
      [10] ⇄ [20] ⇄ [30]
               ↑
            givenNode

  Step 1: newNode.next = givenNode.next = Node(30)
  Step 2: newNode.prev = givenNode = Node(20)

      [10] ⇄ [20]   [25]   [30]
               │      ↑ │     ↑
               │      │ └─────┘ (newNode.next = 30)
               └──────┘        (newNode.prev = 20)

  Step 3: givenNode.next.prev = newNode (Node(30).prev = newNode)
  Step 4: givenNode.next = newNode

      [10] ⇄ [20] ⇄ [25] ⇄ [30]
```

### Why This Order Matters

```
  ✓ CORRECT ORDER (4 steps):
    1. newNode.next = givenNode.next     ← Save forward link
    2. newNode.prev = givenNode          ← Set backward link
    3. givenNode.next.prev = newNode     ← Update old successor
    4. givenNode.next = newNode          ← Redirect given node

  ❌ If you do step 4 before step 3:
    givenNode.next = newNode             ← Now givenNode.next is newNode!
    givenNode.next.prev = newNode        ← This sets NEWNODE.prev, not Node(30).prev!
    → Corrupted list!
```

---

## 2.4 Insert Before a Given Node

### Algorithm

```
function insertBefore(givenNode, value, HEAD):
    if givenNode == NULL:
        error("Given node is NULL")
    
    newNode = createDLLNode(value)
    
    newNode.prev = givenNode.prev            ← Step 1
    newNode.next = givenNode                 ← Step 2
    
    if givenNode.prev ≠ NULL:
        givenNode.prev.next = newNode        ← Step 3: Update predecessor's next
    else:
        HEAD = newNode                       ← givenNode was HEAD, update HEAD
    
    givenNode.prev = newNode                 ← Step 4
    
    return HEAD
```

### Diagram: Insert 15 Before Node(20)

```
  Before:
      [10] ⇄ [20] ⇄ [30]
               ↑
            givenNode

  After:
      [10] ⇄ [15] ⇄ [20] ⇄ [30]
               ↑
            newNode
```

> **Note:** In a SLL, inserting before a given node requires O(n) traversal. In DLL, it's O(1) using `givenNode.prev`.

---

## 2.5 Delete from the Beginning

### Algorithm

```
function deleteFromBeginning(HEAD, TAIL):
    if HEAD == NULL:
        error("List is empty")
    
    temp = HEAD
    
    if HEAD == TAIL:                 ← Only one node
        HEAD = NULL
        TAIL = NULL
    else:
        HEAD = HEAD.next
        HEAD.prev = NULL             ← New HEAD has no predecessor
    
    free(temp)
    return HEAD, TAIL
```

### Trace

```
  Before:
    NULL ◄── [10] ⇄ [20] ⇄ [30] ──► NULL
              ↑                 ↑
             HEAD              TAIL

  After:
    NULL ◄── [20] ⇄ [30] ──► NULL
              ↑          ↑
             HEAD       TAIL
```

**Complexity:** O(1)

---

## 2.6 Delete from the End

### Algorithm

```
function deleteFromEnd(HEAD, TAIL):
    if TAIL == NULL:
        error("List is empty")
    
    temp = TAIL
    
    if HEAD == TAIL:                 ← Only one node
        HEAD = NULL
        TAIL = NULL
    else:
        TAIL = TAIL.prev             ← Move TAIL backward
        TAIL.next = NULL             ← New TAIL has no successor
    
    free(temp)
    return HEAD, TAIL
```

### Trace

```
  Before:
    NULL ◄── [10] ⇄ [20] ⇄ [30] ──► NULL
              ↑                 ↑
             HEAD              TAIL

  Step 1: TAIL = TAIL.prev = Node(20)
  Step 2: TAIL.next = NULL
  Step 3: free(Node(30))

  After:
    NULL ◄── [10] ⇄ [20] ──► NULL
              ↑          ↑
             HEAD       TAIL
```

**KEY DIFFERENCE from SLL:** This is O(1) in DLL! In SLL, deleting from the end is O(n) because you can't go backward.

---

## 2.7 Delete a Given Node (The O(1) Delete)

This is the **superpower** of DLL — delete any node in O(1) given a pointer to it.

### Algorithm

```
function deleteNode(node, HEAD, TAIL):
    if node == NULL:
        error("Node is NULL")
    
    // Update predecessor
    if node.prev ≠ NULL:
        node.prev.next = node.next
    else:
        HEAD = node.next             ← Deleting HEAD
    
    // Update successor
    if node.next ≠ NULL:
        node.next.prev = node.prev
    else:
        TAIL = node.prev             ← Deleting TAIL
    
    free(node)
    return HEAD, TAIL
```

### Diagram: Delete Node(20) — Middle Node

```
  Before:
    [10] ⇄ [20] ⇄ [30]
             ↑
           node (to delete)

  Step 1: node.prev.next = node.next
          Node(10).next = Node(30)

  Step 2: node.next.prev = node.prev
          Node(30).prev = Node(10)

  After:
    [10] ⇄ [30]
           (Node 20 freed)
```

### All Cases for O(1) Delete

```
  Case 1: Delete HEAD (no predecessor)
  ─────────────────────────────────────
    [node] ⇄ [B] ⇄ [C]
      ↑
    HEAD = node.next = B
    B.prev = NULL

  Case 2: Delete TAIL (no successor)
  ─────────────────────────────────────
    [A] ⇄ [B] ⇄ [node]
                    ↑
    TAIL = node.prev = B
    B.next = NULL

  Case 3: Delete middle node
  ─────────────────────────────────────
    [A] ⇄ [node] ⇄ [C]
    A.next = C
    C.prev = A

  Case 4: Delete only node
  ─────────────────────────────────────
    [node]
    HEAD = NULL
    TAIL = NULL
```

---

## 2.8 Insertion/Deletion Summary with Pointer Updates

### Insert Operations — Pointers Changed

| Operation | Pointers Modified | Count |
|-----------|------------------|-------|
| Insert at beginning | newNode.next, HEAD.prev, HEAD | 3 |
| Insert at end | TAIL.next, newNode.prev, TAIL | 3 |
| Insert after node | newNode.next, newNode.prev, successor.prev, givenNode.next | 4 |
| Insert before node | newNode.prev, newNode.next, predecessor.next, givenNode.prev | 4 |

### Delete Operations — Pointers Changed

| Operation | Pointers Modified | Count |
|-----------|------------------|-------|
| Delete from beginning | HEAD, new HEAD.prev | 2 |
| Delete from end | TAIL, new TAIL.next | 2 |
| Delete given middle node | predecessor.next, successor.prev | 2 |

---

## Summary Table

| Operation | DLL Time | SLL Time | Improvement |
|-----------|----------|----------|-------------|
| Insert at beginning | O(1) | O(1) | Same |
| Insert at end (with tail) | O(1) | O(1) | Same |
| Insert before a node | O(1) | O(n) | n → 1 |
| Delete from beginning | O(1) | O(1) | Same |
| Delete from end (with tail) | O(1) | O(n) | n → 1 |
| Delete a given node | O(1) | O(n) | n → 1 |
| Key challenge | Update 2-4 pointers correctly | Update 1-2 pointers | — |

---

## Quick Revision Questions

1. **How many pointers must be updated to insert a node in the middle of a DLL?**
   <details><summary>Answer</summary>4 pointers: newNode.next, newNode.prev, predecessor.next, and successor.prev.</details>

2. **Why is delete-from-end O(1) in DLL but O(n) in SLL?**
   <details><summary>Answer</summary>In DLL, TAIL.prev gives direct access to the second-to-last node. In SLL, there's no backward pointer — you must traverse from HEAD to find it.</details>

3. **When deleting a node in DLL, what two special cases must be checked?**
   <details><summary>Answer</summary>1) Is the node HEAD? (no predecessor — update HEAD). 2) Is the node TAIL? (no successor — update TAIL). Also: is it the only node? (both HEAD and TAIL become NULL).</details>

4. **Why must you set `newNode.next` and `newNode.prev` BEFORE modifying existing nodes' pointers?**
   <details><summary>Answer</summary>Modifying existing nodes first (like givenNode.next = newNode) would overwrite the reference to the successor, losing the link needed for newNode.next and successor.prev.</details>

5. **What is the DLL invariant that must hold after every operation?**
   <details><summary>Answer</summary>For adjacent nodes A and B: `A.next == B` implies `B.prev == A`. HEAD.prev = NULL. TAIL.next = NULL.</details>

6. **Can you insert BEFORE a given node in O(1) in a DLL? Why?**
   <details><summary>Answer</summary>Yes! The given node's `prev` pointer directly accesses the predecessor. Set newNode between predecessor and givenNode, update 4 pointers — all O(1). In SLL, this requires O(n) traversal to find the predecessor.</details>

---

[← Previous: DLL Structure and Advantages](01-dll-structure-and-advantages.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: DLL Traversal and Trade-offs →](03-dll-traversal-and-tradeoffs.md)
