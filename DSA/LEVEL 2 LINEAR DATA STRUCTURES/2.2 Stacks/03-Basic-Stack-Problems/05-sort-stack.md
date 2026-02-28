# Chapter 5: Sort a Stack

[← Previous: Reverse a Stack](04-reverse-stack.md) | [Next: Stack Using Queues →](06-stack-using-queues.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

---

## Overview

Sorting a stack means arranging its elements in ascending or descending order using only stack operations. This problem teaches creative use of temporary storage and recursive thinking with constrained data structures.

---

## Problem Statement

Sort a stack such that the smallest element is on top (ascending from top to bottom). Only stack operations (`push`, `pop`, `peek`, `isEmpty`) are allowed. You may use **one additional temporary stack**.

```
Before:         After:
┌───┐           ┌───┐
│ 3 │ ← top     │ 1 │ ← top (smallest)
│ 1 │           │ 2 │
│ 4 │           │ 3 │
│ 2 │           │ 4 │
│ 5 │ ← bottom  │ 5 │ ← bottom (largest)
└───┘           └───┘
```

---

## Approach 1: Using a Temporary Stack

### Key Idea

```
┌──────────────────────────────────────────────────┐
│         INSERTION SORT WITH TWO STACKS           │
│                                                  │
│  Think of it as INSERTION SORT:                  │
│  - Pop element from input stack                  │
│  - Find its correct position in temp stack       │
│  - Insert it there by shifting larger elements   │
│                                                  │
│  Input Stack    Temp Stack (sorted)              │
│  ┌───┐          ┌───┐                            │
│  │   │          │   │ ← always sorted            │
│  │   │   ←→     │   │   (largest on top)         │
│  │   │          │   │                            │
│  └───┘          └───┘                            │
└──────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION sortStack(inputStack):
    tempStack ← empty stack
    
    WHILE inputStack is NOT empty:
        // Step 1: Pop current element
        current ← inputStack.pop()
        
        // Step 2: Move larger elements from temp back to input
        WHILE tempStack is NOT empty AND tempStack.peek() > current:
            inputStack.push(tempStack.pop())
        
        // Step 3: Place current in correct position
        tempStack.push(current)
    
    // Step 4: Move all elements back to input stack
    WHILE tempStack is NOT empty:
        inputStack.push(tempStack.pop())
```

---

## Step-by-Step Trace

### Input: [5, 2, 4, 1, 3] (top = 3)

```
═══════════════════════════════════════════════════════
  SORTING PROCESS
═══════════════════════════════════════════════════════

Iteration 1: current = 3 (popped from input)
  temp is empty → just push
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 1 │      │ 3 │
  │ 4 │      └───┘
  │ 2 │
  │ 5 │
  └───┘

─────────────────────────────────────────────────────

Iteration 2: current = 1 (popped from input)
  temp.peek() = 3 > 1? YES → move 3 back to input
  temp is empty → push 1
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 3 │      │ 1 │
  │ 4 │      └───┘
  │ 2 │
  │ 5 │
  └───┘

─────────────────────────────────────────────────────

Iteration 3: current = 3 (popped from input)
  temp.peek() = 1 > 3? NO → push 3
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 4 │      │ 3 │
  │ 2 │      │ 1 │
  │ 5 │      └───┘
  └───┘

─────────────────────────────────────────────────────

Iteration 4: current = 4 (popped from input)
  temp.peek() = 3 > 4? NO → push 4
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 2 │      │ 4 │
  │ 5 │      │ 3 │
  └───┘      │ 1 │
             └───┘

─────────────────────────────────────────────────────

Iteration 5: current = 2 (popped from input)
  temp.peek() = 4 > 2? YES → move 4 to input
  temp.peek() = 3 > 2? YES → move 3 to input
  temp.peek() = 1 > 2? NO → push 2
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 3 │      │ 2 │
  │ 4 │      │ 1 │
  │ 5 │      └───┘
  └───┘

─────────────────────────────────────────────────────

Iteration 6: current = 3 (popped from input)
  temp.peek() = 2 > 3? NO → push 3
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 4 │      │ 3 │
  │ 5 │      │ 2 │
  └───┘      │ 1 │
             └───┘

─────────────────────────────────────────────────────

Iteration 7: current = 4 (popped from input)
  temp.peek() = 3 > 4? NO → push 4
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │ 5 │      │ 4 │
  └───┘      │ 3 │
             │ 2 │
             │ 1 │
             └───┘

─────────────────────────────────────────────────────

Iteration 8: current = 5 (popped from input)
  temp.peek() = 4 > 5? NO → push 5
  
  Input:     Temp:
  ┌───┐      ┌───┐
  │   │      │ 5 │
  └───┘      │ 4 │
  (empty)    │ 3 │
             │ 2 │
             │ 1 │
             └───┘

═══════════════════════════════════════════════════════
  TRANSFER BACK (temp → input)
═══════════════════════════════════════════════════════

  Result (Input Stack):
  ┌───┐
  │ 1 │ ← top (smallest)
  │ 2 │
  │ 3 │
  │ 4 │
  │ 5 │ ← bottom (largest)
  └───┘  ✓ SORTED!
```

---

## Approach 2: Recursive Sort

### Pseudocode

```
FUNCTION sortStack_Recursive(stack):
    IF stack.isEmpty():
        RETURN
    
    // Pop top element
    temp ← stack.pop()
    
    // Sort remaining stack
    sortStack_Recursive(stack)
    
    // Insert popped element in sorted position
    sortedInsert(stack, temp)

FUNCTION sortedInsert(stack, item):
    // Base: empty stack or item >= top → push
    IF stack.isEmpty() OR item >= stack.peek():
        stack.push(item)
    ELSE:
        temp ← stack.pop()
        sortedInsert(stack, item)
        stack.push(temp)
```

### Trace for [3, 1, 2] (top = 2)

```
sortStack([3,1,2]):
  pop 2
  sortStack([3,1]):
    pop 1
    sortStack([3]):
      pop 3
      sortStack([]):
        Base case: return    Stack: []
      sortedInsert([], 3):
        Empty → push 3       Stack: [3]
    sortedInsert([3], 1):
      1 < 3 → pop 3         Stack: []
      sortedInsert([], 1):
        Empty → push 1       Stack: [1]
      push 3                  Stack: [1, 3]
  sortedInsert([1,3], 2):
    2 < 3 → pop 3           Stack: [1]
    sortedInsert([1], 2):
      2 >= 1 → push 2       Stack: [1, 2]
    push 3                    Stack: [1, 2, 3]

Result: [1, 2, 3] (top = 3... wait)

Note: This sorts with LARGEST on top.
To get smallest on top, change >= to <= in sortedInsert.
```

---

## Complexity Analysis

### Temporary Stack Approach

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time (Worst)** | O(n²) | Each element may cause all temp elements to move back |
| **Time (Best)** | O(n) | Already sorted in correct order |
| **Space** | O(n) | Temporary stack |

### Recursive Approach

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n²) | Similar to insertion sort |
| **Space** | O(n) | Call stack depth |

