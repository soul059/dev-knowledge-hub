# What is a Stack?

## Overview

A **stack** is a linear data structure that follows a particular order in which operations are performed. It is an ordered collection of items where the addition of new items and removal of existing items always takes place at the same end, called the **top** of the stack.

Think of a stack as a collection of elements with two main operations: **push** (add an element to the top) and **pop** (remove the element from the top).

---

## Visual Representation

```
Stack Visualization:

Empty Stack:          Stack with elements:
┌─────────┐          ┌─────────┐
│         │          │    5    │  ← Top (most recent)
│         │          ├─────────┤
│         │          │    3    │
│         │          ├─────────┤
│         │          │    7    │
│    ⊥    │          ├─────────┤
└─────────┘          │    2    │  ← Bottom (oldest)
                     └─────────┘

After Push(9):       After Pop():
┌─────────┐          ┌─────────┐
│    9    │  ← Top   │    5    │  ← Top
├─────────┤          ├─────────┤
│    5    │          │    3    │
├─────────┤          ├─────────┤
│    3    │          │    7    │
├─────────┤          ├─────────┤
│    7    │          │    2    │
├─────────┤          └─────────┘
│    2    │
└─────────┘
```

---

## Key Characteristics

### 1. **Linear Structure**
- Elements are arranged sequentially
- Each element (except top) has exactly one successor
- Each element (except bottom) has exactly one predecessor

### 2. **Restricted Access**
- Can only access the top element
- Cannot directly access middle or bottom elements
- Must remove elements from top to reach lower ones

### 3. **Dynamic Size** (in most implementations)
- Can grow or shrink as needed
- Size changes with push and pop operations
- May have maximum capacity constraints (array-based)

### 4. **Ordered Collection**
- Maintains insertion order
- Most recently added element is at top
- First added element is at bottom

---

## Stack Operations

### Primary Operations

| Operation | Description | Access Point |
|-----------|-------------|--------------|
| **Push** | Add element to top | Top |
| **Pop** | Remove and return top element | Top |
| **Peek/Top** | View top element without removing | Top |
| **isEmpty** | Check if stack is empty | - |
| **Size** | Get number of elements | - |

### Operation Details

```
1. PUSH Operation:
   Before:           After Push(10):
   ┌─────┐          ┌─────┐
   │  3  │ ← top    │ 10  │ ← top (new)
   ├─────┤          ├─────┤
   │  7  │          │  3  │
   └─────┘          ├─────┤
                    │  7  │
                    └─────┘

2. POP Operation:
   Before:           After Pop():
   ┌─────┐          ┌─────┐
   │ 10  │ ← top    │  3  │ ← top
   ├─────┤          ├─────┤
   │  3  │          │  7  │
   ├─────┤          └─────┘
   │  7  │          Returns: 10
   └─────┘

3. PEEK Operation:
   Stack:            Result:
   ┌─────┐          Returns: 10
   │ 10  │ ← top    (Stack unchanged)
   ├─────┤
   │  3  │
   ├─────┤
   │  7  │
   └─────┘
```

---

## Stack Terminology

| Term | Definition |
|------|------------|
| **Top** | The end where elements are added/removed |
| **Bottom** | The opposite end from top (first element) |
| **Push** | Operation to add an element |
| **Pop** | Operation to remove an element |
| **Peek/Top** | Operation to view top element |
| **Underflow** | Error when trying to pop from empty stack |
| **Overflow** | Error when trying to push to full stack (fixed-size) |
| **Stack Pointer** | Variable/reference pointing to top element |

---

## Example: Stack Operations Sequence

Let's trace a sequence of operations:

```
Operations: Push(5) → Push(3) → Push(8) → Pop() → Push(1) → Pop() → Pop()

Step 1: Initial State
┌─────┐
│  ⊥  │  Empty
└─────┘

Step 2: Push(5)
┌─────┐
│  5  │ ← top
└─────┘

Step 3: Push(3)
┌─────┐
│  3  │ ← top
├─────┤
│  5  │
└─────┘

Step 4: Push(8)
┌─────┐
│  8  │ ← top
├─────┤
│  3  │
├─────┤
│  5  │
└─────┘

Step 5: Pop() → Returns 8
┌─────┐
│  3  │ ← top
├─────┤
│  5  │
└─────┘

Step 6: Push(1)
┌─────┐
│  1  │ ← top
├─────┤
│  3  │
├─────┤
│  5  │
└─────┘

Step 7: Pop() → Returns 1
┌─────┐
│  3  │ ← top
├─────┤
│  5  │
└─────┘

Step 8: Pop() → Returns 3
┌─────┐
│  5  │ ← top
└─────┘

Final State: Stack contains only 5
Pop sequence: 8, 1, 3
```

---

## Why "Stack"?

The name **"stack"** comes from the analogy of a physical stack of items:

