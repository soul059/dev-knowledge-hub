# Chapter 2: Max Stack

[← Previous: Min Stack](01-min-stack.md) | [Next: Stack with Middle Operations →](03-stack-middle-operations.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

---

## Overview

The **Max Stack** extends the Min Stack concept, supporting retrieval of the **maximum** element. The advanced version (LeetCode #716) also supports `popMax()` — removing the maximum element, which requires more sophisticated design.

---

## Problem Statement

```
Design a stack that supports:
  push(x)   — Push element x
  pop()     — Remove and return the top element
  top()     — Return the top element
  peekMax() — Return the maximum element
  popMax()  — Remove and return the maximum element

If there are multiple maximum elements, popMax removes 
the one closest to the top.
```

---

## Level 1: Max Stack (peekMax only)

Same approach as Min Stack — mirror the comparison.

```
CLASS MaxStack_Basic:
    mainStack ← empty stack
    maxStack  ← empty stack
    
    FUNCTION push(x):
        mainStack.push(x)
        IF maxStack is empty OR x >= maxStack.top():
            maxStack.push(x)
        ELSE:
            maxStack.push(maxStack.top())
    
    FUNCTION pop():
        mainStack.pop()
        maxStack.pop()
    
    FUNCTION top():
        RETURN mainStack.top()
    
    FUNCTION peekMax():
        RETURN maxStack.top()
```

### Trace

```
push(3): main=[3]     max=[3]     peekMax→3
push(5): main=[3,5]   max=[3,5]   peekMax→5
push(1): main=[3,5,1] max=[3,5,5] peekMax→5
push(5): main=[3,5,1,5] max=[3,5,5,5] peekMax→5

     Main     Max
    ┌───┐   ┌───┐
    │ 5 │   │ 5 │
    │ 1 │   │ 5 │
    │ 5 │   │ 5 │
    │ 3 │   │ 3 │
    └───┘   └───┘
```

---

## Level 2: popMax() — The Hard Part

`popMax()` must remove an element that might not be at the top!

### Approach: Two Stacks + Buffer

```
CLASS MaxStack_WithPopMax:
    mainStack ← empty stack
    maxStack  ← empty stack
    
    FUNCTION push(x):
        mainStack.push(x)
        IF maxStack is empty OR x >= maxStack.top():
            maxStack.push(x)
        ELSE:
            maxStack.push(maxStack.top())
    
    FUNCTION pop():
        maxStack.pop()
        RETURN mainStack.pop()
    
    FUNCTION top():
        RETURN mainStack.top()
    
    FUNCTION peekMax():
        RETURN maxStack.top()
    
    FUNCTION popMax():
        maxVal ← maxStack.top()
        buffer ← empty stack     // Temporary storage
        
        // Pop until we find the max element
        WHILE mainStack.top() != maxVal:
            buffer.push(pop())   // Uses our pop() which maintains both stacks
        
        pop()    // Remove the max element itself
        
        // Push back buffered elements
        WHILE buffer NOT empty:
            push(buffer.pop())   // Uses our push() to maintain both stacks
        
        RETURN maxVal
```

### popMax() Trace

```
State: main=[1, 5, 3, 2]  max=[1, 5, 5, 5]

popMax():
  maxVal = 5

  Step 1: Pop until we find 5
    top=2 ≠ 5 → buffer.push(pop()) → buffer=[2]
    top=3 ≠ 5 → buffer.push(pop()) → buffer=[2,3]
    top=5 = 5 → pop()  (remove the max)
    
  main=[1]  max=[1]  buffer=[2,3]

  Step 2: Push back buffered elements
    push(3) → main=[1,3]  max=[1,3]
    push(2) → main=[1,3,2]  max=[1,3,3]

  Result: main=[1,3,2]  max=[1,3,3]  peekMax→3 ✓
```

---

## Level 3: Efficient popMax with Doubly Linked List + TreeMap

For O(log n) `popMax()`:

```
CLASS MaxStack_Efficient:
    dll ← Doubly Linked List    // Maintains insertion order
    treeMap ← TreeMap/BST       // Maps values to list of DLL nodes
    
    FUNCTION push(x):
        node ← dll.addToEnd(x)
        treeMap.add(x, node)
    
    FUNCTION pop():
        node ← dll.removeLast()
        treeMap.remove(node.val, node)
        RETURN node.val
    
    FUNCTION top():
        RETURN dll.last().val
    
    FUNCTION peekMax():
        RETURN treeMap.lastKey()    // Largest key in BST
    
    FUNCTION popMax():
        maxVal ← treeMap.lastKey()
        node ← treeMap.getLastNode(maxVal)   // Most recent (closest to top)
        dll.remove(node)                       // O(1) with DLL
        treeMap.remove(maxVal, node)           // O(log n) in BST
        RETURN maxVal
```

---

## Complexity Comparison

```
┌────────────────┬─────────────────┬──────────────────────┐
│ Operation      │ Two Stacks      │ DLL + TreeMap        │
├────────────────┼─────────────────┼──────────────────────┤
│ push           │ O(1)            │ O(log n)             │
│ pop            │ O(1)            │ O(log n)             │
│ top            │ O(1)            │ O(1)                 │
│ peekMax        │ O(1)            │ O(log n)             │
│ popMax         │ O(n) worst case │ O(log n)             │
└────────────────┴─────────────────┴──────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Stack with max retrieval and removal |
| **Basic (peekMax)** | Mirror of Min Stack, O(1) all ops |
| **popMax (simple)** | Buffer approach, O(n) for popMax |
| **popMax (efficient)** | DLL + TreeMap, O(log n) all ops |
| **LeetCode** | #716 |

---

## Quick Revision Questions

1. **How does the Max Stack differ from Min Stack in implementation?**
   > Only the comparison is flipped: track maximum instead of minimum using `>=` instead of `<=`.

2. **Why is popMax() harder than peekMax()?**
   > The max might not be at the top, so we need to dig into the stack to remove it, then restore the remaining elements.

3. **What is the time complexity of popMax() using the buffer approach?**
   > O(n) worst case — when the max is at the bottom, we must pop and re-push all elements.

4. **How does the DLL + TreeMap approach achieve O(log n) popMax?**
   > TreeMap finds the max in O(log n), DLL allows O(1) node removal, and TreeMap entry removal is O(log n).

5. **If there are duplicate maximums, which one does popMax() remove?**
   > The one closest to the top (most recently pushed).

---

[← Previous: Min Stack](01-min-stack.md) | [Next: Stack with Middle Operations →](03-stack-middle-operations.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)
