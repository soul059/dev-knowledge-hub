# Chapter 6: Stack Using Queues

[← Previous: Sort a Stack](05-sort-stack.md) | [Next: Infix, Prefix, Postfix Notation →](../04-Expression-Evaluation/01-notation-types.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

---

## Overview

Implementing a **stack using queues** is a classic data structure design problem. Since a queue is FIFO and a stack is LIFO, we need clever manipulation to simulate LIFO behavior. This problem teaches how to build one abstract data type from another.

---

## Problem Statement

Implement a stack (LIFO) using only queue operations: `enqueue`, `dequeue`, `front`, `isEmpty`, and `size`.

```
Stack Behavior to Achieve:
  push(1), push(2), push(3)
  pop() → 3  (LIFO: last pushed, first popped)
  pop() → 2
  top() → 1
```

---

## Approach 1: Two Queues — Costly Push

Make `push` expensive by rearranging elements to ensure the newest element is always at the front.

### Pseudocode

```
CLASS StackUsingTwoQueues:
    q1 ← empty queue    // Main queue (stores elements in stack order)
    q2 ← empty queue    // Helper queue

    FUNCTION push(x):
        // Step 1: Enqueue to q2
        q2.enqueue(x)
        
        // Step 2: Move all elements from q1 to q2
        WHILE q1 is NOT empty:
            q2.enqueue(q1.dequeue())
        
        // Step 3: Swap q1 and q2
        SWAP(q1, q2)
    
    FUNCTION pop():
        IF q1.isEmpty():
            ERROR "Stack Underflow"
        RETURN q1.dequeue()
    
    FUNCTION top():
        IF q1.isEmpty():
            ERROR "Stack Underflow"
        RETURN q1.front()
    
    FUNCTION isEmpty():
        RETURN q1.isEmpty()
```

### Trace

```
push(1):
  q2: [1]
  Move q1→q2: nothing to move
  Swap: q1=[1], q2=[]
  
  q1: [1]    (front=1)

push(2):
  q2: [2]
  Move q1→q2: q2=[2,1]
  Swap: q1=[2,1], q2=[]
  
  q1: [2, 1]    (front=2 ← most recent!)

push(3):
  q2: [3]
  Move q1→q2: q2=[3,2,1]
  Swap: q1=[3,2,1], q2=[]
  
  q1: [3, 2, 1]    (front=3 ← most recent!)

pop():
  Dequeue from q1 → returns 3 ✓ (LIFO!)
  q1: [2, 1]

pop():
  Dequeue from q1 → returns 2 ✓
  q1: [1]

Visual:
  After 3 pushes, q1 looks like:
  
  Front                    Rear
  ┌───┬───┬───┐
  │ 3 │ 2 │ 1 │
  └───┴───┴───┘
    ↑
   dequeue here = LIFO order!
```

### Complexity

| Operation | Time | Explanation |
|-----------|------|-------------|
| **push** | O(n) | Move all n elements from q1 to q2 |
| **pop** | O(1) | Simple dequeue |
| **top** | O(1) | Simple front |

---

## Approach 2: Two Queues — Costly Pop

Make `pop` expensive by searching for the last element during pop.

### Pseudocode

```
CLASS StackCostlyPop:
    q1 ← empty queue
    q2 ← empty queue

    FUNCTION push(x):
        q1.enqueue(x)          // O(1) — just add to rear
    
    FUNCTION pop():
        IF q1.isEmpty():
            ERROR "Stack Underflow"
        
        // Move all except last to q2
        WHILE q1.size() > 1:
            q2.enqueue(q1.dequeue())
        
        // Last element is the "top" of stack
        result ← q1.dequeue()
        
        // Swap q1 and q2
        SWAP(q1, q2)
        
        RETURN result

    FUNCTION top():
        IF q1.isEmpty():
            ERROR "Stack Underflow"
        
        WHILE q1.size() > 1:
            q2.enqueue(q1.dequeue())
        
        result ← q1.front()
        q2.enqueue(q1.dequeue())
        
        SWAP(q1, q2)
        RETURN result
```

### Trace

```
push(1): q1=[1]
push(2): q1=[1,2]
push(3): q1=[1,2,3]

pop():
  Move all except last to q2:
    dequeue 1 → q2=[1]
    dequeue 2 → q2=[1,2]
  Last in q1: dequeue → 3 (this is the stack top!) ✓
  Swap: q1=[1,2], q2=[]

  q1: [1, 2]

pop():
  Move all except last to q2:
    dequeue 1 → q2=[1]
  Last in q1: dequeue → 2 ✓
  Swap: q1=[1], q2=[]
```

### Complexity

| Operation | Time | Explanation |
|-----------|------|-------------|
| **push** | O(1) | Simple enqueue |
| **pop** | O(n) | Move n-1 elements to find last |
| **top** | O(n) | Same as pop but re-enqueue |

---

## Approach 3: Single Queue

Use only **one queue** with a rotation trick.

### Pseudocode

```
CLASS StackUsingSingleQueue:
    q ← empty queue
    
    FUNCTION push(x):
        q.enqueue(x)
        
        // Rotate queue so newest element is at front
        FOR i = 1 TO q.size() - 1:
            q.enqueue(q.dequeue())
    
    FUNCTION pop():
        IF q.isEmpty():
            ERROR "Stack Underflow"
        RETURN q.dequeue()
    
    FUNCTION top():
        IF q.isEmpty():
            ERROR "Stack Underflow"
        RETURN q.front()
    
    FUNCTION isEmpty():
        RETURN q.isEmpty()
```

### Trace

```
push(1):
  Enqueue 1: q=[1]
  Rotate 0 times (size-1 = 0)
  q: [1]     front=1

push(2):
  Enqueue 2: q=[1,2]
  Rotate 1 time:
    dequeue 1, enqueue 1: q=[2,1]
  q: [2, 1]     front=2 ← newest!

push(3):
  Enqueue 3: q=[2,1,3]
  Rotate 2 times:
    dequeue 2, enqueue 2: q=[1,3,2]
    dequeue 1, enqueue 1: q=[3,2,1]
  q: [3, 2, 1]     front=3 ← newest!

Visual after push(3):
  Front                 Rear
  ┌───┬───┬───┐
  │ 3 │ 2 │ 1 │
  └───┴───┴───┘
    ↑
  dequeue = LIFO!

pop() → dequeue → 3 ✓
pop() → dequeue → 2 ✓
```

### Complexity

| Operation | Time | Explanation |
|-----------|------|-------------|
| **push** | O(n) | Rotate n-1 elements |
| **pop** | O(1) | Simple dequeue |
| **top** | O(1) | Simple front |
| **Space** | O(n) | Only one queue needed |

---

## Comparison of Approaches

```
┌───────────────────┬────────────┬────────────┬─────────────┐
│ Approach          │ push       │ pop        │ Space       │
├───────────────────┼────────────┼────────────┼─────────────┤
│ 2Q Costly Push    │ O(n)       │ O(1)       │ O(n) 2 Qs   │
│ 2Q Costly Pop     │ O(1)       │ O(n)       │ O(n) 2 Qs   │
│ 1Q (rotation)     │ O(n)       │ O(1)       │ O(n) 1 Q    │
└───────────────────┴────────────┴────────────┴─────────────┘

Choose based on operation frequency:
  • More pushes → Use Costly Pop (push is O(1))
  • More pops   → Use Costly Push or 1Q (pop is O(1))
  • Minimal space → Use Single Queue
```

---

## Why This Problem Matters

```
┌──────────────────────────────────────────────────────┐
│  1. Tests understanding of FIFO vs LIFO              │
│  2. Shows data structures can simulate each other    │
│  3. Common interview question (LeetCode #225)        │
│  4. Demonstrates design trade-offs (push vs pop)     │
│  5. Reverse problem: Queue using Stacks also exists  │
│     (LeetCode #232)                                  │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Implement Stack using Queue(s) |
| **Approaches** | 2 Queues (costly push/pop), 1 Queue (rotation) |
| **Key Insight** | Rearrange queue to put newest element at front |
| **Best for many pops** | Costly push or single queue (O(1) pop) |
| **Best for many pushes** | Costly pop (O(1) push) |
| **Space** | O(n) for all approaches |

---

## Quick Revision Questions

1. **Why can't a queue directly behave as a stack?**
   > Queue is FIFO (first in, first out), while stack is LIFO (last in, first out). The access order is opposite.

2. **In the single-queue approach, what does the rotation do?**
   > After enqueueing a new element at the rear, rotation moves all previous elements behind it, making the new element the front (simulating stack top).

3. **Which approach is best when pushes are more frequent than pops?**
   > Costly Pop — push is O(1), so frequent pushes are cheap.

4. **What's the fundamental trade-off in all approaches?**
   > Either push or pop must be O(n); you can't have both O(1) when simulating LIFO with FIFO.

5. **What is the reverse problem (Queue using Stacks) and how does it differ?**
   > LeetCode #232. You use two stacks to simulate FIFO behavior. One stack handles enqueue, the other handles dequeue with amortized O(1) per operation.

---

[← Previous: Sort a Stack](05-sort-stack.md) | [Next: Infix, Prefix, Postfix Notation →](../04-Expression-Evaluation/01-notation-types.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)
