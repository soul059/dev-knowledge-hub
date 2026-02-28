# Stack ADT (Abstract Data Type)

## Overview

A **Stack Abstract Data Type (ADT)** is a formal specification of what a stack is and what operations it supports, without specifying how these operations are implemented. An ADT defines the logical behavior and interface of the data structure, not its physical representation.

The Stack ADT provides a contract that any stack implementation must follow, ensuring consistency across different implementations.

---

## What is an Abstract Data Type?

### ADT Definition

```
Abstract Data Type (ADT)

┌───────────────────────────────────────┐
│         What (Interface)              │
│  • What operations are available      │
│  • What each operation does           │
│  • What parameters they take          │
│  • What they return                   │
└───────────────────────────────────────┘
                   ↕
         (Hides Implementation)
                   ↕
┌───────────────────────────────────────┐
│         How (Implementation)          │
│  • Array-based                        │
│  • Linked list-based                  │
│  • Dynamic array                      │
│  • Any other method                   │
└───────────────────────────────────────┘

ADT = Interface + Contract (not implementation)
```

### Abstraction Layers

```
User Level:
    ↓
┌─────────────────────┐
│   Stack ADT         │  ← What user sees
│   push(), pop()     │
│   peek(), isEmpty() │
└─────────────────────┘
    ↓ (uses)
┌─────────────────────┐
│  Implementation     │  ← Hidden from user
│  Array / LinkedList │
└─────────────────────┘
    ↓
Physical Memory
```

---

## Stack ADT Specification

### Data

```
Stack contains:
  • A collection of elements
  • A reference/pointer to the top element
  • Size/count of elements (optional)

Logical View:
┌──────────┐
│   Top    │ ← Pointer/Reference
├──────────┤
│ Element n│
├──────────┤
│ Element 2│
├──────────┤
│ Element 1│ ← Bottom
└──────────┘
```

### Operations (Interface)

| Operation | Signature | Description | Precondition | Postcondition |
|-----------|-----------|-------------|--------------|---------------|
| **push(item)** | void push(T item) | Add item to top | - | Size increased by 1, item is new top |
| **pop()** | T pop() | Remove and return top | Stack not empty | Size decreased by 1, previous top removed |
| **peek()** | T peek() | Return top without removing | Stack not empty | Stack unchanged |
| **isEmpty()** | boolean isEmpty() | Check if empty | - | Returns true if size=0, false otherwise |
| **size()** | int size() | Get number of elements | - | Returns current size |
| **isFull()** | boolean isFull() | Check if full | - | Returns true if at capacity (fixed-size only) |

---

## Formal ADT Specification

### Mathematical Notation

```
Stack ADT = (D, O, P)

Where:
  D = Domain (Data)
      - Collection of elements of type T
      - Top pointer
      - Size counter

  O = Operations
      - push: Stack × T → Stack
      - pop: Stack → Stack × T
      - peek: Stack → T
      - isEmpty: Stack → Boolean
      - size: Stack → Integer

  P = Properties (Axioms)
      - isEmpty(newStack) = true
      - isEmpty(push(s, x)) = false
      - pop(push(s, x)) = (s, x)
      - peek(push(s, x)) = x
      - size(newStack) = 0
      - size(push(s, x)) = size(s) + 1
```

---

## ADT Operations in Detail

### 1. Push Operation

```
Operation: push(item)

Formal Definition:
  Input:  Stack S, Element e
  Output: Modified Stack S'
  Effect: S' = S ∪ {e} where e becomes top

Pseudocode:
┌─────────────────────────────┐
│ PUSH(S, item)               │
│   IF isFull(S) THEN         │
│     ERROR "Stack Overflow"  │
│   ELSE                      │
│     top = top + 1           │
│     S[top] = item           │
│   END IF                    │
└─────────────────────────────┘

Visual:
Before:        After push(X):
┌────┐        ┌────┐
│ C  │        │ X  │ ← New top
├────┤        ├────┤
│ B  │        │ C  │
├────┤        ├────┤
│ A  │        │ B  │
└────┘        ├────┤
              │ A  │
              └────┘
```

### 2. Pop Operation

```
Operation: pop()

Formal Definition:
  Input:  Stack S (non-empty)
  Output: Element e, Modified Stack S'
  Effect: e = top(S), S' = S \ {e}

Pseudocode:
┌─────────────────────────────┐
│ POP(S)                      │
│   IF isEmpty(S) THEN        │
│     ERROR "Stack Underflow" │
│   ELSE                      │
│     item = S[top]           │
│     top = top - 1           │
│     RETURN item             │
│   END IF                    │
└─────────────────────────────┘

Visual:
Before:        After pop():
┌────┐        ┌────┐
│ C  │        │ B  │ ← New top
├────┤        ├────┤
│ B  │        │ A  │
├────┤        └────┘
│ A  │        Returns: C
└────┘
```

### 3. Peek Operation

