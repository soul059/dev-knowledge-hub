# Chapter 1: Insertion Operations

[← Previous: Array vs Linked List](../01-Singly-Linked-List-Basics/03-array-vs-linked-list.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Deletion and Search →](02-deletion-and-search.md)

---

## Chapter Overview

Insertion is one of the core linked list operations. This chapter covers inserting nodes at the **beginning**, **end**, and **middle** of a singly linked list, with complete pseudocode, step-by-step traces, and diagrams.

---

## 1.1 Insert at the Beginning (Prepend)

The simplest and most efficient insertion — O(1) time.

### Algorithm

```
function insertAtBeginning(HEAD, value):
    newNode = createNode(value)      ← Step 1: Create new node
    newNode.next = HEAD              ← Step 2: Point new node to current HEAD
    HEAD = newNode                   ← Step 3: Update HEAD to new node
    return HEAD
```

### Step-by-Step Trace: Insert 5 at Beginning

**Before:**
```
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Step 1:** Create new node with data = 5
```
  newNode
   │
   ▼
 [ 5|NULL]

  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Step 2:** `newNode.next = HEAD`
```
  newNode
   │
   ▼
 [ 5] ─┐
        │     HEAD
        │      │
        ▼      ▼
       [10] ──► [20] ──► [30] ──► NULL
```

**Step 3:** `HEAD = newNode`
```
  HEAD
   │
   ▼
 [ 5] ──► [10] ──► [20] ──► [30] ──► NULL
```

### Edge Case: Insert into Empty List

```
  Before:  HEAD ──► NULL

  Step 1:  newNode = [ 5|NULL]
  Step 2:  newNode.next = HEAD = NULL   → [ 5|NULL]  (same as before)
  Step 3:  HEAD = newNode

  After:   HEAD ──► [ 5|NULL]

  ✓ Works correctly — no special handling needed!
```

### Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(1) | Just pointer reassignment, no traversal |
| Space | O(1) | Only the new node is allocated |

---

## 1.2 Insert at the End (Append)

### Algorithm (Without TAIL Pointer)

```
function insertAtEnd(HEAD, value):
    newNode = createNode(value)
    
    if HEAD == NULL:                 ← Empty list case
        HEAD = newNode
        return HEAD
    
    current = HEAD
    while current.next ≠ NULL:      ← Traverse to last node
        current = current.next
    
    current.next = newNode           ← Link last node to new node
    return HEAD
```

### Step-by-Step Trace: Insert 40 at End

**Before:**
```
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL
```

**Traverse to last node:**
```
  Step 1: current = HEAD = Node(10)
          current.next = Node(20) ≠ NULL → move

  Step 2: current = Node(20)
          current.next = Node(30) ≠ NULL → move

  Step 3: current = Node(30)
          current.next = NULL → STOP (this is the last node)
```

**Link new node:**
```
  current.next = newNode

  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► [40] ──► NULL
                              ↑
                           newNode
```

### Algorithm (With TAIL Pointer) — O(1) Version

```
function insertAtEnd(HEAD, TAIL, value):
    newNode = createNode(value)
    
    if HEAD == NULL:                 ← Empty list
        HEAD = newNode
        TAIL = newNode
    else:
        TAIL.next = newNode          ← Link directly using TAIL
        TAIL = newNode               ← Update TAIL
    
    return HEAD, TAIL
```

### Complexity

| Version | Time | Space |
|---------|------|-------|
| Without TAIL | O(n) | O(1) |
| With TAIL | O(1) | O(1) |

---

## 1.3 Insert at a Given Position

### Algorithm

```
function insertAtPosition(HEAD, value, position):
    newNode = createNode(value)
    
    if position == 0:                    ← Insert at beginning
        newNode.next = HEAD
        HEAD = newNode
        return HEAD
    
    current = HEAD
    for i = 0 to position - 2:          ← Traverse to node BEFORE target
        if current == NULL:
            error("Position out of bounds")
        current = current.next
    
    if current == NULL:
        error("Position out of bounds")
    
    newNode.next = current.next          ← Step 1: New node points to next
    current.next = newNode               ← Step 2: Previous node points to new
    return HEAD
```

### Step-by-Step Trace: Insert 25 at Position 2

**Before:**
```
  Position:  0      1      2
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL

  We want to insert 25 at position 2 (between 20 and 30)
```

**Traverse to position - 1 = 1:**
```
  i = 0: current = HEAD = Node(10)
         current = current.next = Node(20)
  
  Now current is at position 1 (the node BEFORE position 2)
```

**Step 1:** `newNode.next = current.next`
```
                  current
                    │
                    ▼
  [10] ──► [20] ──► [30] ──► NULL
                ↗
         [25] ─┘    ← newNode.next = Node(30)
```

**Step 2:** `current.next = newNode`
```
  HEAD
   │
   ▼
  [10] ──► [20] ──► [25] ──► [30] ──► NULL
                      ↑
                   newNode
```

### Why Order Matters!

```
  ❌ WRONG ORDER:
     current.next = newNode          ← Breaks link to rest of list!
     newNode.next = current.next     ← current.next is now newNode, INFINITE LOOP!

  ✓ CORRECT ORDER:
     newNode.next = current.next     ← Save the link first
     current.next = newNode          ← Then redirect
```

