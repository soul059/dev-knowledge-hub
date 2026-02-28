# Chapter 5: Prefix Evaluation

[← Previous: Postfix Evaluation](04-postfix-evaluation.md) | [Next: Expression Tree →](06-expression-tree.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)

---

## Overview

Evaluating a **prefix expression** (Polish Notation) uses a stack just like postfix, but scans **right to left**. Where postfix pushes operands and applies operators to the two most recent, prefix scans backwards to achieve the same effect.

---

## Algorithm

```
┌──────────────────────────────────────────────────────┐
│             PREFIX EVALUATION RULES                  │
│                                                      │
│  Scan RIGHT to LEFT:                                 │
│                                                      │
│  1. OPERAND → Push onto stack                        │
│                                                      │
│  2. OPERATOR → Pop two operands, apply operator,     │
│                push result back                      │
│                                                      │
│     Important: First popped  = LEFT operand (op1)    │
│                Second popped = RIGHT operand (op2)   │
│                Result = op1 OPERATOR op2             │
│                                                      │
│  3. END → Stack top contains the final result        │
│                                                      │
│  NOTE: Operand order is OPPOSITE to postfix!         │
└──────────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION evaluatePrefix(expression):
    stack ← empty stack
    
    // Scan RIGHT to LEFT
    FOR i = length(expression) - 1 DOWNTO 0:
        token ← expression[i]
        
        IF token is a NUMBER:
            stack.push(token)
        ELSE:
            // token is an operator
            op1 ← stack.pop()    // LEFT operand (popped first)
            op2 ← stack.pop()    // RIGHT operand (popped second)
            
            result ← applyOperator(op1, token, op2)
            stack.push(result)
    
    RETURN stack.pop()
```

---

## Operand Order: Prefix vs Postfix

```
┌───────────────────────────────────────────────────────────┐
│           CRITICAL DIFFERENCE IN POP ORDER               │
│                                                           │
│  POSTFIX (L→R scan):          POSTFIX (L→R scan):        │
│    First pop  = op2 (RIGHT)    "A B -" → Pop B, Pop A    │
│    Second pop = op1 (LEFT)              → A - B          │
│                                                           │
│  PREFIX (R→L scan):           PREFIX (R→L scan):         │
│    First pop  = op1 (LEFT)     "- A B" → scan R→L:      │
│    Second pop = op2 (RIGHT)     B→push, A→push           │
│                                 - → Pop A(op1), Pop B(op2)│
│                                → A - B                    │
│                                                           │
│  Both give A - B for the same expression!                │
└───────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Traces

### Trace 1: `* + 3 4 2` = (3 + 4) * 2 = 14

```
Scan RIGHT to LEFT: 2, 4, 3, +, *

┌──────┬───────┬────────────────────────┬──────────────┐
│ Step │ Token │ Action                 │ Stack        │
├──────┼───────┼────────────────────────┼──────────────┤
│  1   │  2    │ Push 2                 │ [2]          │
│  2   │  4    │ Push 4                 │ [2, 4]       │
│  3   │  3    │ Push 3                 │ [2, 4, 3]    │
│  4   │  +    │ Pop 3(L), Pop 4(R)    │ [2, 7]       │
│      │       │ 3 + 4 = 7, Push 7     │              │
│  5   │  *    │ Pop 7(L), Pop 2(R)    │ [14]         │
│      │       │ 7 * 2 = 14, Push 14   │              │
└──────┴───────┴────────────────────────┴──────────────┘

Result: 14 ✓
```

### Trace 2: `- + 5 * 1 2 3` = (5 + 1*2) - 3 = 4

```
Expression tree: - (+ 5 (* 1 2)) 3
Scan R→L: 3, 2, 1, *, 5, +, -

┌──────┬───────┬─────────────────────────┬──────────────────┐
│ Step │ Token │ Action                  │ Stack            │
├──────┼───────┼─────────────────────────┼──────────────────┤
│  1   │  3    │ Push 3                  │ [3]              │
│  2   │  2    │ Push 2                  │ [3, 2]           │
│  3   │  1    │ Push 1                  │ [3, 2, 1]        │
│  4   │  *    │ Pop 1(L), Pop 2(R)     │ [3, 2]           │
│      │       │ 1 * 2 = 2, Push 2      │                  │
│  5   │  5    │ Push 5                  │ [3, 2, 5]        │
│  6   │  +    │ Pop 5(L), Pop 2(R)     │ [3, 7]           │
│      │       │ 5 + 2 = 7, Push 7      │                  │
│  7   │  -    │ Pop 7(L), Pop 3(R)     │ [4]              │
│      │       │ 7 - 3 = 4, Push 4      │                  │
└──────┴───────┴─────────────────────────┴──────────────────┘

