# Chapter 3: Call Stack Visualization

## Overview
Understanding the **call stack** is essential for mastering recursion. Every function call creates a **stack frame** that stores local variables, parameters, and the return address. Recursive calls stack on top of each other and unwind in reverse order.

---

## What is the Call Stack?

The call stack is a **LIFO (Last-In-First-Out)** data structure used by the runtime to manage function calls.

```
┌──────────────────────────────────────────────┐
│               THE CALL STACK                  │
│                                              │
│  Think of it as a stack of plates:           │
│                                              │
│    ┌──────────────┐                          │
│    │  newest call  │  ← TOP (active)         │
│    ├──────────────┤                          │
│    │  previous     │                          │
│    ├──────────────┤                          │
│    │  earlier      │                          │
│    ├──────────────┤                          │
│    │  main()       │  ← BOTTOM               │
│    └──────────────┘                          │
│                                              │
│  New calls PUSH onto the top                 │
│  Returning POP from the top                  │
└──────────────────────────────────────────────┘
```

---

## Stack Frame Contents

Each function call creates a **stack frame** containing:

```
┌─────────────────────────────────┐
│         STACK FRAME             │
│                                 │
│  • Function name                │
│  • Parameters (input values)    │
│  • Local variables              │
│  • Return address               │
│    (where to go back)           │
│  • Return value                 │
│    (when function completes)    │
└─────────────────────────────────┘
```

---

## Full Trace: factorial(4)

```
function factorial(n):
    if n <= 1: return 1
    return n * factorial(n - 1)
```

### Phase 1: WINDING (Pushing frames)

```
Call: factorial(4)
                                    
STEP 1: factorial(4)                 STEP 2: factorial(3)
┌──────────────────┐                ┌──────────────────┐
│ factorial(4)     │                │ factorial(3)     │ ← TOP
│ n = 4            │                ├──────────────────┤
│ waiting...       │                │ factorial(4)     │
└──────────────────┘                │ n = 4, waiting   │
                                    └──────────────────┘

STEP 3: factorial(2)                 STEP 4: factorial(1) ← BASE!
┌──────────────────┐                ┌──────────────────┐
│ factorial(2)     │ ← TOP          │ factorial(1)     │ ← TOP (BASE)
├──────────────────┤                ├──────────────────┤
│ factorial(3)     │                │ factorial(2)     │
├──────────────────┤                ├──────────────────┤
│ factorial(4)     │                │ factorial(3)     │
│ n = 4, waiting   │                ├──────────────────┤
└──────────────────┘                │ factorial(4)     │
                                    └──────────────────┘
                                    
          Stack is at MAXIMUM depth = 4 frames
```

### Phase 2: UNWINDING (Popping frames & returning)

```
STEP 5: factorial(1) returns 1       STEP 6: factorial(2) returns 2
┌──────────────────┐                ┌──────────────────┐
│ factorial(1) = 1 │ ← POP!        │ factorial(2)     │
├──────────────────┤                │ = 2 * 1 = 2     │ ← POP!
│ factorial(2)     │                ├──────────────────┤
├──────────────────┤                │ factorial(3)     │
│ factorial(3)     │                ├──────────────────┤
├──────────────────┤                │ factorial(4)     │
│ factorial(4)     │                └──────────────────┘
└──────────────────┘                

STEP 7: factorial(3) returns 6       STEP 8: factorial(4) returns 24
┌──────────────────┐                ┌──────────────────┐
│ factorial(3)     │                │ factorial(4)     │
│ = 3 * 2 = 6     │ ← POP!        │ = 4 * 6 = 24    │ ← POP!
├──────────────────┤                └──────────────────┘
│ factorial(4)     │                
└──────────────────┘                Stack is EMPTY, return 24
```

### Complete Timeline

```
TIME ──────────────────────────────────────────────────────►

WINDING (going deeper)              UNWINDING (returning)
─────────────────────               ──────────────────────
factorial(4) called                 factorial(4) returns 24
  factorial(3) called                 factorial(3) returns 6
    factorial(2) called                 factorial(2) returns 2
      factorial(1) called                factorial(1) returns 1
         ▼ BASE CASE HIT ▼
         
         ◄─── TURNING POINT ───►
```

---

## Memory Usage Visualization

```
  Memory Address
  (conceptual)
       │
  HIGH │  ┌──────────────┐
       │  │ factorial(1)  │  Frame 4 (last pushed)
       │  │ n=1, ret=1    │
       │  ├──────────────┤
       │  │ factorial(2)  │  Frame 3
       │  │ n=2           │
       │  ├──────────────┤
       │  │ factorial(3)  │  Frame 2
       │  │ n=3           │
       │  ├──────────────┤
       │  │ factorial(4)  │  Frame 1
       │  │ n=4           │
       │  ├──────────────┤
       │  │ main()        │  Frame 0
       │  └──────────────┘
  LOW  │
       ▼
       
  Stack grows DOWNWARD in memory (toward lower addresses)
  Each frame ≈ 16-64 bytes depending on variables
```

