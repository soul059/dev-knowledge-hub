# Chapter 4: Recursion vs Iteration

## Overview
Every recursive solution can be converted to an iterative one and vice versa. Understanding when to choose each approach is a critical skill. This chapter compares both paradigms with examples, trade-offs, and conversion techniques.

---

## Side-by-Side Comparison

```
┌──────────────────────────┐    ┌──────────────────────────┐
│       RECURSION          │    │       ITERATION           │
│                          │    │                          │
│  function f(n):          │    │  function f(n):          │
│    if n == 0: return 1   │    │    result = 1            │
│    return n * f(n-1)     │    │    for i = 1 to n:       │
│                          │    │      result *= i         │
│                          │    │    return result          │
│  Uses: call stack        │    │    Uses: loop + variable │
│  Control: base case      │    │    Control: condition    │
│  Space: O(n)             │    │    Space: O(1)           │
└──────────────────────────┘    └──────────────────────────┘
```

---

## Detailed Comparison Table

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| **Mechanism** | Function calls itself | Loop repeats code block |
| **Termination** | Base case | Loop condition becomes false |
| **State** | Stored in stack frames | Stored in variables |
| **Space** | O(n) or more (stack) | Usually O(1) |
| **Speed** | Slower (call overhead) | Faster (no overhead) |
| **Readability** | Often cleaner for trees/graphs | Simpler for linear tasks |
| **Risk** | Stack overflow | Infinite loop |
| **Debugging** | Harder (nested calls) | Easier (step through loop) |

---

## Example 1: Factorial

### Recursive
```
function factorial_rec(n):
    if n <= 1: return 1          // Base case
    return n * factorial_rec(n-1) // Recursive case

// Call chain: 5 → 4 → 3 → 2 → 1 → return
// Space: O(n) stack frames
```

### Iterative
```
function factorial_iter(n):
    result = 1
    for i = 2 to n:
        result = result * i
    return result

// Simple loop, Space: O(1)
```

### Execution Comparison for n=5

```
RECURSIVE:                          ITERATIVE:
factorial(5)                        result = 1
  5 * factorial(4)                  i=2: result = 1*2 = 2
    4 * factorial(3)                i=3: result = 2*3 = 6
      3 * factorial(2)              i=4: result = 6*4 = 24
        2 * factorial(1)            i=5: result = 24*5 = 120
          return 1                  return 120
        return 2
      return 6                     5 multiplications
    return 24                      O(1) space
  return 120
                                   
5 stack frames                     
O(n) space                        
```

---

## Example 2: Fibonacci

### Recursive (Naive)
```
function fib_rec(n):
    if n <= 1: return n
    return fib_rec(n-1) + fib_rec(n-2)

// Time: O(2^n) — exponential!
// Space: O(n) stack depth
```

### Iterative
```
function fib_iter(n):
    if n <= 1: return n
    a = 0, b = 1
    for i = 2 to n:
        temp = a + b
        a = b
        b = temp
    return b

// Time: O(n)
// Space: O(1)
```

```
PERFORMANCE COMPARISON (Fibonacci):

n=10:  Recursive calls = 177      Iterative steps = 9
n=20:  Recursive calls = 21,891   Iterative steps = 19
n=30:  Recursive calls = 2.7M     Iterative steps = 29
n=40:  Recursive calls = 331M     Iterative steps = 39
n=50:  Recursive calls = 40B+     Iterative steps = 49

            ┌──────────────────────────────────────┐
  Time      │                              ╱ Recursive
  (log)     │                           ╱   O(2^n)
            │                        ╱
            │                     ╱
            │               ╱──
            │         ╱──
            │    ╱──                    
            │ ──────────────────── Iterative O(n)
            └──────────────────────────────────────┘
                          n →
```

---

## Example 3: Binary Search

### Recursive
```
function binary_search_rec(arr, target, low, high):
    if low > high: return -1
    mid = (low + high) / 2
    if arr[mid] == target: return mid
    if arr[mid] < target:
        return binary_search_rec(arr, target, mid+1, high)
    else:
        return binary_search_rec(arr, target, low, mid-1)
```

### Iterative
```
function binary_search_iter(arr, target):
    low = 0, high = length(arr) - 1
    while low <= high:
        mid = (low + high) / 2
        if arr[mid] == target: return mid
        if arr[mid] < target: low = mid + 1
        else: high = mid - 1
    return -1
```

```
Both: Time O(log n)
Recursive: Space O(log n)  ← stack frames
Iterative: Space O(1)      ← just variables
```

---

## When to Use Which?