### Why O(n²)?

```
Worst case: Reverse sorted input (for ascending sort)

For each element (n elements):
  May need to move up to n elements from temp back to input
  
Total moves: 1 + 2 + 3 + ... + n = n(n-1)/2 = O(n²)

This is essentially INSERTION SORT using stacks!
```

---

## Comparison with Standard Sorting

```
┌─────────────────────┬───────────┬───────────┬─────────────┐
│ Algorithm           │ Best      │ Worst     │ Space       │
├─────────────────────┼───────────┼───────────┼─────────────┤
│ Stack Sort (temp)   │ O(n)      │ O(n²)     │ O(n)        │
│ Stack Sort (recur)  │ O(n)      │ O(n²)     │ O(n)        │
│ Insertion Sort      │ O(n)      │ O(n²)     │ O(1)        │
│ Merge Sort          │ O(n lg n) │ O(n lg n) │ O(n)        │
│ Quick Sort          │ O(n lg n) │ O(n²)     │ O(lg n)     │
└─────────────────────┴───────────┴───────────┴─────────────┘

Stack Sort ≈ Insertion Sort (same complexity class)
```

---

## Variation: Sort in Descending Order

Simply change the comparison direction:

```
FUNCTION sortStackDescending(inputStack):
    tempStack ← empty stack
    
    WHILE inputStack is NOT empty:
        current ← inputStack.pop()
        
        // Change > to < for descending order
        WHILE tempStack is NOT empty AND tempStack.peek() < current:
            inputStack.push(tempStack.pop())
        
        tempStack.push(current)
    
    WHILE tempStack is NOT empty:
        inputStack.push(tempStack.pop())
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Sort stack using only stack operations |
| **Best Approach** | Temp stack (iterative insertion sort) |
| **Core Idea** | Find correct position by shuffling between two stacks |
| **Time Complexity** | O(n²) worst, O(n) best |
| **Space Complexity** | O(n) |
| **Equivalent To** | Insertion sort |
| **Constraint** | Only stack operations allowed |

---

## Quick Revision Questions

1. **What sorting algorithm does stack sort resemble?**
   > Insertion sort — each element is placed in its correct position among the already-sorted elements.

2. **Why is the worst case O(n²)?**
   > In the worst case, inserting each element requires moving all previously sorted elements back and forth between the two stacks.

3. **Can you sort a stack faster than O(n²) using only stack operations?**
   > No. With only push/pop/peek/isEmpty operations and one extra stack, O(n²) is the best achievable worst case.

4. **What does the temporary stack maintain?**
   > A sorted order at all times — elements are always in sorted sequence in the temp stack.

5. **How do you change from ascending to descending sort?**
   > Change the comparison `tempStack.peek() > current` to `tempStack.peek() < current`.

---

[← Previous: Reverse a Stack](04-reverse-stack.md) | [Next: Stack Using Queues →](06-stack-using-queues.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)
