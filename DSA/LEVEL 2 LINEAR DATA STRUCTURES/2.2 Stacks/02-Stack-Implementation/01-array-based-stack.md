# Array-Based Stack Implementation

## Overview

An **array-based stack** uses a fixed-size or dynamic array as the underlying storage structure. This is the most straightforward and commonly taught stack implementation, offering constant-time operations and excellent cache locality.

---

## Core Concept

```
Array-Based Stack Structure:

┌─────────────────────────────────────────┐
│         Array (Fixed Size)              │
├──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┤
│50│30│20│10│  │  │  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
 ↑           ↑                          ↑
 0           3                         13
           (top)                   (capacity-1)

Components:
1. Array: stores elements
2. Top: index of top element (-1 if empty)
3. Capacity: maximum size of array
```

---

## Data Structure Design

### Basic Structure

```
Stack Structure:
┌────────────────────┐
│ Array: [n elements]│ ← Storage
│ Top: integer       │ ← Index of top element
│ Capacity: integer  │ ← Maximum size
└────────────────────┘

Visual Representation:
            top = 3
              ↓
Index:  0   1   2   3   4   5   6   7
      ┌───┬───┬───┬───┬───┬───┬───┬───┐
Array:│ 5 │ 8 │ 3 │ 9 │   │   │   │   │
      └───┴───┴───┴───┴───┴───┴───┴───┘
      ← Filled →   ← Empty →

Properties:
• Top = -1 means empty stack
• Top = capacity - 1 means full stack
• Elements stored from index 0 to top
```

### Pseudocode Structure

```
STRUCTURE Stack:
    DATA:
        array[CAPACITY]  // Fixed-size array
        top: integer     // Index of top element
        capacity: integer // Maximum size

    INVARIANTS:
        -1 ≤ top < capacity
        If top = -1, stack is empty
        If top = capacity - 1, stack is full
        Elements stored at indices [0...top]
```

---

## Operations Implementation

### 1. Initialize (Constructor)

```
INITIALIZE(capacity):
    array = new Array[capacity]
    top = -1
    capacity = capacity

Visual:
Before:               After Initialize(8):
Nothing              top = -1
                     ┌───┬───┬───┬───┬───┬───┬───┬───┐
                     │   │   │   │   │   │   │   │   │
                     └───┴───┴───┴───┴───┴───┴───┴───┘
                     Empty stack ready to use

Time Complexity: O(1) - just initialization
Space Complexity: O(capacity) - allocate array
```

### 2. Push Operation

```
PUSH(item):
    IF top = capacity - 1 THEN
        ERROR "Stack Overflow"
    ELSE
        top = top + 1
        array[top] = item
    END IF

Step-by-Step Trace: Push(42)

Before:                After:
top = 2                top = 3
  ↓                      ↓
┌───┬───┬───┬───┐      ┌───┬───┬───┬───┐
│ 5 │ 8 │ 3 │   │      │ 5 │ 8 │ 3 │42 │
└───┴───┴───┴───┘      └───┴───┴───┴───┘

Steps:
1. Check if full (top == capacity - 1): No
2. Increment top: 2 → 3
3. Store item: array[3] = 42

Time Complexity: O(1)
Space Complexity: O(1)
```

### 3. Pop Operation

```
POP():
    IF top = -1 THEN
        ERROR "Stack Underflow"
    ELSE
        item = array[top]
        top = top - 1
        RETURN item
    END IF

Step-by-Step Trace: Pop()

Before:                After:
top = 3                top = 2
  ↓                      ↓
┌───┬───┬───┬───┐      ┌───┬───┬───┬───┐
│ 5 │ 8 │ 3 │42 │      │ 5 │ 8 │ 3 │42 │
└───┴───┴───┴───┘      └───┴───┴───┴───┘
                      (42 still there but not accessible)

Steps:
1. Check if empty (top == -1): No
2. Store element to return: item = array[3] = 42
3. Decrement top: 3 → 2
4. Return item: 42

Time Complexity: O(1)
Space Complexity: O(1)

Note: Element not actually deleted, just inaccessible
```

### 4. Peek/Top Operation

```
PEEK():
    IF top = -1 THEN
        ERROR "Stack Empty"
    ELSE
        RETURN array[top]
    END IF

Step-by-Step Trace: Peek()

Stack State:
top = 3
  ↓
┌───┬───┬───┬───┐
│ 5 │ 8 │ 3 │42 │
└───┴───┴───┴───┘

Steps:
1. Check if empty (top == -1): No
2. Return array[top]: 42
3. Stack unchanged

Returns: 42
Time Complexity: O(1)
Space Complexity: O(1)
```

### 5. isEmpty Operation

```
isEmpty():
    RETURN (top = -1)

Examples:
Empty Stack:           Non-Empty Stack:
top = -1               top = 2
  ↓                      ↓
┌───┬───┬───┐         ┌───┬───┬───┐
│   │   │   │         │ 5 │ 8 │ 3 │
└───┴───┴───┘         └───┴───┴───┘

isEmpty() = true      isEmpty() = false

Time Complexity: O(1)
Space Complexity: O(1)
```