This pattern is called **"link before you break"**.

### Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(n) | Traversal to position takes O(n) |
| Space | O(1) | Only one new node + one pointer |

---

## 1.4 Insert After a Given Node

When you already have a reference to a node (not just a position):

### Algorithm

```
function insertAfter(givenNode, value):
    if givenNode == NULL:
        error("Given node is NULL")
    
    newNode = createNode(value)
    newNode.next = givenNode.next        ← New node points to what was next
    givenNode.next = newNode             ← Given node points to new node
```

### Diagram

```
  Before:
  ... ──► [A] ──► [C] ──► ...
           ↑
        givenNode

  Step 1: newNode.next = givenNode.next
  ... ──► [A] ──► [C] ──► ...
                  ↗
           [B] ──┘

  Step 2: givenNode.next = newNode
  ... ──► [A] ──► [B] ──► [C] ──► ...
```

**Complexity:** O(1) — no traversal needed if you have the node reference.

---

## 1.5 Insert to Maintain Sorted Order

### Algorithm

```
function insertSorted(HEAD, value):
    newNode = createNode(value)
    
    // Case 1: Empty list or insert before HEAD
    if HEAD == NULL or value ≤ HEAD.data:
        newNode.next = HEAD
        HEAD = newNode
        return HEAD
    
    // Case 2: Find correct position
    current = HEAD
    while current.next ≠ NULL and current.next.data < value:
        current = current.next
    
    newNode.next = current.next
    current.next = newNode
    return HEAD
```

### Step-by-Step Trace: Insert 25 into Sorted List

**Before:** `10 → 20 → 30 → 40 → NULL`

```
  current = Node(10)
    current.next.data = 20 < 25 → move
  
  current = Node(20)
    current.next.data = 30 < 25? NO → STOP
  
  Insert after current (Node 20):
    newNode.next = Node(30)
    current.next = newNode
  
  After: 10 → 20 → 25 → 30 → 40 → NULL  ✓
```

---

## 1.6 Visual Summary of All Insertions

```
  INSERT AT BEGINNING (O(1)):
  ─────────────────────────
  [new] ──► [A] ──► [B] ──► [C] ──► NULL
    ↑
   HEAD


  INSERT AT END (O(n) or O(1) with tail):
  ────────────────────────────────────────
  [A] ──► [B] ──► [C] ──► [new] ──► NULL


  INSERT AT POSITION k:
  ─────────────────────
  [A] ──► [B] ──► [new] ──► [C] ──► NULL
                    ↑
               position k


  INSERT SORTED:
  ──────────────
  [10] ──► [20] ──► [25] ──► [30] ──► NULL
                      ↑
                  correct spot
```

---

## Summary Table

| Operation | Time | Space | Key Insight |
|-----------|------|-------|-------------|
| Insert at beginning | O(1) | O(1) | Just change HEAD pointer |
| Insert at end (no tail) | O(n) | O(1) | Must traverse to find last node |
| Insert at end (with tail) | O(1) | O(1) | TAIL gives direct access to last node |
| Insert at position k | O(k) | O(1) | Traverse to k-1, then relink |
| Insert after a node | O(1) | O(1) | Direct pointer manipulation |
| Insert sorted | O(n) | O(1) | Find correct spot, then insert |

### Critical Rule: "Link Before You Break"

Always set `newNode.next` BEFORE modifying any existing pointers. Otherwise, you'll lose access to the rest of the list.

---

## Quick Revision Questions

1. **What is the time complexity of inserting at the beginning of a singly linked list?**
   <details><summary>Answer</summary>O(1). Just create a new node, point it to the current HEAD, and update HEAD.</details>

2. **Why must you set `newNode.next = current.next` BEFORE `current.next = newNode`?**
   <details><summary>Answer</summary>If you set `current.next = newNode` first, you lose the reference to the rest of the list. `newNode.next` would then point back to newNode (through current.next), creating a loop.</details>

3. **How does a TAIL pointer improve insertion at the end?**
   <details><summary>Answer</summary>Without TAIL, you must traverse the entire list — O(n). With TAIL, you directly access the last node — O(1).</details>

4. **What happens when you insert at position 0?**
   <details><summary>Answer</summary>It's equivalent to inserting at the beginning. The new node becomes the new HEAD of the list.</details>

5. **How would you insert a node at position `k` in a list of length `n` where `k > n`?**
   <details><summary>Answer</summary>This is an out-of-bounds error. You should either throw an error, or insert at the end depending on the specification.</details>

6. **Can you insert a node in O(1) time at any arbitrary position if you have a pointer to that position?**
   <details><summary>Answer</summary>You can insert AFTER the given node in O(1). To insert BEFORE, you'd need a pointer to the previous node (which requires O(n) traversal in a singly linked list). A trick: insert after and swap data to simulate insert-before in O(1).</details>

---

[← Previous: Array vs Linked List](../01-Singly-Linked-List-Basics/03-array-vs-linked-list.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Deletion and Search →](02-deletion-and-search.md)
