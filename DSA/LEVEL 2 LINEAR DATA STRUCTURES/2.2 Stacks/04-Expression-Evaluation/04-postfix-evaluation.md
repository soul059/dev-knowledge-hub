# Chapter 4: Postfix Evaluation

[← Previous: Infix to Prefix](03-infix-to-prefix.md) | [Next: Prefix Evaluation →](05-prefix-evaluation.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)

---

## Overview

Evaluating a **postfix expression** (Reverse Polish Notation) is remarkably simple with a stack. No parentheses, no precedence rules — just a clean left-to-right scan. This is why compilers convert infix to postfix before evaluation.

---

## Algorithm

```
┌──────────────────────────────────────────────────────┐
│            POSTFIX EVALUATION RULES                  │
│                                                      │
│  Scan LEFT to RIGHT:                                 │
│                                                      │
│  1. OPERAND → Push onto stack                        │
│                                                      │
│  2. OPERATOR → Pop two operands, apply operator,     │
│                push result back                      │
│                                                      │
│     Important: First popped = RIGHT operand (op2)    │
│                Second popped = LEFT operand (op1)    │
│                Result = op1 OPERATOR op2             │
│                                                      │
│  3. END → Stack top contains the final result        │
└──────────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION evaluatePostfix(expression):
    stack ← empty stack
    
    FOR each token in expression:
        IF token is a NUMBER:
            stack.push(token)
        ELSE:
            // token is an operator
            op2 ← stack.pop()    // RIGHT operand (popped first)
            op1 ← stack.pop()    // LEFT operand (popped second)
            
            result ← applyOperator(op1, token, op2)
            stack.push(result)
    
    RETURN stack.pop()    // Final result

FUNCTION applyOperator(a, operator, b):
    IF operator == '+': RETURN a + b
    IF operator == '-': RETURN a - b
    IF operator == '*': RETURN a * b
    IF operator == '/': RETURN a / b
    IF operator == '^': RETURN a ^ b
```

---

## Step-by-Step Traces

### Trace 1: `3 4 + 2 *` = (3 + 4) * 2 = 14

```
┌──────┬───────┬───────────────────────┬──────────────┐
│ Step │ Token │ Action                │ Stack        │
├──────┼───────┼───────────────────────┼──────────────┤
│  1   │  3    │ Push 3                │ [3]          │
│  2   │  4    │ Push 4                │ [3, 4]       │
│  3   │  +    │ Pop 4, Pop 3          │ [7]          │
│      │       │ 3 + 4 = 7, Push 7    │              │
│  4   │  2    │ Push 2                │ [7, 2]       │
│  5   │  *    │ Pop 2, Pop 7          │ [14]         │
│      │       │ 7 * 2 = 14, Push 14  │              │
└──────┴───────┴───────────────────────┴──────────────┘

Result: 14 ✓

Visual:
  Step 1    Step 2    Step 3    Step 4    Step 5
  ┌───┐    ┌───┐    ┌───┐    ┌───┐    ┌────┐
  │   │    │ 4 │    │   │    │ 2 │    │    │
  │ 3 │    │ 3 │    │ 7 │    │ 7 │    │ 14 │
  └───┘    └───┘    └───┘    └───┘    └────┘
```

### Trace 2: `5 1 2 + 4 * + 3 -` = 5 + (1+2)*4 - 3 = 14

```
┌──────┬───────┬────────────────────────┬──────────────────┐
│ Step │ Token │ Action                 │ Stack            │
├──────┼───────┼────────────────────────┼──────────────────┤
│  1   │  5    │ Push 5                 │ [5]              │
│  2   │  1    │ Push 1                 │ [5, 1]           │
│  3   │  2    │ Push 2                 │ [5, 1, 2]        │
│  4   │  +    │ Pop 2,1 → 1+2=3       │ [5, 3]           │
│  5   │  4    │ Push 4                 │ [5, 3, 4]        │
│  6   │  *    │ Pop 4,3 → 3*4=12      │ [5, 12]          │
│  7   │  +    │ Pop 12,5 → 5+12=17    │ [17]             │
│  8   │  3    │ Push 3                 │ [17, 3]          │
│  9   │  -    │ Pop 3,17 → 17-3=14    │ [14]             │
└──────┴───────┴────────────────────────┴──────────────────┘

Result: 14 ✓
```

### Trace 3: `2 3 ^ 4 -` = 2^3 - 4 = 4