```
Operation: peek()

Formal Definition:
  Input:  Stack S (non-empty)
  Output: Element e
  Effect: Returns top element without modification

Pseudocode:
┌─────────────────────────────┐
│ PEEK(S)                     │
│   IF isEmpty(S) THEN        │
│     ERROR "Stack Empty"     │
│   ELSE                      │
│     RETURN S[top]           │
│   END IF                    │
└─────────────────────────────┘

Visual:
Before:        After peek():
┌────┐        ┌────┐
│ C  │        │ C  │ ← Still top (unchanged)
├────┤        ├────┤
│ B  │        │ B  │
├────┤        ├────┤
│ A  │        │ A  │
└────┘        └────┘
Returns: C    Stack unchanged
```

### 4. isEmpty Operation

```
Operation: isEmpty()

Formal Definition:
  Input:  Stack S
  Output: Boolean
  Effect: Returns true if S has no elements

Pseudocode:
┌─────────────────────────────┐
│ isEmpty(S)                  │
│   RETURN (top == -1)        │
│   OR                        │
│   RETURN (size == 0)        │
└─────────────────────────────┘

Examples:
Empty Stack:      Non-Empty Stack:
┌────┐           ┌────┐
│ ⊥  │           │ A  │
└────┘           └────┘
isEmpty() = true  isEmpty() = false
```

### 5. Size Operation

```
Operation: size()

Formal Definition:
  Input:  Stack S
  Output: Integer n ≥ 0
  Effect: Returns number of elements in S

Pseudocode:
┌─────────────────────────────┐
│ SIZE(S)                     │
│   RETURN (top + 1)          │
│   OR                        │
│   RETURN size_counter       │
└─────────────────────────────┘

Example:
┌────┐
│ D  │  ← top (index 3)
├────┤
│ C  │  (index 2)
├────┤
│ B  │  (index 1)
├────┤
│ A  │  (index 0)
└────┘
size() = 4
```

---

## ADT Axioms (Properties)

### Fundamental Axioms

```
Axiom 1: Empty Stack Property
  isEmpty(newStack()) = true

Axiom 2: Non-Empty After Push
  isEmpty(push(S, x)) = false

Axiom 3: Push-Pop Inverse
  pop(push(S, x)) = (S, x)
  "Popping after pushing returns the stack to original state"

Axiom 4: Peek After Push
  peek(push(S, x)) = x
  "Peeking after pushing returns the pushed element"

Axiom 5: Size of Empty
  size(newStack()) = 0

Axiom 6: Size After Push
  size(push(S, x)) = size(S) + 1

Axiom 7: Size After Pop
  size(pop(S)) = size(S) - 1  (if not empty)
```

### Example Verification

```
Verify Axiom 3: pop(push(S, x)) = (S, x)

Start:
S = [A, B]     size = 2

Push(S, C):
S' = [A, B, C] size = 3

Pop(S'):
S'' = [A, B]   size = 2, returns C

Result: S'' = S ✓ (Axiom verified)
```

---

## Stack ADT State Diagram

```
State Transitions:

              newStack()
                  ↓
            ┌──────────┐
            │  Empty   │
            │ size = 0 │
            └──────────┘
                 ↓ push(x)
            ┌──────────┐
            │Has Items │ ←──┐
            │ size > 0 │    │
            └──────────┘    │ push(x)
                 │     ↑────┘
                 │pop()│
          ┌──────┴──────┐
          ↓              ↓
    ┌──────────┐   ┌──────────┐
    │  Empty   │   │Has Items │
    │ size = 0 │   │ size > 0 │
    └──────────┘   └──────────┘

Invariants:
  - size ≥ 0 (always)
  - If size = 0, pop() raises error
  - If size = capacity, push() raises error (fixed-size)
```

---

## Generic Stack ADT

### Type-Parameterized Definition

```pseudocode
STACK<T>:
  Data:
    elements: Collection<T>
    top: Integer
    capacity: Integer (optional)

  Operations:
    push(item: T): void
    pop(): T
    peek(): T
    isEmpty(): Boolean
    size(): Integer
    isFull(): Boolean  (optional)

Where T can be:
  - Integer, Float, Character
  - String, Object
  - Custom data types
  - Any comparable type
```

### Example Instantiations

```
Stack<Integer>:
┌─────┐
│  42 │
├─────┤
│  17 │
├─────┤
│   5 │
└─────┘

Stack<String>:
┌────────┐
│ "cat"  │
├────────┤
│ "dog"  │
├────────┤
│ "bird" │
└────────┘

Stack<Point>:
┌────────┐
│ (5, 7) │
├────────┤
│ (2, 3) │
├────────┤
│ (1, 1) │
└────────┘
```

---

## Complete ADT Example

### Trace of Operations