```
┌─────────────────────────────────────────────────────────┐
│                 DECISION GUIDE                           │
│                                                         │
│  USE RECURSION when:                                    │
│  ✓ Problem has natural recursive structure              │
│    (trees, graphs, divide-and-conquer)                  │
│  ✓ Code clarity is more important than performance      │
│  ✓ Problem involves backtracking                        │
│  ✓ State management is complex                          │
│  ✓ Subproblems are naturally nested                     │
│                                                         │
│  USE ITERATION when:                                    │
│  ✓ Simple repetition (counting, accumulating)           │
│  ✓ Performance is critical                              │
│  ✓ Stack depth could be very large                      │
│  ✓ Problem is naturally sequential                      │
│  ✓ Space efficiency matters                             │
│                                                         │
│  RECURSION IS BETTER for:     ITERATION IS BETTER for:  │
│  • Tree traversal              • Array processing       │
│  • Graph DFS                   • Simple accumulation    │
│  • Merge sort / Quick sort     • Linear search          │
│  • Backtracking problems       • Mathematical series    │
│  • Fractal generation          • I/O operations         │
└─────────────────────────────────────────────────────────┘
```

---

## Converting Recursion to Iteration

### Technique 1: Simple Loop (Tail Recursion → Loop)

```
// Tail recursive (last action is recursive call)
function sum_rec(n, acc=0):
    if n == 0: return acc
    return sum_rec(n-1, acc + n)

// Converted to iteration
function sum_iter(n):
    acc = 0
    while n > 0:
        acc = acc + n
        n = n - 1
    return acc
```

### Technique 2: Explicit Stack (General Recursion → Stack)

```
// Recursive tree traversal
function inorder(node):
    if node == null: return
    inorder(node.left)
    print(node.val)
    inorder(node.right)

// Iterative with explicit stack
function inorder_iter(root):
    stack = []
    current = root
    while current != null or stack not empty:
        while current != null:
            stack.push(current)
            current = current.left
        current = stack.pop()
        print(current.val)
        current = current.right
```

```
CONVERSION PATTERN:

   Recursive                    Iterative
┌──────────────┐           ┌──────────────────┐
│ Call stack    │    →      │ Explicit stack    │
│ (implicit)   │           │ (you manage it)   │
│              │           │                  │
│ Parameters   │    →      │ Stack entries     │
│ in frames    │           │ or variables      │
│              │           │                  │
│ Base case    │    →      │ Loop condition    │
│ return       │           │ break/continue    │
└──────────────┘           └──────────────────┘
```

---

## Tail Call Optimization (TCO)

Some languages optimize tail recursion to use O(1) space:

```
// TAIL RECURSIVE — last action is the recursive call
function factorial_tail(n, acc=1):
    if n <= 1: return acc
    return factorial_tail(n-1, n * acc)  ← TAIL POSITION

// With TCO, compiler converts this to:
function factorial_optimized(n, acc=1):
    loop:
        if n <= 1: return acc
        acc = n * acc
        n = n - 1
        goto loop

// Result: O(1) space instead of O(n)!
```

### TCO Support by Language

| Language | Tail Call Optimization |
|----------|----------------------|
| Scheme/Lisp | Guaranteed by spec |
| Haskell | Yes |
| Scala | @tailrec annotation |
| C/C++ | Compiler dependent (gcc -O2) |
| Java | No |
| Python | No |
| JavaScript | Only Safari (ES6 spec) |

---

## Summary Table

| Criterion | Recursion | Iteration | Winner |
|-----------|-----------|-----------|--------|
| Code clarity (trees) | Clean | Complex | Recursion |
| Code clarity (arrays) | Unnecessary | Simple | Iteration |
| Space efficiency | O(n) stack | O(1) | Iteration |
| Time efficiency | Call overhead | No overhead | Iteration |
| Tree/Graph problems | Natural fit | Need stack | Recursion |
| Backtracking | Essential | Very hard | Recursion |
| Stack overflow risk | Yes | No | Iteration |
| Tail-call scenarios | Can optimize | Already optimal | Tie |

---

## Quick Revision Questions

1. **What is the main space disadvantage** of recursion compared to iteration?
2. **Convert this recursive function to iterative**: `sum(n) = n + sum(n-1)`, base: `sum(0) = 0`
3. **Why is recursive Fibonacci O(2^n)** while iterative is O(n)?
4. **What is tail recursion** and why is it important?
5. **Name three types of problems** where recursion is preferred over iteration.
6. **How do you convert** a non-tail-recursive function to iterative? What data structure helps?

---

[← Previous: Call Stack Visualization](03-call-stack-visualization.md) | [Next: When to Use Recursion →](05-when-to-use-recursion.md)

[← Back to README](../README.md)
