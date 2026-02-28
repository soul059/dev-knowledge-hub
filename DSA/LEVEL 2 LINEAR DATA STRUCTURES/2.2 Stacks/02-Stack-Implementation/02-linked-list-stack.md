# Linked List-Based Stack Implementation

## Overview

A **linked list-based stack** uses a singly linked list as the underlying storage structure. Unlike array-based stacks, linked list stacks can grow and shrink dynamically without predetermined size limits, making them ideal when the maximum size is unknown.

---

## Core Concept

```
Linked List Stack Structure:

       Top Pointer
           ↓
       ┌───────┐    ┌───────┐    ┌───────┐    ┌───────┐
       │ Data:5│ →  │ Data:3│ →  │ Data:7│ →  │ Data:2│ → NULL
       │ Next ─┼────│ Next ─┼────│ Next ─┼────│ Next ─┼──X
       └───────┘    └───────┘    └───────┘    └───────┘
       (Top/         (2nd)        (3rd)        (Bottom)
        Most Recent)                           Oldest)

Key Difference from Array:
• No fixed capacity
• Nodes allocated dynamically
• Top is a pointer, not an index
• NULL indicates empty stack
```

---

## Node Structure

### Node Design

```
Node Structure:
┌─────────────┐
│ data: T     │ ← Stores element
├─────────────┤
│ next: Node* │ ← Points to next node
└─────────────┘

Pseudocode:
CLASS Node:
    DATA:
        data: DataType
        next: Node pointer

    CONSTRUCTOR Node(value):
        data = value
        next = NULL
```

### Visual Node

```
Single Node:
    ┌──────────┐
    │ data: 42 │
    │ next: •─────→ NULL
    └──────────┘

Linked Nodes:
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ data: 10 │    │ data: 20 │    │ data: 30 │
    │ next: •─────→ │ next: •─────→ │ next: •─────→ NULL
    └──────────┘    └──────────┘    └──────────┘
```

---

## Stack Structure Design

### Complete Stack

```
Stack Class:
┌─────────────────┐
│ top: Node*      │ ← Points to top node (NULL if empty)
│ size: integer   │ ← Tracks number of elements (optional)
└─────────────────┘

Empty Stack:
top = NULL
size = 0
┌─────┐
│ top │ → NULL
└─────┘

Non-Empty Stack:
top points to first node
size = 4
┌─────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
│ top │ → │  50  │ → │  40  │ → │  30  │ → │  20  │ → NULL
└─────┘   └──────┘   └──────┘   └──────┘   └──────┘
          (Latest)                          (Oldest)
```

---

## Operations Implementation

### 1. Initialize (Constructor)

```
INITIALIZE():
    top = NULL
    size = 0

Visual:
Before:               After Initialize():
Nothing              top = NULL, size = 0
                     ┌─────┐
                     │ top │ → NULL
                     └─────┘
                     Empty stack

Time Complexity: O(1)
Space Complexity: O(1)
```

### 2. Push Operation

```
PUSH(item):
    // Create new node
    newNode = new Node(item)
    
    // Link new node to current top
    newNode.next = top
    
    // Update top to new node
    top = newNode
    
    // Increment size
    size = size + 1

Step-by-Step Trace: Push(42)

Before Push:
top
 ↓
┌──────┐   ┌──────┐   ┌──────┐
│  30  │ → │  20  │ → │  10  │ → NULL
└──────┘   └──────┘   └──────┘

Step 1: Create new node
        ┌──────┐
        │  42  │
        │ next:│ → NULL
        └──────┘

Step 2: Link to current top
        ┌──────┐
        │  42  │
        │ next:│ ─┐
        └──────┘  │
                  ↓
top              ┌──────┐   ┌──────┐   ┌──────┐
 ↓               │  30  │ → │  20  │ → │  10  │ → NULL
                 └──────┘   └──────┘   └──────┘

Step 3: Update top pointer
top
 ↓
┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
│  42  │ → │  30  │ → │  20  │ → │  10  │ → NULL
└──────┘   └──────┘   └──────┘   └──────┘

Time Complexity: O(1) - constant time
Space Complexity: O(1) - one new node
```

### 3. Pop Operation