```
Operation Sequence:
1. Create stack
2. isEmpty() → ?
3. push(10)
4. push(20)
5. peek() → ?
6. push(30)
7. pop() → ?
8. size() → ?
9. pop() → ?
10. pop() → ?
11. isEmpty() → ?

Execution Trace:

Step 1: stack = newStack()
┌────┐
│ ⊥  │  size = 0
└────┘

Step 2: isEmpty()
Returns: true

Step 3: push(10)
┌────┐
│ 10 │  size = 1
└────┘

Step 4: push(20)
┌────┐
│ 20 │  size = 2
├────┤
│ 10 │
└────┘

Step 5: peek()
Returns: 20
┌────┐
│ 20 │  Stack unchanged
├────┤
│ 10 │
└────┘

Step 6: push(30)
┌────┐
│ 30 │  size = 3
├────┤
│ 20 │
├────┤
│ 10 │
└────┘

Step 7: pop()
Returns: 30
┌────┐
│ 20 │  size = 2
├────┤
│ 10 │
└────┘

Step 8: size()
Returns: 2
(Stack unchanged)

Step 9: pop()
Returns: 20
┌────┐
│ 10 │  size = 1
└────┘

Step 10: pop()
Returns: 10
┌────┐
│ ⊥  │  size = 0
└────┘

Step 11: isEmpty()
Returns: true
```

---

## ADT vs Implementation

### Separation of Concerns

```
┌─────────────────────────────────┐
│      STACK ADT (Contract)       │
├─────────────────────────────────┤
│ Operations:                     │
│  • push(item)                   │
│  • pop(): item                  │
│  • peek(): item                 │
│  • isEmpty(): boolean           │
│  • size(): int                  │
└─────────────────────────────────┘
           ↓ Implements ↓
┌──────────────┬──────────────┬──────────────┐
│Array-Based   │Linked List   │Dynamic Array │
│Implementation│Implementation│Implementation│
├──────────────┼──────────────┼──────────────┤
│Fixed size    │Unlimited size│Auto-resize   │
│Fast access   │Easy growth   │Best of both  │
│Memory waste  │Extra pointers│Amortized O(1)│
└──────────────┴──────────────┴──────────────┘

All implement same ADT interface!
```

---

## Error Handling in ADT

### Exceptional Conditions

```
Underflow Condition:
┌────┐
│ ⊥  │  Empty Stack
└────┘
    ↓ pop()
  ERROR!

Overflow Condition (Fixed-size):
┌────┐
│ E  │  ← Capacity reached
├────┤
│ D  │
├────┤
│ C  │
├────┤
│ B  │
├────┤
│ A  │
└────┘
    ↓ push(F)
  ERROR!

Proper Error Handling:
┌─────────────────────────────┐
│ POP()                       │
│   IF isEmpty() THEN         │
│     THROW StackUnderflow    │
│   END IF                    │
│   ...                       │
└─────────────────────────────┘
```

---

## ADT Benefits

### 1. **Encapsulation**

```
User doesn't need to know:
  × How items are stored (array vs linked list)
  × How top is tracked (pointer vs index)
  × Memory allocation details

User only needs to know:
  ✓ What operations are available
  ✓ How to call them
  ✓ What they return
```

### 2. **Implementation Flexibility**

```
Same ADT → Multiple Implementations

           Stack ADT
              ↓
    ┌─────────┼─────────┐
    ↓         ↓         ↓
Array      LinkedList  Hybrid
Fast       Flexible    Balanced
Fixed      Dynamic     Optimal
```

### 3. **Interchangeability**

```
Can swap implementations without changing code:

Before:
  Stack<int> s = new ArrayStack<int>();

After:
  Stack<int> s = new LinkedStack<int>();

User code remains unchanged!
```

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| **Definition** | Formal specification of stack behavior |
| **Core Operations** | push, pop, peek, isEmpty, size |
| **Key Principle** | LIFO (Last In, First Out) |
| **Type** | Generic (can hold any type) |
| **Interface** | Defines what, not how |
| **Implementations** | Array, Linked List, Dynamic Array |
| **Time Complexity** | O(1) for all basic operations |
| **Error Conditions** | Underflow (pop on empty), Overflow (push on full) |
| **Benefit** | Abstraction, flexibility, reusability |

---

## Quick Revision Questions

1. **Q: What is the difference between an ADT and an implementation?**
   - A: An ADT specifies what operations are available and what they do (interface); an implementation specifies how those operations are realized (array, linked list, etc.).

2. **Q: What are the essential operations that every stack ADT must support?**
   - A: push (add element), pop (remove element), peek (view top), isEmpty (check if empty). Optionally: size, isFull.

3. **Q: What does the axiom "pop(push(S, x)) = (S, x)" mean?**
   - A: If you push an element x onto stack S and immediately pop, you get back the original stack S and the element x.

4. **Q: Why is Stack ADT called "generic"?**
   - A: Because it can work with any data type (integers, strings, objects, etc.) by using type parameters like Stack<T>.

5. **Q: What error occurs when you call pop() on an empty stack?**
   - A: Stack underflow error - there are no elements to remove.

6. **Q: Can you change the implementation of a stack ADT without affecting user code?**
   - A: Yes, because the ADT defines the interface. As long as the new implementation follows the same interface, user code remains unchanged.

---

## Navigation

- **[← Previous: LIFO Principle](02-lifo-principle.md)**
- **[Next: Real-World Analogies →](04-real-world-analogies.md)**
- **[↑ Back to Unit 1](README.md)**
- **[⌂ Home](../README.md)**

---

*Understanding the Stack ADT is crucial - it separates the logical behavior from physical implementation, enabling flexible and modular design!*
