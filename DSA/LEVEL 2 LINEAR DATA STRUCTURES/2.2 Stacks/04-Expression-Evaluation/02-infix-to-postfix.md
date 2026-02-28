# Chapter 2: Infix to Postfix Conversion

[← Previous: Notation Types](01-notation-types.md) | [Next: Infix to Prefix Conversion →](03-infix-to-prefix.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)

---

## Overview

Converting infix expressions to postfix (Reverse Polish Notation) is one of the most important applications of stacks. This algorithm, developed by **Edsger Dijkstra** (the **Shunting-Yard Algorithm**), is the backbone of expression parsing in compilers and calculators.

---

## Algorithm: Shunting-Yard

### Key Rules

```
┌──────────────────────────────────────────────────────────┐
│              SHUNTING-YARD RULES                         │
│                                                          │
│  Scan infix expression LEFT to RIGHT:                    │
│                                                          │
│  1. OPERAND → Add directly to output                     │
│                                                          │
│  2. '(' → Push to stack                                  │
│                                                          │
│  3. ')' → Pop and output until '(' found, discard '('   │
│                                                          │
│  4. OPERATOR →                                           │
│     While stack top has HIGHER or EQUAL precedence*:     │
│       Pop and add to output                              │
│     Push current operator onto stack                     │
│                                                          │
│  5. END → Pop all remaining operators to output          │
│                                                          │
│  *For right-associative (^): only pop if HIGHER          │
└──────────────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION infixToPostfix(expression):
    output ← empty string
    stack ← empty stack
    
    FOR each token in expression:
        IF token is OPERAND:
            output ← output + token
        
        ELSE IF token is '(':
            stack.push(token)
        
        ELSE IF token is ')':
            WHILE stack.top() ≠ '(':
                output ← output + stack.pop()
            stack.pop()    // Remove '(' from stack
        
        ELSE IF token is OPERATOR:
            WHILE stack is NOT empty
                  AND stack.top() ≠ '('
                  AND precedence(stack.top()) >= precedence(token):
                // For ^: use > instead of >=
                output ← output + stack.pop()
            stack.push(token)
    
    // Pop remaining operators
    WHILE stack is NOT empty:
        output ← output + stack.pop()
    
    RETURN output
```

### Precedence Function

```
FUNCTION precedence(op):
    IF op is '+' or '-':  RETURN 1
    IF op is '*' or '/':  RETURN 2
    IF op is '^':         RETURN 3
    RETURN 0
```

---

## Step-by-Step Traces

### Trace 1: `A + B * C`

```
Expected: A B C * +

┌──────┬───────┬────────────┬──────────┬────────────────────┐
│ Step │ Token │ Action     │ Stack    │ Output             │
├──────┼───────┼────────────┼──────────┼────────────────────┤
│  1   │  A    │ Operand    │          │ A                  │
│  2   │  +    │ Push       │ +        │ A                  │
│  3   │  B    │ Operand    │ +        │ A B                │
│  4   │  *    │ * > + Push │ + *      │ A B                │
│  5   │  C    │ Operand    │ + *      │ A B C              │
│  6   │ END   │ Pop all    │          │ A B C * +          │
└──────┴───────┴────────────┴──────────┴────────────────────┘

Step 4 detail: * has precedence 2, + has precedence 1
  2 >= 1? No (we check stack top + vs current *)
  Wait — we check if stack top >= current:
  prec(+)=1 >= prec(*)=2? NO → don't pop, just push *

Result: A B C * +  ✓
```

### Trace 2: `(A + B) * C`

```
Expected: A B + C *

┌──────┬───────┬────────────────┬──────────┬────────────────┐
│ Step │ Token │ Action         │ Stack    │ Output         │
├──────┼───────┼────────────────┼──────────┼────────────────┤
│  1   │  (    │ Push           │ (        │                │
│  2   │  A    │ Operand        │ (        │ A              │
│  3   │  +    │ Push (after () │ ( +      │ A              │
│  4   │  B    │ Operand        │ ( +      │ A B            │
│  5   │  )    │ Pop until (    │          │ A B +          │
│  6   │  *    │ Push           │ *        │ A B +          │
│  7   │  C    │ Operand        │ *        │ A B + C        │
│  8   │ END   │ Pop all        │          │ A B + C *      │
└──────┴───────┴────────────────┴──────────┴────────────────┘

Result: A B + C *  ✓
```

### Trace 3: `A * B + C * D`

```
Expected: A B * C D * +

┌──────┬───────┬─────────────────────┬──────────┬──────────────┐
│ Step │ Token │ Action              │ Stack    │ Output       │
├──────┼───────┼─────────────────────┼──────────┼──────────────┤
│  1   │  A    │ Operand             │          │ A            │
│  2   │  *    │ Push                │ *        │ A            │
│  3   │  B    │ Operand             │ *        │ A B          │
│  4   │  +    │ prec(*)≥prec(+)?    │          │ A B *        │
│      │       │ 2≥1 YES→pop *, push │ +        │ A B *        │
│  5   │  C    │ Operand             │ +        │ A B * C      │
│  6   │  *    │ prec(+)≥prec(*)?    │ + *      │ A B * C      │
│      │       │ 1≥2 NO→push         │          │              │
│  7   │  D    │ Operand             │ + *      │ A B * C D    │
│  8   │ END   │ Pop all             │          │ A B * C D * +│
└──────┴───────┴─────────────────────┴──────────┴──────────────┘

Result: A B * C D * +  ✓
```

### Trace 4: `(A + B) * (C - D)`