```
POP():
    IF top = NULL THEN
        ERROR "Stack Underflow"
    END IF
    
    // Save data to return
    item = top.data
    
    // Save reference to node being removed
    temp = top
    
    // Move top to next node
    top = top.next
    
    // Deallocate removed node
    DELETE temp
    
    // Decrement size
    size = size - 1
    
    RETURN item

Step-by-Step Trace: Pop()

Before Pop:
top
 ↓
┌──────┐   ┌──────┐   ┌──────┐
│  42  │ → │  30  │ → │  20  │ → NULL
└──────┘   └──────┘   └──────┘

Step 1: Save data
item = 42

Step 2: Save temp reference
temp
 ↓
┌──────┐   ┌──────┐   ┌──────┐
│  42  │ → │  30  │ → │  20  │ → NULL
└──────┘   └──────┘   └──────┘
 ↑
top

Step 3: Move top
        top
         ↓
┌──────┐   ┌──────┐   ┌──────┐
│  42  │ → │  30  │ → │  20  │ → NULL
└──────┘   └──────┘   └──────┘
 ↑
temp

Step 4: Delete temp node
top
 ↓
┌──────┐   ┌──────┐
│  30  │ → │  20  │ → NULL
└──────┘   └──────┘
(42 deleted)

Returns: 42
Time Complexity: O(1)
Space Complexity: O(1)
```

### 4. Peek Operation

```
PEEK():
    IF top = NULL THEN
        ERROR "Stack Empty"
    END IF
    
    RETURN top.data

Visual:
top
 ↓
┌──────┐   ┌──────┐   ┌──────┐
│  42  │ → │  30  │ → │  20  │ → NULL
└──────┘   └──────┘   └──────┘

Returns: 42 (stack unchanged)

Time Complexity: O(1)
Space Complexity: O(1)
```

### 5. isEmpty Operation

```
isEmpty():
    RETURN (top = NULL)

Examples:
Empty:                   Non-Empty:
top = NULL              top → Node
┌─────┐                 ┌─────┐   ┌──────┐
│ top │ → NULL          │ top │ → │  42  │ → NULL
└─────┘                 └─────┘   └──────┘

isEmpty() = true        isEmpty() = false

Time Complexity: O(1)
Space Complexity: O(1)
```

### 6. Size Operation

```
SIZE():
    RETURN size

Alternative (without size variable):
SIZE():
    count = 0
    current = top
    WHILE current ≠ NULL:
        count = count + 1
        current = current.next
    RETURN count

With size variable: O(1)
Without size variable: O(n) - must traverse list
```

---

## Complete Implementation

### Pseudocode

```pseudocode
CLASS Node:
    DATA:
        data: DataType
        next: Node pointer

    CONSTRUCTOR Node(value):
        data = value
        next = NULL

CLASS LinkedStack:
    PRIVATE:
        top: Node pointer
        size: integer

    PUBLIC LinkedStack():
        top = NULL
        size = 0

    PUBLIC PUSH(item):
        // Create new node
        newNode = new Node(item)
        
        // Link to current top
        newNode.next = top
        
        // Update top
        top = newNode
        
        // Increment size
        size = size + 1

    PUBLIC POP():
        // Check empty
        IF isEmpty() THEN
            PRINT "Stack Underflow"
            RETURN null
        END IF

        // Save data
        item = top.data
        
        // Save temp reference
        temp = top
        
        // Move top
        top = top.next
        
        // Delete old top
        DELETE temp
        
        // Decrement size
        size = size - 1
        
        RETURN item

    PUBLIC PEEK():
        IF isEmpty() THEN
            PRINT "Stack Empty"
            RETURN null
        END IF
        
        RETURN top.data

    PUBLIC isEmpty():
        RETURN (top = NULL)

    PUBLIC SIZE():
        RETURN size

    PUBLIC DISPLAY():
        IF isEmpty() THEN
            PRINT "Stack is empty"
            RETURN
        END IF

        PRINT "Stack (top to bottom):"
        current = top
        WHILE current ≠ NULL:
            PRINT current.data
            current = current.next
        END WHILE

    PUBLIC DESTRUCTOR():
        // Free all nodes
        WHILE NOT isEmpty():
            POP()
        END WHILE
END CLASS
```

---

## Detailed Operation Trace

### Complete Example Sequence

