# Chapter 3: Stack with Middle Operations

[← Previous: Max Stack](02-max-stack.md) | [Next: Celebrity Problem →](04-celebrity-problem.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

---

## Overview

Design a stack that supports finding and deleting the **middle element** in O(1) time. This requires a creative combination of a doubly linked list and a middle pointer.

---

## Problem Statement

```
Design a stack with these operations, ALL in O(1):
  push(x)      — Push element x
  pop()        — Remove and return top element
  getMiddle()  — Return middle element
  deleteMiddle()— Remove middle element

Middle definition:
  Odd size:  exact middle
  Even size: lower of two middles (n/2 - 1 from top, 0-indexed)

Example:
  Stack (top→bottom): [7, 5, 3, 1]
  Size = 4 (even) → middle = element at position 1 = 5
  
  Stack (top→bottom): [9, 7, 5, 3, 1]
  Size = 5 (odd) → middle = element at position 2 = 5
```

---

## Key Insight

```
┌──────────────────────────────────────────────────────────┐
│  A regular stack (array/linked list) needs O(n) to      │
│  find the middle.                                        │
│                                                          │
│  Solution: Doubly Linked List + Middle Pointer           │
│                                                          │
│  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐                │
│  │ 9 │⟷│ 7 │⟷│ 5 │⟷│ 3 │⟷│ 1 │           │
│  └───┘   └───┘   └───┘   └───┘   └───┘                │
│   top              ↑ mid                 bottom         │
│                                                          │
│  The middle pointer moves by 1 on every push/pop/delete │
│  to stay centered. O(1) adjustment each time!           │
└──────────────────────────────────────────────────────────┘
```

---

## Data Structure

```
CLASS Node:
    val   ← value
    prev  ← pointer to previous node (toward top)
    next  ← pointer to next node (toward bottom)

CLASS MiddleStack:
    top    ← null    // Top of stack (head of DLL)
    mid    ← null    // Middle pointer
    size   ← 0
```

---

## Algorithm

### Push

```
FUNCTION push(x):
    newNode ← Node(x)
    newNode.next ← top
    IF top != null:
        top.prev ← newNode
    top ← newNode
    size ← size + 1
    
    IF size == 1:
        mid ← newNode
    ELSE IF size is even:
        // Middle moves UP (toward top) when size becomes even
        mid ← mid.prev
```

### Pop

```
FUNCTION pop():
    IF size == 0: ERROR
    val ← top.val
    top ← top.next
    IF top != null:
        top.prev ← null
    size ← size - 1
    
    IF size == 0:
        mid ← null
    ELSE IF size is odd:
        // Middle moves DOWN (toward bottom) when size becomes odd
        mid ← mid.next
    
    RETURN val
```

### getMiddle and deleteMiddle

```
FUNCTION getMiddle():
    IF mid == null: ERROR
    RETURN mid.val

FUNCTION deleteMiddle():
    IF size == 0: ERROR
    val ← mid.val
    
    IF size == 1:
        top ← null
        mid ← null
    ELSE IF size is even:
        // After deletion (odd size), mid moves DOWN
        newMid ← mid.next
        removeNode(mid)
        mid ← newMid
    ELSE:
        // After deletion (even size), mid moves UP
        newMid ← mid.prev
        removeNode(mid)
        mid ← newMid
    
    size ← size - 1
    RETURN val

FUNCTION removeNode(node):
    IF node.prev != null:
        node.prev.next ← node.next
    IF node.next != null:
        node.next.prev ← node.prev
    IF node == top:
        top ← node.next
```

---

## Detailed Trace

```
═══ push(1) ═══
  DLL: [1]    mid→1    size=1
       ↑top ↑mid

═══ push(3) ═══
  DLL: [3] ⟷ [1]    size=2 (even) → mid moves up
       ↑top  ↑old_mid
  mid → 3 (moved to top/prev)
  
  DLL: [3] ⟷ [1]
       ↑top
       ↑mid

═══ push(5) ═══
  DLL: [5] ⟷ [3] ⟷ [1]    size=3 (odd) → mid stays
       ↑top   ↑mid

═══ push(7) ═══
  DLL: [7] ⟷ [5] ⟷ [3] ⟷ [1]    size=4 (even) → mid moves up
       ↑top         ↑old_mid
  mid → 5
  
  DLL: [7] ⟷ [5] ⟷ [3] ⟷ [1]
       ↑top   ↑mid

═══ push(9) ═══
  DLL: [9] ⟷ [7] ⟷ [5] ⟷ [3] ⟷ [1]    size=5 (odd) → mid stays
       ↑top         ↑mid

═══ getMiddle() → 5 ✓ ═══

═══ deleteMiddle() ═══
  Remove node 5. size was 5 (odd) → mid moves UP
  newMid = 5.prev = 7
  DLL: [9] ⟷ [7] ⟷ [3] ⟷ [1]    size=4
       ↑top   ↑mid

═══ getMiddle() → 7 ✓ ═══

═══ pop() → 9 ═══
  size 4→3 (odd) → mid moves DOWN
  mid = 7.next = 3
  DLL: [7] ⟷ [3] ⟷ [1]    size=3
       ↑top   ↑mid

═══ getMiddle() → 3 ✓ ═══
```

---

## Middle Pointer Movement Rules

```
┌────────────────────────────────────┬──────────────────────┐
│ Operation                          │ Middle Pointer Move  │
├────────────────────────────────────┼──────────────────────┤
│ push (size becomes even)           │ Move UP (→ prev)     │
│ push (size becomes odd)            │ Stay                 │
│ pop (size becomes odd)             │ Move DOWN (→ next)   │
│ pop (size becomes even)            │ Stay                 │
│ deleteMiddle (was even count)      │ Move DOWN (→ next)   │
│ deleteMiddle (was odd count)       │ Move UP (→ prev)     │
└────────────────────────────────────┴──────────────────────┘

Mnemonic: When size INCREASES to even → more room above → mid goes UP
          When size DECREASES to odd → less room above → mid goes DOWN
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| `push` | O(1) | O(1) per element |
| `pop` | O(1) | — |
| `getMiddle` | O(1) | — |
| `deleteMiddle` | O(1) | — |
| **Total space** | — | O(n) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Stack with O(1) middle access |
| **Data Structure** | Doubly Linked List + middle pointer |
| **Key Trick** | Adjust middle pointer by 1 on each operation |
| **All operations** | O(1) |
| **Space** | O(n) |

---

## Quick Revision Questions

1. **Why can't an array-based stack support O(1) deleteMiddle?**
   > Deleting from the middle of an array requires shifting elements, which is O(n).

2. **Why use a doubly linked list?**
   > DLL allows O(1) deletion of any node (given a pointer) and O(1) movement in both directions.

3. **When does the middle pointer move toward the top?**
   > After push when size becomes even, or after deleteMiddle when size was odd.

4. **For stack [10, 8, 6, 4, 2] (top to bottom), what is the middle?**
   > 6 (position 2 in a 0-indexed 5-element stack).

5. **What happens if we deleteMiddle from a stack of size 2?**
   > The lower element is removed (it's the "middle" for even-sized stacks), leaving a stack of size 1.

---

[← Previous: Max Stack](02-max-stack.md) | [Next: Celebrity Problem →](04-celebrity-problem.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)