```
Physical Stack Examples:

Stack of Books:          Stack of Plates:
    ┌─────────┐              ╱─────────╲
    │ Book 4  │ ← Top        │ Plate 3 │ ← Top
    ├─────────┤              ├─────────┤
    │ Book 3  │              │ Plate 2 │
    ├─────────┤              ├─────────┤
    │ Book 2  │              │ Plate 1 │
    ├─────────┤              ╲─────────╱
    │ Book 1  │ ← Bottom     ← Bottom
    └─────────┘
```

Just like these physical stacks:
- You can only add to the top
- You can only remove from the top
- You cannot remove items from the middle without disturbing the top

---

## Mathematical Representation

A stack **S** can be represented as an ordered list:

**S = (a₁, a₂, a₃, ..., aₙ)**

Where:
- **aₙ** is the top element
- **a₁** is the bottom element
- **n** is the number of elements (size)

Operations:
- **Push(x)**: S = (a₁, a₂, ..., aₙ, x)
- **Pop()**: S = (a₁, a₂, ..., aₙ₋₁), returns aₙ
- **Top()**: returns aₙ (S unchanged)
- **isEmpty()**: returns true if n = 0

---

## Stack State Diagram

```
State Transition Diagram:

                    Push(x)
    ┌───────────────────────────────────┐
    │                                   │
    │                                   ▼
┌────────┐  Push(x)  ┌────────┐  Push(x)  ┌────────┐
│ Empty  │──────────▶│  Has   │──────────▶│  Full  │
│ Stack  │           │Elements│           │ Stack  │
└────────┘           └────────┘           └────────┘
    ▲                    │  ▲                  │
    │                    │  │                  │
    │      Pop()         │  │     Pop()        │
    └────────────────────┘  └──────────────────┘
                         Pop()

States:
- Empty: size = 0
- Has Elements: 0 < size < capacity
- Full: size = capacity (for fixed-size stacks)
```

---

## Distinguishing Features

### Stack vs. Array

| Feature | Stack | Array |
|---------|-------|-------|
| **Access** | Only top element | Any element by index |
| **Order** | LIFO | Random access |
| **Operations** | Push, Pop at one end | Insert, Delete anywhere |
| **Use Case** | Sequential processing | General storage |

### Stack vs. Queue

| Feature | Stack | Queue |
|---------|-------|-------|
| **Principle** | LIFO (Last In, First Out) | FIFO (First In, First Out) |
| **Operations** | Push/Pop at same end (top) | Enqueue at rear, Dequeue at front |
| **Ends** | One active end | Two active ends |
| **Example** | Undo operation | Print queue |

---

## Core Properties

### 1. **Order Preservation**
```
Input Order:  1, 2, 3, 4, 5
Push All:     Stack = [1, 2, 3, 4, 5] (5 at top)
Pop All:      Output = 5, 4, 3, 2, 1
Result:       Reversed order
```

### 2. **Atomicity**
- Each operation completes fully or not at all
- No partial push or pop operations
- Stack maintains consistent state

### 3. **Memory Efficiency**
- Only top element needs to be tracked
- No need to maintain pointers to all elements
- Compact representation possible

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Type** | Linear, Abstract Data Type |
| **Principle** | LIFO (Last In, First Out) |
| **Primary Operations** | Push, Pop, Peek |
| **Access Pattern** | Restricted to top only |
| **Typical Use** | Reversing, backtracking, function calls |
| **Time Complexity** | O(1) for all basic operations |
| **Space Complexity** | O(n) where n = number of elements |
| **Variants** | Array-based, Linked list-based, Dynamic |

---

## Quick Revision Questions

1. **Q: What is the defining characteristic of a stack data structure?**
   - A: LIFO (Last In, First Out) principle - elements can only be added or removed from the top.

2. **Q: Can you access the middle element of a stack directly without popping elements?**
   - A: No, you can only access the top element. To reach middle elements, you must pop all elements above it.

3. **Q: What happens when you try to pop from an empty stack?**
   - A: It causes an underflow error/exception, as there are no elements to remove.

4. **Q: If you push elements 2, 5, 7, 3 (in this order), what will be the first element popped?**
   - A: 3 (the last element pushed, following LIFO principle).

5. **Q: What is the difference between Pop() and Peek() operations?**
   - A: Pop() removes and returns the top element; Peek() only returns the top element without removing it.

6. **Q: Why is a stack called a "restricted" data structure?**
   - A: Because access is restricted to only one end (the top), unlike arrays where any element can be accessed.

---

## Navigation

- **[← Back to README](../README.md)**
- **[Next: LIFO Principle →](02-lifo-principle.md)**
- **[↑ Back to Unit 1](README.md)**

---

*Master this concept before moving forward - understanding what a stack is forms the foundation for all stack-based problem solving!*