```
Operations: Stack() → Push(10) → Push(20) → Push(30) → Pop() → 
            Push(40) → Peek() → Size()

┌────────────────────────────────────────────────────────────┐
│ Step 1: Initialize Stack()                                │
├────────────────────────────────────────────────────────────┤
│ top = NULL, size = 0                                       │
│ ┌─────┐                                                    │
│ │ top │ → NULL                                             │
│ └─────┘                                                    │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 2: Push(10)                                           │
├────────────────────────────────────────────────────────────┤
│ top, size = 1                                              │
│  ↓                                                         │
│ ┌──────┐                                                   │
│ │  10  │ → NULL                                            │
│ └──────┘                                                   │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 3: Push(20)                                           │
├────────────────────────────────────────────────────────────┤
│ top, size = 2                                              │
│  ↓                                                         │
│ ┌──────┐   ┌──────┐                                       │
│ │  20  │ → │  10  │ → NULL                                │
│ └──────┘   └──────┘                                       │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 4: Push(30)                                           │
├────────────────────────────────────────────────────────────┤
│ top, size = 3                                              │
│  ↓                                                         │
│ ┌──────┐   ┌──────┐   ┌──────┐                           │
│ │  30  │ → │  20  │ → │  10  │ → NULL                    │
│ └──────┘   └──────┘   └──────┘                           │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 5: Pop() → Returns 30                                │
├────────────────────────────────────────────────────────────┤
│ top, size = 2                                              │
│  ↓                                                         │
│ ┌──────┐   ┌──────┐                                       │
│ │  20  │ → │  10  │ → NULL                                │
│ └──────┘   └──────┘                                       │
│ (30's node deleted)                                        │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 6: Push(40)                                           │
├────────────────────────────────────────────────────────────┤
│ top, size = 3                                              │
│  ↓                                                         │
│ ┌──────┐   ┌──────┐   ┌──────┐                           │
│ │  40  │ → │  20  │ → │  10  │ → NULL                    │
│ └──────┘   └──────┘   └──────┘                           │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 7: Peek() → Returns 40 (stack unchanged)             │
├────────────────────────────────────────────────────────────┤
│ top, size = 3                                              │
│  ↓                                                         │
│ ┌──────┐   ┌──────┐   ┌──────┐                           │
│ │  40  │ → │  20  │ → │  10  │ → NULL                    │
│ └──────┘   └──────┘   └──────┘                           │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 8: Size() → Returns 3                                │
├────────────────────────────────────────────────────────────┤
│ Final State: 3 elements                                    │
│ [40 → 20 → 10]                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Advantages of Linked List Stack

```
✅ Advantages:

1. Dynamic Size
   • No predetermined capacity needed
   • Grows and shrinks as needed
   • No overflow (until system memory exhausted)

2. Memory Efficiency
   • Only allocates what's needed
   • No wasted space
   • Memory returned on pop

3. No Size Limit (theoretical)
   • Limited only by system memory
   • Can grow very large
   • No hard capacity constraint

4. Flexible
   • Easy to extend functionality
   • Can add features without array constraints
   • Natural for recursive structures

Visual:
Start:         After 1M pushes:
┌─────┐        ┌─────┐   ┌────┐       ┌────┐
│ top │→NULL   │ top │ → │ N1 │ → ... │ N1M│ → NULL
└─────┘        └─────┘   └────┘       └────┘
(No difference in structure, just more nodes)
```

---

## Disadvantages of Linked List Stack

```
❌ Disadvantages:

1. Extra Memory Per Element
   • Each node needs next pointer
   • Overhead per element
   • More total memory than array

2. No Cache Locality
   • Nodes scattered in memory
   • Poor cache performance
   • Slower access than array

3. No Random Access
   • Must traverse from top
   • Cannot jump to position
   • O(n) to access nth element

4. Pointer Management
   • Memory leaks if not careful
   • More complex implementation
   • Need to free nodes properly

Memory Comparison:
Array Stack (100 ints):
400 bytes (just data)

Linked Stack (100 nodes):
100 × (4 bytes data + 8 bytes pointer) = 1200 bytes
3× more memory!
```

---

## Memory Layout

```
Memory Diagram:

Stack Object (on stack/heap):
┌──────────────┐ Address: 1000
│ top: 5000    │ (pointer, 8 bytes)
├──────────────┤
│ size: 3      │ (integer, 4 bytes)
└──────────────┘

Node 1 (on heap):
┌──────────────┐ Address: 5000
│ data: 40     │ (4 bytes)
├──────────────┤
│ next: 5020   │ (8 bytes)
└──────────────┘

Node 2 (on heap):
┌──────────────┐ Address: 5020
│ data: 20     │ (4 bytes)
├──────────────┤
│ next: 5040   │ (8 bytes)
└──────────────┘

