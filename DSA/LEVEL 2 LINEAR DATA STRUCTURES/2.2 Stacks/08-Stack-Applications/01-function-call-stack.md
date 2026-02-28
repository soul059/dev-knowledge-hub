# Chapter 1: Function Call Stack

[← Previous: Decode String](../07-Advanced-Stack-Problems/06-decode-string.md) | [Next: Undo-Redo Operations →](02-undo-redo-operations.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

---

## Overview

The **call stack** (execution stack) is how programming languages manage **function calls**. Every time a function is called, a **stack frame** is pushed; when it returns, the frame is popped. Understanding this is crucial for grasping recursion, debugging stack overflows, and reasoning about program execution.

---

## What is a Stack Frame?

```
┌──────────────────────────────────────────────────────────┐
│  A STACK FRAME contains everything a function needs:     │
│                                                          │
│  ┌─────────────────────────────────────┐                │
│  │ STACK FRAME for function foo()      │                │
│  ├─────────────────────────────────────┤                │
│  │ Return address (where to go back)   │                │
│  │ Parameters (arguments passed)       │                │
│  │ Local variables                     │                │
│  │ Saved registers                     │                │
│  └─────────────────────────────────────┘                │
│                                                          │
│  When foo() calls bar(): bar's frame is pushed ON TOP    │
│  When bar() returns: bar's frame is popped, back to foo  │
└──────────────────────────────────────────────────────────┘
```

---

## Example: Simple Function Calls

```
FUNCTION main():
    x ← 5
    result ← square(x)
    print(result)

FUNCTION square(n):
    return n × n
```

### Call Stack Execution

```
Step 1: main() is called
  ┌─────────────────┐
  │ main()          │
  │  x = 5          │
  └─────────────────┘

Step 2: main() calls square(5)
  ┌─────────────────┐
  │ square(5)       │  ← active (top)
  │  n = 5          │
  │  return addr →  │  (back to main, line 3)
  ├─────────────────┤
  │ main()          │  ← suspended
  │  x = 5          │
  └─────────────────┘

Step 3: square() returns 25
  ┌─────────────────┐
  │ main()          │  ← resumed
  │  x = 5          │
  │  result = 25    │
  └─────────────────┘

Step 4: print(25) executes, main() returns
  Stack is empty. Program ends.
```

---

## Recursion and the Call Stack

### Factorial Example

```
FUNCTION factorial(n):
    IF n <= 1: RETURN 1
    RETURN n × factorial(n - 1)
```

### Call Stack for factorial(4)

```
Call factorial(4):
  ┌─────────────────┐
  │ factorial(4)    │  4 × factorial(3)...
  └─────────────────┘

Call factorial(3):
  ┌─────────────────┐
  │ factorial(3)    │  3 × factorial(2)...
  │ factorial(4)    │
  └─────────────────┘

Call factorial(2):
  ┌─────────────────┐
  │ factorial(2)    │  2 × factorial(1)...
  │ factorial(3)    │
  │ factorial(4)    │
  └─────────────────┘

Call factorial(1):
  ┌─────────────────┐
  │ factorial(1)    │  returns 1 (base case!)
  │ factorial(2)    │
  │ factorial(3)    │
  │ factorial(4)    │
  └─────────────────┘   ← Maximum stack depth = 4

Now UNWIND (pop frames one by one):

factorial(1) returns 1    → pop
factorial(2) returns 2×1=2  → pop
factorial(3) returns 3×2=6  → pop
factorial(4) returns 4×6=24 → pop

Final result: 24
```

---

## Stack Overflow

```
┌──────────────────────────────────────────────────────────┐
│  STACK OVERFLOW occurs when the call stack exceeds       │
│  its maximum size (typically 1-8 MB).                    │
│                                                          │
│  Common causes:                                          │
│  1. Infinite recursion (missing base case)               │
│  2. Very deep recursion (n too large)                    │
│  3. Large local variables in many frames                 │
│                                                          │
│  Example of infinite recursion:                          │
│    FUNCTION bad():                                       │
│      bad()    // No base case! Never stops!              │
│                                                          │
│  Each call adds a stack frame until memory runs out.     │
│  Typical limit: ~10,000 to ~100,000 frames               │
└──────────────────────────────────────────────────────────┘
```

---

## Tail Recursion

```
Normal recursion:                 Tail recursion:
  FUNCTION fact(n):                FUNCTION fact(n, acc=1):
    IF n<=1: RETURN 1                IF n<=1: RETURN acc
    RETURN n × fact(n-1)             RETURN fact(n-1, n×acc)
    //     ↑                         //  ↑
    // Multiply AFTER return         // Nothing after return
    // Must keep frame               // Can REUSE frame!

Tail Call Optimization (TCO):
  Some languages (Scheme, Haskell, Scala) optimize tail
  calls to reuse the current stack frame, preventing
  stack overflow for deep recursion.
  
  NOT guaranteed in: C, Java, Python, JavaScript
```

---

## Simulating Recursion with Explicit Stack

```
// Recursive version
FUNCTION dfs(node):
    IF node is null: RETURN
    process(node)
    dfs(node.left)
    dfs(node.right)

// Iterative version using explicit stack
FUNCTION dfs_iterative(root):
    stack ← empty stack
    stack.push(root)
    
    WHILE stack NOT empty:
        node ← stack.pop()
        IF node is null: CONTINUE
        process(node)
        stack.push(node.right)    // Push right first
        stack.push(node.left)     // So left is processed first
```

---

## Stack Frame in Memory Layout

```
High Address
┌──────────────────────────┐
│        STACK             │ ← Grows downward
│  ┌──────────────────┐   │
│  │ main() frame     │   │
│  ├──────────────────┤   │
│  │ foo() frame      │   │
│  ├──────────────────┤   │
│  │ bar() frame      │   │  ← Stack Pointer (SP)
│  └──────────────────┘   │
│                          │
│        ↕ Free Space      │
│                          │
│  ┌──────────────────┐   │
│  │      HEAP        │   │ ← Grows upward
│  │  (dynamic alloc) │   │
│  └──────────────────┘   │
│                          │
│  ┌──────────────────┐   │
│  │  STATIC / GLOBAL │   │
│  ├──────────────────┤   │
│  │  CODE (TEXT)      │   │
│  └──────────────────┘   │
Low Address

Stack overflow: Stack grows into heap space!
```

---

## Complexity Implications

```
┌───────────────────────────────────────────────────────────┐
│ Recursive Algorithm    │ Stack Depth │ Implication        │
├────────────────────────┼─────────────┼────────────────────┤
│ factorial(n)           │ O(n)        │ Risk for large n   │
│ fibonacci(n) (naive)   │ O(n)        │ Depth=n, calls=2ⁿ │
│ binary search          │ O(log n)    │ Very safe           │
│ merge sort             │ O(log n)    │ Safe               │
│ quicksort (worst)      │ O(n)        │ Can overflow       │
│ tree DFS               │ O(h)        │ h=height of tree   │
│ tower of hanoi         │ O(n)        │ n disks            │
└────────────────────────┴─────────────┴────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **What** | System stack managing function calls |
| **Frame Contents** | Return address, params, locals, saved registers |
| **Push** | When function is called |
| **Pop** | When function returns |
| **Stack Overflow** | Too many frames (infinite/deep recursion) |
| **Tail Recursion** | Optimization to reuse frames |
| **Explicit Stack** | Convert recursion to iteration |

---

## Quick Revision Questions

1. **What is stored in a stack frame?**
   > Return address, parameters, local variables, and saved registers.

2. **Why does recursion use the call stack?**
   > Each recursive call needs its own set of local variables and a return address; the call stack stores this context for each active call.

3. **What causes a stack overflow?**
   > When the call stack exceeds its size limit, usually due to infinite recursion or extremely deep recursion.

4. **What is tail recursion and why is it beneficial?**
   > When the recursive call is the very last operation. Some compilers optimize this by reusing the current frame, preventing stack growth.

5. **How do you convert recursion to iteration?**
   > Use an explicit stack data structure to manually push/pop the state that the call stack would normally manage.

---

[← Previous: Decode String](../07-Advanced-Stack-Problems/06-decode-string.md) | [Next: Undo-Redo Operations →](02-undo-redo-operations.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)
