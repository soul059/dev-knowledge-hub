# Chapter 1: Infix, Prefix, and Postfix Notation

[← Previous: Stack Using Queues](../03-Basic-Stack-Problems/06-stack-using-queues.md) | [Next: Infix to Postfix Conversion →](02-infix-to-postfix.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)

---

## Overview

Mathematical expressions can be written in three different **notations** based on where the **operator** is placed relative to its **operands**. Understanding these notations is crucial for expression evaluation using stacks.

---

## The Three Notations

```
┌──────────────────────────────────────────────────────────┐
│                    NOTATION TYPES                        │
│                                                          │
│  INFIX:    Operator BETWEEN operands     A + B           │
│  PREFIX:   Operator BEFORE operands      + A B           │
│  POSTFIX:  Operator AFTER operands       A B +           │
│                                                          │
│  Also known as:                                          │
│  PREFIX  = Polish Notation (Jan Łukasiewicz, 1924)       │
│  POSTFIX = Reverse Polish Notation (RPN)                 │
│  INFIX   = Standard mathematical notation                │
└──────────────────────────────────────────────────────────┘
```

---

## Infix Notation

The notation we use every day. Operator sits **between** two operands.

```
Examples:
  A + B
  (A + B) * C
  A * B + C / D
  ((A + B) * (C - D)) / E
```

### Challenges with Infix

```
┌──────────────────────────────────────────────┐
│  1. Requires PRECEDENCE rules                │
│     → * and / before + and -                 │
│                                              │
│  2. Requires ASSOCIATIVITY rules             │
│     → Left-to-right for most operators       │
│     → Right-to-left for ^ (exponentiation)  │
│                                              │
│  3. Requires PARENTHESES for overriding      │
│     → (A + B) * C  vs  A + B * C            │
│                                              │
│  4. AMBIGUOUS without rules                  │
│     → 2 + 3 * 4 = 14 or 20?                 │
│     → Answer: 14 (precedence: * before +)    │
└──────────────────────────────────────────────┘
```

### Operator Precedence and Associativity

```
┌──────────────┬────────────┬───────────────┐
│ Operator     │ Precedence │ Associativity │
├──────────────┼────────────┼───────────────┤
│ ( )          │ Highest    │ —             │
│ ^            │ 3          │ Right → Left  │
│ * /          │ 2          │ Left → Right  │
│ + -          │ 1 (Lowest) │ Left → Right  │
└──────────────┴────────────┴───────────────┘
```

---

## Prefix Notation (Polish Notation)

Operator comes **before** its operands. No parentheses needed!

```
Infix:    A + B       →  Prefix:   + A B
Infix:    A + B * C   →  Prefix:   + A * B C
Infix:    (A + B) * C →  Prefix:   * + A B C

Reading prefix: The operator applies to the NEXT two operands
  + A * B C
  │   └─┬─┘
  │     multiply B and C first
  └── then add A to the result
```

### How to Read Prefix

```
Expression: * + A B - C D

Start from LEFT, find an operator followed by two operands:

  * + A B - C D
      └─┬─┘
    + A B = (A+B)    →   * (A+B) - C D
                               └─┬─┘
                           - C D = (C-D)    →   * (A+B) (C-D)
                                                  └────┬────┘
                                              = (A+B) * (C-D)
```

---

## Postfix Notation (Reverse Polish Notation)

Operator comes **after** its operands. Also needs no parentheses!

```
Infix:    A + B       →  Postfix:  A B +
Infix:    A + B * C   →  Postfix:  A B C * +
Infix:    (A + B) * C →  Postfix:  A B + C *

Reading postfix: The operator applies to the PREVIOUS two operands
  A B C * +
    └─┬─┘
    multiply B and C first
  └─────┬─┘
  then add A to the result
```

### How to Read Postfix

```
Expression: A B + C D - *

Scan from LEFT, when you hit an operator, apply to previous two:

  A B + C D - *
  └─┬─┘
  A B + = (A+B)    →   (A+B) C D - *
                              └─┬─┘
                          C D - = (C-D)    →   (A+B) (C-D) *
                                                └─────┬─────┘
                                            = (A+B) * (C-D)
```

---

## Conversion Examples

```
┌──────────────────────┬────────────────────┬────────────────────┐
│ Infix                │ Prefix             │ Postfix            │
├──────────────────────┼────────────────────┼────────────────────┤
│ A + B                │ + A B              │ A B +              │
│ A + B * C            │ + A * B C          │ A B C * +          │
│ (A + B) * C          │ * + A B C          │ A B + C *          │
│ A * B + C * D        │ + * A B * C D      │ A B * C D * +      │
│ (A + B) * (C - D)    │ * + A B - C D      │ A B + C D - *      │
│ A + B * C - D / E    │ - + A * B C / D E  │ A B C * + D E / -  │
│ A ^ B ^ C            │ ^ A ^ B C          │ A B C ^ ^          │
│ ((A+B)*C-(D-E))^(F+G)│ ^ - * + A B C     │ A B + C * D E - -  │
│                      │   - D E + F G      │   F G + ^          │
└──────────────────────┴────────────────────┴────────────────────┘
```

