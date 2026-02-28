# Chapter 5: Print N to 1 and 1 to N

## Overview
These two problems beautifully illustrate the difference between **head recursion** and **tail recursion** — doing work BEFORE vs AFTER the recursive call. Understanding this distinction unlocks a deeper intuition about how recursion flows.

---

## Problem 1: Print N down to 1

### Pseudocode (Tail Recursion — work first, then recurse)
```
function printDescending(n):
    if n <= 0: return          // Base case
    print(n)                   // Do work FIRST
    printDescending(n - 1)     // Then recurse
```

### Trace: printDescending(5)

```
printDescending(5)
│  print 5  ← OUTPUT
│  printDescending(4)
│  │  print 4  ← OUTPUT
│  │  printDescending(3)
│  │  │  print 3  ← OUTPUT
│  │  │  printDescending(2)
│  │  │  │  print 2  ← OUTPUT
│  │  │  │  printDescending(1)
│  │  │  │  │  print 1  ← OUTPUT
│  │  │  │  │  printDescending(0) → BASE, return
│  │  │  │  │  return
│  │  │  │  return
│  │  │  return
│  │  return
│  return

Output: 5 4 3 2 1
(Prints on the way DOWN ↓)
```

---

## Problem 2: Print 1 to N

### Pseudocode (Head Recursion — recurse first, then work)
```
function printAscending(n):
    if n <= 0: return          // Base case
    printAscending(n - 1)     // Recurse FIRST
    print(n)                   // Then do work
```

### Trace: printAscending(5)

```
printAscending(5)
│  printAscending(4)
│  │  printAscending(3)
│  │  │  printAscending(2)
│  │  │  │  printAscending(1)
│  │  │  │  │  printAscending(0) → BASE, return
│  │  │  │  │  print 1  ← OUTPUT
│  │  │  │  │  return
│  │  │  │  print 2  ← OUTPUT
│  │  │  │  return
│  │  │  print 3  ← OUTPUT
│  │  │  return
│  │  print 4  ← OUTPUT
│  │  return
│  print 5  ← OUTPUT
│  return

Output: 1 2 3 4 5
(Prints on the way BACK UP ↑)
```

---

## Visual Comparison

```
┌─────────────────────────────────────────────────────┐
│         TAIL RECURSION              HEAD RECURSION    │
│        (Print N to 1)              (Print 1 to N)    │
│                                                     │
│  print(5) ──────── call(4)    call(4) ──────── print(5) │
│  print(4) ──────── call(3)    call(3) ──────── print(4) │
│  print(3) ──────── call(2)    call(2) ──────── print(3) │
│  print(2) ──────── call(1)    call(1) ──────── print(2) │
│  print(1) ──────── call(0)    call(0) ──────── print(1) │
│                  BASE                BASE              │
│                                                     │
│  Work happens        Work happens                    │
│  GOING DOWN          COMING BACK UP                   │
│  (before call)       (after call)                    │
└─────────────────────────────────────────────────────┘
```

---

## Both in One Function

### Print N to 1 AND 1 to N together:

```
function printBoth(n):
    if n <= 0: return
    print(n)              // Print on way DOWN  → N to 1
    printBoth(n - 1)
    print(n)              // Print on way UP    → 1 to N
```

### Trace: printBoth(3)

```
printBoth(3)
│  print 3        ← WAY DOWN
│  printBoth(2)
│  │  print 2     ← WAY DOWN
│  │  printBoth(1)
│  │  │  print 1  ← WAY DOWN
│  │  │  printBoth(0) → BASE
│  │  │  print 1  ← WAY UP
│  │  print 2     ← WAY UP
│  print 3        ← WAY UP

Output: 3 2 1 1 2 3
        ─────┘└─────
        Down    Up
```

---

## Using Index Parameter

### Alternative approach with starting index:

```
// Print 1 to N using increasing index
function printUp(i, n):
    if i > n: return
    print(i)
    printUp(i + 1, n)

// Print N to 1 using decreasing index
function printDown(i):
    if i <= 0: return
    print(i)
    printDown(i - 1)
```

---

## Key Insight: Pre-order vs Post-order

```
This concept extends to TREES:

       ┌───┐
       │ A │
       ├───┤
      / \
   ┌───┐ ┌───┐
   │ B │ │ C │
   └───┘ └───┘

Pre-order  (process BEFORE children): A B C  [like N to 1]
Post-order (process AFTER children):  B C A  [like 1 to N]
In-order   (process BETWEEN children): B A C

Understanding print before/after recursion
→ Understanding tree traversal orders!
```

---

## Complexity Analysis

```
Both approaches:
  Time:  O(n) — n function calls, each does O(1) work
  Space: O(n) — n stack frames at maximum depth

Recurrence: T(n) = T(n-1) + O(1) = O(n)
```

---

## Summary Table

| Aspect | Print N to 1 | Print 1 to N |
|--------|-------------|-------------|
| Type | Tail recursion | Head recursion |
| Work timing | Before recursive call | After recursive call |
| Direction | Way down | Way back up |
| Tree analogy | Pre-order | Post-order |
| Output for n=4 | 4 3 2 1 | 1 2 3 4 |
| Complexity | O(n) time, O(n) space | O(n) time, O(n) space |

---

## Quick Revision Questions

1. **What is the difference** between doing work before vs after the recursive call?
2. **Why does printAscending print** in ascending order even though n decreases?
3. **Trace printBoth(4)** and show the complete output.
4. **How does this concept** relate to tree traversal orders?
5. **Can you print 1 to N** without head recursion? (Hint: use an increasing parameter)
6. **What is the space complexity** of both approaches and why are they the same?

---

[← Previous: Power Function](04-power-function.md) | [Next: Reverse a String →](06-reverse-a-string.md)

[← Back to README](../README.md)