---

## Stack Overflow

When recursion goes too deep, the stack runs out of space:

```
function infinite(n):
    return infinite(n + 1)    // Never stops!

┌──────────────┐
│ infinite(?)  │  ← STACK OVERFLOW!
├──────────────┤     No more memory!
│ infinite(?)  │
├──────────────┤
│ ... 10000+   │
│   frames ... │
├──────────────┤
│ infinite(2)  │
├──────────────┤
│ infinite(1)  │
├──────────────┤
│ main()       │
└──────────────┘

Typical stack limit: ~1000-10000 frames
(varies by language and system)
```

### Stack Size Limits

| Language | Default Stack Depth | Approximate |
|----------|-------------------|-------------|
| Python | ~1000 | `sys.setrecursionlimit()` |
| Java | ~5000-10000 | `-Xss` flag |
| C/C++ | ~10000+ | OS dependent |
| JavaScript | ~10000-25000 | Engine dependent |

---

## Trace Template

Use this template to trace any recursive function:

```
┌────────┬──────────┬──────────┬───────────┬──────────┐
│  Call  │  Input   │ Base?    │ Recursive │  Return  │
│  #     │          │          │ Call      │  Value   │
├────────┼──────────┼──────────┼───────────┼──────────┤
│  1     │  n=4     │  No      │ fact(3)   │ 4*6=24   │
│  2     │  n=3     │  No      │ fact(2)   │ 3*2=6    │
│  3     │  n=2     │  No      │ fact(1)   │ 2*1=2    │
│  4     │  n=1     │  YES     │  —        │ 1        │
└────────┴──────────┴──────────┴───────────┴──────────┘
         ──── WINDING ────►  ◄──── UNWINDING ────
```

---

## Multiple Recursive Calls: fib(4) Stack

Functions with **two recursive calls** are more complex:

```
function fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)
```

```
fib(4)
├── fib(3)                          Stack during fib(3):
│   ├── fib(2)                      ┌──────────┐
│   │   ├── fib(1) → returns 1      │ fib(1)   │
│   │   │                           ├──────────┤
│   │   └── fib(0) → returns 0      │ fib(2)   │
│   │   returns 1+0 = 1             ├──────────┤
│   │                               │ fib(3)   │
│   └── fib(1) → returns 1          ├──────────┤
│   returns 1+1 = 2                 │ fib(4)   │
│                                   └──────────┘
├── fib(2)                          Max depth = 4
│   ├── fib(1) → returns 1
│   └── fib(0) → returns 0
│   returns 1+0 = 1
│
returns 2+1 = 3

[!] Stack grows and shrinks multiple times!
[!] Max depth = n (not number of total calls)
```

---

## Space Complexity Connection

The **maximum stack depth** determines the **space complexity** of recursion:

```
┌─────────────────────────────────────────────────┐
│         SPACE COMPLEXITY = MAX STACK DEPTH        │
│                                                  │
│  factorial(n):   depth = n      → O(n) space     │
│  fib(n):         depth = n      → O(n) space     │
│  binary_search:  depth = log n  → O(log n) space │
│  tower_hanoi:    depth = n      → O(n) space     │
│                                                  │
│  Note: Space ≠ Total calls                       │
│  fib(n) makes O(2^n) calls but depth is O(n)    │
└─────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Call Stack | LIFO structure managing function calls |
| Stack Frame | Contains parameters, locals, return address |
| Winding | Phase where calls are pushed (going deeper) |
| Unwinding | Phase where frames pop (results return up) |
| Max Depth | Determines space complexity |
| Stack Overflow | When recursion exceeds stack memory |
| Space = O(depth) | Not total calls, but max simultaneous frames |

---

## Quick Revision Questions

1. **What data structure** does the runtime use to manage recursive calls?
2. **What information** is stored in each stack frame?
3. **In which order** are stack frames removed — first-in-first-out or last-in-first-out?
4. **What causes** a stack overflow in recursion?
5. **If fib(n) makes O(2^n) calls**, why is the space complexity only O(n)?
6. **Draw the call stack** at its maximum depth for factorial(5).

---

[← Previous: Base Case and Recursive Case](02-base-case-and-recursive-case.md) | [Next: Recursion vs Iteration →](04-recursion-vs-iteration.md)

[← Back to README](../README.md)