```
┌──────┬───────┬────────────────────────┬──────────────┐
│ Step │ Token │ Action                 │ Stack        │
├──────┼───────┼────────────────────────┼──────────────┤
│  1   │  2    │ Push 2                 │ [2]          │
│  2   │  3    │ Push 3                 │ [2, 3]       │
│  3   │  ^    │ Pop 3,2 → 2^3=8       │ [8]          │
│  4   │  4    │ Push 4                 │ [8, 4]       │
│  5   │  -    │ Pop 4,8 → 8-4=4       │ [4]          │
└──────┴───────┴────────────────────────┴──────────────┘

Result: 4 ✓
```

---

## Operand Order Matters!

```
┌──────────────────────────────────────────────────────────┐
│                    CRITICAL WARNING                      │
│                                                          │
│  For non-commutative operators (-, /, ^):                │
│                                                          │
│    FIRST pop  = RIGHT operand (op2)                      │
│    SECOND pop = LEFT operand  (op1)                      │
│    Result     = op1 ○ op2                                │
│                                                          │
│  Example: "8 3 -"                                        │
│    Pop 3 (right), Pop 8 (left)                          │
│    Result = 8 - 3 = 5  ✓                                │
│    NOT 3 - 8 = -5  ✗                                    │
│                                                          │
│  Example: "10 2 /"                                       │
│    Pop 2 (right), Pop 10 (left)                         │
│    Result = 10 / 2 = 5  ✓                               │
│    NOT 2 / 10 = 0.2  ✗                                  │
└──────────────────────────────────────────────────────────┘
```

---

## Error Handling

```
FUNCTION evaluatePostfix_Safe(expression):
    stack ← empty stack
    
    FOR each token in expression:
        IF token is a NUMBER:
            stack.push(token)
        ELSE IF token is OPERATOR:
            IF stack.size() < 2:
                ERROR "Invalid expression: not enough operands"
            
            op2 ← stack.pop()
            op1 ← stack.pop()
            
            IF token == '/' AND op2 == 0:
                ERROR "Division by zero"
            
            result ← applyOperator(op1, token, op2)
            stack.push(result)
        ELSE:
            ERROR "Invalid token: " + token
    
    IF stack.size() ≠ 1:
        ERROR "Invalid expression: too many operands"
    
    RETURN stack.pop()
```

### Invalid Expression Examples

```
"3 +"       → Error: only 1 operand for binary operator
"3 4 + *"   → Error: only 1 operand for *
"3 4"       → Error: 2 values remain (no operator to combine)
"3 4 + 5"   → Error: 2 values remain after processing
"3 0 /"     → Error: division by zero
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Single left-to-right pass |
| **Space** | O(n) | Stack may hold up to (n+1)/2 operands |

### Why O(n) Space?

```
Worst case for stack size: "1 2 3 4 5 + + + +"

  Stack grows to 5 before any operations:
  [1, 2, 3, 4, 5]  →  size = 5 operands

  For n tokens: at most (n+1)/2 are operands
  (since operators = operands - 1)
```

---

## Practical Applications

```
┌──────────────────────────────────────────────────┐
│  WHERE POSTFIX EVALUATION IS USED                │
│                                                  │
│  1. HP calculators (RPN mode)                    │
│  2. Compiler code generation (intermediate code) │
│  3. PostScript language (page description)       │
│  4. Stack-based VMs (JVM, CLR, Python VM)       │
│  5. Forth programming language                   │
│  6. Spreadsheet formula engines                  │
└──────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Input** | Postfix expression (tokens) |
| **Output** | Numerical result |
| **Algorithm** | Operand→push, Operator→pop two, compute, push result |
| **Scan Direction** | Left to right |
| **Time** | O(n) |
| **Space** | O(n) |
| **Key Pitfall** | Operand order for -, /, ^ |

---

## Quick Revision Questions

1. **Why is postfix evaluation simpler than infix?**
   > No precedence rules or parentheses to handle — just scan left to right, push numbers, and compute on operators.

2. **What determines operand order when popping?**
   > First pop = right operand, second pop = left operand. This matters for non-commutative operators like `-` and `/`.

3. **How many elements should remain on the stack after evaluation?**
   > Exactly 1 (the final result). More or fewer indicates an invalid expression.

4. **Evaluate `6 2 / 3 + 4 *`.**
   > 6/2=3, 3+3=6, 6*4=24. Answer: **24**.

5. **Why do compilers prefer postfix to infix for evaluation?**
   > Postfix maps directly to stack-based instructions, making code generation straightforward. No need to handle precedence during evaluation.

---

[← Previous: Infix to Prefix](03-infix-to-prefix.md) | [Next: Prefix Evaluation →](05-prefix-evaluation.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)
