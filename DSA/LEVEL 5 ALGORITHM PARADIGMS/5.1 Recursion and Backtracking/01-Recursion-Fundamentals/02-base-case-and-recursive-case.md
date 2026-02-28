# Chapter 2: Base Case and Recursive Case

## Overview
Every recursive function is built on two pillars: the **base case** (when to stop) and the **recursive case** (how to break down the problem). Understanding how to design these correctly is the most critical skill in recursion.

---

## The Two Pillars

```
┌───────────────────────────────────────────────────────┐
│                 RECURSIVE FUNCTION                     │
│                                                       │
│  ┌─────────────────┐    ┌──────────────────────────┐  │
│  │    BASE CASE     │    │     RECURSIVE CASE       │  │
│  │                 │    │                          │  │
│  │ • Simplest      │    │ • Break problem down     │  │
│  │   input          │    │ • Make recursive call    │  │
│  │ • Direct answer  │    │ • Combine results        │  │
│  │ • STOPS recursion│    │ • Move toward base case  │  │
│  └─────────────────┘    └──────────────────────────┘  │
│         │                          │                   │
│         ▼                          ▼                   │
│    return value              call self(smaller)        │
└───────────────────────────────────────────────────────┘
```

---

## Base Case — The Foundation

The base case is the **simplest version** of the problem that can be answered directly, without any further recursion.

### Guidelines for Choosing Base Cases

```
┌──────────────────────────────────────────────┐
│         HOW TO CHOOSE A BASE CASE            │
│                                              │
│  Ask yourself:                               │
│  1. What is the SMALLEST valid input?        │
│  2. What input gives an OBVIOUS answer?      │
│  3. When should I STOP breaking down?        │
│                                              │
│  Common base cases:                          │
│  • n == 0 or n == 1                          │
│  • array is empty or has 1 element           │
│  • string is empty or has 1 character        │
│  • reached a leaf node in a tree             │
│  • start >= end (pointers crossed)           │
└──────────────────────────────────────────────┘
```

### Examples of Base Cases

| Problem | Base Case | Return Value |
|---------|-----------|--------------|
| Factorial(n) | n == 0 or n == 1 | 1 |
| Fibonacci(n) | n == 0 or n == 1 | n |
| Sum(arr, i) | i >= length | 0 |
| Power(x, n) | n == 0 | 1 |
| Palindrome(s, l, r) | l >= r | true |
| Binary Search(arr, l, r) | l > r | -1 (not found) |

---

## Recursive Case — The Breakdown

The recursive case defines **how to reduce** the problem and **how to use** the result of the smaller problem.

### The Three Steps

```
RECURSIVE CASE:
  Step 1: REDUCE  → Make the problem smaller
  Step 2: RECURSE → Call function on smaller problem  
  Step 3: COMBINE → Use the result to solve current problem
```

### Example: Sum of first n natural numbers

```
sum(n):
    Base case:  if n == 0, return 0
    
    Recursive case:
      Step 1: REDUCE  → n-1 is the smaller problem
      Step 2: RECURSE → result = sum(n-1)
      Step 3: COMBINE → return n + result
```

### Trace: sum(4)

```
sum(4)
│  Is 4 == 0? No → recursive case
│  return 4 + sum(3)
│              │
│              sum(3)
│              │  Is 3 == 0? No
│              │  return 3 + sum(2)
│              │              │
│              │              sum(2)
│              │              │  Is 2 == 0? No
│              │              │  return 2 + sum(1)
│              │              │              │
│              │              │              sum(1)
│              │              │              │  Is 1 == 0? No
│              │              │              │  return 1 + sum(0)
│              │              │              │              │
│              │              │              │              sum(0)
│              │              │              │              │  Is 0 == 0? YES
│              │              │              │              │  return 0  ← BASE
│              │              │              │              │
│              │              │              │  return 1 + 0 = 1
│              │              │              │
│              │              │  return 2 + 1 = 3
│              │              │
│              │  return 3 + 3 = 6
│              │
│  return 4 + 6 = 10  ✓
```

---

## Multiple Base Cases

Some problems need **more than one** base case:

### Example: Fibonacci

```
fib(n):
    if n == 0: return 0     ← Base case 1
    if n == 1: return 1     ← Base case 2
    return fib(n-1) + fib(n-2)  ← Recursive case
```

### Example: Climbing Stairs

```
ways(n):
    if n == 0: return 1     ← Exactly at the top
    if n < 0:  return 0     ← Overshot, invalid
    return ways(n-1) + ways(n-2)
```

### When You Need Multiple Base Cases

