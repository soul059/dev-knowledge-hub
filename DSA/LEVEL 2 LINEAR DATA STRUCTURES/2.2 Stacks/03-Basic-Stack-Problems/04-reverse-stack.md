# Chapter 4: Reverse a Stack

[← Previous: Reverse a String](03-reverse-string.md) | [Next: Sort a Stack →](05-sort-stack.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

---

## Overview

**Reversing a stack** means rearranging elements so that the bottom element becomes the top and the top becomes the bottom, using only stack operations. This problem teaches important recursive thinking and the concept of using the **call stack as auxiliary storage**.

---

## Problem Statement

Given a stack, reverse the order of elements using only standard stack operations (push, pop, peek, isEmpty). No additional data structures allowed (no extra array/queue).

```
Before:         After:
┌───┐           ┌───┐
│ 5 │ ← top     │ 1 │ ← top
│ 4 │           │ 2 │
│ 3 │           │ 3 │
│ 2 │           │ 4 │
│ 1 │ ← bottom  │ 5 │ ← bottom
└───┘           └───┘
```

---

## Approach 1: Using Auxiliary Stack

The simplest approach uses an extra stack (or two).

```
FUNCTION reverseStack_TwoStacks(stack):
    aux1 ← empty stack
    aux2 ← empty stack
    
    // Move all to aux1 (reverses once)
    WHILE stack is NOT empty:
        aux1.push(stack.pop())
    
    // Move all to aux2 (reverses again → original order)
    WHILE aux1 is NOT empty:
        aux2.push(aux1.pop())
    
    // Move back to original stack (reverses third time → fully reversed)
    WHILE aux2 is NOT empty:
        stack.push(aux2.pop())
```

```
Original → aux1 → aux2 → stack

  ┌───┐     ┌───┐     ┌───┐     ┌───┐
  │ 3 │     │ 1 │     │ 3 │     │ 1 │
  │ 2 │  →  │ 2 │  →  │ 2 │  →  │ 2 │
  │ 1 │     │ 3 │     │ 1 │     │ 3 │
  └───┘     └───┘     └───┘     └───┘
  stack      aux1      aux2      stack
             (rev1)    (rev2)    (rev3=reversed!)
```

**Limitation**: Uses O(n) extra space with additional stacks.

---

## Approach 2: Recursive (No Extra Data Structure)

The elegant approach uses **recursion** (the call stack) as implicit storage.

### Key Idea

```
┌──────────────────────────────────────────────────┐
│           TWO RECURSIVE FUNCTIONS                │
│                                                  │
│  1. insertAtBottom(stack, item):                 │
│     Insert an item at the bottom of the stack    │
│                                                  │
│  2. reverseStack(stack):                         │
│     Reverse the entire stack recursively         │
│                                                  │
│  reverseStack uses insertAtBottom as a helper    │
└──────────────────────────────────────────────────┘
```

### Helper: Insert at Bottom

```
FUNCTION insertAtBottom(stack, item):
    IF stack.isEmpty():
        stack.push(item)
    ELSE:
        temp ← stack.pop()            // Hold top element
        insertAtBottom(stack, item)    // Recurse to reach bottom
        stack.push(temp)              // Put top element back
```

```
InsertAtBottom(stack=[3,2,1], item=4):
  
  Step 1: pop 1 → temp=1, stack=[3,2]
    Step 2: pop 2 → temp=2, stack=[3]
      Step 3: pop 3 → temp=3, stack=[]
        Step 4: empty! push 4 → stack=[4]
      push 3 → stack=[4,3]
    push 2 → stack=[4,3,2]
  push 1 → stack=[4,3,2,1]
  
  Result: 4 is now at the BOTTOM!
  
  ┌───┐        ┌───┐
  │ 1 │        │ 1 │
  │ 2 │   →    │ 2 │
  │ 3 │        │ 3 │
  └───┘        │ 4 │ ← inserted at bottom
               └───┘
```

### Main: Reverse Stack

```
FUNCTION reverseStack(stack):
    IF stack.isEmpty():
        RETURN
    
    temp ← stack.pop()            // Hold top element
    reverseStack(stack)            // Reverse remaining stack
    insertAtBottom(stack, temp)    // Put top element at bottom
```

---

## Complete Trace

### Input: Stack = [1, 2, 3] (top = 3)

```
reverseStack([1,2,3]):
  pop 3 → temp=3
  
  reverseStack([1,2]):
    pop 2 → temp=2
    
    reverseStack([1]):
      pop 1 → temp=1
      
      reverseStack([]):
        Base case: return    Stack: []
      
      insertAtBottom([], 1):
        Empty → push 1       Stack: [1]
    
    insertAtBottom([1], 2):
      pop 1 → temp=1         Stack: []
      insertAtBottom([], 2):
        Empty → push 2       Stack: [2]
      push 1                  Stack: [2,1]
  
  insertAtBottom([2,1], 3):
    pop 1 → temp=1           Stack: [2]
    insertAtBottom([2], 3):
      pop 2 → temp=2         Stack: []
      insertAtBottom([], 3):
        Empty → push 3       Stack: [3]
      push 2                  Stack: [3,2]
    push 1                    Stack: [3,2,1]

FINAL RESULT:
  ┌───┐        ┌───┐
  │ 3 │        │ 1 │
  │ 2 │   →    │ 2 │
  │ 1 │        │ 3 │
  └───┘        └───┘
  Before       After    ✓ Reversed!
```

---

## How the Recursion Works (Visual)

```
Call Tree for reverseStack([1,2,3]):

reverseStack([1,2,3])
├── pop 3
├── reverseStack([1,2])
│   ├── pop 2
│   ├── reverseStack([1])
│   │   ├── pop 1
│   │   ├── reverseStack([])   ← base case
│   │   └── insertAtBottom([], 1) → [1]
│   └── insertAtBottom([1], 2) → [2,1]
└── insertAtBottom([2,1], 3) → [3,2,1]

Unwinding:
  Each popped element goes to the BOTTOM
  of the already-reversed sub-stack.
  
  [] → [1] → [2,1] → [3,2,1]
        ↑      ↑        ↑
        1 at   2 at     3 at
        bottom bottom   bottom
```

---

## Complexity Analysis

### Recursive Approach

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n²) | reverseStack: n calls × insertAtBottom: up to n calls each |
| **Space** | O(n) | Recursion depth (call stack) |

