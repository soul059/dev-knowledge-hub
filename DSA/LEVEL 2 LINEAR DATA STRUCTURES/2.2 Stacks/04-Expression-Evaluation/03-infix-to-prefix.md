# Chapter 3: Infix to Prefix Conversion

[← Previous: Infix to Postfix](02-infix-to-postfix.md) | [Next: Postfix Evaluation →](04-postfix-evaluation.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)

---

## Overview

Converting infix to prefix (Polish Notation) is the mirror of infix-to-postfix conversion. The key difference is that we process the expression **from right to left** and **reverse** the result, with swapped parenthesis handling.

---

## Method 1: Reverse-and-Reuse Algorithm

The most elegant approach reuses the infix-to-postfix algorithm:

```
┌──────────────────────────────────────────────────┐
│          INFIX TO PREFIX: 3 STEPS                │
│                                                  │
│  1. REVERSE the infix expression                 │
│     (swap '(' ↔ ')' during reversal)            │
│                                                  │
│  2. Apply INFIX-TO-POSTFIX algorithm             │
│     (use strict > for ALL operators)             │
│                                                  │
│  3. REVERSE the result                           │
│                                                  │
│  Done! You have the prefix expression.           │
└──────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION infixToPrefix(expression):
    // Step 1: Reverse infix, swapping parentheses
    reversed ← ""
    FOR i = length(expression) - 1 DOWNTO 0:
        IF expression[i] == '(':
            reversed ← reversed + ')'
        ELSE IF expression[i] == ')':
            reversed ← reversed + '('
        ELSE:
            reversed ← reversed + expression[i]
    
    // Step 2: Apply modified infix-to-postfix
    //   Use strict > for ALL operators (not >=)
    postfix ← infixToPostfix_Modified(reversed)
    
    // Step 3: Reverse the result
    prefix ← reverse(postfix)
    
    RETURN prefix
```

---

## Step-by-Step Trace

### Trace 1: `(A + B) * C`

```
Expected prefix: * + A B C

Step 1: Reverse infix (swap parens)
  Original: ( A + B ) * C
  Reversed: C * ) B + A (
  Swap ↔:   C * ( B + A )

Step 2: Apply infix-to-postfix on "C * (B + A)"
  ┌──────┬───────┬──────────────────┬──────────┬─────────┐
  │ Step │ Token │ Action           │ Stack    │ Output  │
  ├──────┼───────┼──────────────────┼──────────┼─────────┤
  │  1   │  C    │ Operand          │          │ C       │
  │  2   │  *    │ Push             │ *        │ C       │
  │  3   │  (    │ Push             │ * (      │ C       │
  │  4   │  B    │ Operand          │ * (      │ C B     │
  │  5   │  +    │ Push             │ * ( +    │ C B     │
  │  6   │  A    │ Operand          │ * ( +    │ C B A   │
  │  7   │  )    │ Pop until (      │ *        │ C B A + │
  │  8   │ END   │ Pop all          │          │ C B A + *│
  └──────┴───────┴──────────────────┴──────────┴─────────┘
  
  Postfix of reversed: C B A + *

Step 3: Reverse the result
  C B A + *  →  * + A B C

Result: * + A B C  ✓
```

### Trace 2: `A + B * C`

```
Expected prefix: + A * B C

Step 1: Reverse (no parens to swap)
  "A + B * C" → "C * B + A"

Step 2: Infix-to-postfix on "C * B + A"
  ┌──────┬───────┬─────────────────────┬──────────┬───────────┐
  │ Step │ Token │ Action              │ Stack    │ Output    │
  ├──────┼───────┼─────────────────────┼──────────┼───────────┤
  │  1   │  C    │ Operand             │          │ C         │
  │  2   │  *    │ Push                │ *        │ C         │
  │  3   │  B    │ Operand             │ *        │ C B       │
  │  4   │  +    │ prec(*) > prec(+)?  │ +        │ C B *     │
  │      │       │ YES → pop *, push + │          │           │
  │  5   │  A    │ Operand             │ +        │ C B * A   │
  │  6   │ END   │ Pop all             │          │ C B * A + │
  └──────┴───────┴─────────────────────┴──────────┴───────────┘

  Note: Use STRICT > for the modified algorithm.

Step 3: Reverse
  C B * A + → + A * B C

Result: + A * B C  ✓
```

### Trace 3: `(A + B) * (C - D)`