---

## Why Postfix and Prefix Matter

```
┌───────────────────────────────────────────────────────────┐
│                 ADVANTAGES                                │
│                                                           │
│  1. NO PARENTHESES needed                                 │
│     → Simpler parsing                                     │
│                                                           │
│  2. NO PRECEDENCE rules needed                            │
│     → Order of operations is explicit in the notation    │
│                                                           │
│  3. EASY EVALUATION with a stack                          │
│     → Postfix: left-to-right scan with stack             │
│     → Prefix:  right-to-left scan with stack             │
│                                                           │
│  4. Used in REAL SYSTEMS                                  │
│     → Calculators (HP RPN calculators)                   │
│     → Compilers (intermediate code generation)           │
│     → PostScript language                                │
│     → Forth programming language                         │
│     → Stack-based virtual machines (JVM bytecode)        │
│                                                           │
│  5. UNAMBIGUOUS                                           │
│     → Each expression has exactly one interpretation     │
└───────────────────────────────────────────────────────────┘
```

---

## Manual Conversion: Infix to Postfix (Intuitive Method)

```
Step 1: Fully parenthesize the expression
Step 2: Move each operator to replace its closing parenthesis
Step 3: Remove all parentheses

Example: A + B * C

Step 1: (A + (B * C))
                  ↓
Step 2: (A  (B C *) +)
              ↓
Step 3: A B C * +

Example: (A + B) * (C - D)

Step 1: ((A + B) * (C - D))
                          ↓
Step 2: ((A B +) (C D -) *)
                   ↓
Step 3: A B + C D - *
```

---

## Manual Conversion: Infix to Prefix (Intuitive Method)

```
Step 1: Fully parenthesize the expression
Step 2: Move each operator to replace its opening parenthesis
Step 3: Remove all parentheses

Example: A + B * C

Step 1: (A + (B * C))
         ↓
Step 2: (+ A (* B C))
         ↓
Step 3: + A * B C

Example: (A + B) * (C - D)

Step 1: ((A + B) * (C - D))
         ↓
Step 2: (* (+ A B) (- C D))
         ↓
Step 3: * + A B - C D
```

---

## Expression Tree Connection

All three notations are different **traversals** of the same expression tree:

```
Expression: (A + B) * (C - D)

Expression Tree:
            *
           / \
          +   -
         / \ / \
        A  B C  D

Inorder   (Infix):    A + B * C - D  → needs parentheses
Preorder  (Prefix):   * + A B - C D
Postorder (Postfix):  A B + C D - *

┌──────────────┬────────────────────┐
│ Traversal    │ Notation           │
├──────────────┼────────────────────┤
│ Inorder      │ Infix              │
│ Preorder     │ Prefix (Polish)    │
│ Postorder    │ Postfix (RPN)      │
└──────────────┴────────────────────┘
```

---

## Summary Table

| Aspect | Infix | Prefix | Postfix |
|--------|-------|--------|---------|
| **Operator Position** | Between operands | Before operands | After operands |
| **Also Called** | Standard | Polish Notation | RPN |
| **Parentheses** | Required | Not needed | Not needed |
| **Precedence Rules** | Required | Not needed | Not needed |
| **Evaluation** | Complex | Stack (R→L scan) | Stack (L→R scan) |
| **Human Readable** | Most natural | Less intuitive | Less intuitive |
| **Machine Friendly** | Least | Very | Very |
| **Tree Traversal** | Inorder | Preorder | Postorder |

---

## Quick Revision Questions

1. **Why is infix notation ambiguous without precedence rules?**
   > Because `2 + 3 * 4` could mean `(2+3)*4 = 20` or `2+(3*4) = 14`. Precedence rules resolve this ambiguity.

2. **What is the key advantage of postfix and prefix over infix?**
   > They require no parentheses and no precedence rules — the order of operations is inherent in the notation.

3. **How does postfix notation relate to tree traversal?**
   > Postfix corresponds to postorder traversal of the expression tree; prefix corresponds to preorder.

4. **Convert `A * (B + C)` to postfix and prefix.**
   > Postfix: `A B C + *` | Prefix: `* A + B C`

5. **Why do compilers convert infix to postfix?**
   > Postfix is easy to evaluate with a stack (simple left-to-right scan), making code generation straightforward.

6. **What real-world calculators use RPN?**
   > HP calculators (HP-12C, HP-48 series) use Reverse Polish Notation for input.

---

[← Previous: Stack Using Queues](../03-Basic-Stack-Problems/06-stack-using-queues.md) | [Next: Infix to Postfix Conversion →](02-infix-to-postfix.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)