```
Expected: A B + C D - *

┌──────┬───────┬──────────────────┬──────────┬──────────────────┐
│ Step │ Token │ Action           │ Stack    │ Output           │
├──────┼───────┼──────────────────┼──────────┼──────────────────┤
│  1   │  (    │ Push             │ (        │                  │
│  2   │  A    │ Operand          │ (        │ A                │
│  3   │  +    │ Push             │ ( +      │ A                │
│  4   │  B    │ Operand          │ ( +      │ A B              │
│  5   │  )    │ Pop until (      │          │ A B +            │
│  6   │  *    │ Push             │ *        │ A B +            │
│  7   │  (    │ Push             │ * (      │ A B +            │
│  8   │  C    │ Operand          │ * (      │ A B + C          │
│  9   │  -    │ Push             │ * ( -    │ A B + C          │
│ 10   │  D    │ Operand          │ * ( -    │ A B + C D        │
│ 11   │  )    │ Pop until (      │ *        │ A B + C D -      │
│ 12   │ END   │ Pop all          │          │ A B + C D - *    │
└──────┴───────┴──────────────────┴──────────┴──────────────────┘

Result: A B + C D - *  ✓
```

### Trace 5: `A ^ B ^ C` (Right-Associative)

```
Expected: A B C ^ ^  (right-to-left: A ^ (B ^ C))

┌──────┬───────┬──────────────────────┬──────────┬────────────┐
│ Step │ Token │ Action               │ Stack    │ Output     │
├──────┼───────┼──────────────────────┼──────────┼────────────┤
│  1   │  A    │ Operand              │          │ A          │
│  2   │  ^    │ Push                 │ ^        │ A          │
│  3   │  B    │ Operand              │ ^        │ A B        │
│  4   │  ^    │ prec(^) > prec(^)?   │ ^ ^      │ A B        │
│      │       │ 3 > 3? NO (strict >) │          │            │
│      │       │ Push (don't pop)     │          │            │
│  5   │  C    │ Operand              │ ^ ^      │ A B C      │
│  6   │ END   │ Pop all              │          │ A B C ^ ^  │
└──────┴───────┴──────────────────────┴──────────┴────────────┘

Key: For ^ we use STRICT > (not >=) because it's right-associative.
This ensures A ^ B ^ C = A ^ (B ^ C), not (A ^ B) ^ C.

Result: A B C ^ ^  ✓
```

---

## Visual Explanation

```
Infix: A + B * C - D

Think of the stack as a HOLDING AREA for operators:

  Input Stream:    A   +   B   *   C   -   D
                   ↓   ↓   ↓   ↓   ↓   ↓   ↓
  
  A → output directly
  + → hold in stack (waiting to see what comes next)
  B → output directly  
  * → higher precedence than +, hold in stack ON TOP of +
  C → output directly
  - → same/lower precedence than *: pop * to output
      same precedence as +: pop + to output
      push - to stack
  D → output directly
  END → pop - to output
  
  Output sequence: A B C * + D -
  
  Stack acts as a "waiting room" where operators wait until
  their right operand is complete.
```

---

## Handling Edge Cases

### Multi-digit Numbers

```
Input: "12 + 34 * 5"

Tokenize first: ["12", "+", "34", "*", "5"]
Then apply the algorithm on tokens, not individual characters.

Result: 12 34 5 * +
```

### Unary Operators

```
Input: "-A + B"  (unary minus)

Strategy: Convert unary minus to (0 - A)
  → "(0 - A) + B"
  → "0 A - B +"

Or use a special symbol like ~ for unary minus:
  → "~A + B" → "A ~ B +"
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Each token is processed once; each operator is pushed/popped at most once |
| **Space** | O(n) | Stack may hold up to n operators in worst case |

---

## Common Mistakes

```
┌──────────────────────────────────────────────────────┐
│  1. Using >= for right-associative operators (^)     │
│     → Must use STRICT > for ^                        │
│                                                      │
│  2. Forgetting to pop remaining operators at end     │
│     → Operators left on stack = missing from output  │
│                                                      │
│  3. Not discarding '(' after matching ')'            │
│     → '(' should never appear in output              │
│                                                      │
│  4. Comparing with '(' in precedence check           │
│     → Always stop popping at '('                     │
│                                                      │
│  5. Wrong precedence values                          │
│     → ^ > */ > +-                                    │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Algorithm** | Shunting-Yard (Dijkstra) |
| **Input** | Infix expression |
| **Output** | Postfix expression |
| **Data Structure** | Stack for operators |
| **Time** | O(n) |
| **Space** | O(n) |
| **Key Rule** | Pop higher/equal precedence before pushing |
| **Exception** | Right-associative: pop only strictly higher |

---

## Quick Revision Questions

1. **What happens when you encounter an operand?**
   > Add it directly to the output. Operands never go on the stack.

2. **When do you pop operators from the stack?**
   > When the incoming operator has lower or equal precedence than the stack top (for left-associative), or when a `)` is encountered.

3. **Why is `^` handled differently from `+`, `-`, `*`, `/`?**
   > Because `^` is right-associative. `A^B^C` means `A^(B^C)`, so we use strict `>` instead of `>=` to avoid premature popping.

4. **What role does `(` play on the stack?**
   > It acts as a barrier — operators above it get popped when `)` arrives, but `(` itself is never output; it's just discarded.

5. **Convert `A + B * C ^ D - E` to postfix.**
   > `A B C D ^ * + E -`

---

[← Previous: Notation Types](01-notation-types.md) | [Next: Infix to Prefix Conversion →](03-infix-to-prefix.md) | [↑ Back to Unit 4](../README.md#unit-4-expression-evaluation) | [🏠 Home](../README.md)