Result: 4 ✓
```

### Trace 3: `^ 2 + 1 2` = 2 ^ (1+2) = 8

```
Scan R→L: 2, 1, +, 2, ^

┌──────┬───────┬────────────────────────┬──────────────┐
│ Step │ Token │ Action                 │ Stack        │
├──────┼───────┼────────────────────────┼──────────────┤
│  1   │  2    │ Push 2                 │ [2]          │
│  2   │  1    │ Push 1                 │ [2, 1]       │
│  3   │  +    │ Pop 1(L), Pop 2(R)    │ [3]          │
│      │       │ 1 + 2 = 3, Push 3     │              │
│  4   │  2    │ Push 2                 │ [3, 2]       │
│  5   │  ^    │ Pop 2(L), Pop 3(R)    │ [8]          │
│      │       │ 2 ^ 3 = 8, Push 8     │              │
└──────┴───────┴────────────────────────┴──────────────┘

Result: 8 ✓
```

---

## Comparison: Postfix vs Prefix Evaluation

```
┌────────────────────┬─────────────────────┬─────────────────────┐
│ Aspect             │ Postfix             │ Prefix              │
├────────────────────┼─────────────────────┼─────────────────────┤
│ Scan direction     │ Left → Right        │ Right → Left        │
│ First pop          │ Right operand       │ Left operand        │
│ Second pop         │ Left operand        │ Right operand       │
│ Time complexity    │ O(n)                │ O(n)                │
│ Space complexity   │ O(n)                │ O(n)                │
│ Simplicity         │ Slightly simpler    │ Slightly harder     │
│ Common usage       │ More common         │ Less common         │
└────────────────────┴─────────────────────┴─────────────────────┘
```

---

## Alternative: Recursive Prefix Evaluation

```
FUNCTION evalPrefixRecursive(tokens, index):
    token ← tokens[index]
    index ← index + 1
    
    IF token is a NUMBER:
        RETURN (token, index)
    
    // token is an operator
    (leftVal, index) ← evalPrefixRecursive(tokens, index)
    (rightVal, index) ← evalPrefixRecursive(tokens, index)
    
    result ← applyOperator(leftVal, token, rightVal)
    RETURN (result, index)

// Usage:
(result, _) ← evalPrefixRecursive(tokens, 0)

This works because prefix notation is naturally recursive:
  op operand1 operand2
  where each operand can itself be a sub-expression.
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Single right-to-left pass |
| **Space** | O(n) | Stack stores operands |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Input** | Prefix expression |
| **Output** | Numerical result |
| **Scan** | Right to left |
| **Pop Order** | First pop = left operand, second = right |
| **Time** | O(n) |
| **Space** | O(n) |
| **Alternative** | Recursive evaluation (no explicit stack) |

---

## Quick Revision Questions

1. **What is the scan direction for prefix evaluation?**
   > Right to left (opposite of postfix which scans left to right).

2. **How does the pop order differ from postfix?**
   > In prefix: first pop = left operand, second pop = right operand. In postfix: first pop = right operand, second pop = left operand.

3. **Evaluate `- * 4 5 + 2 3`.**
   > Scan R→L: 3→push, 2→push, +→2+3=5, 5→push, 4→push, *→4*5=20, -→20-5=15. Answer: **15**.

4. **Why can prefix be evaluated recursively without a stack?**
   > Prefix notation is inherently recursive: each operator is followed by its two operand sub-expressions, forming a natural recursive structure.

5. **Which is more commonly used in practice: prefix or postfix evaluation?**
   > Postfix is more common (used in RPN calculators, stack-based VMs, and compiler intermediate code).

---

[← Previous: Postfix Evaluation](04-postfix-evaluation.md) | [Next: Expression Tree →](06-expression-tree.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)