Node 3 (on heap):
┌──────────────┐ Address: 5040
│ data: 10     │ (4 bytes)
├──────────────┤
│ next: 0      │ (NULL, 8 bytes)
└──────────────┘

Fragmentation:
Nodes may not be contiguous in memory
(unlike array-based stack)
```

---

## Edge Cases and Error Handling

### 1. Pop from Empty Stack

```
Scenario:
top = NULL
┌─────┐
│ top │ → NULL
└─────┘

Pop():
ERROR "Stack Underflow"

Safe Implementation:
POP():
    IF top = NULL THEN
        RETURN error / THROW exception
    END IF
    // ... rest of pop logic
```

### 2. Memory Leak Prevention

```
Incorrect Pop (Memory Leak):
POP():
    item = top.data
    top = top.next  // Old node lost!
    RETURN item

Correct Pop (No Leak):
POP():
    item = top.data
    temp = top
    top = top.next
    DELETE temp     // Free memory!
    RETURN item

Without DELETE:
┌──────┐
│  42  │  ← Orphaned! Memory leak
└──────┘
```

### 3. Destructor

```
Proper Cleanup:
DESTRUCTOR():
    WHILE top ≠ NULL:
        temp = top
        top = top.next
        DELETE temp
    END WHILE

Must free all nodes before stack destroyed!
```

---

## Comparison: Array vs Linked List Stack

| Aspect | Array Stack | Linked List Stack |
|--------|-------------|-------------------|
| **Size** | Fixed | Dynamic |
| **Overflow** | Possible | Only when memory full |
| **Memory Per Element** | Just data | Data + pointer |
| **Cache Performance** | Excellent | Poor |
| **Random Access** | Possible | Difficult |
| **Implementation** | Simpler | More complex |
| **Push/Pop Time** | O(1) | O(1) |
| **Space Efficiency** | Better (no pointers) | Worse (pointers overhead) |
| **Flexibility** | Limited | High |
| **Best For** | Known max size | Unknown size |

---

## Complexity Analysis

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| **Initialize** | O(1) | O(1) | Set top to NULL |
| **Push** | O(1) | O(1) per push | Allocate one node |
| **Pop** | O(1) | O(1) | Deallocate one node |
| **Peek** | O(1) | O(1) | Access top.data |
| **isEmpty** | O(1) | O(1) | Check if top is NULL |
| **Size** | O(1) | O(1) | If size variable maintained |
| **Size** | O(n) | O(1) | If must count nodes |
| **Destructor** | O(n) | O(1) | Free all n nodes |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Storage** | Singly linked list |
| **Top Tracking** | Pointer to first node |
| **Size Limit** | System memory |
| **Time Complexity** | O(1) for push, pop, peek |
| **Space Per Element** | Data + pointer |
| **Memory Efficiency** | Only allocates what's needed |
| **Cache Performance** | Poor (fragmented memory) |
| **Best For** | Unknown/dynamic size needs |
| **Major Advantage** | No size limit, no overflow |
| **Major Disadvantage** | Pointer overhead, cache misses |

---

## Quick Revision Questions

1. **Q: Why does push always insert at the beginning (top) of the linked list?**
   - A: Inserting at the beginning is O(1); inserting at the end would require O(n) traversal.

2. **Q: What happens if you don't free the popped node?**
   - A: Memory leak - the node's memory remains allocated but inaccessible, wasting memory.

3. **Q: How is an empty stack represented in linked list implementation?**
   - A: top pointer is NULL.

4. **Q: What is the extra space overhead per element compared to array stack?**
   - A: One pointer (typically 8 bytes on 64-bit systems) per element for the `next` reference.

5. **Q: Can a linked list stack overflow?**
   - A: Theoretically no (until system memory exhausted), unlike fixed-size array stack.

6. **Q: Why is linked list stack slower despite O(1) operations?**
   - A: Poor cache locality - nodes are scattered in memory, causing more cache misses than contiguous array.

---

## Navigation

- **[← Previous: Array-Based Stack](01-array-based-stack.md)**
- **[Next: Dynamic Array Stack →](03-dynamic-array-stack.md)**
- **[↑ Back to Unit 2](README.md)**
- **[⌂ Home](../README.md)**

---

*Linked list stacks offer flexibility at the cost of complexity - choose based on your needs!*