```
┌─────────────────────────────────────────────────┐
│         MULTIPLE BASE CASES NEEDED               │
│                                                  │
│  • Recursive case has MULTIPLE recursive calls   │
│    (e.g., f(n-1) + f(n-2) needs n=0 AND n=1)   │
│                                                  │
│  • Different inputs need different handling      │
│    (e.g., empty string vs single character)      │
│                                                  │
│  • Edge cases that could cause errors            │
│    (e.g., negative numbers, null inputs)         │
└─────────────────────────────────────────────────┘
```

---

## Designing Base Case and Recursive Case Together

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────┐
│            DESIGN PROCESS                            │
│                                                     │
│  1. DEFINE: What does f(input) return?              │
│                                                     │
│  2. BASE: What inputs have obvious answers?         │
│     → Usually the smallest/empty inputs             │
│                                                     │
│  3. RECURSIVE: Assume f(smaller_input) works.       │
│     → How do I use it to solve f(input)?            │
│                                                     │
│  4. VERIFY: Does recursive case always move         │
│     toward the base case?                           │
│                                                     │
│  5. TEST: Trace through small examples              │
│     (n=0, n=1, n=2, n=3)                           │
└─────────────────────────────────────────────────────┘
```

### Worked Example: String Length

```
Define:  length(s) returns number of characters in s

Base:    length("") = 0
         (empty string has 0 characters)

Recursive: length(s) = 1 + length(s[1:])
           "Remove first char, count rest, add 1"

Verify:  s gets shorter each time → reaches "" ✓

Trace:   length("abc")
         = 1 + length("bc")
         = 1 + 1 + length("c")
         = 1 + 1 + 1 + length("")
         = 1 + 1 + 1 + 0
         = 3  ✓
```

---

## Common Pitfalls

### Pitfall 1: Missing Base Case
```
// BAD - no base case!
function sum(n):
    return n + sum(n-1)    // When does this stop? NEVER!

// Stack overflow error!
```

### Pitfall 2: Base Case Too Late
```
// BAD - crashes before reaching base case
function factorial(n):
    if n == 0: return 1
    return n * factorial(n - 1)

factorial(-1)  // Goes: -1, -2, -3, ... never reaches 0!

// FIX: Add guard
function factorial(n):
    if n <= 0: return 1     // Handle negative too
    return n * factorial(n - 1)
```

### Pitfall 3: Not Reducing Toward Base Case
```
// BAD - doesn't get smaller!
function mystery(n):
    if n == 1: return 1
    if n % 2 == 0:
        return mystery(n / 2)      // Gets smaller ✓
    else:
        return mystery(3 * n + 1)  // Gets BIGGER! ✗ (might not terminate)
```

---

## Pattern: Head vs Tail Position

```
HEAD RECURSION (process AFTER recursive call):
function print_ascending(n):
    if n == 0: return
    print_ascending(n - 1)    ← Recurse FIRST
    print(n)                  ← Then do work

    Output for n=3: 1 2 3

TAIL RECURSION (process BEFORE recursive call):
function print_descending(n):
    if n == 0: return
    print(n)                  ← Do work FIRST
    print_descending(n - 1)   ← Then recurse

    Output for n=3: 3 2 1
```

```
              HEAD                          TAIL
         ┌──────────┐                 ┌──────────┐
         │ Recurse  │                 │ Do work  │
         │ first    │                 │ first    │
         ├──────────┤                 ├──────────┤
         │ Then     │                 │ Then     │
         │ do work  │                 │ recurse  │
         └──────────┘                 └──────────┘
         
         Work done on                Work done on
         the way BACK                the way DOWN
```

---

## Summary Table

| Concept | Description | Example |
|---------|-------------|---------|
| Base Case | Simplest input, direct answer | factorial(0) = 1 |
| Recursive Case | Break down + recurse + combine | factorial(n) = n * factorial(n-1) |
| Multiple bases | When multiple stopping points needed | fib(0)=0, fib(1)=1 |
| Head recursion | Work after recursive call | Ascending print |
| Tail recursion | Work before recursive call | Descending print |
| Progress | Must move toward base case | n → n-1 → ... → 0 |

---

## Quick Revision Questions

1. **What happens** if we forget the base case in a recursive function?
2. **When do we need** multiple base cases? Give an example.
3. **What is the difference** between head recursion and tail recursion?
4. **Design the base case and recursive case** for: "Count digits in a number"
5. **Why is it important** that the recursive case makes the problem smaller?
6. **Trace through** factorial(5) showing the base case hit and return values.

---

[← Previous: What is Recursion?](01-what-is-recursion.md) | [Next: Call Stack Visualization →](03-call-stack-visualization.md)

[← Back to README](../README.md)