### Time Breakdown

```
reverseStack(n elements):
  - Calls itself n times (pops each element)
  - Each call invokes insertAtBottom
  
insertAtBottom on stack of size k:
  - Makes k recursive calls (pops to reach bottom)
  
Total operations:
  insertAtBottom(0) +
  insertAtBottom(1) +
  insertAtBottom(2) + ... +
  insertAtBottom(n-1)
  = 0 + 1 + 2 + ... + (n-1)
  = n(n-1)/2
  = O(n²)
```

### Auxiliary Stack Approach

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Three passes of n pops/pushes |
| **Space** | O(n) | Two extra stacks |

---

## Approach 3: Using a Queue

```
FUNCTION reverseStack_Queue(stack):
    queue ← empty queue
    
    // Move stack to queue
    WHILE stack is NOT empty:
        queue.enqueue(stack.pop())
    
    // Move queue back to stack
    WHILE queue is NOT empty:
        stack.push(queue.dequeue())

Trace: Stack [1,2,3] (top=3)
  Pop to queue: 3, 2, 1 → Queue: [3, 2, 1]
  Dequeue to stack: 3→push, 2→push, 1→push
  Stack: [3, 2, 1] (top=1) ✓ Reversed!
```

---

## Comparison of Approaches

```
┌──────────────────┬────────┬────────┬──────────────────────┐
│ Approach         │ Time   │ Space  │ Extra Data Structure │
├──────────────────┼────────┼────────┼──────────────────────┤
│ Two aux stacks   │ O(n)   │ O(n)  │ Yes (2 stacks)       │
│ Recursion        │ O(n²)  │ O(n)  │ No (uses call stack) │
│ Queue            │ O(n)   │ O(n)  │ Yes (1 queue)        │
└──────────────────┴────────┴────────┴──────────────────────┘

Best time: O(n) with auxiliary structure
Best "no extra DS": Recursion at O(n²) cost
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Reverse order of stack elements |
| **Best Approach** | Recursive if no extra DS allowed |
| **Key Helper** | `insertAtBottom` recursive function |
| **Time (Recursive)** | O(n²) |
| **Space (Recursive)** | O(n) call stack |
| **Alternative** | Aux stack/queue for O(n) time |

---

## Quick Revision Questions

1. **Why is the recursive reversal O(n²)?**
   > `reverseStack` calls `insertAtBottom` for each element, and `insertAtBottom` itself takes O(k) for a stack of size k. Total: 1+2+...+n = O(n²).

2. **What does `insertAtBottom` do?**
   > It recursively pops all elements, pushes the new item when the stack is empty (bottom), then pushes all popped elements back.

3. **Can you reverse a stack in O(n) without extra data structures?**
   > Not with standard stack operations alone. You need either auxiliary storage or accept O(n²) from recursion.

4. **How many times does each element get pushed/popped in the recursive approach?**
   > Each element is popped/pushed O(n) times across all `insertAtBottom` calls.

5. **Why is the queue approach simpler than using two extra stacks?**
   > A queue preserves FIFO order. Pop from stack → enqueue preserves reverse order, then dequeue → push reverses again. Only 2 passes needed vs 3 for two stacks.

---

[← Previous: Reverse a String](03-reverse-string.md) | [Next: Sort a Stack →](05-sort-stack.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)
