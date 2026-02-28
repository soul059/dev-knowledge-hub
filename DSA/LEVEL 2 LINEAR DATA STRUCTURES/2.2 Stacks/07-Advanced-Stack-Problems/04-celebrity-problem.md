# Chapter 4: Celebrity Problem

[← Previous: Stack with Middle Operations](03-stack-middle-operations.md) | [Next: Asteroid Collision →](05-asteroid-collision.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

---

## Overview

The **Celebrity Problem** asks: in a group of n people, find the **celebrity** — a person who is known by everyone but knows nobody. This can be elegantly solved using a **stack-based elimination** strategy in O(n) time.

---

## Problem Statement

```
Given: n people (0 to n-1) and a function knows(a, b) that
returns true if person a knows person b.

A CELEBRITY is someone who:
  1. Is known by ALL other n-1 people
  2. Does NOT know any other person

There can be at most ONE celebrity. There might be none.

Example (n=4):
  knows matrix:
       0  1  2  3
   0 [ 0  0  1  0 ]    Person 0 knows Person 2
   1 [ 0  0  1  0 ]    Person 1 knows Person 2
   2 [ 0  0  0  0 ]    Person 2 knows nobody ← Celebrity!
   3 [ 0  0  1  0 ]    Person 3 knows Person 2

  Person 2: Known by everyone (0,1,3), knows nobody → Celebrity ✓
```

---

## Key Insight: Stack Elimination

```
┌──────────────────────────────────────────────────────────┐
│  For any two people A and B:                             │
│                                                          │
│  If A knows B → A is NOT the celebrity                   │
│                 (celebrities know nobody)                │
│                                                          │
│  If A doesn't know B → B is NOT the celebrity            │
│                 (everyone knows the celebrity)            │
│                                                          │
│  Each comparison ELIMINATES exactly ONE candidate!       │
│  n-1 comparisons → 1 candidate remains                  │
│  Then VERIFY the candidate with 2(n-1) checks.          │
└──────────────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION findCelebrity(n):
    // Phase 1: Push all candidates onto stack
    stack ← empty stack
    FOR i = 0 TO n-1:
        stack.push(i)
    
    // Phase 2: Eliminate candidates
    WHILE stack.size() > 1:
        a ← stack.pop()
        b ← stack.pop()
        
        IF knows(a, b):
            stack.push(b)    // A is eliminated (knows someone)
        ELSE:
            stack.push(a)    // B is eliminated (not known by A)
    
    // Phase 3: Verify the remaining candidate
    candidate ← stack.pop()
    
    FOR i = 0 TO n-1:
        IF i != candidate:
            IF knows(candidate, i):
                RETURN -1    // Candidate knows someone → not celebrity
            IF NOT knows(i, candidate):
                RETURN -1    // Someone doesn't know candidate → not celebrity
    
    RETURN candidate
```

---

## Detailed Trace

### n=5, Celebrity = Person 3

```
knows matrix:
     0  1  2  3  4
 0 [ 0  1  0  1  0 ]
 1 [ 0  0  0  1  0 ]
 2 [ 1  0  0  1  0 ]
 3 [ 0  0  0  0  0 ]   ← knows nobody
 4 [ 0  0  1  1  0 ]

Phase 1: Push all → Stack: [0, 1, 2, 3, 4]

Phase 2: Eliminate
  ┌─────────────────────────────────────────────────────┐
  │ Pop a=4, b=3                                        │
  │ knows(4,3)? YES → 4 eliminated, push 3             │
  │ Stack: [0, 1, 2, 3]                                 │
  │                                                     │
  │ Pop a=3, b=2                                        │
  │ knows(3,2)? NO → 2 eliminated, push 3              │
  │ Stack: [0, 1, 3]                                    │
  │                                                     │
  │ Pop a=3, b=1                                        │
  │ knows(3,1)? NO → 1 eliminated, push 3              │
  │ Stack: [0, 3]                                       │
  │                                                     │
  │ Pop a=3, b=0                                        │
  │ knows(3,0)? NO → 0 eliminated, push 3              │
  │ Stack: [3]                                          │
  └─────────────────────────────────────────────────────┘

Phase 3: Verify candidate = 3
  i=0: knows(3,0)? NO ✓, knows(0,3)? YES ✓
  i=1: knows(3,1)? NO ✓, knows(1,3)? YES ✓
  i=2: knows(3,2)? NO ✓, knows(2,3)? YES ✓
  i=4: knows(3,4)? NO ✓, knows(4,3)? YES ✓
  
  All checks passed → Return 3 ✓
```

---

## Why Verification is Necessary

```
Elimination only guarantees the candidate MIGHT be celebrity.
It doesn't prove all conditions hold.

Example with NO celebrity (n=3):
  knows(0,1)=true, knows(1,2)=true, knows(2,0)=true
  
  Phase 2: Eliminate
    Pop 2,1: knows(2,1)? NO → push 2    Stack: [0,2]
    Pop 2,0: knows(2,0)? YES → push 0   Stack: [0]
  
  Candidate = 0
  Verify: knows(0,1)? YES → 0 knows someone!
  → Return -1 (no celebrity) ✓
```

---

## Alternative: Two-Pointer (No Stack)

```
FUNCTION findCelebrity_TwoPointer(n):
    candidate ← 0
    
    FOR i = 1 TO n-1:
        IF knows(candidate, i):
            candidate ← i    // Current eliminated
    
    // Verify candidate
    FOR i = 0 TO n-1:
        IF i != candidate:
            IF knows(candidate, i) OR NOT knows(i, candidate):
                RETURN -1
    
    RETURN candidate
```

This is equivalent to the stack approach but uses O(1) space.

---

## Complexity Analysis

| Approach | Time | Space | knows() calls |
|----------|------|-------|---------------|
| **Brute Force** | O(n²) | O(1) | n² |
| **Stack** | O(n) | O(n) | 3(n-1) |
| **Two Pointer** | O(n) | O(1) | 3(n-1) |

```
Call breakdown:
  Phase 2 (eliminate): n-1 calls
  Phase 3 (verify):    2(n-1) calls
  Total:               3(n-1) calls = O(n)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Find person known by all, who knows nobody |
| **Key Insight** | Each comparison eliminates one candidate |
| **Phase 1** | Push all onto stack |
| **Phase 2** | Pop pairs, eliminate one, push survivor |
| **Phase 3** | Verify the final candidate |
| **Time** | O(n) |
| **knows() calls** | 3(n-1) |

---

## Quick Revision Questions

1. **Why can there be at most one celebrity?**
   > If A and B are both celebrities, A must know B (everyone knows celebrity B) and not know B (celebrities know nobody) — contradiction.

2. **What does each comparison in Phase 2 accomplish?**
   > It eliminates exactly one person from being the celebrity.

3. **Why is Phase 3 (verification) necessary?**
   > The elimination only narrows to a candidate; it doesn't verify that everyone knows the candidate or that the candidate knows nobody.

4. **How many total calls to knows() does the algorithm make?**
   > 3(n-1): n-1 for elimination + 2(n-1) for verification.

5. **What does the algorithm return when there is no celebrity?**
   > -1, detected during the verification phase when the candidate fails a check.

---

[← Previous: Stack with Middle Operations](03-stack-middle-operations.md) | [Next: Asteroid Collision →](05-asteroid-collision.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)