### 6. isFull Operation

```
isFull():
    RETURN (top = capacity - 1)

Examples:
Not Full:             Full Stack:
top = 2               top = 3 (capacity = 4)
  ↓                     ↓
┌───┬───┬───┬───┐     ┌───┬───┬───┬───┐
│ 5 │ 8 │ 3 │   │     │ 5 │ 8 │ 3 │42 │
└───┴───┴───┴───┘     └───┴───┴───┴───┘

isFull() = false      isFull() = true

Time Complexity: O(1)
Space Complexity: O(1)
```

### 7. Size Operation

```
SIZE():
    RETURN top + 1

Examples:
Empty:                3 Elements:
top = -1              top = 2
size = -1 + 1 = 0     size = 2 + 1 = 3

Full (capacity=4):
top = 3
size = 3 + 1 = 4

Time Complexity: O(1)
Space Complexity: O(1)
```

---

## Complete Implementation Example

### Pseudocode

```pseudocode
CLASS ArrayStack:
    // Data members
    PRIVATE:
        array: Array of integers
        top: integer
        capacity: integer

    // Constructor
    PUBLIC ArrayStack(cap):
        capacity = cap
        array = new Array[capacity]
        top = -1

    // Push operation
    PUBLIC PUSH(item):
        IF isFull() THEN
            PRINT "Stack Overflow"
            RETURN false
        END IF
        
        top = top + 1
        array[top] = item
        RETURN true

    // Pop operation
    PUBLIC POP():
        IF isEmpty() THEN
            PRINT "Stack Underflow"
            RETURN null
        END IF

new_instruction        item = array[top]
        top = top - 1
        RETURN item

    // Peek operation
    PUBLIC PEEK():
        IF isEmpty() THEN
            PRINT "Stack Empty"
            RETURN null
        END IF
        RETURN array[top]

    // Check if empty
    PUBLIC isEmpty():
        RETURN (top = -1)

    // Check if full
    PUBLIC isFull():
        RETURN (top = capacity - 1)

    // Get size
    PUBLIC SIZE():
        RETURN top + 1

    // Display stack
    PUBLIC DISPLAY():
        IF isEmpty() THEN
            PRINT "Stack is empty"
            RETURN
        END IF

        PRINT "Stack (top to bottom):"
        FOR i = top DOWN TO 0:
            PRINT array[i]
        END FOR
END CLASS
```

---

## Detailed Operation Trace

### Complete Example Sequence

```
Operations: Stack(5) → Push(10) → Push(20) → Push(30) → Pop() → 
            Push(40) → Push(50) → Peek() → Size()

┌────────────────────────────────────────────────────────────┐
│ Step 1: Initialize Stack(5)                               │
├────────────────────────────────────────────────────────────┤
│ top = -1, capacity = 5                                     │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │   │   │   │   │   │                                     │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 2: Push(10)                                           │
├────────────────────────────────────────────────────────────┤
│ top = 0                                                    │
│   ↓                                                        │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │   │   │   │   │                                     │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 3: Push(20)                                           │
├────────────────────────────────────────────────────────────┤
│ top = 1                                                    │
│       ↓                                                    │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │20 │   │   │   │                                     │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 4: Push(30)                                           │
├────────────────────────────────────────────────────────────┤
│ top = 2                                                    │
│           ↓                                                │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │20 │30 │   │   │                                     │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 5: Pop() → Returns 30                                │
├────────────────────────────────────────────────────────────┤
│ top = 1                                                    │
│       ↓                                                    │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │20 │30 │   │   │ (30 still in array)               │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 6: Push(40)                                           │
├────────────────────────────────────────────────────────────┤
│ top = 2                                                    │
│           ↓                                                │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │20 │40 │   │   │ (30 overwritten)                   │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 7: Push(50)                                           │
├────────────────────────────────────────────────────────────┤
│ top = 3                                                    │
│               ↓                                            │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │20 │40 │50 │   │                                     │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 8: Peek() → Returns 50 (stack unchanged)             │
├────────────────────────────────────────────────────────────┤
│ top = 3                                                    │
│               ↓                                            │
│ ┌───┬───┬───┬───┬───┐                                     │
│ │10 │20 │40 │50 │   │                                     │
│ └───┴───┴───┴───┴───┘                                     │
│  0   1   2   3   4                                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Step 9: Size() → Returns 4 (top + 1 = 3 + 1)             │
├────────────────────────────────────────────────────────────┤
│ Final State: 4 elements in stack                           │
│ [10, 20, 40, 50]                                           │
└────────────────────────────────────────────────────────────┘
```

---

## Advantages of Array-Based Stack

```
✅ Advantages:

1. Simple Implementation
   • Straightforward logic
   • Easy to understand and debug
   • Few lines of code

2. Memory Efficiency
   • No extra pointer storage
   • Compact memory layout
   • Fixed overhead

3. Cache Friendly
   • Contiguous memory allocation
   • Better cache locality
   • Faster access times

4. Constant Time Operations
   • Push: O(1)
   • Pop: O(1)
   • Peek: O(1)
   • All operations predictable

5. Random Access (if needed)
   • Can access any element by index
   • Useful for debugging
   • Can implement specialized operations
```