```
Expected prefix: * + A B - C D

Step 1: Reverse with paren swap
  "(A + B) * (C - D)" → ")D - C( * )B + A("
  Swap parens: "(D - C) * (B + A)"

Step 2: Infix-to-postfix on "(D - C) * (B + A)"
  ┌──────┬───────┬──────────────────┬──────────┬──────────────┐
  │ Step │ Token │ Action           │ Stack    │ Output       │
  ├──────┼───────┼──────────────────┼──────────┼──────────────┤
  │  1   │  (    │ Push             │ (        │              │
  │  2   │  D    │ Operand          │ (        │ D            │
  │  3   │  -    │ Push             │ ( -      │ D            │
  │  4   │  C    │ Operand          │ ( -      │ D C          │
  │  5   │  )    │ Pop until (      │          │ D C -        │
  │  6   │  *    │ Push             │ *        │ D C -        │
  │  7   │  (    │ Push             │ * (      │ D C -        │
  │  8   │  B    │ Operand          │ * (      │ D C - B      │
  │  9   │  +    │ Push             │ * ( +    │ D C - B      │
  │ 10   │  A    │ Operand          │ * ( +    │ D C - B A    │
  │ 11   │  )    │ Pop until (      │ *        │ D C - B A +  │
  │ 12   │ END   │ Pop all          │          │ D C - B A + *│
  └──────┴───────┴──────────────────┴──────────┴──────────────┘

Step 3: Reverse
  D C - B A + *  →  * + A B - C D

Result: * + A B - C D  ✓
```

---

## Method 2: Direct Stack Algorithm (Right-to-Left Scan)

```
FUNCTION infixToPrefix_Direct(expression):
    output ← empty list
    stack ← empty stack
    
    // Scan RIGHT to LEFT
    FOR i = length(expression) - 1 DOWNTO 0:
        token ← expression[i]
        
        IF token is OPERAND:
            output.prepend(token)       // Add to front
        
        ELSE IF token is ')':
            stack.push(token)
        
        ELSE IF token is '(':
            WHILE stack.top() ≠ ')':
                output.prepend(stack.pop())
            stack.pop()    // Remove ')'
        
        ELSE IF token is OPERATOR:
            WHILE stack is NOT empty
                  AND stack.top() ≠ ')'
                  AND precedence(stack.top()) > precedence(token):
                output.prepend(stack.pop())
            stack.push(token)
    
    WHILE stack is NOT empty:
        output.prepend(stack.pop())
    
    RETURN output as string
```

---

## Key Differences: Postfix vs Prefix Conversion

```
┌────────────────────────┬──────────────────┬──────────────────┐
│ Aspect                 │ Infix→Postfix    │ Infix→Prefix     │
├────────────────────────┼──────────────────┼──────────────────┤
│ Scan direction         │ Left → Right     │ Right → Left     │
│ Output placement       │ Append to end    │ Prepend to front │
│ '(' on stack           │ Push (           │ Push )            │
│ ')' triggers           │ Pop until (      │ Pop until )       │
│ Precedence comparison  │ >= (left-assoc)  │ > (strict)        │
│ Final step             │ Done             │ Reverse (Method1) │
└────────────────────────┴──────────────────┴──────────────────┘
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Two reversals O(n) + conversion O(n) |
| **Space** | O(n) | Stack + output storage |

---

## More Conversion Examples

```
┌──────────────────────────┬─────────────────────┐
│ Infix                    │ Prefix              │
├──────────────────────────┼─────────────────────┤
│ A + B                    │ + A B               │
│ A + B - C                │ - + A B C           │
│ A * B + C * D            │ + * A B * C D       │
│ A + B * C ^ D            │ + A * B ^ C D       │
│ ((A + B) * C) - D        │ - * + A B C D       │
│ A * (B + C) / D          │ / * A + B C D       │
└──────────────────────────┴─────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Goal** | Convert infix to prefix (Polish notation) |
| **Best Method** | Reverse → Infix-to-Postfix → Reverse |
| **Time** | O(n) |
| **Space** | O(n) |
| **Key Difference** | Use strict `>` instead of `>=` for precedence |
| **Parentheses** | Swap `(` ↔ `)` during reversal |

---

## Quick Revision Questions

1. **What are the three steps to convert infix to prefix using the reuse method?**
   > (1) Reverse the expression (swapping parentheses), (2) apply infix-to-postfix, (3) reverse the result.

2. **Why do we swap parentheses during the first reversal?**
   > Because reversing changes the nesting direction. What was an opening becomes a closing and vice versa.

3. **Why use strict `>` instead of `>=` in the modified algorithm?**
   > Using strict `>` ensures left-to-right associativity is preserved after the double reversal.

4. **Convert `A * (B + C) - D` to prefix.**
   > `- * A + B C D`

5. **Which method is simpler to implement: Method 1 or Method 2?**
   > Method 1 (reverse-and-reuse) is simpler because you can reuse your existing infix-to-postfix code with minor modifications.

---

[← Previous: Infix to Postfix](02-infix-to-postfix.md) | [Next: Postfix Evaluation →](04-postfix-evaluation.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)