---

## Disadvantages of Array-Based Stack

```
❌ Disadvantages:

1. Fixed Size
   • Must know maximum size in advance
   • Wasted space if not fully used
   • Overflow if size exceeded

2. No Dynamic Growth
   • Cannot expand beyond initial capacity
   • Overflow error on full stack
   • Must recreate with larger size (expensive)

3. Memory Waste
   • Unused array slots waste space
   • Over-allocation to prevent overflow
   • Space-time tradeoff

Example of Waste:
Capacity = 100, Used = 10
┌──┬──┬──┬──┬──┬──────────────────┐
│10│20│30│40│50│  (95 wasted)     │
└──┴──┴──┴──┴──┴──────────────────┘
 ← 5 used →  ← 95 unused →
```

---

## Edge Cases and Error Handling

### 1. Stack Overflow

```
Scenario: Push when full

Stack (capacity = 3):
top = 2 (capacity - 1)
      ↓
┌───┬───┬───┐
│10 │20 │30 │ ← Full!
└───┴───┴───┘

Push(40):
Error: "Stack Overflow"
Cannot insert!

Handling:
IF isFull() THEN
    THROW StackOverflowException
    OR RETURN error code
END IF
```

### 2. Stack Underflow

```
Scenario: Pop when empty

Stack:
top = -1
  ↓
┌───┬───┬───┐
│   │   │   │ ← Empty!
└───┴───┴───┘

Pop():
Error: "Stack Underflow"
Nothing to remove!

Handling:
IF isEmpty() THEN
    THROW StackUnderflowException
    OR RETURN null/error code
END IF
```

### 3. Peek on Empty Stack

```
Scenario: Peek when empty

Stack:
top = -1
┌───┬───┬───┐
│   │   │   │
└───┴───┴───┘

Peek():
Error: "Stack Empty"

Handling:
IF isEmpty() THEN
    THROW StackEmptyException
END IF
```

---

## Memory Layout

```
Memory Representation:

Stack Object:
┌─────────────────────────┐ ← Base address (e.g., 1000)
│ top: -1                 │ (4 bytes)
├─────────────────────────┤
│ capacity: 5             │ (4 bytes)
├─────────────────────────┤
│ array pointer: 2000     │ (8 bytes on 64-bit)
└─────────────────────────┘

Array Memory (at address 2000):
┌───┬───┬───┬───┬───┐
│   │   │   │   │   │ Each element 4 bytes (int)
└───┴───┴───┴───┴───┘
 ↑   ↑   ↑   ↑   ↑
2000 2004 2008 2012 2016

Total Memory:
• Stack object: 16 bytes
• Array: capacity × element_size
• Total: 16 + (5 × 4) = 36 bytes
```

---

## Complexity Analysis Summary

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| **Initialize** | O(1) | O(capacity) | Allocate array |
| **Push** | O(1) | O(1) | Direct assignment |
| **Pop** | O(1) | O(1) | Decrement pointer |
| **Peek** | O(1) | O(1) | Direct access |
| **isEmpty** | O(1) | O(1) | Simple comparison |
| **isFull** | O(1) | O(1) | Simple comparison |
| **Size** | O(1) | O(1) | Return top + 1 |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Storage** | Fixed-size array |
| **Top Tracking** | Integer index (-1 when empty) |
| **Time Complexity** | O(1) for all operations |
| **Space Complexity** | O(capacity) - fixed |
| **Advantages** | Simple, fast, cache-friendly |
| **Disadvantages** | Fixed size, potential waste |
| **Best For** | Known maximum size, predictable usage |
| **Not Ideal For** | Unknown size, dynamic growth needed |

---

## Quick Revision Questions

1. **Q: Why is top initialized to -1 instead of 0?**
   - A: Because -1 indicates an empty stack. When we push the first element, top becomes 0, which is the first valid array index.

2. **Q: What happens to the popped element in the array?**
   - A: It remains in the array but is no longer accessible (top pointer moved). It gets overwritten on the next push to that position.

3. **Q: How do you check if the stack is full?**
   - A: Check if `top == capacity - 1`. If true, the stack is full.

4. **Q: What is the space complexity of an array-based stack with capacity n?**
   - A: O(n) - the entire array must be allocated regardless of how many elements are actually stored.

5. **Q: Can you access the middle element of an array-based stack?**
   - A: Technically yes (it's an array), but you shouldn't - it violates the stack ADT which only allows top access.

6. **Q: What is the main disadvantage of array-based stack?**
   - A: Fixed size - you must know the maximum capacity in advance, and if exceeded, the stack overflows.

---

## Navigation

- **[← Previous: When to Use Stacks](../01-Stack-Fundamentals/06-when-to-use-stacks.md)**
- **[Next: Linked List-Based Stack →](02-linked-list-stack.md)**
- **[↑ Back to Unit 2](README.md)**
- **[⌂ Home](../README.md)**

---

*Array-based implementation is the foundation - master it before moving to more complex implementations!*
